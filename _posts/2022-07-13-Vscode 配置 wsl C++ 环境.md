---
title: Vscode 配置 wsl C++ 环境
tag: [vscode, wsl]
key: 2022-07-13-Vscode 配置 wsl C++ 环境
modify_date: 2022-11-09
---

本文仅供参考，更详细的配置还是去看[官网]([Debugging in Visual Studio Code](https://code.visualstudio.com/docs/editor/debugging#_start-debugging))。

因为笔试要用到 C++，之前一直用 visual studio 的，但是笔试一般就写个代码片段，不需要大工程，因此选择 vscode 配合 wsl，选 wsl 是因为安装包方便

# CMake调试配置

Ref: [VSCode – How To Debug A WSL C++/CMake Project (matgomes.com)](https://matgomes.com/debugging-wsl-c-cmake-vscode/)

## 安装依赖

`apt-get`安装以下包：

- **GCC/G++** compiler: the C++ compilation tools.
- **GDB** debugger: the equivalent debugger for your compiler.
- **Make**: build system to be used by CMake.
- **CMake**

## vscode配置

### CMake扩展配置

1. `CTRL + SHIFT + P`输入`>CMake: Select a Kit`来选择编译器，比如gcc
2. 输入`>CMake: Configure`创建build目录
3. `>CMake: Build`进行编译

### 添加launch.json

1. 选择左侧调试的按钮，创建launch.json
2. vscode会自动检查debug的环境，生成launch.json，其中的配置可能多余也可能不全
3. 接下来就是根据需要修改launch.json：

- 具体的环境变量在[这里](https://code.visualstudio.com/docs/editor/variables-reference)查看

- json中type的列表使用IntelliSense的提示`CTRL+SPACE`或`CTRL+I`来查看

> 许多IDE都是`CTRL+SPACE`提示，但是由于和中文输入法快捷键冲突[^1]，可以直接使用`CTRL+I`或修改vscode快捷键为`CTRL+L`
>

下面是minimalist的launch.json配置[^2]

```json
{
    "configurations": [
        {
            "name": "TestZeroZero", // 配置的名称           
            "type": "cppdbg", 
            "request": "launch", // debug模式，分为launch和attach
            "program": "${workspaceFolder}/build/tests/multiply_test", // 需要修改为编译后生成的二进制文件
            "cwd": "${fileDirname}", // ${fileDirname}指当前打开的文件目录
            "miDebuggerPath": "/usr/bin/gdb", // gdb路径
            "args": [
                "--gtest_filter=MultiplyTests.TestIntegerZero_Zero"
            ],// 参数，根据自己的程序调整
            "environment": [
                {
                    "name": "GTEST_COLOR",
                    "value": "0"
                }
            ]//debug session中的环境变量
        }
    ]
}
```
{: .copyable} 

[^1]:[修改Visual Studio Code的自定义键盘快捷键_汪子熙的博客-CSDN博客_visual studio怎么更改快捷键](https://jerry.blog.csdn.net/article/details/100576073)
[^2]:[VSCode – How To Debug A WSL C++/CMake Project](https://matgomes.com/debugging-wsl-c-cmake-vscode/#:~:text=To%20simplify%20things%20a%20little%20bit%2C%20the%20following%20Json%20block%20shows%20a%20minimalist%20launch.jsonthat%20should%20be%20enough%20to%20debug%20an%20executable.)

# 不使用CMake的工程调试配置

Ref: [VScode 配置 C++ 环境进行编译和调试](https://blog.csdn.net/weixin_42145502/article/details/107455999)

# 单个文件

## Wsl C++ 环境

```bash
apt get install build-essential
apt get install gdb
```

## vscode 配置

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
{: .copyable} 

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
{: .copyable} 

