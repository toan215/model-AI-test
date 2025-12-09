# Tài Liệu Thiết Kế Phần Mềm (SDS)

## Bynce: Công Cụ AI Tạo Tư Thế Cho Mô Hình MMD

**Phiên bản:** 0.0.0  
**Ngày:** 8 tháng 12, 2025  
**Tên dự án:** Bynce (tên cũ: MiKaPo)

---

## 1. Tổng Quan Hệ Thống

### 1.1 Mục Đích

Bynce là ứng dụng web cho phép điều khiển tư thế của mô hình 3D MMD (MikuMikuDance) theo thời gian thực từ video đầu vào. Hệ thống sử dụng AI để phát hiện tư thế người từ nhiều nguồn đầu vào (video, ảnh, webcam) và áp dụng tư thế đó lên nhân vật 3D.

### 1.2 Phạm Vi

Ứng dụng cung cấp:

- Phát hiện tư thế 3D theo thời gian thực sử dụng Mediapipe
- Xem và điều khiển mô hình MMD bằng Babylon.js
- Hỗ trợ nhiều nguồn đầu vào (video, ảnh, webcam)
- Ghi lại chuyển động và xuất file VMD
- Tùy chỉnh mô hình (vật liệu, khung xương, hoạt ảnh)
- Môi trường nền 360 độ
- Khả năng Progressive Web App (PWA)

### 1.3 Đối Tượng Người Dùng

- Nhà làm phim hoạt hình 3D và người sáng tạo nội dung
- Người đam mê MMD
- Người yêu thích motion capture
- Nghệ sĩ kỹ thuật số làm việc với nhân vật 3D

---

## 2. Kiến Trúc Hệ Thống

### 2.1 Kiến Trúc Tổng Quan

Hệ thống bao gồm các thành phần chính:

**Lớp Ứng Dụng Client:**

- **Giao diện React**: Quản lý UI và tương tác người dùng
- **Babylon.js 3D Scene**: Render và hiển thị mô hình 3D
- **Mediapipe**: Phát hiện tư thế từ video/ảnh
- **WASM Pose Solver**: Tính toán quaternion cho xương

**Nguồn Đầu Vào:**

- Video files (MP4, WebM, v.v.)
- Image files (JPG, PNG, v.v.)
- Webcam stream

**Tài Nguyên Bên Ngoài:**

- Mô hình MMD (PMX format)
- Hoạt ảnh VMD
- Hình nền 360°

### 2.2 Công Nghệ Sử Dụng

| Lớp                    | Công Nghệ        | Phiên Bản | Mục Đích                     |
| ---------------------- | ---------------- | --------- | ---------------------------- |
| **Framework Frontend** | React            | 18.3.1    | Quản lý component UI         |
| **Build Tool**         | Vite             | 5.4.1     | Phát triển và đóng gói nhanh |
| **Ngôn Ngữ**           | TypeScript       | 5.5.3     | Phát triển an toàn kiểu      |
| **3D Engine**          | Babylon.js       | 7.27.0    | Render 3D scene              |
| **MMD Renderer**       | babylon-mmd      | 0.55.0    | Hỗ trợ mô hình MMD           |
| **Phát Hiện Tư Thế**   | Mediapipe        | 0.10.15   | Theo dõi tư thế AI           |
| **Pose Solver**        | Rust + WASM      | Custom    | Tính toán quaternion         |
| **UI Framework**       | Material-UI      | 6.1.1     | Thư viện component           |
| **Icons**              | Font Awesome     | 6.6.0     | Hệ thống icon                |
| **Analytics**          | Vercel Analytics | 1.3.1     | Theo dõi sử dụng             |

### 2.3 Mô Hình Kiến Trúc

#### 2.3.1 Kiến Trúc Component

Ứng dụng tuân theo kiến trúc component của React với phân tách rõ ràng:

- **Component Hiển Thị**: Các phần tử UI (Header, Footer, Drawer)
- **Component Container**: Các module tính năng (Motion, Model, Animation, Skeleton, Materials, Background)
- **Component 3D Scene**: MMDScene là engine render 3D chính

#### 2.3.2 Quản Lý State

- React hooks (`useState`, `useEffect`, `useRef`) cho quản lý state cục bộ
- Props drilling để truyền dữ liệu cha-con
- Callback functions để cập nhật state từ con lên cha

#### 2.3.3 Mô Hình Module

- Tách biệt rõ ràng giữa logic UI và render 3D
- Module WASM độc lập cho tính toán tư thế hiệu năng cao
- Thiết kế component module hóa để dễ bảo trì

---

## 3. Đặc Tả Component

### 3.1 Component Chính

#### 3.1.1 App Component (`App.tsx`)

**Trách nhiệm**: Component gốc và điều phối state

**Biến State Chính**:

- `body`: Dữ liệu điểm mốc tư thế
- `lerpFactor`: Hệ số nội suy
- `selectedModel`: Mô hình MMD hiện tại
- `selectedBackground`: Môi trường nền
- `selectedAnimation`: Hoạt ảnh hiện tại
- `isPlaying`: Trạng thái phát hoạt ảnh
- `boneRotation`: Xoay xương thủ công
- `materials`: Danh sách vật liệu mô hình
- `materialVisible`: Trạng thái hiển thị vật liệu

**Tính năng**:

- Màn hình chào mừng tự động tắt
- Hệ thống điều hướng drawer
- Lazy loading component Motion
- Đồng bộ state giữa các component

#### 3.1.2 MMDScene Component (`MMDScene.tsx`)

**Trách nhiệm**: Quản lý và render 3D scene

**Nhiệm vụ chính**:

- Khởi tạo và quản lý Babylon.js scene
- Load và render mô hình MMD
- Áp dụng tư thế lên khung xương mô hình
- Điều khiển phát hoạt ảnh
- Chức năng import/export VMD
- Chụp ảnh màn hình
- Ghi lại chuyển động

**Phương thức chính**:

- `createScene()`: Khởi tạo Babylon.js scene
- `loadMMD()`: Load mô hình MMD
- `loadAnimation()`: Load hoạt ảnh VMD
- `updateMMDPose()`: Áp dụng tư thế lên mô hình
- `recordFrame()`: Ghi frame chuyển động
- `createVMD()`: Xuất file VMD
- `handleCaptureScreenshot()`: Chụp ảnh màn hình

**Component 3D Scene**:

- ArcRotateCamera với phương thức `spinTo` tùy chỉnh
- Ánh sáng directional và hemispheric
- PhotoDome cho nền 360°
- Ground mesh với vật liệu tùy chỉnh

#### 3.1.3 Motion Component (`Motion.tsx`)

**Trách nhiệm**: Đầu vào video/ảnh và phát hiện tư thế

**Tính năng chính**:

- Hỗ trợ upload file (video/ảnh)
- Tích hợp webcam
- Tích hợp Mediapipe Holistic Landmarker
- Phát hiện tư thế theo thời gian thực
- Scene debug visualization

**Quy trình phát hiện tư thế**:

1. Đầu vào (Video/Ảnh/Webcam)
2. Mediapipe Holistic xử lý
3. Trích xuất landmarks: Body (33 điểm), Hands (21 điểm mỗi tay), Face (478 điểm)
4. Cập nhật state ứng dụng

#### 3.1.4 Component UI Khác

| Component  | File             | Mục Đích                          |
| ---------- | ---------------- | --------------------------------- |
| Header     | `Header.tsx`     | Header và branding ứng dụng       |
| Footer     | `Footer.tsx`     | Thanh công cụ điều hướng với tabs |
| Model      | `Model.tsx`      | Giao diện chọn mô hình            |
| Background | `Background.tsx` | Chọn môi trường nền               |
| Animation  | `Animation.tsx`  | Điều khiển phát hoạt ảnh          |
| Skeleton   | `Skeleton.tsx`   | Điều khiển xoay xương thủ công    |
| Materials  | `Materials.tsx`  | Bật/tắt hiển thị vật liệu         |
| DebugScene | `DebugScene.tsx` | Hiển thị landmarks tư thế         |

### 3.2 WASM Pose Solver (`pose_solver`)

**Ngôn ngữ**: Rust  
**Mục tiêu biên dịch**: WebAssembly  
**Mục đích**: Tính toán pose-to-quaternion hiệu năng cao

**Cấu trúc dữ liệu chính**:

- `Rotation`: Quaternion (x, y, z, w)
- `PoseSolverResult`: Chứa tất cả rotations cho:
  - Thân trên/dưới, cổ
  - Hông trái/phải, chân
  - Cánh tay trên/dưới, cổ tay
  - Tất cả khớp ngón tay (cả 2 tay)
  - Xoay mắt và độ mở mắt
  - Độ mở miệng

**Thuật toán chính**:

- Tính toán rotation dựa trên quaternion (sử dụng `nalgebra`)
- Tính toán hướng Vector3
- Biến đổi phân cấp xương
- Chuyển đổi không gian local/world
- Theo dõi hướng nhìn mắt
- Ánh xạ biểu cảm khuôn mặt

**Chỉ số Landmarks**:

- `MainBodyIndex`: 33 landmarks cơ thể (Mediapipe pose)
- `HandIndex`: 21 landmarks mỗi tay
- `FaceIndex`: Chỉ số landmarks khuôn mặt

---

## 4. Mô Hình Dữ Liệu

### 4.1 Định Nghĩa Kiểu

**Cấu trúc dữ liệu tư thế chính**:

- `Body`: Chứa landmarks cho cơ thể (33 điểm), tay trái (21 điểm), tay phải (21 điểm), và khuôn mặt (478 điểm)

**Cấu trúc ghi chuyển động**:

- `RecordedFrame`: Chứa danh sách BoneFrame và MorphFrame
- `BoneFrame`: Tên xương, rotation (Quaternion), position (Vector3)
- `MorphFrame`: Tên morph, trọng số

### 4.2 Định Dạng Mediapipe Landmark

Mỗi landmark có:

- `x`: Tọa độ X chuẩn hóa [0-1]
- `y`: Tọa độ Y chuẩn hóa [0-1]
- `z`: Độ sâu chuẩn hóa
- `visibility`: Độ tin cậy (tùy chọn)

---

## 5. Thiết Kế Giao Diện Người Dùng

### 5.1 Cấu Trúc Layout

```
┌─────────────────────────────────────────┐
│  Header (Tiêu đề)                       │
├─────────────────────────────────────────┤
│                                          │
│         3D Scene (Cảnh 3D)              │
│         (MMDScene Component)            │
│                                          │
├─────────────────────────────────────────┤
│  Footer (Thanh điều hướng)              │
└─────────────────────────────────────────┘

Drawer (Trượt từ trái):
┌──────────────┐
│ Motion       │
│ - Upload     │
│ - Webcam     │
│ - Video      │
│              │
│ hoặc Model   │
│ hoặc Animation│
│ hoặc Skeleton│
│ hoặc Materials│
│ hoặc Background│
└──────────────┘
```

### 5.2 Tabs Điều Hướng (Footer)

Footer chứa các tab để chuyển đổi giữa các tính năng:

- **Motion**: Đầu vào video/ảnh và phát hiện tư thế
- **Model**: Chọn mô hình MMD
- **Animation**: Phát hoạt ảnh VMD
- **Skeleton**: Điều chỉnh xương thủ công
- **Materials**: Điều khiển hiển thị vật liệu
- **Background**: Chọn môi trường

### 5.3 Màn Hình Chào Mừng

Màn hình splash ban đầu hiển thị:

- Branding ứng dụng ("Welcome to Mikiu!")
- Mô tả ngắn gọn
- Nút "Enter" để tiếp tục

**Tự động tắt**: 6 giây với hiệu ứng fade-out

---

## 6. Tính Năng Chính và Quy Trình

### 6.1 Quy Trình Phát Hiện Tư Thế

1. **Người dùng** upload video/ảnh hoặc bật webcam
2. **Motion component** gửi frames video đến Mediapipe
3. **Mediapipe** trả về landmarks (cơ thể, tay, mặt)
4. **Motion** cập nhật state body trong App
5. **PoseSolver** giải tư thế (landmarks → quaternions)
6. **PoseSolver** trả về rotations
7. **MMDScene** áp dụng rotations lên mô hình
8. **Người dùng** thấy mô hình với tư thế mới

### 6.2 Quy Trình Load Mô Hình

1. Người dùng chọn mô hình từ panel Model
2. `setSelectedModel` cập nhật state
3. MMDScene nhận prop thay đổi
4. Phương thức `loadMMD()` được kích hoạt
5. Mô hình được load qua `babylon-mmd` PMX loader
6. MmdWasmRuntime được khởi tạo
7. Mô hình được thêm vào scene
8. Camera điều chỉnh theo mô hình

### 6.3 Quy Trình Xuất VMD

1. Người dùng bắt đầu ghi qua nút toolbar
2. Mỗi frame: `recordFrame()` ghi dữ liệu xương/morph
3. Dữ liệu tích lũy trong mảng `recordedFrames`
4. Người dùng dừng ghi
5. `createVMD()` tạo file VMD binary
6. Encoding.js chuyển đổi chuỗi sang Shift-JIS
7. File được tải xuống qua blob URL

### 6.4 Phát Hoạt Ảnh

1. Người dùng chọn file hoạt ảnh VMD
2. File được parse bởi babylon-mmd
3. Hoạt ảnh load vào MmdWasmRuntime
4. Điều khiển play/pause cập nhật state `isPlaying`
5. Thời gian hoạt ảnh đồng bộ với `setCurrentAnimationTime`
6. Thanh seek cho phép tua frame

---

## 7. Dependencies (Thư Viện Phụ Thuộc)

### 7.1 Dependencies Chính

- **@babylonjs/core** (7.27.0): Engine render 3D
- **@mediapipe/tasks-vision** (0.10.15): Phát hiện tư thế
- **babylon-mmd** (0.55.0): Hỗ trợ mô hình MMD
- **encoding-japanese** (2.2.0): Mã hóa Shift-JIS
- **pose_solver**: Module WASM pose solver tùy chỉnh
- **react** (18.3.1): Framework UI
- **react-dom** (18.3.1): React DOM

### 7.2 Build Dependencies

- **vite** (5.4.1): Công cụ build
- **vite-plugin-wasm** (3.3.0): Hỗ trợ WASM
- **vite-plugin-top-level-await** (1.4.4): Load WASM async
- **vite-plugin-pwa** (0.20.5): Hỗ trợ PWA
- **@vitejs/plugin-react** (4.3.1): Plugin React

### 7.3 UI Dependencies

- **@mui/material** (6.1.1): Component Material-UI
- **@mui/icons-material** (6.1.1): Icon Material
- **@emotion/react** (11.13.3): CSS-in-JS
- **@fortawesome/react-fontawesome** (0.2.2): Icon Font Awesome
- **@fontsource/roboto** (5.1.0): Font Roboto

---

## 8. Cấu Trúc File

```
Bynce/
├── src/
│   ├── App.tsx                    # Component gốc
│   ├── main.tsx                   # Entry point
│   ├── MMDScene.tsx               # Quản lý 3D scene
│   ├── Motion.tsx                 # Component phát hiện tư thế
│   ├── Model.tsx                  # Chọn mô hình
│   ├── Animation.tsx              # Điều khiển hoạt ảnh
│   ├── Skeleton.tsx               # Điều chỉnh xương
│   ├── Materials.tsx              # Điều khiển vật liệu
│   ├── Background.tsx             # Chọn nền
│   ├── Header.tsx                 # Header app
│   ├── Footer.tsx                 # Footer điều hướng
│   ├── DebugScene.tsx             # Debug visualization
│   ├── index.d.ts                 # Định nghĩa TypeScript
│   └── assets/                    # Tài nguyên tĩnh
│
├── pose_solver/                   # Module Rust WASM
│   ├── src/
│   │   └── lib.rs                # Triển khai pose solver
│   ├── Cargo.toml                # Dependencies Rust
│   └── pkg/                      # Output WASM đã biên dịch
│
├── public/
│   ├── animation/                # File hoạt ảnh VMD
│   ├── avatar/                   # File mô hình MMD
│   ├── background/               # Hình nền 360°
│   └── logo.png                  # Logo app
│
├── index.html                    # HTML entry point
├── vite.config.ts                # Cấu hình Vite
├── tsconfig.json                 # Cấu hình TypeScript
├── package.json                  # NPM dependencies
└── README.md                     # Tài liệu dự án
```

---

## 9. Cấu Hình

### 9.1 Cấu Hình Vite

Plugins được sử dụng:

- `react()`: Hỗ trợ React
- `wasm()`: Hỗ trợ module WASM
- `topLevelAwait()`: Hỗ trợ top-level await
- `VitePWA()`: Progressive Web App với auto-update

### 9.2 Cấu Hình TypeScript

- **Target**: ES2020
- **Module System**: ESNext với bundler resolution
- **JSX**: react-jsx
- **Strict Mode**: Bật
- **Linting**: Bật (noUnusedLocals, noUnusedParameters, noFallthroughCasesInSwitch)

---

## 10. Tối Ưu Hiệu Năng

### 10.1 Chiến Lược Tối Ưu

1. **WASM cho Code Quan Trọng về Hiệu Năng**

   - Tính toán tư thế được chuyển sang Rust/WASM
   - Toán quaternion trong code đã biên dịch (~10x nhanh hơn JS)

2. **Lazy Loading**

   - Motion component chỉ mount khi được truy cập lần đầu
   - Giảm kích thước bundle ban đầu

3. **Tăng Tốc GPU**

   - Babylon.js sử dụng WebGL để render
   - Khuyến nghị sử dụng GPU chuyên dụng

4. **Cập Nhật State Hiệu Quả**

   - Giảm thiểu re-render qua cập nhật state có mục tiêu
   - Sử dụng `useRef` cho giá trị không trigger re-render

5. **Lập Lịch Animation Frame**
   - `requestAnimationFrame` cho cập nhật tư thế mượt mà
   - Ngăn tính toán dư thừa

### 10.2 Chỉ Số Hiệu Năng

- **Initial Load**: Vite HMR nhanh cho development
- **3D Rendering**: Mục tiêu 60 FPS với WebGL
- **Phát Hiện Tư Thế**: ~30 FPS (xử lý Mediapipe)
- **Thực Thi WASM**: < 1ms mỗi frame cho pose solving

---

## 11. Tương Thích Trình Duyệt

### 11.1 Trình Duyệt Hỗ Trợ

- Chrome/Edge: 90+
- Firefox: 90+
- Safari: 14+

### 11.2 Tính Năng Yêu Cầu

- WebGL 2.0
- WebAssembly
- getUserMedia API (cho webcam)
- FileReader API
- JavaScript ES2020 hiện đại

### 11.3 Progressive Web App

- Service worker cho hỗ trợ offline
- Có thể cài đặt trên desktop và mobile
- Tự động cập nhật phiên bản mới

---

## 12. Kiến Trúc Triển Khai

### 12.1 Quy Trình Build

```bash
# Development
npm run dev          # Vite dev server với HMR

# Production
npm run build        # Biên dịch TypeScript + Vite build
npm run preview      # Xem trước production build
```

### 12.2 Nền Tảng Triển Khai

**Chính**: Vercel  
**Cấu hình**: `vercel.json`

### 12.3 Tài Nguyên Tĩnh

- Mô hình, hoạt ảnh, nền được phục vụ từ `/public`
- CDN delivery cho Mediapipe WASM files
- Tích hợp Vercel Analytics

---

## 13. Cân Nhắc Bảo Mật

### 13.1 Xác Thực Đầu Vào

- Xác thực loại file cho uploads (chỉ video/ảnh)
- Xử lý CORS cho tài nguyên bên ngoài
- Xử lý blob URL an toàn

### 13.2 Dependencies Bên Thứ Ba

- Cập nhật dependencies thường xuyên qua `npm audit`
- Nguồn tin cậy: Mediapipe (Google), Babylon.js (Microsoft)
- WASM sandboxing cho pose solver

### 13.3 Quyền Riêng Tư Dữ Liệu

- Tất cả xử lý ở phía client
- Không truyền dữ liệu lên server (trừ analytics)
- Truy cập webcam yêu cầu quyền người dùng

---

## 14. Cải Tiến Tương Lai

### 14.1 Tính Năng Đã Hoàn Thành

- ✅ Phát hiện tư thế
- ✅ Phát hiện khuôn mặt
- ✅ Phát hiện tay (thử nghiệm)
- ✅ Rust-WASM pose solver
- ✅ Chọn nền 360°
- ✅ Upload video/ảnh
- ✅ Đầu vào webcam
- ✅ Chọn mô hình
- ✅ Import/export VMD
- ✅ MMD editor (chỉnh sửa xương, vật liệu, mesh)

### 14.2 Cải Tiến Tiềm Năng

- Phát hiện tư thế nhiều người
- Mô phỏng vật lý cho quần áo/tóc
- Upload mô hình tùy chỉnh
- Lưu trữ đám mây cho dự án
- Tính năng cộng tác
- Phiên bản mobile app

---

## 15. Chiến Lược Kiểm Thử

### 15.1 Kiểm Thử Development

```bash
npm run lint         # Kiểm tra ESLint
```

### 15.2 Checklist Kiểm Thử Thủ Công

- Upload file video → xác minh phát hiện tư thế
- Upload file ảnh → xác minh tư thế tĩnh
- Bật webcam → xác minh theo dõi thời gian thực
- Chuyển mô hình → xác minh load đúng
- Phát hoạt ảnh VMD → xác minh playback
- Ghi chuyển động → xác minh xuất VMD
- Điều chỉnh xoay xương → xác minh điều khiển thủ công
- Bật/tắt vật liệu → xác minh thay đổi hiển thị
- Đổi nền → xác minh môi trường 360°
- Chụp ảnh màn hình → xác minh tải ảnh

### 15.3 Kiểm Thử Trình Duyệt

Kiểm tra trên:

- Chrome (Windows, macOS)
- Firefox (Windows, macOS)
- Safari (macOS)
- Edge (Windows)

---

## 16. Thuật Ngữ

| Thuật Ngữ      | Định Nghĩa                                            |
| -------------- | ----------------------------------------------------- |
| **MMD**        | MikuMikuDance - Phần mềm hoạt ảnh 3D phổ biến ở Nhật  |
| **VMD**        | Vocaloid Motion Data - Định dạng file hoạt ảnh        |
| **PMX**        | Polygon Model eXtended - Định dạng mô hình MMD        |
| **WASM**       | WebAssembly - Định dạng lệnh nhị phân cho trình duyệt |
| **Mediapipe**  | Framework ML của Google để phát hiện tư thế/mặt/tay   |
| **Babylon.js** | Engine 3D mã nguồn mở cho web                         |
| **Quaternion** | Hệ thống số 4D để biểu diễn xoay 3D                   |
| **Landmarks**  | Điểm mốc được phát hiện trên cơ thể/mặt/tay           |
| **Morph**      | Blend shape cho biểu cảm khuôn mặt                    |
| **Bone**       | Khớp xương trong mô hình 3D                           |

---

## 17. Tài Liệu Tham Khảo

- [Tài liệu Mediapipe](https://ai.google.dev/edge/mediapipe/solutions/vision/pose_landmarker/web_js)
- [Tài liệu Babylon.js](https://www.babylonjs.com/)
- [babylon-mmd GitHub](https://github.com/noname0310/babylon-mmd)
- [Tài liệu Vite](https://vitejs.dev/)
- [Đặc tả định dạng VMD](https://mikumikudance.fandom.com/wiki/VMD_file_format)

---

**Phiên bản tài liệu**: 1.0  
**Cập nhật lần cuối**: 8 tháng 12, 2025  
**Người soạn**: Antigravity AI Assistant
