# 字典树（Trie）

```csharp
/// <summary>
/// 字典树节点
/// </summary>
public class TrieNode
{
    public char c;
    public Dictionary<char, TrieNode> child;
    public bool isEnd;

    public TrieNode(char c)
    {
        this.c = c;
        child = new Dictionary<char, TrieNode>();
        isEnd = false;
    }
}

/// <summary>
/// 字典树
/// </summary>
public class TrieTree
{
    private TrieNode _root; // 字典树根节点

    public TrieTree()
    {
        _root = new TrieNode('\0');
    }

    public void Insert(string s)
    {
        TrieNode currentNode = _root;
        foreach (char c in s)
        {
            if (!currentNode.child.ContainsKey(c))
            {
                currentNode.child[c] = new TrieNode(c);
            }

            currentNode = currentNode.child[c];
        }

        currentNode.isEnd = true;
    }

    public bool Search(string s)
    {
        TrieNode currentNode = _root;
        foreach (char c in s)
        {
            if (!currentNode.child.ContainsKey(c))
                return false;
            
            currentNode = currentNode.child[c];
        }

        return currentNode.isEnd;
    }
    
    public bool StartsWith(string prefix)
    {
        TrieNode currentNode = _root;
        foreach (char c in prefix)
        {
            if (!currentNode.child.ContainsKey(c))
                return false;
            
            currentNode = currentNode.child[c];
        }

        return true;
    }
}
```