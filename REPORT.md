# REPORT — Lab Day 12

| Thông tin | Giá trị |
|---|---|
| Họ và tên | Nguyễn Văn Thân |
| MSSV (đúng bằng `--student-id` đã chạy) | 2A202602859 |
| GitHub | [meth04](https://github.com/meth04) |
| Repo bài nộp | [K4-Track4-DAY12-NguyenVanThan-2A202602859](https://github.com/meth04/K4-Track4-DAY12-NguyenVanThan-2A202602859) |
| Seed chia dữ liệu / seed mô hình | 246 / 4107–4111 |
| Số seed huấn luyện (`--seeds`) | 5 |

## Kết quả cuối

L2 là trung bình ± độ lệch chuẩn qua seed; cột va chạm ghi dải qua seed và Wilson 95% của seed xấu nhất. Hai bộ luật chạy xác định một lần.
Nguồn: [openloop.json](evidence/openloop.json) (`summary`), [closedloop.json](evidence/closedloop.json) (`logs`), [gate.json](evidence/gate.json) (`decisions`).

| Bộ điều khiển | L2 vòng hở (m) | Dải va chạm vòng kín (%) | Seed xấu nhất (%); CI95 | Cổng |
|---|---|---|---|---|
| const-vel | 1.252 ± 0.000 | 19.94–19.94 | 19.94; [18.53, 21.44] | BLOCK |
| IDM | 1.470 ± 0.000 | 0.07–0.07 | 0.07; [0.02, 0.25] | baseline |
| EgoMLP | 0.763 ± 0.002 | 9.30–23.27 | 23.27; [21.77, 24.85] | BLOCK |
| PercMLP | 0.615 ± 0.002 | 0.49–1.94 | 1.94; [1.50, 2.51] | BLOCK |
| PercMLP+ | 0.797 ± 0.010 | 0.21–0.28 | 0.28; [0.14, 0.55] | RELEASE |

EgoMLP có dải L2 0.761–0.766 m; PercMLP 0.612–0.618 m.
Pipeline gốc chỉ lưu trung bình và độ lệch chuẩn L2 của PercMLP+; dải va chạm của mô hình này vẫn được lưu đầy đủ cho cả 5 seed.

## Câu 1 · Dữ liệu

Nguồn: [data_audit.json](evidence/data_audit.json). Dữ liệu có 992.679 dòng, 9.993 quỹ đạo thô; độ dài 70–109 bước, trung vị 106, tần số 10 Hz.
Ba vấn đề chính là gia tốc xe dẫn không đáng tin, quỹ đạo trùng và nguy cơ rò rỉ do các khung liền nhau tương quan mạnh.
`Acc_LV` chỉ tương quan 0.045 với đạo hàm `Speed_LV`, trong khi `Acc_FAV` tương quan 0.929; tôi dùng `data.derive_lead`: làm mượt tốc độ xe dẫn bằng trung bình trượt 5 bước rồi lấy đạo hàm trung tâm.
Loại 383 quỹ đạo trùng hệt còn 9.610; nếu chưa loại trùng, bản sao có thể xuất hiện đồng thời trong train và test dù đã chia theo ID.
Sau loại trùng, chia theo quỹ đạo với seed 246 được 6.727 train và 2.883 test; chia theo khung sẽ đưa các trạng thái gần giống của cùng một hành trình vào cả hai tập.
Quan hệ `Spatial_Gap = Pos_LV − Pos_FAV − 4.5` có độ lệch lớn nhất khoảng 3.64e-11 m; kiểm tra này hỗ trợ tính nhất quán hình học của cột khoảng cách.

## Câu 2 · Phát lại

Nguồn: [replay.json](evidence/replay.json). Phát lại 600 đoạn, 9.5 MB: cùng mã chạy hai lần khác 0 đoạn; refactor float32 khác chính xác ở 600 đoạn nhưng 0 đoạn vượt `atol + rtol × |golden|`, với `atol = rtol = 0.001`.
Sai khác lớn nhất là 1.22e-4 m/s²; thứ tự cộng số thực làm `(0.1+0.2)+0.3 = 0.6000000000000001` nhưng `0.1+(0.2+0.3) = 0.6`, nên kiểm tra bằng dung sai tránh báo sai cho thay đổi tính toán nhỏ.
AEB làm lệch 8 đoạn: 76, 135, 235, 300, 304, 305, 491, 547; kiểm tra lại `work/replay_log.jsonl` qua `IDMStream` cho thấy đoạn 76 lệch ở khung 82, gap khoảng 6.16 m, tiến gần 4.20 m/s, TTC khoảng 1.47 s.
Vì TTC < 1.5 s, AEB đổi lệnh khoảng −4.56 thành −6.00 m/s²; các đoạn không chạm điều kiện này giữ nguyên lệnh, nên đây là thay đổi hành vi có điều kiện.
Chế độ nhanh nhất ghi 0 khung trễ và 0 khung rớt; chế độ thời gian thật ghi 581 trễ, 574 rớt trên 59.465 khung, làm 382/600 đoạn đổi kết quả vượt dung sai.
Các bộ đếm trễ/rớt đến từ mô hình lịch và chi phí xử lý của bài lab, chưa phải đo độ trễ phần cứng thật; phát nhanh nhất che lỗi vi phạm hạn chót do không áp hạn chót này.

## Câu 3 · Kịch bản

Nguồn: [scenarios.json](evidence/scenarios.json), [hình lưới IDM](evidence/scenarios_idm.png). Với gap 12 m và tiến gần 5 m/s, TTC = 2.40 s, giảm tốc cần thiết = 1.04 m/s².
Lưới có tốc độ xe thử 10 m/s, 60 gap từ 2.760 đến 59.312 m (p2–p98 của log), 45 mức tiến gần 0–10 m/s, tổng 2.700 ô.
Biên bất khả thi là `gap < dv × t_react + dv²/(2 × 9)` khi dv > 0: vùng này chiếm 0.96% ở độ trễ 0 s và 5.15% ở 0.6 s; không nên quy các va chạm đó thành lỗi riêng của bộ điều khiển.
Bộ phanh cực đại có 0 ô lệch so với biên trên các ô cách biên hơn 5% (2.695 và 2.681 ô được kiểm); việc bỏ vùng sát biên hạn chế ảnh hưởng bước mô phỏng 0.1 s.
IDM có diện tích va chạm tăng 0.96% → 5.15% (tăng 4.19 điểm phần trăm), nhưng phần va chạm tránh được vẫn 0%; độ trễ đã làm mất khoảng cách an toàn.
Khai thác được 614 ca phanh (6.39% quỹ đạo); giảm tốc đỉnh của chuỗi phanh đầu tiên có trung vị -2.535, đuôi phanh mạnh -4.211 m/s² (trường `peak_decel_p99` thực tế lấy phân vị 1% của gia tốc âm).
Mốc phanh mạnh của người lái là 3.625 m/s² theo phân vị 99.9% độ lớn phanh, không phải cực đại tuyệt đối; 5.22% lưới cần phanh mạnh hơn mốc đó. Các ca khai thác trên hình tập trung ở tiến gần thấp, nên log bao phủ kém vùng cut-in khó.

## Câu 4 · Vòng hở và vòng kín

Nguồn: [openloop.json](evidence/openloop.json), [closedloop.json](evidence/closedloop.json) (`ranking`); [hình so sánh](evidence/openloop_vs_closedloop.png). Bảng xếp hạng dùng trung bình qua seed đúng như pipeline; quyết định cổng dùng seed xấu nhất.

| Bộ điều khiển | L2 trung bình (m) | Hạng L2 | Va chạm trung bình (%) | Hạng vòng kín |
|---|---|---|---|---|
| const-vel | 1.252 | 3 | 19.94 | 4 |
| IDM | 1.470 | 4 | 0.07 | 1 |
| EgoMLP | 0.763 | 2 | 16.62 | 3 |
| PercMLP | 0.615 | 1 | 1.09 | 2 |

Ba cặp đảo hạng là (constant-velocity, IDM), (IDM, EgoMLP), (IDM, PercMLP); dải va chạm của các mô hình học nằm trong bảng kết quả cuối.
IDM có L2 cao vì không tối ưu bắt chước quỹ đạo người lái trong 3 s; khi tự lái nó phản hồi theo gap và tốc độ xe dẫn tại từng bước, nhờ đó hạn chế sai số tích lũy và có va chạm thấp nhất.
EgoMLP có L2 0.763 m tốt hơn constant-velocity 1.252 m nhưng vẫn va chạm trung bình 16.62%: đầu vào chỉ có lịch sử tốc độ xe mình, không có gap hay tốc độ tương đối để nhận biết xe trước đang chậm lại.
Lát cắt xe dẫn sắp phanh chỉ chiếm 4.07% cửa sổ; L2 EgoMLP trên lát cắt là 1.057 m, cao hơn trung bình 0.763 m, nên một điểm trung bình che được chất lượng kém ở ca quan trọng.

## Câu 5 · Một lần chạy chưa đủ

Nguồn: [closedloop.json](evidence/closedloop.json) (`logs.*.per_seed`); mỗi seed được kiểm tra trên cùng 2.883 quỹ đạo giữ lại.

| Seed | EgoMLP: va chạm (%) | PercMLP: va chạm (%) | PercMLP+: va chạm (%) |
|---|---|---|---|
| 4107 | 23.27 | 1.56 | 0.21 |
| 4108 | 9.30 | 0.59 | 0.21 |
| 4109 | 17.90 | 1.94 | 0.28 |
| 4110 | 9.43 | 0.49 | 0.21 |
| 4111 | 23.20 | 0.87 | 0.21 |

EgoMLP có dải 9.30–23.27%, rộng 13.98 điểm phần trăm; seed xấu nhất cao gấp 2.50 lần seed tốt nhất. PercMLP có dải 0.49–1.94%.
Wilson 95% của seed EgoMLP xấu nhất là [21.77%, 24.85%]; khoảng này phản ánh bất định tỷ lệ trên tập kiểm tra, còn dải qua seed phản ánh bất định huấn luyện, hai thứ không thay thế nhau.
Chỉ chạy seed 4108 có thể báo 9.30%, trong khi seed 4107 báo 23.27%; mức chênh này có thể khiến đánh giá rủi ro quá lạc quan nếu chỉ chọn lần chạy tốt.
Tôi báo cả trung bình để xếp hạng, dải qua seed để thấy độ ổn định và cận trên Wilson của seed xấu nhất để áp cổng; 5 seed trên một tập test vẫn chưa chứng minh độ tin cậy ở mọi phân bố.

## Câu 6 · Tác nhân phản ứng

Nguồn: [agents.json](evidence/agents.json) (`calibration`, `rear_comparison_t_react_*`). Chọn T từ lưới thử bằng Wasserstein-1 của phân bố thời khoảng cách: T = 1.4 s, W1 = 0.362; tác nhân không phản ứng có W1 = 0.840; người lái có thời khoảng cách trung vị 1.849 s.
W1 thấp hơn cho thấy khớp hơn ở đại lượng hiệu chỉnh này, chưa đủ để gọi tác nhân chân thực trong mọi hành vi. Xe sau bắt đầu cách 16 m và có độ trễ phản ứng 0.6 s.


Độ trễ xe thử 0.0 s; đơn vị %.

| Bộ điều khiển | Phát lại: bất kỳ | Phát lại: phía sau | Phản ứng: bất kỳ | Phản ứng: phía sau |
|---|---|---|---|---|
| const-vel | 73.37 | 0.00 | 73.37 | 0.00 |
| IDM | 69.56 | 68.59 | 0.96 | 0.00 |
| EgoMLP | 66.85 | 0.00 | 66.85 | 0.00 |
| PercMLP | 73.37 | 65.78 | 12.26 | 0.00 |
| PercMLP+ | 69.63 | 67.56 | 2.07 | 0.00 |


Độ trễ xe thử 0.6 s; đơn vị %.

| Bộ điều khiển | Phát lại: bất kỳ | Phát lại: phía sau | Phản ứng: bất kỳ | Phản ứng: phía sau |
|---|---|---|---|---|
| const-vel | 73.37 | 0.00 | 73.37 | 0.00 |
| IDM | 70.07 | 64.93 | 5.15 | 0.00 |
| EgoMLP | 69.78 | 0.00 | 69.78 | 0.00 |
| PercMLP | 71.70 | 52.67 | 20.70 | 0.00 |
| PercMLP+ | 69.78 | 64.56 | 5.22 | 0.00 |


Với độ trễ xe thử 0.6 s, IDM bị tính 64.93% va chạm phía sau khi xe sau phát lại: xe này giữ tốc độ, không phản ứng khi xe thử phanh để tránh xe trước.
Cho xe sau phản ứng làm va chạm phía sau IDM về 0%, tổng từ 70.07% xuống 5.15%; constant-velocity vẫn 73.37% vì không phanh, nên không chịu kiểu va chạm phía sau này.
Thế giới phát lại khiến IDM trông gần constant-velocity, còn thế giới phản ứng phân biệt rõ chất lượng; "va chạm giả" ở đây là hệ quả giả định tác nhân không phản ứng, không có nghĩa mọi va chạm phía sau ngoài đời đều được miễn trách nhiệm.
Các hàng mô hình học dùng seed có tổng va chạm trong thế giới phản ứng lớn nhất theo pipeline; hàng này có thể khác seed xấu nhất trên log hoặc lưới không xe sau, nên không đồng nhất các phép đo.

## Câu 7 · Cổng phát hành và sửa

Nguồn: [gate.json](evidence/gate.json), [fix.json](evidence/fix.json); PercMLP bị chặn bởi 5/7 quy tắc, R1 tái lập và R7 độ phủ đạt.

| Quy tắc rớt | Bằng chứng PercMLP | Nguyên nhân cần xử lý |
|---|---|---|
| R2_closed_loop_logs | candidate upper 95 % bound 2.51 % vs baseline 0.07 % + 1 point (candidate rate 1.94 %) | Sai số khi tự lái tích lũy trên log giữ lại |
| R3_braking_slice | 6.2 % (n=161, upper 11.1 %) vs baseline 0.0 % + 5 points | Yếu trên quỹ đạo có xe dẫn phanh |
| R4_scenario_grid | candidate avoidable-but-collides 15.56 % of the grid vs baseline 0.00 % + 2 points | Thiếu trạng thái cut-in khó trong dữ liệu |
| R5_seed_stability | 0.5 .. 1.9 % over 5 run(s) | Kết quả nhạy với seed huấn luyện |
| R6_reactive_world | 20.7 % of the grid vs baseline 5.1 % | Vẫn đâm xe trước khi xe sau đã phản ứng |

Hai nhóm nguyên nhân là thiếu trạng thái cut-in khó (R4, R6) và lỗi khi tự điều khiển/tính thiếu ổn định (R2, R3, R5); điểm L2 thấp trên log không xử lý được hai nhóm này.
Tôi thêm cửa sổ từ 20.000 **kịch bản** cut-in mô phỏng với nhãn IDM, rồi DAgger 2 vòng trên **log train**: cho mô hình lái, ghi trạng thái nó tự đi vào và gắn nhãn dự báo IDM tại đúng trạng thái đó; không dùng log test để huấn luyện.
Sau sửa, PercMLP+ đạt 7/7: R2 cận trên 0.55%; R3 0/161 ca phanh, cận trên 2.33%; R4 diện tích tránh được vẫn đâm 0.07%; R5 dải 0.21–0.28%; R6 tổng va chạm thế giới phản ứng 5.22% → RELEASE theo cổng.
L2 tăng 0.615 → 0.797 m trong khi va chạm seed xấu nhất giảm 1.94% → 0.28% và diện tích cut-in tránh được vẫn đâm giảm 15.56% → 0.07%, cho thấy cần kiểm thử vòng kín.
PercMLP+ chỉ đạt tiêu chí **không kém hơn trong ngưỡng cổng**, chưa có bằng chứng tốt hơn IDM: IDM va chạm 0.07% trên log và diện tích tránh được vẫn đâm 0%, còn PercMLP+ là 0.28% và 0.07%.
Ở tốc độ kiểm tra 20 m/s, vượt khoảng 4–16 m/s của họ cut-in huấn luyện, diện tích va chạm tránh được của PercMLP là 18.33–40.52%, PercMLP+ 0.07–0.96%, IDM 0%; đây là diện tích **loại vùng bất khả thi**, không phải mọi va chạm.
Tôi chưa đề xuất thay IDM ngoài thực tế: cần thêm dữ liệu kiểm tra độc lập, nhiều seed, kiểm thử cảm biến/độ trễ và tương tác hai phía. Học nhãn IDM tạo phụ thuộc vào chuyên gia nhưng không chứng minh mô hình học không bao giờ vượt chuyên gia; kết luận ở đây chỉ dựa trên phép đo đã có.

## Câu 8 · Bài lab này không chứng minh được gì

Nguồn: [closedloop.json](evidence/closedloop.json) (`noise`), [agents.json](evidence/agents.json), [fix.json](evidence/fix.json), [data_audit.json](evidence/data_audit.json).
**Giới hạn cảm biến và vật lý:** mô phỏng một chiều dọc, dt = 0.1 s, không có nhận dạng vật thể hay đổi làn thật; nhiễu thử nghiệm có độ lệch chuẩn gap 1 m và tốc độ xe dẫn 0.5 m/s, không phải biên ± cố định. Chưa có bằng chứng rằng các giả định này bao trùm lỗi cảm biến ngoài đời.
**Giới hạn tác nhân:** xe dẫn trên log không phản ứng với xe thử; đổi riêng xe sau đã làm tổng va chạm IDM thay đổi 70.07% → 5.15%, nên kết luận phụ thuộc mạnh vào mô hình tương tác. Cần tác nhân phản ứng ở cả trước và sau, được đánh giá trên nhiều hành vi.
**Giới hạn độ phủ và thống kê:** 5 seed cùng 2.883 quỹ đạo test chưa bao trùm phân bố mới; tốc độ xe theo sau p1–p99 chỉ 0.752–17.829 m/s, gap 2.150–72.829 m. Các phân vị này không phải phạm vi nhỏ nhất–lớn nhất và không chứng minh dữ liệu đại diện cho mọi con đường.
**Giới hạn chuyên gia và cổng:** nhãn bổ sung đến từ IDM nên mô hình có thể kế thừa lỗi của IDM; qua ngưỡng cổng 1 hoặc 2 điểm phần trăm chỉ là tiêu chí lab, không phải chứng nhận an toàn. Cần chuyên gia/tiêu chuẩn kiểm tra độc lập để đánh giá cả baseline.
Tôi chỉ chấp nhận RELEASE trong phạm vi mô phỏng đã kiểm; trước phát hành thực cần thu log cut-in/tốc độ cao, giữ tập kiểm tra mới độc lập theo thời gian và địa điểm, kiểm thử mất tín hiệu/độ trễ và chạy trên đường thử kín có giám sát.

## Tái lập và kiểm tra bài nộp

Môi trường chạy: Python 3.13.13, NumPy 2.5.1, pandas 3.0.3, matplotlib 3.11.1.
Chạy từ thư mục repo, sau khi cài `requirements.txt`; dùng UTF-8 để đọc báo cáo tiếng Việt trên Windows:

```powershell
python -X utf8 run_all.py --student-id 2A202602859 --seeds 5
python -X utf8 -m unittest discover -s tests -v
python -X utf8 scripts/check_submission.py
```

Không dùng `--quick`; giữ nguyên tests, ngưỡng cổng và pipeline gốc. Chỉ sửa thân 10 hàm TODO và báo cáo; các JSON/PNG được pipeline sinh tự động. `data/`, `work/` không đưa vào Git.
Kết quả tự kiểm lưu trong [evidence/verify.txt](evidence/verify.txt); dữ liệu thô được tải lại bằng bước `data` của pipeline.
