# GDB 调试

## 1. 基本使用

```c++
#include <iostream>

int main(int argc, char** argv) {
    int itest = 1000;
    const char *str = "this is a test";
    std::cout << "itest is " << itest << ", str is " << str << std::endl;
    std::cout << "参数为：" << std::endl;

    for (int i = 0; i < argc; ++i){
        std::cout << argv[i] << std::endl;
    }
    std::cout << "Hello World\n";

    return 0;

```

编译时加 `-g` 参数
```bash
$ g++ -g -Wall -O0 01-test.cpp -o  01-test.o
```

启动调试

```bash
$ gdb 01-test.o
```

命令：
- `l` -> `list` : 显示源代码，默认10行
- `b` -> `break` : 设置断点 ，`b main` 在main函数设置断点，`b 01-test.cpp:3` 
- `i b` -> `info break` : 查看设置的断点
- `r` -> `run`: 执行， 到断点处停止
- `n` -> `next`: 单步执行
- `p var`-> `print var` : 查看变量名var的值
- `i locals` : 显示全部的局部变量,当前代码之前
- `c` -> `continue`: 继续执行，遇到断点停止




## 02 启动调试


### 传递参数

- `gdb --args <exe> [args]`: 在启动调试时传入 , `gdb --args a.out 1 2 "3 4"` 参数为 1，2, 3 4有空格需要用引号
- `set args <args>` : 在进入gdb后，运行程序前设置参数
- `r <args>`: 执行时后面直接跟参数


### 附加到进程
程序运行过程中，使用gdb调试，主要是使用程序的pid参数
- `gdb attach <pid>`
- `gdb --pid <pid>`

### 逐过程执行
- `next` / `n` : 不会进入函数内部
- `next instruction` -> `ni` : 执行一条汇编指令，用于调试内核汇编代码

### 逐语句
- `step` -> `s` : 单步执行，遇到函数会进入函数
- `step instruction` -> `si` :执行一条汇编指令，会进入调用内部，用于调试内核汇编代码
- `si N`: 一次执行 N 条汇编指令

### 退出函数

- `finish`: 如果在函数内部，退出当前函数，回到函数调用点的下一行代码


### 退出调试
退出调试

- `detach` : 分离，不影响程序继续执行，gdb 无法再继续控制该程序了
- `q` -> `quit`： 退出调试并终止程序


## 3. 断点管理

### 设置断点

- `b file:line` : `b test.cpp:12`, 如果是空行，断点会设置到下一行有代码的位置
- `b function` ： 在函数第一行设置断点， 会在所有同名函数设置断点
- `rb regx`： 使用正则表达式
- `b bpoint condition`: 断点 + 条件， 条件断点
- `tb bpoint`：临时断点，只会执行一次，即命中过一次就失效
- `b *addr` ： 在指定内存地址打断点

### 查看/禁用/删除断点

- `i b` ： 查看所有断点
- `disable/enable 断点编号`: 禁用/启用断点
- `delete 断点 `: 删除断点， 不输入断点号，删除全部断点


## 4. 查看/修改变量


### 查看变量
- `info args` ： 查看函数参数
- `print var` -> `p var` : 查看变量var的值
- `set print null-stop` : 设置字符串的显示规则，不显示无效的0
- `set print pretty`： 显示结构体格式一行一个属性
- `set print array on` ：显示数组，一行一个元素显示
- 使用gdb内嵌函数， 比如 `sizeof`, `strlen`

### 修改变量
- `p var=13`： 改变var的值为13



## 5. 查看/修改内存


 `x /nfu <addr>` n: 内存数量， f:数据显示格式， u:粒度  ， addr 地址 [详细说明](https://sourceware.org/gdb/onlinedocs/gdb/Memory.html)

- `x /选项 内存地址`
  - `x /s str`
  - `x /d`

- `x/Ni addr` : 显示地址addr处开始的N条汇编指令

```bash
(gdb) help x
Examine memory: x/FMT ADDRESS.
ADDRESS is an expression for the memory address to examine.
FMT is a repeat count followed by a format letter and a size letter.
Format letters are o(octal), x(hex), d(decimal), u(unsigned decimal),
  t(binary), f(float), a(address), i(instruction), c(char), s(string)
  and z(hex, zero padded on the left).
Size letters are b(byte), h(halfword), w(word), g(giant, 8 bytes).
The specified number of objects of the specified size are printed
according to the format.  If a negative number is specified, memory is
examined backward from the address.
```


## 6. 查看/修改寄存器

### 查看
- `i reg` ，查看寄存的值

`i r rax` ： 查看 rax寄存器值

`i r`： 查看通用寄存器的值

### 修改
`set var reg=x` ： `set var $pc=xx` 修改pc指针寄存器， `p $ip=xx` 等价

`info line num`:  `info line 12` 查看12行代码的地址值

## 7. 源代码查看/管理

- `list` -> `l` 显示源码，默认10行
- `set listsize xx` : 设置显示的行数
- `list function` : 查看指定函数的代码
- `list main.cpp:15` ： 查看指定文件,指定行


- `layout asm` 显示汇编
- `layout src` 显示源码

## 8. 调用栈

- `backtrace/bt` ， 查看栈回溯信息
- `frame n` ：切换栈帧
- `info f n` : 查看栈帧信息

