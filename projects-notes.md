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

初始化变量时，要清楚是在栈还是堆中申请内存，举例：

```c
// arr 是个数组，数据元素是结构体，申请了 2*sizeof(Color) 字节的栈内存
// 在栈上申请内存，会自动回收
struct Color arr[2];
arr[0].R = arr[0].G = arr[0].B = 255;

// arr 是个数组，元素是指针（指针指向结构体）
// 申请了 2*sizeof(Color*) 字节的栈内存，2*sizeof(Color) 字节的堆内存
// 数组元素被两个 malloc 初始化
struct Color *arr2[2];
arr2[0] = (struct Color *)malloc(sizeof(struct Color));
arr2[1] = (struct Color *)malloc(sizeof(struct Color));

// 在堆上申请了 2 个 Color 大小的连续内存空间
// arr3 是个指针，指向第一个 Color 结构体，可以看到 malloc 申请的也是和数组一样的连续内存空间
// 这比上面那种更高效，因为申请到了连续的内存块
// arr3 又可看成一个数组，因为 arr3[1] 等价于 &(arr3+1)
Color *arr3 = malloc(2 * sizeof(Color));
arr3[0].R = arr3[0].G = arr3[0].B = 255;
```

**根据0x1808算出下一帧的图像**

简要说明 0x1808 规则：
[rule-0x1808](./images/rule-0x1808.jpg)

- 如果我当前状态是 dead(全黑)，如果周围8个像素有3个是alive的，那么我下一个状态是 alive，其它任何情况是dead；
- 如果我当前状态是 alive(非黑色)，如果周围8个像素有2个或3个是alive，那么我下一个状态是 alive，其它任何情况是dead；

上面提到，图像像素用一个二维数组保存，每个颜色用一个 Color 结构体保存。这里要实现的是，根据规则 0x1808 算出这个Color 下一帧的颜色，实现它的函数是：

```c
Color *evaluateOneCell(Image *image, int row, int col, uint32_t rule) {
  Color *new_color = (Color *)malloc(sizeof(Color));
  if(new_color == NULL) return NULL;

  uint8_t red = image->image[row][col].R;
  uint8_t green = image->image[row][col].G;
  uint8_t blue = image->image[row][col].B;

  int curr_state = 1;
  int LIVE_OFFSET = 9;
  if(red == 0 && green == 0 && blue == 0) {
    curr_state = 0;
  }

  ColorCount *cCount = alive_neighbor(image, row, col);
  int next_state = (rule >> (cCount->count + curr_state*LIVE_OFFSET)) & 1;

  // next_state is either 1 or 0
  if(next_state == 0) {
    new_color->R = new_color->G = new_color->B = 0;
  } if (next_state == 1) { // should use the neighbor's color
    new_color->R = cCount->R;
    new_color->G = cCount->G;
    new_color->B = cCount->B;
  }
  
  free(cCount);
  return new_color;
}
```

- 1. 算出当前状态 dead 还是 alive
- 2. 算出8个有几个alive `ColorCount *cCount = alive_neighbor(image, row, col);`
- 3. 进行移位操作，并&1，得到下一个状态。如，当前是 dead，周围有 3 个 alive，将 `0x1808` 右移3位，并 &1，得到1，说明下个状态是 alive。如果当前 alive，移位的时候要加上一个9的offset。

写完后试了一下，从一个初始 ppm 图像文件，根据规则 0x1808，算出后续 1-100 帧的图片，制成 gif 图，如下：
[GliderGuns](./images/GliderGuns.gif)