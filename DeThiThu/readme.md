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
  \( F_i = \frac{1}{N} \).
  ```

