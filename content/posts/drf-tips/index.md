---
title: "Django REST framework 中要注意的几个点"
slug: "drf-tips"
date: "2018-05-22T16:20:00+0000"
lastmod: "2025-01-16T08:14:15+0000"
draft: false
tags:
  - "Django"
  - "DRF"
visibility: "public"
---
# 0X00 Model中要注意的几点

## verbose_name 和 help_text 属性

Model中通常第一个参数指定的是`verbose_name`，还要手动指定一个`help_text`属性。其中`verbose_name`属性是用来我们自己读的，而`help_text`是用于提供字段描述类的功能，比如在DJango Admin中`verbose_name`会变成字段的中文名，而`help_text`则会变成改字段的描述。

## **unicode** 方法

每一个Model类我们最好都要重写一下这个`__unicode__`方法，使之返回一个有意义的数据。比如一个学生信息的Model，我们不去重写这个方法，最后在`ipython`中或是项目中直接调的话就是这个样子的`<QuerySet <Student: 1>, <Student: 2>, <Student: 3>]>`。如果我们重写了这个方法

```python
    def __unicode__(self):
        return '({gender}){name}'.format(gender=self.gender, name=self.name)
```

那么返回值就是``<QuerySet <Student: (男)小明>, <Student: (女)小红>, <Student: (女)小兰d>]>`。不仅是在调试过程中还是程序里都会有不错的效果。

## 关于choices

在设计Model中常会用到choices属性，比较好的用法是这样的。命名的时候使用在字段名后加`choice`的全大写，也就是：`GENDER_CHOICE`。

```python
    GENDER_CHOICE = (
        ('male', u'男'),
        ('female', u'女'),
        ('other', u'其他'),
    )

    gender = models.Charfield(
        u'性别',
        help_text=u'性别',
        max_length=100,
        choices=GENDER_CHOICE,
    )
```

# 0X01 Serializer中要注意的几点

## 针对list方法的Serializer

还是上面学生信息的这个例子，前端调用`GET`方法后想要得到的明显是`男、女、未知`这种，所以我们应该为所有类似的字段搭配返回一个对应的可读字段。例如在获取学生信息时可以这样写Serializer。

```python
    class ListStudentSerialzier(serializers.ModelSerializer):

        gender_cn = serializers.SerializerMethodField()

        def get_gender_cn(self, obj):
            return obj.get_gender_display()

        class Meta:
            model = Student
            fields = (
                'id',
                'name',
                'gender',
                'gender_cn',
                'birthday',
                'hobbys',
            )
```

在针对`List`的Serializer中添加字段`gender_cn`，顾名思义就是性别的中文，定义为`serializers.SerializerMethodField()`，定且在下面跌一个名为`def get_gender_ch(self, obj)`的方法，组装好所需要的数据返回就可以了。参数中的`obj`就是都应的实例化对象，在此处也就是一个`Student`对象。
最后要在`Meta`中的`fields`里加上这个字段。

另外，如果将Model中一个字段定义为`CharField`且Serializer处使用`ListField`进行校验存储的话，数据库中就会是类似`"[1, 2, 3, 4, 5]"`的“列表样子的字符串”。如果想让这种类型的字符串以一个正确的列表方式返回，例如字段`hobbys`，那么应该像`gender_cn`一样编写一个get方法，使用`json.loads()`的方式将“列表样子的字符串”转换为真正的列表返回。

## 针对Create和Update的Serializer

针对Create和Update的Serializer是要向`Django`推数据的，所以需要注意字段的合法性校验。要注意`Serializer`与`Model`中字段类型的对应，例如Serializer中的`ListFIeld`其实就是Model中的`CharField`。
