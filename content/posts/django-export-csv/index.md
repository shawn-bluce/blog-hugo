---
title: "Django 相对优雅地导出 CSV"
slug: "django-export-csv"
date: "2020-06-28T14:40:00+0000"
lastmod: "2025-01-16T07:43:20+0000"
draft: false
tags:
  - "Django"
  - "CSV"
visibility: "public"
---
# 0X00 前言

> 一见程序员，立刻想到 web 开发，立刻想到后台管理系统，立刻想到数据展示，立刻想到数据筛选筛选，立刻想到数据统计，立刻想到导出 Excel 表格。产品经理的想象惟在这一层能够如此跃进。 --鲁迅：我不是，我没有，别瞎说

![我不是，我没有，别瞎说啊](https://blog-1251664340.cos.ap-chengdu.myqcloud.com/20200628225253.jpg)

虽然上面这种说法有点夸张了，不过确实很多很多很多人在工作中遇到过不止一次的需要在一个 web 系统里添加一个”数据导出”的功能，而且通常都是导出成 csv 这种文件。自然我也遇到了很多很多很多次，也写过那种最蠢的手拼逗号的 csv 导出，还看过别人效果更好代码量更少的版本。也就在此总结一下具体这个 csv 导出该怎么搞才好。

最蠢的方案可能就是我最早实习的时候写出来的那种手拼逗号的方案了，为了大家刚吃的早午晚饭着想，就不给大家看了，省得吐出来浪费粮食。真正用的比较多的是这么两种：一种是传统的拼接二维数组的方式来模拟表格，然后通过 Python 的 csv 库直接导出；另一种是使用 djcsv 来进行导出。下面来简单看一下嘞。

# 0X01 原始 csv 导出

其实简单的导出原始的方法就可以解决问题了，下面只列出重点代码。首先导入 csv 模块；然后打开一个文件用来保存这次导出的内容；接下来按行写入表头；然后循环整个 queryset 构建一行行的数据从而写入；最终导出就成功结束了。

```python
    import csv

    with open('export.csv', 'w') as f: # 打开待用的文件
        csv_writer = csv.writer(f)  # 生成一个 csvwriter

        table_title = ['姓名', '性别', '生日']  # 准备表头
        csv_writer.write_row(table_title)

        queryset = Student.objects.all().only('name', 'gender', 'birthday')
        for student in queryset:
            line = [
                student.name,
                student.gender,
                student.birthday.strftime('%Y-%m-%d'),
            ]
            csv_writer.write_row(line)
```

这样导出来的文件就是我们需要的内容了。我们来看一眼成果（不是诸葛大力，是我们导出来的 csv 文件🤣）： 发现问题了没？我们的性别出现了一丢丢问题，本来是用 M/F 来当做男女来存的（这种情况其实非常多的，比如你的类型可能用了 integer 然后用一个 map 去映射到不同的中文名上去），现在却把数据库中真实的内容导出来了。

| 姓名 | 性别 | 生日 |
| -————— |
| Kevin Armstrong | F | 1988-01-02 |
| Patricia Robinson | F | 1988-01-02 |
| Michael Duffy | F | 1988-01-02 |
| Kathryn Hodge | M | 1988-01-02 |
| Justin Carlson | F | 1988-01-02 |
| Larry Jones | F | 1988-01-02 |
| Peter Palmer | F | 1988-01-02 |
| William Smith | F | 1988-01-02 |
| Karen Garcia | M | 1988-01-02 |
| Eric Williams | F | 1988-01-02 |
| Eduardo Bell | F | 1988-01-02 |
| Cynthia Lee | M | 1988-01-02 |
| Brandy Hoffman | F | 1988-01-02 |
| Emily Jones | F | 1988-01-02 |
| Kelly Perry | M | 1988-01-02 |
| Jamie Nixon | F | 1988-01-02 |
| Jeffrey Vega | F | 1988-01-02 |
| John Chen | M | 1988-01-02 |
| Laura Stevens | M | 1988-01-02 |
| Linda Robinson | M | 1988-01-02 |

如果你说这种问题不大，那现在我们要求加一列 ”是否成年“ ，然后这个是否成年又没有存，只有用生日来计算，那咋搞？可能只有在`line = [xxxxx]`的地方再加一行`'成年' if (datetime.datetime.now() - student.birthday).days / 365 >= 18 else '未成年'`才行了。当然这只是理想情况，正常情况下一张业务表可能会有 100 多个字段，导出的时候可能要从这 100 多个字段中选择 80 多个导出来然后还要导出他们外键关联的其他表的数据。这是由上面这种写法就会越来越长，而且尤其是当”不能从数据库中直接取“的数据越来越多的时候就会麻烦了。下面介绍的这种使用 djcsv 导出的方法就很好用了。

# 0X02 使用 djcsv 导出

这种方法需要安装一个三方库 djcsv ，顾名思义它就是用来方便 Django 导出 csv 文件的。这坨代码的具体解释就直接写在下面注释里了。

```python
    from djcsv import write_csv

    def export_csv():
        queryset = Student.objects.all()
        queryset_values = queryset.values(
            'name',
            'gender',
            'birthday',
            'teacher__name',    # 跨表也是可以的
            'teacher__gender',
        )

        # 导出字段表头
        field_header_map = {
            'name': '姓名',
            'gender': '性别',
            'birthday': '生日',
            'teacher__name': '老师姓名',
            'teacher__gender': '老师性别',

        }

        # 格式化数据
        gender_dict = dict(Student.GENDER_CHOICES)
        """
        Student.GENDER_CHOICES = (
            ('M', '男'),
            ('F', '女),
        )
        """

        field_serializer_map = {    # 折页机就是为什么要叫 serializer 的方法了，因为确实有一个翻译在这儿
            'gender': (lambda x: gender_dict.get(x, '其他')),
            'teacher_gender': (lambda x: gender_dict.get(x, '其他')),
        }

        with open('export.csv', 'w') as csv_file:
            write_csv(
                queryset_values,
                csv_file,
                field_header_map=field_header_map,
                field_serializer_map=field_serializer_map
            )
```