.. _zendtool.introduction:

Zend Framework Tool (ZFTool)
============================

`ZFTool`_ 是一个维护模块化zf2应用的实用工具模块。运行在命令行，可以通过zf2模型或者PHAR（看下面）安装。此工具有以下功能：

   - 创建zf2项目，安装一个框架应用；

   - 在已经存在的zf应用中创建新的模型；

   - 列出安装在应用中的所有模块；

   - 获取zf2应用的配置文件；

   - 安装特定版本的zf2类库。

安装ZFTool，只需用下列方法中的一个或者你只用下载并使用PHAR包。

使用 `Composer`_ 安装
------------------------------

    1. 打开控制台 （命令行）
    2. 切换到应用目录
    3. 运行 `composer require zendframework/zftool:dev-master`

手动安装
-------------------

    1. 用 `git` clone或者 `下载zip包`_
    2. 解压到zf应用的 `vendor/ZFTool` 目录
    3. 进入 `vendor/ZFTool` 目录执行 `zf.php` 得到如下结果

无需安装，使用PHAR文件
-----------------------------------------

    1. You don't need to install ZFTool if you want just use it as a shell command.
       You can `download zftool.phar`_ and use it.
       你不必安装ZFTool，如果你只是把它用作一个shell命令，你可以 `下载 zftool.phar`_ 并使用它。

用法
-----

用 Composer 或手动安装：
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

`zf.php` 文件应该安装到 `vendor/ZFTool` 目录（相对于项目根目录）—— 但是，命令需要在项目根目录运行才行。你可以链接 `vendor/ZFTool/zf.php` 到你的项目根目录，或者像下面的例子一样，用 `vendor/ZFTool/zf.php` 替代 `zf.php`。

使用 PHAR:
^^^^^^^^^^^^^^^

在下面的例子中，简单替换 `zftool.phar` 为 `zf.php`。


基本信息
^^^^^^^^^^^^^^^^^

.. code-block:: bash

    > zf.php modules [list]           显示加载的模块

*modules* 项列出了zf2应用安装的所有模块。

.. code-block:: bash

    > zf.php version | --version      显示当前 Zend Framework 版本

*version* 项给出了ZFTool的版本，如果从zf2应用的根目录执行，还有应用使用的zf类库版本号。

项目创建
^^^^^^^^^^^^^^^^

.. code-block:: bash

    > zf.php create project <path>

    <path>              项目路径

此命令把 `ZendSkeletonApplication`_ 安装到指定目录。

模块创建
^^^^^^^^^^^^^^^

.. code-block:: bash

    > zf.php create module <name> [<path>]

    <name>              要创建的模块名
    <path>              到zf2应用根目录的路径（可选）

此命令可用于在现有的ZF2应用程序内创建一个新模块。如果没有提供路径，ZFTool会尝试在当前目录创建一个模块（除非本地文件夹包含一个zf2应用）。

Classmap 生成器
^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    > zf.php classmap generate <directory> <classmap file> [--append|-a] [--overwrite|-w]

    <directory>         PHP 类要扫描的目录 （使用 "." 代表当前目录）
    <classmap file>     生成类映射文件或者标准输出文件的文件名，如果没有提供，默认为<directory>里的autoload_classmap.php 
    --append | -a       如果存在添加到类映射文件
    --overwrite | -w    是否要覆盖现有classmap文件

ZF 类库安装
^^^^^^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    > zf.php install zf <path> [<version>]

    <path>              ZF2 类库安装目录
    <version>           要安装的版本，如果未指定使用最新的

此命令安装指定版本的ZF2库到某一路径中。如果版本没有指定，它将使用最新稳定版本。使用此命令你可以安装 `ZF2 github`_ 的所有tag版本（版本名称是去掉tag名称的 *'release-'* 字符串；例如，标记'release-2.0.0'相当于版本号2.0.0）。

编译 PHAR 文件
^^^^^^^^^^^^^^^^^^^^^

你可以创建一个包含ZFTool项目的 .phar 文件，创建此文件，你需要运行下面的命令：

.. code-block:: bash

    > bin/create-phar

此命令将会在bin目录创建一个 *zftool.phar* 文件。你可以使用此文件执行所有ZFTool功能。
创建 *zftool.phar* 之后，我们建议把ZFTool的bin目录添加在您的环境变量。这样，不管你在那个目录，都可以执行 *zftool.phar* 脚本了。

.. _`ZFTool`: https://github.com/zendframework/ZFTool
.. _`Composer`: http://getcomposer.org
.. _`download zipball`: https://github.com/zendframework/ZFTool/zipball/master
.. _`download zftool.phar`: https://packages.zendframework.com/zftool.phar
.. _`ZendSkeletonApplication`: https://github.com/zendframework/ZendSkeletonApplication
.. _`ZF2 github`: https://github.com/zendframework/zf2
