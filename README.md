# AI Studio - Yapay Zeka Destekli Pazarlama Reklam Üretim Sistemi

## 📋 Proje Hakkında

AI Studio, tek bir ürün görselinden başlayarak yapay zeka teknolojileri kullanarak profesyonel pazarlama reklamları üreten kapsamlı bir backend sistemidir. Sistem, ürün görsellerini analiz eder, kampanya brief'ine göre dinamik varyasyonlar oluşturur, seçilen varyasyonlardan video reklamlar üretir ve pazarlama ekiplerinin ihtiyaç duyduğu tüm kreatif içerikleri otomatik olarak hazırlar.

### 🎯 Ne İşe Yarar?

Bu sistem ile:

- **Ürün Görseli Analizi**: Yüklenen ürün görseli Gemini Vision API ile analiz edilir
- **Otomatik Varyasyon Üretimi**: LLM (Gemini) kullanılarak kampanya brief'ine göre 5 farklı görsel varyasyonu için prompt'lar oluşturulur
- **Görsel Üretimi**: Nano Banana (Gemini Imagen) ile yüksek kaliteli pazarlama görselleri üretilir
- **Görsel Düzenleme**: Mevcut görseller üzerinde istenen değişiklikler yapılabilir
- **Video Reklam Üretimi**: Seçilen varyasyonlardan ve mankenlerden profesyonel pazarlama videoları üretilir (Veo 3)
- **Pazarlama Reklamları**: Sistem, varyasyonlardan birini ve mankenlerden birini seçerek otomatik olarak pazarlama reklamları oluşturur

## 🚀 Özellikler

### ✅ Tamamlanmış Özellikler

- ✅ Ürün görseli yükleme ve analiz
- ✅ LLM destekli varyasyon planlama
- ✅ Otomatik görsel üretimi (Nano Banana/Imagen)
- ✅ Görsel düzenleme ve iyileştirme
- ✅ Video üretimi (Veo 3 entegrasyonu)
- ✅ Varyasyon ve manken seçimi ile pazarlama reklamı üretimi
- ✅ MongoDB ile veri yönetimi
- ✅ Redis + RQ ile asenkron job kuyruğu
- ✅ RESTful API endpoints

## 🛠️ Teknoloji Yığını

### Backend Framework
- **FastAPI**: Modern, hızlı ve async destekli Python web framework
- **Uvicorn**: ASGI server

### Veritabanı & Kuyruk
- **MongoDB**: Doküman tabanlı NoSQL veritabanı (proje, varyasyon ve video kayıtları için)
- **Redis**: In-memory veri yapısı ve job queue için
- **RQ (Redis Queue)**: Arka plan işleri için job queue sistemi

### Yapay Zeka Entegrasyonları
- **Google Gemini API**: 
  - LLM için metin üretimi (varyasyon planlama, video planlama)
  - Vision API ile görsel analiz
- **Nano Banana / Gemini Imagen**: Görsel üretimi için
- **Veo 3**: Video üretimi için

### Diğer Teknolojiler
- **Pydantic**: Veri validasyonu ve model tanımları
- **Motor**: MongoDB için async Python driver
- **Pillow**: Görsel işleme
- **Python Multipart**: Dosya yükleme desteği

## 📁 Proje Yapısı

```
Ai_studio/
├── backend/
│   ├── src/
│   │   ├── main.py                 # FastAPI uygulama giriş noktası
│   │   ├── core/                   # Altyapı katmanı
│   │   │   ├── config.py          # Konfigürasyon yönetimi
│   │   │   ├── db.py              # MongoDB bağlantıları
│   │   │   └── queue.py           # Redis + RQ yapılandırması
│   │   ├── models/                 # Veri modelleri
│   │   │   ├── project.py         # Proje modeli
│   │   │   ├── variation.py       # Varyasyon ve düzenlenmiş görsel modelleri
│   │   │   └── video.py           # Video modeli
│   │   ├── integrations/          # Dış servis entegrasyonları
│   │   │   ├── gemini_client.py   # Gemini LLM ve Vision API
│   │   │   ├── nano_banana.py     # Görsel üretim (Imagen)
│   │   │   └── veo_client.py      # Video üretim (Veo 3)
│   │   ├── services/              # İş mantığı katmanı
│   │   │   ├── variation_planner.py    # Varyasyon planlama
│   │   │   ├── image_orchestrator.py   # Görsel üretim orkestrasyonu
│   │   │   ├── image_editor.py         # Görsel düzenleme
│   │   │   ├── video_planner.py        # Video planlama
│   │   │   └── video_generator.py      # Video üretim koordinasyonu
│   │   ├── workers/               # Arka plan işçileri
│   │   │   ├── image_worker.py    # Görsel üretim worker'ı
│   │   │   └── video_worker.py    # Video üretim worker'ı
│   │   └── api/
│   │       └── routes/            # API endpoint'leri
│   │           ├── projects.py    # Proje yönetimi
│   │           ├── variations.py  # Varyasyon yönetimi
│   │           └── video.py      # Video üretimi
│   ├── storage/                   # Üretilen görseller ve videolar
│   ├── pyproject.toml            # Python proje yapılandırması
│   └── requirements.txt          # Python bağımlılıkları
├── docker-compose.yml            # Docker container yapılandırması
└── README.md                     # Bu dosya
```


## 📡 API Endpoints

### Proje Yönetimi

#### Yeni Proje Oluştur
```http
POST /api/projects/
Content-Type: multipart/form-data

Form Data:
- image: (file) Ürün görseli
- product_name: (string) Ürün adı
- target_audience: (string, optional) Hedef kitle
- campaign_purpose: (string, optional) Kampanya amacı
- brand_tone: (string, optional) Marka tonu
```

**Yanıt:**
```json
{
  "project_id": "507f1f77bcf86cd799439011",
  "status": "processing",
  "variation_ids": ["507f1f77bcf86cd799439012", ...],
  "message": "Project created. Variations are being generated."
}
```

#### Proje Varyasyonlarını Listele
```http
GET /api/projects/{project_id}/variations
```

#### Görsel Düzenle
```http
POST /api/projects/{project_id}/edit-image
Content-Type: multipart/form-data

Form Data:
- image: (file) Düzenlenecek görsel
- edit_instructions: (string) Düzenleme talimatları
- variation_id: (string, optional) Varyasyon ID'si
```

#### Düzenlenmiş Görselleri Listele
```http
GET /api/projects/{project_id}/edited-images?variation_id={variation_id}
```

#### Proje Sil
```http
DELETE /api/projects/{project_id}
```

#### Varyasyon Sil
```http
DELETE /api/projects/{project_id}/variations/{variation_id}
```

### Video Üretimi

Video üretimi için endpoint'ler `src/api/routes/video.py` dosyasında tanımlanmıştır. Seçilen bir varyasyon ve manken ile pazarlama reklamı üretmek için ilgili endpoint'i kullanabilirsiniz.

## 🔄 İş Akışı

### 1. Proje Oluşturma ve Varyasyon Üretimi

1. Kullanıcı ürün görseli ve kampanya bilgilerini yükler
2. Sistem görseli Gemini Vision API ile analiz eder
3. LLM (Gemini) kampanya brief'ine göre 5 farklı varyasyon için prompt'lar oluşturur
4. Her varyasyon için Nano Banana/Imagen ile görsel üretilir
5. Üretilen görseller MongoDB'ye kaydedilir ve storage'a yazılır

### 2. Görsel Düzenleme

1. Kullanıcı bir görsel seçer ve düzenleme talimatları verir
2. Sistem orijinal görsel + talimatlar ile yeni görsel üretir
3. Düzenlenmiş görsel `edited_images` koleksiyonuna kaydedilir

### 3. Video Reklam Üretimi

1. Kullanıcı bir varyasyon seçer
2. Sistem varyasyon görselini ve kampanya bilgilerini kullanarak LLM ile video planı oluşturur
3. Video planı Veo 3 API'sine gönderilir
4. Video üretimi arka planda (worker) gerçekleştirilir
5. Tamamlanan video MongoDB'ye kaydedilir ve storage'a yazılır

### 4. Pazarlama Reklamı Üretimi

Sistem, varyasyonlardan birini ve mankenlerden birini seçerek otomatik olarak pazarlama reklamları üretir. Bu özellik video üretim pipeline'ı ile entegre çalışır.

## 🏗️ Mimari

Sistem katmanlı bir mimari kullanır:

- **API Katmanı**: FastAPI route'ları, HTTP isteklerini işler
- **Servis Katmanı**: İş mantığı, LLM çağrıları, planlama
- **Entegrasyon Katmanı**: Dış AI servisleri (Gemini, Imagen, Veo)
- **Worker Katmanı**: Uzun süren işler (görsel/video üretimi) için arka plan işçileri
- **Veri Katmanı**: MongoDB ile kalıcı veri saklama
- **Kuyruk Katmanı**: Redis + RQ ile asenkron job yönetimi

## 🔐 Güvenlik ve Konfigürasyon

- API anahtarları `.env` dosyasında saklanır (asla commit edilmemelidir)
- CORS ayarları `main.py` içinde yapılandırılmıştır
- Güvenlik filtreleri Gemini API çağrılarında yapılandırılabilir

## 📝 Geliştirme Notları

- Tüm async işlemler `asyncio` ve `motor` (MongoDB async driver) kullanır
- Görsel üretimi için hem eski hem yeni Gemini API formatları desteklenir
- Worker'lar Redis Queue ile yönetilir ve ayrı process'lerde çalışabilir

## 🤝 Katkıda Bulunma

Bu proje açık kaynaklıdır ve katkılarınızı bekliyoruz. Lütfen pull request göndermeden önce kod standartlarına uyduğunuzdan emin olun.

## 📄 Lisans

[Lisans bilgisi buraya eklenecek]

## 📞 İletişim

Sorularınız için issue açabilir veya [iletişim bilgileri] üzerinden ulaşabilirsiniz.

---

**Not**: Bu sistem, pazarlama ekiplerinin kreatif süreçlerini hızlandırmak ve AI destekli içerik üretimini otomatikleştirmek için tasarlanmıştır. Video üretimi ve pazarlama reklamı özellikleri tam olarak çalışır durumdadır.

