# BÁO CÁO BÀI TẬP BÀI 1: KHÔI PHỤC COMMIT ĐÃ MẤT BẰNG GIT REFLOG

---

## 📌 1. Bối cảnh & Mục tiêu

* **Mục tiêu**: 
  * Hiểu và vận dụng cơ chế **Git Reflog** (Reference Log) để theo dõi và quản lý mọi sự thay đổi tham chiếu HEAD cục bộ trong Git repository.
  * Sử dụng thành thạo lệnh `git reflog` để truy vết mã SHA-1 Hash của commit đã bị mất.
  * Khôi phục thành công commit và mã nguồn bị xóa mất sau khi vô tình thực thi lệnh xóa mạnh `git reset --hard HEAD~1`.

* **Kịch bản thực tế**: 
  Trong quá trình phát triển dự án, lập trình viên vô tình chạy lệnh `git reset --hard HEAD~1`. Lệnh này lập tức lùi con trỏ `HEAD` về commit phía trước và xóa sạch các thay đổi trong **Working Directory** lẫn **Staging Area**. Lúc này, lệnh `git log` không còn hiển thị commit chứa tính năng quan trọng nữa. Ràng buộc của bài tập là **không được viết lại mã nguồn thủ công**, mà phải khôi phục chính xác trạng thái cũ bằng Git Reflog.

---

## 🧠 2. Cơ sở lý thuyết: Git Log vs Git Reflog

| Đặc điểm | Git Log (`git log`) | Git Reflog (`git reflog`) |
| :--- | :--- | :--- |
| **Phạm vi theo dõi** | Lịch sử commit của nhánh hiện tại (được duyệt theo danh sách liên kết từ commit mới nhất về commit gốc). | Nhật ký ghi lại **mọi chuyển động của con trỏ HEAD** trên repository cục bộ (commit, checkout, reset, merge, rebase...). |
| **Khi `git reset --hard`** | Các commit bị bỏ qua sẽ **biến mất** khỏi danh sách `git log`. | Vẫn lưu vết lịch sử vị trí cũ của con trỏ HEAD trước khi reset. |
| **Tính chất lưu trữ** | Chia sẻ và đồng bộ khi push lên Remote Server (GitHub/GitLab). | **Chỉ tồn tại ở Cục bộ (Local)** trên máy của bạn (thường được lưu trong `.git/logs/HEAD`). |
| **Thời gian tồn tại** | Vĩnh viễn (cho đến khi xóa nhánh/repo). | Mặc định lưu trữ khoảng 90 ngày (trước khi bị Git Garbage Collection - `git gc` dọn dẹp). |

---

## 🚀 3. Các bước thực hiện chi tiết & Kết quả thực nghiệm

### 🔹 Bước 1: Khởi tạo Repository & Commit ban đầu
Khởi tạo một dự án thử nghiệm mới và tạo commit đầu tiên để tạo nền tảng cho repository.

```bash
# Tạo thư mục và khởi tạo Git
git init

# Đặt cấu hình người dùng cục bộ
git config user.name "Hoc Vien"
git config user.email "hocvien@example.com"

# Tạo tệp tin khởi tạo và commit
powershell -Command "Set-Content -Path init.txt -Value 'Khoi tao du an'"
git add .
git commit -m "Khoi tao du an"
```

**Output kết quả:**
```text
[master (root-commit) 8b7c76a] Khoi tao du an
 1 file changed, 1 insertion(+)
 create mode 100644 init.txt
```

---

### 🔹 Bước 2: Tạo commit chứa tính năng quan trọng
Tạo tệp `feature.txt` chứa nội dung `Day la tinh nang quan trong` và thực hiện commit thay đổi.

```bash
# Tạo file feature.txt
powershell -Command "Set-Content -Path feature.txt -Value 'Day la tinh nang quan trong'"

# Thêm vào Staging Area và Commit
git add .
git commit -m "Them tinh nang quan trong"
```

**Output kết quả:**
```text
[master 47898c5] Them tinh nang quan trong
 1 file changed, 1 insertion(+)
 create mode 100644 feature.txt
```

**Kiểm tra lịch sử commit hiện tại bằng `git log --oneline`:**
```bash
git log --oneline
```
**Output kết quả:**
```text
47898c5 Them tinh nang quan trong
8b7c76a Khoi tao du an
```

---

### 🔹 Bước 3: Giả lập sự cố xóa mất commit (`git reset --hard`)
Giả sử lập trình viên thao tác nhầm lệnh `git reset --hard HEAD~1` để lùi về 1 commit. Lệnh `--hard` sẽ xóa bỏ toàn bộ mã nguồn `feature.txt` khỏi làm việc.

```bash
git reset --hard HEAD~1
```

**Output kết quả:**
```text
HEAD is now at 8b7c76a Khoi tao du an
```

---

### 🔹 Bước 4: Kiểm tra trạng thái sau sự cố
Tiến hành kiểm tra lại lịch sử commit bằng `git log --oneline` và kiểm tra sự tồn tại của file `feature.txt`:

```bash
git log --oneline
```
**Output kết quả:**
```text
8b7c76a Khoi tao du an
```
👉 *Nhận xét*: Commit `47898c5 Them tinh nang quan trong` đã hoàn toàn biến mất khỏi kết quả `git log`. File `feature.txt` cũng không còn tồn tại trong thư mục làm việc.

---

### 🔹 Bước 5: Truy vết commit đã mất bằng Git Reflog
Thực thi lệnh `git reflog` để xem lịch sử nhật ký tham chiếu con trỏ `HEAD`:

```bash
git reflog
```

**Output kết quả thực tế:**
```text
8b7c76a HEAD@{0}: reset: moving to HEAD~1
47898c5 HEAD@{1}: commit: Them tinh nang quan trong
8b7c76a HEAD@{2}: commit (initial): Khoi tao du an
```

🔍 **Phân tích kết quả Reflog**:
* `HEAD@{0}`: Trạng thái hiện tại sau khi chạy lệnh reset lùi về `HEAD~1` (`8b7c76a`).
* `HEAD@{1}`: Trạng thái ngay trước khi reset — đây chính là commit chứa tính năng quan trọng có mã Hash là **`47898c5`**.

---

### 🔹 Bước 6: Khôi phục lại commit đã mất
Sử dụng lệnh `git reset --hard` kết hợp với mã Hash tìm được trong Reflog (`47898c5`) để đưa con trỏ `HEAD` và Working Directory trở lại đúng trạng thái trước khi bị xóa:

```bash
git reset --hard 47898c5
```
*(Lưu ý: Có thể sử dụng cú pháp `git reset --hard HEAD@{1}` cho kết quả tương tự).*

**Output kết quả:**
```text
HEAD is now at 47898c5 Them tinh nang quan trong
```

---

### 🔹 Bước 7: Kiểm tra & Xác nhận khôi phục thành công

#### 1. Kiểm tra lại `git log --oneline`:
```bash
git log --oneline
```
**Output:**
```text
47898c5 Them tinh nang quan trong
8b7c76a Khoi tao du an
```

#### 2. Kiểm tra nội dung file `feature.txt`:
```bash
Get-Content feature.txt
```
**Output:**
```text
Day la tinh nang quan trong
```

#### 3. Kiểm tra nhật ký `git reflog` sau khi khôi phục:
```bash
git reflog
```
**Output:**
```text
47898c5 HEAD@{0}: reset: moving to 47898c5
8b7c76a HEAD@{1}: reset: moving to HEAD~1
47898c5 HEAD@{2}: commit: Them tinh nang quan trong
8b7c76a HEAD@{3}: commit (initial): Khoi tao du an
```

🎉 **Xác nhận**: Commit `47898c5 Them tinh nang quan trong` và toàn bộ tệp tin `feature.txt` đã được khôi phục nguyên vẹn 100%!

---

## 🛠️ 4. Các phương pháp khôi phục thay thế với Git Reflog

Ngoài việc dùng `git reset --hard <hash>` trực tiếp trên nhánh hiện tại, chúng ta còn có các cách xử lý an toàn hơn tùy ngữ cảnh:

1. **Khôi phục sang một nhánh mới (Khuyên dùng trong dự án lớn)**:
   Nếu không muốn ảnh hưởng trực tiếp đến nhánh hiện tại, ta có thể tạo nhánh mới từ commit trong reflog:
   ```bash
   git checkout -b recovery-branch 47898c5
   ```
2. **Cherry-pick commit lost về nhánh hiện tại**:
   ```bash
   git cherry-pick 47898c5
   ```

---

## 💡 5. Đúc kết bài học & Best Practices

1. **Git không mất dữ liệu ngay lập tức**: Khi thực hiện `git reset --hard`, Git chỉ di chuyển con trỏ nhánh và `HEAD`, dữ liệu commit thực chất vẫn nằm trong cơ sở dữ liệu đối tượng (`.git/objects/`) cho đến khi trình dọn dẹp (Garbage Collector) hoạt động.
2. **`git reflog` là cái phao cứu sinh**: Luôn nhớ đến `git reflog` khi lỡ tay thực hiện các lệnh nguy hiểm như `git reset --hard`, `git rebase`, `git branch -D` hoặc `git commit --amend`.
3. **Lưu ý hạn chế của Reflog**: Reflog chỉ ghi lại các thay đổi **đã được commit** (hoặc stashed). Các file chưa từng add hay commit mà bị `reset --hard` sẽ không thể khôi phục qua reflog.
