---
title: 无序映射-哈希表
tags:
  - 数据结构
categories:
  - 数据结构
  - 无序集合/映射
keywords: '数据结构, 哈希表'
description: O1的时间复杂度，秒杀树？
cover: 'https://image.imxyu.cn/file/hashtable.png'
abbrlink: aed82065
date: 2021-11-30 15:21:58
---



## 哈希函数

"键" 转换为"索引"的函数

例如一个学号对应一个学生

但是很多情况下这个键不方便转换成索引：

1、这个键可能很长。例如身份证号和人对应，这个身份证号很长，太大了此时就不方便存储

2、**字符串**转换为索引，例如想把一个名字转换成相应的索引（一个字母的话好说，直接减去字符'a'就能转换成相应的int，但是字符串转换就比较麻烦）

3、**浮点数**、日期等

> 哈希表充分体现了空间换时间的思想。哈希函数的设计很重要，"键"通过哈希函数得到的索引越均匀越好。

### 哈希函数的设计

#### 整数

我们主要来讨论对于大整数来说，如果设计哈希函数

例如我们之前说的身份证号，对它取后六位，相当于mod 1000000

110108198512**166666**   ，后6位中包含了生日，这就导致了只能是1-12月，1-31日，会导致数组空间的浪费，并且很容易有生日相同的（产生冲突）

一个简单的解决办法：**模一个素数**（此处超出范围不证明）

<img src="https://image.imxyu.cn/file/image-20211130194106180.png" alt="image-20211130194106180" style="zoom:50%;" />

#### 浮点型

浮点数在计算机中都是32位（float）或者64位（double）来表示，只不过计算机中将其解析成了浮点数

我们可以直接把这32位或者64位当作一个大的整型来处理，同样的模一个素数即可

#### 字符串

可以把字符串想成26进制的整型（如果只包含小写字母）

<img src="https://image.imxyu.cn/file/image-20211130194254986.png" alt="image-20211130194254986" style="zoom:50%;" />

进而可以看成B进制的整型

化简后可得

<img src="https://image.imxyu.cn/file/image-20211130194341173.png" alt="image-20211130194341173" style="zoom:50%;" />

#### 日期类

同样

转成整型处理，并不是唯一的方法

**设计原则**：

1、一致性，如a==b，则hash（a）==hash(b) 。反过来不一定成立，多个值可能对应相同的键

2、高效性，计算高效简便

3、均匀性，哈希表分布均匀

#### java中的hashcode

我们来看一下java中的hashcode，java中的hashcode是对象的方法，因此我们要将基本数据类型转换成包装类型使用

<img src="https://image.imxyu.cn/file/image-20211130194411619.png" alt="image-20211130194411619" style="zoom:50%;" />

java中的hashcode，返回的是一个整型，并不是直接对应了一个数组的索引，真正转换索引的逻辑是在哈希表的内部完成的。因为转换的过程是对一个素数取模的过程，取模后的数最大值是不确定的，也就是说这个素数要根据哈希表的长度来取。

我们再来看看自定义类实现hashcode

```java
public class Student {

    int grade;
    int cls;
    String firstName;
    String lastName;

    Student(int grade, int cls, String firstName, String lastName){
        this.grade = grade;
        this.cls = cls;
        this.firstName = firstName;
        this.lastName = lastName;
    }

    //这里实现的方法就是我们之前说的可以把每个字符串看成整型
    @Override
    public int hashCode(){

        int B = 31; //任意选取一个数作为进制数
        int hash = 0;
        //下面方法就是我们之前整理后的公式
        hash = hash * B + ((Integer)grade).hashCode();
        hash = hash * B + ((Integer)cls).hashCode();
        //这里想表示大小写字母的哈希值是一样的，代表同一个对象
        hash = hash * B + firstName.toLowerCase().hashCode();
        hash = hash * B + lastName.toLowerCase().hashCode();
        return hash;
    }

}
```

此时我们可以用java中的hashset和hashmap.

它们默认都会调用我们自己实现的hashcode方法来存储。

```java
  Student student = new Student(3, 2, "Bobo", "Liu");
        System.out.println(student.hashCode());

        HashSet<Student> set = new HashSet<>();
        set.add(student);

        HashMap<Student, Integer> scores = new HashMap<>();
        scores.put(student, 100);

        Student student2 = new Student(3, 2, "Bobo", "Liu");
        System.out.println(student2.hashCode());
```

> 如果我们不重写hashcode，默认就会用object类的hashcode方法，默认使用的是对象的地址进行哈希转换。因此如果我们想要表示自己的逻辑需要重写hashcode

同样的，也有可能对于多个对象出现hashcode相同的情况。此时就要真正来看两个类是否相等的，此时我们就要覆盖他们的equals方法。

```java
    @Override
    public boolean equals(Object o){ //注意这里覆盖的参数类型是Object，才是重写父类的方法

        if(this == o)
            return true;

        if(o == null)
            return false;

        if(getClass() != o.getClass())
            return false;

        Student another = (Student)o;
        return this.grade == another.grade &&
                this.cls == another.cls &&
                this.firstName.toLowerCase().equals(another.firstName.toLowerCase()) &&
                this.lastName.toLowerCase().equals(another.lastName.toLowerCase());
    }
```

## 哈希冲突

很难保证每一个"键"通过哈希函数的转换对应不同的索引，当两个不同的键通过相同的哈希函数后产生了相同的索引。这种情况就是我们说的哈希冲突。这也是哈希表中最复杂的部分，解决哈希冲突。

> 如果我们有O(n)的空间，我们可以用O（1）的时间完成各项操作(此时没有冲突)。如果我们只有O(1)的空间，我们只能用O(n)的时间完成各项操作，此时就退化成了线性表。（冲突）

### 哈希冲突的处理

链地址法（Seperate Chaining）

哈希表的底层其实就是数组，我们将一个对象转换成相应的hashcode后，对一个素数取模(哈希表的长度)，就得到了在数组中对应的索引。

如果我们计算出的索引是一样的话，我们就插入相同的位置，同时用链表把他们连接起来。（或者使用另一个查找表-树），在java8之后就是这样操作的，当冲突的节点达到一定数量之后就将其转换成TreeMap(底层是红黑树实现)，因为当冲突的**节点很少**时使用**链表**的增删改查是很快的，使用红黑树还要进行相应的旋转操作来维持平衡会变慢。

<img src="https://image.imxyu.cn/file/image-20211130194522133.png" alt="image-20211130194522133" style="zoom:50%;" />

<img src="https://image.imxyu.cn/file/image-20211130194537419.png" alt="image-20211130194537419" style="zoom:50%;" />

### 实现自己的哈希表

```java
/**
 * 因为我们之前说了哈希表的底层就是一个树的数组，因此我们这里可以使用java的TreeMap（红黑树）来实现
 * @param <K>
 * @param <V>
 */
public class HashTable<K, V> {

    private TreeMap<K, V>[] hashtable;
    private int size;
    private int M; //选择的素数

    public HashTable(int M){
        this.M = M;
        size = 0;
        hashtable = new TreeMap[M];
        for(int i = 0 ; i < M ; i ++)
            hashtable[i] = new TreeMap<>();
    }

    public HashTable(){
        this(97); //这里素数默认选择97，取模后大小肯定小于97,也就是说只开辟了97空间的数组，后续会进行优化
    }

    private int hash(K key){
        //因为hashcode可能是有负数的，但是数组索引是没有负数的，因此& 0x7fffffff就是将其转化成正数（符号位变成0，其他不变）
        return (key.hashCode() & 0x7fffffff) % M;
    }

    public int getSize(){
        return size;
    }

    public void add(K key, V value){
        TreeMap<K, V> map = hashtable[hash(key)];//这样我们就能找到这个key对应的hashtable索引中的树
      //   if(hashtable[hash(key)].containsKey(key))
        if(map.containsKey(key))
            map.put(key, value);
        else{
            map.put(key, value);
            size ++;
        }
    }

    public V remove(K key){
        V ret = null;
        TreeMap<K, V> map = hashtable[hash(key)];

        if(map.containsKey(key)){
            ret = map.remove(key);
            size --;
        }
        return ret;
    }

    public void set(K key, V value){
        TreeMap<K, V> map = hashtable[hash(key)];
        if(!map.containsKey(key))
            throw new IllegalArgumentException(key + " doesn't exist!");

        map.put(key, value);
    }

    public boolean contains(K key){
        return hashtable[hash(key)].containsKey(key);
    }

    public V get(K key){
        return hashtable[hash(key)].get(key);
    }
}
```

上述实现的时间复杂度分析：

数组查询的时间为O（1），冲突的平均个数为N/M(M是取的素数,这里考虑的是平均情况这N个元素被哈希函数平均分散到了每个M个地址)

如果每个地址是链表，平均时间复杂度为O(N/M)

红黑树装载的数量是log(N/M)，平均时间复杂度为O（log(N/M)）

> 这里考虑的是最好的情况，当然如果最坏的情况每个元素转换后都在数组的同一个位置。则链表就变成了O（N），红黑树就变成了O（logN）。在安全领域如果知道了这个哈希计算方法，就可以设计一套数据，使得其变成O(N)的复杂度，这被称为哈希碰撞攻击。



### 改进方案

> 取模的过程其实就是让一个数mod取模后变成小于mod的那个数的范围。如果取模的那个数比较小的话，这个范围就很小，很可能造成冲突。

上述我们实现的复杂度都和M有关，如果我们想要达到O1的时间复杂度，我们就不能使用这种静态的数组进行存储，因为大小是不变的。

这里的索引表示的是每个键所对应的地址，因此我们不能像之前的动态数组一样当数组中元素超过了一定的数值再扩容，而是当每个地址承载的元素过多的时候进行扩容。

设置一个容忍度的值N/M>=**upperTol** ,也就是说平均每个地址承载的元素超过了这个值，就要扩容

同样的，设置一个最小的值 N/M<**lowerTol**，平均每个地址承载的元素少过一定程度，即缩容，减少数组占用空间 

```java
public class HashTable<K, V> {

    private static final int upperTol = 10; //设定一个N/M（平均每个数组中的元素）的容忍的最大值
    private static final int lowerTol = 2;//设定一个N/M（平均每个数组中的元素）的容忍的小值
    private static final int initCapacity = 7;

    private TreeMap<K, V>[] hashtable;
    private int size;//size代表一共包含多少元素
    private int M; //M是Map数组的大小，对这个M进行取余

    public HashTable(int M){
        this.M = M;
        size = 0;
        hashtable = new TreeMap[M];
        for(int i = 0 ; i < M ; i ++)
            hashtable[i] = new TreeMap<>();
    }

    public HashTable(){
        this(initCapacity);
    }

    private int hash(K key){
        return (key.hashCode() & 0x7fffffff) % M;
    }

    public int getSize(){
        return size;
    }

    public void add(K key, V value){
        TreeMap<K, V> map = hashtable[hash(key)];
        if(map.containsKey(key))
            map.put(key, value);
        else{
            map.put(key, value);
            size ++;

            if(size >= upperTol * M) //为了size/M 避免整型和浮点型进行转化可能会精度损失，这里改成乘法
                resize(2 * M);
        }
    }

    public V remove(K key){
        V ret = null;
        TreeMap<K, V> map = hashtable[hash(key)];
        if(map.containsKey(key)){
            ret = map.remove(key);
            size --;

            if(size < lowerTol * M && M / 2 >= initCapacity) //缩容，最小缩到我们设置的初始边界，不能让它等于0
                resize(M / 2);
        }
        return ret;
    }

    public void set(K key, V value){
        TreeMap<K, V> map = hashtable[hash(key)];
        if(!map.containsKey(key))
            throw new IllegalArgumentException(key + " doesn't exist!");

        map.put(key, value);
    }

    public boolean contains(K key){
        return hashtable[hash(key)].containsKey(key);
    }

    public V get(K key){
        return hashtable[hash(key)].get(key);
    }

    private void resize(int newM){
        TreeMap<K, V>[] newHashTable = new TreeMap[newM];
        for(int i = 0 ; i < newM ; i ++)
            newHashTable[i] = new TreeMap<>();

        int oldM = M; //这里要用旧的M找到之前的TreeMap数组中每个元素的位置
        this.M = newM;//这里注意当创建了新的容量后，相应的M也要换成新的M。因为hash这个方法要对M取余
        for(int i = 0 ; i < oldM ; i ++){
            TreeMap<K, V> map = hashtable[i];
            for(K key: map.keySet())
                newHashTable[hash(key)].put(key, map.get(key));
        }

        this.hashtable = newHashTable;
    }
}
```

### 改进方案2

我们之前讲过一个整数要mod一个素数才能让取模后的数比较平均，而上面每次*2之后的数可能不是一个素数。因此我们可以设计每次扩容后的M是一个素数，这些数是科学家研究出来的一些素数都是近似两倍的关系



```java
public class HashTable<K extends Comparable<K>, V> {

    private final int[] capacity
            = {53, 97, 193, 389, 769, 1543, 3079, 6151, 12289, 24593,
            49157, 98317, 196613, 393241, 786433, 1572869, 3145739, 6291469,
            12582917, 25165843, 50331653, 100663319, 201326611, 402653189, 805306457, 1610612741};

    private static final int upperTol = 10;
    private static final int lowerTol = 2;
    private int capacityIndex = 0; //初始值变成上面数组中的索引

    private TreeMap<K, V>[] hashtable;
    private int size;
    private int M;

    public HashTable(){
        this.M = capacity[capacityIndex];
        size = 0;
        hashtable = new TreeMap[M];
        for(int i = 0 ; i < M ; i ++)
            hashtable[i] = new TreeMap<>();
    }

    private int hash(K key){
        return (key.hashCode() & 0x7fffffff) % M;
    }

    public int getSize(){
        return size;
    }

    public void add(K key, V value){
        TreeMap<K, V> map = hashtable[hash(key)];
        if(map.containsKey(key))
            map.put(key, value);
        else{
            map.put(key, value);
            size ++;

            if(size >= upperTol * M && capacityIndex + 1 < capacity.length){ //还要进行边界判断
                capacityIndex ++;
                resize(capacity[capacityIndex]);
            }
        }
    }

    public V remove(K key){
        V ret = null;
        TreeMap<K, V> map = hashtable[hash(key)];
        if(map.containsKey(key)){
            ret = map.remove(key);
            size --;

            if(size < lowerTol * M && capacityIndex - 1 >= 0){
                capacityIndex --;
                resize(capacity[capacityIndex]);
            }
        }
        return ret;
    }

    public void set(K key, V value){
        TreeMap<K, V> map = hashtable[hash(key)];
        if(!map.containsKey(key))
            throw new IllegalArgumentException(key + " doesn't exist!");

        map.put(key, value);
    }

    public boolean contains(K key){
        return hashtable[hash(key)].containsKey(key);
    }

    public V get(K key){
        return hashtable[hash(key)].get(key);
    }

    private void resize(int newM){
        TreeMap<K, V>[] newHashTable = new TreeMap[newM];
        for(int i = 0 ; i < newM ; i ++)
            newHashTable[i] = new TreeMap<>();

        int oldM = M;
        this.M = newM;
        for(int i = 0 ; i < oldM ; i ++){
            TreeMap<K, V> map = hashtable[i];
            for(K key: map.keySet())
                newHashTable[hash(key)].put(key, map.get(key));
        }

        this.hashtable = newHashTable;
    }
}
```

> 这里有小问题<K extends Comparable<K> 之前说k是可以不用实现可比较接口的因为可以是无序的，但是下面用到的红黑树中的k却是可比较的，所以会有问题。 因此当我们使用链表的时候是可以不实现比较接口的，但是当链表转成红黑树时是有限制的，如果这个k没有实现比较接口，可能会转不成红黑树。

### 哈希表O1为何还用树？

现在我们实现的哈希表平均是O1的时间复杂度，那我们牺牲了什么呢？和树对比，树是可比较的，维护了他们的顺序，可以很容易的找到树的最大值、最小值等。而在哈希表中是无序的。

集合，映射： 

有序集合，有序映射（平衡树TreeSet、TreeMap）

无序集合，无序映射（哈希表HashSet、HashMap）

### 更多解决冲突的方法

遇到哈希冲突时索引+1，线性探测

<img src="https://image.imxyu.cn/file/image-20211130194612426.png" alt="image-20211130194612426" style="zoom:50%;" />

通常情况下线性探测的效率比较低，当一个位置出现冲突时要不断向后+1的找，会很慢。

平方探测，成平方探测

+1，+4，+9,+16

二次哈希法，再进行一次哈希

再哈希法（遇到冲突用另外一个哈希函数进行哈希）

Colesced Hashing(综合了Seperate Chaining 和Open Chaining)