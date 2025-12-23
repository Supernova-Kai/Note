# AssetDatabase加载
`AssetDatabase`是一种在`Unity`编辑器下专用的资源加载系统，仅在**编辑器模式生效，打包后失效**

### 特性
- **资源路径：** 必须使用**完整路径**，使用正斜杠`/`，并且**必须包含文件扩展名**（示例：`Assets/Res/Icon/icon1.png`，必须以`Assets`开头，并且不能省略`.png`）
- **仅编辑器生效：** 仅在**编辑器环境内可用**，无法在打包后的游戏中使用，所有相关代码q请用宏`UNITY_EDITOR`进行包裹，否则打包会报错
```csharp
#if UNITY_EDITOR
    // AssetDatabase相关代码
    Sprite sprite = AssetDatabase.LoadAssetAtPath<Sprite>("Assets/Res/Icon/icon1.png");
#endif
```
- **无缓存机制：** 每次调用加载都会从硬盘读取，无内置缓存

# 常用API
### 加载单个资源
**原型**
```csharp
// 基础加载
public static Object LoadAssetAtPath(string assetPath);
// 指定类型加载
public static Object LoadAssetAtPath(string assetPath, Type type);
// 泛型加载（推荐）
public static T LoadAssetAtPath<T>(string assetPath) where T : Object;
```
**示例**
```csharp
// 加载精灵（必须带扩展名）
Sprite sprite = AssetDatabase.LoadAssetAtPath<Sprite>("Assets/Res/Icon/icon1.png");

// 加载预制体
GameObject prefab = AssetDatabase.LoadAssetAtPath<GameObject>("Assets/Res/Prefab/xxx.prefab");

// 加载文本文件
TextAsset text = AssetDatabase.LoadAssetAtPath<TextAsset>("Assets/Res/Text/yyy.txt");
```

---

### 批量加载资源
**原型**
```csharp
public static T[] LoadAllAssetsAtPath<T>(string assetPath) where T : Object;
public static Object[] LoadAllAssetsAtPath(string assetPath);
```
**示例**
```csharp
// 加载图集内所有精灵
Sprite[] sprites = AssetDatabase.LoadAllAssetsAtPath<Sprite>("Assets/Res/Atlas/atlas1.spriteatlas");

// 加载指定文件夹下所有预制体（需遍历文件夹）
string[] guids = AssetDatabase.FindAssets("t:GameObject", new[] { "Assets/Res/Prefab" });
List<GameObject> prefabs = new List<GameObject>();
foreach (string guid in guids)
{
    string path = AssetDatabase.GUIDToAssetPath(guid);
    prefabs.Add(AssetDatabase.LoadAssetAtPath<GameObject>(path));
}
```

---

### 查找资源
通过条件查找资源（常用于编辑器工具）

**原型**
```csharp
// 通过筛选条件查找资源GUID
public static string[] FindAssets(string filter, string[] searchInFolders = null);

// 通过路径获取资源GUID
public static string AssetPathToGUID(string assetPath);

// 通过GUID获取资源路径
public static string GUIDToAssetPath(string guid);
```
**示例**
```csharp
// 查找所有精灵类型资源（t:类型 是固定筛选语法）
string[] spriteGuids = AssetDatabase.FindAssets("t:Sprite", new[] { "Assets/Res/Icon" });

// 遍历查找结果
foreach (string guid in spriteGuids)
{
    string path = AssetDatabase.GUIDToAssetPath(guid);
    Sprite sprite = AssetDatabase.LoadAssetAtPath<Sprite>(path);
}
```

---

### 资源刷新
修改或者创建资源后，刷新引擎

**原型**
```csharp
// 刷新所有资源
public static void Refresh();
```
**示例**
```csharp
// 创建文件后刷新
File.WriteAllText("Assets/Res/Text/new.txt", "test");
AssetDatabase.Refresh();
```

