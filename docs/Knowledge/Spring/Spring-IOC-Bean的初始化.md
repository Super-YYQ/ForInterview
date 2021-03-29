## 一、循环依赖

[参考链接](https://cloud.tencent.com/developer/article/1497692)

### 1.1 问题出现场景

```java
public class AA {
    public AA() {
        new BB();
    }
}
public class BB {
    public BB() {
        new AA();
    }
}

// 测试循环依赖
new AA();

// 报错
Exception in thread "main" java.lang.StackOverflowError
	at com.spring.ioc.bean.循环依赖.BB.<init>(BB.java:10)
	at com.spring.ioc.bean.循环依赖.AA.<init>(AA.java:11)
    ......
```

### 1.2 Spring中三大循环依赖

> 对于Spring循环依赖的情况总结如下：
>
> 1. 不能解决的情况： 1. 构造器注入循环依赖 2. `prototype` field属性注入循环依赖
> 2. 能解决的情况： 1. field属性注入（setter方法注入）循环依赖

#### 1.2.1 构造器注入循环依赖

```java
@Service
public class A {
    public A(B b) {
    }
}
@Service
public class B {
    public B(A a) {
    }
}

// 启动报错 BeanCurrentlyInCreationException
***************************
APPLICATION FAILED TO START
***************************

Description:

The dependencies of some of the beans in the application context form a cycle:

┌─────┐
|  AA defined in file [E:\我的github代码仓库\GitHub\mySpringBoot\target\classes\com\spring\ioc\bean\循环依赖\AA.class]
↑     ↓
|  BB defined in file [E:\我的github代码仓库\GitHub\mySpringBoot\target\classes\com\spring\ioc\bean\循环依赖\BB.class]
└─────┘
```

>  构造器注入构成的循环依赖，此种循环依赖方式**是无法解决的**，只能抛出`BeanCurrentlyInCreationException`异常表示循环依赖。这也是构造器注入的最大劣势。
>
>  `根本原因`：Spring解决循环依赖依靠的是Bean的“中间态”这个概念，而这个中间态指的是`已经实例化`，但还没初始化的状态。而构造器是完成实例化的东东，所以构造器的循环依赖无法解决~~~

#### 1.2.2 field属性注入（setter）循环依赖

```java
@Service
public class A {
    @Autowired
    private B b;
}
@Service
public class B {
    @Autowired
    private A a;
}

// 启动成功
```

#### 1.2.3 `prototype` field属性注入循环依赖

```java
@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
@Service
public class A {
    @Autowired
    private B b;
}

@Scope(ConfigurableBeanFactory.SCOPE_PROTOTYPE)
@Service
public class B {
    @Autowired
    private A a;
}

// 启动没问题

// 获取bean报错
@Service
public class CC {
    @Autowired
    ApplicationContext applicationContext;
    public CC(ApplicationContext applicationContext) {
        this.applicationContext = applicationContext;
        applicationContext.getBean("AA");
    }
}
```

> 结果：**需要注意的是**本例中**启动时是不会报错的**（因为非单例Bean`默认`不会初始化，而是使用时才会初始化），所以很简单咱们只需要手动`getBean()`或者在一个单例Bean内`@Autowired`一下它即可

### 1.3 Spring解决循环依赖原理

> **`Spring的循环依赖的理论依据基于Java的引用传递`**，当获得对象的引用时，**对象的属性是可以延后设置的**。（但是构造器必须是在获取引用之前，毕竟你的引用是靠构造器给你生成的，儿子能先于爹出生？哈哈）

#### 1.3.1 Spring创建Bean的流程

![image-20210218100026188](https://raw.githubusercontent.com/Super-YYQ/PicGoPicture/main/PicGo/image-20210218100026188.png)

 对Bean的创建最为核心三个方法解释如下：

- `createBeanInstance`：例化，其实也就是调用对象的**构造方法**实例化对象
- `populateBean`：填充属性，这一步主要是对bean的依赖属性进行注入(`@Autowired`)
- `initializeBean`：回到一些形如`initMethod`、`InitializingBean`等方法

从对`单例Bean`的初始化可以看出，循环依赖主要发生在**第二步（populateBean）**，也就是field属性注入的处理。

#### 1.3.2 Spring容器的'三级缓存'

在Spring容器的整个声明周期中，单例Bean有且仅有一个对象。这很容易让人想到可以用缓存来加速访问。 从源码中也可以看出Spring大量运用了Cache的手段，在循环依赖问题的解决过程中甚至不惜使用了“三级缓存”，这也便是它设计的精妙之处~

`三级缓存`其实它更像是Spring容器工厂的内的`术语`，采用三级缓存模式来解决循环依赖问题，这三级缓存分别指：

```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {
	...
	// 从上至下 分表代表这“三级缓存”
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256); //一级缓存
	private final Map<String, Object> earlySingletonObjects = new HashMap<>(16); // 二级缓存
	private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16); // 三级缓存
	...
	
	/** Names of beans that are currently in creation. */
	// 这个缓存也十分重要：它表示bean创建过程中都会在里面呆着~
	// 它在Bean开始创建时放值，创建完成时会将其移出~
	private final Set<String> singletonsCurrentlyInCreation = Collections.newSetFromMap(new ConcurrentHashMap<>(16));

	/** Names of beans that have already been created at least once. */
	// 当这个Bean被创建完成后，会标记为这个 注意：这里是set集合 不会重复
	// 至少被创建了一次的  都会放进这里~~~~
	private final Set<String> alreadyCreated = Collections.newSetFromMap(new ConcurrentHashMap<>(256));
}
```

注：`AbstractBeanFactory`继承自`DefaultSingletonBeanRegistry`~

1. `singletonObjects`：用于存放完全初始化好的 bean，**从该缓存中取出的 bean 可以直接使用**
2. `earlySingletonObjects`：提前曝光的单例对象的cache，存放原始的 bean 对象（尚未填充属性），用于解决循环依赖
3. `singletonFactories`：单例对象工厂的cache，存放 bean 工厂对象，用于解决循环依赖

**获取单例Bean的源码如下：**

```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {
	...
	@Override
	@Nullable
	public Object getSingleton(String beanName) {
		return getSingleton(beanName, true);
	}
	/**
	 * Return the (raw) singleton object registered under the given name.
	 * <p>Checks already instantiated singletons and also allows for an early
	 * reference to a currently created singleton (resolving a circular reference).
	 * @param beanName the name of the bean to look for
	 * @param allowEarlyReference whether early references should be created or not
	 * @return the registered singleton object, or {@code null} if none found
	 */
	@Nullable
	protected Object getSingleton(String beanName, boolean allowEarlyReference) {
		Object singletonObject = this.singletonObjects.get(beanName);
		if (singletonObject == null && isSingletonCurrentlyInCreation(beanName)) {
			synchronized (this.singletonObjects) {
				singletonObject = this.earlySingletonObjects.get(beanName);
				if (singletonObject == null && allowEarlyReference) {
					ObjectFactory<?> singletonFactory = this.singletonFactories.get(beanName);
					if (singletonFactory != null) {
						singletonObject = singletonFactory.getObject();
						this.earlySingletonObjects.put(beanName, singletonObject);
						this.singletonFactories.remove(beanName);
					}
				}
			}
		}
		return singletonObject;
	}
	...
	public boolean isSingletonCurrentlyInCreation(String beanName) {
		return this.singletonsCurrentlyInCreation.contains(beanName);
	}
	protected boolean isActuallyInCreation(String beanName) {
		return isSingletonCurrentlyInCreation(beanName);
	}
	...
}
```

1. 先从`一级缓存singletonObjects`中去获取。（如果获取到就直接return）
2. 如果获取不到或者对象正在创建中（`isSingletonCurrentlyInCreation()`），那就再从`二级缓存earlySingletonObjects`中获取。（如果获取到就直接return）
3. 如果还是获取不到，且允许singletonFactories（allowEarlyReference=true）通过`getObject()`获取。就从`三级缓存singletonFactory`.getObject()获取。**（如果获取到了就从**`**singletonFactories**`**中移除，并且放进**`**earlySingletonObjects**`**。其实也就是从三级缓存**`**移动（是剪切、不是复制哦~）**`**到了二级缓存）**

>  **加入`singletonFactories`三级缓存的前提是执行了构造器，所以构造器的循环依赖没法解决**

`getSingleton()`从缓存里获取单例对象步骤分析可知，Spring解决循环依赖的诀窍：**就在于singletonFactories这个三级缓存**。这个Cache里面都是`ObjectFactory`，它是解决问题的关键。

#### 1.3.3 源码解析

`Spring`容器会将每一个正在创建的Bean 标识符放在一个“当前创建Bean池”中，Bean标识符在创建过程中将一直保持在这个池中，而对于创建完毕的Bean将从`当前创建Bean池`中清除掉。 这个“当前创建Bean池”指的是上面提到的`singletonsCurrentlyInCreation`那个集合。





### 1.4 总结

 依旧以上面`A`、`B`类使用属性`field`注入循环依赖的例子为例，对整个流程做文字步骤总结如下：

1. 使用`context.getBean(A.class)`，旨在获取容器内的单例A(若A不存在，就会走A这个Bean的创建流程)，显然初次获取A是不存在的，因此走**A的创建之路~**
2. `实例化`A（注意此处仅仅是实例化），并将它放进`缓存`（此时A已经实例化完成，已经可以被引用了）
3. `初始化`A：`@Autowired`依赖注入B（此时需要去容器内获取B）
4. 为了完成依赖注入B，会通过`getBean(B)`去容器内找B。但此时B在容器内不存在，就走向**B的创建之路~**
5. `实例化`B，并将其放入缓存。（此时B也能够被引用了）
6. `初始化`B，`@Autowired`依赖注入A（此时需要去容器内获取A）
7. `此处重要`：初始化B时会调用`getBean(A)`去容器内找到A，上面我们已经说过了此时候因为A已经实例化完成了并且放进了缓存里，所以这个时候去看缓存里是已经存在A的引用了的，所以`getBean(A)`能够正常返回
8. **B初始化成功**（此时已经注入A成功了，已成功持有A的引用了），return（注意此处return相当于是返回最上面的`getBean(B)`这句代码，回到了初始化A的流程中~）。
9. 因为B实例已经成功返回了，因此最终**A也初始化成功**
10. **到此，B持有的已经是初始化完成的A，A持有的也是初始化完成的B，完美~**