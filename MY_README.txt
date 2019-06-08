1. Kernel version : 4.14.25
2. gcc version : 4.9
3. 自己寫的部份以 /////////////////////////////////////////////////////////////// 框起來
4. 程式以 sample code for 4.14.25 為基礎撰寫

目前遇到的問題：
1. Master 使用 mmap 時，使用測試資料 File3_in 及 File4_in 無法傳完所有資料
2. 看到的問題點是在 master_device.c ksend(sockfd_cli, file->private_data, ioctl_param, 0); 無法傳完所有資料

測試方式請參照原來的 README.txt，建議先啟動 master 再執行 slave

Kernel 下載網址
https://kernel.ubuntu.com/~kernel-ppa/mainline/v4.14.25/