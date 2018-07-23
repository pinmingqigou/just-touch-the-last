***https://juejin.im/post/58c5e402128fe100603cc194***

# JAVA SE9新特性 #
## 引入java shell（jshell）新工具 ##

    G:\>jshell
    |  Welcome to JShell -- Version 9-ea
    |  For an introduction type: /help intro
    
    
    jshell> int a = 10
    a ==> 10
    
    jshell> System.out.println("a value = " + a )
    a value = 10

## 不可变集合类的工厂方法 ##
1.List 和 Set 接口使用 of() 方法创建一个空或者非空的不可变 List 或 Set 对象
    
    	List immutableList = List.of();
    
    	List immutableList = List.of("one","two","three");

2.Map 分别有两个方法用于创建不可变 Map 对象和不可变 Map.Entry 对象：of() 和 ofEntries()。

	jshell> Map emptyImmutableMap = Map.of() 
    emptyImmutableMap ==> {}

    jshell> Map nonemptyImmutableMap = Map.of(1, "one", 2, "two", 3, "three")
    nonemptyImmutableMap ==> {2=two, 3=three, 1=one}`

##接口中的私有方法

	public interface Card{

      private Long createCardID(){
    // Method implementation goes here.
      }
    
      private static void displayCardDetails(){
    // Method implementation goes here.
      }
    }

##HTTP 2 Client
在 Java SE 9 中，Oracle 公司将发布新的 HTTP 2 Client API 来支持 HTTP/2 协议和 WebSocket 特性。现有的 HTTP Client API 存在很多问题（如支持 HTTP/1.1 协议但是不支持 HTTP/2 协议和 WebSocket，仅仅作用在 Blocking 模式中，并存在大量性能问题），他们正在被使用新的 HTTP 客户端的 HttpURLConnection API 所替代。

Oracle 公司准备在 “java.net.http” 包下引入新的 HTTP 2 Client API。它将同时支持 HTTP/1.1 和 HTTP/2 协议，也同时支持同步（Blocking Mode）和异步模式，支持 WebSocket API 使用中的异步模式。
