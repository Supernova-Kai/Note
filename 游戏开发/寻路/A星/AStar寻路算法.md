# A*算法
### 核心原理

AStar算法的核心在于，通过评估每一个坐标的 **实际代价** 与 **预估代价** 来综合评估每一个坐标，其中实际代价与预估代价的定义如下：

- 实际代价 G：从起点到当前节点的已走路径长度（常采用曼哈顿距离）
- 预估代价 H：从当前节点到终点的预估路径长度（同样采用曼哈顿距离，仅作为启发值）
- 综合评估值 F：F = G + H，F 值越小，节点优先级越高，优先搜索

### 核心数据结构
|结构|作用|
|----|-----|
|OpenList（开列表）|存储待评估的节点，优先处理 F 值最小的节点（使用优先队列）
|CloseList（闭列表）|存储已处理 / 不可访问的节点（包括障碍物、已遍历节点），避免重复访问
|父节点字典|记录每个节点的前置节点，用于最终回溯生成完整路径

### 流程
1. 初始化closeList，将障碍物加入到闭列表中
2. 初始化openList，存入起点
3. 处理openList：取 F 最小节点 → 判断是否是终点 → 标记为已处理 → 遍历相邻节点
4. 相邻节点检查合法性，跳过已处理和障碍物节点，合法的节点加入openList并记录父节点
5. 重复第3步和第4步的过程，直到找到终点，回溯路径

### 节点定义代码
```csharp
using System;

/// <summary>
/// 地图坐标点
/// </summary>
public class Point
{
    public int X; // X坐标
    public int Y; // Y坐标

    public Point(int x, int y)
    {
        X = x;
        Y = y;
    }

    public static bool operator ==(Point a, Point b)
    {
        return a.X == b.X && a.Y == b.Y;
    }

    public static bool operator !=(Point a, Point b)
    {
        return a.X != b.X || a.Y != b.Y;
    }

    public override bool Equals(object obj)
    {
        Point p = (Point)obj;
        return this == p;
    }
    
    // 重写GetHashCode，使HashCode只和X、Y坐标有关
    // 保证即使是不同地址的Point对象，只要X、Y相同，就是同样的哈希值
    public override int GetHashCode()
    {
        return HashCode.Combine(X, Y);
    }
}

/// <summary>
/// 寻路节点
/// </summary>
public class PathNode
{
    public Point point; // 地图坐标点
    public int F; // F值(该点的F值)

    public PathNode(Point point, int f)
    {
        this.point = point;
        this.F = f;
    }
}
```

### 寻路代码
```csharp
using System;
using System.Collections.Generic;

public class AStar
{
    // 方向数组 上下左右四个方向（可根据实际情况修改，比如可以斜着走）
    private static readonly int[][] _dirs = new int[4][]
    {
        new[] { 0, -1 },
        new[] { 0, 1 },
        new[] { -1, 0 },
        new[] { 1, 0 }
    };

    /// <summary>
    /// 寻找路径
    /// </summary>
    /// <param name="width">地图宽度</param>
    /// <param name="height">地图高度</param>
    /// <param name="start">起点</param>
    /// <param name="end">终点</param>
    /// <param name="obstacle">障碍物列表</param>
    /// <returns></returns>
    public static List<Point> FindPath(int width, int height, Point start, Point end, List<Point> obstacle = null)
    {
        HashSet<Point> close = new HashSet<Point>(); // 排除列表(已经过的点)
        List<PathNode> openList = new List<PathNode>(); // 待搜索的点(优先队列,用List模拟)
        Dictionary<Point, Point> parent = new Dictionary<Point, Point>(); // 节点之间的关系 Key-当前节点 Value-路径上当前节点的前置节点

        // 将障碍物添加到close中(障碍物其实就是在最初就设定好不可访问的节点)
        if (obstacle != null)
        {
            foreach (Point p in obstacle)
            {
                close.Add(p);
            }
        }

        openList.Add(new PathNode(start, CalculateF(start, start, end))); // 起点入队

        while (openList.Count > 0)
        {
            // 取出待选队列中，F值最小的
            int minFIndex = GetMinFIndex(openList);
            PathNode curNode = openList[minFIndex];
            openList.RemoveAt(minFIndex);

            Point curPos = curNode.point;

            if (curPos == end) // 已到达终点
                return BindPath(parent, end); // 回溯路径

            if (close.Contains(curPos)) // 剪枝 该点在排除列表中,跳过
                continue;

            close.Add(curPos); // 添加到排除列表中，防止后面又走到这个点

            // 拓展相邻点
            foreach (var dir in _dirs) // 遍历方向数组的四个方向
            {
                Point neighbor = new Point(curPos.X + dir[0], curPos.Y + dir[1]);

                // 超出地图范围
                if (neighbor.X < 0 || neighbor.X >= width || neighbor.Y < 0 || neighbor.Y >= height)
                    continue;

                // 该点在排除列表中（已访问过或者是障碍点）
                if (close.Contains(neighbor))
                    continue;

                // 不在openList中则加入openList
                if (!ExistsInOpen(openList, neighbor))
                {
                    parent[neighbor] = curPos;
                    openList.Add(new PathNode(neighbor, CalculateF(neighbor, start, end)));
                }
            }
        }
        return null;
    }

    /// <summary>
    /// 计算某个点的F值
    /// </summary>
    /// <param name="p">当前点</param>
    /// <param name="start">起点</param>
    /// <param name="end">终点</param>
    /// <returns></returns>
    private static int CalculateF(Point p, Point start, Point end)
    {
        int g = Math.Abs(p.X - start.X) + Math.Abs(p.Y - start.Y);
        int h = Math.Abs(p.X - end.X) + Math.Abs(p.Y - end.Y);
        return g + h;
    }

    /// <summary>
    /// 获取列表中F值最小的元素的下标
    /// </summary>
    private static int GetMinFIndex(List<PathNode> list)
    {
        int minIndex = 0;
        for (int i = 1; i < list.Count; i++)
        {
            if (list[i].F < list[minIndex].F)
                minIndex = i;
        }

        return minIndex;
    }

    // p是否在list中
    private static bool ExistsInOpen(List<PathNode> list, Point p)
    {
        foreach (var node in list)
        {
            if (node.point == p)
                return true;
        }

        return false;
    }

    /// <summary>
    /// 回溯路径
    /// </summary>
    private static List<Point> BindPath(Dictionary<Point, Point> parent, Point end)
    {
        List<Point> path = new List<Point>();
        Point cur = end;
        while (parent.ContainsKey(cur))
        {
            path.Add(cur);
            cur = parent[cur];
        }

        path.Add(cur); // 起点
        path.Reverse(); // 翻转一下，从起点到终点
        return path;
    }
}
```