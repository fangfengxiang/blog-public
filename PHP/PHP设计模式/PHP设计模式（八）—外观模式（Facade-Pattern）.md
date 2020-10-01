**外观模式 （Facade Pattern）：** 为子系统中的一组接口提供一个一致的界面，定义一个高层接口，这个接口使得这一子系统更加容易使用。

**(一)为什么需要外观模式**

1，开发阶段，子系统越来越复杂，增加外观模式提供一个简单的调用接口。

2，维护一个大型遗留系统的时候，可能这个系统已经非常难以维护和扩展，但又包含非常重要的功能，为其开发一个外观类，以便新系统与其交互。

3，外观模式可以隐藏来自调用对象的复杂性。

**(二)外观模式UML图**

![Facade Pattern](http://upload-images.jianshu.io/upload_images/5261067-96df2879bc0e6d76.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


**(三)简单实例**

比如说我们去医院就诊，医院有医生员工系统，有药品系统，有患者资料系统。但是我们只是在前台挂个号，就能在其他系统里都看到我们。外观系统就差不多这样。

如果没有挂号系统的话，我们就先要去医生系统通知一下医生，
然后去患者系统取一下患者资料交给医生，再去药品系统登记一下，最后到药房领药。

```
<?php
//医院医生员工系统
class DoctorSystem{

    //通知就诊医生
    static public function getDoctor($name){
        echo __CLASS__.":".$name."医生，挂你号".PHP_EOL;
        return new Doctor($name);
    }
}
//医生类
class Doctor{
    public $name;
    public function __construct($name){
        $this->name = $name;
    }
    public function prescribe($data){
        echo __CLASS__.":"."开个处方给你".PHP_EOL;
        return "祖传秘方，药到必死";
    }
}
//患者系统
class SufferSystem {
    static function getData($suffer){
        $data = $suffer."资料";
        echo  __CLASS__.":".$suffer."的资料是这些".PHP_EOL ;
        return  $data;
    }
}
//医药系统
class MedicineSystem {
    static function register($prescribe){
        echo __CLASS__.":"."拿到处方：".$prescribe."------------通知药房发药了".PHP_EOL;
        Shop::setMedicine("砒霜5千克");
    }
}
//药房
class shop{
    static public $medicine;
    static function setMedicine($medicine){
        self::$medicine = $medicine;
    }
    static function getMedicine(){
        echo __CLASS__.":".self::$medicine.PHP_EOL;
    }
}

//如果没有挂号系统，我们就诊的第一步
//通知就诊医生
$doct = DoctorSystem::getDoctor("顾夕衣");
//患者系统拿病历资料
$data = SufferSystem::getData("何在");
//医生看病历资料，开处方
$prscirbe = $doct->prescribe($data);
//医药系统登记处方
MedicineSystem::register($prscirbe);
//药房拿药
Shop::getMedicine();

echo PHP_EOL.PHP_EOL."--------有了挂号系统以后--------".PHP_EOL.PHP_EOL;

//挂号系统
class Facade{
    static public function regist($suffer,$doct){
        $doct = DoctorSystem::getDoctor($doct);
        //患者系统拿病历资料
        $data = SufferSystem::getData($suffer);
        //医生看病历资料，开处方
        $prscirbe = $doct->prescribe($data);
        //医药系统登记处方
        MedicineSystem::register($prscirbe);
        //药房拿药
        Shop::getMedicine();
    }
}
//患者只需要挂一个号，其他的就让挂号系统去做吧。
Facade::regist("叶好龙","贾中一");
```

外观模式，也叫门面模式。它多用于在多个子系统之间，作为中间层。用户通过Facade对象，直接请求工作，省去了用户调用多个子系统的复杂动作。

外观模式常举的一个例子，就是我们买了好多支股票，但是时间有限。盯盘很复杂，我们搞得一团糟。所以，我们干脆买了股票基金。股票基金就好比于外观模式的Facade对象，而子系统就是股票基金投的各支股票。
