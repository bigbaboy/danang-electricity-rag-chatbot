[README.md](https://github.com/user-attachments/files/31367805/README.md)
# 🔌 Chatbot Tư vấn Khách hàng Điện lực Đà Nẵng

> Ứng dụng kỹ thuật **RAG (Retrieval-Augmented Generation)** để xây dựng hệ thống chatbot tư vấn khách hàng cho Điện lực Đà Nẵng — triển khai thực tế tại **[chatbotdienlucdanang.streamlit.app](https://chatbotdienlucdanang.streamlit.app)**

---

## 📌 Giới thiệu

Đây là đồ án tốt nghiệp của sinh viên **Lê Minh Đạt** (MSSV: 221124029109, Lớp 48K29.1), Khoa Thương mại Điện tử, Đại học Kinh tế — Đại học Đà Nẵng.

**Giảng viên hướng dẫn:** TS. Nguyễn Phong Sơn

Hệ thống giải quyết 3 vấn đề thực tiễn của ngành điện lực:
- Văn bản pháp lý dày đặc, khó tra cứu (QĐ 1279, TT 09/2023, NĐ 58/2025)
- Các LLM phổ thông (ChatGPT, Gemini) không có dữ liệu ngành điện VN → dễ hallucination
- Cổng EVNCSKH.vn thiếu công cụ hỏi đáp tự động và tính toán hỗ trợ

---

## ✨ Tính năng

| # | Tính năng | Mô tả |
|:---:|---|---|
| 01 | **Trợ lý Pháp lý** | Hỏi đáp dựa trên kho văn bản EVN, có trích dẫn nguồn từng điều khoản |
| 02 | **Tính tiền điện** | Biểu giá bậc thang 6 mức theo QĐ 1279/QĐ-BCT, có so sánh tách hộ |
| 03 | **Phân tích tiêu thụ** | Thư viện 9+ thiết bị, biểu đồ pie, benchmark hộ gia đình |
| 04 | **Điện mặt trời** | Tính sản lượng & hoàn vốn theo PSH từng vùng (NĐ 58/2025) |
| 05 | **Quản lý tài liệu** | Upload PDF mới → tự động chunking + embedding incremental |

---

## 🏗️ Kiến trúc hệ thống

```
Khách hàng
     │
     ▼
┌─────────────────────────────────────────────┐
│         TẦNG GIAO DIỆN — Streamlit          │
│  Tab 1  │ Tab 2  │ Tab 3  │ Tab 4  │ Tab 5  │
└─────────────────────────────────────────────┘
     │
     ▼
┌─────────────────────────────────────────────┐
│         TẦNG NGHIỆP VỤ — Python             │
│  rag_pipeline.py  │  utils.py               │
│  db_manager.py    │  doc_pdf_smart.py        │
│  config.py                                  │
└─────────────────────────────────────────────┘
     │
     ▼
┌─────────────────────────────────────────────┐
│           TẦNG DỮ LIỆU                      │
│  FAISS Vector DB  │  Groq Cloud API          │
│  Văn bản EVN PDF  │  LlamaParse OCR          │
└─────────────────────────────────────────────┘
```

---

## ⚙️ Pipeline RAG

```
Câu hỏi → Fast-path check
              │
              ├── Có số kWh + từ khóa tiền → Python tinh_tien_dien() → Kết quả (<50ms)
              │
              └── Không → RAG Pipeline
                              │
                         [Retrieval] Embedding → FAISS tìm top-15 chunks
                              │
                         [Re-ranking] Lọc conf ≥ 0.30, sort giảm dần, lấy top-5
                              │
                         [Confidence] conf = max(0, 1 − L²/2)
                              │
                         [Generation] LLaMA 3.1-8B (Groq) → Câu trả lời + Citations
```

### 🚀 Điểm sáng: Fast-path Intent Detection

Phát hiện và giải quyết bug LLaMA 3.1-8B lặp vô tận khi tính tiền bậc thang. Thay vì đưa vào RAG, hệ thống phát hiện ý định tính tiền và chuyển sang hàm Python deterministic:

- **Điều kiện 1:** Câu hỏi chứa ≥ 1 từ khóa tiền tệ (`tiền`, `hết`, `bao nhiêu`, `hóa đơn`...)
- **Điều kiện 2:** Regex bắt được số + đơn vị kWh (`250kWh`, `300 số điện`, `450 chữ`...)
- **Kết quả:** `<50ms` thay vì `1–2s`, `14/14` test PASS, `0%` lặp vô tận

---

## 📊 Kết quả thực nghiệm

| Chỉ số | Kết quả |
|---|:---:|
| Độ chính xác RAG | **86,7%** (26/30 câu test) |
| Unit test | **105/105 PASS (100%)** |
| Sai lệch tính tiền điện | **0 đồng** |
| Chi phí vận hành | **0 đồng/tháng** |
| Thời gian phản hồi RAG | **1–2 giây** |
| Thời gian phản hồi tính tiền | **<50 ms** |
| Hoàn vốn ĐMT Đà Nẵng (4,8 kWp) | **5,0 năm** |

---

## 🛠️ Tech Stack

| Thành phần | Công nghệ |
|---|---|
| **Frontend** | Streamlit |
| **Backend** | Python 3.10 |
| **RAG Framework** | LangChain |
| **Vector Database** | FAISS (IndexFlatL2) |
| **Embedding Model** | `paraphrase-multilingual-MiniLM-L12-v2` (384 chiều) |
| **LLM** | LLaMA 3.1-8B Instant via Groq Cloud (LPU) |
| **PDF Parser** | PyMuPDF + LlamaParse OCR |
| **Deploy** | Streamlit Community Cloud |

---

## 📁 Cấu trúc dự án

```
DeAn/
├── app.py                  # Entry point — khởi động Streamlit app
├── config.py               # Tất cả tham số cấu hình (biểu giá, LLM, RAG...)
├── rag_pipeline.py         # Pipeline RAG 4 giai đoạn + Fast-path
├── utils.py                # Hàm tính toán thuần (tiền điện, ĐMT, tiêu thụ)
├── db_manager.py           # Quản lý vòng đời FAISS DB (build/load/merge)
├── doc_pdf_smart.py        # Định tuyến PDF: PyMuPDF vs LlamaParse OCR
├── sidebar.py              # Sidebar UI
├── ui_helpers.py           # Hàm render dùng chung
├── tao_vector_db.py        # Script build vector DB từ tài liệu
├── test_tinh_toan.py       # 105 unit tests (pytest)
├── tabs/
│   ├── chat.py             # Tab Trợ lý Pháp lý
│   ├── tien_dien.py        # Tab Tính tiền điện
│   ├── tieu_thu.py         # Tab Phân tích tiêu thụ
│   ├── solar.py            # Tab Điện mặt trời
│   └── docs.py             # Tab Quản lý tài liệu
└── faiss_dienluc_db/
    ├── index.faiss         # Vector database (~2.500 chunks)
    └── index.pkl           # Metadata chunks
```

---

## 🚀 Cài đặt & Chạy local

### 1. Clone repository

```bash
git clone https://github.com/<your-username>/chatbot-dien-luc-da-nang.git
cd chatbot-dien-luc-da-nang
```

### 2. Cài đặt dependencies

```bash
pip install -r requirements.txt
```

### 3. Cấu hình API keys

Tạo file `.env` ở thư mục gốc:

```env
GROQ_API_KEY=your_groq_api_key_here
LLAMA_CLOUD_API_KEY=your_llama_cloud_api_key_here
```

> **Lấy API key miễn phí:**
> - Groq: [console.groq.com](https://console.groq.com)
> - LlamaParse: [cloud.llamaindex.ai](https://cloud.llamaindex.ai)

### 4. Chạy ứng dụng

```bash
streamlit run app.py
```

Truy cập: `http://localhost:8501`

---

## ☁️ Deploy lên Streamlit Cloud

1. Fork repository này lên GitHub
2. Vào [share.streamlit.io](https://share.streamlit.io) → New app
3. Chọn repo, branch `main`, entry point `app.py`
4. Vào **Settings → Secrets**, thêm:

```toml
GROQ_API_KEY = "your_groq_api_key"
LLAMA_CLOUD_API_KEY = "your_llama_cloud_api_key"
```

5. Click **Deploy** → Chờ ~2-3 phút

> ⚠️ **Lưu ý:** Folder `faiss_dienluc_db/` phải được commit lên GitHub vì Streamlit Cloud dùng ephemeral filesystem — mỗi lần restart sẽ mất nếu không commit.

---

## 🔑 Tham số cấu hình chính

```python
# Embedding
EMBEDDING_MODEL = "paraphrase-multilingual-MiniLM-L12-v2"

# LLM
LLM_MODEL        = "llama-3.1-8b-instant"
LLM_TEMPERATURE  = 0.2
LLM_MAX_TOKENS   = 800
LLM_FREQUENCY_PENALTY = 0.6

# Chunking
CHUNK_SIZE    = 1200   # ký tự
CHUNK_OVERLAP = 300    # ký tự

# RAG Search
SEARCH_FETCH_K = 15    # lấy 15 chunks ban đầu
SEARCH_TOP_K   = 5     # giữ 5 chunks tốt nhất

# Confidence thresholds
CONFIDENCE_MIN_RELEVANCE = 0.30   # loại bỏ
CONFIDENCE_MED           = 0.45   # badge vàng
CONFIDENCE_HIGH          = 0.62   # badge xanh
```

---

## 📐 Công thức Confidence Score

```
confidence = max(0.0, min(1.0, 1.0 - L²/2))
```

Trong đó `L²` là khoảng cách L2 bình phương giữa vector câu hỏi và vector chunk. Vì vector được chuẩn hóa độ dài = 1, nên `L² ∈ [0, 2]` và công thức đưa điểm về thang `[0, 1]`.

---

## ⚡ Biểu giá điện (QĐ 1279/QĐ-BCT, 09/05/2025)

| Bậc | Khoảng (kWh) | Đơn giá (đ/kWh) |
|:---:|---|---:|
| 1 | 0 – 50 | 1.984 |
| 2 | 51 – 100 | 2.050 |
| 3 | 101 – 200 | 2.380 |
| 4 | 201 – 300 | 2.998 |
| 5 | 301 – 400 | 3.350 |
| 6 | > 400 | 3.460 |

**VAT: 8%**

---

## ☀️ Peak Sun Hours (PSH) theo vùng

| Vùng | PSH (giờ/ngày) |
|---|:---:|
| Hà Nội & Bắc Bộ | 3.8 |
| **Đà Nẵng & Trung Bộ** | **4.8** |
| TP.HCM & Nam Bộ | 5.2 |
| Tây Nguyên | 5.1 |
| Ninh Thuận – Bình Thuận | 5.8 |

---

## 🧪 Chạy Unit Test

```bash
pytest test_tinh_toan.py -v
```

Kết quả: **105/105 PASS (100%)**

---

## ⚠️ Hạn chế

- Kho tài liệu chưa phủ hết nghiệp vụ đặc thù (điện 3 pha, biểu giá TOU)
- LLaMA 8B độ trễ 1–2s — chưa thật sự real-time
- Chưa có feedback loop từ người dùng
- Chưa kết nối API thực của EVNCPC
- Chưa xử lý prompt injection và rate limiting

---

## 🔮 Hướng phát triển

- [ ] Mở rộng kho tài liệu sang điện 3 pha, biểu giá TOU
- [ ] Nâng cấp LLM lên LLaMA 70B / Gemini Flash
- [ ] Áp dụng Hybrid Search (vector + BM25) + Re-ranker
- [ ] Xây dựng feedback loop: log → phân tích → tinh chỉnh
- [ ] Hỗ trợ tiếng Anh cho khách quốc tế
- [ ] Tích hợp API thực của EVNCPC

---

## 👤 Tác giả

**Lê Minh Đạt**
- MSSV: 221124029109
- Lớp: 48K29.1
- Khoa: Thương mại Điện tử
- Trường: Đại học Kinh tế — Đại học Đà Nẵng
- GVHD: TS. Nguyễn Phong Sơn

---

## 📄 Giấy phép

Dự án này được phát triển cho mục đích học thuật.

---

<div align="center">
  <strong>🌐 Demo trực tiếp: <a href="https://chatbotdienlucdanang.streamlit.app">chatbotdienlucdanang.streamlit.app</a></strong>
</div>
