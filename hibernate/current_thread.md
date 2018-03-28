# Hibernate4 No Session found for current thread原因

Hibernate4 与 spring3 集成之后， 如果在取得session 的地方使用了getCurrentSession, 可能会报一个错：“No Session found for current thread”, 这个错误的原因，网上有很多解决办法, 但具体原因的分析，却没有多少, 这里转载一个原理分析:

SessionFactory的getCurrentSession并不能保证在没有当前Session的情况下会自动创建一个新的，这取决于CurrentSessionContext的实现，SessionFactory将调用CurrentSessionContext的currentSession()方法来获得Session。在Spring中，如果我们在没有配置TransactionManager并且没有事先调用SessionFactory.openSession()的情况直接调用getCurrentSession()，那么程序将抛出“No Session found for current thread”异常。如果配置了TranactionManager并且通过@Transactional或者声明的方式配置的事务边界，那么Spring会在开始事务之前通过AOP的方式为当前线程创建Session，此时调用getCurrentSession()将得到正确结果。

然而，产生以上异常的原因在于Spring提供了自己的CurrentSessionContext实现，如果我们不打算使用Spring，而是自己直接从hibernate.cfg.xml创建SessionFactory，并且为在hibernate.cfg.xml
中设置current_session_context_class为thread，也即使用了ThreadLocalSessionContext，那么我们在调用getCurrentSession()时，如果当前线程没有Session存在，则会创建一个绑定到当前线程。

Hibernate在默认情况下会使用JTASessionContext，Spring提供了自己SpringSessionContext，因此我们不用配置current_session_context_class，当Hibernate与Spring集成时，将使用该SessionContext，故此时调用getCurrentSession()的效果完全依赖于SpringSessionContext的实现。

在没有Spring的情况下使用Hibernate，如果没有在hibernate.cfg.xml中配置current_session_context_class，有没有JTA的话，那么程序将抛出"No CurrentSessionContext configured!"异常。此时的解决办法是在hibernate.cfg.xml中将current_session_context_class配置成thread。

在Spring中使用Hibernate，如果我们配置了TransactionManager，那么我们就不应该调用SessionFactory的openSession()来获得Sessioin，因为这样获得的Session并没有被事务管理。

至于解决的办法，可以采用如下方式:
1.  在spring 配置文件中加入

```
<tx:annotation-driven transaction-manager="transactionManager"/>
```

并且在处理业务逻辑的类上采用注解
```
@Service
public class CustomerServiceImpl implements CustomerService {  
    @Transactional
    public void saveCustomer(Customer customer) {
        customerDaoImpl.saveCustomer(customer);
    }
    ...
}
```

另外在 hibernate 的配置文件中，也可以增加这样的配置来避免这个错误:

```
<property name="current_session_context_class">thread</property>
```

转载自
http://www.yihaomen.com/article/java/466.htm


