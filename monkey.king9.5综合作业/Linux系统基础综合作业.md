## Linux系统基础综合作业

## 一、理论依据

### 1.mymi

##### 函数 print_it

打开指定的文件（`/proc/meminfo`）。

如果打开失败，使用 `perror` 打印错误信息并返回。

使用 `fgets` 逐行读取文件内容到 `buffer` 中。

使用 `strncmp` 检查当前行是否以指定的 `key` 开头。

如果匹配，使用 `strtoll` 将关键字后面的数字转换为长整型（`long long`）。

打印该关键字及其对应的值（以 kB 为单位）。

##### 主函数

调用 print_it函数多次，每次传入不同的内存信息关键字，这些关键字对应于 /proc/meminfo文件中的行。



### 2.myls

#####  函数print_permissions

根据文件的权限位打印相应的字符（`r`、`w`、`x` 或 `-`）

![微信截图_20240905145037](%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20240905145037.png)

#####  函数print_og

 使用 `getpwuid` 和 `getgrgid` 获取用户和组的名称。 

#####  主函数

检查命令行参数，确保提供了一个目录路径。

如果参数不正确，打印用法信息并返回。

使用 `opendir` 打开指定目录。

读取目录中的每个条目，构建完整的文件路径。

使用 `lstat` 获取文件的状态信息。如果失败，打印错误信息并继续下一个条目。

打印文件的 inode 号。

根据文件类型打印相应的字符（块设备、字符设备、目录、符号链接、常规文件等）。

调用 `print_permissions` 打印文件权限。

调用 `print_og` 打印文件的所有者和组。



### 3.mycp

##### 主函数

定义一个字符数组 buff用于存储读取的数据。

fd和 ab 是文件描述符，初始化为 99 和 98。

使用 open 函数打开第一个命令行参数指定的文件（以只读模式）。

使用 open 函数打开第二个命令行参数指定的文件（以只写模式）。如果文件不存在，它会创建一个新文件。

![微信截图_20240905144836](%E5%BE%AE%E4%BF%A1%E6%88%AA%E5%9B%BE_20240905144836.png)

使用 read函数逐字节读取源文件的内容，直到没有更多数据（read 返回 0）。

使用 write函数将读取的字节写入目标文件。



### 4.mystat

##### 主函数

定义一个 struct stat类型的变量 pool，用于存储文件状态信息。

fd和 ab 是文件描述符，初始化为 99 和 98。

使用 open 函数打开第一个命令行参数指定的文件。 

使用 fstat函数获取文件的状态信息，并将其存储在 pool 变量中。

使用位掩码检查文件类型。0170000是文件类型的掩码，0040000`表示目录类型。如果文件是目录，则打印“this is dir”。



### 5.mytools

##### 主函数

无限循环，提示用户输入命令。

ls:使用 `fork` 创建子进程。在子进程中使用 `execl` 执行 `myls` 程序，传递参数。父进程等待子进程结束。

stat:类似于 `ls` 命令，执行 `mystat` 程序。

cp:执行 `mycp` 程序，传递两个参数。

mymi:执行 `mymi` 程序，注意这里的 `"args[0]"` 是字符串，而不是变量。

退出：如果输入 `shut`，循环退出。





## 二、进程前驱图

![微信图片_20240905155800](%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20240905155800.jpg)

## 三、实现和验证过程

#### 1.源代码

##### mymi

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void print_it(const char* filename, const char* key);
int main()
 {
    print_it("/proc/meminfo", "MemTotal:");
    print_it("/proc/meminfo", "MemFree:");
    print_it("/proc/meminfo", "MemAvailable:");
    print_it("/proc/meminfo", "Buffers:");
    print_it("/proc/meminfo", "Cached:");
    print_it("/proc/meminfo", "SwapCached:");
    print_it("/proc/meminfo", "SwapTotal:");
    print_it("/proc/meminfo", "SwapFree:");

    return 0;
 }
void print_it(const char* filename, const char* key) {
    FILE *fp;
    char buffer[1024];
    long long value = 0;
    fp = fopen(filename, "r");
    if (fp == NULL) {
      perror("Error opening file");
      return;
}
    while (fgets(buffer, sizeof(buffer), fp) != NULL) {
    if (strncmp(buffer, key, strlen(key)) == 0) {
        char *endptr;
        value = strtoll(buffer + strlen(key), &endptr, 10);
        while (*endptr && !isspace((unsigned char)*endptr)) endptr++;
        printf("%s \t\t%lld kB\n", key, value);
        break;
    }
}

fclose(fp);
}
```

##### myls

```c
#include <stdio.h>
#include <stdlib.h>
#include <dirent.h>
#include <sys/stat.h>
#include <string.h>
#include <unistd.h>
#include <pwd.h>
#include <grp.h>
// 函数声明
void print_permissions(mode_t mode);
void print_og(uid_t uid, gid_t gid);

int main(int argc, char *argv[]) {
    DIR *d;
    struct dirent *dir;
    struct stat file_stat;

    if (argc != 2) {
        fprintf(stderr, "Usage: %s <directory>\n", argv[0]);
        return 1;
    }
 
    d = opendir(argv[1]);
    if (d) {
        while ((dir = readdir(d)) != NULL) {
            char path[1024];
            snprintf(path, sizeof(path), "%s/%s", argv[1], dir->d_name);
 
            if (lstat(path, &file_stat) == -1) {
                perror("lstat");
                continue;
            }
 
            printf("%ld ", (long)file_stat.st_ino);
 
            switch (file_stat.st_mode & S_IFMT) {
                case S_IFBLK:  printf(" b "); break;
                case S_IFCHR:  printf(" c "); break;
                case S_IFDIR:  printf(" d "); break;
                case S_IFLNK:  printf(" l "); break;
                case S_IFREG:  printf(" - "); break;
                default:       printf(" ? "); break;
            }

            // 权限
            print_permissions(file_stat.st_mode);
     printf("%3ld ", (long)file_stat.st_nlink); // 硬链接数

            // UID, GID
            print_og(file_stat.st_uid, file_stat.st_gid);

            // 文件大小
            printf("%lld \t", (long long)file_stat.st_size);

            // 文件名
            printf("%s\n", dir->d_name);
        }
        closedir(d);
    } else {
        perror("Failed to open directory");
        return 1;
    }

    return 0;

}

// 打印文件权限
void print_permissions(mode_t mode) {
    printf((mode & S_IRUSR) ? "r" : "-");
    printf((mode & S_IWUSR) ? "w" : "-");
    // 对于目录，可执行意味着可以进入
    printf((mode & S_IXUSR) || (mode & S_IFDIR) ? "x" : "-");
    printf((mode & S_IRGRP) ? "r" : "-");
    printf((mode & S_IWGRP) ? "w" : "-");
    printf((mode & S_IXGRP) ? "x" : "-");
    printf((mode & S_IROTH) ? "r" : "-");
    printf((mode & S_IWOTH) ? "w" : "-");
    printf((mode & S_IXOTH) ? "x" : "-");
}

// 打印文件的所有者和组
void print_og(uid_t uid, gid_t gid) {
    struct passwd *pw = getpwuid(uid);
    struct group *gr = getgrgid(gid);

    if (pw) {
        printf("%s \t", pw->pw_name);
    } else {
        printf("%ld \t", (long)uid);
    }

    if (gr) {
        printf("%s \t", gr->gr_name);
    } else {
        printf("%ld \t", (long)gid);
    }

}
```

##### mycp

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdio.h>
#include <unistd.h>
int main(int argc,char* argv[])
{       char buff[64];
        int fd=99,ab=98;
        char c;
        fd=open(argv[1],O_RDONLY);
        ab=open(argv[2],O_WRONLY);
        while((read(fd,buff,1))>0)
        {
        write(ab,buff,1);
        }
        printf("all copy\n");
        close(fd);
        close(ab);
        return 0;
}     
```

##### mystat

```c
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <stdio.h>
#include <unistd.h>
int main(int argc,char* argv[])
{       struct stat pool;
        int fd=99,ab=98;
        char c;
        fd=open(argv[1],O_RDONLY|O_WRONLY);
        fstat(fd,&pool);
        printf("uid:%d\n",pool.st_uid);
        printf("gid:%d\n",pool.st_gid);
        printf("mod:%d\n",pool.st_mode);
        printf("size:%d\n",pool.st_size);
        if((pool.st_mode & 0170000) == 0040000)
        {printf("this is dir");}

        close(fd);
        return 0;
}       
```

##### mytools

```c
#include<sys/wait.h>
#include<unistd.h>
#include <stdio.h>
#include<stdlib.h>
#include<string.h>
//char* getline()
//{
//int buffersize=256;
//char*buffer=malloc(buffersize);
//int len=strlen(buffer);
//if(len>0&&buffer[len-1]=='\n'){buffer[len-1]='\0';}
//return buffer;
//}
int main()
{char input[1024];
char* args[10];
int arg_count;
//char* getline();
        pid_t pid,cpid;
        int status =0;
        while(1)
        {
                printf("intools ");
                if(fgets(input,1024,stdin)!=NULL){
                input[strcspn(input,"\n")]=0;
                char *token =strtok(input," ");
                arg_count=0;
                while(token !=NULL&&arg_count<10)
        {args[arg_count++]=token;
        token=strtok(NULL," ");
        }
        }
        if(strcmp(args[0],"ls")==0)
                {
                pid=fork();
                if(pid==0)
                {execl("myls","myls",args[1],0);
                _exit(99);}
                else {wait(NULL);}
                }
        else if(strcmp(args[0],"stat")==0)
        {
        pid=fork();
                if(pid==0)
                {execl("./mystat","./mystat",args[1],0);
                _exit(1);
 
                }
                else{
                wait(NULL);
                }
        }
        else if(strcmp(args[0],"cp")==0)
        {       pid=fork();
                if(pid==0)
                {execl("./mycp","./mycp",args[1],args[2],0);
                _exit(99);
                }
                else {wait(NULL);}
        }
        else if(strcmp(args[0],"mymi")==0)
        {
        pid=fork();
        if(pid==0)
                {execl("./mymi","./mymi","args[0]",0);
                _exit(99);
                }
        else {wait(NULL);}
        }
        else if(strcmp(args[0],"shut")==0){return 0;}
}
        printf("end\n");
        return 0;
}

```

#### 2.验证

（1）使用工具箱

运行工具箱执行命令ls查看work目录下的目录信息

![微信图片_20240905155031](%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20240905155031.png)

使用工具箱内的cp命令复制test.txt测试文件到mytest.txt，并且使用工具箱中stat命令查看mytest.txt文件属性

![微信图片_20240905155041](%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20240905155041.png)

使用工具箱查看内存信息

![微信图片_20240905155050](%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20240905155050.png)



（2）单独使用mytools工具命令与原系统命令进行对比

查看目录列表的myls

![微信图片_20240905155058](%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20240905155058.png)

查看文件状态的mystat

![微信图片_20240905163918](%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20240905163918.png)

查看当前系统内存信息mymi

原系统命令

![微信图片_20240905163922](%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20240905163922.png)

mymi

![微信图片_20240905163926](%E5%BE%AE%E4%BF%A1%E5%9B%BE%E7%89%87_20240905163926.png)

## 四、结论总结

mytools用于调用下面四个程序，mymi 用于从 `/proc/meminfo` 文件中读取和打印内存相关的信息 ，myls 用于列出指定目录中的文件信息，包括文件权限、硬链接数、所有者和组、文件大小等，mycp 用于将一个文件的内容复制到另一个文件 ，mystat 用于获取并打印指定文件的状态信息，包括用户 ID、组 ID、权限模式和文件大小。 

## 五、小组组内评分

| 姓名   | 具体工作             | 评分 |
| ------ | -------------------- | ---- |
| 杨泽宁 | 编写主要代码         | 106  |
| 陈嘉兴 | 验证、修改、整合代码 | 99   |
| 姜欢   | 绘制前驱图与编辑文档 | 99   |
| 郝鑫龙 | 编写与完善代码       | 99   |
| 王志华 | 编写与完善代码       | 99   |
| 魏子越 | 编辑与总结文档       | 99   |
| 王欣   | 查找资料与整合文档   | 99   |

