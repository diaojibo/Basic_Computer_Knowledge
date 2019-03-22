## linux c++ 读写目录文件
在Unix/Linux系统中，要获取一个指定目录下所有的文件或文件夹，一般用dirent.h（**POSIX标准定义的目录操作头文件**）。

### DIR 
DIR是一个结构体，代表一个目录流的结构


``` c++
struct __dirstream
{
    void *__fd;              /* 'struct hurd_fd' pointer for descriptor.*/
    char *__data;            /* Directory block.  */
    int __entry_data;        /* Entry number `__data' corresponds to.*/
    char *__ptr;             /* Current pointer into the block.*/
    int __entry_ptr;         /* Entry number `__ptr' corresponds to.*/
    size_t __allocation;          /* Space allocated for the block.*/
    size_t __size;                /* Total valid data in the block.*/
    __libc_lock_define (, __lock) /* Mutex lock for this structure.*/
};

typedef struct __dirstream DIR;
```

### struct dirent
包含一个文件或目录信息的结构体。

``` c++
struct dirent
{
    long d_ino;                 /* inode number 索引节点号 */
    off_t d_off;                /* offset to this dirent 在目录文件中的偏移 */
    unsigned short d_reclen;    /* length of this d_name 文件名长 */
    unsigned char d_type;       /* the type of d_name 文件类型 */    
    char d_name [NAME_MAX+1];   /* file name 文件名，最长255字符 */
}
```

### 函数原型

``` c++
DIR* opendir(const char* dirname);
/* 打开一个目录：
        成功 - 返回指向DIR类型对象的指针。
        失败 - 返回NULL    */

int closedir(DIR *dirp);
/* 关闭目录流：
        成功 - 返回0
        失败 - 返回-1    */

struct dirent *readdir(DIR *dirp);
/* 读取目录流：
        成功 - 返回指向struct dirent对象的指针。
        失败 - 返回NULL（出错或流末尾）  */

int readdir_r(DIR *dirp, struct dirent *entry, struct dirent **result);
/* 读取目录流：用 dirp 当前位置的目录初始化entry，并让 result 指向 entry。
        成功 - 返回0
        失败 - 返回error number   */

void rewinddir(DIR *dirp);
/* 重置目录流的位置到开头 */

void seekdir(DIR *dirp, long int loc);
/* 设置目录流的位置，设置以后readdir()会读取到loc位置的目录。 */

long int telldir(DIR *dirp);
/* 返回目录流的当前位置 */
```

C++ 示例代码，输出指定目录下的所有文件或者目录名：

``` c++
#include<iostream>
#include<string>
#include<dirent.h>
using namespace std;

int main()
{
	string dirname;
	DIR *dp;
	struct dirent *dirp;

	cout << "Please input a directory: ";
	cin >> dirname;
	if((dp = opendir(dirname.c_str())) == NULL)
		cout << "Can't open " << dirname << endl;

	while((dirp = readdir(dp)) != NULL)
		cout << dirp->d_name << endl;

	closedir(dp);
	return 0;
}
```


有些情况下，我们只要输出文件而不需要文件夹（目录），这时可以通过dirent结构体中的d_type进行过滤。d_type表示类型，4表示目录，8表示普通文件，0表示未知设备。

``` c++
while((dirp = readdir(dp)) != NULL)
	if(dirp->d_type == 8)  // 只输出文件名，不输出目录名
		cout << dirp->d_name << endl;
```