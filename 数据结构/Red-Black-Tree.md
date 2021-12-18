---
title: 红黑树并不难!
tags:
  - 数据结构
categories:
  - 数据结构
  - 树
keywords: '红黑树，数据结构, 树'
description: 大名鼎鼎的红黑树
cover: 'https://image.imxyu.cn/file/red-black-tree.webp'
abbrlink: 1c341772
date: 2021-11-29 19:11:58
---

## 前言

《算法导论》一书中讲到红黑树中上来就把红黑树的定义抛了出来，让人看了很晦涩，我们来看一下《算法导论》中对红黑树的介绍

<img src="https://image.imxyu.cn/file/image-20211128193804144.png" alt="image-20211128193804144" style="zoom:50%;" />

这个定义乍一眼看上去很生硬，而《算法4》教材中则是先讲解了2-3树这种数据结构，当理解了2-3树和红黑树的等价关系后 回头去看《算法导论》中给出的五条性质是非常自然的。并且《算法4》的作者 Robert Sedewick 也是作为红黑树的发明人之一！ 学习2-3树不仅对理解红黑树有帮助，对于B类树（磁盘存储、文件系统，数据库存储就用到）也是有巨大帮助！

## 2-3树

1. 满足**二分搜索树**的基本性质

2. 节点可以存放一个元素或者两个元素

3. 每个节点有2个或者3个孩子（这也是2-3树名称的由来），对于有两个孩子的节点我们称为**2节点**，对于有3个孩子的节点称为**3节点**(之后我们会一直说2节点和3节点请知悉)

<img src="https://image.imxyu.cn/file/image-20211128194752465.png" alt="image-20211128194752465" style="zoom:50%;" />

 

<img src="https://image.imxyu.cn/file/image-20211128194959618.png" alt="image-20211128194959618" style="zoom:50%;" />

2-3树 是一种**绝对平衡**的树，任意一个节点的高度都是相等的，也就是说根节点到任意一个叶子节点所经过的节点数量一定是相同的。

> 之前说过的二分搜索树、堆、线段树、字典树、并查集、AVL都不是绝对平衡的

下面我们来看一下2-3树是如何维护这种绝对平衡的

### 2-3树的平衡性

2-3树在添加元素的时候，

1. 不能向空的位置添加元素，如果要插入的位置为空则应该先和当前节点进行融合

   <img src="https://image.imxyu.cn/file/image-20211128205938201.png" alt="image-20211128205938201" style="zoom:50%;" />

    

<img src="https://image.imxyu.cn/file/Tree23-1.gif" alt="Tree23-1" style="zoom:50%;" />

2. 对于2-3树不能有4节点（一个节点中有三个元素，4个孩子），**根节点**如果变成了4节点，则向下拆分变成3个2节点

   <img src="https://image.imxyu.cn/file/image-20211128210122106.png" alt="image-20211128210122106" style="zoom:50%;" />

   <img src="https://image.imxyu.cn/file/image-20211128205613305.png" alt="image-20211128205613305" style="zoom:50%;" />

3. 对于叶子节点变成了4节点，向下拆分（此时不是绝对平衡）然后中间的节点和父亲节点作融合 

   如果父亲节点为一个2节点，融合后如下：

   <img src="https://image.imxyu.cn/file/image-20211128210214636.png" alt="image-20211128210214636" style="zoom:50%;" />

   

   如果父亲节点是一个3节点，父亲节点融合后作为一个3节点要进行拆分：

   <img src="https://image.imxyu.cn/file/image-20211128210300126.png" alt="image-20211128210300126" style="zoom:50%;" />

<img src="https://image.imxyu.cn/file/Tree23-3.gif" alt="Tree23-3" style="zoom:50%;" />

一个完整的插入过程：

<img src="https://image.imxyu.cn/file/2-3Tree.gif" alt="2-3Tree" style="zoom:50%;" />

## 红黑树

讲完了2-3树，我们来看红黑树

之前我们讲的树结构每个节点都是只存储一个元素，对于一个元素的操作是很方便的。红黑树也是，只存储一个节点，我们的2-3树和红黑树的不同就在这

<img src="https://image.imxyu.cn/file/image-20211128212321545.png" alt="image-20211128212321545" style="zoom:50%;" />

再来看一个复杂一点的，因为有三个3节点，对应的就有三个红色的边和红色的节点，来表示他们是连在一起的

<img src="https://image.imxyu.cn/file/image-20211128212640958.png" alt="image-20211128212640958" style="zoom:50%;" />

可以方便看成它们是下面这样：

<img src="https://image.imxyu.cn/file/image-20211128212743413.png" alt="image-20211128212743413" style="zoom:50%;" />

因为红黑树也是一种基于二分搜索树的平衡树，我们在之前二叉树中为每个节点添加相应的颜色,并添加节点颜色的判断

基于BST添加的代码如下：

```java
public class RBTree<K extends Comparable<K>, V> {

    private static final boolean RED = true;
    private static final boolean BLACK = false;

    private class Node{
        public K key;
        public V value;
        public Node left, right;
        public boolean color;//为了不必要去记true是RED，BLACK是false，我们在上面添加了两个静态变量

        public Node(K key, V value){
            this.key = key;
            this.value = value;
            left = null;
            right = null;
            color = RED; //因为每次添加一个元素都要先和2节点或者3节点进行融合，因此初始化时都为红色节点
        }
    }
    private Node root;
    private int size;

    public RBTree(){
        root = null;
        size = 0;
    }
      // 判断节点node的颜色，主要是用来判断node为null的情况
    private boolean isRed(Node node){
        if(node == null) //如果node为null，也不存在什么融合，因此定义成黑色节点
            return BLACK;
        return node.color;
    }
    
}
```

此时我们再来看算法导论的定义就很清晰了，这里就不再具体分析了。

<img src="https://image.imxyu.cn/file/image-20211128193804144.png" alt="image-20211128193804144" style="zoom:50%;" />

**红黑树是保持"黑平衡"的二叉树，从根节点开始搜索一直到叶子节点所经历的黑色个数是一样多的**

<img src="https://image.imxyu.cn/file/image-20211129201741097.png" alt="image-20211129201741097" style="zoom:50%;" />

### 红黑树代码实现

添加的节点首先定义是红色的，后面会有融合的过程再去改变他的颜色。

如果是根节点，则把该节点变成黑色的，当插入的节点小于根节点，则直接插入到左侧就好

<img src="https://image.imxyu.cn/file/image-20211129203345168.png" alt="image-20211129203345168" style="zoom:50%;" />

```java
 // 向红黑树中添加新的元素(key, value)
    public void add(K key, V value){
        root = add(root, key, value);
        root.color = BLACK; // 最终根节点为黑色节点
    }
```



因为**红色的节点一定都在左侧**，所以如果插入的节点大于根节点，**此时我们插入到右侧**，我们就要将37节点做一次**左旋转**，旋转的过程和AVL一样

<img src="https://image.imxyu.cn/file/image-20211129203511975.png" alt="image-20211129203511975" style="zoom:50%;" />

为了不失一般性，我们将他们的孩子节点补全

<img src="https://image.imxyu.cn/file/image-20211129203606799.png" alt="image-20211129203606799" style="zoom:50%;" />

并且旋转后，要将他们的颜色进行改变

<img src="https://image.imxyu.cn/file/image-20211129203711715.png" alt="image-20211129203711715" style="zoom:50%;" />

左旋转相应的代码：

```java
	//   node                     x
    //  /   \     左旋转         /  \
    // T1   x   --------->   node   T3
    //     / \              /   \
    //    T2 T3            T1   T2
    private Node leftRotate(Node node){

        Node x = node.right;

        // 左旋转
        node.right = x.left;
        x.left = node;

        x.color = node.color;
        node.color = RED;

        return x;  //返回旋转后树新的根节点，也就是x
    }
```



此处省略了颜色翻转和添加元素的过程，由于过程比较复杂。这里不作讲解，感兴趣的可以自己了解一下。我们只需要知道原理即可，下面贴出完整的代码

```java
public class RBTree<K extends Comparable<K>, V> {

    private static final boolean RED = true;
    private static final boolean BLACK = false;

    private class Node{
        public K key;
        public V value;
        public Node left, right;
        public boolean color;

        public Node(K key, V value){
            this.key = key;
            this.value = value;
            left = null;
            right = null;
            color = RED;
        }
    }

    private Node root;
    private int size;

    public RBTree(){
        root = null;
        size = 0;
    }

    public int getSize(){
        return size;
    }

    public boolean isEmpty(){
        return size == 0;
    }

    // 判断节点node的颜色
    private boolean isRed(Node node){
        if(node == null)
            return BLACK;
        return node.color;
    }

    //   node                     x
    //  /   \     左旋转         /  \
    // T1   x   --------->   node   T3
    //     / \              /   \
    //    T2 T3            T1   T2
    private Node leftRotate(Node node){

        Node x = node.right;

        // 左旋转
        node.right = x.left;
        x.left = node;

        x.color = node.color;
        node.color = RED;

        return x;
    }

    //     node                   x
    //    /   \     右旋转       /  \
    //   x    T2   ------->   y   node
    //  / \                       /  \
    // y  T1                     T1  T2
    private Node rightRotate(Node node){

        Node x = node.left;

        // 右旋转
        node.left = x.right;
        x.right = node;

        x.color = node.color;
        node.color = RED;

        return x;
    }

    // 颜色翻转
    private void flipColors(Node node){

        node.color = RED;
        node.left.color = BLACK;
        node.right.color = BLACK;
    }

    // 向红黑树中添加新的元素(key, value)
    public void add(K key, V value){
        root = add(root, key, value);
        root.color = BLACK; // 最终根节点为黑色节点
    }

    // 向以node为根的红黑树中插入元素(key, value)，递归算法
    // 返回插入新节点后红黑树的根
    private Node add(Node node, K key, V value){

        if(node == null){
            size ++;
            return new Node(key, value); // 默认插入红色节点
        }

        if(key.compareTo(node.key) < 0)
            node.left = add(node.left, key, value);
        else if(key.compareTo(node.key) > 0)
            node.right = add(node.right, key, value);
        else // key.compareTo(node.key) == 0
            node.value = value;

        if (isRed(node.right) && !isRed(node.left))
            node = leftRotate(node);

        if (isRed(node.left) && isRed(node.left.left))
            node = rightRotate(node);

        if (isRed(node.left) && isRed(node.right))
            flipColors(node);

        return node;
    }

    // 返回以node为根节点的二分搜索树中，key所在的节点
    private Node getNode(Node node, K key){

        if(node == null)
            return null;

        if(key.equals(node.key))
            return node;
        else if(key.compareTo(node.key) < 0)
            return getNode(node.left, key);
        else // if(key.compareTo(node.key) > 0)
            return getNode(node.right, key);
    }

    public boolean contains(K key){
        return getNode(root, key) != null;
    }

    public V get(K key){

        Node node = getNode(root, key);
        return node == null ? null : node.value;
    }

    public void set(K key, V newValue){
        Node node = getNode(root, key);
        if(node == null)
            throw new IllegalArgumentException(key + " doesn't exist!");

        node.value = newValue;
    }

    // 返回以node为根的二分搜索树的最小值所在的节点
    private Node minimum(Node node){
        if(node.left == null)
            return node;
        return minimum(node.left);
    }

    // 删除掉以node为根的二分搜索树中的最小节点
    // 返回删除节点后新的二分搜索树的根
    private Node removeMin(Node node){

        if(node.left == null){
            Node rightNode = node.right;
            node.right = null;
            size --;
            return rightNode;
        }

        node.left = removeMin(node.left);
        return node;
    }

    // 从二分搜索树中删除键为key的节点
    public V remove(K key){

        Node node = getNode(root, key);
        if(node != null){
            root = remove(root, key);
            return node.value;
        }
        return null;
    }

    private Node remove(Node node, K key){

        if( node == null )
            return null;

        if( key.compareTo(node.key) < 0 ){
            node.left = remove(node.left , key);
            return node;
        }
        else if(key.compareTo(node.key) > 0 ){
            node.right = remove(node.right, key);
            return node;
        }
        else{   // key.compareTo(node.key) == 0

            // 待删除节点左子树为空的情况
            if(node.left == null){
                Node rightNode = node.right;
                node.right = null;
                size --;
                return rightNode;
            }

            // 待删除节点右子树为空的情况
            if(node.right == null){
                Node leftNode = node.left;
                node.left = null;
                size --;
                return leftNode;
            }

            // 待删除节点左右子树均不为空的情况

            // 找到比待删除节点大的最小节点, 即待删除节点右子树的最小节点
            // 用这个节点顶替待删除节点的位置
            Node successor = minimum(node.right);
            successor.right = removeMin(node.right);
            successor.left = node.left;

            node.left = node.right = null;

            return successor;
        }
    }

    
}
```



## java中的应用

java中的TreeMap和TreeSet中就是基于红黑树的实现

## 红黑树的性能总结



![image-20211129205159044](https://image.imxyu.cn/file/image-20211129205159044.png)
