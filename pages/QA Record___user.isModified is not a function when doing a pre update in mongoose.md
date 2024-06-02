-
- [ ] `()=>{}` 没有this scope [user.isModified is not a function when doing a pre update in mongoose](https://stackoverflow.com/questions/40161459/user-ismodified-is-not-a-function-when-doing-a-pre-update-in-mongoose)
-
- 同样的问题
  collapsed:: true
	- ```javascript
	  // 注释这段代码，create就能够成功，但是解开注释，就不成功
	  tourSchema.pre('save', (next) => {
	    console.log(this.name);
	    this.slug = slugify(this.name, { lower: true });
	    next();
	  });
	  ```
- 解答
  collapsed:: true
	- 根据你的描述，问题可能出在 `this` 关键字在 `tourSchema.pre('save', ...)` 钩子中的使用上。`this` 关键字在回调函数中可能没有指向预期的文档对象，这会导致问题。
	  
	  在 Mongoose 的中间件中，使用箭头函数 (`=>`) 会导致 `this` 绑定到外部上下文，而不是预期的文档对象。为了正确地访问文档对象，应该使用普通的函数表达式而不是箭头函数。
	  
	  你可以试试以下修改：
	  
	  ```javascript
	  const slugify = require('slugify');
	  const mongoose = require('mongoose');
	  
	  const tourSchema = new mongoose.Schema({
	    name: {
	      type: String,
	      required: [true, 'A tour must have a name'],
	      unique: true,
	      trim: true,
	    },
	    slug: String,
	    // 其他字段...
	  });
	  
	  tourSchema.pre('save', function(next) {
	    console.log(this.name);
	    this.slug = slugify(this.name, { lower: true });
	    next();
	  });
	  
	  const Tour = mongoose.model('Tour', tourSchema);
	  
	  module.exports = Tour;
	  ```
	- ### 解释
	  
	  1. **使用普通函数表达式**：
	   替换箭头函数为普通函数表达式，以确保 `this` 正确指向当前文档对象。
	  
	   ```javascript
	   tourSchema.pre('save', function(next) {
	     console.log(this.name); // 正确访问文档对象的 name 属性
	     this.slug = slugify(this.name, { lower: true });
	     next();
	   });
	   ```
	  
	  2. **确保所有字段存在**：
	   确保 `name` 字段在文档中存在，并且在调用 `slugify` 时具有有效值。如果 `name` 字段为空或未定义，`slugify` 函数将无法生成 slug。
	- ### 测试
	  
	  在进行上述修改后，再次尝试创建一个新文档：
	  
	  ```javascript
	  const newTour = {
	  name: 'The Snow Adventurer',
	  // 其他字段...
	  };
	  
	  Tour.create(newTour)
	  .then(tour => {
	    console.log('Tour created successfully:', tour);
	  })
	  .catch(err => {
	    console.error('Error creating tour:', err);
	  });
	  ```
	  
	  确保所有字段的数据都符合模型的验证规则，并且数据库连接已经成功建立。如果上述代码仍然无法解决问题，请提供更详细的错误信息，以便进一步诊断问题。
-