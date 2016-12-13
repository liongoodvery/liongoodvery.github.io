
### Using a Composable Pointcut

```java
//filename: SimpleBeforeAdvice.java
import java.lang.reflect.Method;
import org.springframework.aop.MethodBeforeAdvice;
public class SimpleBeforeAdvice implements MethodBeforeAdvice {
    @Override
    public void before(Method method, Object[] args, Object target)
            throws Throwable {
        System.out.println("Before method: " + method);
    }
}
```

```java
//filename: SampleBean.java
public class SampleBean {
    public String getName() {
        return "Chris Schaefer";
    }

    public void setName(String name) {
    }

    public int getAge() {
        return 32;
    }
}
```

```java
//filename: ComposablePointcutExample.java
import org.springframework.aop.Advisor;
import org.springframework.aop.ClassFilter;
import org.springframework.aop.framework.ProxyFactory;
import org.springframework.aop.support.ComposablePointcut;
import org.springframework.aop.support.DefaultPointcutAdvisor;
import org.springframework.aop.support.StaticMethodMatcher;

import java.lang.reflect.Method;

public class ComposablePointcutExample {
    public static void main(String[] args) {
        SampleBean target = new SampleBean();

        ComposablePointcut pc = new ComposablePointcut(ClassFilter.TRUE, 
            new GetterMethodMatcher());

        System.out.println("Test 1");
        SampleBean proxy = getProxy(pc, target);
        testInvoke(proxy);

        System.out.println("Test 2");
        pc.union(new SetterMethodMatcher());
        proxy = getProxy(pc, target);
        testInvoke(proxy);

        System.out.println("Test 3");
        pc.intersection(new GetAgeMethodMatcher());
        proxy = getProxy(pc, target);
        testInvoke(proxy);
    }

    private static SampleBean getProxy(ComposablePointcut pc, 
            SampleBean target) {
        Advisor advisor = new DefaultPointcutAdvisor(pc, 
            new SimpleBeforeAdvice());

        ProxyFactory pf = new ProxyFactory();
        pf.setTarget(target);
        pf.addAdvisor(advisor);
        return (SampleBean) pf.getProxy();
    }

    private static void testInvoke(SampleBean proxy) {
        proxy.getAge();
        proxy.getName();
        proxy.setName("Chris Schaefer");
    }

    private static class GetterMethodMatcher extends StaticMethodMatcher {
        @Override
        public boolean matches(Method method, Class<?> cls) {
            return (method.getName().startsWith("get"));
        }
    }

    private static class GetAgeMethodMatcher extends StaticMethodMatcher {
        @Override
        public boolean matches(Method method, Class<?> cls) {
            return "getAge".equals(method.getName());
        }
    }

    private static class SetterMethodMatcher extends StaticMethodMatcher {
        @Override
        public boolean matches(Method method, Class<?> cls) {
            return (method.getName().startsWith("set"));
        }
    }
}
```

```out
Test 1
Before method: public int com.apress.prospring4.ch5.SampleBean.getAge()
Before method: public java.lang.String com.apress.prospring4.ch5.SampleBean.getName()
Test 2
Before method: public int com.apress.prospring4.ch5.SampleBean.getAge()
Before method: public java.lang.String com.apress.prospring4.ch5.SampleBean.getName()
Before method: public void com.apress.prospring4.ch5.SampleBean.setName(java.lang.String)
Test 3
Before method: public int com.apress.prospring4.ch5.SampleBean.getAge()
```


### Getting Started with Introductions


```java
//filename: TargetBean.java
public class TargetBean {
    private String name;
    public void setName(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```


```java
//filename: IsModified.java

public interface IsModified {
    boolean isModified();
}
```


```java
//filename: IsModifiedAdvisor.java
import org.springframework.aop.support.DefaultIntroductionAdvisor;
public class IsModifiedAdvisor extends DefaultIntroductionAdvisor {
    public IsModifiedAdvisor() {
        super(new IsModifiedMixin());
    }
}
```


```java
//filename: IsModifiedMixin.java
import org.aopalliance.intercept.MethodInvocation;
import org.springframework.aop.support.DelegatingIntroductionInterceptor;
import java.lang.reflect.Method;
import java.util.HashMap;
import java.util.Map;

public class IsModifiedMixin extends DelegatingIntroductionInterceptor 
        implements IsModified {
    private boolean isModified = false;

    private Map<Method, Method> methodCache = new HashMap<Method, Method>();

    @Override
    public boolean isModified() {
        return isModified;
    }

    @Override
    public Object invoke(MethodInvocation invocation) throws Throwable {
        if (!isModified) {
            if ((invocation.getMethod().getName().startsWith("set"))
                && (invocation.getArguments().length == 1)) {

                Method getter = getGetter(invocation.getMethod());

                if (getter != null) {
                    Object newVal = invocation.getArguments()[0];
                    Object oldVal = getter.invoke(invocation.getThis(), null);

                    if((newVal == null) && (oldVal == null)) {
                        isModified = false;
                    } else if((newVal == null) && (oldVal != null)) {
                        isModified = true;
                    } else if((newVal != null) && (oldVal == null)) {
                        isModified = true;
                    } else {
                        isModified = (!newVal.equals(oldVal));
                    }
                }
            }
        }

        return super.invoke(invocation);
    }

    private Method getGetter(Method setter) {
        Method getter = null;

        getter = (Method) methodCache.get(setter);

        if (getter != null) {
            return getter;
        }

        String getterName = setter.getName().replaceFirst("set", "get");

        try {
            getter = setter.getDeclaringClass().getMethod(getterName, null);

            synchronized (methodCache) {
                methodCache.put(setter, getter);
            }

            return getter;
        } catch (NoSuchMethodException ex) {
            return null;
        }
    }
}
```

```java
//filename: IntroductionExample.java
import org.springframework.aop.IntroductionAdvisor;
import org.springframework.aop.framework.ProxyFactory;
public class IntroductionExample {
    public static void main(String[] args) {
        TargetBean target = new TargetBean();
        target.setName("Chris Schaefer");

        IntroductionAdvisor advisor = new IsModifiedAdvisor();

        ProxyFactory pf = new ProxyFactory();
        pf.setTarget(target);
        pf.addAdvisor(advisor);
        pf.setOptimize(true);

        TargetBean proxy = (TargetBean) pf.getProxy();
        IsModified proxyInterface = (IsModified)proxy;

        System.out.println("Is TargetBean?: " + (proxy instanceof TargetBean));
        System.out.println("Is IsModified?: " + (proxy instanceof IsModified));

        System.out.println("Has been modified?: " + 
            proxyInterface.isModified());

        proxy.setName("Chris Schaefer");

        System.out.println("Has been modified?: " + 
            proxyInterface.isModified());

        proxy.setName("Clarence Ho");

        System.out.println("Has been modified?: " + 
            proxyInterface.isModified());
    }
}
```

```out
Is TargetBean?: true
Is IsModified?: true
Has been modified?: false
Has been modified?: false
Has been modified?: true
Has been modified?: true
```


## Framework Services for AOP

### Configuring AOP Declaratively
- UsingProxyFactoryBean
- Using the Spring aop namespace:
- Using @AspectJ-style annotations

###　Using　ProxyFactoryBean



```java
//filename: MyDependency.java
public class MyDependency {
    public void foo() {
        System.out.println("foo()");
    }

    public void bar() {
        System.out.println("bar()");
    }
}
```


```java
//filename: MyBean.java
public class MyBean {
    private MyDependency dep;
    public void execute() {
        dep.foo();
        dep.bar();
    }

    public void setDep(MyDependency dep) {
        this.dep = dep;
    }
}
```


```java
//filename: ProxyFactoryBeanExample.java
import org.springframework.context.support.GenericXmlApplicationContext;
public class ProxyFactoryBeanExample {
    public static void main(String[] args) {
        GenericXmlApplicationContext ctx = new GenericXmlApplicationContext();
        ctx.load("classpath:META-INF/spring/app-context-xml.xml");
        ctx.refresh();

        MyBean bean1 = (MyBean)ctx.getBean("myBean1");
        MyBean bean2 = (MyBean)ctx.getBean("myBean2");

        System.out.println("Bean 1");
        bean1.execute();

        System.out.println("\nBean 2");
        bean2.execute();
    }
}
```



```java
//filename: MyAdvice.java
import org.springframework.aop.MethodBeforeAdvice;
import java.lang.reflect.Method;
public class MyAdvice implements MethodBeforeAdvice {
    @Override
    public void before(Method method, Object[] args, Object target)
            throws Throwable {
        System.out.println("Executing: " + method);
    }
}
```


```xml
<!--filename:app-context-xml.xml-->
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
           http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="myBean1" class="com.apress.prospring4.ch5.MyBean">
        <property name="dep">
            <ref bean="myDependency1"/>
        </property>
    </bean>

    <bean id="myBean2" class="com.apress.prospring4.ch5.MyBean">
        <property name="dep">
            <ref bean="myDependency2"/>
        </property>
    </bean>

    <bean id="myDependencyTarget" class="com.apress.prospring4.ch5.MyDependency"/>

    <bean id="myDependency1" class="org.springframework.aop.framework.ProxyFactoryBean">
        <property name="target">
            <ref bean="myDependencyTarget"/>
        </property>
        <property name="interceptorNames">
            <list>
                <value>advice</value>
            </list>
        </property>
    </bean>

    <bean id="myDependency2" class="org.springframework.aop.framework.ProxyFactoryBean">
        <property name="target">
            <ref bean="myDependencyTarget"/>
        </property>
        <property name="interceptorNames">
            <list>
                <value>advisor</value>
            </list>
        </property>
    </bean>

    <bean id="advice" class="com.apress.prospring4.ch5.MyAdvice"/>

    <bean id="advisor" class="org.springframework.aop.support.DefaultPointcutAdvisor">
        <property name="advice">
            <ref bean="advice"/>
        </property>
        <property name="pointcut">
            <bean class="org.springframework.aop.aspectj.AspectJExpressionPointcut">
                <property name="expression">
                    <value>execution(* foo*(..))</value>
                </property>
            </bean>
        </property>
    </bean>
</beans>
```

```out
Bean 1
Executing: public void com.apress.prospring4.ch5.MyDependency.foo()
foo()
Executing: public void com.apress.prospring4.ch5.MyDependency.bar()
bar()

Bean 2
Executing: public void com.apress.prospring4.ch5.MyDependency.foo()
foo()
bar()
```





