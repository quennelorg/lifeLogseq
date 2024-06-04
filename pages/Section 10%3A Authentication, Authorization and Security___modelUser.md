- all code
  collapsed:: true
	- ```javascript
	  const mongoose = require('mongoose');
	  const validator = require('validator');
	  
	  // name, email ,photo, password, passwordConfirm
	  const userSchema = new mongoose.Schema({
	    name: {
	      type: String,
	      required: [true, 'user must have a name'],
	      // unique: true,
	      trim: true,
	    },
	    email: {
	      type: String,
	      required: [true, 'user must have a email'],
	      unique: true,
	      lowercase: true,
	      validate: [validator.isEmail, 'Please provide a valid email'],
	    },
	    photo: {
	      type: String,
	      // required: [true, 'user must have a photo'],
	    },
	    password: {
	      type: String,
	      required: [true, 'user must have a password'],
	      maxlength: [40, 'user password must have less or equal then 40 characters'],
	      minlength: [8, 'user password must have more or equal then 8 characters'],
	    },
	    passwordConfirm: {
	      type: String,
	      required: [true, 'Please confirm your password'],
	    },
	  });
	  
	  const User = mongoose.model('User', userSchema);
	  
	  module.exports = User;
	  
	  ```
- Description
	- 这段代码使用 Mongoose 定义了一个用户模型 `User`，并对用户数据进行了相应的验证和限制。它包括用户的姓名、电子邮件、照片、密码和密码确认字段。让我们逐步解释这段代码的各个部分。
	- ### 引入模块
	  
	  ```javascript
	  const mongoose = require('mongoose');
	  const validator = require('validator');
	  ```
	- **mongoose**：Mongoose 是一个用于在 Node.js 中操作 MongoDB 的对象数据建模 (ODM) 库。
	- **validator**：Validator 是一个用于字符串验证的库，它提供了许多常用的验证方法，例如验证电子邮件地址。
	- ### 定义用户模式（Schema）
	  
	  ```javascript
	  const userSchema = new mongoose.Schema({
	  name: {
	    type: String,
	    required: [true, 'user must have a name'],
	    trim: true,
	  },
	  email: {
	    type: String,
	    required: [true, 'user must have a email'],
	    unique: true,
	    lowercase: true,
	    validate: [validator.isEmail, 'Please provide a valid email'],
	  },
	  photo: {
	    type: String,
	  },
	  password: {
	    type: String,
	    required: [true, 'user must have a password'],
	    maxlength: [40, 'user password must have less or equal then 40 characters'],
	    minlength: [8, 'user password must have more or equal then 8 characters'],
	  },
	  passwordConfirm: {
	    type: String,
	    required: [true, 'Please confirm your password'],
	  },
	  });
	  ```
	- #### 字段说明
	  
	  1. **name**:
		- **type**: 数据类型为字符串。
		- **required**: 必填字段，若未提供，则返回错误信息 'user must have a name'。
		- **trim**: 自动去除字符串首尾的空格。
	- 2. **email**:
		- **type**: 数据类型为字符串。
		- **required**: 必填字段，若未提供，则返回错误信息 'user must have a email'。
		- **unique**: 唯一性约束，确保数据库中不会有重复的电子邮件。
		- **lowercase**: 自动将字符串转换为小写。
		- **validate**: 使用 `validator.isEmail` 方法验证电子邮件格式，若格式无效，则返回错误信息 'Please provide a valid email'。
	- 3. **photo**:
		- **type**: 数据类型为字符串。
		- **required**: 这部分注释掉了，表示照片不是必填字段。
	- 4. **password**:
		- **type**: 数据类型为字符串。
		- **required**: 必填字段，若未提供，则返回错误信息 'user must have a password'。
		- **maxlength**: 最大长度为 40 个字符，若超过则返回错误信息 'user password must have less or equal then 40 characters'。
		- **minlength**: 最小长度为 8 个字符，若不足则返回错误信息 'user password must have more or equal then 8 characters'。
	- 5. **passwordConfirm**:
		- **type**: 数据类型为字符串。
		- **required**: 必填字段，若未提供，则返回错误信息 'Please confirm your password'。
	- ### 创建并导出模型
	  
	  ```javascript
	  const User = mongoose.model('User', userSchema);
	  
	  module.exports = User;
	  ```
	- **创建模型**：使用 `mongoose.model` 方法将定义的 `userSchema` 编译为一个模型，并命名为 `User`。
	- **导出模型**：导出 `User` 模型，以便在其他文件中使用。
	- ### 总结
	  
	  这段代码定义了一个用户模型 `User`，包括用户的姓名、电子邮件、照片、密码和密码确认字段。它对这些字段进行了相应的验证和限制，确保数据的完整性和正确性。通过导出这个模型，其他部分的代码可以使用它来进行用户数据的创建、读取、更新和删除等操作。
-