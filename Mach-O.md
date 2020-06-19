## Mach-O格式

![mach-o](https://www.desgard.com/iOS-Source-Probe/image/15058343519881/mach-o.png)

Mach-O头Header

```c
#define    MH_MAGIC    0xfeedface    /* the mach magic number */
#define MH_CIGAM    0xcefaedfe    /* NXSwapInt(MH_MAGIC) */

struct mach_header_64 {
    uint32_t    magic;        /* mach magic 标识符 */
    cpu_type_t    cputype;    /* CPU 类型标识符，同通用二进制格式中的定义 */
    cpu_subtype_t    cpusubtype;    /* CPU 子类型标识符，同通用二级制格式中的定义 */
    uint32_t    filetype;    /* 文件类型 */
    uint32_t    ncmds;        /* 加载器中加载命令的条数 */
    uint32_t    sizeofcmds;    /* 加载器中加载命令的总大小 */
    uint32_t    flags;        /* dyld 的标志 */
    uint32_t    reserved;    /* 64 位的保留字段 */
};
```



Mach-O Data

```c
#define    SEG_PAGEZERO    "__PAGEZERO" /* 当时 MH_EXECUTE 文件时，捕获到空指针 */
#define    SEG_TEXT    "__TEXT" /* 代码/只读数据段 */
#define    SEG_DATA    "__DATA" /* 数据段 */
#define    SEG_OBJC    "__OBJC" /* Objective-C runtime 段 */
#define    SEG_LINKEDIT    "__LINKEDIT" /* 包含需要被动态链接器使用的符号和其他表，包括符号表、字符串表等 */
```



Segment数据结构

```c
struct segment_command_64 { 
    uint32_t    cmd;        /* LC_SEGMENT_64 */
    uint32_t    cmdsize;    /* section_64 结构体所需要的空间 */
    char        segname[16];    /* segment 名字，上述宏中的定义 */
    uint64_t    vmaddr;        /* 所描述段的虚拟内存地址 */
    uint64_t    vmsize;        /* 为当前段分配的虚拟内存大小 */
    uint64_t    fileoff;    /* 当前段在文件中的偏移量 */
    uint64_t    filesize;    /* 当前段在文件中占用的字节 */
    vm_prot_t    maxprot;    /* 段所在页所需要的最高内存保护，用八进制表示 */
    vm_prot_t    initprot;    /* 段所在页原始内存保护 */
    uint32_t    nsects;        /* 段中 Section 数量 */
    uint32_t    flags;        /* 标识符 */
};
```

部分的 Segment （主要指的 `__TEXT` 和 `__DATA`）可以进一步分解为 Section。

一些常见的section：

| Section                   | 用途                                                         |
| ------------------------- | ------------------------------------------------------------ |
| `__TEXT.__text`           | 主程序代码                                                   |
| `__TEXT.__cstring`        | C 语言字符串                                                 |
| `__TEXT.__const`          | `const` 关键字修饰的常量                                     |
| `__TEXT.__stubs`          | 用于 Stub 的占位代码，很多地方称之为*桩代码*。               |
| `__TEXT.__stubs_helper`   | 当 Stub 无法找到真正的符号地址后的最终指向                   |
| `__TEXT.__objc_methname`  | Objective-C 方法名称                                         |
| `__TEXT.__objc_methtype`  | Objective-C 方法类型                                         |
| `__TEXT.__objc_classname` | Objective-C 类名称                                           |
| `__DATA.__data`           | 初始化过的可变数据                                           |
| `__DATA.__la_symbol_ptr`  | lazy binding 的指针表，表中的指针一开始都指向 `__stub_helper` |
| `__DATA.nl_symbol_ptr`    | 非 lazy binding 的指针表，每个表项中的指针都指向一个在装载过程中，被动态链机器搜索完成的符号 |
| `__DATA.__const`          | 没有初始化过的常量                                           |
| `__DATA.__cfstring`       | 程序中使用的 Core Foundation 字符串（`CFStringRefs`）        |
| `__DATA.__bss`            | BSS，存放为初始化的全局变量，即常说的静态内存分配            |
| `__DATA.__common`         | 没有初始化过的符号声明                                       |
| `__DATA.__objc_classlist` | Objective-C 类列表                                           |
| `__DATA.__objc_protolist` | Objective-C 原型                                             |
| `__DATA.__objc_imginfo`   | Objective-C 镜像信息                                         |
| `__DATA.__objc_selfrefs`  | Objective-C `self` 引用                                      |
| `__DATA.__objc_protorefs` | Objective-C 原型引用                                         |
| `__DATA.__objc_superrefs` | Objective-C 超类引用                                         |