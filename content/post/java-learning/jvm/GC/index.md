+++
date = '2026-01-21T20:01:20+08:00'
draft = true
title = 'GC'
description = 'JVM 垃圾回收机制'
categories = ['Java 学习']
tags = ['JVM', 'GC']

+++

> 常见面试题：如何判断对象是否死亡（两种方法）。
>
> - 简单的介绍一下强引用、软引用、弱引用、虚引用（虚引用与软引用和弱引用的区别、使用软引用能带来的好处）。
> - 如何判断一个常量是废弃常量如何判断一个类是无用的类垃圾收集有哪些算法，各自的特点？
> - HotSpot 为什么要分为新生代和老年代？
> - 常见的垃圾回收器有哪些？
> - 介绍一下 CMS,G1 收集器。
> - Minor Gc 和 Full GC 有什么不同呢？

## 堆空间的基本结构

java堆是垃圾收集器管理的主要区域

jdk7及之前的版本，堆被分为三个区域，新生代，老生代，永久代。

8以后的版本，永久代被移动到元空间，使用的是直接内存

![image-20260121205613143](http://cdn.cscat.cn/markdown/image-20260121205613143.png)

## GC 分类

新生代收集

老生代收集

混合收集

整堆收集

## 空间分配担保

就是在 Minor GC 前，检查一下老年代最大可用连续空间是否大于新生代所有的对象的总空间，如果成立就允许，然后还有一个jvm参数是允许冒险，再不成立的情况下，检查老年代最大可用的连续空间是否大于历次晋升到老年代对象的平均大小。

## 死亡对象的判断方法

### ~~引用计数器~~ 

算法简单，但是没法解决循环引用

### 可达性分析算法

将一系列 GC Roots 作为起点，进行遍历，如果一个对象到 GC Roots 没有任何引用链相连的话，此对象就是不可用的，需要被回收。

#### 可以作为 GC roots 的对象

- 虚拟机栈中引用的对象，也就是栈帧中的局部变量
- Native方法的引用对象
- 方法区中类静态属性引用的对象 static
- 方法区中常量引用的对象 final
- jni 引用的对象
- 锁持有的对象

### 对象可以被回收，就代表一定会被回收吗？

可以被回收，但不一定被回收，具体取决于垃圾回收算法，一个是垃圾回收算法还没有执行，那么当然不会被回收，即使发生了垃圾回收，也可能因为某些原因，导致不回收，比如这个对象的回收优先级没有其他的高，还由于Finalize方法，当一个对象有finalize方法时，被标记为垃圾时 不会立即回收，finalize方法的调用是异步的，如果这个对象在finalize方法中创建了引用，那就能实现复活。

## 引用类型介绍

![Java 引用类型总结](https://oss.javaguide.cn/github/javaguide/java/jvm/java-reference-type.png)

实际开发过程中，强引用和软引用用的会比较多，其中软引用的案例如下

```java
import java.awt.image.BufferedImage;
import java.lang.ref.SoftReference;
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;

public class ImageCache {

    // 使用 ConcurrentHashMap 来存储软引用，确保线程安全
    // Key: 图片的标识符 (如文件名或ID)
    // Value: 图片的 BufferedImage 的 SoftReference
    private final Map<String, SoftReference<BufferedImage>> cache = new ConcurrentHashMap<>();

    /**
     * 从缓存获取图片，如果缓存中没有或缓存已失效，则从源加载
     * @param imageId 图片标识符
     * @return BufferedImage 对象
     */
    public BufferedImage getImage(String imageId) {
        // 1. 尝试从缓存中获取
        SoftReference<BufferedImage> softRef = cache.get(imageId);

        // 2. 检查软引用是否仍然持有对象
        if (softRef != null) {
            BufferedImage image = softRef.get(); // 尝试获取对象
            if (image != null) {
                System.out.println("Cache hit for: " + imageId);
                return image; // 缓存命中，直接返回
            } else {
                // 软引用指向的对象已经被 GC 回收了
                System.out.println("Cache expired for: " + imageId + ", clearing soft reference.");
                cache.remove(imageId); // 清理失效的软引用
            }
        }

        // 3. 缓存未命中或已失效，需要从源加载
        System.out.println("Loading image from source for: " + imageId);
        BufferedImage newImage = loadImageFromSource(imageId); // 假设这是一个加载图片的耗时方法

        // 4. 如果成功加载了图片，将其放入缓存（通过软引用）
        if (newImage != null) {
            cache.put(imageId, new SoftReference<>(newImage));
            System.out.println("Image loaded and cached: " + imageId);
        }

        return newImage;
    }

    /**
     * 模拟从磁盘或其他源加载图片的方法
     * @param imageId
     * @return
     */
    private BufferedImage loadImageFromSource(String imageId) {
        // 这里是模拟耗时操作，实际开发中会读取文件、网络等
        System.out.println("Simulating loading image: " + imageId + "...");
        try {
            Thread.sleep(200); // 模拟加载延迟
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }
        // 假设加载成功，返回一个BufferedImage对象
        // 实际会返回读取文件后的 BufferedImage
        return new BufferedImage(100, 100, BufferedImage.TYPE_INT_RGB); // 示例图片
    }

    /**
     * 用于手动清理缓存中的失效软引用，虽然GC会自动处理，但有时主动清理可以避免map过大
     * 在实际应用中，清理失效引用的逻辑可能更复杂，例如在一个后台线程中定期执行
     */
    public void cleanupInvalidReferences() {
        cache.entrySet().removeIf(entry -> entry.getValue().get() == null);
    }

    public static void main(String[] args) {
        ImageCache imageCache = new ImageCache();

        // 模拟加载两次同一张图片
        System.out.println("--- First load ---");
        BufferedImage img1 = imageCache.getImage("logo.png");
        System.out.println("Got image: " + img1);

        System.out.println("\n--- Second load (should be from cache) ---");
        BufferedImage img2 = imageCache.getImage("logo.png");
        System.out.println("Got image: " + img2);

        // 模拟内存压力，强制 GC
        System.out.println("\n--- Forcing GC to simulate memory pressure ---");
        System.gc(); // 建议 GC 运行 (不保证一定执行，也不保证执行频率)
        try {
            Thread.sleep(500); // 给 GC 一点时间
        } catch (InterruptedException e) {
            Thread.currentThread().interrupt();
        }

        // 再次尝试加载，理论上如果内存足够，img1 仍会存在
        // 但如果内存真的很紧张，GC 可能会回收 img1 (SoftReference 的对象)
        System.out.println("\n--- Load after GC ---");
        BufferedImage img3 = imageCache.getImage("logo.png");
        System.out.println("Got image: " + img3);

        // 模拟另一个对象占用大量内存，迫使 GC 回收软引用对象
        System.out.println("\n--- Simulating heavy memory usage ---");
        // 创建一个很大的 byte 数组，模拟大量内存占用
        try {
            // 调整这个数值，尝试让 JVM 内存不足
            byte[] largeArray = new byte[1024 * 1024 * 50]; // 50MB
            System.out.println("Created large byte array to put pressure on memory.");

            System.gc(); // 再次建议 GC
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                Thread.currentThread().interrupt();
            }

            // 此时，如果没有其他强引用指向 logo.png 的 BufferedImage，
            // 并且 JVM 内存确实不足，GC 应该会回收它。
            System.out.println("\n--- Load after heavy memory usage ---");
            BufferedImage img4 = imageCache.getImage("logo.png");
            System.out.println("Got image: " + img4);

        } catch (OutOfMemoryError e) {
            System.err.println("OutOfMemoryError encountered. This might happen if the soft reference was already cleared or allocation failed.");
        }
    }
}
```

用一个 hashMap 来存储缓存，key是图片的标识，value是图片的软引用，

从cache 中查找 图片对应的软引用，如果非空，说明建立过缓存，就尝试获取图片，如果为空，说明曾经创建过的图片的缓存已经被回收，此时需要移除这个失效的软引用。
