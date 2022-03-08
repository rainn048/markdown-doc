# 一些操作

## HashMap

1. 初始化： HashMap<Integer, String> map = new HashMap<Integer, String>();
2. 添加元素： map.put(1, "2");
3. 删除元素： map.remove(1);
4. 判断是否存在key: map.containsKey();
5. 判断是否存在value： map.containsValue()；
6. 替换： map.replace(K key, V oldValue, V newValue)
7. 遍历：
   1. 遍历key：for(Integer i : map.keySet()) { 。。。}
   2. 遍历value： for(String value : map.valuse()) { ... }

## 链表

### LinkedList

1. 初始化： `LinkedList<String> lk = new LinkedList<>();`
2. 增加元素： `lk.add(E e);` // 链表末尾添加元素，返回是否成功；
3. `public void add(int index, E element)，//向指定位置插入元素；`
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