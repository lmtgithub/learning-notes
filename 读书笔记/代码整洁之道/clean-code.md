# 1. 引言
1. 代码质量与代码整洁度成正比
2. 宏大建筑中最细小的部分，比如关不紧的们、有点没铺平的底板、凌乱的书桌，都会将大局的魅力毁灭殆尽。
3. 勒布朗法则：稍后等于永不（Later equals never）。所以现在大家可以回头看看之前在代码中记录的TODO，是不是依旧存在。
4. 破窗理论：此理论认为环境中的不良现象如果被放任存在，会诱使人们仿效，甚至变本加厉。一幢有少许破窗的建筑为例，如果那些窗不被修理好，可能将会有破坏者破坏更多的窗户。最终他们甚至会闯入建筑内，如果发现无人居住，也许就在那里定居或者纵火。一面墙，如果出现一些涂鸦没有被清洗掉，很快的，墙上就布满了乱七八糟、不堪入目的东西；一条人行道有些许纸屑，不久后就会有更多垃圾，最终人们会视若理所当然地将垃圾顺手丢弃在地上。这个现象，就是犯罪心理学中的破窗效应。

# 2. 有意义的命名
## 2.1 名副其实
1. 变量名、函数名、类名的名称应该能很好的解释自身，而非使用额外的注释
```java
// bad case
int i = 12; //购买次数。此处只是举个例子而已，相信不会有人这样去命名的

// good case
int buyTimes = 12; // 购买次数
```
2. 尽量避免很突兀的直接使用数字（“魔数”），除非可以让别人能很快明白这个数字的含义
```java
// 场景：根据用户的状态不同，进行不同的操作

// bad case
switch (status) {
    case 0:
        System.out.println("冻结状态");
        break;
    case 1:
        System.out.println("正常状态");
        break;
    default:
        System.out.println("未知状态");
        break;
}

// good case
switch (status) {
    case FROZEN:
        System.out.println("冻结状态");
        break;
    case NORMAL:
        System.out.println("正常状态");
        break;
    default:
        System.out.println("未知状态");
        break;
}
```
## 2.2 避免误导
1. 避免使用与本意相悖的命名
```java
// 除非该变量的类型真的是一个List，否则最好不要使用XXXList的命名。因为List在程序员眼中是一个特殊的存在
Student[] studentList = new Student[](....); 
```
## 2.3 做有意义的区分
同一作用域内，对于2个相同的对象名称区分要有意义，不能简单的使用数字进行区分。
```java
// 场景：实例化2个Person对象。一个是男、一个女

// bad case
Person p1 = new Person("female");
Person p2 = new Person("male");

// good case
Person famale = new Person("female");
Person male = new Person("male");
```
## 2.4 使用读得出的名称
拿书中的例子来讲，只看函数名`genymdhms`谁可以明确该函数的作用？
```java
String date = genymdhms(new Date());

// 将日期转成yyyy-MM-dd的字符串格式
public String genymdhms(Date date) {
    Date currentTime = new Date();
    SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
    String dateString = formatter.format(currentTime);
    return dateString;
}
```
其实该函数确实比较难命名，个人感觉还不如定义成这样
```java
// DateFormat：日期格式的枚举值
public String dateToString(Date date, DateFormat dateFormat) {
    Date currentTime = new Date();
    SimpleDateFormat formatter = new SimpleDateFormat(dateFormat.getValue());
    String dateString = formatter.format(currentTime);
    return dateString;
}
```
