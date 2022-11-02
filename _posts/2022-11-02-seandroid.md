---
title: Android安全特性-SEAnroid
tag: [selinux, android, security]
key: 2022-11-02-seandroid
permalink: /android-sec/seandroid.html
sidebar:
  nav: android-sec
---


# 1. DAC与MAC

## 1.1 DAC

SELinux出现之前，Linux上的安全模型叫DAC

> DAC（Discretionary Access Control，自主访问控制）。

Linux的DAC基于用户/用户组 + 权限标记，核心思想是：

- 进程理论上所拥有的权限与执行它的用户的权限相同。即命令`ll`下的rwx权限项。

	> Android中的[应用沙盒](https://source.android.com/docs/security/app-sandbox)就是基于 UID 的原始自主访问控制 (DAC) 沙盒

DAC系统通常比较粗放，并且容易出现无意中提权的问题：比如root进程会拥有一切权利。

## 1.2 MAC

因此出现了SELinux，适用于 Linux 操作系统的MAC系统。

> - SELinux（Security-Enhanced Linux，安全增强型Linux）
> - MAC（Mandatory Access Control，强制访问控制）

在 DAC 系统中，存在所有权的概念，即特定资源的所有者可以控制与该资源关联的访问权限。与DAC不同，MAC 系统则会在每次收到访问请求时都先咨询`核心机构`，再做出决定。

在SELinux中`核心机构`的规则便由安全策略配置文件确定。

# 2. SELinux相关基础
## 2.1 SELinux介绍

关于SELinux的直观介绍可以看看[这篇动漫](https://opensource.com/business/13/11/selinux-policy-guide)，也可以直接看[中文翻译](https://www.jianshu.com/p/1d045e60cdfa)。

---

SELinux是基于Linux 安全模块 ([LSM](https://en.wikipedia.org/wiki/Linux_Security_Modules)) 框架实现的。

> Linux安全模块（ Linux Security Module, LSM）框架提供了一种机制，可以将各种安全检查与新的内核扩展挂钩(hook)。
>
> LSM框架可识别各种内核对象以及对这些对象执行的敏感操作，其中每项操作要执行时，系统都会调用 LSM 钩子函数，来确定是否应允许执行相应操作。

SElinux是一个标签系统。系统中的**每个进程、每个文件/目录对象，甚至每个网络端口、设备以及每个可能的主机名**都会被打上一个标签。通过编写标签相关的规则来控制带标签进程访问带标签的资源，如一个文件。这个规则被称为“策略”。这些规则由kernel来强制执行。

### 2.2 SELinux三种模式

- Disabled：禁用
- Permissive：宽容模式 - 仅记录但不强制执行 SELinux 安全政策。
- enforcing：强制模式 - 强制执行并记录安全政策。如果失败，则显示为 EPERM 错误。

### 2.3 SELinux策略规则

即SeLinux Policy Rules，用于指导SELinux。

以 `*.te` 结尾的文件是 SELinux 策略源代码文件，用于定义域及其标签。更新.te文件后需要编译。

> TE（Type Enforcement，类型强制执行）

采用以下格式：`allow source target:class permissions;`，其中：

- *source* - 规则主体subject的类型（或属性）。谁正在请求访问权限。
- *target* - 对象object的类型（或属性）。对哪些内容提出了访问权限请求。
- *class* - 要访问的对象（例如，文件、套接字）的类型。
- *permission* - 要执行的操作（或一组操作，例如读取、写入）。

示例如下，表示应用可以读取和写入带有 `app_data_file` 标签的文件。

```plaintext
allow untrusted_app app_data_file:file { read write };
```

### 2.4 安全上下文

安全上下文（Security Context），也称作标签（Security Label）是 SELinux 用于对启用SELinux的系统上的资源（如进程和文件）进行分类的机制。

![img](https://xdo0.github.io/imgsrc/300px-SELinux-context.png)

# 3. Android上的SELinux

关于Linux上的SELinux可以参考这篇：[SELinux管理 (biancheng.net)](http://c.biancheng.net/linux_tutorial/18/)，而Android上还是参考官方文档——[自定义 SELinux  -AOSP](https://source.android.com/docs/security/features/selinux/customize)。

接下来简单介绍Android端的SELinux：

## 3.1 相关命令

**环境：RedMi Note7 Pro**

### 启用关闭SELinux

```bash
# 查看SELinux状态
getenforce
# 暂时关闭
setenforce 0
# 设置为Permissive
setenforce 1
```

### 查看安全上下文

安全上下文的格式为：`user:role:type:sensitivity[:categories]`。Android中通常可以忽略上下文的 `user`、`role` 和 `sensitivity` 字段。
{:.info}

> Android 并不会使用 SELinux 提供的所有功能。
>
> - 不使用 SELinux 用户。唯一定义的用户是 [`u`](https://android.googlesource.com/platform/system/sepolicy/+/refs/heads/master/private/users)。必要时，系统会使用安全上下文的类别字段表示实际用户。
> - 不使用 SELinux 角色和基于角色的访问权限控制 (RBAC)。定义并使用了两个默认角色：[`r`](https://android.googlesource.com/platform/system/sepolicy/+/refs/heads/master/private/roles_decl)（适用于主体subjects，即进程）和 `object_r`（适用于对象objects，即资源）。
> - 不使用 SELinux 敏感度。已始终设置好默认的 `s0` 敏感度。
> - `categories` 是 SELinux 中[多级安全 (MLS)](https://github.com/SELinuxProject/selinux-notebook/blob/main/src/mls_mcs.md#multi-level-and-multi-category-security) 支持的一部分。从 Android S 开始，类别被用于：
>   - 分隔应用数据，使其不被其他应用访问。
>   - 分隔不同实际用户的应用数据。

- 查看进程：`ps -Z`

![image-20221102172116009](https://xdo0.github.io/imgsrc/image-20221102172116009.png)

- 查看文件：`ls -Z`

![image-20221102172009383](https://xdo0.github.io/imgsrc/image-20221102172009383.png)

另外在`/system/etc/selinux`，`/system/vendor/etc/selinux`等也可以查看上下文，如下图。

![image-20221102222249211](https://xdo0.github.io/imgsrc/image-20221102222249211.png)

其中的.cil文件编写 SELinux 模块新的中间层语言[^5]。

![img](https://xdo0.github.io/imgsrc/cil.png)

### 查看SELinux日志

SELinux中尝试进行的所有违规行为都会被内核记录到 `dmesg` 和 `logcat`。

违规行为会触发AVC异常。

> 当应用或进程（称为主体）发出访问对象（如文件）的请求时，SELinux 会检查AVC（Access Vector Cache，访问向量缓存），其中缓存有主体和对象的访问权限。

[^1]: [Android 中的安全增强型 Linux - Android开源项目](https://source.android.com/docs/security/features/selinux)
[^2]: [Android : SELinux 简析&修改 ](https://www.cnblogs.com/blogs-of-lxl/p/7515023.html)
[^3]: [SELinux管理 (biancheng.net)](http://c.biancheng.net/linux_tutorial/18/)
[^4]: [Selinux安全上下文详解_51CTO博客_selinux](https://blog.51cto.com/u_11970509/2320779)
[^5]:[SELinux CIL - Runsisi's Blog](https://runsisi.com/2020/01/02/selinux-cil/)

