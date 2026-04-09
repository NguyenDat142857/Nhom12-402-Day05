# SPEC — AI Product Hackathon

**Nhóm:** _16_
**Thành viên**:

- Dương Chí Thành - 2A202600047
- Bùi Cao Chinh - 2A202600001
- Phan Xuân Quang Linh - 2A202600492
- Trần Thị Kim Ngân - 2A202600432
- Nguyễn Đức Tiến - 2A202600393
- Nguyễn Trọng Thiên Khôi - 2A202600227

**Track:** ☐ VinFast · ☑ **Vinmec** · ☐ VinUni-VinSchool · ☐ XanhSM · ☐ Open
**Topic:** Vinmec AI Symptom Triage — Gợi ý chuyên khoa thông minh
**Problem statement (1 câu):** Khi đặt lịch khám tại Vinmec online, bệnh nhân bị yêu cầu chọn chuyên khoa ngay từ bước đầu tiên — trong khi ~30-40% bệnh nhân (đặc biệt tân bệnh nhân và người có triệu chứng mới) chưa biết nên khám khoa nào; AI hỏi triệu chứng bằng ngôn ngữ tự nhiên, gợi ý tối đa 2 chuyên khoa phù hợp và cảnh báo các red flag nguy hiểm tiềm ẩn, giúp bệnh nhân đặt lịch đúng ngay từ đầu.

---

## 1. AI Product Canvas

|   | Value | Trust | Feasibility |
|---|-------|-------|-------------|
| **Câu hỏi** | User nào? Pain gì? AI giải gì? | Khi AI sai thì sao? User sửa bằng cách nào? | Cost/latency bao nhiêu? Risk chính? |
| **Trả lời** | _Target user: Bệnh nhân chưa rõ chuyên khoa cần khám (~30-40% lượt đặt lịch, đặc biệt tân bệnh nhân và người có triệu chứng mới/phức tạp). Pain: Vinmec yêu cầu chọn khoa ngay bước đầu khi đặt online — bệnh nhân chưa biết chọn gì thì chọn đại hoặc bỏ cuộc. AI lấp đúng chỗ trống này: thêm option "Chưa biết? Mô tả triệu chứng để AI gợi ý" ngay trên form đặt lịch._ | _AI gợi ý tối đa 2 khoa kèm lý do ngắn. Nếu sai, bác sĩ tiếp nhận điều chỉnh lại — đây là bước bắt buộc trong quy trình Vinmec và là lý do chọn augmentation. Bệnh nhân có nút "Hỏi lại" và "Gặp lễ tân" bất kỳ lúc nào._ | _Cost: ~$0.003–0.008/query (LLM + RAG nhẹ). Latency < 5s. Risk chính: AI gặp triệu chứng trông nhẹ nhưng thực ra là red flag bệnh nặng → gợi ý sai khoa, bệnh nhân không biết mình cần khám gấp._ |

**Automation hay augmentation?** ☐ Automation · ☑ **Augmentation**

Justify: _Augmentation — AI chỉ gợi ý chuyên khoa, bác sĩ luôn là người quyết định chẩn đoán và điều trị cuối cùng. Trong y tế không thể để AI tự động hoàn toàn. Quan trọng hơn: cost of reject = 0 — nếu bệnh nhân không tin gợi ý, họ vẫn có thể tự chọn khoa hoặc hỏi lễ tân như bình thường, không mất gì._

**Learning signal:**

1. User correction đi vào đâu? _Bác sĩ tiếp nhận ghi nhận "khoa AI gợi ý" vs "khoa thực tế khám" → so sánh định kỳ để cải thiện mapping triệu chứng–chuyên khoa._
2. Product thu signal gì để biết tốt lên hay tệ đi? _Tỷ lệ bệnh nhân đi đúng khoa ngay lần đầu (trước vs sau AI). Tỷ lệ bệnh nhân bấm "Gặp lễ tân" ngay sau khi nhận gợi ý (= AI không đủ thuyết phục)._
3. Data thuộc loại nào? ☐ User-specific · ☑ **Domain-specific** · ☐ Real-time · ☑ **Human-judgment** · ☐ Khác

   Có marginal value không? _Có — LLM nền tảng không biết mapping triệu chứng theo từng chuyên khoa cụ thể của Vinmec, danh sách và năng lực từng khoa tại từng cơ sở. Đây là dữ liệu nội bộ độc quyền có lợi thế cạnh tranh._

---

## 2. User Stories — 4 paths

### Feature: _AI Symptom Triage (Hỏi triệu chứng → Gợi ý chuyên khoa)_

**Trigger:** _Bệnh nhân mở MyVinmec app hoặc website để đặt lịch, gặp bước "Chọn chuyên khoa" và bấm vào option "Chưa biết khám khoa nào? Mô tả triệu chứng để AI gợi ý"._

| Path | Câu hỏi thiết kế | Mô tả |
|------|------------------|-------|
| **Happy — AI đúng, tự tin** | User thấy gì? Flow kết thúc ra sao? | _Bệnh nhân nhập "Tôi bị đau đầu dữ dội 2 ngày, buồn nôn, sợ ánh sáng". AI hỏi thêm: "Bạn có bị sốt không?". Bệnh nhân: "Không". AI gợi ý: "Thần kinh (ưu tiên) — triệu chứng phù hợp với đau đầu migraine hoặc căng thẳng thần kinh. Bác sĩ sẽ xác nhận lại khi tiếp nhận." Bệnh nhân bấm "Đặt lịch khoa Thần kinh" → hoàn tất._ |
| **Low-confidence — AI không chắc** | System báo "không chắc" bằng cách nào? User quyết thế nào? | _Bệnh nhân nhập "Tôi vừa tê tay phải thoáng qua, nói đớ khoảng 5 phút rồi hết bình thường". AI nhận ra đây là red flag TIA (đột quỵ thoáng qua) dù triệu chứng đã hết → không chỉ gợi ý khoa mà hiển thị cảnh báo: "⚠️ Triệu chứng này có thể là dấu hiệu cảnh báo đột quỵ dù đã hết — cần được khám Thần kinh trong ngày hôm nay, không nên để đến hôm sau." Kèm nút "Đặt lịch ưu tiên khoa Thần kinh"._ |
| **Failure — AI sai** | User biết AI sai bằng cách nào? Recover ra sao? | _Bệnh nhân có triệu chứng đau khớp + mờ mắt + mệt mỏi, AI gợi ý Cơ xương khớp nhưng thực ra là lupus cần Nội tổng quát. Bác sĩ Cơ xương khớp tiếp nhận phát hiện bất thường → điều phối sang Nội tổng quát. Bệnh nhân mất thêm 1 lượt khám nhưng không bị bỏ sót hoàn toàn vì bác sĩ luôn là điểm kiểm tra cuối._ |
| **Correction — user sửa** | User sửa bằng cách nào? Data đó đi vào đâu? | _Bệnh nhân nhận gợi ý "Nội tổng quát" nhưng biết mình có tiền sử tim mạch → bấm "Gợi ý này chưa đúng với tôi" → chọn lý do "Tôi có bệnh nền liên quan" → AI hỏi thêm về bệnh nền → cập nhật gợi ý sang Tim mạch. Log correction này đưa vào training data để AI học thêm ngữ cảnh bệnh nền._ |

---

## 3. Eval metrics + threshold

**Optimize precision hay recall?** ☐ Precision · ☑ **Recall**

Tại sao? _Với bài toán triage, bỏ sót bệnh nặng nguy hiểm hơn gợi ý dư. Tuy nhiên "gợi ý dư" ở đây được giới hạn cứng tối đa 2 khoa — AI không được phép gợi ý nhiều hơn dù triệu chứng phức tạp. Nếu triệu chứng đa hệ thống, ưu tiên gợi ý Nội tổng quát làm điểm vào, bác sĩ điều phối tiếp — đây là việc của bác sĩ, không phải của AI triage. Đặc biệt: red flag nguy hiểm tiềm ẩn (TIA, xuất huyết não, huyết khối) phải được phát hiện với recall = 100%._

| Metric | Threshold | Red flag (dừng khi) |
|--------|-----------|---------------------|
| _Correct Speciality Rate (Tỷ lệ gợi ý đúng chuyên khoa so với bác sĩ xác nhận)_ | _≥ 75%_ | _< 60% trong 2 tuần liên tiếp_ |
| _Red Flag Detection Recall (Bắt đúng triệu chứng trông nhẹ nhưng nguy hiểm tiềm ẩn)_ | _100%_ | _Bất kỳ 1 red flag bị bỏ sót → dừng ngay để rà soát_ |
| _Max Speciality Suggested (Số khoa gợi ý tối đa)_ | _≤ 2_ | _> 2 khoa trong bất kỳ response nào → lỗi logic cần fix_ |
| _Escalation Rate (Tỷ lệ bệnh nhân chuyển sang gặp lễ tân)_ | _< 30%_ | _> 50% (AI không đủ thuyết phục, cần cải thiện)_ |

---

## 4. Top 3 failure modes

_Lưu ý: Nguy hiểm nhất là khi bệnh nhân không biết AI đang sai mà tin tưởng làm theo — đặc biệt với các triệu chứng trông nhẹ nhưng thực ra là dấu hiệu bệnh nặng._

| # | Trigger | Hậu quả | Mitigation |
|---|---------|---------|------------|
| 1 | _Bệnh nhân mô tả triệu chứng red flag nhưng nghe có vẻ nhẹ: "Tôi bị tê tay thoáng qua rồi hết", "Đau đầu dữ dội lần đầu trong đời", "Phù 1 chân không rõ lý do"._ | _AI xử lý như triệu chứng thông thường → gợi ý khoa sai hoặc không cảnh báo → bệnh nhân đặt lịch bình thường, đến vài ngày sau mới khám → nguy hiểm tính mạng._ | _Xây dựng Red Flag Detector chạy song song độc lập với LLM. Bất kỳ pattern nào khớp danh sách red flag (TIA, xuất huyết não, huyết khối, v.v.) → override gợi ý khoa thường, hiển thị cảnh báo rõ + đề xuất đặt lịch ưu tiên trong ngày._ |
| 2 | _Bệnh nhân có triệu chứng đa hệ thống (nhiều bệnh cùng lúc) → AI cố gắng gợi ý đủ khoa → vượt giới hạn 2 khoa hoặc gợi ý không theo thứ tự ưu tiên._ | _Bệnh nhân nhận được 3-4 khoa → càng phân vân hơn ban đầu → mất trust vào AI, quay lại tự chọn hoặc hỏi lễ tân → AI không giải quyết được pain gốc._ | _Hard constraint: AI không bao giờ gợi ý quá 2 khoa. Trường hợp triệu chứng phức tạp đa hệ thống → luôn ưu tiên Nội tổng quát làm điểm vào đầu tiên, kèm ghi chú: "Bác sĩ Nội tổng quát sẽ đánh giá toàn diện và điều phối thêm nếu cần."_ |
| 3 | _Danh sách chuyên khoa hoặc lịch hoạt động thay đổi (khoa tạm đóng, bác sĩ nghỉ) nhưng knowledge base của AI chưa được cập nhật._ | _AI gợi ý khoa không còn hoạt động tại cơ sở bệnh nhân đang đến → bệnh nhân đặt lịch xong đến nơi bị từ chối hoặc phải chờ xếp lại → mất thời gian, mất trust._ | _Knowledge base về danh sách và lịch hoạt động của từng khoa phải được sync tự động hàng ngày từ hệ thống quản lý bệnh viện Vinmec. AI chỉ gợi ý các khoa đang active tại cơ sở bệnh nhân chọn._ |

---

## 5. ROI 3 kịch bản

|   | Conservative | Realistic | Optimistic |
|---|-------------|-----------|------------|
| **Assumption** | _50 lượt dùng triage/ngày (trong ~30-40% bệnh nhân chưa biết chọn khoa), 60% đi đúng khoa ngay lần đầu_ | _150 lượt/ngày, 75% đi đúng khoa_ | _400 lượt/ngày (mùa cao điểm đăng ký), 85% đi đúng khoa_ |
| **Cost** | _$1/ngày (API calls + infra)_ | _$3/ngày_ | _$8/ngày_ |
| **Benefit** | _Lễ tân giảm 1h/ngày tư vấn câu hỏi lặp lại. Giảm nhẹ tỷ lệ bệnh nhân phải điều phối lại khoa._ | _Giảm 3h/ngày workload lễ tân. Giảm 20% tỷ lệ đi sai khoa. Tăng satisfaction score đặt lịch online._ | _Giảm 8h/ngày, không cần tăng nhân sự lễ tân khi Vinmec mở thêm cơ sở. Có thể nhân rộng toàn hệ thống._ |
| **Net** | _Dương nhẹ — lễ tân đỡ quá tải giờ cao điểm_ | _Tiết kiệm chi phí vận hành rõ rệt, cải thiện trải nghiệm đặt lịch_ | _ROI dương rõ ràng, có thể scale toàn quốc_ |

**Kill criteria:** _Red Flag Detection Recall xuống dưới 100% dù chỉ 1 case → dừng ngay để rà soát. HOẶC Correct Speciality Rate < 60% liên tục 2 tuần → tạm dừng, rebuild knowledge base. HOẶC > 50% bệnh nhân escalate sang lễ tân sau khi dùng AI → AI không giải quyết được pain, cần redesign._

---

## 6. Mini AI spec — Cả team

**Sản phẩm:** Vinmec AI Symptom Triage
**Dành cho:** Bệnh nhân chưa biết chọn chuyên khoa (~30-40% lượt đặt lịch).

**Câu chuyện sản phẩm**

Hiện tại khi khách hàng đặt lịch Vinmec online, bước đầu tiên bệnh nhân phải tự chọn chuyên khoa. Với người đã biết mình cần khám gì thì không thành vấn đề — nhưng với người lần đầu đến, hoặc đang có triệu chứng lạ chưa từng gặp, đây là một rào cản thực sự. Họ hoặc chọn đại một khoa rồi đến viện bị điều phối lại, hoặc phải gọi tổng đài chờ tư vấn. AI triage lấp đúng khoảng trống này: thay vì bắt bệnh nhân tự đoán, hệ thống hỏi họ vài câu về triệu chứng và tự gợi ý khoa phù hợp.

Điểm quan trọng là AI ở đây không chẩn đoán bệnh — nó chỉ giúp bệnh nhân bước vào đúng cửa. Quyết định cuối cùng vẫn là của bác sĩ khi tiếp nhận. Và vì AI có thể sai, sản phẩm được thiết kế để bệnh nhân luôn có thể thoát ra và hỏi lễ tân thật bất kỳ lúc nào — không có điểm nào trong flow bị khóa lại.

Một tính năng quan trọng không kém là nhận diện red flag: những triệu chứng trông có vẻ nhẹ nhưng thực ra là dấu hiệu bệnh nguy hiểm tiềm ẩn — như tê tay thoáng qua rồi hết (có thể là TIA — đột quỵ thoáng qua), hay đau đầu dữ dội lần đầu trong đời. Với những trường hợp này, AI không chỉ gợi ý khoa mà còn cảnh báo rõ và đề xuất đặt lịch trong ngày thay vì để hôm sau.

**Về mặt kỹ thuật**, sản phẩm được xây dựng theo kiến trúc LangGraph agent kết hợp RAG. Agent có 3 tools: hỏi thêm để làm rõ triệu chứng, tra cứu knowledge base nội bộ Vinmec (mapping triệu chứng → chuyên khoa, danh sách khoa đang active tại từng cơ sở), và kiểm tra lịch hoạt động trước khi gợi ý. Agent tự quyết định cần hỏi thêm bao nhiêu câu và tra RAG mấy lần — thay vì hardcode flow cứng.

Phần nhận diện red flag được xử lý trực tiếp trong agent thông qua system prompt có hướng dẫn rõ ràng về các dấu hiệu nguy hiểm tiềm ẩn. Agent luôn trả về structured output có field `is_red_flag` — nếu True thì override kết quả, hiển thị cảnh báo thay vì gợi ý khoa thông thường. Để đảm bảo recall = 100% cho red flag, một LLM call nhỏ thứ hai chạy song song chỉ để verify yes/no — nếu agent nói không nguy hiểm nhưng verifier nói có thì override sang cảnh báo. Cách này tận dụng khả năng hiểu ngôn ngữ tự nhiên của LLM nên không bị giới hạn bởi từ khóa cứng — bệnh nhân gõ "tay tôi bị tê xíu" hay "tay cứ bị kiến bò" hay "tay yếu thoáng qua" đều được nhận diện đúng.

AI luôn gợi ý tối đa 2 khoa. Trường hợp triệu chứng phức tạp đa hệ thống thì ưu tiên Nội tổng quát làm điểm vào đầu tiên — bác sĩ sẽ điều phối tiếp nếu cần, đó là việc của bác sĩ chứ không phải của AI triage.

Input là mô tả triệu chứng tiếng Việt tự nhiên qua chat. Output là tối đa 2 khoa gợi ý kèm lý do ngắn, hoặc cảnh báo red flag nếu phát hiện. Toàn bộ thông tin triệu chứng không được lưu quá 24h. Latency mục tiêu dưới 5 giây.

---

_SPEC Draft v2 — AI Product Hackathon — VinUni A20 — 2026_
