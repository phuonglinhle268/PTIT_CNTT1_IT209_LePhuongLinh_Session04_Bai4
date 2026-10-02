# Báo Cáo: Quản Lý Tệp Tin Bỏ Qua (.gitignore) Và Sửa Lịch Sử Commit (--amend)

## 1. Giải thích cơ chế gỡ bỏ file khỏi Cache Git (`git rm --cached`)

### 1.1. Cơ chế hoạt động của `git rm --cached`
Git quản lý trạng thái của dự án thông qua 3 vùng:
1. **Working Directory (Thư mục làm việc):** Nơi chứa các tệp tin vật lý thực tế trên ổ đĩa.
2. **Staging Area / Index (Vùng chuẩn bị):** Nơi lưu lại chỉ mục các thay đổi chuẩn bị đưa vào commit tiếp theo.
3. **Repository (Kho lưu trữ):** Nơi lưu trữ toàn bộ lịch sử các commit.

Khi một tệp tin đã vô tình được commit lên Git (như `credentials.txt`), Git đã đưa tệp tin đó vào vùng theo dõi (tracked).
- Lệnh `git rm <file>` thông thường sẽ xóa tệp tin ở cả **Staging Area** lẫn **xóa vật lý trên đĩa cứng**.
- Lệnh `git rm --cached <file>` chỉ xóa tệp tin khỏi **Staging Area / Index** (ngừng theo dõi trong Git) nhưng **giữ nguyên vẹn tệp tin vật lý trong Working Directory**. Nhờ đó, thông tin nhạy cảm/cấu hình cục bộ vẫn được bảo toàn để chạy ứng dụng.

---

## 2. Cấu hình `.gitignore` và sửa commit bằng `git commit --amend`

### 2.1. Cấu hình `.gitignore`
Sau khi gỡ file khỏi cache Git, file sẽ chuyển về trạng thái `Untracked`. Cần khai báo tên file (`credentials.txt`) vào tệp `.gitignore` để Git tự động bỏ qua, không hiển thị trong danh sách theo dõi ở các lần làm việc tiếp theo.

### 2.2. Cơ chế của `git commit --amend`
Lệnh `git commit --amend` cho phép chỉnh sửa trực tiếp commit gần nhất (`HEAD`):
- Kết hợp các thay đổi mới trong Staging Area (ở đây là thao tác xóa theo dõi `credentials.txt` và thêm tệp `.gitignore`) vào commit trước đó.
- Cho phép viết lại thông điệp commit (`commit message`) cho sạch sẽ và đúng chuẩn, thay thế hoàn toàn commit lỗi mà không sinh ra commit rác trong lịch sử.

---

## 3. Các bước thực hiện và Nhật ký minh chứng (Terminal Logs)

### Bước 1: Khởi tạo và commit nhầm file `credentials.txt`
```powershell
PS D:\DevOps - RA\Session04> cd git-ss04-bai4
PS D:\DevOps - RA\Session04\git-ss04-bai4> Set-Content -Path app.py -Value "print('Ung dung dang chay...')"
PS D:\DevOps - RA\Session04\git-ss04-bai4> Set-Content -Path credentials.txt -Value "API_KEY=secret_key_123456"
PS D:\DevOps - RA\Session04\git-ss04-bai4> git add .
PS D:\DevOps - RA\Session04\git-ss04-bai4> git commit -m "Feat: Khoi tao du an va vo tinh them credentials.txt"
[master 8860f90] Feat: Khoi tao du an va vo tinh them credentials.txt
 2 files changed, 2 insertions(+)
 create mode 100644 git-ss04-bai4/app.py
 create mode 100644 git-ss04-bai4/credentials.txt

PS D:\DevOps - RA\Session04\git-ss04-bai4> git log -n 1
commit 8860f90851645004a0f8f6e6550758e3fcc271c7 (HEAD -> master)
Author: phuonglinhle268 <plinhle26@gmail.com>
Date:   Fri Oct 2 12:37:29 2026 +0700

    Feat: Khoi tao du an va vo tinh them credentials.txt
```

---

### Bước 2: Gỡ bỏ `credentials.txt` khỏi Git cache và kiểm tra file vật lý
Sử dụng `git rm --cached credentials.txt` để ngừng theo dõi và kiểm tra file vật lý bằng `Test-Path`:
```powershell
PS D:\DevOps - RA\Session04\git-ss04-bai4> git rm --cached credentials.txt
rm 'git-ss04-bai4/credentials.txt'

PS D:\DevOps - RA\Session04\git-ss04-bai4> Test-Path credentials.txt
True

PS D:\DevOps - RA\Session04\git-ss04-bai4> git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
        deleted:    credentials.txt

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        deleted:    ../Screenshot 2026-10-02 103158.png
        deleted:    ../Screenshot 2026-10-02 104756.png

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        ../git-ss04-bai2/
        credentials.txt
```
> **Minh chứng:** `Test-Path credentials.txt` trả về `True` xác nhận file vật lý vẫn tồn tại trên ổ đĩa, trong khi Git đã xếp file vào diện chuẩn bị xóa khỏi commit (`deleted: credentials.txt`).

---

### Bước 3: Cấu hình `.gitignore` và ghi đè commit bằng `git commit --amend`
Tạo tệp `.gitignore` để bỏ qua `credentials.txt`, sau đó gộp vào commit cũ và sửa đổi thông điệp commit:
```powershell
PS D:\DevOps - RA\Session04\git-ss04-bai4> Set-Content -Path .gitignore -Value "credentials.txt"
PS D:\DevOps - RA\Session04\git-ss04-bai4> git add .gitignore
PS D:\DevOps - RA\Session04\git-ss04-bai4> git commit --amend -m "Feat: Khoi tao du an va cau hinh .gitignore"
[master 7dcfe20] Feat: Khoi tao du an va cau hinh .gitignore
 Date: Fri Oct 2 12:37:29 2026 +0700
 2 files changed, 2 insertions(+)
 create mode 100644 git-ss04-bai4/.gitignore
 create mode 100644 git-ss04-bai4/app.py
```

---

### Bước 4: Kiểm tra trạng thái và lịch sử commit sau cùng

#### 1. Kiểm tra trạng thái Git (`git status`):
File `credentials.txt` đã được `.gitignore` tự động bỏ qua hoàn toàn, không còn xuất hiện trong Staging hay Untracked:
```powershell
PS D:\DevOps - RA\Session04\git-ss04-bai4> git status
On branch master
Your branch is ahead of 'origin/master' by 1 commit.
  (use "git push" to publish your local commits)

Changes not staged for commit:
  (use "git add/rm <file>..." to update what will be committed)
  (use "git restore <file>..." to discard changes in working directory)
        deleted:    ../Screenshot 2026-10-02 103158.png
        deleted:    ../Screenshot 2026-10-02 104756.png

Untracked files:
  (use "git add <file>..." to include in what will be committed)
        ../git-ss04-bai2/

no changes added to commit (use "git add" and/or "git commit -a")
```

#### 2. Kiểm tra commit gần nhất (`git log -n 1`):
Chứng minh đã sửa đổi thông điệp commit thành công bằng `--amend`, commit chỉ quản lý `.gitignore` và `app.py`:
```powershell
PS D:\DevOps - RA\Session04\git-ss04-bai4> git log -n 1
commit 7dcfe204473fd67a570396aef823257049a093d8 (HEAD -> master)
Author: phuonglinhle268 <plinhle26@gmail.com>
Date:   Fri Oct 2 12:37:29 2026 +0700

    Feat: Khoi tao du an va cau hinh .gitignore
```
