# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: G-03 t-020 / Nghiem Tra My 
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

Bổ sung của nhóm (nếu có): 
- Chỉ gán xe thật đang xuất hiện trong cảnh đường, không gán hình xe trên biển quảng cáo, phản chiếu hoặc vật thể quá mờ không xác định được là xe bốn bánh.
- Không gán xe máy dù kích thước lớn hoặc đang đi sát ô tô.
- Nếu xe chỉ lộ một phần ở rìa ảnh nhưng vẫn xác định rõ là xe bốn bánh thì vẫn gán bbox cho phần nhìn thấy.

## 2. Luật ID — phần quan trọng nhất

| Tình huống | Luật của nhóm | Vì sao |
| --- | --- | --- |
| Xe bị che một phần rồi hiện lại | Giữ nguyên ID nếu bị che **dưới 25 frame** và hướng di chuyển/vị trí trước-sau vẫn liên tục. | 25 frame tương đương khoảng 2 giây ở 12.5 fps; nếu xe chỉ bị che ngắn thì vẫn là cùng một vật thể. |
| Xe bị che lâu hơn ngưỡng trên | Tạo track mới nếu sau khi che lâu hơn 25 frame không còn chắc chắn xe xuất hiện lại là xe cũ. Nếu vẫn thấy được một phần xe hoặc trajectory rất rõ thì đánh dấu ca này để review thay vì tự đổi ID. | Tránh nối nhầm hai xe giống nhau khi đoạn che quá dài. |
| Xe rời khung hình rồi quay lại | Mặc định: **track mới**. Track cũ kết thúc ở frame cuối cùng còn thấy xe; frame sau đó bấm `Outside`. | Khi xe đã ra khỏi khung, không còn bằng chứng liên tục để bảo đảm đó là cùng xe lúc quay lại. |
| Hai xe cắt nhau / chồng lên nhau | Giữ ID theo đường đi của từng xe trước và sau crossing; đặt thêm keyframe trước, trong và sau đoạn chồng; không reuse ID của xe này cho xe kia. | Lỗi ID switch thường xảy ra ở crossing, nên phải ưu tiên timeline/trajectory hơn cảm giác bbox tức thời. |

## 3. Luật bbox

| Tình huống | Luật của nhóm |
| --- | --- |
| Xe bị cắt bởi rìa ảnh | Bbox chạm đúng rìa ảnh, không đoán phần ngoài ảnh. |
| Xe bị xe khác che một phần | Bbox ôm phần **nhìn thấy được**, không bao cả phần bị che. |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | Bắt đầu track từ frame đầu tiên xác định được là xe bốn bánh; ngưỡng nhóm chọn: thấy được thân xe hoặc cụm bánh/đèn đủ để phân biệt với xe máy/người đi bộ. |
| Xe đang đỗ, không di chuyển | Vẫn giữ cùng ID nếu xe còn nhìn thấy rõ. Nếu đứng im nhiều frame, kiểm kỹ để chắc chắn không phải bbox treo sau khi xe đã rời khung. |
| Keyframe đặt dày ở đâu | Đặt dày ở đoạn xe đổi hướng, phanh, overlap/crossing, vừa xuất hiện lại sau occlusion, hoặc khi bbox nội suy bắt đầu trôi khỏi phần xe nhìn thấy. |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1
- Clip / frame / ID: `clip_01 / frame 87 / ID 5`
- Tình huống: Xe đi qua vùng khó, model ReID tách cùng một xe thành hai ID khác nhau ở frame này.
- Quyết định: Giữ nguyên ID `5` cho xe trong annotation của tôi.
- Lý do: Xe vẫn có trajectory liên tục trước và sau frame `87`; evaluator cho bản nhãn của tôi `IDSW = 0`, còn ReID bị switch ở track gold `5` tại frame `87`, nên không tạo ID mới.

### Ca 2
- Clip / frame / ID: `clip_01 / frame 88-92 / ID 5`
- Tình huống: Bbox của xe dễ bị trôi khi xe đi qua đoạn khó/che khuất; evaluator báo IoU thấp quanh frame `88`, `89`, `91`, `92`.
- Quyết định: Luật chuẩn là thêm keyframe quanh đoạn này và chỉnh bbox theo phần xe nhìn thấy, không để interpolation tự kéo quá xa.
- Lý do: Đây là lỗi geometry/interpolation, không phải lỗi ID. Xe vẫn là ID `5`, nhưng bbox cần sát hơn để tránh FP/bbox drift.

### Ca 3
- Clip / frame / ID: `clip_01 / frame 101-109 / ID 6`
- Tình huống: Track `6` có nhiều frame bbox lệch khi xe đổi hướng hoặc bị che một phần; evaluator báo các frame `101`, `103`, `108`, `109` có IoU thấp.
- Quyết định: Giữ ID `6`, nhưng đặt keyframe dày hơn trong đoạn `101-109` và chỉ khoanh phần xe nhìn thấy.
- Lý do: Nếu chỉ dựa vào interpolation xa giữa hai keyframe, bbox bị lệch khỏi xe; thêm keyframe giúp giữ hình học bbox ổn định mà không làm sai identity.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:
- **Boundary/Outside:** Sau khi chấm với gold, lỗi còn lại chủ yếu là bbox treo ở đầu/cuối track: ID `6` frame `88-100`, ID `5` frame `71-78`, ID `4` frame `51-53` và `149-151`, ID `8` frame `169-171`. Từ nay mỗi track phải được kiểm riêng frame đầu và frame cuối; nếu xe chưa xuất hiện hoặc đã rời khung thì phải bấm `Outside`, không để bbox tồn tại theo interpolation.
- **Keyframe ở đoạn drift:** Các frame `88-92` của ID `5` và `101-109` của ID `6` cho thấy cần đặt keyframe dày hơn ở đoạn đổi hướng/che khuất. Không chỉ chỉnh frame đầu-cuối; phải tua qua các frame giữa hai keyframe để phát hiện bbox trôi.
- **Occlusion/crossing:** Giữ ID nếu xe chỉ bị che ngắn dưới `25` frame và trajectory vẫn rõ. Nếu hai xe overlap, quyết định ID dựa trên đường đi trước-sau crossing, không dựa vào bbox chồng nhau tại một frame đơn lẻ.
- **Kiểm chéo:** Repo local chưa có `reports/review_partner.md`, nên vòng kiểm hiện dựa trên tự kiểm, gold evaluation và so sánh ReID trong notebook. Khi có reviewer thật, mỗi finding phải ghi rõ `frame`, `ID`, loại lỗi, cách sửa và closure.
