.. _user-guide.modules:

Modules
=======

Zend Framework 2使用module系统的每一个module来组织你的每一个应用功能。骨架的应用module为整个应用提供引导，错误和路由配置。module通常用来给应用级的控制器提供功能，比如，一个应用的主页，但是这里我们不打算使用教程默认的module，我们希望使用自己的module使我们的列表页作为首页。

我们打算把代码放在Album模型，它包含我们的controllers，modules，和configuration。我们可以根据需要调整这个模型。

我们从需要的目录开始。

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

就像上面一样，我们的文件放在 ``Album`` 模型的几个目录里。包含 ``Album`` 命名空间的PHP类文件放在 ``src/Album`` 目录，以便需要的时候在module中使用多个命名空间。视图目录也有一个 ``Album`` 子目录，用来存放模型的视图脚本。

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

``ModuleManager`` 会为我们自动调用 ``getAutoloaderConfig()`` 和 ``getConfig()`` 。

自动加载文件
^^^^^^^^^^^^^^^^^

``getAutoloaderConfig()`` 方法返回一个ZF2的 ``AutoloaderFactory`` 数组。配置它以使我们添加一个 ``ClassMapAutoloader`` 类映射文件，另外，把这个模型的命名空间也添加到``StandardAutoloader``。标准的自动加载需要一个命名空间和找到命名空间文件的路径。这符合PSR-0命名标准 <https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md>`_，通过该标准，类直接映射到文件。

由于在开发中，我们不需要通过映射类加载文件，所以我们给自动加载映射类提供了一个空数组。在 ``zf2-tutorial/module/Album`` 中创建一个  ``autoload_classmap.php`` 文件：

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

The config information is passed to the relevant components by the
``ServiceManager``.  We need two initial sections: ``controllers`` and
``view_manager``. The controllers section provides a list of all the controllers
provided by the module. We will need one controller, ``AlbumController``, which
we’ll reference as ``Album\Controller\Album``. The controller key must
be unique across all modules, so we prefix it with our module name.

Within the ``view_manager`` section, we add our view directory to the
``TemplatePathStack`` configuration. This will allow it to find the view scripts for
the ``Album`` module that are stored in our ``view/`` directory.

Informing the application about our new module
----------------------------------------------

We now need to tell the ``ModuleManager`` that this new module exists. This is done
in the application’s ``config/application.config.php`` file which is provided by the
skeleton application. Update this file so that its ``modules`` section contains the
``Album`` module as well, so the file now looks like this:

(Changes required are highlighted using comments.)

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

As you can see, we have added our ``Album`` module into the list of modules
after the ``Application`` module.

We have now set up the module ready for putting our custom code into it.
