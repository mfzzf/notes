## set

`set -euo pipefail` 是一个常见的 Bash 脚本设置，用于提高脚本的健壮性和错误处理能力。具体解释如下：

- `set -e`: 当命令返回非零状态时，立即退出脚本。
- `set -u`: 当脚本中使用未定义的变量时，立即退出脚本。
- `set -o pipefail`: 确保整个管道命令返回的状态码是最后一个失败的命令的状态码，而不是最后一个命令的状态码。

## $0

```shell
if [[ "$0" != ~/scripts/genproto.sh ]]; then
    echo "Please run this script from the root of the repository"
    exit 255
fi
```

1. `if ![["$0" = ~/scripts/genproto.sh]]; then`:
   - `if` 语句用于条件判断。
   - `!` 表示取反，即条件不成立时执行后续代码。
   - `[["$0" = ~/scripts/genproto.sh]]` 试图检查脚本的路径是否为 [genproto.sh](vscode-file://vscode-app/c:/Users/mei17/AppData/Local/Programs/Microsoft VS Code/resources/app/out/vs/code/electron-sandbox/workbench/workbench.html)，但语法错误。
2. `echo "Please run this script from the root of the repository"`:
   - 如果条件成立，输出提示信息，要求从仓库根目录运行脚本。
3. `exit 255`:
   - 退出脚本，并返回状态码 255。
4. `fi`:
   - 结束 `if` 语句。

`$0` 是一个特殊变量，用于表示当前脚本的名称或路径。在脚本中使用 `$0` 可以获取脚本是如何被调用的，包括它的路径。

## globstar

`shopt -s globstar` 是一个 Bash 命令，用于启用 `globstar` 选项。启用该选项后，`**` 通配符可以递归匹配目录中的所有文件和子目录。例如：

```shell
shopt -s globstar
echo **/*.txt
```

这将匹配当前目录及其所有子目录中的所有 `.txt` 文件。

## source

`source ./script/lib.sh` 命令用于在当前脚本中执行 `./script/lib.sh` 文件中的所有命令。`source` 命令会在当前 shell 环境中运行指定的脚本，而不是在子 shell 中运行，这意味着 `lib.sh` 中定义的变量和函数将会在当前脚本中可用。