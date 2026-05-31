# 🚀 AutoITGen: Context-Aware Integration Test Generator

**AutoITGen** là một Plugin dành cho IntelliJ IDEA, ứng dụng sức mạnh của Large Language Models (LLMs) chạy cục bộ để tự động sinh mã kiểm thử tích hợp (True Integration Tests) cho các dự án Java Spring Boot Microservices.

Khác với các công cụ AI hỗ trợ lập trình thông thường (như GitHub Copilot hay ChatGPT), AutoITGen được thiết kế chuyên biệt với khả năng hiểu sâu kiến trúc dự án thông qua **Abstract Syntax Tree (AST)** và vòng lặp **Tác tử Tự phục hồi (Self-Healing Agent)**, giúp tạo ra các bài test chính xác, sẵn sàng chạy (ready-to-run) mà không làm rò rỉ mã nguồn công ty.

---

## 🎯 1. Mục đích Dự án

Việc viết Integration Test thường tiêu tốn rất nhiều thời gian của lập trình viên do phải thiết lập dữ liệu giả, cấu hình môi trường và viết các đoạn code lặp đi lặp lại (boilerplate). Dự án này ra đời nhằm:

* **Tự động hóa hoàn toàn** quá trình viết Integration Test cho tầng Controller trong Spring Boot.
* **Chuyển đổi mô hình kiểm thử:** Nâng cấp từ Slice Test (dùng `@MockBean` - độ tin cậy thấp) lên **True Integration Test** (kiểm thử xuyên suốt từ Controller xuống Database thực tế sử dụng Testcontainers).
* **Giải quyết vấn đề "Ảo giác" (Hallucination) của LLM:** Cung cấp ngữ cảnh siêu chi tiết để ngăn AI bịa ra các hàm hoặc tham số không tồn tại.
* **Bảo mật tối đa (100% Local):** Tích hợp với mô hình LLM chạy cục bộ (Ollama), đảm bảo mã nguồn doanh nghiệp không bao giờ bị gửi lên cloud.

---

## 🧠 2. Phương pháp & Kiến trúc Cốt lõi

Dự án áp dụng phương pháp tiếp cận **Hệ thống Đa tác tử (Multi-Agent System)** kết hợp với **Phân tích Cú pháp Tĩnh (Static AST Analysis)**, bao gồm các công nghệ cốt lõi:

### A. Deep PSI Scanning (Quét Ngữ cảnh Sâu)
Thay vì ném toàn bộ text của file cho AI, Plugin sử dụng Program Structure Interface (PSI) của IntelliJ để trích xuất thông tin một cách thông minh:
* **Quét luồng dữ liệu (Dependency Resolution):** Dò tìm tự động từ `Controller` -> `Service` -> `Repository` để xác định chính xác Database cần tương tác.
* **Khai thác Domain Model:** Tự động nhảy vào ruột các class `Entity` và `DTO` để lấy danh sách các trường (fields) và hàm khởi tạo (constructors). Điều này ép LLM phải khởi tạo object chính xác, triệt tiêu lỗi truyền sai tham số.

### B. Template-Driven Prompting (Ép khuôn Cấu trúc)
Thiết lập bộ "Kỷ luật thép" cho AI. Yêu cầu AI tuân thủ tuyệt đối cấu trúc của **Testcontainers (PostgreSQL)** và sử dụng **Hamcrest Matchers** cho các hàm assert. Triệt tiêu hoàn toàn các thư viện Mocking (Mockito).

### C. Post-Generation Sanitization (Lưới lọc Mã nguồn)
Áp dụng tư duy *Lập trình phòng thủ (Defensive Programming)*. Bất chấp việc AI có thể sinh sai thư viện do tàn dư dữ liệu huấn luyện, hệ thống Sanitizer (Regex) sẽ tự động gọt giũa code, ép sử dụng đúng thư viện hiện đại và chuẩn hóa cấu trúc package/class trước khi ghi file.

### D. Self-Healing Agentic Loop (Vòng lặp Tự phục hồi)
Đây là kiến trúc cao cấp nhất của dự án. 
1. Sau khi AI sinh code, Plugin sẽ nhờ IDE ghi file và gọi API `CodeSmellDetector` (Trình biên dịch của IntelliJ) để quét lỗi.
2. Nếu phát hiện lỗi (Cannot resolve symbol, Syntax error), hệ thống sẽ tự động đóng gói danh sách lỗi và gửi lại cho LLM để yêu cầu sửa chữa.
3. Vòng lặp phản hồi (Feedback loop) này diễn ra tối đa 5 lần cho đến khi file test hoàn toàn không còn lỗi biên dịch (Zero compilation errors).

---

## ⚙️ 3. Luồng Thực thi (Execution Flow)

1.  **Trigger:** User mở file Controller, nhấn chuột phải và chọn *Generate Integration Test*.
2.  **Select API:** UI hiển thị danh sách các Endpoint để user chọn lọc.
3.  **Context Building:** Hệ thống kích hoạt Deep PSI Scanning để gom dữ liệu (Route, Implementation Code, Repository, Entity Schema).
4.  **LLM Generation:** Giao tiếp bất đồng bộ (Async) với Ollama qua cổng 11434. Quá trình có timeout an toàn và hỗ trợ Kill Switch (User có thể hủy ngang bất kỳ lúc nào).
5.  **Validation & Healing:** Quét lỗi tĩnh. Nếu code có lỗi đỏ, tự động lặp lại bước 4 để sửa lỗi.
6.  **Final Polish:** Auto-import các thư viện chuẩn, format code theo style của IDE và hiển thị thành quả cho người dùng.

---

## 📊 4. Kết quả Đạt được (Kết quả Thực nghiệm)

Hệ thống đã chứng minh được tính hiệu quả vượt trội so với việc lập trình viên tự copy/paste code lên các công cụ Chatbot thông thường:

* **Tỷ lệ Biên dịch thành công (Compilation Rate):** Đạt mức cực cao nhờ sự bọc lót của hệ thống Auto-Import và Regex Sanitizer.
* **Chỉ số Pass@1 Cải thiện Đáng kể:** Code sinh ra ở lần đầu tiên (hoặc sau khi tự phục hồi) có thể nhấn chạy (Run) và pass ngay lập tức mà không cần sự can thiệp thủ công của con người (Zero-touch generation).
* **Độ chính xác Ngữ cảnh (Zero Hallucination):** Việc áp dụng Deep PSI quét Constructors của Entity đã loại bỏ hoàn toàn tình trạng AI "đoán mò" cấu trúc database.
* **Tốc độ Thực thi:** Hoạt động cực kỳ mượt mà và trơn tru với các model cỡ nhỏ và vừa (như Qwen 2.5 Coder 7B) chạy trên kiến trúc Apple Silicon.

---

## 🛠 5. Công nghệ & Công cụ (Tech Stack)

* **Ngôn ngữ:** Kotlin (Plugin Development), Java (Target Code)
* **IDE Core:** IntelliJ Platform SDK, PSI (Program Structure Interface), CodeSmellDetector
* **AI Backend:** Ollama (Local LLM Engine)
* **Model Đề xuất:** `qwen2.5-coder:7b`
* **Target Frameworks:** Spring Boot 3.x, JUnit 5, MockMvc, Testcontainers, PostgreSQL.
