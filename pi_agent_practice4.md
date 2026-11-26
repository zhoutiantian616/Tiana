# 实践课4：Pi Agent 的模型调用与运行流程  
在实践课 3 已构建的 Pi 项目中，加两行日志并重新编译。通过对比直接问答和读取文件，观察 Pi 什么时候请求模型，以及工具结果如何进入下一轮对话。  

## 任务一：给 Agent 循环加两行日志  
打开 packages/agent/src/agent-loop.ts。以下是教师演示版本的修改前行号。如果小组版本的行号不同，请搜索对应语句。可以先改靠后的第 412 行，再改第 284 行。  

1. 在模型调用前打印日志  
找到原第 412 行的 const response = await streamFunction(...)，在它上方插入一行。下方展示调用的开头，保留原有参数和后续代码。  
console.error("[P4] Agent 即将请求模型"); // 新增  
const response = await streamFunction(config.model, llmContext, {  
每次执行到这里，Agent 都准备把当前上下文交给模型调用函数。

<img width="1416" height="643" alt="image" src="https://github.com/user-attachments/assets/5ba3446f-0fbc-4b9a-8c27-98a5150f973d" />  

2. 在工具结果加入上下文后打印日志  
找到原第 284 行的 currentContext.messages.push(result);，在它下方插入一行。日志写在 for 循环内，每写入一条工具结果就打印一次。  

<img width="1442" height="751" alt="image" src="https://github.com/user-attachments/assets/01803bee-05ef-41dc-a22d-36b942d83c43" />  

## 任务二：重新编译并启动  
保存源码。在有根目录 package.json 的项目目录中执行：npm run build  

<img width="1571" height="475" alt="image" src="https://github.com/user-attachments/assets/86dd9376-6a74-4d91-b742-de147b2fcb09" />  

<img width="1281" height="409" alt="image" src="https://github.com/user-attachments/assets/3d1da490-cd81-426c-b84c-ab06076ccd83" />

等构建结束且成功返回后，直接启动编译产物：node .\packages\coding-agent\dist\cli.js  
两行 [P4] 日志会写到终端，可能和 Pi 的交互界面混在一起。运行时留意终端输出，记录出现的顺序。  

<img width="2357" height="1623" alt="image" src="https://github.com/user-attachments/assets/2e4ec32c-428a-4b5b-a2ef-196edae48d35" />

## 任务三：对比两次运行  
在项目根目录创建 practice4.txt，写入一行自定实验代号。使用同一模型，分别在两个新会话中输入以下指令。第二条指令只给文件名，把代号留在文件里。  
A：直接问答	请直接用一句话解释什么是 Git 分支。	观察是否执行了工具，出现了几条“即将请求模型”日志  
- 输入：`请直接用一句话解释什么是 Git 分支。`  
- 是否调用工具：否。  
- [P4] 日志（按顺序）：`Agent 即将请求模型` ，共 1 条。  
- 最终回复：Git 分支是一条可独立提交和推进的代码开发线，能在不影响主线的情况下并行修改代码，之后还能合并回主线。  
- 模型调用轮数：1 轮。

<img width="2330" height="635" alt="image" src="https://github.com/user-attachments/assets/ff98097a-72f7-4915-a1b9-0dd8ec7c4b37" />

B：读取文件	请用 read 工具读取当前目录的 practice4.txt，并告诉我文件中的实验代号。观察	Pi 是否读到了自己的代号，两类日志按什么顺序出现  
pi读取到了实验代号，两类日志先出现“工具结果已加入上下文”，再出现“Agent 即将请求模型”。  
- 输入：`请用 read 工具读取当前目录的 practice4.txt，并告诉我文件中的实验代号。`  
- 是否调用工具：是，调用`read` 一次，读取`practice4.txt` 。  
- [P4] 日志（按顺序）：`Agent 即将请求模型` →`工具结果已加入上下文` →`Agent 即将请求模型` 。截图里能看到后两条，第一条在重绘时被覆盖了。  
- 最终回复：文件`practice4.txt` 中的实验代号是`110` 。  
- 模型调用轮数：2 轮。

<img width="2311" height="636" alt="image" src="https://github.com/user-attachments/assets/cd73ca96-8f1d-485f-83d9-07ac022805e7" />

