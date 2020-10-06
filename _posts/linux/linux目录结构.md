---

title: Linux目录结构

date: 2020-08-07 11:05:26

tags: [Linux Filesystem 文件系统]

categories: [Linux学习历程]

---

##### Filesystem Hierarchy Standard


- /bin

Essential command binaries that need to be available in single user mode; for all users, e.g., cat, ls, cp.

- /sbin

Essential system binaries, e.g., fsck, init, route.

- /lib

Libraries essential for the binaries in /bin and /sbin.

- /dev

Device files, e.g., /dev/null, /dev/disk0, /dev/sda1, /dev/tty, /dev/random.

- /srv

Site-specific data served by this system, such as data and scripts for web servers, data offered by FTP servers, and repositories for version control systems.

- /boot

Boot loader files, e.g., kernels, initrd.

- /sys

Contains information about devices, drivers, and some kernel features.

- /usr

/usr nowadays stands for User System Resources. This directory contains **most commands and executables files, libraries and documentation**. In the early days of Unix, it was the directory where the users' home directories were placed (your user home directory would have been /usr/anatoly which now is /home/anatoly), so originally it stood for User.

/usr directory **should be read-only and not modifiable by programs**, which applies to /usr/lib because it contains shared libraries which are critical for proper functionality of applications. However, some content such as /usr/share/applications directory contains resources that are **not system critical, and can be modified by admin-level user when necessary**.

- /root

Home directory for the root user.

- /etc

`Editable Text Configuration" or "Extended Tool Chest`

系统和程序一般都可以通过修改相应的配置文件 , 来进行配置 ;例如 , 要配置系统开机的时候启动那些程序 , 配置某个程序启动的时候显示什么样的风格等等 ;通常这些配置文件都集中存放在 /etc 目录中 , 所以想要配置什么东西的话 , 可以在 /etc 下面寻找我们可能需要修改的文件 ;一些大型套件 , 如 X11 , 在 /etc 下它们自己的子目录 ;系统配置文件可以放在这里或在 /usr/etc ; 不过所有程序总是在 /etc 目录下查找所需的配置文件 , 你也可以将这些文件链接到目录 /usr/etc ;另外 , 还一个需要注意的常见现象就是 , 当某个程序在某个用户下运行的时候 , 可能会在该用户的家目录中生成一个配置文件(一般这个文件最开始就是 /etc 下相应配置文件的拷贝 , 存放相应于"当前用户"的配置) , 这样当前用户可以通过配置这个家目录的配置文件 , 来改变程序的行为 , 并且这个行为只是该用户特有的 ;原因就是 : 一般来说一个程序启动 , 如果需要读取一些配置文件的话 , 它会首先读取当前用户家目录的配置文件 , 如果存在就使用；如果不存在它就到 /etc 下读取全局的配置文件进而启动程序 ;就是这个配置文件不自动生成 , 我们手动在自己的家目录中创建一个文件的话 , 也有许多程序会首先读取到这个家目录的文件并且以它的配置作为启动的选项(例如我们可以在家目录中创建 vim 程序的配置文件 .vimrc , 来配置自己的 vim 程序)

- /opt

Optional application software packages.

- /mnt
	

Temporarily mounted filesystems.

- /run

Run-time variable data: Information about the running system since last boot, e.g., currently logged-in users and running daemons. Files under this directory **must be either removed or truncated at the beginning of the boot process**.

- /var

Variable files: files whose content is expected to **continually change during normal operation of the system**, such as logs, spool files, and temporary e-mail files.

- /tmp

Like /var/tmp

Temporary files to be preserved between reboots.

- /proc

Virtual filesystem **providing process and kernel information as files.** In Linux, corresponds to a procfs mount. Generally, automatically generated and populated by the system, on the fly.

这个目录多说两句，着重看一下进程相关信息：

1. 存储的是当前内核运行状态的一系列特殊文件，它只存在内存当中，而不占用外存空间。它以文件系统的方式为访问系统内核数据的操作提供接口。用户可以通过这些文件查看有关系统硬件及当前正在运行进程的信息，甚至可以通过更改其中某些文件来改变内核的运行状态。
2. 文件本身的大小却会显示为0字节。
3. 以数字命名的目录，它们是进程目录。系统中当前运行的每一个进程都有对应的一个目录在/proc下，以进程的 PID号为目录名。
	- 3044699是进程号；
   - `less /proc/3044699/cmdline`，查看整个用来产生进程的命令行。这个文件的内容是命令行参数包括传递来启动进程的所有参数。所有包含在这个文件的信息即命令和各个启动参数；
   - `ll /proc/3044699/cwd`，进程的当前工作目录；
   - `less /proc/3044699/environ`，包括在VARIABL=value为这个进程定义的所有的环境变量。正如"cmdline"一样，包含在文件中的命令和各个参数的信息没有任何的格式和空格；
   - `ll /proc/3044699/exe`，启动当前进程的可执行文件；
   - `ll /proc/3044699/fd`，进程打开的文件描述符，如果一个进程打开的文件描述符过多，会造成打开文件失败，通过检查这个目录可以查找打开文件失败的原因；
   - `less /proc/3044699/status`，进程名的信息，它的当前的状态，睡眠或者清醒，它的PID，UID，PPID和大量其它基本信息。
4. `less /proc/cpuinfo`，查看cpu信息；
5. `less /proc/crypto`，系统上已安装的内核使用的密码算法及每个算法的详细信息列表；
6. `less /proc/meminfo`，查看内存信息；
7. `less /proc/uptime`，查看启动以来的运行时间，单位是秒；
8. `less /proc/version`，查看系统运行的内核版本信息；

---

##### 参考

- [Filesystem Hierarchy Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard)
- [Linux 目录结构](https://mp.weixin.qq.com/s/w4YA4MzLqm3R-_mr2S4vNQ)