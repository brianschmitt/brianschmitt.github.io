---
layout: post
title:  "Converting a concatenated String into String.Format"
comments: true
tags: [Visual Studio,Macros]
---


So tonight I was browsing unanswered questions on [StackOverflow.com](http://StackOverflow.com) and I came across a [question](http://stackoverflow.com/questions/3352779/vs-macro-add-in-to-convert-string-concatenations-to-string-format-style) for which I had written a solution long ago and thought I would turn it into a blog-post instead.

The macro's purpose is simple - turn a string that you are concatenating into a String.Format statement. This generally produces much more readable code and becomes more maintainable long term. It even attempts to recognize when you are already using a [formatted](http://msdn.microsoft.com/en-us/library/dwhawy9k.aspx) ToString.

I have tested with both C# and VB code, Visual Studio 2010 and 2008. To use Highlight the text you want to convert and invoke the macro, I would recommend that you bind it to a keystroke for quick use later on.

For example this:
{% highlight c# %}
var name = "Brian Schmitt";
Console.WriteLine("Hello " + name);

var money = 1234567.89;
Console.WriteLine("You have " + money.ToString("c") + " dollars");

var action = "Pay";
var util = "Electric";
Console.WriteLine("Would you like to " + action + " your " + util + " Bill");

Console.ReadLine();
{% endhighlight %}

Becomes this:
{% highlight c# %}
var name = "Brian Schmitt";
Console.WriteLine(string.Format("Hello {0}", name));

var money = 1234567.89;
Console.WriteLine(string.Format("You have {0:c} dollars", money));

var action = "Pay";
var util = "Electric";
Console.WriteLine(string.Format("Would you like to {0} your {1} Bill", action, util));
Console.ReadLine();
{% endhighlight %}

 Finally the Macro:

{% highlight vbnet %}
Public Sub ConvertToStringFormat()
    DTE.UndoContext.Open("ConvertToStringFormat")
    Dim textSelection As TextSelection = DTE.ActiveDocument.Selection
    Dim output As String = "string.Format(""{0}"", {1})"
    Dim delimt As String = ", "
    Dim fmtdTostring As String = ".tostring("""
    Dim txtSelection As String() = System.Text.RegularExpressions.Regex.Split(textSelection.Text.Trim, "\+\s_[+\n\r\t]|&amp;\s_[+\n\r\t]|\+|&amp;")
    Dim hardStrings As String = String.Empty
    Dim valueStrings As String = String.Empty
    Dim counter As Int16 = 0
    For Each str As String In txtSelection
        Dim tmpString As String = str.Trim
        If tmpString.StartsWith("""") Then
            hardStrings &amp;= tmpString.Substring(1, tmpString.Length - 2)
        Else
            Dim fmt As String = String.Empty
            Dim indxToString As Int32 = 0
            If tmpString.ToLower.Contains(fmtdTostring) Then
                indxToString = tmpString.ToLower.IndexOf(fmtdTostring)
                fmt = tmpString.Substring(indxToString + 11, tmpString.Length - tmpString.ToLower.IndexOf(""")", indxToString) - 1)
            End If
            If fmt <> String.Empty Then
                hardStrings &amp;= "{" &amp; counter.ToString &amp; ":" &amp; fmt &amp; "}"
                valueStrings &amp;= tmpString.Substring(0, indxToString) &amp; delimt
            Else
                hardStrings &amp;= "{" &amp; counter.ToString &amp; "}"
                valueStrings &amp;= tmpString &amp; delimt
            End If
            counter += 1
        End If
    Next
    If valueStrings <> String.Empty Then valueStrings = valueStrings.Substring(0, valueStrings.Length - delimt.Length)
    textSelection.Text = String.Format(output, hardStrings, valueStrings)
    DTE.UndoContext.Close()
End Sub
{% endhighlight %}
