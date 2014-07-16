.. _user-guide.conclusion:

总结
==========

这个总结回顾了，使用zf2创建一个简单但是功能全面的MVC应用。

在这个简单的教程中，我们触及了框架的很多部分。

用zf2创建的应用的最重要的部分是 :ref:`modules <zend.module-manager.intro>` ，:ref:`MVC ZF2 应用 <zend.mvc.intro>` 的基石。

我们使用 :ref:`service manager <zend.service-manager.intro>` ，来简化我们创建应用所依赖的工作。

为了能解析对控制器和控制器方法的请求，我们使用 :ref:`routes <zend.mvc.routing>` 。

数据持续性，在多数情况下，包括使用 :ref:`Zend\\Db <zend.db.adapter>` 和某种数据库通讯。输入数据会被 :ref:`input filters <zend.input-filter.intro>` 和 :ref:`Zend\\Form <zend.form.intro>` 过滤并验证，它们是模型和视图之间牢固的桥梁。:ref:`Zend\\View <zend.view.quick-start>` 和大量的 :ref:`view helpers <zend.view.helpers>` 处理mvc视图的展示。
