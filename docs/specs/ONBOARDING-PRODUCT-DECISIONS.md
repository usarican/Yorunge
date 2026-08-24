# ONBOARDING PRODUCT DECISIONS — Değerlendirme Algoritmaları, Soru Stratejisi ve Ürün Kararları

> **Doküman:** `ONBOARDING-PRODUCT-DECISIONS.md`  
> **Referans Doküman:** [`ONBOARDING-SPEC.md`](file:///Users/ibrahimutkusarican/Yorunge/docs/specs/ONBOARDING-SPEC.md)  
> **Sürüm:** v1.0  
> **Konum:** `docs/specs/ONBOARDING-PRODUCT-DECISIONS.md`  
> **Hedef:** Yörünge onboarding değerlendirme motorunun soru seçim algoritmalarını, adaptif zorluk mekanizmasını (CAT), 3 kaynaklı hibrit soru havuzunu, soru taksonomisini ve aday deneyimi bütçeleme kararlarını dokümante etmek.

---

## 🎯 1. Değerlendirme Felsefesi & Girdi Analizi

Yörünge'nin değerlendirme felsefesi, adayı başlangıçta doğrudan bir unvan kalıbına (Junior/Mid/Senior) hapsetmek **değildir**. Adayın gerçek taban ve tavan (ceiling/floor) yetkinliklerini adil ve dinamik bir biçimde tespit etmektir.

### 🔹 CV ve GitHub AST Analizinin Rolü (Topic Focus Generation)
- **Ön Tanı (Pre-determination) Değildir:** CV ve GitHub repolarından elde edilen veriler aday için bir "unvan etiketlemesi" yapmaz.
- **Konu Odağı Matrisi (Topic Focus Target):** AST taramasından elde edilen framework kullanımı (ör. Coroutine, StateFlow, Room, Compose, Hilt/Koin) ve CV'deki deneyim odakları, adaya sorulacak **soruların konu başlıklarını ve odak alanlarını** belirler.
- *Örnek:* Repo analizinde `LiveData` kullanıldığı ancak `StateFlow`/`SharedFlow` bulunmadığı tespit edilirse, Quiz motoru adaya `StateFlow` vs `LiveData` mimari karşılaştırma sorusunu öncelikli konu odağı olarak atar.

---

## 🧠 2. Adaptif Değerlendirme Algoritması (CAT - Computerized Adaptive Testing)

Onboarding Adım 3 (Kişiselleştirilmiş Teknik Sorular) aşaması, **Computerized Adaptive Testing (CAT)** prensiplerine göre çalışır.

### 🔹 Dinamik Zorluk Ayarlaması (Dynamic Difficulty Adjustment - DDA)
1. **Başlangıç Noktası (Baseline Question):** Test, adayın AST ve CV karmaşıklığına dayalı ortalama bir zorluk seviyesindeki soru ile başlar.
2. **Doğru Yanıt Durumu (Difficulty Escalation):** Aday soruyu doğru yanıtladığında, sonraki sorunun zorluk seviyesi artırılır (Junior → Mid → Senior → Staff / Edge-Case). Adayın üst yetkinlik tavanı (ceiling skill) sınanır.
3. **Yanlış Yanıt Durumu (Difficulty De-escalation):** Aday soruyu yanlış yanıtladığında veya yetersiz kaldığında, zorluk seviyesi bir alt kademeye çekilerek adayın taban bilgi seviyesi doğrulanır.
4. **Erken Atlama / Kesme Yok:** Doğru cevap sayısı arttıkça sorular atlanmaz veya test erken bitirilmez. Aday 8-12 soruluk bütçenin tamamını çözer; böylece 4 eksenin tamamında yeterli veri toplanır.

```
       [Soru 1: Baseline (Mid-)]
              │
     ┌────────┴────────┐
  (Doğru)           (Yanlış)
     ▼                 ▼
[Soru 2: Mid+]    [Soru 2: Junior]
     │                 │
  (Doğru)           (Doğru)
     ▼                 ▼
[Soru 3: Senior]  [Soru 3: Mid-]
```

### 🔹 Adım 4 (Sistem Tasarımı Mülakatı) Geçiş Mantığı
Adım 3 (Quiz) tamamlandığında 4 eksenli nihai skor ve unvan hesaplanır:
- **Nihai Seviye == Junior:** Adım 4 (Sistem Tasarımı Mülakatı) **otomatik olarak atlanır**. Aday doğrudan Adım 5 (Bütünsel Seviye Raporu) ekranına yönlendirilir.
- **Nihai Seviye >= Mid (Mid / Senior / Staff):** Adım 4 **açılır**. Aday, 100k+ kullanıcı senaryolarını içeren **Metin Tabanlı Sistem Tasarımı Mülakatı** adımına geçer.

---

## 📚 3. Üç Kaynaklı Hibrit Soru Havuzu (Tri-Source Question Engine)

Soru havuzu tek bir statik veritabanı veya %100 jenerik LLM üretimi değildir. 3 farklı kaynağın harmanlanmasıyla oluşur:

```
┌────────────────────────────────────────────────────────────────────────┐
│                   YÖRÜNGE HİBRİT SORU MOTORU                          │
├──────────────────┬──────────────────────┬──────────────────────────────┤
│ 1. CURATED BANK  │ 2. RAG PIPELINE      │ 3. DYNAMIC LLM GENERATION    │
│ (Sektör Standardı│ (Güncel Android Docs │ (GitHub AST Repolarına Özel  │
│  Mülakat Bankası)│  & Release Notes)    │  Kod İçi Bug Hunting)        │
└─────────┬────────┴──────────┬───────────┴──────────────┬───────────────┘
          │                   │                          │
          └───────────────────┼──────────────────────────┘
                              ▼
           [ADAPTİF SEÇİCİ & ZORLUK MOTORU (CAT)]
```

1. **İnternet / Sektör Standart Soru Bankası (Curated Benchmark Bank):**
   - Sektörde kabul görmüş, geçerliliği kanıtlanmış Android/Kotlin mülakat soruları.
   - Temel kavramsal ve teorik doğrulamayı sağlar.
2. **RAG Pipeline (Güncel Android Dokümantasyonu & Release Notes):**
   - Official Android Developers Dokümantasyonu, Jetpack Compose Release Notes, Kotlin 2.0+ güncellemeleri ve Android Developers Blog RAG pipeline'ı ile taranır.
   - Güncel platform bilgisine (yeni Compose API'leri, KMP gelişmeleri, Coroutine/Flow güncellemeleri) dayalı sorular türetilir.
3. **Dinamik LLM Kod Üretimi (AST-Driven Code Generation):**
   - Adayın kendi GitHub repolarındaki kod yapılarına (AST) özel sıfırdan üretilen kod snippet'ları.
   - Adayın kendi yazdığı kod kalıplarındaki anti-pattern'leri veya performans sızıntılarını tespit etmesini isteyen kişiselleştirilmiş sorular.

### 🔹 Soru Eskime ve Güncellik Pipeline'ı (Deprecation Guard)
- Android platformunda deprecated olan (kullanımdan kaldırılan veya yerini yeni API'lere bırakan) konular RAG pipeline tarafından tespit edilir.
- Eskiyen sorular otomatik olarak **"Deprecated Status"** alır veya yeni API versiyonuna göre LLM tarafından refactor edilir.

---

## 🧩 4. Soru Tipleri ve Ölçüm Taksonomisi

Değerlendirme motorunda 5 temel soru tipi kullanılır:

| Soru Tipi | Ölçüm Amacı | Format |
| :--- | :--- | :--- |
| **1. Karşılaştırma & Tercih (Trade-offs)** | Mimari karar verme ve farkındalık | `StateFlow` vs `SharedFlow`, `remember` vs `rememberSaveable` mimari seçim ve gerekçelendirme |
| **2. Kod İçi Hata Tespiti (Bug Hunting)** | Kod okuma ve edge-case farkındalığı | Short Kotlin/Compose snippet; Recomposition tuzakları, Coroutine memory leak satırını bulma |
| **3. Mimari Refactoring** | Kod kalitesi ve Clean Code bilgisi | Kötü yazılmış ViewModel/MVI State kodunu ideal formuna getirme seçimi |
| **4. Edge-Case & Gerekçelendirme** | Senior/Staff seviye derinlik | `SavedStateHandle`, `SupervisorJob`, Custom Modifier davranış detayları |
| **5. Sistem Tasarımı Senaryosu (Adım 4)** | Ölçeklenebilirlik ve sistem mimarisi | Mid+ adaylar için **Metin Tabanlı (Text-based)** 100k kullanıcı mimari diyaloğu |

---

## ⏱️ 5. Soru Bütçeleme, Zamanlama ve Bilişsel Yorgunluk Yönetimi

Aday terk oranını (drop-off rate) minimize etmek ve yüksek tamamlama oranı sağlamak için sıkı bütçeleme kuralları uygulanır:

- **Soru Sayısı Bütçesi:** Adım 3 Quiz alanında **sabit 8 - 12 soru** sorulur.
- **Süre Bütçesi:** Onboarding testinin toplam süresi **10 - 15 dakika** ile sınırlandırılmıştır.
- **Gezinti Kuralları (Back/Forward Navigation):** Sorular arasında *"Önceki Soru"* ve *"Sonraki Soru"* butonları ile serbestçe gezinebilir. Yanıtlar sadece adayın *"Testi Tamamla"* butonuna basmasıyla kilitlenir.
- **Bilişsel Yorgunluk İndeksi (Cognitive Fatigue Control):**
  - Bir test oturumunda **maksimum 2 adet** uzun kod parçalı (Bug Hunting / Refactoring) soru yer alabilir.
  - Diğer sorular net, kısa ve zihinsel yorgunluk yaratmayacak formatta sunulur.
  - İlerleme çubuğu (Progress Indicator) ve tahmini kalan süre adaya şeffaf biçimde gösterilir.

---

## 📊 6. Ürün Metrikleri ve Başarı Kriterleri

- **Onboarding Tamamlama Oranı (Completion Rate):** Adım 1'den Adım 5'e ulaşma oranı **>%85** olmalıdır.
- **Değerlendirme İsabet Algısı (Assessment Relevance Rate):** Adayların çıkan 4 eksenli seviye raporunu "isabetli ve doğru" bulma oranı **>%80** olmalıdır.
- **Soru Havuzu Güncellik İndeksi (Freshness Score):** Soru havuzunun en az **%30'u** RAG ile beslenen güncel Android platform değişikliklerinden oluşmalıdır.

---

## 🧮 7. Skorlama Algoritması — Hibrit Değerlendirme Motoru

> **Karar (2026-08-07):** 0-100 eksen skorları, tek geçişli bir LLM'in doğrudan "kaç puan" dediğine dayanmaz. Her eksen, üç bağımsız ve **tekrarlanabilir** girdinin ağırlıklı bileşimidir. LLM yalnızca (a) kod bulgularını gerekçe metnine çevirmek ve (b) mülakat kriterlerini puanlamak için kullanılır — nihai sayıyı üreten formül her zaman deterministiktir. Amaç: aynı girdiyle her seferinde aynı skor, ve "neden 82 çıktı" sorusuna adım adım cevap verebilmek.

### 🔹 7.1. Eksen Bazlı Ağırlıklandırma

```
axis_score (0-100) = w_quiz · quiz_rating + w_code · code_evidence_score + w_interview · interview_rubric_score
```

| Eksen | Quiz (Glicko-2) | Kod Kanıtı (AST, deterministik) | Mülakat (yapılandırılmış rubrik) |
| :--- | :---: | :---: | :---: |
| Kotlin Idioms | %40 | %60 | — |
| Jetpack Compose | %40 | %60 | — |
| Software Principles | %35 | %35 | %30 (Adım 4 varsa, yoksa quiz+kod %50/%50'ye yeniden normalize edilir) |
| System Design | %25 | %10 | %65 (Adım 4 varsa; Junior adaylarda Adım 4 atlandığı için bu eksen sadece quiz+kod ile hesaplanır ve düşük güvenilirlik bandıyla işaretlenir) |

### 🔹 7.2. Quiz Bileşeni — Glicko-2 Rating Motoru

Basit "doğru cevap yüzdesi" yerine, satranç puanlama sistemlerinde kullanılan **Glicko-2** algoritması uygulanır. Elo'dan farkı: her kullanıcı/soru için ek olarak bir **Rating Deviation (RD)** — yani tahminin güven aralığı — tutulur. Bu, az veriyle (yeni kullanıcı, yeni soru) sahte kesinlik göstermemek için kritiktir.

1. **Soru (item) ratingi:** Her soru bir Glicko-2 ratingine sahiptir. Curated bank ve RAG kaynaklı sorular için başlangıç ratingi **uzman zorluk etiketiyle** çapalanır (bkz. §7.6). AST'ye özel dinamik LLM soruları tek tek çapalanamaz; kategori ortalaması atanır ve yüksek RD ile başlar.
2. **Kullanıcı ratingi güncellemesi:** Kullanıcı bir soruyu doğru/yanlış cevapladıkça, hem kullanıcının hem sorunun ratingi ve RD'si standart Glicko-2 güncelleme formülleriyle yeniden hesaplanır. Yüksek RD'li (belirsiz) sorulara verilen cevaplar, kullanıcı ratingini daha az etkiler.
3. **0-100'e ölçekleme:** `quiz_rating (0-100) = clamp((glicko_rating - 800) / (2400 - 800) * 100, 0, 100)`. Ölçek sınırları (800-2400) uzman çapa aralığına göre kalibre edilir, ilk pilot verisiyle yeniden ayarlanabilir.
4. **CAT ile entegrasyon:** Sonraki sorunun zorluğu, kullanıcının **güncel rating tahminine** göre seçilir (mevcut §2'deki DDA mantığıyla birebir uyumlu) — yani aynı rating hem "hangi soru sorulsun" hem "final skor ne olsun" kararını besler, iki ayrı sistem kurulmaz.

### 🔹 7.3. Kod Kanıtı Bileşeni — Deterministik AST Puanlama

LLM'e kod kalitesine "puan verdirmek" yerine, AST analizinden çıkan **ölçülebilir özellikler** sabit ağırlıklı bir tabloyla toplanır. LLM yalnızca bu bulguları `evidence` metnine çevirir, sayıyı değiştirmez.

| Eksen | Örnek Ölçülebilir Özellikler (ağırlıklı) |
| :--- | :--- |
| Kotlin Idioms | `data class`/`sealed class` kullanım oranı, null-safety ihlali sayısı, Coroutine scope/cancellation hijyeni, Flow operator kullanım çeşitliliği |
| Jetpack Compose | `StateFlow`/`LiveData` oranı, gereksiz recomposition tespiti (stability annotation eksikliği), side-effect API (`LaunchedEffect`/`DisposableEffect`) doğru kullanım oranı |
| Software Principles | Test coverage %, katman bağımlılık ihlali sayısı (ör. UI katmanının doğrudan DB'ye erişimi), modül/paket sınırı netliği |
| System Design | Modülerleşme derecesi, offline-first/senkronizasyon pattern varlığı (sınırlı katkı; ağırlık düşük tutulur çünkü bireysel repo bu ekseni yeterince sergilemez — bkz. plan dokümanı gerekçesi) |

**Çapraz kontrol:** Kod kanıtı skoru ile quiz skoru arasında **>25 puanlık** sapma varsa (ör. yüksek test coverage ama quiz'de coroutine sorularını yanlış cevaplama), sistem bunu `evidence_conflict: true` olarak işaretler ve raporda kullanıcıya şeffaf gösterilir — bu, kopyalanmış/forklanmış kod ile "kandırma" riskine karşı bir denge kontrolüdür.

### 🔹 7.4. Mülakat Bileşeni — Yapılandırılmış LLM-Judge Rubriği

Adım 4'te LLM'e tek bir "0-100 puan" sorulmaz. Bunun yerine 4-5 alt kritere (mimari netlik, trade-off farkındalığı, ölçeklenebilirlik düşüncesi, gerekçelendirme derinliği, edge-case farkındalığı) her biri **1-5 arası** puanlanır, ağırlıklı ortalaması alınır ve 0-100'e ölçeklenir. Tutarlılığı artırmak için aynı yanıt seti **iki kez** puanlanır (self-consistency); iki geçiş arasında >1 puanlık fark olan kriterlerde üçüncü bir hakem geçişi tetiklenir.

### 🔹 7.5. Skor → Seviye Bandı ve Güven Aralığı

```
0-35   → Junior
36-55  → Junior-Mid   (Adım 4 tetiklenmez)
56-75  → Mid
76-100 → Senior
```

Her eksende Glicko-2'den gelen RD, güven aralığına dönüştürülüp raporda gösterilir (ör. *"82 ±6"*). Yüksek RD durumunda (yeni/az kalibre edilmiş soru kategorisi) rapor bunu **"ön değerlendirme, kalibrasyon devam ediyor"** notuyla şeffaf belirtir — sahte kesinlik verilmez.

### 🔹 7.6. Kalibrasyon & Doğrulama Planı

1. **Uzman çapa etiketleme:** Curated bank ve RAG kaynaklı her soruya en az 3 kıdemli Android geliştiricinin bağımsız zorluk oyu (Junior/Mid/Senior/Staff) atanır → başlangıç Glicko-2 ratingi bu ortalamadan türetilir.
2. **Pilot doğrulama seti:** Faz 0'daki 5-10 kişilik sinyal toplamasının **ötesinde**, en az **20-30 gönüllünün** hem sistem tarafından hem de bağımsız 2 kıdemli mühendis tarafından değerlendirilmesi gerekir. Sistem skoru ile insan değerlendirmesi arasındaki korelasyon, §6'daki "isabet algısı >%80" hedefinin somut kanıtı olur (ankete dayalı algı yerine ölçülebilir korelasyon).
3. **Sürekli yeniden kalibrasyon:** Faz 1b'deki görev tamamlama/PR inceleme sonuçları da aynı Glicko-2 güncelleme mantığına girer — onboarding'de kullanılan motor ile "seviye atlama" takibi aynı algoritmayı paylaşır, ayrı bir sistem kurulmaz.

### 🔹 7.7. Veri Modeli Etkisi

Bu algoritmanın çalışması için `backend-architecture.md` §4'teki şemaya bir `ITEM_BANK` tablosu ve `QUIZZES.questions_and_answers` JSONB'sine ek alanlar gerekir — bkz. [`backend-architecture.md` §4.2](file:///Users/ibrahimutkusarican/Yorunge/docs/architecture/backend-architecture.md).
