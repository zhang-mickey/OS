# OS

# 配置环境
## vscode 下载 Dev Container 插件
![image](https://github.com/user-attachments/assets/2d24b17a-a2f3-4efc-846e-87826ab45e23)

## 构建LLVM 容器
创建新文件夹
在根目录下，创建一个 .devcontainer 文件夹，并在其中创建 devcontainer.json 文件

'''
mkdir .devcontainer
touch .devcontainer/devcontainer.json

'''
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
    "remoteUser": "root"
}
```

### 构建自定义容器镜像
在 .devcontainer 文件夹下创建一个 Dockerfile：
```
# 使用基础镜像
FROM ubuntu:20.04

# 安装 LLVM 和 Clang
RUN apt-get update && apt-get install -y llvm clang

# 设置工作目录
WORKDIR /workspace

# 终端使用 bash
CMD ["/bin/bash"]
```

![image](https://github.com/user-attachments/assets/074df335-c867-464d-ba88-632facf048cc)


命令行找到 Dev Container：reopen in container 
