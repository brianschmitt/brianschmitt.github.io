---
layout: post
title:  "Intro to Visual Studio Macros and The Most Important Macro for Presenters"
comments: true
tags: [Presentations,Visual Studio,Macros]
---


In my [last post](http://www.brianschmitt.com/2009/08/better-visual-studio-f1.html) I told you how to use a macro to replace the F1 'Help' functionality to perform a search on the internet.

In this post, I hope to explain some of the basics of Macros, the single most important macro for presenters/speakers, and how you can bind a macro to a keyboard shortcut.
Basics of Macros
Visual Studio offers a very rich extensibility model and **you** can extend and bend it in many ways.

One way to harness the power of Visual Studio is through macros. A macro has access to some of the core functionality within Visual Studio. An advantage of VS Macros is that you do not have to install or compile them. Sharing is very simple as its just plain text; You simply need the relative snippet of code and paste it in.

I feel that one of the hardest things about Macros is discovering they are available and beginning to use them.

So, lets get started with running a Macro, in Visual Studio hit Alt+F8, this will open your Macro Explorer. It is very much like a Solution Explorer for your Macros. It is here that you can select and run your macros. We will be taking a look at the samples already included with your install of Visual Studio.
Single Most Important Macro for Presenters
Expand the Samples and then expand Accessibility, you should see several macros there, and for this exercise we are interested in DecreaseTextEditorFontSize and **IncreaseTextEditorFontSize**.

![Macro Explorer](/posts_images/macro_explorer.jpg)

Double Click to run one of them, if you have a file open in your editor you should notice the size of the font has now changed. Now Double click and run the opposite macro, and it should switch back to the size you had.

If you are a presenter you can and should definitely know about these two macros! They will allow you to quickly change your environment suitable for an audience.

If you would like to see the code at accomplished this feat, right click on the macro and choose edit, this will open up the Macro Editor, it's a stripped down version of your standard Visual Studio Editor, but you should feel comfortable using it.

For those not sitting at your IDE here is the relevant code:

{% highlight vbnet %}
' Increases the font size used within the editor.
Public Sub IncreaseTextEditorFontSize()
    Dim textEditorFontsAndColors As Properties
    textEditorFontsAndColors = DTE.Properties("FontsAndColors", "TextEditor")
    textEditorFontsAndColors.Item("FontSize").Value += fontSizeIncrement
End Sub

' Decreases the font size used within the editor.
Public Sub DecreaseTextEditorFontSize()
    Dim textEditorFontsAndColors As Properties
    Dim fontSize As [Property]
    textEditorFontsAndColors = DTE.Properties("FontsAndColors", "TextEditor")
    fontSize = textEditorFontsAndColors.Item("FontSize")
    If fontSize.Value >= minimumSupportedEditorSize Then
        fontSize.Value -= fontSizeIncrement
    End If
End Sub
{% endhighlight %}

The two methods simply take the current font size and increment it by a defined amount.

See how simple, yet powerful macros can be?  I know, I know - increasing a font size is not that powerful, but this sample shows that you can tap into key places and accomplish repetitive, mundane tasks. I will show more powerful samples in the future.
Key Binding
As one final exercise lets bind a macro to a keyboard shortcut.
- Go to Tools-->Options, Expand Environment and Select Keyboard.
- In the 'Show Commands containing' textbox type 'TextEditorFont'
- In the Press shortcut keys textbox type Ctrl+Shift+Alt+DownArrow, Select the DecreaseTextEditorFontSize Macro above and click Assign.

![Keyboard Options Dialog](/posts_images/bind_font_size.jpg)

Repeat for Ctrl+Shift+Alt+UpArrow and IncreaseTextEditorFontSize.

Now in the future when you need to quickly change the size of your font in the text editor, you can either double click the macro or use your newly created shortcut keys

Note: if you really do present from your main development machine, I would recommend that you create a macro that can quickly change all your standard environment (fonts/colors/size/etc…) settings at one time.
