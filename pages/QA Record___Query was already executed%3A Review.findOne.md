- Q
	- error
	- `Query was already executed: Review.findOne`
	- code
	  collapsed:: true
		- ```javascript
		  eviewSchema.pre(/^findOneAnd/, async function (next) {
		    this.r = await this.findOne();
		    console.log(this.r);
		    next();
		  });
		  ```
- A
	- 你在 `reviewSchema.pre(/^findOneAnd/, async function (next) { ... })` 中使用了 `this.findOne()`，导致查询已经执行，而在 `findOneAndUpdate` 或 `findOneAndDelete` 这样的查询中，查询只能执行一次，所以会报 `Query was already executed` 错误。
	  
	  要解决这个问题，你可以使用 `findOneAndUpdate` 或 `findOneAndDelete` 钩子中的文档中间件来获取当前文档，而不是在查询中再次执行查询。
	  
	  你可以使用 `this.model.findOne()` 来获取当前文档，而不是 `this.findOne()`。`this.model` 返回的是当前的模型，而 `findOne` 方法在模型上执行查询。
	  
	  ```javascript
	  reviewSchema.pre(/^findOneAnd/, async function (next) {
	    this.r = await this.model.findOne(this.getQuery());
	    console.log(this.r);
	    next();
	  });
	  ```
	- ### 解释
	  
	  1. **this.model.findOne(this.getQuery())**:
		- `this.getQuery()` 返回当前查询的条件。
		- `this.model` 是当前的模型，你可以使用 `findOne` 方法在模型上执行查询。
		  
		  2. **避免再次执行查询**:
		- 这样做可以避免在已经执行的查询上再次执行查询，从而避免 `Query was already executed` 错误。
	- ### 完整示例
	  
	  假设你有一个 `Review` 模型，你可以这样定义 `reviewSchema.pre(/^findOneAnd/, ...)` 钩子：
	  
	  ```javascript
	  const mongoose = require('mongoose');
	  
	  const reviewSchema = new mongoose.Schema({
	  review: {
	    type: String,
	    required: [true, 'Review cannot be empty']
	  },
	  rating: {
	    type: Number,
	    required: [true, 'Rating is required']
	  },
	  createdAt: {
	    type: Date,
	    default: Date.now
	  }
	  // 其他字段...
	  });
	  
	  reviewSchema.pre(/^findOneAnd/, async function(next) {
	  this.r = await this.model.findOne(this.getQuery());
	  console.log(this.r);
	  next();
	  });
	  
	  const Review = mongoose.model('Review', reviewSchema);
	  
	  module.exports = Review;
	  ```
	- ### 使用示例
	  
	  假设你正在执行 `findOneAndUpdate` 或 `findOneAndDelete` 操作：
	  
	  ```javascript
	  const Review = require('./models/reviewModel');
	  
	  // 示例：更新一个评论
	  Review.findOneAndUpdate({ _id: 'someReviewId' }, { rating: 4 })
	  .then(updatedReview => {
	    console.log('Review updated:', updatedReview);
	  })
	  .catch(err => {
	    console.error('Error updating review:', err);
	  });
	  
	  // 示例：删除一个评论
	  Review.findOneAndDelete({ _id: 'someReviewId' })
	  .then(deletedReview => {
	    console.log('Review deleted:', deletedReview);
	  })
	  .catch(err => {
	    console.error('Error deleting review:', err);
	  });
	  ```
	- ### 总结
	  
	  通过使用 `this.model.findOne(this.getQuery())` 代替 `this.findOne()`，你可以避免 `Query was already executed` 错误，并且正确地在 `pre` 钩子中获取当前文档。这样可以确保你的中间件能够正常工作，而不会因为查询重复执行而导致错误。