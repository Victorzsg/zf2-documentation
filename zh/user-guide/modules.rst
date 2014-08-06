.. _user-guide.modules:

模型
=======

Zend Framework 2使用模型系统的每一个module来组织应用的每一个功能。骨架应用的module为整个应用提供引导，错误和路由配置。module通常用来给应用级的控制器提供功能，比如，一个应用的主页，但是这里我们不打算使用教程默认的module，我们希望使用自己的module使我们的列表页作为首页。

我们打算把代码放在Album模型，它包含controllers，modules，和configuration。我们可以根据需要调整这个模型。

现在开始创建需要的目录。

设置Album模型
---------------------------

在有以下子目录的 ``module`` 中创建一个 ``Album`` 的目录，用它来保存模型的文件：

.. code-block:: text
   :linenos:

    zf2-tutorial/
        /module
            /Album
                /config
                /src
                    /Album
                        /Controller
                        /Form
                        /Model
                /view
                    /album
                        /album

就像上面一样，我们的文件放在 ``Album`` 模型的几个目录里。``Album`` 命名空间包含的PHP类文件保存在 ``src/Album`` 目录，需要的时候，可以在module中使用多个命名空间。视图目录也有一个 ``Album`` 子目录，用来存放模型的视图脚本。

zf2用 ``ModuleManager`` 来加载和配置一个module。它们会去模型目录（``module/Album``）根目录中的 ``Module.php`` 文件查找一个叫做 ``Album/Module`` 的类。就是说，特定模型的类会有该模型的命名空间，即模型的目录名。

在 ``Album`` 模型创建一个 ``Module.php`` 文件：
在 ``zf2-tutorial/module/Album`` 里创建一个名叫 ``Module.php`` 的文件：

.. code-block:: php
   :linenos:

    namespace Album;

    use Zend\ModuleManager\Feature\AutoloaderProviderInterface;
    use Zend\ModuleManager\Feature\ConfigProviderInterface;

    class Module implements AutoloaderProviderInterface, ConfigProviderInterface
    {
        public function getAutoloaderConfig()
        {
            return array(
                'Zend\Loader\ClassMapAutoloader' => array(
                    __DIR__ . '/autoload_classmap.php',
                ),
                'Zend\Loader\StandardAutoloader' => array(
                    'namespaces' => array(
                        __NAMESPACE__ => __DIR__ . '/src/' . __NAMESPACE__,
                    ),
                ),
            );
        }

        public function getConfig()
        {
            return include __DIR__ . '/config/module.config.php';
        }
    }

``ModuleManager`` 会为我们自动调用 ``getAutoloaderConfig()`` 和 ``getConfig()``。

自动加载文件
^^^^^^^^^^^^^^^^^

``getAutoloaderConfig()`` 方法返回一个ZF2的 ``AutoloaderFactory`` 数组。配置它以添加一个 ``ClassMapAutoloader`` 类映射文件，另外，把这个模型的命名空间也添加到 ``StandardAutoloader``。标准的自动加载需要一个命名空间和找到命名空间文件的路径。这符合 `PSR-0 <https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md>`_ 命名标准，通过该标准，类直接映射到文件。

由于在开发中，不需要通过映射类加载文件，所以我们给自动加载映射类提供了一个空数组。在 ``zf2-tutorial/module/Album`` 中创建一个  ``autoload_classmap.php`` 文件：

.. code-block:: php
   :linenos:

    return array();

由于是空数组，所以无论何时自动加载机去查找 ``Album`` 命名空间中的文件，都会转向 ``StandardAutoloader`` 。

.. 注意::

    如果使用Composer，你只用创建一个空的 ``getAutoloaderConfig() { }`` 并添加到composer。
    json:

    .. code-block:: javascript
       :linenos:

        "autoload": {
            "psr-0": { "Album": "module/Album/src/" }
        },

    如果以这种方式，你得运行 ``php composer.phar update`` 命令来升级composer自动加载文件。

配置
-------------

注册了自动加载，让我们来看看 ``Album\Module`` 中的 ``getConfig()`` 方法。这个方法只是加载了 ``config/module.config.php`` 文件。

在 ``zf2-tutorial/module/Album/config`` 中创建 ``module.config.php`` 文件：

.. code-block:: php
   :linenos:

    return array(
        'controllers' => array(
            'invokables' => array(
                'Album\Controller\Album' => 'Album\Controller\AlbumController',
            ),
        ),
        'view_manager' => array(
            'template_path_stack' => array(
                'album' => __DIR__ . '/../view',
            ),
        ),
    );

``ServiceManager`` 会把配置信息传递给相关组件。我们需要 ``controllers`` 和 ``view_manager`` 两个初始化部分。控制器部分提供了模块的控制器列表。控制器  ``AlbumController`` 放在 ``Album\Controller\Album`` ，这个控制器的键在所有模型中必须是唯一的，所以我们以我们的模型名作为它的前缀。

在 ``view_manager`` 部分，把视图目录添加到 ``TemplatePathStack`` 配置，这样它就会找到放在 ``view/`` 目录的模块视图脚本。

把新模块告诉给应用程序
----------------------------------------------

现在得告诉 ``ModuleManager`` 新模块的存在。这是通过骨架应用的 ``config/application.config.php`` 配置文件实现的。修改这个文件，使它的 ``modules`` 部分也包含 ``Album`` ，这样，文件看起来应该是这样的：

（修改高亮显示的注释部分）

.. code-block:: php
   :linenos:
   :emphasize-lines: 4

    return array(
        'modules' => array(
            'Application',
            'Album',                  // <-- Add this line
        ),
        'module_listener_options' => array(
            'config_glob_paths'    => array(
                'config/autoload/{,*.}{global,local}.php',
            ),
            'module_paths' => array(
                './module',
                './vendor',
            ),
        ),
    );

你看，我们已经把 ``Album`` 模块添加到模块列表的 ``Application`` 模块后面。

现在，我们已经设置好了模块，可以在里面编写我们的代码了。
