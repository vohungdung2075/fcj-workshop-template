---
title: "Blog 1"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Xây Dựng RAG Phức Tạp: Khi Nào Nên Tự Làm, Khi Nào Dùng Amazon Bedrock?

Trong quá trình phân tích giải pháp xây dựng tính năng hỏi đáp tài liệu thông minh (Retrieval-Augmented Generation - RAG) cho dự án LearnSphere, việc lựa chọn giữa dịch vụ Fully Managed và tự xây dựng pipeline RAG tùy chỉnh là một quyết định kiến trúc quan trọng.

Khi tìm hiểu các kiến trúc RAG trên đám mây, các tài liệu kỹ thuật của AWS thường khuyến nghị dịch vụ **Knowledge Bases for Amazon Bedrock**. Tuy nhiên, qua quá trình nghiên cứu và đối chiếu với yêu cầu thực tế, mỗi phương án đều sở hữu những ưu điểm và sự đánh đổi (trade-offs) riêng. Bài viết này phân tích chuyên sâu câu hỏi kiến trúc: **Khi nào nên sử dụng dịch vụ Fully Managed như Amazon Bedrock, và khi nào nên tự xây dựng pipeline RAG tùy chỉnh?**

---

### Amazon Bedrock Knowledge Bases Giải Quyết Bài Toán Gì?

Theo thiết kế chuẩn của AWS, **Knowledge Bases for Amazon Bedrock** là giải pháp RAG hoàn toàn tự động (Fully Managed), giúp đơn giản hóa tối đa quy trình xử lý dữ liệu:

1. **Đọc tài liệu tự động**: Kết nối trực tiếp với nguồn lưu trữ như Amazon S3.
2. **Tự động Chunking**: Chia nhỏ nội dung văn bản theo các chiến lược mặc định hoặc tùy biến.
3. **Tự động Vectorization**: Tạo vector embedding cho các đoạn văn bản.
4. **Quản lý Vector Database**: Lưu trữ vào các cơ sở dữ liệu vector tích hợp (như Amazon OpenSearch Serverless, Pinecone, hoặc Amazon Aurora PostgreSQL).
5. **Truy xuất & Kết hợp LLM**: Tự động tìm kiếm context phù hợp và chuyển sang cho các mô hình Foundation Models (như Anthropic Claude) để trả lời.

Nói cách khác, những công đoạn lập trình hạ tầng phức tạp nhất của RAG đều được AWS đóng gói sẵn out-of-the-box.

---

### Ưu Điểm Khi Sử Dụng Amazon Bedrock Knowledge Bases

Kiến trúc Amazon Bedrock Knowledge Bases đem lại nhiều giá trị vượt trội cho các ứng dụng đám mây:

- **Triển khai cực kỳ nhanh chóng**: Lập trình viên có thể dựng một hệ thống RAG hoàn chỉnh chỉ trong vài giờ mà không cần viết hàng nghìn dòng code xử lý pipeline.
- **Giảm chi phí vận hành (Operational Overhead)**: Không cần tự dựng, bảo trì hay duy trì hạ tầng Vector Database và các worker trích xuất dữ liệu.
- **Tích hợp sẵn bảo mật AWS**: Hưởng lợi trực tiếp từ IAM Policies, mã hóa KMS và cơ chế bảo mật tiêu chuẩn doanh nghiệp của AWS Cloud.

---

### Nhưng Với Bài Toán RAG Tùy Biến Phức Tạp... Khi Nào Nên Tự Làm?

Dù Amazon Bedrock Knowledge Bases rất mạnh mẽ, việc tự xây dựng pipeline RAG tùy chỉnh vẫn là lựa chọn ưu việt trong các kịch bản sau:

1. **Xử lý tài liệu phi cấu trúc & OCR nâng cao**: Khi ứng dụng cần đọc các tài liệu phức tạp như PDF scan (ảnh quét), tài liệu chứa bảng biểu đa cột, hoặc văn bản tiếng Việt cần trích xuất OCR chuyên biệt (sử dụng Tesseract.js hoặc Amazon Textract tùy biến).
2. **Tùy biến chiến lược Chunking & Metadata**: Khi dự án yêu cầu chia nhỏ văn bản theo logic nghiệp vụ riêng (Heading, Section, hoặc theo từng bài học) và cần gắn Metadata chi tiết phục vụ phân quyền Multi-tenancy (đảm bảo người dùng chỉ truy cập đúng dữ liệu thuộc khóa học của họ).
3. **Đồng bộ dữ liệu thời gian thực**: Khi hệ thống cần cập nhật hoặc xoá dữ liệu trong Vector Store ngay lập tức tại thời điểm người dùng xoá tài liệu, thay vì chờ các chu kỳ sync định kỳ.
4. **Tối ưu hóa chi phí vận hành**: Khi triển khai các ứng dụng thực tế hoặc môi trường học tập, việc tự làm pipeline RAG kết hợp với các dịch vụ AI linh hoạt (như Groq API hoặc OpenAI API) giúp kiểm soát ngân sách chính xác theo lượng sử dụng thực tế.

---

### Góc Nhìn Nghiên Cứu & Bài Học Kiến Trúc

Tổng kết tư duy thiết kế hệ thống RAG trên AWS:

- **Chọn Amazon Bedrock Knowledge Bases khi**: Ưu tiên tốc độ ra mắt sản phẩm (Time-to-Market), giảm thiểu hạ tầng tự quản lý, và bài toán RAG nằm trong phạm vi chuẩn hóa của dịch vụ.
- **Chọn Tự Làm Pipeline RAG khi**: Bài toán đòi hỏi khả năng kiểm soát 100% dữ liệu đầu vào, tùy biến OCR/Chunking chuyên sâu, và cần tối ưu chi phí hạ tầng linh hoạt theo kịch bản vận hành.

---

### Kết Luận

Việc hiểu rõ bản chất và sự đánh đổi của từng giải pháp RAG trên AWS giúp kiến trúc sư phần mềm lựa chọn đúng công nghệ cho đúng bài toán thực tế, đảm bảo sự cân bằng giữa tốc độ phát triển, tính linh hoạt và hiệu quả chi phí.

---

### LINK BÀI VIẾT THAM KHẢO & BÀI ĐĂNG GỐC

- **Bài đăng thực tế trên Facebook Group AWS Study Group**:  
  [https://www.facebook.com/groups/awsstudygroupfcj/permalink/2226903721407921/](https://www.facebook.com/groups/awsstudygroupfcj/permalink/2226903721407921/)

![Ảnh bài đăng thực tế trên AWS Study Group](/images/blog1-facebook-post.png)

- **AWS News Blog** – *Knowledge Bases now delivers fully managed RAG experience in Amazon Bedrock*:  
  [https://aws.amazon.com/vi/blogs/aws/knowledge-bases-now-delivers-fully-managed-rag-experience-in-amazon-bedrock/](https://aws.amazon.com/vi/blogs/aws/knowledge-bases-now-delivers-fully-managed-rag-experience-in-amazon-bedrock/)