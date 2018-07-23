#什么是泛型#
泛型是Java SE 1.5 的新特性，《Java 核心技术》中对泛型的定义是：
> **“泛型” 意味着编写的代码可以被不同类型的对象所重用。**

可见泛型的提出是为了编写重用性更好的代码。

泛型的本质是**参数化类型**，也就是说所操作的数据类型被指定为一个参数。 

#为什么引入泛型#
在引入泛型之前，要想实现一个通用的、可以处理不同类型的方法，你需要使用 Object 作为属性和方法参数，比如这样：
    public class Generic {
	    private Object[] mData;
	
	    public Generic(int capacity) {
	        mData = new Object[capacity];
	    }
	
	    public Object getData(int index) {
	        //...
	        return mData[index];
	    }
	
	    public void add(int index, Object item) {
	        //...
	        mData[index] = item;
	    }
	}

它使用一个 Object 数组来保存数据，这样在使用时可以添加不同类型的对象:

	Generic generic = new Generic(10);
    generic.add(0,"shixin");
    generic.add(1, 23);

然而由于 Object 是所有类的父类，所有的类都可以作为成员被添加到上述类中；当需要使用的时候，必须进行强制转换，而且这个强转很有可能出现转换异常：

    String item1 = (String) generic.getData(0);
    String item2 = (String) generic.getData(1);

上面第二行代码将一个 Integer 强转成 String，运行时会报错 ：

    java.lang.ClassCastException: java.lang.Integer cannot be cast to java.lang.String
    
    at net.sxkeji.shixinandroiddemo2.test.generic.GenericTest.getData(GenericTest.java:46)

**可以看到，使用 Object 来实现通用、不同类型的处理，有这么两个缺点：**

- 每次使用时都需要强制转换成想要的类型
- 在编译时编译器并不知道类型转换是否正常，运行时才知道，不安全

根据《Java 编程思想》中的描述，泛型出现的动机在于：
> 有许多原因促成了泛型的出现，而最引人注意的一个原因，就是为了创建容器类。

事实上，在 JDK 1.5 出现泛型以后，许多集合类都使用泛型来保存不同类型的元素，比如 Collection:

**实际上引入泛型的主要目标有以下几点：**

- 类型安全 
	- 泛型的主要目标是提高 Java 程序的类型安全
	- 编译时期就可以检查出因 Java 类型不正确导致的 ClassCastException 异常
	- 符合越早出错代价越小原则
- 消除强制类型转换 
	- 泛型的一个附带好处是，使用时直接得到目标类型，消除许多强制类型转换
	- 所得即所需，这使得代码更加可读，并且减少了出错机会
- 潜在的性能收益 
	- 由于泛型的实现方式，支持泛型（几乎）不需要 JVM 或类文件更改
	- 所有工作都在编译器中完成
	- 编译器生成的代码跟不使用泛型（和强制类型转换）时所写的代码几乎一致，只是更能确保类型安全而已

#泛型的使用方式#
泛型的本质是**参数化类型**，也就是说所操作的数据类型被指定为一个参数。

类型参数的意义是告诉编译器这个集合中要存放实例的类型，从而在添加其他类型时做出提示，在编译时就为类型安全做了保证。

这种参数类型可以用在类、接口和方法的创建中，分别称为泛型类、泛型接口、泛型方法。

	 /**
	 *      Description: 泛型类
	 */
	public class GenericClass<F> {
	    private F mContent;
	
	    public GenericClass(F content){
	        mContent = content;
	    }
	
	    /**
	     * 泛型方法
	     * @return
	     */
	    public F getContent() {
	        return mContent;
	    }
	
	    public void setContent(F content) {
	        mContent = content;
	    }
	
	    /**
	     * 泛型接口
	     * @param 
	     */
	    public interface GenericInterface<T>{
	        void doSomething(T t);
	    }
	} 

##泛型类##
泛型类和普通类的区别就是类名后有类型参数列表 ，既然叫“列表”了，当然这里的类型参数可以有多个，参数名称由开发者决定。

类名中声明参数类型后，内部成员、方法就可以使用这个参数类型，比如上面的 GenericClass 就是一个泛型类，它在类名后声明了类型 F，它的成员、方法就可以使用 F 表示成员类型、方法参数/返回值都是 F 类型。

泛型类最常见的用途就是作为容纳不同类型数据的容器类，比如 Java 集合容器类。

##泛型接口##
和泛型类一样，泛型接口在接口名后添加类型参数，比如上面的 GenericInterface，接口声明类型后，接口方法就可以直接使用这个类型。

实现类在实现泛型接口时需要指明具体的参数类型，不然默认类型是 Object，这就失去了泛型接口的意义。

未指明类型的实现类，默认是 Object 类型：

    public class Generic implements GenericInterface{
	
	    @Override
	    public void doSomething(Object o) {
	        //...
	    }
	}

指明了类型的实现：

	public class Generic implements GenericInterface<String>{
	    @Override
	    public void doSomething(String s) {
	        //...
	    }
	}

泛型接口比较实用的使用场景就是用作策略模式的公共策略,比如 [http://blog.csdn.net/u011240877/article/details/53399019](http://blog.csdn.net/u011240877/article/details/53399019 "Java 解惑：Comparable 和 Comparator 的区别 ") 中介绍的 Comparator，它就是一个泛型接口：

    public interface Comparator<T> {
	
	    public int compare(T lhs, T rhs);
	
	    public boolean equals(Object object);
	}

泛型接口定义基本的规则，然后作为引用传递给客户端，这样在运行时就能传入不同的策略实现类。

##泛型方法##
泛型方法是指使用泛型的方法，如果它所在的类是个泛型类，那就很简单了，直接使用类声明的参数。

如果一个方法所在的类不是泛型类，或者他想要处理不同于泛型类声明类型的数据，那它就需要自己声明类型，举个例子：

    /**
	 * 传统的方法，会有 unchecked ... raw type 的警告
	 * @param s1
	 * @param s2
	 * @return
	 */
	public Set union(Set s1, Set s2){
	    Set result = new HashSet(s1);
	    result.addAll(s2);
	    return result;
	}
	
	/**
	 * 泛型方法，介于方法修饰符和返回值之间的称作 类型参数列表 <A,V,F,E...> (可以有多个)
	 *      类型参数列表 指定参数、返回值中泛型参数的类型范围，命名惯例与泛型相同
	 * @param s1
	 * @param s2
	 * @param <E>
	 * @return
	 */
	public <E> Set<E> union2(Set<E> s1, Set<E> s2){
	    Set<E> result = new HashSet<>(s1);
	    result.addAll(s2);
	    return result;
	}

注意上述代码在返回值前面也有个 <E>，它和类名后面的类型参数列表意义一致，指明了这个方法中类型参数的意义、范围。

#泛型的通配符#
有时候希望传入的类型有一个指定的范围，从而可以进行一些特定的操作，这时候就是通配符边界登场的时候了。

泛型中有三种通配符形式：

- **<?>** 无限制通配符
- **<? extends E>** :extends关键字声明了类型的上界，表示参数化的类型可能是所指定的类型，或者是此类型的子类
- **<? super E>** : super关键字声明了类型的下界，表示参数化的类型可能是指定的类型，或者是此类型的父类

##无限制通配符 < ?>##
要使用泛型，但是不确定或者不关心实际要操作的类型，可以使用无限制通配符（尖括号里一个问号，即 <?> ），表示可以持有任何类型。

大部分情况下，这种限制是好的，但这使得一些理应正确的基本操作都无法完成，比如交换两个元素的位置，看代码：

    private void swap(List<?> list, int i, int j){
	    Object o = list.get(i);
	    list.set(j,o);
	}

这个代码看上去应该是正确的，但 Java 编译器会提示编译错误，set 语句是非法的。编译器提示我们把方法中的 List<?> 改成 List<Object> 就好了，这是为什么呢？ ? 和 Object 不一样吗？

的确因为 ? 和 Object 不一样，List<?> 表示未知类型的列表，而 List<Object> 表示任意类型的列表。

比如传入个 List<String> ，这时 List 的元素类型就是 String，想要往 List 里添加一个 Object，这当然是不可以的。

借助带类型参数的泛型方法，这个问题可以这样解决：

     private <E> void swapInternal(List<E> list, int i, int j) {
	    //...
	    list.set(i, list.set(j, list.get(i)));
	}
	
	private void swap(List<?> list, int i, int j){
	    swapInternal(list, i, j);
	}

swap 可以调用 swapInternal，而带类型参数的 swapInternal 可以写入。Java容器类中就有类似这样的用法，公共的 API 是通配符形式，形式更简单，但内部调用带类型参数的方法。

##上界通配符 < ? extends E>##
在类型参数中使用 extends 表示这个泛型中的参数必须是 E 或者 E 的子类，这样有两个好处：
- 如果传入的类型不是 E 或者 E 的子类，编辑不成功
- 泛型中可以使用 E 的方法，要不然还得强转成 E 才能使用

举个例子：

    /**
	 * 有限制的通配符之 extends （有上限），表示参数类型 必须是 BookBean 及其子类，更灵活
	 * @param arg1
	 * @param arg2
	 * @param <E>
	 * @return
	 */
	private <K extends ChildBookBean, E extends BookBean> E test2(K arg1, E arg2){
	    E result = arg2;
	    arg2.compareTo(arg1);
	    //.....
	    return result;
	}

可以看到，类型参数列表中如果有多个类型参数上限，用逗号分开。

##下界通配符 < ? super E>##
在类型参数中使用 super 表示这个泛型中的参数必须是 E 或者 E 的父类。

根据代码介绍吧：

    private <E> void add(List<? super E> dst, List<E> src){
	    for (E e : src) {
	        dst.add(e);
	    }
	}

可以看到，上面的 dst 类型 “大于等于” src 的类型，这里的“大于等于”是指 dst 表示的范围比 src 要大，因此装得下 dst 的容器也就能装 src。

##通配符比较##
通过上面的例子我们可以知道，无限制通配符 < ?> 和 Object 有些相似，用于表示无限制或者不确定范围的场景。

两种有限制通配形式 < ? super E> 和 < ? extends E> 也比较容易混淆，我们再来比较下。

它们的目的都是为了使方法接口更为灵活，可以接受更为广泛的类型。

- < ? super E> 用于灵活写入或比较，使得对象可以写入父类型的容器，使得父类型的比较方法可以应用于子类对象。
- < ? extends E> 用于灵活读取，使得方法可以读取 E 或 E 的任意子类型的容器对象。

用《Effective Java》 中的一个短语来加深理解：
> 为了获得最大限度的灵活性，要在表示 生产者或者消费者 的输入参数上使用通配符，使用的规则就是：生产者有上限、消费者有下限：

>  PECS: producer-extends, costumer-super

因此使用通配符的基本原则：

- 如果参数化类型表示一个 T 的生产者，使用 < ? extends T>;
- 如果它表示一个 T 的消费者，就使用 < ? super T>；
- 如果既是生产又是消费，那使用通配符就没什么意义了，因为你需要的是精确的参数类型。

小总结一下：

- T 的生产者的意思就是结果会返回 T，这就要求返回一个具体的类型，必须有上限才够具体；
- T 的消费者的意思是要操作 T，这就要求操作的容器要够大，所以容器需要是 T 的父类，即 super T；

举个例子：

        private  <E extends Comparable<? super E>> E max(List<? extends E> e1){
	        if (e1 == null){
	            return null;
	        }
	        //迭代器返回的元素属于 E 的某个子类型
	        Iterator<? extends E> iterator = e1.iterator();
	        E result = iterator.next();
	        while (iterator.hasNext()){
	            E next = iterator.next();
	            if (next.compareTo(result) > 0){
	                result = next;
	            }
	        }
	        return result;
    	}

上述代码中的类型参数 E 的范围是 <E extends Comparable<? super E>>，我们可以分步查看：

- 要进行比较，所以 E 需要是可比较的类，因此需要 extends Comparable<…>（注意这里不要和继承的 extends 搞混了，不一样）
- Comparable< ? super E> 要对 E 进行比较，即 E 的消费者，所以需要用 super
- 而参数 List< ? extends E> 表示要操作的数据是 E 的子类的列表，指定上限，这样容器才够大

#泛型的类型擦除#

Java 中的泛型和 C++ 中的模板有一个很大的不同：

- C++ 中模板的实例化会为每一种类型都产生一套不同的代码，这就是所谓的代码膨胀。
- Java 中并不会产生这个问题。虚拟机中并没有泛型类型对象，所有的对象都是普通类。

在 Java 中，泛型是 Java 编译器的概念，用泛型编写的 Java 程序和普通的 Java 程序基本相同，只是多了一些参数化的类型同时少了一些类型转换。

实际上泛型程序也是首先被转化成一般的、不带泛型的 Java 程序后再进行处理的，编译器自动完成了从 Generic Java 到普通 Java 的翻译，Java 虚拟机运行时对泛型基本一无所知。

当编译器对带有泛型的java代码进行编译时，它会去执行**类型检查**和**类型推断**，然后生成普通的不带泛型的字节码，这种普通的字节码可以被一般的 Java 虚拟机接收并执行，这在就叫做 **类型擦除**（type erasure）。

实际上无论你是否使用泛型，集合框架中存放对象的数据类型都是 Object，这一点不仅仅从源码中可以看到，通过反射也可以看到。

    List<String> strings = new ArrayList<>();
	List<Integer> integers = new ArrayList<>();
	System.out.println(strings.getClass() == integers.getClass());//true

上面代码输出结果并不是预期的 false，而是 true。其原因就是泛型的擦除。

##擦除的实现原理##

一直有个疑问，Java 编译器在编译期间擦除了泛型的信息，那运行中怎么保证添加、取出的类型就是擦除前声明的呢？

从[http://javarevisited.blogspot.hk/2011/09/generics-java-example-tutorial.html](http://javarevisited.blogspot.hk/2011/09/generics-java-example-tutorial.html)了解到，原来泛型也只是一个语法糖，摘几段话加深理解：

> The buzzing keyword is “Type Erasure”, you guessed it right it’s the same thing we used to in our schools for erasing our mistakes in writing or drawing :).

> The Same thing is done by java compiler, when it sees code written using Generics it completely erases that code and convert it into raw type i.e. code without Generics. All type related information is removed during erasing. So your ArrayList becomes plain old ArrayList prior to JDK 1.5, formal type parameters e.g. < K, V> or < E> gets replaced by either Object or Super Class of the Type.

> Also, when the translated code does not have correct type, the compiler inserts a type casting operator. This all done behind the scene so you don’t need to worry about what important to us is that Java compiler guarantees type-safety and flag any type-safety relate error during compilation.

> In short Generics in Java is syntactic sugar and doesn’t store any type related information at runtime. All type related information is erased by Type Erasure, this was the main requirement while developing Generics feature in order to reuse all Java code written without Generics.

大概意思就是：
> Java 编辑器会将泛型代码中的类型完全擦除，使其变成原始类型。

> 当然，这时的代码类型和我们想要的还有距离，接着 Java 编译器会在这些代码中加入类型转换，将原始类型转换成想要的类型。这些操作都是编译器后台进行，可以保证类型安全。

> 总之泛型就是一个语法糖，它运行时没有存储任何类型信息。

##擦除导致的泛型不可变性##
泛型中没有逻辑上的父子关系，如 List 并不是 List 的父类。两者擦除之后都是List，所以形如下面的代码，编译器会报错：

    /**
	 * 两者并不是方法的重载。擦除之后都是同一方法，所以编译不会通过。
	 * 擦除之后：
	 * 
	 * void m(List numbers){}
	 * void m(List strings){} //编译不通过，已经存在相同方法签名
	 */
	void method(List<Object> numbers) {
	
	}
	
	void method(List<String> strings) {
	
	}

泛型的这种情况称为 不可变性，与之对应的概念是 协变、逆变：

- 协变：如果 A 是 B 的父类，并且 A 的容器（比如 List< A>） 也是 B 的容器（List< B>）的父类，则称之为协变的（父子关系保持一致）
- 逆变：如果 A 是 B 的父类，但是 A 的容器 是 B 的容器的子类，则称之为逆变（放入容器就篡位了）
- 不可变：不论 A B 有什么关系，A 的容器和 B 的容器都没有父子关系，称之为不可变

Java 中数组是协变的，泛型是不可变的。

如果想要让某个泛型类具有协变性，就需要用到边界。

##擦除的拯救者：边界##
我们知道，泛型运行时被擦除成原始类型，这使得很多操作无法进行.

如果没有指明边界，类型参数将被擦除为 Object。

如果我们想要让参数保留一个边界，可以给参数设置一个边界，泛型参数将会被擦除到它的第一个边界（边界可以有多个），这样即使运行时擦除后也会有范围。

比如：

    public class GenericErasure {
	    interface Game {
	        void play();
	    }
	    interface Program{
	        void code();
	    }
	
	    public static class People<T extends Program & Game>{
	        private T mPeople;
	
	        public People(T people){
	            mPeople = people;
	        }
	
	        public void habit(){
	            mPeople.code();
	            mPeople.play();
	        }
	    }
	}

上述代码中, People 的类型参数 T 有两个边界,编译器事实上会把类型参数替换为它的第一个边界的类型。

#泛型的规则#
- 泛型的参数类型只能是类（包括自定义类），不能是简单类型。
- 同一种泛型可以对应多个版本（因为参数类型是不确定的），不同版本的泛型类实例是不兼容的。
- 泛型的类型参数可以有多个
- 泛型的参数类型可以使用 extends 语句，习惯上称为“有界类型”
- 泛型的参数类型还可以是通配符类型，例如 Class

#泛型的使用场景#
当类中要操作的引用数据类型不确定的时候，过去使用 Object 来完成扩展，JDK 1.5后推荐使用泛型来完成扩展，同时保证安全性。