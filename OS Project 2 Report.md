# OS Project 2 Report

組員：涂世昱，林宸慶，王啟時

## Programming design

### fcntl

使用傳統的方式讓資料從檔案讀取出來後，經由網路傳輸模組將資料送至slave。

### mmap

使用記憶體映射的方式，將檔案的內容映射到一段記憶體中，以增加檔案的存取速度。

 

- **初始化**

  - **master**

    開啟 mast er device 及指定的檔案 (filex_in)

    dev_fd = open("/dev/master_device", O_RDWR)

    file_fd = open (file_name, O_RDWR)

  - **slave**

    dev_fd = open("/dev/slave_device", O_RDWR)

    file_fd = open (file_name, O_RDWR | O_CREAT | O_TRUNC)

- **fcntl**

  - **master**

    將 file 寫入 (write) 到 master device
  
  - **slave**

    將 slave device 的資料讀取 (read) 下來並且寫入到 file_out 檔案當中

- **mmap**

  - **master**
    
    將 file 當中的資料 mmap 到 file_address, master device 的資料 mmap 到 kernel_address, 將 file_address 複製 (memcpy) 到kernel_address

  - **slave**

    用同樣的方式將 slave device 當中的資料複製到 file_out 檔案當中

- **close**

  - close sockets

  - close file_fd

  - close dev_fd

## Kernel program

- **init**

  - **master**

    - 初始 debug filesystem

    - 初始 master device
    
    - 初始 TCP socket

      - address family 設為 AF_INET, 即使用 IPv4 協定

      - 將 master device address kbind 到 socket

      - 監聽 (klisten) 

  - **slave**

    - 初始 debug filesystem

    - 初始 slave device

     
- **fcntl**

  - **master**

    首先, user program 會用 ioctl 提出 create socket 及 accept connection from slave 的請求，kernel 則會建立一個 client socket 並且得到 slave device address，就可以 connect 並且傳資料了。在 user program 當中，master 會將 file_in 讀取並且 write 到 master device，而後 kernel 就會將 msg ksend 到 client socket。

  - **slave**

    首先將 IP address 從 user space copy 到 kernel，仿照初始 TCP socket 的方式設定 client socket，而後將它 kconnect 到 master device address。在 user program 當中 slave 呼叫 read() 到 slave device，slave device 就會從 client socket krecv msg 並且將 msg 從 kernel space copy 到 user space 的 buffer當中，再把它寫到 file_out 當中。 
    
- **mmap**

  - **master**
    
    user program 可以透過 mmap 的方式將 user space 的 file 映射到 kernel address (master device) 當中，我們一樣再用 TCP 的方式 ksend 到 slave device 當中。

  - **slave**

    同理，slave device 將 krecv 資料存到 slave device ，user program 則再把 slave device address 當中的資料用 mmap 的方式映射到 user space 當中的 file_out 當中。   


## Result

- master command

```bash
sudo ./master [input data] <fcntl | mmap>
```

- slave command

```bash
sudo ./slave [output data] <fcntl | mmap> 127.0.0.1
```

**output**

> input = ../data/file_1      
using fcntl  


- terminal output

```bash
Transmission time: 0.059900 ms, File size: 4 bytes
```

- dmesg output

```bash
[ 1400.557909] slave device ioctl
[ 1400.557911] slave device ioctl create socket
[ 1400.557923] sock_create sk= 0xffff8fc5647f1e40
[ 1400.557924] sockfd_cli = 0xffff8fc5647f1e40  socket is created
[ 1400.557986] connected to : 127.0.0.1 2325
[ 1400.557987] kfree(tmp)
[ 1400.558004] slave device ioctl
[ 1401.140370] family = 2, type = 1, protocol = 6
[ 1401.140403] ------------[ cut here ]------------
[ 1401.140415] WARNING: CPU: 7 PID: 9395 at ./include/net/sock.h:1712 inet_accept+0x12d/0x140
[ 1401.140417] Modules linked in: slave_device(OE) master_device(OE) ksocket(OE) ccm msr sd_mod btusb btrtl btbcm btintel uvcvideo bluetooth videobuf2_vmalloc videobuf2_memops videobuf2_v4l2 videobuf2_core videodev media ecdh_generic uas usb_storage joydev mousedev snd_hda_codec_hdmi snd_hda_codec_realtek snd_hda_codec_generic snd_soc_skl snd_soc_skl_ipc snd_soc_sst_ipc snd_soc_sst_dsp snd_hda_ext_core snd_soc_sst_match snd_soc_core tpm_crb snd_compress ac97_bus snd_pcm_dmaengine arc4 intel_rapl x86_pkg_temp_thermal intel_powerclamp coretemp kvm_intel kvm iwlmvm i915 nls_iso8859_1 nls_cp437 vfat fat mac80211 irqbypass squashfs crct10dif_pclmul zstd_decompress crc32_pclmul ghash_clmulni_intel xxhash pcbc loop aesni_intel iwlwifi aes_x86_64 crypto_simd glue_helper cryptd iTCO_wdt intel_cstate iTCO_vendor_support
[ 1401.140514]  wmi_bmof i2c_algo_bit cfg80211 drm_kms_helper input_leds snd_hda_intel snd_hda_codec drm psmouse snd_hda_core thinkpad_acpi snd_hwdep intel_gtt nvram snd_pcm rfkill e1000e intel_uncore agpgart snd_timer syscopyarea intel_rapl_perf mei_me tpm_tis sysfillrect snd tpm_tis_core ucsi_acpi sysimgblt processor_thermal_device tpm typec_ucsi i2c_i801 pcspkr mei fb_sys_fops intel_pch_thermal battery typec intel_soc_dts_iosf soundcore int3403_thermal ac int340x_thermal_zone wmi i2c_hid hid int3400_thermal evdev acpi_thermal_rel mac_hid pcc_cpufreq vmw_vmci sg scsi_mod crypto_user ip_tables x_tables ext4 crc32c_generic crc16 mbcache jbd2 fscrypto serio_raw atkbd libps2 crc32c_intel xhci_pci xhci_hcd i8042 serio [last unloaded: slave_device]
[ 1401.140614] CPU: 7 PID: 9395 Comm: master Tainted: G        W  OE   4.14.111 #7
[ 1401.140617] Hardware name: LENOVO 20KHCTO1WW/20KHCTO1WW, BIOS N23ET59W (1.34 ) 11/08/2018
[ 1401.140620] task: ffff8fc5ce1c9e80 task.stack: ffff9a6501ac8000
[ 1401.140625] RIP: 0010:inet_accept+0x12d/0x140
[ 1401.140628] RSP: 0018:ffff9a6501acbe30 EFLAGS: 00010286
[ 1401.140632] RAX: 0000000000000001 RBX: ffff8fc59e0d7000 RCX: 0000000000000000
[ 1401.140634] RDX: 000000000000018a RSI: 0000000000000200 RDI: 00000000ffffffff
[ 1401.140637] RBP: ffff8fc50ccb98c0 R08: 0000000000000001 R09: ffffffff95455cb4
[ 1401.140639] R10: 0000000000000000 R11: 0000000000000001 R12: ffffffffc11114c8
[ 1401.140642] R13: 0000000012345677 R14: 00007ffdb76b79f0 R15: 0000000000000000
[ 1401.140645] FS:  00007ff5dd430500(0000) GS:ffff8fc5e25c0000(0000) knlGS:0000000000000000
[ 1401.140648] CS:  0010 DS: 0000 ES: 0000 CR0: 0000000080050033
[ 1401.140651] CR2: 00007ffdb77a3080 CR3: 000000048e808002 CR4: 00000000003606e0
[ 1401.140653] Call Trace:
[ 1401.140668]  kaccept+0x8c/0xd0 [ksocket]
[ 1401.140676]  master_ioctl+0x15c/0x2a0 [master_device]
[ 1401.140682]  do_vfs_ioctl+0x90/0x5e0
[ 1401.140691]  ? __audit_syscall_entry+0xa9/0xf0
[ 1401.140697]  ? syscall_trace_enter+0x1c2/0x2c0
[ 1401.140702]  SyS_ioctl+0x74/0x80
[ 1401.140708]  do_syscall_64+0x74/0x170
[ 1401.140716]  entry_SYSCALL_64_after_hwframe+0x3d/0xa2
[ 1401.140720] RIP: 0033:0x7ff5dd35bcbb
[ 1401.140722] RSP: 002b:00007ffdb76b7a88 EFLAGS: 00000206 ORIG_RAX: 0000000000000010
[ 1401.140727] RAX: ffffffffffffffda RBX: 0000000000000000 RCX: 00007ff5dd35bcbb
[ 1401.140729] RDX: 00007ffdb76b79f0 RSI: 0000000012345677 RDI: 0000000000000003
[ 1401.140731] RBP: 00007ffdb76b8b70 R08: 00007ff5dd42abc0 R09: 00007ff5dd42abc0
[ 1401.140733] R10: 00007ff5dd498540 R11: 0000000000000206 R12: 00000000004010e0
[ 1401.140735] R13: 00007ffdb76b8c50 R14: 0000000000000000 R15: 0000000000000000
[ 1401.140740] Code: 4a 88 ab 00 f7 d2 44 21 e2 65 8b 0d 9e 6b b8 6a 44 23 20 09 ca 4a 8d 04 a0 3b 50 40 74 03 89 50 40 e8 68 4a a5 ff e9 21 ff ff ff <0f> 0b e9 37 ff ff ff 0f 0b e9 25 ff ff ff 0f 1f 44 00 00 0f 1f 
[ 1401.140821] ---[ end trace 504335c7ce7455aa ]---
[ 1401.140825] aceept sockfd_cli = 0xffff8fc50ccb98c0
[ 1401.140830] got connected from : 127.0.0.1 57486
[ 1401.140921] master_IOCTL_MMAP ioctl_param = 32 offset = 0  nsend = 32 GEFEWFEWFGREGGEWG
               EREWRERREWRE
               

[ 1401.140926] master: 49858318
[ 1401.140928] master closed
[ 1401.140970] memcpy
[ 1401.140974] slave_IOCTL_MMAP offset =32
[ 1401.141111] slave device ioctl
[ 1401.141116] 0
[ 1401.141119] slave device ioctl
[ 1401.141124] slave: 49858318


```

 

## Comparison

### file1

**fcntl**

| master                                              | slave                                              |
| --------------------------------------------------- | -------------------------------------------------- |
| Transmission time: 48.788300 ms, File size: 4 bytes | Transmission time: 0.152500 ms, File size: 4 bytes |



**mmap**

| master                                              | slave                                              |
| --------------------------------------------------- | -------------------------------------------------- |
| Transmission time: 30.723600 ms, File size: 4 bytes | Transmission time: 0.071100 ms, File size: 4 bytes |



### file2

**fcntl**

| master                                                | slave                                                |
| ----------------------------------------------------- | ---------------------------------------------------- |
| Transmission time: 52.311000 ms, File size: 577 bytes | Transmission time: 0.292400 ms, File size: 577 bytes |



**mmap**

| master                                                | slave                                                |
| ----------------------------------------------------- | ---------------------------------------------------- |
| Transmission time: 36.138700 ms, File size: 577 bytes | Transmission time: 0.200000 ms, File size: 577 bytes |



### file3

**fcntl**

| master                                                 | slave                                                 |
| ------------------------------------------------------ | ----------------------------------------------------- |
| Transmission time: 44.779200 ms, File size: 9695 bytes | Transmission time: 0.424700 ms, File size: 9695 bytes |



**mmap**

| master                                                 | slave                                                 |
| ------------------------------------------------------ | ----------------------------------------------------- |
| Transmission time: 37.499400 ms, File size: 9695 bytes | Transmission time: 0.275500 ms, File size: 9695 bytes |



### file4

**fcntl**

| master                                                     | slave                                                    |
| ---------------------------------------------------------- | -------------------------------------------------------- |
| Transmission time: 939.505900 ms, File size: 1502860 bytes | Transmission time: 5.836000 ms, File size: 1502860 bytes |



**mmap**

| master                                                    | slave                                                    |
| --------------------------------------------------------- | -------------------------------------------------------- |
| Transmission time: 34.412400 ms, File size: 1502860 bytes | Transmission time: 3.771600 ms, File size: 1502860 bytes |



使用傳統方式對檔案進行操作時，若需要對檔案內容進行頻繁的讀寫動作，往往會需要花費大量的 I/O 讀寫時間，但是透過 mmap 的方式，以 PAGE_SIZE 作為對映的單位，則能有效提高讀寫效率，進而降低傳輸所需要的時間。



## Work list

- 涂世昱：功能驗證
- 林宸慶：程式除錯
- 王啟時：撰寫出版程式碼
