---
title: "如何在 Django 与 DRF 中优雅地校验权限"
slug: "django-permission"
date: "2020-07-05T05:24:00+0000"
lastmod: "2025-01-16T08:46:40+0000"
draft: false
tags:
  - "Django"
  - "DRF"
  - "Permission"
visibility: "public"
---
# 0X00 Django 中的权限结构、定义

我们知道在创建了一个 Django 项目之后，默认就有两个公开可用的 model：User 和 Group，这两个 model 的一项功能就是用来做权限管理的。系统中会有很多项权限，单个 user 可以配置拥有哪些权限，也可以将权限配置给 group。然后校验单个权限的时候其实就是将 user 本身的权限，和 user 所在的所有组的权限做一个并集，看本次操作的权限是否在这个并集里。在，那就校验通过；不在，那就只有 HTTP 403 了。

![](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/20200705173202.png)

> HTTP 401 和 HTTP 403 的区别：401 的描述是 Forbidden，而 401 是 Unauthorized，前者是没有权限，而后者干脆没通过认证。举个例子，你想查看公司财务的详细报表，财务经理一看你就是个一线小程序员，就给你一个 401，告诉你这不是你可以看的东西。如果你想看别的公司财务的详细报表，别人公司财务经理一看你根本不是他们公司的人，就直接给你了个 401 了。（俗话说十个比喻九个不准，我这个比喻当然也并不非常准确，不过对于分不清 401 和 403 的同学而言应该也问题不大🤣）

Django 自己对每一个 model 都创建了 create，update，delete 的权限，我们可以直接拿来用，也可以自己添加新权限。Django 自己是针对各个 model 做的权限，所以最简单的权限建立是在 model 层进行的。就比如下面这种，如果我想要为 Student 这个 model 建立相关的权限，就可以通过修改 `Meta` 类里的 `permissions` 来实现。

```python
    class Student(models.Model):
      	user = models.OneToOneField('django.contrib.auth.models.User')
      	name = models.CharField() # 纯展示，就不详细定义了
        birthday = models.DatetimeField()
        phone = models.CharField()

        class Meta:
    				verbose_name = '学生'
            verbose_name_plural = verbose_name
            permissions = (
            		# ('权限名', '权限描述'),
              	('check_classmate_score', '查看同班同学的成绩'),
              	('send_class_notify', '发送班级通知'),
            )
```

不过这里也看到了，每次对权限进行 CUD 的时候都是在改 model 的，所以每次改动完 model 记得都要进行一次`migrate`操作才行。不过不用担心性能问题，这个 migrate 只对 Permission 表进行 CUD 操作，而并非改表，所以非常快就搞定了。注意的一点是，不管你把这些 permissions 写在哪个 model 下，最终他们创建好的数据都还是在 Permission 表里，也就是数据库（我这里用的是 MySQL）里的`auth_permission`表了，这个表结构和数据是下面这样的。

```
    +-----------------+--------------+------+-----+---------+----------------+
    | Field           | Type         | Null | Key | Default | Extra          |
    +-----------------+--------------+------+-----+---------+----------------+
    | id              | int(11)      | NO   | PRI | <null>  | auto_increment |
    | name            | varchar(255) | NO   |     | <null>  |                |
    | content_type_id | int(11)      | NO   | MUL | <null>  |                |
    | codename        | varchar(100) | NO   |     | <null>  |                |
    +-----------------+--------------+------+-----+---------+----------------+



    +-----+------------------------------------+-----------------+-------------------------------+
    | id  | name                               | content_type_id | codename                      |
    +-----+------------------------------------+-----------------+-------------------------------+
    | 1   | Can add log entry                  | 1               | add_logentry                  |
    | 2   | Can change log entry               | 1               | change_logentry               |
    | 3   | Can delete log entry               | 1               | delete_logentry               |
    | 4   | Can add group                      | 2               | add_group                     |
    | 5   | Can change group                   | 2               | change_group                  |
    | 6   | Can delete group                   | 2               | delete_group                  |
    | 7   | Can add permission                 | 3               | add_permission                |
    | 8   | Can change permission              | 3               | change_permission             |
    | 9   | Can delete permission              | 3               | delete_permission             |
    | 10  | Can add user                       | 4               | add_user                      |
    | 11  | Can change user                    | 4               | change_user                   |
    | 12  | Can delete user                    | 4               | delete_user                   |
    +-----+------------------------------------+-----------------+-------------------------------+
```

# 0X01 在 Django 中校验与分配权限

权限在表里之后“唯一”生效的地方就是 django-admin 了，就是说如果登录 django-admin 的用户没有`add_user`的权限，那你创建用户的时候就会被拒绝。但是我们平时更多的时候并不是在 django-admin 里，而是自己实现的前台页面，那该怎么自己校验权限呢？Django 中给 user 实例了两个方法：`user.has_perm`和`user.has_perms`。显然，前者校验单个权限，后者校验多个权限。咱们先来看一下是如何校验的：

```python
    In [1]: from django.contrib.auth.models import User

    In [2]: shawn = User.objects.create(username='shawn')

    In [3]: shawn.has_perm('auth.add_user')	# 单个权限校验直接传权限的 code_name
    Out[3]: False

    In [4]: shawn.has_perms(['auth.add_user', 'auth.del_user'])  # 多个全线同时检验时将多个 code_name 放在列表中
    Out[4]: False

    In [5]: admin = User.objects.create(username='admin')

    In [6]: admin.is_superuser = True		# 设置 admin 用户为超级管理员

    In [7]: admin.save()

    In [8]: admin.has_perm('auth.add_user')
    Out[8]: True

    In [9]: admin.has_perms(['auth.add_user', 'auth.del_user'])
    Out[9]: True
```

我们从 Django 源码中可以看到，如果正在校验的用户是“活跃的”而且是“超级管理员”，那就直接不校验了，通过。否则就去校验一波。同时校验多个权限的时候用了`all`去逐个校验，遇到一个没有的权限也就 False 了。具体的细节这里就不多说了，感兴趣的话可以看一下源码，在`django.contrib.auth.models.User`中。

```python
    def has_perm(self, perm, obj=None):
        """
        Return True if the user has the specified permission. Query all
        available auth backends, but return immediately if any backend returns
        True. Thus, a user who has permission from a single auth backend is
        assumed to have permission in general. If an object is provided, check
        permissions for that object.
        """
        # Active superusers have all permissions.
        if self.is_active and self.is_superuser:
                return True

        # Otherwise we need to check the backends.
        return _user_has_perm(self, perm, obj)

    def has_perms(self, perm_list, obj=None):
        """
        Return True if the user has each of the specified permissions. If
        object is passed, check if the user has all required perms for it.
        """
        return all(self.has_perm(perm, obj) for perm in perm_list)
```

然后要考虑的就是如何将权限分给用户了，我们可以在 Django 源码中看到 user 和 permission 的关联关系是：ManyToMany ，所以直接修改用户的`user_permissions`字段就可以了。针对 group 也是一样的。

```python
    In [36]: shawn = User.objects.get(username='shawn')

    In [37]: perm = Permission.objects.get(codename='add_user')

    In [38]: shawn.user_permissions.add(perm)

    In [39]: shawn.save()

    In [40]: shawn.has_perm('auth.add_user')
    Out[40]: True
```

> 这里有一个比较干扰人的问题，就是为什么我的 codename 明明是 add_user 但是校验的时候却要用`auth.add_user`。当是这个问题也是困扰了我比较久，还因为这个问题导致了我程序出现 bug（比如本来该校验`car.open_door`的地方我校验了`open_door`）。比较明显的一处是因为“重名“，因为我们 model 多了之后会出现好多处重名的权限名，所以校验的时候应该在codename 前面加上 app_name。可以看一下我们 settings 里面的 INSTALLED_APPS 配置，有一条就是`django.contrib.auth`。

# 0X02 DRF 中的权限设计

我们在配置 DRF 的时候通常会在 settings 里写下类似这样的配置，这意味着给默认的 permission_class 设置成了`IsAuthenticated`，也就是说登录即可。

```python
    REST_FRAMEWORK = {
        'DEFAULT_PERMISSION_CLASSES': (
            'rest_framework.permissions.IsAuthenticated',
        ),
    	  #..............
    }
```

但是这也只是一个默认的权限配置，你用 DRF 的`ViewSet`创建的新 API 默认是登录即可，但是总会有其他的情况，这种时候通常来说我我们会重写 ViewSet 中的两个方法：`get_permissions`和`get_queryset`。先说正经的权限，也就是`get_permissions`好了。

```python
    class APIView(View):
        permission_classes = api_settings.DEFAULT_PERMISSION_CLASSES
        # .........

    		def get_permissions(self):
            """
            Instantiates and returns the list of permissions that this view requires.
            """
            return [permission() for permission in self.permission_classes]

        def check_permissions(self, request):
            """
            Check if the request should be permitted.
            Raises an appropriate exception if the request is not permitted.
            """
            for permission in self.get_permissions():
                if not permission.has_permission(request, self):
                    self.permission_denied(
                        request, message=getattr(permission, 'message', None)
                    )

        def check_object_permissions(self, request, obj):
            """
            Check if the request should be permitted for a given object.
            Raises an appropriate exception if the request is not permitted.
            """
            for permission in self.get_permissions():
                if not permission.has_object_permission(request, self, obj):
                    self.permission_denied(
                        request, message=getattr(permission, 'message', None)
                    )
```

我们从源码中可以看到这个方法其实很很简单，就只是把`self.permission_classes一遍，通过 codename 进行实例化，最终返回一个实例化的 permission 列表。然后在`check_permissions`或者`check_object_permissions`的时候对这个列表轮询一遍，调用 permission 实例的`has_object_permission(request, view)`方法来判断是否有权限执行本次请求，最终通过返回的 True/False 来确定是否有权限继续操作，如果没有权限就直接终止请求了。

根据上述条件，我们就可以自己写一个权限校验类了，一个简单的实现时下面这种样子的：

```python
    from rest_framework import permissions


    class PermsRequired(permissions.BasePermission):
    		"""需要有多个权限之一，也就是说有其中一个权限即可"""

        def __init__(self, *perms):
            self.perms = perms	# 接受多个参数，全部都是权限

        def __call__(self):
            return self  # 被 call（前期可以理解成被当成函数调用） 的时候返回自身

        def has_permission(self, request, view):
            user = request.user		# 既然传进来了 request，那就可以拿到 user 了

            if user.is_superuser:	# superuser 拥有一切权限
                return True

            user_perms = user.get_all_permissions()	 # 取得用户所拥有的的所有权限

            # 取本次需要的权限和用户所有的权限的交集，如果有交集则校验通过（也就是 “或” 校验
            return True if user_perms & set(self.perms) else False

          	# 如果想要“与”校验，可以改成下面这种
            # return all([perm in user_perms for perm in set(self.perms)])
```


# 0X03 DRF 如何校验权限

现在我们知道 Django 和 DRF 中的权限大概是怎么实现的了，那么具体怎么用在 DRF 框架的 web 程序中呢？下面看一段实例代码。这坨代码就是我们最常见的一个 DRF 开出来的 ViewSet 了，同时支持`GET/POST/PUT/PATCH/DELETE`五种 HTTP 方法。

> 在 DRF 的 ViewSet 中，可以使用 self.action 来访问到当前请求的 function。默认情况下的 CRUD 对应的方法名是：`list/create/retrieve/partial_update/update/delete`这五个，分别对应了`GET /`,`POST /`,`GET /id/`,`PATCH /id/`,`PUT /id/`,`DELETE /id/`

```python
    class StudentViewSet(viewsets.ModelViewSet):
      	queryset = models.Student.objects.all()
      	# ...........

        def get_permissions(self):
          	if self.action == 'list':		# 如果是对根目录进行 GET 则需要列表权限
     						self.permission_classes = (
                    perms.PermsRequired('student.list_student'),
                )
            if self.action == 'create':		# 如果对根目录进行 POST 则需要创建权限
                self.permission_classes = (
                    perms.PermsRequired('student.create_student'),
                )

            return super(StudentViewSet, self).get_permissions()

        def get_serializer_class(self):
          	# ........
            return self.serializer_class

        def get_queryset(self):
            user = self.request.user

            # 如果用户拥有查看所有学生信息的权限，则返回所有学生信息
            if user.has_perm('student.query_all_students'):
                return self.queryset

            # 如果用户有查看自己班里学生信息的权限，则返回自己班里的学生信息
            if user.has_perm('ratel.query_self_class_students'):
                return self.queryset.filter(
                    student_class=user.student.student_class
                )
            # ...还有其他校验

            # 如果什么权限都没找到，那就只能返回自己的数据了
            return models.Student.objects.filter(id=user.student.id)
```


我们可以从上面看到有两个地方在校验权限，一个是`get_permissions()`，另一个是`get_queryset()`。前者是严谨的“权限”，也就是说“你想要执行某某某操作，你就需要拥有某某某权限，没有就不信”；后者是广义的“权限”，会从大到小来校验，比如这里可以先判断你是否拥有查看所有人信息的权限，有，就给你所有人的信息；没有，就看你能不能看班里人的信息，有，就给你班里人的信息；没有，就只返回给你自己的数据。

这两种用法是在日常工作中用法最多的了。其他再有的话就是在 serializer 的 validate 中进行的校验，比如有一个功能叫做“强制删除”，这个功能可以跳过一切检测（比如检测学生是否在校，学生是否毕业超过3 年，学生饭卡是否还有钱），就直接删除。这种权限可能只会分配给高级别管理员使用，但是又不可能针对这种功能再单独开新的 API 来做，这样太奇怪了。所以我们可以通过一个参数来控制前置删除，比如`force_delete`。这样我们就可以写出类似下面的代码

```python
    def validate(self, validated_data):
      	user = self.context['request'].user
        force_delete = validated_data.get('force_delete', False)

        if force_delete:
          	if not user.has_perm('student.force_delete'):
            		raise serializers.ValidationError('你没有强制删除用户的权限')

      	return validated_data
```

# 0X04 THE END

今天是个美好的周日，中午的时候想要喝了咖啡写篇博客就去玩 _火焰纹章_ 的的，然而越看这个权限越越觉得自己不懂，之前一度以为自己挺明白的，结果一直是处在知其然的状态，并没有知其所以然（虽然现在知道也也不彻底）。然后就一点点看之前的代码，看实现方案，看着看着又得进去看源码，结果计划一个多小时搞定的东西搞了快四个小时。不过还好，也不亏，收获颇丰~
