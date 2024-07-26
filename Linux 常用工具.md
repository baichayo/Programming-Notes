# Linux 常用工具

## find

用于在文件系统中搜索文件和目录

```bash
find [路径] [搜索条件] [操作]
```

- `-name` ：按文件名搜索
  - `find /path/to/search -name "filename.txt"` 	查找 `/path/to/search` 目录及其子目录中名为 `filename.txt` 的文件

- `-iname` ：按文件名（不区分大小写）搜索
  - `find /path/to/search -iname "filename.txt"`
- `-type` ：按文件类型搜索
  - `find /path/to/search -type f`  	所有普通文件
  - `f d l c b p s `   普通文件 目录 符号链接 字符设备文件 块设备文件 管道文件 套接字文件
- `-size` ：按文件大小搜索
  - `find /path/to/search -size +10M` 	大于 10MB 的文件
- `-mtime` ：按修改时间搜索
  - `find /path/to/search -mtime -7` 	最近 7 天内修改过的文件
  - `find /path/to/search -mtime +30` 30 天前修改过的文件
- `-user` ：按文件所有者搜索
  - `find /path/to/search -user username` 	
- `-prem` ：按权限搜索
  - `find /path/to/search -perm 644` 	查找 `/path/to/search` 目录及其子目录中权限为 `644` 的文件
- `-exec` ：对找到的文件执行命令
  - `find /path/to/search -name "*.log" -exec rm {} \;` 	查找 `/path/to/search` 目录及其子目录中的所有 `.log` 文件，并删除它们。`{}` 代表找到的文件，`\;` 结束 `-exec` 选项



- `-print` ：打印找到的文件路径
  - `find /path/to/search -name "*.txt" -print` 	打印所有 `.txt` 文件的路径
- `-delete` ：删除找到的文件
  - `find /path/to/search -name "*.tmp" -delete` 	删除 `/path/to/search` 目录及其子目录中的所有 `.tmp` 文件
- `-ls` ：列出找到的文件的详细信息
  - `find /path/to/search -name "*.txt" -ls` 	列出所有 `.txt` 文件的详细信息
- `-and` ：与条件
  - `find /path/to/search -name "*.txt" -and -size +1M`  	查找 `/path/to/search` 目录及其子目录中所有大于 1MB 的 `.txt` 文件





## grep

`grep` 是一个用于搜索文本的命令行工具。它从文件或标准输入中查找符合指定模式的行，并输出这些行。

### 基本用法

```bash
grep [选项] [模式] [文件...]
```

- **`[选项]`**: 用于控制 `grep` 的行为。
- **`[模式]`**: 要搜索的文本模式，可以是一个字符串或正则表达式。
- **`[文件...]`**: 要搜索的文件。如果没有指定文件，`grep` 将从标准输入读取数据。

### 常用选项

- `-i` ：忽略大小写
  - `grep -i "pattern" file.txt` 	查找 `file.txt` 中所有匹配 `pattern`（忽略大小写）的行
- `-v` ：反转匹配，输出不匹配的行
  - `grep -v "pattern" file.txt` 	输出 `file.txt` 中不包含 `pattern` 的所有行
- `-r` ：递归搜索目录
  - `grep -r "pattern" /path/to/directory` 	在 `/path/to/directory` 目录及其子目录中递归搜索 `pattern`
- `-l` ：仅输出包含匹配模式的文件名
  - `grep -l "pattern" *.txt` 	列出当前目录中所有包含 `pattern` 的 `.txt` 文件
- `-n` ：显示匹配行的行号
  - `grep -n "pattern" file.txt` 	输出 `file.txt` 中所有匹配 `pattern` 的行及其行号
- `-H` ：输出匹配行的文件名（在多个文件中查找时）
  - `grep -H "pattern" file1.txt file2.txt` 在 `file1.txt` 和 `file2.txt` 中查找 `pattern` 并输出文件名
- `-o` ： 仅输出匹配的部分
  - `grep -o "pattern" file.txt` 	仅输出 `file.txt` 中匹配 `pattern` 的部分，而不是整行
- `-c` ：统计匹配的行数
  - `grep -c "pattern" file.txt` 	统计 `file.txt` 中包含 `pattern` 的行数
- `-e` ：使用多个模式
  - `grep -e "pattern1" -e "pattern2" file.txt` 	查找 `file.txt` 中匹配 `pattern1` 或 `pattern2` 的行
- `-w` ：匹配整个单词
  - `grep -w "pattern" file.txt` 	查找 `file.txt` 中以 `pattern` 为单词的行，而不是作为单词的一部分
- `-x` ：完全匹配整行
  - `grep -x "pattern" file.txt` 	查找 `file.txt` 中整行完全匹配 `pattern` 的行

所以 `grep` 基本上是来查找文件的，平时用 `find . | grep "main"` 来递归地查找当前目录包含特定字符的文件，`find . | grep "main" | xargs cat` 来输出行， 但其实 `grep-r "main" .` 就可以胜任



## ag







## awk

`awk` 是一种强大的文本处理工具，通常用于从文件或标准输入流中提取数据、转换数据格式和执行其他文本处理任务。它以行为单位逐行扫描文本文件，并按照用户指定的规则进行处理。

- `awk '{ print $1 }' file.txt` ：提取文件中每行的第一个字段，默认使用空格作为字段分隔符
- `-F`：指定字段分隔符



## xargs







## lsof

lsof（list open file）是一个列出当前系统打开的文件描述符的工具。通过它我们可以了解进程打开了哪些文件描述符，或者文件描述符被哪些进程打开。

- `-i`：显示 socket 文件描述符，使用方法：
  - `lsof -i [46][protocol][@hostname|hostaddr][:service|port]`
  - `sudo lsof -i 4tcp@127.0.0.1:22` （注意不能有空格）
  
- `-u` ：显示指定用户启动的所有进程打开的所有文件描述符

- `-c` ：显示指定的命令打开的文件描述符

- `-p` ：显示指定进程打开的所有文件描述符

- `-t` ：显示打开了目标文件描述符的进程的 PID





## netstat

`netstat` 是一个用于查看网络连接、路由表和网络统计信息的命令行工具。它可以帮助你了解系统的网络活动情况。

- `-a`: 显示所有的网络连接，包括监听和已建立的连接。
- `-t`: 仅显示 TCP 连接信息。
- `-u`: 仅显示 UDP 连接信息。
- `-n`: 显示数字格式的 IP 地址和端口号，避免进行主机名和服务名解析。
- `-l`: 显示所有正在监听的端口及其相关信息。
- `-p`: 显示与每个网络连接关联的 PID（进程标识符）和程序名。
- `-r`: 显示系统的路由表，包括目标网络、网关、接口等信息。
- `-s`: 显示网络相关的统计信息，如收发的数据包、错误等。
- `-c`: 持续显示输出，每隔一段时间刷新一次。
- `-W`: 指定显示的宽度，可用于格式化输出。
- `-i`: 显示网络接口的统计信息，如传输、丢包等。

常用命令 `netstat -antp` 

组合技 `sudo netstat -antp | grep 12345 | awk '{print $7}' | cut -d '/' -f 1 | xargs sudo kill`



## ps

`ps` 是一个用于显示当前系统中运行进程信息的命令。

- `-A`：显示所有进程，包括其他用户的进程。
- `-u`：显示属于指定用户的进程信息。
- `-x`：显示没有控制终端的进程。
- `-e`：显示所有进程。
- `-f`：显示完整的进程信息，包括命令行参数和其他详细信息。
- `-l`：显示长格式输出，包括更多的列信息。
- `-o`：自定义输出格式，可以通过逗号分隔的选项指定要显示的列。
- `-p`：显示指定进程ID的进程信息。
- `-t`：显示指定终端的进程信息。
- `--sort`：按指定的列进行排序输出。
- `--forest`：以树状形式显示进程层级关系。
- `--pid`：显示指定进程ID的信息。
- `--ppid`：显示指定父进程ID的进程信息。
- `--user`：显示指定用户的进程信息。
- `--group`：显示指定用户组的进程信息

常用命令 `ps aux` 或 `ps -ef`

组合技 `ps -ef | grep 12345 | awk '{print $2}' | xargs -I {}  kill {}`



## cut

- `-c` ：按字符提取。
  - `cut -c 1-5 filename` 	提取第 1 到第 5 个字符
  - `cut -c 1,4 filename`     提取第 1 个和第 4 个字符
- `-f` ：按字段提取（字段默认由制表符 (TAB) 分隔）
  - `cut -f 1,3 filename` 	提取第 1 和第 3 个字段
- `-d` ：指定字段分隔符
  - `cut -d ':' -f 2 filename` 	使用冒号 `:` 作为分隔符，提取第 2 个字段
- `--complement` ：提取除指定字符或字段外的其他部分
  - `cut -c 1-5 --complement filename`	提取除第 1 到第 5 个字符外的所有字符



## sort

- `-n`: 按数值排序

  - `sort -n filename` 	按数值进行排序

- `-r`: 反向排序
  - `sort -r filename` 	按相反的顺序排序

- `-k`: 按指定字段排序
  - `sort -k 2 filename` 	按第 2 个字段排序。字段默认由空白字符（空格或制表符）分隔。

- `-t`: 指定字段分隔符
  - `sort -t ',' -k 2 filename` 	使用逗号 `,` 作为字段分隔符，并按第 2 个字段排序。

- `-s` ：稳定排序
  - `sort -k1，1 | sort -s -k2，2` 	首先按第 1 列进行排序，然后按第 2 列进行稳定排序（`k1, 1` 的意思是 第一行到第一行）

- `-u`: 删除重复行
  - `sort -u filename` 	排序并删除重复的行

- `-f`: 忽略大小写排序
  - `sort -f filename` 	忽略字母的大小写进行排序。例如，`a` 和 `A` 会被视为相同。

- `-d`: 只对字母和数字排序
  - `sort -d filename` 	仅对字母、数字和空白字符进行排序

- `-b`: 忽略行首的空白字符
  - `sort -b filename` 




## less

`less` 是一个功能强大的文本查看器

#### 常用键操作

- **空格键**：向下滚动一页。
- **b 键**：向上滚动一页。
- **j 键**：向下滚动一行。
- **k 键**：向上滚动一行。
- **g 键**：跳转到文件开头。
- **G 键**：跳转到文件结尾。
- **/pattern**：向下搜索模式 `pattern`。
- **?pattern**：向上搜索模式 `pattern`。
- **n**：重复前一个搜索（同一方向）。
- **N**：重复前一个搜索（相反方向）。
- **h**：显示帮助。
- **q**：退出 `less`。

