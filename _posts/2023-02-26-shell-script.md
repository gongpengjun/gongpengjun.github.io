---

layout: post
title: 常用Shell脚本模版
date: 2023-02-26 09:00:00
categories: script
---

经常写shell脚本时忘了某个具体语法怎么写，写个模版备忘。

### 1、参数判断

```shell
#!/bin/bash

script_name=`basename "$0"`
if [ $# -gt 1 ]; then
    echo "USAGE: ./${script_name} {file_name} OR cat | ./${script_name}"
    echo "e.g.: ./${script_name} sample.txt"
    echo "e.g.: cat sample.txt | ./${script_name}"
    exit;
fi
```

说明：

- `$0` 表示脚本文件名

- `$1` 表示第一个参数，以此类推
- `$#` 表示参数个数

### 2、遍历文件

```shell
#!/bin/bash
filename="${1:-/dev/stdin}"
n=1
while read -r line;
do
  echo "line $n : ${line}"
  n=$((n+1))
done < ${filename}
```

说明：

- `${varname:-default}`变量展开 `${1:-/dev/stdin}`表示如果$1不存在，则将"/dev/stdin"赋值给filename
- `read -r line` 参数`-r`表示禁用`\`的转义功能
- `read -a lines` 参数`-a`  读取多行文本并写入lines数组中

输出:

```shell
$ cat os.txt
CentOS 7.2
Ubuntu
Debian
LinuxMint
$ ./read_line_by_line.sh os.txt
line 1 : CentOS 7.2
line 2 : Ubuntu
line 3 : Debian
line 4 : LinuxMint
$ cat os.txt | ./read_line_by_line.sh
line 1 : CentOS 7.2
line 2 : Ubuntu
line 3 : Debian
line 4 : LinuxMint
```

参考：https://linuxhint.com/while_read_line_bash/

### 3、遍历数组

```shell
#!/bin/bash

declare -a array=("element 1"
                  "element 2" "element 3"
                  "element 4")

echo "iterate element only in array:"
for element in "${array[@]}"
do
   echo "${element}"
done

echo "iterate element and index in array:"
length=${#array[@]}
for (( i=0; i<${length}; i++ ));
do
  echo "array[$i]: ${array[$i]}"
done
```

输出：

```
iterate element only in array:
element 1
element 2
element 3
element 4
iterate element and index in array:
array[0]: element 1
array[1]: element 2
array[2]: element 3
array[3]: element 4
```

参考：

- [bash read](https://phoenixnap.com/kb/bash-read)
- [Variable_Expansion](https://en.wikibooks.org/wiki/Bourne_Shell_Scripting/Variable_Expansion)
- https://stackoverflow.com/a/8880633/328435



