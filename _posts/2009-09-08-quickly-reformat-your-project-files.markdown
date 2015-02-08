---
layout: post
title:  "Quickly Reformat your Project Files"
comments: true
tags: [Visual Studio,Macros]
---


Recently on twitter [Phil Haacked asked about reformatting the files](http://twitter.com/haacked/status/3832509024) in a project where it didn't match his preferences. We have all been there, looking at a file that we downloaded or was given, and would find it easier to follow if we could reformat it.

There is an often overlooked feature of Visual Studio that will reformat our code for us. I frequently use it on HTML documents to quickly restructure code in View Source.

This functionality can be found under Edit-->Advanced-->Format Document (or CTRL+K, CTRL+D)  as well as Format Selection – which only applies to the currently highlighted text.

![image](/posts_images/format_selection.png)

One thing to mention is that you should define how you want your code formatted by going to Tools-->Options-->Text Editor and then select your language of choice. (Some languages offer a greater degree of customization)

Now that you know how to reformat a document and how you can define its result, we will write a macro that will achieve what Phil was looking for by - loop through each source file - open, format, save, and then close it:

{% highlight vbnet %}
Sub FormatAll()
    For Each proj As Project In DTE.Solution.Projects
        FormatFileRecur(proj.ProjectItems())
    Next
End Sub

Sub FormatFileRecur(ByVal projectItems As EnvDTE.ProjectItems)
    For Each pi As EnvDTE.ProjectItem In projectItems
        If pi.Collection Is projectItems Then
            Dim pi2 As EnvDTE.ProjectItems = pi.ProjectItems
            Try
                If Not pi.IsOpen Then pi.Open(Constants.vsViewKindCode)
                pi.Document.Activate()
                DTE.ExecuteCommand("Edit.FormatDocument")
                If Not pi.Document.Saved Then pi.Document.Save()
                pi.Document.Close()
            Catch ex As Exception
                'Ignore this error - some project items cannot be opened.
            End Try

            If pi2 IsNot Nothing Then
                FormatFileRecur(pi2)
            End If
        End If
    Next
End Sub
{% endhighlight %}
