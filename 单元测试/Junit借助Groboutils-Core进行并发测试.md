众所周知，junit是不支持并发测试的。那当使用junit并且有并发测试需求的时候怎么办呢？此时可以借助一个插件（Groboutils Core）来完成这种功能。

maven仓库地址：[Groboutils Core](https://mvnrepository.com/artifact/net.sourceforge.groboutils/groboutils-core)

### 第一步：引入依赖
```xml
<dependency>
  <groupId>net.sourceforge.groboutils</groupId>
  <artifactId>groboutils-core</artifactId>
  <version>5</version>
  <scope>test</scope>
</dependency>
```
### 第二步：进行单测
```java
@Test
public void testConcurrentInitOrBind() {

    // mock一个返回
    doReturn(Lists.newArrayList(userMemberCard)).when(operateCardDao).queryCardByRegisterMobileAndTenantId(anyString(), anyLong());

    TestRunnable runner = new TestRunnable() {
        // 在runTest方法中填写自己的测试方法
        @Override
        public void runTest() throws Throwable {
        InitCardResVo resVoFirst = operateCardService.initOrBindCard(requestVo);
        System.out.println("result resVoFirst is:" + resVoFirst.toString());
        }
    };

    // 一个数组，代表并发个数。此处并发5个
    TestRunnable[] trs = new TestRunnable[5];
    for (int i = 0; i < 5; i++) {
        trs[i] = runner;
    }
    MultiThreadedTestRunner mttr = new MultiThreadedTestRunner(trs);
    try {
        mttr.runTestRunnables();
    } catch (Throwable ex) {
        ex.printStackTrace();
    }
}
```

