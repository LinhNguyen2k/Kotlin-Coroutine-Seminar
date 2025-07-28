# Kotlin Coroutines Seminar

## I. Giới thiệu về Kotlin Coroutine

### 1.1 Coroutine là gì?
- Couroutine là một mẫu thiết kế cho cơ chế xử lý đồng thời trên Android để đơn giản hóa mã nguồn khi thực thi bất đồng bộ
- Coroutine giống như **light-weight thread** nhưng không phải là thread.
- Giống thread ở chỗ:
  - Có thể chạy song song.
  - Có thể đợi nhau và trao đổi dữ liệu.
- **Khác biệt lớn nhất**:
  - Có thể chạy hàng nghìn coroutine mà không ảnh hưởng nhiều đến performance.
- Một thread **có thể chạy nhiều coroutine**.
- Coroutine **không nhất thiết chạy trên background thread**, chúng có thể chạy trên **main thread**.

---

## II. Build First Coroutine with Kotlin

### Thêm dependencies

```kotlin
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:1.2.1'
implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.1.1'
```

### Cấu trúc một coroutine

```kotlin
GlobalScope.launch {
    delay(10000L)
    println("World,")
}
println("Hello,")
Thread.sleep(20000L)
println("Kotlin")
```

### Output

```text
Hello,     // Giả sử Hello được in ra ở giây thứ 1
World,     // In ra ở giây thứ 11
Kotlin     // In ra ở giây thứ 21
```

- GlobalScope là coroutine scope. Chúng ta không thể launch một coroutine nếu nó không có scope.
- launch khởi động một coroutine mới và không trả về kết quả cho phương thức gọi. 
- async bắt đầu một coroutine mới và cho phép bạn trả về kết quả bằng một lệnh tạm ngưng có tên là await.

---

## III. Blocking vs Non-Blocking / Normal Function vs Suspend Function

### 1. Blocking
![Blocking Function](thread_blocked.png)
- Chạy tuần tự từ trên xuống dưới.
- **Chặn thread** đang chạy.

### 2. Non-blocking
![Suspend Function](suspend_resume.png)
- Coroutine hỗ trợ mô hình **non-blocking**:
  - Không cần nhiều thread.
  - Một thread vẫn có thể chạy nhiều coroutine theo kiểu non-blocking.

### 3. Suspend Function
- Là function có thể **bị tạm dừng** (`suspend`) và **resume lại** mà không block thread.
- Cho phép coroutine tạm dừng ở giữa rồi tiếp tục khi có kết quả.
- Bạn chỉ có thể gọi các hàm suspend từ các hàm suspend khác hoặc bằng cách sử dụng một hàm tạo coroutine như launch để bắt đầu một coroutine mới.

#### Ví dụ hoạt động của `suspend`

```
FunctionA -> suspend -> FunctionB -> resume FunctionA
```

### 4. RunBlocking

- Khi muốn coroutine chạy theo kiểu **blocking**:

```kotlin
runBlocking {
    println("Hello")
    delay(5000L)
}
println("World")
```

---

## IV. Coroutine Context và Dispatcher

### 1. Coroutine Context

- Mỗi coroutine có một `CoroutineContext`.
- Là tập hợp các **element cấu hình** cho coroutine.

### 2. Các loại Element (Dispatcher)

| Dispatcher                  | Mô tả                                                                 |
|----------------------------|------------------------------------------------------------------------|
| `Dispatchers.Main`         | Chạy trên **UI Main Thread**                                          |
| `Dispatchers.IO`           | Chạy trên background thread - dùng cho **I/O, DB, Networking**        |
| `Dispatchers.Default`      | Chạy background thread - dùng cho **CPU-bound tasks**                 |
| `newSingleThreadContext()` | Tạo **thread riêng** có tên                                            |
| `newFixedThreadPoolContext(n)` | Tạo thread pool có **n thread**                         |
| `Dispatchers.Unconfined`   | Chạy **ở thread gọi coroutine**, không bị "confined" (ràng buộc)      |

### 3. Default Context

- Nếu không set context:
```kotlin
GlobalScope.launch {
    // mặc định là Dispatchers.Default + Job()
}
```

### 4. Ví dụ với `Dispatchers.Unconfined`

```kotlin
fun main() = runBlocking {
    launch(Dispatchers.Unconfined) {
        println("Unconfined      : I'm working in thread ${Thread.currentThread().name}")
        delay(1000)
        println("Unconfined      : After delay in thread ${Thread.currentThread().name}")
    }
}
```

### Output

```text
Unconfined      : I'm working in thread main
Unconfined      : After delay in thread kotlinx.coroutines.DefaultExecutor
```

> `Dispatchers.Unconfined`: ban đầu chạy trên thread gọi coroutine, sau `suspend` sẽ resume trên một thread khác (ví dụ: `DefaultExecutor`).

---
### 5. Hàm withContext
- Nó là một suspend function cho phép coroutine chạy code trong block với một context cụ thể do chúng ta quy định. 
```kotlin
GlobalScope.launch(Dispatchers.IO) {
    println("Start fetch: ${Thread.currentThread().name}")
    val user = fetchUserFromApi()
    println("After fetch: ${Thread.currentThread().name}")

    withContext(Dispatchers.Main) {
        println("On Main thread: ${Thread.currentThread().name}")
    }
}
```
### Output
```text
Start fetch: DefaultDispatcher-worker-2
After fetch: DefaultDispatcher-worker-2
On Main thread: main
```

