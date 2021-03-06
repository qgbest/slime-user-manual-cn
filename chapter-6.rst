六、定制
=======

6.1 Emacs端定制
--------------

Slime的Emacs部分可以通过Emacs的定制系统来进行配置，通过M-x customize-group slime RET。由于此定制系统是自描述的，因此在本文档里我们只会包含某些重要的或者可能有歧义的选项。

* slime-truncate-lines

  在Slime生成的line-by-line摘要缓冲区里使用truncate-lines的值。这是默认的，来保证行不会被以某种回溯的列表来折叠。它可能会造成某些信息超出屏幕。

* slime-complete-symbol-function

  用来补全Lisp符号的函数。有三种补全方式：slime-simple-complete-symbol，slime-complete-symbol*（见 8.5 混合补全），和slime-fuzzy-complete-symbol（见 8.6 模糊补全）。

  默认的是slime-simple-complete-symbol，类似于Emacs的补全方式。

* slime-filename-translations

  这个变量控制Emacs和Lisp之间文件名的转换。当你在不同的机器上运行Emacs和Lisp，而它们的文件系统并不相同，或者它们有相同的文件系统但是布局不同，例如使用了SMB的文件共享的时候，这个变量就有用了。

* slime-net-coding-system

  如果你想在Emacs和Lisp之间传送Unicode字符，你应该设置此变量。例如你使用SBCL，你应该这样设置：

::

   (setq slime-net-coding-system 'utf-8-unix)

  要显示Unicode字符你还需要适当的字体，否则这些字符会被渲染成空心方块。如果你使用的是Allegro CL和GNU Emacs，你也可以使用emacs-mule-unix作为编码系统。GNU Emacs给较新的编码提供了很不错的字体。（不同的Lisp系统可以使用不同的编码，见 2.5.2 多种Lisp。）

6.1.1 钩子
^^^^^^^^^

* slime-mode-hook
  
  每次有一个缓冲区进入slime-mode，这个钩子就会启动。它最有用的是在你的Lisp源代码缓冲区里设置当前缓冲区的配置。有个例子是用它来启动slime-autodoc-mode（见 8.7 slime-autodoc-mode）。

* slime-connected-hook

  这个钩子在每次连接到Lisp服务器的时候建立。有个例子是用它来生成一个打印窗口（见 8.13 打印窗口）。

* sldb-hook

  这个钩子在SLDB启动时创建。这个钩子的函数是从SLDB缓冲区建立后调用的。有个例子是添加add sldb-print-condition函数到这个钩子，这会让所有SLDB调试的状况被REPL缓冲区记录到。

6.2 Lisp端（Swank）
----------------

Slime的Lisp服务器端（也叫做“Swank”），提供几个变量可供设置。初始化文件~/.swank.lisp在启动时会被自动求值，可以用来设置这些变量。

6.2.1 通信模式
^^^^^^^^^^^^^

最重要的可以配置的变量是SWANK:*COMMUNICATION-STYLE*，它指定了Lisp端接收和读取Emacs发出的信息的协议。通信方式的选择会在全局影响Slime的行为。

可选的通信方式有：

* NIL

  这种方式只是简单地循环地读取通信套接字里的输入和当Slime协议事件发生是提供服务。这种简单意味着当处于Slime的控制的时候Lisp不能做其它的处理。

* :FD-HANDLER

  这种方式使用经典的Unix的“select-loop()”。Swank通过一个事件分发框架（例如CMUCL和SBCL里的SERVE-EVENT）来注册一个通信套接字，当数据到达时接收一个回调。在这种方式下，从Emacs而来的请求只在Lisp进入了事件循环时才会被检测到并处理。这种方式简单并且可预测。

* :SIGIO

  这种方式使用带有SIGIO信号处理函数的信号驱动I/O。Lisp接收来自Emacs的请求，附带一个信号，使Lisp中断任何正在做的事来处理请求。这种方式的好处是回应即时，因为Emacs可以使Lisp执行某些操作，即使Lisp正在忙于某些事情。它也允许Emacs并发地发起请求，例如发起一个长期的请求（例如编译）然后在它完成前发起几个短期的请求。它的缺点是它可能其它用户的Lisp代码的SIGIO信号冲突，而且在某些特殊的时刻中断Lisp进程的话则可能产生很严重的后果。

* :SPAWN

  这种方式通过Lisp里的多线程支持来在每个线程里执行一个请求。这种方式跟:SIGIO有某些类似的特性，但它不使用信号，并且所有来自Emacs的请求都可以并行处理。

默认的请求处理方式是根据你的Lisp系统的能力来选择的。通常的选择顺序是：:SPAWN，然后是:SIGIO，然后:FD-HANDLER，最后是NIL。你可以通过调用SWANK-BACKEND::PREFERRED-COMMUNICATION-STYLE来查看默认的方式。你也可以通过在你的Swank初始化文件里设置SWANK:*COMMUNICATION-STYLE*变量来覆盖默认设置。

6.2.2 其它配置
^^^^^^^^^^^^

这些Lisp变量可以通过你的~/.swank.lisp文件设置：

* SWANK:*CONFIGURE-EMACS-INDENTATION*

  这个变量控制宏里&body参数的缩进方式是否会被探测到并发送给Emacs。它默认开启。

* SWANK:*GLOBALLY-REDIRECT-IO*

  当它为真，标准输出流（例如*standard-output*）会被全局重定向到Emacs里的REPL。当它为假（默认情况），这些流只是在处理请求时通过动态绑定临时重定向到Emacs。主意*standard-input*现在不会被全局重定向到Emacs，因为当它尝试从Emacs里读取信息时，它跟Lisp原生的REPL交互得很差。

* SWANK:*GLOBAL-DEBUGGER*

  当它为真（默认情况），它让*DEBUGGER-HOOK*全局设置为SWANK:SWANK-DEBUGGER-HOOK，当后让Slime处理Lisp进程里的所有调试工作。这是用来调试多线程或回调驱动的应用的。

* SWANK:*SLDB-PRINTER-BINDINGS* 和 SWANK:*MACROEXPAND-PRINTER-BINDINGS* 和 SWANK:*SWANK-PPRINT-BINDINGS*

  这些变量可以在不同的情况下设置打印器。这些变量的值是打印器变量名和对应的值组成的联合列表。例如，在SLDB中开启pretty打印器来处理调用栈，你可以这样：

::

   (push '(*print-pretty* . t) swank:*sldb-printer-bindings*)

* SWANK:*USE-DEDICATED-OUTPUT-STREAM*

  这个变量控制了是否用一个不安全但很有效的hack来从Lisp打印输出到Emacs。默认是nil，并且强烈建议不要使用它。

  当它为t时，会建立一个单独的套接字来把Lisp的输出打印到Emacs，这比使用协议发送信息来将输出发送到Emacs快。但是，由于不能保证一个专用的输出流和一个给予协议消息的流之间的时间，Lisp命令的输出到达的时间可能在REPL相应时间之前或之后。输出结果和REPL的显示结果可能以错误的顺序呈现，甚至在REPL里交叉出现。使用一个专用的输出流也会让用SSH跟一个在远程服务器上的Lisp程序通信变得困难。（见 7.1 连接到远程Lisp）

* SWANK:*DEDICATED-OUTPUT-STREAM-PORT*

  当*USE-DEDICATED-OUTPUT-STREAM*是t，流会在此端口开启。默认值是0，表示流会在某个随机端口开启。

* SWANK:*LOG-EVENTS*

  将这个变量设置为t会让所有与Emacs交换的协议信息都打印到*TERMINAL-IO*。这在底层调试和观察Slime底层如何运行是很有用。*TERMINAL-IO*的输出可以在Lisp系统自己的监视器里看到，通常是*inferior-lisp*缓冲区。
