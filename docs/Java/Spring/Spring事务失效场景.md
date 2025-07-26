# Spring 事务失效场景

Spring 的事务是通过 AOP (面向切面编程) 实现的，它在方法执行前后插入了事务管理逻辑。

## 1. `@Transactional`注解应用在非 public 方法上

问题所在：Spring 的 AOP 默认是基于 JDK 的动态代理(针对接口)或 CGLIB 代理(针对类)来实现的。无论是哪种代理，它们都是在方法调用时创建代理对象，并拦截`public`方法。如果`@Transactional`注解加在`private`、`protected`或包级私有(default)方法上，代理就无法拦截到这些方法的调用，事务自然也就不会生效。

解决方案：始终将`@Transactional`注解应用于`public`方法上。

## 2. 同一个类中的方法内部调用(自调用)

问题所在：假设`UserService`类中有一个`methodA()`方法，它被`@Transactional`标记，然后在`methodA()`内部又直接调用了同一个类中的另一个`methodB()`方法。如果`methodB()`也被`@Transactional`标记，那么`methodB()`上的事务注解会失效。

这是因为，当你从`methodA()`内部调用`methodB()`时，你调用的是**原始对象**的`methodB()`方法，而不是 Spring 生成的**代理对象**的`methodB()`方法。代理对象只有在外部调用时才会被拦截，内部调用绕过了代理，自然也就绕过了事务切面。

解决方案：

- 注入自身代理对象：将当前 Bean 注入到自身，然后通过注入的代理对象来调用内部方法。
- 拆分业务逻辑：将被内部调用的方法提取到一个独立的 Service 类中，这样外部 Service 调用它时，自然会走代理，这是更推荐的解耦方式。

## 3. 事务传播行为配置不当

事务传播行为(Propagation)定义了事务方法之间的关系，如果不理解或者配置错误，也可能导致事务失效或行为异常。

问题所在：最常见的是`Propagation.NOT_SUPPORTED`或`Propagation.NEVER`。

- `NOT_SUPPORTED`：如果当前存在事务，则将其挂起。执行当前方法时，不参与事务。
- `NEVER`：如果当前存在事务，则抛出异常。
- `REQUIRES_NEW`：总是开启一个新事务，并挂起当前事务(如果存在)。虽然它会开启新事务，但如果你期望他加入外部事务，那显然就失效了。

解决方案：仔细理解并选择正确的事务传播行为。默认的`propagation.REQUIRED`通常是大多数场景的正确选择，它表示如果当前存在事务，就加入该事务；如果不存在，就创建一个新事务。

## 4. 数据库不支持事务或未正确配置事务管理器

Spring 的事务管理器是基于底层数据库事务的。

问题所在：

- 你使用的数据库引擎不支持事务(例如，MYSQL 的 MYISAM 引擎就不支持事务，而 InnoDB 支持)。
- 没有为 Spring 配置正确的事务管理器(`PlatformTransactionMapper`)。
- 数据源配置有问题，或者数据库连接池不支持事务。

解决方案:

- 确保数据库支持事务，并使用支持事务的存储引擎。
- 在 Spring 配置中正确定义`PlatformTransactionMapper`，并确保它关联到正确的数据源。
- 检查数据库连接池的配置，确保其符合事务要求。

## 5. 方法没有被 Spring 管理(不是 Bean)

如果类没有被 Spring IoC 容器管理，Spring 就无法为其生成代理，自然就没有事务。

问题所在：类没有使用`@Componet`、`@Service`、`@Repository`等注解，或者没有通过 XML 配置将其声明为 Spring Bean。

解决方案：确保类是被 Spring 管理的 Bean。

## 6. 异常类型不匹配或异常被捕获

Spring 默认只对 运行时异常(RuntimeException)及其子类进行事务回滚。对于受检异常(Checked Exception)，Spring 默认是不回滚的。此外，如果在事务方法内部捕获了异常并吞掉，Spring 也无法感知到异常的发生。

问题所在：

- 方法抛出了一个受检异常(如 IOException)，但你期望它回滚。
- 在事务方法内部使用了`try-catch`块捕获了异常，并且没有再次抛出或者抛出一个`RuntimeException`。

解决方案：

- 明确指定回滚规则：在`@Transactional`注解中通过`rollbackFor`属性指定哪些异常需要回滚。
- 不要吞掉异常：如果在`try-catch`块中捕获了异常，并且希望事务回滚，必须在`catch`中重新抛出异常(最好是`RuntimeException`或自定义的`RuntimeException`子类)，或者至少抛出`rollbackFor`指定的异常类型。

## 7. 事务模式不匹配(proxy vs. AspectJ)

Spring 事务的默认实现是基于代理的。但如果使用了 AspectJ 编译时织入的方式，情况会不同。

问题所在：

- 默认情况下，Spring 事务基于代理(JDK 动态代理或 CGLIB 代理)。
- 如果使用了 AspectJ 编译时织入，它会直接修改字节码，所以内部调用等问题可能不会出现。但如果混合使用，或者期望一种机制却用了另一种，就可能混淆。

解决方案：确保理解并一致地使用 Spring 事务的底层实现方式。对于大多数应用，使用默认的代理模式就足够了，并且要注意前面提到的代理限制。

## 总结

事务失效的场景，说到底就是 Spring 事务切面没有按照期望的方式生效，记住以下几点：

1. Public 方法：`@transactional`只能在`public`方法上生效。
2. 外部调用：只有通过 Spring 代理对象调用的方法，其事务注解才会生效。内部调用会绕过代理。
3. 异常类型：默认只对运行时异常回滚。受检异常需要显式配置`rollbackFor`。
4. 不要吞异常：`catch`块中的异常如果没有再次抛出，事务就不会回滚。
5. 正确配置：确保事务管理器和数据库都配置正确且支持事务。
