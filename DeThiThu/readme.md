# Nhập Môn Xử Lý Ảnh Số 
## **Bài thi thử**
**Sinh viên thực hiện:** Trần Như Khả Ý
**MSSV:** 2374802010582  
**Môn học:** Nhập môn Xử lý ảnh số  
**Giảng viên:** Đỗ Hữu Quân
---
## Công nghệ sử dụng
```python
import os
from PIL import Image
import cv2
import colorsys
from PIL import Image
import numpy as np
import imageio.v2 as iio
import scipy.ndimage as sn
from skimage import feature
from skimage.morphology import label
from skimage.measure import regionprops
import matplotlib.pylab as plt
import matplotlib.patches as mpatches
from skimage.filters.thresholding import threshold_otsu
```
- **Python**: Ngôn ngữ chính  
- **Pillow (PIL)**: Đọc, chuyển đổi, và lưu ảnh  
- **NumPy**: Xử lý ảnh dưới dạng mảng số học  
- **ImageIO**: Đọc file ảnh với định dạng hiện đại  
- **Matplotlib**: Hiển thị ảnh trực quan  
- **Skimage**: Tách đối tượng và nền trong ảnh - ngưỡng hóa ảnh  
  - `threshold_otsu`: Tự động tìm ngưỡng phân tách ảnh bằng phương pháp Otsu  
  - `label`: Gán nhãn các vùng liên thông trong ảnh nhị phân  
  - `regionprops`: Tính toán các đặc trưng hình học của từng vùng (diện tích, tâm, hình dạng...)  
  - `feature`: Trích xuất đặc trưng như cạnh, góc, kết cấu (texture)  
- **scipy.ndimage**: Xử lý ảnh nâng cao - làm mịn, xoay, dịch chuyển, lọc ảnh
## Chi tiết các phép biến đổi & công thức
### 1. Mean filter: (bộ lọc trung bình)
- Mục đích: dùng để làm mịn ảnh và giảm nhiễu bằng cách thay mỗi pixel bằng giá trị trung bình của các pixel lân cận.
- Công thức: giá trị pixel:
  ```math
  F_i = \frac{1}{N}
  ```
 - Với N là số phần tử trong bộ lọc.
- Code chính:
```python
# đọc ảnh và chuyển sang ảnh xám
a = iio.imread('a.jpg', mode ='L')
# mean filter (bộ lọc trung bình)
#Khởi tạo bộ lọc trung bình 5x5  mỗi phần tử có giá trị 1/25.
k= np.ones((5,5))/25
# thực hiện phép tính và lưu ảnh mới
b = sn.convolve (a, k).astype(np.uint8)
print (b)
```
### 2. Canny filter :
- Mục đích:  xác định biên của đối tượng trên first derivative (các giá trị có sẵn của ảnh tới cạnh biên).
- Code chính:
```python
# opening the image and converting it to grayscale
a = iio.imread('a.jpg', mode='L')
b = feature.canny(a, sigma=3).astype(np.uint8)
```
- **Lưu ý**: 
  - sigma nhỏ : phát hiện biên chi tiết hơn (nhưng dễ nhiễu).
  - sigma lớn: làm mịn hơn nhưng có thể bỏ sót biên nhỏ.
### 3. Đổi màu ảnh từ không gian màu BGR sang một màu ngẫu nhiên (RGB)
- Mục đích: Tạo hiệu ứng màu, kiểm tra độ nhạy mô hình theo màu sắc, hoặc xử lý ảnh sáng tạo.
- Code chính:
```python
# Đổi thứ tự kênh BGR → RGB
data_rgb = data[:, :, ::-1]
# Tạo thứ tự kênh ngẫu nhiên bằng cách tráo 3 kênh màu
channels = [0, 1, 2]
random.shuffle(channels)
# Đổi màu ảnh bằng cách thay đổi thứ tự kênh RGB
bdata = data_rgb[:, :, channels]
```
### 4. Chuyển ảnh sang không gian màu HSV và tách riêng kênh Hue, Saturation, Value
- Mục đích : Hệ HSV biểu diễn dữ liệu trong ba kênh là Hue, Saturation, và Value.
  - Kênh Value biểu diễn thông tin intensity.
  - Kênh Hue biểu diễn tỉ lệ giữa hai số lớn nhất từ 3 giá trị RGB cho mỗi pixel.
  - Kênh Saturation xác định độ đậm đặc màu qua tỉ lệ khác nhau giữa màu thấp nhất với màu lớn nhất trong giá trị RGB.
- Code chính:
  - Chuyển RGB sang HSV
```python
rgb2hsv = np.vectorize(colorsys.rgb_to_hsv)
h, s, v = rgb2hsv(img[:, :, 0]/255, img[:, :, 1]/255, img[:, :, 2]/255)
```
  - Chỉ giữ kênh Hue:
```python
h_img = np.vectorize(colorsys.hsv_to_rgb)(h, np.ones_like(s), np.ones_like(v))
h_img = (np.array(h_img).transpose((1, 2, 0)) * 255).astype(np.uint8)
```
  - Chỉ giữu kênh Value:
```python
v_img = np.vectorize(colorsys.hsv_to_rgb)(np.zeros_like(h), np.ones_like(s), v)
v_img = (np.array(v_img).transpose((1, 2, 0)) * 255).astype(np.uint8)
```
  - Chỉ giữ kênh Saturation:
```python
s_img = np.vectorize(colorsys.hsv_to_rgb)(np.zeros_like(h), s, np.ones_like(v))
s_img = (np.array(s_img).transpose((1, 2, 0)) * 255).astype(np.uint8)
```
### 5. Image inverse transformation
- Mục đích: làm đảo ngược mức sáng của ảnh từ vùng sáng sang vùng tối và ngược lại từ vùng tối sang vùng sáng. 
 - Có công thức là
   ```math
   s = (L-1) - r .
   ```
    Trong đó: 
   - `s`: giá trị pixel mới
   - `L-1`: giá trị cường độ tối đa
   - `r`: giá trị pixel hiện tại.
- Code chính:
```python
def Image_inverse_transformation(im_1):
    return 255 - im_1
```
### 6. Gamma-Correction: 
- Mục đích: Dùng để tăng chất lượng của ảnh, làm sáng các vùng tối (với γ < 1) hoặc làm tối các vùng sáng (với γ < 1).
- Công thức:
```math
 s = c * r^γ.
```
- Trong đó:
   - `s`: giá trị pixel mới
   - `γ`: giá trị gamma
   - `r`: giá trị pixel hiện tại
   - `c`: một hằng số .
- Code chính: 
```python
def Gamma_Correction(im_1, gamma=None):
    normalized = im_1 / 255.0
    if gamma is None:
        gamma = np.random.uniform(0.5, 2.0)
    corrected = np.power(normalized, gamma)
    return np.clip(corrected * 255, 0, 255).astype(np.uint8)
```
### 7. Thay đổi cường độ điểm ảnh với Log Transformation: 
- Mục đích: dùng để làm nổi bật các chi tiết trong vùng tối và giảm chói vùng sáng. 
- Công thức:
```math
 s = c * log(1 + r).
``` 
- Trong đó:
  - `s`: giá trị pixel mới
  - `r`: giá trị pixel hiện tại
  - `c`: một hằng số 
```python
def Log_Transformation(im_1):
    b1 = im_1.astype(float)
    b2 = np.max(b1)
    c = (128.0 * np.log(1 + b1)) / np.log(1 + b2)
    return np.clip(c, 0, 255).astype(np.uint8)
```
### 8.  Histogram equalization: 
- Mục đích: dùng để cân bằng độ sáng tối cho ảnh nổi bật và dễ nhìn hơn. Hàm sử dụng xác suất, CDF(Cumulative Distribution) để tính. 
- Trong bài này, thực hiện cân bằng lược đồ bằng cách đếm số pixel cho từng mức xám để tạo histogram (cdf = hist.cumsum()) sau đó chuẩn hóa cdf và dùng nó để tạo ảnh mới.
```python
def Histogram_equalization(im_1):
    flat = im_1.flatten()
    hist, _ = np.histogram(flat, 256, [0, 255])
    cdf = hist.cumsum()
    cdf_m = np.ma.masked_equal(cdf, 0)
    cdf_m = (cdf_m - cdf_m.min()) * 255 / (cdf_m.max() - cdf_m.min())
    cdf = np.ma.filled(cdf_m, 0).astype('uint8')
    equalized = cdf[flat]
    return np.reshape(equalized, im_1.shape)
```
### 9. Thay đổi ảnh với Contrast Stretching: 
- Mục đích: dùng để tăng độ tương phản tương tự Histogram Equalization nhưng bằng cách kéo dãn dãy giá trị pixel (thay đổi giá tri pixel) thay vì dùng xác suất, CDF để tính.
- Công thức
```math
s = (r - r_min) * (255 / (r_max - r_min))
```
- Trong đó: 
  -`s`: giá trị pixel mới sau khi giãn.
  - `r`: giá trị pixel gốc (trong ảnh đầu vào).
  - `r_min`: giá trị pixel nhỏ nhất trong ảnh gốc.
  - `r_max`: giá trị pixel lớn nhất trong ảnh gốc.
```python
def Contrast_Stretching(im_1):
    a = np.min(im_1)
    b = np.max(im_1)
    stretched = 255 * (im_1 - a) / (b - a)
    return np.clip(stretched, 0, 255).astype(np.uint8)
```
### 10. Adaptive Histogram Equalization (sử dụng CLAHE với ô lưới 8x8):
- Mục đích : giúp làm rõ chi tiết từng vùng trong ảnh, thích hợp cho các ảnh phức tạp hoặc độ tương phản không đồng đều.
- Công thức:
<img width="491" height="171" alt="image" src="https://github.com/user-attachments/assets/eda318bb-af34-4873-9230-2f1a19da8a40" />

- Trong đó:
  - `sk` : giá trị pixel mới ứng với mức xám k
  - `nj` : số lượng pixel có mức xám j trong vùng
  - `M x N`: kích thước của vùng (ô)
  - `L`: số mức xám.
- Code chính:
```python
def Adaptive_Histogram_Equalization(im_1):
    clahe = cv2.createCLAHE(clipLimit=2.0, tileGridSize=(8, 8))
    return clahe.apply(im_1)
```
### 11. Tăng kích thước ảnh cả chiều dài và rộng:
- Mục đích : Thêm viền hoặc không gian nền xung quanh.
- Code chính:
```python
d1 = cv2.copyMakeBorder(data1, 30, 30, 30, 30, cv2.BORDER_CONSTANT, value=(255, 255, 255))
```
### 12. Xoay ảnh theo chiều kim đồng hồ và và lật ngang: 
- Mục đích: xoay ảnh theo góc xoay. Ví dụ ảnh bị nghiêng thì xoay về cho đúng hướng chuẩn.
- Dùng hàm rotate(image, degree) để xoay một ảnh với Image: là ảnh trong bộ nhớ, Degree: là góc xoay
- Dùng hàm fliplr() để xoay ngang.
- Ví dụ xoay 1 ảnh với 1 góc 45 độ lúc này degree = 45 -> rotate(image,45)
- Code chính:
```python
# xoay  ảnh quang ninh 
xoay = nd.rotate(data2, -45, reshape=True)
d2 = np.fliplr(xoay)
```
### 13. Tăng kích thước ảnh và áp dụng Gaussian blur với kernel 7x7 để làm mịn.
- Mục đích: Phóng to hoặc thu nhỏ ảnh theo tỷ lệ và làm mịn ảnh sau khi phóng to
- Công thức toán học:
```math
I'(x,y)= I(x/sx , y /sy)
```
- `sx, sy` : lần lượt là hệ số zoom theo chiều ngang và dọc
và 

<img width="413" height="122" alt="image" src="https://github.com/user-attachments/assets/04c51d4b-1059-4e85-8b24-f0132a51fa8b" />

- Áp dụng công thức lọc Gaussian lên từng điểm ảnh, trong đó mỗi điểm ảnh được làm mờ bằng trung bình trọng số của các điểm lân cận.
-  Phạm vi ảnh hưởng: x,y ∈ [−3,3]

- Ví dụ muốn phóng to đối tượng x2 lần: sx = sy = 2, giá trị ảnh mới tại (x,y) là nội suy từ ảnh gốc tại (x/2,y/2)
- Code chính:
```python
# Tăng kích thước ảnh pagoda len 5 làn
d3 = nd.zoom (data3, (5, 5, 1))
# Áp dụng Gaussian blur với kernel 7x7
d3_blur = cv2.GaussianBlur(d3, (7, 7), 0)
```
### 14. Áp dụng công thức điều chỉnh độ sáng và tương phản của ảnh:
- Mục đích : điều chỉnh độ sáng và độ tương phản .
- Công thức:
<img width="1027" height="619" alt="image" src="https://github.com/user-attachments/assets/a1223d9e-bcb8-4591-ab05-e7a99fdcc4a4" />
- Code chính:
```python
#  Áp dụng ảnh pagoda theo công thứuc
d4_img = cv2.cvtColor(d3, cv2.COLOR_BGR2RGB)
# Tham số điều chỉnh
alpha = np.random.uniform(0.5 , 2.0)   # Tăng tương phản (0.5 - 2.0)
beta = np.random.randint(-50 , 50)   # Tăng độ sáng (-50 đến +50)
# Áp dụng công thức điều chỉnh
d4 = np.clip(alpha *d4_img + beta, 0, 255).astype(np.uint8)
```



