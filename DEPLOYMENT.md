# 🚀 Deployment Report (Lab 06)
**Lớp:** Trí tuệ Nhân tạo & Agent
**Học viên:** (Bạn điền mã sinh viên & Tên vào đây)

## 1. Môi trường triển khai
- **Nền tảng Cloud:** Railway
- **Base Image Docker:** `python:3.11-slim` (Multi-stage build)
- **Tình trạng:** Thành công 100%

## 2. Public API & Frontend URL
> **Đường link Website trực tuyến:** 
> *(Hãy xóa dòng này và Dán link Railway của bạn vào đây - Ví dụ: https://history-bot.up.railway.app)*

## 3. Nhật ký kiểm tra (Testing)

### 3.1. Theo dõi tiến trình (Dashboard)
*(Chụp 1 màn hình trang Dashboard của Railway cho thấy chữ **"Success"** xanh lá cây ở lần deploy cuối cùng và dán ảnh vào dòng dưới)*:
[Dán ảnh Dashboard vào đây]

### 3.2. Giao diện thực tế chạy trên Internet
*(Khác với Localhost, bạn hãy bấm vào link Railway ở trên trình duyệt. Chụp ảnh màn hình giao diện Chatbot Tuyệt đẹp của bạn đang hoạt động trên môi trường Internet)*:
[Dán ảnh Giao diện Chatbot vào đây]

### 3.3. Test API sức khoẻ (Health-check Probe)
*(Hãy gõ vào đuôi URL trình duyệt của bạn `...railway.app/health`. Trình duyệt sẽ hiện ra 1 mã JSON có chữ "status": "ok". Chụp tấm ảnh đó và dán vào đây để chứng minh Backend chạy tốt)*:
[Dán ảnh truy cập /health vào đây]

---
**Cam kết:** Code triển khai trên máy chủ đảm bảo tiêu chuẩn 12-factor apps, được giới hạn Rate-limit bằng Token và đóng gói trọn vẹn qua Container.
