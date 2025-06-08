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
### Trong bài này có 2 thuật toán chính :
- Point Processing gồm phần nhỏ là :Image inverse transformation, Gamma-Correction, Log Transformation, Histogram equalization, Contrast Stretching.
- Biến đổi ảnh theo miền tần suất (Fourier) gồm 2 phần là: Biến đổi ảnh với Fast Fourier và lọc ảnh trong miền tần suất có 2 cái nhỏ là: Butterworth Lowpass Filter, Butterworth Highpass Filter.
## Giải thích cách hoạt động:
#### 1. Point Processing: thuật toán này dùng để xử lý ảnh bằng cách thay đổi giá trị của từng pixel , dựa trên giá trị cường độ ban đầu của nó.
 1.1 Biến đổi cường độ ảnh (Image inverse transformation): Là phép biến đổi đảo ngược mức sáng của ảnh từ vùng sáng sang vùng tối và ngược lại từ vùng tối sang vùng sáng. 
 - Có công thức là s = (L-1) - r . Trong đó, giá trị pixel mới (s), giá trị cường độ tối đa (L-1), giá trị pixel hiện tại (r). Trong bài này, s được gán là im_2, L=256 và r là im_1.
```python
# Công thức biến đổi cường độ ảnh
im_2 = 255 - im_1
```
1.2 Thay đổi chất lượng ảnh với Power law (Gamma-Correction): Dùng để tăng chất lượng của ảnh, làm sáng các vùng tối (với γ < 1) hoặc làm tối các vùng sáng (với γ < 1).
-Có công thức là s = c * r^γ. Trong đó, giá trị pixel mới (s), giá trị gamma (γ), giá trị pixel hiện tại (r) và một hằng số c. Trong bài này,thực hiện bằng cách chuẩn hóa giá trị pixel về [0,1], áp dụng phép biến đổi rồi chuyển về thang [0,255].
```python
b1 = im_1.astype(float)
#tìm giá trị lớn nhất trong b1
b2=np.max(b1)
#chuẩn hóa b1 (tức là chuyển các giá trị b1 vê thuộc khoảng [0,1])
b3 = (b1+1)/b2
#tính tương qun hàm mũ gama
b2 = np.log(b3)*gamma
#tính tương quan gama và quy về khoảng [0,1]
c= np.exp (b2) * 255.0
```
1.3 Thay đổi cường độ điểm ảnh với Log Transformation: dùng để làm nổi bật các chi tiết trong vùng tối và giảm chói vùng sáng. 
- Công thức s = c * log(1 + r). Trong đó, giá trị pixel mới (s), giá trị pixel hiện tại (r) và một hằng số c. Trong bài này, thực hiện biến đổi log chuyển hóa về 128 giúp giữ ảnh vừa phải không bị quá chói.
```python
#chuyển matrix từ số nguyên sang số thực 
b1 = im_1.astype (float)
#tìm giá trị lớn nhất trong b1
b2 = np.max (b1)
#biến đổi log
c = (128.0 * np.log(1 + b1))/np.log (1 + b2)
```
1.4 Histogram equalization: dùng để cân bằng độ sáng tối cho ảnh nổi bật và dễ nhìn hơn. Hàm sử dụng xác suất, CDF(Cumulative Distribution) để tính. 
- Trong bài này, thực hiện cân bằng lược đồ bằng cách đếm số pixel cho từng mức xám để tạo histogram (cdf = hist.cumsum()) sau đó chuẩn hóa cdf và dùng nó để tạo ảnh mới.
```python
# thực hiện cân bằng lược đồ
num_cdf_m = (cdf_m - cdf_m.min()) * 255
den_cdf_m = (cdf.max () - cdf_m.min ())
cdf_m = num_cdf_m/den_cdf_m
```
1.5 Thay đổi ảnh với Contrast Stretching: dùng để tăng độ tương phản tương tự Histogram Equalization nhưng bằng cách kéo dãn dải giá trị pixel (thay đổi giấ tri pixel) thay vì dùng xác suất, CDF để tính. Công thức s = (r - r_min) * (255 / (r_max - r_min)), trong bài này a là min và b là max.
```python
#công thức biến đổi giãn độ tương phản
im2 = 255* (c- a)/(b - a)
```
#### 2. Fourier: Dùng để biến đổi ảnh theo miền tần suất. 
2.1  Biến đổi ảnh với Fast Fourier: sử dụng thuật toán fft2 để phân tích ảnh và dùng fftshift để dịch tâm tần số về giữa ảnh.
```python
# Thực hiện biến đổi fft
c= abs(scipy.fftpack.fft2 (iml))
# Dịch tâm tần số về giữa (center shift)
d= scipy.fftpack.fftshift (c)
```
2.2  Lọc ảnh trong miền tần suất: Có hai dạng là lowpass và highpass. 
- Butterworth Lowpass Filter: sử dụng điểm ảnh có tần suất thấp từ phép biến đổi Fourier dùng để khử nhiễu và làm mịn ảnh.Trong bài này, r là khoảng cách từ điểm [i, j] đến tâm ảnh, d_0 là bán kính cắt, d_0 = 30.0, t1 = 1 (bậc của bộ lọc)
  ```python
  #sử dụng bán kính cắt để loại bỏ tần số cao(nhiễu) với điều kiện 
  H[i, j] = 1 / (1 + (r / d_0) ** t1)
  ```
- Butterworth highpass Filter: sử dụng điểm ảnh có tần suất cao từ phép biến đổi Fourier dùng để làm sắc biên của ảnh. Trong bài này, r là khoảng cách từ điểm [i, j] đến tâm ảnh, d_0 là bán kính cắt, d_0 = 30.0, t2 = 2*t1(bậc của bộ lọc)
```python
 H[i, j] = 1 / (1 + (r / d_0) ** t2)
```
### Phần bài tập:
#### Sử dụng thuật toán Point Processing để viết chương trình cho phép người dùng chọn phương pháp biến đổi ảnh sau:
- Image inverse transformation:dùng biến đổi đảo ngược mức sáng của ảnh từ vùng sáng sang vùng tối và ngược lại. Mỗi pixel sẽ bị 255 trừ ra để đổi ngược màu (sáng thành tối và ngược lại)
```python
#công thức tính Image inverse transformation
def Image_inverse_transformation(im_1):
    return 255 - im_1
```
- Gamma-Correction:Dùng để tăng chất lượng của ảnh. Trong bài này, dùng chuẩn hóa pixel về [0,1] áp dụng công thức gamma làm tăng độ sáng và chuyển về thang [0,255].
```python
#công thức tính Gamma-Correction:
def Gamma_Correction(im_1, gamma=0.5):
    normalized = im_1 / 255.0
    corrected = np.power(normalized, gamma)
    return np.clip(corrected * 255, 0, 255).astype(np.uint8)
```
- Log Transformation: Biến đổi log làm rõ vùng tối, giảm chói. Chia cho log(1+b2) để chuẩn hóa kết quả
 ```python
def Log_Transformation(im_1):
    b1 = im_1.astype(float)
    b2 = np.max(b1)
    c = (128.0 * np.log(1 + b1)) / np.log(1 + b2)
    return np.clip(c, 0, 255).astype(np.uint8)
```
- Histogram equalization: trong bài dùng để tạo ảnh mới rõ nét với độ sáng cân bằng bằng cách chuẩn hóa histogram bằng hàm tích lũy CDF
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
- Contrast Stretching: dùng để kéo giãn độ sáng giữa các pixel để nhìn rõ hơn bằng cách tìm giá trị sáng và tối nhất trong ảnh đó và dùng công thức kéo giãn khoảng sáng để ảnh rõ hơn.
```python
def Contrast_Stretching(im_1):
    a = im_1.min()
    b = im_1.max()
    stretched = 255 * (im_1 - a) / (b - a)
    return np.clip(stretched, 0, 255).astype(np.uint8)
```
#### Tạo hàm thực thi và hiển thị ảnh
- Tạo thư mục ảnh đầu vào, đầu ra :
```python
def apply_transformation(transformation_func, method_name):
    input_folder = "exercise"
    output_folder = "output"
    os.makedirs(output_folder, exist_ok=True)
```
- Lấy 3 ảnh trong thư mục đầu vào, mở và chuyển ảnh sang đen trắng và chuyển thành mảng
```python
 image_files = [f for f in os.listdir(input_folder) if f.lower().endswith((".png", ".jpg", ".jpeg"))][:3]
    processed_images = []
    for file_name in image_files:
        img_path = os.path.join(input_folder, file_name)
        img = Image.open(img_path).convert("L")
        im_np = np.asarray(img)
```
- Gọi hàm biến đổi ảnh
```python
processed_np = transformation_func(im_np)
processed_img = Image.fromarray(processed_np)
```
- Lưu ảnh
```python
output_path = os.path.join(output_folder, f"{os.path.splitext(file_name)[0]}_{method_name}.png")
processed_img.save(output_path)
```
- Hiển thị ảnh sau xử lý
```python
fig, axes = plt.subplots(1, len(processed_images), figsize=(15, 5))
    if len(processed_images) == 1:
        axes = [axes]
    for ax, (image, fname) in zip(axes, processed_images):
        ax.imshow(image, cmap='gray')
        ax.set_title(fname)
        ax.axis('off')
    plt.suptitle(f"Kết quả: {method_name}")
    plt.show()
```
- Tạo menu cho người dùng bằng match-case (gọn gàng, dễ đọc, dễ mở rộng hơn): cho phép người dùng chọn phương pháp biến đổi ảnh. 
```python
def menu():
    print("=== ỨNG DỤNG BIẾN ĐỔI ẢNH ===")
    print("\tI. Image inverse transformation")
    print("\tG. Gamma Correction")
    print("\tL. Log Transformation")
    print("\tH. Histogram Equalization")
    print("\tC. Contrast Stretching")
    print("\t0. Thoát")

    luachon = input("Nhập lựa chọn của bạn: ").upper()
    match luachon:
        case 'I':
            apply_transformation(Image_inverse_transformation, "Inverse")
        case 'G':
            apply_transformation(Gamma_Correction, "Gamma")
        case 'L':
            apply_transformation(Log_Transformation, "Log")
        case 'H':
            apply_transformation(Histogram_equalization, "Histogram_Eq")
        case 'C':
            apply_transformation(Contrast_Stretching, "Contrast")
        case '0':
            print("Tạm biệt!")
        case _:
            print("Lựa chọn không hợp lệ!")

menu()
```









