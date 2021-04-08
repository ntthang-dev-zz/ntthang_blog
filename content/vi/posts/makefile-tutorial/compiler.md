---
title: "Compiler là gì ?"
description: ""
date: 2018-02-18T22:34:04+01:00
publishDate: 2018-02-18T22:21:42+01:00
author: "Jackie Nguyen"
images: []
draft: false
tags: ["makefile", "compiler"]
---

[Compiler là gì ? Cross Compiler là gì ? Nó có liên quan gì tới một hệ thống nhúng ?](#) Mình cùng tìm hiểu một số khái niệm cơ bản trong bài này nhé.

## Compiler

Compiler hay còn gọi là [trình biên dịch](https://vi.wikipedia.org/wiki/Tr%C3%ACnh_bi%C3%AAn_d%E1%BB%8Bch) có thể được hiểu là công việc dịch chuỗi câu lệnh được viết từ một ngôn ngữ lập trình thành chương trình tương đương dưới dạng ngôn ngữ máy tính, thường là ngôn ngữ ở cấp thấp hơn, ngôn ngữ máy. Đơn giản dễ hiểu thì có thể tạm nói là nhờ Complier này mà `file .c` chúng ta viết mới được dịch thành file `.hex` `.bin` để nạp được xuống một MCU bất kỳ.

### Quá trình biên dịch

![Compiler GCC.](https://hocarm.org/content/images/2020/04/compiler_gcc.gif)

Chúng ta có thể xem sơ đồ chi tiết các bước từ Code/Build/Run ở hình sau

![excuting a program](https://hocarm.org/content/images/2020/04/excuting_a_program.png)

Thông thường nếu dùng chương trình để lập trình như Keil C chẳng hạn thì chỉ cần ấn một nút Build/Run xong là chúng ta chỉ việc ngồi chờ và chương trình được nạp trực tiếp vào chip luôn, nhưng ẩn đằng sau những nút này là một loạt hoạt động theo các bước như hình trên.

## Cross Compiler/ Toolchain là gì ?

[Cross Compiler](https://en.wikipedia.org/wiki/Cross_compiler) hay còn gọi là Toolchain có thể được hiểu là một source code được viết trên máy tính chạy trên chip Intel, sau khi thông qua một cross compiler sẽ cho ra file nhị phân có khả năng chạy được trên một nền tảng chip khác là ARM. Một ví dụ cơ bản nhất là mình đã dùng một máy tính hệ điều hành Ubuntu để build ra một file image có thể chạy trên Raspberry Pi

![Cross compiler](https://hocarm.org/content/images/2020/04/Example_of_Cross_compiler.png)

Quá trình tạo ra và sử dụng `cross compiler`/ `tool chain` có liên quan tới 3 đối tượng :

- **Build** : hệ thống tạo ra `tool chain`, thường là các máy tính các máy tính dùng chip Intel và hệ điều hành Linux hoặc Windows.
- **Host** : hệ thống chạy `tool chain` để compile source code, host cũng giống build thường là các máy tính dùng chip Intel và Windows hoặc Linux là hệ điều hành.
- **Target** : là hệ thống chạy chương trình do host tạo ra, thường target là các máy tính nhúng dùng chip ARM, tuy nhiên nó cũng có thể là một máy tính bình thường dùng chip Intel.

![GCC Cross compiler](https://hocarm.org/content/images/2020/04/gcc-cross-compiler.png)

Vậy các thành phần của Cross Compiler là gì ?

![Component toolchain](https://hocarm.org/content/images/2020/04/component_toolchain-e1489290193482.jpg)

- **Binutils** : Là một tập các công cụ để tạo và quản lý file nhị phân `(bin)` của target CPU
- `as` : là `assembler`, nó sinh ra mã nhị phân `(binary code)` từ `assembler` source code
- `ld` : trình liên kết `(linker)`
- `ar, ranlib` : sinh ra file nén `.a`, sử dụng như là thư viện
- `objdump, readelf, size, nm, strings`: phân tích file nhị phân
- `strip` : để loại bỏ những phần thừa trong file nhị phân để giảm kích thước của chúng

Thông thường để cross-compiler một chương trình ta phải cài đặt biến môi trường mới có thể compile đúng được
Ví dụ

```bash
$ export PATH=/path/to/compiler/bin:$PATH
$ export CROSS_COMPILE=arm-none-linux-gnueabi- 
$ export CC=${CROSS_COMPILE}gcc 
$ export CXX=${CROSS_COMPILE}g++ 
$ export CPP=${CROSS_COMPILE}cpp 
$ export AR=${CROSS_COMPILE}ar 
$ export AS=${CROSS_COMPILE}as 
$ export LD=${CROSS_COMPILE}ld 
$ export RANLIB=${CROSS_COMPILE}ranlib 
$ export STRIP=${CROSS_COMPILE}strip 
```

### C/C++ Library

Library được dùng làm interface giữa applications và kernel, cung cấp các C API chuẩn để dễ dàng phát triển ứng dụng. Một số `libb` có thể kể đến như: `glibc, uClibc, eglibc, dietlibc, newlib, …`

![c-lib](https://hocarm.org/content/images/2020/04/c-lib.png)

### Kernel header

Cung cấp các API cần thiết cho Applications và C Library giao tiếp với Kernel.

![kernel-header](https://hocarm.org/content/images/2020/04/kernel-header.png)

### GCC compiler

`gcc`, `c++`, `g++` : compiler
Trình biên dịch trong hệ thống Linux, compile cho rất nhiều ngôn ngữ và nhiều kiến trúc CPU khác nhau như ARM, MIPS, PowerPC, SuperH, x86; tuy nhiên mình chỉ đề cập đến ngôn ngữ C/C++ và kiến trúc CPU là ARM và x86.

### GDB Debugger

Trình gỡ rối, trợ giúp cho quá trình phát hiện lỗi khi develop application.

## Ví dụ với GCC Compiler

### Cài đặt GCC

Trước hết, mình thực hiện các bước với GCC trên máy tính dùng **Ubuntu** nhé

Thực hiện check version hiện có của `gcc/g++` và cài đặt

```sh
$ gcc --version
$ g++ --version
$ sudo apt-get install gcc g++
```

### Ví dụ

Xét một ví dụ cơ bản với chương trình C tính căn bậc 2 của 4 như sau

```c
#include <stdio.h>  
#include <math.h>  
   
int main(int argc, char **argv) {      
      double x;     
      x = sqrt(4);  
      printf("x = %f \n", x);  
      return 1;  
} 
```
Để thực hiện thì chúng ta lưu code trên dưới dạng file là `main.c`, sau đó thực hiện gõ lệnh command sau trên ubuntu

```sh
$ export CFLAGS="-I./include -DDEBUG -Wall -g" 
$ export LDFLAGS+=" -L./lib -lm"  
$ gcc -c main.c ${CFLAGS}         #tạo file object từ source 
$ gcc -o prog main.o  ${LDFLAGS}  #tạo file chương trình nhị phân từ file object 
$ ./prog   #chạy chương trình
```

### Giải thích

Trên đây là một format cơ bản nhất của GCC

**`CFLAGS`** 

C compiler flags đưa các options vào trong compiler để thực hiện quá trình compile source code thành object sẽ bao gồm các thông tin :

- Đường dẫn các header bắt đầu với `-I`, ví dụ `-I./include` .
- Các define được bắt đầu với `-D`, ví dụ `DDEBUG` để define DEBUG .
- Các option đặc biệt khác của compiler như `-g` để bật chức năng `debug gdb` của `gcc compiler`, `-wall` để trace các cảnh báo (warning) trong quá trình C.

```sh
$ export CFLAGS="-I./include --DDEBUG -Wall -g"`
```

**`LDFLAGS`**

Linker flags dùng trong quá trình linking các thư viện, nó bao gồm các thông tin:

- Đường dẫn tới thư viện, được bắt đầu bằng `-L`, ví dụ `-L./lib` .
- Các thư viện bắt đầu với `-l` là viết tắt của `lib`, ví dụ `-lm` tương ứng với `libm`, thư viện math có sẵn trong hệ thống.

```sh
$ export LDFLAGS+=" -L./lib -lm"
```

**`gcc`** Compiler cho C source và **`g++`** cho C++ source.

## Tạm kết

Thế là xong được những bước cơ bản đầu tiên với Crosscompiler, tìm hiểu được một chút về cách để compile source .c đơn giản. Mới bước đầu làm quen thế là đủ, hẹn mọi người ở bài tiếp theo.