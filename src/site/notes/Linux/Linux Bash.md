---
{"dg-publish":true,"permalink":"/linux/linux-bash/","title":"Linux Base","created":"2024-01-04T21:29:41.004+08:00","updated":"2024-03-07T11:08:36.303+08:00"}
---

### 1. Shebang（脚本头部）

在脚本的第一行，指定脚本使用的解释器。

```bash
#!/bin/bash
```

这告诉系统使用Bash解释器来执行脚本。

### 2. 变量

在Shell中，变量不需要显式声明类型。变量名可以包含字母、数字和下划线，但不能以数字开头。

```bash
variable_name="你好，世界！"
```

### 3. 输出

使用`echo`命令输出文本。

```bash
echo "这是一条消息。"
```

### 4. 注释

在Shell脚本中，注释以`#`开头。

```bash
# 这是一条注释
```

### 5. 输入

使用`read`命令从用户那里获取输入。

```bash
echo "你叫什么名字？"
read user_name
echo "你好，$user_name！"
```

### 6. 条件语句

使用`if`语句进行条件判断。

```bash
if [ 条件 ]; then
  # 如果条件为真执行的代码
else
  # 如果条件为假执行的代码
fi
```

在条件中，可以使用测试命令如`-eq`（等于）、`-ne`（不等于）、`-lt`（小于）等。

### 7. 循环语句

使用`for`和`while`进行循环。

```bash
# For 循环
for item in 列表; do
  # 对列表中的每个项目执行的代码
done

# While 循环
while [ 条件 ]; do
  # 当条件为真执行的代码
done
```

### 8. 函数

定义和调用函数。

```bash
# 函数定义
function_name() {
  # 要执行的代码
}

# 调用函数
function_name
```

### 9. 退出脚本

使用`exit`命令退出脚本。

```bash
exit 0  # 0表示成功退出，非零表示出现错误
```

### 10. 条件测试

在Shell脚本中，可以使用`test`命令或方括号`[]`进行条件测试。

```bash
# 使用 test 命令
if test 条件; then
  # 如果条件为真执行的代码
fi

# 使用方括号
if [ 条件 ]; then
  # 如果条件为真执行的代码
fi
```

### 11. 处理命令输出

将命令的输出赋值给变量：

```bash
result=$(command)
```

### 12. 条件测试运算符

* `-eq`: 等于
* `-ne`: 不等于
* `-lt`: 小于
* `-le`: 小于等于
* `-gt`: 大于
* `-ge`: 大于等于

### 13. 字符串比较

```bash
string1="你好"
string2="世界"

if [ "$string1" = "$string2" ]; then
  echo "字符串相等。"
else
  echo "字符串不相等。"
fi
```

### 14. 数组

```bash
my_array=("项1" "项2" "项3")

# 遍历数组
for item in "${my_array[@]}"; do
  echo $item
done
```