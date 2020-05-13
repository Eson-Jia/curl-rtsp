
# gdb 调试

## 编译时携带调试信息

使用`gdb main`调试 gcc main.c -o main `pkg-config --cflags --libs libcurl` 命令编译成可执行文件`main`,如果执行`b 行号`会报错`No symbol table is loaded. Use the "file" command`，这是因为在编译的时候没有使用`-g`指定将源代码信息编译到可执行文件，在最后追加`-g`即可。

```doc
-g  Produce debugging information in the operating system's native format (stabs, COFF, XCOFF, or DWARF).  GDB can work with this debugging information
```

## 基本命令

| 命令                    | 描述                                                                          | 前置条件                                   |
| ----------------------- | ----------------------------------------------------------------------------- | ------------------------------------------ |
| gdb excutable           | 启动并载入可执行文件                                                          |                                            |
| file excutalbe          | 载入可执行文件                                                                | 运行`gdb`之后                              |
| set args [params]       | 设置                                                                          |
| b <linenum>             | 在指定行号添加断点                                                            | 可执行文件在编译时需要添加`-g`添加调试信息 |
| b <filename>:<linenum>  | 在指定文件中的指定行号添加断点                                                | 同上                                       |
| b <function>            | 在指定函数上添加断点                                                          | 同上                                       |
| b <filename>:<function> | 在指定文件的指定函数上添加断点                                                | 同上                                       |
| print,p <variable>      | 打印指定变量的值                                                              |                                            |
| run,r                   | 开始运行可执行文件                                                            |                                            |
| continue,c              | 继续往下执行                                                                  |                                            |
| n                       | 执行一行源程序代码，此行中的函数调用也一并执行，相当于其他调试中的`step over` |                                            |
| s                       | 执行一行源程序代码，如果有函数调用则进入，相当于调试器中的`step into`         |
| quit,q                  | 退出                                                                          |

# C

## sscanf

```c
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *const argv[])
{
    char *control = malloc(1024);
    sscanf(argv[1], " a = control: %s", control); //注意这里输入模板有多个空格
    printf("%s", control);
    return 0;
}
//a.out 'a=control:rtsp://192.168.5.180/trackID=2'
//output: rtsp://192.168.5.180/trackID=2
```

虽然输入模板`" a = control: %s"`中包含多个空格，输入参数中不包含空格但是仍能解析成功
