# ip2region xdb C++ 生成实现

# 编译
1. 切换到当前目录
2. 编译

```
$ make
g++ -std=c++11 -O2 xdb_make.cc xdb_make_test.cc -o xdb_make
```

# `xdb` 数据生成
## 使用说明
```
$ ./xdb_make --help
./xdb_make [command options]
options:
 --db string        ip2region binary xdb file path
 --src string       source ip text file path
```

## 数据生成
```
$ ./xdb_make --db ip2region.xdb --src ../../data/ip.merge.txt
took: 1.46s
```

## 数据正确性测试
```
$ make                                                        # 1. 编译
$ ./xdb_maker                                                 # 2. 本目录生成 xdb 文件
$ diff <(xxd ./ip2region.xdb) <(xxd ../../data/ip2region.xdb) # 3. 比较本目录和仓库中的 xdb 文件
                                                              #    只有生成的时间不同
1c1
< 00000000: 0200 0100 3c6a f965 2302 0f00 75ea a800  ....<j.e#...u...
---
> 00000000: 0200 0100 469b de65 2302 0f00 75ea a800  ....F..e#...u...
```

# `xdb` 数据编辑
## 使用说明
* 新的IP归属地文件可以包含空行
* 新的IP归属地文件顺序可以乱序, 程序会自动排序
* 新的IP归属地文件顺序可以重叠, 只要无二义性, 程序会自动合并
* 最终的结果会将相邻的且归属地相同的行自动合并

```
$ ./xdb_edit --help
./xdb_edit [command options]
options:
 --old filename       old source ip text file path
 --new filename       new source ip text file path
```

## 数据更新
```
$ ./xdb_edit --old ../../data/ip.merge.txt --new 1.txt
took: 1.46s
```

## 数据正确性测试
### 测试一: 测试数据文件包含空行以及重复的情况
```
$ cat -n 1.txt
     1
     2	1.0.128.0|1.0.128.255|测试归属地
     3	1.0.128.0|1.0.128.255|测试归属地
     4
$ ./xdb_edit --old ../../data/ip.merge.txt --new 1.txt
took: 1.83s
$ git diff ../../data/
diff --git a/data/ip.merge.txt b/data/ip.merge.txt
index 8976bd3..6da5e18 100644
--- a/data/ip.merge.txt
+++ b/data/ip.merge.txt
@@ -7,7 +7,7 @@
 1.0.32.0|1.0.63.255|中国|0|广东省|广州市|电信
 1.0.64.0|1.0.79.255|日本|0|广岛县|0|0
 1.0.80.0|1.0.127.255|日本|0|冈山县|0|0
-1.0.128.0|1.0.128.255|泰国|0|清莱府|0|TOT
+1.0.128.0|1.0.128.255|测试归属地
 1.0.129.0|1.0.132.191|泰国|0|曼谷|曼谷|TOT
 1.0.132.192|1.0.132.255|泰国|0|Nakhon-Ratchasima|0|TOT
 1.0.133.0|1.0.133.255|泰国|0|素攀武里府|0|TOT
@@ -320906,8 +320906,7 @@
 100.47.160.0|100.47.191.255|美国|0|密歇根|0|美国电话电报
 100.47.192.0|100.47.255.255|美国|0|0|0|美国电话电报
 100.48.0.0|100.63.255.255|美国|0|0|0|Sprint
-100.64.0.0|100.122.255.255|0|0|0|内网IP|内网IP
-100.123.0.0|100.127.255.255|0|0|0|内网IP|内网IP
+100.64.0.0|100.127.255.255|0|0|0|内网IP|内网IP
 100.128.0.0|100.255.255.255|美国|0|0|0|T-Mobile
 101.0.0.0|101.0.3.255|中国|0|福建省|福州市|电信
 101.0.4.0|101.0.7.255|印度尼西亚|0|东爪哇|泗水|0
```

### 测试二: 测试数据文件乱序以及数据有交叉, 归属地相同的情况
```
$ cat -n 1.txt
     1
     2	1.0.128.5|1.0.128.255|测试归属地
     3	1.0.128.0|1.0.128.9|测试归属地
     4
$ ./xdb_edit --old ../../data/ip.merge.txt --new 1.txt
took: 1.83s
$ git diff ../../data/
diff --git a/data/ip.merge.txt b/data/ip.merge.txt
index 8976bd3..6da5e18 100644
--- a/data/ip.merge.txt
+++ b/data/ip.merge.txt
@@ -7,7 +7,7 @@
 1.0.32.0|1.0.63.255|中国|0|广东省|广州市|电信
 1.0.64.0|1.0.79.255|日本|0|广岛县|0|0
 1.0.80.0|1.0.127.255|日本|0|冈山县|0|0
-1.0.128.0|1.0.128.255|泰国|0|清莱府|0|TOT
+1.0.128.0|1.0.128.255|测试归属地
 1.0.129.0|1.0.132.191|泰国|0|曼谷|曼谷|TOT
 1.0.132.192|1.0.132.255|泰国|0|Nakhon-Ratchasima|0|TOT
 1.0.133.0|1.0.133.255|泰国|0|素攀武里府|0|TOT
@@ -320906,8 +320906,7 @@
 100.47.160.0|100.47.191.255|美国|0|密歇根|0|美国电话电报
 100.47.192.0|100.47.255.255|美国|0|0|0|美国电话电报
 100.48.0.0|100.63.255.255|美国|0|0|0|Sprint
-100.64.0.0|100.122.255.255|0|0|0|内网IP|内网IP
-100.123.0.0|100.127.255.255|0|0|0|内网IP|内网IP
+100.64.0.0|100.127.255.255|0|0|0|内网IP|内网IP
 100.128.0.0|100.255.255.255|美国|0|0|0|T-Mobile
 101.0.0.0|101.0.3.255|中国|0|福建省|福州市|电信
 101.0.4.0|101.0.7.255|印度尼西亚|0|东爪哇|泗水|0
```

### 测试三: 测试数据文件乱序以及数据有交叉的, 归属地不同情况
```
$ cat -n 1.txt
     1
     2	1.0.128.5|1.0.128.255|测试归属地
     3	1.0.128.0|1.0.128.9|测试归属地123
     4
$ ./xdb_edit --old ../../data/ip.merge.txt --new 1.txt
数据有二义性: 1.0.128.0|1.0.128.9|测试归属地123, 1.0.128.5|1.0.128.255|测试归属地
```

### 测试四: 测试将一个IP数据拆成多个IP
```
$ cat -n 1.txt
     1	36.136.1.0|36.136.7.255|中国|0|广西|来宾市|移动
     2	36.136.8.0|36.136.15.255|中国|0|广西|玉林市|移动
     3	36.136.16.0|36.136.23.255|中国|0|广西|河池市|移动
$ ./xdb_edit --old ../../data/ip.merge.txt --new 1.txt
took: 1.83s
$ git diff ../../data/
diff --git a/data/ip.merge.txt b/data/ip.merge.txt
index 8976bd3..7be0227 100644
--- a/data/ip.merge.txt
+++ b/data/ip.merge.txt
@@ -54778,7 +54778,11 @@
 36.134.84.0|36.134.85.255|中国|0|安徽省|合肥市|移动
 36.134.86.0|36.134.87.255|中国|0|广西|南宁市|移动
 36.134.88.0|36.134.89.255|中国|0|内蒙古|呼和浩特市|移动
-36.134.90.0|36.141.255.255|中国|0|0|0|移动
+36.134.90.0|36.136.0.255|中国|0|0|0|移动
+36.136.1.0|36.136.7.255|中国|0|广西|来宾市|移动
+36.136.8.0|36.136.15.255|中国|0|广西|玉林市|移动
+36.136.16.0|36.136.23.255|中国|0|广西|河池市|移动
+36.136.24.0|36.141.255.255|中国|0|0|0|移动
 36.142.0.0|36.142.1.255|中国|0|四川省|成都市|移动
 36.142.2.0|36.142.31.255|中国|0|甘肃省|兰州市|移动
 36.142.32.0|36.142.127.255|中国|0|甘肃省|0|移动
@@ -320906,8 +320910,7 @@
 100.47.160.0|100.47.191.255|美国|0|密歇根|0|美国电话电报
 100.47.192.0|100.47.255.255|美国|0|0|0|美国电话电报
 100.48.0.0|100.63.255.255|美国|0|0|0|Sprint
-100.64.0.0|100.122.255.255|0|0|0|内网IP|内网IP
-100.123.0.0|100.127.255.255|0|0|0|内网IP|内网IP
+100.64.0.0|100.127.255.255|0|0|0|内网IP|内网IP
 100.128.0.0|100.255.255.255|美国|0|0|0|T-Mobile
 101.0.0.0|101.0.3.255|中国|0|福建省|福州市|电信
 101.0.4.0|101.0.7.255|印度尼西亚|0|东爪哇|泗水|0
 ```

### 测试五: 测试将多个IP数据并成一个IP数据
```
$ cat -n 1.txt
     1
     2	1.0.16.0|1.0.127.255|测试归属地
     3
$ ./xdb_edit --old ../../data/ip.merge.txt --new 1.txt
took: 1.83s
$ git diff ../../data/
diff --git a/data/ip.merge.txt b/data/ip.merge.txt
index 8976bd3..acc27a5 100644
--- a/data/ip.merge.txt
+++ b/data/ip.merge.txt
@@ -3,10 +3,7 @@
 1.0.1.0|1.0.3.255|中国|0|福建省|福州市|电信
 1.0.4.0|1.0.7.255|澳大利亚|0|维多利亚|墨尔本|0
 1.0.8.0|1.0.15.255|中国|0|广东省|广州市|电信
-1.0.16.0|1.0.31.255|日本|0|0|0|0
-1.0.32.0|1.0.63.255|中国|0|广东省|广州市|电信
-1.0.64.0|1.0.79.255|日本|0|广岛县|0|0
-1.0.80.0|1.0.127.255|日本|0|冈山县|0|0
+1.0.16.0|1.0.127.255|测试归属地
 1.0.128.0|1.0.128.255|泰国|0|清莱府|0|TOT
 1.0.129.0|1.0.132.191|泰国|0|曼谷|曼谷|TOT
 1.0.132.192|1.0.132.255|泰国|0|Nakhon-Ratchasima|0|TOT
@@ -320906,8 +320903,7 @@
 100.47.160.0|100.47.191.255|美国|0|密歇根|0|美国电话电报
 100.47.192.0|100.47.255.255|美国|0|0|0|美国电话电报
 100.48.0.0|100.63.255.255|美国|0|0|0|Sprint
-100.64.0.0|100.122.255.255|0|0|0|内网IP|内网IP
-100.123.0.0|100.127.255.255|0|0|0|内网IP|内网IP
+100.64.0.0|100.127.255.255|0|0|0|内网IP|内网IP
 100.128.0.0|100.255.255.255|美国|0|0|0|T-Mobile
 101.0.0.0|101.0.3.255|中国|0|福建省|福州市|电信
 101.0.4.0|101.0.7.255|印度尼西亚|0|东爪哇|泗水|0
 ```
