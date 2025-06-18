## ***File readme*** phần bài tập *** 2, 3, 4 ***
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
## Bài tập 2: Viết chương trình tạo menu cho phép người dùng chọn các phương pháp biến đổi ảnh như sau:
#### Sử dụng thuật toán biến đổi ảnh theo miền tần suất (Fourier) gồm :  Fast Fourier, Butterworth Lowpass Filter, Butterworth Highpass Filter
1. Với phương pháp Fast Fourier: sử dụng thuật toán fft2 để phân tích ảnh và dùng fftshift để dịch tâm tần số về giữa ảnh. Trong bài này, dùng fftshift để dịch tâm tần số về giữa, chuẩn hóa về thang [0,255]
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

2. Với phương pháp Butterworth Lowpass Filter:  sử dụng những điểm ảnh có tần suất thấp từ biến đổi Fourier có khả năng làm mịn ảnh và khử nhiễu. Trong bài này, áp dụng fft2 để phân tích ảnh, fftshift để dịch tâm tần số về giữa, sử dụng công thức H[i, j] = 1 / (1 + (r / d_0) ** (2 * t1)) để giữ những điểm có tần suất thấp. 
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

3. Với phương pháp Butterworth Highpass Filter:  sử dụng những điểm ảnh có tần suất cao từ biến đổi Fourier để làm sắc biên của ảnh.
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

#### Tạo hàm thực thi và hiển thị ảnh
1. Main code:
```python
def apply_transformation():
```
- Tạo thư mục ảnh đầu vào, đầu ra :
```python

    input_folder = "exercise"
    output_folder = "output_2"
    os.makedirs(output_folder, exist_ok=True)
```

- Lấy 3 ảnh trong thư mục đầu vào, mở và chuyển ảnh sang đen trắng và chuyển thành mảng
```python
# Lọc ra 3 file ảnh đầu tiên có đuôi png/jpg/jpeg trong thư mục "exercise"
    image_files = [f for f in os.listdir(input_folder) if f.lower().endswith((".png", ".jpg", ".jpeg"))][:3]
    processed_images = []
# Áp dụng biến đổi
    for file_name in image_files:
        img_path = os.path.join(input_folder, file_name)
        img = Image.open(img_path).convert("L")
        im_np = np.asarray(img)
# Gọi hàm biến đổi ảnh
        processed_np = transformation_func(im_np)
        processed_img = Image.fromarray(processed_np)
```
- Lưu ảnh
```python
# lưu ảnh
        output_path = os.path.join(output_folder, f"{os.path.splitext(file_name)[0]}_{method_name}.png")
        processed_img.save(output_path)
```
- Hiển thị ảnh sau xử lý
```python
 # Hiển thị ảnh
    fig, axes = plt.subplots(1, len(processed_images), figsize=(15, 5))
 # Trường hợp chỉ có 1 ảnh, ép axes thành list để duyệt được
    if len(processed_images) == 1:
        axes = [axes]
    for ax, (image, fname) in zip(axes, processed_images):
        ax.imshow(image, cmap='gray')
        ax.set_title(fname)
        ax.axis('off')
    plt.suptitle(f"Kết quả: {method_name}")
    plt.show()

```
- Tạo menu cho người dùng : cho phép người dùng chọn phương pháp biến đổi ảnh. Khi người dùng nhập 1 chữ cái (F,L,H) chương trình gọi đúng hàm tương ứng .
```python
def menu():
    while True:
        print("=== ỨNG DỤNG BIẾN ĐỔI ẢNH ===")
        print("\tF. Fast Fourier")
        print("\tL. Butterworth Lowpass Filter")
        print("\tH. Butterworth Highpass Filter")
        print("\tE. Thoát")

        luachon = input("Nhập lựa chọn của bạn: ").upper()
        match luachon:
            case 'F':
                apply_transformation(Fast_Fourier, "Fast")
            case 'L':
                apply_transformation(Butterworth_Lowpass_Filter, "Lowpass")
            case 'H':
                apply_transformation(Butterworth_Highpass_Filter, "Highpass")
            case 'E':
                print("Tạm biệt!")
                break
            case _:
                print("Lựa chọn không hợp lệ!")

menu()
```
## Bài tập 3: Viết chương trình thay đổi thứ tự màu RGB của ảnh trong thư mục exercise và sử dụng ngẫu nhiên một trong các phép biến đổi ảnh trong câu 1. Lưu và hiển thị ảnh đã biến đổi.
#### Trong bài nay sử dụng các thuật toán biến đổi ảnh trong câu 1 bao gồm Image inverse transformation, Gamma-Correction, Log Transformation, Histogram equalization, Contrast Stretching.
1. Phép biến đổi cường độ ảnh còn được gọi là Image inverse transformation : Trong bài, mỗi pixel sẽ bị 255 trừ ra để đổi ngược màu (sáng thành tối và ngược lại)
- Định nghĩa hàm  Image inverse transformation()
```python
def Image_inverse_transformation(im_1):
    return 255 - im_1
```
2. Phương pháp thay đổi chất lượng ảnh với Power Law ( Gamma-Correction): Trong bài này, dùng chuẩn hóa pixel về [0,1] áp dụng công thức gamma làm tăng độ sáng và chuyển về thang [0,255].
- Định nghĩa hàm  Gamma-Correction()
```python
def Gamma_Correction(im_1, gamma=0.5):
    im_1 = np.asarray(im_1, dtype=np.float32) / 255.0
    corrected = np.power(im_1, gamma)
    return np.clip(corrected * 255, 0, 255).astype(np.uint8)
```
3. Phương pháp thay đổi cường độ ảnh với log transformation: trong bài này chia cho log(1+b2) để chuẩn hóa kết quả.
- Định nghĩa hàm  Log_Transformation()
```python
def Log_Transformation(im_1):
    b1 = im_1.astype(np.float32)
    b2 = np.max(b1)
    if b2 == 0:
        return np.zeros_like(im_1, dtype=np.uint8)
    c = (128.0 * np.log(1 + b1)) / np.log(1 + b2)
    return np.clip(c, 0, 255).astype(np.uint8)
```
4. Histogram equalization: trong bài dùng để tạo ảnh mới rõ nét với độ sáng cân bằng bằng cách chuẩn hóa histogram bằng hàm tích lũy CDF.
- Định nghĩa hàm Histogram_equalization()
```python
def Histogram_equalization(im_1):
    flat = im_1.flatten()
    hist, _ = np.histogram(flat, 256, [0, 256])
    cdf = hist.cumsum()
    cdf_m = np.ma.masked_equal(cdf, 0)
    cdf_m = (cdf_m - cdf_m.min()) * 255 / (cdf_m.max() - cdf_m.min())
    cdf = np.ma.filled(cdf_m, 0).astype('uint8')
    equalized = cdf[flat]
    return np.reshape(equalized, im_1.shape)
```
5. Thay đổi ảnh với Contrast Stretching: dùng công thức kéo dãn khoảng sáng để ảnh rõ hơn.
- Định nghĩa hàm Contrast Stretching()
```python
def Contrast_Stretching(im_1):
    a = np.min(im_1)
    b = np.max(im_1)
    if a == b:
        return np.zeros_like(im_1, dtype=np.uint8)
    stretched = 255.0 * (im_1 - a) / (b - a)
    return np.clip(stretched, 0, 255).astype(np.uint8)
```
### Tạo hàm thực thi và hiển thị ảnh
1. Tạo danh sách chứa tên các phép biến đổi và hàm tương ứng,  
 ```python
 # Danh sách hàm xử lý xám
gray_transforms = [
    ("Negative", Image_inverse_transformation),
    ("Log", Log_Transformation),
    ("Gamma", lambda img: Gamma_Correction(img, gamma=0.5)), #gán gamma = 0.5
    ("Histogram", Histogram_equalization),
    ("Stretch", Contrast_Stretching)
]
```
2. Đảo thứ tự màu RGB ngẫu nhiên nhờ random.shuffle()
- Định nghĩa hàm đảo shuffle_rgb()
```python
# ===== Hàm đổi thứ tự màu RGB =====
def shuffle_rgb(img_rgb):
    img_np = np.array(img_rgb)
    channels = [0, 1, 2] # chỉ số của kênh R, G, B
    random.shuffle(channels)  # trộn ngẫu nhiên thứ tự
    shuffled = img_np[:, :, channels]
    return Image.fromarray(shuffled), channels
```
3. Main code:
```python
def random_color_and_gray_transform():
```
- Tạo thư mục ảnh đầu vào, đầu ra :
```python
    input_folder = "exercise"  #
    output_folder = "output_3"
    os.makedirs(output_folder, exist_ok=True)
```
- Lấy 3 ảnh trong thư mục đầu vào, mở và chuyển sang RGB.
```python
    image_files = [f for f in os.listdir(input_folder) if f.lower().endswith((".png", ".jpg", ".jpeg"))]
    if not image_files:
        print("Không tìm thấy ảnh trong thư mục 'exercise'")
        return
    for file_name in image_files:
        img_path = os.path.join(input_folder, file_name)
        img = Image.open(img_path).convert("RGB")
```
- Đổi thứ tự màu RGB và chuyển sang ảnh xám
```python
        # đổi thứ tự RGB ngẫu nhiên
        shuffled_img, channel_order = shuffle_rgb(img)

        # chuyển sang ảnh xám để xử lý
        gray_img = shuffled_img.convert("L")
        gray_np = np.asarray(gray_img)

        # chọn biến đổi ảnh xám ngẫu nhiên
        method_name, method_func = random.choice(gray_transforms)
        transformed = method_func(gray_np)
```
- Lưu ảnh
```python
        #  lưu ảnh
        output_img = Image.fromarray(transformed)
        output_path = os.path.join(output_folder, f"{os.path.splitext(file_name)[0]}_RGB{channel_order}_{method_name}.png")
        output_img.save(output_path)
```
- Hiển thị kết quả
```python
        # Hiển thị
        plt.imshow(output_img, cmap='gray')
        plt.title(f"{file_name} - RGB{channel_order} - {method_name}")
        plt.axis('off')
        plt.show()
#gọi hàm chính xử lý
random_color_and_gray_transform()
```
## Bài tập 4. Viết chương trình thay đổi thứ tự màu RGB của ảnh trong thư mục exercise và sử dụng ngẫu nhiên một trong các phép biến đổi ảnh trong câu 2. Nếu ngẫu nhiên là phép Butterworth Lowpass thì chọn thêm Min Filter để lọc ảnh. Nếu ngẫu nhiên là phép Butterworth Highpass thì chọn thêm Max Filter để lọc ảnh. Lưu và hiển thị ảnh đã biến đổi.
#### Trong bài nay sử dụng các thuật toán biến đổi ảnh trong câu 2 bao gồm Fast Fourier, Butterworth Lowpass Filter, Butterworth Highpass Filter.
1. Phép biến đổi Fast Fourier: Trong bài này, dùng fftshift để dịch tâm tần số về giữa, chuẩn hóa về thang [0,255]
- Định nghĩa hàm fast_fourier()
```python
def fast_fourier(img):
    # Biến đổi Fourier 2D
    f = np.fft.fft2(img)
    # Dịch tâm phổ (center the frequency spectrum)
    fshift = np.fft.fftshift(f)
    # Tính biên độ phổ và lấy log 
    mag = 20 * np.log(np.abs(fshift) + 1)
    # Chuyển về kiểu uint8 sau khi giới hạn giá trị trong [0,255]
    return Image.fromarray(np.clip(mag, 0, 255).astype(np.uint8))
```
2. Phép biến đổi Butterworth Lowpass thêm Min Filter để lọc ảnh
- Định nghĩa hàm butter_lowpass()
```python
def butter_lowpass(img, D0=30):
    # Lấy kích thước ảnh
    r, c = img.shape
    # Tạo lưới tọa độ và dịch về tâm ảnh
    x, y = np.meshgrid(np.arange(r) - r//2, np.arange(c) - c//2, indexing='ij')
    d = np.sqrt(x**2 + y**2)  # khoảng cách tới tâm ảnh
    # Tạo mặt nạ Butterworth thông thấp bậc 2
    h = 1 / (1 + (d / D0)**2)
    # Biến đổi Fourier
    f = np.fft.fft2(img)
    fshift = np.fft.fftshift(f)
    # Áp dụng mặt nạ lọc
    fnew = fshift * h
    # Biến đổi ngược để về không gian ảnh
    inv = np.abs(np.fft.ifft2(np.fft.ifftshift(fnew)))
    # Giới hạn giá trị ảnh và làm mịn thêm bằng minimum_filter
    out = np.clip(inv, 0, 255).astype(np.uint8)
    return Image.fromarray(minimum_filter(out, size=3))
```
3. Phép biến đổi Butterworth Highpass thêm Max Filter để lọc ảnh
- Định nghĩa hàm butter_lowpass()
```python
def butter_highpass(img, D0=30):
    # Lấy kích thước ảnh
    r, c = img.shape
    # Tạo lưới tọa độ và dịch về tâm ảnh
    x, y = np.meshgrid(np.arange(r) - r//2, np.arange(c) - c//2, indexing='ij')
    d = np.sqrt(x**2 + y**2)  # khoảng cách tới tâm ảnh
    # Tạo mặt nạ Butterworth thông cao bậc 2
    h = 1 / (1 + (D0 / (d + 1e-8))**2)  # cộng 1e-8 để tránh chia cho 0
    # Biến đổi Fourier
    f = np.fft.fft2(img)
    fshift = np.fft.fftshift(f)
    # Áp dụng mặt nạ lọc
    fnew = fshift * h
    # Biến đổi ngược để về không gian ảnh
    inv = np.abs(np.fft.ifft2(np.fft.ifftshift(fnew)))
    # Giới hạn giá trị ảnh và làm sắc nét thêm bằng maximum_filter
    out = np.clip(inv, 0, 255).astype(np.uint8)
    return Image.fromarray(maximum_filter(out, size=3))
```
#### Tạo hàm thực thi và hiển thị ảnh
1. Tạo danh sách chứa tên các phương pháp và hà tương ứng
```python
filters = [
    ("Fourier", fast_fourier),
    ("Lowpass", butter_lowpass),
    ("Highpass", butter_highpass)
]
```
2. Thư mục ảnh đầu vào , đầu ra
```python
folder = 'exercise'
output = 'output_4'
os.makedirs(output, exist_ok=True)
```
3. Main code
- Lấy 3 ảnh trong thư mục đầu vào, mở và chuyển sang RGB
```python
def process_images():
    for file in os.listdir(folder):
        if not file.lower().endswith((".jpg", ".png", ".jpeg")):
            continue

        img_path = os.path.join(folder, file)
        img = Image.open(img_path).convert('RGB')
```
- Đảo thứ tự màu RGB , chuyển sang ảnh xám và chọn hàm lọc ngẫu nhiên
```python
        # Đảo thứ tự màu RGB → GBR
        r, g, b = img.split()
        img = Image.merge('RGB', (g, b, r))

        gray = img.convert('L')
        gray_np = np.asarray(gray)

        # Chọn hàm lọc ngẫu nhiên
        name, func = random.choice(filters)
        print(f"Đang áp dụng: {name} cho ảnh {file}")

        result = func(gray_np)
```
- Lưu ảnh và hiển thị
```python
        # Lưu
        save_name = f"{name}_{file}"
        result.save(os.path.join(output, save_name))
        # Hiển thị kết quả
        plt.imshow(result, cmap='gray')
        plt.title(save_name)
        plt.axis('off')
        plt.show()

# CHẠY
process_images()
``` 






 





