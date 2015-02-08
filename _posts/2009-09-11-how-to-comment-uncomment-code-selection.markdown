---
layout: post
title:  "How to Comment-Uncomment Code Selection in Style Sheets - (or How to fix Visual Studio's broken implementation)"
comments: true
tags: []
---


If you commonly comment out portions of code, then you are used to highlighting a portion of text and using the shortcut of CTRL+K, CTRL+C (or the tool bar) to comment out the selected text.

While this will work for most source files, it does not currently work inside Visual Studio for style sheets (.css) files. (See screen shot and note the status bar message)

![Comment Selection not currently available](/posts_images/option_not_available.jpg)

Why Microsoft has left out this basic functionality is beyond me; In this post, I would like to demonstrate how you can write your own macro to fix it.

First we will record a basic macro - later we will modify it to suit our needs.

![record temporary macro](/posts_images/record_macro.jpg)

Start out by opening a style sheet and placing your cursor at the relevant piece of code, next start recording by selecting Record TemporaryMacro (Tools-->Macros-->Record... or CTRL+Shift+R). Using the keyboard highlight some style information and Ctrl+X to cut your selection to the clipboard. Next add the beginning of our comment /* and paste your code back in. Lastly add the comment close */ and stop recording your macro.

![temporarymacro](/posts_images/temporary_macro.jpg)

Browse your Macro Explorer and you will notice a "RecordingModule" and if expanded a "Temporary Macro". Right click on the macro and choose edit causing your Macro Editor to open. If you completed the steps outlined above you should have ended up with something roughly similar to this:

{% highlight vbnet %}
Sub TemporaryMacro()
    DTE.ActiveDocument.Selection.LineUp(True, 5)
    DTE.ActiveDocument.Selection.StartOfLine(vsStartOfLineOptions.vsStartOfLineOptionsFirstText, True)
    DTE.ActiveDocument.Selection.Cut()
    DTE.ActiveDocument.Selection.Text = "/*"
    DTE.ActiveDocument.Selection.Paste()
    DTE.ActiveDocument.Selection.Text = "*/"
End Sub
{% endhighlight %}

You can see that it recorded the exact steps I took: I highlighted five lines, cut the text, typed, pasted and finally typed some more. If you go back to your text editor and undo your changes and run the macro you should end up with the same ending result.

![commented style - css](/posts_images/commented_css.jpg)

As you can see, for repeatable actions, Macros are great - however at this point you might be thinking - What if I want to comment out 3 lines (or 10 or more)?. Now that we have our starting point, we can modify our temporary macro to better accommodate our concerns and match the common VS implementation.

Let's start modifying by simply grabbing the currently selected text, as you can see VS stores this in DTE.ActiveDocument.Selection and then we will append /* and */ to the beginning and end.

So we end up with something like this:

{% highlight vbnet %}
Sub TemporaryMacro()
    Dim txtSel As TextSelection
    txtSel = DTE.ActiveDocument.Selection
    txtSel.Text = "/*" + txtSel.Text + "*/"
End Sub
{% endhighlight %}

That's a fairly good substitute for our previous recorded macro; We can highlight lines with our mouse or keyboard and execute the macro, and it will comment out a variable number of lines. It works, but could be better.

With the above variation, if you highlight multiple lines and then 'undo' the changes you will notice that it will undo it line by line, a minor annoyance, but luckily the macro system has something just for this scenario. It is called the UndoContext and is fairly easy to use:

{% highlight vbnet %}
Sub TemporaryMacro()
    Try
        DTE.UndoContext.Open("Comment CSS")
        Dim txtSel As TextSelection
        txtSel = DTE.ActiveDocument.Selection
        txtSel.Text = "/*" + txtSel.Text + "*/"
    Finally
        DTE.UndoContext.Close()
    End Try
End Sub
{% endhighlight %}

You should wrap it in a Try/Finally - This is important, but I won't go into 'why' here. Now highlight multiple lines run the macro and then undo it, your changes are preformed as a single transaction and rolled back as one too. It is a nice little enhancement that will give your final macro some polish.

One final enhancement to our macro, In VS you can Comment and also Uncomment a section of code - that would be a nice feature too. However, instead of writing it as a separate macro, let's detect if the code is currently commented, and if so, uncomment it. In the end we should be able to bind a single shortcut for both comment and uncomment.

Here is our final code:

{% highlight vbnet %}
Sub CommentCSS()
'Detect if in a CSS file
    If Not DTE.ActiveDocument.Name.EndsWith("css") Then Return
    Try
        DTE.UndoContext.Open("Comment CSS")
        Dim txtSel As TextSelection = DTE.ActiveDocument.Selection
        Dim currText As String = txtSel.Text
        If currText.Length > 0 Then
            Dim newTxt As String
            If currText.Trim.StartsWith("/*") AndAlso currText.Trim.EndsWith("*/") Then
                newTxt = currText.Replace("/*", "").Replace("*/", "")
            Else
                newTxt = "/*" + currText + "*/"
            End If
            txtSel.Delete() 'This is to help keep formatting correct when multiline
            txtSel.Insert(newTxt, vsInsertFlags.vsInsertFlagsInsertAtEnd)
         End If
    Finally
        DTE.UndoContext.Close()
    End Try
End Sub
{% endhighlight %}

Now to bind our Macro to a keyboard chord: Go to Tools-->Options-->Environment-->Keyboard - In the show commands type CommentCSS, in the dropdown "Use new shortcut in" change the selection to CSS Source Editor, finally in the "Press Shortcut Keys" type CTRL+K, CTRL+C and assign it.

![key binding](/posts_images/key_binding.jpg)
