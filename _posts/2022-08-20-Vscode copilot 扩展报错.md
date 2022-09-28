---
title: Vscode copilot 扩展报错
tag: [vscode]
key: 2022-08-20-Vscode copilot 扩展报错
---



GitHub Copilot could not connect to server. Extension activation failed: "getaddrinfo EAI_AGAIN api.github.com"

## 解决方案一：修改 DNS 或 HOST

[GitHub Copilot could not connect to server ,VsCode 驾驶员 Copilot 报错解决方法~](https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F522807476)

测试 `nslookup api.github.com` 果然无法解析域名

但我是在远程服务器上使用，没有 sudo 权限，**无法修改 DNS 和 host**

> The whole point of the permissions system is preventing non-privileged users and attackers from doing stuff like writing to system files, running hostile code, and so on.

## 解决方案二：配置 vscode remote workspace 代理

紧接着参考：

[Extension proxy support · Issue #12588 · microsoft/vscode](https%3A%2F%2Fgithub.com%2Fmicrosoft%2Fvscode%2Fissues%2F12588)

在 vscode 选择 `首选项`-`选择远程工作区`-`搜索proxy`，可以看到 `HTTP: proxy` 和 `HTTP: proxy Support` 来为 vscode-server 和其上的扩展配置代理

然而发现对 copilot 并不起作用

## 最后

参考

- [Can't use Copilot with Proxy · Discussion #13113 · github-community/community](https%3A%2F%2Fgithub.com%2Fgithub-community%2Fcommunity%2Fdiscussions%2F13113)
- [pycharm login in github copilot，Always show waiting for github authentication · Discussion #16960 · ](https%3A%2F%2Fgithub.com%2Forgs%2Fgithub-community%2Fdiscussions%2F16960)

[error: "unable to get local issuer certificate" behind zScaler proxy · Discussion #8866 · github-com](https%3A%2F%2Fgithub.com%2Forgs%2Fgithub-community%2Fdiscussions%2F8866)

[Copilot in IntelliJ IDEA crashed when logging in to server · Discussion #16230 · github-community](https%3A%2F%2Fgithub.com%2Forgs%2Fgithub-community%2Fdiscussions%2F16230)

发现 copilot 扩展不支持代理，我尝试安装旧版本的 copilot 依旧无效

# Intellij 安装 copilot 也出问题

> GitHub Copilot: Failed to initiate the GitHub login process. Please try again.

[when login to github  in goland  show Failed to initiate the GitHub login process. Please try again ](https%3A%2F%2Fgithub.com%2Forgs%2Fgithub-community%2Fdiscussions%2F16965)

可以开个代理，再执行 `ipconfig /flushdns`，接下来就登录成功了
