Cài adguard home trên tv box cũ điện thoại cũ !
Có thể tự chạy khi khởi đông lại !
    +Có app termux
     +LƯU Ý MÁY PHẢI ROOT NHÉ 

-Code tải về và giải nén:
pkg install wget && pkg install tsu && wget https://github.com/AdguardTeam/AdGuardHome/releases/download/v0.107.28/AdGuardHome_linux_armv7.tar.gz && tar vxf AdGuardHome_linux_armv7.tar.gz

(Nhớ Check chip là v gì nhé mình v7 nên để v7 , ai v6 thì sửa thành armv6 nha , còn lại tương tự)

-Code chạy adguard home:

  cd AdGuardHome && rm -f adguardhome.log && su -c './AdGuardHome 2>&1 | tee adguardhome.log'

-cho adguard chạy mỗi khi khởi động nguồn:

+Tải termux boot 
+Chạy:
  mkdir ~/.termux/boot && nano ~/.termux/boot/start-sshd

 +Điền vào file start-sshd:

#!/data/data/com.termux/files/usr/bin/sh
termux-wake-lock && sshd
cd AdGuardHome && rm -f adguardhome.log && su -c './AdGuardHome 2>&1 | tee adguardhome.log'

+Điền xong :
Ctr +o rồi enter để lưu
Ctr +x để thoát

Cấp quyền thực thi 
Chmod +x ~/.termux/boot/start-sshd
