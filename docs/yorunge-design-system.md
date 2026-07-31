# Yörünge — Tasarım Sistemi (v0.2)

**Domain:** yorunge.dev

Bu sürümde üç değişiklik var: **açık tema** eklendi, **tipografi** klişe kombinasyonlardan uzaklaştırılarak yeniden araştırıldı, ve genel arayüz kompozisyonu **"AI ile üretilmiş" hissinden** uzaklaştırıldı. Aşağıda her birinin gerekçesi var — sonraki güncellemelerde neden bu kararların alındığını hatırlamak için.

---

## 1. Neden değişti — kısa özet

**Önceki sürümün sorunu:** Space Grotesk + Inter ikilisi ve izole "kart ızgarası" bileşen düzeni — araştırma bunun tam olarak şu an AI araçlarının varsayılan çıktısı olduğunu doğruluyor: Anthropic'in kendi frontend-design rehberi bu ikiliyi ve Inter/Roboto/Arial'i açıkça "kullanma" listesine alıyor, çünkü model her "modern ve özgün" istendiğinde ilk bu ikiliye gidiyor ve bu artık kendi başına bir klişe haline geldi.

**Ne yaptık:** Tipografiyi konunun kendisinden (gözlemevi/gök bilimi/enstrüman panosu) besleyerek yeniden seçtik; bileşenleri izole kartlar yerine gerçek bir ekran kompozisyonu olarak kurduk; açık temayı klişe krem/parşömen yerine soğuk bir "gündüz göğü" tonuna dayandırdık.

---

## 2. Renk — Koyu & Açık

| Token | Koyu | Açık | Kullanım |
|---|---|---|---|
| `--bg` | `#0B0E1A` | `#EEF1F7` | Ana zemin |
| `--bg-2` | `#10142A` | `#E3E8F3` | İkincil zemin |
| `--surface` | `#161B33` | `#FFFFFF` | Kart/panel zemini |
| `--surface-2` | `#1D2440` | `#F6F8FC` | Yükseltilmiş panel |
| `--text` | `#F0EDE4` | `#121630` | Birincil metin |
| `--text-dim` | `#9BA3C4` | `#565F82` | İkincil metin |
| `--amber` | `#F2A65A` | `#A85F14` | Tamamlanmış görev, başarı, birincil CTA |
| `--blue` (Orbit) | `#6C9BFF` | `#2F4FAE` | Aktif/devam eden görev |
| `--dim` | `#2A2F42` | `#D9DEEC` | Kilitli/pasif düğüm |
| `--pink` (Alev) | `#E17690` | `#A13A54` | Uyarı, dikkat |
| `--border` | `#2E3556` | `#D5DAEA` | Kenarlık, ayraç |

**Açık tema neden krem/parşömen değil:** Sıcak krem zemin + terracotta aksan (yaklaşık `#F4F1EA` + `#D97757`), şu an AI araçlarının "editoryal/sıcak" istendiğinde varsayılan olarak ürettiği ikinci en tanınabilir kombinasyon. Bunun yerine soğuk, hafif mavimsi bir "gündüz göğü" zemini (`#EEF1F7`) ve mürekkep lacivert metin (`#121630`) kullandık — hem klişeden kaçınıyor hem de "gece gözlemevi / gündüz gözlemevi" fikrini tutarlı tutuyor.

**Kontrast notu:** Açık temada amber, mavi ve pembe tonları koyulaştırıldı (ör. amber `#F2A65A` → `#A85F14`) — aksi halde açık zeminde WCAG AA kontrastını karşılamazlar.

**Sabit renk:** Yörünge sisteminin merkezindeki "Sen" güneşi her iki temada da aynı amber tonda sabit kalır — bir mühür gibi, temadan bağımsız kimlik.

---

## 3. Tipografi

| Rol | Font | Neden |
|---|---|---|
| Display (H1–H2) | **Newsreader** (italic vurgu) | Google'ın kitap/dergi kökenli serifi; bilimsel almanak ve gözlemevi kaydı ciddiyetini taşıyor. Fraunces/Playfair kadar yaygınlaşmadı, hâlâ ayırt edici. İtalik kesimi kişisel/duygusal anları vurgulamak için kullanılır ("Sen", başarı mesajları). |
| Body/UI | **Hanken Grotesk** | Inter'in ekran-optimize edilmiş ama daha az "her yerde görülen" alternatifi; sayı ve veri okunurluğu güçlü. |
| Data/Mono | **Space Mono** | IBM Plex Mono yerine seçildi — adı bile marka temasıyla (yörünge/uzay) örtüşüyor, eski görev-kontrol terminali karakteri taşıyor. Fonksiyonel + anlamlı bir seçim. |

**Kaçındığımız kombinasyon:** Space Grotesk + Inter. Araştırma, bu ikilinin 2026 itibarıyla "AI tarafından üretildi" izlenimi veren en yaygın tespit noktalarından biri haline geldiğini gösteriyor — hem Anthropic'in hem OpenAI'ın kendi tasarım rehberleri bunu doğrudan yasaklıyor.

**Tip ölçeği:**
- H1: 38–58px / Newsreader 500 italic
- H2: 25–27px / Newsreader 500
- Body: 15–16.5px / Hanken Grotesk 400
- Caption: 12–13px / Hanken Grotesk 500, `--text-dim`
- Mono/data: 11.5–13.5px / Space Mono 400

---

## 4. Kompozisyon — "AI slop" hissinden kaçınma

**Sorun:** İzole, birbirinden kopuk "demo kartları" ızgarası (her biri aynı boyutta, aynı gölgede, aynı iç boşlukta) — bu, üretken araçların en sık ürettiği ve en kolay tanınan yapı.

**Çözüm:** Bileşenler bölümü artık tek bir gerçek ekran kesiti (`mock-panel`) olarak kurulu: karşılama başlığı, mini yörünge göstergesi, eksen çubukları ve görev kartları aynı anlatının parçası olarak bir arada. İzole kart ızgarası yalnızca gerçekten karşılaştırma gerektiren yerlerde (renk paleti gibi) kullanılıyor.

**İleride dikkat edilecekler:**
- Yeni ekranlar tasarlanırken "4 eşit kutu" refleksinden kaçının; önce gerçek kullanıcı akışını (ör. "görev tamamlama anı") kompoze edin, bileşenler o akışın içinden çıksın
- Buton/rozet şekillerinde aşırı yuvarlak (pill) kullanım tek başına yeterli bir "kimlik" değil — Yörünge sisteminin kendisi (halkalar, yörünge çizgileri) asıl ayırt edici imza olarak kalmalı
- Her yeni ekranda gölge/blur/gradient miktarını kontrol edin — atmosfer için yıldız dokusu zaten var, üstüne "AI glow" eklemeyin

---

## 5. Genişleme Notu (değişmedi)

Yeni bir domain (backend, iOS, vb.) eklendiğinde: yörünge sistemine yeni bir halka eklenir, kullanıcı aktif takip ettiği domain(ler)i seçer, marka dili ve tipografi sistemi değişmeden kalır.

---

*Bu doküman, `yorunge-design-system.html` içindeki canlı stil rehberiyle (açık/koyu tema geçişli) birlikte kullanılmalıdır.*
