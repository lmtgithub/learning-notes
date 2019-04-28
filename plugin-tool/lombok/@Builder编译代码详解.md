# 结论
> 结论先行，阅读止步于此也可。
1. `@Builder`无法**直接**使用字段默认值
2. 需要在字段上使用`@Default`，才可以使用字段默认值
3. `@Builder`会自动生成一个**全参的构造函数**

# 证明结论1 & 结论3
## 前提
我们有一个学生类（为了方便，我们只使用1个字段。该字段带默认值）
```java
@Builder
public class Student {
  private int stuStatus = 1;
}
```
通过观察编译之后的代码，通过`Student$StudentBuilder.class`我们可以知道`@Builder`注解生成了一个`StudentBulder`的静态内部类。
![编译之后的代码](pic/../../../pic/@Builder.png)

将Student.class与Student$StudentBuilder.class反编译之后代码如下：
```java
public class Student {
   private int stuStatus = 1;
   // 注意此处：@Builder已经生成了一个全参构造函数。所以不能直接使用new Student()。如果想使用那么需要使用另一个注解：@NoArgsConstructor
   Student(int stuStatus) {
      this.stuStatus = stuStatus;
   }
   public static StudentBuilder builder() {
      return new StudentBuilder();
   }
}

public class Student$StudentBuilder {
   private int stuStatus;
   public Student$StudentBuilder stuStatus(int stuStatus) {
      this.stuStatus = stuStatus;
      return this;
   }
   public Student build() {
      return new Student(this.stuStatus);
   }
   public String toString() {
      return "Student.StudentBuilder(stuStatus=" + this.stuStatus + ")";
   }
}
```
此时我们执行如下代码：
```java
Student student = Student.builder().build();
```
此时`student`的`stuStatus`字段并不是你所期望的1，而是0。原因我们通过上述`Student$StudentBuilder`代码可以看出来：执行`build()`方法的时候使用的静态内部类的`stuStatus`，此时该字段为0。所以最终的`student`的`stuStatus`字段值为0。

**Q：如何才能使用stuStatus的默认值呢？**<br>
**A：** 为使用默认值的字段使用@Default注解(详细说明见下文)。

# 证明结论2
> 在`Builder.java`中可以发现，`@Builder`内部还有几个注解，此时我们需要使用的就是`@Default`注解
## 前提
我们有一个学生类（此时使用`@Default`注解）
```java
@Builder
public class Student {
  @Default
  private int stuStatus = 1;
}
```
将Student.class与Student$StudentBuilder.class反编译之后代码如下：
```java
public class Student {
   private int stuStatus;
   // 返回stuStatus的默认值1
   private static int $default$stuStatus() {
      return 1;
   }
   Student(int stuStatus) {
      this.stuStatus = stuStatus;
   }
   public static StudentBuilder builder() {
      return new StudentBuilder();
   }
   // $FF: synthetic method
   static int access$000() {
      return $default$stuStatus();
   }
}

public class Student$StudentBuilder {
   private boolean stuStatus$set;
   private int stuStatus;
   public Student$StudentBuilder stuStatus(int stuStatus) {
      this.stuStatus = stuStatus;
      this.stuStatus$set = true;
      return this;
   }
   public Student build() {
      int stuStatus = this.stuSstuStatustauts;
     // 会判断stuStatus是否被显式的set，如果没有被set，则使用默认值。
      if(!this.stuStatus$set) {
         stuStatus = Student.access$000();
      }
      return new Student(stuStatus);
   }
   public String toString() {
      return "Student.StudentBuilder(stuStatus=" + this.stuStatus + ")";
   }
}
```
此时我们执行如下代码：
```java
Student student = Student.builder().build();
```
此时`student`的`stuStatus`字段就是你所期望的1。原因：我们通过上述`Student$StudentBuilder`代码可以看出来：执行`build()`方法的时候会判断程序是否设置了`stuStatus`字段，如果没有设置(stuStatus$set是false)则调用`Student.access$000()`(该方法即是获取字段默认值)为stuStatus赋值。
# 写在后面
通过分析，明白了lombok为何不能直接使用字段默认值了。但是至于lombok为何这么做就不了解了。我给字段设置默认值的原因就是想不为该字段赋值的时候，字段能直接使用默认值，但是lombok却直接忽略了默认值，而是额外需要@Default注解。
