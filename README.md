# Bài tập tuần cuối:  Sử dụng các công cụ gỡ lỗi và đánh giá hiệu năng cơ bản.

- File code để chạy các lệnh debug.
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

