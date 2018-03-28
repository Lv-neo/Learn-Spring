# createSQLQuery is not valid without active transaction 异常
	org.hibernate.HibernateException: createSQLQuery is not valid without active transaction

### 没有声明事务

需要在spring-hibernate.xml中增加事务声明

```
<tx:advice transaction-manager="transactionManager" id="txAdvice">
    <tx:attributes>
        <tx:method name="get*" read-only="true"/>
        <tx:method name="find*" read-only="true"/>
        <tx:method name="count*" read-only="true"/>
        <tx:method name="*" />
    </tx:attributes>
</tx:advice>

<aop:config>
    <aop:pointcut expression="execution(* xx.xx.service.*.*(..))" id="txPointCut"/>
    <aop:advisor advice-ref="txAdvice" pointcut-ref="txPointCut"/>
</aop:config>

```


参考阅读：hibernate线程问题
http://blog.csdn.net/yinjian520/article/details/8666695

