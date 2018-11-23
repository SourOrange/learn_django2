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
