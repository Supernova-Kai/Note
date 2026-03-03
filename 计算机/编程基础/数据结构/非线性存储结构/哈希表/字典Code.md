# 字典（Dictionary）
```csharp
public class MyDictionary<TKey, TValue>
{
    /// <summary>
    /// 字典条目
    /// </summary>
    private struct Entry
    {
        public int HashCode;
        public int Next;
        public TKey Key;
        public TValue Value;
    }

    private int[] _buckets; // 桶数组(桶数组的下标是通过Key的HashCode计算出来的桶索引;桶数组的值存的是其在条目数组对应元素的索引)
    private Entry[] _entries; // 条目数组(存储真正的数据)
    private int _count; // 实际元素数量

    private const int DefaultCapacity = 4; // 默认初始容量
    private const float LoadFactorThreshold = 0.72f; // 扩容负载因子阈值(负载因子超过该值会触发扩容)
    private readonly IEqualityComparer<TKey> _keyComparer; // 键比较器
    private readonly IEqualityComparer<TValue> _valueComparer; // 值比较器

    /// <summary>
    /// 获取字典中实际包含元素数量
    /// </summary>
    public int Count => _count;

    public MyDictionary()
    {
        _buckets = new int[DefaultCapacity];
        Array.Fill(_buckets, -1); // 初始化桶数组全部填充-1,都没有指向条目数组
        _entries = new Entry[DefaultCapacity];
        _count = 0;
        _keyComparer = EqualityComparer<TKey>.Default;
        _valueComparer = EqualityComparer<TValue>.Default;
    }

    /// <summary>
    /// 扩容
    /// </summary>
    private void ExpandCapacity()
    {
        int newCapacity = _buckets.Length * 2; // 以两倍大小库容
        int[] newBuckets = new int[newCapacity]; // 新的桶数组
        Array.Fill(newBuckets, -1); // 先全部初始化为-1
        Entry[] newEntries = new Entry[newCapacity]; // 新的条目数组

        // 拷贝旧条目数组的数据到新条目数组
        for (int i = 0; i < _count; i++)
        {
            newEntries[i] = _entries[i];
        }

        // 重新计算桶索引，重建冲突链表
        for (int j = 0; j < _count; j++)
        {
            int hashCode = newEntries[j].HashCode;
            if (hashCode != -1)
            {
                int bucketIndex = hashCode % newBuckets.Length;
                newEntries[j].Next = newBuckets[bucketIndex];
                newBuckets[bucketIndex] = j;
            }
        }

        _buckets = newBuckets;
        _entries = newEntries;
    }

    /// <summary>
    /// 检查负载因子,是否需要扩容
    /// </summary>
    private void CheckExpand()
    {
        float loadFactor = (float)_count / _buckets.Length; // 当前负载因子
        if (loadFactor >= LoadFactorThreshold) // 大于等于阈值触发扩容
            ExpandCapacity();
    }

    /// <summary>
    /// 添加键值对
    /// </summary>
    public void Add(TKey key, TValue value)
    {
        if (key == null)
            throw new ArgumentNullException(nameof(key));

        // 1. 获取Key的HashCode
        int hashCode = key.GetHashCode();

        // 2. 通过HashCode获取其在桶数组中的索引
        int bucketIndex = hashCode % _buckets.Length;

        // 3. 检查Key是否已经存在
        for (int entryIndex = _buckets[bucketIndex]; entryIndex != -1; entryIndex = _entries[entryIndex].Next)
        {
            if (_entries[entryIndex].HashCode == hashCode && _keyComparer.Equals(_entries[entryIndex].Key, key)) // 遇到HashCode相等的,说明添加了重复的Key
            {
                throw new ArgumentException($"键 {key} 已存在", nameof(key));
            }
        }
        
        // 4. 检查负载因子,是否需要扩容
        CheckExpand();

        // 5. 条目数组写入新条目(写入位置就是当前_count,条目数组的索引无意义,按顺序向后写就可以了)
        _entries[_count].HashCode = hashCode;

        // 把之前桶数组记录的索引放到这里的Next,如果之前没有使用过该索引,这里就是-1,因为桶数组初始化是-1
        // 如果之前使用过,也就是现在发生了哈希冲突,这里记录的是上一个桶索引对应的条目数组的位置,将其放到Next,形成一个链表
        _entries[_count].Next = _buckets[bucketIndex];

        _entries[_count].Key = key;
        _entries[_count].Value = value;

        // 6. 更新桶数组
        _buckets[bucketIndex] = _count; // 更新桶数组,指向新的元素
        _count++;
    }

    /// <summary>
    /// 移除指定Key的元素
    /// </summary>
    public bool Remove(TKey key)
    {
        if (key == null)
            throw new ArgumentNullException(nameof(key));

        int hashCode = key.GetHashCode();
        int bucketIndex = hashCode % _buckets.Length;
        
        int lastIndex = -1; // 记录链表上一个索引
        for (int entryIndex = _buckets[bucketIndex]; entryIndex != -1; entryIndex = _entries[entryIndex].Next)
        {
            if (_entries[entryIndex].HashCode == hashCode && _keyComparer.Equals(_entries[entryIndex].Key, key)) // 条目数组中找到了该元素
            {
                if (lastIndex == -1) // 删除的是头节点：桶指向当前节点的下一个节点
                {
                    _buckets[bucketIndex] = _entries[entryIndex].Next;
                }
                else
                {
                    // 删除的是中间/尾节点：上一个节点指向当前节点的下一个节点
                    _entries[lastIndex].Next = _entries[entryIndex].Next;
                }

                _entries[entryIndex] = default;
                _count--;
                return true;
            }
            
            lastIndex = entryIndex;
        }

        return false;
    }

    /// <summary>
    /// 清空字典
    /// </summary>
    public void Clear()
    {
        Array.Fill(_buckets, -1);
        Array.Clear(_entries, 0, _count);
        _count = 0;
    }

    /// <summary>
    /// 是否包含指定键
    /// </summary>
    /// <param name="key">键</param>
    /// <returns>包含返回true，否则false</returns>
    public bool ContainsKey(TKey key)
    {
        if (key == null)
            throw new ArgumentNullException(nameof(key));

        int hashCode = key.GetHashCode();
        int bucketIndex = hashCode % _buckets.Length;
        
        for (int entryIndex = _buckets[bucketIndex]; entryIndex != -1; entryIndex = _entries[entryIndex].Next)
        {
            if (_entries[entryIndex].HashCode == hashCode && _keyComparer.Equals(_entries[entryIndex].Key, key))
                return true;
        }

        return false;
    }

    /// <summary>
    /// 是否包含指定值
    /// </summary>
    /// <param name="value">值</param>
    /// <returns>包含返回true，否则false</returns>
    public bool ContainsValue(TValue value)
    {
        // 遍历所有有效条目匹配值
        for (int i = 0; i < _count; i++)
        {
            var v = _entries[i].Value;
            if (_valueComparer.Equals(v, value))
                return true;
        }

        return false;
    }

    /// <summary>
    /// 索引器（获取/设置指定键的值）
    /// </summary>
    /// <param name="key">键</param>
    /// <returns>对应的值</returns>
    public TValue this[TKey key]
    {
        get
        {
            if (key == null)
                throw new ArgumentNullException(nameof(key));

            int hashCode = key.GetHashCode();
            int bucketIndex = hashCode % _buckets.Length;
            
            for (int entryIndex = _buckets[bucketIndex]; entryIndex != -1; entryIndex = _entries[entryIndex].Next)
            {
                if (_entries[entryIndex].HashCode == hashCode && _keyComparer.Equals(_entries[entryIndex].Key, key))
                    return _entries[entryIndex].Value;
            }

            throw new KeyNotFoundException($"Key is not exists");
        }
        set
        {
            if (key == null)
                throw new ArgumentNullException(nameof(key));

            int hashCode = key.GetHashCode();
            int bucketIndex = hashCode % _buckets.Length;
            
            for (int entryIndex = _buckets[bucketIndex]; entryIndex != -1; entryIndex = _entries[entryIndex].Next)
            {
                if (_entries[entryIndex].HashCode == hashCode && _keyComparer.Equals(_entries[entryIndex].Key, key))
                {
                    _entries[entryIndex].Value = value;
                    return;
                }
            }

            // 键不存在则添加
            Add(key, value);
        }
    }
}
```
