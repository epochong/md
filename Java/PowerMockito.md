# PowerMockito

### 什么是MOCK？

 模拟真实对象行为的一个假的对象，该对象中的数据可以按照自己的期望赋予。

### 为什么要MOCK？

1.在单体测试过程中，有的对象很难构造或者获取;

2.调用的别人的逻辑还没有实现。

## PowerMock

#### PowerMock简单实现原理

- 当某个测试方法被注解@PrepareForTest标注以后，在运行测试用例时，会创建一个新的org.powermock.core.classloader.MockClassLoader实例，然后加载该测试用例使用到的类（系统类除外）。
- PowerMock会根据你的mock要求，去修改写在注解@PrepareForTest里的class文件（当前测试类会自动加入注解中），以满足特殊的mock需求。例如：去除final方法的final标识，在静态方法的最前面加入自己的虚拟实现等。
- 如果需要mock的是系统类的final方法和静态方法，PowerMock不会直接修改系统类的class文件，而是修改调用系统类的class文件，以满足mock需求。

#### PowerMock的常用注解

##### @PrepareForTest({YourClass.class})/@RunWith(PowerMockRunner.class)

> 当使用PowerMockito.whenNew方法时，必须加注解@PrepareForTest和@RunWith。注解@PrepareForTest里写的类是需要mock的new对象代码所在的类。

- 当需要mock final方法的时候，必须加注解@PrepareForTest和@RunWith。注解@PrepareForTest里写的类是final方法所在的类。 

- 当需要mock静态方法的时候，必须加注解@PrepareForTest和@RunWith。注解@PrepareForTest里写的类是静态方法所在的类。

- 当需要mock私有方法的时候, 只是需要加注解@PrepareForTest，注解里写的类是私有方法所在的类

- 当需要mock系统类的静态方法的时候，必须加注解@PrepareForTest和@RunWith。注解里写的类是需要调用系统方法所在的类

##### @InjectMocks：

mock将要注入的实例

创建一个实例，用@Mock或@Spy注解创建的mock将被注入到该实例中。

使用@Mock生成的类，所有方法都不是真实的方法，而且返回值都是NULL

##### @Mock：

创建一个mock，该对象所有的方法被置空，即都不是真实的方法。

##### @Spy：

创建一个mock对象，@Spy修饰的对象必须先手动new出来，但是，该对象所有方法都是真实方法，返回值都是和真实方法一样的。

##### @Before

初始化方法  对于每一个测试方法都要执行一次（注意与BeforeClass区别，后者是对于所有方法执行一次）

##### @After

释放资源  对于每一个测试方法都要执行一次（注意与AfterClass区别，后者是对于所有方法执行一次）

##### @Test

测试方法，在这里可以测试期望异常和超时时间 
@Test(expected=ArithmeticException.class)检查被测方法是否抛出ArithmeticException异常 

##### @Ignore

忽略的测试方法 

##### @BeforeClass

对所有测试，只执行一次，且必须为static void 

##### @AfterClass

针对所有测试，只执行一次，且必须为static void 
**一个JUnit4的单元测试用例执行顺序为：** 
@BeforeClass -> @Before -> @Test -> @After -> @AfterClass; 
**每一个测试方法的调用顺序为：** 

@Before -> @Test -> @After; 

## 常用方法

### PowerMockito.doReturn().when()

当使用PowerMockito.doReturn(null).when(handler, "getFareRules", Integer.valueOf(requestDTO.getFareId()), "GB");时
handler的getFareRules方法不会被真的调用，在getFareRules里面打一些日志，这些日志不会输出，也就是说根本没有真的去调用该方法，而是直接
调用了代理方法，返回在doReturn设置的值。

### Mockito.when().thenReturn()

**对于希望调用后什么都不做的，或者抛出异常的，可以使用`PowerMockito.doNothing().when(...)...`、`doThrow(Throwable).when(...)...`的语法**

## 使用Mock对象

mock 构造方法、final方法、private 方法、static方法 需要在测试类添加@PrepareForTest()注解，其中mock构造方法，@PrepareForTest中的类是所在类，其他的都是方法所属类。

### 普通Mock

```java
PowerMockito.when(orderRechargeService.updateByRequestNo(Matchers.any())).thenReturn(true);
/**
* mock 无返回值的方法
*/
PowerMockito.doNothing().when(mockService).methodWithVoid();
/**
* 当使用mock对象去调用某个方法时，要使用thenCallRealMethod();
*/   
PowerMockito.when(mock,"methodName").thenCallRealMethod();
```

### Mock 构造方法

```java
File file = PowerMockito.mock(File.class);   
PowerMockito.whenNew(File.class).withArguments("bing").thenReturn(file);   
```

### mock final 方法

```java
ClassWithFinalMethod mock= PowerMockito.mock(ClassWithFinalMethod.class); 
PowerMockito.when(mock.finalMethod()).thenReturn(someThing);  
```

### mock private 方法

```java
PowerMockito.when(service,"privateMethod").thenReturn(someThing);
```

### mock 静态方法

```java
PowerMockito.mockStatic(ClassWithStaticMethod.class);
PowerMockito.when(ClassWithStaticMethod.staticMethod()).thenReturn(someThing);  
```

### 设置field

```java
Whitebox.setInternalState(service, "anyfield", value);  
Whitebox.getInternalState(service,"field");
```

### answer

> 根据不同的输入返回不同的结果

```java
PowerMockito.when(service.method()).then(new Answer<methodReturnObject>(){
	public methodReturnObject answer(InvocationOnMock invocation){
		invocation.callRealMethod()：调用真正的方法
		invocation.getArguments()：获取所有参数
		invocation.getMethod()：返回mock实例调用的方法
		invocation.getMock()：获取mock实例
			
		if ... return null;
	}
});
```

### suppress 禁用

> PowerMockito.suppress方法用来禁用某个域或方法.

PowerMockito.suppress(PowerMockito.constructor(BaseEntity.class)):表示禁用BaseEntity的构造函数
PowerMockito.suppress(PowerMockito.constructor(BaseEntity.class, String.class, Integer.class)):表示禁用参数为String和Integer类型的BaseEntity构造方法。
PowerMockito.suppress(PowerMockito.method(BaseEntity.class, "performAudit", String.class)):表示禁用BaseEntity的performAudit方法。
@SuppressStaticInitializationFor("BaseEntity"):表示禁用BaseEntity的静态初始化。注意引号部分通常需要全名，比如"com.gitshah.powermock.BaseEntity"。
PowerMockito.suppress(PowerMockito.field(BaseEntity.class,"identifier"))：禁用identifier域。

#### assert or verify 

- 验证方法是否调用

```java
Mockito.verify(mockService).mockedMethod(parameters);
```

- 验证方法是否从未调用

```java
Mockito.verify(mockService,never()).mockedMethod(parameters);
```

- \- 验证方法调用次数

```java
Mockito.verify(mockService,times(1)).mockedMethod(parameters);
```

- -验证是否调用过private方法

```java
PowerMockito.verifyNew();
PowerMockito.verifyPrivate();
PowerMockito.verifyStatic();
```

