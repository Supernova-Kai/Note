# 栈（Stack）
```csharp
public class MyStack<T>
{
    private T[] _array;
    private int _top; // 栈顶指针，指向"下一个要插入元素的位置"，栈顶元素索引为 _top - 1

    /// <summary>
    /// 默认初始大小
    /// </summary>
    private const int DefaultCapacity = 4;
    
    /// <summary>
    /// 获取栈中实际包含的元素数量
    /// </summary>
    public int Count => _top;
    
    /// <summary>
    /// 获取栈的当前容量（可容纳的元素数）
    /// </summary>
    public int Capacity => _array.Length;
    
    public MyStack()
    {
        _array = new T[DefaultCapacity];
        _top = 0;
    }
    
    /// <summary>
    /// 扩容
    /// </summary>
    private void ExpandCapacity()
    {
        int newCapacity = _array.Length * 2;
        T[] newArray = new T[newCapacity]; // 新数组
        
        // 拷贝原数组数据到新数组
        for (int i = 0; i < _top; i++)
        {
            newArray[i] = _array[i];
        }
        
        _array = newArray;
    }

    public void Push(T item)
    {
        if (_top == _array.Length)
            ExpandCapacity();
        
        _array[_top] = item;
        _top++;
    }

    public T Pop()
    {
        if (_top == 0)
            throw new InvalidOperationException("Stack is empty");
        
        T item = _array[_top - 1];
        _array[_top - 1] = default(T);
        _top--;
        return item;
    }

    public T Peek()
    {
        if (_top == 0)
            throw new InvalidOperationException("Stack is empty");
        
        return _array[_top - 1];
    }

    public bool Contains(T item)
    {
        for (int i = 0; i < _top; i++)
        {
            if (_array[i].Equals(item))
                return true;
        }
        return false;
    }
}
```
