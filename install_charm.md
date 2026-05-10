# 如何在Ubuntu 24.04中配置好python的配对加密库(`Charm-Crypto`)
## 关于uv的包管理
uv是python最新的一个包管理工具，主要是用来建立虚拟环境，可以达到项目即环境的目的，相较于传统的conda，可以更好的隔离项目环境，同时 `uv pip install` 相较于传统的 `pip install` 安装速度更快。
### uv常用指令
#### 项目初始化 (`uv init`)
```python
uv init #在当前目录初始化一个应用 (application) 项目
uv init <项目名> #显式指定创建一个应用项目（等同于默认行为）
uv init --lib #初始化一个库 (library) 项目，用于开发被其他项目引用的包 
uv init --package #初始化一个打包的应用 (packaged application)，使用 src/ 目录布局，便于发布到 PyPI 
```
初始化后的项目结构（以默认应用为例）：
```text
my-project/
├── .gitignore          # Git 忽略文件
├── .python-version     # 锁定项目 Python 版本
├── README.md           # 项目说明文档
├── main.py             # 主程序入口文件
└── pyproject.toml      # 项目核心配置文件
```
#### 依赖管理 (`uv add, uv remove, uv sync`)
uv 通过 pyproject.toml 和 uv.lock 文件来管理项目依赖，确保环境一致性。
```python
uv add <包名> #添加一个或多个依赖包，自动更新 pyproject.toml 并生成/更新 uv.lock 锁文件
uv add --dev <包名>	#添加仅在开发环境需要的依赖，如 pytest、ruff 等
uv remove <包名> #移除项目依赖
uv sync	#核心命令。根据 pyproject.toml 和 uv.lock 文件，同步安装所有依赖到项目的虚拟环境中
uv tree	#以树形结构展示项目的全部依赖关系
```
#### 运行与执行 (`uv run`)
uv run 让你无需手动激活虚拟环境即可执行命令，它会自动确保环境就绪。
```python
uv run python <脚本.py>	#在项目环境中运行指定的 Python 脚本 
uv run <命令>	#在项目环境中运行任何系统命令，例如 uv run pytest 或 uv run ruff check 
```
#### Python 版本管理 (`uv python`)
uv 可以像 pyenv 一样管理和安装不同的 Python 版本。
```python
uv python list #列出所有可安装的 Python 版本
uv python install <版本> #安装指定版本的 Python，如 uv python install 3.11 3.12
uv python pin <版本> #为当前项目固定使用的 Python 版本，并写入 .python-version 文件
uv python find #查看当前项目正在使用的 Python 解释器路径 
```
#### 保留命令（`uv pip`等）
为了照顾旧有习惯或特定场景，uv 也提供了兼容性指令。
```python
uv pip	#提供与经典 pip 工具兼容的接口，可直接使用 requirements.txt 文件。
uv venv	#创建一个独立的虚拟环境，类似于 python -m venv。
```

## 配置配对环境库（`Charm-Crypto`）
### 准备编译环境
首先，确保你的系统已安装必要的编译工具和依赖包:
- 基础编译工具链：这包括了GCC、G++和Make。
  ```bash
    sudo apt update
    sudo apt install -y build-essential  # 安装 gcc, g++, make 等核心工具
  ```

- 编译依赖库：安装Charm编译过程中需要用到的一些库。
  ```bash
    sudo apt install -y m4 flex bison libssl-dev libgmp-dev
  ```

- 手动安装`libpbc-dev`
  ```bash
    wget https://crypto.stanford.edu/pbc/files/pbc-0.5.14.tar.gz
    tar -xzf pbc-0.5.14.tar.gz
    cd pbc-0.5.14

    ./configure
    make
    sudo make install
    sudo ldconfig
  ```

- 在虚拟环境中安装 Python 依赖：
  由于Charm 要求特定版本的 pyparsing，因此额外执行`uv add pyparsing` 即可。

- 安装 Charm
  ```bash
    git clone https://github.com/JHUISI/charm.git
    cd charm
    ./configure.sh #checking for PBC... yes  checking for GMP... yes
    make
    sudo make install
  ```

- 验证是否成功
  ```python
    from charm.toolbox.pairinggroup import PairingGroup, G1, pair

    group = PairingGroup('SS512')
    g = group.random(G1)
    a = group.random()
    b = group.random()

    print(pair(g ** a, g ** b))
  ```

## 由于我的疏忽，其实已经有更新的框架了
后面安装可以直接用
```python
uv pip install charm-crypto-framework
```