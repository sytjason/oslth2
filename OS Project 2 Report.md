# OS Project 2 Report

組員：涂世昱，林宸慶，王啟時

## Programming design

### fcntl

使用傳統的方式讓資料從檔案讀取出來後，經由網路傳輸模組將資料送至
slave。

### mmap

使用記憶體映射的方式，將檔案的內容映射到一段記憶體中，以增加檔案的存取速度。

 

**初始化**

**master**

開啟 master device 及指定的檔案 (filex_in)

dev_fd = open("/dev/master_device", O_RDWR)

ile_fd = open (file_name, O_RDWR)

  

**slave**

dev_fd = open("/dev/slave_device", O_RDWR)

file_fd = open (file_name, O_RDWR | O_CREAT | O_TRUNC)


  

## Result

 使用傳統方式對檔案進行操作時，若需要對檔案內容進行頻繁的讀寫動作，往往會需要花費大量的 I/O 讀寫時間，但是透過 mmap 的方式，以 PAGE_SIZE 作為對映的單位，則能有效提高讀寫效率，進而降低傳輸所需要的時間。


 

## Comparison

### file1

**fcntlTransmission time: 5.836000 ms, File size: 1502860 bytes**

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



## Work list

- 涂世昱：功能驗證
- 林宸慶：程式除錯
- 王啟時：撰寫出版程式碼