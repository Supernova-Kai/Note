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

# 代码示例
```csharp
// 加载精灵（必须带扩展名）
Sprite sprite = AssetDatabase.LoadAssetAtPath<Sprite>("Assets/Res/Icon/icon1.png");

// 加载预制体
GameObject prefab = AssetDatabase.LoadAssetAtPath<GameObject>("Assets/Res/Prefab/xxx.prefab");

// 加载文本文件
TextAsset text = AssetDatabase.LoadAssetAtPath<TextAsset>("Assets/Res/Text/yyy.txt");
```
我们可以发现这种加载方式，有一个很麻烦的点，那就是资源的路径必须是完整的路径，并且需要带文件拓展名，不过好在`AssetDatabse`提供了搜索资源的接口，可以参考下面的示例，通过资源名获取资源路径并加载

```csharp
string[] assets = AssetDatabase.FindAssets(assetName); // 通过资源名搜索资源，该方法返回一个Guid数组
if (assets.Length > 0) // 没找到会返回空数组，需要判断一下找到没
{
    string guid = assets[0];
    string assetPath = AssetDatabase.GUIDToAssetPath(guid); // Guid转路径（完整路径、带文件拓展名）
            
    myImage.sprite = AssetDatabase.LoadAssetAtPath<Sprite>; // 显示到image上
}
```