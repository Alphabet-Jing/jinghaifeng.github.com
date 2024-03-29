---
layout: post
keywords: blog
description: blog
title: "公共技术点之 Java 反射 Recflection"
categories: [Java]
tags: [Java]
---
{% include JB/setup %}

#公共技术点之 Java 反射 Recflection
[**转自 codeKK**](http://codekk.com/open-source-project-analysis/detail/Android/Mr.Simple/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8B%20Java%20%E5%8F%8D%E5%B0%84%20Reflection)

## 1. 了解 Java中的反射
###1.1 什么是Java的反射
Java 反射时可以让我们在运行时获取类的函数、属性、父类、接口等 Class 内部信息的机制。通过反射还可以让我们在运行期实例化对象，调用方法，通过调用 get/set 方法获取变量的值，即使方法或属性是私有的也可以通过反射的形式调用，这种“看透 class "的能力叫做`内省`，这种能力在框架开发中尤为重要。有些情况下，我们需要使用的类在运行时才能确定，这个时候我们不能在编译期就使用它，因此只能通过反射的形式来使用在运行时才存在的类（该类符合某种特定的规范，例如 JDBC ），这是反射用得比较多的场景。

还有比较常见的场景就是编译时我们对于类的内部信息不可知，必须在运行时才能获取类的具体信息。比如 ORM 框架，在运行时才能够获取类中的各个属性，然后通过反射的形式获取其属性名和值，存入数据库。这也是反射比较经典的场景之一。

###1.2 Class 类
既然反射是操作 Class 信息的，那么 Class 又是什么？

 ![java class](https://raw.githubusercontent.com/android-cn/android-open-project-analysis/master/tech/reflection/image/arch.png)

但我们编写完一个 Java 项目后，所有的 Java 文件都会被编译成一个 .class 文件，这些 class 对象承载了这个类型的父类、接口、构造函数、方法、属性等原始信息，这些 class 文件在程序运行时会被 ClassLoader 加载到虚拟机上。当一个类被加载以后， Java 虚拟机就会在内存中自动生成一个 Class 对象。我们通过 new 的形式创建对象实际上就是通过这些 Class 来创建的，只是这个过程对于我们不是透明的。

下面章节中，我们会为大家演示反射常用的 api ， 从代码的角度理解反射。

##2. 反射 Class 以及构造对象
###2.1 获取 Class 对象
在你想检查一个类信息之前，你首先需要获取类的 Class 对象。 Java 中的所有类型包括基本类型，即使时数组都有与之关联的 Class 类对象。如果你在编译期间知道一个类的名字的话，那么你可以通过如下方式获取一个类的 Class 信息。
	
	Class<?> myObjectClass = MyObject.class;

如果你已经得到了某个对象，但是你想获取这个对象的 Class 对象，那么你可以通过下面方法得到：

	Student me = new Student(mr.simple);
	Class<?> clazz = me.getclass();

如果你在编译器获取不到目标类型，但是你知道它的完整路径，那么你可以通过如下形式获取 Class 对象：

	Class<?> myObjectClass = Class.forName("com.simple.User");

在使用 Class.forName() 方法时，你必须提供一个类的全名，这个全名包括所在的包的名字。

如果在调用 Class.forName()方法时，没有在编译路径（ classpath ）下找到对应的类,那么将会抛出 ClassNotFoundException。

**接口说明**

	//加载指定的 Class 对象，参数 1 为要加载的类的完整路径，例如 "com.simple.Student".（常用方式）
	public static Class<?> forName (String className)

	//加载指定的 Class 对象，参数 1 要为加载类的完整路径，例如"com.simple.Student"；
	//参数 2 为时候要初始化该 Class 对象，参数 3 为指定加载该类的 ClassLoader .
	public static Class<?> forName(String className,boolean shouInitialize,ClassLoader classLoader)

###2.2 通过 Class 对象构造目标类型的对象
一旦你拿到 Class 对象之后，你就可以为所欲为！当你善用它的时候就是神兵利器，当你心怀鬼胎的时他就会变成恶魔。但获取 Class 对象只是一步，我们需要在执行那些强大的行为之前通过 Class 对象构造出该类型的对象，然后才能通过该对象释放能量。

我们知道，在 Java 中构建对象，必须通过该类的构造函数，那么其实反射也时一样的。但是他们的确实有区别。通过反射构造的对象，我们首先要获取类的 Constructor 构造器对象，然后通过 构造器 来创建目标的对象。

	private static classForName(){
		try{
			// 获取 Class 对象
			Class<?> clz = Class.forName("org.java.advance.reflect.Student");
			// 通过 Class 对象获取 Constructor ,Student 的构造器有一个字符串参数，因此这里需要传递参数的类型 （Student 类见后面的代码）
			Constructor<?> constructor = clz.getContructor(String.class);
			// 通过Constructor 来创建 Student 对象
			Object obj = constructor.newInstance("mr.simple");
			System.out.println("obj:"+obj.toString());
		} catch (Exception e) {
			e.printStackTrace();
		}
	}
通过上述代码，我们就可以在运行时通过完整的类名来构建对象。

**获取构造函数接口**

	// 获取一个公有的构造函数接口，参数为可变参数，如果构造函数有参数，那么需要将参数的类型传给 getConstructor 方法
	public Constructor<T> getConstructor (Class...<?> parameterTypes)
	//获取目标类所有的公有构造函数
	public Constructors[]<?> getConstructors()
**注意：**当你通过反射获取到 Constructor、 Method、Field 之后，在反射调用之前需将此对象的 accessible 标志设置为 true，以此来提升反射速度。值为 true 则指示反射的对象在使用时应该取消 Java 语言访问检查，为 false 则指示反射对象应该实施 Java 语言访问检查。
	 
	 Constructor<?> constructor = clz.getConstructor(String.class);
	// 设置 Constructor 的 Accessible
   	constructor.setAccessible(true);

   	// 设置 Methohd 的 Accessible
   	Method learnMethod = Student.class.getMethod("learn"， String.class);
   	learnMethod.setAccessible(true);

例子相关类：
**Person.java**
	
	public class Person {
    		String mName;

    		public Person(String aName) {
        		mName = aName;
    		}

    		private void sayHello(String friendName) {
        		System.out.println(mName + " say hello to " + friendName);
    		}

    		protected void showMyName() {
        		System.out.println("My name is " + mName);
    		}

    		public void breathe() {
        		System.out.println(" take breathe ");
    		}
	}

**Student.java**

	public class Student extends Person implements Examination {
    		// 年级
    		int mGrade;

    		public Student(String aName) {
        		super(aName);
    		}

    		public Student(int grade, String aName) {
        		super(aName);
        		mGrade = grade;
    		}

    		private void learn(String course) {
        		System.out.println(mName + " learn " + course);
    		}

    		public void takeAnExamination() {
        		System.out.println(" takeAnExamination ");
    		}

    		public String toString() {
        		return " Student :  " + mName;
    		}
	}

**Breathe.java**

	// 呼吸接口
	public interface Breathe {
    		public void breathe();
	}

**Examination.java**

	// 考试接口
	public interface Examination {
    		public void takeAnExamination();
	}

##3 反射获取类中函数
###3.1 获取当前类中定义的方法
要获取当前类中定义的所有方法可以通过 Class 中的 getDeclaredMethods 函数，它会获取到当前类中的 public 、default 、protected 、 private 的所有方法。而 getDeclaredMethod(String name ,  Class..<?> parameterTypes)则是获取某个指定的方法。

	public class ReflectionTest {
    		public static void main(String[] args) {
        		String fClazz = new String("string");
        		Method[] methods = fClazz.getClass().getDeclaredMethods();
        		for (Method method : methods) {
            			System.out.println("method's name : "+method.getName());
            			if (method.getName().equals("lastIndexOfSupplementary")){
                		System.out.println("lastIndexOfSupplementary is 	contained!!!!!!!!!!");
            			}
        		}

        		try {
            			Method indexOfSupplementary = fClazz.getClass().getDeclaredMethod("indexOfSupplementary",int.class,int.class);
            			Class<?>[] paramClasses = indexOfSupplementary.getParameterTypes();
            			for (Class<?> paramClass : paramClasses) {
                			System.out.println("parameter:"+paramClass.getName());
            			}

            		System.out.println("Method name : "+ indexOfSupplementary.getName()+"\n is private = "+ Modifier.isPrivate(indexOfSupplementary.getModifiers()));
            		indexOfSupplementary.setAccessible(true);
            		System.out.println(indexOfSupplementary.invoke(fClazz,'i',0) + " >>>>>>");
        		} catch (NoSuchMethodException e) {
            			e.printStackTrace();
        		} catch (InvocationTargetException e) {
            			e.printStackTrace();
        		} catch (IllegalAccessException e) {
            			e.printStackTrace();
        		}

    		}
	}

**注意：
* 参数为 int 对应 int.class、float 对应 float.class ... ...
* 若方法为 private ，需要给 method.setAccessible(true) ，赋予执行权限。

###3.2 获取当前类、父类中定义的公有方法
要获取当前类以及父类中的所有 public 方法可以通过 Class 中的 getMethods 函数，而 getMethod 则是获取某个指定的方法。
	
	private static void showMethods() {
		Student student = new Student("mr.simple");
        	// 获取所有方法
        	Method[] methods = student.getClass().getMethods();
        	for (Method method : methods) {
            		System.out.println("method name : " + method.getName());
        	}

        	try {
            		// 通过 getMethod 只能获取公有方法，如果获取私有方法则会抛出异常，比如这里就会抛异常
            		Method learnMethod = student.getClass().getMethod("learn", String.class);
            		// 是否是 private 函数，属性是否是 private 也可以使用这种方式判断
            		System.out.println(learnMethod.getName() + " is private " + Modifier.isPrivate(learnMethod.getModifiers()));
            		// 调用 learn 函数
            		learnMethod.invoke(student, "java");
        	} catch (Exception e) {
            		e.printStackTrace();
        	}
    	}

**接口说明**

	// 获取 Class 对象中指定函数名和参数的函数，参数 1 为函数名，参数 2为 参数类型表
	public Method getDeclaredMethod(String name, Class<?> parameterTypes)
	// 获取该 Class 对象中的所有函数 （不包含从父类继承的函数）
	public Method[] getDeclaredMethods()
	//获取指定的 Class 对象的 公有函数 ，参数 1 为函数名，参数 2 为参数类型列表
	public Method getMethod(String name , Class<?> parameterTypes)
	//获取该 Class 对象中的所有公有函数，包括 父类和接口类继承下来的函数。
	public Method[] getMethods()

**注意：**
这里需要注意的是 getDeclaredMethod 和 getDeclaredMethods 包含 private、protected、default、public 的函数，并且通过这两个函数获取到的只是在自身中定义的函数，从父类中集成的函数不能够获取到。而 getMethod 和 getMethods 只包含 public 函数，父类中的公有函数也能够获取到。

##4 反射获取类中的属性
###4.1 获取当前类中定义的属性
要获取当前类中定义的所有属性可以通过 Class 中的 getDeclaredFields 函数，它会获取当前类中的**public**、**default**、**protected**、**private**的所有属性。而 **geteclaredField**则是获取某个指定的属性。

    private static void showDeclaredFields() {
        Student student = new Student("mr.simple");
        // 获取当前类和父类的所有公有属性
        Field[] publicFields = student.getClass().getDeclaredFields();
        for (Field field : publicFields) {
            System.out.println("declared field name : " + field.getName());
        }

        try {
            // 获取当前类和父类的某个公有属性
            Field gradeField = student.getClass().getDeclaredField("mGrade");
            // 获取属性值
            System.out.println(" my grade is : " + gradeField.getInt(student));
            // 设置属性值
            gradeField.set(student, 10);
            System.out.println(" my grade is : " + gradeField.getInt(student));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

###4.2 获取当前类、父类中定义的公有属性
要获取当前类以及父类中的所有**public**属性可以通过**Class**中的**getFields**函数，而 getField 这是获取某个指定属性。

    private static void showFields() {
        Student student = new Student("mr.simple");
        // 获取当前类和父类的所有公有属性
        Field[] publicFields = student.getClass().getFields();
        for (Field field : publicFields) {
            System.out.println("field name : " + field.getName());
        }

        try {
            // 获取当前类和父类的某个公有属性
            Field ageField = student.getClass().getField("mAge");
            System.out.println(" age is : " + ageField.getInt(student));
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

**接口说明**

	// 获取 Class 对象中指定属性名的属性，参数一为属性名
	public Method getDeclaredField (String name)

	// 获取该 Class 对象中的所有属性( 不包含从父类继承的属性 )
	public Method[] getDeclaredFields ()

	// 获取指定的 Class 对象中的**公有**属性，参数一为属性名
	public Method getField (String name)

	// 获取该 Class 对象中的所有**公有**属性 ( 包含从父类和接口类集成下来的公有属性 )
	public Method[] getFields ()

**注意**
这里需要注意的是 getDeclaredField 和 getDeclaredFields 包含 private、protected、default、public 的属性，并且通过这两个函数获取到的只是在自身中定义的属性，从父类中集成的属性不能够获取到。而 getField 和 getFields 只包含 public 属性，父类中的公有属性也能够获取到。

##5 反射父类与接口
###5.1 获取父类
获取 Class 父类

    Student student = new Student("mr.simple");
    Class<?> superClass = student.getClass().getSuperclass();
    while (superClass != null) {
        System.out.println("Student's super class is : " + superClass.getName());
        // 再获取父类的上一层父类，直到最后的 Object 类，Object 的父类为 null
        superClass = superClass.getSuperclass();
    }

###5.2 获取接口
获取 Class 对象中实现的接口

    private static void showInterfaces() {
        Student student = new Student("mr.simple");
        Class<?>[] interfaceses = student.getClass().getInterfaces();
        for (Class<?> class1 : interfaceses) {
            System.out.println("Student's interface is : " + class1.getName());
        }
    }

##6 获取注解信息
在框架开发中，注解加反射的组合使用最为常见。定义注解时，我们通过 @ Target 指定注解能够作用的类型。
	
    @Target({
            ElementType.METHOD, ElementType.FIELD, ElementType.TYPE
    })
    @Retention(RetentionPolicy.RUNTIME)
    static @interface Test {

    }

上述注解的 @Target 表示该注解只能使用在函数上，还有 Type、Field、PARAMETER等类型。通过反射API 我们也能够获取一个Class 对像获取类型、属性、函数等相关对象，通过这些对象的 getAnnotation 接口获取到对应的注解信息。首先我们需要给目标对象上添加上注解。

	@Test(tag = "Student class Test Annoatation")
	public class Student extends Person implements Examination {
    		// 年级
    		@Test(tag = "mGrade Test Annotation ")
    		int mGrade;

   		 // ......
	}

然后通过相关注解函数得到注解信息。

    private static void getAnnotationInfos() {
        Student student = new Student("mr.simple");
        Test classTest = student.getClass().getAnnotation(Test.class);
        System.out.println("class Annotatation tag = " + classTest.tag());

        Field field = null;
        try {
            field = student.getClass().getDeclaredField("mGrade");
            Test testAnnotation = field.getAnnotation(Test.class);
            System.out.println("属性的 Test 注解 tag : " + testAnnotation.tag());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

输出结果为：

	class Annotatation tag = Student class Test Annoatation
	属性的 Test 注解 tag : mGrade Test Annotation

**接口说明**

	// 获取指定类型的注解
	public <A extends Annotation> A getAnnotation(Class<A> annotationClass) ;
	// 获取 Class 对象中的所有注解
	public Annotation[] getAnnotations() ;
##杂谈
反射作为 Java 语言的重要特性，在开发中有着极为重要的作用。很多开发框架都是基于反射来实现对目标对象的操作，而反射配合注解更是设计开发框架的主流选择，例如 ActiveAndroid，因此深入了解反射的作用以及使用对于日后开发和学习必定大有益处
