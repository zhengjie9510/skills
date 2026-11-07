# Global Rules

代码首先服务于人类阅读，而不是只服务于机器执行；避免晦涩的写法、过度抽象和隐藏逻辑。

## Python 环境

系统已安装 uv。任何涉及 Python 的操作（包括内联命令 `python -c "..."`） **必须**在虚拟环境中执行。

### 强制前置步骤

每次执行 Python 相关命令前， **必须先确认**：

1. 当前项目目录下存在 `.venv` 文件夹
2. 如果不存在， **必须先执行** `uv venv` 创建虚拟环境
3. **然后再执行**实际的 Python 命令

### 唯一允许的调用方式

- 安装包：`uv pip install <package>`（ **必须**使用清华源：`--index-url https://pypi.tuna.tsinghua.edu.cn/simple`）
- 运行脚本：`uv run python script.py`
- 运行内联代码：`uv run python -c "..."`
- 其他 Python 操作：`uv run <command>`

### 绝对禁止

- 禁止直接使用 `python`、`python3`、`pip`、`pip3` 等裸命令
- 禁止在没有 `.venv` 的情况下运行任何 Python 代码
- 禁止使用 `pip install` 而不通过 `uv pip install`
- 禁止跳过虚拟环境创建步骤

## Node.js 环境

1. 项目无 `package.json` 时先执行 `npm init -y`
2. 禁止全局安装：`npm install -g`、`yarn global add`

## Git 提交

Git 提交必须使用用户本人的身份，禁止任何 AI 署名（包括 Author、Committer 和 Co-Authored-By）。
