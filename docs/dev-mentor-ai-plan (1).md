# Android Mentör AI — Proje Planı

## Konumlandırma (düzeltilmiş)
Bu bir "Android'den AI Engineer'a geçiş" ürünü **değil**. Bu, **Android geliştiricilerine özel** bir mentörlük sistemi:
- GitHub'ından, CV'sinden ve (ileride) Medium/LinkedIn gibi hesaplarından mevcut seviyeni belirliyor
- Seviyene uygun konular, sorular üretiyor
- Sana ödevler veriyor, GitHub'dan tamamlanmasını kontrol ediyor, geri bildirim veriyor
- Herhangi bir Android/Kotlin sorununu sorabileceğin, güncel Android bilgisiyle (yeni Compose API'leri, KMP gelişmeleri, Android Studio/Gradle değişiklikleri, platform güncellemeleri) beslenen bir sistem
- İçerikler kopya eğitim/makale değil, RAG ile beslenen kaynaklardan **AI tarafından üretilmiş**, güncel ve atıflı içerikler (makale formatında olabilir, ama orijinal üretim)

Bu aslında önceki taramadan çıkan sonucu **daha da güçlendiriyor**: Araştırmadaki hiçbir oyuncu (CodeSignal, roadmap.sh, Boot.dev, CodeCrafters) Android'e özel değil — hepsi genel/backend ağırlıklı. Android'e özelleşmiş, güncel platform bilgisiyle beslenen bir mentör hâlâ boş bir niş. Üstelik sen bunun tam ortasındasın: 4 yıllık Android deneyimi + 5 kişilik mobil takımı yöneten Product Owner olarak, bir mid-level Android geliştiricinin gerçekte neye takıldığını, hangi konuların "seviye atlatan" konular olduğunu zaten biliyorsun. Bu senin asıl farkın — AI engineering bilgin değil, Android'deki derinliğin.

**Ürün, ilk aşamada sadece Android geliştiricilerine özel — ancak mimari, ileride talep olursa başka branşlara (backend, iOS, vb.) genişleyebilecek şekilde tasarlanmalı** (rubric'ler ve içerik katmanı branşa özel modüller olarak ayrılmalı).

---

## Değerlendirme Modeli (4 Eksen)
Kullanıcı, sisteme girişte 4 eksende değerlendirilir:

1. **Kotlin Bilgisi** — dil idiomatikliği, coroutines/flow kullanımı, null safety, sealed class/data class kullanımı vb.
2. **Jetpack Compose Bilgisi** — state management, recomposition farkındalığı, side-effect API kullanımı, performans pattern'leri
3. **Yazılım Prensipleri Bilgisi** — SOLID, Clean Architecture/MVI, test edilebilirlik, modülerlik
4. **Sistem Tasarımı Bilgisi** — ölçeklenebilirlik, modül sınırları, mimari kararların gerekçelendirilmesi, offline-first/senkronizasyon gibi konular

**Yöntem: Hibrit (kod analizi + adaptif senaryo diyaloğu)**
- Kotlin ve Compose eksenleri büyük ölçüde **doğrudan kod analizinden** çıkarılabilir (GitHub repoları üzerinden)
- Yazılım Prensipleri eksenine kod analizi + kısa senaryo soruları destek verir (repo'da net görünmeyen kararlar için)
- Sistem Tasarımı eksenine **ağırlıklı olarak mülakat tarzı senaryo diyaloğu** ile bakılır — çünkü bireysel/kişisel GitHub repoları genelde büyük ölçekli sistem tasarımı kararlarını yeterince sergilemez ("100 bin kullanıcılı bir feature'ı nasıl tasarlardın", "bu modülü nasıl bölerdin" tarzı sorular)
- Her eksen için bir seviye/skor (örn. junior/mid/senior veya 0-100) üretilir; roadmap bu 4 skora göre kişiselleştirilir

> Not: Bu, projenin en riskli/en yeni parçası olduğu için build sıralamasında **ilk sırada** yer alıyor (bkz. Faz 1a). Geri kalan agent döngüsüne (görev atama, GitHub takibi) yatırım yapmadan önce bu motorun gerçekten değerli sonuçlar üretip üretmediği doğrulanmalı.

---

## Faz 0 — Hafifletilmiş, Paralel Doğrulama (build ile eş zamanlı, yapılandırılmamış)
Yapılandırılmış 10-15 görüşme atlandı — bunun yerine build başlarken paralel yürüyen, düşük maliyetli bir sinyal toplama:

- [ ] Bu hafta içinde 5-10 kişiye (Migrosone içi/dışı Android geliştiriciler, topluluk gruplarındaki tanıdıklar) kısa bir mesaj: "GitHub'ını analiz edip seviyeni söyleyen, mülakat tarzı sorular soran, düzenli ödevler veren bir sistem olsa kullanır mısın?" — amaç yapılandırılmış görüşme değil, hızlı bir ilgi/itiraz sinyali
- [ ] Bu geri bildirimler, Faz 1a'nın rubric ve soru tasarımına girdi olarak kullanılır ama build'i durdurmaz

**Not:** Bu adım artık bir "geç/kal" kapısı değil — build paralelde başlıyor. Asıl doğrulama yükü Faz 1a'ya taşındı.

---

## Faz 1a — Değerlendirme Motoru (İlk Vertical Slice, 3-4 hafta)
Agent döngüsünün tamamını inşa etmeden önce, en riskli/en yeni parçayı izole şekilde inşa edip doğrula.

**Kapsam:**
- GitHub OAuth (GitHub App) + CV yükleme
- Kod analizi: Kotlin idiomatikliği, Coroutines/Flow, Compose pattern'leri, mimari netliği, test coverage, modülerlik, performans farkındalığı
- Adaptif senaryo diyaloğu: özellikle Sistem Tasarımı ve Yazılım Prensipleri eksenlerinde mülakat tarzı sorular
- Claude API ile 4 eksenli rubric üzerinden puanlama
- Çıktı: 4 eksende skor + kısa gerekçelendirme (roadmap'e henüz gerek yok, sadece "bu değerlendirme bana doğru/değerli geliyor mu" sorusuna cevap aranıyor)

**Test:**
- Önce kendi GitHub'ında ve kendi mülakat cevaplarınla dene
- Sonra 3-5 gönüllüyle (Faz 0'daki hafif temas listesinden) dene

**Çıkış kriteri:** Gönüllülerin çoğu değerlendirmeyi "isabetli" buluyorsa → Faz 1b'ye geç. Bulmuyorsa → rubric'i ve soru tasarımını, agent döngüsüne yatırım yapmadan revize et.

---

## Faz 1b — Agent Döngüsü (Faz 1a doğrulandıktan sonra, 4-6 hafta)

**Seviye + Roadmap:**
- 4 eksendeki skorlara göre somut, Android'e özel bir yol haritası

**Ajan döngüsü:**
- Roadmap'e göre haftalık/2 haftalık somut, GitHub'da doğrulanabilir görevler ata
- GitHub webhook ile commit/PR'ları dinle
- Claude API ile PR/commit kalitesini değerlendir, geri bildirim ver
- Tamamlanan görevlere göre 4 eksendeki seviyeyi güncelle, roadmap'i yeniden hesapla
- Serbest soru-cevap: kullanıcı istediği an bir Android sorununu paylaşabilsin, güncel platform bilgisiyle (RAG/arama ile beslenen) yanıt alsın

**Güncel Android bilgisi katmanı:**
- Android geliştirici blogu, Compose/KMP release notes, Android Developers dokümantasyonu gibi kaynaklardan düzenli bir bilgi tabanı (RAG) besleme
- Bu kaynaklardan AI tarafından **orijinal, atıflı içerik üretimi** (makale formatında olabilir) — kopyala-yapıştır değil

**Teknik yığın (karar verildi):**
- Backend: Python + FastAPI (LangGraph ile ajan orkestrasyonu; kod-analiz alt-akışı + mülakat/diyalog alt-akışı + görev/takip alt-akışı aynı uygulama içinde ayrı modüller olarak — ayrı bir "AI endpoint" servisine MVP'de gerek yok, sadece ileride agent çalıştırmaları API'yi bloklamaya başlarsa background worker/queue'ya (Celery/RQ/Arq) taşınabilir)
- Frontend: Next.js (React + TypeScript) — web deneyimi olmadığı için en çok dokümante edilmiş, en çok AI-asistanlı kodlama desteği olan seçenek tercih edildi
- Database: **Supabase (managed Postgres)** — kendi sunucusunda barındırılmıyor; GitHub OAuth token'ları ve kişisel kod/CV verisi barındırdığı için şifreleme/yedekleme/güvenlik yamaları managed servise bırakılıyor. Sadece Postgres kısmı kullanılacak, Supabase'in auth/storage gibi diğer özellikleri opsiyonel
- Backend deployment: kendi sunucusunda (VPS — Hetzner/DigitalOcean, başlangıç için ~4GB RAM yeterli), **Coolify** üzerinden Docker container olarak — git push ile deploy, otomatik HTTPS (Let's Encrypt), container yönetimi Coolify tarafından sağlanıyor; ham nginx/systemd config'i ile uğraşmaya gerek yok (DB ayrı: backend kendi VPS'de, DB Supabase'de)
- İzleme: ücretsiz bir uptime monitoring (ör. UptimeRobot) kurulacak — sunucu/servis çöktüğünde haber alınması için
- Agent: LangGraph + Claude API
- GitHub entegrasyonu: GitHub App + webhooks
- Güncel bilgi: RAG pipeline (Android resmi dokümantasyon + seçili kaynaklar)
- Güvenlik notu: token'lar DB'de şifreli saklanacak, API key'ler environment variable/secrets manager'da tutulacak — bu, Supabase'e taşınsa da backend tarafının kendi sorumluluğu

**Çıkış kriteri:** 10-20 pilot Android geliştirici, en az 2 tam döngü (görev al → tamamla → değerlendirilme → seviye atla) yaşıyor olmalı.

---

## Faz 2 — Genişletme
- LinkedIn: scraping yok — resmi data export ya da manuel paste akışı
- Medium RSS entegrasyonu
- Mobil uygulama (web'de retention kanıtlandıktan sonra)
- Belki ileride diğer mobil platformlara (iOS/KMP) veya branşlara (backend vb.) genişleme — ama önce Android'de derinlik

## Faz 3 — Monetizasyon Testi
- **B2C:** Bireysel Android geliştiriciler için abonelik (seviye atlama, portföy güçlendirme hedefine bağlı)
- **B2B:** Migrosone gibi şirketlere "takımının Android skill gap'ini gör" dashboard'u (4 eksende takım bazlı görünüm) — kendi takımın bile ilk pilot alan olabilir

---

## Hemen Yapılacaklar
1. Bu hafta 5-10 kişiye hafif/yapılandırılmamış sinyal mesajı at (Faz 0)
2. Paralelde: GitHub App kaydı + OAuth akışını kur
3. Faz 1a'yı (değerlendirme motoru) izole bir vertical slice olarak inşa etmeye başla — agent döngüsüne henüz dokunma

## İzlenecek Eşikler
- Faz 1a'da gönüllülerin çoğu değerlendirmeyi isabetsiz/yüzeysel buluyorsa → agent döngüsüne (Faz 1b) geçmeden rubric ve soru tasarımını revize et
- Sistem Tasarımı mülakatına kullanıcı tepkisi olumsuzsa (yapay/gereksiz bulunuyorsa) → bu eksen için diyalog yerine daha hafif bir yöntem düşünülmeli
- Değerlendirme doğruluğuyla ilgili şikayetler gelirse → değerlendirilen konu yüzeyini daralt (ör. sadece Compose + Coroutines)
