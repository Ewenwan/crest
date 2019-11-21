CREST
=====

crest是一款针对C语言的自动测试工具，支持深度优先（DFS）、控制流图（CFG）、一致随机（uniform random）和随机分支（random branch）四种搜索策略，使用符号执行技术，使用Yices作为约束求解器进行求解。

安装步骤

安装Cmaml：由于Crest需要CIL的支持，而CIL需要Ocaml的支持，所以先安装Ocaml，在googlegroup中作者说Ocaml 4.01.0可以bugfree，否则CIL可能会停止响应，但是我直接用的Ubuntu的apt-get，安装的3.12.1，运行起来也没啥问题。

安装Yices，这个版本是特别古老的版本，Yices官网都已经给的是Yices2了，但是也提供老版本的下载，只是没有源码了，只有编译好的binary，自己按需下载，依赖包这种东西，错一个版本几乎就不能用，我不信邪，装了Yices2，结果。。我还是用了Yices1.0.40。之后按照Crest的要求，把crest/src/Makefile Yices的目录更新一下，方便编译。

安装CIL：依赖包都安装完了，现在开始安装Crest，但是之前，要先安装CIL，去Crest/CIL/目录configure一下，看看缺什么包自己安装就好了，不过一般都是ocaml-find和libgmp-dev两个包。configure通过之后再make就好了。

安装Crest：去Crest/src/目录下，make即可完成安装。


test实验

github上面给了一个test的运行例子，我也就是照着做了一遍，大概流程如下：
首先，测试的C程序是test目录下的uniform_test.c，关键代码如下：

```c
if (a == 5) {
    if (b == 19) {
        if (c == 7) {
            if (d == 4) {
                fprintf(stderr, "GOAL!n");
            }
        }
    }
}
```
可以看到，如果约数求解成功，应该有八个分支，并且其中一个分支应该输出“GOAL！”。然后把程序改装一下，需要进行一些代码插桩，比如将变量利用CREST的函数声明，然后输出的时候使用stderr进行输出，这样就可以看到例子了。这些事情test的代码都已经做好，具体可以看看test是怎么做的。然后使用bin目录下的crest对代码进行插桩和编译工作。使用run_crest运行已经插桩好的二进制文件，可以得出如下输出：

Iteration 0 (0s): covered 0 branches [0 reach funs, 0 reach branches].
Iteration 1 (0s): covered 1 branches [1 reach funs, 8 reach branches].
Iteration 2 (0s): covered 3 branches [1 reach funs, 8 reach branches].
Iteration 3 (0s): covered 5 branches [1 reach funs, 8 reach branches].
Iteration 4 (0s): covered 7 branches [1 reach funs, 8 reach branches].
GOAL!
Iteration 5 (0s): covered 8 branches [1 reach funs, 8 reach branches].

可以看到，已经得出了8个分支，而且也输出了“GOAL！”。任务算是完成了吧。

PS：这仅仅是对单个C文件的测试，据作者说还可以测试工程，等回来再行实验。

CREST is a concolic test generation tool for C.

Thanks for downloading and trying out CREST.

You can get the latest version of CREST, as well as news and
announcements at CREST's homepage: https://burn.im/crest .

If you want to cite CREST, please refer to the (short) paper: Burnim,
Sen, "Heuristics for Dynamic Test Generation", Proceedings of the 23rd
IEEE/ACM International Conference on Automated Software Engineering
(ASE), 2008.

You can find a list of papers using CREST at
https://burn.im/crest/#publications .

NOTE: CREST is no longer being actively developed, but questions are
still answered on the CREST-users mailing list --
crest-users@googlegroups.com and
https://groups.google.com/forum/#!forum/crest-users .


Preparing a Program for CREST
=====

To use CREST on a C program, use functions CREST_int, CREST_char,
etc., declared in "crest.h", to generate symbolic inputs for your
program.  For examples, see the programs in test/.

For simple, single-file programs, you can use the build script
"bin/crestc" to instrument and compile your test program.

CREST can be used to instrument multi-file programs, too --
instructions may be added later.  In the meantime, you can take a look
at an example, instrumented form of grep-2.2, available at
https://github.com/jburnim/crest/tree/master/benchmarks/grep-2.2 .
For further information, please see this
[post](https://groups.google.com/forum/#!topic/crest-users/KwgP9JkajOw)
on the CREST-users mailing list.


Running Crest
=====

CREST is run on an instrumented program as:

    bin/run_crest PROGRAM NUM_ITERATIONS -STRATEGY

Possibly strategies include: dfs, cfg, random, uniform_random, random_input.
Some strategies take optional parameters.

Example commands to test the "test/uniform_test.c" program:

    cd test
    ../bin/crestc uniform_test.c
    ../bin/run_crest ./uniform_test 10 -dfs

This should produce output roughly like:

    ... [GARBAGE] ...
    Read 8 branches.
    Read 13 nodes.
    Wrote 6 branch edges.

    Iteration 0 (0s): covered 0 branches [0 reach funs, 0 reach branches].
    Iteration 1 (0s): covered 1 branches [1 reach funs, 8 reach branches].
    Iteration 2 (0s): covered 3 branches [1 reach funs, 8 reach branches].
    Iteration 3 (0s): covered 5 branches [1 reach funs, 8 reach branches].
    Iteration 4 (0s): covered 7 branches [1 reach funs, 8 reach branches].
    GOAL!
    Iteration 5 (0s): covered 8 branches [1 reach funs, 8 reach branches].

NOTE: run_crest and crestc currently leave a lot of files lying
around, some of which are temporary and some of which must be kept.
In particular, "cfg_branches" and "branches" are output by the
instrumentation process and are needed to run run_crest, and run_crest
produces "coverage", a list of the ID's of all covered branches.


Setup
=====

CREST depends on Yices 1, an SMT solver tool and library available at
http://yices.csl.sri.com/old/download-yices1.shtml.  To build and run
CREST, you must download and install Yices *version 1* and change
YICES_DIR in src/Makefile to point to Yices location.

CREST uses CIL to instrument C programs for testing.  A modified
distribution of CIL is included in directory cil/.  To build CIL,
simply run "configure" and "make" in the cil/ directory.

Finally, CREST can be built by running "make" in the src/ directory.


License
=====

CREST is distributed under the revised BSD license.  See LICENSE for
details.

This distribution includes a modified version of CIL, a tool for
parsing, analyzing, and transforming C programs.  CIL is written by
George Necula, Scott McPeak, Wes Weimer, Ben Liblit, and Matt Harren.
It is also distributed under the revised BSD license.  See cil/LICENSE
for details.
