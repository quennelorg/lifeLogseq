- code
  collapsed:: true
	- ```javascript
	  userSchema.pre('save', async function (next) {
	    // Only run this function if password was actually modified
	    if (!this.isModified('password')) return next();
	  
	    // Hash the password with cost of 12
	    this.password = await bcrypt.hash(this.password, 12);
	  
	    // Delete passwordConfirm field
	    this.passwordConfirm = undefined;
	    next();
	  });
	  ```
- Description
	-
	- ### 1. `pre('save')` 钩子
		- Mongoose 的 `pre` 钩子可以在文档保存到数据库之前执行一些操作。在这里，`pre('save')` 钩子在保存用户文档之前执行。
	- ### 2. 异步函数
		- 由于处理密码哈希是一个异步操作，因此使用了 `async` 关键字，使函数成为一个异步函数。
	- ### 3. 只在密码被修改时运行
		- ```
		  `if (!this.isModified('password')) return next();`
		  ```
			- **检查密码是否被修改**：使用 `isModified('password')` 方法检查 `password` 字段是否被修改。
			- **跳过哈希操作**：如果密码没有被修改，直接调用 `next()` 结束中间件的执行，继续保存文档。
	- ### 4. 哈希密码
		- ```
		  `this.password = await bcrypt.hash(this.password, 12);`
		  ```
			- **哈希算法**：使用 `bcrypt` 库对密码进行哈希处理。
			- **成本因子（cost factor）12**：成本因子（也称为 salt rounds）决定了哈希过程的复杂度。值越高，计算时间越长，安全性越高。这里使用了成本因子 12。
			- **更新密码字段**：哈希后的密码覆盖原来的明文密码。
	- ### 5. 删除 `passwordConfirm` 字段
		- ```
		  `this.passwordConfirm = undefined;`
		  ```
			- **删除 `passwordConfirm` 字段**：假设 `passwordConfirm` 是用于用户注册时确认密码的字段，只在输入时使用，不需要存储在数据库中。将其设为 `undefined` 可以确保它不会被保存到数据库。
	- ### 6. 调用 `next()` 继续保存流程
		- ```
		  `next();`
		  ```
			- **调用 `next()`**：通知 Mongoose 可以继续保存文档到数据库。中间件必须调用 `next()`，否则保存操作会被挂起。
	- ### 代码总结
		- 这段代码的整体逻辑如下：
			- 1.  在保存用户文档之前，检查密码是否被修改。
			- 2.  如果密码未修改，直接继续保存文档。
			- 3.  如果密码被修改，对密码进行哈希处理。
			- 4.  删除 `passwordConfirm` 字段以确保不保存它。
			- 5.  调用 `next()` 继续保存文档。