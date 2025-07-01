# Nhập Môn Xử Lý Ảnh Số - Lab 4 
## **PHÂN VÙNG ẢNH**
**Sinh viên thực hiện:** Trần Như Khả Ý
**MSSV:** 2374802010582  
**Môn học:** Nhập môn Xử lý ảnh số  
**Giảng viên:** Đỗ Hữu Quân
---
## Công nghệ sử dụng
```python
import cv2
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
- Thư viện opencv (viết tắt là cv2) là thư viện mạnh trong lĩnh vực xử lý ảnh và thị giác máy tính dùng để xử lý ảnh và video, hỗ trợ các thao tác như đọc, ghi, biến đổi ảnh,...
  
## Chi tiết các phép biến đổi & công thức
### 1. Phân vùng theo histogram: 
- Một ngưỡng được xác định dựa theo histogram của ảnh. Mỗi pixel trong ảnh được so sánh với ngưỡng, nếu giá trị pixel nhỏ hơn ngưỡng, thì pixel trong phân vùng được gán giá trị 0. Ngược lại, gán giá trị 1.
#### 1.1 Phương pháp Otsu
- Mục đích : xác định ngưỡng phân vùng dựa trên phân phối histogram, giúp chia hình ảnh thành 2 lớp dựa trên các giá trị cường độ thang độ xám của các điểm ảnh của nó.
- Công thức: tính toán ngưỡng bằng cách tối đa hóa phương sai giữa các lớp của giá trị pixel 
  ![image](https://github.com/user-attachments/assets/1f9c73c8-f8f3-4d07-9c41-0ac5ca7f1758)

- Trong đó: 
- `σ²B(k)`:Phương sai giữa lớp ứng với ngưỡng 𝑘 
- `mG`: Trung bình mức xám toàn ảnh 
- `P1(k)`: Xác suất (tích lũy) của lớp 1 (foreground) đến mức xám 𝑘
- `m(k)`: Trung bình mức xám của lớp foreground đến mức 𝑘
- Code chính:
```python
data = Image.open('fruit.jpg').convert('L')
a = np.asarray(data)
thres = threshold_otsu (a) # Thực hiện phương pháp ngưỡng hóa Otsu (Otsu's thresholding)
# Giữ lại các pixel có cường độ lớn hơn ngưỡng
b = a > thres
b = a > thres
```
---
#### 1.2 Phương pháp Adaptive Thresholding
- Cải tiến phân vùng chính xác hơn Otsu. Chia ảnh thành nhiều ảnh nhỏ và tính threshold cho từng ảnh nhỏ.
- Mục đích: Phân vùng ảnh trong điều kiện ánh sáng không đồng đều hoặc nhiễu, nơi mà Otsu không hiệu quả.
- Công thức toán học :
```math
T(x,y)=mean(Nx,y)−C
```
- Trong đó:
- `T(x,y)`: ngưỡng tại điểm (x, y)
- `Nx,y`: vùng lân cận của điểm (x, y) (ví dụ: 11x11 pixel)
- `mean(...)`: trung bình mức xám các điểm trong vùng
- `C`: hằng số điều chỉnh
- Code chính:
```python
data = Image.open('fruit.jpg').convert('L')
a = np.asarray (data)
# Thực hiện phương pháp ngưỡng hóa Otsu (Otsu's thresholding)
b = threshold_local (a, 39, offset=10)
```
---
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
---
### 3. Biến đổi đối tượng trong ảnh
#### 3.1 Sử dụng binary_dilation 
- Dilation cho phép các pixel ở foreground của 1 ảnh có thể со giãn. 
- Mục đích : Làm đầy các lỗ nhỏ trong đối tượng, mở rộng các vùng foreground (màu trắng) trong ảnh nhị phân.
- Công thức toán học:
![image](https://github.com/user-attachments/assets/7964af7e-eee1-469c-be51-6915d1c29f5d)

- Với 𝐴 và B là các tập hợp trong không gian 𝑍^2 (tập các điểm nguyên trong mặt phẳng)
- Code chính:
```python
data = Image.open('dil_img.gif').convert('L')
b = nd.binary_dilation(data, iterations=50)
```
---
#### 3.2 Sử dụng binary_opening
- Mục đích: loại bỏ nhiễu nhỏ (đốm sáng nhỏ) và làm biên các vùng sáng của đối tượng mượt hơn.
- Công thức toán học:
![image](https://github.com/user-attachments/assets/27998609-06c7-4529-936b-974488e04d99)

- Code chính:
```python
data = Image.open('dil_img.gif').convert('L')
# Định nghĩa phần tử cấu trúc (structuring element)
s = [[0, 1, 0], [1, 1, 1], [0, 1, 0]]
b = nd.binary_opening (data, structure=s, iterations=25)
```
---
#### 3.3 Sử dụng binary_erosion
- Mục đích: Dùng để co đối tượng bằng cách loại bỏ pixels ở biên của đối tượng.
- Công thức toán học:
![image](https://github.com/user-attachments/assets/66217b84-cb23-43b7-b292-884a9ca87863)

- Với 𝐴 và B là các tập hợp trong không gian 𝑍^2 (tập các điểm nguyên trong mặt phẳng)
- Code chính:
```python
data = Image.open('dil_img.gif').convert('L')
# Định nghĩa phần tử cấu trúc (structuring element)
S = [[0, 1, 0], [1, 1, 1], [0, 1, 0]] 
b = nd.binary_erosion (data, structure=s, iterations=50)
```
---
##### 3.4 Sử dụng binary_closing
- Mục đích: làm trơn các đoạn viền, lấp các lỗ nhỏ, và làm đầy các khoảng trống trong đường viền.
- Công thức toán học:
![image](https://github.com/user-attachments/assets/113c9642-05b6-41d2-8b7a-0d3219b21af7)

- Với 𝐴 và B là các tập hợp trong không gian 𝑍^2 (tập các điểm nguyên trong mặt phẳng)
- Code chính:
```python
data = Image.open('dil_img.gif').convert('L')
# Định nghĩa phần tử cấu trúc
s = [[0, 1, 0], [1, 1, 1], [0, 1, 0]]
b = nd.binary_closing (data, structure=s, iterations=50)
```
## BÀI TẬP
### 1. Viết chương trình chọn LangBiang trong ảnh Đà Lạt từ thư mục exercise. Tịnh tiến vùng chọn sang phải 100px. Sử dụng phương pháp Otsu để phân vùng LangBiang theo ngưỡng 0.3. Lưu vào máy với tên lang_biang.jpg và hiển thị trên màn hình.
- Trong bài này sử dụng phép tịnh tiến và phân vùng theo histogram với phương pháp otsu.
  - Phép tịnh tiến: dùng để dịch chuyển đối tượng sang phải 100px
  - Phương pháp otsu: dùng ngưỡng 0.3 để phân vùng đối tượng, giúp chia hình ảnh thành 2 lớp dựa trên các giá trị cường độ thang độ xám của các điểm ảnh của nó.
- Code chính:
```python
bmg = data1 [0:350,0:500] # chọn đối tượng LangBiang trong ảnh ĐL và lưu ảnh đã cắt
data2 = Image.open('langBiang.jpg').convert('L') # Đọc ảnh đã cắt và chuyển sang ảnh xám (grayscale)
a = np.asarray(data2).astype(np.float32)
# Tịnh tiến ảnh sang phải 100px
bdata = nd.shift(a, shift=(0, 100), order=0, mode='constant', cval=0)
a_norm = bdata/255 # Thực hiện phương pháp ngưỡng hóa Otsu 0.3 
b = a_norm > 0.3
```
---
### 2. Viết chương trình chọn Hồ Xuân Hương trong ảnh Đà Lạt từ thư mục exercise. Xoay đối tượng vừa chọn 1 góc 45° và dùng phương pháp Adaptive Thresholding với ngưỡng 60 và lưu vào máy với tên là ho_xuan_huong.jpg.
- Trong bài sử dụng phép xoay(rotate),  phân vùng theo histogram với phương pháp adaptive thresholding.
  - Phép xoay (rotate): dùng để xoay toàn bộ đối tượng theo góc xoay (trong bài góc xoay là 45 độ).
  - Phương pháp Adaptive Thresholding: chia ảnh thành nhiều ảnh nhỏ và tính threshold cho từng ảnh nhỏ, với ngưỡng là 60
- Code chính:
```python
bmg = data1 [0:690, 500: 1000] # chọn đối tượng hồ xuân hương trong ảnh ĐL và lưu ảnh đã cắt
# Đọc ảnh đã cắt và chuyển sang ảnh xám (grayscale)
data2 = Image.open('hoXuanHuong.jpg').convert('L')
a = np.asarray(data2).astype(np.float32)
d1 = nd.rotate (a, 45, reshape=False) #  Xoay ảnh
# Thực hiện phương pháp ngưỡng hóa Otsu 60 (Otsu's thresholding) dùng block_size=61 (phải là số lẻ)
b = threshold_local (d1, block_size=61, offset=10)
binary_result = d1 > b
```
---
### 3. Viết chương trình chọn Quảng trường Lâm Viên trong ảnh Đà Lạt từ thư mục exercise. Dùng phương pháp Coordinate Mapping và Binary Closing cho vùng vừa chọn. Lưu vào máy với tên là quangquang_truong_lam_vien.jpg.
- Trong bài sử dụng phương pháp biến đổi theo tọa độ - Coordinate Mapping và biến đổi đối tượng trong ảnh với phương pháp Binary closing.
  - Phương pháp biến đổi theo tọa độ - coordinate mapping: dùng để biến dạng ảnh bằng cách thay đổi tọa độ của các điểm ảnh.
  - Phương pháp binary closing: dùng để làm trơn các đoạn viền, làm đầy các khoảng trống trong đường viền.
- Code chính:
```python
# Coordinate Mapping (biến dạng nhẹ)
V, H = a.shape
M = np.indices((V, H)).astype(np.float32)
d = 5
q = 2 * d * np.random.ranf(M.shape).astype(np.float32) - d
mp = M + q
d1 = nd.map_coordinates(a, mp, order=1, mode='reflect')
# Ngưỡng hóa Otsu
threshold = threshold_otsu(d1)
d1_bin = d1 > threshold
# Binary Closing
s = np.array([[0, 1, 0],
              [1, 1, 1],
              [0, 1, 0]], dtype=np.uint8)
b = nd.binary_closing(d1_bin, structure=s, iterations=50)
```
### 4. Viết chương trình cho phép người dùng nhập chức năng muốn xử lý. (Có thể chọn 1 chức năng duy nhất hoặc kết hợp 2 chức năng của geometric_transformation và segment).
- Trong bài này sử dụng 2 phương pháp biến đổi chính là biến đổi hình học và phân vùng ảnh.
- Biến đổi hình học gồm: tịnh tiến, thay đổi kích thước ảnh, xoay và biến dạng ảnh. Dùng để dịch chuyển, xoay, co giãn, lật, biến dạng ảnh.
- Phân vùng ảnh gồm : otsu, adaptive thresholding, Binary_erosion và Binary_dilation. Dùng để làm nổi bật hoặc cô lập các đối tượng quan tâm để phân tích, nhận dạng.
- Code chính:
  - Phần viết cây thư mục và hiển thị:
    ```python
    tree_data = {
    "geometric_transformation": {
        "coordinate_mapping": {},
        "Rotate": {},
        "Scale": {},
        "Shift": {}
    },
    "segment": {
        "Adaptive_thresholding": {},
        "Binary_dilation": {},
        "Binary_erosion": {},
        "Otsu": {}
    } }
    def print_tree(data, prefix=""):
    keys = list(data.keys())
    for i, key in enumerate(keys):
        is_last = (i == len(keys) - 1)
        connector = "└── " if is_last else "├── "
        print(prefix + connector + key)
        new_prefix = prefix + ("    " if is_last else "│   ")
        if isinstance(data[key], dict):
            print_tree(data[key], new_prefix)
    ```
  - Các hàm tương ứng với chức năng:
  ```python
  func_map = {
    "coordinate_mapping": coordinate_mapping,
    "Rotate": Rotate,
    "Scale": phongTo,
    "Shift": Shift,
    "Adaptive_thresholding": Adaptive_thresholding,
    "Binary_dilation": Binary_dilation,
    "Binary_erosion": Binary_erosion,
    "Otsu": Otsu
  }
  ```
  - Main code:
  ```python
  # ==== Chương trình chính ==== print("==== Danh sách chức năng xử lý ảnh ====")
  print_tree(tree_data)
  # cho phép ng dùng nhập chức năng
  choices = input("\nNhập 1 hoặc 2 tên chức năng cách nhau bằng dấu phẩy (ví dụ: Rotate, Otsu): ").strip().split(',')
  img_np = iio.imread('exercise/dalat.jpg')
  
  # Áp dụng lần lượt các hàm
  img_result = img_np
  for ch in choices:
      ch = ch.strip().capitalize()
      if ch in func_map:
          img_result = func_map[ch](img_result)
      else:
          print(f"[!] Không tìm thấy chức năng: {ch}")
    ```
---
## Tài liệu tham khảo:
- ***Rafael C. Gonzalez, Richard E. Wods - Digital Image Processing-Pearson (2017)***
- Slide bài giảng Nhập môn Xử lý ảnh số - Văn Lang University
  
