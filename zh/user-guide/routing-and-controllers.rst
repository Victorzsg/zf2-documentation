.. _user-guide.routing-and-controllers:

路由和控制器
=======================

我们要创建一个简单的唱片库存系统，首页列出我们的库存，并且允许我们添加，编辑和删除唱片。所以，需要以下几个页面：

+---------------+------------------------------------------------------------+
| 页面          | 描述                                                |
+===============+============================================================+
| Home          | 这里会展示我们的唱片，并有编辑和删除他们的链接，           |
|               | 还有一个添加唱片的链接。                                   |
+---------------+------------------------------------------------------------+
| Add new album | 这个页面有一个添加唱片的表单                               |
+---------------+------------------------------------------------------------+
| Edit album    | 这个页面有一个编辑唱片的表单                               |
+---------------+------------------------------------------------------------+
| Delete album  | 这个页面确认我们是否删除并可以删除唱片                     |
+---------------+------------------------------------------------------------+

在开始编辑文件之前，理解zf框架需要怎么组织页面是很重要的。应用中的每一个页面
对应于 *模型* *控制器* 中的一组 *方法* 。因此，你通常把一组相关的方法放在一个控制器；例如，一个新闻控制器可能有 ``current``，``archived`` 和 ``view`` 方法。

由于我们的4个页面都和唱片相关，所以我们用 ``Album`` 模型的一个控制器 ``AlbumController`` 中四个方法来管理他们。这4个方法是：

+---------------+---------------------+------------+
| 页面          | 控制器              | 方法       |
+===============+=====================+============+
| Home          | ``AlbumController`` | ``index``  |
+---------------+---------------------+------------+
| Add new album | ``AlbumController`` | ``add``    |
+---------------+---------------------+------------+
| Edit album    | ``AlbumController`` | ``edit``   |
+---------------+---------------------+------------+
| Delete album  | ``AlbumController`` | ``delete`` |
+---------------+---------------------+------------+

通过模型的配置文件 ``module.config.php`` 来设置路由，使URL映射到特定的操作。

.. code-block:: php
   :linenos:
   :emphasize-lines: 9-27

    return array(
        'controllers' => array(
            'invokables' => array(
                'Album\Controller\Album' => 'Album\Controller\AlbumController',
            ),
        ),

        // The following section is new and should be added to your file
        'router' => array(
            'routes' => array(
                'album' => array(
                    'type'    => 'segment',
                    'options' => array(
                        'route'    => '/album[/][:action][/:id]',
                        'constraints' => array(
                            'action' => '[a-zA-Z][a-zA-Z0-9_-]*',
                            'id'     => '[0-9]+',
                        ),
                        'defaults' => array(
                            'controller' => 'Album\Controller\Album',
                            'action'     => 'index',
                        ),
                    ),
                ),
            ),
        ),

        'view_manager' => array(
            'template_path_stack' => array(
                'album' => __DIR__ . '/../view',
            ),
        ),
    );

路由名是 `album`，它有一个 `segment` 类型。匹配路由时，segment路由允许我们在URL模式中指定占位符来匹配相应的参数。在这种情况下，路由是 ``/album[/:action][/:id]`` ，它会匹配任何以 ``/album`` 开头的URL。下一段是可选的方法名，再下一段将会匹配可选的id，方括号表明这一段是可选的。这些约束可以确保这段字符是我们需要的，所以我们限制我们的方法以一个字母开头，后面跟着字母数字、下划线或者连接符。我们也限制id是一个数字。

这个路由允许我们有以下URLs：

+---------------------+------------------------------+------------+
| URL                 | Page                         | Action     |
+=====================+==============================+============+
| ``/album``          | Home (list of albums)        | ``index``  |
+---------------------+------------------------------+------------+
| ``/album/add``      | Add new album                | ``add``    |
+---------------------+------------------------------+------------+
| ``/album/edit/2``   | Edit album with an id of 2   | ``edit``   |
+---------------------+------------------------------+------------+
| ``/album/delete/4`` | Delete album with an id of 4 | ``delete`` |
+---------------------+------------------------------+------------+

创建控制器
=====================

现在我们来创建控制器。在Zend Framework 2中，控制器是一个通常被称为 ``{Controller name}Controller``的一个类。注意 ``{控制器名}`` 必须以大写字母开头。这个控制器在模型 ``Controller`` 目录的叫做 ``{Controller name}Controller.php`` 的文件中，我们这里是 ``module/Album/src/Album/Controller``。每个方法是控制器类中的一个public方法，``{方法名}`` 必须以小写字母开头 ``{action name}Action``。

.. note::

    习惯上，zf2对控制器没有太多的限制，除了它们必须继承自 ``Zend\Stdlib\Dispatchable`` 接口。框架的这种机制是通过两个抽象类 ``Zend\Mvc\Controller\AbstractActionController`` 和 ``Zend\Mvc\Controller\AbstractRestfulController`` 实现的。我们将使用标准的 ``AbstractActionController``，但是如果想写一个基于REST的web服务，使用 ``AbstractRestfulController`` 是很好的选择。

我们开始在 ``zf2-tutorials/module/Album/src/Album/Controller`` 中创建我们的的控制器类 ``AlbumController.php``：

.. code-block:: php
   :linenos:

    namespace Album\Controller;

    use Zend\Mvc\Controller\AbstractActionController;
    use Zend\View\Model\ViewModel;

    class AlbumController extends AbstractActionController
    {
        public function indexAction()
        {
        }

        public function addAction()
        {
        }

        public function editAction()
        {
        }

        public function deleteAction()
        {
        }
    }
    
.. note::

    确保在 ``config/application.config.php`` 的"modules"部分添加一个新的 ``Album`` 模型。你还得为此模型提供一个 :ref:`模型类<zend.module-manager.module-class>` 来保证Album模型可以被MVC框架识别。

.. note::

    在 ``module/Album/config/module.config.php`` 文件的‘controller’部分，我们已经把我们的控制器通知给了模型。

现在来编写我们想使用的这四个方法。要先设置视图，不然它们不会运行。每个方法的URLs是：

+------------------------------------------------+----------------------------------------------------+
| URL                                            | Method called                                      |
+================================================+====================================================+
| ``http://zf2-tutorial.localhost/album``        | ``Album\Controller\AlbumController::indexAction``  |
+------------------------------------------------+----------------------------------------------------+
| ``http://zf2-tutorial.localhost/album/add``    | ``Album\Controller\AlbumController::addAction``    |
+------------------------------------------------+----------------------------------------------------+
| ``http://zf2-tutorial.localhost/album/edit``   | ``Album\Controller\AlbumController::editAction``   |
+------------------------------------------------+----------------------------------------------------+
| ``http://zf2-tutorial.localhost/album/delete`` | ``Album\Controller\AlbumController::deleteAction`` |
+------------------------------------------------+----------------------------------------------------+

现在我们有了正常运行的路由和并编写好了每个页面的方法。

改编写视图和模型层了。

初始化视图脚本
---------------------------

为了把视图整合进我们的应用，我们需要创建一些视图脚本文件。这些文件会被 ``DefaultViewStrategy`` 运行并通过控制器方法传递任意的变量和模型视图。这些文件保存在模型的以控制器命名的视图目录。现在创建这4个文件：

* ``module/Album/view/album/album/index.phtml``
* ``module/Album/view/album/album/add.phtml``
* ``module/Album/view/album/album/edit.phtml``
* ``module/Album/view/album/album/delete.phtml``

配置好这些，我们就可以配置我们的数据库和模型啦。