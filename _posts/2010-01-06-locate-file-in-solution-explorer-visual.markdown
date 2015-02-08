---
layout: post
title:  "Locate File in Solution Explorer – Visual Studio Macro"
comments: true
tags: [Visual Studio,Macros]
---


Today's Macro is very basic, but I use it almost daily.

I use a customized IDE and one of the performance tweaks I have performed is to Turn Off the Track Active Items (Tools --> Options --> Projects and Solutions).

This feature, when enabled, syncs the selected item in your solution explorer to the file being viewed.

There are times that I have traced down into a file and then needed to locate it in the Solution Explorer; This Macro will assist you finding the current file while leaving the feature turned off.

(Other add-ins like ReSharper offer a similar feature &quot;Locate in Solution Explorer&quot;)

About the Macro: The First Command toggles on the feature, the second toggles it back off - this allows Visual Studio to find the item in the solution explorer.

Then the third line simply causes the solution explorer to be displayed, this works if the solution explorer window is hidden or closed.

Bind it to a shortcut key and you are all set - mine is bound to (ALT+L, ALT+L)

{% highlight vbnet %}
Public Sub LocateFileInSolutionExplorer()
     DTE.ExecuteCommand(&quot;View.TrackActivityinSolutionExplorer&quot;)
     DTE.ExecuteCommand(&quot;View.TrackActivityinSolutionExplorer&quot;)
     DTE.ExecuteCommand(&quot;View.SolutionExplorer&quot;)
End Sub
{% endhighlight %}
