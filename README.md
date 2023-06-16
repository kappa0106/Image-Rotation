# Image-Rotation
## Table of Contents
- [Image-Rotation](#image-rotation)
  - [Table of Contents](#table-of-contents)
  - [作業要求](#作業要求)
  - [I. Methods](#i-methods)
    - [1.1 Transformation Matrix](#11-transformation-matrix)
    - [1.2 Nearest-neighbor Interpolation](#12-nearest-neighbor-interpolation)
    - [1.3 Bilinear Interpolation](#13-bilinear-interpolation)
    - [1.4 Zooming Comparison](#14-zooming-comparison)
  - [II. Results and Discussion](#ii-results-and-discussion)
    - [2.1 Results](#21-results)
    - [2.2 Discussion](#22-discussion)
## 作業要求
Please select an image and rotate it by 45 degrees. An example is shown below.
You should implement this operation by using near-neighbor and bilinear interpolation methods. Bicubic interpolation is optional. You can get extra credits if you implement all three methods.
Please compare the results of these two interpolation methods and give some discussion.

![7x6Rmj.png](https://upload.cc/i1/2023/06/16/7x6Rmj.png)
![XsOcaQ.png](https://upload.cc/i1/2023/06/16/XsOcaQ.png)



## I. Methods

使用 near-neighbor interpolation 和 bilinear interpolation 旋轉圖片，使用的圖片如下

![image.jpg](https://github.com/kappa0106/Image-Rotation/blob/main/image.jpg)


### 1.1 Transformation Matrix
定義旋轉圖片的變換矩陣，將想要旋轉的角度傳入，回傳變換矩陣。

* Transformation Matrix

```math
R = \begin{bmatrix}cos\theta & -sin\theta \\ sin\theta & cos\theta \end{bmatrix} 
```

* Code
```python
def transform_matrix(angle):
    radian = np.radians(angle)
    matrix = [[np.cos(radian), -np.sin(radian)],
              [np.sin(radian), np.cos(radian)]]
    return matrix

```

### 1.2 Nearest-neighbor Interpolation
Nearest-neighbor Interpolation 是一種簡單的插值方法，它會選擇最近的像素值來估計新的像素值。不考慮周圍其他像素的值，容易導致輸出中出現塊狀和像素化外觀。

![Nearest-neighbor](https://upload.cc/i1/2023/06/16/dYgmbE.png)


* 實作步驟
使用雙重迴圈遍歷圖片的每個像素，從(0,0)開始，計算對應的原始座標，但計算出的值可能為浮點數，因此直接將計算出的值取整數。

```python
def nearest(image, matrix):
    width, height = image.size
    x_center = width / 2
    y_center = height / 2

    rotated_img = Image.new("RGB", (width, height))
    for j in range(height):
        for i in range(width):
            x = int((i - x_center) * matrix[0][0] +
                    (j - y_center) * matrix[0][1] + x_center)
            y = int((i - x_center) * matrix[1][0] +
                    (j - y_center) * matrix[1][1] + y_center)

            if 0 <= x < width and 0 <= y < height:
                rotated_img.putpixel((i, j), image.getpixel((x, y)))

    return rotated_img

```

* result

![output_nearest.jpg](https://github.com/kappa0106/Image-Rotation/blob/main/output_nearest.jpg)

### 1.3 Bilinear Interpolation
Bilinear Interpolation 會考慮相鄰的像素值來估計新的像素值。採用周圍像素的加權平均值，因此輸出的圖片會有更平滑的過渡邊緣，也保留了更多的圖片細節。

![BI](https://upload.cc/i1/2023/06/16/1ziWnV.png)
![Bilinear Interpolation](https://upload.cc/i1/2023/06/16/JYN6tA.png)


* 實作步驟
旋轉的步驟與 Nearest-neighbor Interpolation 大致相同，但去掉了取整數的步驟，計算新像素值的公式為:

```math
f(x,y) \approx f(0,0)(1-x)(1-y)+f(1,0)x(1-y)+f(0,1)(1-x)y+f(1,1)xy
```


```python
def bilinear(image, matrix):
    width, height = image.size
    x_center = width / 2
    y_center = height / 2

    rotated_img = Image.new("RGB", (width, height))
    for j in range(height):
        for i in range(width):
            x = (i - x_center) * matrix[0][0] + (j - y_center) * matrix[0][1] + x_center
            y = (i - x_center) * matrix[1][0] + (j - y_center) * matrix[1][1] + y_center

            x1, y1 = int(x), int(y)
            x2, y2 = x1 + 1, y1 + 1

            if 0 <= x1 < width and 0 <= y1 < height and 0 <= x2 < width and 0 <= y2 < height:
                dec_x, dec_y = x - x1, y - y1
                a, b, c, d = image.getpixel((x1, y1)), image.getpixel(
                    (x2, y1)), image.getpixel((x1, y2)), image.getpixel((x2, y2))

                pixel = (
                    int((1 - dec_x) * (1 - dec_y) * a[0] + dec_x * (1 - dec_y) * b[0] + (1 - dec_x) * dec_y * c[0] + dec_x * dec_y * d[0]),
                    int((1 - dec_x) * (1 - dec_y) * a[1] + dec_x * (1 - dec_y) * b[1] + (1 - dec_x) * dec_y * c[1] + dec_x * dec_y * d[1]),
                    int((1 - dec_x) * (1 - dec_y) * a[2] + dec_x * (1 - dec_y) * b[2] + (1 - dec_x) * dec_y * c[2] + dec_x * dec_y * d[2])
                )

                rotated_img.putpixel((i, j), pixel)

    return rotated_img

```

* result

![output_bilinear.jpg](https://github.com/kappa0106/Image-Rotation/blob/main/output_bilinear.jpg)
### 1.4 Zooming Comparison

為了更好的比較兩種方法的差別，寫了一個函式將局部放大，以觀察同個位置不同方法的細節。

```python
def zoom(image, x_start, y_start, zoom_factor):
    cropped_img = image.crop((x_start - zoom_factor // 2, y_start - zoom_factor //
                             2, x_start + zoom_factor // 2, y_start + zoom_factor // 2))
    return cropped_img

```

## II. Results and Discussion

### 2.1 Results
![result.jpg](https://github.com/kappa0106/Image-Rotation/blob/main/result.jpg))

### 2.2 Discussion

觀察縮放後的圖像，可以看出 Nearest-neighbor Interpolation 和 Bilinear Interpolation 兩種方法之間有蠻明顯的差異。

Nearest-neighbor Interpolation 簡單快速，但生成的圖像呈現塊狀和像素化，照片的品質較低。

而 Bilinear Interpolation 所產生的圖片保持更平滑的過渡和更精細的細節，提供視覺上更好的結果，它有效地減少了鋸齒狀邊緣、像素化和失真，但計算量更大，並且在某些情況下可能會有引入模糊或插值偽影的情形。

總而言之，Bilinear Interpolation 比 Nearest-neighbor Interpolation 表現出更好的效果，但在使用插植方法時，仍然需要根據圖片的特點及需求去選擇適合的插植方法。


