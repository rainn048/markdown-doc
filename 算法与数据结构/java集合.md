# 一些操作

## 数组

1. 数组拷贝： `Arrays.copyOfRange(num, start, end);`
2. 数据扩容：`Arrays.copyOf(原数组名，新数组长度);`

## HashMap

1. 初始化： HashMap<Integer, String> map = new HashMap<Integer, String>();
2. 添加元素： map.put(1, "2");
3. getOrDefault(); 获取指定 key 对应对 value，如果找不到 key ，则返回设置的默认值
4. 删除元素： map.remove(1);
5. 判断是否存在key: map.containsKey();
6. 判断是否存在value： map.containsValue()；
7. 替换： map.replace(K key, V oldValue, V newValue)
8. 遍历：
   1. 遍历key：for(Integer i : map.keySet()) { 。。。}
   2. 遍历value： for(String value : map.valuse()) { ... }

## Set

1. 添加元素：`add(E e)`
2. 判断是否存在： `contains(Object o)`
3. 是否为空： `isEmpty()`
4. 删除元素; `remove(Object o)`
5. 大小： `size()`
6. 转数组： `toArray()`

## 链表

### ArrayList

```java
add(Object e)
add(int index ,Object e)
addAll(Collection c) 
addAll(int index , Collection c)
size()
get(int index)
set(int index,Object e)
indexOf(Object c)
lastIndexOf(Object c)
isEmpty()
remove(int index)
remove(Object c)
removeAll(Collection<?> c)
contains(Object c)
containsAll(Collection<?> c)
clear()
clone()
iterator()
retainAll(Collection<?> c)
subList(int fromIndex,int toIndex)
trimToSize()  回收多余容量
toArray()
toArray(T[] a)
```

### LinkedList

1. 初始化： `LinkedList<String> lk = new LinkedList<>();`
2. 增加元素： `lk.add(E e);` // 链表末尾添加元素，返回是否成功；
3. `public void add(int index, E element)，//向指定位置插入元素；`

    ```java
    public boolean addAll(int index, Collection<? extends E> c)，将一个集合的所有元素添加到链表的指定位置后面，返回是否成功；
    public void addFirst(E e)，添加到第一个元素；
    public void addLast(E e)，添加到最后一个元素；
    public boolean offer(E e)，向链表末尾添加元素，返回是否成功；
    public boolean offerFirst(E e)，头部插入元素，返回是否成功；
    public boolean offerLast(E e)，尾部插入元素，返回是否成功；
    ```

4. 删除元素

    ```JAVA
    public void clear()，//清空链表；
    public E removeFirst()，//删除并返回第一个元素；
    public E removeLast()，//删除并返回最后一个元素；
    public boolean remove(Object o)，//删除某一元素，返回是否成功；
    public E remove(int index)，//删除指定位置的元素；
    public E poll()，//删除并返回第一个元素；
    public E remove()，//删除并返回第一个元素；
    ```

5. 查

    ```java
    public boolean contains(Object o)，判断是否含有某一元素；
    public E get(int index)，返回指定位置的元素；
    public E getFirst(), 返回第一个元素；
    public E getLast()，返回最后一个元素；
    public int indexOf(Object o)，查找指定元素从前往后第一次出现的索引；
    public int lastIndexOf(Object o)，查找指定元素最后一次出现的索引；
    public E peek()，返回第一个元素；
    public E element()，返回第一个元素；
    public E peekFirst()，返回头部元素；
    public E peekLast()，返回尾部元素；
    ```

6. 改 ： `public E set(int index, E element)，//设置指定位置的元素；`
7. 遍历: `for (int size = linkedList.size(), i = 0; i < size; i++) {...}`
8. 遍历2: `for (String str: linkedList) {...}`

## 队列

1. 创建： `Queue<String> queue = new LinkedList<String>();`
2. 添加元素; `boolean add(E e); boolean offer(E e);`
3. remove(), poll() 删除并返回头部: `remove(); poll();`
4. element(), peek() 获取但不删除：`element(); peek();`

## 优先队列(PriorityQueue)

```java
// 不用比较器，默认升序排列
Queue<Integer> q = new PriorityQueue<>();
```

## String

```java
    String​(StringBuffer buffer) //分配一个新字符串，其中包含当前包含在字符串缓冲区参数中的字符序列。
    String​(StringBuilder builder) //分配一个新字符串，其中包含当前包含在字符串构建器参数中的字符序列。
    String​(char[] value) //分配新的 String ，使其表示当前包含在字符数组参数中的字符序列
    String​(String original) // 初始化新创建的String对象，使其表示与参数相同的字符序列; 换句话说，新创建的字符串是参数字符串的副本
    static String valueOf​(char c) //返回 char参数的字符串表示形式。
    static String valueOf​(char[] data) //返回 char数组参数的字符串表示形式。
    static String valueOf​(char[] data, int offset, int count) //返回 char数组参数的特定子数组的字符串表示形式。
    static String valueOf​(double d) //返回 double参数的字符串表示形式。
    static String valueOf​(float f) //返回 float参数的字符串表示形式。
    static String valueOf​(int i) //返回 int参数的字符串表示形式。
    static String valueOf​(long l) //返回 long参数的字符串表示形式。
    static String valueOf​(Object obj) //返回 Object参数的字符串表示形式。

    char charAt​(int index) //返回指定索引处的 char值。
    boolean equals​(Object anObject) //将此字符串与指定的对象进行比较。
    int length() //返回此字符串的长度。
    String[] split​(String regex) //将此字符串拆分为给定 regular expression的匹配 项 。
    String[] split​(String regex, int limit) //将此字符串拆分为给定 regular expression的匹配 项 。
    public String substring(int beginIndex)
    public String substring(int beginIndex, int endIndex) //beginIndex -- 起始索引（包括）。endIndex -- 结束索引（不包括）。
```

## stack

```java
boolean empty() //测试堆栈是否为空。
Object peek( ) //查看堆栈顶部的对象，但不从堆栈中移除它。
Object pop( ) // 移除堆栈顶部的对象，并作为此函数的值返回该对象。
Object push(Object element) // 把项压入堆栈顶部。
int search(Object element) // 返回对象在堆栈中的位置，以 1 为基数。
```

## 获取随机数

```java
// 第一种方法：
Random rand = new Radnom();
//获取[0, 100)之间的整数
int i = rand.nextInt(100)

// 第二种方法
final double d = Math.random();
//获取[0, 100)之间的整数
final int i = (int)(d*100);
```
