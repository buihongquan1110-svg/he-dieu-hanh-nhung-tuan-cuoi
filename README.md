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

## Bài 2.2: Thực hiện sử dụng gdb để điều khiển luồng cơ bản chương trình từ host -> target
### Bước 1: Thêm Breakpoint và chạy vào chương trình chính:

- Đầu tiên, chúng ta đặt điểm dừng tại hàm main và cho chương trình chạy đến đó: (gdb) break main -> Kết quả: GDB báo đã tạo Breakpoint số 1 tại hàm main.
- (gdb) continue -> Kết quả: Chương trình chạy và dừng lại tại dòng printf("Bat dau chuong trinh Debugging...\n");
### Bước 2: Dieu khien luong next, step:


- (gdb) next -> Kết quả: Lệnh này (n viết tắt) sẽ thực thi hàm printf và nhảy xuống dòng int fd = open...
- gõ tiếp next (hoặc chỉ cần nhấn phím Enter để lặp lại lệnh trước đó) khoảng 2-3 lần cho đến khi GDB báo đang đứng ở dòng: leak_memory(); // Cho bài 2.3
- Bây giờ thay vì gõ next, hãy gõ step để chui vào bên trong hàm đó:
```bash
(gdb) step -> Kết quả: Lệnh step (s viết tắt) sẽ đưa chui vào bên trong hàm leak_memory(), đứng tại dòng printf("Dang cap phat bo nho... \n");
```
- Gõ next vài lần để chạy hết hàm leak_memory() và GDB sẽ tự động nhảy ngược trở lại hàm main.

### Bước 3: In và Gán giá trị biến


- Hãy chạy lệnh next liên tục cho đến khi bạn chui vào bên trong vòng lặp while(counter < 3) và đứng ở dòng printf("Vong lap thu %d\n", counter); (hoặc dòng counter++;).
- Tại đây, biến counter đã được khởi tạo, chúng ta bắt đầu test:
- In giá trị hiện tại:
```bash
(gdb) print counter -> Kết quả: GDB in ra $1 = 0 (vì vòng lặp mới chạy lần đầu).
```
- Gán giá trị mới: Giả sử muốn can thiệp, không cho nó chạy 3 vòng nữa mà bắt nó nhảy lên vòng cuối luôn: (gdb) set var counter = 2 , in laij dder ktra: (gdb) print counter -> Kết quả: GDB in ra $2 = 2. Bạn đã can thiệp thành công vào RAM của BeagleBone từ xa!



### Bước 4: Xem giá trị thanh ghi

- có thể xem trạng thái CPU của BeagleBone Black ngay lúc này:
```bash
(gdb) info registers -> Kết quả: Một loạt các thanh ghi (r0, r1, r2, sp, lr, pc...) của chip ARM sẽ được in ra.
```

### Bước 5: Xem và Xóa Breakpoint


- Kiểm tra lại xem mình đang có bao nhiêu điểm dừng:
```bash
(gdb) info breakpoints -> Kết quả: Liệt kê danh sách các breakpoint, bạn sẽ thấy cái số 1 ở hàm main.
```
- Xoa breakpoint ddi: (gdb) delete 1
- Bây giờ vì không còn breakpoint nào, và biến counter cũng đã bị bạn ép thành 2 (vòng lặp cuối), hãy gõ lệnh chạy nốt: (gdb) continue
- Kết quả: Chương trình sẽ in ra nốt câu "Ket thuc chuong trinh." và thoát. GDB sẽ báo [Inferior 1 (process ...) exited normally].

- Kết quả sau khi làm được
<img width="2560" height="1920" alt="image" src="https://github.com/user-attachments/assets/eb877192-a9f6-4a98-aabe-b16514480e66" />


<img width="2560" height="1920" alt="image" src="https://github.com/user-attachments/assets/7a39de7b-3728-47e7-90f7-f2aa548e5fe7" />



<img width="2560" height="1920" alt="image" src="https://github.com/user-attachments/assets/de82465b-aace-42d7-9090-f2c3050830e4" />



<img width="2560" height="1920" alt="image" src="https://github.com/user-attachments/assets/5de14892-c761-4ddd-82d4-4376ecb1943e" />







