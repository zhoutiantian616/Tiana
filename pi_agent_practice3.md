# Pi Agent源码获取、依赖安装与本地编译指导  
## 任务一：实验准备  
检查并准备 Node.js、npm 和 Git，记录实际版本并截图。

<img width="805" height="229" alt="image" src="https://github.com/user-attachments/assets/e2510099-6926-463e-ac58-f091f82820a3" />  

## 任务三 阅读配置，确认安装与编译的依据  
用编辑器打开仓库根目录，查看以下文件：  
README.md：	Development 中提供了哪些开发命令  
package.json：	engines、workspaces、scripts、devDependencies  
package-lock.json：	锁定依赖版本和依赖关系的文件  
packages/：	同一仓库中的多个子包  
packages/coding-agent/package.json：	CLI 包的构建脚本和 bin 入口  

engines.node定义:  

<img width="518" height="127" alt="image" src="https://github.com/user-attachments/assets/8fd3d90d-b4b3-465c-939e-d526205e5fd0" />  

scripts.build定义:  

<img width="1705" height="1372" alt="image" src="https://github.com/user-attachments/assets/4721e394-6cf5-45bb-955a-c6b2dcb3d9bb" />  

回答问题：  
1. Node.js 版本要求在哪里？  
根 package.json 中第 69–71 行的engines.node，值为>=22.19.0；  
2. npm run build 实际从哪里读取命令？  
根 package.json 第 15 行的`scripts.build` ，它本身只是 11 步`cd … && npm run build` 的调度链；每进一个子包，npm 读的是该包自己的`scripts.build` ，编译参数再由各包`tsconfig.build.json` 决定。

## 任务四 安装项目依赖  
执行安装：
确认当前目录包含根 package.json，执行：npm install --ignore-scripts  
该命令主要完成以下工作：  
- 根据 package.json 和 package-lock.json 解析、下载并安装依赖；锁文件与声明一致时使用锁定版本。  
- 在本地建立 node_modules 等依赖目录，并关联工作区中的本地包。  
- 安装项目需要的库和开发工具，例如编译器、打包工具。  
--ignore-scripts 跳过安装阶段自动触发的生命周期脚本，例如 postinstall 和 prepare；不影响后续主动执行 npm run build。  
这里没有 -g，安装的是当前项目的依赖。它不会把本地修改后的 Pi 自动安装成全局 pi 命令。命令来源见 Development，参数含义见 npm install 文档。

<img width="2323" height="716" alt="image" src="https://github.com/user-attachments/assets/9f99228b-31f4-40fc-a7ac-7096a38adc88" />  

## 任务五 本地编译与构建产物验证  
1. 首次构建，仍在仓库根目录执行：npm run build  
首次构建需要联网获取模型目录数据。这里的“模型数据”指模型目录、能力等配置数据，不是下载大模型权重。本地客户端构建与模型推理服务是两件事。依据：README、AI 包脚本。  
2. 检查并运行构建产物：  
Test-Path .\packages\coding-agent\dist\bundle\cli.js
node .\packages\coding-agent\dist\bundle\cli.js --version  
第一条应输出 True，第二条应输出当前源码对应的 Pi 版本。这个入口来自 CLI 包的 bin.pi 配置，见 packages/coding-agent/package.json。  

dist 保存构建产物。以后修改程序时应修改 src 下的源码，再重新构建；直接修改 dist 中的文件，可能在下次构建时被覆盖。  

<img width="1709" height="456" alt="image" src="https://github.com/user-attachments/assets/4f338d34-1897-4d3d-a36f-9acf7aea761b" />  

## 任务六 启动本地源码版本并完成一次会话  
1. 先验证源码入口
在仓库根目录执行：.\pi-test.bat --version  
这个脚本名称包含 test，但它是运行本地源码的入口，不是执行整个项目测试套件。当前版本的 .bat 调用 .ps1，由 Node.js 加载源码解析器并运行 packages/coding-agent/src/cli.ts。依据：pi-test.bat、pi-test.ps1。  

2. 启动交互界面：.\pi-test.bat
本次使用上述命令验证本地源码。直接输入 pi，可能启动第一次实践课安装的全局版本，无法单独证明运行的是刚下载的源码。  
本地源码版默认与已安装的 Pi 共用当前用户的配置目录 ~/.pi/agent。因此，第一次实践课已完成登录且凭据仍有效的同学，运行 .\pi-test.bat 后通常无需再次 /login。这表示复用了用户配置，并不表示运行的是全局安装版本。只有未配置模型、凭据失效或需要更换账户时，才需重新配置。  

发送一个简单问题，例如：  
请用一句话说明源码和编译产物的区别。只回答问题，不修改文件。  
确认实际收到模型回复。模型能否回复取决于账户、网络和模型服务配置；若失败，应将“本地程序启动成功”和“模型调用失败”分别记录，不能把两者混为编译失败。  

<img width="2296" height="704" alt="image" src="https://github.com/user-attachments/assets/63ba2415-a943-4104-83db-46109df33529" />  

回答问题：  
1. 本次如何确保运行的是本地源码？  
用仓库根目录的`.\pi-test.bat` 启动，不要直接输入`pi` ——`pi` 可能是装的全局版本。看脚本内部就清楚了： pi-test.ps1 用`node --import source-resolver.ts` 加载源码解析器，然后直接执行`packages/coding-agent/src/cli.ts` ，既不读全局安装，也不经过`dist` 。再配合截图里的启动命令和`git rev-parse HEAD` 记录，就能说明运行的是本次 clone 的这份源码。

2. 源码入口与 dist/bundle/cli.js 入口分别适合什么场景？  
- 源码入口（`pi-test.bat` →`src/cli.ts` ） ：适合开发调试，改完代码直接启动就生效，不用重新构建；代价是启动慢一些，且依赖`node_modules` 和源码齐全。
- `dist/bundle/cli.js` ：构建产物，接近发布形态，启动快、自包含，适合验证构建是否成功、演示和日常使用；缺点是它不会反映构建之后对源码的修改，改了源码必须重新构建。

