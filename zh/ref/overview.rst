.. _introduction.overview:

********
概述
********

Zend Framework 2 是用来开发web应用和服务的开源软件，基于PHP5.3+版本。Zend Framework 2 使用100% `面向对象`_ 编程，使用了PHP5.3的大部分新特性，例如 `命名空间`_, `延迟静态绑定`_, `lambda函数和闭包`_。

Zend Framework 2 从Zend Framework 1 进化而来，它是一款超过1500万下载量的成功的PHP框架。

.. note::

    由于使用了PHP5.3+的新特性，并重写了主要组件，*ZF2* 并不向下兼容 *ZF1*。

Zend framework 2组件结构是独特的；每个组件很少依赖其他组件。zf2遵循 `SOLID`_ 面向对象设计原则。这种松散耦合结构允许开发人员使用他们想要的任何组件。我们称此为“松耦合”设计。整个框架和每一个组件都支持 `Pyrus`_ 和 `Composer`_ 安装以及依赖跟踪机制，这更加增强了这种理念。

我们用 `PHP单元测试`_ 来检测我们的代码，用 `Travis CI`_ 作为持续集成服务。

尽管它们可以分开使用，但相结合时，Zend Framework 2 标准库中的组件形成一个强大的、可扩展的web应用程序框架。它提供了健壮的、性能高效的 `MVC`_ 方案，数据库抽象简单易用，表单组件使用 `HTML5表单渲染`_，验证并过滤。因此，开发人员使用一个易于使用的、面向对象的接口整合所有这些操作。其它组件，例如 :doc:`Zend\\Authentication <zend.authentication.introduction>` 和
:doc:`Zend\\Permissions\\Acl <zend.permissions.acl.introduction>`，提供用户身份验证和所有公共凭据存储的授权。

另外，有了 ``ZendService`` 命名空间，客户端库可以直接访问最流行的web服务。无论您的应用程序需要什么，都可能找到一个经过彻底测试的Zf2组件，大大减少开发时间。

`Zend Technologies`_ 是 'Zend Framework 2' 的主要赞助者，但是许多公司为框架的组件或特性作出了贡献。例如谷歌、微软和StrikeIron与Zend一道，提供了zf2开发者们希望使用的接口、web服务以及其他技术。
 
没有Zend Framework 2社区的帮助，zf2无法交付和支持所有这些功能。社区成员以及贡献者们，可以通过 `mailing lists`_，`IRC channels`_ 和别的论坛找到。有什么关于zf2的问题，可以去下面这些社区。

.. _`object-oriented`: http://en.wikipedia.org/wiki/Object-oriented_programming
.. _`namespaces`: http://php.net/manual/en/language.namespaces.php
.. _`late static binding`: http://php.net/lsb
.. _`lambda functions and closures`: http://php.net/manual/en/functions.anonymous.php
.. _`SOLID`: http://en.wikipedia.org/wiki/SOLID_%28object-oriented_design%29
.. _`Pyrus`: http://pear.php.net/manual/en/pyrus.php
.. _`Composer`: http://getcomposer.org/
.. _`PHPUnit`: http://www.phpunit.de
.. _`Travis CI`: http://travis-ci.org/
.. _`MVC`: http://en.wikipedia.org/wiki/Model%E2%80%93view%E2%80%93controller#PHP
.. _`HTML5 form rendering`: http://www.w3.org/TR/html5/forms.html#forms
.. _`Zend Technologies`: http://www.zend.com
.. _`mailing lists`: http://framework.zend.com/archives
.. _`IRC channels`: http://www.zftalk.com
