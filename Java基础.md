# String 疑难点
### 1. String st1 = "abc" 与 String st2 = new String("abc") ——> st1 != st2
* st1 创建过程：方法区常量池创建一个 “abc” 对象，产生一个内存地址。然后把 “abc” 内存地址赋值给 st1
* st2 创建过程："abc" 属于字符串，字符串属于常量，所以先在常量池中创建一个 “abc”对象。然后再堆内存中创建一个拷贝的副本。
> String 构造方法新创建的字符串是该参数字符串的副本

### 2. String st1 = "a" + "b" + "c" 与 String st2 = "abc" ——> st1 == st2
* 先进行 “a”、“b”、“c” 的拼接，然后在常量池中创建一个 “abc” 常量对象

### 3. String st1 = "ab" 与 String st2 = "abc" 与 String st3 = st1 + "c" ——> st2 != st3
**数据与字符串（+）拼接原理：**
* 由 StringBuilder 类和里面的 append 方法实现拼接，然后调用 toString 把拼接的对象转换成字符串对象，最后把得到的字符串对象的地址赋给变量，所以 st3 的地址指向的是一个 StringBuffer 对象

### 4. String 与 StringBuilder、StringBuffer 区别
1. String 是不可变的字符串。底层是一个 final 修饰的 char[]
2. String 对象赋值之后就会在字符串常量池中缓存
3. StringBuilder 和 StringBuffer 都继承于 AbstractStringBuilder，他们的底层使用的是没有用 final 修饰的 char[]
4. StringBuilder 是**线程不安全**的，它的执行效率比 StringBuffer 要高
5. StringBuffer 是**线程安全**的，执行效率比 StringBuilder 要低



# 排序算法复杂度总结

| 排序算法 | 平均时间复杂度 | 空间复杂度 | 稳定性 |
| -------- | -------------- | ---------- | ------ |
| 冒泡排序 | O(n2)          | O(1)       | 稳定   |
| 选择排序 | O(n2)          | O(1)       | 不稳定 |
| 插入排序 | O(n2)          | O(1)       | 稳定   |
| 希尔排序 | O(nlogn)       | O(1)       | 不稳定 |
| 归并排序 | O(nlogn)       | O(n)       | 稳定   |
| 快速排序 | O(nlogn)       | O(nlogn)   | 不稳定 |
| 堆排序   | O(nlogn)       | O(1)       | 不稳定 |
| 计数排序 | O(n+k)         | O(k)       | 稳定   |
| 桶排序   | O(n+k)         | O(n+k)     | 稳定   |
| 基数排序 | O(n*k)         | O(n+k)     | 稳定   |

**快速排序代码**
```java
//快速排序
void quick_sort(int s[], int l, int r)
{
    if (l < r)
    {
        //Swap(s[l], s[(l + r) / 2]); //将中间的这个数和第一个数交换 参见注1
        int i = l, j = r, x = s[l];
        while (i < j)
        {
            while(i < j && s[j] >= x) // 从右向左找第一个小于x的数
                j--;  
            if(i < j) 
                s[i++] = s[j];
            
            while(i < j && s[i] < x) // 从左向右找第一个大于等于x的数
                i++;  
            if(i < j) 
                s[j--] = s[i];
        }
        s[i] = x;
        quick_sort(s, l, i - 1); // 递归调用 
        quick_sort(s, i + 1, r);
    }
}
```