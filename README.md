# Drone Resupply Solver

Solver C++17 cho bài toán **Pickup-Delivery Problem with Drone Resupply
(PDP-DR)**. Hệ thống phối hợp xe tải và drone để phục vụ khách hàng, với mục
tiêu tối thiểu hóa thời gian hoàn thành toàn bộ hệ thống (`C_max`).

## Chức năng chính

- Mô phỏng xe tải và drone trong cùng một lịch trình.
- Hỗ trợ hai loại hoạt động:
  - `D`: drone mang hàng từ depot đến gặp truck, sau đó truck giao hàng.
  - `P` → `DL`: pickup-delivery truyền thống, `P` phải trước `DL`.
- Genetic Algorithm kết hợp Tabu Search.
- Local Search cho truck assignment, drone assignment và drone trip.
- Kiểm tra capacity, ready time, precedence, drone endurance và timeline.
- Benchmark kết quả GA + Tabu với nghiệm tham chiếu MILP.

## Mô hình mặc định

| Tham số | Giá trị |
|---|---:|
| Trucks | 2 |
| Drones | 2 |
| Truck capacity | 50 |
| Truck speed | 30 km/h |
| Drone speed | 60 km/h |
| Drone endurance | 90 phút |
| Truck service time | 3 phút |
| Resupply time | 5 phút |

Truck sử dụng Manhattan distance; drone sử dụng Euclidean distance. Depot có
ba chế độ:

```text
0 = Center
1 = Border
2 = Outside
```

## Luồng xử lý

```text
Đọc instance
    ↓
Tạo population bằng Random / Greedy / Sweep / Nearest Neighbor
    ↓
Decode chromosome và tính C_max + penalty
    ↓
Genetic Algorithm
    ↓
Tabu Search trên các nghiệm tốt
    ↓
Assignment Local Search
    ↓
Validation và in kết quả
```

Chromosome gồm:

```text
sequence      thứ tự phục vụ khách hàng
truck_assign  truck phục vụ mỗi vị trí
drone_assign  drone phục vụ, 0 = không dùng drone
break_bit     bắt đầu drone trip mới hay tiếp tục trip hiện tại
```

## Cấu trúc dự án

```text
src/
├── main_ga_tabu.cpp       Entry point chính
├── main_init.cpp          Kiểm thử population ban đầu
├── pdp_types.h            Data structures
├── pdp_reader.cpp/.h      Đọc instance, tạo distance matrix
├── pdp_init.cpp/.h        Khởi tạo population
├── pdp_fitness.cpp/.h     Decode và tính fitness
├── pdp_ga.cpp/.h          Genetic Algorithm
├── pdp_tabu.cpp/.h        Tabu Search
├── pdp_localsearch.cpp/.h Local Search
├── pdp_validation.cpp/.h  Kiểm tra nghiệm
└── pdp_cache.h            Cache kết quả đánh giá

Instances 2/               Dữ liệu đầu vào
reference/                 MILP reference và tài liệu
results/                   Kết quả benchmark
benchmark_compare.sh       Benchmark GA + Tabu vs MILP
Makefile                   Build commands
```

## Build

Yêu cầu: C++17 compiler và GNU Make.

```bash
make
```

Executable tạo ra:

```text
main_ga_tabu
```

Build thủ công:

```bash
clang++ -O2 -std=c++17 \
  src/pdp_reader.cpp src/pdp_utils.cpp src/pdp_fitness.cpp \
  src/pdp_init.cpp src/pdp_ga.cpp src/pdp_tabu.cpp \
  src/pdp_localsearch.cpp src/pdp_validation.cpp \
  src/main_ga_tabu.cpp -o main_ga_tabu
```

## Chạy solver

```bash
./main_ga_tabu "Instances 2/U_10_0.5_Num_1.txt"
```

Chọn depot:

```bash
./main_ga_tabu "Instances 2/U_10_0.5_Num_1.txt" --depot 1
```

Kết quả hiển thị gồm:

- `C_max` cuối cùng.
- Truck routes và timeline.
- Drone resupply events.
- Penalty và trạng thái khả thi.
- Kết quả validation.
- Runtime.

## Kiểm thử khởi tạo population

`main_init.cpp` dùng để kiểm tra riêng bước khởi tạo và đánh giá population:

```bash
clang++ -O2 -std=c++17 \
  src/pdp_reader.cpp src/pdp_utils.cpp src/pdp_fitness.cpp \
  src/pdp_init.cpp src/pdp_validation.cpp src/main_init.cpp \
  -o main_init

./main_init "Instances 2/U_10_0.5_Num_1.txt" 100
```

Chương trình báo cáo số nghiệm khả thi, tỷ lệ khả thi, `C_max` trung bình và
nghiệm khởi tạo tốt nhất.

## Benchmark

Trên Linux, WSL hoặc Git Bash:

```bash
bash benchmark_compare.sh
```

Kết quả được ghi vào:

```text
results/benchmark_ga_vs_milp.csv
```

Gap được tính theo công thức:

```text
Gap (%) = (GA_Cost - MILP_Optimal) / MILP_Optimal * 100
```

Script benchmark thay đổi số customer, beta, instance number và vị trí depot.

## Ghi chú

- Đây là research/prototype solver, không phải hệ thống điều phối production.
- Core solver chạy single-threaded; benchmark có thể chạy nhiều process độc lập.
- `IntegratedLocalSearch` được cài đặt như một module, trong khi entry point
  chính tập trung vào GA, Tabu Search và assignment-level local search.
- Shell benchmark cần môi trường Unix-like trên Windows.

## Tài liệu liên quan

- [`program_summary.tex`](program_summary.tex): technical summary.
- [`CACHE_IMPLEMENTATION.md`](CACHE_IMPLEMENTATION.md): thiết kế solution cache.
- [`CACHE_INTEGRATION_GUIDE.md`](CACHE_INTEGRATION_GUIDE.md): hướng dẫn tích hợp cache.
