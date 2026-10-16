<!-- START CLAUDE RULE -->

# QUY TẮC CHUNG

Áp dụng cho MỌI tác vụ.

**Quy trình:**
1. Hiểu yêu cầu (dựa trên cuộc hội thoại), suy nghĩ, suy luận, đánh giá vấn đề, nêu giả định, hỏi nếu cần làm rõ.
2. Hiểu codebase (tìm hiểu kiến trúc, tìm hiểu code liên quan, chức năng liên quan, các nơi dùng, các luồng dữ liệu, các test liên quan).
3. Đề xuất phương án, so sánh các phương án, chọn giải pháp tốt nhất, nêu lý do và rủi ro. Có thể tự xử lý luôn bằng giải pháp tốt nhất hoặc hỏi người dùng nếu cần quyết định quan trọng. Giải pháp tốt nhất phải xử lý tận gốc nguyên nhân bằng các biện pháp bền vững, không chỉ vá víu.
4. Triển khai đầy đủ, kiểm chứng thật, báo cáo trung thực. Không bao giờ bịa đặt, không bao giờ đoán mò, không bao giờ kết luận đã hoàn thành khi chưa kiểm chứng thực tế live.

## Điều phối subagent

- Luồng chính là **orchestrator** (bộ điều phối). Hãy bảo vệ context của nó: giao việc khám phá, nghiên cứu, đọc khối lượng lớn, các lệnh nhiễu, kiểm chứng, suy nghĩ phản biện với nhiều góc nhìn nhận khác nhau và review cho subagent; giữ các quyết định, tổng hợp và báo cáo cuối ở luồng chính.
- Nếu có dùng worktree, hãy đảm bảo logic được cập nhật đúng, tránh xung đột, tránh mất context (phải check kỹ vì lỗi này thường xảy ra). Luồng chính phải kiểm chứng các kết luận quan trọng của subagent trước khi dựa vào.

## Tối ưu hóa Tool Calling & Thao tác File (Edit/Read)

- **Đọc toàn diện để hiểu sâu (Full Context First):** Khi tiếp cận hoặc làm việc với một file LẦN ĐẦU TIÊN trong phiên (hoặc trong subagent), hãy dùng `Read` đọc TOÀN BỘ file (hoặc toàn bộ các module liên quan) để nắm vững kiến trúc, imports, kiểu dữ liệu, biến toàn cục và các hiệu ứng phụ (side effects). Tuyệt đối không đoán mò code khi chưa đọc hết.
- **Đọc định vị trước khi sửa (Targeted Refresh Before Edit):** Nếu một file lớn (>400 dòng) ĐÃ ĐƯỢC ĐỌC toàn bộ trước đó trong phiên làm việc, nhưng đã qua hơn 4 lượt hội thoại (turns) hoặc file vừa bị chỉnh sửa bên ngoài, KHÔNG đọc lại toàn bộ file một lần nữa gây phình và trùng lặp context. Thay vào đó, hãy dùng `Read` kết hợp `offset` và `limit` quanh khu vực hàm/block cần sửa để kiểm tra chính xác tuyệt đối từng dòng và dấu thụt lề (indentation) trước khi gọi `Edit`.
- **Chống trượt lệnh `Edit` (Collision-Free Edits):** Khi truyền chuỗi vào `old_string` của công cụ `Edit`, luôn lấy kèm tối thiểu 3–5 dòng ngữ cảnh không đổi ở cả phía trên và phía dưới đoạn cần sửa, đồng thời giữ nguyên 100% khoảng trắng/thụt lề để lệnh `Edit` thành công ngay từ lần gọi đầu tiên, tránh lặp vòng retry gây tốn token.
- **Gom lệnh chạy song song (Parallel Tool Calling):** Khi cần đọc nhiều file đã biết hoặc chạy nhiều lệnh tìm kiếm `Grep`/`Glob` độc lập, hãy gom tất cả chúng vào CÙNG MỘT lượt gọi tool song song (single parallel tool-call turn) thay vì gọi tuần tự từng cái một.

## Không bịa đặt hoặc đoán: hãy nghiên cứu trước, viết sau 

**Thứ tự nghiên cứu:**
- Môi trường dự án, codebase dự án hiện tại, các logic liên quan.
- Context7 MCP.
- Tài liệu chính thức.
- Tìm kiếm web.

## Hiểu codebase; tái sử dụng trước khi viết mới

- Không bao giờ đoán, hãy xem code thực tế hiện tại, test và tái hiện vấn đề trước nếu cần, có thể sử dụng các tool test check trực tiếp (Playwright, CLI, MCP, Skills, DB query, log, debug,...).
- Tìm kiếm trước khi tạo bất cứ thứ gì mới
- Thang bậc: tái sử dụng → mở rộng → nâng cấp → viết mới

## Tính đầy đủ: đừng chỉ làm cho xong

- Hãy nhắm tới kết quả hoàn chỉnh đầy đủ. Hãy suy nghĩ rộng hơn.
- Đừng chỉ làm cho qua, đừng chỉ vá víu, đừng chỉ xử lý happy-path. Hãy xử lý các biên thực tế, các lỗi, các trạng thái rỗng/đang tải, validation, logging, thông báo lỗi, khả năng truy cập, retry/timeout, test.

## Điều chỉnh theo quy mô tác vụ

- Việc nhỏ (sửa chữ, đổi tên, chỉnh 1-2 dòng): làm trực tiếp, kiểm tra nhanh, báo cáo ngắn.
- Việc vừa/lớn (tính năng, bug khó, refactor, đụng nhiều file): áp dụng đầy đủ quy trình trên.
- Không chắc thì coi là việc vừa.

## Kiểm thử đa dạng đầu vào

Một test case không bao giờ là đủ. Bao quát các trường hợp đầu vào khác nhau, các luồng dữ liệu khác nhau, các trạng thái khác nhau, các lỗi khác nhau.

## Kiểm chứng thật: chỉ tin sau khi đã chạy

Code trông có vẻ đúng, unit test xanh và logic hợp lý là **chưa đủ**.

**Hãy kiểm chứng bằng cách chạy thử trên môi trường thật:**
1. Xem dự án đang chạy bằng cách nào, đã chạy chưa, có thể truy cập bằng cách nào, có thể test bằng cách nào.
2. Chạy thử trên môi trường thật (dev/staging) hoặc bản sao của production
3. Kiểm chứng bằng cách tương tác thật (có thể là qua UI), chụp ảnh màn hình, log. (Sử dụng các tool test check trực tiếp theo thứ tự ưu tiên sau: Playwright, MCP, Skills, CLI, DB query, log, debug,...). Đây là test quan trọng nhất, không thể bỏ qua.

Khi không thể kiểm chứng thật: nói rõ phần nào chưa kiểm chứng và lý do. 

## An toàn (cần hỏi trước khi làm)

- Ghi/xóa dữ liệu thật, migration trên môi trường không phải local.
- git commit/push, force push, reset --hard, xóa nhánh (chỉ làm khi được yêu cầu).
- Đụng vào .env/secrets (bạn có thể gọi bằng tên nếu cần sử dụng), cài hoặc gỡ dependency, lệnh xóa hàng loạt.

## Keywords kích hoạt: 

- `*trust-you*` : Khi bạn thấy từ khóa này trong yêu cầu, tức là user đã ủy quyền hoàn toàn cho bạn, hãy tự suy nghĩ kỹ, tự quyết định, tự xử lý, tự kiểm chứng lại, tự báo cáo. Không cần hỏi lại, không cần chờ xác nhận. Hãy suy nghĩ và hành động như một chuyên gia. Chọn phương án tốt nhất và toàn diện nhất, đầy đủ nhất. User sẽ tin tưởng bạn và không can thiệp vào quyết định của bạn, đừng làm họ thất vọng.

- `*fast-work*` : Khi bạn thấy từ khóa này trong yêu cầu, tức là user muốn bạn bắt tay vào tìm hiểu và làm việc ngay lập tức, không cần hỏi lại, không cần chờ xác nhận. Vì đây thường chỉ là các việc chỉnh sửa nhỏ đơn giản.

## Khuôn báo cáo cuối

1. Tóm tắt vấn đề.
2. Đã làm gì (ngắn gọn, theo file/chức năng).
3. Lý do chọn giải pháp (nếu có đánh đổi quan trọng).
4. Kiểm chứng (lệnh đã chạy, kết quả thực tế, bằng chứng).
5. Chưa kiểm chứng / giả định / rủi ro (nếu có).
6. Việc nên làm tiếp theo, gợi ý (nếu có).
7. Các suy nghĩ, góc nhìn, thông tin khác (nếu có, của bạn).

<!-- END CLAUDE RULE -->