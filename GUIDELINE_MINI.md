# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nguyễn Đình Bảo Phúc`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán | Không gán |
| --- | --- |
| xe con, SUV, taxi, xe bán tải | người đi bộ |
| van, minivan | xe đạp |
| xe buýt, minibus | **xe máy / mô tô** |
| xe tải, xe đầu kéo | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): Không có.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps) | Giữ continuity khi vẫn còn bằng chứng spatial/appearance. |
| Xe bị che lâu hơn ngưỡng trên | Kiểm tra frame trước và sau; chỉ tạo ID mới khi không còn bằng chứng đáng tin cậy cho track cũ. | Tránh nối nhầm qua khoảng mất dấu dài. |
| Xe rời khung hình rồi quay lại | mặc định: **track mới** | Đây là một lần xuất hiện mới trong video. |
| Hai xe cắt nhau / chồng lên nhau | Giữ ID theo lịch sử chuyển động và bbox trước/sau crossing; kiểm tra midpoint. | ID phải đi theo xe, không đi theo vị trí tạm thời. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | bbox chạm đúng rìa, không đoán phần ngoài ảnh |
| Xe bị xe khác che một phần | bbox ôm phần **nhìn thấy được** |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; không gán một điểm ảnh chưa chắc chắn |
| Xe đang đỗ, không di chuyển | Vẫn giữ bbox nếu xe còn trong cảnh; không bấm Outside chỉ vì xe đứng yên. |
| Keyframe đặt dày ở đâu | Entry/exit, đổi hướng, đổi scale, crossing và hai bên midpoint nếu interpolation bị drift. |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01`, frame 1–15, ID 3
- Tình huống: bbox gần như đứng yên theo cảnh báo validator.
- Quyết định: giữ track liên tục, không Outside.
- Lý do: xe vẫn hiển thị; cảnh báo static cần được kiểm bằng mắt.

### Ca 2
- Clip / frame / ID: `clip_01`, frame 1–7, ID 1
- Tình huống: track đi sát rìa ảnh và kết thúc sớm.
- Quyết định: giữ bbox đến frame 7, không kéo sang frame sau.
- Lý do: chỉ gán phần xe còn nhìn thấy và đặt ranh giới track đúng thời điểm.

### Ca 3
- Clip / frame / ID: `clip_01`, frame 147–159, ID 8
- Tình huống: track ngắn ở cuối clip; evaluator báo một phần track tham chiếu chưa được phủ.
- Quyết định: giữ một ID trong phần quan sát được, không tự thêm bbox ngoài vùng nhìn thấy.
- Lý do: cần xem lại entry/exit trong CVAT trước khi rework.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Validator không có lỗi format; cảnh báo static phải được kiểm bằng mắt trước khi kết luận bbox treo.
- Evaluator báo thiếu đoạn ở các track tham chiếu 2, 5 và 8; nếu rework thì sửa trong CVAT rồi export lại, không sửa MOT trực tiếp.
