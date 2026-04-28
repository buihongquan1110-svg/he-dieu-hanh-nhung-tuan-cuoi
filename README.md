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
### Bước 2: Điều khiển luồng next, step:


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
- Gán giá trị mới: Giả sử muốn can thiệp, không cho nó chạy 3 vòng nữa mà bắt nó nhảy lên vòng cuối luôn: (gdb) set var counter = 2 , in lại để kiểm tra: (gdb) print counter -> Kết quả: GDB in ra $2 = 2. Vậy đã can thiệp thành công vào RAM của BeagleBone từ xa!



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
- Xóa breakpoint đi: (gdb) delete 1
- Bây giờ vì không còn breakpoint nào, và biến counter cũng đã bị bạn ép thành 2 (vòng lặp cuối), hãy gõ lệnh chạy nốt: (gdb) continue
- Kết quả: Chương trình sẽ in ra nốt câu "Ket thuc chuong trinh." và thoát. GDB sẽ báo [Inferior 1 (process ...) exited normally].

- Kết quả sau khi làm được
<img width="2560" height="1920" alt="image" src="https://github.com/user-attachments/assets/eb877192-a9f6-4a98-aabe-b16514480e66" />


<img width="2560" height="1920" alt="image" src="https://github.com/user-attachments/assets/7a39de7b-3728-47e7-90f7-f2aa548e5fe7" />



<img width="2560" height="1920" alt="image" src="https://github.com/user-attachments/assets/de82465b-aace-42d7-9090-f2c3050830e4" />



<img width="2560" height="1920" alt="image" src="https://github.com/user-attachments/assets/5de14892-c761-4ddd-82d4-4376ecb1943e" />


## Bài 2.3: Phân tích về Bộ nhớ
- Vì trong code đã có hàm leak_memory() đã được gọi, chúng ta có thể test luôn.
- Trên BBB (cửa sổ screen), gõ lệnh:
```bash
valgrind --leak-check=full ./bai_tap sex hien ra laf:
```
```bash
# valgrind --leak-check=full ./bai_tap
==1062== Memcheck, a memory error detector
==1062== Copyright (C) 2002-2024, and GNU GPL'd, by Julian Seward et al.
==1062== Using Valgrind-3.26.0 and LibVEX; rerun with -h for copyright info
==1062== Command: ./bai_tap
==1062==
Bat dau chuong trinh Debugging...
Dang cap phat bo nho nhung khong free...
Vong lap thu 0
Vong lap thu 1
Vong lap thu 2
Ket thuc chuong trinh.
==1062==
==1062== HEAP SUMMARY:
==1062== 	in use at exit: 400 bytes in 1 blocks
==1062==   total heap usage: 2 allocs, 1 frees, 4,496 bytes allocated
==1062==
==1062== 400 bytes in 1 blocks are definitely lost in loss record 1 of 1
==1062==	at 0x48352D4: malloc (in /usr/libexec/valgrind/vgpreload_memcheck-arm-linux.so)
==1062==
==1062== LEAK SUMMARY:
==1062==	definitely lost: 400 bytes in 1 blocks
==1062==	indirectly lost: 0 bytes in 0 blocks
==1062==  	possibly lost: 0 bytes in 0 blocks
==1062==	still reachable: 0 bytes in 0 blocks
==1062==     	suppressed: 0 bytes in 0 blocks
==1062==
==1062== For lists of detected and suppressed errors, rerun with: -s
==1062== ERROR SUMMARY: 1 errors from 1 contexts (suppressed: 0 from 0)
```

—> 400 bytes in 1 blocks are definitely lost: Valgrind tính toán cực kỳ chính xác. Trong code bạn viết malloc(100 * sizeof(int)). Một biến kiểu int chiếm 4 bytes, nhân với 100 phần tử là tròn trĩnh 400 bytes bị bỏ quên do không có hàm free().
- `"at 0x48352D4: malloc..."`: Nó chỉ ra chính xác hàm gây rò rỉ là hàm malloc. (Nếu biên dịch đủ thư viện chuẩn, đôi khi nó còn trỏ thẳng đến tên hàm leak_memory().

- Kết quả:

<img width="2560" height="1920" alt="image" src="https://github.com/user-attachments/assets/5b0d1c5b-c6cf-40c5-bb42-23a1a945cb49" />



<img width="2560" height="1920" alt="image" src="https://github.com/user-attachments/assets/e8368181-fd9f-46df-a64a-925c65a28c73" />


## Bài 2.5: Phân tích hiệu năng
```bash
# perf record ./bai_tap
Couldn't synthesize bpf events.
Bat dau chuong trinh Debugging...
Dang cap phat bo nho nhung khong free...
Vong lap thu 0
Vong lap thu 1
```


```bash
#
Ket thuc chuong trinh.[ 1613.562089] perf: interrupt took too long (2860 > 2500), lowering kernel.perf_event_max_sample_rate to 69900
[ 1613.573209] perf: interrupt took too long (89802 > 3575), lowering kernel.perf_event_max_sample_rate to 2200

[ perf record: Woken up 1 times to write data ]
[ perf record: Captured and wrote 0.007 MB perf.data (28 samples) ]
# perf report
der/--header-only option
#
#
# Total Lost Samples: 0
#
# Samples: 28  of event 'cycles'
# Event count (approx.): 4314261
#
# Overhead  Command  Shared Object    	Symbol                        	 
# ........  .......  ...................  ...................................
#
 	6.37%  bai_tap  ld-linux-armhf.so.3  [.] __GI___tunables_init
 	6.35%  bai_tap  [kernel.kallsyms]	[k] __fput
 	6.27%  bai_tap  [kernel.kallsyms]	[k] percpu_counter_add_batch
 	6.22%  bai_tap  [kernel.kallsyms]	[k] flush_cache_range
 	6.15%  bai_tap  [kernel.kallsyms]	[k] __rseq_handle_notify_resume
 	6.09%  bai_tap  ld-linux-armhf.so.3  [.] do_lookup_x
 	6.04%  bai_tap  ld-linux-armhf.so.3  [.] _dl_lookup_symbol_x
 	5.99%  bai_tap  [kernel.kallsyms]	[k] filemap_map_pages
 	5.96%  bai_tap  libc.so.6        	[.] __fstat64_time64
 	5.94%  bai_tap  [kernel.kallsyms]	[k] _raw_spin_unlock_irqrestore
 	5.91%  bai_tap  [kernel.kallsyms]	[k] lock_page_memcg
 	5.86%  bai_tap  [kernel.kallsyms]	[k] vfs_write

```
- Kết quả:

<img width="2560" height="1920" alt="image" src="https://github.com/user-attachments/assets/e023ad10-1d89-430a-a2e4-d7ce98294351" />

## Bài 2.6: Phân tích Tracing
- Chạy 2 lệnh:
```bash
strace -o strace_log.txt ./bai_tap
cat strace_log.txt
```
- Kết quả:

<img width="2560" height="1920" alt="image" src="https://github.com/user-attachments/assets/5de21283-055b-4c73-82c4-1c08c2efca75" />


## Bài 2.4: Phân tích Core Dump
- Với bài này thì phải quay lai phải bai_tap.c và uncomment dòng // cause_crash(); rồi chạy lệnh biên dịch tạo 1 file khac nữa là bai_tap_crash bằng lệnh ../output/host/bin/arm-linux-gcc -g bai_tap.c -o bai_tap_crash.
- Rồi copy file sang sdcard bằng lệnh:
```bash
sudo cp bai_tap_crash /media/nhatthanh/rootfs/root/
sudo chmod +x /media/nhatthanh/rootfs/root/bai_tap_crash
Sync
```

- rồi mở terminal BBB lên rồi chạy lệnh: ulimit -c unlimited để cho phép hệ thống chạy file sinh lỗi core dump. Rồi chạy file crash tạo đc ở trên: ./bai_tap_crash -> Màn hình sẽ in ra: Chuan bi gay loi Core Dump... sau đó văng ra lỗi Segmentation fault (core dumped).
- Kiểm tra xem file core đã xuất hiện chưa: ls -l core
- Đây là lệnh cuối cùng của đồ án. Dùng GDB để đọc file core và tìm ra kẻ gây lỗi: gdb ./bai_tap_crash core
- Khi giao diện (gdb) hiện ra, bạn gõ lệnh truy vết: (gdb) bt
- Bức ảnh chốt sổ: Lệnh bt (Backtrace) sẽ in ra luồng chạy của chương trình và chỉ đích danh lỗi xảy ra tại dòng *ptr = 42 nằm trong hàm cause_crash()


<img width="2560" height="1920" alt="image" src="https://github.com/user-attachments/assets/9b92134a-b86e-40a3-b2e6-f13f92dda60c" />


<img width="2560" height="1920" alt="image" src="https://github.com/user-attachments/assets/9883a013-154d-4dae-af66-2719e06165c2" />




### P/S: Bài 2.4 sẽ làm cuối cùng vì là theo code để chạy các bài trc thì cần bỏ dòng cause_crash, còn bài 2.4 cần dòng này.










