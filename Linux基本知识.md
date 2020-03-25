# 文件权限

![image-20200324150727482](Linux基本知识.assets/image-20200324150727482.png)

权限共有三组：第一组是文件所有者的权限，第二组文件加入该群组账号的权限，第三组不属于前两者的权限

## 修改文件属性和权限

- `chgrp` ：改变文件所属群组 

  ```bash
  [root@study ~]# chgrp [-R] dirname/filename ... 
  选项与参数： -R : 进行递归(recursive)的持续变更，亦即连同次目录下的所有文件、目录 都更新成为这个群组之意。常常用在变更某一目录内所有的文件之情况。
  范例： 
  [root@study ~]# chgrp users initial-setup-ks.cfg 
  [root@study ~]# ls -l 
  -rw-r--r--. 1 root users 1864 May 4 18:01 initial-setup-ks.cfg 
  [root@study ~]# chgrp testing initial-setup-ks.cfg 
  chgrp: invalid group: `testing' <== 发生错误讯息啰～找不到这个群组名～
  ```

- `chown` ：改变文件拥有者和群组

  ```bash
  [root@study ~]# chown [-R] 账号名称 文件或目录 
  [root@study ~]# chown [-R] 账号名称:组名 文件或目录 
  选项与参数： -R : 进行递归(recursive)的持续变更，亦即连同次目录下的所有文件都变更
  范例：将 initial-setup-ks.cfg 的拥有者改为 bin 这个账号： 
  [root@study ~]# chown bin initial-setup-ks.cfg 
  [root@study ~]# ls -l 
  -rw-r--r--. 1 bin users 1864 May 4 18:01 initial-setup-ks.cfg
  范例：将 initial-setup-ks.cfg 的拥有者与群组改回为 root： 
  [root@study ~]# chown root:root initial-setup-ks.cfg 
  [root@study ~]# ls -l 
  -rw-r--r--. 1 root root 1864 May 4 18:01 initial-setup-ks.cfg
  ```

- `chmod` ：改变文件的权限, SUID, SGID, SBIT 等等的特性

  ### 数字类型修改权限

  修改文件权限涉及`owner/group/others` 三种身份各有自己的 `read/write/execute` 权限。

  各个权限分数`r:4   w:2   x:1`
  每种身份(owner/group/others)各自的三个权限(r/w/x)分数是需要累加的，例如当权限为： [-rwxrwx---] 分数 则是：
  `owner = rwx = 4+2+1 = 7` 

  `group = rwx = 4+2+1 = 7` 

  `others= --- = 0+0+0 = 0`

  ```bash
  [root@study ~]# chmod [-R] xyz 文件或目录 
  选项与参数： xyz : 就是刚刚提到的数字类型的权限属性，为 rwx 属性数值的相加。 -R : 进行递归(recursive)的持续变更，亦即连同次目录下的所有文件都会变更
  ```

  ### 符号类型修改权限

  `(1)user (2)group (3)others` 三种身份啦！那么我们就可以藉由 u, g, o 来代表三种身份的权限！此外， a 则代表 all 亦即全部 的身份！

  ![image-20200324160035074](Linux基本知识.assets/image-20200324160035074.png)

  ```bash
  [root@study ~]# chmod u=rwx,go=rx .bashrc 
  # 注意喔！那个 u=rwx,go=rx 是连在一起的，中间并没有任何空格符！ [root@study ~]# ls -al .bashrc 
  -rwxr-xr-x. 1 root root 176 Dec 29 2013 .bashrc
  
  [root@study ~]# chmod a+w .bashrc
  [root@study ~]# chmod a-x .bashrc
  ```

*在 Linux 底下，我们的文件是否能被执行，则是藉由是否具有『x』这个权限来决定的！跟 档名是没有绝对的关系的！*

rwx都是和文档内容有关，和文件是否存在无关。

#### 权限对于目录的含义

- **r (read contents in directory)**：可进入文件夹，但不能看到内容
  表示具有读取目录结构列表的权限，所以当你具有读取(r)一个目录的权限时，表示你可以查询该目录下的 文件名数据。 可以利用 ls 指令将该目录的内容列表显示出来！

- **w (modify contents of directory)**：可以进入并且看到内容，但不能移动修改
  这个可写入的权限对目录来说，是很了不起的！ 因为他表示你具有异动该目录结构列表的权限，也就是底 下这些权限：

  - 建立新的文件与目录； 
  - 删除已经存在的文件与目录(不论该文件的权限为何！) 
  - 将已存在的文件或目录进行更名； 
  - 搬移该目录内的文件、目录位置

- **x (access directory)**：不能进入文件夹

  目录的 x 代表的是用户能否进入该目录成为工作目录的用途

## 文件目录分布

根据`Filesystem Hierarchy Standard (FHS)`标准

![image-20200324164636830](Linux基本知识.assets/image-20200324164636830.png)

- 可分享的：可以分享给其他系统挂载使用的目录，所以包括执行文件与用户的邮件等数据， 是能够分享给 网络上其他主机挂载用的目录；
- 不可分享的：自己机器上面运作的装置文件或者是与程序有关的 socket 文件等， 由于仅与自身机器有关， 所以当然就不适合分享给其他主机了。
- 不变的：有些数据是不会经常变动的，跟随着 distribution 而不变动。 例如函式库、文件说明文件、系统管 理员所管理的主机服务配置文件等等；
- 可变动的：经常改变的数据，例如登录文件、一般用户可自行收受的新闻组等

最主要的：

- / (root, 根目录)：与开机系统有关； 
-  /usr (unix software resource)：与软件安装/执行有关； 
-  /var (variable)：与系统运作过程有关

## 文件目录管理

```bash
.       代表此层目录
..      代表上一层目录
-        代表前一个工作目录
~       代表『目前用户身份』所在的家目录
~account 代表 account 这个用户的家目录(account 是个账号名称
```

- cd：变换目录 

-  pwd：显示当前目录   **-P** ：显示出确实的路径，而非使用链接 (link) 路径。

  ```bash
  [root@study mail]# pwd 
  /var/mail    <==列出目前的工作目录
  [root@study mail]# pwd -P 
  /var/spool/mail   <==怎么回事？有没有加 -P 差很多～
  [root@study mail]# ls -ld 
  /var/mail lrwxrwxrwx. 1 root root 10 May 4 17:51 /var/mail -> spool/mail 
  # 看到这里应该知道为啥了吧？因为 /var/mail 是连结档，连结到 /var/spool/mail 
  # 所以，加上 pwd -P 的选项后，会不以连结文件的数据显示，而是显示正确的完整路径啊
  ```

- mkdir：建立一个新的目录 

  ```bash
  [root@study ~]# mkdir [-mp] 目录名称 
  选项与参数： -m ：配置文件案的权限喔！直接设定，不需要看预设权限 (umask) 的脸色～ 
  -p ：帮助你直接将所需要的目录(包含上层目录)递归建立起来！建立多级目录
  
  [root@study tmp]# mkdir -p test1/test2/test3/test4 
  # 加了这个 -p 的选项，可以自行帮你建立多层目录
  
  [root@study tmp]# mkdir -m 711 test2
  ```

-  rmdir：删除一个空的目录

  ```bash
  [root@study ~]# rmdir [-p] 目录名称 
  选项与参数： -p ：连同『上层』『空的』目录也一起删除
  # 不过要注意的是，这个 rmdir 仅能『删除空的目录』喔！
  ```

  那如果要将所有目录下的东西都杀掉, 这 个时候就必须使用『 `rm -r test` 』

### 文件的检视  ls

```bash
root@study ~]# ls [-aAdfFhilnrRSt] 文件名或目录名称.. 
[root@study ~]# ls [--color={never,auto,always}] 文件名或目录名称.. [root@study ~]# ls [--full-time] 文件名或目录名称.. 
选项与参数： 
-a ：全部的文件，连同隐藏档( 开头为 . 的文件) 一起列出来(常用) 
-A ：全部的文件，连同隐藏档，但不包括 . 与 .. 这两个目录 
-d ：仅列出目录本身，而不是列出目录内的文件数据(常用) 
-f ：直接列出结果，而不进行排序 (ls 预设会以档名排序！) 
-F ：根据文件、目录等信息，给予附加数据结构，例如： *:代表可执行文件； /:代表目录； =:代表 socket 文件； |:代表 FIFO 文件；
-h ：将文件容量以人类较易读的方式(例如 GB, KB 等等)列出来； 
-i ：列出 inode 号码，inode 的意义下一章将会介绍； 
-l ：长数据串行出，包含文件的属性与权限等等数据；(常用) 
-n ：列出 UID 与 GID 而非使用者与群组的名称 (UID 与 GID 会在账号管理提到！) 
-r ：将排序结果反向输出，例如：原本档名由小到大，反向则为由大到小；
-R ：连同子目录内容一起列出来，等于该目录下的所有文件都会显示出来； 
-S ：以文件容量大小排序，而不是用档名排序； 
-t ：依时间排序，而不是用档名。 
--color=never ：不要依据文件特性给予颜色显示； 
--color=always ：显示颜色 
--color=auto ：让系统自行依据设定来判断是否给予颜色 
--full-time ：以完整时间模式 (包含年、月、日、时、分) 输出 
--time={atime,ctime} ：输出 access 时间或改变权限属性时间 (ctime) 而非内容变更时间 (modification time)
```

### 复制、删除与移动： cp, rm, mv

- CP

  ```bash
  [root@study ~]# cp [-adfilprsu] 来源文件(source) 目标文件(destination) 
  [root@study ~]# cp [options] source1 source2  .... directory 
  选项与参数： 
  -a ：相当于 -dr --preserve=all 的意思，至于 dr 请参考下列说明；(常用) 
  -d ：若来源文件为链接文件的属性(link file)，则复制链接(实际)文件属性而非文件本身； 
  -f ：为强制(force)的意思，若目标文件已经存在且无法开启，则移除后再尝试一次； -i ：若目标文件(destination)已经存在时，在覆盖时会先询问动作的进行(常用) 
  -l ：进行硬式连结(hard link)的连结档建立，而非复制文件本身； 
  -p ：连同文件的属性(权限、用户、时间)一起复制过去，而非使用默认属性(备份常用)； 
  -r ：递归持续复制，用于目录的复制行为；(常用) 
  -s ：复制成为符号链接文件 (symbolic link)，亦即『快捷方式』文件； 
  -u ：destination 比 source 旧才更新 destination，或 destination 不存在的情况下才复制。
  --preserve=all ：除了 -p 的权限相关参数外，还加入 SELinux 的属性, links, xattr 等也复制了。 
  最后需要注意的，如果来源档有两个以上，则最后一个目的文件一定要是『目录』才行！
  ```

- rm

  ```bash
  [root@study ~]# rm [-fir] 文件或目录 
  选项与参数： 
  -f ：就是 force 的意思，忽略不存在的文件，不会出现警告讯息； 
  -i ：互动模式，在删除前会询问使用者是否动作 
  -r ：递归删除啊！最常用在目录的删除了！这是非常危险的选项！！！
  ```

- mv (移动文件与目录，或更名)

  ```bash
  [root@study ~]# mv [-fiu] source destination 
  [root@study ~]# mv [options] source1 source2 source3 .... directory 
  选项与参数： 
  -f ：force 强制的意思，如果目标文件已经存在，不会询问而直接覆盖； 
  -i ：若目标文件 (destination) 已经存在时，就会询问是否覆盖！ 
  -u ：若目标文件已经存在，且 source 比较新，才会更新 (update)
  ```

- 取得路径的文件名与目录名称

  ```bash
  [root@study ~]# basename /etc/sysconfig/network 
  network         <== 很简单！就取得最后的档名～
  [root@study ~]# dirname /etc/sysconfig/network 
  /etc/sysconfig  <== 取得的变成目录名了！
  ```

### 文件内容查阅

- cat 由第一行开始显示文件内容   -n打印行号  -E显示换行符($)  -T显示Tab(^)

-  tac 从最后一行开始显示，可以看出 tac 是 cat 的倒着写！ 

-  nl 显示的时候，顺道输出行号！ 

-  more 一页一页的显示文件内容 

  more 这个程序的运作过程中，你有几个按键可以按的：

  - 空格键 (space)：代表向下翻一页； 
  - Enter                ：代表向下翻『一行』；
  - /字符串           ：代表在这个显示的内容当中，向下搜寻『字符串』这个关键词；
  -  :f                     ：立刻显示出文件名以及目前显示的行数；
  -  q                     ：代表立刻离开 more ，不再显示该文件内容。
  - b 或 [ctrl]-b       ：代表往回翻页，不过这动作只对文件有用，对管线无用

- **less 与 more 类似，但是比 more 更好的是，他可以往前翻页！** 

  - 空格键 ：向下翻动一页； 
  -  [pagedown]：向下翻动一页； 
  -  [pageup] ：向上翻动一页； 
  -  /字符串 ：向下搜寻『字符串』的功能； 
  - ?字符串 ：向上搜寻『字符串』的功能； 
  - n   ：重复前一个搜寻 (与 / 或 ? 有关！)
  -  N   ：反向的重复前一个搜寻 (与 / 或 ? 有关！)
  -  g   ：前进到这个资料的第一行去；
  -  G   ：前进到这个数据的最后一行去 (注意大小写)；
  -  q    ：离开 less 这个程序；

-  head 只看头几行  *-n ：后面接数字，代表显示几行的意思*

-  tail 只看尾巴几行 

-  od 以二进制的方式读取文件内容！

### 修改时间

三个时间的意义：

- modification time (mtime)： 当该文件的『内容数据』变更时，就会更新这个时间！内容数据指的是文件的内容，而不是文件的属性或 权限喔！（默认显示）
- status time (ctime)： 当该文件的『状态 (status)』改变时，就会更新这个时间，举例来说，像是权限与属性被更改了，都会更新 这个时间啊。
- access time (atime)： 当『该文件的内容被取用』时，就会更新这个读取时间 (access)。举例来说，我们使用 cat 去读取 /etc/man_db.conf ， 就会更新该文件的 atime 了。

```bash
[root@study ~]# touch [-acdmt] 文件 
选项与参数： 
-a ：仅修订 access time； 
-c ：仅修改文件的时间，若该文件不存在则不建立新文件； 
-d ：后面可以接欲修订的日期而不用目前的日期，也可以使用 
--date="日期或时间" 
-m ：仅修改 mtime ； 
-t ：后面可以接欲修订的时间而不用目前的时间，格式为[YYYYMMDDhhmm]
```

- 观察文件类型：file

```bash
[root@study ~]# file ~/.bashrc 
/root/.bashrc: ASCII text <==告诉我们是 ASCII 的纯文本档啊
```

### 指令与文件的搜寻

- which (寻找『执行档』)

  ```bash
  [root@study ~]# which [-a] command 
  选项或参数： 
  -a ：将所有由 PATH 目录中可以找到的指令均列出，而不止第一个被找到的指令名称
  范例一：搜寻 ifconfig 这个指令的完整文件名 
  [root@study ~]# which ifconfig /sbin/ifconfig
  ```

- `whereis` (由一些特定的目录中寻找文件文件名)

  通常 find 不很常用的！因 为速度慢之外， 也很操硬盘！一般我们都是先使用 `whereis` 或者是 locate 来检查，如果真的找不到 了，才以 find 来搜寻呦！

  ```bash
  [root@study ~]# whereis [-bmsu] 文件或目录名 
  选项与参数： 
  -l :可以列出 whereis 会去查询的几个主要目录而已
  -b :只找 binary 格式的文件 
  -m :只找在说明文件 manual 路径下的文件 
  -s :只找 source 来源文件 
  -u :搜寻不在上述三个项目当中的其他特殊文件
  范例一：请找出 ifconfig 这个档名 
  [root@study ~]# whereis ifconfig 
  ifconfig: /sbin/ifconfig /usr/share/man/man8/ifconfig.8.gz
  ```

  