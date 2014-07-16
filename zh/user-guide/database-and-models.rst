.. _user-guide.database-and-models:

数据库和模型
===================

数据库
------------

现在我们有了 ``Album`` 模块，并且建立了控制器方法和视图脚本，是时候看看我们应用程序的模型部分了。模型是处理完成应用核心目标的部分（所谓的“商业逻辑”），对我们来说，就是操作数据库。我们利用zf2的 ``Zend\Db\TableGateway\TableGateway`` 类在数据库表中查找、添加、更新和删除数据。 

我们将通过PHP的PDO驱动来使用Mysql。创建一个数据库 ``zf2tutorial`` ，运行下面的SQL语句来创建album表，并给它添加一些数据。

.. code-block:: sql
   :linenos:

    CREATE TABLE album (
      id int(11) NOT NULL auto_increment,
      artist varchar(100) NOT NULL,
      title varchar(100) NOT NULL,
      PRIMARY KEY (id)
    );
    INSERT INTO album (artist, title)
        VALUES  ('The  Military  Wives',  'In  My  Dreams');
    INSERT INTO album (artist, title)
        VALUES  ('Adele',  '21');
    INSERT INTO album (artist, title)
        VALUES  ('Bruce  Springsteen',  'Wrecking Ball (Deluxe)');
    INSERT INTO album (artist, title)
        VALUES  ('Lana  Del  Rey',  'Born  To  Die');
    INSERT INTO album (artist, title)
        VALUES  ('Gotye',  'Making  Mirrors');

（选择的这些测试数据恰好是在英国亚马逊的畅销书写作的时间！）

现在数据库中有一些数据，我们可以给它写一个简单的模型。

模型文件
---------------

Zend Framework没有提供一个 ``Zend\Model`` 组件，因为模型是你的商业逻辑，这些该由你来决定他们是怎么运行的。有许多组件可以使用，它们取决于你的需要。一种方法是用类表示应用程序中的每个实体模型，然后使用映射程序加载和保存实体对象到数据库。另一种是使用对象映射技术（ORM），例如Doctrine 或 Propel。

本教程中，我们打算创建一个简单的模型类，创建一个 ``AlbumTable`` 类来使用 ``Zend\Db\TableGateway\TableGateway`` 类，在 ``Zend\Db\TableGateway\TableGateway`` 类中，每一个唱片对象是一个 ``Album`` 对象 （成为*实体*）。这是表数据网关设计模式的实现，与数据表中数据实现对接。注意表数据网关模式会成为大型系统中的限制。很容易犯的一个错误是，将数据库连接代码放在控制器方法中，因为这些都被 ``Zend\Db\TableGateway\AbstractTableGateway`` 公开了。 *不要这样做！* 

让我们开始在 ``module/Album/src/Album/Model`` 创建一个叫做 ``Album.php`` 的文件：

.. code-block:: php
   :linenos:

    namespace Album\Model;

    class Album
    {
        public $id;
        public $artist;
        public $title;

        public function exchangeArray($data)
        {
            $this->id     = (!empty($data['id'])) ? $data['id'] : null;
            $this->artist = (!empty($data['artist'])) ? $data['artist'] : null;
            $this->title  = (!empty($data['title'])) ? $data['title'] : null;
        }
    }

我们的 ``Album`` 实体对象只是一个简单的PHP类。为了和 ``Zend\Db`` 的 ``TableGateway`` 类一起运行，我们需要实例化 ``exchangeArray()`` 方法。这个方法仅仅把传递过来的数据复制到对象的属性。使用表单的时候，我们稍后会添加一个输入过滤机制。

下一步，我们在 ``module/Album/src/Album/Model`` 目录创建 ``AlbumTable.php`` 文件：

.. code-block:: php
   :linenos:

    namespace Album\Model;

    use Zend\Db\TableGateway\TableGateway;

    class AlbumTable
    {
        protected $tableGateway;

        public function __construct(TableGateway $tableGateway)
        {
            $this->tableGateway = $tableGateway;
        }

        public function fetchAll()
        {
            $resultSet = $this->tableGateway->select();
            return $resultSet;
        }

        public function getAlbum($id)
        {
            $id  = (int) $id;
            $rowset = $this->tableGateway->select(array('id' => $id));
            $row = $rowset->current();
            if (!$row) {
                throw new \Exception("Could not find row $id");
            }
            return $row;
        }

        public function saveAlbum(Album $album)
        {
            $data = array(
                'artist' => $album->artist,
                'title'  => $album->title,
            );

            $id = (int) $album->id;
            if ($id == 0) {
                $this->tableGateway->insert($data);
            } else {
                if ($this->getAlbum($id)) {
                    $this->tableGateway->update($data, array('id' => $id));
                } else {
                    throw new \Exception('Album id does not exist');
                }
            }
        }

        public function deleteAlbum($id)
        {
            $this->tableGateway->delete(array('id' => (int) $id));
        }
    }

这里稍微麻烦一些。首先，我们给 ``TableGateway`` 类的构造函数设置protected属性 ``$tableGateway`` 。我们使用它对我们的模型数据库表进行操作。

然后，我们创建一些应用将会使用到的辅助函数。 ``fetchAll()`` 从数据库中取出所有的唱片信息放在一个 ``结果集`` 中， ``getAlbum()`` 取出一条 ``唱片`` 对象数据， ``saveAlbum()`` 或者在数据库中添加一条信息，或者修改一条已经存在的信息， ``deleteAlbum()`` 把某条数据完全删除。每一个方法的代码是有效且容易理解的。

使用服务管理来配置表入口并注入到album表
----------------------------------------------------------------------------------

为了使用相同的 ``AlbumTable`` 实例，我们会用 ``服务管理`` 来说明如何创建一个。最简单的做法是创建了一个叫做 ``getServiceConfig()`` 方法的模型类，它会被 ``ModuleManager`` 自动调用并被 ``ServiceManager`` 使用。这样，需要的时候我们就能在控制器中检索到它。

配置 ``ServiceManager``，我们或者提供即将被实例化的类的名字，或者当 ``ServiceManager`` 需要的时候，提供一个工厂（关闭或回调）来实例化对象。我们使用 ``getServiceConfig()`` 来提供一个创建 ``唱片表`` 的工厂。把此方法添加到 ``module/Album`` 下面的文件 ``Module.php`` 里。

.. code-block:: php
   :linenos:
   :emphasize-lines: 5-8,14-32

    namespace Album;

    // Add these import statements:
    use Album\Model\Album;
    use Album\Model\AlbumTable;
    use Zend\Db\ResultSet\ResultSet;
    use Zend\Db\TableGateway\TableGateway;

    class Module
    {
        // getAutoloaderConfig() and getConfig() methods here

        // Add this method:
        public function getServiceConfig()
        {
            return array(
                'factories' => array(
                    'Album\Model\AlbumTable' =>  function($sm) {
                        $tableGateway = $sm->get('AlbumTableGateway');
                        $table = new AlbumTable($tableGateway);
                        return $table;
                    },
                    'AlbumTableGateway' => function ($sm) {
                        $dbAdapter = $sm->get('Zend\Db\Adapter\Adapter');
                        $resultSetPrototype = new ResultSet();
                        $resultSetPrototype->setArrayObjectPrototype(new Album());
                        return new TableGateway('album', $dbAdapter, null, $resultSetPrototype);
                    },
                ),
            );
        }
    }

这个方法返回一个 ``工厂模式`` 数组，它们在被传递到 ``ServiceManager`` 之前被 ``ModuleManager`` 合并到了一起。``Album\Model\AlbumTable`` 工厂使用 ``ServiceManager`` 创建一个到``唱片表`` 的 ``唱片表入口``。通过获得一个 ``Zend\Db\Adapter\Adapter`` （来自 ``ServiceManager``)，并用它创建一个 ``表入口`` 对象，我们通知 ``ServiceManager`` 一个 ``唱片表入口`` 已经被创建。不论何时创建一行新的数据， ``TableGateway`` 被告知去调用 ``Album`` 对象。表入口类使用原型模式来创建结果集和实体。这就意味着，代替需要的时候才加载，系统会去复制一个事先实例化的对象。更多信息，请查看
`PHP Constructor Best Practices and the Prototype Pattern <http://ralphschindler.com/2012/03/09/php-constructor-best-practices-and-the-prototype-pattern>`_。

最后，我们需要配置 ``ServiceManager`` ，使其知道如何获得一个 ``Zend\Db\Adapter\Adapter``。这是通过使用一个称为 ``Zend\Db\Adapter\AdapterServiceFactory`` 的工厂做到的，该工厂可以在合并的配置系统中配置。Zend Framework 2的 ``ModuleManager`` 把每个模型 ``module.config.php`` 文件的所有配置信息合并到 ``config/autoload`` （``*.global.php`` 和 ``*.local.php``
文件中）。我们把数据库配置信息添加到 ``global.php`` ，你应该会把此文件提交到版本控制系统中。如果愿意，你可以用 ``local.php`` （不在VCS里面）储存数据凭证。


修改 ``config/autoload/global.php`` （在Zend Skeleton 根目录，不在 
Album 模型） 文件：

.. code-block:: php
   :linenos:

    return array(
        'db' => array(
            'driver'         => 'Pdo',
            'dsn'            => 'mysql:dbname=zf2tutorial;host=localhost',
            'driver_options' => array(
                PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES \'UTF8\''
            ),
        ),
        'service_manager' => array(
            'factories' => array(
                'Zend\Db\Adapter\Adapter'
                        => 'Zend\Db\Adapter\AdapterServiceFactory',
            ),
        ),
    );

你应该把数据库凭证放在 ``config/autoload/local.php``，以使它们不在git仓库（因为 ``local.php`` 会被git忽略）：

.. code-block:: php
   :linenos:

    return array(
        'db' => array(
            'username' => 'YOUR USERNAME HERE',
            'password' => 'YOUR PASSWORD HERE',
        ),
    );

回到控制器
----------------------

现在 ``ServiceManager`` 可以给我们创建一个 ``AlbumTable`` 实例，我们可以在控制器添加一个方法来使用它。为 ``AlbumController`` 类添加 ``getAlbumTable()`` 方法：

.. code-block:: php
   :linenos:

    // module/Album/src/Album/Controller/AlbumController.php:
        public function getAlbumTable()
        {
            if (!$this->albumTable) {
                $sm = $this->getServiceLocator();
                $this->albumTable = $sm->get('Album\Model\AlbumTable');
            }
            return $this->albumTable;
        }

还要添加：

.. code-block:: php
   :linenos:

    protected $albumTable;

到类的头部。

现在当我们需要操作模型的时候，就可以调用控制器中的 ``getAlbumTable()`` 方法。

如果服务探测器在 ``Module.php`` 中被正确的配置，当调用 getAlbumTable()`` 方法的时候，就会获得一个 ``Album\Model\AlbumTable`` 实例。

列出唱片
--------------

为了列出唱片，我们需要把它们从model中检索出来并传递到视图中去。为此，我们在 ``AlbumController`` 控制器中编写 ``indexAction()`` 方法。修改 ``AlbumController`` 中的 ``indexAction()`` 方法成这样：

.. code-block:: php
   :linenos:

    // module/Album/src/Album/Controller/AlbumController.php:
    // ...
        public function indexAction()
        {
            return new ViewModel(array(
                'albums' => $this->getAlbumTable()->fetchAll(),
            ));
        }
    // ...

在zf2中，为了在view中设置变量，我们返回一个 ``ViewModel``。这些会自动被传递到视图脚本中。``ViewModel`` 对象也允许我们改变正在使用的视图脚本，但是默认使用的是 ``{controller name}/{action
name}``。现在编写 ``index.phtml`` 视图脚本：

.. code-block:: php
   :linenos:

    <?php
    // module/Album/view/album/album/index.phtml:

    $title = 'My albums';
    $this->headTitle($title);
    ?>
    <h1><?php echo $this->escapeHtml($title); ?></h1>
    <p>
        <a href="<?php echo $this->url('album', array('action'=>'add'));?>">Add new album</a>
    </p>

    <table class="table">
    <tr>
        <th>Title</th>
        <th>Artist</th>
        <th>&nbsp;</th>
    </tr>
    <?php foreach ($albums as $album) : ?>
    <tr>
        <td><?php echo $this->escapeHtml($album->title);?></td>
        <td><?php echo $this->escapeHtml($album->artist);?></td>
        <td>
            <a href="<?php echo $this->url('album',
                array('action'=>'edit', 'id' => $album->id));?>">Edit</a>
            <a href="<?php echo $this->url('album',
                array('action'=>'delete', 'id' => $album->id));?>">Delete</a>
        </td>
    </tr>
    <?php endforeach; ?>
    </table>

我们先来给页面设置title（应用在布局中），使用 ``headTitle()`` 视图辅助函数设置 ``<head>`` 部分的title，它将展示在浏览器的标题栏。然后我们创建一个添加新唱片的链接。

zf2提供了  ``url()`` 视图辅助函数来创建我们需要的链接。 ``url()`` 的第一个参数是我们希望使用的路径名，第二个参数是变量占位符的数组。这里，使用我们的‘album’路径，并且设置两个占位符变量 ``action`` 和 ``id``。

我们循环从控制器方法中分配的 ``$albums`` 变量。zf2视图系统会自动确保这些变量被提取到视图的作用域，这样我们不用担心像使用zf1一样在它们前面放置 ``$this->``；但是，你喜欢的话也可以这么做。



We then create a table to display each album’s title and artist, and provide
links to allow for editing and deleting the record. A standard ``foreach:`` loop
is used to iterate over the list of albums, and we use the alternate form using
a colon and ``endforeach;`` as it is easier to scan than to try and match up
braces. Again, the ``url()`` view helper is used to create the edit and delete
links.

.. note::

    We always use the ``escapeHtml()`` view helper to help protect
    ourselves from Cross Site Scripting (XSS) vulnerabilities (see http://en.wikipedia.org/wiki/Cross-site_scripting).

If you open ``http://zf2-tutorial.localhost/album`` you should see this:

.. image:: ../images/user-guide.database-and-models.album-list.png
    :width: 940 px


