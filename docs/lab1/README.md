# 简介

本次实验我们将学习如何编译和裁剪 Linux 内核。作为操作系统的核心，内核是操作系统中最重要和最基本的部分，这次实验将帮助同学们加深对操作系统的理解和认识。本次实验具体分为下面三个小节：

- 编译 Linux，并裁剪内核
- 创建初始内存盘
- 添加一个自定义的 Linux syscall

每个小节的最后可能有若干**「可选」**练习，不完成这些练习亦可获得满分。

<p style="color:red">注意：你可以通过在页面搜索「可选」来查询所有的此类事项。</p>

实验一完成后，你的 osh-2025-labs 作业仓库应具有这样的结构：

```
osh-2025-labs
- README.md
- lab0
- lab1
  - README.md      // 可选，建议在此文件中介绍你的实验过程
  - syscall
    - bzImage
    - initrd.c
  - riscv/arm/...  // 可选，该目录命名为选择的交叉编译的平台（如选择 arm 则命名为 arm）
    - bzImage      // 交叉编译得到的内核镜像文件
  - bzImage
  - .config        // 可选，编译内核时的配置文件
  - initrd.cpio.gz
```

实验一共分为三个小节，分值各占总分数的 40%、20%、40%。你最多获得 105% 的分数，其中包含 5% 的可选项得分，这一项得分是取所有你完成的「可选」项的最高分。

???+ info "实验平台提示"

    按实验零的要求，我们建议同学们在 vlab 平台或虚拟机上进行课程实验，以免对自己的机器系统造成破坏。

???+ question "如何提问？"

    [提问的智慧](https://lug.ustc.edu.cn/wiki/doc/smart-questions/)

    - 提问前，请先阅读报错信息，或查询必应等搜索引擎；
    - 向助教提问时，请详细描述问题，并提供相关指令及相关问题的报错截图，如果涉及系统问题请提供系统版本；
    - 欢迎大家在实验文档评论区、课程群内讨论实验的相关问题，你的提问可能会解决他人同样的问题。
