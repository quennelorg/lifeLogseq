- 在 Express 应用中设置 Pug（以前称为 Jade）模板引擎，允许您使用 Pug 编写视图模板，从而生成动态的 HTML 页面。Pug 语法简洁且有助于生成干净、可维护的 HTML。以下是如何在 Express 中设置和使用 Pug 模板引擎的详细步骤：
- ### 步骤 1: 创建新的 Express 项目
  
  首先，确保您已经安装了 Node.js 和 npm。然后，您可以使用以下命令创建一个新的 Express 项目：
  
  ```bash
  mkdir my-express-app
  cd my-express-app
  npm init -y
  npm install express
  ```
- ### 步骤 2: 安装 Pug
  
  接下来，安装 Pug 模板引擎：
  
  ```bash
  npm install pug
  ```
- ### 步骤 3: 设置 Express 使用 Pug
  
  创建一个 `app.js` 文件，并配置 Express 使用 Pug 作为模板引擎：
  
  ```javascript
  const express = require('express');
  const path = require('path');
  
  const app = express();
  
  // 设置 Pug 作为模板引擎
  app.set('view engine', 'pug');
  
  // 设置视图文件夹
  app.set('views', path.join(__dirname, 'views'));
  
  // 定义一个路由
  app.get('/', (req, res) => {
  res.render('index', { title: 'Hey', message: 'Hello there!' });
  });
  
  // 启动服务器
  const port = 3000;
  app.listen(port, () => {
  console.log(`Server is running on http://localhost:${port}`);
  });
  ```
- ### 步骤 4: 创建视图文件
  
  在项目根目录中创建一个 `views` 文件夹，并在其中创建一个 `index.pug` 文件：
  
  ```pug
  //- views/index.pug
  doctype html
  html
  head
    title= title
  body
    h1= message
  ```
- ### 步骤 5: 运行应用
  
  回到终端，运行以下命令启动服务器：
  
  ```bash
  node app.js
  ```
  
  然后，打开浏览器访问 `http://localhost:3000`，您应该会看到页面显示 `Hello there!`，标题是 `Hey`。
- ### 详细解释
- **视图引擎设置**：`app.set('view engine', 'pug')` 告诉 Express 使用 Pug 作为视图引擎。
- **视图文件夹设置**：`app.set('views', path.join(__dirname, 'views'))` 指定视图文件存放的目录。
- **渲染视图**：在路由中使用 `res.render('index', { title: 'Hey', message: 'Hello there!' })` 渲染 `views` 文件夹中的 `index.pug` 文件，并传递一些变量到模板中。
- **Pug 语法**：`index.pug` 文件中的 Pug 语法是缩进敏感的，使用缩进来表示 HTML 元素的嵌套关系。等号 `=` 后跟变量名，用于插入动态数据。
- ### 进一步优化
  
  可以进一步优化项目结构和代码：
- **使用 Express 生成器**：快速生成基本项目结构 `npx express-generator --view=pug my-express-app`
- **中间件和路由拆分**：将路由逻辑拆分到单独的文件中，使用中间件处理静态文件、错误处理等。
- **CSS 和静态文件**：在 `public` 文件夹中存放 CSS、JavaScript 和图片等静态文件，并使用 `app.use(express.static(path.join(__dirname, 'public')))` 提供访问。
  
  以上就是在 Express 应用中设置 Pug 模板引擎的完整步骤和一些优化建议。希望对您有帮助！