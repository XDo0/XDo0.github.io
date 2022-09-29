---
title: Vscode 配置 wsl C++ 环境
tag: [vscode, wsl]
key: 2022-07-13-Vscode 配置 wsl C++ 环境
---

因为笔试要用到 C++，之前一直用 visual studio 的，但是笔试一般就写个代码片段，不需要大工程，因此选择 vscode 配合 wsl，选 wsl 是因为安装包方便

# Wsl C++ 环境

```bash
apt get install build-essential
apt get install gdb
```

# vscode 配置

1. 安装 C++ 扩展
2. 开启 wsl 窗口
3. 写个 cpp 文件，选择调试，选择 g++ 配置，就会自动生成以下文件

`.vscode` 文件夹下的 `launch.json`

```json
{
    "name": "g++ build and debug active file",
    "type": "cppdbg",
    "request": "launch",
    "program": "${fileDirname}/${fileBasenameNoExtension}",
    "args": [],
    "stopAtEntry": false,
    "cwd": "${workspaceFolder}",
    "environment": [],
    "externalConsole": false,
    "MIMode": "gdb",
    "setupCommands": [
        {
            "description": "Enable pretty-printing for gdb",
            "text": "-enable-pretty-printing",
            "ignoreFailures": true
        }
    ],
    "preLaunchTask": "g++ build active file",
    "miDebuggerPath": "/usr/bin/gdb"

}

```

还有 `task.json`

```json
{
    "tasks": [
        {
            "type": "cppbuild",
            "label": "C/C++: g++ 生成活动文件",
            "command": "/usr/bin/g++",
            "args": [
                "-fdiagnostics-color=always",
                "-g",
                "${file}",
                "-o",
                "${fileDirname}/${fileBasenameNoExtension}"
            ],
            "options": {
                "cwd": "${fileDirname}"
            },
            "problemMatcher": [
                "$gcc"
            ],
            "group": {
                "kind": "build",
                "isDefault": true
            },
            "detail": "调试器生成的任务。"
        }
    ],
    "version": "2.0.0"
}

```
