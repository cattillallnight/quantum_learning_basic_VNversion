# QuantumComputing — bộ công cụ học & lập trình lượng tử

Ba dịch vụ độc lập, chạy chung bằng **một lệnh Docker duy nhất**.

| Dịch vụ | Thư mục | Địa chỉ | Mô tả |
|---|---|---|---|
| **quantumlabs** | `quantumlabs/` | http://localhost:8080 | Nền tảng học tiếng Việt: 20 mô-đun, 60 mô phỏng tương tác, trắc nghiệm, trình dựng mạch, trợ lý AI. Site tĩnh (nginx). |
| **qlp** | `qlp/` | http://localhost:5000 | Playground Qiskit thật: 8 bài học, soạn thảo chạy mã trên Qiskit Aer, chấm bài tự động, chatbot RAG + Ollama. |
| **qserver** | `quantum-programming-server/` | http://localhost:8888 | Máy chủ lập trình: JupyterLab + Qiskit + Dask, kết nối được IBM Quantum. |

## Chạy

```bash
docker compose up -d --build
```

Lần đầu mất vài phút để tải image. Sau đó mở các địa chỉ ở bảng trên.

Dừng / gỡ:

```bash
docker compose down
```

Xem log một dịch vụ:

```bash
docker compose logs -f qlp
```

## Trợ lý AI cục bộ (tùy chọn)

Chatbot của `qlp` hoạt động ở hai chế độ:

1. **Offline-RAG** — luôn có sẵn, không cần cài gì. Tra cứu cơ sở tri thức (`qlp/content/`) + chẩn đoán lỗi theo luật.
2. **LLM đầy đủ** — cài [Ollama](https://ollama.com) rồi tải model:

```bash
ollama pull gemma4:e2b
```

Container `qlp` đã trỏ sẵn tới Ollama trên máy thật qua `host.docker.internal:11434`. Kiểm tra kết nối:

```bash
curl http://localhost:5000/api/health
```

Khi `llm_available` là `true`, khung chat dùng model cục bộ; nếu `false`, xem `llm_error` để biết nguyên nhân (thường là Ollama chưa chạy hoặc chưa tải model).

## Cấu trúc

```
quantumcomputing/
├── docker-compose.yml            # stack duy nhất cho cả 3 dịch vụ
├── quantumlabs/                  # nền tảng học (nginx + HTML/CSS/JS thuần)
│   ├── index.html  style.css  script.js  chatbot.js
│   └── Dockerfile  docker-compose.yml   (chạy riêng lẻ được)
├── qlp/                          # playground Qiskit (Flask)
│   ├── app.py  sandbox.py  rag.py  chat.py
│   ├── content/lessons/*.md      # thêm bài học = thêm file .md
│   └── static/  tests/  docs/
└── quantum-programming-server/   # JupyterLab + Qiskit + Dask
    ├── Dockerfile
    ├── requirements-docker.txt
    ├── notebooks/                # 01_bell_state, 02_dask_pipeline, 03_benchmark
    └── scripts/                  # verify_bell, ibm_setup, benchmark_sim, dask_pipeline
```

## Ghi chú kỹ thuật

- **Phiên bản:** Qiskit 2.6.1 · qiskit-aer 0.18.2 · Python 3.12 (qlp) / 3.11 (qserver) · Flask 3.x
- **Sandbox của `qlp`** chỉ là lớp bảo vệ cơ bản (chặn import, giới hạn CPU/RAM/thời gian) — không đủ cho dịch vụ công khai, nên chạy trong Docker không mở cổng ra ngoài.
- **Tiến độ học** lưu trong `localStorage` của trình duyệt, không mất khi tắt container.
- **Volume `qserver-data`** giữ dữ liệu notebook ở `/app/data` qua các lần khởi động lại.
- **IBM Quantum:** chưa cấu hình token. Xem `quantum-programming-server/scripts/ibm_setup.py` và Bài 8 của `qlp` để biết cách kết nối QPU thật.
- **Đổi cổng:** sửa số cổng trong `docker-compose.yml` ở gốc rồi `docker compose up -d`.
