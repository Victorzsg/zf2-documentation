.. _introduction.installation:

************
安装
************

.. See the :ref:`requirements appendix <requirements>` for a detailed list of requirements for Zend Framework.

- **不了解Zend Framework？** 
  `下载最新的稳定版本`_。有 ``.zip`` 和 ``.tar.gz`` 格式可供下载。

- **体验最新尖端特性？**
  使用 `Git`_ 客户端下载 `Zend Framework的Git代码库`_。Zend Framework是开源软件，在 `GitHub`_ 上Git仓库对开发者是公开的。如果你想给zf框架做贡献，或者相对于发布版经常需要修改您的框架，可以考虑使用Git来获取框架。

一旦有了复制了zf框架，你的应用需要能够访问到放在library目录的zf的类。有 `几种方法`_。

找不到zf2的安装在哪里，会发生以下错误::

 Fatal error: Uncaught exception 'RuntimeException' with message
 'Unable to load ZF2. Run `php composer.phar install` or define 
 a ZF2_PATH environment variable.'

解决这个问题，你可以把zf的类文件路径添加到 *PHP* 的 `include_path`_。还有，要在httpd.conf（或同类）中设置一个 'ZF2_PATH' 环境变量，例如在Linux下 ``SetEnv ZF2_PATH /var/ZF2``。

友善的 `Rob Allen`_ 已经给社区提供了一个入门教程，`Getting Started with Zend Framework 2`_。其他的zf社区成员也在  `积极的扩展这个教程`_。



.. _`Download the latest stable release.`: http://packages.zendframework.com/
.. _`Git`: http://git-scm.com/
.. _`GitHub`: http://github.com/
.. _`Zend Framework's Git repository`: https://github.com/zendframework/zf2
.. _`several ways to achieve this`: http://www.php.net/manual/en/configuration.changes.php
.. _`include_path`: http://www.php.net/manual/en/ini.core.php#ini.include-path
.. _`Rob Allen`: http://akrabat.com/about
.. _`Getting Started with Zend Framework 2`: http://zf2.readthedocs.org/en/latest/user-guide/overview.html
.. _`expanding the tutorial`: http://zend-framework-community.634137.n4.nabble.com/zf2-tutorial-td4656144.html
