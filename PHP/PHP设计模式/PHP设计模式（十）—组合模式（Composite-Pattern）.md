**组合模式 （Composite Pattern）：**将对象组合成树形结构以表示“部分整体”的层次结构。组合模式使得用户对单个对象和组合对象的使用具有一致性。组合模式也叫合成模式，有时候又叫做部分-整体模式。

**(一)为什么需要组合模式**

1，使我们在树型结构的问题中，模糊了简单元素和复杂元素的概念，客户程序可以像处理简单元素一样来处理复杂元素,从而使得客户程序与复杂元素的内部结构解耦。

2，组合模式让你可以优化处理递归或分级数据结构。

**(二)组合模式UML图**

![Composite Pattern](http://upload-images.jianshu.io/upload_images/5261067-27adc46fb553faa5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


 - Component是组合中的对象声明接口，在适当的情况下，实现所有类共有接口的默认行为。声明一个接口用于访问和管理Component子部件。
 - Leaf 在组合中表示叶子结点对象，叶子结点没有子结点。
   
 - Composite定义有枝节点行为，用来存储子部件，在Component接口中实现与子部件有关操作，如增加(add)和删除(remove)等。

**(三)简单实例**

如果我们在做一个OA系统，公司的人事管理该如何设计呢。传统的就是树状结构。经理下面有部门主管，然后是员工。

![人事部门图](http://upload-images.jianshu.io/upload_images/5261067-8f104207d2f2c087.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


```
<?php
class Manager{
    public $name;
    protected $c_nodes = array();//存放子节点，部门经理，普通员工等
    public function __construct($name){
        $this->name = $name;
    }
    //添加部门经理
    public function addGm(GM $gm){
        $this->c_nodes[] = $gm;
    }
    //添加普通员工
    public function addStaff(Staff $staff){
        $this->c_nodes[] = $staff;
    }
    //获取全部子节点
    public function get_C_nodes(){
        return $this->c_nodes;
    }
}

//部门经理 就用general manager 简写 GM
Interface Gm{
    public function add(Staff $staff);
    public function get_c_nodes();
}
//销售经理
class Sgm implements Gm{
    public $name;
    protected $c_nodes = array();
    public function __construct($name){
        $this->name = $name;
    }
    //添加员工
    public function add(Staff $staff){
        $this->c_nodes = $staff;
    }
    //获取子节点
    public function get_C_nodes(){
        return $this->c_nodes;
    }
    //区别于其他经理，销售经理有一个销售方法
    public function sell(){
        echo "安利一下我司的产品";
    }
}
//员工接口
Interface staff{
    public function work();
}
//销售部员工
class Sstaff implements staff{
    public $name;
    public function work(){
        echo '在销售经理带领下，安利全世界';
    }
}
//实例化
$manager = new Manager("总经理");
$sgm = new Sgm("销售经理");
$staff = new Sstaff("何在");
//组装成树
$manager->addGm($sgm);
$sgm->add($staff);
```

我们想象一下，如果我们的层级非常深，如销售经理下面还有，销售主管，分区经理，分区主管。那怎么办？我们new的时候，就要new很多不同的类。而且如果要加一个任职期限的属性，还得每个类去添加一遍。

我们想做的是，可以把树形结构，当成“部分-整体结构”来处理，通俗地讲，就是把一个树形结构当成一个关系型的结构。例如：数据库存储的一行行的方式。找了张图，清楚些。

![省市存储图](http://upload-images.jianshu.io/upload_images/5261067-77dde427118513f7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

上面这张图把中国 -湖南-（长沙）（衡阳）-这个树状图以关系型的层级方式存储于数据库中。对于公司人事，其实我们也是可以用这种方法的。那就是利用组合模式。再去看一眼UML图把和人事管理图吧。我们发现总经理和部门经理还是有很多相同的地方。组合模式主要就是把根节点和所有树枝节点归结到一起去，这样就隐藏了树形的层级。

```
<?php
//抽象构件
Abstract class Component{
    public $name;
    abstract function doSomething();
    public function __construct($name){
        $this->name = $name;
    }
}
//普通员工  树叶构件 不能添加子节点
class Leaf extends Component{
    public $lever;
    public function doSomething(){
        echo "层级--{$this->lever}--work";
    }
}
//总经理 部门经理 主管等 树枝构件
class Composite extends Component{
    public $c_nodes = array();
    public $lever = 1;
    //添加子节点
    public function add(Component $component){
        $component->lever = $this->lever + 1;
        $this->c_nodes[] = $component;
    }
    public function doSomething(){
        echo "我是层级--{$this->lever}--".PHP_EOL;
    }
}
$manager = new Composite("总经理");
$sgm = new Composite("销售经理");
$staff = new  Leaf("何在");
//组装成树
$manager->add($sgm);
$sgm->add($staff);
```

这样，我们就把根节点（总经理）和所有树枝节点（部门经理，主管）的树枝结构隐藏了，通过$lever属性来区分。如果对于不同树枝节点有不同的方法，我们也可以在Composite 类中的doSomething()方法中延迟绑定具体的方法实现，使不同层级具有不同能力

```
   public function doSomething(){
        switch($this->lever){
            case 1:$this->manager();
                break;
            case 2:$this->sell();
        }
    }
    private function manager(){}
    private function sell(){}
```

当然，当层级太深时，若有多个层级有相同的doSomething能力。这种方法还是可以的。但是当层级太深且不同层级具有不同的doSomething能力时，就会导致一个类中空置了多个不用的private方法，而doSomething只调用一个。

$lever 的另一个功能就是便于递归遍历出所有公司人员

```
//遍历树 - 函数
function display(Composite $composite){
    $composite->doSomething();
    foreach($composite->c_nodes as $c_node)
         $c_node instanceof Leaf ? $c_node->doSomething() : display($c_node);                   
}
display($manager);
```

当然这种遍历方法只能前序遍历，即从根节点总经理向下找，没法从任何一个员工向上找出他的上级。如果，你想实现后序遍历，可以在Component类中添加一个parent属性，并在composite 的add方法中设置子节点的parent属性。

组合模式也分为透明模式和安全模式，上面的例子是安全模式。透明模式是把composite的方法也放到抽象类component中。

> 有许多关于分级数据结构的例子，使得组合模式非常有用武之地。关于分级数据结构的一个普遍性的例子是你每次使用电脑时所遇到的:文件系统。文件系统由目录和文件组成。每个目录都可以装内容。目录的内容可以是文件，也可以是目录。按照这种方式，计算机的文件系统就是以递归结构来组织的。如果你想要描述这样的数据结构，那么你可以使用组合模式Composite。
