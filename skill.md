# Kỹ năng: Quy trình làm việc và quy ước GitHub

Kỹ năng này mô tả quy trình làm việc và các quy ước Git mà các AI agent nên tuân thủ khi làm việc trong repository này.

## 1. Chiến lược nhánh

- `main`: Nhánh **PROD**, chứa code ổn định đã được kiểm thử và sẵn sàng triển khai.
- `dev`: Nhánh phát triển chung, nơi merge code từ các thành viên sau khi review.

Cả hai nhánh đều được bật **Branch protection** trên GitHub: bắt buộc qua PR, ít nhất 1 approve, cấm force push và cấm xóa.

Mỗi thành viên phải tự tạo branch riêng cho từng task, không code trực tiếp trên `main` hoặc `dev`.

### Quy tắc đặt tên branch

| Tiền tố          | Mục đích                     | Tạo từ | Merge vào |
|------------------|------------------------------|--------|-----------|
| `feature/<task>` | Phát triển tính năng mới     | `dev`  | `dev`     |
| `fix/<task>`     | Sửa lỗi trong quá trình phát triển | `dev`  | `dev`     |
| `refactor/<task>`| Tái cấu trúc code            | `dev`  | `dev`     |
| `docs/<task>`    | Cập nhật tài liệu            | `dev`  | `dev`     |
| `test/<task>`    | Thêm/sửa test                | `dev`  | `dev`     |
| `chore/<task>`   | Bảo trì, cấu hình, dependencies | `dev`  | `dev`     |
| `hotfix/<lỗi>`   | Sửa lỗi khẩn cấp trên production | `main` | `main` **và** `dev` |

Quy tắc viết tên: chữ thường, tiếng Anh không dấu, nối các từ bằng dấu `-`, ngắn gọn. Có thể kèm số issue.

Ví dụ: `feature/login-api`, `fix/database-connection`, `docs/github-guide`, `feature/12-user-profile`, `hotfix/payment-timeout`.

## 2. Quy trình làm việc

### Bước 1: Lấy code mới nhất
```bash
git switch dev
git pull --ff-only origin dev
```

### Bước 2: Tạo branch riêng
```bash
git switch -c feature/<task>
```

### Bước 3: Code và commit
```bash
git status                 # kiểm tra file thay đổi
git add <file>             # hoặc git add . nếu chắc chắn không có file thừa
git diff --staged          # xem lại nội dung sẽ commit
git commit -m "feat: add login API"
```
Commit nhỏ, thường xuyên; mỗi commit tập trung vào một thay đổi.

### Bước 4: Đồng bộ với `dev` và push branch
```bash
git fetch origin
git merge origin/dev
# xử lý conflict (nếu có), sau đó chạy lại build/test
git push -u origin feature/<task>
```
Nếu có conflict: sửa file, `git add <file>`, `git commit` để hoàn tất merge, rồi **chạy lại test** trước khi push.

### Bước 5: Tạo Pull Request (PR)
- Tạo PR từ branch cá nhân vào `dev`.
- Tiêu đề PR theo commit convention, ví dụ: `feat: add login API`.
- Mô tả rõ: nội dung thay đổi, cách kiểm thử, kết quả kiểm thử, issue liên quan (`Closes #12`).
- Chờ ít nhất 1 thành viên review và approve trước khi merge.
- Khi có góp ý: sửa trên cùng branch, commit và `git push` lại — PR tự cập nhật.
- Không tự ý merge code chưa được review.
- Kiểu merge vào `dev`: **Squash and merge** (mỗi PR thành 1 commit gọn trên `dev`).

### Bước 6: Dọn dẹp sau khi PR được merge
```bash
git switch dev
git pull --ff-only origin dev
git branch -d feature/<task>   # xóa branch local
git fetch --prune              # dọn tham chiếu branch đã xóa trên remote
```
Branch trên GitHub xóa bằng nút **Delete branch** sau khi merge (hoặc bật tự động xóa trong Settings).

### Bước 7: Release lên production
1. Khi `dev` đã ổn định, quản lý repo tạo PR từ `dev` vào `main`.
2. Kiểm thử và xác nhận sẵn sàng release.
3. Merge bằng **Create a merge commit** (không squash, để `main` và `dev` không lệch lịch sử).
4. Gắn tag phiên bản theo Semantic Versioning:
```bash
git switch main
git pull --ff-only origin main
git tag -a v1.0.0 -m "release: v1.0.0"
git push origin v1.0.0
```

### Quy trình hotfix (lỗi khẩn cấp trên production)
```bash
git switch main
git pull --ff-only origin main
git switch -c hotfix/<lỗi>
# sửa lỗi, commit: git commit -m "fix: ..."
git push -u origin hotfix/<lỗi>
```
1. Tạo PR `hotfix/<lỗi>` → `main`, review và merge, gắn tag tăng bản PATCH (ví dụ `v1.0.1`).
2. Tạo tiếp PR `hotfix/<lỗi>` → `dev` (hoặc `main` → `dev`) để `dev` cũng có bản sửa lỗi.

## 3. Quy ước commit Git

Theo chuẩn [Conventional Commits](https://www.conventionalcommits.org/):
```
<type>(<scope>): <mô tả ngắn gọn>

[body – tùy chọn: giải thích vì sao thay đổi]

[footer – tùy chọn: BREAKING CHANGE: ..., Closes #12]
```
- `scope` là tùy chọn, chỉ module bị ảnh hưởng: `auth`, `api`, `ui`, `db`, ...
- Thay đổi phá vỡ tương thích: thêm `!` sau type/scope (`feat(api)!: ...`) hoặc footer `BREAKING CHANGE: ...`.

### Các loại commit

| Type | Ý nghĩa                 | Ví dụ                           |
|------|-------------------------|---------------------------------|
| `feat` | Thêm tính năng        | `feat(auth): add login API`     |
| `fix`  | Sửa lỗi                | `fix: handle null response`     |
| `docs` | Cập nhật tài liệu       | `docs: update installation guide` |
| `refactor` | Tái cấu trúc code, không đổi hành vi | `refactor: simplify auth service` |
| `test` | Thêm hoặc sửa test     | `test: add login unit tests`    |
| `style`| Format code, không đổi logic | `style: format Python code`    |
| `perf` | Cải thiện hiệu năng    | `perf: optimize database query` |
| `build`| Build system, dependencies | `build: upgrade to Python 3.12` |
| `ci`   | Cấu hình CI/CD         | `ci: add GitHub Actions test workflow` |
| `chore`| Công việc bảo trì khác | `chore: update .gitignore`      |
| `revert`| Hoàn tác commit trước  | `revert: revert "feat: add login API"` |

### Quy tắc viết
- Dùng tiếng Anh, **thể mệnh lệnh**: `add`, `fix`, `update` (không dùng `added`, `fixes`).
- Viết thường chữ đầu, **không** có dấu chấm cuối câu.
- Dòng tiêu đề tối đa ~72 ký tự.
- Mỗi commit tập trung vào một thay đổi cụ thể.
- Không dùng message chung chung như `update`, `fix bug`, `test`, `wip`.
- Không commit file `.env`, API key, password hoặc dữ liệu nhạy cảm — kiểm tra `.gitignore`. Nếu lỡ push secret, phải **đổi (rotate) key ngay**, xóa commit là không đ��

## 4. Quy tắc bắt buộc

1. Không push trực tiếp lên `main` hoặc `dev` (được ép buộc bằng Branch protection).
2. Mỗi task sử dụng một branch riêng, đặt tên đúng quy ước.
3. Pull/fetch code mới nhất trước khi làm việc; merge `origin/dev` và xử lý conflict trước khi tạo PR.
4. Chạy build/test ở local trước khi push và trước khi tạo PR.
5. Code phải được review và approve qua PR (ít nhất 1 người, không tự approve PR của mình).
6. `main` luôn giữ trạng thái ổn định; chỉ release từ `dev` sang `main` (trừ `hotfix/`).
7. Mỗi lần release phải gắn tag phiên bản.
8. Không force push lên branch đã có người khác cùng làm.
9. Xóa branch cá nhân (cả remote và local) sau khi PR được merge.

## 5. Sơ đồ quy trình

```text
                    hotfix/X ──PR──┐
                       │           ▼
main (PROD)  ──────────┴───────●───●──── (tag v1.0.0, v1.0.1)
                               ↑   │
                               │   │ PR merge ngược hotfix
                    PR release │   ▼
dev (DEVELOPMENT) ─────────────●───●────
                    ↑       ↑       ↑
                    │ PR    │ PR    │ PR
                    │       │       │
               feature/A feature/B fix/C
                 (Hà)     (Minh)   (Long)
```

Tùng và Tâm cũng tạo branch riêng theo cùng quy tắc.

**Nguyên tắc chung:**
Pull `dev` → Tạo branch → Code & commit → Merge `origin/dev` + test → Push → PR vào `dev` → Review & approve → Merge → Xóa branch → Release `dev` → `main` + tag.
