

## makefile

$@  目标文件的完整名称

$+  所有的依赖文，以空格隔开这些依赖文件，这个依赖文件可能会有重复的 

$<  第一个依赖文件的名称 

$?  所有时间戳比目标文件晚的依赖文件，比目标文件早的依赖文件就不需要了 以空格隔开 

$^  所有不重复的依赖文件，以空格隔开

```c
$(wildcard *.c)  -> 找到所有的.c文件，引用它的值
    返回值：1.c 2.c
        a := $(wildcard *.c)
        a只保存了 1.c 2.c这个两个字符串

字符串的模式替换 -> patsubst
    patsubst <from>,<to>,<text>
    $(patsubst %.c,%.o,"1.c 2.c") -> 经过这个函数之后  就会将所有的.c模式替换为.o
    $(patsubst %.c,%.o,$(wildcard *.c))
```

### 模版

```c
#最终的目标名
TARGET := ../BIN/main


#指定一个编译器
CC := gcc


#依赖文件
CFILES := $(wildcard ../CODE/SRC/*.c)
#当前目录下的C文件，其实就是main函数
CFILES += $(wildcard ./*.c)		

#去掉路径
CFILES := $(notdir $(CFILES))

#目标文件
OBJS := $(patsubst %.c,../OBJ/%.o,$(CFILES))


#头文件搜索路径  
INCPATH := -I../CODE/INC
INCPATH += -I../LIB/inc


#库的搜索路径
LIBPATH := -L../LIB/lib
#LIBPATH += -L../LIB/lib


#库的名字
LIBFILES := 
#LIBFILES += 


#tftp 路径
TFTPPATH := ~/tftpboot


#最终的可执行文件依赖.o文件
#将所有的.o文件进行链接
$(TARGET) : $(OBJS)	
	$(CC)  $^ -o $@ $(INCPATH) $(LIBPATH) $(LIBFILES)
	
	cp $(TARGET) $(TFTPPATH) 


#编译C文件
../OBJ/%.o : ../CODE/SRC/%.c
	$(CC) -c $< -o $@ $(INCPATH) $(LIBPATH) $(LIBFILES)


../OBJ/%.o : ./%.c
	$(CC) -c $< -o $@ $(INCPATH) $(LIBPATH) $(LIBFILES)

print:
	@echo $(CFILES)
	@echo -------
	@echo $(OBJS)


#加标记 你make clean就来这里执行
clean:
	rm -rf $(OBJS) $(TARGET)

```

## 交叉开发



### 文件接收

```c
tftp -g -r (Login.bmp) 172.90.1.31

cp xxx ~/tftpboot/
```

### gcc编译的过程

```c
1 预处理(arm-linux-)gcc    -E     1.c     -o    1.i

//将c代码变成汇编
2 编译 (arm-linux-)gcc    -S   1.c或1.i   -o    1.s

//将指令代号(汇编语言)转换为计算机语言(二进制指令)
3 汇编 (arm-linux-)gcc    -c   1.c或1.s   -o    1.o

4 链接 (arm-linux-)gcc         1.c或1.o   -o    main

```

### gdb调试 

```c
常用调试:printf("%s   %d\n",__FUNCTION__,__LINE__);
gcc -g main.c -o main
gdb main -> 这个时候就会进入调试

进入调试之后我们可以通过命令来进行代码的调试
    设置断点，可以加多个断点
        b 行号 -> 程序运行到这一行就会停下来
        b 符号(函数) -> 程序运行到这个函数就会停下来

    info b      查看程序断点的情况

    r   运行代码

    n   单步运行 n会将函数的调用看成一步

    s   单步运行 s会进入这个函数进行单步执行
 
    l   列举源文件

    p   打印程序里面变量的值
        p a   ->打印这个时候a的值  从而分析我们的变量的值是否符合预期

    help  帮助

    q   退出

    c   继续运行
```

### 动态库

```c
gcc编译找不到头文件时，使用	-I(文件路径)
1、编译生成动态库
arm-linux-gcc  -shared  -fpic  xxx.c  -o  libxxx.so
说明:
	-shared   表示要编译生成一个共享库(动态库)
	-fpic     表示生成与位置无关的代码
	
2、生成可执行文件				
gcc xxx.c -o xxx -I./include -L./lib -lxxx

-I: 编译时指定头文件搜索路径
-L: 编译时指定库的搜索路径
-l: 后面接库的名字(libmyxqh.so, 则库名为: myxqh)

将xxx文件放在tftpboot目录下
cp xxx ~/tftpboot/

开发板接收文件
cd /usr/lib
tftp -g -r libxxx.so 172.90.1.31

tftp -g -r (xxx) 172.90.1.31 <- ubuntu地址ip





```

### 静态库

```c
把各源文件编译成.o文件(目标文件)
arm-linux-gcc -c xxx.c -o xxx.o  -I./include
用目标文件生成一个静态库
arm-linux-ar  -rc libxxx.a  xxx.o xxx.o
静态库的名称:libxxx.a

生成可执行文件		
arm-linux-gcc xxx.c -o xxx -I./include -L./lib -lxxx

-I: 编译时指定头文件搜索路径
-L: 编译时指定库的搜索路径
-l: 后面接库的名字(libmyxqh.a, 则库名为: myxqh)		
```

## 系统文件IO

#### open

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
       
            专门打开一个文件
   int open(const char *pathname, int flags);

        打开或者创建一个文件
   int open(const char *pathname, int flags, mode_t mode)

        创建一个文件
   int creat(const char *pathname, mode_t mode);
        pathname：路径名
            带路径的文件名(相对路径绝对路径都是可以的)
            路径以及名字不能弄错了
            以字符串去描述这个路径名
            
			                  flags
            下面的三个标志必须任选其一
                O_RDONLY    只读
                O_WRONLY    只写
                O_RDWR      读写
            下面的一些常用可以选择，如果你有需求下面的标志，直接将其 | 起来
            
                O_CREAT     创建标记，如果这个这个文件存在，这个标记是被忽略的
                            只有这个文件不存在，这个标记才会有作用
                            当我们给了这个标记之后mode这个参数必须要给
                            否则这个权限就会有问题
              
                O_EXCL  	测试文件是否存在，这个标记一般都是搭配O_CREAT一起使用                        		     这个文件存在则我不动他
                         	如果这个文件不存在我就创建并且将这个文件打开
                            
                            给了这个标记必给O_CREAT标记
                            创建这个文件的时候open函数会看这个文件是不是已经存在
                            不存在则创建并打开，如果文件本身就存在了，则open失败
                            并且errno == EEXIST
                            
                O_NONBLOCK  非阻塞标记，当我们不给这个标记的时候，open默认是阻塞的
                
                O_APPEND    追加标记，如果没有这个表示，open的时候光标默认在文件的开头
                    		光标(偏移量)直接定位到文件的末尾

                O_TRUNC     截短标记
                    		如果你的文件是一个普通文件，并且有可写的权限，那么打开这个文							  件之后，如果带上这个标记 内容会被清空
                    		
                            mode:权限
                S_IRWXU     用户可读可写可执行
                S_IRUSR     用户可读
                S_IWUSR     用户可写
                S_IXUSR     用户可执行
                S_IRWXG     组用户可读可写可执行
                .....
                S_IRWXO     其它用户可读可写可执行
                一般我们都用8进制表示权限  
                0664 -> S_IRUSR | S_IWUSR | S_IRGRP | S_IWGRP | S_IROTH
                 		
```

#### read

```c
#include <unistd.h>
阻塞函数
   ssize_t read(int fd, void *buf, size_t count);
        fd：文件描述符  
        buf：内存，你读到的内容放在哪个内存
            这个内存在用的时候要有足够的空间
        count：你要读多少个字节   单位是字节
        
    	返回值： 成功返回实际读到的字节数 <= count
                失败返回-1，同时errno被设置
```

#### write

```c
#include <unistd.h>
   ssize_t write(int fd, const void *buf, size_t count);
        fd：文件描述符  
        buf：你要写入的内容，就是一块内存，里面存入了什么内容
        count：你要写入多少个字节   单位是字节
    返回值：
        成功返回实际写入的字节数 <= count
        失败返回-1，同时errno被设置

每读写一个字节光标就会往后面走一个字节
```

####  lseek 

```c
#include <sys/types.h>
#include <unistd.h>
       
    off_t lseek(int fd, off_t offset, int whence);
        fd：文件描述符
        
        offset：偏移量  可正可负 单位是字节
                正代表向后偏移
                负代表向前偏移
                
        whence：基准定位方式
            SEEK_SET    基于文件的开头
                           offset给一个负数，这个时候会留下一个空洞
                           当对空洞没有把握的时候尽量不要去留空洞
            SEEK_CUR    基于光标的当前位置
            SEEK_END    基于文件的末尾
                            offset给一个正数，这个时候会留下一个空洞
                           当对空洞没有把握的时候尽量不要去留空洞
        返回值：
            成功返回偏移好了的位置到文件开头所有的字节数 --- 这个玩意儿就可以求文件的大小   
            失败返回-1，同时errno被设置
```

#### perror

```c
#include <stdio.h>

           void perror(const char *s);
```



#### setvbuf

```c
int setvbuf(FILE *stream, char *buffer, int mode, size_t size);

‌参数说明‌
‌stream‌
	需设置缓冲区的流指针，如 stdin、stdout、stderr 或通过 fopen 打开的文件

‌buffer‌

    用户自定义缓冲区的首地址。若为 NULL，函数自动分配缓冲
    若提供自定义缓冲区，其生命周期需持续到流关。
‌mode‌
缓冲模式类型：

    ‌_IOFBF‌（全缓冲）：缓冲区填满或流关闭时触发 I/O 操作。
    ‌_IOLBF‌（行缓冲）：遇到换行符 \n 或缓冲区填满时触发 I/O 操作。
    ‌_IONBF‌（无缓冲）：禁用缓冲，数据直接读写。

‌size‌
	缓冲区大小（字节数）。若 mode 为 _IONBF，此参数无效‌。

‌返回值‌
	成功返回 0，失败返回非零值（如参数无效）‌。





```





#### mmap

```c
SYNOPSIS
       #include <sys/mman.h>
       
        void *mmap(void *addr, size_t length, int prot, int flags,
              int fd, off_t offset);
        addr：地址 我们自己可以指定你要映射到内存的哪个地址上面去
                填NULL，表示操作系统自动给我指定一块
        length：你要映射的长度
                    如果是文件，就看你映射哪一节
                    单位是字节，但是以页进行内存分配
                    1页 = 4096字节
                如果是lcd屏幕   800 * 480 * 4
        prot：映射后对于内存的操作权限
            而这个内存本身对应的就是一个文件，而我们打开文件的时候就有权限
            因此一般按照你打开的文件的权限来
                PROT_EXEC   可执行
                PROT_READ   可读
                PROT_WRITE  可写
                PROT_NONE   没有
                PROT_READ | PROT_WRITE 读写
        flags：映射的标志
            MAP_SHARED  公有映射，对于内存的操作可以直接影响文件 --- 这种选择居多
            MAP_PRIVATE 私有映射，对于内存的操作不能直接影响文件
        fd：文件描述符，你要对哪个文件进行映射 
        offset：偏移量   要以页为单位进行偏移
            填0 表示不偏移
    返回值：
        成功返回映射后的首地址，后续就可以通过这个地址操作文件了
        失败返回MAP_FAILED，同时errno被设置

   int munmap(void *addr, size_t length);
        解映射，不用了要及时解除
    addr：一般就是映射的地址
    length：一般映射多大解除多大
    成功返回0失败返回-1
            
            
```

(1) MAP_PRIVATE（私有映射）

流程：

初始映射：
内核将文件内容加载到物理页，但标记为“只读”。
页表将虚拟地址映射到这些物理页。

写操作触发：
当进程尝试写入映射区域时，触发 页错误（Page Fault）。
内核为该页分配一个新物理页（拷贝原内容），并更新页表指向新页。
新页标记为“可写”，原页保持只读（可能被其他私有映射共享）。

修改隔离：
后续对该页的写操作仅影响新分配的物理页。
原文件和未修改的其他私有映射进程看不到这些修改。
数据持久化：
私有映射的修改不会写回原文件，除非显式调用 msync 并指定 MS_SYNC/MS_ASYNC。

(2) MAP_SHARED（共享映射）

流程：

初始映射：
内核将文件内容加载到物理页，标记为“可读可写”。
多个共享映射的进程共享同一物理页（或通过 COW 机制隔离写入）。

写操作触发：
写入操作直接修改共享的物理页。
该页被标记为脏页（Dirty Page），表示需要写回磁盘。

共享修改：
所有共享该映射的进程立即看到写入的变化。

数据持久化：
脏页最终通过内核的 后台写回线程（如 Linux 的 pdflush）异步写回原文件。
可显式调用 msync 加速同步。



#### time

```c
NAME
       time - get time in seconds

SYNOPSIS
       #include <time.h>

       time_t time(time_t *tloc);

DESCRIPTION
       time()  returns  the  time  as  the  number of seconds since the Epoch,
       1970-01-01 00:00:00 +0000 (UTC).

       If tloc is non-NULL, the return value is  also  stored  in  the  memory
       pointed to by tloc.

RETURN VALUE
       On  success,  the value of time in seconds since the Epoch is returned.
       On error, ((time_t) -1) is returned, and errno is set appropriately.

```

#### localtime

```c
NAME
       asctime,   ctime,   gmtime,   localtime,  mktime,  asctime_r,  ctime_r,
       gmtime_r, localtime_r - transform date and time to broken-down time  or
       ASCII

SYNOPSIS
       #include <time.h>

       char *asctime(const struct tm *tm);
       char *asctime_r(const struct tm *tm, char *buf);

       char *ctime(const time_t *timep);
       char *ctime_r(const time_t *timep, char *buf);

       struct tm *gmtime(const time_t *timep);
       struct tm *gmtime_r(const time_t *timep, struct tm *result);

       struct tm *localtime(const time_t *timep);
       struct tm *localtime_r(const time_t *timep, struct tm *result);

       time_t mktime(struct tm *tm);

struct tm {
    int tm_sec;    /* Seconds (0-60) */
    int tm_min;    /* Minutes (0-59) */
    int tm_hour;   /* Hours (0-23) */
    int tm_mday;   /* Day of the month (1-31) */
    int tm_mon;    /* Month (0-11) */
    int tm_year;   /* Year - 1900 */
    int tm_wday;   /* Day of the week (0-6, Sunday = 0) */
    int tm_yday;   /* Day in the year (0-365, 1 Jan = 0) */
    int tm_isdst;  /* Daylight saving time */
};


```

#### dup dup2

```c
NAME		//文件描述符重定向
       dup, dup2, dup3 - duplicate a file descriptor

SYNOPSIS
       #include <unistd.h>

       int dup(int oldfd);
       int dup2(int oldfd, int newfd);


```



##### U盘

U盘会自动挂载到 /mnt/udisk

## 触摸屏

```
触摸屏的文件 /dev/input/event0
linux触摸屏维护的头文件 #include <linux/input.h>
    这个头文件里面维护了一个结构体
struct input_event {
	struct timeval time;//你的输入事件触发的时间
               
	__u16 type; //你的输入事件的类型  用于区分什么输入设备
                    	//如鼠标  键盘 触摸屏                  
                    	#define EV_ABS			0x03   这个就是我们的触摸屏事件
                        如果获取触摸屏坐标 这个type必须等于 EV_ABS
	__u16 code;//编码  在触摸屏事件里面用于区分x轴与y轴     源点在左上方
                        #define ABS_X			0x00   x轴
                        #define ABS_Y			0x01   y轴
                        #define BTN_TOUCH		0x14a  这个值压力值
	__s32 value;// 值  触摸屏事件里面就是x y的坐标
  
};
struct timeval {
    time_t      tv_sec;     /* 秒 */
    suseconds_t tv_usec;    /* 微秒 */
};


```

## 文件操作函数

#### 获取文件的属性

###### stat

```
SYNOPSIS
       #include <sys/types.h>
       #include <sys/stat.h>
       #include <unistd.h>
       
       int stat(const char *pathname, struct stat *statbuf);
   			 pathname：路径名 你要获取哪一个文件的属性
   			 statbuf：用于保存获取到的文件属性
        struct stat {
            dev_t     st_dev;         /* 容纳这个文件的设备id */
            ino_t     st_ino;         /* inode结点，这个文件的地址 */
            mode_t    st_mode;        /* 文件的权限以及类型 */
            nlink_t   st_nlink;       /* 硬链接数 */
            uid_t     st_uid;         /* 文件所有者的用户id */
            gid_t     st_gid;         /* 文件所有者的组用户id */
            dev_t     st_rdev;        /* 设备id (如果该文件是一个设备) */
            off_t     st_size;        /* 文件内容的大小 */
            blksize_t st_blksize;     /* 块大小 */
            blkcnt_t  st_blocks;      /* 文件有多少块 */
            struct timespec st_atim;  /* 最后访问的时间 */
            struct timespec st_mtim;  /* 最后修改内容的时间 */
            struct timespec st_ctim;  /* 最后修改属性的时间 */

                
            #define st_atime st_atim.tv_sec      /* Backward compatibility */
            #define st_mtime st_mtim.tv_sec
            #define st_ctime st_ctim.tv_sec
        };
        struct timespec {
            time_t tv_sec;                /* Seconds */
            long   tv_nsec;               /* Nanoseconds */
        };
```

#### 获取你的进程的工作路径

###### getwd

```
   char *getwd(char *buf);//这个函数可能会出现越界问题
      buf：用于保存我们的工作路径 -> 就是一个字符串
      获取成功就会将工作路径的首地址返回给你 
```



###### getcwd

```
char *getcwd(char *buf, size_t size);  
		buf：用于保存我们的工作路径
        size:你最多获取多少个字节  指定之后越界风险没有了



```



#### 设置进程的工作路径

###### chdir

```
	SYNOPSIS
       #include <unistd.h>
       int chdir(const char *path);
       path：你要将工作目录换到哪里去  这里就是指定我们的工作目录
       
       
       
```

###### fchdir 

```
   	  int fchdir(int fd);
      fd：通过一个文件描述符来指定工作路径
    成功返回0失败返回-1，同时errno被设置
```

#### 文件的截短

###### truncate

```
 int truncate(const char *path, off_t length);
    path：你要弄哪个一个文件
    length：你要截到多大
 int ftruncate(int fd, off_t length);
    fd：用fd来指明你要截短的文件
    length：你要截到多大
        length < 原来的长度     截短到你指定的字节
        length > 原来的长度     扩展文件，会留空洞，如果你不及时将这个空洞补起来
                                    有可能会浪费存储空间
```

#### 删除文件

###### unlink

```
删除文件
#include<unistd.h>
int unlink(const char *pathname); 删除一个文件
    pathname：你要删除的文件
```

###### rmdir

```
删除一个文件夹
#include<unistd.h>
  int rmdir(const char *pathname);
    pathname：指定你要删除的文件夹的路径
```

## 目录操作函数

###### opendir/ fdopendir



```
SYNOPSIS
       #include <sys/types.h>
       #include <dirent.h>
  DIR *opendir(const char *name);
    name：你要打开的文件夹
    
  DIR *fdopendir(int fd);
    fd：用文件描述符去指定你要打开的目录
    成功返回一个DIR的指针，这个指针相当于文件描述符，用于指明你打开的这个目录       
```

###### readdir

```
SYNOPSIS
       #include <dirent.h>
   struct dirent *readdir(DIR *dirp);
    dirp：就是opendir返回的那个DIR指针
    返回值：
        成功返回目录项  struct dirent *
        struct dirent {
           ino_t          d_ino;       /* inode */
           off_t          d_off;       /* 目录项的偏移，你读到了第几项*/
           unsigned short d_reclen;    /* 这个结构体的长度 */
           unsigned char  d_type;      /* 文件的类型，不是所有的操作系统都是这么支持的
            d_type成员:
                DT_BLK      This is a block device.
                DT_CHR      This is a character device.
                DT_DIR      This is a directory.
                DT_FIFO     This is a named pipe (FIFO).
                DT_LNK      This is a symbolic link.
                DT_REG      This is a regular file.
                DT_SOCK     This is a UNIX domain socket.
                DT_UNKNOWN  The file type could not be determined.  
                
            char     d_name[256]; /* 这个目录项代表的那个文件的名字，不带路径的 */
       };
            如果没有跨平台的需求的时候，linux里面你该怎么用就怎么用
            但是如果要考虑跨平台，那么这个结构体里面你只能使用 d_ino d_name
    readdir每读一项就会往下面走一项，再次读的时候就会读到下一项
    如果读完了返回NULL

```

###### closedir

```
SYNOPSIS

      #include <sys/types.h>   
      #include <dirent.h>

   int closedir(DIR *dirp);
```

###### mkdir

```
SYNOPSIS
       #include <sys/stat.h>
       #include <sys/types.h>
int mkdir(const char *pathname, mode_t mode);
```

## 标准IO接口

            FILE * stdin; 标准输入 -> 对应的文件为 0 STDIN_FILENO (输入键盘的文件)
            FILE * stdout;标准输出 -> 对应的文件为 1 STDOUT_FILENO(终端的文件)
            FILE * stderr;标准出错 -> 对应的文件为 2 STDERR_FILENO
            
                    1 行缓冲：必须要填满一行才能同步到文件，
      
                    2 全缓冲：必须要填满整个缓冲区才能进行同步
                    3 无缓冲：只要有字节就会进行同步
                        perror就是无缓冲
                    遇到“\n”会刷新缓冲区    

###### fopen/ fdopen/ freopen

```
SYNOPSIS 总览
       #include <stdio.h>
       
        FILE *fopen(const char *path, const char *mode);
        path：你要打开的文件的路径名
        mode：打开文件的基础方式，这里是以字符串的形式进行指定的
            "r": 只读打开这个文件，如果这个文件不存在就会报错，打开后光标在开头
            "r+": 读写打开，如果这个文件不存在就会报错，打开后光标在开头
            "w": 只写打开这个文件，如果这个文件不存在则创建，打开后内存被截短(内容被清除了)
            "w+":读写打开，如果这个文件不存在则创建，打开后内存被截短(内容被清除了)
            "a":追加打开，如果这个文件不存在则创建，光标在末尾
            "a+":追加读写打开，如果这个文件不存在则创建，原始的写位置在文件的末尾
                原始读位置在文件的开头
    返回值：
        成功返回FILE *指针 -- 文件流 -> 作用类似于文件描述符
            用于描述打开后的这个文件的
        失败返回NULL
```

###### fgetc, fgets, getc, getchar

```
 SYNOPSIS
        #include <stdio.h>

        int getc(FILE *stream);//去stream指定的文件流里面获取一个字节
        int getchar(void);//指定去stdin里面读取一个字节
        int fgetc(FILE *stream);//去stream指定的文件流里面获取一个字节
            getc是用宏实现的，利用内存去换取执行的时间
            fgetc是函数,利用执行时间去换取更多的内存空间
        返回值返回给你读取的字节

```

###### fputc, fputs, putc, putchar

```
    SYNOPSIS
        #include <stdio.h>

        int fputc(int c, FILE *stream);//将c输出到stream代表的这个文件流
            fputc 为函数
            putc 为宏
        int putc(int c, FILE *stream);//将c输出到stream代表的这个文件流

        int putchar(int c); //指定往 stdout里面输出字符c
  
标准io将文件读完之后，会在缓冲区的末尾填充一个叫EOF的字节
```

###### fgets gets puts fputs

```
    char *fgets(char *s, int size, FILE *stream);//从stream代表的文件里面获取一行字符
    char *gets(char *s);//从stdin里面获取一行  这个函数有bug
                        //s代表的内存有可能越界  这个函数尽量不要用
        gets -> fgets(s,size,stdin)

    int fputs(const char *s, FILE *stream);//将s代表的字符串输出到stream代表的文件
    int puts(const char *s);//将s字符串输出到stdout
        char buf[3] = {'a','v','b'};
        fputs(buf,stdout);//很有可能出问题  传过去的字符串要注意以\0结尾
    上面这些基本都是针对于文本文件的
```

###### fread, fwrite

```c
SYNOPSIS
       #include <stdio.h>
       
        size_t fread(void *ptr, size_t size, size_t nmemb, FILE *stream);
        ptr：你读到的内容要放在哪个内存
        size：单个元素的大小
        nmemb：你要读多少个元素     总大小 = size * nmemb
        stream: 你要读哪个文件流
    返回值：
        返回成功读取的元素个数 <= nmemb
        
        失败返回0，同时errno被设置关键区别
        ‌场景‌				‌返回值‌		   	‌辅助判断函数‌
        正常读取			n（0 < n ≤ nmemb）		无需
        文件末尾（EOF）		0 或部分值				feof(file)
        错误发生			  0						ferror(file)
     /* size_t rc = fread(buf, sizeof(int), 10, file);
        if (rc < 10) {
            if (feof(file)) {
                printf("EOF: Read %zu integers (partial)\n", rc);
            } else if (ferror(file)) {
                perror("Read error");
            }
        }
     */

   size_t fwrite(const void *ptr, size_t size, size_t nmemb, FILE *stream);
        ptr：你要写入的内容
        size：你的元素的大小
        nmemb：元素个数
        stream：你要写入的文件
    返回值：
        实际写入成功的元素个数
        失败返回-1，同时errno被设置
```

###### fflush 

```
   int fflush(FILE *stream);
        stream：你要刷新的流
            这个传入NULL，表示刷新所有的流 
            一般尽量避免直接传入NULL
        程序如果正常退出，退出之前会将所有的流刷新
```

###### fgetpos, fseek, fsetpos, ftell, rewind 

```
   int fseek(FILE *stream, long offset, int whence);//定位光标
    stream：文件流
    offset：偏移量，单位字节  可正可负
    whence：基础定位方式
        SEEK_SET    开头
        SEEK_CUR    当前
        SEEK_END    末尾
    返回值：
        成功返回0，失败返回-1，同时errno被设置

   long ftell(FILE *stream);
        返回光标当前位置到文件开头所有的字节数
            lseek的效果 fseek + ftell
        fseek(fp,0x00,SEEK_END);
        long filesize = ftell(fp);//获取到文件的大小了

   void rewind(FILE *stream);//直接将光标定位到开头
        -> fseek(fp,0x00,SEEK_SET);
```

###### fclose

```
   SYNOPSIS 总览
       #include <stdio.h>
   int fclose(FILE *stream);
```

#### 格式化IO

###### scanf,  fscanf, sscanf

```
SYNOPSIS
       #include <stdio.h>
   int scanf(const char *format, ...);
   int fscanf(FILE *stream, const char *format, ...);
   int sscanf(const char *str, const char *format, ...);
        stream: 指定你的输入流是哪一个  fscanf(stdin,...) <-> scanf(...)
        str: 你的匹配来源于字符串，而不是文件流，你的输入来源就变成了一个内存
        format：格式化的字符串
            你必须要按照我制定的规则去走才可以
            有两种字符需要注意
                1 普通字符：原封不动的进行匹配
                    如果你没有这些字符就会匹配失败
                    "abc" -> 你在输入的时候就必须看到abc才能匹配
                        否则匹配不了
                        scanf("%d\n",&a); -> 必须要按2次回车才能匹配
                        scanf("abc%d",&a); -> 你必须要输入abc1200
                                                面前随便有一个不一样就无法匹配
                2 空白字符 会将空白字符给pass掉

                3 转义字符
                    %d -> 整数匹配 [0-9]+
                    %c -> 匹配任意的字符
                    %s -> 匹配字符串
                    %f -> 匹配浮点型
                    .....
        ... :和我们的转义字符匹配的地址
            也就是你的转义字符给了多少个，后面的地址就要给多少个
                    %%

        返回值为成功匹配的个数

格式化的输出
```

######  printf,   fprintf,  dprintf， sprintf,  snprintf

```
int printf(const char *format, ...);//固定在stdout上面
   int fprintf(FILE *stream, const char *format, ...);//往stream输出
   int dprintf(int fd, const char *format, ...);//往fd里面输出
   int sprintf(char *str, const char *format, ...);//往str内存里面输出
   int snprintf(char *str, size_t size, const char *format, ...);//往str内存里面输出size个字节
        stream：文件流
        fd：文件描述符
        str：内存
        size：多少字节
        format：格式化字符串
            见上面就行了，注意的是空白符也会原封不动的输出
        ... ：  每个%对应一个右值就行了  
```

# 并发----------------



### 创建一个进程

###### fork 

```
NAME
       fork - create a child process
            创建一个子进程
SYNOPSIS
       #include <sys/types.h>
       #include <unistd.h>
       
       pid_t fork(void);
    这个函数没有参数，程序走到这句代码的时候
        只要没有失败，我们自己就会开辟出来一个进程 -- 子进程
    pid_t：
        父进程里面的fork会返回子进程的id > 0
        子进程里面的fork就会返回 0 
        <0  fork调用失败，同时errno被设置
```







### 获取自己的id和父id

###### getpid, getppid

```
SYNOPSIS
       #include <sys/types.h>
       #include <unistd.h>
   pid_t getpid(void); //返回自己的id
   pid_t getppid(void);//返回自己的父id
```

### 进程的退出

###### exit， _exit

```c
NAME
       exit - cause normal process termination

SYNOPSIS
       #include <stdlib.h>

    功能: 使进程正常终止
void exit(int status);
	status: 退出码，退出状态(一般写0表示正常退出)
		退出码的具体含义，由用户自己来进行指定
注意：
        exit 正常退出，做一些清理工作(如: 把缓冲区中的内容，进行同步...)

NAME
       _exit, _Exit - terminate the calling process

SYNOPSIS
       #include <unistd.h>

void _exit(int status);
	status: 退出码，退出状态(一般写0表示正常退出)
		退出码的具体含义，由用户自己来进行指定
注意:
	_exit  立马退出，终止进程，不会做清理工作
```

### 等待子进程退出

###### wait, waitpid, waitid

```c
 
       #include <sys/types.h>
       #include <sys/wait.h>

pid_t wait(int *wstatus);//阻塞等待它的子进程的状态发生改变
   
        wstatus：获取子进程退出的时候的状态码
                用于描述子进程的退出信息的
                    我们需要用宏去解析这个状态信息
                    WIFEXITED(wstatus)   
                        如果你的子进程是正常退出的，那么这个宏就会返回true
                    WEXITSTATUS(wstatus)
                        返回子进程的退出码，WIFEXITED(wstatus)为true的时候这个玩意儿才有意义
                    WIFSIGNALED(wstatus)
                        如果子进程是被信号杀死的，这个宏就会返回true
                    ........
            
            
    pid_t waitpid(pid_t pid, int *wstatus, int options);
        pid:进程id
            pid == -1   等待任意的子进程
            pid == 0    等待与调用进程同组的子进程
                            进程都会在进程组里面，每一个进程都会分到某一个组里面
                                每个组会有一个组长，组长的id就是组id
            pid < -1    等待与pid绝对值组的任意子进程
            pid > 0     等待id为pid的子进程，指定等待子进程                 
        wstatus：用于存储子进程的退出状态。如果不需要这个状态，可以传递 NULL。
        options：
            0    阻塞等待
        WNOHANG：如果指定的子进程没有结束，则立即返回 0，而不是阻塞等待。
            
        WUNTRACED：如果子进程被停止（比如由于接收到一个停止信号），则返回它的状态。这个选项通常与 WIFSTOPPED 宏一起使用来检查停止的子进程。
            
        WCONTINUED：如果子进程在调用 waitpid 之前被停止，但之后被继续执行，则返回它的状态。
    返回值：
        成功返回等待的子进程的id，失败返回-1，同时errno被设置
```

### exec函数族

###### execl, execlp, execle

```c
NAME
       execl, execlp, execle, execv, execvp, execvpe - execute a file

SYNOPSIS
       #include <unistd.h>

extern char **environ;

	exec函数族是让一个进程去执行另外一个程序文件，或者说，exec函数族的作用就是让一个指定的程序文件中的数据和指令替换到 当前调用进程  的数据和指令
int execl(const char *path, const char *arg, .../* (char  *) NULL */);
int execlp(const char *file, const char *arg, .../* (char  *) NULL */);
int execle(const char *path, const char *arg, .../*, (char *) NULL, char * const envp[] */);
int execv(const char *path, char *const argv[]);
int execvp(const char *file, char *const argv[]);
int execvpe(const char *file, char *const argv[],char *const envp[]);

功能: 
	exec函数族，功能都是类似的，都是用来执行外部程序的，仅仅是参数的写法略有不同
	exec函数族，名字中的l, 是list(列表的意思)
	exec函数族，名字中的p, 是path(路径)的意思，即要执行的那个外部程序的路径，是从PATH变量指定的路径中进行搜索
	exec函数族，名字中的v, 是vector(数组/向量) 的意思

说明：
	参数列表最后面加上NULL

返回值:
	只有失败的时候才返回-1，同时errno被设置
	成功的时候没有返回值，因为调用进程的程序指令已经被替换成新的程序了


```



### 守护进程

###### getsid

```c
NAME
       getsid - get session ID  //查看进程的会话ID

SYNOPSIS
       #include <sys/types.h>
       #include <unistd.h>

 	pid_t getsid(pid_t pid);
 	
返回值:
       On  success,  a  session  ID is returned. 
       On error, (pid_t) -1 will be returned, and errno is set 			  	       appropriately.

```

###### setsid

```c
NAME
       setsid - creates a session and sets the process group ID

SYNOPSIS
       #include <sys/types.h>
       #include <unistd.h>

       pid_t setsid(void);

返回值:
       On success, the (new) session ID of the calling  process  is  returned.
    
       On  error,  (pid_t) -1  is  returned,  and errno is set to 							indicate the error.



```

###### 模型

```
https://blog.csdn.net/JMW1407/article/details/108412836
```

本人代码

```c
#include <stdio.h>
#include <sys/types.h>
#include <unistd.h>
#include <stdlib.h>
#include <sys/stat.h>
#include <signal.h>
#include <sys/wait.h>

/*
父进程，子进程，孙子进程
子进程退出，由孙子进程执行守护进程
这样父进程可以执行自己任务

*/

//父进程替子进程收尸      		守护进程已经不受父进程托管
void sighandler(int signal)
{
	if(SIGCHLD == signal)
	{
		int wstatus;
		pid_t wpid = waitpid(-1, &wstatus, 0);
		if(wpid<0)
		
{
			perror("wpid ");
		}
		printf("Child %d exited with status %d\n", wpid, WEXITSTATUS(wstatus));

	}
}

int main()
{
	pid_t pid = fork();			//第一步，创建子进程
	if(pid < 0)
	{
		perror("fork pid");
		return 1;
	}
	else if(pid == 0)	//子进程
	{
		pid_t ppid = fork();	//第二步，创建孙子进程，子进程退出
		if(ppid < 0)
		{
			perror("fork ppid");
			return 1;
		}
		else if(ppid)
		{
			printf("son death this your grandson :%d\n",ppid);
			exit(2);
		}
		setsid();				//第三步，创建新会话
		chdir("/");				//第四步，将当前目录改为根目录
        umask(0);				//第五步，重新设置文件权限掩码
        close(STDIN_FILENO);	//第六步，关闭不需要的文件描述符
        close(STDOUT_FILENO);
        close(STDERR_FILENO);
        //=========================守护进程的执行任务===================
		int i =0;
		char buf[50]={0};
        while(1)
        {
			sleep(3);
			sprintf(buf,"touch /mnt/hgfs/share/ff/%d.txt",i++);
			system(buf);
		
        }	
        //=============================================================
	}
	
	//=========================父进程的执行任务===================
	signal(SIGCHLD,sighandler);
	while(1)
	{
		sleep(1);
		printf("I'm father doing \n");

	}
	//=============================================================
	return 0;
}

```



# 进程通信------------------

###  无名管道

###### pipe

它在文件系统中没有名字(没有inode), 它的内容在内核空间中，访问pipe的方式是通过文件系统的API(read,  write)


它不能用open, 但是read/write又需要一个文件描述符。所以在创建这个无名管道的时候，就必须返回文件描述符。


无名管道在创建的时候，**在内核空间开辟一块缓冲区，作为无名管道文件的内容存储空间，同时返回两个文件描述符(一个用来读，一个用来写)**



```c
NAME
       pipe, - create pipe

SYNOPSIS
       #include <unistd.h>

    功能: 用来在内核空间中创建一个无名管道，pipefd用来保存创建好的无名管道的两个文件描述符，pipe创建的管道，默认是"阻塞方式"
int pipe(int pipefd[2]);
	参数:
		pipefd[0]  保存读的文件描述符
		pipefd[1]  保存写的文件描述符
返回值:
	成功返回0
	失败返回-1，同时errno被设置
```

pipe(无名管道)只能作用于具有亲缘关系之间的进程间通信，就因为pipe没有名字。假设它在文件系统中有一个名字，它就可以用于任意进程间的通信了。



###  有名管道

###### mkfifo

fifo是在pipe的基础上，给有名管道在文件系统中创建了一个inode(它会在文件系统中有一个文件名)，但是它的内容是保存在内核空间中的。

有名管道和无名管道的操作步骤几乎一样，除了有 名管道在文件系统中有一个名字


>操作有名管道：
>		创建管道文件  --->  open  --->  read/write   ---->  close



```c
NAME
       mkfifo, mkfifoat - make a FIFO special file (a named pipe)

SYNOPSIS
       #include <sys/types.h>
       #include <sys/stat.h>
	功能: 用于在文件系统中创建一个fifo的入口
int mkfifo(const char *pathname, mode_t mode);
	pathname: 要创建的有名管道在文件系统中的名字(路径)
	mode: 创建的有名管道的权限
 	有两种方式可以指定文件的权限：  ----> rwx rwx rwx	
	(1)
		用户  S_IRUSR  S_IWUSR  S_IXUSR
		组用户 S_IRGRP  S_IWGRP S_IXGRP
		其他用户 S_IROTH S_IWOTH S_IXOTH
		如：
			S_IRUSR | S_IRGRP | S_IROTH
			==> r--r--r--
	(2)使用八进制来描述权限
		如： 0666
			==> 110 110 110 
			==> rw-rx-rw-
返回值:
	成功返回0
	失败返回-1，同时errno被设置

描述:
	FIFO(有名管道)它和PIPE(无名管道)类似，除了它在文件系统中有一个名字。它可以被多个进程打开用来读或者写。   当进程用FIFO来交换数据时，内核没有把数据写入到文件系统，而是保存在内核空间中，因此FIFO在文件系统中时没有内容，它仅仅是作为文件系统的一个引用入口，提供一个文件名，给其他进程去open.


注意:
	创建一个有名管道的时候要排除一个文件已经存在的错误  errno == EEXIST

        
阻塞方式(默认):
	阻塞：等待
	  如果文件没有内容，read会阻塞，直到有数据或者出错
      如果文件没有空间了，write会阻塞，直到有空间或者出错
        
O_NONBLOCK  非阻塞标志位  把文件以非阻塞的方式打开
	非阻塞： 不等待
		如果文件没有内容了，read不会阻塞，直接返回一个错误
		如果文件没有空间了，write不会阻塞，直接返回一个错误
				
打开FIFO的一端，会阻塞，直到另一端也打开。但是如果以可读可写的方式打开则写端不会阻塞，读端依然会阻塞。
如果写入数据的时候，读端没有打开，则读端读不到上一次写入的数据
```

管道通信的特点：



- **单向通信** ： 管道适用于单向通信，数据只能从管道的一端流向另一端，防止用户误操作。
- **亲缘关系限制**: 无名管道只用于具有亲缘关系的进程，通常是父子进程。而有名管道则没有这个限制
- **生命周期随进程**: 无名管道的生命周期随进程，当进程结束时，管道也会销毁。
- **基于字节流的通信方式**： 提供面向字节流的通信服务，没有格式，读多少，完全由上层决定
- **半双工模式**: 管道是半双工的，数据同一时刻，只能向一个方向流动。需要双方通信时，需要创建两个管道
- **缓冲区限制**： 管道本质是内核中的一块缓冲区，多个进程需要访问同一块缓冲区实现通信。在无名管道中，写快，读慢，写满时不能继续写； 写关闭，会读到0， 标识读到管道文件的结尾； 读关闭，写继续写，操作系统会终止写进程
- **同步于互斥**： 管道已经实现了同步，具有数据一致性。内核会对管道操作进行同步和互斥，确保数据正确的传输。



### 信号

在ubuntu中可以通过 **kill  -l**可以查看系统中的所有信号
![image-20240725151024858](D:/文件接收/2_并发/2_进程间通信/note/进程间通信.assets/image-20240725151024858.png) 



我们可以通过 man  7   signal  查看所有信号的信息

以下是一些常用的信号

```c
       Signal     Value     Action   Comment
       ──────────────────────────────────────────────────────────────────────
       SIGINT        2       Term    Interrupt from keyboard
       SIGFPE        8       Core    Floating-point exception
       SIGKILL       9       Term    Kill signal
       SIGSEGV      11       Core    Invalid memory reference
       SIGPIPE      13       Term    Broken pipe: write to pipe with no
                                     readers; see pipe(7)
       SIGALRM      14       Term    Timer signal from alarm(2)
       SIGCHLD   20,17,18    Ign     Child stopped or terminated

2): 按键Ctrl+C产生，默认能终止进程
8): 浮点数异常，如除数为0的时候，会产生这个信号
9): 由用户主动发出，用于结束指定的进程
11): 由于程序非法访问内存，系统发出该信号，默认终止进程(段错误)
13): 管道破裂  往管道中写数据的时候，读端不存在，进程会收到该信号
14): 闹钟信号  一般用于定时，当时间到了，就会主动发生该信号
20,17,18): 子进程结束的时候，父进程会收到该信号
```



####  相关API

######  kill

```c
NAME
       kill - send signal to a process

SYNOPSIS
       #include <sys/types.h>
       #include <signal.h>

    功能: 给指定的进程发送指定的信号
int kill(pid_t pid, int sig);
	pid：给pid对应的进程发送信号
	sig : 可以用信号的值，也可以用信号的名字，建议用名字
返回值:
	成功返回0
	失败返回-1，同时errno被设置
```

######  raise

```c
NAME
       raise - send a signal to the caller

SYNOPSIS
       #include <signal.h>
	功能: 给调用进程发送一个信号
int raise(int sig);
	sig: 可以用信号的值，也可以用信号的名字，建议用名字
返回值:
	成功返回0
	失败返回-1，同时errno被设置

	raise(sig) <==> kill(getpid(), sig)
```

###### signal

```c
NAME
       signal - ANSI C signal handling

SYNOPSIS
       #include <signal.h>

typedef void (*sighandler_t)(int);

	功能: 处理指定的信号
sighandler_t signal(int signum, sighandler_t handler);
	signum: 需要处理的信号(可以是信号的值，也可以是信号的名字)
	handler: 信号处理器(信号处理函数)
说明:
	信号处理器可以是  SIG_IGN  表示忽略指定的信号
	信号处理器可以是  SIG_DEF  表示默认的处理方式(默认处理方式大多数是终止进程)
	信号处理器可以是  自定义的函数，该函数原型如下:
		void (*sighandler_t)(int);
	该函数不需要返回值，由一个int型的参数(就是需要进行处理的信号，即signal的第一个参数)

注意:
	SIGKILL和SIGSTOP不等被捕获和忽略

当制定的信号发送时，系统会自动调用信号处理器
```

###### sigaction

```c
#include <signal.h>  
  
int sigaction(int sig,
            const struct sigaction *act, 
            struct sigaction *oact);

sig：指定要修改其处理方式的信号。

act：指向一个 sigaction 结构体的指针，该结构体包含了新的信号处理信息。如果为 NULL，则不会改变信号的处理方式，只是用来查询当前信号的处理方式。

oact：如果非 NULL，则指向一个 sigaction 结构体的指针，用来保存调用 sigaction 之前的信号处理方式。

struct sigaction {  
    void     (*sa_handler)(int); // 信号处理函数，或 SIG_IGN、SIG_DFL  
    void     (*sa_sigaction)(int, siginfo_t *, void *); // 实时信号的处理函数  
    sigset_t   sa_mask;          // 在处理信号时，将要阻塞的信号集  
    int        sa_flags;         // 调用信号处理程序的选项  
    void     (*sa_restorer)(void);// 已废弃，不用设置  
};

sa_handler：这是传统的信号处理函数指针，类似于 signal 函数的第二个参数。可以设置为 SIG_IGN 忽略信号，或 SIG_DFL 采用默认处理，或指向一个自定义的信号处理函数。

sa_sigaction：这是对于实时信号（在 POSIX.1-1993 中定义的信号）的处理函数，提供了更多的上下文信息，如信号的来源和附加数据。

sa_mask：在信号处理函数执行期间，需要阻塞的信号集合。这有助于防止在信号处理函数执行期间被其他信号中断。

sa_flags：指定信号处理的选项，例如 SA_RESTART（如果信号中断了系统调用，则自动重启系统调用），SA_NODEFER（在执行信号处理函数时不自动阻塞当前信号），以及 SA_SIGINFO（使用 sa_sigaction 而不是 sa_handler）。
    

sigset_t mask;  

// 初始化信号集为空  
sigemptyset(&mask);  

// 假设我们想阻塞 SIGINT 信号  
sigaddset(&mask, SIGINT); 

```

```c
void handler(int sig, siginfo_t *info, void *ucontext)
{
    ...
}

siginfo_t 
{  
    int      si_signo;     /* 信号编号 */  
    int      si_errno;     /* errno值，表示引起信号的错误码（如果有的话）*/  
    pid_t    si_pid;       /* 发送信号的进程ID */  
    uid_t    si_uid;       /* 发送信号的进程的真实用户ID */  
    int      si_status;    /* 退出值或信号（如果是子进程通过信号终止）*/  
    clock_t  si_utime;     /* 消耗的用户时间（在信号产生时）*/  
    clock_t  si_stime;     /* 消耗的系统时间（在信号产生时）*/  
    sigval_t si_value;     /* 信号值，与信号关联的数据 */  
    int      si_fd;        /* 文件描述符（与某些信号相关，如SIGPOLL/SIGIO）*/
}
```

```c
 //注册函数一般步骤

void sigint_handler(int signo) 
{
   
}
	struct sigaction sa;

    // 设置信号处理函数为sigint_handler
    sa.sa_handler = sigint_handler;
    // 清空sa_mask，即不阻塞任何其他信号
    sigemptyset(&sa.sa_mask);
    // 设置一些标志位，这里使用默认值0
    sa.sa_flags = 0;

    // 注册对SIGINT信号的处理方式
    if (sigaction(SIGINT, &sa, NULL) == -1) 
    {
        perror("sigaction");
      
    }

```



###### pause

```c
NAME
       pause - wait for signal

SYNOPSIS
       #include <unistd.h>

     功能: 使得调用进程/线程休眠，直到接受到一个信号
int pause(void);
```

###### alarm

```c
NAME
       alarm - set an alarm clock for delivery of a signal

SYNOPSIS
       #include <unistd.h>
	
    功能: 设置一个指定时间的闹钟，时间超过则发送SIGALRM信号
unsigned int alarm(unsigned int seconds);
	seconds: 时间秒数
		如果设置为0， 表示取消之前设置的闹钟
		不为0，则表示设置闹钟(如果之前已经设78787钟，则把之前的闹钟取消，重新设置)

返回值:
	闹钟正常超时，返回0
	由闹钟被取消则返回被取消的那个闹钟的剩余时间
```

### 消息队列

#### 通用的获取KEY的API

###### ftok

```c
NAME
       ftok - convert a pathname and a project identifier to a System V IPC key

SYNOPSIS
       #include <sys/types.h>
       #include <sys/ipc.h>

    功能： 把一个存在且能访问的路径名，和一个项目ID\(它的低8位不能为0)转化为一个IPC对象的key
key_t ftok(const char *pathname, int proj_id);
	pathname: 一个存在且能访问的路径名
	proj_id: 项目ID(低8位不能为0)

返回值:
	成功返回一个key, 用来获取或者创建一个IPC对象
	失败返回-1，同时errno被设置
```

#### 

######  msgget

```c
NAME
       msgget - get a System V message queue identifier

SYNOPSIS
       #include <sys/types.h>
       #include <sys/ipc.h>
       #include <sys/msg.h>

    功能: 用来创建或获取一个消息队列
int msgget(key_t key, int msgflg);
	key: 用来创建/获取消息队列的key, 即ftok的返回值
	msgflg: 创建或获取消息队列时的标志或权限
		IPC_CREAT
			消息队列不存在则创建，如： IPC_CREAT | 0664
			消息队列如果存在，则忽略该标志
		IPC_EXCL
			如果消息队列存在，则报错EEXIT

	创建时权限的说明:
		一般就是用8进制描述，且只有读写的权限，执行权限无效

返回值:
	成功返回一个消息队列的ID
	失败返回-1，同时errno被设置
```

###### msgsnd/msgrcv

```c
NAME
       msgrcv, msgsnd - System V message queue operations

SYNOPSIS
       #include <sys/types.h>
       #include <sys/ipc.h>
       #include <sys/msg.h>

    功能: 把消息写入到消息队列
int msgsnd(int msqid, const void *msgp, size_t msgsz, int msgflg);
	msqid: 消息队列的ID, 即msgget的返回值
	msgp: 要发送的消息结构的地址
	msgsz: 要发送的消息的大小，单位是字节(不包括消息类型，就是消息数据的大小)
	msgflag: 发送标志，一般设置为0表示阻塞，或者设置为IPC_NOWAIT，表示不阻塞
返回值:
	成功返回0
	失败返回-1，同时errno被设置

消息队列的一般格式：
struct msgbuf {
     long mtype;       /* message type, must be > 0 */
    					//消息类型  必须大于0
     char mtext[1];    /* message data */
						//消息的数据
};

	功能: 从消息队列中接受消息
ssize_t msgrcv(int msqid, void *msgp, size_t msgsz, long msgtyp,int msgflg);
	msqid: 消息队列的ID, 即msgget的返回值
	msgp: 用于接收消息的存储空间的首地址
	msgsz: 待接收消息的大小，单位是字节(不包括消息类型，就是消息数据的大小)
	msgtyp： 消息类型，有以下情况
		等于0 : 读取消息队列中的第一条消息
		大于0 : 表示读取消息队列中的第一条类型为msgtyp的消息
		小于0 : 表示读取消息队列中的第一条类型小于或者等于msgtyp绝对值的消息
	msgflag: 接收标志，一般设置为0表示阻塞，或者设置为IPC_NOWAIT，表示不阻塞

返回值:
	成功返回实际读取到的消息大小
	失败返回-1，同时errno被设置
```

######  msgctl

```c
NAME
       msgctl - System V message control operations

SYNOPSIS
       #include <sys/types.h>
       #include <sys/ipc.h>
       #include <sys/msg.h>

    功能: 用于控制消息队列  如： 控制消息队列的所有者，权限，删除....
int msgctl(int msqid, int cmd, struct msqid_ds *buf);
	msqid: 消息队列的ID
	cmd: 控制指令，有很多，常用的就是删除
		IPC_STAT: 获取消息队列的属性信息，把属性信息存储到buf指向的空间中
		IPC_RMID : 删除消息队列， 此时第三个参数填NULL
	buf: 消息队列属性信息结构体指针
返回值:
	成功返回0
	失败返回-1，同时errno被设置
```

### 共享内存

#### system V------------

测试代码

```c
//共享内存 进程1
int main()
{
    //获取一个key
    key_t key =  ftok("/",345);
    if(-1 == key)
    {
        perror("ftok error");
        return -1;
    }
    //创建或者打开一个共享内存
    int shmid = shmget(key,4096,IPC_CREAT | 0666);
    if(-1 == shmid)
    {
        perror("shmget error");
        return -2;
    }

    //将共享内存映射到进程的内存空间
    char * ptr = shmat(shmid, NULL,0);
    if(NULL == ptr)
    {
        perror("shmat error");
        return -3;
    }
    //操作共享内存  将消息写到共享内存里面去
    //strcpy(ptr,"pengleihenshuai");
    char buf[1024];
    memset(ptr,0,4096);
    while(1)
    {
        scanf("%s",buf); 
        strcat(ptr,buf); 
    }


    //解映射
    shmdt(ptr);

    //这个代码马上走完了 不能将共享内存删除  删除了别人就用不了

    return 0;
}
```



####  共享内存的API

######  shmget

```c
SYNOPSIS
       #include <sys/ipc.h>
       #include <sys/shm.h>

    功能: 用于创建/获取一段由内核提供的共享内存
int shmget(key_t key, size_t size, int shmflg);
	key: 键值  即ftok成功的返回值
	size: 共享内存的大小，单位为字节，会自动向上取整为PAGE_SIZE(4K)的整数倍
	shmflg:
		创建的标志与权限  如：  IPC_CREAT | 0664
返回值:
	成功返回共享内存的ID
	失败返回-1，同时errno被设置
        
创建或打开：如果 shmflg 包含了 IPC_CREAT 标志，那么 shmget 会尝试创建一个新的共享内存段。如果共享内存段已经存在（即已经有一个具有相同 key 的共享内存段），并且没有指定 IPC_EXCL，那么 shmget 会返回该共享内存段的标识符（即成功“打开”了已存在的共享内存段）。
        
创建且唯一：如果 shmflg 同时包含了 IPC_CREAT 和 IPC_EXCL 标志，那么 shmget 会尝试创建一个新的共享内存段。如果共享内存段已经存在，shmget 会失败并返回 -1，设置 errno 为 EEXIST。
        
仅打开：如果 shmflg 不包含 IPC_CREAT，那么 shmget 会尝试打开一个已存在的共享内存段。如果共享内存段不存在，shmget 会失败并返回 -1，设置 errno 为 ENOENT。
```

######  shmat，shmdt

```c
SYNOPSIS
       #include <sys/types.h>
       #include <sys/shm.h>
	功能： 映射指定的共享内存到进程的用户空间
void *shmat(int shmid, const void *shmaddr, int shmflg);
	shmid: 共享内存的ID, 即shmget成功的返回值
        
	shmaddr: 一般填NULL, 表示由操作系统自动选择一块合适的用户空间进行映射
        
	shmflag: 映射标志  一般填0，表示使用默认权限，即与shmget指定的权限保持一致
返回值:
	成功返回一个映射好的用户空间的首地址
	失败返回(void*)-1,同时errno被设置

    功能: 解除映射
int shmdt(const void *shmaddr);
	shmaddr： 待解除映射区域的首地址，即shmat成功的返回值
返回值:
	成功返回0
	失败返回-1，同时errno被设置
```

######  shmctl

```c
SYNOPSIS
       #include <sys/ipc.h>
       #include <sys/shm.h>

   功能: 用于控制共享内存
int shmctl(int shmid, int cmd, struct shmid_ds *buf);
	shmid: 共享内存的ID，通过调用shmget()函数获得
	cmd: 控制指令，有很多，常用的就是删除
		IPC_STAT: 获取共享内存的属性信息，把属性信息存储到buf指向的空间中
		IPC_RMID : 删除共享内存， 此时第三个参数填NULL
	buf: 共享内存属性信息结构体指针
返回值:
	成功返回0
	失败返回-1，同时errno被设置
```



#### posix---------------

```c
1. POSIX 共享内存的工作原理
关键步骤
创建共享内存对象：

int shm_fd = shm_open("/my_shm", O_CREAT | O_RDWR, 0666);
shm_open()：
接受路径名（如 "/my_shm"）作为共享内存对象的唯一标识。
不创建物理文件：路径名仅用于内核的命名空间管理，而非文件系统文件。
返回一个 文件描述符（shm_fd），后续用于 ftruncate() 和 mmap()。
映射到内存：

void *ptr = mmap(0, size, PROT_READ | PROT_WRITE, MAP_SHARED, shm_fd, 0);
mmap() 将共享内存对象映射到进程的地址空间。
删除共享内存对象：

shm_unlink("/my_shm"); // 逻辑删除，实际释放需等待所有进程解除映射
```



### 

### 信号量

####  system V------------



######  semget

```c
NAME
       semget - get a System V semaphore set identifier

SYNOPSIS
       #include <sys/types.h>
       #include <sys/ipc.h>
       #include <sys/sem.h>

       int semget(key_t key, int nsems, int semflg);
            key：我们申请的一个key
            nsems:信号量集中信号量的数量
            semflg：标志
                IPC_CREAT | 权限    创建
        返回值:
               成功返回这个信号量集合的id，
               失败返回-1，同时errno被设置
                   
如果 semflg 包含了 IPC_CREAT 标志，那么 semget 会尝试创建一个新的信号量集。如果信号量集已经存在（即已经有一个具有相同 key 的信号量集），并且没有指定 IPC_EXCL，那么 semget 会返回该信号量集的标识符（即成功“打开”了已存在的信号量集）。
                   
如果 nsems 参数指定的信号量数量大于已存在信号量集中的信号量数量，并且 semflg 包含了 IPC_CREAT，则已存在的信号量集会被扩展以包含指定数量的信号量。但是，如果 semflg 没有包含 IPC_CREAT，并且尝试访问的信号量数量大于已存在信号量集中的信号量数量，则 semget 会失败。    
```

######  semop（PV操作）

```c
NAME
       semop, semtimedop - System V semaphore operations

SYNOPSIS
       #include <sys/types.h>
       #include <sys/ipc.h>
       #include <sys/sem.h>

    功能: 用于操作信号量数组中的信号量(PV操作)
   int semop(int semid, struct sembuf *sops, size_t nsops);
        semid：你要操作哪一个信号量集
        sops：我们要怎么去操作这个信号量
        nsops:sops是一个数组  这个参数就是指明sops里面有几个元素
            你可以一次操作多个信号量
        struct sembuf{
            unsigned short sem_num;  /* 你要操作哪一个信号量 0 1 2 3...*/
            
            short          sem_op;   
            				/* 你的基础操作PV操作就看它了
                            正数表示你要让sem的值增长，那么就是V操作
                            负数表示你要让sem的值减小，那么就是P操作   
                            +1  -> 增长  因此是V
                            -1  -> 减小  因此是P */
                            
            short          sem_flg;  /* 操作标志*/
                             0          表示阻塞    
                             IPC_NOWAIT 非阻塞
                             SEM_UNDO   撤销
                                        
        };
       返回值:
            成功返回0
            失败返回-1，同时errno被设置


   int semtimedop(int semid, struct sembuf *sops, size_t nsops,
                  const struct timespec *timeout);//限时等待
        timeout：超时时间 就在这里等这么多时间  没有上锁就会走人
            防止等待的时间过长
```

######  semctl

```c
 #include <sys/types.h>
   #include <sys/ipc.h>
   #include <sys/sem.h>
        信号量集的控制接口
   int semctl(int semid, int semnum, int cmd, ...);
        semid：你要控制哪个信号量集合
            
        semnum：你要操作信号量集合里面哪一个信号量
                    从0开始，1，2，3，4.......
                    当我们一次操作所有的信号量的时候，这个参数就没有意义了
            
        cmd：命令号 常用的如下
            GETALL  获取所有的
            GETVAL  获取某一个
            SETALL  设置所有的
            SETVAL  设置某一个
            IPC_RMID删除
            
        第四个参数：根据cmd不同，这个参数会有不同
                cmd == GETVAL  获取某一个信号量的值
                    获取第二个参数指定的哪个信号量的值
                    通过返回值就将这个信号量的值返回给你了
                    第四个参数就没有意义了
                    eg:
                        int val = semctl(semid,1,GETVAL);
                cmd == SETVAL   
                    设置第二个参数指定的哪个信号量的值
                    第四个参数就是代表你要设置的值
                    eg:
                        semctl(semid,1,SETVAL,2);
                            将第二个信号量的值设置为2
                cmd == GETALL   获取所有的信号量的值
                    第二个参数会被忽略
                    第四个参数为ushort的数组
                        ushort values[5];
                        semctl(semid,0,GETALL,values);
                    所有的信号量的值就会写入values这个数组
                cmd == SETALL 设置所有的信号量的值
                    第二个参数会被忽略
                    第四个参数为ushort的数组
                        ushort values[5] = {1,2,3,4,5};
                        semctl(semid,0,SETALL,values);
                        这么弄了之后，信号量集里面第一个信号的值为1
                        第二信号的值为2......
                cmd == IPC_SET  设置头部信息
                    第四个参数就是struct semid_ds的指针
                cmd == IPC_RMID
                    第2个参数没有作用了，被忽略....
        由于第四个参数的种类有很多
            因此为了方便去描述第四个参数，可以利用联合体去描述
                union semun {
                    int              val;    /* Value for SETVAL */
                    struct semid_ds *buf;    /* Buffer for IPC_STAT, IPC_SET */
                    unsigned short  *array;  /* Array for GETALL, SETALL */
                    struct seminfo  *__buf;  /* Buffer for IPC_INFO
                                                (Linux-specific) */
                };

```

###### 示例代码

```c
#include <stdio.h>  
#include <stdlib.h>  
#include <sys/types.h>  
#include <sys/ipc.h>  
#include <sys/sem.h>  
#include <unistd.h>  
  
union semun {  
    int val;  
    struct semid_ds *buf;  
    unsigned short *array;  
};  
  
int main() {  
    key_t key = ftok("/tmp", 'R'); // 创建唯一的键值  
    int semid = semget(key, 1, IPC_CREAT | 0666);  
    if (semid == -1) {  
        perror("semget");  
        exit(EXIT_FAILURE);  
    }  
  
    union semun sem_union;  
    sem_union.val = 1; // 初始化信号量的值为1  
    if (semctl(semid, 0, SETVAL, sem_union) == -1) {  
        perror("semctl");  
        exit(EXIT_FAILURE);  
    }  
  
    // P操作，等待信号量  
    struct sembuf sb = {0, -1, 0};  
    if (semop(semid, &sb, 1) == -1) {  
        perror("semop");  
        exit(EXIT_FAILURE);  
    }  
  
    // 访问共享资源...  
  
    // V操作，释放信号量  
    sb.sem_op = 1;  
    if (semop(semid, &sb, 1) == -1) {  
        perror("semop");  
        exit(EXIT_FAILURE);  
    }  
  
    // 删除信号量集  
    if (semctl(semid, 0, IPC_RMID, 0) == -1) {  
        perror("semctl(IPC_RMID)");  
        exit(EXIT_FAILURE);  
    }  
  
    return 0;  
}
```

```c
//多信号量的使用
#include <stdio.h>  
#include <stdlib.h>  
#include <unistd.h>  
#include <sys/ipc.h>  
#include <sys/sem.h>  
  
#define SEM_KEY 1234  
#define NUM_SEMS 3  // 假设信号量集中有三个信号量  
#define TARGET_SEM 1  // 我们想要操作的目标信号量索引  
  
struct sembuf sops[1];  // 只需要一个sembuf结构  
  
int main() {  
    int semid;  
    unsigned short semvals[NUM_SEMS] = {1, 1, 1};  // 初始化所有信号量  
  
    // 创建信号量集  
    semid = semget(SEM_KEY, NUM_SEMS, 0666 | IPC_CREAT);  
    if (semid == -1) {  
        perror("semget");  
        exit(EXIT_FAILURE);  
    }  
  
    // 初始化信号量值  
    union semun arg;  
    arg.array = semvals;  
    if (semctl(semid, 0, SETALL, arg) == -1) {  
        perror("semctl");  
        exit(EXIT_FAILURE);  
    }  
  
    // 准备等待目标信号量的semop操作  
    sops[0].sem_num = TARGET_SEM;  
    sops[0].sem_op = -1;  // 等待目标信号量  
    sops[0].sem_flg = 0;  
  
    // 执行semop操作，等待目标信号量  
    if (semop(semid, sops, 1) == -1) {  
        perror("semop (wait)");  
        exit(EXIT_FAILURE);  
    }  
  
    // 模拟资源使用...  
    printf("正在使用由信号量 %d 保护的资源...\n", TARGET_SEM);  
    sleep(5);  // 假设操作需要5秒  
  
    // 准备释放目标信号量的semop操作  
    sops[0].sem_op = 1;  // 释放目标信号量  
  
    // 执行semop操作，释放目标信号量  
    if (semop(semid, sops, 1) == -1) {  
        perror("semop (release)");  
        exit(EXIT_FAILURE);  
    }  
  
    // 清理信号量集  
    if (semctl(semid, 0, IPC_RMID, 0) == -1) {  
        perror("semctl (remove)");  
        exit(EXIT_FAILURE);  
    }  
  
    return 0;  
}
```



#### posix---------------

###### sem_open

```c
SYNOPSIS
       #include <fcntl.h>           /* For O_* constants */
       #include <sys/stat.h>        /* For mode constants */
       #include <semaphore.h>
   sem_t *sem_open(const char *name, int oflag);//这个玩意儿只能打开
   sem_t *sem_open(const char *name, int oflag,
                   mode_t mode, unsigned int value);//可以创建
		name：你要打开的这个有名信号量的名字
			这个名字为路径名 -- 必须以 / 开头
				并且整个路径名里面只能有一个 / 
				因此就是在 /里面的某一个东西
				如：
					"/1.sem"
		oflag:标志
			0 		打开
			O_CREAT	创建
		mode：权限
			0666
			或者使用宏
		value：创建的时候你需要给这个信号量值
			只有创建的时候才有这个value
		成功返回一个代表这个sem的指针sem_t *
		失败返回SEM_FAILED，同时errno被设置
   Link with -pthread.  链接此库
       
```

###### sem_init

```c
SYNOPSIS
       #include <semaphore.h>
        int sem_init(sem_t *sem, int pshared, unsigned int value);
		sem：代表此sem的指针
		pshared：共享方式
			0 线程间的共享
			1 进程间共享
				sem就必须要指向共享内存
		value：初始化的时候这个sem的值
		成功返回0，失败返回-1，同时errno被设置
   Link with -pthread.链接此库
```

###### sem_wait

```c
SYNOPSIS
       #include <semaphore.h>
int sem_wait(sem_t *sem); //阻塞版本 上不了锁就卡死

   int sem_trywait(sem_t *sem);//非阻塞版本 上不了就报错

   int sem_timedwait(sem_t *sem, const struct timespec *abs_timeout);//超时等待
												//等到一个绝对的时间点   跟上面的操作一样
		sem：你要上锁的信号量
		abs_timeout:超时的时间点      
```

###### sem_post

```c
SYNOPSIS
       #include <semaphore.h>
   int sem_post(sem_t *sem);
		sem：你要解哪一个
   Link with -pthread.
   
```

###### sem_getvalue

```c
SYNOPSIS
       #include <semaphore.h>
       int sem_getvalue(sem_t *sem, int *sval);
		sem：你要获取谁的
		sval：保存这个值
		int value;
		sem_getvalue(sem,&value);
   Link with -pthread.   
```

###### sem_close

```c
SYNOPSIS
       #include <semaphore.h>
int sem_close(sem_t *sem);
      
```

###### sem_unlink

```c
从系统删除
sem_unlink - remove a named semaphore
   	int sem_unlink(const char *name);
```

# 线程------------

#### 创建一个新的线程

###### pthread_create

```c
SYNOPSIS
       #include <pthread.h>
   int pthread_create(pthread_t *thread, const pthread_attr_t *attr,
                      void *(*start_routine) (void *), void *arg);
        thread：保存开辟出来的线程id
            通过这个id可以找到开辟出来的线程
        attr：线程的属性 线程的属性有很多，一般我们采用默认属性
            后面如果需要设置什么属性的时候单独去设置即可
            填NULL表示采用默认属性
        start_routine：线程的任务函数  就是一个函数指针
            线程弄出来之后需要执行代码，程序给线程讲要做什么事情
            通过这个函数指针传入函数地址，线程开辟出来之后就会自动去执行这个函数
            就是线程开辟出来之后通过这个函数指针调用这个函数
                    这个函数的类型必须是void *的返回
                    void *的参数
                    void * functionname(void * arg)//这个就是线程函数的原型
                    {

                    }
        arg：参数，线程函数是有一个void*的参数的
            因此我们需要在这个参数这里讲上面的任务函数的参数传入
    返回值：
        成功返回0，失败返回-1，同时errno被设置
        成功开辟，线程就会运行起来，任务函数执行完毕，它也就死了
            如果是一个死循环就一直执行
        主线程死子线程必死
        子线程死主线程不受影响
        子线程里面调用exit，进程一样退出
   Compile and link with -pthread.       
```

#### 线程的退出

###### pthread_exit

```c
SYNOPSIS
       #include <pthread.h>
 void pthread_exit(void *retval);
        retval：线程退出的时候返回值
            这个玩意儿返回的时候返回的是一个指针，因此不能是栈的地址
            要返回不会释放的地址 --- 堆
   Compile and link with -pthread.       
```

#### 取消线程

###### pthread_cancel

```c
SYNOPSIS
       #include <pthread.h>
   int pthread_cancel(pthread_t thread);
    thread：你要干哪一个线程

   Compile and link with -pthread.
        线程能否被杀死取决于它的属性，默认是能被杀死了
        如果想这个线程被别人杀不死，我们可以设置属性       
```

###### pthread_setcancelstate

```c
SYNOPSIS
       #include <pthread.h>
int pthread_setcancelstate(int state, int *oldstate);
    state：属性
        PTHREAD_CANCEL_ENABLE   可以被取消
        PTHREAD_CANCEL_DISABLE 不可以被取消
    oldstate：用于保存线程原来的状态，如果不需要原来的状态，则填NULL    
```

###### pthread_setcanceltype 

```c
SYNOPSIS
   #include <pthread.h>
 
     功能: 设置线程的取消类型
int pthread_setcanceltype(int type, int *oldtype);
	type: 线程的取消类型，有以下两种：
		PTHREAD_CANCEL_DEFERRED		 延时到取消点才取消
        PTHREAD_CANCEL_ASYNCHRONOUS  随时可以取消
取消点:
	取消点，指由系统和标准库提供的一些函数，线程在执行到这些函数的时候，允许被取消
		具体的取消点，参考man手册: man  7  pthreads
	常见的取消点:
		open   read   write   close  
		fread ......
```

#### 等待线程结束

###### pthread_join

```c
SYNOPSIS
       #include <pthread.h>
        功能: 等待指定的线程结束，这种等待操作称之为连接(join),如果指定的线程已经结束，则立即返回，否则阻塞
   int pthread_join(pthread_t thread, void **retval); //这个函数会阻塞等待
        thread：你要等哪一个线程死
        retval：线程退出的时候有返回值的，返回值是一个指针
            因此这里就要用二级指针去保存那个地址
            如果不需要退出码可以填：NULL


```

#### 分离线程

###### pthread_detach

```c
 SYNOPSIS
       #include <pthread.h> 
  int pthread_detach(pthread_t thread);
        将thread这个线程分离
        如果想分离自己就获取自己的id
   Compile and link with -pthread.
        pthread_self - obtain ID of the calling thread
```

#### 获取线程ID

###### pthread_self

```c
SYNOPSIS
       #include <pthread.h>
   pthread_t pthread_self(void);返回自己这个线程的id       
```

## 线程同步

#### 互斥锁初始化

###### pthread_mutex_init 

```c
SYNOPSIS
       #include <pthread.h>

互斥锁是一个pthread_mutex_t类型的变量，在使用之前必须要初始化

1、静态初始化
pthread_mutex_t mutex = PTHREAD_MUTEX_INITIALIZER;

2、动态初始化
int pthread_mutex_init(pthread_mutex_t *restrict mutex, 
                       const pthread_mutexattr_t *restrict attr);
	mutex: 待初始化的互斥锁的地址
	attr: 待初始化互斥锁的属性  一般填NULL,使用默认属性即可

返回值:
	成功返回0
	失败返回错误码

说明:
	1、restrict是一个类型限定符，用于指针类型的声明，以指示编译器该指针指向的内存区域不会被其他指针所访问
	2、部分情况必须使用动态初始化
		a. 动态分配于堆空间的互斥锁
		b. 栈中分配的互斥锁
		c. 初始化但是不使用默认属性的互斥锁

当使用动态分配互斥锁时，需要调用 pthread_mutex_destroy() 函数进行销毁，静态初始化的互斥锁，不需要调用 pthread_mutex_destroy() 函数进行销毁
```

#### 销毁互斥锁

###### pthread_mutex_destroy

```c
SYNOPSIS
       #include <pthread.h>
	功能: 销毁一个互斥锁
int pthread_mutex_destroy(pthread_mutex_t *mutex);
	mutex: 需要进程销毁的互斥锁的地址
返回值:
	成功返回0
    失败返回错误码
```

####  上锁和解锁

###### pthread_mutex_lock,  pthread_mutex_trylock,  pthread_mutex_unlock 

```c
SYNOPSIS
       #include <pthread.h>

int pthread_mutex_lock(pthread_mutex_t *mutex);
	锁定互斥锁，如果该锁已经被其他线程锁定，则阻塞等待

int pthread_mutex_trylock(pthread_mutex_t *mutex);
	尝试锁定互斥锁，如果该锁已经被其他线程锁定，则立即返回

int pthread_mutex_unlock(pthread_mutex_t *mutex);
	解锁互斥锁
```

### 条件变量

条件变量的基本工作原理:

条件变量允许线程在某些条件未满足时挂起（阻塞），并在条件可能变为满足时被唤醒。这里的关键是，线程在调用`pthread_cond_wait`之前必须持有与条件变量相关联的互斥锁。`pthread_cond_wait`函数会原子地释放锁并阻塞当前线程，直到另一个线程通过`pthread_cond_signal`或`pthread_cond_broadcast`唤醒它。当线程被唤醒时，它会重新尝试获取之前释放的锁。



###### pthread_cond_init

```
#include <pthread.h>  
  
int pthread_cond_init(pthread_cond_t *cond, const pthread_condattr_t *attr);
参数
    cond：指向要初始化的条件变量的指针。
    attr：一个指向 pthread_condattr_t 类型的指针，用于指定条件变量的属性。如果传递 NULL，则使用默认属性。在大多数实现中，默认属性就足够了，因此这个参数通常被设置为 NULL。
    
返回值
    成功时，返回 0。
    出错时，返回一个错误码。
```

###### pthread_cond_wait

```c
#include <pthread.h>  
  
int pthread_cond_wait(pthread_cond_t *cond, pthread_mutex_t *mutex);
参数
	cond：指向条件变量的指针，线程将在此条件变量上等待。
	mutex：指向互斥锁的指针，该互斥锁在调用 pthread_cond_wait 之前应由调用线程锁定。
返回值
    成功时，返回 0。
    出错时，返回错误码。但是，由于 pthread_cond_wait 是在等待条件变量时阻塞的，所以它通常不会返回错误码，除非在调用它之前互斥锁已经损坏或以某种方式不正确。
    
    
1. 自动释放锁：在调用pthread_cond_wait时，线程会自动释放与条件变量相关联的互斥锁。这允许其他线程（包括可能修改条件变量所保护的数据的线程）访问这些资源。
    
2. 重新获取锁：当线程被pthread_cond_signal或pthread_cond_broadcast唤醒后，它会尝试重新获取之前释放的互斥锁。如果锁仍然被其他线程持有，则唤醒的线程将阻塞在锁上，直到锁变得可用。
```

###### pthread_cond_signal

```c
#include <pthread.h>  
  
int pthread_cond_signal(pthread_cond_t *cond);
参数
	cond：指向条件变量的指针，该条件变量用于同步线程。
返回值
    成功时，返回 0。
    出错时，返回错误码。
功能描述
pthread_cond_signal 函数用于唤醒在指定条件变量上等待的线程中的一个。如果有多个线程在等待这个条件变量，具体唤醒哪一个线程是由系统实现的，因此开发者不应该依赖于唤醒的特定线程。如果没有线程在等待指定的条件变量，pthread_cond_signal 调用将没有效果。
```

###### pthread_cond_broadcast

```c

#include <pthread.h>  
  
int pthread_cond_broadcast(pthread_cond_t *cond);
参数
	cond：指向条件变量的指针，该条件变量用于同步线程。
返回值
    成功时，返回 0。
    出错时，返回错误码。
功能描述
pthread_cond_broadcast 函数用于唤醒所有在指定条件变量上等待的线程。如果有多个线程在等待这个条件变量，那么所有这些线程都将被唤醒。但是，正如 pthread_cond_signal 一样，被唤醒的线程仍然需要重新获取与条件变量相关联的互斥锁（mutex），并在继续执行之前重新检查条件是否满足。
```

###### pthread_cond_destroy

```
#include <pthread.h>  
  
int pthread_cond_destroy(pthread_cond_t *cond);

参数
    cond：指向要销毁的条件变量的指针。
返回值
    成功时，返回 0。
    出错时，返回一个错误码。
注意事项
在调用 pthread_cond_destroy 之前，必须确保没有线程正在等待该条件变量。如果有线程正在等待，则销毁条件变量将导致未定义行为。

通常，在销毁条件变量之前，会先销毁与之相关联的互斥锁（如果有的话），但这并不是必需的，因为互斥锁和条件变量在 POSIX 线程库中是被独立管理的。然而，从逻辑上讲，如果条件变量和互斥锁一起用于同步对共享数据的访问，那么它们应该在同一时间点上被销毁。

销毁条件变量后，该指针就变为未定义状态，不应再被访问或用作其他条件变量的指针。如果需要再次使用条件变量，必须先通过 pthread_cond_init 重新初始化它。

```



### 补充

###### realloc

```c
#include <stdlib.h> 
void* realloc(void* ptr, size_t size);
/*
ptr 是指向之前分配的内存块的指针。如果 ptr 是 NULL，则 realloc 的行为与 malloc 相同。
size 是新的内存块的大小（以字节为单位）。
函数返回指向新内存块的指针（可能与原始指针相同，也可能不同），如果分配失败则返回 NULL。
*/
```

###### calloc

```c
#include <stdlib.h> 
void* calloc(size_t num, size_t size);
/*
num 是要分配的元素数量。
size 是每个元素的大小（以字节为单位）。
函数返回指向分配的内存的指针，如果分配失败则返回 NULL。
*/
```



##### 冒泡排序

```c
void bubbleSort(int arr[], int n) {  
    int i, j, temp;  
    for (i = 0; i < n-1; i++) {  
        // 标志位，用于优化，如果在某一趟排序中没有发生交换，说明数组已经有序  
        int swapped = 0;  
        for (j = 0; j < n-i-1; j++) {  
            if (arr[j] > arr[j+1]) {  
                // 交换arr[j]和arr[j+1]  
                temp = arr[j];  
                arr[j] = arr[j+1];  
                arr[j+1] = temp;  
                swapped = 1;  
            }  
        }  
        // 如果没有发生交换，说明数组已经有序，提前退出  
        if (swapped == 0) {  
            break;  
        }  
    }  
}  
```

##### extern 和extern“C“关键字

[C/C++中的 extern 和extern“C“关键字的理解和使用（对比两者的异同）_c extern c-CSDN博客](https://blog.csdn.net/m0_46606290/article/details/119973574)

