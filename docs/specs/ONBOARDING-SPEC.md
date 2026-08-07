# ONBOARDING SPEC — Değerlendirme, Kod Analizi ve Teknik Sorular Motoru

> **Doküman:** `ONBOARDING-SPEC.md`  
> **Sürüm:** v4.0 (5 Adımlı Sade Wizard Akışı, Yan Menüsüz Stepper UX ve Esnek Quiz Kontrolü)  
> **Konum:** `docs/specs/ONBOARDING-SPEC.md`  
> **Hedef:** Kullanıcıyı yan menüsüz (distraction-free), adım adım (5-Step Wizard Stepper) bir değerlendirme akışına sokmak; GitHub & CV analizinden sonra kişiselleştirilmiş teknik sorular ve (Mid+ geliştiriciler için) Sistem Tasarımı mülakatı ile 4 eksenli nihai seviyesini belirlemek.

---

## 🎯 Navigasyon & Mimari Kararları

1. **Sidebar (Yan Menü) Gizliliği:** Onboarding süresince (Adım 1 - Adım 5 arası) hiçbir yan menü (Sidebar) bulunmaz. Ekran tamamen **Daylight Observatory Stepper Header** (Üst Adım Çubuğu) ile yönetilir. Sidebar ilk kez Adım 5 sonunda *"Yol Haritama Git →"* butonuna basıldığında ana platformda görünür.
2. **Quiz İçi Gezinti (Step 3 Back Navigation):** Adım 3 (Teknik Sorular) içinde kullanıcı sorular arasında *"Önceki"* ve *"Sonraki"* butonları ile serbestçe gezinebilir. Ancak *"Testi Tamamla & Cevapları Gönder"* butonuna bastıktan sonra yanıtlar kilitlenir ve geriye dönülemez.
3. **Otomatik Geçiş (Step 2 Loading):** Adım 2 (Analiz Ekranı) repoları, CV'yi ve soru kişiselleştirmesini %100 tamamladığında sistem otomatik olarak Adım 3'e geçer.

---

## 🗺️ 5 Adımlı Onboarding Akış Şeması

```
┌────────────────────────────────────────────────────────────────────────┐
│ ONBOARDING STEPPER HEADER: [1. Bağlantı] -> [2. Analiz] -> [3. Quiz] -> [4. Mülakat] -> [5. Rapor] │
└────────────────────────────────────────────────────────────────────────┘

[ADIM 1: GitHub Connect, CV & Repo Seçimi]
       ↓
[ADIM 2: Canlı Analiz & Soru Hazırlama (Otomatik %100 Geçiş)]
       ↓
[ADIM 3: Kişiselleştirilmiş Teknik Sorular (8-12 Soru, 4 Kategori)]
       │ (Sorular arası ileri/geri serbest, "Tamamla" ile kilitlenir)
       ↓
  ┌────┴────────────────────────┐
  ▼                             ▼
[Mid / Senior Seviye?]       [Junior Seviye?]
  │                             │
  ▼ (Evet)                      ▼ (Hayır - Atla)
[ADIM 4: Sistem Tasarımı]  ────┤
[        Konuşmalı Mülakat]     │
  └─────────────────────────────┤
                                ▼
                   [ADIM 5: Bütünsel Seviye Raporu & Skor]
                                │ (CTA: "Yol Haritama Git →")
                                ▼
                   [ANA PLATFORM & SIDEBAR AÇILIR]
```

---

## 📋 Adım Adım Detaylı Spesifikasyonlar

### 🔹 ADIM 1: GitHub Connect, CV ve Repo Seçimi
- **Kullanıcı Amacı:** GitHub hesabını yetkilendirir, CV/LinkedIn bilgilerini ekler ve analiz edilecek repolarını seçer.
- **Bileşenler:**
  - Bağlı GitHub kullanıcı profili ve avatarı.
  - Temsil gücü yüksek Android repoları listesi (yıldız sayısı, ana dil etiketleri, seçim kutuları).
  - CV Sürükle-Bırak yükleme alanı veya LinkedIn/Medium link girişi.
- **CTA:** *"Analizi Başlat →"* (Adım 2'ye geçer).

---

### 🔹 ADIM 2: Canlı Analiz & Soru Hazırlama (Telemetri Ekranı)
- **Kullanıcı Amacı:** Sistem repoları ve CV'yi tararken kişiselleştirilmiş soru havuzunun hazırlanışını canlı telemetri günlüğünde izler.
- **Görsel & Telemetri Logları (`Space Mono`):**
  - `[1/3] Repositories AST syntax trees parsed...`
  - `[2/3] CV & Domain background keywords extracted...`
  - `[3/3] Generating personalized 8-12 technical questions for your profile...`
- **İlerleme & Geçiş:** %0'dan %100'e ilerleyen bar. %100'e ulaştığında **otomatik olarak** Adım 3'e geçer.

---

### 🔹 ADIM 3: Kişiselleştirilmiş Teknik Sorular (8 - 12 Soru)
- **Kullanıcı Amacı:** 4 ana kategoride (Kotlin, Jetpack Compose, Android Core, Yazılım Prensipleri) bilgi seviyesini ölçecek 8-12 soruyu yanıtlar.
- **Soru Tipleri & Case'ler:**
  - **Karşılaştırma (Trade-offs):** `StateFlow` vs `SharedFlow`, `remember` vs `rememberSaveable`.
  - **Kod İçi Hata Tespiti:** Recomposition tuzakları, Coroutine memory leak satırları.
  - **Mimari Refactoring:** ViewModel Context sızıntısı düzeltme, MVI State yapıları.
  - **Kısa Gerekçelendirme:** Senior edge-case soruları (`SavedStateHandle`, `SupervisorJob`).
- **Gezinti kuralı:** Sorular arasında *"Önceki Soru"* ve *"Sonraki Soru"* ile gezinebilir. *"Testi Tamamla"* butonuna basılınca cevaplar kilitlenir.

---

### 🔹 ADIM 4: Koşullu Sistem Tasarımı Mülakatı (Mid+ Seviye İçin)
- **Kullanıcı Amacı:** Kod taraması ve Quiz sonuçlarına göre seviyesi Mid-Level ve üzeri çıkan geliştiriciler için 2-3 soruluk mimari ve sistem tasarımı diyaloğu.
- **Mantık (Decision Engine):**
  - **Seviye >= Mid:** Adım 4 açılır. Kıdemli mentör diyalog paneli ve sesli/yazılı yanıt konsolu ile 100k kullanıcılı sistem tasarımı sorulur.
  - **Seviye == Junior:** Adım 4 otomatik atlanır, doğrudan Adım 5'e geçilir.

---

### 🔹 ADIM 5: Bütünsel Seviye Raporu & Senior Index
- **Kullanıcı Amacı:** Kod AST analizi + Quiz + (Var ise) Mülakat sonuçlarının harmanlandığı 4 eksenli nihai seviye raporunu görür.
- **İçerik:**
  - Genel Seviye Rozeti (Örn: *Mid-Senior Android Developer - Skor: 82/100*).
  - 4 Eksenli Detay Kartları (Kotlin, Compose, Principles, System Design) ve kod kanıtları.
- **Ana Aksiyon (CTA):** *"Yol Haritama Git →"* butonuna tıklandığında Onboarding biter, kullanıcı **Ana Platforma ve Yan Menüye (Sidebar)** giriş yapar.
