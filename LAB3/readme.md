# Nhập Môn Xử Lý Ảnh Số - Lab 3 
## **BIẾN ĐỔI HÌNH HỌC**
**Sinh viên thực hiện:** Trần Như Khả Ý
**MSSV:** 2374802010582
**Môn học:** Nhập môn Xử lý ảnh số  
**Giảng viên:** Đỗ Hữu Quân

## Các công nghệ sử dụng:
- **Python**: Ngôn ngữ chính                           
- **Pillow (PIL)**: Đọc, chuyển đổi, và lưu ảnh              
- **NumPy**: Xử lý ảnh dưới dạng mảng số học, hỗ trợ thao tác trên ma trận         
- **ImageIO**: Đọc file ảnh với định dạng hiện đại      
- **Matplotlib**: Hiển thị ảnh trực quan
- **os** : hỗ trợ quản lý file
- **scipy** : xử lý ảnh đa chiều, dùng cho các phép biến đổi, lọc ảnh.
  
## Chi tiết các phép biến đổi :
---
### 1. Chọn đối tượng trong ảnh: Là phép trích ảnh nhỏ trong ảnh lớn ban đầu.
- Mục đích: cắt 1 vùng nhỏ (1 đối tượng) trong ảnh.
- Chọn vùng theo tọa độ [(y1 : y2),(x1 : x2)]
- y1, y2 lần lượt là dòng trên cùng của vùng cần cắt và vùng dưới cùng ; x1, x2 lần lượt là cột bên trái của vùng cần cắt và cột bên phải .
- Ví dụ: muốn lấy hình quả cam trong ảnh fruit.jpg với tọa độ 800 : 1200, 570:980. Cắt ảnh từ dòng 800–1199, cột 570–979 (400 × 410 pixel)
- code chính: 
```python
data = iio.imread('fruit.jpg')
bmg = data[800:1200, 570:980]
```
---
### 2. Tịnh tiến đơn
- Mục đích: dịch chuyển toàn bộ ảnh có thể sang trái/ phải/ lên trên/ xuống dưới.
- Sau khi dịch chuyển ảnh thì sẽ xuất hiện vùng trống và vùng trống đó màu đen (giá trị 0).
- Công thức toán học:
```math
I'(x,y)=I(x−Δx,y−Δy)
```
- ` I'(x,y) ` :  cường độ điểm ảnh sau xử lý
- ` I(x,y) ` : cường độ ảnh ban đầu
- `Δx` : số pixel muốn dịch theo chiều ngang (trục x từ trái sang phải) 
- `Δy` : số pixel muốn dịch theo chiều dọc (trục y từ trên xuống)
- Ví dụ : muốn tịnh tiến ảnh sang trái 20px và lên trên 10px: I'(x,y) = I( x - (-20), y - (-10)) = I(x + 20, y + 10)
- Code chính:
```python
data = iio.imread('fruit.jpg')
bdata = nd.shift (data, (-10, -20), order=0) # nd.shift(data, (dy,dx), order=0)
```
- **Lưu ý**:
- Dịch lên trên: dy âm
- Dịch xuống dưới: dy dương
- Dịch sang trái : dx âm
- Dịch sang phải : dx dương.

---
### 3. Thay đổi kích thước ảnh
- Mục đích: Phóng to hoặc thu nhỏ ảnh theo tỷ lệ.
- Công thức toán học:
```math
I'(x,y)= I(x/sx , y /sy)
```
- `sx, sy` : lần lượt là hệ số zoom theo chiều ngang và dọc
- Ví dụ muốn phóng to đối tượng x2 lần: sx = sy = 2, giá trị ảnh mới tại (x,y) là nội suy từ ảnh gốc tại (x/2,y/2)
- Code chính: 
```python
data = iio.imread('fruit.jpg')
data2 = nd.zoom (data, (2, 2, 1)) #phóng to
data3 = nd.zoom(data, (0.5, 0.9, 1)) # thu nhỏ
```
---
### 4. Xoay ảnh
- Mục đích: xoay ảnh theo góc xoay. Ví dụ ảnh bị nghiêng thì xoay về cho đúng hướng chuẩn.
- Dùng hàm rotate(image, degree) để xoay một ảnh với Image: là ảnh trong bộ nhớ, Degree: là góc xoay
- Ví dụ xoay 1 ảnh với 1 góc 45 độ lúc này degree = 45 -> rotate(image,45)
- Code chính:
```python
data = iio.imread('fruit.jpg')
d2 = nd.rotate (data, 20, reshape=False)
```
---
### 5. Dilation và Erosion (Giãn và co ảnh nhị phân)
- Mục đích: Dùng để loại bỏ những pixel nhiễu.
- Dilation thay thế pixel tọa độ (i, j) bằng giá trị lớn nhất của những pixel lân cận (kề).(giãn) 
- Erosion thay thế pixel tọa độ (i, j) bằng giá trị nhỏ nhất của những pixel lân cận (kề).(co)
- code chính
```python
data = iio.imread('world_cup.jpg', mode = 'L')
d1 = nd.binary_dilation (data)
d2 = nd.binary_dilation (data, iterations=3) # lặp ảnh giãn 3 lần, vùng trắng dày hơn
```
---
### 6. Coordinate Mapping (biến dạng theo tọa độ)
- Mục đích: Tạo hiệu ứng ngẫu nhiên, biến dạng ảnh.
- Công thức:
```math
 (x ′,y ′)=(x+δx(x,y), y+δy (x,y))
```
- `𝛿𝑥,𝛿𝑦`: dịch chuyển ngẫu nhiên theo mỗi điểm ảnh
- Code chính:
```python
V, H= data.shape
M = np.indices((V, H))
d = 5
q=2 * d * np.random.ranf (M.shape) - d
mp = (M + q).astype (int)
dl = nd.map_coordinates (data, mp)
```
---
### 7. Biến đổi chung (Generic Transformation)
- Mục đích: Dùng khi ta muốn biến đổi các ảnh chung phép toán do người dùng định nghĩa, mỗi pixel bị dời vị trí theo dạng sóng
- Công thức:
```math
x′=x+10*cos( x/10), y′=y+10*cos( y/10 )
```
- Code chính:
```python
def GeoFun (outcoord):
    a = 10 * np.cos (outcoord[0]/10.0) + outcoord[0]
    b = 10 * np.cos (outcoord[1]/10.0) + outcoord[1]
    return a, b
```
## Phần bài tập trong lớp
### Bài 1. Viết chương trình chọn quả kiwi từ ảnh colorful-ripe-tropical-fruits.jpg trong thư mục exercise. Tinh tiến quả kiwi sang phải 30 pixels.
- Trong bài này dùng phép chọn và phép tịnh tiến đối tượng đó.
- Với mục đích: cắt 1 vùng nhỏ (đối tượng) trong ảnh và dùng hàm shift() để dịch chuyển đối tượng đó sang phải 30px
- Code chính:
```python
bmg = data [120:270, 130:320] # cắt đối tượng trong ảnh
iio.imsave('kiwi.jpg', bmg) # lưu ảnh để dễ gọi
data = iio.imread('kiwi.jpg', mode ='L') # đọc ảnh mới vừa cắt
bdata = nd.shift (data, (0, 30), order=0) # phép tịnh tiến đối tượng sang phải 30px
```
### Bài 2. Viết chương trình chọn quả đu đủ và quả dưa hấu từ ảnh colorful-ripe-tropical-fruits.jpg trong thư mục exercise. Đổi màu hai đối tượng này.
- Trong bài này sử dụng phép chọn và đổi màu RGB cho đối tượng.
- Với mục đích: cắt 2 đối tượng quả đu đủ và dưa hấu, sau đó biến đổi màu cho 2 đối tượng này.
- Code chính:
```python
# đọc ảnh gốc
data = iio.imread('exercise/colorful-ripe-tropical-fruits.jpg')
# cắt vùng chứa đu đủ và dưa hấu
bmg1 = data [320:810,120:660] # cắt quả đu đu từ 
bmg2 = data [300:1100, 1635:2140] # cắt quả dưa hấu

# phép biến đổi màu cho 2 đối tượng
#Đổi màu đu đủ thành màu xanh (RGB: 0, 255, 0)
dudu[ : , : ] = [0, 255, 0]  # Xanh lá cây
#Đổi màu dưa hấu thành màu tím (RGB: 128, 0, 128)
duahau[ : , : ]= [128, 0, 128]  # Tím
```
### Bài 3. Viết chương trình chọn ngọn núi và con thuyền từ ảnh quang_ninh.jpg trong thư mục exercise. Xoay 2 đối tượng này 1 góc 45 độ và lưu vào máy.
- Trong bài này sử dụng phép chọn đối tượng và phép xoay để xoay đối tượng dùng hàm rotate(image, degree) 
- Mục đích: cắt 2 đối tượng núi và con thuyền sau đó dùng hàm rotate() xoay 2 ảnh với góc xoay là 45 độ để xem sự thay đổi
- Code chính:
```python
# cắt vùng chứa thuyền và núi
bmg1 = data [430:550,490:660]
bmg2 = data [20:350, 400:700]

# xoay ảnh chiếc thuyền và lưu
d1 = nd.rotate (bmg1, 45, reshape=False)
iio.imsave("thuyen_xoay45.jpg", d1) # lưu ảnh vào máy

# xoay ảnh núi và lưu
d2 = nd.rotate (bmg2, 45, reshape=False)
iio.imsave("nui_xoay45.jpg", d2) # lưu ảnh vào máy
```
### Bài 4. Viết chương trình chọn ngôi chùa từ ảnh pagoda.jpg trong thư mục exercise. Tăng kích thước ngôi chùa lên 5 lần và lưu vào máy.
- Trong bài sử dụng phép chọn và phép thay đổi kích thước ảnh dùng hàm zoom()
- Mục đích:  cắt đối tượng ngôi chùa sau đó tăng kích thước đối tượng lên 5 lần
- Code chính:
```python
bmg = data [170:240, 50:140] # cắt vùng chứa ngôi chùa
data_a = nd.zoom (bmg, (5, 5, 1)) # phép thay đổi kích thước tăng 5 lần 
```
### Bài 5. Viết chương trình tạo menu
- Tịnh tiến
- Xoay
- Phóng to
- Thu nhỏ
- Coordinate Map
Khi chọn phím T, X, P, H, C thì hỏi muốn thực hiện trên hình nào từ 3 hình trong thư mục
exercise. Người dùng chọn hình nào thì thực hiện phép biến đổi trên hình đó.
#### Trong bài này sử dụng các phép biến đổi gồm tịnh tiến, xoay, phóng to, thu nhỏ, coordinate map( biến dạng theo tọa độ)
1. Phép tịnh tiến: trong bài này sử dụng hàm shift() để dịch chuyển đối tượng theo pixel
- Định nghĩa hàm tinhtien()
```python
def tinhTien(im_np):
    if im_np.ndim == 2:
        return nd.shift(im_np, (100, 25), order=0)
    else:
        # Áp dụng tịnh tiến từng kênh màu
        return np.stack([nd.shift(im_np[:, :, i], (100, 25), order=0) for i in range(3)], axis=2)
```
- Output của phép biến đổi này:
![image](https://github.com/user-attachments/assets/91818631-0162-4f1b-a88e-e08b896eff36)

2. Phép xoay: trong bài này sử dụng hàm rotate(image, degree) để xoay một ảnh với Image: là ảnh trong bộ nhớ, Degree: là góc xoay, và góc quay mặc định là 20
- Định nghĩa hàm xoay()
```python
def xoay(im_np):
    return nd.rotate(im_np, 20, reshape=False)
```
- Output của phép biến đổi này:
![image](https://github.com/user-attachments/assets/70a271dd-5b54-40a7-8842-f9b5d9aa917d)

3. Phép phóng to: trong bài này sử dụng hàm zoom() để thay đổi kích thước và ở đây ứng dụng phóng to ảnh cho hệ số zoom là 2.
- Định nghĩa hàm phongTo()
```python
def phongTo(im_np):
    if im_np.ndim == 2:
        return nd.zoom(im_np, 2)
    else:
        return nd.zoom(im_np, (2, 2, 1))
```
4. Phép thu nhỏ: trong bài này sử dụng hàm zoom() để thay đổi kích thước ảnh và ở đây áp dụng thu nhỏ ảnh với hệ số zoom là 0.5.
- Định nghĩa hàm thuNho()
```python
def thuNho(im_np):
    if im_np.ndim == 2:
        return nd.zoom(im_np, 0.5)
    else:
        return nd.zoom(im_np, (0.5, 0.5, 1))
```
5. Phép coordinate map: trong bài này sử dụng hàm map_coordinates() và random() để tạo hiệu ứng ngẫu nhiên. 
- Định nghĩa hàm coordinate_mapping()
```python
def coordinate_mapping(im_np):
    # Nếu là ảnh xám
    if im_np.ndim == 2:
        channels = [im_np]
    else:
        # Tách kênh nếu là ảnh màu
        channels = [im_np[:, :, i] for i in range(3)]

    V, H = channels[0].shape
    M = np.indices((V, H))
    d = 5
    q = 2 * d * np.random.ranf(M.shape) - d
    mp = (M + q).astype(int)

    mapped_channels = [nd.map_coordinates(ch, mp) for ch in channels]
    if len(mapped_channels) == 1:
        return mapped_channels[0].astype(np.uint8)
    else:
        return np.stack(mapped_channels, axis=2).astype(np.uint8)
```
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
 image_files = [f for f in os.listdir(input_folder) if f.lower().endswith((".png", ".jpg", ".jpeg"))][:4]
    processed_images = []
# Áp dụng biến đổi
    for file_name in image_files:
        img_path = os.path.join(input_folder, file_name)
        img = Image.open(img_path)

        # Nếu muốn ảnh xám, chuyển sang mode 'L'
        if force_gray:
            img = img.convert("L")
  # Gọi hàm biến đổi ảnh
        im_np = np.asarray(img)
        processed_np = transformation_func(im_np)
        if processed_np.ndim == 2:
            processed_img = Image.fromarray(processed_np.astype(np.uint8), mode='L')
        else:
            processed_img = Image.fromarray(processed_np.astype(np.uint8))
```
- Lưu ảnh
```python
# lưu ảnh
        output_path = os.path.join(output_folder, f"{os.path.splitext(file_name)[0]}_{method_name}.png")
        processed_img.save(output_path)
        processed_images.append((processed_img, file_name))
```
- Hiển thị ảnh sau xử lý
```python
 # Hiển thị ảnh
     fig, axes = plt.subplots(1, len(processed_images), figsize=(15, 5))
    if len(processed_images) == 1:
        axes = [axes]
    for ax, (image, fname) in zip(axes, processed_images):
        ax.imshow(image)
        ax.set_title(fname)
        ax.axis('off')
    plt.suptitle(f"Kết quả: {method_name}")
    plt.show()
```
- Tạo menu cho người dùng : cho phép người dùng chọn phương pháp biến đổi ảnh. Khi người dùng nhập 1 chữ cái (T, X, P, H, C) chương trình gọi đúng hàm tương ứng .
```python
def menu():
    while True:
        print("=== ỨNG DỤNG BIẾN ĐỔI ẢNH ===")
        print("\tT. Tịnh tiến ")
        print("\tX. Xoay ảnh")
        print("\tP. Phóng to ảnh")
        print("\tH. Thu nhỏ ảnh")
        print("\tC. Coordinate Mapping")
        print("\tE. Thoát")

        luachon = input("Nhập lựa chọn của bạn: ").upper()
        match luachon:
            case 'T':
                apply_transformation(tinhTien, "TinhTien", force_gray=True)
            case 'X':
                apply_transformation(xoay, "Xoay", force_gray=False)
            case 'P':
                apply_transformation(phongTo, "PhongTo", force_gray=False)
            case 'H':
                apply_transformation(thuNho, "ThuNho", force_gray=False)
            case 'C':
                apply_transformation(coordinate_mapping, "CoordinateMap", force_gray=True)
            case 'E':
                print("Tạm biệt!")
                break
            case _:
                print("Lựa chọn không hợp lệ!")
# ===== CHƯƠNG TRÌNH ======
menu()
```
--- 
## Phần bài tập tăng cường
### Bài tập 1: Chọn ảnh quả kiwi bất kì .Tịnh tiến quả kiwi 50 pixel sang phải và 30 pixel xuống dưới. Áp dụng hiệu ứng sóng (wave effect) lên quả kiwi bằng cách sử dụng biến đổi tọa độ (map_coordinates) với hàm sin. Lưu ảnh kết quả vào file kiwi_wave.jpg
- Trong bài này sử dụng các phép biến đổi như chọn đối tượng, tịnh tiến, áp dụng hiệu ứng wave bằng phương pháp map_coordinates với hàm sin.
- Mục đích: biến dạng ảnh với hiệu ứng wave bằng cách dùng biến đổi tọa độ map_coordinates với hàm sin.
- Code chính:
```python
bmg = data [120:270, 130:320]
# Tịnh tiến ảnh kiwi 50 pixel sang phải, 30 pixel xuống dưới
bdata = nd.shift (data, shift=(30, 50), order=0)
# Tạo hiệu ứng sóng bằng sin với map_coordinates
rows, cols = bdata.shape
x, y = np.meshgrid(np.arange(cols), np.arange(rows))
# Thêm hiệu ứng sóng vào tọa độ y
# A: biên độ sóng, f: tần số
A , f =5, 0.05
y_wave = y + A * np.sin(2 * np.pi * f * x)
# Áp dụng map_coordinates để biến đổi theo tọa độ sóng
coords = np.array([y_wave, x])
wave_img = nd.map_coordinates(bdata, coords, order=1, mode='reflect')
```
### Bài tập 2: Chọn quả đu đủ và dưa hấu từ google. Đổi màu đu đủ thành gradient từ đỏ sang xanh lá, và dưa hấu thành gradient từ vàng sang tím. Ghép hai quả lên một nền trong suốt (alpha channel) và lưu dưới dạng PNG.
- Trong bài này sử dụng phép biến đổi màu gradient lên 2 đối tượng
- Mục đích: đổi màu ảnh sang màu gradient tạo ấn tượng nổi bật.
- Code chính: định nghĩa hàm apply_vertical_gradient()
```python
# === 2. Hàm tạo gradient theo chiều dọc ===
def apply_vertical_gradient(img_np, color_start, color_end):
    h, w, _ = img_np.shape
    gradient_img = img_np.copy()
    for y in range(h):
        alpha = y / h
        r = int((1 - alpha) * color_start[0] + alpha * color_end[0])
        g = int((1 - alpha) * color_start[1] + alpha * color_end[1])
        b = int((1 - alpha) * color_start[2] + alpha * color_end[2])
        for x in range(w):
            if gradient_img[y, x, 3] > 0:
                gradient_img[y, x, 0] = r
                gradient_img[y, x, 1] = g
                gradient_img[y, x, 2] = b
    return gradient_img

# === 3. Áp dụng gradient ===
dudu_grad = apply_vertical_gradient(dudu_np, (255, 0, 0), (0, 255, 0))         # đỏ → xanh lá
duahau_grad = apply_vertical_gradient(duahau_np, (255, 255, 0), (128, 0, 128)) # vàng → tím
```
### Bài tập 3: Chọn ảnh núi và thuyền .Xoay cả hai đối tượng 45 độ, giữ kích thước ban đầu (reshape=False). Tạo hiệu ứng phản chiếu dọc (vertical mirror) cho cả hai đối tượng sau khi xoay.Ghép cả hai đối tượng lên một canvas trắng và lưu vào mountain_boat_mirror.jpg
- Trong bài này sử dụng hàm rotate() để ứng dụng xoay ảnh , tạo hiệu ứng phản chiếu dọc- lật ảnh theo trục ngang bằng hàm flipud().
- Mục đích: tăng độ đa dạng hình học, tạo hiệu ứng đối xứng.
- Code chính:
```python
# xoay ảnh chiếc thuyền và lưu
d1_rot = nd.rotate (bmg1, 45, reshape=False)
# xoay ảnh núi và lưu
d2_rot = nd.rotate (bmg2, 45, reshape=False)
# Tạo hiệu ứng phản chiếu dọc (lật theo chiều dọc)
d1 = np.flipud(d1_rot)
d2 = np.flipud(d2_rot)
```
### Bài tập 4: Chọn ngôi chùa bất kì.Phóng to ngôi chùa lên 5 lần. Áp dụng một biến đổi hình học tùy chỉnh (geometric transform) để tạo hiệu ứng "uốn cong" (warping) ngôi chùa. Lưu ảnh kết quả vào pagoda_warped.jpg.
- Trong bài này sử dụng phép biến đổi hình học tùy chỉnh (geometric transform) và phóng to đối tượng lên 5 lần.
- Mục đích: tạo hiệu ứng uốn cong nghệ thuật, có thể tăng dữ liệu cho biến dạng thực tế.
- Code chính:
```python
# Tạo lưới tọa độ gốc
x, y = np.meshgrid(np.arange(w), np.arange(h))
# Tạo biến dạng (warping) theo hàm sin
# Làm cong theo chiều ngang (x) → dịch x theo sin của y
amplitude = 20    # biên độ uốn cong
frequency = 2 * np.pi / 150  # tần số sóng

# Tính x mới bằng cách cộng thêm giá trị uốn cong
x_warped = x + amplitude * np.sin(frequency * y)
y_warped = y  # giữ y không đổi

# Áp dụng biến đổi tọa độ
warped = np.zeros_like(data)
for i in range(c):  # áp dụng từng kênh màu
    warped[..., i] = nd.map_coordinates(data[..., i], [y_warped, x_warped], order=1, mode='reflect')
```
### Bài tập 5: Tạo một chương trình menu tương tác cho phép người dùng chọn các phép biến đổi sau: Tịnh tiến (hỏi số pixel di chuyển theo x và y). Xoay (hỏi góc xoay và chọn reshape=True/False).Phóng to/thu nhỏ (hỏi hệ số zoom). Làm mờ Gaussian (hỏi giá trị sigma). Biến đổi sóng (hỏi biên độ sóng). Người dùng chọn ảnh từ 3 ảnh bất kì.
#### Trong bài này sử dụng các phép biến đổi tịnh tiến, xoay, phóng to/ thu nhỏ, làm mờ và biến đổi sóng.
1. Phép tịnh tiến : Dùng để dịch chuyển đối tượng bằng hàm shift(), cho phép hỏi số pixel di chuyển theo x (trục ngang), y (trục dọc). 
```python
def translate(img):
    dx = int(input("  Dịch theo X: "))
    dy = int(input("  Dịch theo Y: "))
    shift = (dy, dx, 0) if img.ndim == 3 else (dy, dx)
    return nd.shift(img, shift, order=0), f"translate_{dx}_{dy}.jpg"
```
2. Phép xoay: sử dụng hàm rotate(image, degree) để xoay một ảnh với Image: là ảnh trong bộ nhớ, Degree: là góc xoay, và cho phép người dùng nhập góc xoay và chọn reshape=True/False. True là tự động mở rộng kích thước ảnh để không bị mất góc ảnh, False là giữ nguyên kích thước ảnh gốc có thể bị mất gốc sau xoay.
```python
def rotate(img):
    angle = float(input("  Góc xoay: "))
    reshape = input("  reshape=True? (y/n): ").strip().lower() == 'y'
    return nd.rotate(img, angle, reshape=reshape), f"rotate_{int(angle)}_{reshape}.jpg"
```
3. Phép phóng to hoặc thu nhỏ : sử dụng hàm zoom() để thay đổi kích thước ảnh, cho phép hỏi hệ số zoom (z). Nếu 
```python
def zoom(img):
    z = float(input("  Hệ số zoom: "))
    zoom_factors = (z, z, 1) if img.ndim == 3 else (z, z)
    return nd.zoom(img, zoom_factors), f"zoom_{z}.jpg"
```





