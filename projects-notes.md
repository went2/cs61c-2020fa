# 说明

完成 projects 期间的心得记录。

## project 1 Conway's Game of Life

**二维数组的处理**

如果一组数据可以组织成矩形（每行数据都有相同的列），可以用二维数据进行表示。

二维数组：元素是数组的数组，[ [x,y,z], [a,b,c], [d,e,f], [l,m,n] ]，表示一个有 4 个元素的二维数组，每个元素各自是数组，包含 3 个元素。暂且将二维数组称为外层数组，其元素称为内层数组。

矩形与二维数组的映射关系是：
  - 外层数组的长度映射到矩形的高（或行数 rows）
  - 内层数组的长度映射为矩形的宽（或列数 columns）

循环数组时，外层循环高，内层循环宽，如:

```c
for(int i=0; i<rows; i++) {
  // 此处会循环每一行
  for(int j=0; j<columns; j++) {
    // 此处会循环i行的所有列
  }
}
```

**如何定义一个动态的二维数组**

C语言中声明一个数组一般都要给定数组的长度，以便在声明时申请内存空间。那么如何声明一个动态的二维数组，在运行时进行初始化？看例子：

```c
typedef struct Color {
  uint8_t R;
  uint8_t G;
  uint8_t B;
} Color;

typedef struct Image {
  Color **image;
  uint32_t rows;
  uint32_t cols;
} Image;
```

两个结构体定义，`Color` 保存一个 RGB 颜色，`Image` 保存一个 ppm 类型的图片。

`Image` 结构体定义中的 `Color **image` 是表示二维像素的一种手法，它是一个指针，这个指针指向一个数组，这个数组元素的类型是 `Color*`(指向 Color 的指针)，所以这个（指针数组的）指针的类型是 `Color **image`，其结构可视为：

```
Image
 |
 +-- Color **image ---> [Color *ptr][Color *ptr][Color *ptr]...
                           |           |           |
                           v           v           v
                       [Color]      [Color]     [Color]...
                       [Color]      [Color]     [Color]...
                       [Color]      [Color]     [Color]...
                          .            .           .
                          .            .           .
```

动态初始化 Image 时，用以下过程：

```c
Image *image = malloc(sizeof(Image));
img->rows = height;
img->cols = width;
// img->image 是外层数组，外层数组保存的是 height 个数的指针
img->image = malloc(height * sizeof(Color*));
for(int i=0; i<height; i++) {
  img->image[i] = malloc(width * sizeof(Color));
}
```