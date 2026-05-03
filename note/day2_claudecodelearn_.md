来源：  
1.[ClaudeCode从0到1全攻略：MCP/SubAgent/Agent Skill/Hook/图片/上下文处理/后台任务](https://www.bilibili.com/video/BV14rzQB9EJj?vd_source=e5ec6153142cd56468f2c45b7eb9666e)  
2.[ClaudeCode从0到1全攻略,文字版](https://mp.weixin.qq.com/s/45q-5Gzz-VlvqkxFI2vdIQ)  
3.[claude code doc](https://code.claude.com/docs/en/overview)  
本文涉及内容:  
1.基础交互 2.复杂任务处理 3.终端控制 4.回滚与上下文管理 5.图片与 Figma 设计稿还原   
6.CLAUDE.md 项目记忆 7.Hook 8.Agent Skill 9.SubAgent 10.Plugin
## 目录

- [part1 实践：做一个代办软件](#part1-实践做一个代办软件)
- [part2 模式与权限控制](#part2-模式与权限控制)
  - [1.三种模式：](#1三种模式)
  - [2.举例](#2举例)
- [part3 Claude Code 里直接执行终端命令：这是它和普通聊天工具最大的区别之一](#part3-claude-code-里直接执行终端命令这是它和普通聊天工具最大的区别之一)
- [part4 为什么自动模式下它还会向你确认命令？](#part4-为什么自动模式下它还会向你确认命令)
  - [因为文件编辑和终端命令不是一个风险等级](#因为文件编辑和终端命令不是一个风险等级)
- [part5 最高权限模式：dangerously-skip-permissions](#part5-最高权限模式dangerously-skip-permissions)
- [part6 重构架构：从 HTML 单文件升级到 React + TypeScript + Vite](#part6-重构架构从-html-单文件升级到-react--typescript--vite)
- [part7 当前终端一旦被开发服务占住，Claude Code 就没法继续正常处理你的新输入。](#part7-当前终端一旦被开发服务占住claude-code-就没法继续正常处理你的新输入)
- [part8 回滚](#part8-回滚)
- [part9 MCP](#part9-mcp)
- [part10 常用设置](#part10-常用设置)
- [part11 CLAUDE.md](#part11-claudemd)
- [part12 项目级记忆和用户级记忆要分开理解](#part12-项目级记忆和用户级记忆要分开理解)
- [part13 hook：把重复动作自动化](#part13-hook把重复动作自动化)
  - [1.什么是hook](#1什么是hook)
  - [2.hook适合做什么](#2hook适合做什么)
  - [3.hook的思维方式](#3hook的思维方式)
- [part14 agentskill：把常用套路沉淀成随时可调用的能力：高频、固定套路、对格式要求强。](#part14-agentskill把常用套路沉淀成随时可调用的能力高频固定套路对格式要求强)
  - [eg：日报 Skill](#eg日报-skill)
- [part15 subagent：不是模板，而是“分身”](#part15-subagent不是模板而是分身)
  - [1.SubAgent 适合什么场景？](#1subagent-适合什么场景)
  - [2.如何理解 SubAgent？](#2如何理解-subagent)
- [part16 Plugin：把 Skill、SubAgent、Hook 等能力打包成一键安装包](#part16-plugin把-skillsubagenthook-等能力打包成一键安装包)
  - [1.Plugin 的价值在哪里？](#1plugin-的价值在哪里)
  - [2.前端设计类 Plugin 为什么特别值得关注？](#2前端设计类-plugin-为什么特别值得关注)
- [part17 建议](#part17-建议)
  - [1.小任务直接做，大任务先 Plan Mode](#1小任务直接做大任务先-plan-mode)
  - [2.把 dangerously-skip-permissions 当作实验环境加速器，不要当默认配置](#2把-dangerously-skip-permissions-当作实验环境加速器不要当默认配置)
  - [3.尽早写好 CLAUDE.md](#3尽早写好-claudemd)
  - [4.重复性输出用 Skill，重分析任务用 SubAgent](#4重复性输出用-skill重分析任务用-subagent)
  - [5.真正想做设计还原，尽量接 MCP](#5真正想做设计还原尽量接-mcp)
  - [6.回滚只是应急，Git 仍然是正式版本控制核心](#6回滚只是应急git-仍然是正式版本控制核心)
- [总结：你如何设计自己的 AI 开发工作流。](#总结你如何设计自己的-ai-开发工作流)



## part1 实践：做一个代办软件
创建文件夹
```base
mkdir my-todo
cd my-todo
```
启动
```
claude
```
提出一个清晰的需求：给我做一个待办软件，使用 HTML 实现  
<img width="612" height="327" alt="image" src="https://github.com/user-attachments/assets/befa48b5-c1ea-4adf-bc4c-a958454c6244" />
* 是否允许本次创建文件？  
* 是否允许本次会话期间后续所有编辑自动通过？  
* 或者拒绝这次修改？  

效果：
<img width="561" height="735" alt="image" src="https://github.com/user-attachments/assets/bcd92823-7783-4f42-980c-94d487497fe2" />


## part2 模式与权限控制
### 1.三种模式：  <img width="627" height="252" alt="image" src="https://github.com/user-attachments/assets/0dc897fb-6d9c-40e4-903a-1dc6c9a16a66" />

1.默认模式(? for shortcuts)：最适合第一次进项目  
默认模式下，Claude Code 每次创建文件、修改文件，都会先征求你的同意。  
2.自动模式(accept edits on) :  
Claude Code 后续在当前会话中对文件的创建和编辑，就不再反复询问。自动模式只代表“文件编辑自动通过”，不代表它执行所有终端命令也自动通过。  
3.规划模式(Plan Mode)：复杂任务先讨论，不直接开改
开启后，它不会直接修改文件，而是先：  
* 理解目标  
* 拆分步骤  
* 设计目录结构  
* 说明实施方案  
* 给出执行计划

非常适合： 
* 架构重构  
* 技术栈迁移  
* 复杂页面改造  
* 大模块新增  
* 需求还没完全想清楚的时候  

### 2.举例
比如你已经有了一个单文件 index.html 的待办应用，但你觉得这种结构不适合继续扩展，想改造成：React + TypeScript + Vite  
切换到规划模式：请将当前待办应用重构为 React + TypeScript + Vite 项目，保留现有功能，并保持 UI 风格一致。  
在规划模式下，Claude Code 会先产出一份方案。  
这份方案里通常会包括：
* 改造目标  
* 推荐目录结构  
* 需要新增哪些文件  
* 原功能如何迁移  
* 状态管理怎么做  
* 可能需要的测试点  

然后它会问你：
* 直接执行计划  
* 执行但保留谨慎权限  
* 继续修改计划  

## part3 Claude Code 里直接执行终端命令：这是它和普通聊天工具最大的区别之一
在 Claude Code 里，可以输入 ! 号进入 Bash 模式并执行命令。比如你已经让它生成了 index.html，现在想直接打开这个文件看效果，就不一定非要去文件管理器手工双击。
```
! start index.html
```
查看当前目录下文件
```
ls
```
tips：Claude Code 的真正威力，不是生成一段代码，而是能帮助推进整个开发链路。
## part4 为什么自动模式下它还会向你确认命令？
### 因为文件编辑和终端命令不是一个风险等级

## part5 最高权限模式：dangerously-skip-permissions
不建议使用
## part6 重构架构：从 HTML 单文件升级到 React + TypeScript + Vite
在规划模式下，claudecode会先给一份计划  
<img width="627" height="332" alt="image" src="https://github.com/user-attachments/assets/df9525bc-f6db-454a-a070-3875f5184e0e" />

如果觉得还不够完整，可以继续补要求，让它重新整理计划。只有对方案满意时，再让它真正执行。这一点要养成习惯：小活儿直接做，大活儿先出计划。
## part7 当前终端一旦被开发服务占住，Claude Code 就没法继续正常处理你的新输入。
npm run dev 这类命令通常是持续运行的。它会一直监听文件变化、持续占用前台终端。所以如果你在同一个会话里还想继续让 Claude Code 改代码、回答问题、追加功能，就必须把这个服务放到后台。  
解决方法：CTRL+b放到后台，如果后面你想结束这个服务，也可以在任务管理界面中停止它。

## part8 回滚
Claude Code 的回滚不是 Git，擅长撤销“自己写入的内容”，但对那些通过终端命令产生的，不能完整恢复。

## part9 MCP
MCP 是让大模型连接外部工具和外部上下文的一种标准方式。
## part10 常用设置  
claude -c:直接恢复最近一次会话  
/compact：保留重点，压缩无关噪音   
/clear：彻底清空上下文  
## part11 CLAUDE.md  
项目记忆文件 CLAUDE.md  
可以先让Claude Code 帮你初始化一份 CLAUDE.md，然后你自己继续补充内容。  
<img width="923" height="366" alt="image" src="https://github.com/user-attachments/assets/d9cba031-2d24-4213-acc8-716f4f2ee287" />

这个文件非常适合放：
* 项目背景
* 技术栈
* 目录约束
* 开发规范
* 输出语言
* 风格偏好
* 注意事项

这样 Claude Code 每次进入这个项目时，就会读取这些规则。

## part12 项目级记忆和用户级记忆要分开理解
/memory 会有两层：  
<img width="849" height="347" alt="image" src="https://github.com/user-attachments/assets/ad70f286-b05b-48b7-82c7-fd1de2046bed" />  
1.项目级 ./CLAUDE.md  
只对当前项目生效。适合写这个项目自己的规则。  
2.用户级 ~/.claude/CLAUDE.md  
对你这个用户整体生效。适合写你个人长期不变的偏好。  
## part13 hook：把重复动作自动化 
### 1.什么是hook
当 Claude Code 完成某个动作时，自动触发你预先定义的一段逻辑。最典型的场景，就是代码格式化。  
<img width="1185" height="393" alt="image" src="https://github.com/user-attachments/assets/3950cbb7-a8ca-4f7f-acad-d3e2792277af" />
比如你不想每次都手工运行格式化工具。那么就可以配置一个 Hook：只要 Claude Code 刚刚创建或编辑了文件，就自动执行一次格式化。这个逻辑非常适合做成 Hook。
### 2.hook适合做什么
* 运行 prettier
* 运行 lint
* 自动补充文件头
* 执行测试
* 生成日志
* 发送通知
* 校验命名规则
把“每次都得手动做的事”变成自动执行的流程。
### 3.hook的思维方式
设计 Hook 时，其实只需要想清楚三件事：
<img width="642" height="189" alt="image" src="https://github.com/user-attachments/assets/848c3ca4-c725-457e-a2fd-45e2bd5be2ec" />  

## part14 agentskill：把常用套路沉淀成随时可调用的能力：高频、固定套路、对格式要求强。
有些任务不是很复杂，但重复频率很高，而且每次格式都差不多。例如：
* 每日总结
* 周报
* 会议纪要
* 发布说明
* Bug 报告
* Code Review 模板
* PR 描述

agentskill：一份给 Claude Code 看的能力说明书。它通常包含两部分：  
* 第一部分：元信息  
名称  
描述  
什么时候适合调用
* 第二部分：具体要求
输出格式  
固定结构  
语气要求  
需要包含哪些信息  
# eg：日报 Skill
让 Claude Code 按固定格式帮你写日报，那你就可以创建一个类似 daily-report 的 Skill。
## part15 subagent：不是模板，而是“分身”
* Skill 更像是“给主助手加了一套固定写法”。和项目上下文有关联
* 而 SubAgent 更像是“给主助手配了一个专门干某类活的分身”。和上下文没有关联
###  1.SubAgent 适合什么场景？
* 代码审核
* 大型目录分析
* 安全检查
* 文档巡检
* 架构审查
* 风险评估

共同特点：
* 中间过程很多
* 可能要读大量代码
* 需要独立分析
* 不希望把主会话塞得很脏

### 2.如何理解 SubAgent？
主会话是你当前坐在桌前的主工程师。SubAgent 是你临时叫来的某个专项顾问。
* 有自己的任务边界
* 有自己的行为规则
* 有自己的上下文
* 有自己的工具权限
它干完活之后，只把最终审核结果汇报回来。这样，主会话就不会被一堆中间分析过程撑爆。

总结：
<img width="765" height="495" alt="image" src="https://github.com/user-attachments/assets/9a81cff0-b703-4908-9d38-1e2aa3e4399d" />

## part16 Plugin：把 Skill、SubAgent、Hook 等能力打包成一键安装包
安装一个 Plugin，可能就相当于一次性装上了：
* 一个 Skill
* 一个 Hook
* 一个 SubAgent
* 一组配置
甚至一些外部工具接入规则
### 1.Plugin 的价值在哪里？
它最大的价值不是给个人玩，而是给团队复用。比如一个团队里，已经沉淀出一套成熟的前端设计工作流：
* 有固定的界面规范
* 有常用的提示模板
* 有统一的样式策略
* 有审核要求
如果这些东西都靠口口相传，很容易失真。  
但如果能打成 Plugin，团队成员装上就能直接用，效率会高很多。
### 2.前端设计类 Plugin 为什么特别值得关注？
因为 AI 生成前端页面时，经常会出现一种“很像 AI 做的页面”的感觉。  
而一些专门的前端设计类 Plugin，目标就是改善这种共性，让 Claude Code 在做页面时更贴近成熟设计规范，而不是只会输出那种标准化 AI Demo 风格。  
所以如果你是做前端的，或者你经常让 Claude Code 出页面，Plugin 这条线非常值得深入研究。

## part17 建议
### 1.小任务直接做，大任务先 Plan Mode
不要所有任务都一句话扔给 Claude Code 开干。正确做法是：
* 小改动、单功能、单文件：直接做
* 重构、迁移、新架构、大功能：先出计划
这会明显降低返工率。  
### 2.把 dangerously-skip-permissions 当作实验环境加速器，不要当默认配置
我建议不要开
### 3.尽早写好 CLAUDE.md
很多人不是不会用 Claude Code，而是不会给它稳定上下文。把这些信息尽早写进去：
* 技术栈
* 目录约束
* 输出语言
* 项目背景
* 风格偏好
* 不能碰的地方
* 修改前必须先做什么
这是提升稳定性的关键动作之一。
### 4.重复性输出用 Skill，重分析任务用 SubAgent
* 高频格式任务 → Skill
* 深度分析任务 → SubAgent

### 5.真正想做设计还原，尽量接 MCP
文字描述 UI，通常不够准；  
只看截图，通常不够细；  
能接结构化设计信息，才更接近工程可用。  
所以如果你们本来就在用 Figma，尽快研究 MCP，会非常值。  
### 6.回滚只是应急，Git 仍然是正式版本控制核心
Claude Code 可以帮你快速撤销错误方向，但不要因为它有 Rewind，就忽略 Git。真正重要的项目，一定要继续保留：
* 分支
* 提交记录
* 回滚点
* 可审计变更  
Claude Code 是开发搭档，不是版本治理系统。  

## 总结：你如何设计自己的 AI 开发工作流。









