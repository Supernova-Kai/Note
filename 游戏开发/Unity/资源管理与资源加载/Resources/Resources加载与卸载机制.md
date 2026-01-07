# Resources加载机制

在《Resources加载》中提到：“`Resources`有内部缓存机制，其在将资源从硬盘加载到内存时，会优先检查内存中是否已经存在该资源的示例，如果存在，则直接返回该实例，不会重新从硬盘加载”

假设有下面这一段代码
```csharp
using UnityEngine;
using UnityEngine.UI;

public class UIMain : MonoBehaviour
{
    private Sprite sprite1;
    private Sprite sprite1;
    
    private void Start()
    {
        sprite1 = Resources.Load<Sprite>("icon");
        sprite2 = Resources.Load<Sprite>("icon");
    }
}
```

那么问题来了，此时sprite1和sprite2指向的是否使同一块内存，答案是肯定的，由于Resources在加载资源时优先检查内存中是否已存在该资源的实例，所有即使多次调用加载，也不会导致内存中出现多份该资源的实例

在上面代码中的sprite1和sprite2指向的是内存中的同一块地址

# Resources卸载机制

我们知道，`Resources.UnloadAsset`可以用于将资源从内存中卸载，但是其本身是有检查机制的，总结起来就是一句话：“只有当没有任何变量和属性**持有**该资源的引用时，才能够卸载”

还是以代码为例
```csharp
using UnityEngine;
using UnityEngine.UI;

public class UIMain : MonoBehaviour
{
    private Sprite sprite1;
    private Sprite sprite1;
    private Image image;
    
    private void Start()
    {
        sprite1 = Resources.Load<Sprite>("icon"); // sprite1持有了icon资源的引用
        sprite2 = Resources.Load<Sprite>("icon"); // sprite2持有了icon资源的引用

        image.sprite = sprite; // image也持有了icon资源的引用

        Resources.UnloadAsset(sprite1); // 此时卸载，无效，因为还有变量持有icon资源的属性

        // 先清除持有了icon引用的变量和属性
        sprite1 = null;
        sprite2 = null;
        image.sprite = null;

        Resources.UnloadAsset(sprite1); // 此时卸载，有效，因为已经没有持有icon资源引用的变量和属性了

    }
}
```