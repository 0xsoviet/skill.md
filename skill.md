---
title: "Kỹ năng: Git control cho AI agents"
version: "1.0"
scope: "global"
description: "Kỹ năng Git để các AI agent tự động thực hiện quy trình làm việc, review, hotfix và release trên bất kỳ repository nào được giao. Nếu repository có quy ước riêng, ưu tiên quy ước của repository đó."
tags: ["git","workflow","conventional-commits","ai-agent","skill"]
---

<!-- NOTE: openmausbot yêu cầu SKILL.md có YAML frontmatter ở đầu file. Phần header trên là frontmatter để phục vụ importer. -->

# Kỹ năng: Quy trình làm việc và quy ước GitHub

Kỹ năng này mô tả quy trình làm việc và các quy ước Git mà các AI agent nên tuân thủ khi làm việc trong các repository do người dùng chỉ định. Mục tiêu là giúp AI agent hiểu và thực hành "git control" (quản lý nhánh, commit, PR, release, hotfix) một cách nhất quán trên nhiều repository. Nếu một repository cụ thể có quy ước riêng (ví dụ file CONTRIBUTING, README, hoặc SKILL repo-level), AI agent phải ưu tiên các quy ước đó và báo lại cho người giao nhiệm vụ nếu có mâu thuẫn.

## Hướng dẫn sử dụng nhanh cho AI agent

- Phạm vi: áp dụng cho bất kỳ repository nào được giao trừ khi repository đó có quy ước khác.
- Trước khi thao tác: luôn đọc README, CONTRIBUTING, và bất kỳ file SKILL hay metadata nào trong repo để tìm quy ước đặc thù.
- Mục tiêu hành vi: không push trực tiếp lên nhánh bảo vệ; tạo branch theo quy ước; tạo PR có mô tả kiểm thử; chờ review; merge theo quy ước repository.

## 1. Chiến lược nhánh

- `main` (hoặc `master` nếu repo dùng tên đó): Nhánh PROD, chứa code ổn định đã được kiểm thử và sẵn sàng triển khai.
- `dev` (nếu repo có nhánh develop): Nhánh phát triển chung, nơi merge code từ các thành viên sau khi review. Nếu repo không có `dev`, AI agent sẽ làm việc trực tiếp với nhánh mặc định (default branch) theo quy ước của repo.

Cả hai nhánh (nếu có) nên có Branch protection: bắt buộc qua PR, ít nhất 1 approve, cấm force push và cấm xóa. Nếu repo có cấu hình khác, tuân theo cấu hình đó.

Mỗi task phải tạo branch riêng; không code trực tiếp trên `main` hoặc `dev`.

### Quy tắc đặt tên branch (khuyến nghị chung)

| Tiền tố          | Mục đích                     | Tạo từ | Merge vào |
|------------------|------------------------------|--------|-----------|
| `feature/<task>` | Phát triển tính năng mới     | `dev` hoặc default | `dev` hoặc default |
| `fix/<task>`     | Sửa lỗi trong quá trình phát triển | `dev` hoặc default | `dev` hoặc default |
| `refactor/<task>`| Tái cấu trúc code            | `dev` hoặc default | `dev` hoặc default |
| `docs/<task>`    | Cập nhật tài liệu            | `dev` hoặc default | `dev` hoặc default |
| `test/<task>`    | Thêm/sửa test                | `dev` hoặc default | `dev` hoặc default |
| `chore/<task>`   | Bảo trì, cấu hình, dependencies | `dev` hoặc default | `dev` hoặc default |
| `hotfix/<issue>` | Sửa lỗi khẩn cấp trên production | `main` | `main` **và** `dev` (nếu có) |

Quy tắc viết tên: chữ thường, tiếng Anh không dấu, nối các từ bằng dấu `-`, ngắn gọn. Có thể kèm số issue.

Ví dụ: `feature/login-api`, `fix/database-connection`, `docs/github-guide`, `feature/12-user-profile`, `hotfix/payment-timeout`.

## 2. Quy trình làm việc (giữ nguyên các bước, áp dụng chung)

### Bước 1: Lấy code mới nhất
```bash
# chuyển đến nhánh phát triển hoặc nhánh mặc định nếu repo không có dev
git switch dev 2>/dev/null || git switch <default-branch>
git pull --ff-only origin $(git symbolic-ref --short HEAD)
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

### Bước 4: Đồng bộ với nhánh base và push branch
```bash
git fetch origin
# merge nhánh base (dev hoặc default) trước khi push
git merge origin/dev 2>/dev/null || git merge origin/$(git symbolic-ref --short refs/remotes/origin/HEAD | sed 's@refs/remotes/origin/@@')
# xử lý conflict (nếu có), sau đó chạy lại build/test
git push -u origin feature/<task>
```
Nếu có conflict: sửa file, `git add <file>`, `git commit` để hoàn tất merge, rồi chạy lại test trước khi push.

### Bước 5: Tạo Pull Request (PR)
- Tạo PR từ branch cá nhân vào nhánh base (dev hoặc default).
- Tiêu đề PR theo commit convention, ví dụ: `feat: add login API`.
- Mô tả rõ: nội dung thay đổi, cách kiểm thử, kết quả kiểm thử, issue liên quan (`Closes #12`).
- Chờ ít nhất 1 reviewer (theo quy ước repo) review và approve trước khi merge.
- Khi có góp ý: sửa trên cùng branch, commit và `git push` lại — PR tự cập nhật.
- Không tự ý merge code chưa được review (trừ khi quy ước repo cho phép tự-merge).
- Kiểu merge: tuân theo quy ước repo (Squash, Merge commit, Rebase). Nếu không rõ, dùng Squash vào dev/default và tạo merge commit khi release vào main.

### Bước 6: Dọn dẹp sau khi PR được merge
```bash
git switch dev 2>/dev/null || git switch <default-branch>
git pull --ff-only origin $(git symbolic-ref --short HEAD)
git branch -d feature/<task>   # xóa branch local
git fetch --prune              # dọn tham chiếu branch đã xóa trên remote
```

### Bước 7: Release lên production
1. Khi nhánh phát triển ổn định (dev hoặc default), tạo PR vào `main` (hoặc theo quy ước repo).
2. Kiểm thử và xác nhận sẵn sàng release.
3. Merge theo quy ước (nhiều repo muốn Create a merge commit để giữ lịch sử).
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
git switch -c hotfix/<issue>
# sửa lỗi, commit: git commit -m "fix: ..."
git push -u origin hotfix/<issue>
```
1. Tạo PR `hotfix/<issue>` → `main`, review và merge, gắn tag tăng bản PATCH (ví dụ `v1.0.1`).
2. Tạo PR `hotfix/<issue>` → `dev` (hoặc merge `main` → `dev`) để `dev` cũng có bản sửa lỗi.

## 3. Quy ước commit Git (Conventional Commits)

Theo chuẩn [Conventional Commits](https://www.conventionalcommits.org/):
```
<type>(<scope>): <mô tả ngắn gọn>

[body – tùy chọn: giải thích vì sao thay đổi]

[footer – tùy chọn: BREAKING CHANGE: ..., Closes #12]
```
- `scope` là tùy chọn, chỉ module bị ảnh hưởng: `auth`, `api`, `ui`, `db`, ...
- Thay đổi phá vỡ tương thích: thêm `!` sau type/scope (`feat(api)!: ...`) hoặc footer `BREAKING CHANGE: ...`.

### Các loại commit (khuyến nghị chung)

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
- Dùng tiếng Anh cho commit title (khuyến nghị) và thể mệnh lệnh: `add`, `fix`, `update`.
- Dòng tiêu đề tối đa ~72 ký tự.
- Mỗi commit tập trung vào một thay đổi cụ thể.
- Không commit file `.env`, API key, password hoặc dữ liệu nhạy cảm — kiểm tra `.gitignore`. Nếu lỡ push secret, phải đổi (rotate) key ngay và thông báo cho chủ repo.

## 4. Quy tắc bắt buộc (đối với AI agent)

1. Không push trực tiếp lên nhánh bảo vệ (main/dev) trừ khi được phép.
2. Mỗi task sử dụng một branch riêng, đặt tên đúng quy ước.
3. Pull/fetch code mới nhất trước khi làm việc; merge nhánh base và xử lý conflict trước khi tạo PR.
4. Chạy build/test ở local (hoặc trong môi trường runner) trước khi push và trước khi tạo PR.
5. Code phải được review và approve qua PR (ít nhất 1 người) trừ khi quy ước repo cho phép khác.
6. `main` luôn giữ trạng thái ổn định; chỉ release từ nhánh phát triển sang `main` (trừ `hotfix/`).
7. Mỗi lần release phải gắn tag phiên bản.
8. Không force push lên branch đã có người khác cùng làm.
9. Xóa branch cá nhân (cả remote và local) sau khi PR được merge.

## 5. Sơ đồ quy trình (ví dụ minh họa)

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
```

**Nguyên tắc chung:**
Pull base → Tạo branch → Code & commit → Merge base + test → Push → PR vào base → Review & approve → Merge → Xóa branch → Release → Tag.

---

Nếu bạn muốn, tôi có thể:
- Đổi tên file thành `SKILL.md` (chữ in hoa) nếu openmausbot bắt buộc tên đó.
- Rút gọn nội dung thành phiên bản "quickstart" cho AI agent.
- Thêm checklist YAML bổ sung (capabilities, required tools) để importer auto-parse.
