---
title: DeepSeek自动完成会计对账,太方便了吧！
author: Excel精英培训
date: 
url: http://mp.weixin.qq.com/s?__biz=MjM5NDYyNzAzNQ==&mid=2652988888&idx=1&sn=bca90d216db295f4d611597c5450debd&chksm=bcb767a625b2c09933b9a0bcc6332c1ed60b60d8e60aaa0001183668823b4b7e9c3a29acfe2e&mpshare=1&scene=24&srcid=0305AYAI8vkZcFmSsbQPn0t2&sharer_shareinfo=4546e6b77b18c056f049c34f713f235f&sharer_shareinfo_first=4546e6b77b18c056f049c34f713f235f#rd
---

如果问什么工作最让会计崩溃，那一定是复杂的表格核对。

如下图所示左侧是合同金额（应收款），而右侧是客户的还款金额。现在需要核对出左表中哪些合同的款已还完了。并且需要在两个表同时标注颜色。

提起对账，很多同就会说，那还不容易用Vlookup啊。仔细看后，发现这不是简单的查找，而是实现一对多核对，多是几个？未知。比如下面的A客户的A0001的10000，需要和右表中的4000+4000+2000核对

这样的工作如果手工去做，不加班才怪呢。为了解决这个老大难问题，兰色让Deepseek写了个代码，超级好用。先看效果演示。

点会计对账，分别选取两个表，选完点确定，对账结果秒生成。

不但用颜色标识两个表格数据，而且把核对的行数都列出来了。

以下是提问词：

```
请帮用VBA编写一段代码，运行后弹出选择表1窗口，选取表1后再弹出选取表2的窗口，（注：两个表都要求选取两列，如不符需要重选），点退出按钮中止程序，选取两个表格后对两个表进行核对，核对规则如下：1 、两个表按两列核对。2 如果两个表两列值均相同，则两个表的第2列均填庑相同的随机色 3 如果表1的行在第2个表中找不到两列都相同的行，可以按第1列从表2中查找第1列相同的多行内容，如果多行内容第2列值之和等于表1第2列的值，则两个表对应的第2列均填充相同的随机色，要求，两个表的值均只能成功核对一次。
```

当然这么复杂的代码无法一色编写完成，经过3个多小时的交流，终于前天凌晨1点多被Deepseek搞定。在这儿兰色就把代码分享给同学们。

```
Option Explicit  
Sub 会计对账()    ' 定义核心对象    Dim rng1 As Range, rng2 As Range    Dim arr1 As Variant, arr2 As Variant    Dim dictExact As Object, dictCol1 As Object    Dim processed1() As Boolean, processed2() As Boolean  
    ' 用户选择数据区域    On Error Resume Next    Set rng1 = Application.InputBox("选择表1数据区域，不要选标题行", Type:=8)    Set rng2 = Application.InputBox("选择表2数据区域，不要选标题行", Type:=8)    On Error GoTo 0    If rng1 Is Nothing Or rng2 Is Nothing Then Exit Sub  
    ' 插入信息记录列    InsertNewColumn rng1, "匹配表2信息"    InsertNewColumn rng2, "匹配表1信息"  
    ' 初始化数据结构    arr1 = rng1.value    arr2 = rng2.value    ReDim processed1(1 To UBound(arr1, 1))    ReDim processed2(1 To UBound(arr2, 1))  
    ' 创建字典对象    Set dictExact = CreateObject("Scripting.Dictionary")    Set dictCol1 = CreateObject("Scripting.Dictionary")    dictExact.CompareMode = 1    dictCol1.CompareMode = 1  
    ' 填充字典    BuildDictionaries arr2, dictExact, dictCol1  
    ' 主处理逻辑    ProcessMatches arr1, arr2, dictExact, dictCol1, processed1, processed2, rng1, rng2  
    ' 清理对象    Set dictExact = Nothing    Set dictCol1 = NothingEnd Sub  
Private Sub InsertNewColumn(rng As Range, header As String)    ' 在数据区域右侧插入新列    Dim newCol As Range    Set newCol = rng.Offset(0, rng.Columns.Count).EntireColumn    newCol.Insert Shift:=xlToRight    rng.Parent.Cells(rng.row, newCol.Column).value = headerEnd Sub  
Private Sub BuildDictionaries(arr As Variant, ByRef dictExact As Object, ByRef dictCol1 As Object)    Dim i As Long    For i = 1 To UBound(arr, 1)        ' 精确匹配字典        Dim keyExact As String        keyExact = GetKey(arr(i, 1), arr(i, 2))        If Not dictExact.Exists(keyExact) Then            dictExact.Add keyExact, New Collection        End If        dictExact(keyExact).Add i  
        ' 列1字典        Dim keyCol1 As String        keyCol1 = CStr(arr(i, 1))        If Not dictCol1.Exists(keyCol1) Then            dictCol1.Add keyCol1, New Collection        End If        dictCol1(keyCol1).Add i    Next iEnd Sub  
Private Sub ProcessMatches(arr1 As Variant, arr2 As Variant, _                           dictExact As Object, dictCol1 As Object, _                           ByRef processed1() As Boolean, ByRef processed2() As Boolean, _                           rng1 As Range, rng2 As Range)    Dim i As Long, color As Long    For i = 1 To UBound(arr1, 1)        If processed1(i) Then GoTo Continue  
        ' 精确匹配处理        Dim keyExact As String        keyExact = GetKey(arr1(i, 1), arr1(i, 2))        If dictExact.Exists(keyExact) Then            Dim j As Variant            For Each j In dictExact(keyExact)                Dim exactRow As Long                exactRow = CLng(j)                If Not processed2(exactRow) Then                    color = GenerateRandomColor()                    ApplyColor rng1, i, 2, color                    ApplyColor rng2, exactRow, 2, color  
                    ' 记录匹配信息                    WriteMatchInfo rng1, i, FormatMatchInfo(arr2, exactRow)                    WriteMatchInfo rng2, exactRow, FormatMatchInfo(arr1, i)  
                    processed1(i) = True                    processed2(exactRow) = True                    Exit For                End If            Next j        End If  
        If processed1(i) Then GoTo Continue  
        ' 部分匹配处理        Dim keyCol1 As String        keyCol1 = CStr(arr1(i, 1))        If dictCol1.Exists(keyCol1) Then            Dim matchedRows As Collection            Set matchedRows = New Collection  
            ' 收集候选行            Dim dictKey As Variant            For Each dictKey In dictCol1(keyCol1)                Dim candidateRow As Long                candidateRow = CLng(dictKey)                If candidateRow <= UBound(arr2, 1) Then                    If Not processed2(candidateRow) And IsNumeric(arr2(candidateRow, 2)) Then                        matchedRows.Add candidateRow                    End If                End If            Next dictKey  
            ' 寻找子集            Dim sumRows As Collection            Set sumRows = New Collection            If FindSumSubset(arr2, matchedRows, CDbl(arr1(i, 2)), sumRows) Then                color = GenerateRandomColor()                ApplyColor rng1, i, 2, color                processed1(i) = True  
                ' 构建匹配信息                Dim sumInfo As String, rowItem As Variant                sumInfo = ""                For Each rowItem In sumRows                    Dim targetRow As Long                    targetRow = CLng(rowItem)                    If targetRow <= UBound(arr2, 1) Then                        ApplyColor rng2, targetRow, 2, color                        processed2(targetRow) = True  
                        ' 为表2写入匹配信息                        WriteMatchInfo rng2, targetRow, FormatMatchInfo(arr1, i)  
                        ' 累积表1的匹配信息                        If sumInfo <> "" Then sumInfo = sumInfo & "+"                        sumInfo = sumInfo & FormatMatchInfo(arr2, targetRow)                    End If                Next rowItem  
                ' 为表1写入组合信息                WriteMatchInfo rng1, i, sumInfo            End If        End If  
Continue:    Next iEnd Sub  
Private Function FindSumSubset(arr As Variant, _                              candidates As Collection, _                              target As Double, _                              ByRef result As Collection) As Boolean    Set result = New Collection    Dim tempSum As Double    tempSum = 0  
    Dim elem As Variant    For Each elem In candidates        Dim currentRow As Long        currentRow = CLng(elem)  
        If currentRow > UBound(arr, 1) Then Exit Function  
        tempSum = tempSum + CDbl(arr(currentRow, 2))        result.Add currentRow  
        If Abs(tempSum - target) < 0.000001 Then            FindSumSubset = True            Exit Function        ElseIf tempSum > target Then            Exit For        End If    Next elem  
    Set result = Nothing    FindSumSubset = FalseEnd Function  
Private Function FormatMatchInfo(arr As Variant, row As Long) As String    ' 格式化信息为"值（行号）"    Dim value As String    value = FormatNumber(arr(row, 2), 2)  ' 保留两位小数    FormatMatchInfo = value & "（行" & row & "）"End Function  
Private Sub WriteMatchInfo(rng As Range, row As Long, info As String)    ' 写入信息到新增列    Dim targetCell As Range    Set targetCell = rng.Parent.Cells(rng.row + row - 1, rng.Column + rng.Columns.Count)  
    ' 追加模式（处理多对一情况）    If Len(targetCell.value) > 0 Then        targetCell.value = targetCell.value & "+" & info    Else        targetCell.value = info    End IfEnd Sub  
Private Function GenerateRandomColor() As Long    GenerateRandomColor = RGB( _        Int((255 - 200 + 1) * Rnd + 200), _        Int((255 - 200 + 1) * Rnd + 200), _        Int((255 - 200 + 1) * Rnd + 200))End Function  
Private Sub ApplyColor(rng As Range, row As Long, col As Long, color As Long)    rng.Cells(row, col).Interior.color = colorEnd Sub  
Private Function GetKey(col1 As Variant, col2 As Variant) As String    GetKey = CStr(col1) & "|" & FormatNumber(col2, 2)  ' 标准化数值格式End Function
```

嘿嘿，是不是被这么长的代码给惊到了，期实兰色其实一句也没写，全是Deepseek编写的，兰色就是执行代码提反馈意见。

这么长的代码怎么用？开发工具 - Visual Basic - 插入-模块，把代码粘进去点三角运行。

如果你想和兰色一样把它放在工具栏中，可以看一下上周二发的一篇教程：[DeepSeek自动拆分Excel表格，太强了吧！](https://mp.weixin.qq.com/s?__biz=MjM5NDYyNzAzNQ==&mid=2652988753&idx=1&sn=d03eee28a7a61d8a31656998843eadcd&scene=21#wechat_redirect)

兰色说：让DeepSeek写代码就可以实现以前我很多无法实现的功能，如本文的复杂核对，多表合并取数等。虽然是不需要写，但还是需要懂点基础理论的，同学们有时间可以学一下VBA编程，本月平台的VBA班也会招生。

本周日，兰色将继续在视频号直播分享学习Deepseek在Excel应用的心得，想学的同学可以点下方链接预约。

如果错过以上课的同学可以点击下面链接购买兰色的Deepseek课程。