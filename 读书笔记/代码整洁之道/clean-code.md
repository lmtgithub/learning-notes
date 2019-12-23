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
## 2.5 类名
类名和对象名应该是名词或名词短语，例如：Customer、Member等，不能使用动词。
## 2.6 方法名
方法名应该是动词或者动词短语，应该是个动宾结构。例如：deleteRecord、saveActivity等
## 2.7 每个概念对应一个词
相同的概念使用相同的词语，例如：获取单条对象使用getXXX()；获取多个对象使用listXXX()；避免查询对象时混用getXXX(),fetchXXX(),obtainXXX()。

给每个抽象概念选定一个词，并一以贯之，保持一致，就可以避免花费多余的浏览时间才能找到正确的方法。
```java
// 根据学号获取学生对象
public Student getStudent(String strNumber){}

// 根据班级获取班级所有学生
public List<Student> listStudent(){}
```
## 2.8 别用双关语
避免将同一单词用到不同的概念。同一术语用于不同概念基本就属于双关语了。
```java
// 加法函数
public int add(int a, int b){}

// bad case。将目标值加入到list集合中,此处使用add就会和上面的函数混淆。
public List<Integer> add(List<Integer> container, int target){};

// good case。使用append或者insert都可以
public List<Integer> append(List<Integer> container, int target){}
```
# 3. 函数
## 3.1 短小
函数缩进层级不要多余两层，这样更易阅读和理解。
## 3.2 只做一件事
函数应该只做一件事，做好这件事，只做这一件事。

正如下面代码：

优化前：需要阅读人自己区分canGetCoupon中做了哪几件事，最终通过阅读代码和注释才明白做了2件事：验证活动和验证用户信息。

优化后：用函数分别处理验证活动和验证用户信息，人们可以很清楚的知道canGetCoupon一共做了2件事。
```java
// 优化前代码
public boolean canGetCoupon(int userId, int activityId) {
    // 第一步：校验活动是否有效
    Activity activity = activityDao.queryById(activityId);
    if(activity == null || activity.getStatus() != ActivityStatus.EFFECTIVE.value()) {
        return false;
    }

    // 第二步：校验用户信息是否完整
    UserInfo user = userDao.queryById(userId);
    if (StringUtils.isBlank(user.getName())){
        log.warn("未填写姓名");
        return false;
    }
    if (StringUtils.isBlank(user.getPhone())) {
        log.warn("未填写手机号");
        return false;
    }
    return true;
}

// 优化后代码
public boolean canGetCoupon(int userId, int activityId) {
    return valideActivity(activityId) && valideUserInfo(userId);
}

private boolean valideActivity(int activityId) {
    Activity activity = activityDao.queryById(activityId);
    if(activity == null || activity.getStatus() != ActivityStatus.EFFECTIVE.value()) {
        return false;
    }
    return true;
}

private boolean valideUserInfo(int userId) {
    UserInfo user = userDao.queryById(userId);
    if (StringUtils.isBlank(user.getName())){
        log.warn("未填写姓名");
        return false;
    }
    if (StringUtils.isBlank(user.getPhone())) {
        log.warn("未填写手机号");
        return false;
    }
    return true;
}
```
## 3.3 使用描述性的名字
函数越小，功能越集中，就越好起名字。并且命名方式应该保持一致。

### 3.4 函数参数
函数的参数不要超过3个，超过3个之后需要封装成类。

### 3.5 无副作用
通过下列校验用户名和密码的代码，来介绍副作用的危害。
```java
public boolean checkPassword(String username, String password) {
    User user = userDao.getUserByName(username);
    if (user == null) {
        return false;
    }
    String expectedPwd = user.getPwd();
    if (password.equals(expectedPwd)) {
        Session.init();
        return true;
    }
    return false;
}
```
此处的副作用就是在`Session.init()`上，原本是校验密码的函数，结果写写入了Session。对于不懂内部实现的人来说很容易用出问题，会将原先的Session抹掉。如果函数命名成`checkPasswordAndInitSession(String username, String password)`会更好一些，但是却违背了“只做一件事儿”的原则，所以将这种操作分成2步比较好。第一步:`checkPassword()`;第二步:`Session.init()`

## 3.6 抽离try-catch代码块
try-catch代码块搞乱了代码接口，将错误处理与正常流程混为一谈，最好将try{}和catch{}的代码主体部分抽离出来，另外形成函数。
```java
public void insertUser(User user) {
    try {
        insert(user);
    } catch (Exception ex) {
        dealWithError(ex);
    }
}
private void insert(User user) {
    if (user == null) {
        throw new Exception("插入对象为空");
    }
    userDao.insert(user);
}

private void dealWithError(Exception ex) {
    log.error("插入失败", ex);
    // do something else
}
```
## 3.7 避免重复
代码重复不仅使得代码更加冗长，而且后期维护也较为困难。面向对象编程、面向切面编程都可以减少一些重复代码。

## 3.8 如何写好函数
上面介绍了这么多函数的规则，但是如何写好函数呢？从一开始就遵循规则写代码函数，应该很少人会做到。做不到的话，就第一遍代码只要实现工能够即可，然后慢慢打磨代码，分解函数、修改名称、消除重复代码等操作。
# 4. 注释
尽可能不添加注释，使用代码自解释。即通过代码便可了解这段代码的功能。除非不得已使用注释，也要使注释变得简单。
```java
// 新增一名学生(此处的注释就显得额外多余)
public void insertStudent(Student student) {
    // do something
}
```
