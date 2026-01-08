---
title: "MongoDB Data Modelling"
date: 2022-09-22T12:54:31+05:00
draft: false

description: "Normalize or De-normalize your data according to the need. Learn mongoose advance data modelling techniques that will increase your database efficiency and performance."

resources:
  - name: "featured-image"
    src: "featured-image.png"

tags: ["underthehood"]
categories: ["DATABASE"]
theme: "full"
images: ["https://theundersurfers.netlify.app/mongoose-data-modelling/featured-image.png"]
seo:
  images: ["https://theundersurfers.netlify.app/mongoose-data-modelling/featured-image.png"]
---

<!--more-->

# Mongoose/Mongo Advance Concepts

1. Relationship types
2. Referencing/Normalization vs Embedding/Denormalization
3. Embedding or Referencing Documents
4. Types of referencing
   - Parent Referencing
   - Child Referencing
   - Two-Way Referencing

## Relationship Types:

### 1:1

_In systems analysis, a one-to-one relationship is a type of cardinality that refers to the relationship between two entities A and B in which one element of A may only be linked to one element of B, and vice versa. In mathematical terms, there exists a bijective function from A to B_

### 1:MANY

_In systems analysis, a one-to-many relationship is a type of cardinality that refers to the relationship between two entities A and B in which an element of A may be linked to many elements of B, but a member of B is linked to only one element of A. For instance, think of A as books, and B as pages._

### MANY:MANY

_In systems analysis, a many-to-many relationship is a type of cardinality that refers to the relationship between two entities, say, A and B, where A may contain a parent instance for which there are many children in B and vice versa. For example, think of A as Authors, and B as Books_

## Referencing VS Embedding

### Referenced / Normalized

When you normalize your data, you are dividing your data into multiple collections with references between those collections. Each piece of data will be in a collection, but multiple documents will reference it

{{< figure src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/Advance%20Mongoose%20MongoDB%209fd8fe5ee9ad4e9d82076468970b0da3_Untitled_1663832182353..png?alt=media&token=cf1defb9-5e28-421d-8f47-0178a4d2b1c8" >}}

😃 Easy to query each document on its own.

😞 Need 2 queries to get data from the referenced document.

### Embedding / Denormalization

Denormalization allows you to avoid some application-level joins, at the expense of having more complex and expensive updates.

{{< figure src="https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/Advance%20Mongoose%20MongoDB%209fd8fe5ee9ad4e9d82076468970b0da3_Untitled%201_1663832184454..png?alt=media&token=b3257e12-ee5d-44a5-a766-66db123db196" >}}

😃 We can get all info in one query.

😞 Cannot query the embedded document on its own.

## When to Embed & When to Refer?

Before finalizing your decision to embed or refer to the document first check these three criteria.

1. Relationship Types

   For **many:many** or **one:many** types Referencing is preferred.

2. Data Access Patterns

   If data is updated a lot or the **read/write** ratio is low go with Referencing else go with embedding

3. Data Closeness

   How much is data related? For example how many movies & images are associated with each other? If images are frequently queried with movies go with reference.

## Types of Referencing

### Child Referencing

Main Parent document. You can see the object IDs of the child documents are referred to in the parent documents. Use this referencing in case of `1:FEW`

```json
{
	"_id" : ObjectID('23'),
	"app": "My Movie Database",
	"logs": [
	ObjectID('1'),
  ObjectID('2'),
	// ... Millions of ObjectID
	ObjectID('28273927')
    ]
}
```

_Child Document._

```json
{
    "_id": ObjectID('1'),
    "type": "error",
    "timestamp": 1412184926
}
```

_Child Document._

```json
{
     "_id " : ObjectID('28273927'),
     "type": "error",
     "timestamp": 1412844672
}
```

😟 But there is still one problem. If there are millions of child objects. Each one has to be referred to in the parent object causing an increase in the size of the parent document. The maximum BSON document size is **16 megabytes.**

### Parent Referencing

Now instead of saving the object ids of all the child documents in the parent, we can save the `objectId` of the parent document in each child. That will solve the problem of increasing size. If you are using **`one:many` or `one:ton`** you can use this referencing.

```json
// PARENT
{
    "_id" : ObjectID('23'),
    "app": "My Movie Database"
}

// CHILD
{
    "_id": ObjectID('1'),
    "app": ObjectID('23'),
    "type": "error",
    "timestamp": 1412184926
}

// CHILD
{
    "_id": ObjectID('28273927'),
    "app": ObjectID('23'),
    "type": "error",
    "timestamp": 1412844672
}
```

### TWO-WAY Referencing

So if there is a case you are using `many:many` relationship types you can prefer this type of reference.

```json
// PARENT
{
    "_id": ObjectID('23'),
    "title": "Interstellar",
    "release Year: 2014,
    "actors": [
        ObjectID('67'),
        // ... and many more
    ]

}

// CHILD
{
    "_id": ObjectID('67'),
    "name": "Matthew McConaughey",
    "age": 50,
    "movies": [
        ObjectID('23'),
        // ... and many more
    ]
}
```

## Populating Documents

One more thing you need to take care of after referencing your documents is to populate them. To get all the entities it is better to populate the document first.

- Population

Populating when using child referencing.

```js
findOne({ title: 'Casino Royale' }).populate('author').
```

- Virtual Populate
  Populating parent when using parent referencing

  ```js
  // Specifying a virtual with a `ref` property is how you enable virtual
  // population
  AuthorSchema.virtual("posts", {
    ref: "BlogPost",
    localField: "_id",
    foreignField: "author",
  });
  ```

  For a more detailed explanation refer to the documentation
  [https://mongoosejs.com/docs/populate.html#populate-virtuals](https://mongoosejs.com/docs/populate.html#populate-virtuals)

  # Acknowledgment

  - This node.js course helped me implement the above code [node.js bootcamp](https://www.udemy.com/course/nodejs-express-mongodb-bootcamp/)
