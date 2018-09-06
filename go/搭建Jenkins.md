### 为Golang项目搭建Jenkins

[TOC]



#### 开通云服务器 

> 私有网络, 不需要公网IP, 弹性云盘

1. 首次登陆后, 挂在磁盘, 格式化磁盘 

   参考链接: https://cloud.tencent.com/document/product/362/5739

2. 安装JDK

   

3. 安装Jenkins 

   参考: https://jenkins.io/doc/book/installing/#debianubuntu



#### 配置Jenkins

1. 访问 http://10.0.1.48:8080

2. 安装推荐插件

   - 安装Go

     参考: http://docs.studygolang.com/doc/install

     ```bash
     sudo tar -C /usr/local -xzf go$VERSION.$OS-$ARCH.tar.gz
     ```

     配置环境变量:

     ```bash
     sudo vim /etc/profile
     ```

     ```bash
     export GOROOT=/usr/local/go
     export GOPATH=$GOROOT/bin
     export PATH=$PATH:$GOPATH
     ```

     ```bash
     source /etc/profile
     ```

     ```bash
     go version
     ```

     

   - 安装Git

   - 安装 [Next Build Number Plugin](https://wiki.jenkins.io/display/JENKINS/Next+Build+Number+Plugin)

3. 创建第一个管理员用户

4. 邮件提醒



#### 添加构建任务

1. 构建一个自由风格的软件项目并指定任务名称- 确定 保存

   任务名称最好是仓库名称

2. 添加 任务描述

3. 指定Github project

4. 源码管理- - - 配置

   1. git

   2. repository url + credential

   3. Branch to build

   4.  additional behaviours - - - check out to a sub-directory

      ```bash
      $WORKSPACE/../src/github.com/haiyuzhang-deepglint/DemoForJenkins
      ```

      

5. 构建触发器

   1. 轮询 SCM
   2. 日程表: H/2 * * * *

6. 构建环境

   1. Add timestamps to the Console Output
   2. Set up Go programming language tools
   3. Select Go versions

7. 构建

   1. 执行shell

   2. paste the script below to the input box

      ```bash
      cd $GOPATH/src/github.com/deepglint
      
      #check dgc-infra
      #sudo git clone git@github.com:deepglint/dgc-infra.git
      #git clone https://haiyuzhang-deepglint:haW-yEg-R8J-A9v@github.com/deepglint/dgc-infra.git
      cd dgc-infra
      git checkout master
      git pull
      ```
      govendor sync

      ```bash
      cd $GOPATH/src/github.com/deepglint/godeye/src
      
      # govendor sync
      #echo "ssh-add-l"
      #ssh-add -l
      echo "$USER"
      eval "$(ssh-agent -s)"
      ssh-add ~/.ssh/id_rsa
      ssh-add -l
      govendor sync
      
      cd ..
      GOOS=linux GOARCH=amd64 go build -v -o godeye src/main.go
      
      cp godeye $WORKSPACE/godeye
      ```

8. 构建后操作

   1. 添加构建后操作步骤
   2. 归档成品 --- demo
   3. 邮件提醒

9. 保存

10. 构建

   1. 执行shell

      ```bash
      # Create GOPATH
      export GOPATH=$WORKSPACE/..
      export PATH=$GOPATH:$PATH
      
      # change working directory
      cd $GOPATH/src/github.com/deepglint/godeye/src
      # govendor sync
      govendor sync
      
      # change working directory
      cd $GOPATH/src/github.com/deepglint/godeye
      # run bash shell script
      ./build_ci.sh
      ```

      

   2. 

11. 构建后操作

12. 



#### 安装Docker

因为我们需要把编译后的文件push到腾讯云的镜像仓库, 所以需要安装Docker.

##### https://docs.docker.com/install/linux/docker-ce/ubuntu/#os-requirements

**https://docs.docker.com/install/linux/docker-ce/ubuntu/#upgrade-docker-ce**

1. uninstall old versions

   ```bash
   sudo apt-get remove docker docker-engine docker.io
   ```

2. set up the repository

   - update the apt package index

     ```bash
     sudo apt-get update
     ```

   - Install packages to allow `apt` to use a repository over HTTPS:

     ```bash
     sudo apt-get install \
         apt-transport-https \
         ca-certificates \
         curl \
         software-properties-common
     ```

   - Add Docker’s official GPG key:

     ```bash
     curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
     ```

     Verify that you now have the key with the fingerprint

     ```bash
     sudo apt-key fingerprint 0EBFCD88
     ```

   - Use the following command to set up the **stable** repository. You always need the **stable** repository, even if you want to install builds from the **edge** or **test** repositories as well. To add the **edge** or **test** repository, add the word `edge` or `test` (or both) after the word `stable` in the commands below.

     ```bash
     sudo add-apt-repository \
        "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
        $(lsb_release -cs) \
        stable"
     ```

3. Install Docker CE

   - Update the `apt` package index.

     ```bash
      sudo apt-get update
     ```

   - Install the *latest version* of Docker CE, or go to the next step to install a specific version:

     ```bash
     sudo apt-get install docker-ce
     ```

   - To install a *specific version* of Docker CE, list the available versions in the repo, then select and install:

     1).  List the versions available in your repo:

     ```bash
      apt-cache madison docker-ce
     ```

     2).  Install a specific version by its fully qualified package name, which is package name (`docker-ce`) “=” version string (2nd column), for example, `docker-ce=18.03.0~ce-0~ubuntu`.

     ```bash
     sudo apt-get install docker-ce=<VERSION>
     ```

   - Verify that Docker CE is installed correctly by running the `hello-world` image.

     ```bash
     sudo docker run hello-world
     ```

4. 



#### 将Docker和Jenkins目录移到数据盘

请先对数据盘进行分区, 如果没有, 请按照以下步骤进行分区并自动挂载.

https://cloud.tencent.com/document/product/362/6735



1. 停止服务

   ```bash
   sudo service docker stop
   ```

   ```bash
   sudo service jenkins stop
   ```

2. 备份内容

   ```bash
   sudo mv /var/lib/docker /var/lib/docker_data
   ```

   ```bash
   sudo mv /var/lib/jenkins /var/lib/jenkins_data
   ```

3. 创建空目录

   在新分区的数据盘创建空目录

   ```bash
   sudo mkdir /data/docker
   ```

   ```bash
   sudo mkdir /data/jenkins
   ```

4. 链接目录

   ```bash
   sudo ln -s /data/docker/ /var/lib/
   ```

   ```bash
   sudo ln -s /data/jenkins/ /var/lib/
   ```

5. 恢复备份的内容

   ```bash
   sudo mv /var/lib/docker_data/* /var/lib/docker
   ```

   ```bash
   sudo mv /var/lib/jenkins_data/* /var/lib/jenkins
   ```

6. 启动服务

   ```bash
   sudo service docker start
   ```

   ```bash
   sudo service jenkins start
   ```

7. 查看



#### 安装kubectl



#### Jenkins执行shell权限问题

http://www.cnblogs.com/twyth/articles/8377028.html

##### 修改Jenkins配置文件

```bash
# 打开配置文件
vi /etc/default/jenkins
# 修改$JENKINS_USER，并去掉当前行注释
$JENKINS_USER="root"
```

##### 修改Jenkins相关文件夹用户权限

```bash
chown -R root:root /var/lib/jenkins
chown -R root:root /var/cache/jenkins
chown -R root:root /var/log/jenkins
```

##### 重启Jenkins服务并检查运行Jenkins的用户是否已经切换为root

```bash
# 重启Jenkins（若是其他方式安装的jenkins则重启方式略不同）
service jenkins restart
# 查看Jenkins进程所属用户
ps -ef | grep jenkins
# 若显示为root用户，则表示修改完成
```

##### Jenkins服务问题

```bash
# daemon: fatal: refusing to execute unsafe program: /usr/bin/java (/etc is group and world writable)-增加etc权限
chmod -R 755 /etc    
```

```bash
# mesg: ttyname failed: Inappropriate ioctl for device
# 以root用户登陆，打开命令行终端
vim /root/.profile
# 找到.profile文件中的mesg n
# 将其替换成tty -s && mesg n
# 重启ubuntu，问题解决
```

#### Govendor

govendor只是用来管理项目的依赖包，如果GOPATH中本身没有项目的依赖包，则需要通过go get先下载到GOPATH中，再通过govendor add +external 拷贝到vendor目录中。

http://www.huweihuang.com/article/golang/govendor-usage/



####  Golang.X

> The easiest way to install is to run 
>
>`go get -u golang.org/x/tools/...`. 
>
>You can also manually git clone the repository to 
>
>`$GOPATH/src/golang.org/x/tools`.
>
>由于[golang.org](https://link.jianshu.com/?t=http://golang.org)在国内访问不了，所以上述扩展会安装失败，我们需要使用[github.com](https://link.jianshu.com/?t=http://github.com)上的 [https://github.com/golang/tools](https://link.jianshu.com/?t=https://github.com/golang/tools) 代替 [https://golang.org/x/tools](https://link.jianshu.com/?t=https://golang.org/x/tools)，具体操作方法如下：
>
>```bash
>go get -u -v github.com/golang/tools/cmd/gorename
>go get -u -v github.com/golang/tools/cmd/guru
>go get -u -v github.com/golang/tools/cmd/godoc
>```
>
>建立软链接：
>
>```bash
>ln -s $GOPATH/src/github.com/golang $GOPATH/golang.org/golang
>cd $GOAPTH/src/golang.org
>mv golang x
>```



#### 卸载Golang

> http://docs.studygolang.com/doc/install#uninstall
>
> To remove an existing Go installation from your system delete the `go` directory. This is usually `/usr/local/go`under Linux, Mac OS X, and FreeBSD or `c:\Go` under Windows.
>
> You should also remove the Go `bin` directory from your `PATH` environment variable. Under Linux and FreeBSD you should edit `/etc/profile` or `$HOME/.profile`. If you installed Go with the [Mac OS X package](http://docs.studygolang.com/doc/install#osx) then you should remove the `/etc/paths.d/go` file. Windows users should read the section about [setting environment variables under Windows](http://docs.studygolang.com/doc/install#windows_env).

#### go get vs git clone

https://blog.csdn.net/qq_22038327/article/details/80340024

go get 的参数说明：

```
-d 只下载不安装
-f 只有在你包含了-u参数的时候才有效，
   不让-u去验证import中的每一个都已经获取了，
   这对于本地fork的包特别有用
-fix 在获取源码之后先运行fix，然后再去做其他的事情
-t 同时也下载需要为运行测试所需要的包
-u 强制使用网络去更新包和它的依赖包
-v 显示执行的命令
123456789
所以，go get 不仅仅是下载咯？
```

#### Git clone

指定用户名和密码获取代码库代码:

```bash
git clone git://username:password@github
```

for example:

```bash
git clone git@github.com:deepglint/dgc-infra.git
git clone https://haiyuzhang-deepglint:haW-yEg-R8J-A9v@github.com/deepglint/dgc-infra.git
```

#### Git SSH

asdf

sdf



#### Govendor fetch blocked packages

| Path                                              | Origin                                              |
| ------------------------------------------------- | --------------------------------------------------- |
| golang.org/x/crypto/ssh/terminal                  | github.com/golang/crypto/ssh/terminal               |
| golang.org/x/sys/unix                             | github.com/golang/sys/unix                          |
| golang.org/x/sys/windows                          | github.com/golang/sys/windows                       |
| golang.org/x/sys/windows/registry                 | github.com/golang/sys/windows/registry              |
| golang.org/x/sys/windows/svc/eventlog             | github.com/golang/sys/windows/svc/eventlog          |
| golang.org/x/net/context                          | github.com/golang/net/context                       |
| golang.org/x/net/http/httpguts                    | github.com/golang/net/http/httpguts                 |
| golang.org/x/net/http2                            | github.com/golang/net/http2                         |
| golang.org/x/net/http2/hpack                      | github.com/golang/net/http2/hpack                   |
| golang.org/x/net/idna                             | github.com/golang/net/idna                          |
| golang.org/x/net/internal/timeseries              | github.com/golang/net/internal/timeseries           |
| golang.org/x/net/trace                            | github.com/golang/net/trace                         |
| golang.org/x/text/secure/bidirule                 | github.com/golang/text/secure/bidirule              |
| golang.org/x/text/transform                       | github.com/golang/text/transform                    |
| golang.org/x/text/unicode/bidi                    | github.com/golang/text/unicode/bidi                 |
| golang.org/x/text/unicode/norm                    | github.com/golang/text/unicode/norm                 |
| google.golang.org/genproto/googleapis/rpc/status  | github.com/google/go-genproto/googleapis/rpc/status |
| google.golang.org/grpc                            | github.com/grpc/grpc-go                             |
| google.golang.org/grpc/balancer                   | github.com/grpc/grpc-go/balancer                    |
| github.com/grpc/grpc-go/balancer/base             | github.com/grpc/grpc-go/balancer/base               |
| google.golang.org/grpc/balancer/roundrobin        | github.com/grpc/grpc-go/balancer/roundrobin         |
| google.golang.org/grpc/channelz                   | github.com/grpc/grpc-go/channelz                    |
| google.golang.org/grpc/codes                      | github.com/grpc/grpc-go/codes                       |
| google.golang.org/grpc/connectivity               | github.com/grpc/grpc-go/connectivity                |
| google.golang.org/grpc/credentials                | github.com/grpc/grpc-go/credentials                 |
| google.golang.org/grpc/encoding                   | github.com/grpc/grpc-go/encoding                    |
| google.golang.org/grpc/encoding/proto             | github.com/grpc/grpc-go/encoding/proto              |
| google.golang.org/grpc/grpclb/grpc_lb_v1/messages | github.com/grpc/grpc-go/grpclb/grpc_lb_v1/messages  |
| google.golang.org/grpc/grpclog                    | github.com/grpc/grpc-go/grpclog                     |
| google.golang.org/grpc/internal                   | github.com/grpc/grpc-go/internal                    |
| google.golang.org/grpc/internal/grpcrand          | github.com/grpc/grpc-go/internal/grpcrand           |
| google.golang.org/grpc/keepalive                  | github.com/grpc/grpc-go/keepalive                   |
| google.golang.org/grpc/metadata                   | github.com/grpc/grpc-go/metadata                    |
| google.golang.org/grpc/naming                     | github.com/grpc/grpc-go/naming                      |
| google.golang.org/grpc/peer                       | github.com/grpc/grpc-go/peer                        |
| google.golang.org/grpc/resolver                   | github.com/grpc/grpc-go/resolver                    |
| google.golang.org/grpc/resolver/dns               | github.com/grpc/grpc-go/resolver/dns                |
| google.golang.org/grpc/resolver/passthrough       | github.com/grpc/grpc-go/resolver/passthrough        |
| google.golang.org/grpc/stats                      | github.com/grpc/grpc-go/stats                       |
| google.golang.org/grpc/status                     | github.com/grpc/grpc-go/status                      |
| google.golang.org/grpc/tap                        | github.com/grpc/grpc-go/tap                         |
| google.golang.org/grpc/transport                  | github.com/grpc/grpc-go/transport                   |
|                                                   |                                                     |



#### build遗留问题

1. dgc-infra没有使用govendor

   Dgc-infra/dgcauth/dgctoken.go:70/71 引用的github.com/applyboy/gin-jwt 没有版本管理.

   src/controller/accountController.go:22:24: cannot use c (type *"github.com/deepglint/godeye/src/vendor/github.com/gin-gonic/gin".Context) as type *"github.com/gin-gonic/gin".Context in argument to dgcauth.LoginInterface
   src/controller/accountController.go:28:22: cannot use c (type *"github.com/deepglint/godeye/src/vendor/github.com/gin-gonic/gin".Context) as type *"github.com/gin-gonic/gin".Context in argument to dgcauth.RefreshToken

2. dgc_event_pusher中引用的github.com/gin-gonic/gin 和 dgc-infra中引用的github.com/gin-gonic/gin 版本不一致

   libs/ws/hub.go:80:44: cannot use context (type *"github.com/deepglint/dgc_event_pusher/vendor/github.com/gin-gonic/gin".Context) as type *"github.com/gin-gonic/gin".Context in argument to dgcauth.CheckSignatureImpl

3. 合库解决以上问题. 在一个库下管理所有外部依赖, 内部依赖则完全一致.

#### 待解决问题

1. 添加软连接, 来把docker, jenkins放到数据盘上.
2. 一键部署到测试和生产环境.
3. 实现特定编译后的文件生成docker镜像, 推送到镜像仓库, 一键部署到特定的环境(生产环境和测试环境)
4. 申请镜像仓库deploy账号 (刘彤)