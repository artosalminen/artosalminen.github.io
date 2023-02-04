---
title: "How to merge a nested list in Mongo document with another found in adjacent collection"
date: 2023-02-05T08:04:11+02:00
series: ["MongoDB", "Coding"]
weight: 1
tags: ["MongoDB", "aggregate", "Coding"]
author: "Arto Salminen"
---

Let's consider a situation where you have two collections, `houses` and `people`. Each house has a collection
of key holders, which link to the persons collection with their IDs. Key holders list also holds information
when the key was given for the identified person.

In bson, the situation in the database looks like this:

```json
{
  "houses": [
    {
      "_id": ObjectId("5fba17c1c4566e57fafdcd7e"),
      "address": "Main street 1",
      "keyHolders": [
        {
          "keyDelivered": "2022-02-02T02:02:02",
          "personId": "5fbb5ab778045a985690b5fc"
        },
        {
          "keyDelivered": "2021-01-01T01:01:01",
          "personId": "5fbb5ab778045a985690b5fd"
        }
      ]
    },
    {
      "_id": ObjectId("5fba17c1c4566e57fafdcd7f"),
      "address": "Broadway 3",
      "keyHolders": [
        {
          "keyDelivered": "1993-03-03T03:03:03",
          "personId": "5fbb5ab778045a985690b5fc"
        }
      ]
    }
  ],
  "persons": [
    {
      "_id": ObjectId("5fbb5ab778045a985690b5fc"),
      "name": "Jack Bauer",
      
    },
    {
      "_id": ObjectId("5fbb5ab778045a985690b5fd"),
      "name": "James Bond",
      
    }
  ]
}
```

You can also do some mapping for the source list, for instance convert
foreign keys from strings to ObjectIds.

The aggregate pipeline to do the trick looks like this:

```json
[
  {
    $addFields: {
      mappedItems: {
        $map: {
          input: "$keyHolders",
          in: {
            $mergeObjects: [
              "$$this",
              {
                personId: {
                  $toObjectId: "$$this.personId"
                }
              }
            ]
          }
        }
      }
    }
  },
  {
    $lookup: {
      from: "persons",
      localField: "mappedItems.personId",
      foreignField: "_id",
      as: "itemsCollection"
    }
  },
  {
    $project: {
      address: 1,
      keyHolders: {
        $map: {
          input: "$mappedItems",
          as: "i",
          in: {
            $mergeObjects: [
              "$$i",
              {
                $first: {
                  $filter: {
                    input: "$itemsCollection",
                    cond: {
                      $eq: [
                        "$$this._id",
                        "$$i.personId"
                      ]
                    }
                  }
                }
              }
            ]
          }
        }
      }
    }
  }
]
```

And the result:

```json
[
  {
    "_id": ObjectId("5fba17c1c4566e57fafdcd7e"),
    "address": "Main street 1",
    "keyHolders": [
      {
        "_id": ObjectId("5fbb5ab778045a985690b5fc"),
        "keyDelivered": "2022-02-02T02:02:02",
        "name": "Jack Bauer",
        "personId": ObjectId("5fbb5ab778045a985690b5fc")
      },
      {
        "_id": ObjectId("5fbb5ab778045a985690b5fd"),
        "keyDelivered": "2021-01-01T01:01:01",
        "name": "James Bond",
        "personId": ObjectId("5fbb5ab778045a985690b5fd")
      }
    ]
  },
  {
    "_id": ObjectId("5fba17c1c4566e57fafdcd7f"),
    "address": "Broadway 3",
    "keyHolders": [
      {
        "_id": ObjectId("5fbb5ab778045a985690b5fc"),
        "keyDelivered": "1993-03-03T03:03:03",
        "name": "Jack Bauer",
        "personId": ObjectId("5fbb5ab778045a985690b5fc")
      }
    ]
  }
]
```

And here is a Mongo playground link for you: https://mongoplayground.net/p/3s0sFfp7uIY