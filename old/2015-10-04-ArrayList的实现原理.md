---
layout: post
title: ArrayList 内部用什么实现的？
comments: true
date: 2016-01-04 12:21:35+00:00
categories:
- Android
- Tech
tags:
- java
- ArrayList
- 集合
- design
main-class: 'Java'
color: '#7D669E'
introduction: '回答这样的问题，不要只回答个皮毛，可以再介绍一下 ArrayList 内部是如何实现数组的增加和删除的，因为数组在创建的时候长度是固定的，那么就有个问题我们往 ArrayList 中不断的添加对象，它是如何管理这些数组呢？.ArrayList 内部是用 Object[]实现的。接下来我们分别分析 ArrayList 的构造、add、remove、clear 方法的实现原理。'
---



### ArrayList 内部用什么实现的？

#### 一、构造函数

- 空参构造

   {% highlight java %}
     /**
     * Constructs a new {@code ArrayList} instance with zero initial capacity.
     */
     public ArrayList() {
     array = EmptyArray.OBJECT;
     }
   {% endhighlight %}
   
   
    
- array 是一个 Object[]类型。当我们 new 一个空参构造时系统调用了 EmptyArray.OBJECT 属性，EmptyArray仅仅是一个系统的类库，该类源码如下：

    {% highlight java %}
     public final class EmptyArray {
          private EmptyArray() {}
          public static final boolean[] BOOLEAN = new boolean[0];
          public static final byte[] BYTE = new byte[0];
          public static final char[] CHAR = new char[0];
          public static final double[] DOUBLE = new double[0];
          public static final int[] INT = new int[0];
          public static final Class<?>[] CLASS = new Class[0];
          public static final Object[] OBJECT = new Object[0];
          public static final String[] STRING = new String[0];
          public static final Throwable[] THROWABLE = new Throwable[0];
          public static final StackTraceElement[] STACK_TRACE_ELEMENT = new StackTraceElement[0];
     }
    {% endhighlight %}


 
也就是说当我们 new 一个空参 ArrayList 的时候，系统内部使用了一个 new Object[0]数组。

- 带参构造 1

    {% highlight java %}
    /**
    * Constructs a new instance of {@code ArrayList} with the specified
    * initial capacity.
    **
    @param capacity
    * the initial capacity of this {@code ArrayList}.
    */
    public ArrayList(int capacity) {
    if (capacity < 0) {
        throw new IllegalArgumentException("capacity < 0: " + capacity);
    } 
    array = (capacity == 0 ? EmptyArray.OBJECT : new Object[capacity]);
    }
    {% endhighlight %}

 该构造函数传入一个 int 值，该值作为数组的长度值。如果该值小于 0，则抛出一个运行时异常。如果等于 0，则使用一个空数组，如果大于 0，则创建一个长度为该值的新数组。
 
- 带参构造 2

    {% highlight java %}
        /**
        * Constructs a new instance of {@code ArrayList} containing the elements of
        * the specified collection.
        **
        @param collection
        * the collection of elements to add.
        */
        public ArrayList(Collection<? extends E> collection) {
        if (collection == null) {
            throw new NullPointerException("collection == null");
        } 
        Object[] a = collection.toArray();
        if (a.getClass() != Object[].class) {
            Object[] newArray = new Object[a.length];
            System.arraycopy(a, 0, newArray, 0, a.length);
            a = newArray;
        } 
            array = a;
            size = a.length;
        }
    {% endhighlight %}



- 如果调用构造函数的时候传入了一个 Collection 的子类，那么先判断该集合是否为 null，为 null 则抛出空指针异常。

- 如果不是则将该集合转换为数组 a，然后将该数组赋值为成员变量 array，将该数组的长度作为成员变量 size。这里面它先判断 a.getClass 是否等于 Object[].class，其实一般都是相等的，我也暂时没想明白为什么多加了这个判断，toArray 方法是 Collection 接口定义的， 因此其所有的子类都有这样的方法， list 集合的 toArray 和 Set 集合的 toArray.返回的都是 Object[]数组。

- 这里讲些题外话，其实在看 Java 源码的时候，作者的很多意图都很费人心思，我能知道他的目标是啥，但是不知道他为何这样写。比如对于 ArrayList， array 是他的成员变量，但是每次在方法中使用该成员变量的时候作者都会重新在方法中开辟一个局部变量，然后给局部变量赋值为 array，然后再使用，有人可能说这是为了防止并发修改 array，毕竟 array 是成员变量，大家都可以使用因此需要将 array 变为局部变量，然后再使用，这样的说法并不是都成立的，也许有时候就是老外们写代码的一个习惯而已。


#### 二、add 方法


- add 方法有两个重载，这里只研究最简单的那个。


{% highlight java %}
     /**
     * Adds the specified object at the end of this {@code ArrayList}.
     **
     @param object
     * the object to add.
     * @return always true
     */
     @Override public boolean add(E object) {
        Object[] a = array;
        int s = size;
        if (s == a.length) {
        Object[] newArray = new Object[s +
        (s < (MIN_CAPACITY_INCREMENT / 2) ?
        MIN_CAPACITY_INCREMENT : s >> 1)];
        System.arraycopy(a, 0, newArray, 0, s);
        array = a = newArray;
     } 
        a[s] = object;
        size = s + 1;
        modCount++;
        return true;
     }
{% endhighlight %}


1. 首先将成员变量 array 赋值给局部变量 a，将成员变量 size 赋值给局部变量 s。
2. 判断集合的长度 s 是否等于数组的长度（如果集合的长度已经等于数组的长度了，说明数组已经满了，该重新分 配 新 数 组 了 ） ， 重 新 分 配 数 组 的 时 候 需 要 计 算 新 分 配 内 存 的 空 间 大 小 ， 如 果 当 前 的 长 度 小 于MIN_CAPACITY_INCREMENT/2（这个常量值是 12，除以 2 就是 6，也就是如果当前集合长度小于 6）则分配 12 个长度，如果集合长度大于 6 则分配当前长度 s 的一半长度。这里面用到了三元运算符和位运算，s >> 1，意思就是将s 往右移 1 位，相当于 s=s/2，只不过位运算是效率最高的运算。
3. 将新添加的 object 对象作为数组的 a[s]个元素。
4. 修改集合长度 size 为 s+1
5. modCotun++,该变量是父类中声明的，用于记录集合修改的次数，记录集合修改的次数是为了防止在用迭代器迭代集合时避免并发修改异常，或者说用于判断是否出现并发修改异常的。
6. return true，这个返回值意义不大，因为一直返回 true，除非报了一个运行时异常。



#### 三、remove 方法

- remove 方法有两个重载，我们只研究 remove（int index）方法。

 {% highlight java %}
     /**
     * Removes the object at the specified location from this list.
     * *
     @param index
     * the index of the object to remove.
     * @return the removed object.
     * @throws IndexOutOfBoundsException
     * when {@code location < 0 || location >= size()}
     */
     @Override public E remove(int index) {
           Object[] a = array;
           int s = size;
           if (index >= s) {
           throwIndexOutOfBoundsException(index, s);
        } 
        @SuppressWarnings("unchecked")
        E result = (E) a[index];
        System.arraycopy(a, index + 1, a, index, --s - index);
        a[s] = null; // Prevent memory leak
        size = s;
        modCount++;
        return result;
     }
 {% endhighlight %}
 
 
1. 先将成员变量 array 和 size 赋值给局部变量 a 和 s。
2. 判断形参 index 是否大于等于集合的长度，如果成了则抛出运行时异常
3. 获取数组中脚标为 index 的对象 result，该对象作为方法的返回值
4. 调用 System 的 arraycopy 函数，拷贝原理如下图所示。
5. 接下来就是很重要的一个工作，因为删除了一个元素，而且集合整体向前移动了一位，因此需要将集合最后一个元素设置为 null，否则就可能内存泄露。
6. 重新给成员变量 array 和 size 赋值
7. 记录修改次数
8. 返回删除的元素（让用户再看最后一眼）



#### 四、clear 方法
{% highlight java %}
     /**
     * Removes all elements from this {@code ArrayList}, leaving it empty.
     **
     @see #isEmpty
     * @see #size
     */
     @Override public void clear() {
         if (size != 0) {
         Arrays.fill(array, 0, size, null);
         size = 0;
         modCount++;
     }
{% endhighlight %}


- 如果集合长度不等于 0，则将所有数组的值都设置为 null，然后将成员变量 size 设置为 0 即可，最后让修改记录加 1。