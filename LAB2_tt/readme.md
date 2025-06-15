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
- Với phương pháp Fast Fourier: sử dụng thuật toán fft2 để phân tích ảnh và dùng fftshift để dịch tâm tần số về giữa ảnh. Trong bài này, dùng fftshift để dịch tâm tần số về giữa, chuẩn hóa về thang [0,255]
- Định nghĩa hàm Fast_Fourier() 
```python
 def Fast_Fourier(im_1):
    # Dịch tâm tần số về giữa (center shift)
    c = abs(scipy.fftpack.fft2(im_1))
    d = scipy.fftpack.fftshift(c)
    d = np.log(1 + d)  # log để nhìn rõ phổ tần số hơn
    d = (d - np.min(d)) / (np.max(d) - np.min(d)) * 255 # Chuẩn hóa ảnh về [0,255]
    return d.astype(np.uint8)
```
- Output của phép biến đổi này:
 ![image](https://github.com/user-attachments/assets/c5d9e657-f360-467c-b742-27abd95e60ca)

- Với phương pháp Butterworth Lowpass Filter:  sử dụng những điểm ảnh có tần suất thấp từ biến đổi Fourier có khả năng làm mịn ảnh và khử nhiễu. Trong bài này, áp dụng fft2 để phân tích ảnh, fftshift để dịch tâm tần số về giữa, sử dụng công thức H[i, j] = 1 / (1 + (r / d_0) ** (2 * t1)) để giữ những điểm có tần suất thấp. 
- Định nghĩa hàm  Butterworth_Lowpass_Filter()
```python 
def Butterworth_Lowpass_Filter(im_1):
    # thực hiện biến đổi Fourier nhanh (FFT)
    c = scipy.fftpack.fft2(im_1)
    # dịch phổ Fourier về trung tâm
    d = scipy.fftpack.fftshift(c)
    # khởi tạo các biến cho hàm tích chập
    M, N = d.shape
    H = np.ones((M, N))
    center1 = M / 2
    center2 = N / 2
    d_0 = 30.0  # bán kính cắt
    t1 = 1 # bậc của bộ lọc 
# định nghĩa hàm tích chập 
    for i in range(M):
        for j in range(N):
            # tính khoảng cách Euclid từ gốc tọa độ
            r = math.sqrt((i - center1) ** 2 + (j - center2) ** 2)
            # sử dụng bán kính cắt để loại bỏ tần số cao
            H[i, j] = 1 / (1 + (r / d_0) ** (2 * t1)) 
# thực hiện tích chập
    con = d * H
    # tính biên độ của ảnh sau biến đổi Fourier ngược
    e = abs(scipy.fftpack.ifft2(scipy.fftpack.ifftshift(con)))
    e = (e - np.min(e)) / (np.max(e) - np.min(e)) * 255
    return e.astype(np.uint8)
```
- Output của phép biến đổi này:
![image](https://github.com/user-attachments/assets/647fc6ef-3ec5-4b20-a51b-881102a3b208)

- Với phương pháp Butterworth Highpass Filter:  sử dụng những điểm ảnh có tần suất cao từ biến đổi Fourier để làm sắc biên của ảnh.
- Định nghĩa hàm Butterworth Highpass Filter()
```python
def Butterworth_Highpass_Filter(im_1):
    # thực hiện biến đổi Fourier nhanh (FFT)
    c = scipy.fftpack.fft2(im_1)
    # dịch phổ Fourier về trung tâm
    d = scipy.fftpack.fftshift(c)
    # khởi tạo các biến cho hàm tích chập
    M, N = d.shape
    H = np.ones((M, N))
    center1 = M / 2
    center2 = N / 2
    d_0 = 30.0 # bán kính cắt
    t1 = 1 # bậc của bộ lọc

    # định nghĩa hàm tích chập
    for i in range(M):
        for j in range(N):
            # tính khoảng cách Euclid từ gốc tọa độ
            r = math.sqrt((i - center1) ** 2 + (j - center2) ** 2)
            # sử dụng bán kính cắt để loại bỏ tần số thấp
            H[i, j] = 1 - 1 / (1 + (r / d_0) ** (2 * t1))

    # thực hiện tích chập
    con = d * H
    # tính biên độ của ảnh sau biến đổi Fourier ngược
    e = abs(scipy.fftpack.ifft2(scipy.fftpack.ifftshift(con)))
    e = (e - np.min(e)) / (np.max(e) - np.min(e)) * 255
    return e.astype(np.uint8)
```
- Output của phép biến đổi này:
![image](https://github.com/user-attachments/assets/6c466be8-0fff-4aca-8bf8-429e65c4199a)








 




