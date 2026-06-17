---
title: python开发命令行工具argparse中Sub-commands和Argument-groups有什么区别
author: 测开工程师的烦恼
date: 
url: https://mp.weixin.qq.com/s?__biz=MzA5NzM0NjU1Nw==&mid=2647871556&idx=1&sn=8d5e1f017c1d0938002e76c834398893&chksm=8956ff292eabdbaf1bf6230b6fff11f91c8a86607bc8a61b64cc68ae896c3127898c3eab8948&mpshare=1&scene=24&srcid=01165bvOqWPv8uPUXKPcdLpr&sharer_shareinfo=96bbe14007a986c193cd2451497b1265&sharer_shareinfo_first=96bbe14007a986c193cd2451497b1265#rd
---

## 导读

在使用 Python 的 `argparse`模块开发命令行工具时，我们经常会遇到这样的场景：

* 工具功能越来越多，参数也越来越多，所有参数混在一起看起来很乱；
* 想要实现类似 `git commit`、`git push`这样的子命令功能；
* 希望将相关的参数分组显示，让帮助信息更清晰。

这时候，`argparse`提供了两个看起来很相似的功能：**Sub-commands（子命令）**和 **Argument-groups（参数组）**。

很多同学会疑惑：这两个功能有什么区别？什么时候用哪个？

这篇文章我们就来彻底搞懂这两个概念，让你在开发命令行工具时不再纠结。

---

## 一、什么是 Sub-commands（子命令）？

**Sub-commands**就像 `git`命令那样，一个主命令下可以有多个子命令，每个子命令有自己独立的参数。

### 典型例子

```
# git 就是典型的子命令模式  
git commit -m "message"  
git push origin main  
git pull origin main
```

每个子命令（`commit`、`push`、`pull`）都是独立的，有自己的参数和逻辑。

### 代码示例

```
import argparse  
  
def main():  
    parser = argparse.ArgumentParser(description='文件管理工具')  
    subparsers = parser.add_subparsers(dest='command', help='可用命令')  
      
    # 创建 upload 子命令  
    upload_parser = subparsers.add_parser('upload', help='上传文件')  
    upload_parser.add_argument('--file', required=True, help='要上传的文件路径')  
    upload_parser.add_argument('--bucket', help='存储桶名称')  
      
    # 创建 download 子命令  
    download_parser = subparsers.add_parser('download', help='下载文件')  
    download_parser.add_argument('--file', required=True, help='要下载的文件名')  
    download_parser.add_argument('--output', help='保存路径')  
      
    args = parser.parse_args()  
      
    if args.command == 'upload':  
        print(f"上传文件: {args.file} 到 {args.bucket}")  
    elif args.command == 'download':  
        print(f"下载文件: {args.file} 到 {args.output}")  
  
if __name__ == '__main__':  
    main()
```

**使用效果：**

```
# 上传文件  
python tool.py upload --file data.txt --bucket my-bucket  
  
# 下载文件  
python tool.py download --file data.txt --output ./local/
```

---

## 二、什么是 Argument-groups（参数组）？

**Argument-groups**是用来**组织和管理参数**的，它不会改变命令的结构，只是把相关的参数分组显示，让帮助信息更清晰。

### 典型场景

当你有很多参数时，可以把它们按功能分组：

```
# 所有参数都在一个命令下，但分组显示  
python tool.py --input file.txt --output dir/ --verbose --debug
```

### 代码示例

```
import argparse  
  
def main():  
    parser = argparse.ArgumentParser(description='数据处理工具')  
      
    # 创建参数组  
    input_group = parser.add_argument_group('输入选项')  
    input_group.add_argument('--input', required=True, help='输入文件路径')  
    input_group.add_argument('--format', choices=['json', 'csv'], help='输入格式')  
      
    output_group = parser.add_argument_group('输出选项')  
    output_group.add_argument('--output', required=True, help='输出目录')  
    output_group.add_argument('--overwrite', action='store_true', help='是否覆盖已存在文件')  
      
    debug_group = parser.add_argument_group('调试选项')  
    debug_group.add_argument('--verbose', action='store_true', help='详细输出')  
    debug_group.add_argument('--debug', action='store_true', help='调试模式')  
      
    args = parser.parse_args()  
    print(f"处理文件: {args.input} -> {args.output}")  
  
if __name__ == '__main__':  
    main()
```

**使用效果：**

```
# 查看帮助信息，参数会按组显示  
python tool.py --help  
  
# 输出：  
# 输入选项:  
#   --input INPUT        输入文件路径  
#   --format {json,csv}  输入格式  
#  
# 输出选项:  
#   --output OUTPUT      输出目录  
#   --overwrite          是否覆盖已存在文件  
#  
# 调试选项:  
#   --verbose            详细输出  
#   --debug              调试模式
```

---

## 三、两者的核心区别

### 1. **功能定位不同**

| 特性 | Sub-commands | Argument-groups |
| --- | --- | --- |
| **作用** | 创建多个独立的命令 | 组织参数显示 |
| **命令结构** | 改变命令结构（主命令 + 子命令） | 不改变命令结构 |
| **参数隔离** | 每个子命令的参数完全独立 | 所有参数都在同一命令下 |

### 2. **使用场景不同**

**Sub-commands 适用于：**

* ✅ 工具功能模块化，每个功能相对独立
* ✅ 不同功能需要的参数差异很大
* ✅ 希望命令结构清晰，类似 `git`、`docker`这样的工具
* ✅ 不同子命令可能有完全不同的逻辑

**Argument-groups 适用于：**

* ✅ 所有参数都属于同一个命令
* ✅ 参数很多，需要分类展示让帮助信息更清晰
* ✅ 参数之间有逻辑关联，但都在同一个操作下使用
* ✅ 希望帮助信息更易读，但不需要改变命令结构

### 3. **代码结构对比**

**Sub-commands 结构：**

```
parser = argparse.ArgumentParser()  
subparsers = parser.add_subparsers()  
  
# 每个子命令都是独立的 parser  
cmd1_parser = subparsers.add_parser('cmd1')  
cmd1_parser.add_argument('--arg1')  
  
cmd2_parser = subparsers.add_parser('cmd2')  
cmd2_parser.add_argument('--arg2')
```

**Argument-groups 结构：**

```
parser = argparse.ArgumentParser()  
  
# 所有参数都在同一个 parser 下，只是分组显示  
group1 = parser.add_argument_group('组1')  
group1.add_argument('--arg1')  
  
group2 = parser.add_argument_group('组2')  
group2.add_argument('--arg2')
```

---

## 四、实际应用场景举例

### 场景 1：文件管理工具（适合 Sub-commands）

```
import argparse  
  
def main():  
    parser = argparse.ArgumentParser(description='文件管理工具')  
    subparsers = parser.add_subparsers(dest='command')  
      
    # upload 子命令  
    upload = subparsers.add_parser('upload', help='上传文件')  
    upload.add_argument('file', help='文件路径')  
    upload.add_argument('--bucket', required=True)  
      
    # download 子命令  
    download = subparsers.add_parser('download', help='下载文件')  
    download.add_argument('file', help='文件名')  
    download.add_argument('--output', default='./')  
      
    # delete 子命令  
    delete = subparsers.add_parser('delete', help='删除文件')  
    delete.add_argument('file', help='文件名')  
    delete.add_argument('--force', action='store_true')  
      
    args = parser.parse_args()  
      
    if args.command == 'upload':  
        handle_upload(args.file, args.bucket)  
    elif args.command == 'download':  
        handle_download(args.file, args.output)  
    elif args.command == 'delete':  
        handle_delete(args.file, args.force)  
  
if __name__ == '__main__':  
    main()
```

**使用：**

```
python tool.py upload data.txt --bucket my-bucket  
python tool.py download data.txt --output ./files/  
python tool.py delete data.txt --force
```

### 场景 2：数据处理工具（适合 Argument-groups）

```
import argparse  
  
def main():  
    parser = argparse.ArgumentParser(description='数据处理工具')  
      
    # 输入参数组  
    input_group = parser.add_argument_group('输入配置')  
    input_group.add_argument('--input-file', required=True, help='输入文件')  
    input_group.add_argument('--input-format', choices=['json', 'csv', 'xml'])  
    input_group.add_argument('--encoding', default='utf-8')  
      
    # 处理参数组  
    process_group = parser.add_argument_group('处理配置')  
    process_group.add_argument('--filter', help='过滤条件')  
    process_group.add_argument('--sort', help='排序字段')  
    process_group.add_argument('--limit', type=int, help='限制条数')  
      
    # 输出参数组  
    output_group = parser.add_argument_group('输出配置')  
    output_group.add_argument('--output-file', required=True)  
    output_group.add_argument('--output-format', choices=['json', 'csv'])  
    output_group.add_argument('--pretty', action='store_true')  
      
    args = parser.parse_args()  
    process_data(args)  
  
if __name__ == '__main__':  
    main()
```

**使用：**

```
python tool.py \  
  --input-file data.csv --input-format csv \  
  --filter "age > 18" --sort name --limit 100 \  
  --output-file result.json --output-format json --pretty
```

---

## 五、可以同时使用吗？

**可以！**两者并不冲突，可以在子命令内部再使用参数组。

```
import argparse  
  
def main():  
    parser = argparse.ArgumentParser(description='高级工具')  
    subparsers = parser.add_subparsers(dest='command')  
      
    # process 子命令，内部使用参数组  
    process_parser = subparsers.add_parser('process', help='处理数据')  
      
    input_group = process_parser.add_argument_group('输入选项')  
    input_group.add_argument('--input', required=True)  
    input_group.add_argument('--format', choices=['json', 'csv'])  
      
    output_group = process_parser.add_argument_group('输出选项')  
    output_group.add_argument('--output', required=True)  
    output_group.add_argument('--overwrite', action='store_true')  
      
    args = parser.parse_args()  
    # ...  
  
if __name__ == '__main__':  
    main()
```

---

## 总结

### 核心要点

1. **Sub-commands（子命令）**：

* 用于创建多个独立的命令
* 改变命令结构：`主命令 子命令 [参数]`
* 适合功能模块化、参数差异大的场景
* 典型例子：`git commit`、`docker run`

2. **Argument-groups（参数组）**：

* 用于组织和管理参数显示
* 不改变命令结构，所有参数在同一命令下
* 适合参数多、需要分类展示的场景
* 主要作用是让帮助信息更清晰

3. **选择建议**：

* 功能独立、参数差异大 → 用 **Sub-commands**
* 参数多、需要分类展示 → 用 **Argument-groups**
* 两者可以结合使用

### 快速决策树

```
需要多个独立命令？  
├─ 是 → 使用 Sub-commands  
└─ 否 → 参数很多需要分组显示？  
    ├─ 是 → 使用 Argument-groups  
    └─ 否 → 直接用普通参数即可
```

---

每日踩一坑，生活更轻松。

本期分享就到这里啦，祝你在测开与后端之路上越走越稳，越走越远。