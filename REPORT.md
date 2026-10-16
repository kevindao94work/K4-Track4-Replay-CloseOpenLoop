# REPORT — Lab Day 12

> Điền **bằng lời và số liệu của chính bạn** (lấy từ màn hình và từ `evidence/*.json`). Mỗi câu 4–8 dòng là đủ. Số không có trong bằng chứng của bạn sẽ bị trừ điểm. Báo cáo công khai trên GitHub; giảng viên đối chiếu với `evidence/` và với bài của các bạn khác.

| | |
|---|---|
| Họ và tên | Đào Gia Bảo |
| MSSV (đúng bằng `--student-id` bạn đã chạy) | `2A202602793` |
| GitHub | `kevindao94work` |
| Seed chia dữ liệu / seed mô hình (xem `evidence/closedloop.json`, mục `student`) | `3463 / 9124` |
| Số seed huấn luyện bạn dùng (`--seeds`) | `5` (9124–9128) |

## Kết quả cuối

Điền từ `evidence/closedloop.json`, `openloop.json`, `gate.json` (cho mỗi mô hình học, lấy **dải qua các seed**).

| Bộ điều khiển | L2 vòng hở (m) | Va chạm vòng kín trên log (%) | Quyết định cổng |
|---|---|---|---|
| constant-velocity | 1,265 | 20,40 | BLOCK |
| IDM | 1,520 | 0,14 | baseline |
| EgoMLP | 0,771 | 16,86–38,22 | BLOCK |
| PercMLP | 0,627 | 1,18–7,35 | BLOCK |
| PercMLP+ (sau khi sửa) | 0,807 | 0,28–0,35 | RELEASE |

Va chạm vòng kín là phần trăm trên 2.883 quỹ đạo kiểm thử; dải của mô hình học là min–max qua 5 seed. Quyết định RELEASE chỉ có nghĩa PercMLP+ đạt các tiêu chí tương đối của cổng.

## Câu 1 · Dữ liệu
Bước 0 bạn phát hiện những vấn đề gì ở dữ liệu? Vì sao không dùng cột `Acc_LV`, và bạn thay bằng gì? Bao nhiêu quỹ đạo bị loại vì trùng, và vì sao chia theo quỹ đạo?

`evidence/data_audit.json` ghi 992.679 dòng và 9.993 quỹ đạo thô; loại 383 quỹ đạo trùng hệt, còn 9.610 quỹ đạo sạch.
`Acc_LV` chỉ tương quan 0,045 với `d(Speed_LV)/dt`, trong khi `Acc_FAV` tương quan 0,929 với đạo hàm tốc độ xe theo sau; gia tốc cột xe dẫn không đáng tin.
Pipeline vì thế làm mượt `Speed_LV` rồi vi phân để có `a_lead`, dùng đại lượng đó khai thác ca phanh.
Chia theo quỹ đạo giữ mọi khung và cửa sổ gần nhau cùng một tập, tránh rò rỉ giữa train/test; loại bản sao còn tránh cùng một chuyển động xuất hiện ở cả hai tập.

## Câu 2 · Phát lại
Bản refactor float32 khác bit ở bao nhiêu đoạn, vượt dung sai ở bao nhiêu? Thay đổi AEB làm lệch bao nhiêu đoạn, và vì sao lệch ở đúng những khung đó (mở một đoạn ra xem)? Phát theo thời gian thật làm lộ ra điều gì mà phát nhanh nhất che đi?

Phát lại cùng bộ điều khiển khác 0/600 đoạn; refactor float32 khác bit ở 600/600 đoạn nhưng sai khác lớn nhất chỉ 9,73×10⁻⁵ và 0 đoạn vượt dung sai `atol=rtol=10⁻³`.
AEB chỉ đổi 5/600 đoạn; ở đoạn 131, khung 82 có gap 5,1062 m, tốc độ khép 3,42 m/s và TTC 1,49 s, nên lệnh đổi từ −4,44 sang −6,00 m/s² theo ngưỡng TTC 1,5 s.
Phát nhanh nhất ghi 0 khung trễ và 0 khung rớt vì không có deadline; phát thời gian thật ghi 581 khung trễ và 575 khung rớt trong 59.437 khung, làm 365 đoạn đổi lệnh.
Các số này cho thấy phép phát nhanh kiểm tra tính toán nhưng không kiểm tra khả năng hoàn thành lệnh trước khi khung kế tiếp đến.

## Câu 3 · Kịch bản
Lưới của bạn (tốc độ cruise, giới hạn khoảng cách, số ô): bao nhiêu % ô **không ai tránh được**? Độ trễ phản ứng 0 → 0,6 s làm vùng đâm của IDM tăng bao nhiêu? Log có phủ vùng khó không? (dẫn giới hạn phanh mạnh nhất của người lái trong log, và phần lưới đòi hỏi hơn thế)

Lưới dùng tốc độ cruise 10 m/s, khoảng cách 2,76–59,31 m và tốc độ khép 0–10 m/s: 60×45 = 2.700 ô.
Biên bất khả thi chiếm 0,96% ô khi không trễ và 5,15% khi trễ 0,6 s, tăng 4,19 điểm phần trăm; diện tích va chạm IDM khớp các tỷ lệ đó và kiểm chứng có 0 ô lệch khỏi biên ở cả hai độ trễ.
Giới hạn phanh mạnh nhất suy ra từ người lái là 3,625 m/s²; 5,22% lưới cần phanh mạnh hơn mức này, còn log chỉ cho 614 ca phanh (6,39% quỹ đạo), tập trung chủ yếu ở vùng tốc độ khép thấp.
Va chạm bên trong biên là không thể tránh được theo mô hình vật lý; va chạm ngoài biên là lỗi có thể tránh của bộ điều khiển, nên hai loại không được gộp thành một tỷ lệ.

## Câu 4 · Vòng hở và vòng kín
Dán bảng L2 và tỷ lệ va chạm của bốn bộ điều khiển (kèm hạng ở mỗi cách chấm). Cặp nào đổi thứ tự? Vì sao IDM có L2 như vậy mà vẫn lái tốt? Vì sao EgoMLP có L2 tốt hơn constant-velocity mà vẫn đâm?

Theo thứ tự L2 / va chạm vòng kín, bốn bộ điều khiển đạt: constant-velocity 1,265 m / 20,40%; IDM 1,520 m / 0,14%; EgoMLP 0,771 m / 16,86–38,22%; PercMLP 0,627 m / 1,18–7,35% qua 5 seed.
Xếp hạng L2 là PercMLP, EgoMLP, constant-velocity, IDM; xếp hạng va chạm là IDM, PercMLP, constant-velocity, EgoMLP. Các cặp đảo thứ tự gồm constant-velocity–IDM, constant-velocity–EgoMLP, IDM–EgoMLP và IDM–PercMLP.
IDM có L2 lớn hơn vì L2 chấm độ giống dự báo tốc độ trên log, còn luật IDM phản ứng với gap và tốc độ khép trong vòng kín; độ giống quỹ đạo người lái không đồng nghĩa với giữ an toàn khi tự lái.
EgoMLP chỉ thấy lịch sử tốc độ xe mình, thiếu gap và tốc độ xe dẫn nên không biết lúc nào cần phanh; PercMLP có thêm hai tín hiệu đó nhưng vẫn tích lũy sai số khi tự lái.

## Câu 5 · Một lần chạy chưa đủ
EgoMLP dao động bao nhiêu giữa các seed? Khoảng tin cậy Wilson nói gì? Nếu bạn chỉ chạy đúng một seed thì có thể báo kết quả nào, và vì sao điều đó nguy hiểm?

EgoMLP có tỷ lệ va chạm từ 16,86% đến 38,22% qua các seed 9124–9128, chênh 21,37 điểm phần trăm trên cùng 2.883 quỹ đạo kiểm thử.
Seed tốt nhất 9125 va chạm 486/2.883 lần (16,86%; Wilson 95%: 15,54–18,27%); seed xấu nhất 9127 va chạm 1.102/2.883 lần (38,22%; 36,47–40,01%).
Hai khoảng Wilson không chồng lấp, nên biến thiên này không chỉ là dao động nhỏ quanh cùng một tỷ lệ.
Chọn riêng seed 9125 có thể khiến mô hình trông gần constant-velocity, còn seed 9127 cho kết quả kém hơn nhiều; một lần chạy sẽ che độ bất ổn và có thể đổi quyết định phát hành.

## Câu 6 · Tác nhân phản ứng
Dán bảng "xe sau phát lại" và "xe sau phản ứng". `T` bạn hiệu chỉnh là bao nhiêu, xe sau chân thực đến mức nào so với xe không phản ứng? Vì sao IDM bị tính "va chạm giả" khi xe sau chỉ phát lại, và điều này làm đổi kết luận về bộ điều khiển ra sao?

Hiệu chỉnh theo phân bố headway cho IDM thời khoảng cách `T=1,4 s` (Wasserstein-1 = 0,347; headway trung vị người lái 1,885 s); xe không phản ứng có W1 = 0,882.
Ở độ trễ ego 0,6 s, IDM với xe sau phát lại có va chạm phía sau 64,93% và va chạm bất kỳ 70,07%; với xe sau phản ứng, va chạm phía sau là 0% và va chạm bất kỳ 5,15% (chủ yếu là va chạm phía trước).
Trong cùng lưới, constant-velocity có va chạm phía sau 0% ở cả hai thế giới nhưng va chạm phía trước 73,37%; PercMLP giảm va chạm phía sau từ 40,59% khi phát lại xuống 0% khi xe sau phản ứng.
Xe sau phát lại giữ tốc độ log dù xe thử vừa phanh nên có thể tạo va chạm đuôi giả; kết quả phụ thuộc cách mô phỏng xe sau, và IDM hiệu chỉnh vẫn chỉ là một tác nhân phản ứng đơn giản.

## Câu 7 · Cổng phát hành và sửa
Quy tắc nào chặn PercMLP và nguyên nhân gốc là gì (dẫn số)? Bạn sửa bằng gì và vì sao nó chữa đúng nguyên nhân? Sau khi qua cổng, PercMLP+ có **tốt hơn** IDM không? Nó có nên được phát hành không, và bạn đề xuất làm gì trước khi quyết định?

PercMLP bị chặn ở R2–R6: R2 có cận trên 95% là 8,36% so với IDM 0,14% + 1 điểm; R3 là 23,4% (cận trên 30,0%) so với 2,1% + 5 điểm; R4 là 27,52% diện tích va chạm có thể tránh so với 0% + 2 điểm; R5 trải 1,18–7,35%; R6 là 32,7% so với 5,1% + 2 điểm. R1 và R7 đạt.
R4 phản ánh khoảng trống độ phủ: log không có các cảnh cut-in tổng hợp; R2/R3 phản ánh lỗi nhỏ trong bắt chước bị khuếch đại khi mô hình tự lái trên trạng thái ngoài phân bố. R5 cho thấy thêm độ nhạy theo seed.
Sửa bằng 20.000 cut-in gắn nhãn IDM và hai vòng DAgger mỗi seed: cho policy hiện tại lái trên quỹ đạo train, lấy trạng thái nó thật sự gặp, rồi gắn nhãn dự báo 3 giây của IDM tại chính trạng thái đó.
PercMLP+ đạt cả R1–R7; R2 có cận trên 0,64% so với 0,14% + 1 điểm, R4 là 0,15% so với 0% + 2 điểm, và R5 dao động 0,28–0,35%. L2 tăng từ 0,627 lên 0,807 m dù tỷ lệ va chạm giảm còn 0,28–0,35%.
Đạt cổng chỉ chứng minh không vượt các ngưỡng đã đặt: IDM vẫn có 0,14% va chạm so với mức trung bình khoảng 0,30% của PercMLP+; kết quả chưa chứng minh ưu thế hoặc đủ an toàn để triển khai.

## Câu 8 · Bài lab này không chứng minh được gì
Chọn **ít nhất ba** giới hạn quan trọng nhất đối với việc quyết định phát hành và giải thích vì sao mỗi cái làm kết luận của bạn kém chắc chắn. Điều gì cần làm tiếp để thu hẹp khoảng cách đến thế giới thật?

Ngay cả khi PercMLP+ có 0,28–0,35% va chạm log qua 5 seed và 0,15% diện tích va chạm có thể tránh trên lưới cut-in, các giới hạn mô phỏng vẫn làm kết luận phát hành kém chắc chắn.
Mô phỏng chỉ xét chuyển động dọc một chiều, nên không đo đánh lái, đổi làn, người đi bộ hay tương tác nhiều xe; vì vậy nó bỏ sót nhiều đường dẫn tai nạn thực tế.
Không có cảm biến thật, và thử nhiễu gap/tốc độ dẫn còn đơn giản; dữ liệu thực tế về trễ, mất khung, che khuất và sai số hiệu chuẩn có thể làm kết quả xấu hơn.
Xe dẫn được phát lại từ log, không né hoặc phanh để phản ứng với xe thử; mô hình vật lý cũng giả định bước 0,1 s và giới hạn actuator cố định, nên các tỷ lệ va chạm phụ thuộc các giả định này.
Tiếp theo cần thử cảm biến, nhiễu và timing; mở rộng phân bố kịch bản, tác nhân giao thông và cut-in; rồi kiểm chứng sim-to-real.
Trước phát hành cần chạy shadow có kiểm soát, HIL và đánh giá trên đường thử phù hợp, kèm safety review độc lập.
