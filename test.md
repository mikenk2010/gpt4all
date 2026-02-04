# Ventra Rocket Ranking — Tournament Management System
## Workflow Summary & Feature Overview

**Date:** 2026-02-04 | **Version:** 1.0 | **Status:** Proposal

---

## 1. Tổng Quan Hệ Thống

Mở rộng Ventra Rocket Ranking từ hệ thống quản lý trận đấu cầu lông casual thành **nền tảng tổ chức giải đấu chuyên nghiệp** với:
- Quản lý giải đấu đầy đủ (tạo giải, đăng ký, bốc thăm, lịch thi đấu)
- Ghi điểm trực tiếp trên mobile bởi trọng tài
- Trình chiếu điểm số real-time lên TV từng sân
- Hiển thị kết quả chung cuộc và lễ trao giải trên TV

**Quy mô mục tiêu:** ≤60 VĐV, 4 sân, 5 nội dung BWF

---

## 2. Roles & Permissions

```mermaid
graph TD
    ADMIN["🔐 ADMIN<br/>Quản trị hệ thống"]
    BTC["📋 BAN TỔ CHỨC (BTC)<br/>Quản lý giải đấu"]
    REF["🧑‍⚖️ TRỌNG TÀI<br/>Ghi điểm trận đấu"]

    ADMIN -->|Tạo & quản lý tài khoản| BTC
    ADMIN -->|Tạo & quản lý tài khoản| REF
    BTC -->|Tạo account & phân công trận| REF

    subgraph "Quyền ADMIN"
        A1["Quản lý tất cả users"]
        A2["Dashboard hệ thống"]
        A3["Toàn quyền truy cập"]
    end

    subgraph "Quyền BTC"
        B1["CRUD giải đấu"]
        B2["Quản lý đăng ký VĐV"]
        B3["Bốc thăm & xếp lịch"]
        B4["Phân công trọng tài"]
        B5["Xác nhận thanh toán"]
        B6["Điều khiển trận đấu"]
    end

    subgraph "Quyền Trọng Tài"
        C1["Xem trận được phân công"]
        C2["Ghi điểm trực tiếp"]
        C3["Bắt đầu / Kết thúc trận"]
    end

    ADMIN --- A1 & A2 & A3
    BTC --- B1 & B2 & B3 & B4 & B5 & B6
    REF --- C1 & C2 & C3
```

---

## 3. Workflow Tổng Thể — Tổ Chức Giải Đấu

```mermaid
flowchart TB
    START(("🏸 Bắt đầu"))

    subgraph PHASE1["📝 PHASE 1: Tạo Giải Đấu"]
        P1A["BTC tạo giải đấu<br/>(tên, ngày, địa điểm, banner)"]
        P1B["Chọn nội dung thi đấu<br/>(đơn nam/nữ, đôi nam/nữ, đôi pha)"]
        P1C["Cấu hình luật<br/>(điểm/set, số set, deuce)"]
        P1D["Cấu hình sân & lệ phí"]
        P1A --> P1B --> P1C --> P1D
    end

    subgraph PHASE2["👥 PHASE 2: Đăng Ký & Thanh Toán"]
        P2A["BTC nhập danh sách VĐV<br/>(tên, avatar, trình độ)"]
        P2B["VĐV tự khai trình độ<br/>BTC duyệt & điều chỉnh"]
        P2C["VĐV chuyển khoản lệ phí"]
        P2D["BTC xác nhận thanh toán<br/>(mark Paid/Unpaid)"]
        P2E["Duyệt đăng ký<br/>(Approved / Rejected)"]
        P2A --> P2B --> P2C --> P2D --> P2E
    end

    subgraph PHASE3["🎯 PHASE 3: Bốc Thăm & Xếp Lịch"]
        P3A["Chia bảng / Xếp hạt giống<br/>(auto hoặc manual)"]
        P3B["Tạo bracket<br/>(vòng bảng / loại trực tiếp / kết hợp)"]
        P3C["Xếp lịch thi đấu<br/>(phân sân + thời gian)"]
        P3D["Phân công trọng tài<br/>cho từng trận"]
        P3A --> P3B --> P3C --> P3D
    end

    subgraph PHASE4["⚡ PHASE 4: Thi Đấu Trực Tiếp"]
        P4A["Trọng tài login<br/>& xem trận được phân công"]
        P4B["TV hiển thị warm-up<br/>(thông tin VĐV + trọng tài)"]
        P4C["Trọng tài ghi điểm<br/>(tap-to-score trên mobile)"]
        P4D["TV cập nhật điểm real-time"]
        P4E["Trận kết thúc<br/>→ TV hiển thị kết quả"]
        P4F["Auto chuyển trận tiếp theo"]
        P4A --> P4B --> P4C --> P4D --> P4E --> P4F
        P4F -.->|Còn trận| P4B
    end

    subgraph PHASE5["🏆 PHASE 5: Kết Quả & Trao Giải"]
        P5A["Tổng hợp BXH<br/>theo từng nội dung"]
        P5B["TV chiếu BXH chung cuộc"]
        P5C["TV chiếu lễ trao giải<br/>(Top 3 + animation)"]
        P5D["Export kết quả"]
        P5A --> P5B --> P5C --> P5D
    end

    START --> PHASE1 --> PHASE2 --> PHASE3 --> PHASE4 --> PHASE5

    style PHASE1 fill:#e3f2fd,stroke:#1565c0
    style PHASE2 fill:#fff3e0,stroke:#e65100
    style PHASE3 fill:#f3e5f5,stroke:#7b1fa2
    style PHASE4 fill:#e8f5e9,stroke:#2e7d32
    style PHASE5 fill:#fff8e1,stroke:#f9a825
```

---

## 4. Workflow Chi Tiết — Ghi Điểm Trực Tiếp (Core Feature)

```mermaid
sequenceDiagram
    participant BTC as 📋 BTC (Mobile/Laptop)
    participant REF as 🧑‍⚖️ Trọng Tài (Mobile)
    participant SYS as 🖥️ Hệ Thống
    participant TV as 📺 TV (Sân)

    Note over BTC,TV: === TRƯỚC TRẬN ===

    BTC->>SYS: Phân công trọng tài cho trận
    SYS->>REF: Hiện trận trong danh sách
    SYS->>TV: Hiển thị WARM-UP<br/>(Avatar VĐV + Trọng tài + Thông tin trận)

    Note over BTC,TV: === BẮT ĐẦU TRẬN ===

    REF->>SYS: Bấm "Bắt đầu trận"
    SYS->>TV: Chuyển sang LIVE SCORING<br/>(Điểm số + Set + Trạng thái)

    Note over BTC,TV: === ĐANG ĐẤU ===

    loop Mỗi điểm
        REF->>SYS: Tap +1 điểm (Team A hoặc B)
        SYS-->>REF: Cập nhật UI ngay (optimistic)
        SYS->>TV: Cập nhật điểm real-time<br/>(animation flash)
    end

    opt Sai điểm
        REF->>SYS: Bấm UNDO
        SYS->>TV: Cập nhật điểm (hoàn tác)
    end

    Note over SYS: Auto-detect hết set<br/>(đạt điểm quy định + deuce)

    SYS->>REF: Thông báo hết set → Chuyển set mới
    SYS->>TV: Cập nhật set score

    Note over BTC,TV: === KẾT THÚC TRẬN ===

    SYS->>REF: Confirm kết thúc trận
    REF->>SYS: Xác nhận
    SYS->>TV: Hiển thị KẾT QUẢ<br/>(Team thắng + Score + Avatar)

    Note over SYS: Sau 10 giây

    SYS->>TV: Chuyển về IDLE<br/>hoặc WARM-UP trận tiếp
```

---

## 5. TV Display — State Machine

```mermaid
stateDiagram-v2
    [*] --> IDLE: TV mở URL sân

    IDLE --> WARMUP: Match assigned<br/>to court
    WARMUP --> LIVE: Trọng tài bấm<br/>"Bắt đầu"
    LIVE --> RESULT: Match completed
    RESULT --> WARMUP: Trận tiếp theo<br/>đã sẵn sàng
    RESULT --> IDLE: Không còn trận

    IDLE --> CEREMONY: BTC kích hoạt<br/>lễ trao giải
    CEREMONY --> [*]: Giải kết thúc

    state IDLE {
        [*] --> ShowLogo
        ShowLogo: 🏸 Logo giải + "Chờ trận tiếp theo"
    }

    state WARMUP {
        [*] --> ShowPlayers
        ShowPlayers: 👤 Avatar + Tên + Level VĐV
        ShowPlayers --> ShowReferee
        ShowReferee: 🧑‍⚖️ Thông tin trọng tài
        ShowReferee --> ShowMatchInfo
        ShowMatchInfo: 📋 Nội dung + Vòng + Thời gian
    }

    state LIVE {
        [*] --> ShowScore
        ShowScore: 📊 Điểm số real-time + Set
        ShowScore --> ScoreAnimation: Có điểm mới
        ScoreAnimation: ✨ Flash animation
        ScoreAnimation --> ShowScore
    }

    state RESULT {
        [*] --> ShowWinner
        ShowWinner: 🏆 Team thắng + Score chi tiết
        ShowWinner --> AutoTransition
        AutoTransition: ⏱️ Đợi 10 giây
    }

    state CEREMONY {
        [*] --> ShowStandings
        ShowStandings: 📊 BXH từng nội dung
        ShowStandings --> ShowChampions
        ShowChampions: 🥇🥈🥉 Top 3 + Animation
        ShowChampions --> Confetti
        Confetti: 🎊 Confetti effect
    }
```

---

## 6. Tổng Quan Features

### 6.1 Nội Dung Thi Đấu (5 nội dung BWF)

| # | Nội Dung | Loại | Ghi Chú |
|---|----------|------|---------|
| 1 | Đơn Nam | Singles | 1v1 |
| 2 | Đơn Nữ | Singles | 1v1 |
| 3 | Đôi Nam | Doubles | 2v2, đăng ký theo cặp |
| 4 | Đôi Nữ | Doubles | 2v2, đăng ký theo cặp |
| 5 | Đôi Nam Nữ | Mixed Doubles | 2v2, đăng ký theo cặp |

### 6.2 Thể Thức Hỗ Trợ

| Thể Thức | Mô Tả | Phù Hợp |
|-----------|--------|----------|
| Vòng bảng (Round Robin) | Mọi đội trong bảng đấu nhau | Giải phong trào, ít đội |
| Loại trực tiếp (Single Elimination) | Thua = bị loại | Giải lớn, nhanh gọn |
| Kết hợp (Group + Knockout) | Vòng bảng → Vòng loại trực tiếp | Phổ biến nhất |

### 6.3 Luật Tính Điểm (Tùy Chỉnh)

| Cấu Hình | Tùy Chọn | Mặc Định |
|-----------|----------|----------|
| Điểm / set | 11, 15, 21 | 21 (BWF) |
| Số set | 1, 3, 5 | 3 (Best of 3) |
| Deuce | Bật / Tắt | Bật |
| Điểm tối đa (deuce) | 25, 30, không giới hạn | 30 (BWF) |

### 6.4 Feature Map

```mermaid
mindmap
  root((🏸 Tournament<br/>Management))
    📝 Tạo Giải
      Thông tin cơ bản
      Chọn nội dung
      Cấu hình luật
      Cấu hình sân & phí
    👥 Đăng Ký
      BTC nhập VĐV
      Avatar upload
      Trình độ tự khai
      BTC duyệt trình độ
      Thanh toán thủ công
        Mark Paid/Unpaid
    🎯 Bốc Thăm
      Auto chia bảng
      Manual seeding
      Bracket generation
      Drag & drop
    📅 Lịch Thi Đấu
      Phân sân
      Phân thời gian
      Auto-schedule
      Phân công trọng tài
    ⚡ Ghi Điểm Live
      Tap-to-score mobile
      Undo / Pause
      Auto detect hết set
      Optimistic update
    📺 TV Display
      Warm-up screen
      Live scoreboard
      Kết quả trận
      Lễ trao giải
      Auto-flow theo state
    📊 Kết Quả
      BXH theo nội dung
      Thống kê trận đấu
      Export kết quả
```

---

## 7. Kiến Trúc Kỹ Thuật (High-Level)

```mermaid
graph LR
    subgraph "Frontend (Next.js)"
        BTC_UI["📋 BTC Dashboard<br/>(Laptop/Tablet)"]
        REF_UI["🧑‍⚖️ Referee Scoring<br/>(Mobile)"]
        TV_UI["📺 TV Display<br/>(Browser on TV)"]
        ADMIN_UI["🔐 Admin Panel<br/>(Laptop)"]
    end

    subgraph "Mock Layer (MSW)"
        MSW["Mock Service Worker<br/>Simulates API"]
    end

    subgraph "Backend (Future - Team BE)"
        API["REST API"]
        DB["PostgreSQL<br/>(Supabase)"]
        RT["Realtime Service<br/>(Supabase Realtime)"]
    end

    BTC_UI --> MSW
    REF_UI --> MSW
    TV_UI --> MSW
    ADMIN_UI --> MSW

    MSW -.->|"Thay thế bằng API thật"| API
    API --> DB
    API --> RT
    RT -.->|"Push updates"| TV_UI
    RT -.->|"Push updates"| BTC_UI

    style MSW fill:#fff3e0,stroke:#e65100
    style API fill:#e3f2fd,stroke:#1565c0
```

---

## 8. Lộ Trình Phát Triển

```mermaid
gantt
    title Lộ Trình Phát Triển Tournament Module
    dateFormat  YYYY-MM-DD
    axisFormat  %d/%m

    section Phase 1: Foundation
    MSW Setup & Mock Data           :p1a, 2026-02-10, 3d
    Tournament CRUD UI              :p1b, after p1a, 4d
    Data Models & Types             :p1c, 2026-02-10, 2d
    Routing Structure               :p1d, after p1c, 2d

    section Phase 2: Registration
    Registration Management UI      :p2a, after p1b, 4d
    Payment Status Tracking         :p2b, after p2a, 2d
    Draw & Seeding Interface        :p2c, after p2b, 5d
    Bracket Visualization           :p2d, after p2c, 4d

    section Phase 3: Referee Scoring
    Referee Account Management      :p3a, after p2d, 3d
    Tap-to-Score Interface          :p3b, after p3a, 5d
    Score Validation Logic          :p3c, after p3b, 3d
    Match State Machine             :p3d, after p3c, 2d

    section Phase 4: TV Display
    TV Auto-Flow State Machine      :p4a, after p3d, 3d
    Warm-up & Live Score Screens    :p4b, after p4a, 4d
    Result & Ceremony Screens       :p4c, after p4b, 3d
    Animations & Polish             :p4d, after p4c, 3d

    section Phase 5: Dashboard
    BTC Live Overview               :p5a, after p4d, 3d
    Match Scheduling UI             :p5b, after p5a, 4d
    Results & Export                 :p5c, after p5b, 3d
    UX Polish & Testing             :p5d, after p5c, 4d
```

---

## 9. Tóm Tắt Cho Lãnh Đạo

| Hạng Mục | Chi Tiết |
|-----------|----------|
| **Mục tiêu** | Nền tảng tổ chức giải cầu lông chuyên nghiệp, real-time |
| **Quy mô** | ≤60 VĐV, 4 sân, 5 nội dung BWF |
| **Thể thức** | Vòng bảng, loại trực tiếp, hoặc kết hợp (tùy chỉnh) |
| **Highlight** | Ghi điểm mobile (tap-to-score) + TV scoreboard real-time |
| **Approach** | FE-first với mock data → BE team build API sau |
| **Ngôn ngữ** | Tiếng Việt (MVP), có thể mở rộng đa ngôn ngữ cho SaaS |
| **Thanh toán** | Thủ công (chuyển khoản → BTC xác nhận), auto payment phase sau |
| **Tiềm năng** | Sản phẩm nội bộ → SaaS cho các CLB/giải cầu lông toàn quốc |

---

*Ventra Rocket Ranking — Tournament Management System Proposal v1.0*
