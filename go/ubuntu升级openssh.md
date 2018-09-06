等保，指定操作系统是Ubuntu 14.04，用户最近安全漏洞扫描，Ubuntu主机的ssh版本太低，OpenSSH_6.6.1p1，需要需要对该主机的SSH版本进行升级。

准备升级的安全包，本次升级我准备了三个文件（openssh-7.7p1.tar.gz 、openssl-1.0.2n.tar.gz、zlib-1.2.11.tar.gz，只有ssl不是最新版本，但是在1.0.2中是最高的）。zlib下载我看都是要钱的，实际这个东西是免费的，我是在http://www.zlib.net/下载的。

一、将以上的安装包拷贝到/tmp下（root用户下安装）。

二、安装zlib-1.2.11.tar.gz。1：执行解压命令tar -zxvf zlib-1.2.11.tar.gz。2：切换目录cd zlib-1.2.11。3：执行命令./configure。4：执行命令make。5：执行命令make install。

三、安装openssl-1.0.2n.tar.gz。1：执行解压命令tar -zxvf openssl-1.0.2n.tar.gz。2：切换目录cd openssl-1.0.2n。3：执行命令./config shared zlib。4：执行命令make。5：执行命令make install。6：执行命令 mv /usr/bin/openssl /usr/bin/openssl_old。7：执行命令mv /usr/include/openssl /usr/include/openssl_old。8：执行命令ln -s /usr/local/ssl/bin/openssl /usr/bin/openssl。9：执行命令ln -s /usr/local/ssl/include/openssl /usr/include/openssl。10：执行命令echo "/usr/local/ssl/lib" >> /etc/ld.so.conf。11：执行命令/sbin/ldconfig -v。

四、安装openssh-7.7p1.tar.gz。1：执行解压命令tar -zxvf openssh-7.7p1.tar.gz。2：切换目录cd openssh-7.7p1。3：执行命令./configure --prefix=/usr --sysconfdir=/etc/ssh  --with-zlib --with-ssl-dir=/usr/local/ssl  --with-md5-passwords --mandir=/usr/share/man。4：执行命令make。5：执行命令make install。6：执行指令cp sshd /etc/init.d

这块需要说明，在执行第3步的时候，会报错如下：

configure: error: Your OpenSSL headers do not match your
        library. Check config.log for details.
        If you are sure your installation is consistent, you can disable the check
        by running "./configure --without-openssl-header-check".

​        Also see contrib/findssl.sh for help identifying header/library mismatches.

报错后我检查执行的指令，确认没有问题，执行./configure --without-openssl-header-check这个指令禁用检查，继续执行make和make install安装成功。

五、重启服务器检查是否开机自动启动，检查版本。

root@ceshi-virtual-machine:~# ssh -V
OpenSSH_7.7p1, OpenSSL 1.0.2g-fips  1 Mar 2016



![image-20180724210902894](/Users/zhanghaiyu/haiyuzhang-deepglint.github.io/go/images/openssh/ssh_version.png)