# CSC4007 — Lab 4 Analysis Report

> Sinh viên điền báo cáo này sau khi chạy baseline và các biến thể nâng cấp.

## 1. Thông tin chung

- Họ tên: Nguyễn Mạnh Duy
- MSSV: 1671040006
- Lớp: KHMT 16-01
- Link GitHub repo:https://github.com/FIT-DNU-CS-16-01/csc4007-lab4-manhduy04.git
- Link W&B project/run nếu có:https://wandb.ai/duy19912745-dainam-vietnam/csc4007-lab4-lstm-gru?nw=nwuserduy19912745

## 2. Baseline bắt buộc

Mô hình baseline trong Lab 4:

```text
Tokenized text → Embedding → 1-layer LSTM → Dropout → Linear classifier
```

Cấu hình đã chạy:

Tham số	Giá trị
seed	42
vocab_size	20000
max_len	256
embed_dim	128
hidden_dim	128
num_layers	1
bidirectional	False
dropout	0.3
lr	1e-3
batch_size	64
epochs_trained	6

Kết quả baseline:

Split	Loss	Accuracy	Macro-F1
Validation	0.31	0.867	0.866
Test	0.33	0.861	0.860

Nhận xét ngắn về baseline:

- Mô hình LSTM baseline cho kết quả khá ổn định trên tập IMDB.
- Validation loss giảm đều qua các epoch chứng tỏ mô hình học được đặc trưng sentiment.
- Macro-F1 gần bằng Accuracy cho thấy dữ liệu khá cân bằng giữa hai lớp positive và negative.
- Tuy nhiên mô hình vẫn gặp khó khăn ở các review dài có nhiều chuyển ý như “but”, “however”.

## 3. Bảng ablation

| Run           | model_type | bidirectional | num_layers | max_len | hidden_dim | dropout | Test Accuracy | Test Macro-F1 | Nhận xét                  |
| ------------- | ---------- | ------------: | ---------: | ------: | ---------: | ------: | ------------: | ------------: | ------------------------- |
| baseline_lstm | lstm       |         False |          1 |     256 |        128 |     0.3 |         0.861 |         0.860 | Baseline ổn định          |
| gru_baseline  | gru        |         False |          1 |     256 |        128 |     0.3 |         0.868 |         0.867 | Huấn luyện nhanh hơn LSTM |
| bilstm        | lstm       |          True |          1 |     256 |        128 |     0.4 |         0.882 |         0.881 | Cải thiện tốt nhất        |
| stacked_bigru | gru        |          True |          2 |     256 |        128 |     0.4 |         0.878 |         0.876 | Mạnh nhưng dễ overfit     |


## 4. So sánh công bằng

Các run có dùng cùng dataset không?
→ Có, tất cả đều dùng IMDB dataset.

Các run có dùng cùng train/validation/test split không?
→ Có.

Các run có dùng cùng seed không?
→ Có, đều dùng seed = 42.

Metric chính để chọn mô hình là gì?
→ Validation Macro-F1.

Có dùng test set để chọn mô hình không? Vì sao không nên?
→ Không. Test set chỉ dùng để đánh giá cuối cùng nhằm tránh data leakage và overfitting vào tập test.

## 5. Phân tích learning curves

Dựa vào `outputs/figures/loss_curve.png` và `outputs/figures/metric_curve.png`:

- Train loss giảm đều qua các epoch.
- Validation loss giảm mạnh ở giai đoạn đầu rồi chậm dần.
- Validation Macro-F1 cải thiện tốt nhất khoảng epoch 4–5.
- Ở epoch cuối, khoảng cách giữa train loss và validation loss tăng nhẹ → có dấu hiệu overfitting.

Nhận xét:

- Baseline LSTM chưa overfit mạnh nhưng mô hình stacked có xu hướng overfit rõ hơn.
- Nếu tăng epoch quá nhiều, validation metric có thể giảm.
- Có thể thử giảm learning rate hoặc tăng dropout để cải thiện khả năng tổng quát hóa.
- Early stopping nên được áp dụng khoảng epoch 5.

## 6. Confusion matrix

Dựa vào `outputs/figures/confusion_matrix.png`:

- Mô hình nhầm negative thành positive nhiều hơn.
- False positive cao hơn false negative một chút.
- Các review có mixed sentiment thường bị dự đoán thành positive.

Nhận xét:

- BiLSTM giảm được số lượng false negative so với baseline.
- Trong bài toán thực tế, false positive có thể khiến hệ thống đánh giá sai trải nghiệm người dùng.
- Confusion matrix cho thấy accuracy cao nhưng vẫn tồn tại lỗi ở các review phức tạp.

## 7. Error analysis

Dựa trên `outputs/error_analysis/error_analysis.csv`, đã chọn 10 mẫu sai có confidence cao và phân tích nguyên nhân dựa trên nội dung review.

| STT | Trích đoạn review | Nhãn đúng | Mô hình dự đoán | Confidence | Nguyên nhân giả định |
|---:|---|---|---|---:|---|
| 1 | “This is definitely one of the best Kung fu movies in the history of Cinema...” | Negative | Positive | 0.9998 | Dựa vào từ tích cực mà bỏ qua ngữ cảnh tổng thể |
| 2 | “This movie was pure genius... Johnny Depp is magnificent.” | Negative | Positive | 0.9998 | Tập trung quá nhiều vào từ tích cực, bỏ qua nội dung tiêu cực gián tiếp |
| 3 | “Given this film's incredible reviews... sadly, it collapses like a pack of cards.” | Negative | Positive | 0.9997 | Ngữ cảnh chuyển sang chê bai nửa sau bị bỏ qua |
| 4 | “It's not Citizen Kane, but it does deliver... I didn't watch it for the dialog.” | Positive | Negative | 0.9997 | Phủ định kiểu “not ... but” gây nhầm lẫn |
| 5 | “Brain of Blood starts as Abdul Amir... This is director Adamson's masterpiece...” | Positive | Negative | 0.9996 | Review dài và nhiều chi tiết phức tạp làm mất trọng tâm sentiment |
| 6 | “Just watched on UbuWeb... it was fascinating to watch even with the guitar score...” | Negative | Positive | 0.9996 | Từ ngữ tích cực về trải nghiệm được ưu tiên hơn đánh giá chung |
| 7 | “This is among one of many USA attempts of remaking a old classic British TV show...” | Negative | Positive | 0.9996 | Đoạn review dài với nhiều thông tin dẫn dắt gây nhiễu sentiment |
| 8 | “The Curse of Monkey Island... and everything about it is just awesome.” | Positive | Negative | 0.9996 | Từ “awesome” chưa đủ bù lại cảm nhận tiêu cực nửa đầu review |
| 9 | “Masterpiece. Carrot Top blows the screen away...” | Negative | Positive | 0.9995 | Mở đầu mạnh bằng “Masterpiece” khiến mô hình thiên về positive |
| 10 | “This was Laurel and Hardy's last silent film... this aspect of their careers just seems to have ended with a whimper.” | Positive | Negative | 0.9994 | Kết luận hơi tiêu cực làm lu mờ phần đánh giá tích cực ban đầu |

Gợi ý nhóm lỗi:

- phủ định;
- mixed sentiment;
- sarcasm;
- review dài;
- chuyển ý bằng “but/however”.

## 8. Kết luận

Mô hình tốt nhất của em là:

- Run name: bilstm
- Cấu hình:
  - model_type = lstm
  - bidirectional = True
  - hidden_dim = 128
  - dropout = 0.4
  - max_len = 256
- Test accuracy: 0.882
- Test macro-F1: 0.881

Giải thích vì sao mô hình này tốt hơn baseline:

- BiLSTM đọc chuỗi theo cả hai hướng nên hiểu ngữ cảnh tốt hơn.
- Mô hình xử lý tốt hơn các review dài và câu có chuyển ý.
- Validation Macro-F1 ổn định hơn baseline.
- Confusion matrix cân bằng hơn giữa hai lớp.

## 9. Tự đánh giá

- [x] Em đã chạy baseline LSTM.
- [x] Em đã thử ít nhất 2 biến thể nâng cấp.
- [x] Em đã lưu checkpoint tốt nhất.
- [x] Em đã phân tích learning curves.
- [x] Em đã phân tích confusion matrix.
- [x] Em đã phân tích ít nhất 10 mẫu sai.
- [x] Em đã commit code và report lên GitHub.
