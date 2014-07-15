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

The ``ModuleManager`` will call ``getAutoloaderConfig()`` and ``getConfig()``
automatically for us.

Autoloading files
^^^^^^^^^^^^^^^^^

Our ``getAutoloaderConfig()`` method returns an array that is compatible with
ZF2’s ``AutoloaderFactory``. We configure it so that we add a class map file to
the ``ClassMapAutoloader`` and also add this module’s namespace to the
``StandardAutoloader``. The standard autoloader requires a namespace and the
path where to find the files for that namespace. It is PSR-0 compliant and so
classes map directly to files as per the `PSR-0 rules
<https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md>`_.

As we are in development, we don’t need to load files via the classmap, so we provide an empty array for the
classmap autoloader. Create a file called ``autoload_classmap.php`` under ``zf2-tutorial/module/Album``:

.. code-block:: php
   :linenos:

    return array();

As this is an empty array, whenever the autoloader looks for a class within the
``Album`` namespace, it will fall back to the to ``StandardAutoloader`` for us.

.. note::

    If you are using Composer, you could instead just create an empty
    ``getAutoloaderConfig() { }`` and add to composer.json:

    .. code-block:: javascript
       :linenos:

        "autoload": {
            "psr-0": { "Album": "module/Album/src/" }
        },

    If you go this way, then you need to run ``php composer.phar update`` to update 
    the composer autoloading files.

Configuration
-------------

Having registered the autoloader, let’s have a quick look at the ``getConfig()``
method in ``Album\Module``.  This method simply loads the
``config/module.config.php`` file.

Create a file called ``module.config.php`` under ``zf2-tutorial/module/Album/config``:

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
