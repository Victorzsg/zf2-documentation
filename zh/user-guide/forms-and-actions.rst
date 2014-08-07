.. _user-guide-forms-and-actions:

表单和方法
=================

添加新唱片
-----------------

现在我们来编写添加唱片功能。有两点：

* 展示一个用户提交信息的表单
* 处理提交的表单数据并保存到数据库

用 ``Zend\Form`` 来做这个。``Zend\Form`` 组件管理表单和表单验证，添加 ``Zend\InputFilter`` 到我们的 ``唱片`` 对象。创建一个扩展自 ``Zend\Form\Form`` 的 ``Album\Form\AlbumForm`` 新类来定义我们的表单。在 ``module/Album/src/Album/Form`` 中创建一个 ``AlbumForm.php`` 文件：

.. code-block:: php
   :linenos:

    namespace Album\Form;

    use Zend\Form\Form;

    class AlbumForm extends Form
    {
        public function __construct($name = null)
        {
            // we want to ignore the name passed
            parent::__construct('album');

            $this->add(array(
                'name' => 'id',
                'type' => 'Hidden',
            ));
            $this->add(array(
                'name' => 'title',
                'type' => 'Text',
                'options' => array(
                    'label' => 'Title',
                ),
            ));
            $this->add(array(
                'name' => 'artist',
                'type' => 'Text',
                'options' => array(
                    'label' => 'Artist',
                ),
            ));
            $this->add(array(
                'name' => 'submit',
                'type' => 'Submit',
                'attributes' => array(
                    'value' => 'Go',
                    'id' => 'submitbutton',
                ),
            ));
        }
    }

在 ``AlbumForm`` 构造函数中，做了以下事情。首先，给这个表单设置名称，调用父类的构造函数。创建4个表单元素：id，title，artist和提交按钮。对每一项，我们都设置多个属性和选项，包括要被展示的标签。

表单验证。在zf2中，这是通过输入过滤实现的，它既可以单独使用，也可以在继承自 ``InputFilterAwareInterface`` 接口的任何类中定义，比如一个模型实例。在这里，我们打算给唱片类添加一个输入过滤器，它放在 ``module/Album/src/Album/Model`` 中的 ``Album.php`` 文件里：

.. code-block:: php
   :linenos:
   :emphasize-lines: 5-7,9,14,23-84

    namespace Album\Model;

    // Add these import statements
    use Zend\InputFilter\InputFilter;
    use Zend\InputFilter\InputFilterAwareInterface;
    use Zend\InputFilter\InputFilterInterface;

    class Album implements InputFilterAwareInterface
    {
        public $id;
        public $artist;
        public $title;
        protected $inputFilter;                       // <-- Add this variable

        public function exchangeArray($data)
        {
            $this->id     = (isset($data['id']))     ? $data['id']     : null;
            $this->artist = (isset($data['artist'])) ? $data['artist'] : null;
            $this->title  = (isset($data['title']))  ? $data['title']  : null;
        }

        // Add content to these methods:
        public function setInputFilter(InputFilterInterface $inputFilter)
        {
            throw new \Exception("Not used");
        }

        public function getInputFilter()
        {
            if (!$this->inputFilter) {
                $inputFilter = new InputFilter();

                $inputFilter->add(array(
                    'name'     => 'id',
                    'required' => true,
                    'filters'  => array(
                        array('name' => 'Int'),
                    ),
                ));

                $inputFilter->add(array(
                    'name'     => 'artist',
                    'required' => true,
                    'filters'  => array(
                        array('name' => 'StripTags'),
                        array('name' => 'StringTrim'),
                    ),
                    'validators' => array(
                        array(
                            'name'    => 'StringLength',
                            'options' => array(
                                'encoding' => 'UTF-8',
                                'min'      => 1,
                                'max'      => 100,
                            ),
                        ),
                    ),
                ));

                $inputFilter->add(array(
                    'name'     => 'title',
                    'required' => true,
                    'filters'  => array(
                        array('name' => 'StripTags'),
                        array('name' => 'StringTrim'),
                    ),
                    'validators' => array(
                        array(
                            'name'    => 'StringLength',
                            'options' => array(
                                'encoding' => 'UTF-8',
                                'min'      => 1,
                                'max'      => 100,
                            ),
                        ),
                    ),
                ));

                $this->inputFilter = $inputFilter;
            }

            return $this->inputFilter;
        }
    }

输入过滤感知接口 ``InputFilterAwareInterface`` 定义了两个方法：``setInputFilter()`` 和
``getInputFilter()``。我们只用实例化 ``getInputFilter()``，所以我们仅仅在 ``setInputFilter()`` 中抛出一个异常。

在 ``getInputFilter()`` 中，实例化了 ``InputFilter``，然后添加上需要的输入项。我们为每一个需要过滤和验证的属性添加一个表单。在 ``id`` 列，添加一个 ``Int`` 过滤器，因为只需要整型数字。文本域中，添加了两个过滤器，``StripTags`` 和
``StringTrim`` 用来去除不需要的HTML代码和空格。我们还把它们设置为 *required* 并添加了一个 ``StringLength`` 来确保输入的字符长度不会超出数据库允许存储的长度。

现在先显示表格，提交的时候再处理它们。下面是 ``AlbumController`` 控制器的 ``addAction()`` 方法：

.. code-block:: php
   :linenos:
   :emphasize-lines: 6-7,10-31

    // module/Album/src/Album/Controller/AlbumController.php:

    //...
    use Zend\Mvc\Controller\AbstractActionController;
    use Zend\View\Model\ViewModel;
    use Album\Model\Album;          // <-- Add this import
    use Album\Form\AlbumForm;       // <-- Add this import
    //...

        // Add content to this method:
        public function addAction()
        {
            $form = new AlbumForm();
            $form->get('submit')->setValue('Add');

            $request = $this->getRequest();
            if ($request->isPost()) {
                $album = new Album();
                $form->setInputFilter($album->getInputFilter());
                $form->setData($request->getPost());

                if ($form->isValid()) {
                    $album->exchangeArray($form->getData());
                    $this->getAlbumTable()->saveAlbum($album);

                    // Redirect to list of albums
                    return $this->redirect()->toRoute('album');
                }
            }
            return array('form' => $form);
        }
    //...

把 ``AlbumForm`` 添加use列表后，完善一下 ``addAction()`` 方法：

.. code-block:: php
   :linenos:

    $form = new AlbumForm();
    $form->get('submit')->setValue('Add');

实例化 ``AlbumForm`` ，给“Add”提交按钮添加label。这样做是因为我们想编辑唱片的时候，使用不同的label重用表单。

.. code-block:: php
   :linenos:

    $request = $this->getRequest();
    if ($request->isPost()) {
        $album = new Album();
        $form->setInputFilter($album->getInputFilter());
        $form->setData($request->getPost());
        if ($form->isValid()) {

如果 ``Request`` 对象的 ``isPost()`` 返回TRUE，表单将被提交，在唱片实例中给表单设置的过滤器也将被提交。然后，设置提交到表单的数据，使用表单的成员函数 ``isValid()`` 来检查数据是否合法。

.. code-block:: php
   :linenos:

    $album->exchangeArray($form->getData());
    $this->getAlbumTable()->saveAlbum($album);

如果该表单是有效的，就用 ``saveAlbum()`` 把提取出来的数据存入模型中。

.. code-block:: php
   :linenos:

    // Redirect to list of albums
    return $this->redirect()->toRoute('album');

存入新唱片信息后，我们使用控制器插件 ``Redirect`` 重定向到唱片列表页。

.. code-block:: php
   :linenos:

    return array('form' => $form);

最后，返回想要传递给视图的变量，在这里，只是表单对象。注意zf2允许你返回一个即将传递给视图的变量的数组，并且在暗中为你创建一个 ``ViewModel``。这样就省去了一些代码的编写。

现在我们在add.phtml视图中渲染这个表单：

.. code-block:: php
   :linenos:

    <?php
    // module/Album/view/album/album/add.phtml:

    $title = 'Add new album';
    $this->headTitle($title);
    ?>
    <h1><?php echo $this->escapeHtml($title); ?></h1>
    <?php
    $form->setAttribute('action', $this->url('album', array('action' => 'add')));
    $form->prepare();

    echo $this->form()->openTag($form);
    echo $this->formHidden($form->get('id'));
    echo $this->formRow($form->get('title'));
    echo $this->formRow($form->get('artist'));
    echo $this->formSubmit($form->get('submit'));
    echo $this->form()->closeTag();

同样，我们像之前一样显示一个标题，然后渲染表单。zf提供了一些视图辅助函数，使这个操作更加简单。 ``form()`` 视图辅助函数有 ``openTag()`` 和 ``closeTag()`` 两个方法来打开和关闭表单。对于每个有label的元素，使用 ``formRow()``，但是对于两个独立的元素，使用 ``formHidden()`` 和
``formSubmit()``。

.. image:: ../images/user-guide.forms-and-actions.add-album-form.png
    :width: 940 px

另外，渲染表单的过程仅仅用绑定的 ``formCollection`` 视图辅助函数就行了。例如，在上面的视图中，把所有表单-渲染输出语句都替换成：

.. code-block:: php
   :linenos:

    echo $this->formCollection($form);

Note: 仍需调用表单的 ``openTag`` 和 ``closeTag`` 方法。上面的代码中，把其他的输出语句用调用 ``formCollection`` 方法代替。

这会遍历表单结构，给每一个元素调用合适的标签，元素和错误的视图辅助函数，但是你还是得把formCollection($form)元素放在打开和关闭的表单标签里面。这些辅助函数减少了视图脚本的复杂性，默认渲染的HTML表单是可以接受的。

现在，你应该可以在应用首页使用“Add new album”来添加一条唱片记录。

编辑唱片
----------------

编辑唱片差不多和添加一样，所以代码非常相似。这次，我们使用 ``AlbumController`` 中的 ``editAction()`` 方法：

.. code-block:: php
   :linenos:

    // module/Album/src/Album/Controller/AlbumController.php:
    //...

        // Add content to this method:
        public function editAction()
        {
            $id = (int) $this->params()->fromRoute('id', 0);
            if (!$id) {
                return $this->redirect()->toRoute('album', array(
                    'action' => 'add'
                ));
            }

            // Get the Album with the specified id.  An exception is thrown
            // if it cannot be found, in which case go to the index page.
            try {
                $album = $this->getAlbumTable()->getAlbum($id);
            }
            catch (\Exception $ex) {
                return $this->redirect()->toRoute('album', array(
                    'action' => 'index'
                ));
            }

            $form  = new AlbumForm();
            $form->bind($album);
            $form->get('submit')->setAttribute('value', 'Edit');

            $request = $this->getRequest();
            if ($request->isPost()) {
                $form->setInputFilter($album->getInputFilter());
                $form->setData($request->getPost());

                if ($form->isValid()) {
                    $this->getAlbumTable()->saveAlbum($album);

                    // Redirect to list of albums
                    return $this->redirect()->toRoute('album');
                }
            }

            return array(
                'id' => $id,
                'form' => $form,
            );
        }
    //...

这些代码应该驾轻就熟。让我们看看和添加唱片有什么不同。首先，我们找到匹配路由中的 ``id``，使用它来加载要编辑的唱片：

.. code-block:: php
   :linenos:

    $id = (int) $this->params()->fromRoute('id', 0);
    if (!$id) {
        return $this->redirect()->toRoute('album', array(
            'action' => 'add'
        ));
    }

    // Get the album with the specified id.  An exception is thrown 
    // if it cannot be found, in which case go to the index page.
    try {
        $album = $this->getAlbumTable()->getAlbum($id);
    }
    catch (\Exception $ex) {
        return $this->redirect()->toRoute('album', array(
            'action' => 'index'
        ));
    }

"params"是一个控制器插件，提供了一种方便的方法从匹配路由中来检索参数。我们用它来检索 ``module.config.php`` 创建的路由中的 ``id``。如果 ``id`` 是0，重定向到添加方法，否则，我们继续从数据库获取这张专辑信息。

我们必须检查并确保指定 ``id`` 的唱片真的能够找到。如果不能，数据访问方法就会抛出异常。我们获取这个异常并把用户重定向到首页。

.. code-block:: php
   :linenos:

    $form = new AlbumForm();
    $form->bind($album);
    $form->get('submit')->setAttribute('value', 'Edit');

表单的 ``bind()`` 方法把模型附加上去。有两种方法：

* 显示表单时，每个元素的初始值被提取出来。
* 用isValid()验证成功之后，表单的数据被放回模型。



这些方法通过hydrator对象完成。有很多hydrators，默认的是 ``Zend\Stdlib\Hydrator\ArraySerializable``，它会找到模型中的两个方法 ``getArrayCopy()`` 和
``exchangeArray()``。在 ``Album`` 实体中，我们已经编写了 ``exchangeArray()``，所以只用编写 ``getArrayCopy()``：

.. code-block:: php
   :linenos:
   :emphasize-lines: 10-14

    // module/Album/src/Album/Model/Album.php:
    // ...
        public function exchangeArray($data)
        {
            $this->id     = (isset($data['id']))     ? $data['id']     : null;
            $this->artist = (isset($data['artist'])) ? $data['artist'] : null;
            $this->title  = (isset($data['title']))  ? $data['title']  : null;
        }

        // Add the following method:
        public function getArrayCopy()
        {
            return get_object_vars($this);
        }
    // ...

由于使用"bind()"以其hydrator，我们不必把表单的数据放进 ``$album``，因为那已经被做了，我们只用调用映射器的 ``saveAlbum()`` 方法把改动存入数据库。

编辑视图模板看起来和添加唱片的非常像：

.. code-block:: php
   :linenos:

    <?php
    // module/Album/view/album/album/edit.phtml:

    $title = 'Edit album';
    $this->headTitle($title);
    ?>
    <h1><?php echo $this->escapeHtml($title); ?></h1>

    <?php
    $form = $this->form;
    $form->setAttribute('action', $this->url(
        'album',
        array(
            'action' => 'edit',
            'id'     => $this->id,
        )
    ));
    $form->prepare();

    echo $this->form()->openTag($form);
    echo $this->formHidden($form->get('id'));
    echo $this->formRow($form->get('title'));
    echo $this->formRow($form->get('artist'));
    echo $this->formSubmit($form->get('submit'));
    echo $this->form()->closeTag();

仅有的变化时使用 ‘Edit Album’ 标题并设置为提交到 ‘edit’ 方法。

现在你应该可以编辑唱片了。

删除唱片
-----------------

为了完善应用，我们添加删除操作。在列表页，每个唱片有一个删除链接，点击删除的时候，唱片会被删除。这是错的。注意HTTP规范，我们回忆一下，不要用GET进行一个不可逆转的操作，你应该用POST代替。

当用户点击删除的时候，应该有个确认表单，如果他们选择的是“yes”，我们才执行删除操作。形式不重要，我们直接来辩解视图（毕竟，``Zend\Form`` 是可选的）。

开始编辑 ``AlbumController::deleteAction()``：

.. code-block:: php
   :linenos:

    // module/Album/src/Album/Controller/AlbumController.php:
    //...
        // Add content to the following method:
        public function deleteAction()
        {
            $id = (int) $this->params()->fromRoute('id', 0);
            if (!$id) {
                return $this->redirect()->toRoute('album');
            }

            $request = $this->getRequest();
            if ($request->isPost()) {
                $del = $request->getPost('del', 'No');

                if ($del == 'Yes') {
                    $id = (int) $request->getPost('id');
                    $this->getAlbumTable()->deleteAlbum($id);
                }

                // Redirect to list of albums
                return $this->redirect()->toRoute('album');
            }

            return array(
                'id'    => $id,
                'album' => $this->getAlbumTable()->getAlbum($id)
            );
        }
    //...

和之前一样，从匹配的路由中获取 ``id``，检查请求对象的 ``isPost()`` 来决定是否展示确认页面或者删除唱片。我们使用表格对象的 ``deleteAlbum()`` 方法来删除唱片，然后返回到唱片列表页。如果请求不是POST，我们根据 ``id`` 去数据库获取正确的数据，传递给视图。

视图脚本是一个简单的表单：

.. code-block:: php
   :linenos:

    <?php
    // module/Album/view/album/album/delete.phtml:

    $title = 'Delete album';
    $this->headTitle($title);
    ?>
    <h1><?php echo $this->escapeHtml($title); ?></h1>

    <p>Are you sure that you want to delete
        '<?php echo $this->escapeHtml($album->title); ?>' by
        '<?php echo $this->escapeHtml($album->artist); ?>'?
    </p>
    <?php
    $url = $this->url('album', array(
        'action' => 'delete',
        'id'     => $this->id,
    ));
    ?>
    <form action="<?php echo $url; ?>" method="post">
    <div>
        <input type="hidden" name="id" value="<?php echo (int) $album->id; ?>" />
        <input type="submit" name="del" value="Yes" />
        <input type="submit" name="del" value="No" />
    </div>
    </form>

在此脚本中，我们向用户展示了一个带有“yes”和“no”按钮的确认提示。操作中，删除的时候，我们特别检查了“yes”值。

确保主页显示唱片列表
-------------------------------------------------------

最后一点，此刻，主页 ``http://zf2-tutorial.localhost/`` 没有显示唱片列表。

这是由于 ``Application`` 模型中的 ``module.config.php`` 没有设置路由。改变它，可以打开
``module/Application/config/module.config.php`` ，找到home路由：

.. code-block:: php
   :linenos:

    'home' => array(
        'type' => 'Zend\Mvc\Router\Http\Literal',
        'options' => array(
            'route'    => '/',
            'defaults' => array(
                'controller' => 'Application\Controller\Index',
                'action'     => 'index',
            ),
        ),
    ),

把 ``controller`` 从 ``Application\Controller\Index`` 改为
``Album\Controller\Album``:

.. code-block:: php
   :linenos:
   :emphasize-lines: 6

    'home' => array(
        'type' => 'Zend\Mvc\Router\Http\Literal',
        'options' => array(
            'route'    => '/',
            'defaults' => array(
                'controller' => 'Album\Controller\Album', // <-- change here
                'action'     => 'index',
            ),
        ),
    ),

就是这样 —— 现在你拥有了一个完全可用的应用程序！
