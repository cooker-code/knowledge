---
title: 国产大模型 + claude code 及IDE插件、MCP配置，纯小白必看，纯国内网络，没有比这个再详细的了
author: 我才是嘎嘎乐
date: 
url: https://mp.weixin.qq.com/s?__biz=MzkzOTUxODQ5MA==&mid=2247485033&idx=1&sn=d175bbae671b5bc3560bfc1865863135&chksm=c358285981188f2b598c2402d3ec90b46cac1c2e41c935740772e6428cf3993154cabbd45f9f&mpshare=1&scene=24&srcid=1029YWz6gL9n3gds6Cw1kD4I&sharer_shareinfo=9703e8bbe805c4f221ef28dfb9034eaf&sharer_shareinfo_first=9703e8bbe805c4f221ef28dfb9034eaf#rd
---

# 这几天给公司非研发人员做提效培训，没想到安装claude code 配置环境竟然出现了这么多问题。

# （也难怪大模型领域更新的太快了，下篇文字会发纯国内网络环境配置最新的claude code 的重大更新skills的方法，欢迎关注。）

# 

# 以下是详细内容：

# 一、claude code 的安装

三种情况没装过、装过升级、装过卸载重装

## 1.1 从来没装过

1. 1. 打开命令行窗口，输入：claude -v 看看是不是什么都没有
2. 2. 输入命令：npm install -g @anthropic-ai/claude-code

   npm install -g @anthropic-ai/claude-code --registry=https://registry.npmmirror.com

   第一个命令慢就输入第二个，第二个是国内源
3. 3. 校验安装：claude -v

## 1.2 升级版本

1. 1. 打开命令行窗口
2. 2. 先查询： claude -v
3. 3. 再输入安装命令： npm install -g @anthropic-ai/claude-code ，后验证

## 1.3 卸载重装 （下文只说卸载清理，重装请参考1.1）

### 1.3.1 手动版：

1. 1. 打开命令行窗口
2. 2. 输入卸载 Claude Code : npm uninstall -g @anthropic-ai/claude-code
3. 3. 校验是否卸载：claude -v ，找不到识别不了就删除了这个包
4. 4. 手动删除配置文件夹和文件：

1. 配置文件夹路径：C:\Users\<你的用户名>\.claude
2. 配置文件：C:\Users\<你的用户名>\.claude.json 和.claude.json.backup

5. 5. 环境变量

```
# 检查环境变量中是否有 Claude 相关路径  
# Win+R 输入: sysdm.cpl -> 高级 -> 环境变量  
# 查看 PATH 中是否有 Claude 相关路径，有则删除
```

1. 6. npm 缓存和残留

```
# 清理 npm 缓存  
npm cache clean --force  
  
# 检查全局 node_modules（可能有残留，包含@anthropic-ai/claude-code）  
npm list -g --depth=0  
  
# 查看 npm 全局安装目录  
npm config get prefix  
# 然后手动检查该目录下是否有 Claude 相关文件
```

1. 7. 清理npm 缓存

npm cache clean --force

1. 8. 检查全局 node\_modules（可能有残留，包含@anthropic-ai/claude-code）

npm list -g --depth=0

```
# 1. 卸载 Claude 相关的包  
npm uninstall -g ccusage  
  
# 2. 如果不再使用 MCP 服务器（这些是为 Claude Desktop 准备的）  
npm uninstall -g @modelcontextprotocol/server-brave-search  
npm uninstall -g @modelcontextprotocol/server-filesystem  
npm uninstall -g @modelcontextprotocol/server-puppeteer  
  
# 3. 清理缓存  
npm cache clean --force
```

1. 9. AppData 中的残留

```
# 删除这些目录（如果存在）  
Remove-Item-Recurse-Force"$env:APPDATA\Claude"-ErrorAction SilentlyContinue  
Remove-Item-Recurse-Force"$env:LOCALAPPDATA\Claude"-ErrorAction SilentlyContinue  
Remove-Item-Recurse-Force"$env:APPDATA\anthropic-ai"-ErrorAction SilentlyContinue  
Remove-Item-Recurse-Force"$env:LOCALAPPDATA\anthropic-ai"-ErrorAction SilentlyContinue
```

1. 10. VSCode 扩展（如果使用）

```
# 如果安装了 Claude VSCode 扩展  
code --list-extensions | findstr claude  
# 卸载找到的扩展  
code --uninstall-extension <扩展ID>
```

1. 11. 注册表（可选，谨慎操作）

```
Win+R -> regedit  
搜索: "claude" 或 "anthropic"  
删除找到的相关键值
```

1. 12. 检查项目中的残留

```
# 在你的工作目录中搜索  
Get-ChildItem -Path C:\Users\clay_ -Recurse -Include .claude.json,.claude -Force -ErrorAction SilentlyContinue
```

建议操作顺序：

1. 1. 清理 npm 缓存
2. 2. 检查并删除 AppData 目录
3. 3. 检查环境变量
4. 4. 搜索并删除项目中的残留配置
5. 5. 重启电脑

### 1.3.2 自动版：

1. 1. 管理员身份打开powershell ：

1. 2. 将以下内容全部粘贴到命令行

```
# ============ 第一步：卸载所有 Claude 相关 npm 包 ============  
Write-Host"正在卸载 Claude 相关 npm 包..."-ForegroundColor Yellow  
  
npm uninstall -g ccusage  
npm uninstall -g @modelcontextprotocol/server-brave-search  
npm uninstall -g @modelcontextprotocol/server-filesystem  
npm uninstall -g @modelcontextprotocol/server-puppeteer  
npm uninstall -g @anthropic-ai/claude-code  
  
# ============ 第二步：清理用户配置文件 ============  
Write-Host"正在清理用户配置..."-ForegroundColor Yellow  
  
Remove-Item-Recurse-Force"$env:USERPROFILE\.claude"-ErrorAction SilentlyContinue  
Remove-Item-Force"$env:USERPROFILE\.claude.json"-ErrorAction SilentlyContinue  
Remove-Item-Force"$env:USERPROFILE\.claude.json.backup"-ErrorAction SilentlyContinue  
  
# ============ 第三步：清理 AppData ============  
Write-Host"正在清理 AppData..."-ForegroundColor Yellow  
  
Remove-Item-Recurse-Force"$env:APPDATA\Claude"-ErrorAction SilentlyContinue  
Remove-Item-Recurse-Force"$env:LOCALAPPDATA\Claude"-ErrorAction SilentlyContinue  
Remove-Item-Recurse-Force"$env:APPDATA\anthropic-ai"-ErrorAction SilentlyContinue  
Remove-Item-Recurse-Force"$env:LOCALAPPDATA\anthropic-ai"-ErrorAction SilentlyContinue  
Remove-Item-Recurse-Force"$env:APPDATA\anthropic"-ErrorAction SilentlyContinue  
Remove-Item-Recurse-Force"$env:LOCALAPPDATA\anthropic"-ErrorAction SilentlyContinue  
  
# ============ 第四步：清理 npm 缓存和配置 ============  
Write-Host"正在清理 npm 缓存..."-ForegroundColor Yellow  
  
npm cache clean --force  
  
# 检查 npm 全局目录中的残留  
$npmGlobalPath = npm config get prefix  
Remove-Item-Recurse-Force"$npmGlobalPath\node_modules\@anthropic-ai"-ErrorAction SilentlyContinue  
Remove-Item-Recurse-Force"$npmGlobalPath\node_modules\.cache"-ErrorAction SilentlyContinue  
  
# ============ 第五步：清理项目中的配置 ============  
Write-Host"正在搜索并清理项目配置..."-ForegroundColor Yellow  
  
# 搜索所有 .claude.json  
Get-ChildItem-Path"$env:USERPROFILE"-Recurse-Include".claude.json"-Force-ErrorAction SilentlyContinue | Remove-Item-Force  
  
# 搜索所有 .claude 目录  
Get-ChildItem-Path"$env:USERPROFILE"-Recurse-Directory-Filter".claude"-Force-ErrorAction SilentlyContinue | Remove-Item-Recurse-Force  
  
# ============ 第六步：清理 VSCode 扩展（如果有） ============  
Write-Host"正在检查 VSCode 扩展..."-ForegroundColor Yellow  
  
$claudeExtensions = code --list-extensions2>$null | Select-String-Pattern"claude|anthropic"  
if ($claudeExtensions) {  
    foreach ($extin$claudeExtensions) {  
        Write-Host"卸载扩展: $ext"-ForegroundColor Cyan  
        code --uninstall-extension$ext  
    }  
}  
  
# ============ 第七步：清理环境变量 ============  
Write-Host"请手动检查环境变量..."-ForegroundColor Yellow  
Write-Host"Win+R 输入: sysdm.cpl -> 高级 -> 环境变量"-ForegroundColor Cyan  
Write-Host"检查 PATH 中是否有 Claude 相关路径"-ForegroundColor Cyan  
  
# ============ 第八步：清理注册表（可选） ============  
Write-Host"清理注册表（可选，需要手动）..."-ForegroundColor Yellow  
Write-Host"Win+R 输入: regedit"-ForegroundColor Cyan  
Write-Host"搜索: claude, anthropic"-ForegroundColor Cyan  
  
# ============ 验证清理结果 ============  
Write-Host"`n============ 验证清理结果 ============"-ForegroundColor Green  
  
Write-Host"`n全局 npm 包:"-ForegroundColor Cyan  
npm list -g--depth=0 | Select-String-Pattern"claude|anthropic"  
  
Write-Host"`n用户目录检查:"-ForegroundColor Cyan  
Test-Path"$env:USERPROFILE\.claude"  
Test-Path"$env:USERPROFILE\.claude.json"  
  
Write-Host"`nAppData 检查:"-ForegroundColor Cyan  
Test-Path"$env:APPDATA\Claude"  
Test-Path"$env:LOCALAPPDATA\Claude"  
  
Write-Host"`n============ 清理完成 ============"-ForegroundColor Green  
Write-Host"建议重启电脑后再重新安装"-ForegroundColor Yellow
```

1. 3. 检查环境变量

```
Win+R -> sysdm.cpl -> 高级 -> 环境变量   
检查 PATH 和其他变量中是否有：  
- Claude 相关路径  
- Anthropic 相关路径  
检查 参数 配置
```

1. 4. 重启电脑，powershell 输入以下脚本验证：

```
Write-Host "`n========== Claude 卸载验证 =========="-ForegroundColor Cyan  
  
# 验证 1: npm 全局包  
Write-Host"`n[1/4] 检查 npm 全局包..."-ForegroundColor Yellow  
$npmCheck = npm list -g--depth=02>&1 | Select-String-Pattern"claude|anthropic|ccusage"  
if ($npmCheck) {  
    Write-Host"❌ 失败：仍有 Claude 相关包"-ForegroundColor Red  
    $npmCheck  
} else {  
    Write-Host"✅ 通过：无 Claude 相关包"-ForegroundColor Green  
}  
  
# 验证 2: where claude  
Write-Host"`n[2/4] 检查 claude 命令..."-ForegroundColor Yellow  
$whereResult = where.exe claude 2>&1  
if ($whereResult-match"无法找到|Could not find") {  
    Write-Host"✅ 通过：未找到 claude 命令"-ForegroundColor Green  
} else {  
    Write-Host"❌ 失败：找到 claude 命令"-ForegroundColor Red  
    $whereResult  
}  
  
# 验证 3: 配置文件  
Write-Host"`n[3/4] 检查配置文件..."-ForegroundColor Yellow  
$configFiles = Get-ChildItem-Path"$env:USERPROFILE"-Recurse-Include".claude*"-Force-ErrorAction SilentlyContinue  
if ($configFiles) {  
    Write-Host"❌ 失败：仍有配置文件残留"-ForegroundColor Red  
    $configFiles | Select-Object FullName  
} else {  
    Write-Host"✅ 通过：无配置文件残留"-ForegroundColor Green  
}  
  
# 验证 4: AppData 目录  
Write-Host"`n[4/4] 检查 AppData 目录..."-ForegroundColor Yellow  
$appDataPaths = @(  
    "$env:APPDATA\Claude",  
    "$env:LOCALAPPDATA\Claude",  
    "$env:APPDATA\anthropic",  
    "$env:LOCALAPPDATA\anthropic"  
)  
$foundPaths = $appDataPaths | Where-Object { Test-Path$_ }  
if ($foundPaths) {  
    Write-Host"❌ 失败：仍有 AppData 残留"-ForegroundColor Red  
    $foundPaths  
} else {  
    Write-Host"✅ 通过：无 AppData 残留"-ForegroundColor Green  
}  
  
# 总结  
Write-Host"`n========== 验证总结 =========="-ForegroundColor Cyan  
$allPassed = -not ($npmCheck-or ($whereResult-notmatch"无法找到|Could not find") -or$configFiles-or$foundPaths)  
if ($allPassed) {  
    Write-Host"🎉 全部验证通过！Claude 已完全卸载"-ForegroundColor Green  
    Write-Host"可以安全地重新安装了"-ForegroundColor Green  
} else {  
    Write-Host"⚠️ 部分验证未通过，请检查上述失败项"-ForegroundColor Yellow  
}
```

# 二、Claude code 配置

## 2.1 登录智谱AI开放平台

地址：https://bigmodel.cn/

点击红色框套餐

选择套餐：尽量选择 pro 套餐，因为后续有专属的视觉、联网应用拓展。不使用也可以使用其他方法浏览网页或搜索。

之前订阅的升级到100/月的套餐还有优惠：

新订阅的扫这个也有优惠，自己从官方订也可以：

## 2.2 获取API Key

1. 1. 点击控制台 -> API Key -> 添加新Key -> 复制key -> 保存备用

## 2.3 配置claude code 使用GLM4.6

官方配置说明参考（编程套餐的）：https://docs.bigmodel.cn/cn/coding-plan/overview

1. 1. 在 C:\Users\<user>\.claude 文件夹下新建 settings.json 文件
2. 2. 手动配置登陆信息 ：文件位置如图，没有就创建一个

```
//在这里配置，可以指定一直使用最高智商模型  
{  
  "env": {  
    "ANTHROPIC_BASE_URL":"https://open.bigmodel.cn/api/anthropic",  
    "ANTHROPIC_AUTH_TOKEN":"your-api-key",//这里换成你自己的apikey  
    "ANTHROPIC_DEFAULT_HAIKU_MODEL": "glm-4.6",  
    "ANTHROPIC_DEFAULT_SONNET_MODEL": "glm-4.6",  
    "ANTHROPIC_DEFAULT_OPUS_MODEL": "glm-4.6"  
  }  
}
```

1. 3. 启动 （按以上方式在settings.json里配置了模型会显示GLM4.6）

一直点回车：

# 三、IDE编程工具插件配置

左侧菜单依次：资源管理器（看项目文件结果）、搜索、源代码管理（版本管理git）。。。插件市场

可能会提示需要登陆（2.0.24版本过一会可能会登陆过去），主要是也没有全自动狂飙模式。

## 3.1 安装插件

插件市场搜索：claude code yolo 安装，重启ide，点击资源管理器，点击任意一个文件，文件右上角

## 3.2 测试

# 四、安装MCP

## 4.1 安装（安装playwright mcp举例）

输入/选择MCP

提示没有任何mcp服务

输入：claude mcp add playwright npx @playwright/mcp@latest

重启，再输入/查看：

## 4.2 测试

自己打开浏览器检索：

## 4.3 智谱AI的视觉和联网（coding pro专用）

[这下国产AI编程靠谱了，智谱GLM4.6再进化，附视觉、联网解决方案，建议小白收藏实践](https://mp.weixin.qq.com/s?__biz=MzkzOTUxODQ5MA==&mid=2247484925&idx=1&sn=6ace7e3f438c69a341d57f1be1e7817d&scene=21#wechat_redirect)

下一篇更新：claude code skills 纯国产环境使用指南。