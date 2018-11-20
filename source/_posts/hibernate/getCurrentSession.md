---
title:  getCurrentSession说明
tag: java
---
<!-- toc -->
#  getCurrentSession说明
此设置的作用如下：
What does sessionFactory.getCurrentSession() do? First, you can call it
as many times and anywhere you
like, once you get hold of your SessionFactory (easy thanks to
HibernateUtil). The getCurrentSession()
method always returns the "current" unit of work. Remember that we
switched the configuration option for this
mechanism to "thread" in hibernate.cfg.xml? Hence, the scope of the
current unit of work is the current Java
thread that executes our application. However, this is not the full
truth. A Session begins when it is first
needed, when the first call to getCurrentSession() is made. It is then
bound by Hibernate to the current
thread. When the transaction ends, either committed or rolled back,
Hibernate also unbinds the Session from
the thread and closes it for you. If you call getCurrentSession() again,
you get a new Session and can start a
new unit of work. This thread-bound programming model is the most
popular way of using Hibernate.

意思是说：

sessionFactory.getCurrentSession()可以完成一系列的工作，当调用时，
hibernate将session绑定到当前线程，事务结束后，hibernate
将session从当前线程中释放，并且关闭session。当再次调用getCurrentSession
()时，将得到一个新的session，并重新开始这一系列工作。
这样调用方法如下：

Session session = HibernateUtil.getSessionFactory().getCurrentSession();
session.beginTransaction();
Event theEvent = new Event();
theEvent.setTitle(title);
theEvent.setDate(theDate);
session.save(theEvent);
session.getTransaction().commit();

不需要close session了，但必须在支持事务的上下文中，因为实在事务提交或回滚时关闭的。

 

前提是改值设置为了thread.

1 getCurrentSession创建的session会和绑定到当前线程,而openSession不会。

2 getCurrentSession创建的线程会在事务回滚或事物提交后自动关闭,而openSession必须手动关闭

这里getCurrentSession本地事务(本地事务:jdbc)时 要在配置文件里进行如下设置

    * 如果使用的是本地事务（jdbc事务）
 <property name="hibernate.current_session_context_class">thread</property>
 * 如果使用的是全局事务（jta事务）
 <property name="hibernate.current_session_context_class">jta</property> 

 getCurrentSession () 使用当前的session
openSession()         重新建立一个新的session

在一个应用程序中，如果DAO 层使用Spring 的hibernate 模板，通过Spring 来控制session 的生命周期，则首选getCurrentSession ()。

使用Hibernate的大多数应用程序需要某种形式的“上下文相关的” session，特定的session在整个特定的上下文范围内始终有效。然而，对不同类型的应用程序而言，要为什么是组成这种“上下文”下一个定义通常是困难的；不同的上下文对“当前”这个概念定义了不同的范围。在3.0版本之前，使用Hibernate的程序要么采用自行编写的基于 ThreadLocal的上下文session，要么采用HibernateUtil这样的辅助类，要么采用第三方框架（比如Spring或Pico)，它们提供了基于代理(proxy)或者基于拦截器(interception)的上下文相关session。

从3.0.1版本开始，Hibernate增加了SessionFactory.getCurrentSession()方法。一开始，它假定了采用JTA事务，JTA事务定义了当前session的范围和上下文(scope and context)。Hibernate开发团队坚信，因为有好几个独立的JTA TransactionManager实现稳定可用，不论是否被部署到一个J2EE容器中，大多数(假若不是所有的）应用程序都应该采用JTA事务管理。基于这一点，采用JTA的上下文相关session可以满足你一切需要。

更好的是，从3.1开始，SessionFactory.getCurrentSession()的后台实现是可拔插的。因此，我们引入了新的扩展接口 (org.hibernate.context.CurrentSessionContext)和新的配置参数 (hibernate.current_session_context_class)，以便对什么是“当前session”的范围和上下文(scope and context)的定义进行拔插。

请参阅 org.hibernate.context.CurrentSessionContext接口的Javadoc,那里有关于它的契约的详细讨论。它定义了单一的方法，currentSession()，特定的实现用它来负责跟踪当前的上下文session。Hibernate内置了此接口的两种实现。

org.hibernate.context.JTASessionContext - 当前session根据JTA来跟踪和界定。这和以前的仅支持JTA的方法是完全一样的。详情请参阅Javadoc。

org.hibernate.context.ThreadLocalSessionContext - 当前session通过当前执行的线程来跟踪和界定。详情也请参阅Javadoc。

这两种实现都提供了“每数据库事务对应一个session”的编程模型，也称作每次请求一个session。Hibernate session的起始和终结由数据库事务的生存来控制。假若你采用自行编写代码来管理事务（比如，在纯粹的J2SE,或者 JTA/UserTransaction/BMT），建议你使用Hibernate Transaction API来把底层事务实现从你的代码中隐藏掉。如果你在支持CMT的EJB容器中执行，事务边界是声明式定义的，你不需要在代码中进行任何事务或 session管理操作。请参阅第 11 章 事务和并发一节来阅读更多的内容和示例代码。

hibernate.current_session_context_class 配置参数定义了应该采用哪个org.hibernate.context.CurrentSessionContext实现。注意，为了向下兼容，如果未配置此参数，但是存在org.hibernate.transaction.TransactionManagerLookup的配置，Hibernate会采用org.hibernate.context.JTASessionContext。一般而言，此参数的值指明了要使用的实现类的全名，但那两个内置的实现可以使用简写，即"jta"和"thread"。

1、getCurrentSession()与openSession()的区别？
* 采用getCurrentSession()创建的session会绑定到当前线程中，而采用openSession()
创建的session则不会
* 采用getCurrentSession()创建的session在commit或rollback时会自动关闭，而采用openSession()
创建的session必须手动关闭
2、使用getCurrentSession()需要在hibernate.cfg.xml文件中加入如下配置：
* 如果使用的是本地事务（jdbc事务）
<property name="hibernate.current_session_context_class">thread</property>
* 如果使用的是全局事务（jta事务）

<property name="hibernate.current_session_context_class">jta</property>

利于ThreadLocal模式管理Session
   早在Java1.2推出之时，Java平台中就引入了一个新的支持：java.lang.ThreadLocal，给我们在编写多线程程序
   时提供了一种新的选择。ThreadLocal是什么呢？其实ThreadLocal并非是一个线程的本地实现版本，它并不是一个Thread，
   而是thread local variable(线程局部变量)。也许把它命名为ThreadLocalVar更加合适。线程局部变量(ThreadLocal)
   其实的功用非常简单，就是为每一个使用某变量的线程都提供一个该变量值的副本，是每一个线程都可以独立地改变自己的副本，
   而不会和其它线程的副本冲突。从线程的角度看，就好像每一个线程都完全拥有一个该变量。
   ThreadLocal是如何做到为每一个线程维护变量的副本的呢？其实实现的思路很简单，在ThreadLocal类中有一个Map，
   用于存储每一个线程的变量的副本。比如下面的示例实现(为了简单，没有考虑集合的泛型)：
public class HibernateUtil {

public static final ThreadLocal session =new ThreadLocal();

public static final SessionFactory sessionFactory;
   static {
      try {
        sessionFactory = new Configuration().configure().buildSessionFactory();
      } catch (Throwable ex) {
           throw new ExceptionInInitializerError(ex);
      }    
}

     public static Session currentSession () throws HibernateException {
        Session s = session.get ();
        if(s == null) {
          s = sessionFactory.openSession ();
          session.set(s);
           }
         return s;
       }

    public static void closeSession() throws HibernateException {
           Session s = session.get ();
        if(s != null) {
            s.close();
        }
        session.set(null);
    }
}

openSession() 与 getCurrentSession() 有何不同和关联呢？

 

在 SessionFactory 启动的时候， Hibernate 会根据配置创建相应的 CurrentSessionContext ，在 getCurrentSession() 被调用的时候，实际被执行的方法是 CurrentSessionContext.currentSession() 。在 currentSession() 执行时，如果当前 Session 为空， currentSession 会调用 SessionFactory 的 openSession 。所以 getCurrentSession() 对于 Java EE 来说是更好的获取 Session 的方法。

转载自：http://wenku.baidu.com/link?url=OimQ-55pxZvbAF6EB24aNP4XEJxzfPQEE1Q8gixNlbzPS8coXpuNoekV08ij4aIkQU3P3WC0v0FteFXqgYQuyGHD9lzigQGJuhObHIQ9fH3


