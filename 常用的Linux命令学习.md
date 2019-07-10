
### 文件系统中跳转

不像 Windows ，每个存储设备都有一个独自的文件系统。类 Unix 操作系统， 比如 Linux，总是只有一个单一的文件系统树，不管有多少个磁盘或者存储设备连接到计算机上。 根据负责维护系统安全的系统管理员的兴致，存储设备连接到（或着更精确些，是挂载到）目录树的各个节点上。  

+ pwd - Print name of current working directory

+ cd - Change directory

+ ls - List directory contents


1. 以 “.” 字符开头的文件名是隐藏文件。这仅表示，ls 命令不能列出它们， 用 ls -a 命令就可以了。当你创建帐号后，几个配置帐号的隐藏文件被放置在 你的家目录下。稍后，我们会仔细研究一些隐藏文件，来定制你的系统环境。 另外，一些应用程序也会把它们的配置文件以隐藏文件的形式放在你的家目录下面。

2. 文件名和命令名是大小写敏感的。文件名 “File1” 和 “file1” 是指两个不同的文件名。

3. Linux 没有“文件扩展名”的概念，不像其它一些系统。可以用你喜欢的任何名字 来给文件起名。文件内容或用途由其它方法来决定。虽然类 Unix 的操作系统， 不用文件扩展名来决定文件的内容或用途，但是有些应用程序会。

4. 虽然 Linux 支持长文件名，文件名可能包含空格，标点符号，但标点符号仅限 使用 “.”，“－”，下划线。最重要的是，不要在文件名中使用空格。如果你想表示词与 词间的空格，用下划线字符来代替。过些时候，你会感激自己这样做。


### 探究操作系统

+ ls – List directory contents

+ file – Determine file type
    随着探究操作系统的进行，知道文件包含的内容是很有用的。我们将用 file 命令来确定文件的类型。我们之前讨论过， 在 Linux 系统中，并不要求文件名来反映文件的内容。然而，一个类似 “picture.jpg” 的文件名，我们会期望它包含 JPEG 压缩图像，但 Linux 却不这样要求它。  
    当调用 file 命令后，file 命令会打印出文件内容的简单描述。  
+ less – View file contents
    less 就是 more。  

| 目录 | 解释 |
| --- | ------ |
| / | 根目录，万物起源。|
| /bin  | 包含系统启动和运行所必须的二进制程序。|
| /boot | 包含 Linux 内核、初始 RAM 磁盘映像（用于启动时所需的驱动）和 启动加载程序。有趣的文件：/boot/grub/grub.conf or menu.lst，被用来配置启动加载程序。/boot/vmlinuz，Linux 内核。|
| /dev 	| 这是一个包含设备结点的特殊目录。“一切都是文件”，也适用于设备。 在这个目录里，内核维护着所有设备的列表。|
| /etc 	| 这个目录包含所有系统层面的配置文件。它也包含一系列的 shell 脚本， 在系统启动时，这些脚本会开启每个系统服务。这个目录中的任何文件应该是可读的文本文件。有趣的文件：虽然/etc 目录中的任何文件都有趣，但这里只列出了一些我一直喜欢的文件：/etc/crontab， 定义自动运行的任务。/etc/fstab，包含存储设备的列表，以及与他们相关的挂载点。/etc/passwd，包含用户帐号列表。|
| /home | 在通常的配置环境下，系统会在/home 下，给每个用户分配一个目录。普通用户只能在自己的目录下写文件。这个限制保护系统免受错误的用户活动破坏。|
| /lib 	| 包含核心系统程序所使用的共享库文件。这些文件与 Windows 中的动态链接库相似。|
| /lost+found | 每个使用 Linux 文件系统的格式化分区或设备，例如 ext3文件系统， 都会有这个目录。当部分恢复一个损坏的文件系统时，会用到这个目录。这个目录应该是空的，除非文件系统 真正的损坏了。|
| /media | 在现在的 Linux 系统中，/media 目录会包含可移动介质的挂载点， 例如 USB 驱动器，CD-ROMs 等等。这些介质连接到计算机之后，会自动地挂载到这个目录结点下。|
| /mnt 	| 在早些的 Linux 系统中，/mnt 目录包含可移动介质的挂载点。|
| /opt 	| 这个/opt 目录被用来安装“可选的”软件。这个主要用来存储可能 安装在系统中的商业软件产品。|
| /proc | 这个/proc 目录很特殊。从存储在硬盘上的文件的意义上说，它不是真正的文件系统。 相反，它是一个由 Linux 内核维护的虚拟文件系统。它所包含的文件是内核的窥视孔。这些文件是可读的， 它们会告诉你内核是怎样监管计算机的。|
| /root | root 帐户的家目录。|
| /sbin | 这个目录包含“系统”二进制文件。它们是完成重大系统任务的程序，通常为超级用户保留。|
| /tmp 	| 这个/tmp 目录，是用来存储由各种程序创建的临时文件的地方。一些配置导致系统每次 重新启动时，都会清空这个目录。|
| /usr 	| 在 Linux 系统中，/usr 目录可能是最大的一个。它包含普通用户所需要的所有程序和文件。|
| /usr/bin | /usr/bin 目录包含系统安装的可执行程序。通常，这个目录会包含许多程序。|
| /usr/lib 	包含由/usr/bin 目录中的程序所用的共享库。|
| /usr/local | 这个/usr/local 目录，是非系统发行版自带程序的安装目录。 通常，由源码编译的程序会安装在/usr/local/bin 目录下。新安装的 Linux 系统中会存在这个目录， 并且在管理员安装程序之前，这个目录是空的。|
| /usr/sbin | 包含许多系统管理程序。|
| /usr/share | /usr/share 目录包含许多由/usr/bin 目录中的程序使用的共享数据。 其中包括像默认的配置文件、图标、桌面背景、音频文件等等。|
| /usr/share/doc | 大多数安装在系统中的软件包会包含一些文档。在/usr/share/doc 目录下， 我们可以找到按照软件包分类的文档。|
| /var 	| 除了/tmp 和/home 目录之外，相对来说，目前我们看到的目录是静态的，这是说， 它们的内容不会改变。/var 目录存放的是动态文件。各种数据库，假脱机文件， 用户邮件等等，都位于在这里。|
| /var/log 	| 这个/var/log 目录包含日志文件、各种系统活动的记录。这些文件非常重要，并且 应该时时监测它们。其中最重要的一个文件是/var/log/messages。注意，为了系统安全，在一些系统中， 你必须是超级用户才能查看这些日志文件。|


### 操作文件和目录

+ cp – Copy files and directories

+ mv – Move/rename files and directories

+ mkdir – Create directories

+ rm – Remove files and directories

+ ln – Create hard and symbolic links
硬链接和软连接的不同

### 使用命令

what is command


1. 是一个可执行程序，就像我们所看到的位于目录/usr/bin 中的文件一样。 这一类程序可以是用诸如 C 和 C++语言写成的程序编译的二进制文件, 也可以是由诸如shell，perl，python，ruby等等脚本语言写成的程序 。

2. 是一个内建于 shell 自身的命令。bash 支持若干命令，内部叫做 shell 内部命令 (builtins)。例如，cd 命令，就是一个 shell 内部命令。

3. 是一个 shell 函数。这些是小规模的 shell 脚本，它们混合到环境变量中。 在后续的章节里，我们将讨论配置环境变量以及书写 shell 函数。但是现在， 仅仅意识到它们的存在就可以了。

4. 是一个命令别名。我们可以定义自己的命令，建立在其它命令之上。


+ type – Indicate how a command name is interpreted
说明怎样解释一个命令名

+ which – Display which executable program will be executed
显示会执行哪个可执行程序

+ man – Display a command’s manual page
显示命令手册页

+ apropos – Display a list of appropriate commands
显示一系列适合的命令

+ info – Display a command’s info entry
显示命令 info

+ whatis – Display a very brief description of a command
显示一个命令的简洁描述

+ alias – Create an alias for a command
创建命令别名


### 重定向

"I/O"代表输入/输出， 通过这个工具，你可以重定向命令的输入输出，命令的输入来自文件，而输出也存到文件。 也可以把多个命令连接起来组成一个强大的命令管道。(可以看出，这部分的命令，十分没有特点)  

+ cat - Concatenate files
cat 命令读取一个或多个文件，然后复制它们到标准输出  
管道线：  
命令从标准输入读取数据并输送到标准输出的能力被一个称为管道线的 shell 特性所利用。 使用管道操作符”|”（竖杠），一个命令的标准输出可以通过管道送至另一个命令的标准输入  

+ sort - Sort lines of text
过滤器：  
管道线经常用来对数据完成复杂的操作。有可能会把几个命令放在一起组成一个管道线。 通常，以这种方式使用的命令被称为过滤器。过滤器接受输入，以某种方式改变它，然后 输出它。第一个我们想试验的过滤器是 sort。想象一下，我们想把目录/bin 和/usr/bin 中 的可执行程序都联合在一起，再把它们排序  

+ uniq - Report or omit repeated lines
uniq 命令经常和 sort 命令结合在一起使用。uniq 从标准输入或单个文件名参数接受数据有序 列表（详情查看 uniq 手册页），默认情况下，从数据列表中删除任何重复行。所以，为了确信 我们的列表中不包含重复句子（这是说，出现在目录/bin 和/usr/bin 中重名的程序），我们添加 uniq 到我们的管道线中  
```
ls /bin /usr/bin | sort | uniq | less
```

+ grep - Print lines matching a pattern

+ wc - Print newline, word, and byte counts for each file
wc（字计数）命令是用来显示文件所包含的行数、字数和字节数。  

+ head - Output the first part of a file

+ tail - Output the last part of a file
有时候你不需要一个命令的所有输出。可能你只想要前几行或者后几行的输出内容。 head 命令打印文件的前十行，而 tail 命令打印文件的后十行。默认情况下，两个命令 都打印十行文本，但是可以通过”-n”选项来调整命令打印的行数。  

```
head -n 5 ls-output.txt
```

+ tee - Read from standard input and write to standard output and files
为了和我们的管道隐喻保持一致，Linux 提供了一个叫做 tee 的命令，这个命令制造了 一个”tee”，安装到我们的管道上。tee 程序从标准输入读入数据，并且同时复制数据 到标准输出（允许数据继续随着管道线流动）和一个或多个文件。当在某个中间处理 阶段来捕捉一个管道线的内容时，这很有帮助。这里，我们重复执行一个先前的例子， 这次包含 tee 命令，在 grep 过滤管道线的内容之前，来捕捉整个目录列表到文件 ls.txt  
```
ls /usr/bin | tee ls.txt | grep zip
```

### 软件包管理

软件包管理是指系统中一种安装和维护软件的方法。今天，通过从 Linux 发行版中安装的软件包， 已能满足许多人所有的软件需求。这不同于早期的 Linux，人们需要下载和编译源码来安装软件。 编译源码没有任何问题，事实上，拥有对源码的访问权限是 Linux 的伟大奇迹。它赋予我们（ 其它每个人）检测和提高系统性能的能力。只是若有一个预先编译好的软件包处理起来要相对 容易快速些。这章中，我们将查看一些用于包管理的命令行工具。虽然所有主流 Linux 发行版都 提供了强大且精致的图形管理程序来维护系统，但是学习命令行程序也非常重要。因为它们 可以完成许多让图形化管理程序处理起来困难（或者不可能）的任务。  

不同的 Linux 发行版使用不同的打包系统，一般而言，大多数发行版分别属于两大包管理技术阵营： Debian 的”.deb”，和红帽的”.rpm”。也有一些重要的例外，比方说 Gentoo， Slackware，和 Foresight，但大多数会使用这两个基本系统中的一个。  

1. 查找资源库中的软件包
Debian : apt-get update; apt-cache search search_string  
Red Hat : yum search search_string  
eg:搜索一个 yum 资源库来查找 emacs 文本编辑器  
yum search emacs  

2. 从资源库中安装一个软件包
上层工具允许从一个资源库中下载一个软件包，并经过完全依赖解析来安装它。  
Debian : apt-get update; apt-get install package_name
Red Hat : yum install package_name

3. 通过软件包文件来安装软件
如果从某处而不是从资源库中下载了一个软件包文件，可以使用底层工具来直接（没有经过依赖解析）安装它。  
Debian 	: dpkg --install package_file
Red Hat : rpm -i package_file

4. 卸载软件
Debian 	: apt-get remove package_name
Red Hat : yum erase package_name

5. 经过资源库来更新软件包
Debian 	: apt-get update; apt-get upgrade  
Red Hat : yum update  

6. 经过软件包文件来升级软件
Debian 	: dpkg --install package_file
Red Hat : rpm -U package_file

7. 列出所安装的软件包
Debian 	: dpkg --list
Red Hat : rpm -qa

8. 确定是否安装了一个软件包
Debian 	: dpkg --status package_name
Red Hat : rpm -q package_name

9. 显示所安装软件包的信息
Debian 	: apt-cache show package_name
Red Hat : yum info package_name

10. 查找安装了某个文件的软件包
Debian 	: dpkg --search file_name
Red Hat : rpm -qf file_name
