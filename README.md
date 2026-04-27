# Bài tập tuần cuối:  Sử dụng các công cụ gỡ lỗi và đánh giá hiệu năng cơ bản.

- File code để chạy các lệnh debug:
```bash
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>

void cause_crash() {
    printf("Chuan bi gay loi Core Dump...\n");
    int *ptr = NULL;
    *ptr = 42; // Gây lỗi Segmentation fault
}

void leak_memory() {
    printf("Dang cap phat bo nho nhung khong free...\n");
    int *leak = (int *)malloc(100 * sizeof(int));
    // Không có lệnh free(leak);
}

int main() {
    printf("Bat dau chuong trinh Debugging...\n");

    // Truy cập file để test strace (Bài 2.6)
    int fd = open("/tmp/test_file.txt", O_CREAT | O_WRONLY, 0644);
    if (fd != -1) close(fd);

    leak_memory(); // Cho bài 2.3

    // Tạm dừng để bạn có thời gian test GDB (Bài 2.2)
    int counter = 0;
    while(counter < 3) {
        printf("Vong lap thu %d\n", counter);
        counter++;
        sleep(1);
    }

    // Bỏ comment dòng dưới để test Core Dump (Bài 2.4)
     cause_crash(); 

    printf("Ket thuc chuong trinh.\n");
    return 0;
}
```


## Bài 2.1: Cài đặt thành công gdbserver trên Target.

- Bài 2.1 và 2.2 có địa chỉ ip ảo luôn là 192.168.7.2


- Tạo file main.c, rồi biên dịch: ../output/host/bin/arm-linux-gcc -g bai_tap.c -o bai_tap
-  kéo lên xuống tìm và tích đủ các mục này:
```bash
[*] gdb
```
- Khi bạn tích vào gdb, ngay bên dưới nó sẽ hiện ra các menu con. Hãy tích vào [*] gdbserver (để làm Bài 2.1 và 2.2).
(Mẹo nhỏ: Tích luôn vào [*] full debugger để cài luôn bản GDB đầy đủ lên BBB, phòng trường hợp bạn không kết nối mạng được từ laptop xuống BBB thì vẫn gỡ lỗi trực tiếp được).
```bash
[*] strace (Dùng cho Bài 2.6)
[*] ltrace (Dùng cho Bài 2.6)
[*] valgrind (Dùng cho Bài 2.3)
Kernel ---> Linux Kernel Tools ---> Tích chọn [*] perf
```
- Kết quả của bài 2.1

<img width="2560" height="1920" alt="image" src="https://github.com/user-attachments/assets/770e6afa-8b69-4456-8d96-37a892444fbd" />

- Minh chứng: GDB trên Host xác nhận đã kết nối thành công và dừng tại điểm bắt đầu (_start) của chương trình.



