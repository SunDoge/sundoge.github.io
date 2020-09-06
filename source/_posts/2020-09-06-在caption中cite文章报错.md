---
title: 在caption中cite文章报错
date: 2020-09-06 19:18:29
tags:
  - Latex
---

今天修改论文的时候Latex报了一个挺离谱的bug

![image.png](https://i.loli.net/2020/09/06/t549w6mfIFNzOsY.png)

为了方便搜索，下面是文字版

```
Undefined control sequence.

The compiler is having trouble understanding a command you have used. Check that the command is spelled correctly. If the command is part of a package, make sure you have included the package in your preamble using \usepackage{...}.
 Learn more

\citeauthoryear #1#2->\def \@thisauthor 
                                        {#1}\ifx \@lastauthor \@thisauthor \...
l.172 		}
         
The control sequence at the end of the top line
of your error message was never \def'ed. If you have
misspelled it (e.g., `\hobx'), type `I' and the correct
spelling (e.g., `I\hbox'). Otherwise just continue,
and I'll forget about whatever was undefined.
 self-supervised, line 172

Illegal parameter number in definition of \reserved@a.

<to be read again> 
                   }
l.172 		}
         
You meant to type ## instead of #, right?
Or maybe a } was forgotten somewhere earlier, and things
are all screwed up? I'm going to assume that you meant ##.
 self-supervised, line 172

Illegal parameter number in definition of \reserved@a.

<to be read again> 
                    
l.172 		}
         
You meant to type ## instead of #, right?
Or maybe a } was forgotten somewhere earlier, and things
are all screwed up? I'm going to assume that you meant ##.
 self-supervised, line 172

Illegal parameter number in definition of \reserved@a.

<to be read again> 
                   2
l.172 		}
         
You meant to type ## instead of #, right?
Or maybe a } was forgotten somewhere earlier, and things
are all screwed up? I'm going to assume that you meant ##.
 self-supervised, line 172

Undefined control sequence.

The compiler is having trouble understanding a command you have used. Check that the command is spelled correctly. If the command is part of a package, make sure you have included the package in your preamble using \usepackage{...}.
 Learn more

\cite ...\citeauthoryear ##1##2{\def \@thisauthor 
                                                  {##1}\ifx \@lastauthor \@t...
l.172 		}
         
The control sequence at the end of the top line
of your error message was never \def'ed. If you have
misspelled it (e.g., `\hobx'), type `I' and the correct
spelling (e.g., `I\hbox'). Otherwise just continue,
and I'll forget about whatever was undefined.
 self-supervised, line 172

Illegal parameter number in definition of \reserved@a.

<to be read again> 
                   1
l.172 		}
         
You meant to type ## instead of #, right?
Or maybe a } was forgotten somewhere earlier, and things
are all screwed up? I'm going to assume that you meant ##.
 self-supervised, line 172

Argument of \caption@ydblarg has an extra }.

<inserted text> 
                \par 
l.172 		}
         
I've run across a `}' that doesn't seem to match anything.
For example, `\def\a#1{...}' and `\a}' would produce
this error. If you simply proceed now, the `\par' that
I've just inserted will cause me to report a runaway
argument that might be the root of the problem. But if
your `}' was spurious, just type `2' and it will go away.
 self-supervised, line 172

Runaway argument?

{\@caption \@captype }{\@tempswatrue \@citex }\def \reserved@b {\@tempswafalse \ETC.
! Paragraph ended before \caption@ydblarg was complete.
<to be read again> 
                   \par 
l.172 		}
         
I suspect you've forgotten a `}', causing me to apply this
control sequence to too much text. How can we recover?
My plan is to forget the whole thing and hope for the best.
```

从log里面基本是没法定位问题，但是通过排除法，最终把问题定位到`~\cite{speednet}`上。我先检查了bibtex，没有发现问题，随后我意识到可能是`\cite`在`\caption`里面，所以才有`Illegal parameter number`的问题。上网搜了以下，找到以下信息：

[https://tex.stackexchange.com/questions/227833/cite-references-in-figure-caption](https://tex.stackexchange.com/questions/227833/cite-references-in-figure-caption)

答案作者的解释是cite会变成caption的argument，解决方案是加上`\protect`。也就是说变成`~\protect\cite{speednet}`，问题解决。