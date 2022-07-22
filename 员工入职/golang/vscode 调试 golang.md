[TOC]

# vscode 调试 golang

1、配置 vscode launc.json 如下

> 不需要手动配置，点击 debug 调试的时候 vscode 就会默认帮你配置好了

```json
{
    // 使用 IntelliSense 了解相关属性。
    // 悬停以查看现有属性的描述。
    // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
    "version": "0.2.0",
    "configurations": [
        {
			"name": "Launch",
			"type": "go",
			"request": "launch",
			"mode": "auto",
			"program": "${fileDirname}",
			"env": {},
			"args": []
		}
    ]
}
```

2、打断点启动调试