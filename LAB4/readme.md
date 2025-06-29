# Nhập Môn Xử Lý Ảnh Số - Lab 4 
## **PHÂN VÙNG ẢNH**
**Sinh viên thực hiện:** Trần Như Khả Ý
**MSSV:** 2374802010582  
**Môn học:** Nhập môn Xử lý ảnh số  
**Giảng viên:** Đỗ Hữu Quân
---
## Công nghệ sử dụng
```python
from PIL import Image
import numpy as np
import imageio.v2 as iio
import scipy.ndimage as nd
import matplotlib.pyplot as plt
from skimage.filters import threshold_local, threshold_otsu
```
- **Python**: Ngôn ngữ chính                          
- **Pillow (PIL)**: Đọc, chuyển đổi, và lưu ảnh              
- **NumPy**: Xử lý ảnh dưới dạng mảng số học          
- **ImageIO**: Đọc file ảnh với định dạng hiện đại      
- **Matplotlib**: Hiển thị ảnh trực quan
- **Skimage** : Tách đối tượng và nền trong ảnh - ngưỡng hóa ảnh
---
## Cài đặt thư viện
```python
1. CÀI ĐẶT THƯ VIỆN
pip install opencv-python
```
- Thư viện này là thư viện mạnh trong lĩnh vực xử lý ảnh và thị giác máy tính dùng để xử lý ảnh và video, hỗ trợ các thao tác như đọc, ghi, biến đổi ảnh,...
  
## Chi tiết các phép biến đổi & công thức
### 1. Phân vùng theo histogram: 
- Một ngưỡng được xác định dựa theo histogram của ảnh. Mỗi pixel trong ảnh được so sánh với ngưỡng, nếu giá trị pixel nhỏ hơn ngưỡng, thì pixel trong phân vùng được gán giá trị 0. Ngược lại, gán giá trị 1.
#### 1.1 Phương pháp Otsu
- Mục đích : xác định ngưỡng phân vùng dựa trên phân phối histogram, giúp chia hình ảnh thành 2 lớp dựa trên các giá trị cường độ thang độ xám của các điểm ảnh của nó.
- Công thức: tính toán ngưỡng bằng cách tối đa hóa phương sai giữa các lớp của giá trị pixel
```math
  σ² = ω₁ * ω₂ * (μ₁ - μ₂)²
```
Trong đó: 
- `σ^2b​(t)`: phương sai giữa hai lớp tại ngưỡng 
- `ω1(t)`: xác suất của lớp 1 (dưới ngưỡng t – thường là nền)
- `𝜔2(𝑡)`: xác suất của lớp 2 (trên ngưỡng t – thường là đối tượng)
- `μ1(t),μ2(t)`: trung bình mức xám của hai lớp
- Code chính:
```python
data = Image.open('fruit.jpg').convert('L')
a = np.asarray(data)
thres = threshold_otsu (a) # Thực hiện phương pháp ngưỡng hóa Otsu (Otsu's thresholding)
# Giữ lại các pixel có cường độ lớn hơn ngưỡng
b = a > thres
b = a > thres
```
#### 1.2 Phương pháp Adaptive Thresholding
- Cải tiến phân vùng chính xác hơn Otsu. Chia ảnh thành nhiều ảnh nhỏ và tính threshold cho từng ảnh nhỏ.
- Mục đích: Phân vùng ảnh trong điều kiện ánh sáng không đồng đều hoặc nhiễu, nơi mà Otsu không hiệu quả.
- Code chính:
```python
data = Image.open('fruit.jpg').convert('L')
a = np.asarray (data)
# Thực hiện phương pháp ngưỡng hóa Otsu (Otsu's thresholding)
b = threshold_local (a, 39, offset=10)
```
### 2.  Phân vùng theo region
- Một region là một nhóm các pixel có cùng thuộc tính.
- Mục đích : tách các đối tượng dính liền nhau trong ảnh bằng các phép biến đổi như ngưỡng hóa ảnh (thresholding), phép co (erosion), phép biến đổi khoảng cách (distance transform) và thuật toán watershed.
- Code chính:
```python
# Ngưỡng hóa ảnh (thresholding) để lấy các pixel thuộc về đối tượng (cell)
thresh, bl = cv2.threshold(a, 0, 255, cv2.THRESH_BINARY_INV + cv2.THRESH_OTSU)
b2 = cv2.erode (bl, None, iterations = 2)  # thực hiện phép co (erosion) để giảm nhiễu
# Áp dụng biến đổi khoảng cách (distance transform)
dist_trans = cv2.distanceTransform (b2, 2, 3)
# Ngưỡng hóa ảnh distance transform để lấy các pixel nền trước (foreground)
thresh, dt = cv2.threshold (dist_trans, 1, 255, cv2.THRESH_BINARY)
labelled, ncc = label (dt) # gán nhãn 
labelled = labelled.astype (np.int32)
cv2.watershed(data, labelled) # Thực hiện phân đoạn bằng thuật toán watershed
```
### 3. Biến đổi đối tượng trong ảnh
#### 3.1 Sử dụng binary_dilation 
- Dilation cho phép các pixel ở foreground của 1 ảnh có thể со giãn. 
- Mục đích : Làm đầy các lỗ nhỏ trong đối tượng, mở rộng các vùng foreground (màu trắng) trong ảnh nhị phân.
- Code chính:
```python
data = Image.open('dil_img.gif').convert('L')
b = nd.binary_dilation(data, iterations=50)
```
#### 3.2 Sử dụng binary_opening
- Mục đích: loại bỏ nhiễu nhỏ (đốm sáng nhỏ) và làm biên các vùng sáng của đối tượng mượt hơn.
- Code chính:
```python
data = Image.open('dil_img.gif').convert('L')
# Định nghĩa phần tử cấu trúc (structuring element)
s = [[0, 1, 0], [1, 1, 1], [0, 1, 0]]
b = nd.binary_opening (data, structure=s, iterations=25)
```
#### 3.3 Sử dụng binary_erosion
- Mục đích: Dùng để co đối tượng bằng cách loại bỏ pixels ở biên của đối tượng.
- Code chính:
- 


## Tài liệu tham khảo:
- https://www.baeldung.com/cs/otsu-segmentation
- 
