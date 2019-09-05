### spring bean单例多例

#### singleton

* bean初始化加载是根据`单例模式`加载一次
* `共享`变量
* `线程不安全`
* 用单例，是因为没必要每个请求都新建一个对象，这样子既浪费CPU又浪费内存；
#### prototype
* 每次都是`new`初始化，所以每次都是不一样的对象变量
* 不共享，`线程安全`
* 用多例，是为了`防止并发`问题；即一个请求改变了对象的状态，此时对象又处理另一个请求，而之前请求对对象状态的改变导致了对象对另一个请求做了错误的处理；

#### spring默认使用单例，因为接口注入
* spring的controller/service/dao等一般是线程安全
  * 共享资源不要定义全局的变量，只定义常量和注入的bean，这样基本不会有线程安全问题的出现。
```java
@RestController
@RequestMapping("/jobs")
public class JobsTaskCornPoolController {	
	@Autowired
	private ThreadPoolTaskScheduler threadPoolTaskScheduler;
	//全局变量
	private String cronExpress = "";//这里定义全局容易出现线程安全，通过改成局部变量就基本不会存在问题
  ......
  }
```
