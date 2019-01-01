---
layout: post
title: Formatting SharePoint Data in SQL Reporting Services Report
tags: [sp2007, ssrs]
---

Depending on the field type, SharePoint may format the data as “#n;Value”.  Not very pretty.  I found this solution somewhere, but don’t remember where.  You can add the below function to your SSRS report by going to Report (when on the Data or Layout tab in Visual Studio/Business Intelligence Development Studio) –> Report Properties –> Code.

<pre><code class="vbnet">
Function GetNameFromSP(pFullID As String) As String
 Dim strRet As String
 Dim iPos As Integer

 If pFullID = NOTHING Then Return ""
 If pFullID = "" Then Return ""
    iPos = Instr(pFullID, ";")
 If iPos < 1 Then Return pFullID

 Return Mid(pFullID, iPos +2)
End Function
</code></pre>

To use this code in an expression, type:

`=Code.GetNameFromSP(Fields!MyField.Value)`