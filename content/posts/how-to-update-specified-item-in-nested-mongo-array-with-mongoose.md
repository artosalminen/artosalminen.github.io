---
title: "How to update a specified item in a nested array with Mongoose"
date: 2023-02-11
series: ["MongoDB", "Mongoose", "coding"]
weight: 1
tags: ["MongoDB", "Mongoose", "coding"]
author: "Arto Salminen"
---

In this blog post, we'll go over how to update a specified item in a nested array inside a Mongoose document.

##Setting up the environment

First, let's set up the environment by creating a Mongoose schema and model. Here's an example schema that has a nested array of items:

```javascript
const mongoose = require('mongoose');

const schema = new mongoose.Schema({
  items: [{
    name: String,
    quantity: Number
  }]
});

const Model = mongoose.model('Model', schema);
```

##Updating a specified item in a nested array

To update a specified item in a nested array, we can use the MongoDB `$` operator along with the `$set` operator. The `$` operator is used to specify which item in the array should be updated.

Here's an example of how to update the quantity field of the first item in the items array with the name equal to `'item1'`:

```javascript
Model.updateOne(
  {'items.name': 'item1'},
  {'$set': {'items.$.quantity': 10}},
  (err, result) => {
    if (err) {
      console.error(err);
    } else {
      console.log(result);
    }
  }
);
```

In this example, the `Model.updateOne()` method is used to update the first document that matches the query `{'items.name': 'item1'}`. The `$` operator is used in the `$set` operator to specify that the update should be applied to the first item in the items array that matches the query condition. In this case, the quantity field of the item with name equal to `'item1'` will be updated to 10.

Note that the `updateOne()` method only updates the first document that matches the query. If there are multiple items in the items array with the name equal to `'item1'`, only the first one will be updated. To update all items that match the query, you can use the `updateMany()` method instead.
