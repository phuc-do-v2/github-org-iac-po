# Báo cáo phạm vi và kết quả Task 2 – Org Teams

Ngày cập nhật: **2026-10-01**. Người thực hiện Task 2: **Phúc**.

## Phạm vi yêu cầu gốc

| Task | Owner | Phạm vi |
| --- | --- | --- |
| Task 1 – Org Members | Hoàng | Research auth/permission; invite/add, đổi organization role và remove organization member |
| Task 2 – Org Teams | Phúc | Research team/team membership; create/update/delete team; add/remove member khỏi team; đổi role trong team; verify bằng plan/apply, state và GitHub |

Code trong repository chỉ tạo `github_team` và `github_team_membership`. Data source `github_membership` chỉ đọc trạng thái organization member để chặn user chưa active. Task 2 không mời user, đổi organization role hoặc xóa user khỏi organization.

## Bằng chứng hiện có

| Hạng mục | Mức bằng chứng | Kết quả |
| --- | --- | --- |
| Research provider và permission | Tài liệu chính thức | Đã ghi tại [provider-research.md](provider-research.md) |
| Create team + direct membership | Live plan/apply + GitHub API + state | Đạt: 2 add, team `poc-devops` ID `19806617`, owner có team role `maintainer` |
| Hội tụ sau apply | Live plan | Đạt: plan kế tiếp exit code `0`, không có thay đổi |
| Thiếu provider token | Live apply | Quan sát HTTP 401 trước mutation |
| User chưa thuộc organization | Live plan | Lookup trả 404 và plan dừng; Task 2 không tự gửi invitation |
| Guard và validation | Mocked plan tests | 12 test đạt; đây là bằng chứng logic cục bộ, không thay thế live lifecycle |

## Chưa được chứng minh live

- Update thuộc tính hoặc tên team và xác nhận resource identity được giữ.
- Delete một team disposable sau khi review destructive plan.
- Add một organization member bình thường vào team.
- Đổi team role `member` ↔ `maintainer` cho user bình thường.
- Remove direct team membership và xác nhận user vẫn còn trong organization.
- Chạy plan không đổi và đối chiếu state/GitHub sau từng flow trên.

Các case nested team, drift và import là kiểm thử mở rộng. Chúng có giá trị để đánh giá triển khai tương lai nhưng không phải bằng chứng đã hoàn thành yêu cầu gốc khi chưa chạy live.

## Điều chỉnh kết luận

Kết quả hiện tại chỉ chứng minh **khả thi bước đầu cho create team, direct membership của owner và state convergence**. Chưa đủ bằng chứng để kết luận Terraform đã quản lý trọn lifecycle Org Teams, thay thế phần lớn thao tác UI, hoặc sẵn sàng cho production.

Task 2 phụ thuộc Task 1 cung cấp một organization member bình thường ở trạng thái `active` để hoàn tất các flow add, role và remove. Việc invite, accept, đổi organization role và remove organization member vẫn thuộc Task 1.

## Việc còn lại để hoàn tất Task 2

1. Nhận một active ordinary member từ Task 1.
2. Chạy live update team; lưu plan/apply, state, GitHub observation và no-change plan.
3. Chạy live add member, đổi team role hai chiều và remove direct membership.
4. Chạy live delete trên team disposable sau khi hiển thị và review destructive plan.
5. Cập nhật [team-poc-results.md](team-poc-results.md) bằng kết quả thực tế và limitation quan sát được.

Chỉ sau các bước này mới kết luận được lifecycle Task 2 theo yêu cầu ban đầu.
