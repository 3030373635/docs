---
title: ORM
---


# 一、字段

## 1.1 字段列表

```python
AutoField(Field)
        - int自增列，必须填入参数 primary_key=True

    BigAutoField(AutoField)
        - bigint自增列，必须填入参数 primary_key=True

        注：当model中如果没有自增列，则自动会创建一个列名为id的列
        from django.db import models

        class UserInfo(models.Model):
            # 自动创建一个列名为id的且为自增的整数列
            username = models.CharField(max_length=32)

        class Group(models.Model):
            # 自定义自增列
            nid = models.AutoField(primary_key=True)
            name = models.CharField(max_length=32)

    SmallIntegerField(IntegerField):
        - 小整数 -32768 ～ 32767

    PositiveSmallIntegerField(PositiveIntegerRelDbTypeMixin, IntegerField)
        - 正小整数 0 ～ 32767
    IntegerField(Field)
        - 整数列(有符号的) -2147483648 ～ 2147483647

    PositiveIntegerField(PositiveIntegerRelDbTypeMixin, IntegerField)
        - 正整数 0 ～ 2147483647

    BigIntegerField(IntegerField):
        - 长整型(有符号的) -9223372036854775808 ～ 9223372036854775807

    BooleanField(Field)
        - 布尔值类型

    NullBooleanField(Field):
        - 可以为空的布尔值

    CharField(Field)
        - 字符类型
        - 必须提供max_length参数， max_length表示字符长度

    TextField(Field)
        - 文本类型

    EmailField(CharField)：
        - 字符串类型，Django Admin以及ModelForm中提供验证机制

    IPAddressField(Field)
        - 字符串类型，Django Admin以及ModelForm中提供验证 IPV4 机制

    GenericIPAddressField(Field)
        - 字符串类型，Django Admin以及ModelForm中提供验证 Ipv4和Ipv6
        - 参数：
            protocol，用于指定Ipv4或Ipv6， 'both',"ipv4","ipv6"
            unpack_ipv4， 如果指定为True，则输入::ffff:192.0.2.1时候，可解析为192.0.2.1，开启刺功能，需要protocol="both"

    URLField(CharField)
        - 字符串类型，Django Admin以及ModelForm中提供验证 URL

    SlugField(CharField)
        - 字符串类型，Django Admin以及ModelForm中提供验证支持 字母、数字、下划线、连接符（减号）

    CommaSeparatedIntegerField(CharField)
        - 字符串类型，格式必须为逗号分割的数字

    UUIDField(Field)
        - 字符串类型，Django Admin以及ModelForm中提供对UUID格式的验证

    FilePathField(Field)
        - 字符串，Django Admin以及ModelForm中提供读取文件夹下文件的功能
        - 参数：
                path,                      文件夹路径
                match=None,                正则匹配
                recursive=False,           递归下面的文件夹
                allow_files=True,          允许文件
                allow_folders=False,       允许文件夹

    FileField(Field)
        - 字符串，路径保存在数据库，文件上传到指定目录
        - 参数：
            upload_to = ""      上传文件的保存路径
            storage = None      存储组件，默认django.core.files.storage.FileSystemStorage

    ImageField(FileField)
        - 字符串，路径保存在数据库，文件上传到指定目录
        - 参数：
            upload_to = ""      上传文件的保存路径
            storage = None      存储组件，默认django.core.files.storage.FileSystemStorage
            width_field=None,   上传图片的高度保存的数据库字段名（字符串）
            height_field=None   上传图片的宽度保存的数据库字段名（字符串）

    DateTimeField(DateField)
        - 日期+时间格式 YYYY-MM-DD HH:MM[:ss[.uuuuuu]][TZ]

    DateField(DateTimeCheckMixin, Field)
        - 日期格式      YYYY-MM-DD

    TimeField(DateTimeCheckMixin, Field)
        - 时间格式      HH:MM[:ss[.uuuuuu]]

    DurationField(Field)
        - 长整数，时间间隔，数据库中按照bigint存储，ORM中获取的值为datetime.timedelta类型

    FloatField(Field)
        - 浮点型

    DecimalField(Field)
        - 10进制小数
        - 参数：
            max_digits，小数总长度
            decimal_places，小数位长度

    BinaryField(Field)
        - 二进制类型
```

## 1.2 自定义无符号整数字段

```python
class UnsignedIntegerField(models.IntegerField):
    def db_type(self, connection):
        return 'integer UNSIGNED'

PS: 返回值为字段在数据库中的属性，Django字段默认的值为：
    'AutoField': 'integer AUTO_INCREMENT',
    'BigAutoField': 'bigint AUTO_INCREMENT',
    'BinaryField': 'longblob',
    'BooleanField': 'bool',
    'CharField': 'varchar(%(max_length)s)',
    'CommaSeparatedIntegerField': 'varchar(%(max_length)s)',
    'DateField': 'date',
    'DateTimeField': 'datetime',
    'DecimalField': 'numeric(%(max_digits)s, %(decimal_places)s)',
    'DurationField': 'bigint',
    'FileField': 'varchar(%(max_length)s)',
    'FilePathField': 'varchar(%(max_length)s)',
    'FloatField': 'double precision',
    'IntegerField': 'integer',
    'BigIntegerField': 'bigint',
    'IPAddressField': 'char(15)',
    'GenericIPAddressField': 'char(39)',
    'NullBooleanField': 'bool',
    'OneToOneField': 'integer',
    'PositiveIntegerField': 'integer UNSIGNED',
    'PositiveSmallIntegerField': 'smallint UNSIGNED',
    'SlugField': 'varchar(%(max_length)s)',
    'SmallIntegerField': 'smallint',
    'TextField': 'longtext',
    'TimeField': 'time',
    'UUIDField': 'char(32)',
```

## 1.3 注意事项

```python
1.触发Model中的验证和错误提示有两种方式：
        a. Django Admin中的错误信息会优先根据Admiin内部的ModelForm错误信息提示，如果都成功，才来检查Model的字段并显示指定错误信息
        b. 使用ModelForm
        c. 调用Model对象的 clean_fields 方法，如：
            # models.py
            class UserInfo(models.Model):
                nid = models.AutoField(primary_key=True)
                username = models.CharField(max_length=32)

                email = models.EmailField(error_messages={'invalid': '格式错了.'})

            # views.py
            def index(request):
                obj = models.UserInfo(username='11234', email='uu')
                try:
                    print(obj.clean_fields())
                except Exception as e:
                    print(e)
                return HttpResponse('ok')

           # Model的clean方法是一个钩子，可用于定制操作，如：上述的异常处理。

    2.Admin中修改错误提示
        # admin.py
        from django.contrib import admin
        from model_club import models
        from django import forms


        class UserInfoForm(forms.ModelForm):
            age = forms.IntegerField(initial=1, error_messages={'required': '请输入数值.', 'invalid': '年龄必须为数值.'})

            class Meta:
                model = models.UserInfo
                # fields = ('username',)
                fields = "__all__"
                exclude = ['title']
                labels = { 'name':'Writer', }
                help_texts = {'name':'some useful help text.',}
                error_messages={ 'name':{'max_length':"this writer name is too long"} }
                widgets={'name':Textarea(attrs={'cols':80,'rows':20})}

        class UserInfoAdmin(admin.ModelAdmin):
            form = UserInfoForm

        admin.site.register(models.UserInfo, UserInfoAdmin)
```

# 二、字段参数

```pytho
null                数据库中字段是否可以为空
    db_column           数据库中字段的列名
    default             数据库中字段的默认值
    primary_key         数据库中字段是否为主键
    db_index            数据库中字段是否可以建立索引
    unique              数据库中字段是否可以建立唯一索引
    unique_for_date     数据库中字段【日期】部分是否可以建立唯一索引
    unique_for_month    数据库中字段【月】部分是否可以建立唯一索引
    unique_for_year     数据库中字段【年】部分是否可以建立唯一索引
        例如要创建联合索引（唯一，不唯一索引）
                    在表对应的类里面加如下代码:
                       class Meta:
                           unique_together = (("name","age")) 唯一联合索引
                           index_together =  (("name","age")) 不唯一联合索引


    verbose_name        Admin中显示的字段名称
    blank               Admin中是否允许用户输入为空
    editable            Admin中是否可以编辑
    help_text           Admin中该字段的提示信息
    choices             Admin中显示选择框的内容，用不变动的数据放在内存中从而避免跨表操作
                        如：gf = models.IntegerField(choices=[(0, '何穗'),(1, '大表姐'),],default=1)

    error_messages      自定义错误信息（字典类型），从而定制想要显示的错误信息；
                        字典健：null, blank, invalid, invalid_choice, unique, and unique_for_date
                        如：{'null': "不能为空.", 'invalid': '格式错误'}

    validators          自定义错误验证（列表类型），从而定制想要的验证规则
                        from django.core.validators import RegexValidator
                        from django.core.validators import EmailValidator,URLValidator,DecimalValidator,\
                        MaxLengthValidator,MinLengthValidator,MaxValueValidator,MinValueValidator
                        如：
                            test = models.CharField(
                                max_length=32,
                                error_messages={
                                    'c1': '优先错信息1',
                                    'c2': '优先错信息2',
                                    'c3': '优先错信息3',
                                },
                                validators=[
                                    RegexValidator(regex='root_\d+', message='错误了', code='c1'),
                                    RegexValidator(regex='root_112233\d+', message='又错误了', code='c2'),
                                    EmailValidator(message='又错误了', code='c3'), ]
                            )
```

# 三、元信息

```pytho
class UserInfo(models.Model):
        nid = models.AutoField(primary_key=True)
        username = models.CharField(max_length=32)
        class Meta:
            # 数据库中生成的表名称 默认 app名称 + 下划线 + 类名
            db_table = "table_name"

            # 联合索引
            index_together = [
                ("pub_date", "deadline"),
            ]

            # 联合唯一索引
            unique_together = (("driver", "restaurant"),)

            # admin中显示的表名称
            verbose_name

            # verbose_name加s
            verbose_name_plural
  
            #  abstract=True
            # 抽象模型，一般用于设置公共模型字段的，一旦设置这个相关以后，那么dajngo在数据迁移的时候就不会为这个模型单独创建一个数据表了
```

# 四、表关系以及参数

```python
ForeignKey(ForeignObject) # ForeignObject(RelatedField)
        to,                         # 要进行关联的表名
        to_field=None,              # 要关联的表中的字段名称
        on_delete=None,             # 当删除关联表中的数据时，当前表与其关联的行的行为
                                        - models.CASCADE，删除关联数据，与之关联也删除
                                        - models.DO_NOTHING，删除关联数据，引发错误IntegrityError
                                        - models.PROTECT，删除关联数据，引发错误ProtectedError
                                        - models.SET_NULL，删除关联数据，与之关联的值设置为null（前提FK字段需要设置为可空）
                                        - models.SET_DEFAULT，删除关联数据，与之关联的值设置为默认值（前提FK字段需要设置默认值）
                                        - models.SET，删除关联数据，
                                                      a. 与之关联的值设置为指定值，设置：models.SET(值)
                                                      b. 与之关联的值设置为可执行对象的返回值，设置：models.SET(可执行对象)

                                                        def func():
                                                            return 10

                                                        class MyModel(models.Model):
                                                            user = models.ForeignKey(
                                                                to="User",
                                                                to_field="id"
                                                                on_delete=models.SET(func),)
        related_name=None,          # 反向操作时，使用的字段名，用于代替 【表名_set】 如： obj.表名_set.all()
        related_query_name=None,    # 反向操作时，使用的连接前缀，用于替换【表名】     如： models.UserGroup.objects.filter(表名__字段名=1).values('表名__字段名')
        limit_choices_to=None,      # 在Admin或ModelForm中显示关联数据时，提供的条件：
                                    # 如：
                                            - limit_choices_to={'nid__gt': 5}
                                            - limit_choices_to=lambda : {'nid__gt': 5}

                                            from django.db.models import Q
                                            - limit_choices_to=Q(nid__gt=10)
                                            - limit_choices_to=Q(nid=8) | Q(nid__gt=10)
                                            - limit_choices_to=lambda : Q(Q(nid=8) | Q(nid__gt=10)) & Q(caption='root')
        db_constraint=True          # 是否在数据库中创建外键约束
        parent_link=False           # 在Admin中是否显示关联数据


    OneToOneField(ForeignKey)
        to,                         # 要进行关联的表名
        to_field=None               # 要关联的表中的字段名称
        on_delete=None,             # 当删除关联表中的数据时，当前表与其关联的行的行为

                                    ###### 对于一对一 ######
                                    # 1. 一对一其实就是 一对多 + 唯一索引
                                    # 2.当两个类之间有继承关系时，默认会创建一个一对一字段
                                    # 如下会在A表中额外增加一个c_ptr_id列且唯一：
                                            class C(models.Model):
                                                nid = models.AutoField(primary_key=True)
                                                part = models.CharField(max_length=12)

                                            class A(C):
                                                id = models.AutoField(primary_key=True)
                                                code = models.CharField(max_length=1)

    ManyToManyField(RelatedField)
        to,                         # 要进行关联的表名
        related_name=None,          # 反向操作时，使用的字段名，用于代替 【表名_set】 如： obj.表名_set.all()
        related_query_name=None,    # 反向操作时，使用的连接前缀，用于替换【表名】     如： models.UserGroup.objects.filter(表名__字段名=1).values('表名__字段名')
        limit_choices_to=None,      # 在Admin或ModelForm中显示关联数据时，提供的条件：
                                    # 如：
                                            - limit_choices_to={'nid__gt': 5}
                                            - limit_choices_to=lambda : {'nid__gt': 5}

                                            from django.db.models import Q
                                            - limit_choices_to=Q(nid__gt=10)
                                            - limit_choices_to=Q(nid=8) | Q(nid__gt=10)
                                            - limit_choices_to=lambda : Q(Q(nid=8) | Q(nid__gt=10)) & Q(caption='root')
        symmetrical=None,           # 仅用于多对多自关联时，symmetrical用于指定内部是否创建反向操作的字段
                                    # 做如下操作时，不同的symmetrical会有不同的可选字段
                                        models.BB.objects.filter(...)

                                        # 可选字段有：code, id, m1
                                            class BB(models.Model):

                                            code = models.CharField(max_length=12)
                                            m1 = models.ManyToManyField('self',symmetrical=True)

                                        # 可选字段有: bb, code, id, m1
                                            class BB(models.Model):

                                            code = models.CharField(max_length=12)
                                            m1 = models.ManyToManyField('self',symmetrical=False)

        through=None,               # 自定义第三张表时，使用字段用于指定关系表
        through_fields=None,        # 自定义第三张表时，使用字段用于指定关系表中那些字段做多对多关系表
                                        from django.db import models

                                        class Person(models.Model):
                                            name = models.CharField(max_length=50)

                                        class Group(models.Model):
                                            name = models.CharField(max_length=128)
                                            members = models.ManyToManyField(
                                                Person,
                                                through='Membership',
                                                through_fields=('group', 'person'),
                                            )

                                        class Membership(models.Model):
                                            group = models.ForeignKey(Group, on_delete=models.CASCADE)
                                            person = models.ForeignKey(Person, on_delete=models.CASCADE)
                                            inviter = models.ForeignKey(
                                                Person,
                                                on_delete=models.CASCADE,
                                                related_name="membership_invites",
                                            )
                                            invite_reason = models.CharField(max_length=64)
        db_constraint=True,         # 是否在数据库中创建外键约束
        db_table=None,              # 默认创建第三张表时，数据库中表的名称
```

# 五、ORM操作

## 5.1 基本操作

```python
				# 增
  			# 
        # models.Tb1.objects.create(c1='xx', c2='oo')  增加一条数据，可以接受字典类型数据 **kwargs

        # obj = models.Tb1(c1='xx', c2='oo')
        # obj.save()

        # 查
        #
        # models.Tb1.objects.get(id=123)         # 获取单条数据，不存在则报错（不建议）
        # models.Tb1.objects.all()               # 获取全部
        # models.Tb1.objects.filter(name='seven') # 获取指定条件的数据
        # models.Tb1.objects.exclude(name='seven') # 获取指定条件的数据

        # 删
        #
        # models.Tb1.objects.filter(name='seven').delete() # 删除指定条件的数据

        # 改
        # models.Tb1.objects.filter(name='seven').update(gender='0')  # 将指定条件的数据更新，均支持 **kwargs
        # obj = models.Tb1.objects.get(id=1)
        # obj.c1 = '111'
        # obj.save()                                                 # 修改单条数据
```

## 5.2 进阶操作

```python
 # 获取个数
        #
        # models.Tb1.objects.filter(name='seven').count()

        # 大于，小于
        #
        # models.Tb1.objects.filter(id__gt=1)              # 获取id大于1的值
        # models.Tb1.objects.filter(id__gte=1)              # 获取id大于等于1的值
        # models.Tb1.objects.filter(id__lt=10)             # 获取id小于10的值
        # models.Tb1.objects.filter(id__lte=10)             # 获取id小于10的值
        # models.Tb1.objects.filter(id__lt=10, id__gt=1)   # 获取id大于1 且 小于10的值

        # in
        #
        # models.Tb1.objects.filter(id__in=[11, 22, 33])   # 获取id等于11、22、33的数据
        # models.Tb1.objects.exclude(id__in=[11, 22, 33])  # not in

        # isnull
        # Entry.objects.filter(pub_date__isnull=True)

        # contains
        #
        # models.Tb1.objects.filter(name__contains="ven")
        # models.Tb1.objects.filter(name__icontains="ven") # icontains大小写不敏感
        # models.Tb1.objects.exclude(name__icontains="ven")

        # range
        #
        # models.Tb1.objects.filter(id__range=[1, 2])   # 范围bettwen and

        # 其他类似
        #
        # startswith，istartswith, endswith, iendswith,

        # order by
        #
        # models.Tb1.objects.filter(name='seven').order_by('id')    # asc
        # models.Tb1.objects.filter(name='seven').order_by('-id')   # desc

        # group by
        #
        # from django.db.models import Count, Min, Max, Sum
        # models.Tb1.objects.filter(c1=1).values('id').annotate(c=Count('num'))
        # SELECT "app01_tb1"."id", COUNT("app01_tb1"."num") AS "c" FROM "app01_tb1" WHERE "app01_tb1"."c1" = 1 GROUP BY "app01_tb1"."id"

        # limit 、offset
        #
        # models.Tb1.objects.all()[10:20]

        # regex正则匹配，iregex 不区分大小写
        #
        # Entry.objects.get(title__regex=r'^(An?|The) +')
        # Entry.objects.get(title__iregex=r'^(an?|the) +')

        # date
        #
        # Entry.objects.filter(pub_date__date=datetime.date(2005, 1, 1))
        # Entry.objects.filter(pub_date__date__gt=datetime.date(2005, 1, 1))

        # year
        #
        # Entry.objects.filter(pub_date__year=2005)
        # Entry.objects.filter(pub_date__year__gte=2005)

        # month
        #
        # Entry.objects.filter(pub_date__month=12)
        # Entry.objects.filter(pub_date__month__gte=6)

        # day
        #
        # Entry.objects.filter(pub_date__day=3)
        # Entry.objects.filter(pub_date__day__gte=3)

        # week_day
        #
        # Entry.objects.filter(pub_date__week_day=2)
        # Entry.objects.filter(pub_date__week_day__gte=2)

        # hour
        #
        # Event.objects.filter(timestamp__hour=23)
        # Event.objects.filter(time__hour=5)
        # Event.objects.filter(timestamp__hour__gte=12)

        # minute
        #
        # Event.objects.filter(timestamp__minute=29)
        # Event.objects.filter(time__minute=46)
        # Event.objects.filter(timestamp__minute__gte=29)

        # second
        #
        # Event.objects.filter(timestamp__second=31)
        # Event.objects.filter(time__second=2)
        # Event.objects.filter(timestamp__second__gte=31)
```

## 5.3 高级操作

```python
# extra
        #
        # extra(self, select=None, where=None, params=None, tables=None, order_by=None, select_params=None)
        #    Entry.objects.extra(select={'new_id': "select col from sometable where othercol > %s"}, select_params=(1,))
        #    Entry.objects.extra(where=['headline=%s'], params=['Lennon'])
        #    Entry.objects.extra(where=["foo='a' OR bar = 'a'", "baz = 'a'"])
        #    Entry.objects.extra(select={'new_id': "select id from tb where id > %s"}, select_params=(1,), order_by=['-nid'])

        # F(更新字段)
        #
        # from django.db.models import F
        # models.Tb1.objects.update(num=F('num')+1)


        # Q(构造复杂的where条件)
        #
        # 方式一：
        # Q(nid__gt=10)
        # Q(nid=8) | Q(nid__gt=10)
        # Q(Q(nid=8) | Q(nid__gt=10)) & Q(caption='root')

        # 方式二：
        # con = Q()
        # q1 = Q()
        # q1.connector = 'OR'
        # q1.children.append(('id', 1))
        # q1.children.append(('id', 10))
        # q1.children.append(('id', 9))
        # q2 = Q()
        # q2.connector = 'OR'
        # q2.children.append(('c1', 1))
        # q2.children.append(('c1', 10))
        # q2.children.append(('c1', 9))
        # con.add(q1, 'AND')
        # con.add(q2, 'AND')
        #
        # models.Tb1.objects.filter(con)


        # 执行原生SQL
        #
        # from django.db import connection, connections
        # cursor = connection.cursor()  # cursor = connections['default'].cursor()
        # cursor.execute("""SELECT * from auth_user where id = %s""", [1])
        # row = cursor.fetchone()
```

## 5.4 其他操作

```python
##################################################################
# PUBLIC METHODS THAT ALTER ATTRIBUTES AND RETURN A NEW QUERYSET #
##################################################################

def all(self)
    # 获取所有的数据对象

def filter(self, *args, **kwargs)
    # 条件查询
    # 条件可以是：参数，字典，Q

def exclude(self, *args, **kwargs)
    # 条件查询
    # 条件可以是：参数，字典，Q

def select_related(self, *fields)
     性能相关：表之间进行join连表操作，一次性获取关联的数据。
     model.tb.objects.all().select_related()
     model.tb.objects.all().select_related('外键字段')
     model.tb.objects.all().select_related('外键字段__外键字段')

def prefetch_related(self, *lookups)
    性能相关：多表连表操作时速度会慢，使用其执行多次SQL查询在Python代码中实现连表操作。
            # 获取所有用户表
            # 获取用户类型表where id in (用户表中的查到的所有用户ID)
            models.UserInfo.objects.prefetch_related('外键字段')



            from django.db.models import Count, Case, When, IntegerField
            Article.objects.annotate(
                numviews=Count(Case(
                    When(readership__what_time__lt=treshold, then=1),
                    output_field=CharField(),
                ))
            )

            students = Student.objects.all().annotate(num_excused_absences=models.Sum(
                models.Case(
                    models.When(absence__type='Excused', then=1),
                default=0,
                output_field=models.IntegerField()
            )))

def annotate(self, *args, **kwargs)
    # 用于实现聚合group by查询

    from django.db.models import Count, Avg, Max, Min, Sum

    v = models.UserInfo.objects.values('u_id').annotate(uid=Count('u_id'))
    # SELECT u_id, COUNT(ui) AS `uid` FROM UserInfo GROUP BY u_id

    v = models.UserInfo.objects.values('u_id').annotate(uid=Count('u_id')).filter(uid__gt=1)
    # SELECT u_id, COUNT(ui_id) AS `uid` FROM UserInfo GROUP BY u_id having count(u_id) > 1

    v = models.UserInfo.objects.values('u_id').annotate(uid=Count('u_id',distinct=True)).filter(uid__gt=1)
    # SELECT u_id, COUNT( DISTINCT ui_id) AS `uid` FROM UserInfo GROUP BY u_id having count(u_id) > 1

def distinct(self, *field_names)
    # 用于distinct去重
    models.UserInfo.objects.values('nid').distinct()
    # select distinct nid from userinfo

    注：只有在PostgreSQL中才能使用distinct进行去重

def order_by(self, *field_names)
    # 用于排序
    models.UserInfo.objects.all().order_by('-id','age')

def extra(self, select=None, where=None, params=None, tables=None, order_by=None, select_params=None)
    # 构造额外的查询条件或者映射，如：子查询

    Entry.objects.extra(select={'new_id': "select col from sometable where othercol > %s"}, select_params=(1,))
    Entry.objects.extra(where=['headline=%s'], params=['Lennon'])
    Entry.objects.extra(where=["foo='a' OR bar = 'a'", "baz = 'a'"])
    Entry.objects.extra(select={'new_id': "select id from tb where id > %s"}, select_params=(1,), order_by=['-nid'])

 def reverse(self):
    # 倒序
    models.UserInfo.objects.all().order_by('-nid').reverse()
    # 注：如果存在order_by，reverse则是倒序，如果多个排序则一一倒序


 def defer(self, *fields):
    models.UserInfo.objects.defer('username','id')
    或
    models.UserInfo.objects.filter(...).defer('username','id')
    #映射中排除某列数据

 def only(self, *fields):
    #仅取某个表中的数据
     models.UserInfo.objects.only('username','id')
     或
     models.UserInfo.objects.filter(...).only('username','id')

 def using(self, alias):
     指定使用的数据库，参数为别名（setting中的设置）


##################################################
# PUBLIC METHODS THAT RETURN A QUERYSET SUBCLASS #
##################################################

def raw(self, raw_query, params=None, translations=None, using=None):
    # 执行原生SQL
    models.UserInfo.objects.raw('select * from userinfo')

    # 如果SQL是其他表时，必须将名字设置为当前UserInfo对象的主键列名
    models.UserInfo.objects.raw('select id as nid from 其他表')

    # 为原生SQL设置参数
    models.UserInfo.objects.raw('select id as nid from userinfo where nid>%s', params=[12,])

    # 将获取的到列名转换为指定列名
    name_map = {'first': 'first_name', 'last': 'last_name', 'bd': 'birth_date', 'pk': 'id'}
    Person.objects.raw('SELECT * FROM some_other_table', translations=name_map)

    # 指定数据库
    models.UserInfo.objects.raw('select * from userinfo', using="default")

    ################### 原生SQL ###################
    from django.db import connection, connections
    cursor = connection.cursor()  # cursor = connections['default'].cursor()
    cursor.execute("""SELECT * from auth_user where id = %s""", [1])
    row = cursor.fetchone() # fetchall()/fetchmany(..)


def values(self, *fields):
    # 获取每行数据为字典格式

def values_list(self, *fields, **kwargs):
    # 获取每行数据为元祖

def dates(self, field_name, kind, order='ASC'):
    # 根据时间进行某一部分进行去重查找并截取指定内容
    # kind只能是："year"（年）, "month"（年-月）, "day"（年-月-日）
    # order只能是："ASC"  "DESC"
    # 并获取转换后的时间
        - year : 年-01-01
        - month: 年-月-01
        - day  : 年-月-日

    models.DatePlus.objects.dates('ctime','day','DESC')

def datetimes(self, field_name, kind, order='ASC', tzinfo=None):
    # 根据时间进行某一部分进行去重查找并截取指定内容，将时间转换为指定时区时间
    # kind只能是 "year", "month", "day", "hour", "minute", "second"
    # order只能是："ASC"  "DESC"
    # tzinfo时区对象
    models.DDD.objects.datetimes('ctime','hour',tzinfo=pytz.UTC)
    models.DDD.objects.datetimes('ctime','hour',tzinfo=pytz.timezone('Asia/Shanghai'))

    """
    pip3 install pytz
    import pytz
    pytz.all_timezones
    pytz.timezone(‘Asia/Shanghai’)
    """

def none(self):
    # 空QuerySet对象


####################################
# METHODS THAT DO DATABASE QUERIES #
####################################

def aggregate(self, *args, **kwargs):
   # 聚合函数，获取字典类型聚合结果
   from django.db.models import Count, Avg, Max, Min, Sum
   result = models.UserInfo.objects.aggregate(k=Count('u_id', distinct=True), n=Count('nid'))
   ===> {'k': 3, 'n': 4}

def count(self):
   # 获取个数

def get(self, *args, **kwargs):
   # 获取单个对象

def create(self, **kwargs):
   # 创建对象

def bulk_create(self, objs, batch_size=None):
    # 批量插入
    # batch_size表示一次插入的个数
    objs = [
        models.DDD(name='r11'),
        models.DDD(name='r22')
    ]
    models.DDD.objects.bulk_create(objs, 10)

def get_or_create(self, defaults=None, **kwargs):
    # 如果存在，则获取，否则，创建
    # defaults 指定创建时，其他字段的值
    obj, created = models.UserInfo.objects.get_or_create(username='root1', defaults={'email': '1111111','u_id': 2, 't_id': 2})

def update_or_create(self, defaults=None, **kwargs):
    # 如果存在，则更新，否则，创建
    # defaults 指定创建时或更新时的其他字段
    obj, created = models.UserInfo.objects.update_or_create(username='root1', defaults={'email': '1111111','u_id': 2, 't_id': 1})

def first(self):
   # 获取第一个

def last(self):
   # 获取最后一个

def in_bulk(self, id_list=None):
   # 根据主键ID进行查找
   id_list = [11,21,31]
   models.DDD.objects.in_bulk(id_list)

def delete(self):
   # 删除

def update(self, **kwargs):
    # 更新

def exists(self):
   # 是否有结果
```

## 5.5 连表正反操作

```python
基于对象的子查询
    一对一: 正向：按字段 反向：按表名小写
    一对多: 正向：按字段 反向：按表名小写_set
    多对多: 正向：按字段 反向：按表名小写_set

基于queryset(多表连接查询)
    用__ 告诉orm连接哪个表
  
    一对一: 正向：按字段 反向：按表名小写
    一对多: 正向：按字段 反向：按表名小写
    多对多: 正向：按字段 反向：按表名小写
```

# 六、其他

## 6.1 Django原生SQL获取cursor字典

```python
import pymysql
    from django.db import connection, connections

    connection.connect()
    conn = connection.connection
    cursor = conn.cursor(cursor=pymysql.cursors.DictCursor)
    cursor.execute("""SELECT * from app01_userinfo""")
    row = cursor.fetchone()
    connection.close()
```

## 6.2 数字自增、字符串更新

```python
# 数字自增
    from django.db.models import F
    models.UserInfo.objects.update(num=F('num') + 1)

    # 字符串更新
    from django.db.models.functions import Concat
    from django.db.models import Value

    models.UserInfo.objects.update(name=Concat('name', 'pwd'))
    models.UserInfo.objects.update(name=Concat('name', Value('666')))
```

## 6.3 ORM函数相关

```python
# ########### 基础函数 ###########

    # 1. Concat，用于做类型转换
    # v = models.UserInfo.objects.annotate(c=Cast('pwd', FloatField()))

    # 2. Coalesce，从前向后，查询第一个不为空的值
    # v = models.UserInfo.objects.annotate(c=Coalesce('name', 'pwd'))
    # v = models.UserInfo.objects.annotate(c=Coalesce(Value('666'),'name', 'pwd'))

    # 3. Concat，拼接
    # models.UserInfo.objects.update(name=Concat('name', 'pwd'))
    # models.UserInfo.objects.update(name=Concat('name', Value('666')))
    # models.UserInfo.objects.update(name=Concat('name', Value('666'),Value('999')))

    # 4.ConcatPair，拼接（仅两个参数）
    # v = models.UserInfo.objects.annotate(c=ConcatPair('name', 'pwd'))
    # v = models.UserInfo.objects.annotate(c=ConcatPair('name', Value('666')))

    # 5.Greatest，获取比较大的值;least 获取比较小的值;
    # v = models.UserInfo.objects.annotate(c=Greatest('id', 'pwd',output_field=FloatField()))

    # 6.Length，获取长度
    # v = models.UserInfo.objects.annotate(c=Length('name'))

    # 7. Lower,Upper,变大小写
    # v = models.UserInfo.objects.annotate(c=Lower('name'))
    # v = models.UserInfo.objects.annotate(c=Upper('name'))

    # 8. Now，获取当前时间
    # v = models.UserInfo.objects.annotate(c=Now())

    # 9. substr，子序列
    # v = models.UserInfo.objects.annotate(c=Substr('name',1,2))

    # ########### 时间类函数 ###########
    # 1. 时间截取，不保留其他：Extract, ExtractDay, ExtractHour, ExtractMinute, ExtractMonth,ExtractSecond, ExtractWeekDay, ExtractYear,
    # v = models.UserInfo.objects.annotate(c=functions.ExtractYear('ctime'))
    # v = models.UserInfo.objects.annotate(c=functions.ExtractMonth('ctime'))
    # v = models.UserInfo.objects.annotate(c=functions.ExtractDay('ctime'))
    #
    # v = models.UserInfo.objects.annotate(c=functions.Extract('ctime', 'year'))
    # v = models.UserInfo.objects.annotate(c=functions.Extract('ctime', 'month'))
    # v = models.UserInfo.objects.annotate(c=functions.Extract('ctime', 'year_month'))
    """
    MICROSECOND
    SECOND
    MINUTE
    HOUR
    DAY
    WEEK
    MONTH
    QUARTER
    YEAR
    SECOND_MICROSECOND
    MINUTE_MICROSECOND
    MINUTE_SECOND
    HOUR_MICROSECOND
    HOUR_SECOND
    HOUR_MINUTE
    DAY_MICROSECOND
    DAY_SECOND
    DAY_MINUTE
    DAY_HOUR
    YEAR_MONTH
    """

    # 2. 时间截图，保留其他：Trunc, TruncDate, TruncDay,TruncHour, TruncMinute, TruncMonth, TruncSecond, TruncYear
    # v = models.UserInfo.objects.annotate(c=functions.TruncHour('ctime'))
    # v = models.UserInfo.objects.annotate(c=functions.TruncDate('ctime'))
    # v = models.UserInfo.objects.annotate(c=functions.Trunc('ctime','year'))
```

## 6.4 ORM自定义函数

```python
from django.db.models.functions.base import Func
    class CustomeFunc(Func):
        function = 'DATE_FORMAT'
        template = '%(function)s(%(expressions)s,%(format)s)'

        def __init__(self, expression, **extra):
            expressions = [expression]
            super(CustomeFunc, self).__init__(*expressions, **extra)

    v = models.UserInfo.objects.annotate(c=CustomeFunc('ctime',format="'%%Y-%%m'"))