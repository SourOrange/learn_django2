# learn_django2
object relation mapping
## 在Django中，我们把 ORM 称为 模型，即 models ，并且这一章，我们主要学习数据库和类的对应，你可以想象，类对应的就是数据表的名称，而类的属性大概就是数据表中的字段 列名，django 自带 数据库 sqlite3,关系型数据库，轻量级别， models 负责跟 数据库交互，并且建议，我们在开发中就默认使用该数据库，而在部署的时候，可以用 mysql 之类的数据库，不难，直接4 5 行代码而已, 在 settings 中，原本的是下面这个:
<pre>
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
</pre>
<br>
现在，当你在部署的时候，请修改为:
<pre>
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.mysql',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'), # 这里写你的数据库名
        'PASSWORD': '数据库密码',
        'HOST':127.0.0.1,
        'PORT':'3306',
    }
</pre>
另外，可能吧，你还需要在 项目主文件夹下 的 __init__.py 文件中第一行导入 import pymysql ，第二行写，pymysql.install_as_MySQLdb() ,如果是其他数据库，也是一样的。
现在，我们在 teacher 文件夹下的 models.py 中(通常每一个应用下都有一个models)，编写一个类，这个类对应的的就是一个数据库，你可以想象，数据库就是一个 excel 表格，每一个列的列名就是类的属性，
<pre>
from django.db import models
# Create your models here
class Teacher(models.Model):
    # 每个类都继承 models.Model
    name = models.CharField(max_length=12)
    age = models.IntegerField()
    address = models.CharField(max_length=50)
    course = models.CharField(max_length=20)
</pre>
嗯，就是这样先，以上的 name, age, address, course, 你想象为一个 excel 表中的，4个列名，另外 name, address, course, 是一个字符串，并且需要给限定多少字符，你想想，要是不做限制，那么万一别人写一本书的内容进去，你的程序会被挤爆的，也是不行的吧，所以要做限制。而 age 就不用。写到这里，我们还要想一下，既然和 数据库 打交道，那我们怎么让数据库知道，所以接下来，执行命令， <strong>python manage.py makemigrations</strong> ,会显示Create Model 'Teacher',并且生成一个 0001_initial.py 的文件夹，大概是这样, 这一步是告诉 数据库，请做好迁移的准备事项，我们要搬一些货进去了啊。第二步，执行命令 <strong>python manage.py migrate</strong> , 这一步就是真正的把数据填进去了，没出错的话会显示很多内容在命令行。

----
Teacher.objects.filter(age=18), 查找出年龄是18岁的所有实例，如果要是 大于或者小于呢， （age>18 或 age<18）则会报错。
Teacher.objects.filter(age__gt=18) 这样才不会报错，具体的格式是 属性名 + 双下划线 + 字母 + = 值。
通用的查找格式
gt:大于 | gte:大于等于 | lt:小于 | lte:小于等于 | range:范围 | year:年份 | isnull:是否为空
-----------
 · 模糊查找：
 · exact: 精确等于 | iexact: 不区分大小写 | contains: 包含 | startwith:以..开头 | endwith:以..结尾
 
