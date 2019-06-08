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
sudo ./slave [output data] <fcntl | mmap>
```

**output**

> input = ../data/file_1      
using fcntl  


- master output

```bash
File size =32
Before switch
Master read size = 32
Master read size = 0
Transmission time: 6949.230000 ms, File size: 4 bytes

```

- slave output

```bash
ioctl success
ret: 32
ret: 0
Transmission time: 0.163600 ms, File size: 4 bytes

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
