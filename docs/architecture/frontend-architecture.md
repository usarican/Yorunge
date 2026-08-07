# Yörünge — Frontend Mimari ve Teknik Tasarım Dokümanı

> **Doküman:** `frontend-architecture.md`  
> **Sürüm:** v1.0  
> **Konum:** `docs/architecture/frontend-architecture.md`  
> **Hedef:** Yörünge platformunun Next.js 14+ (App Router) tabanlı ön yüz altyapısını, 5 adımlı Onboarding Wizard akışını, Daylight Observatory tasarım sistemini, state yönetimini, SSE/WebSocket canlı veri akışlarını ve Docker kapsayıcı yapısını tanımlamak.

---

## 1. Genel Mimari Bakış

Frontend, Next.js 14 App Router standartlarına uygun olarak tasarlanmıştır. Uygulama **iki ana navigasyon moduna** ayrılır:

1. **Distraction-Free Wizard Modu (`(onboarding)`):** Yan menü (Sidebar) barındırmayan, 5 adımlı üst `Daylight Stepper Header` ile yönetilen odaklanmış değerlendirme alanı.
2. **Full Platform Modu (`(dashboard)`):** Onboarding tamamlandıktan sonra açılan, sol tarafta daraltılabilir Sidebar, üst arama/profil çubuğu, kişisel yol haritası ve görev sandboxtı barındıran platform arayüzü.

```
src/
├── app/
│   ├── (onboarding)/              # Distraction-Free Wizard Layout (No Sidebar)
│   │   ├── layout.tsx             # Stepper Header Layout
│   │   └── page.tsx               # 5-Step Stepper Controller
│   ├── (dashboard)/               # Main App Layout (With Sidebar)
│   │   ├── layout.tsx             # App Sidebar & Topbar
│   │   ├── roadmap/page.tsx       # Yol Haritası & Beceri Ağacı (Skill Tree)
│   │   ├── tasks/[id]/page.tsx    # Görev Detay & PR Takip Ekranı
│   │   ├── interview/page.tsx     # Serbest Mentör Sohbet / Soru-Cevap
│   │   └── settings/page.tsx      # Profil & Entegrasyon Ayarları
│   └── api/                       # Next.js API Routes (BFF / Webhook proxies)
```

---

## 2. Teknoloji Yığını (Frontend Tech Stack)

| Bileşen | Seçilen Teknoloji | Seçim Gerekçesi |
| :--- | :--- | :--- |
| **Framework** | Next.js 14+ (App Router) | React Server Components (RSC), SEO optimize edilmiş sayfalar, entegre routing. |
| **Dil** | TypeScript 5+ | Sıkı tip denetimi, IDE autocomplete ve sıfır runtime tip hatası. |
| **Styling** | Tailwind CSS + Vanilla CSS Tokens | Esnek utility-first sınıf yapısı, HSL CSS değişkenleri (`docs/design-system/DESIGN.md`). |
| **Tipografi** | Google Fonts (`Inter` + `Space Mono`) | Kod ve telemetri loglarında `Space Mono`, UI metinlerinde `Inter`. |
| **İstemci State** | Zustand | Onboarding adımları, quiz cevapları ve aktif modal durumları için hafif, hızlı global state. |
| **Sunucu State / Cache**| TanStack Query (React Query v5) | API isteklerinin önbelleklenmesi, optimistic updates ve otomatik revalidation. |
| **İkon Seti** | Lucide React | Modern, minimalist ve erişilebilir ikon kütüphanesi. |
| **Animasyonlar** | Framer Motion | Smooth adım geçişleri, telemetri log kaymaları ve mikro etkileşimler. |

---

## 3. Klasör Düzeni (Directory Structure)

```
frontend/
├── Dockerfile                      # Multi-stage Next.js standalone container
├── package.json
├── tailwind.config.js              # Daylight Observatory tema renk paleti ve fontları
├── tsconfig.json
├── src/
│   ├── app/                        # App Router Sayfa Düzeni
│   ├── components/                 # UI Bileşen Kataloğu
│   │   ├── ui/                     # Temel Tasarım Atomları (Button, Card, Badge, Input, Modal)
│   │   ├── onboarding/             # Step 1-5 Wizard Özel Bileşenleri
│   │   │   ├── Step1Connect.tsx    # Repo & CV Seçim Ekranı
│   │   │   ├── Step2Telemetry.tsx  # Canlı Telemetri Log Çubuğu (%0-%100)
│   │   │   ├── Step3Quiz.tsx       # 8-12 Teknik Soru Ekranı (Back/Next Gezintili)
│   │   │   ├── Step4Interview.tsx  # Sistem Tasarımı Sohbet Konsolu
│   │   │   └── Step5Report.tsx     # Bütünsel Rapor Kartı
│   │   ├── dashboard/              # Sidebar, Topbar, Skill Tree kartları
│   │   └── shared/                 # Code Block Syntax Highlighter, Markdown renderer
│   ├── hooks/                      # Custom React Hooks
│   │   ├── useTelemetrySSE.ts      # HTTP SSE Canlı Log Takip Hook'u
│   │   ├── useInterviewStream.ts   # Streaming AI Yanıt Hook'u
│   │   └── useAssessmentState.ts   # Zustand Onboarding state hook'u
│   ├── stores/                     # Zustand Store Tanımları
│   │   └── onboardingStore.ts      # Quiz cevapları, aktif adım ve telemetri state'i
│   ├── lib/                        # Yardımcı Araçlar & API Client
│   │   ├── api.ts                  # Axios / Fetch sarmalayıcısı (JWT Auth)
│   │   └── utils.ts                # Classnames (clsx + tailwind-merge)
│   └── types/                      # TypeScript Tip Tanımları
│       ├── assessment.ts           # 4-Eksen skor ve quiz tipleri
│       └── roadmap.ts              # Node, Task ve Submission tipleri
```

---

## 4. State Yönetimi ve Canlı Veri Akışları (SSE & Streaming)

### 4.1. Zustand Onboarding Store Yapısı (`onboardingStore.ts`)
```typescript
interface OnboardingState {
  currentStep: 1 | 2 | 3 | 4 | 5;
  selectedRepos: string[];
  uploadedCvFile: File | null;
  telemetryLogs: string[];
  telemetryProgress: number; // 0 - 100
  quizQuestions: Question[];
  quizAnswers: Record<string, string>;
  setStep: (step: 1 | 2 | 3 | 4 | 5) => void;
  appendTelemetryLog: (log: string, progress: number) => void;
  setQuizAnswer: (questionId: string, answer: string) => void;
}
```

### 4.2. Telemetri SSE Akışı (`useTelemetrySSE.ts`)
Adım 2 (Analiz Ekranı) sunucudaki ağır AST parsing sürecini anlık izler:
```typescript
export function useTelemetrySSE(assessmentId: string) {
  const appendLog = useOnboardingStore((s) => s.appendTelemetryLog);
  const setStep = useOnboardingStore((s) => s.setStep);

  useEffect(() => {
    const eventSource = new EventSource(`/api/v1/onboarding/telemetry/${assessmentId}`);
    
    eventSource.onmessage = (event) => {
      const data = JSON.parse(event.data);
      appendLog(data.log, data.progress);
      
      if (data.progress === 100) {
        eventSource.close();
        setTimeout(() => setStep(3), 800); // Otomatik Adım 3'e geçiş
      }
    };

    return () => eventSource.close();
  }, [assessmentId]);
}
```

---

## 5. Docker & Dağıtım Mimarisi (Multi-Stage Production `Dockerfile`)

Next.js uygulamasının VPS üzerinde minimum bellek (RAM) harcaması için `output: 'standalone'` modunda multi-stage `Dockerfile` inşa edilir:

```dockerfile
# 1. Base Stage
FROM node:20-alpine AS base

# 2. Dependencies Stage
FROM base AS deps
RUN apk add --no-cache libc6-compat
WORKDIR /app

COPY package.json package-lock.json ./
RUN npm ci

# 3. Builder Stage
FROM base AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .

ENV NEXT_TELEMETRY_DISABLED=1
ENV NODE_ENV=production

RUN npm run build

# 4. Runner Stage (Production image)
FROM base AS runner
WORKDIR /app

ENV NODE_ENV=production
ENV NEXT_TELEMETRY_DISABLED=1

RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nextjs

COPY --from=builder /app/public ./public
COPY --from=builder --chown=nextjs:nodejs /app/.next/standalone ./
COPY --from=builder --chown=nextjs:nodejs /app/.next/static ./.next/static

USER nextjs

EXPOSE 3000
ENV PORT=3000
ENV HOSTNAME="0.0.0.0"

CMD ["node", "server.js"]
```

---

## 6. Tasarım ve Erişilebilirlik Standartları

1. **Daylight Observatory Tema:** Karanlık gözlemevi estetiği; koyu lacivert/siyah arka plan (`#0B0F19`), neon cam göbeği vurguları (`#00F2FE`), yumuşak gradyanlar ve glassmorphism kartları.
2. **Keyboard Navigation & Accessiblity:** Quiz soruları ve modal kapatma tuşlarında `Tab` ve `Esc` klavye navigasyon desteği.
3. **Responsive Stepper:** Mobil ekranlarda stepper header metinleri gizlenir, sadece sayısal adımlar ve % göstergesi yer alır.
