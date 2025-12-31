Tentu, ini adalah draf **README.md** yang sangat lengkap, profesional, dan mencakup semua fitur teknis yang telah kita bangun (Next.js, Supabase, Leaflet, DnD Kit, dan Trigger Database).

Anda bisa menyalin kode di bawah ini dan menempelkannya ke file `README.md` di root project Anda.

---

```markdown
# 🚚 FieldSync - Logistics & Courier Management System

**FieldSync** adalah platform manajemen logistik modern yang dirancang untuk memantau pengiriman jarak jauh (last-mile delivery). Aplikasi ini memungkinkan Admin (Dispatcher) untuk mengelola tugas pengiriman, memantau lokasi kurir secara real-time, dan mengatur rute pengiriman menggunakan antarmuka peta interaktif.

![FieldSync Banner](https://via.placeholder.com/1200x400?text=FieldSync+Dashboard+Preview)
*(Ganti link gambar di atas dengan screenshot aplikasi Anda nanti)*

## 🌟 Fitur Utama

### 🖥️ Untuk Admin (Dispatcher)
- **Live Map Monitoring**: Memantau lokasi semua paket dan kurir dalam satu peta interaktif.
- **Dispatch Board (Drag & Drop)**: Mengelola penugasan kurir dengan sistem papan Kanban (Drag & Drop) menggunakan `@dnd-kit`.
- **Manajemen Paket**:
  - Input paket baru dengan **Location Picker** (integrasi OpenStreetMap).
  - Otomatis menghitung jarak dari Warehouse ke lokasi tujuan.
- **Manajemen Kurir**: Melihat status online/offline kurir.
- **Pengaturan Warehouse**: Menentukan titik pusat gudang untuk kalkulasi jarak otomatis.

### 📱 Sistem Cerdas (Backend & Database)
- **Kalkulasi Jarak Otomatis**: Menggunakan **PostgreSQL Trigger** dan rumus **Haversine** untuk menghitung jarak paket secara otomatis saat data diinput.
- **Real-time Updates**: Data sinkronisasi menggunakan React Query.
- **Role-Based Access**: Login khusus untuk Admin.

---

## 🛠️ Tech Stack

Project ini dibangun menggunakan teknologi terkini:

- **Frontend Framework**: [Next.js 14](https://nextjs.org/) (App Router)
- **Language**: [TypeScript](https://www.typescriptlang.org/)
- **Styling**: [Tailwind CSS](https://tailwindcss.com/) & [Shadcn UI](https://ui.shadcn.com/)
- **State Management**: [TanStack Query (React Query)](https://tanstack.com/query/latest)
- **Maps**: [React Leaflet](https://react-leaflet.js.org/) & OpenStreetMap
- **Drag & Drop**: [@dnd-kit](https://dndkit.com/)
- **Backend & Database**: [Supabase](https://supabase.com/) (PostgreSQL, Auth, Realtime)
- **Icons**: [Lucide React](https://lucide.dev/)

---

## 🚀 Cara Menjalankan Project (Local Development)

Ikuti langkah-langkah ini untuk menjalankan project di komputer Anda.

### 1. Clone Repository
```bash
git clone [https://github.com/username-anda/fieldsync.git](https://github.com/username-anda/fieldsync.git)
cd fieldsync

```

### 2. Install Dependencies

```bash
npm install
# atau
yarn install

```

### 3. Konfigurasi Environment Variables

Buat file `.env.local` di root folder dan isi dengan kredensial Supabase Anda:

```env
NEXT_PUBLIC_SUPABASE_URL=[https://your-project-id.supabase.co](https://your-project-id.supabase.co)
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key-here

```

### 4. Setup Database (Supabase SQL)

Buka **SQL Editor** di Dashboard Supabase Anda dan jalankan script berikut secara berurutan untuk membuat tabel dan fungsi yang diperlukan.

#### A. Buat Tabel

```sql
-- 1. Tabel Profiles (Extends Supabase Auth)
CREATE TABLE public.profiles (
  id UUID REFERENCES auth.users(id) ON DELETE CASCADE PRIMARY KEY,
  email TEXT,
  full_name TEXT,
  role TEXT DEFAULT 'courier', -- 'admin' or 'courier'
  is_online BOOLEAN DEFAULT false,
  latitude FLOAT,
  longitude FLOAT,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- 2. Tabel Warehouse (Gudang Utama)
CREATE TABLE public.warehouse (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name TEXT,
  address TEXT,
  latitude FLOAT,
  longitude FLOAT
);

-- 3. Tabel Tasks (Paket)
CREATE TABLE public.tasks (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  recipient_name TEXT NOT NULL,
  latitude FLOAT NOT NULL,
  longitude FLOAT NOT NULL,
  status TEXT DEFAULT 'pending', -- pending, assigned, delivered
  assigned_to UUID REFERENCES public.profiles(id),
  distance_from_warehouse FLOAT, -- Diisi otomatis oleh Trigger
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

```

#### B. Buat Fungsi Logika (Jarak & Trigger)

```sql
-- 1. Rumus Haversine (Hitung KM)
CREATE OR REPLACE FUNCTION public.calculate_distance_km(
    lat1 float, lon1 float, 
    lat2 float, lon2 float
)
RETURNS float AS $$
DECLARE
    R float := 6371;
    dLat float := radians(lat2 - lat1);
    dLon float := radians(lon2 - lon1);
    a float;
    c float;
BEGIN
    lat1 := radians(lat1);
    lat2 := radians(lat2);
    a := sin(dLat/2)^2 + cos(lat1) * cos(lat2) * sin(dLon/2)^2;
    c := 2 * asin(sqrt(a));
    RETURN R * c;
END;
$$ LANGUAGE plpgsql;

-- 2. Trigger Function
CREATE OR REPLACE FUNCTION public.update_task_distance_trigger()
RETURNS TRIGGER AS $$
DECLARE
    wh_lat float;
    wh_lon float;
BEGIN
    SELECT latitude, longitude INTO wh_lat, wh_lon FROM public.warehouse LIMIT 1;
    IF wh_lat IS NOT NULL THEN
        NEW.distance_from_warehouse := public.calculate_distance_km(wh_lat, wh_lon, NEW.latitude, NEW.longitude);
    END IF;
    RETURN NEW;
END;
$$ LANGUAGE plpgsql;

-- 3. Pasang Trigger ke Tabel Tasks
CREATE TRIGGER set_distance_on_task_change
    BEFORE INSERT OR UPDATE OF latitude, longitude
    ON public.tasks
    FOR EACH ROW
    EXECUTE FUNCTION public.update_task_distance_trigger();

```

### 5. Jalankan Aplikasi

```bash
npm run dev

```

Buka [http://localhost:3000](https://www.google.com/search?q=http://localhost:3000) di browser Anda.

---

## 📖 Panduan Penggunaan

### 1. Membuat Akun Admin

Karena halaman pendaftaran (Sign Up) ditutup untuk umum, Anda harus membuat user admin secara manual lewat Supabase Dashboard:

1. Masuk ke menu **Authentication** -> **Users** -> **Add User**.
2. Copy **User ID (UUID)** user baru tersebut.
3. Jalankan SQL ini:
```sql
INSERT INTO public.profiles (id, email, full_name, role)
VALUES ('UUID_USER_ANDA', 'email@anda.com', 'Super Admin', 'admin');

```



### 2. Mengatur Lokasi Gudang

1. Login ke aplikasi.
2. Masuk ke menu **Settings**.
3. Cari lokasi gudang di peta atau ketik nama jalan, lalu klik **Simpan**.
4. Semua paket baru akan dihitung jaraknya dari titik ini.

### 3. Dispatching (Mengirim Paket)

1. Masuk ke **Manajemen Paket** untuk input paket baru (pilih lokasi di peta).
2. Masuk ke **Dispatch Board**.
3. Drag paket dari kolom **Pending** ke kolom **Kurir** yang tersedia.
4. Total jarak tempuh kurir akan terhitung otomatis.

---

## 📂 Struktur Folder

```
src/
├── app/                  # App Router Next.js
│   ├── (auth)/           # Route Login
│   ├── (dashboard)/      # Route Admin (Map, Tasks, Dispatch)
│   └── layout.tsx        # Root Layout
├── components/
│   ├── dispatch/         # Komponen Kanban Board (DnD)
│   ├── layout/           # Sidebar & Header
│   ├── maps/             # Komponen Leaflet & LocationPicker
│   └── ui/               # Shadcn UI Components
├── hooks/                # Custom React Hooks (useTasks, useCouriers)
├── lib/                  # Konfigurasi Supabase & Utils
├── services/             # API Calls ke Supabase
└── types/                # TypeScript Interfaces

```

---

## 🤝 Kontribusi

Kontribusi sangat diterima! Silakan fork repository ini dan buat Pull Request untuk fitur baru atau perbaikan bug.

1. Fork Project
2. Buat Feature Branch (`git checkout -b feature/AmazingFeature`)
3. Commit Perubahan (`git commit -m 'Add some AmazingFeature'`)
4. Push ke Branch (`git push origin feature/AmazingFeature`)
5. Buka Pull Request

---

## 📄 Lisensi

Didistribusikan di bawah Lisensi MIT. Lihat `LICENSE` untuk informasi lebih lanjut.

```

```
