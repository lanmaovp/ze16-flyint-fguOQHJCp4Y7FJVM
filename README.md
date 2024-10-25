
# 标准库 IO


输入输出功能并非C语言的组成部分，ANSI标准定义了相关的库函数


# 输入输出


流stream是与设备关联的数据的源或者目的地。


* 文本流：由文本行组成的序列
不同系统的特性可能不一样，比如行最大长度和行结束符
* 二进制流：未经处理的字节序列


程序运行时，默认打开 stdin, stdout, stderr


标准 IO 常量


* EOF：文件尾，实际值是几个字符避免混淆
* FOPEN\_MAX：一个程序同时最多能够打开的文件数量，与编译器有关，值至少为 8
* FILENAM\_MAX：编译器支持的最长文件名


## 文件流操作


文件指针指向包含文件信息的结构，包括缓冲区，读写状态等


### 打开与关闭



```


|  | FILE *fopen(const char *filename, const char *mode);                    // 打开文件流 |
| --- | --- |
|  | /* |
|  | mode: |
|  | * r, 打开读 |
|  | * w, 打开或创建写，删除原有内容 |
|  | * a, 打开或者创建，文件尾追加 |
|  | * +, 更新，读或写，交叉操作前需要执行fflush或者定位操作 |
|  | * r+, 打开文件更新（读和写） |
|  | * w+, 创建文件更新，删除原有内容 |
|  | * a+, 打开或创建文件，文件尾更新 |
|  | * b, 二进制流模式 |
|  |  |
|  | 应始终检查返回值 |
|  | */ |
|  |  |
|  | FILE *freopen(const char *filename, const char *mode, FILE *stream);    // 先关闭再打开文件流 |
|  |  |
|  | FILE *fdopen(int fd, const char *type);   // POSIX标准，常用于由创建管道和网络通信通道函数返回的描述符，不能直接用 fopen 打开 |
|  | // type 参数: r, w, a, (+) |
|  | // 注意读写模式下读写操作之间需要进行调用定位函数 |
|  |  |
|  |  |
|  | int fclose(FILE *stream);                                               // 刷新输出缓冲，释放系统缓冲区，关闭流 |
|  |  |
|  |  |
|  | int fileno(FILE *fp);   // POSIX 标准 |


```

### 流的定向


标准IO可用于单字节和多字节字符集，具体由流的定向决定。


* freopen函数清除一个流的定向
* fwide函数设置流的定向、



```


|  | #include |
| --- | --- |
|  | #include |
|  |  |
|  | int fiwde(FILE *fp, int mode); |
|  | /* |
|  | mode<0, 试图设置为单字节 |
|  | mode>0，试图设置为多字节 |
|  | mode=0，不设置定向，返回流的定向 |
|  |  |
|  | 不会改变已经设置的流定向 |
|  | */ |


```

### 文件操纵函数


#### 重命名



```


|  | int remove(FILE *stream);                               // 删除文件 |
| --- | --- |
|  |  |
|  | int rename(const char *oldname, const char *newname); // 重命名文件 |


```

#### 临时文件



```


|  | FILE *tmpfile(void);     // 以`wb+`模式创建临时文件，在关闭或者程序结束时自动删除 |
| --- | --- |
|  | // 实现上tmpnam，创建文件，unlink(不会删除内容)，文件的关闭会在程序结束中自动进行 |
|  |  |
|  | char *tmpnam(char s[L_tmpnam]); // 创建不同现有文件名的字符串，NULL返回指向静态数组的指针 |
|  |  |
|  | // UNIX 优势是不存在时间间隙，避免其他进程创建同名文件 |
|  | char *mkdtemp(char *temple);  // 创建目录 |
|  | int mkstmp(char *temple);     // 创建文件，不会自动删除 |


```

## 缓冲操作


标准IO提供缓冲的目的是尽可能减少read/write的调用次数


* 全缓冲：缓冲区满了才进行实际IO操作
* 行缓冲：遇到换行符才进行IO操作


	+ 行缓冲的长度是固定的，行缓冲满了即使没有换行符也会进行IO操作
	+ 标准IO要求从不带缓冲或者行缓冲（需要从内核请求数据）获取数据，会立即刷新所有行缓冲的输出流
* 不带缓冲


ISO C标准


* 当且仅当标准输入和标准输出不指向交互式设备才是全缓冲的
* 标准错误不会是全缓冲的


**默认情况**


* 标准错误不带缓冲
* 若是指向终端设备的流，则是行缓冲的，否则是全缓冲的


更换缓冲的类型：流被打开之后，且在执行任何操作之前



```


|  | int fflush(FILE *stream);                                               // 刷新缓冲区 |
| --- | --- |
|  | /* |
|  | - 对于输出流，刷新写入缓冲区的内容至目标文件 |
|  | - 对于输入流，其结果是未定义的 |
|  | - NULL，刷新所有的缓冲区 |
|  |  |
|  | 实际使用技巧：每个调试 printf 之后立马调用 fflush |
|  | */ |
|  |  |
|  | int setvbuf(FILE *stream, char *buf, int mode, size_t size);            // 必须在执行读写操作之前设置缓冲 |
|  | /* |
|  | mode: |
|  | - _IOFBF 完全缓冲 |
|  | - _IOLBF 行缓冲 |
|  | - _IONBF 不设置缓冲 |
|  | */ |
|  |  |
|  | void setbuf(FILE *stream, char *buf);     // char buf[BUFSIZ] |
|  | /* |
|  | - buff为 NULL, 关闭缓冲 |
|  | - 否则，等价于 `_IOFBF` |
|  |  |
|  | // 注意：buf 不要使用自动变量类型，尽量使用系统缓冲区或者动态分配内存 |
|  | */ |


```

​\#TODO\#​查看流缓冲状态 《UNIX高级环境编程》


## 读写流操作


### 字符 IO


​\#TODO\#​输入输出函数家族 《C和指针》P301



```


|  | int fgetc(FILE *stream);  // unsigned char 转 int, 兼容EOF |
| --- | --- |
|  | int getc(FILE *stream);   // 等价于 fgetc，注意实现为宏 |
|  | int getchar(void);        // 等价于 getc(stdin) |
|  |  |
|  | int fputc(int c, FILE *stream); |
|  | int putc(int c, FILE *stream);  // 等价于 fputc, 注意实现为宏 |
|  | int putchar(int c);             // 等价于 fputc(stdout) |
|  |  |
|  | // 只有fgetc和fputc是函数，其他都是宏 |
|  |  |
|  | int ungetc(int c, FILE *stream);  // 将字符退回流中，依赖于当前位置 |
|  | // 不同于写操作，仅涉及流本身而无关设备存储 |


```

### 未格式化的行IO



```


|  | char *fgets(char *s, int n, FILE *stream);  // 自动包含换行符，\n 换为 \0，最多n-1 |
| --- | --- |
|  | char *gets(char *s);                        // 不自动包含换行符。没有缓冲区长度参数，可能导致越界；不推荐使用，已废弃 |
|  |  |
|  |  |
|  | int fputs(const char *s, FILE *stream);     // 不自动包含换行符\n，逐字符输入任意个数换行符 |
|  | int puts(const char *s);                    // 自动添加换行符\n |
|  |  |
|  | ssize_t getline(char **lineptr, size_t *n, FILE *stream); |
|  | // 根据输入动态分配内存，无需预先确定输入字符串的最大长度 |
|  | // 在存储的字符串中将换行符替换为字符串结束符'\0' |


```

### 格式化 IO



```


|  | int fprintf(FILE *stream, const char *format, ...); |
| --- | --- |
|  | int printf(const char *format, ...);                  // 等价于 `fprintf(stdout, fotmat, ...)` |
|  | int sprintf(char *s, const char *format, ...);        // 包含结束符 NUL，无长度参数，可能越界溢出 |
|  | int snprintf(char *buf, szie_t n, const char *format, ...); // 超出部分截断，出错返回负值 |
|  |  |
|  | int fdprintf(int fd, const char *format, ...); |
|  |  |
|  | // 变长参数列表变体 |
|  | int vprintf(const char *format, va_list arg); |
|  | int vfprintf(FILE *stream, const char *format, va_list arg); |
|  | int vsprintf(char *s, const char *format, va_list arg); |


```

转换格式：


* 普通字符：复制到输出流
* 转换说明：控制参数的格式转换


	+ 开头 %
	+ 标志\[可选]
	
	
		- ​`-`​左对齐，缺省为右对齐
		- ​`+`​显示正负号
		- 空格 有符号值转换
		- `0`​ 宽度不足时填充 0
		- ​`#`​指定另一种输出形式
	+ 宽度数值\[可选]：指定最小字段宽度
	+ 精度数值\[可选]：点号开始，后接十进制数值
	+ 长度修饰符\[可选]：指定参数的长度
	
	
		- h 按照short/unsigned short 输出
		- l 按照long/unsigned long 输出
		- L 按照long double 输出
	+ 结尾: 转换字符, d, c, s, f, x等



```


|  | int fscanf(FILE *stream, const char *format, ...); |
| --- | --- |
|  | int scanf(const char *format, ...); |
|  | int sscanf(const char *s, char *format, ...); |
|  |  |
|  | int fdscanf(int fd, const char *format,...); |
|  |  |
|  | // 变长参数列表变体 ... |


```

* 参数必须是指针
* 到文件尾或者出错返回EOF，否则返回实际输入的字符数


转换格式


* 空格或者制表符
* 普通字符：匹配下一个输入
* 转换说明


	+ 开始标志`%`​\[可选]
	+ 赋值屏蔽符号`*`​\[可选]
	+ 最大字段宽度数值\[可选]
	+ 限定符 `h\l\L`​，指定参数的长度\[可选]
	+ 结束标志：转换字符 d,f,c, s, x等等


​\#TODO\#​4种使用场景P309


### 二进制 IO


直接IO/二进制IO，通常一次处理一个结构，能够处理null字节和换行符。


注意事项：只能用于同一系统，不同系统的偏移对齐以及存储格式可能不同。因此网络通信需要指定规范。



```


|  | size_t fread(void *buffer, size_t size, size_t nobj, FILE *stream); |
| --- | --- |
|  |  |
|  | size_t fwrite(const void *buffer, size_t size, size_t nobj, FILE *stream); |
|  |  |
|  | /* 返回值是实际读写的元素的个数而非字节数 |
|  | fread：少于nobj, 出错或者EOF，需要进一步分辨 |
|  | fwrite：少于nobj，错误 |
|  | */ |


```

### 内存流


无关文件，直接在缓冲区和主存之间进行字节IO。非常适用于字符串。


Linux支持。



```


|  | FILE *fmemopen(void *buf, size_t size, const char *type); // buf=null, 读写无意义；对于null字节的处理十分特殊 |
| --- | --- |
|  |  |
|  | FILE *open_memstream(char **bufp, size_t sizep);   // 面向字节的流 |
|  |  |
|  | FILE *open_wmemstream(wchar_t **bufp, size_t *sizep);  // 面向宽字节的流 |


```

‍


## 文件定位函数


* 二进制文件：使用字节偏移量，不一定支持 SEEK\_END
* 文本文件：格式不同不能使用字节，orgin\=SEEK\_SET, offset\=0/ftell



```


|  | int fseek(FILE *stream, long offset, int origin); |
| --- | --- |
|  | /* |
|  | - 二进制文件 |
|  | - origin |
|  | - `SEEK_SET` 文件开始处 |
|  | - `SEEK_CUR` 当前位置 |
|  | - `SEEK_END` 文件结束处，可能不支持 |
|  | - 文本文件 |
|  | - `SEEK_SET`；offset是 0 或者 `ftell`返回值 |
|  | - `SEEK_CUR`/`SEEK_END`：offset只能是 0 |
|  |  |
|  | 注意事项 |
|  | - 行末指示符将被清除 |
|  | - 退回的字符将被丢弃 |
|  | - 更新模式中的读写操作切换 |
|  | */ |
|  |  |
|  | long ftell(FILE *stream); |
|  |  |
|  | void rewind(FILE *stream);    // 重置为起始位置 |
|  | //等价于 `fseek(stream, 0L, SEEK_SET); clearerr(stream);` |


```


```


|  | // off_t, 大于32位 UNIX 标准 |
| --- | --- |
|  | off_t ftello(FILE *fp); |
|  | int seeko(FILE *fp, off_t offset, int origin); |


```


```


|  | // fpos_t ISO C标准，更加通用 |
| --- | --- |
|  | int fgetpos(FILE *stream, fpos_t *ptr); |
|  |  |
|  | int fsetpos(FILE *stream, const fpos_t *ptr); |


```

‍


## 错误处理函数


发生错误或者到达文件尾时会设置状态指示符


整型表达式 errno 包含错误编号，定义在 


* 只有库函数失败时才会设置 errno, 成功执行并不会修改 errno
* 任何函数都不会将常量置为0



```


|  | /* ----- 流错误 -----*/ |
| --- | --- |
|  | int feof(FILE *stream);     // 流设置了文件结束指示符，返回非0值 |
|  |  |
|  | int ferror(FILE *stream);   // 流设置了错误指示符，返回非0值 |
|  |  |
|  | int clearerr(FILE *stream); // 清除流的所有指示符 |


```


```


|  | #include |
| --- | --- |
|  | char *strerror(int crrno);  // 映射错误信息 |
|  |  |
|  | #include |
|  | int perror(const char *s);  // 打印字符串和 errno 错误信息 |
|  | // 类似于 fprintf(stderr, "%s: %s\n", s, "error essage"); |


```

# 标准IO的替代


标准IO 的效率不高，调用行IO需要进行两次数据的复制


* 快速IO fio: 使用指针而不是复制整行
* sfio: 提高速度，同时推广IO流
* mmap


适用于嵌入式系统的更低内存要求的实现


* uClibc C库
* Newlib C库


‍


# 参考


* 《C程序设计语言》
* 《UNIX 环境高级编程》


 本博客参考[slower加速器](https://jisuanqi.org)。转载请注明出处！
