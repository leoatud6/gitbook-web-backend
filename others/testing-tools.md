# Testing Tools

## JMeter



## Postman





## JUnitTest





## Mockito [(ClickMe)](http://liangfei.me/2017/07/06/mockito-details-1-usage/)

### Mock  

* Class Mock (@mock)
* Partial Mock (@Spy)
* when thenReturn() --> result
* when thenThrow() --> exception

```java
+----------+      +------+      +--------+
| Mock/Spy | ===> | Stub | ===> | Verify |
+----------+      +------+      +--------+

//当classs 很难创建的时候，使用 mock(xxx.class):::
mockDao = mock(PersonDao.class);
when(mockDao.getPerson(1)).thenReturn(new Person(1, "Person1"));
when(mockDao.update(isA(Person.class))).thenReturn(true);
personService = new PersonService(mockDao);

//test method
boolean result = personService.update(1, "new name");
assertTrue("must true", result);
//验证是否执行过一次getPerson(1)
verify(mockDao, times(1)).getPerson(eq(1));
//验证是否执行过一次update
verify(mockDao, times(1)).update(isA(Person.class));

```



## TestNG

 类似JUnit, 有自己的配置文件，需要安装



| **@BeforeSuite**  | 注解的方法将只运行一次，运行所有测试前此套件中。                                                                                                              |
| ----------------- | ------------------------------------------------------------------------------------------------------------------------------------- |
| **@AfterSuite**   | 注解的方法将只运行一次此套件中的所有测试都运行之后。                                                                                                            |
| **@BeforeClass**  | 注解的方法将只运行一次先行先试在当前类中的方法调用。                                                                                                            |
| **@AfterClass**   | 注解的方法将只运行一次后已经运行在当前类中的所有测试方法。                                                                                                         |
| **@BeforeTest**   | 注解的方法将被运行之前的任何测试方法属于内部类的 \<test>标签的运行。                                                                                                |
| **@AfterTest**    | 注解的方法将被运行后，所有的测试方法，属于内部类的\<test>标签的运行。                                                                                                |
| **@BeforeGroups** | 组的列表，这种配置方法将之前运行。此方法是保证在运行属于任何这些组第一个测试方法，该方法被调用。                                                                                      |
| **@AfterGroups**  | 组的名单，这种配置方法后，将运行。此方法是保证运行后不久，最后的测试方法，该方法属于任何这些组被调用。                                                                                   |
| **@BeforeMethod** | 注解的方法将每个测试方法之前运行。                                                                                                                     |
| **@AfterMethod**  | 被注释的方法将被运行后，每个测试方法。                                                                                                                   |
| **@DataProvider** | 标志着一个方法，提供数据的一个测试方法。注解的方法必须返回一个Object\[] \[]，其中每个对象\[]的测试方法的参数列表中可以分配。该@Test 方法，希望从这个DataProvider的接收数据，需要使用一个dataProvider名称等于这个注解的名字。 |
| **@Factory**      | 作为一个工厂，返回TestNG的测试类的对象将被用于标记的方法。该方法必须返回Object\[]。                                                                                     |
| **@Listeners**    | 定义一个测试类的监听器。                                                                                                                          |
| **@Parameters**   | 介绍如何将参数传递给@Test方法。                                                                                                                    |
| **@Test**         | 标记一个类或方法作为测试的一部分。                                                                                                                     |







