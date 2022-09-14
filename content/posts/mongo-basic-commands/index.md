---
title: "Mongo Basic Commands"
date: 2022-09-14T18:51:00+05:00
draft: false

description: "Create, Update & Querying MongoDB Commands"

resources:
  - name: "featured-image"
    src: "featured-image.webp"

tags: ["web", "database", "mongodb", "mongodb shell"]
categories: ["DATABASE"]
theme: "full"
---

<!--more-->

Following are some of the basic MongoDB commands to get started with mongo. I will show you how to create, update and query your first database.

## Creating Database

You can create a database using `use testdb`. Then you can add your documents using `insertOne` or `insertMany`. I have created a simple JSON file here with some sample data.

{{< figure src="https://cdn.hashnode.com/res/hashnode/image/upload/v1663157443585/rnX4qKITP.png" >}}

A sample template if you want to copy it.

```json
[
  {
    "_id": 1,
    "name": "mack",
    "age": 25,
    "email": "xyz@gmail.com",
    "interest": {
      "name": "painting",
      "type": "art"
    },
    "skills": ["mongodb", "mysql"]
  },

  {
    "_id": 2,
    "name": "jon",
    "age": 23,
    "email": "jon@gmail.com",
    "interest": {
      "name": "running",
      "type": "sports"
    },
    "skills": ["mongodb", "mysql"]
  },

  {
    "_id": 3,
    "name": "mark",
    "age": 34,
    "email": "jon@gmail.com",
    "interest": {
      "name": "robots",
      "type": "tech"
    },
    "skills": ["mongodb", "mysql"]
  },

  {
    "_id": 4,
    "name": "irfa",
    "age": 24,
    "email": "irfa@gmail.com",
    "interest": {
      "name": "robots",
      "type": "tech"
    },
    "skills": ["robotics", "mysql"]
  },

  {
    "_id": 5,
    "name": "henry",
    "age": 24,
    "email": "henry@gmail.com",
    "interest": {
      "name": "swimming",
      "type": "sports"
    },
    "skills": ["js", "mysql"]
  },

  {
    "_id": 6,
    "name": "jack",
    "age": 24,
    "email": "jack@gmail.com",
    "interest": {
      "name": "painting",
      "type": "art"
    },
    "skills": ["flask", "django"]
  }
]
```

Now adding these objects using `insertMany`

{{< figure src="https://cdn.hashnode.com/res/hashnode/image/upload/v1663158156547/15IptoiKs.jpeg">}}

I have added IDs manually but you don’t have to add them mongo will do it for you.

{{< figure src="https://cdn.hashnode.com/res/hashnode/image/upload/v1663157491200/ZVNaS0mdv.jpeg" >}}

You can see all the objects are been added with the desired IDs I specified. If you didn’t specify an id, don’t worry mongo will add the random ids for your documents.

## Updating Documents

There are different methods used in order to update a document depending on our needs. Following are some operators which will help you update the document

### $set

The [$set](https://www.mongodb.com/docs/manual/reference/operator/update/set/#mongodb-update-up.-set) operator replaces the value of a field with the specified value.

```js
db.student.updateOne(
  { _id: 1 },
  {
    $set: {
      name: "mikeTyson",
      email: "mikeTyson@gmail.com",
      interest: {
        name: "boxing",
        type: "sport",
      },
      skills: ["boxing", "fighting"],
    },
  }
);
```

This will set the existing fields in the document where id equals 1.

{{< figure src="https://cdn.hashnode.com/res/hashnode/image/upload/v1663157522551/bs7SasdOe.jpeg" >}}

You can then use `db.collection.find()` to see the result. You can see that the object has been updated now. Use `db.collection.find().pretty()` that will help you reading the object on the terminal easily.

### $addToSet

If you wanna add some new value to an array in the object use `addToSet` operator. The [$addToSet](https://www.mongodb.com/docs/manual/reference/operator/update/addToSet/#mongodb-update-up.-addToSet) operator adds a value to an array unless the value is already present, in which case [$addToSet](https://www.mongodb.com/docs/manual/reference/operator/update/addToSet/#mongodb-update-up.-addToSet)
does nothing to that array.

```js
db.student.updateOne(
  { _id: 1 },
  {
    $addToSet: {
      skills: "legend",
    },
  }
);
```

This will add a new element to our skills array.

{{< figure src="https://cdn.hashnode.com/res/hashnode/image/upload/v1663157545693/zWMq4CSpk.jpeg" >}}

See _`legend`_ has now been added to the array.

### $push

It helps push new values to the array in a single instance or many. The [$push](https://www.mongodb.com/docs/manual/reference/operator/update/push/#mongodb-update-up.-push)
operator appends a specified value to an array.

```js
db.student.updateOne(
  { _id: 1 },
  {
    $push: {
      skills: "legendry",
    },
  }
);
```

{{< figure src="https://cdn.hashnode.com/res/hashnode/image/upload/v1663157596267/ls1JiszID.jpeg" >}}

### $pull

push and pull operators make dealing with the arrays updates easy. The [$pull](https://www.mongodb.com/docs/manual/reference/operator/update/pull/#mongodb-update-up.-pull) operator removes from an existing array all instances of a value or values that match a specified condition.

```js
db.student.updateOne(
  { _id: 1 },
  {
    $pull: {
      skills: "legendry",
    },
  }
);
```

{{< figure src="https://cdn.hashnode.com/res/hashnode/image/upload/v1663158231904/fWmC9rQxB.jpeg" >}}

## Querying your Collection

Now you wanna extract information about a specific person/object 🤔. Don’t worry just go ahead and use these operators 😀.

### find()

Use this if you wanna find all the documents that match the condition.

```js
db.student.find({ "interest.type": "tech" }).pretty();
db.student.find({ age: { $lt: 25 } }).pretty();
```

{{< figure src="https://cdn.hashnode.com/res/hashnode/image/upload/v1663158278990/9_wi4D5O-.jpeg" >}}
{{< figure src="https://cdn.hashnode.com/res/hashnode/image/upload/v1663158287013/pQfnHNjqT.jpeg" >}}

### findOne()

It will return the first document that matches the condition specified.

```js
db.student.findOne({ age: { $lt: 25 } }).pretty();
```

{{< figure src="https://cdn.hashnode.com/res/hashnode/image/upload/v1663158297759/YDdRGmd1f.jpeg" >}}

You can see it just returned the first document having an age less than 25.

### Limiting using limit()

While querying your documents use `limit()` to specify how many matching documents to be returned.

```js
db.student
  .find({ age: { $lt: 25 } })
  .limit(2)
  .pretty();
```

{{< figure src="https://cdn.hashnode.com/res/hashnode/image/upload/v1663158311386/HeCPqsQi0.jpeg" >}}

### Sorting using sort()

It helps sort the resulting documents with respect to the specified field.

```js
db.student.find().sort({ age: -1 }).pretty();
```

It will sort the result with decreasing ages i.e. descending order.

{{< figure src="https://cdn.hashnode.com/res/hashnode/image/upload/v1663158328440/XtdX8TqpE.jpeg" >}}

```js
db.student.find().sort({ age: -1 }).pretty();
```

{{< figure src="https://cdn.hashnode.com/res/hashnode/image/upload/v1663158343166/QWF2O1OYa.jpeg" >}}

So that was it. Follow me for more 😀.
