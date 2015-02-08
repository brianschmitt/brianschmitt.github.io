---
layout: post
title:  "Remove Visual Studio Regions with Find and Replace"
comments: true
tags: [Visual Studio,Regex,Macros]
---


Recently Uncle Bob was talking on twitter about [regions](http://twitter.com/unclebobmartin/status/7676319628). He says &quot;Purge All Regions&quot; - Well today's snippet is going to do just that.

The inspiration for this originated with Kyle Bailey's post from a [few years ago](http://codebetter.com/blogs/kyle.baley/archive/2007/12/17/removing-regions-or-quot-how-to-keep-your-code-expanded-quot.aspx). I had taken it, modified it, and today I share it with you; it can be used as a macro as Kyle created it, or you can use it right in Visual Studio's Find and Replace.

Every developer knows how to use Find and Find/Replace, however I have only found a few that know that you can use regular expressions. The regular expressions that Visual Studio supports in the Find Dialog is a slimed down version and is [specific to Visual Studio](http://msdn.microsoft.com/en-us/library/2k3te2cs.aspx).

![image](/posts_images/find_replace_regex.png)

The following will work in both C# and VB, in the Find Options simply select “Use Regular Expressions” and replace with nothing. Change the Look in option to specify the scope of your search and you can make the change solution wide or just your current file.

```
^.*\#(end)*(:Wh)*region.*\n
```
Simple, quick and effective!
