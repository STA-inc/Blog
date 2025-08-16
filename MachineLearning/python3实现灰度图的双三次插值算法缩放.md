场景：
根据某需求得到大量宽高不一的灰度图，形式为:
[
    [1,2,3,4]
    [5,6,7,8]
    [9,10,11,12]
]
的二维数组。
需要对其进行标准化转换为固定大小的尺寸，在此使用双三次插值算法实现，对网上的代码略作修改：
原博客：
[双三次插值算法详解 含python实现](https://www.cnblogs.com/wojianxin/p/12516762.html)
在这里感谢这位大佬。
```python
from PIL import Image
import numpy as np
import math


# 产生16个像素点不同的权重
def BiBubic(x):
    x = abs(x)
    if x <= 1:
        return 1 - 2 * (x ** 2) + (x ** 3)
    elif x < 2:
        return 4 - 8 * x + 5 * (x ** 2) - (x ** 3)
    else:
        return 0
# 双三次插值算法
# dstH为目标图像的高，dstW为目标图像的宽
def BiCubic_interpolation(img, dstH, dstW):
    scrH, scrW = img.shape
    # img=np.pad(img,((1,3),(1,3),(0,0)),'constant')
    retimg = np.zeros((dstH, dstW, 3), dtype=np.uint8)
    for i in range(dstH):
        for j in range(dstW):
            scrx = i * (scrH / dstH)
            scry = j * (scrW / dstW)
            x = math.floor(scrx)
            y = math.floor(scry)
            u = scrx - x
            v = scry - y
            tmp = 0
            for ii in range(-1, 2):
                for jj in range(-1, 2):
                    if x + ii < 0 or y + jj < 0 or x + ii >= scrH or y + jj >= scrW:
                        continue
                    tmp += img[x + ii, y + jj] * BiBubic(ii - u) * BiBubic(jj - v)
            retimg[i, j] = np.clip(tmp, 0, 255)
    return retimg

im_path = 'F:/42.0.png'
image = np.array(Image.open(im_path))
print(image.shape[1])
# 举例：将图片统一转换为256*256的图片
image2 = BiCubic_interpolation(image, 256, 256)
image2 = Image.fromarray(image2.astype('uint8')).convert('RGB')
image2.save('F:/BiCubic_interpolation.jpg')
```

