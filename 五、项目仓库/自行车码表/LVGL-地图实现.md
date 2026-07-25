#LVGL 
#实操/开发/嵌入式/STM32/项目/自行车码表 


### 定位实现
	void latlon_to_world_pixel(double lat,double lon,int zoom,uint32_t *x,uint32_t *y)
```c
    double mapSize = 256 * (1 << zoom);  //[!Describe]Google规定：一张Tile是256*256Px
    *x = (lon + 180.0) / 360.0 * mapSize;
    double latRad = lat * M_PI / 180.0;
    *y =
    (1.0 - 
    log(tan(latRad)
    +1.0/cos(latRad))/M_PI)
    /2.0
    *mapSize;
```
这个功能Google使用了`Mercator投影`进行实现。因为地球是圆形，而我们的屏幕是一个矩形。需要对坐标进行转换，传入的经纬度需要被转换为屏幕上的定位。


### 缩放实现

> World Pixel 世界坐标
     我们在GUI界面显示地图定位时，需要了解到这个概念。在实际编写中，我们给图片以像素点为单位，建立起坐标系。由此我们的位置就是(x,y)


> [!NOTE] 计算公式
> 
```
	世界宽 = 256 × 2^Zoom
```



想要实现在屏幕上缩放，先来看一下缩放的逻辑：

其实“缩放”这个操作并不是我们想象中的直接把图片放大，而是由：
```
zoom = 1
+---------+---------+
| (0,0)   | 深圳     |
+---------+---------+
| (0,1)   | (1,1)   |
+---------+---------+
```
这样一个4x4的图片矩阵进一步切成了：
```
zoom = 2
+---+---+---+---+
|0,0|1,0|深圳|3,0|
+---+---+---+---+
|0,1|1,1|2,1|3,1|
+---+---+---+---+
|0,2|1,2|2,2|3,2|
+---+---+---+---+
|0,3|1,3|2,3|3,3|
+---+---+---+---+
```

那么这之中的一个小瓦片呢，就是一个`Tile`。

> [!NOTE] Tile的编号变化
> 如图，在zoom=1时，深圳位于`(1,0)`，放大之后深圳的位置并没有变化，但是`Tile`值变为了`(2,0)`

那么问题来了，Tile的值到底是怎么变化的呢？
同一个GPS点，当
- zoom = 1时：
```c 
word_x = 400;
```
- zoom = 2时：
```c
world = 800;
```

这是因为，在zoom变大时，**`World Pixel`也随之变大了**

那么我们观察一下下载的瓦片地图的目录结构：


```c
sprintf(
    imagePath,
    "S:/%d/%d/%d/tile.bmp",
    zoom,
    tile_x,
    tile_y
);
```
通过这一行我们就能得到目标图片的路径：
> zoom = 14
> tile_x = 13372
> tile_y = 7134

最终路径：
```
S:/14/13372/7134/tile.bmp
```

### 位移刷新实现


	static void Refresh_Map(uint32_t center_tile_x, uint32_t center_tile_y, uint8_t zoom)
```c
  for (uint8_t i = 0; i < 9; i++) {
    uint32_t current_tile_x, current_tile_y;
    current_tile_x = center_tile_x + (i % 3) - 1;
    current_tile_y = center_tile_y + (i / 3) - 1;
    load_tile(image[i], current_tile_x, current_tile_y, zoom);
  }
```