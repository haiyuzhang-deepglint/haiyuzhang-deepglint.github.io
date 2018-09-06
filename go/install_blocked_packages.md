### 安装被墙的库

解决方案1:

- 手动从Github查找并下载相应的package

  示例 (下载golang的sys):

  ```bash
  cd $GOPATH/src/github.com/golang
  git clone https://github.com/golang/sys.git
  ```

解决方案2: github上有个库,  可以现在并运行 然后自动下载被墙了的package.

可以写一个脚本去下载所需要的package:

示例:

```bash
mkdir -p $GOPATH/src/golang.org/x/
cd !$
git clone https://github.com/golang/sys.git
git clone https://github.com/golang/tools.git
git clone https://github.com/golang/net.git
```

