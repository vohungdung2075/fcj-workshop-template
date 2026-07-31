---
title: "Blog 1"
date: 2026-07-30
weight: 1
chapter: false
pre: " <b> 3.1. </b> "
---

# Tại Sao Nhóm Mình Không Dùng Amazon Bedrock Knowledge Bases Dù Đang Làm Chatbot RAG?

Chào mọi người,

Trong quá trình làm dự án chatbot hỏi đáp tài liệu (RAG), nhóm mình sử dụng Amazon Bedrock để gọi Claude trả lời câu hỏi dựa trên các tài liệu người dùng upload. Khi tìm hiểu cách xây dựng RAG trên AWS, mình thấy hầu như tài liệu nào cũng nhắc đến **Knowledge Bases for Amazon Bedrock**.

Điều này cũng dễ hiểu. Knowledge Bases gần như làm thay toàn bộ pipeline RAG:
- Đọc tài liệu
- Chunk (chia nhỏ văn bản)
- Tạo embedding
- Lưu vector
- Retrieval (truy xuất dữ liệu)

Nghe rất hấp dẫn. Ban đầu mình cũng nghĩ: *"Vậy cần gì phải tự xây nữa?"* Nhưng sau khi đọc AWS Blog và đối chiếu với kiến trúc của nhóm, tụi mình lại quyết định **không sử dụng Knowledge Bases mà vẫn tự xây pipeline với FAISS**.

---

### Knowledge Bases Giúp Làm Gì?

AWS mô tả Knowledge Bases là một dịch vụ **Fully Managed RAG**. Chỉ cần chỉ định nơi lưu tài liệu (ví dụ Amazon S3), Bedrock sẽ tự động:
1. Đọc tài liệu từ nguồn lưu trữ.
2. Chia nhỏ nội dung (Chunking).
3. Tạo vector embedding.
4. Lưu vào Vector Database.
5. Truy xuất context khi có câu hỏi.
6. Gửi context cho Foundation Model (LLM).

Nói cách khác, rất nhiều bước mà developer thường phải tự viết đã được AWS tự động hóa hoàn toàn.

---

### Ban Đầu Mình Định Dùng Luôn

Lúc mới đọc tài liệu mình nghĩ đây gần như là lựa chọn hoàn hảo:
- Không cần quản lý FAISS.
- Không cần viết pipeline ingest thủ công.
- Không cần quản lý Vector Database phức tạp.
- Mọi thứ đều đã có sẵn out-of-the-box.

---

### Nhưng Khi Nhìn Lại Yêu Cầu Thực Tế Của Dự Án...

Đây mới là điều khiến nhóm mình thay đổi quyết định. Chatbot của nhóm không chỉ upload file PDF chuẩn. Người dùng còn upload:
- PDF scan (ảnh quét)
- File DOCX
- Tài liệu chứa cấu trúc bảng biểu phức tạp
- Tài liệu chứa hình ảnh minh họa

Có những file phải **OCR trước**, có file phải xử lý riêng, file cần chunk theo **Heading**, file lại cần chunk theo **Section**. Nếu dùng Knowledge Bases thì phần Ingest sẽ do AWS quản lý hoàn toàn, trong khi nhóm mình lại muốn **kiểm soát toàn bộ pipeline**.

---

### Đây Là Lý Do Nhóm Mình Chọn Tự Xây Pipeline Với FAISS

Dù dùng FAISS khiến nhóm phải tự làm nhiều việc hơn, nhưng đổi lại nhóm có thể chủ động:
- **Tự OCR bằng Amazon Textract** khi gặp tài liệu dạng ảnh/scan.
- **Tự quyết định chiến lược Chunking** tối ưu cho từng loại văn bản.
- **Tự thêm Metadata** theo từng User và từng loại tài liệu.
- **Tự xử lý bài toán Multi-tenant** (phân quyền dữ liệu giữa các người dùng).
- **Tự cập nhật Vector Store** ngay lập tức khi người dùng xóa tài liệu.

Đây đều là những yêu cầu cốt lõi mà dự án của nhóm bắt buộc phải có.

---

### Điều Mình Học Được

Sau khi đọc bài viết trên AWS Blog, mình nhận ra một điều khá thú vị: **Knowledge Bases không phải là "phiên bản tốt hơn" của FAISS**. Nó chỉ là một cách tiếp cận khác để xây dựng RAG:

- Nếu muốn **triển khai nhanh, ít tốn công vận hành**, Knowledge Bases gần như là lựa chọn lý tưởng.
- Nếu muốn **kiểm soát toàn bộ pipeline Ingest và Retrieval** để tùy biến sâu, việc tự xây với FAISS vẫn có nhiều lợi thế hơn.

Theo mình, không có lựa chọn nào đúng tuyệt đối. Quan trọng là kiến trúc nào **phù hợp nhất với yêu cầu của dự án**.

---

### Kết Luận

Ban đầu mình nghĩ: *AWS đã có Knowledge Bases thì chắc không cần tự xây RAG nữa*. Nhưng sau khi tìm hiểu kỹ hơn, mình mới nhận ra Knowledge Bases giải quyết rất tốt bài toán triển khai nhanh. Trong khi đó, những dự án cần tùy biến sâu về xử lý tài liệu, chunking, metadata hay retrieval vẫn có lý do chính đáng để xây dựng pipeline riêng.

Với chatbot của nhóm mình, dùng FAISS không phải vì nó "tốt hơn" Knowledge Bases, mà đơn giản là **phù hợp hơn với cách hệ thống đang được thiết kế**.

---

### LINK BLOG THAM KHẢO

- **AWS News Blog** – *Knowledge Bases now delivers fully managed RAG experience in Amazon Bedrock*:  
  [https://aws.amazon.com/vi/blogs/aws/knowledge-bases-now-delivers-fully-managed-rag-experience-in-amazon-bedrock/](https://aws.amazon.com/vi/blogs/aws/knowledge-bases-now-delivers-fully-managed-rag-experience-in-amazon-bedrock/)