# ViPoC — Phòng thí nghiệm PHP2Shell (tiếng Việt)

**Website:** mở `index.html` hoặc bật GitHub Pages từ nhánh `main`, thư mục `/ (root)`.  
**Mục tiêu:** giải thích bản chất, nguyên nhân gốc, pipeline điều tra, dịch ngược nhị phân, mô phỏng an toàn và khắc phục, dựa trên [KillaBoi/StealMyBitcoin-PoC](https://github.com/KillaBoi/StealMyBitcoin-PoC).  
**Tác giả nội dung Việt hóa:** Long Ngo · 2026. Nguồn PoC: KillaBoi. Không tuyên bố mã PoC gốc là sản phẩm của ViPoC.

> **Chỉ phục vụ giáo dục và kiểm thử được ủy quyền.** Trang web không gửi yêu cầu mạng, không thực thi mã khai thác, không thu thập khóa ví hay dữ liệu thực. Tệp Python được giữ lại để tham chiếu và **không được website gọi tới**.

## 1. Tóm tắt vấn đề

Hai lỗ hổng PHP2Shell công bố 21/10/2026 có thể cho phép thực thi mã không xác thực, tùy điều kiện triển khai:

| Mã định danh | Nguyên nhân gốc | Hậu quả |
| --- | --- | --- |
| CVE-2026-109618 / GHSA-w8hw-w9fx-j79w | Xử lý hết bộ nhớ gián đoạn quá trình khởi tạo cấu trúc, làm đường dọn dẹp sử dụng trạng thái không hợp lệ | Hỏng bộ nhớ, có thể RCE |
| CVE-2026-109619 / GHSA-6wmf-4rv5-p3rp | Trạng thái toàn cục giữ con trỏ sang vùng nhớ đã giải phóng giữa các yêu cầu | Use-after-free, có thể RCE |

**Đừng nhầm:** đây là lỗ hổng PHP runtime, không phải giải mã Bitcoin hay phá thuật toán mật mã.

## 2. Đọc pipeline PoC theo cách phòng thủ

1. **Điều kiện môi trường:** bản PHP, dạng worker và binary cụ thể ảnh hưởng bố cục bộ nhớ. Chuỗi phiên bản không bảo đảm giống nhau.
2. **Xử lý lỗi bộ nhớ:** đầu vào multipart có thể gây mức cấp phát bộ nhớ cao hơn kích thước dữ liệu đường truyền. Cần đánh giá đường xử lý khi thiếu bộ nhớ.
3. **Quan sát rò rỉ heap:** trạng thái stale trong worker có thể làm phản hồi chứa dữ liệu cũ. Không kết luận chỉ dựa vào HTTP 200.
4. **Quan sát rò rỉ code:** con trỏ hàm bị lộ làm giảm sự bất định của ASLR. Heap và vùng mã cần được đánh giá độc lập.
5. **Đường cleanup và callback:** một cấu trúc không hợp lệ có thể làm cuộc gọi gián tiếp lệch luồng, nếu bất biến vòng đời không được giữ.

Trang web cung cấp mô phỏng tổng hợp, không có request body, offset, địa chỉ hay lệnh khai thác.

## 3. Quy trình dịch ngược

1. Ghi lại SHA-256 của bản nhị phân PHP từ nguồn chính thức và thông tin nền tảng.
2. Đọc vòng đời session upload progress và quản lý tài nguyên trong mã nguồn PHP.
3. Dùng Ghidra, readelf, objdump để khảo sát call graph, data flow và mã dọn dẹp.
4. So sánh phiên bản trước và sau bản vá PHP để truy ra invariant được sửa.
5. Viết các ca kiểm thử bộ nhớ **vô hại** trong môi trường riêng với ASan/UBSan và dữ liệu tổng hợp.
6. Tách biệt giả thuyết, quan sát và bằng chứng; không công bố bản ghi chứa cookie/khóa bí mật.

## 4. Khắc phục

Ưu tiên cập nhật lên PHP **8.5.12**, **8.4.27**, **8.3.36**, **8.2.35** hoặc mới hơn trong cùng nhánh, hoặc bản vá được nhà phân phối hỗ trợ. PHP cũ hơn đã EOL cần nâng cấp sang nhánh được hỗ trợ. Nếu không cần, tắt tính năng `session.upload_progress.enabled` như biện pháp giảm thiểu, không thay thế cập nhật. Giới hạn truy cập tệp bí mật của worker PHP và giám sát hiện tượng OOM, crash, cookie bất thường và tiến trình con.

## 5. Cấu trúc

- `index.html`: website thuần HTML/CSS/JavaScript, Việt hóa 100%, đáp ứng điện thoại và desktop; không phụ thuộc CDN.
- `README.md`: tài liệu tiếng Việt này.
- `docs/README-goc.md`: bản README tiếng Anh gốc để kiểm chứng xuất xứ.
- `stealmybitcoin_poc.py`: tệp Python upstream, **không được trang web thực thi**.

## 6. Nguồn đối chiếu

- [Mã PoC của KillaBoi](https://github.com/KillaBoi/StealMyBitcoin-PoC)
- [Tài liệu PHP: Session Upload Progress](https://www.php.net/manual/en/session.upload-progress.php)
- [PHP2Shell — trung tâm an ninh mạng Hà Lan](https://github.com/NCSC-NL/php2shell)
- [Docker — tư vấn bảo mật PHP2Shell](https://www.docker.com/blog/security-update-on-php2shell-impact-for-docker-customers/)
- [GHSA-w8hw-w9fx-j79w](https://github.com/php/php-src/security/advisories/GHSA-w8hw-w9fx-j79w)
- [GHSA-6wmf-4rv5-p3rp](https://github.com/php/php-src/security/advisories/GHSA-6wmf-4rv5-p3rp)

**Ghi chú khoa học:** Các báo cáo chạy thành công và con số tỷ lệ thực nghiệm thuộc README gốc, không phải số liệu ViPoC tự kiểm chứng.
