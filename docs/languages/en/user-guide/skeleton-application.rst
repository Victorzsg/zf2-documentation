.. _user-guide.skeleton-application:

开始：骨架应用
=======================================

为创建我们的应用，我们以`ZendSkeletonApplication <https://github.com/zendframework/ZendSkeletonApplication>`_ （可以通过 `github <https://github.com/>`_ 获得）开始。
使用composer(http://getcomposer.org)来创建一个新zend framework应用。

.. code-block:: bash
   :linenos:

    php composer.phar create-project --stability="dev" repository-url="https://packages.zendframework.com" zendframework/skeleton-application path/to/install
    php composer.phar update

.. 注意::

    另一个安装ZendSkeletonApplication的方法是使用github。去https://github.com/zendframework/ZendSkeletonApplication 单击“ZIP”按钮就会下载一个``ZendSkeletonApplication-master.zip``相似的文件。

    把此文件解压到虚拟主机目录并重命名为``zf2-tutorial``。

    ZendSkeletonApplication使用Composer (http://getcomposer.org)来解决它的依赖性，在这里，依赖指的是zf2自己。

    To install Zend Framework 2 into our application we simply type:
    安装zf2到我们的应用，我们只用输入：

    .. code-block:: bash
       :linenos:

        php composer.phar self-update
        php composer.phar install
        php composer.phar update

    从``zf2-tutorial`` 目录。等一会，我们会看到类似下面的输出：

    .. code-block:: bash
       :linenos:

        Installing dependencies from lock file
        - Installing zendframework/zendframework (dev-master)
          Cloning 18c8e223f070deb07c17543ed938b54542aa0ed8

        Generating autoload files

.. 注意::

    如果看到这样的信息： 

    .. code-block:: bash
       :linenos:

        [RuntimeException]      
          The process timed out. 

    说明你的网速太慢而不能及时下载完整的包，导致composer超时。为避免这个错误，用下面的命令代替：

    .. code-block:: bash
       :linenos:

        php composer.phar install
        php composer.phar update

    运行：

    .. code-block:: bash
       :linenos:

        COMPOSER_PROCESS_TIMEOUT=5000 php composer.phar install
        COMPOSER_PROCESS_TIMEOUT=5000 php composer.phar update
        
.. 注意::

   使用windows系统wamp的用户：
   
   1. 安装windows的composer
      通过以下命令检查是否正确安装 
      
      .. code-block:: bash
         :linenos:
         
         composer
         
   2. 安装windows版本git，也需要把git路径添加到windows环境变量
      通过以下命令检查git是否正确安装
      
      .. code-block:: bash
         :linenos:
         
         git
         
   3. 现在用命令行安装zf2
      
      .. code-block:: bash
         :linenos:
         
         composer create-project --repository-url="https://packages.zendframework.com" -s dev zendframework/skeleton-application path/to/install
   

现在我们来设置web服务器设置。

使用Apache Web Server
---------------------------

你的创建一个Apache虚拟主机，以便可以通过 ``http://zf2-tutorial.localhost`` 访问``zf2-tutorial/public``目录的index.php页面。

Setting up the virtual host is usually done within ``httpd.conf`` or
``extra/httpd-vhosts.conf``.  If you are using ``httpd-vhosts.conf``, ensure
that this file is included by your main ``httpd.conf`` file.  Some Linux distributions 
(ex: Ubuntu) package Apache so that configuration files are stored in ``/etc/apache2`` 
and create one file per virtual host inside folder ``/etc/apache2/sites-enabled``.  In 
this case, you would place the virtual host block below into the file 
``/etc/apache2/sites-enabled/zf2-tutorial``.

Ensure that ``NameVirtualHost`` is defined and set to “\*:80” or similar, and then
define a virtual host along these lines:

.. code-block:: apache
   :linenos:

    <VirtualHost *:80>
        ServerName zf2-tutorial.localhost
        DocumentRoot /path/to/zf2-tutorial/public
        SetEnv APPLICATION_ENV "development"
        <Directory /path/to/zf2-tutorial/public>
            DirectoryIndex index.php
            AllowOverride All
            Order allow,deny
            Allow from all
        </Directory>
    </VirtualHost>

Make sure that you update your ``/etc/hosts`` or
``c:\windows\system32\drivers\etc\hosts`` file so that ``zf2-tutorial.localhost``
is mapped to ``127.0.0.1``. The website can then be accessed using
``http://zf2-tutorial.localhost``.

.. code-block:: none
   :linenos:

    127.0.0.1               zf2-tutorial.localhost localhost

Restart Apache.

If you've done it correctly, it should look something like this:

.. image:: ../images/user-guide.skeleton-application.hello-world.png
    :width: 940 px

To test that your ``.htaccess`` file is working, navigate to
``http://zf2-tutorial.localhost/1234`` and you should see this:

.. image:: ../images/user-guide.skeleton-application.404.png
    :width: 940 px

If you see a standard Apache 404 error, then you need to fix ``.htaccess`` usage
before continuing.  If you're are using IIS with the URL Rewrite Module, import the following:

.. code-block:: apache
   :linenos:

    RewriteCond %{REQUEST_FILENAME} !-f
    RewriteRule ^ index.php [NC,L]

You now have a working skeleton application and we can start adding the specifics
for our application.

Using the Built-in PHP CLI Server
---------------------------------

Alternatively — if you are using PHP 5.4 or above — you can use the built-in CLI server (cli-server). To do this, you
just start the server in the root directory:

.. code-block:: bash
    :linenos:
    
    php -S 0.0.0.0:8080 -t public/ public/index.php

This will make the website available on port 8080 on all network interfaces, using
``public/index.php`` to handle routing. This means the site is accessible via http://localhost:8080
or http://<your-local-IP>:8080.

If you’ve done it right, you should see the same result as with Apache above.

To test that your routing is working, navigate to
http://localhost:8080/1234 and you should see the same error page as with Apache above.

.. note::

    The built-in CLI server is **for development only**.

Error reporting
---------------

Optionally, *when using Apache*, you can use the ``APPLICATION_ENV`` setting in 
your ``VirtualHost`` to let PHP output all its errors to the browser. This can be 
useful during the development of your application.

Edit ``index.php`` from the ``zf2-tutorial/public/`` directory and change it to
the following:

.. code-block:: php
   :linenos:

    <?php

    /**
     * Display all errors when APPLICATION_ENV is development.
     */
    if ($_SERVER['APPLICATION_ENV'] == 'development') {
        error_reporting(E_ALL);
        ini_set("display_errors", 1);
    }
    
    /**
     * This makes our life easier when dealing with paths. Everything is relative
     * to the application root now.
     */
    chdir(dirname(__DIR__));
    
    // Decline static file requests back to the PHP built-in webserver
    if (php_sapi_name() === 'cli-server' && is_file(__DIR__ . parse_url($_SERVER['REQUEST_URI'], PHP_URL_PATH))) {
        return false;
    }

    // Setup autoloading
    require 'init_autoloader.php';
    
    // Run the application!
    Zend\Mvc\Application::init(require 'config/application.config.php')->run();
