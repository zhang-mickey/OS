# OS

# 配置环境
## vscode 下载 Dev Container 插件
![image](https://github.com/user-attachments/assets/2d24b17a-a2f3-4efc-846e-87826ab45e23)

## 构建LLVM 容器
创建新文件夹
在根目录下，创建一个 .devcontainer 文件夹，并在其中创建 devcontainer.json 文件

```
mkdir .devcontainer
touch .devcontainer/devcontainer.json

```
在 devcontainer.json 中，添加如下配置，以便使用带有 LLVM 工具链的开发容器：
```
{
    "name": "Custom LLVM Container",
    "build": {
        "dockerfile": "Dockerfile",
        "context": "."
    },
    "settings": {
        "terminal.integrated.shell.linux": "/bin/bash"
    },
    "extensions": [
        "ms-vscode.cpptools",
        "llvm-vs-code-extensions.vscode-clangd"
    ],
    "workspaceFolder": "/workspace",
    "remoteUser": "root",
    "mounts": [
        "source=${localWorkspaceFolder}/aos-labs-2024-main,target=/workspace,type=bind,consistency=cached"
    ],
    ###通过 mounts 属性来指定将本地的 OS 文件夹挂载到容器中的某个路径。比如将本地的代码文件夹挂载到容器中的 /workspace 目录
    "postCreateCommand": "apt-get update && apt-get install -y make"
}
```

### 构建自定义容器镜像
在 .devcontainer 文件夹下创建一个 Dockerfile：
```
# 使用基础镜像
FROM ubuntu:20.04

# 安装 LLVM 和 Clang
#RUN apt-get update && apt-get install -y llvm clang
# 更新包列表并安装基本开发工具

RUN apt-get update && apt-get install -y \
    build-essential \
    llvm \
    clang \
    gcc \
    g++ \
    libc6-dev \
    make \
    wget \
    git \
    curl

# 设置工作目录
WORKDIR /workspace

# 终端使用 bash
CMD ["/bin/bash"]
```
命令行找到 Dev Container：reopen in container 

### 验证挂载
这样，你就可以在容器中直接访问和操作本地文件，并且所有更改都会反映在本地文件系统中。
```
cd /workspace
ls
```

### 查看llvm版本(>=12)
```
clang --version
```
# 
构建和启动 QEMU 模拟器 用于在我们机器上(X86)模拟RISC-V架构的CPU
```
make qemu-nox
```
报错
```
make qemu-nox
***
*** Error: Couldn't find an i386-*-elf version of GCC/binutils.
*** Is the directory with i386-jos-elf-gcc in your PATH?
*** If your i386-*-elf toolchain is installed with a command
*** prefix other than 'i386-jos-elf-', set your CROSS_COMPILE
*** environment variable to that prefix and run 'make' again.
*** To turn off this error, run 'gmake CROSS_COMPILE= ...'.
***
```

解决
### cross-compiler toolchain for i386-elf
```
apt-get update
apt-get install build-essential bison flex libgmp-dev libmpc-dev libmpfr-dev autoconf2.13 texinfo libncurses5-dev
```
Download Binutils Source:
```
mkdir -p ~/cross
cd ~/cross
wget https://ftp.gnu.org/gnu/binutils/binutils-2.37.tar.gz
tar -xzvf binutils-2.37.tar.gz
cd binutils-2.37
```
Build and Install Binutils:
```
mkdir build
cd build
../configure --target=i386-elf --prefix=$HOME/cross --disable-nls
make
make install
```
1、configure，这一步一般用来生成 Makefile，为下一步的编译做准备，可以通过在 configure 后加上参数来对安装进行控制

2、make，这一步就是编译，既然configure已经生成了Makefile，那make就可以直接进行编译了。大多数的源代码包都经过这一步进行编译
3、make insatll，这条命令来进行安装
make clean：清除编译产生的可执行文件及目标文件(object file，*.o)，也就是说make之后再来一条make clean就等于没有make过，这点在make出错时变得极为有用。

4线程的make吧，make -j4可以让你感受4倍cpu的畅快


Download GCC Source:
```
cd ~/cross
wget https://ftp.gnu.org/gnu/gcc/gcc-10.4.0/gcc-10.4.0.tar.gz
tar -xzvf gcc-10.4.0.tar.gz
cd gcc-10.4.0
./contrib/download_prerequisites
```
Build and Install GCC:
```
cd ~/cross
mkdir gcc-build
cd gcc-build
../gcc-10.4.0/configure --target=i386-elf --prefix=$HOME/cross --with-sysroot=$sysrootdir --enable-languages=c --disable-nls --disable-libssp --disable-libmudflap --disable-sjlj-exceptions --disable-multilib
make -j4
make install
```
Update Your PATH
```
export PATH=$HOME/cross/bin:$PATH
```
can add this line to your .bashrc or .zshrc file to make the change permanent:

```
echo 'export PATH=$HOME/cross/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

Verify the Installation
```
i386-elf-gcc --version
i386-elf-as --version
i386-elf-ld --version
```



![image](https://github.com/user-attachments/assets/074df335-c867-464d-ba88-632facf048cc)


