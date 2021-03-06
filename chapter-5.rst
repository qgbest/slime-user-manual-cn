五、杂项
=======

5.1 slime-selector
------------------

slime-selector用来快速切换到重要的缓冲区：REPL、SLDB、你正在hack的Lisp源代码等等。一旦触发slime-selector，该命令会要求你输入一个字母来指定显示哪一个缓冲区。这里是一些选项。

* ?

  一个帮助缓冲区，它会列出所有slime-selector可以显示的缓冲区。

* r

  当前Slime连接的REPL缓冲区。

* d

  当前连接最近使用的SLDB缓冲区。

* l

  最近访问的Lisp源代码缓冲区。

* s

  *slime-scratch*缓冲区。

* c

  Slime连接缓冲区。

* t

  Slime线程缓冲区。

slime-selector没有一个默认的键绑定，但是我们建议你给它设置一个全局的键绑定。你可以像这样将它设置为C-c s：

::

   (global-set-key "\C-c s" 'slime-selector)

然后你就可以通过C-c s r从任何地方切换到REPL。

宏def-slime-selector-method可以用来定义slime-selector可识别的新缓冲区。

5.2 slime-macroexpansion-minor-mode
------------------

一个Slime宏扩展缓冲区提供了一些其它命令（这些命令一直是可用的，只是只有在宏扩展缓冲区里才绑定到了特定的键上）

* C-c C-m 或 M-x slime-macroexpand-1-inplace

  像slime-macroexpand-1一样不过原来的形式被展开后的形式替代了。

* g 或 M-x slime-macroexpand-1-inplace

  最后的宏展开再次被执行，当前宏展开缓冲区的内容被新展开的内容替换掉了。

* q 或 M-x slime-temp-buffer-quit

  关闭展开缓冲区。

* C-_ 或 M-x slime-macroexpand-undo

  取消最后一次宏展开操作。

5.3 多重连接
------------------

Slime可以同时连接到多个Lisp进程。当带有前缀参数地调用M-x slime命令时，如果已经有一个Lisp进程了，它会创建一个新的Lisp连接。这很方便，但是这需要一些技巧来确保你的Slime命令是在你期望的Lisp进程里执行的。

有些缓冲区是连接到特定的Lisp进程的。每个Lisp进程都有自己的REPL缓冲区，在相应缓冲区里输入的所有表达式和所有命令都会被发送到相应的连接。其它Slime创建的进程也类似地跟它们最开始的进程绑定在一起，包括SLDB缓冲区、搜索结果等等。这些缓冲区是跟Lisp进程交互的结果，所以在这些缓冲区里执行的命令也发送到相应的进程。

在其它地方执行的命令，例如slime-mode源代码缓冲区，总是使用“默认的“链接。通常来讲是最近建立的连接，但是这可以通过“连接列表“缓冲区重新指定。

* C-c C-x c 或 M-x slime-list-connections

  生成一个缓冲区并列出所有已建立的连接。

* C-c C-x t 或 M-x slime-list-threads

  生成一个缓冲区并显示当前线程。

slime-list-connections生成的缓冲区显示对每个连接有个一行的简介。简介显示了连接的序列号、Lisp实现的名字以及其它的Lisp进程信息。当前的“默认”连接通过一个星号来指示。

connection-list缓冲区里可用的命令有：

* RET 或 M-x slime-goto-connection

  显示光标处连接的REPL缓冲区。

* d 或 M-x slime-connection-list-make-default

  让光标处的连接成为“默认“连接。它会被slime-mode源代码缓冲区的命令使用。

* g 或 M-x slime-update-connection-list

  更新缓冲区里的连接列表。

* q 或 M-x slime-temp-buffer-quit

  退出连接列表缓冲区（杀掉缓冲区，回到之前的窗口设置）

* R 或 M-x slime-restart-connection-at-point

  重启光标处的进程的连接。

* M-x slime-connect
  
  连接到一个运行中的Swank服务器。

* M-x slime-disconnect

  退出所有连接。

* M-x slime-abort-connection

  取消当前的连接尝试。
