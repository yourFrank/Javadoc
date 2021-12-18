---
title: 一颗自平衡的树-AVL树
tags:
  - 数据结构
categories:
  - 数据结构
  - 树
keywords: 'AVL，数据结构, 树'
description: ALV树是如何实现的自平衡？
cover: 'https://image.imxyu.cn/file/AVL%E6%A0%91.webp'
abbrlink: 973716ad
date: 2021-11-28 15:21:58
---

## 前言

之前讲的普通的二分搜索树BST有些情况会退化成链表：eg:比如[1,2,3,4,5,6]的顺序插入 。 相应的时间复杂度也会退化成链表O(n)

<img src="https://image.imxyu.cn/file/image-20211126184435788.png" alt="image-20211126184435788" style="zoom:50%;" />


AVL树是最早的可以自平衡的二叉树，那么什么是平衡二叉树呢？

## 平衡二叉树

平衡二叉树的增删改查的时间复杂度可以保持O(logn)，因此我们要让二叉树尽可能保持平衡

### 满二叉树

定义：除了叶子节点，其他节点都有左右两个子树（如果一个二叉树的层数为K，且结点总数是(2^k) -1，则它就是满二叉树。）

满二叉树的情况下是可以让整个树的高度达到一个最低的状态。**满二叉树是平衡二叉树。**

<img src="https://image.imxyu.cn/file/image-20211126185212581.png" alt="image-20211126185212581" style="zoom:50%;" />

### 完全二叉树

简单来说：将所有的元素从左向右依次一层一层的铺开，得到的就是完全二叉树。空缺的部分肯定是在整个树的右下部分，整棵树的叶子节点的最大深度值和最小深度值不会超过1（例如图中叶子结点最大深度是最后一层，叶子结点的最小深度在倒数第二层22和13这两个节点），也就是说所有的叶子结点要么在树的最后一层，要么就在树的倒数第二层。 **完全二叉树是一个平衡二叉树。**

<img src="https://image.imxyu.cn/file/image-20211126185626985.png" alt="image-20211126185626985" style="zoom:50%;" />

之前讲堆的时候说到，堆就是一个完全二叉树

![img](https://image.imxyu.cn/file/1590962-20190318210706226-1501863648.png)

### 线段树

线段树虽然不是一个完全二叉树，但是对于线段树来说：叶子结点要么在最后一层，要么在倒数第二层，所有**叶子节点**的深度不会超过1（要么在最后一层，要么在倒数第二层）。**因此线段树是一个平衡二叉树。**



<img src="https://image.imxyu.cn/file/image-20211126190116085.png" alt="image-20211126190116085" style="zoom:50%;" />

### AVL树

定义：对于任意一个节点，左子树和右子树的高度差不能超过1。 **AVL树是一个平衡的二分搜索树**。

> 注意：之前的堆和线段树是叶子节点的高度差不能超过1，对于AVL树来说是任意节点的左右子树高度差不超过1。是有区别的

如下图，AVL树是平衡的

<img src="https://image.imxyu.cn/file/image-20211126190851205.png" alt="image-20211126190851205" style="zoom:50%;" />

此时如果我们用之前二分搜索树的实现添加了2和7这两个节点，就会变成下图这样，此时就不是一个AVL树了(节点8的左子树高度为3，右子树为11、 节点12的左子树为4，右子树高度为2 。 不满足任意节点子树高度差为1的性质)

<img src="https://image.imxyu.cn/file/image-20211126191400153.png" alt="image-20211126191400153" style="zoom:50%;" />



## AVL树

这一节我们来学习一下如何构造AVL树？在二分搜索树的基础上实现自平衡

### 标注高度

首先我们把每个节点都标注上高度。叶子节点的高度就是1，依次向上记录

<img src="https://image.imxyu.cn/file/image-20211126192050720.png" alt="image-20211126192050720" style="zoom:50%;" />

### 平衡因子

平衡因子就是左右子树的高度差

 其中蓝色标注的就是每个节点的平衡因子，黑色标注的是每个节点的高度。



<img src="https://image.imxyu.cn/file/image-20211126192359543.png" alt="image-20211126192359543" style="zoom:50%;" />

通过平衡因子我们也可以看到，8和12两个节点的平衡因子为2，不满足定义



（基于之前的二分搜索树）添加高度和平衡因子后修改代码如下：

```java
//这里选取包含k,v键值对版本的二分搜索树来改造 
public class AVLTree<K extends Comparable<K>, V> {

    private class Node {
        public K key;
        public V value;
        public Node left, right;
        public int height; //添加了高度的记录

        public Node(K key, V value) {
            this.key = key;
            this.value = value;
            left = null;
            right = null;
            height = 1;  //默认为1，因为二分搜索树添加一个元素最后的位置肯定是叶子节点，叶子节点的高度为1
        }
    }
    public AVLTree() {
        root = null;
        size = 0;
    }
    private Node root;
    private int size;
    public int getSize() {
        return size;
    }
    public boolean isEmpty() {
        return size == 0;
    }

    // 获得节点node的高度
    private int getHeight(Node node) {
        if (node == null)
            return 0;
        return node.height;
    }

    // 添加平衡因子方法：获得节点node的平衡因子。这里忽略正负
    private int getBalanceFactor(Node node) {
        if (node == null)
            return 0;
        return getHeight(node.left) - getHeight(node.right);
    }

    // 向二分搜索树中添加新的元素(key, value)
    public void add(K key, V value) {
        root = add(root, key, value); //此处调用下面递归的方法添加
    }

    // 向以node为根的二分搜索树中插入元素(key, value)，递归算法
    // 返回插入新节点后二分搜索树的根
    private Node add(Node node, K key, V value) {

        if (node == null) {
            size++;
            return new Node(key, value); //新添加的节点的高度值默认为1
        }

        if (key.compareTo(node.key) < 0)
            node.left = add(node.left, key, value);  //在左子树中添加
        else if (key.compareTo(node.key) > 0)
            node.right = add(node.right, key, value); //在右子树中添加
        else // key.compareTo(node.key) == 0
            node.value = value;

        // 无论是在左子树还是右子树添加，当前节点都要更新height值，左右子树高度最大值+1的方式
        node.height = 1 + Math.max(getHeight(node.left), getHeight(node.right));

        // 计算平衡因子
        int balanceFactor = getBalanceFactor(node);
        if (Math.abs(balanceFactor) > 1) //如果平衡因子的绝对值大于1说明不平衡了
            System.out.println("unbalanced : " + balanceFactor); //此处只是添加了一条语句表示不平衡了，没有进行自平衡操作

        return node;
    }

}
```

> 因为AVL树既是一个二分搜索树，也是一个平衡二叉树，为了检查我们的实现的树是否满足这两条性质。我们添加两个方法来帮我们检查：

1、 判断是一个二分搜索树

对于二分搜索树，中序遍历一定是按照节点大小的顺序输出的。我们可以采用中序遍历来帮我们判断

```java
// 判断该二叉树是否是一棵二分搜索树
    public boolean isBST(){

        ArrayList<K> keys = new ArrayList<>();
        inOrder(root, keys);
        for(int i = 1 ; i < keys.size() ; i ++)
            if(keys.get(i - 1).compareTo(keys.get(i)) > 0)//判断相邻节点的大小，如果不是按照顺序的返回false
                return false;
        return true;
    }

	//中序遍历
    private void inOrder(Node node, ArrayList<K> keys){

        if(node == null)
            return;

        inOrder(node.left, keys);
        keys.add(node.key);
        inOrder(node.right, keys);
    }

```

2、判断是一个平衡二叉树

对于平衡二叉树，每个节点的平衡因子都不超过1，同样采取递归的方式

```java
  // 判断该二叉树是否是一棵平衡二叉树
    public boolean isBalanced(){
        return isBalanced(root);
    }

    // 判断以Node为根的二叉树是否是一棵平衡二叉树，递归算法
    private boolean isBalanced(Node node){

        if(node == null) //递归base判断:空节点不存在左右子树，因此左右子树高度是0，返回true
            return true;

        int balanceFactor = getBalanceFactor(node); //获取当前节点的平衡因子
        if(Math.abs(balanceFactor) > 1)
            return false;
        //因为平衡二叉树的性质是要求每个节点的高度差都小于等于1，当前节点满足后继续递归去看左右两个孩子
        return isBalanced(node.left) && isBalanced(node.right);
    }
```

### 维护平衡性

对于BST当我们添加了一个节点后，每个节点的高度就会变化，相应的平衡因子也会发生改变。因此我们要从下向上维护平衡性（回溯的过程）

#### 情况一：左侧的左侧插入元素

**什么时候维护平衡？**

第一种情况是插入的元素在不平衡节点的左侧的左侧，导致AVL树不平衡

（例如下面图2中的节点12和节点8已经是不平衡的节点了，并且这里是左子树的高度比右子树的高度大1（注意这里说的节点不平衡是该节点的左右两个子树高度不一样，并不是说整个树不平衡了，AVL树不平衡的定义是任意节点左右子树高度**大于**1，注意区分），此时新插入的元素5/元素2在节点12/节点8的左侧的左侧）

> 很多小伙伴会联想到当插入的元素在左侧的右侧，同样也不平衡。别急，下面我们会一 一介绍



图1：<img src="https://image.imxyu.cn/file/balance.gif" alt="balance" style="zoom:30%;" />图2： <img src="https://image.imxyu.cn/file/image-20211128100038616.png" alt="image-20211128100038616" style="zoom:30%;" />



代码如下：

```java
// 向以node为根的二分搜索树中插入元素(key, value)，递归算法
    // 返回插入新节点后二分搜索树的根
    private Node add(Node node, K key, V value){

        if(node == null){
            size ++;
            return new Node(key, value);
        }

        if(key.compareTo(node.key) < 0)
            node.left = add(node.left, key, value);
        else if(key.compareTo(node.key) > 0)
            node.right = add(node.right, key, value);
        else // key.compareTo(node.key) == 0
            node.value = value;

        // 更新height
        node.height = 1 + Math.max(getHeight(node.left), getHeight(node.right));

        // 计算平衡因子
        int balanceFactor = getBalanceFactor(node);
//        if(Math.abs(balanceFactor) > 1)
//            System.out.println("unbalanced : " + balanceFactor);

        // 平衡维护
        if (balanceFactor > 1 && getBalanceFactor(node.left) >= 0) //平衡因子左子树比右子树的差>1（此时该节点不平衡了），并且左子树的平衡因子也是>=0
              return rightRotate(node); // 实现平衡维护，下一小节进行具体实现：）

        return node;
    }
```

> 注意：代码中: balanceFactor > 1说明该节点不平衡了，因为是>1是正数，说明是左子树的高度比右子树高度大1，对应了上图2中节点8的状态，getBalanceFactor(node.left) >= 0 说明节点的左子树的左子树高度>=左子树的右子树高度，并且该节点的左侧对应了上右边图中节点5的状态。

**如何维护平衡？**

旋转前此时y节点不平衡（左侧高度为3，右侧高度为1）：

旋转前：<img src="https://image.imxyu.cn/file/image-20211128103114304.png" alt="image-20211128103114304" style="zoom:30%;" />  右旋转后：<img src="https://image.imxyu.cn/file/image-20211128103154948.png" alt="image-20211128103154948" style="zoom:30%;" />



具体旋转的图解：

<img src="https://image.imxyu.cn/file/balance2.gif" alt="balance2" style="zoom:50%;" />

这样执行完旋转操作后，仍然满足了二分搜索树的性质（节点的大小关系没变），同时也维护了平衡性 

代码如下:

```java
 // 对节点y进行向右旋转操作，返回旋转后新的根节点x
    //        y                              x
    //       / \                           /   \
    //      x   T4     向右旋转 (y)        z     y
    //     / \       - - - - - - - ->    / \   / \
    //    z   T3                       T1  T2 T3 T4
    //   / \
    // T1   T2
    private Node rightRotate(Node y) {
        Node x = y.left;
        Node T3 = x.right;

        // 向右旋转过程
        x.right = y;
        y.left = T3;

        // 更新height，因为T1,T2,T3,T4仍然在叶子节点，所以高度还是1,不需要维护
        y.height = Math.max(getHeight(y.left), getHeight(y.right)) + 1; //先维护y节点的，在维护x节点的高度
        x.height = Math.max(getHeight(x.left), getHeight(x.right)) + 1;

        return x;
    }
```

> 记忆点：对某个点进行右旋转，就是将这个点旋转到右侧

#### 情况二：右侧的右侧插入元素

第二种情况是插入的元素在不平衡节点的右侧的右侧，右子树的高度比左子树的高度>1 

旋转前：<img src="https://image.imxyu.cn/file/image-20211128110814887.png" alt="image-20211128110814887" style="zoom:30%;" /> 左旋转后：<img src="https://image.imxyu.cn/file/image-20211128111056190.png" alt="image-20211128111056190" style="zoom:30%;" />



相应左旋转的代码也容易写出来：

```java
 // 对节点y进行向左旋转操作，返回旋转后新的根节点x
    //    y                             x
    //  /  \                          /   \
    // T1   x      向左旋转 (y)       y     z
    //     / \   - - - - - - - ->   / \   / \
    //   T2  z                     T1 T2 T3 T4
    //      / \
    //     T3 T4
    private Node leftRotate(Node y) {
        Node x = y.right;
        Node T2 = x.left;

        // 向左旋转过程
        x.left = y;
        y.right = T2;

        // 更新height
        y.height = Math.max(getHeight(y.left), getHeight(y.right)) + 1;
        x.height = Math.max(getHeight(x.left), getHeight(x.right)) + 1;

        return x;
    }
```

> 记忆点：对某个点进行左旋转，就是将该点旋转到左侧

add() 部分代码中的平衡维护相应实现

```java
// 平衡维护
//左侧的左侧插入
if (balanceFactor > 1 && getBalanceFactor(node.left) >= 0)  
    return rightRotate(node);
//右侧的右侧插入
if (balanceFactor < -1 && getBalanceFactor(node.right) <= 0)
    return leftRotate(node);
```

> balanceFactor < -1 说明右侧的高度比左侧大于1。getBalanceFactor(node.right) <= 0 说明右孩子的右子树高度>=右孩子的左子树高度。

#### 情况三： 左侧的右侧插入元素

在左侧的右侧插入元素，此时就不能直接右旋转，因为对于12和10都比8大，直接对12右旋转完就会变成链表的情况

<img src="https://image.imxyu.cn/file/image-20211128130334826.png" alt="image-20211128130334826" style="zoom:50%;" />

 我们可以把之前的情况一和二命名为LL和RR

<img src="https://image.imxyu.cn/file/image-20211128130754629.png" alt="image-20211128130754629" style="zoom:50%;" />



情况三我们可以命名为LR

1、首先对于LR我们可以对x节点**左旋转**，此时就会变成之前LL的情况

<img src="https://image.imxyu.cn/file/image-20211128131008312.png" alt="image-20211128131008312" style="zoom:40%;" />    <img src="https://image.imxyu.cn/file/image-20211128131234864.png" alt="image-20211128131234864" style="zoom:40%;" />



2、之后我们就可以采用之前情况一LL的情况对y节点进行**右旋转**即可

<img src="https://image.imxyu.cn/file/image-20211128103154948.png" alt="image-20211128103154948" style="zoom:50%;" />



#### 情况四： 右侧的左侧插入元素

同理还有一种情况：在右侧的左侧插入元素

1、首先我们对x节点进行**右旋转**就变成了之前RR的情况



<img src="https://image.imxyu.cn/file/image-20211128132759253.png" alt="image-20211128132759253" style="zoom:40%;" /><img src="https://image.imxyu.cn/file/image-20211128132831254.png" alt="image-20211128132831254" style="zoom:40%;" />

2、之后我们就可以采用之前情况二RR对 y节点进行**左旋转**就行

<img src="https://image.imxyu.cn/file/image-20211128111056190.png" alt="image-20211128111056190" style="zoom:50%;" />

 

 #### 添加元素维护平衡代码

```java
// 向以node为根的二分搜索树中插入元素(key, value)，递归算法
// 返回插入新节点后二分搜索树的根
private Node add(Node node, K key, V value){

    if(node == null){
        size ++;
        return new Node(key, value);
    }

    if(key.compareTo(node.key) < 0)
        node.left = add(node.left, key, value);
    else if(key.compareTo(node.key) > 0)
        node.right = add(node.right, key, value);
    else // key.compareTo(node.key) == 0
        node.value = value;

    // 更新height
    node.height = 1 + Math.max(getHeight(node.left), getHeight(node.right));

    // 计算平衡因子
    int balanceFactor = getBalanceFactor(node);

    // 平衡维护

    //LL
    if (balanceFactor > 1 && getBalanceFactor(node.left) >= 0)
        return rightRotate(node);

    //RR
    if (balanceFactor < -1 && getBalanceFactor(node.right) <= 0)
        return leftRotate(node);

    //LR
    if (balanceFactor > 1 && getBalanceFactor(node.left) < 0) {
        node.left = leftRotate(node.left); //1、先进行左旋转，返回的根节点链接到node.left。此时就变成了LL的情况
        return rightRotate(node); //2、再执行和LL相同操作，进行rightRotate
    }
    //RL
    if (balanceFactor < -1 && getBalanceFactor(node.right) > 0) {
        node.right = rightRotate(node.right);//先进行右旋转，返回的根节点链接到node.right。此时就变成了RR的情况
        return leftRotate(node);// 2、再执行和RR相同操作，leftRotate
    }

    return node;
}
```

#### 测试对比AVL和BST

经过了上述的改进，我们就将BST改进成了AVL树实现了自平衡，对于任意大小的元素顺序添加后都能达到O（logn）的复杂度

 ```java
 public class Main {
 
     public static void main(String[] args) {
 
         System.out.println("Pride and Prejudice");
 
         ArrayList<String> words = new ArrayList<>();
         if(FileOperation.readFile("pride-and-prejudice.txt", words)) {
             System.out.println("Total words: " + words.size());
 
             // Collections.sort(words);  //如果对单词进行排序后，BST就会退化成链表，AVL优势更加明显
 
             // Test BST
             long startTime = System.nanoTime();
 
             BST<String, Integer> bst = new BST<>();
             for (String word : words) {
                 if (bst.contains(word))
                     bst.set(word, bst.get(word) + 1);
                 else
                     bst.add(word, 1);
             }
 
             for(String word: words)
                 bst.contains(word);
 
             long endTime = System.nanoTime();
 
             double time = (endTime - startTime) / 1000000000.0;
             System.out.println("BST: " + time + " s");
 
 
             // Test AVL Tree
             startTime = System.nanoTime();
 
             AVLTree<String, Integer> avl = new AVLTree<>();
             for (String word : words) {
                 if (avl.contains(word))
                     avl.set(word, avl.get(word) + 1);
                 else
                     avl.add(word, 1);
             }
 
             for(String word: words)
                 avl.contains(word);
 
             endTime = System.nanoTime();
 
             time = (endTime - startTime) / 1000000000.0;
             System.out.println("AVL: " + time + " s");
         }
 
         System.out.println();
     }
 }
 ```

1、上述代码我们对傲慢与偏见这本书 使用BST和AVL分别进行单词的存储和单词的查询。计算了消耗的时间

<img src="https://image.imxyu.cn/file/image-20211128142111716.png" alt="image-20211128142111716" style="zoom:50%;" />



2、如果我们将单词先进行排序后再插入，此时就是最坏的情况，对于BST树就会退化成链表。而对于AVL查询每个单词仍然是O（logn）的查询

插入前 先将集合排序执行 Collections.sort(words); 

此时我们可以看到AVL树的优势是非常明显的

<img src="https://image.imxyu.cn/file/image-20211128142504453.png" alt="image-20211128142504453" style="zoom:50%;" />

#### 删除元素维护平衡

同样的，当我们删除一个元素后，我们也要维护相应的树的平衡性。直接复用上面维护平衡性的代码就好

在原有的BST的 remove(）中需要修改的只是当我们删除元素后不能直接返回新的根节点，而是对该节点的平衡性进行维护。在不断向上回溯的过程中调整所有根节点的平衡性。

```java
 private Node remove(Node node, K key){

        if( node == null )
            return null;

        Node retNode;  //将要返回的根节点保存下来
        if( key.compareTo(node.key) < 0 ){
            node.left = remove(node.left , key);
            // return node;
            retNode = node;
        }
        else if(key.compareTo(node.key) > 0 ){
            node.right = remove(node.right, key);
            // return node;
            retNode = node;
        }
        else{   // key.compareTo(node.key) == 0

            // 待删除节点左子树为空的情况
            if(node.left == null){
                Node rightNode = node.right;
                node.right = null;
                size --;
                // return rightNode;
                retNode = rightNode;
            }

            // 待删除节点右子树为空的情况
            else if(node.right == null){
                Node leftNode = node.left;
                node.left = null;
                size --;
                // return leftNode;
                retNode = leftNode;
            }

            // 待删除节点左右子树均不为空的情况
            else{
                // 找到比待删除节点大的最小节点, 即待删除节点右子树的最小节点
                // 用这个节点顶替待删除节点的位置
                Node successor = minimum(node.right);
                //successor.right = removeMin(node.right); //因为之前的removeMin方法没有实现平衡性
                successor.right = remove(node.right, successor.key); //所以这里递归调用一下当前实现平衡的remove方法
                successor.left = node.left;

                node.left = node.right = null;

                // return successor;
                retNode = successor;
            }
        }
		
        if(retNode == null)  //如果删除的是叶子节点，新的根节点返回为null，此时要进行一下判断
            return null;

        //下面的代码和之前添加元素一样，对新的根节点的平衡性维护
        // 更新height
        retNode.height = 1 + Math.max(getHeight(retNode.left), getHeight(retNode.right));

        // 计算平衡因子
        int balanceFactor = getBalanceFactor(retNode);

        // 平衡维护
        // LL
        if (balanceFactor > 1 && getBalanceFactor(retNode.left) >= 0)
            return rightRotate(retNode);

        // RR
        if (balanceFactor < -1 && getBalanceFactor(retNode.right) <= 0)
            return leftRotate(retNode);

        // LR
        if (balanceFactor > 1 && getBalanceFactor(retNode.left) < 0) {
            retNode.left = leftRotate(retNode.left);
            return rightRotate(retNode);
        }

        // RL
        if (balanceFactor < -1 && getBalanceFactor(retNode.right) > 0) {
            retNode.right = rightRotate(retNode.right);
            return leftRotate(retNode);
        }

        return retNode;
    }
```

### AVL完整代码

综上，AVL树只是在BST上在添加元素和删除元素上做了平衡性的调整

```java
public class AVLTree<K extends Comparable<K>, V> {

    private class Node{
        public K key;
        public V value;
        public Node left, right;
        public int height;

        public Node(K key, V value){
            this.key = key;
            this.value = value;
            left = null;
            right = null;
            height = 1;
        }
    }

    private Node root;
    private int size;

    public AVLTree(){
        root = null;
        size = 0;
    }

    public int getSize(){
        return size;
    }

    public boolean isEmpty(){
        return size == 0;
    }

    // 判断该二叉树是否是一棵二分搜索树
    public boolean isBST(){

        ArrayList<K> keys = new ArrayList<>();
        inOrder(root, keys);
        for(int i = 1 ; i < keys.size() ; i ++)
            if(keys.get(i - 1).compareTo(keys.get(i)) > 0)
                return false;
        return true;
    }

    private void inOrder(Node node, ArrayList<K> keys){

        if(node == null)
            return;

        inOrder(node.left, keys);
        keys.add(node.key);
        inOrder(node.right, keys);
    }

    // 判断该二叉树是否是一棵平衡二叉树
    public boolean isBalanced(){
        return isBalanced(root);
    }

    // 判断以Node为根的二叉树是否是一棵平衡二叉树，递归算法
    private boolean isBalanced(Node node){

        if(node == null)
            return true;

        int balanceFactor = getBalanceFactor(node);
        if(Math.abs(balanceFactor) > 1)
            return false;
        return isBalanced(node.left) && isBalanced(node.right);
    }

    // 获得节点node的高度
    private int getHeight(Node node){
        if(node == null)
            return 0;
        return node.height;
    }

    // 获得节点node的平衡因子
    private int getBalanceFactor(Node node){
        if(node == null)
            return 0;
        return getHeight(node.left) - getHeight(node.right);
    }

    // 对节点y进行向右旋转操作，返回旋转后新的根节点x
    //        y                              x
    //       / \                           /   \
    //      x   T4     向右旋转 (y)        z     y
    //     / \       - - - - - - - ->    / \   / \
    //    z   T3                       T1  T2 T3 T4
    //   / \
    // T1   T2
    private Node rightRotate(Node y) {
        Node x = y.left;
        Node T3 = x.right;

        // 向右旋转过程
        x.right = y;
        y.left = T3;

        // 更新height
        y.height = Math.max(getHeight(y.left), getHeight(y.right)) + 1;
        x.height = Math.max(getHeight(x.left), getHeight(x.right)) + 1;

        return x;
    }

    // 对节点y进行向左旋转操作，返回旋转后新的根节点x
    //    y                             x
    //  /  \                          /   \
    // T1   x      向左旋转 (y)       y     z
    //     / \   - - - - - - - ->   / \   / \
    //   T2  z                     T1 T2 T3 T4
    //      / \
    //     T3 T4
    private Node leftRotate(Node y) {
        Node x = y.right;
        Node T2 = x.left;

        // 向左旋转过程
        x.left = y;
        y.right = T2;

        // 更新height
        y.height = Math.max(getHeight(y.left), getHeight(y.right)) + 1;
        x.height = Math.max(getHeight(x.left), getHeight(x.right)) + 1;

        return x;
    }

    // 向二分搜索树中添加新的元素(key, value)
    public void add(K key, V value){
        root = add(root, key, value);
    }

    // 向以node为根的二分搜索树中插入元素(key, value)，递归算法
    // 返回插入新节点后二分搜索树的根
    private Node add(Node node, K key, V value){

        if(node == null){
            size ++;
            return new Node(key, value);
        }

        if(key.compareTo(node.key) < 0)
            node.left = add(node.left, key, value);
        else if(key.compareTo(node.key) > 0)
            node.right = add(node.right, key, value);
        else // key.compareTo(node.key) == 0
            node.value = value;

        // 更新height
        node.height = 1 + Math.max(getHeight(node.left), getHeight(node.right));

        // 计算平衡因子
        int balanceFactor = getBalanceFactor(node);

        // 平衡维护
        // LL
        if (balanceFactor > 1 && getBalanceFactor(node.left) >= 0)
            return rightRotate(node);

        // RR
        if (balanceFactor < -1 && getBalanceFactor(node.right) <= 0)
            return leftRotate(node);

        // LR
        if (balanceFactor > 1 && getBalanceFactor(node.left) < 0) {
            node.left = leftRotate(node.left);
            return rightRotate(node);
        }

        // RL
        if (balanceFactor < -1 && getBalanceFactor(node.right) > 0) {
            node.right = rightRotate(node.right);
            return leftRotate(node);
        }

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

        Node retNode;  //将要返回的根节点保存下来
        if( key.compareTo(node.key) < 0 ){
            node.left = remove(node.left , key);
            // return node;
            retNode = node;
        }
        else if(key.compareTo(node.key) > 0 ){
            node.right = remove(node.right, key);
            // return node;
            retNode = node;
        }
        else{   // key.compareTo(node.key) == 0

            // 待删除节点左子树为空的情况
            if(node.left == null){
                Node rightNode = node.right;
                node.right = null;
                size --;
                // return rightNode;
                retNode = rightNode;
            }

            // 待删除节点右子树为空的情况
            else if(node.right == null){
                Node leftNode = node.left;
                node.left = null;
                size --;
                // return leftNode;
                retNode = leftNode;
            }

            // 待删除节点左右子树均不为空的情况
            else{
                // 找到比待删除节点大的最小节点, 即待删除节点右子树的最小节点
                // 用这个节点顶替待删除节点的位置
                Node successor = minimum(node.right);
                //successor.right = removeMin(node.right);
                successor.right = remove(node.right, successor.key);
                successor.left = node.left;

                node.left = node.right = null;

                // return successor;
                retNode = successor;
            }
        }

        if(retNode == null)  //如果删除的是叶子节点，新的根节点返回为null，此时要进行一下判断
            return null;

        //下面的代码和之前添加元素一样，对新的根节点的平衡性维护
        // 更新height
        retNode.height = 1 + Math.max(getHeight(retNode.left), getHeight(retNode.right));

        // 计算平衡因子
        int balanceFactor = getBalanceFactor(retNode);

        // 平衡维护
        // LL
        if (balanceFactor > 1 && getBalanceFactor(retNode.left) >= 0)
            return rightRotate(retNode);

        // RR
        if (balanceFactor < -1 && getBalanceFactor(retNode.right) <= 0)
            return leftRotate(retNode);

        // LR
        if (balanceFactor > 1 && getBalanceFactor(retNode.left) < 0) {
            retNode.left = leftRotate(retNode.left);
            return rightRotate(retNode);
        }

        // RL
        if (balanceFactor < -1 && getBalanceFactor(retNode.right) > 0) {
            retNode.right = rightRotate(retNode.right);
            return leftRotate(retNode);
        }

        return retNode;
    }

    
}
```



### 基于AVL树的Map和Set的实现

直接复用AVL树的方法即可

AVL-MAP

```java
public class AVLMap<K extends Comparable<K>, V> implements Map<K, V> {

    private AVLTree<K, V> avl;

    public AVLMap(){
        avl = new AVLTree<>();
    }

    @Override
    public int getSize(){
        return avl.getSize();
    }

    @Override
    public boolean isEmpty(){
        return avl.isEmpty();
    }

    @Override
    public void add(K key, V value){
        avl.add(key, value);
    }

    @Override
    public boolean contains(K key){
        return avl.contains(key);
    }

    @Override
    public V get(K key){
        return avl.get(key);
    }

    @Override
    public void set(K key, V newValue){
        avl.set(key, newValue);
    }

    @Override
    public V remove(K key){
        return avl.remove(key);
    }
}

```

AVL-SET

```java
public class AVLSet<E extends Comparable<E>> implements Set<E> {

    private AVLTree<E, Object> avl;

    public AVLSet(){
        avl = new AVLTree<>();
    }

    @Override
    public int getSize(){
        return avl.getSize();
    }

    @Override
    public boolean isEmpty(){
        return avl.isEmpty();
    }

    @Override
    public void add(E e){
        avl.add(e, null);
    }

    @Override
    public boolean contains(E e){
        return avl.contains(e);
    }

    @Override
    public void remove(E e){
        avl.remove(e);
    }
}

```

