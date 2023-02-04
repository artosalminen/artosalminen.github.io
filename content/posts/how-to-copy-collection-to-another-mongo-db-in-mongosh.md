---
title: "How to Copy a Collection of Documents to Another Database with mongosh"
date: 2023-02-04T09:32:28+02:00
series: ["MongoDB"]
weight: 1
tags: ["MongoDB", "Mongosh"]
author: "Arto Salminen"
---

If you need to copy a collection of documents from one Mongo collection to another, you can
use the `mongodump` and `mongorestore`. But if the source database is in the same instance
as the target, then you may be able to take a shortcut with `mongosh`.

Here is the script to use:

```
db.<source collection>.find().forEach(function(d){ db.getSiblingDB('<dest db>')['<dest collection>'].insert(d); });
```

For example, if you are copying all documents from current databases `cars` collection to a database
called `another-db`'s collection `vehicles`:

```
db.cars.find().forEach(function(d){ db.getSiblingDB('another-db')['vehicles'].insert(d); });
```

Of course, you can add filters to the `find` method if you want to copy only subset of documents.