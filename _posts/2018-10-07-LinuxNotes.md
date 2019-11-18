---
layout: post
title: Linux那些事儿
---

##### Linux自学资料

1. [Linux 基礎學習篇訓練教材](http://linux.vbird.org/linux_basic_train/)
2. Running Linux



##### 第 1 堂課：初次使用 Linux 與指令列模式初探

CPU 架構：

1. X86 個人電腦
2. ARM 手持式裝置

作業系統主要包含的部份是：核心+系統呼叫。

Linux distribution = Linux Kernel + Softwares + Tools + 可完整安裝程序

- [Ctrl] + [Alt] + [F2] ~ [F6] ：文字介面登入 tty2 ~ tty6 終端機

- [Ctrl] + [Alt] + [F1] ：圖形介面桌面

##### 第 2 堂課：指令下達行為與基礎檔案管理

```bash
$ command  [-options]  [parameter1...]
```

usr 不是 user 喔！是 unix software resource 的縮寫

##### 第 3 堂課：檔案管理與 vim 初探

萬用字元

1. *
2. ?
3. [ ]
4. [ - ]
5. [^ ]

- **yy** -- 為複製游標所在行，5yy 為複製 5 行，nyy 為複製 n 行
- **p** -- 為在游標底下貼上剛剛刪除/複製的資料

##### 第 4 堂課：Linux 基礎檔案權限與基礎帳號管理

##### 第 5 堂課：權限應用、程序之觀察與基本管理

- 程式 (program)：通常為 binary program ，放置在儲存媒體中 (如硬碟、光碟、軟碟、磁帶等)， 為實體檔案的型態存在；
- 程序 (process)：程式被觸發後，執行者的權限與屬性、程式的程式碼與所需資料等都會被載入記憶體中，作業系統並給予這個記憶體內的單元一個識別碼 (**PID**)，可以說，程序就是一個正在運作中的程式。

##### 第 6 堂課：基礎檔案系統管理

##### 第 7 堂課：認識 bash 基礎與系統救援

1. **env** -- set environment and execute command, or print environment
2. **set**
3. **export**

通常 login shell 讀取設定檔的流程是：

- /etc/profile：這是系統整體的設定，你最好不要修改這個檔案；

- ~/.bash_profile 或 ~/.bash_login 或 ~/.profile (只會讀 1 個，依據優先順序決定)：屬於使用者個人設定，你要改自己的資料，就寫入這裡！

由於 login shell 已經讀取了 /etc/profile 因此已經設定了大部分的全域變數設定，所以 non-login shell 只需要少部份的設定即可。 故 non-login shell 只會讀取一個個人設定檔，亦即是：

- ~/.bashrc

由於 ~/.bash_profile 也是讀取 ~/.bashrc ，因此使用者只需要將設定放置於家目錄下的 .bashrc 就可以讓兩者讀取了。



**系統救援**

- systemd



##### 第 8 堂課：bash 指令連續下達與資料流重導向

##### 第 9 堂課：正規表示法與 shell script 初探

- ^word
- word$
- .
- \
- *
- [list]
- [n1-n2]
- [^list]
- \{n,m\}
- [:alnum:]
- [:alpha:]
- [:blank:]
- [:cntrl:]
- [:digit:]
- [:graph:]
- [:lower:]
- [:print:]
- [:punct:]
- [:upper:]
- [:space:]
- [:xdigit:]

##### 第 10 堂課：使用者管理與 ACL 權限設定

##### 第 11 堂課：基礎設定、備份、檔案壓縮打包與工作排程

firewall-cmd

**TODO**: <u>“只要來自 172.16.0.0/16 的 ssh 連線要求，均予以放行”？</u>

- tar
- crontab

##### 第 12 堂課：軟體管理與安裝及登錄檔初探

| distribution 代表 | 軟體管理機制 |   使用指令    | 線上升級機制(指令) |
| :---------------: | :----------: | :-----------: | :----------------: |
|  Red Hat/Fedora   |     RPM      | rpm, rpmbuild |   YUM (**yum**)    |
|   Debian/Ubuntu   |     DPKG     |     dpkg      | APT (**apt-get**)  |

rsyslog

##### 第 13 堂課：服務管理與開機流程管理

systemd

systemctl

![HackTheWorld](/images/HackTheWorld.png)

一般正常的情況下， Linux 的開機流程會是如下所示：

1. 載入 BIOS 的硬體資訊與進行自我測試，並依據設定取得第一個可開機的裝置；
2. 讀取並執行第一個開機裝置內 MBR 的 **boot Loader** (亦即是 grub2, spfdisk 等程式)；
3. 依據 boot loader 的設定載入 Kernel ，Kernel 會開始偵測硬體與載入驅動程式；
   - 載入 kernel file 與 initramfs 檔案在記憶體內解壓縮
   - initramfs 會在記憶體模擬出系統根目錄，提供 kernel 相關的驅動程式模組
   - 核心裝置驅動程式完整的驅動硬體
4. 在硬體驅動成功後，Kernel 會主動呼叫 systemd 程式，並以 default.target 流程開機；
   - systemd 執行 sysinit.target 初始化系統及 basic.target 準備作業系統；
   - systemd 啟動 multi-user.target 下的本機與伺服器服務；
   - systemd 執行 multi-user.target 下的 /etc/rc.d/rc.local 檔案；
   - systemd 執行 multi-user.target 下的 getty.target 及登入服務；
   - systemd 執行 graphical 需要的服務

##### 第 14 堂課：進階檔案系統管理Hac