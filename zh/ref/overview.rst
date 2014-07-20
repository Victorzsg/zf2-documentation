.. _introduction.overview:

********
概述
********

Zend Framework 2 是用来开发web应用和服务的开源软件，基于PHP5.3+版本。Zend Framework 2 使用100% `面向对象`_ 编程，使用了PHP5.3的大部分新特性，例如 `命名空间`_, `延迟静态绑定`_, `lambda函数和闭包`_。

Zend Framework 2 从Zend Framework 1 进化而来，它是一款超过1500万下载量的成功的PHP框架。

.. note::

    *ZF2* 并不乡下兼容 *ZF1*，这是由于使用了PHP5.3+的新特性，并重写了它的主要组件。

Zend framework 2组件结构是独特的；每个组件被设计的很少依赖其他组件。zf2遵循 `SOLID`_ 面向对象设计原则。这个松散耦合的体系结构允许开发人员使用他们想要的任何组件。我们称此为“松耦合”设计。对于整个框架和每一个组件，我们支持 `Pyrus`_ 和 `Composer`_ 安装以及依赖跟踪机制，更加增强了这种设计。

我们用 `PHP单元测试`_ 来检测我们的代码，用 `Travis CI`_ 作为持续集成服务。

尽管它们可以分开使用，但是相结合时，Zend Framework 2 标准库中的组件形成一个强大的、可扩展的web应用程序框架。 Also, it offers a robust, high performance `MVC`_ implementation, a
database abstraction that is simple to use, and a forms component that implements `HTML5 form rendering`_,
validation, and filtering so that developers can consolidate all of these operations using one easy-to-use, object
oriented interface. Other components, such as :doc:`Zend\\Authentication <zend.authentication.introduction>` and
:doc:`Zend\\Permissions\\Acl <zend.permissions.acl.introduction>`, provide user authentication and authorization against
all common credential stores. 

Still others, with the ``ZendService`` namespace, implement client libraries to simply access the most
popular web services available. Whatever your application needs are, you're likely to find a Zend Framework 2
component that can be used to dramatically reduce development time with a thoroughly tested foundation.
 
The principal sponsor of the project 'Zend Framework 2' is `Zend Technologies`_, but many companies have contributed 
components or significant features to the framework. Companies such as Google, Microsoft, and StrikeIron have 
partnered with Zend to provide interfaces to web services and other technologies they wish to make available 
to Zend Framework 2 developers.

Zend Framework 2 could not deliver and support all of these features without the help of the vibrant Zend Framework 2
community. Community members, including contributors, make themselves available on `mailing lists`_, 
`IRC channels`_ and other forums. Whatever question you have about Zend Framework 2, the community is always 
available to address it.

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
