- [[Section 11: Modelling Data and Advanced Mongoose/MongoDB Data Modelling]]
- [[Section 11: Modelling Data and Advanced Mongoose/Designing Our Data Model]]
- [[Section 11: Modelling Data and Advanced Mongoose/Modelling Locations (Geospatial Data)Designing Our Data Model]]
- Modelling Tour Guides: Embedding
  collapsed:: true
	- tour model
		- ```javascript
		  tourSchema: {
		    ....
		        guides: Array,
		  }
		  tourSchema.pre('save', async function (next) {
		    const guidesPromise = this.guides.map(async (id) => User.findById(id));
		    this.guides = await Promise.all(guidesPromise);
		    next();
		  });
		  ```
- Modelling Tour Guides: Child Referencing
  collapsed:: true
	- tour model
	  collapsed:: true
		- ```javascript
		  tourSchema: {
		    ....
		   guides: [{ type: mongoose.Schema.ObjectId, ref: 'User' }],
		  
		  }
		  ```
- Populating Tour Guides
  collapsed:: true
	- tourController
	  collapsed:: true
		- ```javascript
		  getTour()  
		  const tour = await Tour.findById(req.params.id).populate({
		      path: 'guides',
		      select: '-__v -passwordChangedAt',
		    });
		  
		  ```
	- or
	- tourmodel
	  collapsed:: true
		- ```javascript
		  
		  tourSchema.pre(/^find/, function (next) {
		    this.populate({
		      path: 'guides',
		      select: '-__v -passwordChangedAt',
		    });
		    next();
		  });
		  ```
	-
- Modelling Reviews: Parent Referencing
  collapsed:: true
	- reviewModel
	  collapsed:: true
		- ```javascript
		  // review / rating / createdAt / ref to tour / ref to user
		  const mongoose = require('mongoose');
		  
		  const reviewSchema = new mongoose.Schema(
		    {
		      review: {
		        type: String,
		        require: [true, 'Review can not be empty!'],
		      },
		      rating: {
		        type: Number,
		        min: 1,
		        max: 5,
		      },
		      createdAt: {
		        type: Date,
		        default: Date.now(),
		      },
		      tour: {
		        type: mongoose.Schema.ObjectId,
		        ref: 'Tour',
		        required: [true, 'Review must belong to a tour.'],
		      },
		      user: {
		        type: mongoose.Schema.ObjectId,
		        ref: 'User',
		        require: [true, 'Review must belong to a user'],
		      },
		    },
		    {
		      toJSON: { virtuals: true },
		      toObject: { virtuals: true },
		    },
		  );
		  
		  const Review = mongoose.model('Review', reviewSchema);
		  
		  module.exports = Review;
		  
		  ```
	-
- Creating and Getting Reviews
  collapsed:: true
	- app.js
	  collapsed:: true
		- ```javascript
		  const reviewRouter = require('./routes/reviewRoutes');
		  app.use('/api/v1/reviews', reviewRouter);
		  
		  ```
	- reviewRoute.js
	  collapsed:: true
		- ```javascript
		  const express = require('express');
		  const reviewController = require('./../controllers/reviewController');
		  
		  const router = express.Router();
		  
		  // get create
		  
		  router
		    .route('/')
		    .get(reviewController.getAllReviews)
		    .post(reviewController.createReview);
		  
		  module.exports = router;
		  
		  ```
	- reviewController.js
	  collapsed:: true
		- ```javascript
		  const AppError = require('./../utils/appError');
		  const Review = require('./../models/reviewModel');
		  const catchAsync = require('./../utils/catchAsync');
		  
		  exports.getAllReviews = catchAsync(async (req, res, next) => {
		    const reviews = await Review.find();
		    res.status(200).json({
		      status: 'success',
		      results: reviews.length,
		      data: {
		        reviews,
		      },
		    });
		  });
		  
		  exports.createReview = catchAsync(async (req, res, next) => {
		    const newReview = await Review.create(req.body);
		  
		    res.status(201).json({
		      status: 'success',
		      data: {
		        review: newReview,
		      },
		    });
		  });
		  
		  ```
- Populating Reviews
  collapsed:: true
	- reviewModel
	  collapsed:: true
		- ```javascript
		  reviewSchema.pre(/^find/, function (next) {
		    this.populate({
		      path: 'tour',
		      select: 'name',
		    }).populate({
		      path: 'user',
		      select: 'name photo',
		    });
		    next();
		  });
		  ```
- Virtual Populate: Tours and Reviews
  collapsed:: true
	- reviewModel
	  collapsed:: true
		- ```javascript
		  reviewSchema.pre(/^find/, function (next) {
		    // this.populate({
		    //   path: 'tour',
		    //   select: 'name',
		    // }).populate({
		    //   path: 'user',
		    //   select: 'name photo',
		    // });
		    this.populate({
		      path: 'user',
		      select: 'name photo',
		    });
		    next();
		  });
		  ```
	- tourModel
	  collapsed:: true
		- ```javascript
		  
		  // virtual populate
		  tourSchema.virtual('reviews', {
		    ref: 'Review',
		    foreignField: 'tour',
		    localField: '_id',
		  });
		  ```
	- tourController
	  collapsed:: true
		- ```javascript
		  exports.getTour = catchAsync(async (req, res, next) => {
		    const tour = await Tour.findById(req.params.id).populate('reviews');
		    // Tour.findOne({_id: req.params.id})
		    if (!tour) {
		      return next(new AppError('No tour found with that ID', 404));
		    }
		  
		    res.status(200).json({
		      status: 'success',
		      data: {
		        tour,
		      },
		  });
		  ```
- Implementing Simple Nested Routes
  collapsed:: true
	- tourRoute
	  collapsed:: true
		- ```javascript
		  // POST /tour/234fawe123/reviews
		  // GET /tour/234fawe123/reviews
		  // GET /tour/234fawe123/reviews/123daqwe123
		  
		  router
		    .route('/:tourId/reviews')
		    .post(
		      authController.protect,
		      authController.restrictTo('user'),
		      reviewController.createReview,
		    );
		  ```
	- reviewController
	  collapsed:: true
		- ```javascript
		  
		  exports.createReview = catchAsync(async (req, res, next) => {
		    // Allow nested routes
		    if (!req.body.tour) req.body.tour = req.params.tourId;
		    if (!req.body.user) req.body.user = req.user.id;
		  
		    const newReview = await Review.create(req.body);
		  
		    res.status(201).json({
		      status: 'success',
		      data: {
		        review: newReview,
		      },
		    });
		  });
		  ```
- Nested Routes with Express
  collapsed:: true
	- tourRoute
	  collapsed:: true
		- ```javascript
		  router.use('/:tourId/reviews', reviewRouter);
		  
		  ```
	- reviewRoute
	  collapsed:: true
		- ```javascript
		  const router = express.Router({ mergeParams: true });
		  
		  ```
- Adding a Nested GET Endpoint
  collapsed:: true
	- reviewController
	  collapsed:: true
		- ```javascript
		  exports.getAllReviews = catchAsync(async (req, res, next) => {
		    let filter = {};
		    if (req.params.tourId) filter = { tour: req.params.tourId };
		  
		    const reviews = await Review.find(filter);
		    res.status(200).json({
		      status: 'success',
		      results: reviews.length,
		      data: {
		        reviews,
		      },
		    });
		  });
		  ```
- Building Handler Factory Functions: Delete
  collapsed:: true
	- handleFactory.js
	  collapsed:: true
		- ```javascript
		  const catchAsync = require('../utils/catchAsync');
		  const AppError = require('../utils/appError');
		  
		  exports.deleteOne = (Model) =>
		    catchAsync(async (req, res, next) => {
		      const doc = await Model.findByIdAndDelete(req.params.id);
		  
		      if (!doc) {
		        return next(new AppError('No document found with that ID', 404));
		      }
		  
		      res.status(204).json({
		        status: 'success',
		        data: null,
		      });
		    });
		  
		  ```
	- reviewController.js
	  collapsed:: true
		- ```javascript
		  const factory = require('./handleFactory');
		  exports.deleteReview = factory.deleteOne(Review);
		  
		  ```
	- reviewRoute.js
	  collapsed:: true
		- ```javascript
		  router.route('/:id').delete(reviewController.deleteReview);
		  
		  ```
	- tourController.js
	  collapsed:: true
		- ```javascript
		  exports.deleteTour = factory.deleteOne(Tour);
		  
		  ```
- Factory Functions: Update and Create
  collapsed:: true
	- handleFactory.js
	  collapsed:: true
		- ```javascript
		  exports.updateOne = (Model) =>
		    catchAsync(async (req, res, next) => {
		      const doc = await Model.findByIdAndUpdate(req.params.id, req.body, {
		        new: true,
		        runValidators: true,
		      });
		  
		      if (!doc) {
		        return next(new AppError('No tour found with that ID', 404));
		      }
		  
		      res.status(200).json({
		        status: 'success',
		        data: {
		          data: doc,
		        },
		      });
		    });
		  ```
	- revireController.js
	  collapsed:: true
		- ```javascript
		  
		  exports.setTourUserIds = (req, res, next) => {
		    // Allow nested routes
		    if (!req.body.tour) req.body.tour = req.params.tourId;
		    if (!req.body.user) req.body.user = req.user.id;
		    next();
		  };
		  exports.createReview = factory.createOne(Review);
		  exports.updateReview = factory.updateOne(Review);
		  
		  ```
	- reviewRoute.js
	  collapsed:: true
		- ```javascript
		  router
		    .route('/')
		    .get(reviewController.getAllReviews)
		    .post(
		      authController.protect,
		      authController.restrictTo('user'),
		      reviewController.setTourUserIds,
		      reviewController.createReview,
		    );
		  
		  router
		    .route('/:id')
		    .patch(reviewController.updateReview)
		    .delete(reviewController.deleteReview);
		  module.exports = router;
		  ```
- Factory Functions: Reading
  collapsed:: true
	- handleFactory
	  collapsed:: true
		- ```javascript
		  exports.getOne = (Model, popOptions) =>
		    catchAsync(async (req, res, next) => {
		      let query = Model.findById(req.params.id);
		      if (popOptions) query = query.populate(popOptions);
		  
		      const doc = await query;
		      if (!doc) {
		        return next(new AppError('No document found with that ID', 404));
		      }
		  
		      res.status(200).json({
		        status: 'success',
		        data: {
		          data: doc,
		        },
		      });
		    });
		  
		  exports.getAll = (Model) =>
		    catchAsync(async (req, res, next) => {
		      // to allow for nested get reviews on tour(hack)
		      let filter = {};
		      if (req.params.tourId) filter = { tour: req.params.tourId };
		  
		      const features = new APIFeature(Model.find(), req.query)
		        .filter()
		        .sort()
		        .limitFields()
		        .paginate();
		      const doc = await features.query;
		  
		      res.status(200).json({
		        status: 'success',
		        results: doc.length,
		        data: {
		          data: doc,
		        },
		      });
		    });
		  
		  ```
- Adding a /me Endpoint
  collapsed:: true
	- userRoute
	  collapsed:: true
		- ```javascript
		  router.get('/me', authController.protect, getMe, getUser);
		  
		  ```
	- userController
	  collapsed:: true
		- ```javascript
		  exports.getMe = (req, res, next) => {
		    req.params.id = req.user.id;
		    next();
		  };
		  ```
- Adding Missing Authentication and Authorization
  collapsed:: true
	- use protect
	- use middleware protect router
- Importing Review and User Data
  collapsed:: true
	- js
	  collapsed:: true
		- ```javascript
		  const fs = require('fs');
		  const mongoose = require('mongoose');
		  const dotenv = require('dotenv');
		  const Tour = require('./../../models/tourModel');
		  const User = require('./../../models/userModel');
		  const Review = require('./../../models/reviewModel');
		  
		  dotenv.config({ path: './../../config.env' });
		  
		  const DB = process.env.DATABASE.replace(
		    '<PASSWORD>',
		    process.env.DATABASE_PASSWORD,
		  );
		  
		  mongoose.connect(DB).then(() => console.log('DB connection successful!'));
		  
		  // READ JSON FILE
		  const tours = JSON.parse(fs.readFileSync(`${__dirname}/tours.json`, 'utf-8'));
		  const users = JSON.parse(fs.readFileSync(`${__dirname}/users.json`, 'utf-8'));
		  const reviews = JSON.parse(
		    fs.readFileSync(`${__dirname}/reviews.json`, 'utf-8'),
		  );
		  
		  // IMPORT DATA INTO DB
		  const importData = async () => {
		    try {
		      await Tour.create(tours);
		      await User.create(users, { validateBeforeSave: false });
		      await Review.create(reviews);
		      console.log('Data successfully loaded!');
		    } catch (err) {
		      console.log(err);
		    }
		    process.exit();
		  };
		  
		  // DELETE ALL DATA FROM DB
		  const deleteData = async () => {
		    try {
		      await Tour.deleteMany();
		      await User.deleteMany();
		      await Review.deleteMany();
		      console.log('Data successfully deleted!');
		      process.exit();
		    } catch (err) {
		      console.log(err);
		    }
		  };
		  
		  if (process.argv[2] === '--import') {
		    importData();
		  } else if (process.argv[2] === '--delete') {
		    deleteData();
		  }
		  
		  ```
- Improving Read Performance with Indexes
  collapsed:: true
	- ```javascript
	  const doc = await features.query.explain();
	  
	  tourSchema.index({ price: 1 });
	  tourSchema.index({ price: 1, ratingsAverage: 1 });
	  tourSchema.index({ slug: 1 });
	  ```
- Calculating Average Rating on Tours
  collapsed:: true
	- reviewModel
	  collapsed:: true
		- ```javascript
		  reviewSchema.statics.calcAverageRatings = async function (tourId) {
		    const stats = await this.aggregate([
		      {
		        $match: { tour: tourId },
		      },
		      {
		        $group: {
		          _id: '$tour',
		          nRating: { $sum: 1 },
		          avgRating: { $avg: '$rating' },
		        },
		      },
		    ]);
		    console.log(stats);
		    if (stats.length > 0) {
		      await Tour.findByIdAndUpdate(tourId, {
		        ratingsQuantity: stats[0].nRating,
		        ratingsAverage: stats[0].avgRating,
		      });
		    } else {
		      await Tour.findByIdAndUpdate(tourId, {
		        ratingsQuantity: 0,
		        ratingsAverage: 4.5,
		      });
		    }
		  };
		  
		  reviewSchema.pre('save', function () {
		    // this points to current review
		    this.constructor.calcAverageRatings(this.tour);
		  });
		  
		  // findByIdAndUpdate
		  // findByIdAndDelete
		  reviewSchema.pre(/^findOneAnd/, async function (next) {
		    this.r = await this.model.findOne();
		    console.log(this.r);
		    next();
		  });
		  
		  reviewSchema.post(/^findOneAnd/, async function () {
		    // await this.findOne(); does not work here, query has already execute
		    await this.r.constructor.calcAverageRatings(this.r.tour);
		  });
		  ```
- Preventing Duplicate Reviews
  collapsed:: true
	- ```javascript
	  reviewSchema.index({ tour: 1, user: 1 }, { unique: true });
	  
	  ```
- Geospatial Queries: Finding Tours Within Radius
  collapsed:: true
	- tourRoute
	  collapsed:: true
		- ```javascript
		  router
		    .route('/tours-within/:distance/center/:latlng/unit/:unit')
		    .get(getToursWithin);
		  // /tours-within?distance=233&center=-40,45&unit=mi
		  // /tours-within?233&center/-40,45&unit/mi
		  
		  ```
	- tourController
	  collapsed:: true
		- ```javascript
		  exports.getToursWithin = catchAsync(async (req, res, next) => {
		    const { distance, latlng, unit } = req.params;
		    const [lat, lng] = latlng.split(',');
		  
		    const radius = unit === 'mi' ? distance / 3963.2 : distance / 6378.1;
		  
		    if (!lat || !lng) {
		      next(
		        new AppError(
		          'Please provide latitude and lng in the format lat,lng.',
		          400,
		        ),
		      );
		    }
		  
		    const tours = await Tour.find({
		      startLocation: { $geoWithin: { $centerSphere: [[lng, lat], radius] } },
		    });
		    res.status(200).json({
		      status: 'success',
		      results: tours.length,
		      data: {
		        data: tours,
		      },
		    });
		  });
		  ```
	- tourModel
	  collapsed:: true
		- ```javascript
		  tourSchema.index({ startLocation: '2dsphere' });
		  
		  ```
- Geospatial Aggregation: Calculating Distances
-
-