# Pi Agent源码获取、依赖安装与本地编译指导  
## 任务一：实验准备  

<img width="805" height="229" alt="image" src="https://github.com/user-attachments/assets/e2510099-6926-463e-ac58-f091f82820a3" />  

## 任务三 阅读配置，确认安装与编译的依据  
engines.node:  

<img width="518" height="127" alt="image" src="https://github.com/user-attachments/assets/8fd3d90d-b4b3-465c-939e-d526205e5fd0" />  

scripts.build:  

<img width="1705" height="1372" alt="image" src="https://github.com/user-attachments/assets/4721e394-6cf5-45bb-955a-c6b2dcb3d9bb" />  

1.Node.js 版本要求在哪里？  
根 package.json 中第 69–71 行的engines.node，值为>=22.19.0；  
2.npm run build 实际从哪里读取命令？  
根 package.json 第 15 行的`scripts.build` ，它本身只是 11 步`cd … && npm run build` 的调度链；每进一个子包，npm 读的是该包自己的`scripts.build` ，编译参数再由各包`tsconfig.build.json` 决定。  
## 任务四 安装项目依赖  
<img width="2323" height="716" alt="image" src="https://github.com/user-attachments/assets/9f99228b-31f4-40fc-a7ac-7096a38adc88" />


