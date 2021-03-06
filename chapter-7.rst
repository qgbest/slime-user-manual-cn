七、小技巧
===========

7.1 连接到远程Lisp
------------------

7.1.1 设置Lisp镜像
^^^^^^^^^^^^^^^^^^

如果你不想通过一般的基于Emacs的方式加载swank，只需要加载swank-load.lisp文件就可以了。只需要在一个运行中的Lisp镜像 [#f1]_ 里执行以下代码：

::

   (load "/path/to/swank-loader.lisp")

现在，我们需要做的就是启动swank服务器。在第一个例子里，我们假设使用默认配置。

::

   (swank:create-server)


由于我们将要使用ssh [#f2]_ 来建立链接并且只打开了一个端口，我们不希望swank使用另一个端口作为输出（目前这在swank里是默认的）：

::

   (setf swank:*use-dedicated-output-stream* nil)


如果你有其它特别的需求（例如在结束后重新连接到swank），请查看swank:create-server的其它参数。其中的一些参数如下：

* :PORT

  指定服务器所监听的端口号（默认端口：4005）

* :STYLE

  见 6.2.1 通信模式

* :DONE-CLOSE

  布尔值，指明了在服务器在接受了第一个连接后是否还会接收其它连接（默认值：nil）。对于你希望可以随时连接的长期运行的Lisp进程，指定:done-close t

* :CODING-SYSTEM

  字符串，指明了Emacs和Lisp之间进行通讯的编码

更加完整的实例如下：

::

   (swank:create-server :port 4005  :dont-close t :coding-system "utf-8-unix")

在Emacs端，你会进行类似如下的设置来连接到同一台机器上的Lisp镜像：

::

   (setq slime-net-coding-system 'utf-8-unix)
   (slime-connect "127.0.0.1" 4005)

7.1.2 设置Emacs
^^^^^^^^^^^^^^^^

现在我们需要在本地机器和远程机器之间建立连接。

::

   ssh -L4005:127.0.0.1:4005 username@remote.example.com


这里调用的ssh在本地机器的4005端口和远程机器的4005端口上建立了一个ssh连接 [#f3]_ 。

最后，我们启动SLIME：

::

   M-x slime-connect RET RET

RET RET按键表示我们要使用默认主机（127.0.0.1）和默认端口（4005）。虽然我们是连接到远程机器上的，ssh连接让Emacs以为我们是在本地操作。

7.1.3 设置路径名翻译
^^^^^^^^^^^^^^^^^^^^^

远程运行swank的一个主要问题就是，Emacs以为可以通过普通的文件名找到文件。如果我们希望例如slime-compile-and-load-file （C-c C-k）或者slime-edit-definition （M-.）这样的函数正常工作，我们需要找到一种方法让本地的Emacs正确地找到远程文件。

要做到这一点主要有两种方式。第一种是挂载，使用NFS或者类似的东西。将远程机器的硬盘挂载到本地机器的文件系统上，这样的话例如/opt/project/source.lisp的文件名就可以在两台机器上都指向同一个文件。不幸的是，NFS很慢，而且容易出错，并且不总是可行的。幸运的是我们有ssh连接以及Emacs的tramp-mode来完成这项工作。

我们需要做的事情是让Emacs接收一个远程机器上的文件名，然后将其翻译为某种tramp可以理解和使用的格式，反之亦然。假设远程机器的主机名叫做remote.example.com，cl:machine-instance返回“remote”，我们以“user”用户登陆，我们使用slime-tramp扩展包来设置适当的翻译方式，如下：

::

   (push (slime-create-filename-translator :machine-instance "remote.example.com"
                                           :remote-host "remote"
                                           :username "user")
         slime-filename-translations)

7.2 重定向全局IO到REPL
----------------------

默认情况下SLIME并不会改变*standard-output*和REPL以外的其它事物。如果你有一些其它的线程，例如format，write-string等等，相应的输出仅仅能在*inferior-lisp*缓冲区或者是终端下看到，通常来说这很不方便。所以如果你有这样的代码：

::

   (run-in-new-thread
       (lambda ()
           (write-line "In some random thread.~%" *standard-output*)))

并且想让它输出到SLIME的REPL缓冲区而不是\*inferior-lisp\*缓冲区，只需要将swank:\*globally-redirect-io\*设置为T。

注意，这个变量的值只会在swank接收连接的时候被检查，所以你需要把它写在~/.swank.lisp文件里。否则你要自己调用swank::globally-redirect-io-to-connection命令，但是你不应该这样做除非你知道原因。

7.3 自动连接到SLIME
---------------------

如果想要在打开一个Lisp文件的时候自动启动SLIME，将以下设置加入~/.emacs文件里：

::

   (add-hook 'slime-mode-hook
             (lambda ()
               (unless (slime-connected-p)
                 (save-excursion (slime)))))

脚注

.. rubric:: Footnotes

.. [#f1] SLIME也提供了一个功能相同的ASDF系统定义

.. [#f2] 有一种不使用ssh来连接的方法，但是其副作用是将允许所有东西连接到你的Lisp镜像，所以我们不讨论这种方式

.. [#f3] 默认情况下swank监听来自4005端口的连接，如果我们调用swank:create-server函数时指定:port参数，我们就可以使用其它端口了。
