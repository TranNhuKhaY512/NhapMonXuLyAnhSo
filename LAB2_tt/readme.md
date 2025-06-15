## **File readme** phần bài tập **2, 3, 4**
### Các công nghệ được sử dụng
```python
from PIL import Image
import os
import numpy as np
import matplotlib.pyplot as plt
import scipy.fftpack
import math
import random #thêm vào ở bài 3, 4
from scipy.ndimage import minimum_filter, maximum_filter # thêm vào ở bài 4
```
### Trong bài nay sử dụng các thư viện như PIL dùng để đọc và xử lý ảnh , Numpy hỗ trợ các thao tác trên ma trận ảnh . Matplotlib dùng để hiển thị ảnh, Scipy (fftpack) thực hiện biến đổi Fourier, os và math hỗ trợ quản lý file và các phép toán học cơ bản, random dùng để hiển thị ngẫu nhiên, scipy.ndimage dùng cho các phép lọc ảnh (minium_filter và maximum_filter).
## Thuật toán sử dụng: 
### Trong bài này có 2 thuật toán chính :
- Point Processing gồm phần nhỏ là :Image inverse transformation, Gamma-Correction, Log Transformation, Histogram equalization, Contrast Stretching.
- Biến đổi ảnh theo miền tần suất (Fourier) gồm 2 phần là: Biến đổi ảnh với Fast Fourier và lọc ảnh trong miền tần suất có 2 cái nhỏ là: Butterworth Lowpass Filter, Butterworth Highpass Filter.
## Giải thích cách hoạt động:
### Bài tập 2: Viết chương trình tạo menu cho phép người dùng chọn các phương pháp biến đổi ảnh như sau:
#### Sử dụng thuật toán biến đổi ảnh theo miền tần suất (Fourier) gồm :  Fast Fourier, Butterworth Lowpass Filter, Butterworth Highpass Filter
- Với phương pháp Fast Fourier: sử dụng thuật toán fft2 để phân tích ảnh và dùng fftshift để dịch tâm tần số về giữa ảnh.
- Định nghĩa hàm Fast_Fourier() 
![image](https://github.com/user-attachments/assets/b292c7bb-c867-40f8-ac58-c8e3a3f26f2f)



 




