# 列表（List）
```csharp

public class MyList<T>
{
    private T[] _array;
    private int _count;

    /// <summary>
    /// 获取列表中实际包含的元素数量
    /// </summary>
    public int Count => _count;
    
    /// <summary>
    /// 获取列表的当前容量（可容纳的元素数）
    /// </summary>
    public int Capacity => _array.Length;

    /// <summary>
    /// 默认初始大小
    /// </summary>
    private const int DefaultCapacity = 4;

    public MyList()
    {
        _array = new T[DefaultCapacity];
        _count = 0;
    }
    
    /// <summary>
    /// 扩容
    /// </summary>
    private void ExpandCapacity()
    {
        int newCapacity = _array.Length * 2;
        T[] newArray = new T[newCapacity]; // 新数组
        
        // 拷贝原数组数据到新数组
        for (int i = 0; i < _count; i++)
        {
            newArray[i] = _array[i];
        }
        
        _array = newArray;
    }

    /// <summary>
    /// 将对象添加到 MyList 的末尾
    /// </summary>
    /// <param name="item">要添加的对象</param>
    public void Add(T item)
    {
        // 容量不足时扩容
        if (_count == _array.Length)
            ExpandCapacity();

        _array[_count] = item;
        _count++;
    }

    /// <summary>
    /// 将元素插入到 MyList 的指定索引位置
    /// </summary>
    /// <param name="index">指定索引</param>
    /// <param name="item">要插入的对象</param>
    public void Insert(int index, T item)
    {
        if (index < 0 || index > _count)
            throw new ArgumentOutOfRangeException();

        // 容量不足时扩容
        if (_count == _array.Length)
            ExpandCapacity();
        
        // 1.从后往前，将插入位置的元素后移
        for (int i = _count; i > index; i--)
        {
            _array[i] = _array[i - 1];
        }
        
        // 2.插入新元素
        _array[index] = item;
        _count++;
    }

    /// <summary>
    /// 从末尾删除
    /// </summary>
    public void Remove()
    {
        if (_count == 0)
            throw new InvalidOperationException("List is empty");
        
        _array[_count - 1] = default(T);
        _count--;
    }

    /// <summary>
    /// 移除 MyList 中指定索引处的元素
    /// </summary>
    /// <param name="index">要移除的元素的从零开始的索引</param>
    public void RemoveAt(int index)
    {
        if (index < 0 || index >= _count)
            throw new ArgumentOutOfRangeException();
        
        // 1.删除位置后面的元素前移
        for (int i = index; i < _count - 1; i++)
        {
            _array[i] = _array[i + 1];
        }
        
        // 2.最后一个位置清空
        _array[_count - 1] = default(T);
        _count--;
    }

    /// <summary>
    /// 索引器
    /// </summary>
    public T this[int index]
    {
        get
        {
            if (index < 0 || index >= _count)
                throw new IndexOutOfRangeException();
            
            return _array[index];
        }

        set
        {
            if (index < 0 || index >= _count)
                throw new IndexOutOfRangeException();
            
            _array[index] = value;
        }
    }

    /// <summary>
    /// 获取某个元素的索引，如果不存在则返回-1
    /// </summary>
    public int IndexOf(T item)
    {
        for (int i = 0; i < _count; i++)
        {
            if (_array[i].Equals(item))
                return i;
        }

        return -1;
    }

    /// <summary>
    /// 获取某个元素是否存在集合中
    /// </summary>
    /// <returns></returns>
    public bool Contains(T item)
    {
        return IndexOf(item) != -1;
    }
}
```
