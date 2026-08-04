su -c 'pkill AdGuardHome'

su -c svc wifi disable
su -c svc wifi enable
su -c ip link show wlan0
su -c ip a show wlan0

# vấn đề với giờ
busybox mount -o remount,rw /system 2>/dev/null || toybox mount -o remount,rw /system
chmod 644 /system/framework/framework-res.apk


hwclock -w
// fake đổi giờ
date 070612002026.00

su -c settings put global auto_time 0 && sleep 1 && su -c settings put global auto_time 1

Đã
# cách dung firefox pem
# file cacert.pem tại đường dẫn đó, hãy tải file chứng chỉ chuẩn từ curl (Mozilla)
wget https://curl.se/ca/cacert.pem -O /data/data/com.termux/files/home/AdGuardHome/cacert.pem
 
pkg update ca-certificates -y
su -c 'SSL_CERT_FILE=/data/data/com.termux/files/usr/etc/tls/cert.pem ./AdGuardHome 2>&1'


# ép đổi ngày giờ 
su -c date 070421551900

// file termux boot start-sshd
~ $ cat ~/.termux/boot/start-sshd
#!/data/data/com.termux/files/usr/bin/sh
termux-wake-lock && sshd && cd AdGuardHome && su -c 'SSL_CERT_FILE=/data/data/com.termux/files/home/AdGuardHome/cacert.pem ./AdGuardHome 2>&1' &
su -c settings put global auto_time 0 && sleep 1 && su -c settings put global auto_time 1 && wget https://curl.se/ca/cacert.pem


# Tắt tính năng check Internet ngầm của Android, giúp box yếu không bị dồn ứ request gây khựng và rớt gói.
su
settings put global captive_portal_mode 0





===============================================================================
SYSTEM & CPU FREQUENCY MANAGEMENT LOG
===============================================================================
Device Target   : Termux (Android System)
CPU Cores       : 4 Cores (Quad-Core)
Operation       : Dynamic Frequency Scaling & Pinning (CPU Underclocking)
Timestamp       : 2026-08-04

-------------------------------------------------------------------------------
1. INITIAL TEMPERATURE & CPU FREQUENCY CHECK
-------------------------------------------------------------------------------
$ cat /sys/class/thermal/thermal_zone*/temp
58

$ cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_cur_freq
480000
480000
480000
480000

-------------------------------------------------------------------------------
2. CPU HARDWARE SPECIFICATIONS (SUPPORTED FREQUENCIES)
-------------------------------------------------------------------------------
$ cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_frequencies
480000 648000 720000 816000 912000 1008000

-------------------------------------------------------------------------------
3. FREQUENCY LIMIT CHECK (PERMISSION & SUPERUSER EXECUTIONS)
-------------------------------------------------------------------------------
$ cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_max_freq
cat: /sys/devices/system/cpu/cpu0/cpufreq/scaling_max_freq: Permission denied

$ su -c cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_max_freq
480000

$ cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_cur_freq
480000
480000
480000
480000

-------------------------------------------------------------------------------
4. SUPERUSER COMMAND EXECUTED (LOCK FREQUENCY TO 480MHz)
-------------------------------------------------------------------------------
$ su -c "for cpu in /sys/devices/system/cpu/cpu*/cpufreq; do
    echo 480000 > $cpu/scaling_min_freq
    echo 480000 > $cpu/scaling_max_freq
done"

===============================================================================
SUMMARY:
- Thermal state: 58°C
- Frequency locked: 480,000 kHz (480 MHz) across all 4 cores
- Status: Successfully applied via Root Superuser
===============================================================================
