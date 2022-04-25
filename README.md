# CS3001H.TESTS

本项目是根据RISC-V官方指令集测试项目 [riscv-tests](https://github.com/riscv-software-src/riscv-tests) 裁剪而成，与教学内容匹配。具有以下特性

- 只支持rv32i标准
- 不含有任何特权架构的内容
- ~~支持Verilog自动化测试~~（TODO）



## 功能测试

功能测试是针对CPU**指令集功能的兼容性**进行测试（并不保证微架构没有任何错误），项目结构如下

```
isa
│ -- link.ld        // 链接文件
│ -- macros.h       // 宏文件
│ -- Makefile       // 编译脚本
└ -- rv32mi         // rv32i测试文件夹
     | -- add.S     // ADD指令测试文件
     | -- addi.S    // ADDI指令测试文件
     ...
```

### 测试原理

根据指令的功能可以将指令划分为：算数逻辑指令、访存指令、分支指令。根据指令寻址方式可以将指令划分为：立即数寻址指令、寄存器寻址指令、内存寻址指令。根据可能的微架构可以提出几类问题：数据相关，控制相关，结构相关，等等。测试程序会对这几类功能进行测试。

CPU执行测试程序中的指令，将执行的结果存储到指定寄存器中，与存储在另一寄存器中的先验结果进行比较，若错误，则跳转到相应的错误处理程序地址。

代码段起始位置为`0x00000000`，没有数据段。

### 前期准备

请确保CPU已基本正确实现`addi, lui, bne, jal`这几条基本指令，这是测试的基础。此外，不同的指令测试程序所用到的指令不尽相同，建议按照一定的顺序进行测试，比如`jal -> bne -> lui -> addi -> ...`。

### 如何使用

- 单独测试

  使用.coe文件初始化IP存储器，执行。注意内存读写为**小端**模式。

- 批量测试

  见[批量测试](#批量测试)

- 自定义测试

  1. 修改汇编程序`.S`文件内的测试条目
  2. 根据需要修改编译脚本`Makefile`和链接脚本`link.ld`（一般情况下不用修改）
  3. 在交叉编译环境下（Vlab已配置）执行`make`
  4. 使用`bin2coe`工具将生成的.bin文件转换为.coe文件

### 如何判断测试是否通过

每执行一段测试，都会检查寄存器的值是否正确，若不正确则直接跳转到错误处理代码段。在测试文件的最后还会执行`TEST_PASSFAIL`代码段来判断整个测试成功与否，寄存器`TESTRES`为1反映测试完成，寄存器`TESTNUM`非0反映测试成功。

反映测试正确的方法应该是**多样化的**，这部分可以根据情况自行修改，如观察仿真波形、发送LED显示、发送数码管显示、发送串口等。



## 性能测试

性能测试是针对CPU**执行程序的性能**进行测试，项目结构如下

```
benchmark
│ -- datasheet.h    // 测试数据集
│ -- link.ld        // 链接脚本
│ -- Makefile       // 编译脚本
└ -- sort.c         // 测试程序
```

### 测试原理

项目使用冒泡排序作为测试程序。文件`datasheet.h`中存储了2048项待排序数据和待验证数据，作为测试数据。文件`sort.c`为冒泡排序主程序，先执行`sort`函数，再执行`verify`函数，如果排序出错，返回出错的序号，如果排序正确，返回0。

数据段起始地址为`0x00000000`，代码段起始地址为`0x00004000`。

### 前期准备

请确保CPU已正确实现`addi, slli, lw, sw, jal, jalr, bge, blt, bltu `这几条指令，并通过功能测试。

### 如何使用

同[功能测试](#功能测试)。

### 如何计算得分

请选择合适的时钟频率，并计算程序执行的时钟周期数。

<img src="https://latex.codecogs.com/svg.image?score=\frac{p}{p_{baseline}}=\frac{t_{baseline}}{t}=\frac{c_{baseline}/f_{baseline}}{c/f}">



## 批量测试

TODO

