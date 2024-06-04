- tourModel
  collapsed:: true
	- ```javascript
	  const tourSchema = new mongoose.Schema(
	    {
	      name: {
	        type: String,
	        required: [true, 'A tour must have a name'],
	        unique: true,
	        trim: true,
	        maxlength: [40, 'A tour name must have less or equal then 40 characters'],
	        minlength: [10, 'A tour name must have more or equal then 10 characters'],
	        // validator: [validator.isAlpha, 'Tour name must only contain characters'],
	      },
	      slug: String,
	      duration: {
	        type: Number,
	        required: [true, 'A tour must have a duration'],
	      },
	      maxGroupSize: {
	        type: Number,
	        required: [true, 'A tour must have a maxGroupSize'],
	      },
	      difficulty: {
	        type: String,
	        required: [true, 'A tour must have a difficulty'],
	        enum: {
	          values: ['easy', 'medium', 'difficult'],
	          message: 'Difficulty is either: easy, medium, difficult',
	        },
	      },
	      ratingsAverage: {
	        type: Number,
	        default: 4.5,
	        min: [1, 'Rating must be above 1.0'],
	        max: [5, 'Rating must be below 5.0'],
	      },
	      ratingsQuantity: { type: Number, default: 0 },
	      price: { type: Number, required: [true, 'A tour must have a price'] },
	      priceDiscount: {
	        type: Number,
	        validate: {
	          validator: (val) => {
	            // this only points to current doc on NEW document creation
	            return val < this.price;
	          },
	          message: 'Discount price ($VALUE) should be below regular price',
	        },
	      },
	      summary: {
	        type: String,
	        trim: true,
	        required: [true, 'A tour must have a summary'],
	      },
	      description: {
	        type: String,
	        trim: true,
	      },
	      imageCover: {
	        type: String,
	        required: [true, 'A tour must have a imageCover'],
	      },
	      images: [String],
	      createAt: {
	        type: Date,
	        default: Date.now(),
	        select: false,
	      },
	      startDates: [Date],
	      secretTour: {
	        type: Boolean,
	        default: false,
	      },
	      startLocation: {
	        // GeoJSON
	        type: {
	          type: String,
	          default: 'Point',
	          enum: ['Point'],
	        },
	        coordinates: [Number],
	        address: String,
	        description: String,
	      },
	      locations: [
	        {
	          type: {
	            type: String,
	            default: 'Point',
	            enum: ['Point'],
	          },
	          coordinates: [Number],
	          address: String,
	          description: String,
	          day: Number,
	        },
	      ],
	    },
	    {
	      toJSON: { virtuals: true },
	      toObject: { virtuals: true },
	    },
	  );
	  ```