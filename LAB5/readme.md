
# Nhập Môn Xử Lý Ảnh Số - Lab 5
## **XÁC ĐỊNH ĐỐI TƯỢNG TRONG ẢNH**
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
- **Skimage** : Tách đối tượng và nền trong ảnh - ngưỡng hóa ảnh
---
## Cài đặt thư viện
```python
1. CÀI ĐẶT THƯ VIỆN
pip install opencv-python
```
- Thư viện opencv (viết tắt là cv2) là thư viện mạnh trong lĩnh vực xử lý ảnh và thị giác máy tính dùng để xử lý ảnh và video, hỗ trợ các thao tác như đọc, ghi, biến đổi ảnh,...
  
## Chi tiết các phép biến đổi & công thức
### 1.Gán nhãn ảnh
- Mục đích: Tìm tất cả các vùng liên thông (connected components) từ ảnh nhị phân bằng cách bắt đầu từ một điểm đã biết trong mỗi vùng, rồi mở rộng dần vùng đó bằng phép giãn có điều kiện cho đến khi không mở rộng được nữa (đạt hội tụ).
- Công thức toán học:
![image](https://github.com/user-attachments/assets/48f3b43a-36dd-4fa8-a4b7-583521490543)
- Trong đó :
- `B`: phần tử cấu trúc
- Quy trình kết thúc khi $$X_k = X_{k-1}$$ chứa toàn bộ các thành phần liên thông của điểm ảnh tiền cảnh (foreground pixels) trong ảnh.
- Code chính:
```python
data = Image.open('geometric.png').convert('L')
a = np.asarray(data)
# Áp dụng phương pháp phân ngưỡng Otsu
thres = threshold_otsu(a)
# Giữ lại các điểm ảnh có cường độ lớn hơn ngưỡng
b = a > thres
# Gán nhãn cho các vùng trong ảnh nhị phân b
c = label(b)
```
### 2. Dò tìm cạnh theo chiều dọc
- Là một phương pháp dùng để phân đoạn ảnh dựa trên sự thay đổi cường độ đột ngột và cục bộ.
- Mục đích :Phát hiện vị trí các biên trong ảnh.
- Công thức: tính toán dò tìm cạnh theo chiều dọc bằng cách lấy đạo hàm theo trục hoành (x)   
```math
g_x(x, y) = \frac{\partial f(x, y)}{\partial x}
```
- Code chính:
```python
data = Image.open('geometric.png').convert('L')
bmg = abs(data - nd.shift(data, (0,1), order=0))
```
---
### 3. Dò tìm cạnh với Sobel Filter
- Là một bộ lọc đạo hàm dùng để dò tìm cạnh trong ảnh số, bằng cách tính xấp xỉ đạo hàm bậc nhất theo các hướng ngang (x) và dọc (y).
- Mục đích: phát hiện cạnh của các đối tượng trong ảnh, nơi cường độ thay đổi mạnh.
- Công thức toán học :
![image](https://github.com/user-attachments/assets/b96be4c3-11fa-4365-aec4-9eb8983e1e99)

- Trong đó:
- $$g_x(x, y) = \frac{\partial f(x, y)}{\partial x}$$ : Đạo hàm bậc 1 theo trục x
- $$g_y(x, y) = \frac{\partial f(x, y)}{\partial y}$$ : Đạo hàm bậc 1 theo trục y 
- Code chính:
```python
data = Image.open('geometric.png')
a = nd.sobel(data, axis =0)
b = nd.sobel(data, axis = 1)
bmg = abs(a) + abs(b)
```
---
### 4. Xác định góc của đối tượng
- Mục đích : phát hiện các điểm góc trong ảnh – những điểm mà tại đó gradient thay đổi mạnh ở nhiều hướng.
- Công thức toán học:
![image](https://github.com/user-attachments/assets/976a0efa-42a5-4f1d-a39b-df420e85ae12)
- Trong đó:
- ` M`: 	Ma trận cấu trúc, tính từ đạo hàm ảnh
- `k`: Hằng số điều chỉnh
- `det(M)`: định thức của ma trận cấu trúc
- `trace(M)` : đo tổng độ biến thiên theo cả hai hướng x và y (tổng các phần tử trên đường chéo chính của ma trận M.)
- Code chính:
```python
def Harris(indata , alpha=0.2):
    # Tính đạo hàm bậc 1 theo x và y bằng Sobel filter
    x = nd.sobel(indata, 0)
    y = nd.sobel(indata, 1)
    # Tính các thành phần của ma trận cấu trúc (structure tensor)
    x1 = x**2
    y1 = y**2
    xy= abs(x*y)
    # Làm mượt các ma trận thành phần bằng bộ lọc Gaussian
    x1 = nd.gaussian_filter(x1, 3)
    y1 = nd.gaussian_filter(y1, 3)
    xy= nd.gaussian_filter(xy,3)
 # Tính định thức và trace của ma trận cấu trúc
    detC = x1 * y1 - 2*xy
    trC = x1 + y1
    R= detC - alpha * trC**2  # công thức điểm góc Harris
    return R
data = Image.open('geometric.png')
bmg = Harris(data)
```
---
### 5.Dò tìm hình dạng cụ thể trong ảnh với Hough Transform
- Là một kỹ thuật dùng để phát hiện các đường cong hình học có dạng xác định (có thể là đường thẳng, đường tròn, ellipse,...) trong ảnh
- Mục đích : Tìm tất cả các điểm nằm trên cùng một đường thẳng (hoặc đường tròn...), kể cả ảnh chứa nhiều nhiễu hoặc không biết rõ vị trí vật thể.
#### 5.1 Dò tìm đường thẳng trong ảnh
- Mục đích : Phát hiện các đường thẳng trong ảnh từ tập hợp các điểm biên trong ảnh nhị phân.
- Công thức toán học:
![image](https://github.com/user-attachments/assets/6525ef6d-f752-4b33-b7c6-2e10b19444fb)
- Trong đó:
- `x,y`: Tọa độ của điểm biên trong ảnh
- `θ` : góc
- `ρ`: Khoảng cách từ gốc tọa độ đến đường thẳng đó.
- Code chính: hàm def LineHough()
```python
    # Tính toán khoảng cách (rho) tối đa, là đường chéo của ảnh
    R = int(np.sqrt(V * V + H * H))
    # Khởi tạo không gian Hough (ma trận tích lũy) với kích thước (R x 90)
    ho = np.zeros((R, 90), float) # Hough space
---------------------------------
# Tính toán các giá trị rho cho điểm (x, y) với tất cả các góc theta
            rh = x * np.cos(theta) + y * np.sin(theta)
            # Duyệt qua từng cặp (rho, theta) đã tính
            for i in range(len(rh)):
                # Kiểm tra xem cặp (rho, theta) có nằm trong giới hạn của không gian Hough không
                if 0 <= rh[i] < R and 0 <= tp[i] < 90:
                    # Tăng giá trị (bỏ phiếu) tại ô tương ứng trong không gian Hough
                    ho[int(rh[i]), int(tp[i])] += mx
```
---
#### 5.2 Dò tìm đường tròn
- Xác định vị trí và bán kính của các đường tròn có mặt trong ảnh số
- Mục đích: Phát hiện các nhóm điểm biên nằm trên chu vi của một đường tròn có bán kính và tâm xác định.
- Công thức toán học:
  ![image](https://github.com/user-attachments/assets/ab30e72f-4807-40a1-9afe-37ed6e2711f4)
- Trong đó:
- `(x,y)`: tọa độ điểm ảnh
- `(a,b)`: tọa độ tâm đường tròn
- `r` : bán kính
- Code chính:
```python
# Đọc file ảnh 'bird.png' và lưu vào biến data
data = iio.imread('bird.png')
# Chuyển ảnh màu (data) sang ảnh thang độ xám
image_gray = rgb2gray(data)
# Áp dụng thuật toán Harris để phát hiện góc trên ảnh xám, k là tham số độ nhạy
coordinate = corner_harris(image_gray, k = 0.001)
```
### 6. Image matching 
- SIFT là một thuật toán phát hiện và mô tả đặc trưng ảnh được dùng để tìm các điểm tương ứng giữa nhiều ảnh khác nhau.
- Mục đích: phát hiện các điểm đặc trưng nổi bật (keypoints) trong ảnh, sử dụng để so khớp ảnh.
- Công thức:
![image](https://github.com/user-attachments/assets/413704aa-cbbf-4fb8-89e2-56f25a951532)
- Trong đó:
- `L(x,y)` : ảnh sau khi làm mờ Gaussian.
- `(x,y)`: tọa độ điểm ảnh
- Code chính:
```python
# Dùng hàm harris_keypoint() để phát hiện đặc trưng
def harris_keypoints(gray, k=0.04, thresh=0.01):
    harris = cv2.cornerHarris(np.float32(gray), blockSize=2, ksize=3, k=k)
    harris = cv2.dilate(harris, None)
    keypoints = np.argwhere(harris > thresh * harris.max())
    return [cv2.KeyPoint(float(x), float(y), 3) for y, x in keypoints]
--------------------------------
# tính descriptor bằng sift
sift = cv2.SIFT_create()
kp1, des1 = sift.compute(gray1, kp1)
kp2, des2 = sift.compute(gray2, kp2)
print("SIFT descriptor:", des1.shape, des2.shape)
------------------------------------
#Ghép cặp và lọc bằng RANSAC
bf = cv2.BFMatcher(cv2.NORM_L2, crossCheck=True)  # L2 norm cho SIFT
matches = bf.match(des1, des2)
matches = sorted(matches, key=lambda m: m.distance)
```
---
## Tài liệu tham khảo:
- ***Rafael C. Gonzalez, Richard E. Wods - Digital Image Processing-Pearson (2017)***
- Slide bài giảng Nhập môn Xử lý ảnh số - Văn Lang University
  
