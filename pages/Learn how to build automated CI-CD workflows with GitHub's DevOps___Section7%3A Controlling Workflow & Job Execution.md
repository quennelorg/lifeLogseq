- 默认来说，一个worklfow的执行顺序是从第一个步骤执行到最后一个步骤，其中只要有一个失败，则后续的就都不会执行，但有时候，我们需要更多的控制这个flow的执行，比如其中一个失败，跳过并继续执行下一个
- 这张图表述了 GitHub Actions 中如何通过条件表达式来控制作业（Jobs）和步骤（Steps）的执行。
- ### 主要内容
  collapsed:: true
	- #### 作业（Jobs）
		- **Conditional execution via `if` field**:
			- 通过 `if` 字段实现条件执行。
	- #### 步骤（Steps）
		- **Conditional execution via `if` field**:
			- 通过 `if` 字段实现条件执行。
		- **Ignore errors via `continue-on-error` field**:
			- 通过 `continue-on-error` 字段忽略错误。
	- ### 评估条件
	- **Evaluate conditions via Expressions**:
		- 通过表达式评估条件。
	- ### 详细解释
	  
	  1. **作业的条件执行**:
		- 可以在 `jobs` 部分使用 `if` 字段来定义作业是否应执行。例如:
		  ```yaml
		  jobs:
		    example-job:
		      runs-on: ubuntu-latest
		      if: github.ref == 'refs/heads/main'
		      steps:
		        - name: Checkout
		          uses: actions/checkout@v2
		  ```
		  以上例子中，只有在当前分支是 `main` 时，`example-job` 作业才会被执行。
		- 2. **步骤的条件执行**:
			- 同样地，可以在 `steps` 部分使用 `if` 字段来定义步骤是否应执行。例如:
			  ```yaml
			  jobs:
			    example-job:
			      runs-on: ubuntu-latest
			      steps:
			        - name: Checkout
			          uses: actions/checkout@v2
			        - name: Run tests
			          if: success()
			          run: npm test
			  ```
			  以上例子中，只有在前面的步骤成功时，`Run tests` 步骤才会被执行。
			- 3. **忽略步骤错误**:
				- 使用 `continue-on-error` 字段可以忽略步骤中的错误，使工作流继续执行。例如:
				  ```yaml
				  jobs:
				    example-job:
				      runs-on: ubuntu-latest
				      steps:
				        - name: Checkout
				          uses: actions/checkout@v2
				        - name: Run tests
				          run: npm test
				          continue-on-error: true
				  ```
				  以上例子中，即使 `Run tests` 步骤失败，后续步骤仍然会继续执行。
	- ### 结论
	  这张图主要展示了 GitHub Actions 中如何通过条件表达式来控制作业和步骤的执行，包括条件执行和错误处理。通过使用 `if` 字段和 `continue-on-error` 字段，用户可以灵活地定义工作流的执行逻辑，以满足不同的需求。
- ## [`steps` context](https://docs.github.com/en/actions/learn-github-actions/contexts#steps-context)
-
- continue
  collapsed:: true
	- ```yaml
	  name: Contiue Website Deployment
	  on:
	    push:
	      branches:
	        - main
	  jobs:
	    lint:
	      runs-on: ubuntu-latest
	      steps:
	        - name: Get code
	          uses: actions/checkout@v3
	        - name: Cache dependencies
	          id: cache
	          uses: actions/cache@v3
	          with:
	            path: node_modules
	            key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
	        - name: Install dependencies
	          if: steps.cache.outputs.cache-hit != 'true'
	          run: npm ci
	        - name: Lint code
	          run: npm run lint
	    test:
	      runs-on: ubuntu-latest
	      steps:
	        - name: Get code
	          uses: actions/checkout@v3
	        - name: Cache dependencies
	          id: cache
	          uses: actions/cache@v3
	          with:
	            path: node_modules
	            key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
	        - name: Install dependencies
	          if: steps.cache.outputs.cache-hit != 'true'
	          run: npm ci
	        - name: Test code
	          continue-on-error: true
	          id: run-tests
	          run: npm run test
	        - name: Upload test report
	          uses: actions/upload-artifact@v3
	          with:
	            name: test-report
	            path: test.json
	    build:
	      needs: test
	      runs-on: ubuntu-latest
	      steps:
	        - name: Get code
	          uses: actions/checkout@v3
	        - name: Cache dependencies
	          id: cache
	          uses: actions/cache@v3
	          with:
	            path: node_modules
	            key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
	        - name: Install dependencies
	          if: steps.cache.outputs.cache-hit != 'true'
	          run: npm ci
	        - name: Build website
	          id: build-website
	          run: npm run build
	        - name: Upload artifacts
	          uses: actions/upload-artifact@v3
	          with:
	            name: dist-files
	            path: dist
	    deploy:
	      needs: build
	      runs-on: ubuntu-latest
	      steps:
	        - name: Get build artifacts
	          uses: actions/download-artifact@v3
	          with:
	            name: dist-files
	        - name: Output contents
	          run: ls
	        - name: Deploy
	          run: echo "Deploying..."
	    report:
	      needs: [lint, deploy]
	      if: failure()
	      runs-on: ubuntu-latest
	      steps:
	        - name: Output information
	          run: | 
	            echo "Something went wrong"
	            echo "${{ toJSON(github) }}"
	  
	  ```
-
- execute flow
  collapsed:: true
	- ```yaml
	  name: Website Deployment
	  on:
	    push:
	      branches:
	        - main
	  jobs:
	    lint:
	      runs-on: ubuntu-latest
	      steps:
	        - name: Get code
	          uses: actions/checkout@v3
	        - name: Cache dependencies
	          id: cache
	          uses: actions/cache@v3
	          with:
	            path: node_modules
	            key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
	        - name: Install dependencies
	          if: steps.cache.outputs.cache-hit != 'true'
	          run: npm ci
	        - name: Lint code
	          run: npm run lint
	    test:
	      runs-on: ubuntu-latest
	      steps:
	        - name: Get code
	          uses: actions/checkout@v3
	        - name: Cache dependencies
	          id: cache
	          uses: actions/cache@v3
	          with:
	            path: node_modules
	            key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
	        - name: Install dependencies
	          if: steps.cache.outputs.cache-hit != 'true'
	          run: npm ci
	        - name: Test code
	          id: run-tests
	          run: npm run test
	        - name: Upload test report
	          if: failure() && steps.run-tests.outcome == 'failure'
	          uses: actions/upload-artifact@v3
	          with:
	            name: test-report
	            path: test.json
	    build:
	      needs: test
	      runs-on: ubuntu-latest
	      steps:
	        - name: Get code
	          uses: actions/checkout@v3
	        - name: Cache dependencies
	          id: cache
	          uses: actions/cache@v3
	          with:
	            path: node_modules
	            key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
	        - name: Install dependencies
	          if: steps.cache.outputs.cache-hit != 'true'
	          run: npm ci
	        - name: Build website
	          id: build-website
	          run: npm run build
	        - name: Upload artifacts
	          uses: actions/upload-artifact@v3
	          with:
	            name: dist-files
	            path: dist
	    deploy:
	      needs: build
	      runs-on: ubuntu-latest
	      steps:
	        - name: Get build artifacts
	          uses: actions/download-artifact@v3
	          with:
	            name: dist-files
	        - name: Output contents
	          run: ls
	        - name: Deploy
	          run: echo "Deploying..."
	    report:
	      needs: [lint, deploy]
	      if: failure()
	      runs-on: ubuntu-latest
	      steps:
	        - name: Output information
	          run: | 
	            echo "Something went wrong"
	            echo "${{ toJSON(github) }}"
	  
	  ```
-
- matrix job
  collapsed:: true
	- code
	  collapsed:: true
		- ```yaml
		  name: Matrix Demo
		  on: push
		  jobs:
		    build:
		      continue-on-error: true
		      strategy:
		        matrix:
		          node-version: [12, 14, 16]
		          operating-system: [ubuntu-latest, windows-latest]
		          include:
		            - node-version: 18
		              operating-system: ubuntu-latest
		          exclude:
		            - node-version: 12
		              operating-system: windows-latest
		      runs-on: ${{ matrix.operating-system }}
		      steps:
		        - name: Get Code
		          uses: actions/checkout@v3
		        - name: Install NodeJS
		          uses: actions/setup-node@v3
		          with:
		            node-version: ${{ matrix.node-version }}
		        - name: Install Dependencies
		          run: npm ci
		        - name: Build project
		          run: npm run build
		  ```
	- 1.  **矩阵策略**:
		- `matrix.node-version` 定义了 Node.js 的版本：`12`, `14`, `16`。
		- `matrix.operating-system` 定义了操作系统：`ubuntu-latest`, `windows-latest`。
		- `include` 包含了一个特定的组合：`node-version: 18` 和 `operating-system: ubuntu-latest`。
		- `exclude` 排除了一个特定的组合：`node-version: 12` 和 `operating-system: windows-latest`。
	- 2.  **组合生成**:
		- GitHub Actions 会根据 `matrix` 定义生成所有可能的组合，并为每个组合创建一个独立的作业实例。例如，以下是一些生成的组合：
			- Node.js 12 on Ubuntu
			- Node.js 14 on Ubuntu
			- Node.js 16 on Ubuntu
			- Node.js 14 on Windows
			- Node.js 16 on Windows
			- Node.js 18 on Ubuntu
		- 排除和包含的组合会相应地调整这些组合。
	- 3.  **作业配置**:
		- `continue-on-error: true` 表示即使作业失败，也会继续后续的作业。
		- `runs-on: ${{ matrix.operating-system }}` 表示作业将运行在由 `matrix.operating-system` 定义的操作系统上。
	- ### 作用
		- 这个配置文件用于在不同版本的 Node.js 和不同的操作系统上并行运行多个作业，从而确保项目在各种环境下都能正常构建和运行。
		- 矩阵策略有助于在多种配置下自动化测试和构建流程，提高项目的兼容性和稳定性。
-
- Reusable workflows
	- 可复用工作流程可以在不同的工作流程中多次使用，以减少重复的代码和配置，提高工作流程的可维护性和可读性
	- ### 具体描述
		- 1.  **Workflow 1**:
			- 包含 Job 1 和 Steps。
			- Workflow 1 是一个独立的工作流程，可以完成特定的任务，如图中的示例“Upload website code to hosting server”。
		- 2.  **Workflow 2**:
			- 包含 Job 1 和 Job 2，每个 Job 中包含 Steps。
			- Workflow 2 可以调用 Workflow 1 作为其中一个步骤，从而复用 Workflow 1 中的任务。
	- ### 主要要点
		- **Job 和 Steps**：
			- 每个 Workflow 包含一个或多个 Job。
			- 每个 Job 包含一个或多个 Steps。
			- Job 是工作流程中可并行执行的单元，而 Steps 是 Job 中按顺序执行的任务。
		- **可复用的工作流程**：
			- Workflow 2 中的 Job 2 可以调用 Workflow 1，从而实现代码和配置的复用。
			- 通过复用工作流程，可以避免在不同的工作流程中重复相同的代码和配置，提高效率和一致性。
		- **示例**：
			- 图中示例展示了如何将“上传网站代码到托管服务器”这个任务封装在 Workflow 1 中，然后在 Workflow 2 中复用这个任务。
	- ### 应用场景
		- 1.  **减少重复代码**：将常见的任务（如代码检查、构建、部署等）封装在单独的工作流程中，其他工作流程可以直接调用。
		- 2.  **提高可维护性**：当需要修改常见任务时，只需修改单独的工作流程，所有调用该工作流程的其他工作流程将自动更新。
		- 3.  **提高一致性**：通过复用工作流程，可以确保所有工作流程中的任务执行逻辑一致，避免因复制粘贴导致的错误。
	- code
		- Reusable Code
		  collapsed:: true
			- ```yaml
			  name: Reusable Deploy
			  on: 
			    workflow_call:
			      inputs:
			        artifact-name:
			          description: The name of the deployable artifact files
			          required: false
			          default: dist
			          type: string
			      outputs:
			        result:
			          description: The result of the deployment operation
			          value: ${{ jobs.deploy.outputs.outcome }}
			      # secrets:
			        # some-secret:
			          # required: false
			  jobs:
			    deploy:
			      outputs:
			        outcome: ${{ steps.set-result.outputs.step-result }}
			      runs-on: ubuntu-latest
			      steps:
			        - name: Get Code
			          uses: actions/download-artifact@v3
			          with:
			            name: ${{ inputs.artifact-name }}
			        - name: List files
			          run: ls
			        - name: Output information
			          run: echo "Deploying & uploading..."
			        - name: Set result output
			          id: set-result
			          run: echo "step-result=success" >> $GITHUB_OUTPUT
			  ```
		- use-Reusable Code
		  collapsed:: true
			- ```yaml
			  name: Using Reusable Workflow
			  on:
			    push:
			      branches:
			        - main
			  jobs:
			    lint:
			      runs-on: ubuntu-latest
			      steps:
			        - name: Get code
			          uses: actions/checkout@v3
			        - name: Cache dependencies
			          id: cache
			          uses: actions/cache@v3
			          with:
			            path: node_modules
			            key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
			        - name: Install dependencies
			          if: steps.cache.outputs.cache-hit != 'true'
			          run: npm ci
			        - name: Lint code
			          run: npm run lint
			    test:
			      runs-on: ubuntu-latest
			      steps:
			        - name: Get code
			          uses: actions/checkout@v3
			        - name: Cache dependencies
			          id: cache
			          uses: actions/cache@v3
			          with:
			            path: node_modules
			            key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
			        - name: Install dependencies
			          if: steps.cache.outputs.cache-hit != 'true'
			          run: npm ci
			        - name: Test code
			          id: run-tests
			          run: npm run test
			        - name: Upload test report
			          if: failure() && steps.run-tests.outcome == 'failure'
			          uses: actions/upload-artifact@v3
			          with:
			            name: test-report
			            path: test.json
			    build:
			      needs: test
			      runs-on: ubuntu-latest
			      steps:
			        - name: Get code
			          uses: actions/checkout@v3
			        - name: Cache dependencies
			          id: cache
			          uses: actions/cache@v3
			          with:
			            path: node_modules
			            key: deps-node-modules-${{ hashFiles('**/package-lock.json') }}
			        - name: Install dependencies
			          if: steps.cache.outputs.cache-hit != 'true'
			          run: npm ci
			        - name: Build website
			          id: build-website
			          run: npm run build
			        - name: Upload artifacts
			          uses: actions/upload-artifact@v3
			          with:
			            name: dist-files
			            path: dist
			    deploy:
			      needs: build
			      uses: ./.github/workflows/reusable.yml
			      with:
			        artifact-name: dist-files
			      # secrets:
			        # some-secret: ${{ secrets.some-secret }}
			    print-deploy-result:
			      needs: deploy
			      runs-on: ubuntu-latest
			      steps:
			        - name: Print deploy output
			          run: echo "${{ needs.deploy.outputs.result }}"
			    report:
			      needs: [lint, deploy]
			      if: failure()
			      runs-on: ubuntu-latest
			      steps:
			        - name: Output information
			          run: | 
			            echo "Something went wrong"
			            echo "${{ toJSON(github) }}"
			  
			  ```
-
- Summary
	- ### Conditional Jobs & Steps（条件作业和步骤）
		- **控制步骤或作业执行**：使用 `if` 和动态表达式来控制步骤或作业的执行。
		- **改变默认行为**：可以使用 `failure()`、`success()`、`cancelled()` 或 `always()` 来改变默认行为。
		- **忽略步骤失败**：使用 `continue-on-error` 来忽略步骤失败。
	- ### Matrix Jobs（矩阵作业）
		- **并行运行多个作业配置**：能够在并行运行多个作业配置。
		- **添加或删除个别组合**：可以添加或删除个别组合。
		- **控制单个失败作业的影响**：通过 `continue-on-error` 控制单个失败作业是否应取消所有其他矩阵作业。
	- ### Reusable Workflows（可复用工作流程）
		- **通过 `workflow_call` 事件复用工作流程**：工作流程可以通过 `workflow_call` 事件进行复用。
		- **复用任何逻辑**：可以复用任何逻辑（包括尽可能多的作业和步骤）。
		- **使用输入、输出和机密信息**：可以根据需要使用 `inputs`、`outputs` 和 `secrets`。
	- 这些功能使得 GitHub Actions 在构建和部署自动化过程中更加灵活和高效。通过条件作业和步骤，开发者可以细粒度地控制作业执行逻辑；通过矩阵作业，开发者可以并行测试多种配置组合；通过可复用工作流程，开发者可以复用通用任务，从而减少重复代码并提高工作流程的可维护性。