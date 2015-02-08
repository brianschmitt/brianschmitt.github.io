---
layout: post
title:  "Better Visual Studio F1"
comments: true
tags: [Visual Studio,Macros,Help]
---


So recently there have been some great tips on turning off the F1 key in Visual Studio.

[Roy Osherove](http://weblogs.asp.net/rosherove/archive/2009/08/13/six-things-that-will-happen-when-you-uninstall-your-msdn-documentation.aspx) and [Infinities Loop](http://weblogs.asp.net/infinitiesloop/archive/2008/07/18/visual-studio-tip-disable-f1.aspx)

While that is useful in turning off the Visual Studio help, it may not be useful in the event that you actually NEED help.

I have found it useful to re-bind F1 to a macro that will take the currently highlighted word from your Visual Studio text editor and perform a search at the designated site.

I have provided code below to allow you to quickly search [StackOverflow](http://www.stackoverflow.com/), Google, MSDN, and [Searchdotnet](http://www.searchdotnet.com/).

Caveats - The below script will only work in the text editor, I can provide additional code that will also use the selected text from the output window or the html-editor.  (I tried to keep it simple.)

I love macros in VS and think they are highly under used, I will be posting more soon, so subscribe and welcome to my new blog!

Take your pick of the four provided below (or bind several to the key combinations F1, Alt+F1, Ctrl+F1, etc...)

{% highlight vbnet %}
Imports EnvDTE
Imports System.Web
Public Module Search
#Region "Search Internet Sites"
  Public Const GOOGLE_FORMAT As String = "www.google.com/search?q={0}"
  Public Const STACKOVERFLOW_FORMAT As String = "http://www.stackoverflow.com/search?q={0}"
  Public Const SEARCHDOTNET_FORMAT As String = "http://searchdotnet.com/results.aspx?cx=002213837942349435108:jki1okx03jq&amp;q={0}&amp;sa=Search&amp;cof=FORID:9#1144"
  Public Const MSDN_FORMAT As String = "http://social.msdn.microsoft.com/Search/en-US/?query={0}&amp;ac=8"
  Public Sub SearchStackOverflowForSelectedText()
      SearchWebPage(STACKOVERFLOW_FORMAT)
  End Sub

  Public Sub SearchGoogleForSelectedText()
      SearchWebPage(GOOGLE_FORMAT)
  End Sub

  Public Sub SearchSearchDotNetForSelectedText()
      SearchWebPage(SEARCHDOTNET_FORMAT)
  End Sub

  Public Sub SearchMSDNForSelectedText()
      SearchWebPage(MSDN_FORMAT)
  End Sub

  Private Sub SearchWebPage(ByVal SearchURLFormat As String)
      Dim sel As EnvDTE.TextSelection = DTE.ActiveWindow.Selection
      Dim srchTxt As String = sel.Text.Trim
      If srchTxt.Length > 0 Then
          DTE.ItemOperations.Navigate(String.Format(SearchURLFormat, HttpUtility.UrlEncode(srchTxt)))
      End If
  End Sub
#End Region
End Module
{% endhighlight %}
