# 说明

完成 cs61c projects 期间的笔记。

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

以上两个结构体定义，`Color` 用来保存一个 RGB 颜色，`Image` 用来保存一个 ppm 类型的图片。

`Image` 结构体定义中的 `Color **image` 是表示二维像素的一种手法，它是一个指针，这个指针指向一个数组，这个数组元素的类型是 `Color*`，所以这个指针的类型是 `Color **image`，其结构可视为：

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

上述代码中：

`img->image[i]`: 是指向 Color 的指针（`Color*`)，表示一个 Color 结构体的数组
`img->image[i][j]`: 是一个 Color 结构体

初始化变量时，要清楚在栈还是堆中申请内存，举例：

```c
// arr 是个数组，元素是结构体
// 在栈上申请内存，会自动回收
// 占用了 2*sizeof(Color) 字节的栈内存
struct Color arr[2];
arr[0].R = arr[0].G = arr[0].B = 255;

// arr 是个数组，元素是指针（指针指向结构体）
// 数组元素被两个 malloc 初始化
// 占用了 2*sizeof(Color*) 字节的栈内存，2*sizeof(Color) 字节的堆内存
struct Color *arr2[2];
arr2[0] = (struct Color *)malloc(sizeof(struct Color));
arr2[1] = (struct Color *)malloc(sizeof(struct Color));

// 在堆上申请了 2 个 Color 大小的连续内存空间
// arr3 是个指针，指向第一个 Color 结构体
// 这比上面那种更高效，因为申请到了连续的内存块
// arr3 又可看成一个数组，因为 arr3[1] 等价于 &(arr3+1)
Color *arr3 = malloc(2 * sizeof(Color));
arr3[0].R = arr3[0].G = arr3[0].B = 255;
```