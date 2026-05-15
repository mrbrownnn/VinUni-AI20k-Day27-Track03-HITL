# Báo Cáo Hoàn Thành Lab: Day 27 — Track 3 (HITL PR Review Agent)

**Họ và tên:** Phạm Văn Thành  
**Mã sinh viên:** 2A202600272  

## 1. Giới thiệu
Báo cáo này tổng kết lại quá trình thiết kế, triển khai và hoàn thiện hệ thống **Human-in-the-Loop (HITL) Pull-Request Review Agent** dựa trên **LangGraph**, được thiết kế để tự động hoá quy trình review code đồng thời vẫn đảm bảo sự can thiệp của con người (Senior Reviewers) đối với những PR có độ rủi ro cao.

## 2. Các hạng mục đã triển khai

Hệ thống được chia làm 5 bài tập nhỏ và tất cả đều đã được hoàn thành:

### 2.1. Exercise 1: Confidence-based Routing (`exercise_1_confidence.py`)
- **Node Analysis**: Tích hợp LLM (`get_llm().with_structured_output`) để đọc `pr_diff` và phân tích PR với cấu trúc Pydantic (`PRAnalysis`), trả về độ tự tin (Confidence score).
- **Node Routing**: Thiết lập logic chuyển hướng tự động giữa 3 nhánh:
  - `auto_approve` khi confidence $\ge 73\%$
  - `human_approval` khi confidence từ $58\%$ đến $72\%$
  - `escalate` khi confidence $< 58\%$

### 2.2. Exercise 2: Human In The Loop với `interrupt()` (`exercise_2_hitl.py`)
- Khởi tạo **MemorySaver Checkpointer** để lưu trạng thái Graph, đảm bảo Graph có thể pause và resume mà không bị mất dữ liệu.
- Cấu hình Node `human_approval` để trigger `interrupt()`, đẩy toàn bộ `payload` gồm diff, nhận xét, và mức độ tự tin để người dùng quyết định (Approve / Reject / Edit).
- Xây dựng vòng lặp ở `main` nhận bắt và khôi phục trạng thái bằng `Command(resume=...)`.

### 2.3. Exercise 3: Escalation Q&A (`exercise_3_escalation.py`)
- Tối ưu hóa LLM Prompt: Yêu cầu mô hình sinh ra 2-4 câu hỏi chi tiết nếu mức độ tự tin thấp (Low confidence).
- Gắn `interrupt()` trong Node `escalate` để thu thập câu trả lời từ người đánh giá (Reviewer).
- Tạo mới Node `synthesize`: Re-prompt LLM với `Original Diff` kèm theo tập hợp `Q&A` vừa thu thập được từ Reviewer để đưa ra bản Review cuối cùng với độ tự tin (Confidence) cao hơn.

### 4. Exercise 4: Structured SQLite Audit Trail (`exercise_4_audit.py`)
- Code hoàn chỉnh hàm `audit` sử dụng `write_audit_event`.
- Thêm cơ chế logging `AuditEntry` cho *tất cả* các Node trong LangGraph: `analyze`, `route`, `auto_approve`, `human_approval` (chặn trước và sau), `escalate` (chặn trước và sau), `synthesize`, và `commit`. 
- Cung cấp dữ liệu theo dòng thời gian chuẩn cho công cụ Audit Replay (`audit.replay`).

### 5. Exercise 5: Streamlit Approval UI (`app.py`)
- Bọc toàn bộ các node đã code vào trong **AsyncSqliteSaver Checkpointer**.
- Đưa Graph lên một giao diện **Streamlit** trực quan để Reviewer có thể tương tác.
- Triển khai **Sidebar**: Gọi SQL Command để lấy ra top 10 Sessions gần đây nhất. Cung cấp khả năng reload trực tiếp vào session.
- Thiết kế **Approval Card** & **Escalation Card**: Tạo các form tương tác, Input Question, và các nút Approve / Reject / Edit, và xử lý luồng Resume Data trả về LangGraph thông qua `app.ainvoke(Command(...))`.

## 3. Xác minh hoạt động
- **Cú pháp (Syntax Check)**: Đã kiểm tra cú pháp và hoàn thiện chuẩn hoá code. Không có hàm nào báo lỗi hay còn chứa `# TODO` logic bị bỏ sót.
- **Tuân thủ quy chuẩn**: Không thay đổi các file cấu hình và helper gốc. 
- **Bảo mật**: Cơ chế `.env` được bảo mật đúng chuẩn, không lưu trữ token cá nhân trên Github Repository.

## 4. Kết luận
Hệ thống **HITL PR Review Agent** hoàn toàn sẵn sàng cho môi trường thử nghiệm (Staging). Tính năng tự động học hỏi từ human input, kết hợp với công cụ Audit lưu vết sẽ là nền tảng bền vững để tích hợp sâu vào quy trình DevOps nội bộ trong các doanh nghiệp.
