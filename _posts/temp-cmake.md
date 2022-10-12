---
title: cmake学习
tag: [cmake,C++]
key: 2020-12-20-cmake学习
---

Refer:

- [CMake examples 中文版](https://sfumecjf.github.io/cmake-examples-Chinese)

cmake其实就是把链接-构建-编译过程用代码写下来。

```bash
mkdir build
cd build/
cmake ..
make VERBOSE=1
```

CMakeLists.txt

- 每个函数都需要固定参数

- 函数之间需要一定顺序，比如`target_include_directories()`为一个target（可能是一个库library也可能是可执行文件）添加头文件路径，因此在此之前必须有个target。