# Sort




### 1. comparable接口

##### 代码测试:

```java
package Sort.comparable;

public class student implements Comparable<student>{
    private String username;
    private int age;

    public student() {
    }

    public student(String username, int age) {
        this.username = username;
        this.age = age;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    @Override
    public String toString() {
        return "student{" +
                "username='" + username + '\'' +
                ", age=" + age +
                '}';
    }


    @Override
    public int compareTo(student o) {
        return this.getAge()-o.getAge();
    }
}
package Sort.comparable;

public class comparableTest {
    //创建两个对象
    public static void main(String[] args) {
        student s1 = new student();
        student s2 = new student();
        s1.setUsername("张三");
        s1.setAge(15);
        s2.setUsername("李四");
        s2.setAge(17);
        Comparable max = getMax(s1, s2);
        System.out.println(max);
    }

    public static Comparable getMax(Comparable c1,Comparable c2){
        int result = c1.compareTo(c2);
        if (result>=0){
            return c1;
        }else {
            return c2;
        }
    }
}
```

### 2. 冒泡排序

##### 排序原理:

1. 比较相邻的元素,如果前一个元素比后一个元素大,就交换这两个元素;
2. 对每一对相邻的元素做同样的工作,从开始第一对元素到结尾的最后一个元素;最终最后位置的元素就是最大值;

##### 代码实现:

```java
/*冒泡排序*/
public class MyBubble {
    public static void sort(Comparable[] a ){
        for (int i = a.length; i >0 ; i--) {
            for (int j = 0; j < i; j++) {
                //比较两个索引处的大小
                if (greater(a[j],a[j])){
                    exchange(a,j,j+1);
                }
            }
        }
    }
    //比较两个元素的大小
    public static boolean greater(Comparable a,Comparable b){
        return a.compareTo(b)>0;
    }


    //两元素互换位置
    public static void exchange(Comparable[] a,int i,int j){
        Comparable temp;
        temp=a[i];
        a[i]=a[j];
        a[j]=temp;

    }


}
```

##### 代码测试:

```java
import java.util.Arrays;

public class MyBubbleTest  {
    public static void main(String[] args) {
        Integer[] arr={1,4,6,7,2,9};
        MyBubble.sort(arr);
        System.out.println(Arrays.toString(arr));
    }
}
```

### 3. 选择排序

##### 排序原理:

1. 每一次遍历的过程中,都假定第一个索引处的元素是最小值,和其他索引处的值一次进行比较,如果当前索引处的值大于其他某个索引处的值,则假定其他某个索引处的值为最小值,最后可以找到最小值所在的索引;
2. 交换第一个索引处和最小值所在的索引处的值;

##### 代码实现:

```java
public class MySelection {
    public static void sort(Comparable [] a){
        for (int i = 0; i <= a.length - 2; i++) {
            int minIndex=i;
            for (int j = i+1; j < a.length; j++) {
                if (greater(a[minIndex],a[j])){
                    minIndex=j;
                }
            }
            exchange(a,i,minIndex);
        }

    }
    //比较两个元素的大小
    public static boolean greater(Comparable a,Comparable b){
        return a.compareTo(b)>0;
    }


    //两元素互换位置
    public static void exchange(Comparable[] a,int i,int j){
        Comparable temp;
        temp=a[i];
        a[i]=a[j];
        a[j]=temp;

    }
}
```

##### 代码测试:

```java
import java.util.Arrays;

public class MySelectionTest {
    public static void main(String[] args) {
        Integer[] arr={1,4,6,5,9,7};
        MySelection.sort(arr);
        System.out.println(Arrays.toString(arr));
    }
}
```

### 4. 插入排序

##### 原理:

1. 把所有的元素分为两组,已经排序的和未排序的;
2. 找到未排序的组中的第一个元素,向已经排序的组中进行插入;
3. 倒叙遍历已经排序的元素,一次和待插入的元素进行比较,知道找到一个元素小于等于待插入元素,那么就把待插入元素放到这个位置,其他元素向后移动一位;

##### 代码实现:

```java
/*插入排序*/
public class MyInsertion {
    public static void sort(Comparable[] a){
        for (int i = 1; i < a.length; i++) {
            for (int j = i; j >=0 ; j--) {
                if (greater(a[j-1],a[j])){
                    exchange(a,j,j-1);
                }else {
                    break;
                }
            }
        }

    }
    
    //比较两个元素的大小
    public static boolean greater(Comparable a,Comparable b){
        return a.compareTo(b)>0;
    }

    //两元素互换位置
    public static void exchange(Comparable[] a,int i,int j){
        Comparable temp;
        temp=a[i];
        a[i]=a[j];
        a[j]=temp;

    }
}
```

##### 代码测试:

```java
import java.util.Arrays;

public class MyInsertionTest {
    public static void main(String[] args) {
        Comparable[] arr={1,4,6,8,9,3,6,8};
        MyInsertion.sort(arr);
        System.out.println(Arrays.toString(arr));
    }
}
```

### 5. 希尔排序

##### 排序原理:

1. 选定一个增长量h,按照曾长量h作为数据分组的依据,对数据进行分组;
2. 对分好组的每一组数据完成插入排序;
3. 减小曾长量,最小减为3,重复第二步操作;

##### 增长量h的确定(没一固定的原则):

```java
int h=1;
while(h<leng/2){
    h=2*h+1;
}
 //循环结束后我们就可以确定h的最大值;
 h的减小规则为:
 h=h/2
```

##### 代码实现:

```java
public class MyShell {
    public static void sort(Comparable[] a){
        //得到曾长量h的值
        int h=1;
        while(h<a.length/2){
            h=h*2+1;
        }
        while (h>=1){
            //找到待插入的元素
            for (int i = h; i <a.length ; i++) {
                //把待插入的元素插入到有序序列中
                for (int j = i; j >=h ; j-=h) {
                    if (greater(a[j-h],a[j])){
                        exchange(a,j-h,j);
                    }else {
                        //待插入元素找到合适位置,结束循环
                        break;
                    }

                }

            }
            //减小曾长量h
            h=h/2;
        }

    }

    //比较两个元素的大小
    public static boolean greater(Comparable a,Comparable b){
        return a.compareTo(b)>0;
    }

    //两元素互换位置
    public static void exchange(Comparable[] a,int i,int j){
        Comparable temp;
        temp=a[i];
        a[i]=a[j];
        a[j]=temp;

    }
}
```

##### 代码测试:

```java
import java.util.Arrays;

public class MyShellTest {
    public static void main(String[] args) {
        Comparable[] arr={1,5,8,9,5,8,5,9,45,67,23,7};
        MyShell.sort(arr);
        System.out.println(Arrays.toString(arr));
    }
}
```

### 6. 归并排序

##### 排序原理:

- 尽可能的一组数据拆分成两个元素的子组,并对每一个子组继续拆分,知道拆分后的每个子组的元素个数是1为止;
- 将相邻的两个子组进行合并成一个有序的大组;
- 不断的重复步骤2,知道最终只有一个组为止;

##### 归并排序API设计:

| 类名     | Merge                                                        |
| -------- | ------------------------------------------------------------ |
| 构造方法 | Merge():创建Merge对象                                        |
| 成员方法 | 1.public static void sort(Comparable[] a):对组内元素进行排序 2. private static void sort(Comparable[] a ,int left,int right):对组内索引lo到索引hi的元素进行排序 3.private static void merge(Comparable[] a ,int left,int mid,int right):从索引left到索引mid为一个子组,从索引mid+1到索引right为另一个子组,吧数组a中的这两个子组的数据合并成一个有序的大组(从索引leftPos到索引rightPos) 4.private static boolean less(Comparable v,Comparable w):判断v是否小于w |
| 成员变量 | 1.private static Comparable[] assist:完成归并操作需要的辅助数组 |

##### 代码实现:

```java
package Sort.merge;

public class MyMerge {
    //归并所需要的辅助数组
    private static Comparable[] assist;

    //比较a是否小于b
    public static boolean less(Comparable v,Comparable w){
        return v.compareTo(w)<=0;
    }
    
    //对数组a中的元素进行排序(private递归型方法sort的驱动程序)
    public static void sort(Comparable[] a){
        //初始化辅助数组
        assist = new Comparable[a.length];
        int left=0;
        int right=a.length-1;
        sort(a,left,right);

    }

    //对数组a中lo到hi的元素进行排序
    private static void sort(Comparable[] a,int left,int right){
        //安全性检验
        if(right<=left)
            return;

        int mid=(left+right)/2;
        sort(a,left,mid);
        sort(a,mid+1,right);
        merge(a,left,mid+1,right);

    }

/*对数组中,从lo到mid为一组,mid+1到hi为一组,
 leftPos,rightPos分别记录左右两数组中的开始位置,rightEnd记录最后的位置*/
    private static void merge(Comparable[]a ,int leftPos,int rightPos,int rightEnd){
        //记录左边数组的最后位置
        int leftEnd=rightPos-1;
        
        //记录辅助数组的位置
        int assistPos=leftPos;
        
        //当左右数组都还有元素
        while (leftPos<=leftEnd && rightPos<=rightEnd)
            //判断元素大小
            if (less(a[leftPos],a[rightPos]))
                assist[assistPos++]=a[leftPos++];
            else
                assist[assistPos++]=a[rightPos++];

        //当左数组没有元素时
        while (rightPos<=rightEnd)
            assist[assistPos++]=a[rightPos++];
        
        //当右数组没有元素时
        while (leftPos<=leftEnd)
            assist[assistPos++]=a[leftPos++];

        //拷贝回原数组
        for (int i = 0; i < rightPos; i++,rightEnd--) {
            a[rightEnd]=assist[rightEnd];

        }

        
    }
}
```

##### 测试代码:

```java
package Sort.merge;

import java.util.Arrays;

public class MyMergeTest {
    public static void main(String[] args) {
        Comparable[] arr={3,4,5,7,9,7,4,9,32,45,33,55,33};
        MyMerge.sort(arr);
        System.out.println(Arrays.toString(arr));
    }
}
```

### 7. 快速排序

##### 排序原理:

1. 首先设定一个分界值,通过该分界值将数组分成左右两个部分;
2. 将大于或等于分界值的数据放到数组右边,小于分界值的数据放到数组的左边,此时左边部分中各元素都小于分界值,而右边部分中各元素都大于或等于分界值;
3. 然后,左边和右边的数据可以独立排序.对于左侧的数组数据,又可以取一个分界值,将该部分数据分成左右两个部分,同样在左边放置较小值,右边放置较大值.右侧的数组数据也可以做类似处理;
4. 重复上述过程,可以看出,这是一个递归定义.通过递归将左侧部分拍好序,在递归排好右边部分的顺序,当左侧和右侧两个部分的数据排完序后,整个数组的排序也就完成了;

##### 快速排序API设计:

| 类名     | Quick                                                        |
| -------- | ------------------------------------------------------------ |
| 构造方法 | Quick():创建Quick对象                                        |
| 成员方法 | 1.public static void sort(Comparable[] a):对组内元素进行排序 2. private static void sort(Comparable[] a ,int left,int right):对组内索引lo到索引hi的元素进行排序 3.private static int partition(Comparable[] a ,int left , int right):对数组a中,从索引left到索引right进行分组,并返回分组界限对应的索引 4.private static boolean less(Comparable v,Comparable w):判断v是否小于w |

##### 切分原理:

##### 一个数组气氛成两个子数组的基本思想:

1. 找一个基准值,用两个指针分别指向数组的头部和尾部;
2. 先从尾部向头部开始搜索一个比基准值小的元素,搜索到即停止,并记录指针的位置;
3. 再从头部向尾部开始搜索一个比基准值大的元素,所有到即停止,并记录指针的位置;
4. 交换当前左边指针位置和右边指针位置的元素;
5. 重复2,3,4步骤,知道左边指针的值大于右边指针的值停止;

##### 代码实现:

```java
public class MyQuick {


    public static void sort(Comparable[] a){
        int left=0;
        int right=a.length-1;
        sort(a,left,right);

    }

    //对数组索引left到right进行排序
    private static void sort(Comparable [] a,int left,int right){
        if (right<left){
            return;
        }
    //对数组索引left到right之间的元素进行分组(左子组,右子组)
        int partition = partition(a, left, right);//返回的是变化后的分界值索引

        //让左子组有序
        sort(a,left,partition-1);
        //让右子组有序
        sort(a,partition+1,right);


    }

    //比较a是否小于b
    public static boolean less(Comparable v,Comparable w){
        return v.compareTo(w)<=0;
    }

    //两元素互换位置
    public static void exchange(Comparable[] a,int i,int j){
        Comparable temp;
        temp=a[i];
        a[i]=a[j];
        a[j]=temp;

    }
    //对数组a中,索引left到right之间的元素进行分组,并返回分界值对应的索引
    public static int partition(Comparable[] a,int left,int right){
        //确定分界值
        Comparable key=a[left];
        //定义两个指针,分别指向切分元素最小索引处和最大索引处的下一个元素
        int leftPos=left;
        int rightPos=right+1;
        //切分
        while(true){
            //从右往左扫描,移动rightPos指针,遇到比key小的值就停止
            while (less(key,a[--rightPos])){
                if (rightPos==left){
                    break;
                }
            }

            //从左往右扫描,移动leftPos指针,遇到比key大的值就停止
            while (less(a[++leftPos],key)){
                if (leftPos==right){
                    break;
                }
            }
            //判断leftPos>=rightPos,如果是,则结束,不是则交换元素
            if (leftPos>=rightPos){
                break;
            }else {
                exchange(a,leftPos,rightPos);

            }
            //分界值和相遇的元素交换
        }
        exchange(a,left,rightPos);

           return rightPos;
    }

}
```

##### 代码测试:

```java
package Sort.quick;

import java.util.Arrays;

public class MyQuickTest {
    public static void main(String[] args) {
        Integer [] arr={6,1,2,7,9,3,4,5,8};
        MyQuick.sort(arr);
        System.out.println(Arrays.toString(arr));
    }
}
```


