# HƯỚNG DẪN: Biến Điện Thoại / TV Box Cũ Thành Bộ Lọc Quảng Cáo AdGuard Home Bằng Termux (Yêu Cầu Root)
# Cài adguard home trên tv box cũ điện thoại cũ !
# Cài adguard home lên termux
# install adguard home on termux

Chào bạn, đây là một hướng dẫn rất hữu ích để tận dụng điện thoại hoặc TV Box cũ làm bộ lọc quảng cáo (DNS Server) cho cả gia đình.
Ở đây mình cài trên kiwi box s1 cpu allwinner h2+ ram 1gb đã up fw adnroid 7.1 đã root sẵn.
Để giúp các bạn mới bắt đầu dễ dàng làm theo mà không bị lỗi, mình đã tối ưu, sửa một số lỗi nhỏ trong đoạn code của bạn (như lệnh `chmod` viết hoa, sai đường dẫn khi khởi động) và trình bày lại thật sạch sẽ, rõ ràng từng bước dưới đây.

---

## Chuẩn bị trước khi cài đặt

* **Thiết bị:** Điện thoại Android hoặc TV Box cũ **ĐÃ ROOT** (bắt buộc vì AdGuard Home cần quyền Root để chạy trên cổng mạng hệ thống).
* Cài đặt IP tĩnh cho thiết bị chạy AdGuard Home: Đây là bước cực kỳ quan trọng. Thiết bị dùng làm AdGuard Home phải được đặt IP tĩnh (Static IP) trong cài đặt WiFi của máy hoặc cấu hình gán IP cố định (DHCP Reservation) trên Modem/Router tổng. Nếu IP này bị thay đổi mỗi khi khởi động lại, các máy khác trong nhà sẽ bị mất mạng Internet.

* **Ứng dụng cần tải:**
1. **Termux** (Nên tải bản mới nhất từ F-Droid, không tải trên CH Play vì bản CH Play đã cũ và lỗi).
2. **Termux:Boot** (Ứng dụng giúp chạy code khi khởi động máy, cũng tải từ F-Droid).

---

## Bước 1: Tải và giải nén AdGuard Home

Mở ứng dụng **Termux** lên, copy toàn bộ lệnh bên dưới, dán vào Termux rồi nhấn **Enter**:

```bash
pkg update && pkg upgrade -y && pkg install wget nano -y

```

*(Lệnh này để cập nhật hệ thống Termux và cài các công cụ cần thiết như wget, tsu, nano).*

# Tắt tính năng check Internet ngầm của Android, giúp box yếu không bị dồn ứ request gây khựng và rớt gói.
```
su
settings put global captive_portal_mode 0
```

Tiếp theo, tùy thuộc vào loại chip (CPU) của máy bạn mà chọn **1 trong 3 dòng lệnh** dưới đây để tải AdGuard Home:

* **Nếu máy bạn chạy chip 32-bit (ARMv7 - dòng máy cũ):**
```bash
wget https://github.com/AdguardTeam/AdGuardHome/releases/download/v0.107.52/AdGuardHome_linux_armv7.tar.gz && tar -vxf AdGuardHome_linux_armv7.tar.gz

```


* **Nếu máy bạn chạy chip 64-bit (ARMv8 / ARM64 - đa số máy đời mới hơn):**
```bash
wget https://github.com/AdguardTeam/AdGuardHome/releases/download/v0.107.52/AdGuardHome_linux_arm64.tar.gz && tar -vxf AdGuardHome_linux_arm64.tar.gz

```


* **Nếu máy chạy chip cũ hơn nữa (ARMv6):**
```bash
wget https://github.com/AdguardTeam/AdGuardHome/releases/download/v0.107.52/AdGuardHome_linux_armv6.tar.gz && tar -vxf AdGuardHome_linux_armv6.tar.gz

```



*(Lưu ý: Mình đã cập nhật link lên bản `v0.107.52` ổn định hơn bản cũ).*

---

## Bước 2: Chạy thử AdGuard Home lần đầu

Sau khi giải nén xong, bạn chạy lệnh sau để vào thư mục và kích hoạt AdGuard Home bằng quyền Root:

```bash
cd ~/AdGuardHome && su -c './AdGuardHome'

```

> **Lúc này:** Điện thoại sẽ hiển thị một bảng hỏi xin quyền Root (Magisk/Su), bạn hãy chọn **Cho phép (Grant)**. Trên màn hình Termux sẽ xuất hiện các đường dẫn IP dạng `http://127.0.0.1:3000` hoặc `http://192.168.x.x:3000`.
> Bạn dùng máy tính hoặc điện thoại khác chung mạng WiFi, mở trình duyệt web truy cập vào địa chỉ IP đó để cài đặt AdGuard Home lần đầu nhé.
> Sau khi cài xong, bạn có thể nhấn **Ctrl + C** ở Termux để tắt đi và làm bước tiếp theo.

---

## Bước 3: Cấu hình tự động chạy khi khởi động máy (Auto-start)

Để mỗi lần bật TV Box hoặc điện thoại lên, AdGuard Home tự chạy mà không cần mở Termux thủ công:

1. Tạo thư mục boot của Termux:
```bash
mkdir -p ~/.termux/boot

```


2. Tạo và mở file cấu hình bằng trình chỉnh sửa Nano:
```bash
nano ~/.termux/boot/start-adguard

```


3. Copy toàn bộ đoạn code dưới đây và dán vào màn hình Termux:
```bash
#!/data/data/com.termux/files/usr/bin/sh

# wake termux and sshd
termux-wake-lock && sshd

# AdGuardHome 
cd AdGuardHome
su -c 'SSL_CERT_FILE=/data/data/com.termux/files/home/AdGuardHome/cacert.pem ./AdGuardHome 2>&1' &

# Update cacert.pem after boot
( sleep 300 && wget -q -O cacert.pem.tmp https://curl.se/ca/cacert.pem && mv cacert.pem.tmp cacert.pem ) &

```


4. **Cách lưu và thoát:**
* Nhấn tổ hợp phím: **Ctrl + O** rồi nhấn **Enter** (để lưu file).
* Nhấn tiếp tổ hợp phím: **Ctrl + X** (để thoát ra lại màn hình chính).


5. **Cấp quyền thực thi cho file (Rất quan trọng):**
Bạn chạy lệnh sau (chữ `chmod` phải viết thường hoàn toàn):
```bash
chmod +x ~/.termux/boot/start-adguard

```



---

## Bước cuối cùng cần làm trên điện thoại

1. Mở ứng dụng **Termux:Boot** lên một lần (bấm vào biểu tượng app trên màn hình điện thoại) để kích hoạt tính năng chạy cùng hệ thống.
2. Vào phần **Cài đặt pin** trên điện thoại/TV Box, tìm ứng dụng **Termux** và **Termux:Boot**, chọn **Không tối ưu hóa pin (Unrestricted / No restriction)** để tránh việc Android tự động tắt ứng dụng khi chạy ngầm.

Bây giờ bạn có thể khởi động lại máy (Reboot) để tận hưởng thành quả! Chúc bạn thành công!
