# curl-rtsp

使用`curl`发送`rtsp`请求

## gdb 调试

使用`gdb main`调试 gcc main.c -o main `pkg-config --cflags --libs libcurl` 命令编译成可执行文件`main`,如果执行`b 行号`会报错`No symbol table is loaded. Use the "file" command`，这是因为在编译的时候没有使用`-g`指定将源代码信息编译到可执行文件，在最后追加`-g`即可。
