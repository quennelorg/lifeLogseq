- 在 Mongoose 中，文档中间件 (Document Middleware)，也称为 "钩子" (hooks)，是指在特定的文档生命周期事件（如保存、验证、删除等）发生之前或之后执行的函数。这些中间件函数允许你在数据库操作之前或之后执行自定义逻辑，从而增强数据处理的灵活性和功能。
- ### 文档中间件类型
  
  Mongoose 提供了几种类型的文档中间件，常见的包括：
  
  1. **Pre 中间件**：在某个事件发生之前执行。
  2. **Post 中间件**：在某个事件发生之后执行。
- ### 常见的文档中间件事件
  
  以下是一些常见的文档中间件事件：
- `save`：文档保存之前 (`pre('save')`) 或保存之后 (`post('save')`) 执行。
- `validate`：文档验证之前 (`pre('validate')`) 或验证之后 (`post('validate')`) 执行。
- `remove`：文档删除之前 (`pre('remove')`) 或删除之后 (`post('remove')`) 执行。
- `updateOne` 和 `deleteOne`：文档更新或删除之前 (`pre('updateOne')`, `pre('deleteOne')`) 或之后 (`post('updateOne')`, `post('deleteOne')`) 执行。
- ### 示例
- #### Pre 中间件
  
  假设我们有一个 `User` 模型，希望在保存用户之前将用户名转换为小写：
  
  ```javascript
  const mongoose = require('mongoose');
  
  const userSchema = new mongoose.Schema({
    username: String,
    email: String
  });
  
  // 定义 pre 'save' 中间件
  userSchema.pre('save', function(next) {
    // 将用户名转换为小写
    this.username = this.username.toLowerCase();
    next();
  });
  
  const User = mongoose.model('User', userSchema);
  
  async function main() {
    await mongoose.connect('mongodb://localhost:27017/test', { useUnifiedTopology: true, useNewUrlParser: true });
  
    const user = new User({ username: 'JohnDoe', email: 'john@example.com' });
    await user.save();
  
    console.log(user.username); // 输出 "johndoe"
  
    await mongoose.disconnect();
  }
  
  main().catch(console.error);
  ```
- #### Post 中间件
  
  假设我们希望在保存用户之后记录保存操作：
  
  ```javascript
  const mongoose = require('mongoose');
  
  const userSchema = new mongoose.Schema({
    username: String,
    email: String
  });
  
  // 定义 post 'save' 中间件
  userSchema.post('save', function(doc) {
    console.log(`User ${doc.username} has been saved.`);
  });
  
  const User = mongoose.model('User', userSchema);
  
  async function main() {
    await mongoose.connect('mongodb://localhost:27017/test', { useUnifiedTopology: true, useNewUrlParser: true });
  
    const user = new User({ username: 'JohnDoe', email: 'john@example.com' });
    await user.save();
  
    // 这里会输出 "User JohnDoe has been saved."
  
    await mongoose.disconnect();
  }
  
  main().catch(console.error);
  ```
- ### 总结
- **文档中间件 (Document Middleware)**：是指在文档生命周期事件发生之前或之后执行的函数，用于执行自定义逻辑。
- **Pre 中间件**：在事件发生之前执行，例如 `pre('save')` 在文档保存之前执行。
- **Post 中间件**：在事件发生之后执行，例如 `post('save')` 在文档保存之后执行。
- **使用场景**：可以用于数据预处理（如格式化数据）、验证、日志记录等。
  
  文档中间件提供了一种强大的机制，使得在文档生命周期的不同阶段插入自定义逻辑变得非常简单和直观。