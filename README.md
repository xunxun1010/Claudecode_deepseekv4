# Claudecode_deepseekv4
在vscode中使用模型为deepseeek-v4-pro的Claudecode  
1.下载Claude Code for VS Code插件  
<img width="960" height="1031" alt="image" src="https://github.com/user-attachments/assets/389a9ad1-5052-47bb-b2c3-0f6f198b445a" />
2.绕开Claude的登陆界面
在settings.json中输入以下命令
```
"terminal.integrated.fontFamily": "monospace",
    "claudeCode.preferredLocation": "panel",
    "claudeCode.environmentVariables": [
        { "name": "ANTHROPIC_BASE_URL", "value": "https://xxxx" }, 
        { "name": "ANTHROPIC_AUTH_TOKEN", "value": "xxxx" }
```
3.下载ccswitch
配置deepseekapi，注意要选择    
<img width="825" height="140" alt="image" src="https://github.com/user-attachments/assets/d741dc84-5cac-4b8a-8dc8-8ccab3ea5dab" />
4.测试
