---
title: "Write Express Controllers like a pro."
date: 2022-08-23T16:00:41+05:00
draft: false

description: "Building express controller like a pro. Use a controller handler module to make other controllers. "

resources:
  - name: "featured-image"
    src: "featured-image.webp"

tags: ["setup"]
categories: ["WEB"]
theme: "full"
---

<!--more-->

# Write express controllers like a pro

![https://images.pexels.com/photos/1637439/pexels-photo-1637439.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=1](https://images.pexels.com/photos/1637439/pexels-photo-1637439.jpeg?auto=compress&cs=tinysrgb&w=1260&h=750&dpr=1)

Yeah I know these are not express controllers they are Nintendo’s 😅. But If you are bored writing the express controllers the boring way I will share with you a better solution today.

## The old way

Let's suppose you have some endpoints related /cars, /bike, etc. And for each of their controller, you wanna define some middleware's related GET, UPDATE, PATCH, and DELETE requests. Let's say you are defining the update middleware first like bellow

```js
exports.updateBike = catchAsync(async (req, res, next) => {
  const bike = await Bike.findByIdAndUpdate(req.params.id, req.body, {
    new: true,
    runValidators: true,
  });

  if (!bike) {
    return next(new AppError("No bike found with that ID", 404));
  }

  res.status(200).json({
    status: "success",
    data: {
      bike,
    },
  });
});
```

{{< admonition type=tip title="Note" open=true >}}
Note that the `catchAysync` in the above function is just to wrap the function in `try-catch` block due to the use of `async-await`. And the `AppError` is just custom designed error to be passed to the global error controller by doing `next(error)`.
{{< /admonition >}}

```js
// catchAysnc Module

/**
 * making you async function to catch errors
 * without using try-catch explicitly in every function
*/
**module.exports = fn => {
  return (req, res, next) => {
    fn(req, res, next).catch(next);
  };
};**
```

And for the /cars endpoint you are repeating the same code.

```js
exports.updateCar = catchAsync(async (req, res, next) => {
  const car = await Car.findByIdAndUpdate(req.params.id, req.body, {
    new: true,
    runValidators: true,
  });

  if (!car) {
    return next(new AppError("No car found with that ID", 404));
  }

  res.status(200).json({
    status: "success",
    data: {
      car,
    },
  });
});
```

So when your app will be growing large it becomes difficult to write the same middleware code for the GET, UPDATE, PATCH, and DELETE functionality, for all the endpoints. There is a better way for defining your code.

## Handler Factory.js

You just need to define a controller module once and it will save you from rewriting your code. You can simply change the above two blocks of code to bellow

```js
exports.updateCar = factory.updateOne(Car);
```

Do the same with `bikeController` module

```js
exports.updateBike = factory.updateOne(Bike);
```

You can do the same with all the other CRUD operations as shown.

```js
exports.getAllCars = factory.getAll(Car);
exports.getCar = factory.getOne(Car);
exports.createCar = factory.createOne(Car);
exports.updateCar = factory.updateOne(Car);
exports.deleteCar = factory.deleteOne(Car);
```

See by just using the `factory` object we save our time and solve the boilerplate. You just need to implement this `hadlerFactory.js` module.

### Implementing Handler Factory

Implementing this module is similar like implementing the basic controller. You just have to define the controller as a general controller that can be used with your other controllers.

```js
exports.createOne = (Model) =>
  catchAsync(async (req, res, next) => {
    const doc = await Model.create(req.body);

    res.status(201).json({
      status: "success",
      data: {
        data: doc,
      },
    });
  });
```

You just have to pass the mongoose `Model` to the method and it will create a new object in the DB using that model. Similarly, you can write your whole handler.

```js
const catchAsync = require("./../utils/catchAsync");
const AppError = require("./../utils/appError");

exports.deleteOne = (Model) =>
  catchAsync(async (req, res, next) => {
    const doc = await Model.findByIdAndDelete(req.params.id);

    if (!doc) {
      return next(new AppError("No document found with that ID", 404));
    }

    res.status(204).json({
      status: "success",
      data: null,
    });
  });

exports.updateOne = (Model) =>
  catchAsync(async (req, res, next) => {
    const doc = await Model.findByIdAndUpdate(req.params.id, req.body, {
      new: true,
      runValidators: true,
    });

    if (!doc) {
      return next(new AppError("No document found with that ID", 404));
    }

    res.status(200).json({
      status: "success",
      data: {
        data: doc,
      },
    });
  });

exports.createOne = (Model) =>
  catchAsync(async (req, res, next) => {
    const doc = await Model.create(req.body);

    res.status(201).json({
      status: "success",
      data: {
        data: doc,
      },
    });
  });

exports.getOne = (Model, popOptions) =>
  catchAsync(async (req, res, next) => {
    let query = Model.findById(req.params.id);
    if (popOptions) query = query.populate(popOptions);
    const doc = await query;

    if (!doc) {
      return next(new AppError("No document found with that ID", 404));
    }

    res.status(200).json({
      status: "success",
      data: {
        data: doc,
      },
    });
  });
```

So now you can use this factory module while building your other controller.

```js
exports.getAllBikes = factory.getAll(Bike);
exports.getBike = factory.getOne(Bike);
exports.createBike = factory.createOne(Bike);
exports.updateBike = factory.updateOne(Bike);
exports.deleteBike = factory.deleteOne(Bike);
```

{{< admonition type=tip title="Note" open=true >}}
_💡 Note that you can design your own error like `AppError`. or simply pass some message string and throw an error. You can refer to the bellow code if you wanna implement the `AppError` module that makes passing errors to the express global error controller easy by just passing the error message and status code as a parameter i.e. `next(new AppError(”my error message”, 404))`._
{{< /admonition >}}

```js
// simply pass err-msg & status code as param to gen an err
class AppError extends Error {
  constructor(message, statusCode) {
    super(message);

    this.statusCode = statusCode;
    this.status = `${statusCode}`.startsWith("4") ? "fail" : "error";
    this.isOperational = true;

    Error.captureStackTrace(this, this.constructor);
  }
}

module.exports = AppError;
```

## Acknowledgement

- This node.js course helped me implement the above code [node.js bootcamp](https://www.udemy.com/course/nodejs-express-mongodb-bootcamp/)
