- 在 Mongoose 中，虚拟属性 (Virtual Properties) 是一种定义在模式 (Schema) 上的属性，它们不会直接存储在 MongoDB 数据库中，但可以基于文档的其他字段动态计算和生成。虚拟属性通常用于格式化或组合数据，以便在从数据库检索文档时更方便地使用和展示这些数据。
  title:: Using MongoDB with Mongoose/Virtual Properties
- ### 使用虚拟属性的场景
  虚拟属性可以用于：
  1. **计算派生值**：例如，根据用户的名字和姓氏计算全名。
  2. **格式化数据**：例如，格式化日期或货币值。
  3. **组合多个字段**：例如，将多个地址字段组合成一个完整的地址字符串。
- ### 定义虚拟属性
  可以使用 Mongoose 的 `Schema.virtual()` 方法来定义虚拟属性。下面是一些示例，展示如何定义和使用虚拟属性。
- ### 示例
- #### 1. 基本示例
  假设我们有一个 `User` 模型，它包含 `firstName` 和 `lastName` 字段，我们可以定义一个虚拟属性 `fullName` 来生成用户的全名。
  
  ```javascript
  const mongoose = require('mongoose');
  
  const userSchema = new mongoose.Schema({
    firstName: String,
    lastName: String
  });
  
  // 定义虚拟属性 fullName
  userSchema.virtual('fullName').get(function() {
    return `${this.firstName} ${this.lastName}`;
  });
  
  const User = mongoose.model('User', userSchema);
  
  async function main() {
    await mongoose.connect('mongodb://localhost:27017/test', { useUnifiedTopology: true, useNewUrlParser: true });
  
    const user = new User({ firstName: 'John', lastName: 'Doe' });
    console.log(user.fullName); // 输出 "John Doe"
  
    await mongoose.disconnect();
  }
  
  main().catch(console.error);
  ```
- #### 2. 带有 Setter 的虚拟属性
  虚拟属性不仅可以定义 Getter 方法，还可以定义 Setter 方法。例如，我们可以将 `fullName` 拆分成 `firstName` 和 `lastName`。
  
  ```javascript
  const mongoose = require('mongoose');
  
  const userSchema = new mongoose.Schema({
    firstName: String,
    lastName: String
  });
  
  // 定义虚拟属性 fullName
  userSchema.virtual('fullName')
    .get(function() {
        return `${this.firstName} ${this.lastName}`;
    })
    .set(function(name) {
        const split = name.split(' ');
        this.firstName = split[0];
        this.lastName = split[1];
    });
  
  const User = mongoose.model('User', userSchema);
  
  async function main() {
    await mongoose.connect('mongodb://localhost:27017/test', { useUnifiedTopology: true, useNewUrlParser: true });
  
    const user = new User();
    user.fullName = 'Jane Doe';
    console.log(user.firstName); // 输出 "Jane"
    console.log(user.lastName);  // 输出 "Doe"
  
    await mongoose.disconnect();
  }
  
  main().catch(console.error);
  ```
- ### 总结
- **虚拟属性**：在 Mongoose 中，虚拟属性是一种定义在模式上的属性，它们不会直接存储在数据库中，但可以基于文档的其他字段动态计算和生成。
- **使用场景**：虚拟属性可以用于计算派生值、格式化数据和组合多个字段。
- **定义方式**：使用 `Schema.virtual()` 方法定义虚拟属性，可以为其设置 Getter 和 Setter 方法，以便动态获取和设置值。
  
  虚拟属性的使用可以帮助简化数据处理逻辑，使得从数据库检索的数据更加直观和易于使用。