---
title: "Increase Mongo querying efficiency using Indexes."
date: 2022-09-24T21:11:39+05:00
draft: false

description: "You can increase the speed of searching your document by using the idea of indexes in MongoDB. It will take some space but in the end, will provide you with great speed."

resources:
  - name: "featured-image"
    src: "featured-image.jpg"

tags: ["database", "mongodb", "mongoose", "indexes"]
categories: ["DATABASE"]
theme: "full"
---

<!--more-->

You can increase the speed of searching your document by using the idea of indexes in MongoDB. It will take some space but in the end, will provide you with great speed.

## What is an index?

Indexes support the efficient resolution of queries. Without indexes, MongoDB must scan every document of a collection to select those documents that match the query statement. This scan is highly inefficient and requires MongoDB to process a large volume of data.

_Indexes are special data structures, that store a small portion of the data set in an easy-to-traverse form. The index stores the value of a specific field or set of fields, ordered by the value of the field as specified in the index._

## Example

For example, MongoDB has indexed the `id` field by default and the benefit is that _it doesn’t have to go through each document to search for an id. Just go through the id index to get the specific document._

Open the MongoDB compass, open a collection, and go to the indexes tab. You will see that the `_id` field is already indexed there.

![Untitled](https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/Improve%20querying%20performance%20in%20mongo%201d443163a39d4aea8478b53427a29623_Untitled_1664030219682..png?alt=media&token=68e508c8-5611-4d52-a653-904f923d1353)

## Analyzing Performance

Let's try querying some documents. I have a `Tour` collection having some info about the tour name, duration, price, etc. Now only the `_id` field is indexed by the mongo default.

![Untitled](https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/Improve%20querying%20performance%20in%20mongo%201d443163a39d4aea8478b53427a29623_Untitled%201_1664030221052..png?alt=media&token=f5dc86de-eaed-497f-9ef6-d2b861fb2269)

I have an API attached to my database. I will try searching for some documents based on the price.

```js
{{URL}}api/v1/tours?price[lt]=1000
```

In MongoDB, the `[explain` command](https://docs.mongodb.com/manual/reference/method/cursor.explain/) tells the MongoDB server to return stats about how it executed a query, rather than the results of the query. [Mongoose queries](https://mongoosejs.com/docs/queries.html) have an `[explain()` function](https://mongoosejs.com/docs/api/query.html#query_Query-explain) that converts a query into an `explain()`
.

![Untitled](https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/Improve%20querying%20performance%20in%20mongo%201d443163a39d4aea8478b53427a29623_Untitled%202_1664030222512..png?alt=media&token=49bc991a-09e4-43f7-9c78-360499768876)

Here `query.explained()` method in mongoose gives us some statistics related to the query URL.

- As the result of the query 3 documents are returned
- But MongoDB has to go through the 9 documents to get these three document
- Because the **Price** field is not indexed in mongo Db, so it has to go through each document to price the document with a specific price.
- If there would thousands of documents then mongo have to go through all of them.
- So it is better to index a field if you often querying that field in your app.

![Untitled](https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/Improve%20querying%20performance%20in%20mongo%201d443163a39d4aea8478b53427a29623_Untitled%203_1664030223683..png?alt=media&token=625709ee-5028-4ffb-a312-390a37efa1f1)

After setting the index on the PRICE field you can see our performance increased. This time `totalDocsExamined` are 3 instead of 9.

## Compound Index

MongoDB supports _compound indexes,_ where a single index structure holds references to multiple fields within a collection's documents.

```js
// Creating Index
tourSchema.index({ price: 1 });

// Creating Compound Index
tourSchema.index({ price: 1, ratingsAverage: -1 });
```

You can also make a compound index like shown above. It is used when you are trying to query on the basis of two fields so you can mark them both as an index

```js
{{URL}}api/v1/tours?price[lt]=1000&ratingsAverage[gte]=4.
```

## Trade-Off

Now go to the MongoDB compass and examine the size of the documents and indexes.

![Untitled](https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/Improve%20querying%20performance%20in%20mongo%201d443163a39d4aea8478b53427a29623_Untitled%204_1664030224841..png?alt=media&token=c6cd3e05-72a1-4bea-90d2-607f624d1bc2)

After adding some indexes you can see that indexes are taking up a lot more space than the whole document alone so this is just a trade-off for getting a more efficient rereading speed.

![Untitled](https://firebasestorage.googleapis.com/v0/b/imagehosting-d913b.appspot.com/o/Improve%20querying%20performance%20in%20mongo%201d443163a39d4aea8478b53427a29623_Untitled%205_1664030225986..png?alt=media&token=e491675d-1f17-4676-8ab8-6f88f096c01e)

## Which field to index?

How do we decide which field we need to index? And why don't we set indexes on all the fields?

Well, we kind of used the strategy that I used to set the indexes on the price and on the average rating. So basically we need to carefully study the access patterns of our application in order to figure out which fields are queried the most and then set the indexes for these fields.

_For example, I don’t want to set an index here on the `group size` because I don't really believe that many people will query for that parameter in my app. and so I don't need to create an index there._

Because we really do not want to overdo it with indexes. So we don't want to blindly set indexes on all the fields and then hope for the best basically. And the reason for that is that each index uses resources. And also, each index needs to be updated each time that the underlying collection is updated. So if you have a collection with a high write-read ratio, so a collection that is mostly written to, then it would make absolutely no sense to create an index on any field in this collection because the cost of always updating the index and keeping it in memory outweighs the benefit of having the index in the first place if we rarely have searched, so have queries, for that collection.

So in summary, when deciding whether to index a certain field or not, we must kind of balance the frequency of queries using that exact field with the cost of maintaining this index, and also with the read-write pattern of the resource.

So that is the power of indexes. They really can make our read performance on databases much, much better. And so in your own applications, you should really never ignore them.

## Acknowledgment

- This node.js course helped me implement the above code [node.js bootcamp](https://www.udemy.com/course/nodejs-express-mongodb-bootcamp/)
