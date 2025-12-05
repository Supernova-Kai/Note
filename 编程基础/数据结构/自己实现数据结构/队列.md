# 队列（Queue）
```csharp
public class MyQueue<T>
{
    private T[] _array;
    private int _head; // 队头指针：指向队首元素的索引
    private int _tail; // 队尾指针：指向下一个入队元素的索引
    private int _count; // 队列中实际元素数量

    /// <summary>
    /// 默认初始容量
    /// </summary>
    private const int DefaultCapacity = 4;

    /// <summary>
    /// 获取队列中实际包含的元素数量
    /// </summary>
    public int Count => _count;

    /// <summary>
    /// 获取队列的当前容量（可容纳的元素数）
    /// </summary>
    public int Capacity => _array.Length;

    /// <summary>
    /// 初始化 MyQueue 类的新实例，该实例为空且具有默认初始容量
    /// </summary>
    public MyQueue()
    {
        _array = new T[DefaultCapacity];
        _head = 0;
        _tail = 0;
        _count = 0;
    }

    /// <summary>
    /// 扩容：将容量扩展为当前2倍（兼容初始容量为0的极端场景）
    /// </summary>
    private void ExpandCapacity()
    {
        int newCapacity = _array.Length * 2;
        T[] newArray = new T[newCapacity];

        // 拷贝原队列有效元素到新数组（按队列顺序，从队头到队尾）
        int index = 0;
        int current = _head;
        while (index < _count)
        {
            newArray[index] = _array[current];
            current = (current + 1) % _array.Length; // 循环取原数组元素
            index++;
        }

        // 重置指针：新数组是顺序存储，队头在0，队尾在_count
        _array = newArray;
        _head = 0;
        _tail = _count;
    }

    /// <summary>
    /// 将元素添加到队列的尾部
    /// </summary>
    /// <param name="item">要添加的元素</param>
    public void Enqueue(T item)
    {
        // 容量不足时扩容
        if (_count == _array.Length)
            ExpandCapacity();

        // 元素入队：放到队尾指针位置
        _array[_tail] = item;
        // 队尾指针循环后移（取模实现循环）
        _tail = (_tail + 1) % _array.Length;
        _count++;
    }

    /// <summary>
    /// 移除并返回队列头部的元素
    /// </summary>
    /// <returns>队列头部的元素</returns>
    public T Dequeue()
    {
        // 空队列校验
        if (_count == 0)
            throw new InvalidOperationException("无法出队：队列为空");

        // 获取队首元素
        T item = _array[_head];
        // 清空队首位置（释放引用，帮助GC）
        _array[_head] = default(T);
        // 队头指针循环后移
        _head = (_head + 1) % _array.Length;
        _count--;

        return item;
    }

    /// <summary>
    /// 返回队列头部的元素但不移除
    /// </summary>
    /// <returns>队列头部的元素</returns>
    public T Peek()
    {
        if (_count == 0)
            throw new InvalidOperationException("无法查看队首元素：队列为空");

        // 直接返回队头指针指向的元素
        return _array[_head];
    }

    /// <summary>
    /// 判断指定元素是否存在于队列中
    /// </summary>
    /// <param name="item">要检查的元</param>
    /// <returns>存在则为 true，否则为 false</returns>
    public bool Contains(T item)
    {
        // 空队列直接返回false
        if (_count == 0)
            return false;

        // 从队头开始遍历循环数组，直到遍历完所有有效元素
        int current = _head;
        for (int i = 0; i < _count; i++)
        {
            if (_array[current].Equals(item))
                return true;

            // 循环后移指针
            current = (current + 1) % _array.Length;
        }

        return false;
    }
}
```
