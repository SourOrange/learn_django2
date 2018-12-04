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

- 接着运行shell 去调试，毕竟要写什么数据在表中的话，最好在shell中完成，因为在 class 中继续写的话，会更加的难读，并且在shell中是可以随时增删查改的。所以运行 python manage.py shell, 接着会进入一个调试器，其实就是python的解释器，然后，我们要给数据库中添加一个 Teacher类的实例，就像是添加一个老师，甚至多个老师。当然一开始什么都没有，是空的。为了进入 models ,我们第一步，要导入， from teacher.models import Teacher, 然后，查看里面到底有没有什么数据线，使用命令 Teacher.objects.all(), 可以发现是一个空的，什么类型呢， type(Teacher.objects.all()) ，queryset,实际上算是一个列表。
... 现在我们来往里加一个老师进去先
<pre>
t1 = Teacher()
t1.name = "Eason"
t1.age = 50
t1.address = "香港九龙电视台3楼203号房"
t1.course = "中国好声音"
t1.save()
</pre>
.. 以上是我们在往数据库中 Teacher 表中 添加的一行数据，或者一个实例，如果你不清楚人家的地址，可以选择不填，但是最后一定要 save,否则不能算是添加到数据中。 然后 可以继续 t2 = Teacher(), 继续和上面的一样......当你填入了几个实例后，再打印一下 看看 Teacher.objects.all() 看看有几个数据，当然了你会看到好几个，不过，当你直接打印 print(t1) 的时候，你会发现 Teacher:object, 所以，更让人可读的是去修改 models.py 中 Teacher 类，添加一个魔法函数
----
<pre>
class Teacher(models.Model):
    .
    .
    .
    def __str__(self):
        return self.name
</pre>
这样么可读性更好，嗯，接着说说查找的一些技术吧。
Teacher.objects.filter(age=18), 查找出年龄是18岁的所有实例，如果要是 大于或者小于呢， （age>18 或 age<18）则会报错。
Teacher.objects.filter(age__gt=18) 这样才不会报错，具体的格式是 属性名 + 双下划线 + 字母 + = 值。
通用的查找格式
gt:大于 | gte:大于等于 | lt:小于 | lte:小于等于 | range:范围 | year:年份 | isnull:是否为空
-----------
 · 模糊查找：
 · exact: 精确等于 | iexact: 不区分大小写 | contains: 包含 | startwith:以..开头 | endwith:以..结尾
 
## models 的关联
- OneToOne (1 对 1)
-- 可以想象一下，一个学校有一个校长
- ManyToMany(多 对 多)
-- 一个学校内有多名老师，老师们可能教的不止一个班级，所以有不同的学生
- ManyToOne(多 对 一)
-- 一个企业，有很多业务，腾讯，做了很多游戏。

## OneToOne 的构建
- 首先，我们来创建一个 1对1 的例子，当然了，这是在 models中创建的两个类，首先创建两个类
<pre>
class School(models.Model):
    '''和Manager类 进行一比一的关系'''
    school_id = models.IntegerField()
    school_name = models.CharField(max_length=20)

    # 如果在 School 中创建 1对1 ，那就是这么写
    # my_manager = models.OneToOneField(Manager)

    def __str__(self):
        return self.school_name


class Manager(models.Model):
    manager_id = models.IntegerField()
    manager_name = models.CharField(max_length=20)

    # 可以在School中创建也可以在Manager中创建 1对1 的关系
    my_school = models.OneToOneField(School)

    def __str__(self):
        return self.manager_name
</pre>

- 接着是执行两个命令，python manage.py makemigrations , python manage.py migrate 
- 创建数据并保存到数据库中，还记得我们第一次创建 Teacher 中的实例吗，我们首先是 t1 = Teacher() 最后 t1.save(), 是啊，这是一种方法，当然了，我们还有另外一种方法，直接一条命令解决， t1 = Teacher.objects.create(name="..", age=..), 这条命令会自动保存，无需我们额外的写 save(), 所以，我们接下来是启动 python manage.py shell 后，导入 from teacher.models import School, Manager  ,记住，我们暂时不需要导入 Teacher, 好了，和以前一样，一开始我们尝试看看有什么, School.objects.all() 或者 Manager.objects.all(), 你会发现为空的，那么我们为了实验，创建2个学校2个校长，分别11对应。
- s1 = School.objects.create(school_id=22, school_name="BeiDaUniversity")
- s2 = School.objects.create(school_id=58, school_name="ShenZhenUniversity")
好了，再次 School.objects.all(),你会发现存了两个学校，当然你可以试试 s1.school_id ，s2.school.name 的访问，我们再来创建校长
- m1 = Manager.objects.create(manager_id=35, manager_name="Beida", my_school=s1) # 注意，最后一个属性 my_school, 值对应的是 s1,也就是北大学院，当然这是1对1的，所以你想好了，要对应哪个再填上去。
- m2 = Manager.objects.create(manager_id=66, manager_name="Shenzhen", my_school=s2)
## 建议，对于创建实例的方法，我们推荐 用  ..objects.create(...) 

继续啦，当你创建完上面的 s1 s2 m1 m2 之后，我们来查找一下，肯定啊，要查某个学校的校长是谁，或者是校长叫做什么的，他的学校叫什么，所以就有从子表查母表的，或者母表查子表，上面的两个类中， Manger 类是子表，因为他里面有一句对应School，所以就是子表，
Manager.objects.ger(manager_name="Shenzhen").my_school.school_name, 这是一句子表查母表的，意思是 校长叫做shenzhen的学校的名字是？，实际上，你不需要去记住这句命令，因为你又多种方法去实现查询，你会熟悉的，你还可以使用 m2.my_school.school_name 查到学校名。。。
而，母表查子表也是一样的，有一种方法和上面的一样，School.objects.get(manager__manager_name="Beida"), 这是母表查子表，里面的格式是:
母表类.objects.get(子表类名__子表属性名="") ,记住是双下划线。当然了，不需要去这么记住，直接用 s1.子类名.manager_name ,就可以查到 s1学校的校长是谁了。(s1.manager.manager_name) 是吧，不难，就是大概那些方法，试试就行了。

## 如何修改属性的值
你还记得把，学校的名字是 拼音组成的，s1.school_name = "BeiDaUniversity", 如果我们在后面要修改，那么可以直接 s1.school_name = "北京大学"，s1.save() 注意，一定要记得save,不然绝对不行，虽然不会报错，但是最终依旧是 拼音的，单独一个一个修改，可以这么做。不过，如果要修改多个的话，并且，修改为一样的名，可以使用 update() 方法，也就是 School.objects.update(school_name="北京大学")， 如果你这么做了，两个学校都是 北京大学了。所以看你情况而定吧。当然了，现在是两个学校，要修改很容易，所以你可以把两个都改为北京大学，然后再 运用第一个修改的方法，利用 s1.school_name s1.save() 的方法去修改第二个学校为原来的学校名

## delete
    直接删除吧。
## 一对多的关系 ForeignKey
是啊，就是 ForeignKey ，而不是什么OneToManyField, 当然你也可以直接把后者记住为 ForeignKey 就行了，我们这里举一个例子，学校里有多个场地，有篮球场
足球场，羽毛球场，这时候，我们定义一个场子先吧。再思考一下这个关系要加在哪里
<pre>
class Playground(models.Model):
    playground_name = models.CharField(max_length=20)
    # 因为是多个场地，所以在多的一边定义
    my_school = models.ForeignKey(School)
    def __str__(self):
        return self.playground_name
       
</pre>
是啊，就是在多的一方定义，思考一下，我们总是要查这个场地属于哪个学校的，哪个场地又是属于哪个学校的，所以就可以把 字段定义为 将来要方便查找的 my_school, 或者其他你可以关联到学校的，然后 用 ForeignKey(对应的一方)。当然了，这么做后，你记得去 迁移一下先，毕竟你添加了一个 Playground 表，记住用那两条命令。接着去 shell 中添加实例吧。p1 = Playground.objects.create(playground_name="篮球场", my_school=s1), 这样就创建完毕了，注意一下，s1 代表了 School 中的一个例子。并且在School 中会多一个默认的没有显示出来的字段名，playground_set, 也就是所有场地的集合，方便你通过学校去查询所有的场地。现在你可以通过场子查询到属于哪个学校了， p1.my_school 就可以查询到。反过来通过学校就是 s1.playground_set.all() 查找学校的所有场地。当然了，你要是找其中的名字叫做足球场的也行，s1.playground_set.all().filter(playground_name="足球场") 这样就行了。至于删除的，实际上，就是 delete,某课程中没有具体说明，所以这个大家私底下去学吧，加油！！！

## 多对多 ManyToManyField
这次就是 多对多的 关系了，多对多 写在哪边都可以。并且写一个就行，不需要两边都写，比如有 老师和学生 就是多对多的关系，老师有多个学生，学生有多个老师。
所以在某一边写就行了。我们在前面已经写过了 老师，现在我们写学生类
<pre>
class Student(models.Model):
    student_name = models.CharField(max_length=20)
    # 我们在学生这边定义就行了，因为是有多个老师，所以最好用复数来区分
    teacheres = models.ManyToManyField(Teacher)
    
    def __str__(self):
        return self.student_name
</pre>
默认的，在老师类那边，也有一个 student_set, 依然是看不到的，但是在操作中可以使用来查询，现在还是记得要先 迁移，两个命令。再去 shell 往学生类中添加学生。 s1 = Student.objects.create(student_name="Jordan"), 记住，不需要往里面添加 teacheres,这个属性，重点是要怎么对应上，我们要把 s1 对应上某个老师，比如 t1, 那么我们要这么做 ， s1.teacheres.add(t1), 这样就成功添加了，然后你可以查一下 s1.teacheres.all() 看看所有的老师，因为我们只添加了一个，所以会显示 t1 的信息，接下去所有查询都是差不多的 s1.teacheres.all()[0].name 。通过老师查学生呢，就是 t1.student_set.all()。 基本上数据表的关系就是这样了。其中这只是简要的让大家看看熟悉一下，具体的内容，还是建议大家一定要去看官网的内容，毕竟有些人不靠谱，学习需要自己不断去探索。


## 上面的数据表之间的关联之后，我们讲讲 模板
我记得在 learn_django 那个md中有讲过，至少是怎么配置模板的具体内容吧。这次主要是讲讲模板中的语法，实际上就是一些数据填空，但是有一些语法，不难，非常简单。不过说说配置先吧，好像有点忘记了。首先在你的项目中 settings.py 中，找到 template, 你的template应该是空的就像这样 template= ['DIR':[]].然后我们要这么做 'DIRS': [os.path.join(BASE_DIR, "templates")], 这里唯一有两点，BASE_DIR,是你的项目的具体目录地址，在该settings中，最上面就帮你写好了，我们不用改，直接写 BASE_DIR, 然后 “templates”, 是你要建立的文件夹的名字，后期中的模板都放在这里面，你也可以叫其他名字，但是一般，遵守约定，就叫templates, 好了。模板设置后，我们和前面的一样，不和数据库打交道，直接 是 url view template ，这三个步骤。
这三个步骤，可以随便，但是你熟悉的话为前提。不然就是 先 url 再 view 最后 template.  那么继续。 先在 urls.py 中 继续写。 urlpatterns = [url(r'^one', tv.one)], 然后再 views.py 中写 一个 one 函数。
<pre>
def one(request):
    return render(request, 'one.html')
</pre>
很明显，上面定义了一个 one.html, 所以我们在 templates 文件中建立一个 one.html. 简单，我们直接返回一个 h1 标题 hello my first tmp 就行。因为是测验。之后就是 debug run。在网页中输入  主机:8000/one 就可以看到内容了。
讲完了 第一个模板，我们讲讲第二个模板，这个模板带有一个变量 ，我们顶一个 url(r'^two', tv.two). 
<pre>
def two(request):
    d = dict()
    d['kuangjia'] = 'Django'
    return render(request, 'two.html', context=d)
</pre>
接着建一个 two.html,随便一一个 h1 标签中写一句 I'm learning {{ kuangjia }} . 接着 主机:8000/two. 可以看到 I'm learning Django. 这是变量中的用法，当然了， 该标签你可以插入css的各种属性。
第三个例子，是 for 标签，或者语法吧，也是一样，写一个 url(r'^three', tv.three), 
<pre>
def three(request):
    s = dict()
    s['score'] = [10, 23,44, 50, 33]
    return render(request, 'three.html', context=s)
</pre>
然后， three.html, 
<pre>
{% for i in score %}
    {{ i }}
{% endfor %}
</pre>
这样就可以看到成绩了，主机:8000/three 。
