.. _user-guide.overview:

开始学习Zend Framework 2
=====================================

这个教程试图通过创建一个简单的MVC数据库驱动应用，来说明如何使用zf2。最后你会得到一个能够运行的zf2应用程序，然后你再摸索更多关于它们是怎么协调运行的代码。

.. _user-guide.overview.assumptions:

一些假设
----------------

这个教程假设你已经安装了5.3.23以上的版本的PHP，还有Apache服务器和通过PDO扩展可以连接的MySQL数据库，你的Apache必须安装并开启了mod_rewrite扩展。

必须确保Apache配置了支持 ``.htaccess`` 文件。通常可以这样设置：

.. code-block:: apache
   :linenos:

    AllowOverride None

to

.. code-block:: apache
   :linenos:

    AllowOverride FileInfo

在 ``httpd.conf`` 文件。 查看发行文档可以获得更多的细节。如果没有正确的配置mod_rewrite和.htaccess，除了教程中的首页，你访问不了任何页面。

.. 注意::

   另外，如果你使用5.4以上的PHP版本，你可能使用built-in服务器代替Apache来开发。

教程的示例应用
------------------------

我们即将创建的应用，是一个展示我们有多少唱片的简单的库存系统，主页会列出我们仓库的唱片，并且允许我们添加，编辑，删除CD，在我们的站点中，我们需要四个页面：

+----------------+------------------------------------------------------------+
| Page           | Description                                                |
+================+============================================================+
| List of albums | This will display the list of albums and provide links to  |
|                | edit and delete them. Also, a link to enable adding new    |
|                | albums will be provided.                                   |
+----------------+------------------------------------------------------------+
| Add new album  | This page will provide a form for adding a new album.      |
+----------------+------------------------------------------------------------+
| Edit album     | This page will provide a form for editing an album.        |
+----------------+------------------------------------------------------------+
| Delete album   | This page will confirm that we want to delete an album and |
|                | then delete it.                                            |
+----------------+------------------------------------------------------------+

我们需要把我们的数据存入数据库，只需要一张表保存这些字段：

+------------+--------------+-------+-----------------------------+
| Field name | Type         | Null? | Notes                       |
+============+==============+=======+=============================+
| id         | integer      | No    | Primary key, auto-increment |
+------------+--------------+-------+-----------------------------+
| artist     | varchar(100) | No    |                             |
+------------+--------------+-------+-----------------------------+
| title      | varchar(100) | No    |                             |
+------------+--------------+-------+-----------------------------+

