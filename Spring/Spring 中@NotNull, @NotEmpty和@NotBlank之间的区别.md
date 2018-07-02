[Spring 中@NotNull, @NotEmpty和@NotBlank之间的区别](https://www.cnblogs.com/Terry-Wu/p/8134732.html)
---------------
学习中看到使用了:
```java
@NotNull    
@Size(min = 1,max = 10)
private String name;
```

#### 简述三者区别

* @NotNull://CharSequence, Collection, Map 和 Array 对象不能是 null, 但可以是空集（size = 0）。  
* @NotEmpty://CharSequence, Collection, Map 和 Array 对象不能是 null 并且相关对象的 size 大于 0。  
* @NotBlank://String 不是 null 且去除两端空白字符后的长度（trimmed length）大于 0。 


当一个string对象是null时方法返回true，但是当且仅当它的trimmed length等于零时返回false。即使当string是null时该方法返回true，但是由于@NotBlank还包含了@NotNull，所以@NotBlank要求string不为null。

示例：
```java
String name = null;
@NotNull: false
@NotEmpty: false
@NotBlank: false

String name = "";
@NotNull: true
@NotEmpty: false
@NotBlank: false

String name = " ";
@NotNull: true
@NotEmpty: true
@NotBlank: false

String name = "Great answer!";
@NotNull: true
@NotEmpty: true
@NotBlank: true
```
