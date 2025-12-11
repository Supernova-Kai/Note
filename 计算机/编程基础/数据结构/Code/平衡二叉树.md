# 平衡二叉树（AVL Tree）
```csharp
/// <summary>
/// 平衡二叉树
/// </summary>
public class AVLTree
{
    /// <summary>
    /// 节点结构
    /// </summary>
    private class TreeNode
    {
        public int Value;      // 数据
        public TreeNode Left;  // 左子节点
        public TreeNode Right; // 右子节点
        public int Height;     // 节点高度

        public TreeNode(int value)
        {
            Value = value;
            Left = null;
            Right = null;
            Height = 0; // 初始叶子节点高度为0
        }
    }
    
    /// <summary>
    /// 根节点
    /// </summary>
    private TreeNode _root;

    public AVLTree()
    {
        _root = null;
    }

    // 获取节点高度
    private int GetHeight(TreeNode node)
    {
        if (node == null)
            return -1; // 空节点高度为-1

        return node.Height;
    }

    /// <summary>
    /// 更新节点高度
    /// </summary>
    private void UpdateHeight(TreeNode node)
    {
        if (node == null)
            return;

        node.Height = Math.Max(GetHeight(node.Left), GetHeight(node.Right)) + 1;
    }

    /// <summary>
    /// 获取平衡因子
    /// </summary>
    private int GetBalanceFactor(TreeNode node)
    {
        if (node == null)
            return 0;

        return GetHeight(node.Left) - GetHeight(node.Right);
    }

    /// <summary>
    /// 左旋操作
    /// </summary>
    /// <param name="node">失衡节点</param>
    /// <returns></returns>
    private TreeNode RotateLeft(TreeNode node)
    {
        TreeNode newRoot = node.Right; // 新根：失衡节点的右子节点
        TreeNode temp = newRoot.Left; // 暂存新根的左子节点

        // 调整父子关系
        newRoot.Left = node; // 原失衡节点变为新根的左子节点
        node.Right = temp; // 暂存的节点转为原失衡节点的右子节点

        // 更新高度
        UpdateHeight(node);
        UpdateHeight(newRoot);

        return newRoot; // 返回新根，替换原失衡节点的位置
    }

    /// <summary>
    /// 右旋操作
    /// </summary>
    /// <param name="node">失衡节点</param>
    /// <returns></returns>
    private TreeNode RotateRight(TreeNode node)
    {
        TreeNode newRoot = node.Left; // 新根：失衡节点的左子节点
        TreeNode temp = newRoot.Right; // 暂存新根的右子节点

        // 调整父子关系
        newRoot.Right = node; // 原失衡节点变为新根的右子节点
        node.Left = temp; // 暂存的节点转为原失衡节点的左子节点

        // 更新高度
        UpdateHeight(node);
        UpdateHeight(newRoot);

        return newRoot; // 返回新根，替换原失衡节点的位置
    }

    // 平衡修复：插入/删除后调用，返回修复后的子树根节点
    private TreeNode Balance(TreeNode node)
    {
        if (node == null)
            return null;

        // 1. 更新当前节点高度
        UpdateHeight(node);

        // 2. 获取当前节点平衡因子
        int balanceFactor = GetBalanceFactor(node);

        // 情况1：LL型（右旋）
        if (balanceFactor > 1 && GetBalanceFactor(node.Left) >= 0)
        {
            return RotateRight(node);
        }

        // 情况2：RR型（左旋）
        if (balanceFactor < -1 && GetBalanceFactor(node.Right) <= 0)
        {
            return RotateLeft(node);
        }

        // 情况3：LR型（先左旋左子节点，再右旋当前节点）
        if (balanceFactor > 1 && GetBalanceFactor(node.Left) < 0)
        {
            node.Left = RotateLeft(node.Left);
            return RotateRight(node);
        }

        // 情况4：RL型（先右旋右子节点，再左旋当前节点）
        if (balanceFactor < -1 && GetBalanceFactor(node.Right) > 0)
        {
            node.Right = RotateRight(node.Right);
            return RotateLeft(node);
        }

        // 未失衡，直接返回原节点
        return node;
    }

    /// <summary>
    /// 插入
    /// </summary>
    /// <param name="value">插入值</param>
    public void Insert(int value)
    {
        _root = InsertNode(_root, value);
    }

    private TreeNode InsertNode(TreeNode node, int value)
    {
        // 1. 找到了空位置,创建新节点
        if (node == null) 
        {
            return new TreeNode(value);
        }

        // 2. 处理重复值（二叉搜索树通常不允许重复）
        if (value == node.Value)
        {
            return node; // 直接返回原节点，不插入
        }

        // 3. 递归查找插入位置，并挂载新节点
        if (value > node.Value) // 插入值比当前节点值大,需要去右边
        {
            
            node.Right = InsertNode(node.Right, value); // 将递归结果赋值给node.Right，完成挂载
        }
        else // 插入值比当前节点值小,需要去左边
        {
            node.Left = InsertNode(node.Left, value);
        }
        
        return Balance(node);
    }

    /// <summary>
    /// 删除
    /// </summary>
    /// <param name="value">删除值</param>
    public void Delete(int value)
    {
        _root = DeleteNode(_root, value);
    }

    private TreeNode DeleteNode(TreeNode node, int value)
    {
        // 1. 未找到要删除的节点，返回null
        if (node == null)
        {
            return null;
        }

        // 2. 递归查找要删除的节点
        if (value < node.Value) // 删除值比当前节点值小,需要去左边
        {
            node.Left = DeleteNode(node.Left, value); //将删除后的子树重新挂载到node.Left
        }
        else if (value > node.Value) // 删除值比当前节点值大,需要去右边
        {
            node.Right = DeleteNode(node.Right, value);
        }
        else // 找到了值相等的节点
        {
            if (node.Left == null && node.Right == null) // 删除的节点没有左右子节点,直接删除该节点即可
            {
                return null;
            }
            else if (node.Left != null && node.Right == null) // 删除的节点只有左子节点
            {
                UpdateHeight(node.Left); // 更新子节点高度
                return node.Left; // 用左子节点替换当前节点
            }
            else if (node.Right != null && node.Left == null) // 删除的节点只有右子节点
            {
                UpdateHeight(node.Right);
                return node.Right; // 用右子节点替换当前节点
            }
            else // 删除的节点既有左子节点,又有右子节点
            {
                int minValue = MinValue(node.Right); // 找出该节点右子树中的最小值
                node.Value = minValue; // 用该最小值替换删除节点的值
                node.Right = DeleteNode(node.Right, minValue); // 记得去右子树把最小节点删了
            }
        }
        
        return Balance(node);
    }
    
    /// <summary>
    /// 寻找某个节点的子树中的最小值
    /// </summary>
    private int MinValue(TreeNode node)
    {
        int minValue = node.Value;
        // 二叉搜索树左子树值更小，循环找最左节点
        while (node.Left != null)
        {
            minValue = node.Left.Value;
            node = node.Left;
        }
        return minValue;
    }
   
}
```