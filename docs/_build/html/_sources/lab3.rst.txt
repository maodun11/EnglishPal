============================================================
**Lab3**--The Service Layer
============================================================

-----------------------------------------------------------

:小组成员信息:

* 毛顿 201836900224
* 欧洲 201836900207
* 杨晗涵 201836900210
* 刘威 201836900222
* 来锦韬 201836900220

:项目GitHub地址: `EnglishPal <https://github.com/AWel11/EnglishPal>`_

:项目Read The Docs地址: `Read The Docs <https://readthedocs.org/projects/englishpal/>`_

Abstract
========

理解应用service layer。

Introduction
============

Service layer（又名orchestration layer或use-case layer）位于Flask API与domain model之间，负责管理orchestration logic，具体来说包括以下几点：

* 从repository中取出数据库数据
* 将从Flask API接受到的requset输入与数据库数据进行比较，处理异常与错误
* 调用domain model中的服务，对输入进行一系列操作
* 保存改动至数据库

当然这是还未引入uow（unit of work）的servie layer，与database layer仍具有高耦合性。

本实验的关键就在于对上面几个步骤的理解，并根据实验需求进行合理应用。

Materials and Methods
=====================

Materials
```````````````

* `Architecture Patterns with Python <https://www.cosmicpython.com/book/chapter_04_service_layer.html>`_ Chapter4 关于service layer的描述

Methods
````````

* 了解services layer的功能。 
* 阅读参考书籍与配套代码，加深对services layer的理解。
* 分析实验代码，明确实验需求，进行补充与调试。

Results
=======

*orm.py*::

   # Software Architecture and Design Patterns -- Lab 3 starter code
   # An implementation of the Service Layer
   # Copyright (C) 2021 Hui Lan
   
   
   # word and its difficulty level
   WORD_DIFFICULTY_LEVEL = {'starbucks':5, 'luckin':4, 'secondcup':4, 'costa':3, 'timhortons':3, 'frappuccino':6}
   
   
   class UnknownUser(Exception):
       pass
   
   
   class NoArticleMatched(Exception):
       pass
   
   
   def is_valid_user(username, password, users):
       return username in {u.username for u in users} and password in {u.password for u in users}
   
   
   def read(user, user_repo, article_repo, session):
       # fetch data
       users = user_repo.list()
       articles = article_repo.list()
   
       # check validity of user
       if not is_valid_user(user.username, user.password, users):
           raise UnknownUser(f'Invalid user')
   
       # get user's vocabulary level
       user = user_repo.get(user.username)
       Ulevels = [WORD_DIFFICULTY_LEVEL[newword.word] for newword in user.newwords]
       Ulevels.sort(reverse=True)
       num = len(Ulevels) if len(Ulevels) < 3 else 3
       top_Ulevels = Ulevels[:num]
       Ulevel = sum(top_Ulevels) / num
   
       # choose a suitable article and read it
       Alevels = [article.level for article in articles]
       Alevels_with_indices = sorted(enumerate(Alevels), key=lambda x:x[1])
       for Alevel_with_index in Alevels_with_indices:
           if Alevel_with_index[1] >= Ulevel:
               article = articles[Alevel_with_index[0]]
               user.read_article(article)
               session.commit()
               return article.article_id
       raise NoArticleMatched(f'No article matched') 

结果

.. image:: imgs/lab3/result.png

Discussions
===========

这次实验的目的较单一，相对于Lab2要对SqlAlchemy有一定的理解并熟悉规定语法来说，Lab3只要明确需求、编码、调试就结束了，可以说是相当线性。

因此这里就简单的描述下流程，不深究每条语句的作用或原因。

需求分析
````````````````````````

结合service layer的功能，确定 :code:`read()` 流程

#. 从 :code:`user_repo` 和 :code:`article_repo` 中取出数据。
#. 检查传入user的合法性，若不合法则抛出 :code:`UnknownUser` 异常。
#. 计算user的词汇水平。
#. 挑选一篇难度适合的article给user读，并返回 :code:`article_id` ; 若没有符合条件的article，则抛出 :code:`NoArticleMatched` 异常。

注意要点
```````````

* :code:`UnknownUser` 异常和 :code:`NoArticleMatched` 均继承自 :code:`Exception` ，因此无需添加其他属性或方法，使用时直接传入要提示的字符串信息即可。
* 传入的user仅仅是个 :code:`model.User` 对象，没有和数据库关联起来。在确认用户信息合法后，需要调用 :code:`user_repo` 中的 :code:`get()` 方法来获取关联对象。
* 计算user词汇水平时，先将该用户的词汇映射为对应的难度，按难度从高到低排序，并取出前几个（最多3个）计算平均难度，即是该用户的词汇水平。
* 选择难度适合的article时，先根据article的level进行排序，这样能选出与用户水平最贴切的文章，并调用 :code:`user.read_article()` 保存记录。
* 最后调用 :code:`session.commit()` 保存数据库操作。

实现细节请看源码。

问题回答
``````````````````````````````````

**Does your function read in services.py follow the Single Responsibility Principle (SRP) principle? Why or why not?**

:code:`read()` 并没有遵守SRP准则。若把上面需求分析中的每一点看成一个responsibility，那显然service layer已经break the principle了; 即使将整个流程当作一个任务，把service layer看成Flask API与domain layer的桥梁，它里面也包含了 :code:`session.commit()` 这本应属于数据库的操作。

References
==========

* `Architecture Patterns with Python <https://www.cosmicpython.com/book/chapter_04_service_layer.html>`_ Chapter4 关于service layer的描述
* `Code <https://github.com/cosmicpython/code>`_ 参考书籍对应代码 
