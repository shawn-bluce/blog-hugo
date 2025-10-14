---
title: "Django 中的 url"
slug: "django-url"
date: "2019-11-20T12:50:00+0000"
lastmod: "2025-01-16T08:28:09+0000"
draft: false
tags:
  - "Django"
  - "Python"
visibility: "public"
---
# 0X00 url的源头

使用`django-admin startproject test_project`创建一个新的Django项目之后在`settings.py`中可以找到一个配置项`ROOT_URLCONF`，默认情况下值为项目目录下的`urls`，也就是`test_project.urls`。

默认情况下这个`urls.py`的内容大致是这样的

```python
    """learn_django URL Configuration

    The `urlpatterns` list routes URLs to views. For more information please see:
        https://docs.djangoproject.com/en/2.2/topics/http/urls/
    Examples:
    Function views
        1. Add an import:  from my_app import views
        2. Add a URL to urlpatterns:  path('', views.home, name='home')
    Class-based views
        1. Add an import:  from other_app.views import Home
        2. Add a URL to urlpatterns:  path('', Home.as_view(), name='home')
    Including another URLconf
        1. Import the include() function: from django.urls import include, path
        2. Add a URL to urlpatterns:  path('blog/', include('blog.urls'))
    """
    from django.contrib import admin
    from django.urls import path


    urlpatterns = [
        path('admin/', admin.site.urls),
    ]
```

这里就是根目录了，新项目使用`python manage.py runserver`启动之后访问`http://127.0.0.1:8000/`就是访问到这个url的根路由了，默认情况下有一个`admin/`可选，也就是Django自己的后台管理页面。**Django所有的url都是从这个文件发散出去的** ，`urlpatterns`里除了将url路由至`view`就是其他的子url配置。换句话说，通常情况下Django中所有url最终都应该被路由到View上才对。

# 0X01 路由到子url和view

上面提到通常情况下Django中所有url最终都应该被路由到View上，那就来看一下究竟该怎么做。现在有一个项目，项目中有一个app叫`student`是用来管理一些学生信息的，app中有一个`views.py`，具体内容就暂时不列出了，在此处关系不大；还有一个`urls.py`，内容是这样的

```python
    from django.urls import path

    from .views import StudentView, ExamView


    urlpatterns = [
        path('^$', StudentView.as_view()),    # 直接将这个子url的根目录路由到一个view
        path('^exam/$', ExamView.as_view()),    # 直接将url路由至一个view
    ]
```

现在要来修改根的`urls.py`了

```python
    from django.contrib import admin
    from django.urls import path, include

    urlpatterns = [
        path('admin/', admin.site.urls),
        path('student/', include('test_project.student.urls')), # 其实就加了这一行，这里的include就是包含另一个urls的配置文件
    ]
```

现在就是这样的，如果访问`http://127.0.0.1:8000/admin`还是之前的DjangoAdmin默认管理页面没问题，如果访问`http://127.0.0.1:8000/student/`的话就是到`StudentView`而`http://127.0.0.1:8000/student/exam/`就是`ExamView`了。

# 0X02 参数与正则

url中常见的出现一些参数，一般来说参数分成两种：第一种是在path中作为路由的一部分，另一种是在后面以GET的查询参数方式出现例如`/student?name=shawn&age__gt=16`这种。第二种方式比较简单，在传入到view后，从view的`request.GET`就能取到了。但是也经常会遇到第一种，例如这样一个path`/article/2019/11/20/why-linux`，可以猜测它指的是2019年11月20日的一篇名为'why-linux'的文章。那么这种该怎么取呢？其实很简单，在view里`request.path.split('/')`然后取下标就行了（当然这很蠢且很不靠谱，但是还不失为一种方案哈哈哈哈）。

这种时候比较靠谱的方式是使用url中的参数，有一个子url配置如下

```python
    urlpatterns = [
        path('<int:year>/<int:month>/<int:day>/<str:name>/', StudentView.as_view()),
    ]
```

其中`<int:year>`就是指的一个参数，前三个是整型参数，最后一个是字符串参数。我们知道`view`中`GET`方法的定义是`def get(self, request, *args, **kwargs)`，那么其实后面的这个`**kwargs`里就是这里传进来的参数了，可以通过`year, month, day, name = kwargs['year'], kwargs['month'], kwargs['day'], kwargs['name']`这种类似的方式来取到对应的值

还是上面这个例子，咱们知道年号一定是正整数，月份一定是112之间，日期一定是131之间（先不考虑闰年和大小月的问题，只是方便探讨url）。那上面这个例子中的url如果我传一个`/article/0/666/233/test/`过去其实是没有意义的，所以需要一些简单的校验。那么众所周知，正则表达式非常适合做这种事情。下面来修改一下刚刚的这个url配置好了，修改后的配置Django就可以根据正则来匹配了。

```python
    from django.urls import re_path

    urlpatterns = [
        path(r'^[1-9]\d*/0[1-9]|1[0-2]/xxxxx', StudentView.as_view()),  # 完整正则好长，就不都贴在这儿了
    ]
```

> Django在匹配的时候是按照`urlpatterns`这个列表的下标顺序来的，所一说如果先符合了上面的规则，即时再符合下面的规则也不会继续判断下去了。

# 0X03 DRF中的router.register

一般使用Django的同时也会使用**Django REST framework** 了，所以也简单介绍一下在DRF中特有的一种路由方式好了。因为DRF中大量使用`ViewSet`而非标准的Django View，所以可以使用DRF封装的下面这种方式来建立路由

```python
    from rest_framework import routers
    from . import views

    router = routers.DefaultRouter()    # 实例化一个router
    router.register(r'student/', views.xxxxxxxxViewSet) # 注册viewset
    router.register(r'teacher/', views.xxxxxxxxViewSet)

    urlpatterns = router.urls   # 最后还是要生成urlpatterns
```

> 虽说是叫做`router`不过翻译成路由器总是有点怪怪的，哈哈哈
