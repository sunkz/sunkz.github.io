title: Shell In 5 Minutes
author: Sunkz
tags:
  - shell
  - linux

photos:

  - https://s2.ax1x.com/2019/11/02/Kb6Knf.png

categories: []
date: 2019-01-22 00:26:00

---
### 变量

```shell
A=2 #定义变量A
unset A #撤销变量A
readOnly B="skz" #只读常量B , 不能unset
export B #将B提升额为全部变量 , 可在.sh文件中$B 调用
```

### 传参

```shell
$n $# #传参 入参总个数
-------------------
#!/bin/bash
echo $1 $2
echo $#
--------------------
bash a.sh skz wxy
skz wxy
2
```

### 运算

```shell
$[运算式] #  计算求和
-------------------
#!/bin/bash
sum=$[$1+$2]
echo $sum
--------------------
bash a.sh 3 2
5
```

### 条件

```
两个整数之间比较
-lt 小于（less than）	-le 小于等于（less equal）   -eq 等于（equal）	
-gt 大于（greater than）   -ge 大于等于（greater equal）	-ne 不等于（Not equal）

按照文件权限进行判断
-r 有读的权限（read）	-w 有写的权限（write）  -x 有执行的权限（execute）

按照文件类型进行判断
-f 文件存在并且是一个文件（file） -e 文件存在（existence） -d 文件存在并是一个目录（directory）
```

```shell
[ condition ] # condition前后有空格
--------------------------
[ 16 -lt 10 ]
--------------------------
[ -w a.txt ] && [ -e b.txt ]
```

### 流程

```shell
#!/bin/bash
if [ $1 -eq "1" ]
then
        echo "skz"
elif [ $1 -eq "2" ]
then
        echo "wxy"
fi
```

```shell
!/bin/bash
case $1 in
"1")
        echo "skz"
;;
"2")
        echo "wxy"
;;
*)
        echo "xyz"
;;
esac
```

```shell
#!/bin/bash
sum=0
for((i=0;i<=$#;i++))
	do
        sum=$[$sum+$i]
	done
echo $sum
-------------------------
bash a.sh 1 2 3
6
-------------------------
#!/bin/bash 
for j in $@
do      
        echo "$j"
done
---------------------
bash a.sh 1 2
1
2
```

```shell
#!/bin/bash
s=0
i=1
while [ $i -le 100 ]
do
        s=$[$s+$i]
        i=$[$i+1]
done
echo $s
----------------------
bash a.sh
5050
```

### Read

```shell
#!/bin/bash
read -t 5 -p "Enter your name in 5 seconds : " NAME
echo $NAME
-----------------------
Enter your name in 5 seconds skz
skz
```

### 函数

```shell
#!/bin/bash
function sum()
{
    s=0
    s=$[ $1 + $2 ]
    echo "$s"
}

read -p "Please input the number1: " n1;
read -p "Please input the number2: " n2;
sum $n1 $n2;
--------------------------------------------
bash a.sh 
Please input the number1: 2
Please input the number2: 5
7
```

### grep

```shell
grep -E -n "java|python" a.txt # 输出所有带java或python的行且显示所在的行号
```

### Cut

```
s;w
k;x
z;;y
```

```shell
cut -d ";" -f 2,3 a.txt # 以;为分割符, 切取第2,3列
w
x
y
```

### Sed

```shell
sed -e '1a b;b' -e '3d' -e 's/s/w/g' a.txt >b.txt #第一行增加 b;b 删除第三行 s替换为w  /g全文替换
w;w
b;b
k;x
-------------------------------------
sed -i '' 's/oldStr/newStr/' a.txt # mac os 需要加上 ‘’
sed -i 's/oldStr/newStr/' a.txt #centos不需要
sed -i '/skz/d' # 删除含有skz的行
sed -i '/^ *$/d' # 删除所有空行
-----------------------------------------
sed -n '/str/p' file_name > a.txt  # 查找含有str的行信息存入 a.txt
```

### awk

```shell
a 3 t
e 4 t
e 3 t
e 1 y
--------------
awk '{print $1 $3}'  #输出第一列 第三列 默认分隔符空格 -f ‘’指定分隔符
awk '$1=="e" && $2=="t"' #输出第一列=e 第三列=t
```
