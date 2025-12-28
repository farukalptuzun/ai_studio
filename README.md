# AI Studio Backend

AI destekli reklam görseli ve video üretim platformu. Tek bir ürün görselinden başlayarak, yapay zeka ile farklı reklam varyasyonları üretir ve bunları kısa video reklamlara dönüştürür.

## 🎯 Proje Amacı

AI Studio, kreatif ekiplerin ve reklam platformlarının ürün görsellerinden otomatik olarak profesyonel reklam içerikleri üretmesini sağlar. Sistem:

- **Ürün görselini analiz eder** ve kampanya brief'ine göre dinamik varyasyon fikirleri üretir
- **5 farklı görsel varyasyonu** oluşturur (kullanıcı tercihlerine göre özelleştirilebilir)
- **Seçilen görselleri kısa video reklamlara** dönüştürür
- **Manken entegrasyonu** ile lifestyle görselleri destekler
- **Template sistemi** ile hızlı içerik üretimi sağlar

## 🚀 Hızlı Başlangıç

### Gereksinimler

- Python 3.9+
- MongoDB
- Redis
- Google Gemini API Key
- Nano Banana API Key (görsel üretimi için)
- Google Veo 3 API Key (video üretimi için)

### Kurulum

```bash
# 1. Sanal ortam oluştur
python -m venv venv
source venv/bin/activate  # Windows: venv\Scripts\activate

# 2. Bağımlılıkları yükle
cd backend
pip install -r requirements.txt

# 3. Ortam değişkenlerini ayarla
cp .env.example .env
# .env dosyasını düzenle ve API anahtarlarını ekle

# 4. Uygulamayı başlat
uvicorn src.main:app --reload
```

API dokümantasyonu: `http://localhost:8000/docs`

## 🔄 Pipeline Nasıl Çalışır?

### 1. Proje Oluşturma ve Varyasyon Planlama

```
Kullanıcı → Ürün Görseli + Kampanya Bilgileri
    ↓
LLM (Gemini) → Görsel Analizi + 5 Varyasyon Prompt'u Üretimi
    ↓
Variation Plan → MongoDB'ye Kayıt
```

**Özellikler:**
- Kullanıcı özelleştirmesi: `brand_segment`, `photo_design_description`, `user_custom_prompt`
- LLM, kullanıcının tercihlerine göre ilk 4 varyasyonu şekillendirir
- 5. varyasyon her zaman "ürün elinde" formatındadır
- Her varyasyon için aspect ratio, stil ve detaylı prompt üretilir

### 2. Görsel Üretimi

```
Variation Plan → Her Varyasyon İçin
    ↓
Nano Banana API → Görsel Üretimi (Orijinal ürün görseli referans alınarak)
    ↓
Storage → Görsel Kaydedilir
    ↓
MongoDB → Variation Status: "ready"
```

**Özellikler:**
- Asenkron işlem: Redis + RQ ile job queue
- Orijinal ürün görseli referans olarak kullanılır
- Her varyasyon bağımsız olarak üretilir
- Hata durumunda retry mekanizması

### 3. Video Üretimi

```
Kullanıcı → Varyasyon Seçimi + Video Prompt
    ↓
LLM (Gemini) → Video Planı + Veo 3 Prompt Optimizasyonu
    ↓
Manken Entegrasyonu (Opsiyonel) → Görseller Birleştirilir
    ↓
Veo 3 API → Video Üretimi
    ↓
Storage → Video Kaydedilir
    ↓
MongoDB → Video Status: "ready"
```

**Özellikler:**
- Manken entegrasyonu: Ürün + manken görselleri birleştirilir
- Video prompt optimizasyonu: LLM ile Veo 3 için optimize edilmiş prompt
- Asenkron video üretimi
- Progress tracking

## 🏗️ Mimari

### Katmanlar

```
┌─────────────────────────────────────┐
│   API Layer (FastAPI Routes)        │
│   - projects, variations, video     │
│   - templates, advertisements        │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│   Services Layer                    │
│   - variation_planner               │
│   - image_orchestrator              │
│   - video_planner, video_generator │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│   Integrations Layer                │
│   - gemini_client (LLM)             │
│   - nano_banana (Image Gen)         │
│   - veo_client (Video Gen)          │
└─────────────────────────────────────┘
              ↓
┌─────────────────────────────────────┐
│   Workers (Background Jobs)         │
│   - image_worker                    │
│   - video_worker                    │
└─────────────────────────────────────┘
```

### Klasör Yapısı

```
backend/
├── src/
│   ├── main.py                 # FastAPI uygulama girişi
│   ├── api/routes/            # API endpoint'leri
│   │   ├── projects.py
│   │   ├── variations.py
│   │   ├── video.py
│   │   ├── templates.py
│   │   └── advertisements.py
│   ├── core/                  # Altyapı
│   │   ├── config.py         # Konfigürasyon
│   │   ├── db.py             # MongoDB bağlantısı
│   │   └── queue.py          # Redis + RQ
│   ├── models/               # Pydantic modelleri
│   │   ├── project.py
│   │   ├── variation.py
│   │   └── video.py
│   ├── services/             # İş mantığı
│   │   ├── variation_planner.py
│   │   ├── image_orchestrator.py
│   │   └── video_planner.py
│   ├── integrations/         # Harici API'ler
│   │   ├── gemini_client.py
│   │   ├── nano_banana.py
│   │   └── veo_client.py
│   └── workers/              # Arka plan işleri
│       ├── image_worker.py
│       └── video_worker.py
└── storage/                  # Üretilen görseller/videolar
```

## 🛠️ Kullanılan Teknolojiler

### Backend Framework
- **FastAPI**: Modern, hızlı Python web framework
  - Async/await desteği
  - Otomatik OpenAPI dokümantasyonu
  - Type hints ile tip güvenliği

### Veritabanı
- **MongoDB**: Doküman tabanlı NoSQL veritabanı
  - Esnek şema yapısı
  - Proje, varyasyon ve video kayıtları

### Job Queue
- **Redis**: In-memory veri yapısı
- **RQ (Redis Queue)**: Python job queue
  - Asenkron görsel/video üretimi
  - Background task yönetimi

### AI Servisleri

#### LLM - Gemini
- **Kullanım Alanları:**
  - Ürün görseli analizi
  - Varyasyon prompt'u üretimi (5 farklı)
  - Video planı ve prompt optimizasyonu
- **Özellikler:**
  - Vision model ile görsel analizi
  - Kullanıcı özelleştirmelerine göre prompt şekillendirme
  - Context-aware prompt üretimi

#### Görsel Üretimi - Nano Banana
- **Kullanım:** Varyasyon görsellerinin üretimi
- **Özellikler:**
  - Orijinal ürün görseli referans alınarak üretim
  - Aspect ratio kontrolü
  - Yüksek kaliteli görsel çıktısı

#### Video Üretimi - Veo 3
- **Kullanım:** Kısa video reklam üretimi
- **Özellikler:**
  - Görselden video üretimi
  - Manken entegrasyonu desteği
  - LLM ile optimize edilmiş prompt

### Diğer
- **Pydantic**: Veri validasyonu ve model tanımları
- **Pillow (PIL)**: Görsel işleme
- **Docker**: Containerization (opsiyonel)

## 📡 API Endpoints

### Projeler
- `POST /api/projects` - Yeni proje oluştur
- `GET /api/projects/{project_id}` - Proje detayları
- `GET /api/projects` - Tüm projeleri listele

### Varyasyonlar
- `GET /api/variations/project/{project_id}` - Projeye ait varyasyonları listele
- `GET /api/variations/{variation_id}` - Varyasyon detayları

### Video
- `POST /api/video/generate` - Video üretimi başlat
- `GET /api/video/{video_id}` - Video durumu ve detayları

### Şablonlar
- `GET /api/templates` - Mevcut şablonları listele
- `POST /api/templates` - Yeni şablon oluştur

### Reklamlar
- `GET /api/advertisements` - Reklam kampanyalarını listele
- `POST /api/advertisements` - Yeni reklam kampanyası oluştur

## 🔧 Konfigürasyon

`.env` dosyasında ayarlanması gereken değişkenler:

```env
# MongoDB
MONGODB_URI=mongodb://localhost:27017
MONGODB_DB_NAME=ai_studio

# Redis
REDIS_HOST=localhost
REDIS_PORT=6379

# API Keys
GEMINI_API_KEY=your_gemini_key
NANO_BANANA_API_KEY=your_nano_banana_key
VEO_API_KEY=your_veo_key

# Storage
STORAGE_PATH=./storage
```

## 🐳 Docker ile Çalıştırma

```bash
docker-compose up -d
```

Bu komut şunları başlatır:
- Backend API (port 8000)
- MongoDB (port 27017)
- Redis (port 6379)

## 📝 Geliştirme Notları

### Varyasyon Üretimi

Sistem, kullanıcı özelleştirmelerine göre 5 farklı varyasyon üretir:

1. **İlk 4 Varyasyon**: Kullanıcının tercihlerine göre şekillendirilir
   - `brand_segment`: Hedef kitle segmenti
   - `photo_design_description`: Fotoğraf tasarım tercihi
   - `user_custom_prompt`: Kullanıcının özel metni
   
2. **5. Varyasyon**: Her zaman "ürün elinde" formatı (lifestyle)

### Video Üretimi Senaryoları

1. **Variation ID ile**: Mevcut bir varyasyon görselinden video üret
2. **Dosya Yükleme ile**: Yeni görsel yükleyerek video üret
3. **Manken Entegrasyonu**: Ürün + manken görsellerini birleştirerek video üret

## 🤝 Katkıda Bulunma

1. Fork yapın
2. Feature branch oluşturun (`git checkout -b feature/amazing-feature`)
3. Commit yapın (`git commit -m 'Add amazing feature'`)
4. Push yapın (`git push origin feature/amazing-feature`)
5. Pull Request açın

## 📄 Lisans

Bu proje private bir projedir.

---

**Not:** Bu backend, kreatif ekiplerin ve reklam platformlarının üzerine UI ekleyip entegre edebileceği, esnek ve genişletilebilir bir AI creative pipeline sunar.
