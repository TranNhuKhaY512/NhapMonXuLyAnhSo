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
  
### 3. Thay đổi kích thước ảnh
- Mục đích: Phóng to hoặc thu nhỏ ảnh theo tỷ lệ.
- Công thức toán học:
```math
I'(x,y)= I(x/sx , y /sy)
```
- `sx, sy` : lần lượt là hệ số zoom theo chiều ngang và dọc
- Code chính: 
```python
data = iio.imread('fruit.jpg')
data2 = nd.zoom (data, (2, 2, 1)) #phóng to
data3 = nd.zoom(data, (0.5, 0.9, 1)) # thu nhỏ
```

### 4. Xoay ảnh
- Mục đích: xoay ảnh theo góc xoay.
- Dùng hàm rotate(image, degree) để xoay một ảnh với Image: là ảnh trong bộ nhớ, Degree: là góc xoay
- Code chính:
```python
data = iio.imread('fruit.jpg')
d2 = nd.rotate (data, 20, reshape=False)
```

