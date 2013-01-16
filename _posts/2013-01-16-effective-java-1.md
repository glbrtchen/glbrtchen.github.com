---
layout: post
title: "Effective Java 学习笔记(1)"
description: ""
category: 
tags: [Java]
---
{% include JB/setup %}

第一条: 考虑用静态工厂方法代替构造器
------------------------------------

Boolean的一个静态方法

	public static Boolean valueOf(boolean b) {
		return b : Boolean.TRUE : Boolean.FALSE
	}

以上这种方式即静态工厂方法, 与设计模式中的工厂方法模式无对应关系
静态工厂方法的优缺点
优点1: 对比构造器, 静态工厂方法有自己的名字. 静态工厂方法的名称可以区分返回对象的含义. 例如:

	public class Person {
		public static Person male() {
			return new Person(true);
		}
		public static Person female() {
			return new Person(false);
		}
		private boolean male;
		public Persion(boolean male) {
			this.male = male;
		}
	}
	Person male = new Person(true);
	Person female = new Person(false);
	Person male = Person.male();
	Person female = Person.female();

静态工厂方法的优势在于可以隐藏参数的细节, 方便调用者区分构造出的对象的含义

优点2: 不必每次调用都创建一个新的对象, Boolean.valueOf的实现

	public class Person {
		private static MALE = new Person(true);
		private static FEMALE = new Person(false);
		public static Person male() {
			return MALE;
		}
		public static Person female() {
			return FEMALE;
		}
		private boolean male;
		public Persion(boolean male) {
			this.male = male;
		}
	}
	Person male = new Person(true);
	Person female = new Person(false);
	Person male = Person.male();
	Person female = Person.female();

优点3: 可以返回返回类型的任何子类型的对象, 隐藏了类型的细节, 对于类型调整会更加方便
java.util.EnumSet会有RegularEnumSet/JumboEnumSet, 实际返回的都是EnumSet

服务提供者框架(Service Provider Framework)有三个重要组件:
1. 服务接口(Service Interface)
2. 提供者注册接口(Provider Registration API)
3. 服务访问接口(Service Access API)
4. 服务提供者接口(Service Provider Interface) - 可选

	// Service provider framework sketch

	// Service interface
	public interface Service {
		// Service-specific methods go here
	}

	// Service provider interface
	public interface Provider {
		Service newService();
	}

	// Noninstantiable class for service registration and access
	public class Services {
		private Services() {  } // Prevents instantiation (Item 4)

		// Maps service names to services
		private static final Map<String, Provider> providers = new ConcurrentHashMap<String, Provider>();
		public static final String DEFAULE_PROVIDER_NAME = "<def>";
		// Provider registration API
		public static void registerDefaultProvider(Provider p) {
			registerProvider(DEFAULE_PROVIDER_NAME, p);
		}
		public static void registerProvider(String name, Provider p) {
			providers.put(name, p);
		}

		// Service access API
		public static Service newInstance() {
			return newInstance(DEFAULE_PROVIDER_NAME);
		}
		public static Service newInstance(String name) {
			Provider p = providers.get(name);
			if (p == null)
				throw new IllegalArgumentException("No provider registered with name: " + name");
			return p.newService();
		}
	}

优点4: 创建参数化实例时, 会使代码更简洁
	
	Map<String, List<String>> m = new HashMap<String, List<String>>();

如果提供如下方法: 

	public static <K, V> HashMap<K, V> newInstance() {
		return new HashMap<K, V>();
	}
	Map<String, List<String>> m = HashMap.newInstance();

缺点1: 类如果不含公有的或者受保护的构造器, 就不能被子类化. 导致鼓励使用复合(composition), 而不是继承.
缺点2: 与其他静态方法没有任何区别. 常用静态工厂方法的例子, 可以作为识别静态工厂方法的范例:
valueOf/of/getInstance/newInstance/getType/newType

第二条: 遇到多个构造器参数时要考虑用构建器
------------------------------------------
如果有大量构造参数时, 有如下几种方式:
1. 重叠构造器(telescoping constructor)模式
重叠构造器模式可行, 但是当有许多参数的时候, 客户端代码会很难编写, 并且仍然较难以阅读.
2. JavaBeans模式
因为构造过程被分解到多个调用中, 在构造过程中JavaBean可能处于不一致状态, 这样需要额外的努力来保障类的线程安全
3. Builder模式

	// Builder Pattern
	public class NutritionFacts {
		private final int servingSize;
		private final int servings;
		private final int calories;
		private final int fat;
		private final int sodium;
		private final int carbohydrate;

		public static class Builder {
			// Required parameters
			private final int servingSize;
			private final int servings;

			// Optional parameters - initilized to default values
			private int calories = 0;
			private int fat = 0;
			private int carbohydrate = 0;
			private int sodium = 0;

			public Builder(int servingSize, int servings) {
				this.servingSize = servingSize;
				this.servings = servings;
			}

			public Builder calories(int val) {
				this.calories = val; return this;
			}
			public Builder fat(int val) {
				this.fat = val; return this;
			}
			public Builder carbohydrate(int val) {
				this.carbohydrate = val; return this;
			}
			public Builder sodium(int val) {
				this.sodium = val; return this;
			}

			public NutritionFacts build() {
				return new NutritionFacts(this);
			}
		}

		private NutritionFacts(Builder builder) {
			servingSize = builder.servingSize;
			servings = builder.servings;
			calories = builder.calories;
			fat = builder.fat;
			sodium = builder.sodium;
			carbohydrate = builder.carbohydrate;
		}
	}

以上代码为Builder模式的范例, 在构造NutritionFacts时, 可以采用如下方式:
	NutritionFacts cocaCola = new NutritionFacts.Builder(240, 8).calories(100).sodium(35).carbohydrate(27).build();
在调用build方法时, 如果发生异常可以抛出IllegalStateException, 并且提示哪一个字段违反了约束; 另外一种方式是在Builder设置参数的时候抛出IllegalArgumentException

Builder模式需要额外创建一个对象, 因此性能上有略微增加的开销, 也就是在性能比较敏感的场景, 需要考虑避免增加开销; 同时Builder模式用于参数超过4个, 并且都是可变参数的情况下会更好.

第三条: 用私有构造器或者枚举类型强化Singleton属性
-------------------------------------------------

实现单例模式有如下三种:
1. 公有域方法

	// Singleton with public final field
	public class Elvis {
		public static final Elvis INSTANCE = new Elvis();
		private Elvis() { ... };
		public void leaveTheBuilding() { ... };
	}

2. 静态工厂方法

	// Singleton with static factory
	public class Elvis {
		private static final Elvis INSTANCE = new Elvis();
		private Elvis() { ... };
		public static Elvis getInstance() { return INSTANCE; }
		public void leaveTheBuilding() { ... };
	}

3. 在Java 1.5之后, 可以使用单元素的枚举类型

	// Enum singleton - the preferred approach
	public enum Elvis {
		INSTANCE;
		public void leaveTheBuilding() { ... };
	}

公有域方法的好处在于比较明显的说明该类是单例模式, 性能上可能比静态工厂方法有优势; 静态工厂方法相对更加具有灵活性, 可以很容易的修改为非单例模式, 或者每个线程一个实例的模式, 上两种方法在序列化时, 会导致产生"假冒的Elvis", 为防止这种情况发生, 需要增加一个readResolve方法:

	// readResolve method to preserve singleton property
	private Object readResolve() {
		// Return the one true Elvis and let the garbage collector take care of the Elvis impersonator.
		return INSTANCE;
	}

第三种方法是目前认为最好的一种方法

第四条: 通过私有构造器强化不可实例化的能力
------------------------------------------

通常一些工具类不需要被实例化, 但是编译器本身会提供一个默认的无参构造器, 会导致该类有意无意的被实例化. 因此可以提供一个私有构造器来限制此类情况. 例如java.lang.Math就有一个私有构造器.

	// Noninstantiable utility class
	public class UtilityClass {
		// Suppress default constructor for noninstantiablity
		private UtilityClass() {
			throw new AssertionError();
		}
		... // Remainder omitted
	}

该方法限制了子类化: 所有构造器都必须显式或隐式的调用超类构造器, 这种情况下, 没有可访问的超类构造器.

第五条: 避免创建不必要的对象
----------------------------

1. 首先

	String s = new String("stringette");
	String s = "stringette";

是有明显区别的. JSL, 3.10.5中保障了, 相同的字符串字面常量不会创建两次, new String则会每次都创建一个对象
2. 不可变的对象, 可以避免创建, 例如Boolean的一个静态工厂方法, 每次返回的都是不可变的TRUE和FALSE对象

	public static Boolean valueOf(boolean b) {
		return b : Boolean.TRUE : Boolean.FALSE
	}

3. 已知不会被修改的可变对象, 也可以避免创建.

	class Person {
		private final Date birthDate;
		// Other fields, methods, and constructor omitted

		/**
		* The starting and ending dates of the baby boom.
		*/
		private static final Date BOOM_START;
		private static final Date BOOM_END;

		static {
			Calender gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
			gmtCal.set(1946, Calendar.JANUARY, 1, 0, 0, 0);
			BOOM_START = gmtCal.getTime();
			gmtCal.set(1965, Calendar.JANUARY, 1, 0, 0, 0);
			BOOM_END = gmtCal.getTime();
		}

		public boolean isBabyBoomer() {
			return birthDate.compareTo(BOOM_START) >= 0 && birthDate.compareTo(BOOM_END) < 0;
		}
	}

延迟初始化(lazy initializing)带来资源的节省, 但其带来的好处已经无法抵消代码复杂度带来的问题. 因此不建议.
4. 适配器(adapter)或者视图(view)的情况
Map接口的keySet方法, 返回Map对象的Key的Set对象, 每次调用返回的均为同一个, 而不是每次都创建一个Set对象. 因为返回的结果均由同一个Map支撑, 创建keySet的多个实例并无害处, 但是没有必要. 
在多线程的情况下, 返回结果需要有特别考虑.
5. autoboxing, 优先使用基本类型而不是装箱基本类型, 要当心无意识的自动装箱.

	// Hideously slow program! Can you spot the object creation?
	public static void main(String[] args) {
		Long sum = 0L;
		for (long i = 0; i < Integer.MAX_VALUE; i++) {
			sum += i;
		}
		System.out.println(sum);
	}

维护对象池(object pool)的讨论
除非必要(对象创建销毁的代价很大, 例如数据库连接, Socket等), 在现代JVM中对于小对象不需要特别构建对象池. 对象池增加了代码的复杂度和内存占用(footprint), 同时现代JVM的优化和垃圾回收技术已经缩小了对象池的性能优势.

第六条: 消除过期的对象引用
--------------------------

一段有内存泄漏可能的程序

	// Can you spot the "memory leak"?
	public class Stack {
		private Object[] elements;
		private int size = 0;
		private static final int DEFAULT_INITIAL_CAPACITY = 16;

		public Stack() {
			elements = new Object[DEFAULT_INITIAL_CAPACITY];
		}

		public void push(Object e) {
			ensureCapacity();
			elements[size++] = e;
		}
		public Object pop() {
			if (size == 0)
				throw new EmptyStackException();
			return elements[--size];
		}

		/**
		* Ensure space for at least one more element, roughly doubling the capacity each time the array needs to grow.
		*/
		private void ensureCapacity() {
			if (elements.length = size)
				elements = Arrays.copyOf(elements, 2 * size + 1);
		}
	}

在执行pop方法的时候, 虽然会把对象数组elements减小, 但是弹出Stack的Object对象还是存在一个隐性的引用, 在极端情况下会导致磁盘交换(Disk Paging), 或者内存溢出. 实测代码还是会做垃圾回收, 可能程序压栈的对象太小new Object(). 但是在几个垃圾回收的间隔, 如果存在大量申请内存的情况, 是可能会导致内存溢出. 当然也跟JVM 版本有关系.
原则上类自己管理内存, 程序员就应该警惕内存泄露问题. 
内存泄露的另一个来源是缓存. 如果一个对象载入到缓存中, 在进行GC的时候, 因此还存在一个对象引用所以不会被回收. 使用WeakHashMap来代表缓存. 检查内存泄露的问题需要使用Heap Profiler来检查. 例如JProfiler, jmap/jhat, visualvm等.

第七条: 避免使用终结方法
------------------------

终结方法(finalizer)通常是不可预测的, 也是很危险的, 一般情况下是不必要的.
Java中如果需要回收资源, 在try-finally块来完成是最好的方法.
终结方法在JVM定义中并不保证会及时执行, 并且会带来性能损失.
System.gc和System.runFinalization只是增加finalizer的执行几率, 但是没有确保终结方法一定被执行. System.runFinalizersOnExit和Runtime.runFinalizersOnExit声称确保终结方法被执行, 不过存在致命缺陷.
终止对象封装的资源可以提供一个显式的终止方法, 然后在try-finally块中调用. 通过finalizer可以编织一个资源释放的安全网, 同时可以释放本地对等体(native peer)
如果使用了终结方法, 那么就要记住调用super.finalize
使用终结方法守卫者(finalizer guardian)可以确保被终结对象的终结方法能被调用

	// Finalizer Guardian idiom
	public class Foo {
		// Sole purpose of this object is to finalize outer Foo object
		private final Object finalizerGuardian = new Object() {
			@Override protected void finalize() throws Throwable {
				... // Finalize outer Foo object
			}
		};
		... // Remainder omitted
	}

子类的finalize方法即使没有调用super.finalize, 该类的finalize也可以执行.