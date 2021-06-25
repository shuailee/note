#### Optional 简介

Optional 类是一个可以为null的容器对象。Optional 类主要解决的问题是臭名昭著的空指针异常（NullPointerException） ，本质上，这是一个包含有可选值的包装类，这意味着 Optional 类既可以含有对象也可以为空。
在 Java 8 之前，任何访问对象方法或属性的调用都可能导致 NullPointerException：

```java
//我们并不能开心的一路
String isocode = user.getAddress().getCountry().getIsocode().toUpperCase();
//在这个小示例中，如果我们需要确保不触发异常，就得在访问每一个值之前对其进行明确地检查,这很容易就变得冗长，难以维护：
if (user != null) {
    Address address = user.getAddress();
    if (address != null) {
        Country country = address.getCountry();
        if (country != null) {
            String isocode = country.getIsocode();
            if (isocode != null) {
                isocode = isocode.toUpperCase();
            }
        }
    }
}

```
为了简化这个过程，使用Optional类：
```java
public static String getChampionName(Competition comp) throws IllegalArgumentException {
    return Optional.ofNullable(comp)
            .map(c->c.getResult())
            .map(r->r.getChampion())
            .map(u->u.getName())
        	//orElse 如果存在该值，返回值， 否则返回 other。
        	//orElseThrow 如果存在该值，返回包含的值，否则抛出由 IllegalArgumentException 继承的异常
            .orElseThrow(()->new IllegalArgumentException("The value of param comp isn't available."));
}
```
Optional给了我们一个真正优雅的Java风格的方法来解决null安全问题。虽然没有直接提供一个操作符写起来短，但是代码看起来依然很爽很舒服 

#### Optional 类详解  

- 创建 Optional  实例，这个类型的对象可能包含值，也可能为空。你可以使用同名方法创建一个空的 Optional

```java
public void whenCreateEmptyOptional_thenNull() {
    Optional<User> emptyOpt = Optional.empty();
    emptyOpt.get(); //此处尝试访问 emptyOpt 变量的值会导致 NoSuchElementException，因为包装对象里内容是空的
}
```

- **of() 和 ofNullable()** 方法创建包含值的 Optional,两个方法有值时都会返回一个Optional对象，不同之处在于,如果你把 *null* 值作为参数传递进去，*of()* 方法会抛出 *NullPointerException*： ofNullable不会抛异常，返回一个空的Optional

```java
Optional<User> opt = Optional.ofNullable(user);
```

- **isPresent()** 用来检查Optional是否有值
- **ifPresent(Consumer<? super T> consumer)**   如果值存在则使用该值调用 consumer , 否则不做任何事情。 
- **map(Function<? super T,? extends U> mapper)**  如果有值，则对其执行调用映射函数得到返回值。如果返回值不为 null，则创建包含映射返回值的Optional作为map方法返回值，否则返回空Optional。 
- **orElse**  和  **orElseGet()**    如果存在该值，返回值， 否则返回 other。区别在于前者在有值时始终创建orelse中的空对象，后者则不会创建；

```java
//不建议这么用Optional 因为跟直接判空没什么区别
public static String getName(User u) {
    Optional<User> user = Optional.ofNullable(u);
    //isPresent() 用来检查Optional是否有值,调用get()方法会返回该对象
    if (!user.isPresent())
        return "Unknown";
    return user.get().name;
}

//正确使用Optional的姿势：
public static String getName(User u) {
    return Optional.ofNullable(u)
                    .map(user->user.name)
        			//如果存在该值，返回值， 否则返回 other。
                    .orElse("Unknown");
}

//u 不为空输出name
public static String getName(User u) {
    Optional.ofNullable(u).ifPresent(s-> System.out.println(s.getName()));
}

//
String locale1=Optional.ofNullable(request.getHead()).map(s->{return s.getLocale().replace("_","-");}).orElse("");
```

其他应用案例：

- **map() 对值应用(调用)作为参数的函数，然后将返回的值包装在 Optional 中。**这就使对返回值进行链试调用的操作成为可能 —— 这里的下一环就是 *orElse()*。 

```java
public void whenMap_thenOk() {
    User user = new User("anna@gmail.com", "1234");
    //user不为空，返回user中的emial
    String email = Optional.ofNullable(user)
      .map(u -> u.getEmail()).orElse("default@gmail.com");
}
```

- filter()可以用来检验参数的合法性。 

```java
public void setName(String name) throws IllegalArgumentException {
    this.name = Optional.ofNullable(name)
        			   .filter(User::isNameValid)
                        .orElseThrow(()->new IllegalArgumentException("Invalid username."));
}

 //判断对象中列表元素是否包含某个值
List<String> insuracnelist=onfigUtils.insuranceConfig.getInsuranceMapConfig().getJwsgx();
boolean hasJPJWSGXInsurance = Optional.ofNullable(insuracnelist)
    						.filter(t -> t.eques("jp1")).isPresent();
```

