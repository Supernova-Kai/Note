# Resources加载
`Resources`是`Unity`中内置的一种资源动态加载系统

### 特性

- **资源路径：** 使用`Resources`加载方式的资源，**必须**放在`Asset`目录下的`Resources`文件夹下（支持多个，打包时会自动包含该文件夹内所有资源）
- **路径使用规则：** 加载路径**不需要**包含`Resources`的前缀，并且**可以省略资源的文件拓展名**（比如`Resources/Icon/icon1.png`仅需要写`Icon/icon1`）
- **缓存机制：** `Resources`加载本身有内部缓存机制，当多次调用`Load`时，会先检查内存中是否已存在该资源的实例，如果内存中已经存在，则直接返回该实例，不会从硬盘重新加载

**注意：**
目前所说的“加载”指的都是讲资源文件从硬盘加载到内存中，并不是常理解的显示到UI界面上，将资源显示到UI界面上展示一般称之为“实例化”，这两种情况最好用规范的说法，以免混淆

# 优缺点
### 优点

- **使用简单**：只需要相对路径即可加载
- **跨平台：** 不同平台如何从硬盘提取资源以及不同的路径规则，Unity本身已处理好，无需关心
- **动态加载：** 该方式实现了运行时的动态加载，不需要像传统方式那样将资源挂载到场景或脚本中
- **内置缓存：** 重复加载同一资源会优先读取内存缓存，减少硬盘IO

### 缺点

- **包体大小：** 所有放在`Resources`文件夹下的资源，打包时都会包含在内，增大了安装包的体积
- **不支持热更新：** `Resources`内的资源会成为安装包的一部分，无法通过热更新的方式进行更新，如果需要更新，只能通过下载最新的安装包，覆盖安装新包的方式来更新
- **资源不会自动卸载：** 资源卸载必须手动管理，如果忘记卸载，容易造成内存泄漏
- **无法分包：** 所有`Resources`资源打包后时一个整体，无法按需下载、加载

# 注意事项

- **路径和文件名区分大小写：** 在`Unity`编辑器中，有一种假象，那就是`Resources`加载的路径是不区分大小写的，但是实际上，是否区分大小写是根据平台的，**不论是`Android`还是`ios`平台，都是严格区分大小写的**，所以在实际使用中，还是严格按大小写使用最好
- **同名资源：** 由于`Unity`允许文件名相同但是类型不同的资源，所以使用`Load(path)`这种没有传入筛选类型的方法，可能会加载到不是我们想要的资源，尽量使用泛型方法

- **正斜杠：** `Unity`中对于所有路径，都使用正斜杠（即使在Windows上）

# 核心API
### 同步加载单个资源

**原型**
```csharp
// 非泛型写法
// path - 指定路径
// systemTypeInstance - 返回对象筛选器，传入类型，会只返回该类型的资源
public static Object Load(string path);
public static Object Load(string path, Type systemTypeInstance);

// 泛型
public static T Load<T>(path);
```

**使用示例**

```csharp
// 将icon1加载并实例化到Image上
        
// 非泛型
UnityEngine.Object obj = Resources.Load("Icon/icon1");
_image.sprite = obj as Sprite;
        
// 泛型
_image.sprite = Resources.Load<Sprite>("Icon/icon2");


// 加载预制体
GameObject prefab = Resources.Load<GameObject>("Prefab/xxx");

// 加载文本文件
TextAsset text = Resources.Load<TextAsset>("Text/yyy");

```

**注意**

同步加载适合用于小型资源，在加载大型资源时，容易导致阻塞主线程

---

### 异步加载单个资源
异步加载资源，不会阻塞主线程，适合大型资源（模型、场景等），异步加载返回的是`ResourceRequest`类型的对象，最好配合协程使用

**原型**
```csharp
// 非泛型
public static ResourceRequest LoadAsync(string path);
public static ResourceRequest LoadAsync(string path, Type systemTypeInstance);

// 泛型
public static ResourceRequest LoadAsync<T>(string path);
```

**使用示例**
```csharp
public class ResourcesAsyncLoad : MonoBehaviour
{
    void Start()
    {
        // 启动异步加载协程
        StartCoroutine(LoadModelAsync());
    }

    IEnumerator LoadModelAsync()
    {
        // 发起异步加载请求
        ResourceRequest request = Resources.LoadAsync<GameObject>("Prefabs/Model");

        // 实时显示加载进度
        while (!request.isDone) // 还没有完成前
        {
            float progress = request.progress;
            Debug.Log("加载进度：" + (progress * 100) + "%");
            yield return null;
        }

        // 加载完成后实例化
        GameObject bigModel = request.asset as GameObject;
        if (bigModel != null)
        {
            Instantiate(bigModel);
        }
    }
}
```

---

### 批量加载资源
加载指定路径下所有指定类型的资源，返回该类型的数组

```csharp
// 非泛型
public static Object[] LoadAll (string path); // 没有筛选，即加载所有资源
public static Object[] LoadAll (string path, Type systemTypeInstance);

// 泛型
public static T[] LoadAll<T>(string path) // 加载该路径下所有T类型的
```
---

### 卸载单个资源
释放内存中指定的资源对象（仅卸载资源本身，不销毁场景实例）

**原型**
```csharp
public static void UnloadAsset (Object assetToUnload);
```

**使用示例**
```csharp
// 先加载一个精灵
Sprite sprite = Resources.Load<Sprite>("Icon/icon1");
        
// 卸载该资源
Resources.UnloadAsset(sprite);
```

---
### 卸载未使用的资源
全局清理内存中所有未被代码引用的资源，可同步或者异步执行

**原型**
```csharp
AsyncOperation Resources.UnloadUnusedAssets();
```

**使用示例**
```csharp
using UnityEngine;
using System.Collections;

public class ResourcesUnloadUnused : MonoBehaviour
{
    void Start()
    {
        // 同步卸载（简单场景）
        // Resources.UnloadUnusedAssets();
        // System.GC.Collect(); // 强制GC回收

        // 异步卸载（避免卡顿）
        StartCoroutine(UnloadUnusedAsync());
    }

    IEnumerator UnloadUnusedAsync()
    {
        AsyncOperation operation = Resources.UnloadUnusedAssets();
        while (!operation.isDone)
        {
            Debug.Log("卸载进度：" + (operation.progress * 100) + "%");
            yield return null;
        }
        // 卸载完成后强制GC
        System.GC.Collect();
        Debug.Log("未使用资源卸载完成！");
    }
}
```

---
### 查找内存中所有资源
查找内存中所有指定类型的资源（包括未激活、隐藏的），用于调试 / 内存检测
```csharp
public static Object[] FindObjectsOfTypeAll (Type type);
public static T[] FindObjectsOfTypeAll<T>();
```

