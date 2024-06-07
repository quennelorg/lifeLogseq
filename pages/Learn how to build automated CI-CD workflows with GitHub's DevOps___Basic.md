- 什么是GitHub Action
  collapsed:: true
	- GitHub Actions 是 GitHub 提供的一种持续集成和持续部署（CI/CD）服务。它允许开发者在 GitHub 仓库中自动化他们的工作流，例如构建、测试和部署代码。通过定义工作流文件，开发者可以在代码仓库的特定事件（如代码推送、Pull Request 等）发生时自动触发相应的任务。
	  
	  GitHub Actions 的主要特点包括：
	  
	  1. **工作流文件**：工作流文件使用 YAML 格式定义，通常存放在仓库的 `.github/workflows` 目录中。每个工作流文件可以包含一个或多个独立的工作流，描述了具体的自动化任务。
	  
	  2. **事件触发**：工作流可以根据特定的事件触发，例如 `push`（代码推送）、`pull_request`（Pull Request 创建或更新）、`schedule`（定时任务）等。
	  
	  3. **作业（Jobs）和步骤（Steps）**：一个工作流由一个或多个作业组成，每个作业可以在不同的虚拟环境中并行运行。每个作业又由一系列步骤组成，每一步执行一个具体的操作（例如运行命令、调用脚本或使用预定义的 GitHub Action）。
	  
	  4. **预定义 Actions**：GitHub Actions 提供了大量预定义的 Actions，可以直接在工作流中使用。例如，常见的操作包括检查代码、运行测试、构建 Docker 镜像、部署到云服务等。开发者也可以创建和分享自定义的 Actions。
	  
	  5. **虚拟环境**：GitHub 提供多种运行环境（如 Ubuntu、Windows、macOS），供工作流中的作业使用。开发者可以选择合适的环境来运行他们的任务。
	  
	  一个简单的 GitHub Actions 工作流示例：
	  ```yaml
	  name: CI
	  
	  on:
	    push:
	      branches:
	        - main
	  
	  jobs:
	    build:
	      runs-on: ubuntu-latest
	  
	      steps:
	      - name: Checkout code
	        uses: actions/checkout@v2
	  
	      - name: Set up Node.js
	        uses: actions/setup-node@v2
	        with:
	          node-version: '14'
	  
	      - name: Install dependencies
	        run: npm install
	  
	      - name: Run tests
	        run: npm test
	  ```
	  
	  这个工作流在每次推送到 `main` 分支时触发，主要包含以下几个步骤：
	  1. 检出代码。
	  2. 设置 Node.js 环境。
	  3. 安装项目依赖。
	  4. 运行测试。
	  
	  通过 GitHub Actions，开发者可以轻松实现代码的自动化测试和部署，提高开发效率和代码质量。