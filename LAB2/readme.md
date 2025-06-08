## Các công nghệ được sử dụng
```python
from PIL import Image
import os
import math
import scipy
import numpy as np
import imageio.v2 as iio
import matplotlib.pylab as plt

```
### Trong bài này sử dụng các thư viện như: PIL và imageio dùng để đọc và xử lý ảnh cơ bản, Numpy hỗ trợ các thao tác trên ma trận ảnh . Matplotlib dùng để hiển thị ảnh, Scipy (fftpack) thực hiện biến đổi Fourier, còn os và math hỗ trợ quản lý file và các phép toán học cơ bản.
## Thuật toán sử dụng
### Trong bài này có 2 thuật toán chính đầu tiên là Point Processing gồm phần nhỏ là :Image inverse transformation, Gamma-Correction, Log Transformation, Histogram equalization, Contrast Stretching và thuật toán thứ 2 là Biến đổi ảnh theo miền tần suất (Fourier) gồm 3 phần nhỏ là:  Fast Fourier, Butterworth Lowpass Filter, Butterworth Highpass Filter
## Giải thích cách hoạt động:
#### 1. Point Processing: thuật toán này dùng để xử lý ảnh bằng cách thay đổi giá trị của từng pixel , dựa trên giá trị cường độ ban đầu của nó.
 1.1 Biến đổi cường độ ảnh (Image inverse transformation): Là phép biến đổi đảo ngược mức sáng của ảnh từ vùng sáng sang vùng tối và ngược lại từ vùng tối sang vùng sáng. Có công thức là giá trị pixel mới (s) = giá trị cường độ tối đa (L-1) - giá trị pixel hiện tại (r). Trong bài này, s được gán là im_2, L=256 và r là im_1.
```python
im_2 = 255 - im_1
```
- 


