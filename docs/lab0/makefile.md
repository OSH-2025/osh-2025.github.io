# Makefile

`Makefile` 是一个用来描述源文件之间的依赖关系以及如何编译和链接这些源文件的脚本文件。它可以大大简化编译过程，使得代码的维护和管理更加方便。我们建议同学们在 lab2 以及 lab3 中使用 `Makefile` 编写编译脚本，以便助教检查实验。
`Makefile` 的功能非常强大，我们在此给出一些基础的介绍以及一些参考资料，如有感兴趣的同学可以参考或自行搜索相关资料。

`Makefile` 有一套专用的语法系统，大致语法规则如下：

```Makefile
target ... : prerequisites ...
    recipe
    ...
    ...
```

**target**
可以是一个 object file（目标文件），也可以是一个可执行文件，还可以是一个标签（label）。

**prerequisites**
生成该target所依赖的文件和/或target。

**recipe**
该 target 要执行的命令（任意的 shell 命令）。

这是一个文件的依赖关系，也就是说，`target` 这一个或多个的目标文件依赖于 `prerequisites` 中的文件，其生成规则定义在 `recipe` 中。

`prerequisites` 中如果有一个以上的文件比 `target` 文件要新的话，`recipe` 所定义的命令就会被执行。

以下是一个简单的 Makefile 示例：

```Makefile
# 定义编译器和编译选项
CC = g++
CFLAGS = -O2
VERSION = -std=c++11

# 定义目标文件和源文件
TARGET = program
SRCS = main.cpp utils.cpp

# 生成可执行文件
$(TARGET): $(SRCS)
    $(CC) $(CFLAGS) $(VERSION) -o $@ $^

# 清理文件
clean:
    rm -f $(TARGET)
```

在这个示例中：

`CC` 定义了使用的编译器（这里是 g++）。
`CFLAGS` 定义了编译选项（这里开启了编译优化选项-O2）。
`VERSION` 指定了编译器使用的 C++版本。
`TARGET` 定义了最终生成的可执行文件的名称。
`SRCS` 定义了源文件的列表。
`$(TARGET): $(SRCS)` 表示 `TARGET` 依赖于 `SRCS` 中的所有源文件。
`$(CC) $(CFLAGS) -o $@ $^` 是编译和链接的命令，`$@` 表示目标文件，`$^` 表示所有依赖文件。
`clean` 是一个伪目标，用于清理生成的可执行文件。
`$`后面紧跟某个变量即为在此处用这个变量的值替换其内容，例如，`$(CC)`将会被替换成变量 CC 的值，即 `g++`。

如果在当前目录只使用 `make` 命令，`Makefile` 会查找到文件中的第一个目标文件，在本示例中为 `$(TARGET)`，即 `program`，并使用 `$(CC) $(CFLAGS) -o $@ $^`，即 `g++ -O2 -o program main.cpp utils.cpp` 编译目标文件。

注：从网页上复制的 Makefile 文件会将 tab 缩进替换为四个空格，复制后请替换回去，以免 Makefile 无法识别命令。

## CMake

`CMake` 是个一个开源的跨平台自动化建构系统，用来管理软件建置的程序，并不依赖于某特定编译器，并可支持多层目录、多个应用程序与多个函数库。

`CMake` 通过使用简单的配置文件 `CMakeLists.txt`，自动生成不同平台的构建文件（如 `Makefile`、`Ninja` 构建文件、`Visual Studio` 工程文件等），简化了项目的编译和构建过程。

`CMake` 本身不是构建工具，而是生成构建系统的工具，它生成的构建系统可以使用不同的编译器和工具链。

---

这样介绍 `CMake` 可能不太好理解，我们可以先从为什么需要 `CMake` 入手。下面以 `make` 作为构建工具为例

1. `CMake` 是**跨平台的构建系统生成器**，通过统一的 `CMakeLists.txt` 文件，可以在不同平台上生成对应的构建脚本，如 `Makefile`, `Visual Studio`, `Xcode` 项目等；
2. `CMake` 可以更好的管理项目结构和进行项目构建的自动化

例如：以一个大型 C++ 项目为例

```bash
# 项目根目录
my_project/
├── src/                # 主代码
│   ├── core/           # 核心库模块
│   │   ├── math.cpp    # 数学工具
│   │   └── utils.cpp   # 通用工具
│   └── app.cpp         # 主应用程序
├── libs/               # 第三方库
│   └── third_party.so
├── include/            # 头文件
├── tests/              # 单元测试
├── build/              # 构建输出目录（CMake/Makefile通用）
└── docs/               # 文档
```

可能的 `Makefile` 实现方式：

```makefile
# 编译器定义
CXX = g++
CXXFLAGS = -Wall -Iinclude -Ilibs
LDFLAGS = -Llibs -lthird_party

# 源文件路径
SRC_DIR = src
CORE_DIR = $(SRC_DIR)/core
APP_SRC = $(SRC_DIR)/app.cpp
CORE_SRCS = $(CORE_DIR)/math.cpp $(CORE_DIR)/utils.cpp

# 生成的目标文件
OBJS = $(APP_SRC:.cpp=.o) $(CORE_SRCS:.cpp=.o)

# 最终可执行文件名
TARGET = my_app

# 默认目标
all: $(TARGET)

# 链接目标文件生成可执行文件
$(TARGET): $(OBJS)
	$(CXX) -o $@ $^ $(LDFLAGS)

# 编译每个 .cpp 文件为 .o 文件
%.o: %.cpp
	$(CXX) $(CXXFLAGS) -c $< -o $@

# 清理生成的文件
clean:
	rm -f $(OBJS) $(TARGET)
```

其缺点体现在：

1. **手动管理源文件路径**：需显式列出所有 `.cpp` 文件（如 `CORE_SRCS`），增减文件时需修改脚本。
2. **硬编码依赖路径**：第三方库路径（-Llibs -lthird_party）和头文件路径（-Iinclude）直接写死，跨平台需调整。
3. **平台依赖性强**：`rm` 命令在 Windows 下不兼容，需额外处理。

可能的 `CMake` 实现：

```bash
# 设定 cmake 兼容最低版本与工程名
cmake_minimum_required(VERSION 3.10)
project(MyProject)

# 设置 C++ 标准
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 自动收集所有源文件
file(GLOB_RECURSE SRCS
    "src/app.cpp"
    "src/core/*.cpp"
)

# 创建可执行文件
add_executable(${PROJECT_NAME} ${SRCS})

# 全局编译选项
add_compile_options(-Wall -Wextra)

# 添加头文件路径
target_include_directories(${PROJECT_NAME} PRIVATE include)

# 链接第三方库
target_link_directories(${PROJECT_NAME} PRIVATE libs)
target_link_libraries(${PROJECT_NAME} PRIVATE third_party)
```

1. **自动收集源文件**：通过 file(GLOB) 或 aux_source_directory 自动收集文件，无需手动维护列表。
2. **跨平台路径管理**：CMake 自动处理不同操作系统的路径格式（如 / vs \）。
3. **清晰的依赖管理**：
   - target_include_directories 管理头文件路径。
   - target_link_libraries 直接链接库名（无需 -l 前缀），CMake 自动查找 .so 或 .dll。
4. **生成多平台构建文件**：通过命令生成 Makefile、Visual Studio 或 Xcode 项目

使用 `CMake` 的构建步骤：(以 `linux` 环境为例)

- 在项目根目录下(注意 `CMakeLists.txt` 应当也在根目录下)

```bash
mkdir build & cd build
cmake .. & make
```

## 练习

**「必做」**

以下是一些示例代码文件：

**main.cpp**

```cpp
#include <iostream>
#include "bubblesort.hpp"

int main() {
    std::vector<int> arr = {64, 34, 25, 12, 22, 11, 90};
    bubbleSort(arr);
    for (int num : arr) {
        std::cout << num << " ";
    }
    std::cout << std::endl;
    return 0;
}
```

**bubblesort.cpp**

```cpp
#include <iostream>
#include <vector>

void bubbleSort(std::vector<int>& arr) {
    int n = arr.size();
    for (int i = 0; i < n - 1; ++i) {
        for (int j = 0; j < n - i - 1; ++j) {
            if (arr[j] > arr[j + 1]) {
                std::swap(arr[j], arr[j + 1]);
            }
        }
    }
}
```

**bubblesort.hpp**

```cpp
#include <iostream>
#include <vector>

void bubbleSort(std::vector<int>& arr);
```

请你将上述三段代码复制到lab0/目录下，并在该目录下：

参照上述示例 Makefile 格式，编写 Makefile 文件来编译这段代码，输出可执行文件命名为 bubble_sort.

**要求：**

- 使用 `make` 命令时，编译生成可执行文件
- 使用 `make clean` 命令时，删除生成的可执行文件

**「选做」**

1. 请你参考 C++的宏机制以及 g++的编译选项，在 Makefile 中加入一段代码，以及对 cpp 源代码进行适当修改，来打印出冒泡排序中每次交换元素之后的 vector 的值。
2. 用 `cmake` 自动生成 `Makefile`，完成项目的编译，可以不改变项目的目录结构。(`CMakeLists.txt`的语法可参考https://www.runoob.com/cmake/cmake-basic.html)

**要求：**

- 使用 `make debug` 命令时，编译生成的可执行文件在每次交换元素之后会打印出 vector 的值 (debug 信息)，与此同时，使用 `make` 命令生成的可执行文件不会打印 debug 信息

**参考资料**：

1. https://seisman.github.io/how-to-write-makefile/introduction.html
2. https://www.gnu.org/software/make/manual/make.html
3. https://makefiletutorial.com/
4. https://zhuanlan.zhihu.com/p/22604568059
