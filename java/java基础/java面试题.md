## java面试题

定义一个int型的数组：int[] arr = new int[]{12,3,3,34,56,77,432};
让数组的每个位置上的值去除以首位置的元素，得到的结果，作为该位置上的新值。遍历新的数组。

乍一眼看去是不是很简单 很多人的做法可能是这种 

```java
public class day02 {
    public static void main(String[] args) {
        int [] arr =new int [] {12,3,3,34,56,77,432};
        //正常方法
        System.out.println("正常方法");
        for (int i = 0; i < arr.length; i++) {
           arr[i]=arr[i]/arr[0];

       }

        for (int i = 0; i < arr.length; i++) {

            System.out.print(arr[i]+" ");
        }
    }
}
```

> 坑点分析：
>
>   因为是要求数组每个位置上的值 去除以首位置的元素 因为他会更新值 当第一个值 12/12 它的值变成了1 
>
>   因为数组第一个值变成了1 在继续遍历下去 后面的值除以1还是本身 这种写法就第一个值生效了后面的没有

### 正确做法

第一种 直接倒序遍历

```java
public class day02 {
    public static void main(String[] args) {
        int [] arr =new int [] {12,3,3,34,56,77,432};
        //方法1通过倒序循环
        System.out.println("方法一");
        for (int i = arr.length-1; i >=0; i--) {
             arr[i]=arr[i]/arr[0];

        }
        for (int i = 0; i < arr.length; i++) {

            System.out.print(arr[i]+" ");
        }


    }
}
```



第二种 吧第一个值单独取出来

```java
public class day02 {
    public static void main(String[] args) {
        int [] arr =new int [] {12,3,3,34,56,77,432};
         //方法2吧第一个数组取出
        System.out.println("方法二");
        int temp =arr[0];
        for (int i = 0; i < arr.length; i++) {
            arr[i]=arr[i]/temp;
            System.out.println(arr[i]+"");
        }
        for (int i = 0; i < arr.length; i++) {

            System.out.print(arr[i]+" ");
        }


    }
}
```

