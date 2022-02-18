# Tree


### 1. 二叉查找树

###### 二叉树结点类API的设计:

| 类名     | Node<Key,Value>                                              |
| -------- | ------------------------------------------------------------ |
| 构造方法 | Node(Key key,Value value,Node left,Node right):创建Node对象  |
| 成员变量 | 1. public Node left:记录左子结点 2. public Node right:记录右子结点 3. public Key key:存储键 4. public Value value:存储值 |

##### 代码实现:

```java
public class Node<Key,Value> {
    //存储键
    public Key key;
    //存储值
    public Value value;
    //存储左子结点
    public Node left;
    //存储右子结点
    public Node right;

    public Node(Key key, Value value, Node left, Node right) {
        this.key = key;
        this.value = value;
        this.left = left;
        this.right = right;
    }
}
```

##### 二叉查找树的API设计

| 类名     | BinaryTree<Key extends Comparable<key>,Vlaue value>          |
| -------- | ------------------------------------------------------------ |
| 构造方法 | BinaryTree():创建BinaryTree对象                              |
| 成员变量 | 1. private Node root:记录根结点 2. private int N :记录树中元素的个数 |
| 成员方法 | 1. public void put(Key key,Value value):向书中插入一个键值对 2. private Node put(Node x,Key key,Value value):给指定树x上,添加一个键值对,并返回添加后的新树 3. public Value get(Key key):根据key从书中找出相对应的值 4. private Value get(Node x,Key key):从指定的数x中,找出key对应的值 5. public] void delete(Key key):根据key,删除数中对应的键值对 6. private Node delete(Node x,Key key):删除指定树x上的键为key的键值对,并返回删除后的新树 7. public int size():获取树中元素的个数 |

##### 二叉查找树实现

1. 如果当前树中没有任何一个结点,则直接把新节点当做根结点使用
2. 如果当前树不为空,则从根结点开始:
   - 如果新结点的key小于当前结点的key,则继续找当前结点的左子节点;
   - 如果新结点的key大于当前结点的key,则继续找当前结点的右子节点;
   - 如果新结点的key等于当前结点的key,则树中已经存在这样的结点,替换该结点的value值即可;

##### 代码实现:

```java
//向指定的树x中添加key-value,并返回添加元素后的新树
    public Node put(Node x,Key key,Value value){
        //如果x子树为空
        if (x==null){
            //元素个数+1
            N++;
            return new Node(key,value,null,null);
        }
        //如果x子树不为空
        //比较x结点的键和key的大小
        int cmp = key.compareTo(x.key);
        if (cmp>0){
            //如果key大于x结点的键,则继续寻找x结点的右子树
            x.right = put(x.right, key, value);
        }else if (cmp<0){
            //如果key小于x结点的键,则继续寻找x结点的左子树
            x.left=put(x.left,key,value);
        }else {
            //如果key等于x结点的键,则替换x结点的值为value
            x.value=value;
        }
        return x;
    }
```

##### 查询方法get实现思想:

从根结点开始:

1. 如果要查询的key小于当前结点的key,则继续寻找当前结点的左子结点;
2. 如果要查询的key大于当前结点的key,则继续寻找当前结点的右子结点;
3. 如果要查询的key等于当前结点的key,则树中返回当前结点的value.

##### 代码实现:

```java
//从指定的数x中,查询key对应的值
    public Value get(Node x,Key key){
        //如果x子树为空
        if (x==null){
            return null;
        }

        //如果x子树不为空
        //比较x结点的键和key的大小
        int cmp = key.compareTo(x.key);
        if (cmp>0){
            //如果key大于x结点的键,则继续寻找x结点的右子树
            return get(x.right,key);

        }else if (cmp<0){
            //如果key小于x结点的键,则继续寻找x结点的左子树
            return get(x.left,key);

        }else {
            //如果key等于x结点的键,则找到了键为key的结点,返回x结点的value即可
            return x.value;

        }
    }
```

##### 删除方法delete的实现思想:

1. 找到被删除结点;
2. 找到被删除结点右子树的最小结点minNode
3. 删除右子树中的最小结点
4. 让被删除结点的左子树称为最小结点minNode的左子树,让被删除结点的右子树称为最小结点minNode的右子树
5. 让被删除结点的父结点指向最小结点minNode

##### 代码实现:

```java
//删除指定树中key对应的value,并返回删除后的新树
    public Node delete(Node x,Key key){
        //如果x树为空
        if (x==null){
            return null;
        }
        //如果x树不为空
        //比较x结点的键和key的大小
        int cmp = key.compareTo(x.key);
        if (cmp>0){
            //如果key大于x结点的键,则继续寻找x结点的右子树
            x.right= delete(x.right,key);

        }else if (cmp<0){
            //如果key小于x结点的键,则继续寻找x结点的左子树
            x.left=delete(x.left,key);

        }else {
            //元素个数-1
            N--;
            //如果key等于x结点的键,则要删除的结点就是x;

            //寻找右子树的最小结点
            if (x.right==null){
                return x.left;
            }
            if (x.left==null){
                return x.right;
            }

            //默认情况记为右子树
            Node minNode = x.right;
            while (minNode.left!=null){
                minNode=minNode.left;
            }

            //删除右子树中的最小结点
            Node n = x.right;
            while (n.left!=null){
                if (n.left.left==null){
                    n.left=null;
                }else{
                    //变换n结点
                    n=n.left;
                }
            }
            //让x结点的左子树成为minNode的左子树
            minNode.left=x.left;

            //让x结点的右子树成为minNode的右子树
            minNode.right=x.right;

            //让x结点的父结点指向minNode
            x = minNode;


        }
        return x;
    }
```

##### 完整代码:

```java
public class MyBinaryTree<Key extends Comparable<Key>,Value> {
    //记录根结点
    private Node root;
    //记录树中元素的个数
    private int N;

    private class Node {
        //存储键
        public Key key;
        //存储值
        private Value value;
        //存储左子结点
        public Node left;
        //存储右子结点
        public Node right;

        public Node(Key key, Value value, Node left, Node right) {
            this.key = key;
            this.value = value;
            this.left = left;
            this.right = right;
        }
    }

    //获取树种元素的个数
    public int size(){
        return N;
    }

    //向树中添加元素key-value
    public void put(Key key, Value value){
        root = put(root, key, value);

    }

    //向指定的树x中添加key-value,并返回添加元素后的新树
    public Node put(Node x,Key key,Value value){
        //如果x子树为空
        if (x==null){
            //元素个数+1
            N++;
            return new Node(key,value,null,null);
        }
        //如果x子树不为空
        //比较x结点的键和key的大小
        int cmp = key.compareTo(x.key);
        if (cmp>0){
            //如果key大于x结点的键,则继续寻找x结点的右子树
            x.right = put(x.right, key, value);
        }else if (cmp<0){
            //如果key小于x结点的键,则继续寻找x结点的左子树
            x.left=put(x.left,key,value);
        }else {
            //如果key等于x结点的键,则替换x结点的值为value
            x.value=value;
        }
        return x;
    }

    //查询树中指定key对应的value
    public Value get(Key key){
        return get(root,key);
    }

    //从指定的数x中,查询key对应的值
    public Value get(Node x,Key key){
        //如果x子树为空
        if (x==null){
            return null;
        }

        //如果x子树不为空
        //比较x结点的键和key的大小
        int cmp = key.compareTo(x.key);
        if (cmp>0){
            //如果key大于x结点的键,则继续寻找x结点的右子树
            return get(x.right,key);

        }else if (cmp<0){
            //如果key小于x结点的键,则继续寻找x结点的左子树
            return get(x.left,key);

        }else {
            //如果key等于x结点的键,则找到了键为key的结点,返回x结点的value即可
            return x.value;

        }
    }

    //删除树中key对应的value
    public void delete(Key key){
        delete(root,key);
    }

    //删除指定树中key对应的value,并返回删除后的新树
    public Node delete(Node x,Key key){
        //如果x树为空
        if (x==null){
            return null;
        }
        //如果x树不为空
        //比较x结点的键和key的大小
        int cmp = key.compareTo(x.key);
        if (cmp>0){
            //如果key大于x结点的键,则继续寻找x结点的右子树
            x.right= delete(x.right,key);

        }else if (cmp<0){
            //如果key小于x结点的键,则继续寻找x结点的左子树
            x.left=delete(x.left,key);

        }else {
            //元素个数-1
            N--;
            //如果key等于x结点的键,则要删除的结点就是x;

            //寻找右子树的最小结点
            if (x.right==null){
                return x.left;
            }
            if (x.left==null){
                return x.right;
            }

            //默认情况记为右子树
            Node minNode = x.right;
            while (minNode.left!=null){
                minNode=minNode.left;
            }

            //删除右子树中的最小结点
            Node n = x.right;
            while (n.left!=null){
                if (n.left.left==null){
                    n.left=null;
                }else{
                    //变换n结点
                    n=n.left;
                }
            }
            //让x结点的左子树成为minNode的左子树
            minNode.left=x.left;

            //让x结点的右子树成为minNode的右子树
            minNode.right=x.right;

            //让x结点的父结点指向minNode
            x = minNode;
        }
        return x;
    }

}
```

### 2. 二叉树的最小和最大键

##### 查找二叉树中最小的键

在某些情况下,我们需要查找出数中存储所有元素的键的最小值,比如我们的树中存储的是学生的排名和姓名数据,那么需要查找出排名最低是多少名>这里我们设计如下两个方法来完成:

| public Key min()         | 找出树中最小的值               |
| ------------------------ | ------------------------------ |
| private Node min(Node x) | 找出指定树x中,最小键所在的结点 |

```java
//找出整个树中的最小键
    public Key min(){
        return min(root).key;
    }

    //找出指定树中的最小键所在的结点
    private Node min(Node x){
        //判断x是否还有左子结点,如果有则继续向左找,如果没有,则x就是最小键所在的结点
        if (x.left!=null){
            return min(x.left);
        }else {
            return x;
        }

    }
```

##### 查找二叉树中最大的键

| public Key max()         | 找出树中最大的键               |
| ------------------------ | ------------------------------ |
| private Node max(Node x) | 找出指定树x中,最大键所在的结点 |

```java
//找出整个树中最大键
    public Key max(){
        return max(root).key;
    }

    //找到指定树中最大的键所在的结点
    private Node max(Node x){
        //判断x是否还有右结点,如果有,则继续向右找,如果没有,则x就是最大的键所在的结点
        if (x.right!=null){
            return max(x.right);
        }else{
            return x;
        }
    }
```

### 3. 二叉树的基础遍历

很多情况下,我们可能需要像遍历数组一样遍历树,从而拿出书中存储的每一个元素,由于树状结构和线性结构不一样,他没有办法从头开始依次向后遍历,所以存在如何遍历,也就是按照什么样的**搜索路径**进行遍历的问题.

#### 我们可以把二叉树的遍历分为以下三种方式:

1. 前序遍历:
   1. 先访问根结点,然后在访问左子树,最后访问右子树
2. 中序遍历:
   1. 先访问左子树,中间访问根结点,最后访问右子树
3. 后序遍历:
   1. 先访问左子树,在访问右子树,最后访问根结点

#### 前序遍历

```java
public Queue<Key> preErgodic():使用前序遍历,获取真个树中的所有键
private void proErgodic(Node x, Queue<Key> keys):使用前序遍历,把指定树x中的所有键放入到keys队列中
```

实现过程中,我们通过前序遍历,把每个结点的key取出,放入到队列中返回即可.

##### 实验步骤:

1. 把当前结点的key放入到队列中;
2. 找到当前结点的左子树,如果不为空,递归遍历左子树;
3. 找到当前结点的右子树,如果不为空,递归遍历右子树

##### 代码实现:

```java
//获取整个树中的所有键
    public MyQueue<Key> preErgodic(){
        MyQueue<Key> keys = new MyQueue<>();
        preErgodic(root,keys);
        return keys;
    }

    //获取指定树x中的所有键,并放到keys队列中
    private void preErgodic(Node x ,MyQueue<Key> keys){
        //判断x是否为空
        if (x==null){
            return;
        }
        //把x结点的key放入keys队列中
        keys.inQueue(x.key);

        //递归遍历x结点的左子树
        if (x.left!=null){
            preErgodic(x.left,keys);
        }

        //递归遍历x结点的右子树
        if (x.right!=null){
            preErgodic(x.right,keys);
        }

    }
```

##### 代码测试:

```java
package Tree.BinaryTree;

import List.Queue.MyQueue;

public class BinaryTreeErgodicTest {
    public static void main(String[] args) {
        MyBinaryTree<String, String> tree = new MyBinaryTree<>();
        tree.put("E","5");
        tree.put("B","2");
        tree.put("G","7");
        tree.put("A","1");
        tree.put("D","4");
        tree.put("F","6");
        tree.put("H","8");
        tree.put("C","3");
        MyQueue<String> keys = tree.preErgodic();
        for(String key:keys){
            String value = tree.get(key);
            System.out.println(value);
        }
    }
}
```

#### 中序遍历

```java
public Queue<Key> minErgodic():使用中序遍历,获取整个树的所有键
private void midErgodic(Node x, Queue<Key> keys):使用中序遍历,把指定树x的素有键放入到keys队列中
```

##### 实现步骤:

1. 找到当前结点的左子树,如果不为空,递归遍历左子树;
2. 把当前结点的key放到队列中;
3. 找到当前结点的右子树,如果不为空,递归遍历右子树;

##### 代码实现:

```java
//获取整个树中的所有键
    public MyQueue<Key> midErgodic(){
        MyQueue<Key> keys = new MyQueue<>();
        midErgodic(root,keys);
        return keys;
    }

    //获取指定树x中的所有键,并放到keys队列中
    private void midErgodic(Node x ,MyQueue<Key> keys){
        //判断x是否为空
        if (x==null){
            return;
        }

        //递归遍历x结点的左子树
        if (x.left!=null){
            preErgodic(x.left,keys);
        }

        //把x结点的key放入keys队列中
        keys.enQueue(x.key);

        //递归遍历x结点的右子树
        if (x.right!=null){
            preErgodic(x.right,keys);
        }

    }
```

#### 后序遍历

##### 实现步骤:

1. 找到当前结点的左子树,如果不为空,递归遍历左子树;
2. 找到当前结点的右子树,如果不为空,递归遍历右子树;
3. 把当前结点的key放入到队列中;

```java
public Queue<Key> afterErgodic():使用中序遍历,获取整个树的所有键
private void afterErgodic(Node x, Queue<Key> keys):使用中序遍历,把指定树x的素有键放入到keys队列中
```

##### 代码实现:

```java
//获取整个树中的所有键
    public MyQueue<Key> afterErgodic(){
        MyQueue<Key> keys = new MyQueue<>();
        afterErgodic(root,keys);
        return keys;
    }

    //获取指定树x中的所有键,并放到keys队列中
    private void afterErgodic(Node x ,MyQueue<Key> keys){
        //判断x是否为空
        if (x==null){
            return;
        }

        //递归遍历x结点的左子树
        if (x.left!=null){
            preErgodic(x.left,keys);
        }

        //递归遍历x结点的右子树
        if (x.right!=null){
            preErgodic(x.right,keys);
        }

        //把x结点的key放入keys队列中
        keys.enQueue(x.key);

    }
```

### 4. 二叉树层序遍历

所谓的层序遍历,就是从根结点(第一层)开始,依次向下,获取每一层所有结点的值;

##### 实现:

```java
public Queue<Key> layerErgodic():使用层序遍历,获取整个树中的所有键
```

##### 实验步骤:

1. 创建队列,存储每一层结点;
2. 使用循环从队列中弹出一个结点:
   1. 获取当前结点的key;
   2. 如果当前结点的左子节点不为空,则把左子节点放入队列中;
   3. 如果当前结点的右子结点不为空,则把右子节点放入队列中;

##### 代码实现:

```java
public MyQueue<Key> layerErgodic() throws InterruptedException {
       //定义两个队列,分别存储树中的键和树中的结点
       MyQueue<Key> keys = new MyQueue<>();
       MyQueue<Node> nodes = new MyQueue<>();

       //默认把根结点root放入队列中
       nodes.enQueue(root);

       while (!nodes.isEmpty()){
           //如果队列不为空,则弹出一个结点,把该结点的key放入keys中
           Node n = nodes.deQueue();
           keys.enQueue(n.key);

           //如果该结点有左子树,则把左子树放入队列中
           if (n.left!=null){
               nodes.enQueue(n.left);
           }

           //如果该结点有右子树,则把右子树放入队列中
           if (n.right!=null){
               nodes.enQueue(n.right);
           }

       }

       return keys;
   }
```

##### 代码测试:

```java
MyBinaryTree<String, String> tree = new MyBinaryTree<>();
       tree.put("E", "5");
       tree.put("B", "2");
       tree.put("G", "7");
       tree.put("A", "1");
       tree.put("D", "4");
       tree.put("F", "6");
       tree.put("H", "8");
       tree.put("C", "3");
       MyQueue<String> keys = tree.layerErgodic();
       for (String key:keys){
           String s = tree.get(key);
           System.out.println(s);
       }
```

### 5. 二叉树的最大深度问题

##### 需求:

给定一棵树,计算树的最大深度(树的根结点到最远叶子结点的最长路径上的结点树);

##### 实现:

public int maxDepth():计算整个树的最大深度 private int maxDepth(Node x):计算指定树的最大深度

##### 实现步骤:

1. 如果根结点为空,则最大深度为0;
2. 计算左子树的最大深度;
3. 计算右子树的最大深度;
4. 当前树的最大深度=左子树的最大深度和右子树的最大深度中的较大者+1

##### 代码实现:

```java
//计算树的最大深度
    public int maxDepth(){
        return maxDepth(root);

    }

    private int maxDepth(Node x){
        if (x==null) return 0;

        //记录整个树的最大深度
        int max=0;
        //记录左子树的最大深度
        int maxLeft=0;
        //记录右子树的最大深度
        int maxRight=0;

        //计算x结点左子树的最大深度
        if (x.left!=null){
            maxLeft=maxDepth(x.left);
        }

        //计算x结点右子树的最大深度
        if (x.right!=null){
            maxRight=maxDepth(x.right);
        }

        //计算整个树的最大深度
        max=Math.max(maxLeft+1,maxRight+1);
        return max;



    }
```

##### 代码测试:

```java
MyBinaryTree<String, String> tree = new MyBinaryTree<>();
        tree.put("E", "5");
        tree.put("B", "2");
        tree.put("G", "7");
        tree.put("A", "1");
        tree.put("D", "4");
        tree.put("F", "6");
        tree.put("H", "8");
        tree.put("C", "3");
        MyQueue<String> keys = tree.layerErgodic();
        for (String key:keys){
            String s = tree.get(key);
            System.out.println(s);
        }

        int i = tree.maxDepth();
        System.out.println(i);
    
```


