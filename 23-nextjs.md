# 23 · Next.js ve Server Components

> **Konu özeti:** İstemci tarafı render'ın sınırları, render stratejileri ve hydration, Next.js App Router ve dosya tabanlı yönlendirme, Server Components, Client Components ve `"use client"` sınırı, sunucuda veri çekme, Suspense ile streaming ve `use`, Server Functions (Server Actions) ve güvenlik, Next.js 16 önbellekleme (`"use cache"`, `cacheLife`, `cacheTag`, `updateTag`) ve kısmi ön-render, metadata, Route Handlers, `proxy.ts`, `next/image`, `next/font`, ortam değişkenleri ve hangi aracın ne zaman seçileceği.
> **Sürüm:** Next.js 16.x (App Router)

---

## İçindekiler

1. [Sezgisel Bakış](#1-sezgisel-bakış)
2. [Render Stratejileri ve Hydration](#2-render-stratejileri-ve-hydration)
3. [Next.js'e Giriş](#3-nextjse-giriş)
4. [Server Components](#4-server-components-)
5. [Client Components ve `"use client"` Sınırı](#5-client-components-ve-use-client-sınırı-)
6. [Server Components'te Veri Çekme](#6-server-componentste-veri-çekme)
7. [Streaming: Suspense ile Parça Parça Gönderim](#7-streaming-suspense-ile-parça-parça-gönderim-)
8. [Server Functions (Server Actions)](#8-server-functions-server-actions-)
9. [Önbellekleme ve Kısmi Ön-Render](#9-önbellekleme-ve-kısmi-ön-render-nextjs-16)
10. [Metadata ve SEO](#10-metadata-ve-seo)
11. [Diğer Önemli Parçalar](#11-diğer-önemli-parçalar)
12. [Ne Zaman Hangisi?](#12-ne-zaman-hangisi)
13. [Her Şey Bir Arada: Next.js Blog](#13-her-şey-bir-arada-nextjs-blog)
14. [Kapanış: Bu Yolculukta Neler Öğrendin?](#14-kapanış-bu-yolculukta-neler-öğrendin)
15. [Özet](#15-özet)

---

## 1. Sezgisel Bakış

### İstemci tarafı render (CSR) — şimdiye kadarki model

```
1. Tarayıcı sayfayı ister
2. Sunucu BOŞ HTML gönderir:  <div id="root"></div>  + büyük bir JS dosyası
3. Tarayıcı JS'i indirir, çalıştırır
4. React bileşenleri tarayıcıda render eder → ekran ilk kez dolar
5. Bileşenler veri ister → "Yükleniyor..."
6. Veri gelir → ekran son halini alır
```

### CSR'nin sorunları

- **İlk ekran yavaş:** JS indirilip çalışana kadar boş/iskelet sayfa (LCP).
- **Şelaleler:** JS → render → veri → sonraki bileşen → başka veri...
- **SEO ve paylaşım:** Arama motorları ve önizlemeler boş HTML görür.
- **Her şey tarayıcıya gider:** Kütüphaneler bundle'ı büyütür; veritabanı için ayrı API gerekir.

### Restoran benzetmesi

| Model | Benzetme |
|---|---|
| **CSR** | Müşteriye malzeme kutusu ve tarif kitabı gönder: "Masada kendin pişir." |
| **SSR** | Yemeği mutfakta pişir, **hazır tabakla** gönder; tuz/sos (etkileşim) masada |
| **Server Components** | Bazı bileşenler **sadece mutfakta** var olur (tarifleri masaya gitmez), bazıları masaya da gider |

> **Next.js**, bu modeli eksiksiz uygulayan en yaygın React framework'üdür.

> SPA'lar kötü değildir: Giriş arkasındaki paneller ve yoğun etkileşimli uygulamalar için Vite + React Router + TanStack Query hâlâ mükemmel. Next.js içerik ağırlıklı, SEO ve ilk açılış hızının kritik olduğu uygulamalarda parlar.

---

## 2. Render Stratejileri ve Hydration

| Strateji | HTML nerede, ne zaman üretilir | Artısı | Eksisi |
|---|---|---|---|
| **CSR** | Tarayıcıda, her ziyarette | Basit, sunucu gerektirmez | Yavaş ilk ekran, zayıf SEO |
| **SSR** | Sunucuda, **her istekte** | Hızlı ilk ekran, güncel veri | Her istekte sunucu işi |
| **SSG** | Sunucuda, **build'de bir kez** | En hızlı (CDN) | Veri değişince yeniden build |
| **ISR / önbellekli render** | Build'de/ilk istekte üretilir, aralıklarla/olaylarla yenilenir | Statik hız + güncellenebilir veri | Kısa süre bayat veri |
| **Streaming** | Sunucuda, **parça parça** gönderilir | Yavaş bölüm sayfayı bekletmez | |

> Modern Next.js tek bir sayfada bunları **karıştırır.**

### Hydration (canlandırma)

```
1. Sunucu HTML gönderir → kullanıcı içeriği GÖRÜR
2. Tarayıcı JS'i indirir
3. React mevcut HTML'e "bağlanır": olay yöneticileri eklenir, state başlar → ETKİLEŞİMLİ
```

- **Kural:** Sunucu HTML'i ile istemcideki ilk render **aynı** olmalı; değilse **hydration uyuşmazlığı** uyarısı.
- Sunucuda anlamlı çıktı üretemeyen bileşenler için React 19.3 **`browser()`** API'si *(Konu 20.11)*.

---

## 3. Next.js'e Giriş

### Kurulum

```bash
npx create-next-app@latest notlar-uygulamasi
```

```bash
npm run dev     # http://localhost:3000 (Turbopack)
npm run build   # production build
npm start       # production sunucusu
```

### İki yönlendirici

| Yönlendirici | Klasör | Durum |
|---|---|---|
| **App Router** | `app/` | Server Components, streaming, Server Functions — **yeni projeler için** |
| **Pages Router** | `pages/` | Eski sistem (`getServerSideProps`, `getStaticProps`); hâlâ destekleniyor |

### Dosya tabanlı yönlendirme ⭐

**Klasör yapısı = URL yapısı:**

```
app/
├── layout.tsx              ← Kök düzen (<html>, <body>)
├── page.tsx                ← "/"
├── globals.css
├── hakkimizda/
│   └── page.tsx            ← "/hakkimizda"
└── blog/
    ├── layout.tsx          ← /blog altını sarar
    ├── page.tsx            ← "/blog"
    ├── loading.tsx         ← yüklenirken (otomatik <Suspense>)
    ├── error.tsx           ← hata (otomatik Error Boundary)
    ├── yeni/
    │   └── page.tsx        ← "/blog/yeni"
    └── [slug]/
        ├── page.tsx        ← "/blog/react-19", ...
        └── not-found.tsx   ← notFound() çağrılınca
```

| Dosya | Görevi | React Router karşılığı |
|---|---|---|
| `page.tsx` | URL'nin sayfası (klasör ancak `page` varsa URL olur) | `Component` |
| `layout.tsx` | Altındaki sayfaları sarar; geçişlerde yeniden oluşturulmaz | Düzen rotası + `<Outlet />` |
| `loading.tsx` | Yüklenirken (otomatik `<Suspense>`) | `HydrateFallback` / `useNavigation` |
| `error.tsx` | Hata sınırı | `ErrorBoundary` |
| `not-found.tsx` | `notFound()` veya eşleşmeyen URL | `*` rotası |
| `route.ts` | API uç noktası | — |

### Kök düzen

```tsx
// app/layout.tsx
import type { ReactNode } from "react";
import type { Metadata } from "next";
import Link from "next/link";
import "./globals.css";

export const metadata: Metadata = {
  title: { default: "Notlarım", template: "%s · Notlarım" },
  description: "Next.js ile not defteri",
};

export default function RootLayout({ children }: { children: ReactNode }) {
  return (
    <html lang="tr">
      <body className="min-h-dvh bg-slate-50 text-slate-900">
        <header className="border-b bg-white">
          <nav className="mx-auto flex max-w-4xl gap-4 p-4">
            <Link href="/" className="font-bold">📝 Notlarım</Link>
            <Link href="/blog">Blog</Link>
          </nav>
        </header>
        <main className="mx-auto max-w-4xl p-4">{children}</main>
      </body>
    </html>
  );
}
```

- `<Outlet />` yerine **`children`.**

### Dinamik segmentler

```tsx
// app/blog/[slug]/page.tsx
export default async function YaziSayfasi({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params;   // params bir Promise
  return <h1>Yazı: {slug}</h1>;
}
```

| Sözdizimi | Anlamı |
|---|---|
| `[slug]` | Tek parametre |
| `[...yol]` | Geri kalan her şey (splat) |
| `[[...yol]]` | Opsiyonel splat |
| `(grup)` | **Rota grubu:** URL'ye eklenmez; dosya gruplama / farklı düzen |

- `params` ve `searchParams` **asenkrondur** (`await`).

### Gezinme

```tsx
import Link from "next/link";
<Link href="/blog/react-19">Oku</Link>   // otomatik prefetch
```

```tsx
"use client";
import { useRouter, usePathname, useSearchParams } from "next/navigation";

const router = useRouter();
router.push("/blog");      // programla yönlendirme
router.refresh();          // sunucu bileşenlerini yeniden çek
const yol = usePathname();
```

> ⚠️ App Router'da Hook'lar **`next/navigation`**'dan; `next/router` Pages Router içindir.

---

## 4. Server Components ⭐

> App Router'da **her bileşen varsayılan olarak bir Server Component'tir.**

```tsx
// app/blog/page.tsx
import { db } from "@/lib/db";

export default async function BlogSayfasi() {
  const yazilar = await db.yazilariGetir();   // doğrudan veritabanı

  return (
    <section>
      <h1 className="text-2xl font-bold">Blog</h1>
      <ul>
        {yazilar.map((y) => (
          <li key={y.slug}>
            <a href={`/blog/${y.slug}`}>{y.baslik}</a>
            <small> — {new Date(y.tarih).toLocaleDateString("tr-TR")}</small>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

### Olağandışı olanlar

- Bileşen bir **`async` fonksiyon.**
- Veri **doğrudan** alınır: veritabanı, dosya sistemi, gizli API anahtarlı servisler; API katmanı gerekmez.
- **Yükleniyor/hata state'i yok.**
- **Kodu tarayıcıya hiç gitmez**; sadece ürettiği sonuç (HTML + "RSC payload") gider.

### Yapılamayanlar / yapılabilenler

| ❌ Yapılamaz | ✅ Yapılabilir |
|---|---|
| `useState`, `useReducer`, `useEffect`, `useRef` | `async/await` |
| Olay yöneticileri (`onClick`...) | Veritabanı, dosya sistemi |
| `window`, `localStorage`, `document` | Gizli ortam değişkenleri |
| Context kullanmak | Sunucu kütüphaneleri |

### Gizli bilgileri korumak

```ts
// lib/db.ts
import "server-only";   // Client Component'e import edilirse build HATASI
```

---

## 5. Client Components ve `"use client"` Sınırı ⭐

```tsx
// app/blog/[slug]/BegenButonu.tsx
"use client";

import { useState } from "react";

export function BegenButonu({ baslangic }: { baslangic: number }) {
  const [sayi, setSayi] = useState(baslangic);
  return <button onClick={() => setSayi((s) => s + 1)}>❤️ {sayi}</button>;
}
```

```tsx
// app/blog/[slug]/page.tsx — Server Component
import { BegenButonu } from "./BegenButonu";

export default async function YaziSayfasi({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params;
  const yazi = await db.yaziGetir(slug);
  return (
    <article>
      <h1>{yazi.baslik}</h1>
      <p>{yazi.icerik}</p>                       {/* statik: JS yok */}
      <BegenButonu baslangic={yazi.begeni} />    {/* etkileşimli: JS'i tarayıcıya gider */}
    </article>
  );
}
```

> ⚠️ **"Client Component" ismi yanıltıcı:** Client Component'ler de **sunucuda render edilir** (ilk HTML), sonra hydrate olur. Fark: **kodu tarayıcıya gönderilir.** Dersin başından beri yazdığımız tüm bileşenler Client Component'ti.

### Sınır nasıl çalışır?

`"use client"` bir **modül sınırıdır**: o dosya ve **import ettiği her şey** istemci koduna dahil olur.

```
app/blog/[slug]/page.tsx          (Server)
├── YaziBasligi                    (Server)
├── YaziIcerigi                    (Server)
└── BegenButonu.tsx  "use client"  (Client) ─┐
    └── Ikon.tsx                   (Client) ◄┘ import edildiği için istemci
```

### Etkileşimi yapraklara it ⭐

```tsx
// ❌ Tüm sayfa istemci: Server Component avantajları kaybolur
"use client";
export default function YaziSayfasi() { /* ... */ }

// ✅ Sadece etkileşimli küçük parça istemci
// page.tsx (Server) → <BegenButonu /> (Client)
```

### Server Component'i Client Component'e `children` olarak vermek

Client Component, Server Component'i **import edemez**; ama `children`/prop olarak **alabilir**:

```tsx
// AcilirPanel.tsx
"use client";
import { useState, type ReactNode } from "react";

export function AcilirPanel({ baslik, children }: { baslik: string; children: ReactNode }) {
  const [acik, setAcik] = useState(false);
  return (
    <section>
      <button onClick={() => setAcik(!acik)}>{baslik} {acik ? "▲" : "▼"}</button>
      {acik && children}
    </section>
  );
}
```

```tsx
// page.tsx — Server
<AcilirPanel baslik="Yorumlar">
  <YorumListesi slug={slug} />   {/* Server Component; kodu istemciye GİTMEZ */}
</AcilirPanel>
```

### Sunucudan istemciye props: serileştirilebilir olmalı

- ✅ string, number, boolean, null, düz nesne/dizi, `Date`, `Map`, `Set`, JSX, **Promise'ler**, **Server Function'lar**
- ❌ Normal fonksiyonlar (`onClick={() => ...}`), sınıf örnekleri

### Context ve provider'lar

```tsx
// app/Providers.tsx
"use client";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { useState, type ReactNode } from "react";
import { TemaProvider } from "./TemaContext";

export function Providers({ children }: { children: ReactNode }) {
  const [queryClient] = useState(() => new QueryClient());   // her kullanıcı için ayrı
  return (
    <QueryClientProvider client={queryClient}>
      <TemaProvider>{children}</TemaProvider>
    </QueryClientProvider>
  );
}
```

```tsx
// app/layout.tsx (Server)
<body><Providers>{children}</Providers></body>
```

- `children` sunucudan geldiği için sayfalar Server Component olarak kalır.

> 📌 React 19.3: Server Component'ler `'use client'` modülünden aktarılan Context'i sarmalayıcı Provider olmadan **doğrudan** render edebilir.

### Karar tablosu

| İhtiyaç | Server | Client |
|---|---|---|
| Veritabanı / gizli anahtar / dosya sistemi | ✅ | ❌ |
| Büyük kütüphaneyle sadece görüntüleme | ✅ (JS istemciye gitmez) | |
| `useState`, `useReducer`, `useEffect` | ❌ | ✅ |
| `onClick`, `onChange`, form etkileşimi | ❌ | ✅ |
| `window`, `localStorage` | ❌ | ✅ |
| Context kullanmak | ❌ | ✅ |

> **Varsayılan Server, gerektiğinde Client.**

---

## 6. Server Components'te Veri Çekme

### Doğrudan await

```tsx
export default async function KullaniciSayfasi({ params }: { params: Promise<{ id: string }> }) {
  const { id } = await params;
  const res = await fetch(`https://api.ornek.com/kullanicilar/${id}`);
  if (!res.ok) throw new Error("Kullanıcı yüklenemedi");   // → en yakın error.tsx
  const kullanici = await res.json();
  return <h1>{kullanici.ad}</h1>;
}
```

### Paralel istekler

```tsx
// ❌ Sıralı
const kullanici = await kullaniciGetir(id);
const siparisler = await siparisleriGetir(id);

// ✅ Paralel
const [kullanici, siparisler] = await Promise.all([kullaniciGetir(id), siparisleriGetir(id)]);
```

### `notFound()` ve `redirect()`

```tsx
import { notFound, redirect } from "next/navigation";

const yazi = await db.yaziGetir(slug);
if (!yazi) notFound();                // → not-found.tsx
if (yazi.taslak) redirect("/blog");
```

---

## 7. Streaming: Suspense ile Parça Parça Gönderim ⭐

```tsx
import { Suspense } from "react";

export default async function YaziSayfasi({ params }: { params: Promise<{ slug: string }> }) {
  const { slug } = await params;
  const yazi = await db.yaziGetir(slug);   // hızlı

  return (
    <article>
      <h1>{yazi.baslik}</h1>
      <p>{yazi.icerik}</p>

      <Suspense fallback={<p>Yorumlar yükleniyor...</p>}>
        <Yorumlar slug={slug} />           {/* yavaş; sayfayı BEKLETMEZ */}
      </Suspense>
    </article>
  );
}

async function Yorumlar({ slug }: { slug: string }) {
  const yorumlar = await db.yorumlariGetir(slug);
  return <ul>{yorumlar.map((y) => <li key={y.id}>{y.metin}</li>)}</ul>;
}
```

```
0 ms     : Başlık + içerik + "Yorumlar yükleniyor..." gönderilir → kullanıcı okumaya başlar
2000 ms  : Yorumlar hazır → aynı bağlantıdan gönderilir, fallback'in yerine geçer
```

- **`loading.tsx`** segmentin tamamını otomatik `<Suspense>` ile sarar.

### Promise'i istemci bileşenine aktarmak: `use`

```tsx
// page.tsx (Server)
export default function Sayfa() {
  const istatistikPromise = db.istatistikleriGetir();   // await YOK: istek başladı
  return (
    <Suspense fallback={<p>Grafik yükleniyor...</p>}>
      <EtkilesimliGrafik veriPromise={istatistikPromise} />
    </Suspense>
  );
}
```

```tsx
// EtkilesimliGrafik.tsx
"use client";
import { use, useState } from "react";

export function EtkilesimliGrafik({ veriPromise }: { veriPromise: Promise<Istatistik[]> }) {
  const veri = use(veriPromise);   // Promise sunucuda oluşturuldu → render'lar arası sabit
  const [secili, setSecili] = useState<string | null>(null);
  // ...
}
```

> Veri isteği sunucuda erkenden başlar (şelale yok), grafik etkileşimli kalır.

---

## 8. Server Functions (Server Actions) ⭐

`"use server"` ile işaretlenen fonksiyon **sunucuda çalışır** ama istemciden **normal fonksiyon gibi çağrılır**; Next.js arada HTTP isteğini kendisi oluşturur.

```ts
// app/blog/actions.ts
"use server";

import { z } from "zod";
import { redirect } from "next/navigation";
import { updateTag } from "next/cache";
import { db } from "@/lib/db";

const YaziSemasi = z.object({
  baslik: z.string().trim().min(3, "Başlık en az 3 karakter olmalı."),
  icerik: z.string().trim().min(10, "İçerik en az 10 karakter olmalı."),
});

export type YaziFormDurumu = {
  hatalar?: { baslik?: string[]; icerik?: string[] };
  degerler?: { baslik?: string; icerik?: string };
};

export async function yaziOlustur(_onceki: YaziFormDurumu, formData: FormData): Promise<YaziFormDurumu> {
  const degerler = {
    baslik: String(formData.get("baslik") ?? ""),
    icerik: String(formData.get("icerik") ?? ""),
  };
  const sonuc = YaziSemasi.safeParse(degerler);
  if (!sonuc.success) {
    return { hatalar: z.flattenError(sonuc.error).fieldErrors, degerler };
  }

  const yazi = await db.yaziEkle(sonuc.data);
  updateTag("yazilar");              // önbelleği geçersiz kıl
  redirect(`/blog/${yazi.slug}`);
}
```

```tsx
// app/blog/yeni/YaziFormu.tsx
"use client";

import { useActionState } from "react";
import { yaziOlustur, type YaziFormDurumu } from "../actions";

export function YaziFormu() {
  const [durum, formAction, isPending] = useActionState<YaziFormDurumu, FormData>(yaziOlustur, {});

  return (
    <form action={formAction} className="space-y-3">
      <input name="baslik" defaultValue={durum.degerler?.baslik} placeholder="Başlık" />
      {durum.hatalar?.baslik && <p className="text-red-600">{durum.hatalar.baslik[0]}</p>}

      <textarea name="icerik" defaultValue={durum.degerler?.icerik} rows={8} />
      {durum.hatalar?.icerik && <p className="text-red-600">{durum.hatalar.icerik[0]}</p>}

      <button disabled={isPending}>{isPending ? "Yayınlanıyor..." : "Yayınla"}</button>
    </form>
  );
}
```

### Kazanımlar

- **API uç noktası yok:** `fetch("/api/...")` yerine doğrudan fonksiyon çağrısı.
- **Tip güvenliği:** İstemci ve sunucu aynı tipleri paylaşır.
- **Progressive enhancement:** Server Component içinde `<form action={yaziOlustur}>` JS yüklenmeden bile çalışır.
- `useActionState`, `useFormStatus`, `useOptimistic` aynen çalışır *(Konu 20)*.

### ⚠️ Güvenlik: Server Function'lar herkese açık uç noktalardır

- Gelen veri **mutlaka doğrulanır** (Zod).
- **Yetki kontrolü** her fonksiyonda yapılır; butonu gizlemek yetmez.

```ts
export async function yaziSil(slug: string) {
  "use server";
  const kullanici = await oturumuGetir();
  if (!kullanici || !(await yazarMi(kullanici, slug))) throw new Error("Yetkisiz işlem");
  await db.yaziSil(slug);
  updateTag("yazilar");
}
```

---

## 9. Önbellekleme ve Kısmi Ön-Render (Next.js 16)

> Next.js 16 ile önbellek **açık (opt-in)**: varsayılan her şey **dinamik**, neyin önbelleğe alınacağı açıkça söylenir. Ayrıntılar hızla geliştiği için uygulamada **resmi dokümantasyona** bakılmalıdır.

### Cache Components'i açmak

```ts
// next.config.ts
import type { NextConfig } from "next";
const nextConfig: NextConfig = { cacheComponents: true };
export default nextConfig;
```

### `"use cache"`

```ts
// lib/veri.ts
import { cacheLife, cacheTag } from "next/cache";
import { db } from "./db";

export async function yazilariGetir() {
  "use cache";
  cacheLife("hours");        // profiller: "minutes", "hours", "days", "max"...
  cacheTag("yazilar");       // geçersiz kılma etiketi
  return db.yazilariGetir();
}
```

| TanStack Query (istemci) | Next.js (sunucu) |
|---|---|
| `queryKey` | Fonksiyon + argümanlar / `cacheTag` |
| `staleTime` | `cacheLife` |
| `invalidateQueries` | `updateTag` / `revalidateTag` |

### Geçersiz kılma

```ts
import { revalidateTag, updateTag } from "next/cache";

updateTag("yazilar");              // HEMEN geçersiz; kullanıcı bu istekte yeni veriyi görür
revalidateTag("yazilar", "max");   // arka planda yenile (stale-while-revalidate)
```

| Durum | Araç |
|---|---|
| Kullanıcı kendi değişikliğini hemen görmeli | `updateTag` |
| Başkalarının değişiklikleri biraz gecikebilir | `revalidateTag` |

### Kısmi ön-render (Partial Prerendering)

```
┌─────────── Statik iskelet (build'de veya önbellekten, ANINDA) ──────────────┐
│  Header, menü, yazı içeriği ("use cache")                                     │
│  ┌────────── Dinamik delik (istekte üretilir, STREAMING ile gelir) ─────────┐ │
│  │  <Suspense> → kullanıcıya özel öneriler, sepet, canlı yorumlar         │ │
│  └─────────────────────────────────────────────────────────────────────────┘ │
└───────────────────────────────────────────────────────────────────────────────┘
```

- Statik kısım CDN'den anında; dinamik kısımlar aynı yanıtta akarak gelir.
- `cookies()`, `headers()`, `searchParams` okuyan kısımlar dinamiktir: `"use cache"` içinde kullanılamaz, `<Suspense>` içinde olmalıdır.
- Bir bileşenin her istekte üretilmesini açıkça istemek: `await connection()` (`next/server`).

> 📌 Next.js 13–15'teki `fetch` varsayılan önbelleği, `export const revalidate = 60`, `fetch(url, { next: { revalidate } })`, `unstable_cache` yaklaşımlarının yerini Next.js 16'da `"use cache"` aldı.

---

## 10. Metadata ve SEO

```tsx
// Statik
export const metadata: Metadata = {
  title: "Blog",
  description: "React ve Next.js üzerine yazılar",
};
```

```tsx
// Dinamik
import type { Metadata } from "next";

export async function generateMetadata(
  { params }: { params: Promise<{ slug: string }> }
): Promise<Metadata> {
  const { slug } = await params;
  const yazi = await yaziGetir(slug);
  if (!yazi) return { title: "Bulunamadı" };
  return {
    title: yazi.baslik,                                  // "%s · Notlarım" şablonuna girer
    description: yazi.icerik.slice(0, 150),
    openGraph: { title: yazi.baslik, type: "article" },  // sosyal medya önizlemesi
  };
}
```

- `<head>`'e **sunucuda** yazılır → arama motorları ve önizlemeler hemen görür.
- React 19'un bileşen içi `<title>`'ı da çalışır; Next.js'te `metadata` API'si tercih edilir.

---

## 11. Diğer Önemli Parçalar

### Route Handlers (API uç noktaları)

```ts
// app/api/yazilar/route.ts → GET /api/yazilar
import { db } from "@/lib/db";

export async function GET() {
  const yazilar = await db.yazilariGetir();
  return Response.json(yazilar);
}

export async function POST(request: Request) {
  const govde = await request.json();
  // doğrula, yetki kontrol et, kaydet...
  return Response.json({ tamam: true }, { status: 201 });
}
```

> Kendi formların için Server Function; Route Handler'lar **dışarıya** açılan API'ler (mobil, webhook) içindir.

### `proxy.ts` (eski adıyla `middleware.ts`)

Her istekten **önce** çalışır: oturum kontrolü, dil algılama, yönlendirmeler.

```ts
// proxy.ts (proje kökünde)
import { NextResponse, type NextRequest } from "next/server";

export function proxy(request: NextRequest) {
  const oturum = request.cookies.get("oturum");
  if (!oturum && request.nextUrl.pathname.startsWith("/panel")) {
    return NextResponse.redirect(new URL("/giris", request.url));
  }
  return NextResponse.next();
}

export const config = { matcher: ["/panel/:path*"] };
```

> `proxy.ts` iyimser bir ilk kontroldür; asıl yetki kontrolü veriye erişilen yerde yapılır.

### `next/image`

```tsx
import Image from "next/image";
<Image src="/hero.jpg" alt="Kapak" width={1200} height={600} priority />   {/* ana görsel */}
```

- Boyutlandırma, WebP/AVIF, lazy loading, CLS önleme otomatik *(Konu 19.13)*.

### `next/font`

```tsx
import { Inter } from "next/font/google";
const inter = Inter({ subsets: ["latin", "latin-ext"] });   // latin-ext: Türkçe karakterler
<body className={inter.className}>
```

- Fontlar build'de indirilip kendi sunucundan sunulur; ek bağlantı ve kayma yok.

### Ortam değişkenleri

```bash
DATABASE_URL=postgres://...          # sadece sunucu
NEXT_PUBLIC_HARITA_ANAHTARI=abc123   # tarayıcıya da gider (gizli bilgi KONMAZ)
```

### `error.tsx`

Hata sınırları **Client Component** olmalıdır:

```tsx
"use client";
export default function Hata({ error, reset }: { error: Error; reset: () => void }) {
  return (
    <div role="alert">
      <p>Bir hata oluştu: {error.message}</p>
      <button onClick={reset}>Tekrar dene</button>
    </div>
  );
}
```

---

## 12. Ne Zaman Hangisi?

| Seçenek | En uygun olduğu yer |
|---|---|
| **Vite + React Router (Data) + TanStack Query** | Giriş arkasındaki SPA'lar: paneller, iç araçlar, SaaS; SEO önemsiz; statik barındırma |
| **Next.js (App Router)** | İçerik ağırlıklı siteler, e-ticaret, bloglar; SEO ve ilk açılış kritik; tam yığın |
| **React Router Framework modu** | Next.js benzeri yetenekler, web standartlarına yakın API; Data modundan kademeli geçiş |
| **TanStack Start** | TanStack Router tabanlı tam yığın; güçlü tip güvenliği |
| **Astro** | Çoğunlukla statik içerik; React'i sadece gereken yerde "ada" olarak kullanır |

### Yayınlama

- **Vercel** en sorunsuz deneyim; Netlify, Cloudflare, AWS de destekler; `next start` veya Docker ile kendi sunucunda.
- Sunucu özellikleri (SSR, Server Functions) kullanıldığında **çalışan bir sunucu** gerekir; statik barındırma yetmez.

---

## 13. Her Şey Bir Arada: Next.js Blog

Server Component'lerle veri okuma, `"use cache"` + `cacheTag`, dinamik rota + `generateMetadata`, `notFound`, `loading.tsx`, `<Suspense>` ile akan yorumlar, `useOptimistic` ile iyimser beğeni, `useActionState` + Zod ile yazı formu, `updateTag` + `redirect`, `server-only`.

### Yapılandırma

```ts
// ============ next.config.ts ============
import type { NextConfig } from "next";
const nextConfig: NextConfig = { cacheComponents: true };
export default nextConfig;
```

### Sahte veritabanı

```ts
// ============ lib/db.ts ============
import "server-only";

export type Yazi = { slug: string; baslik: string; icerik: string; tarih: string; begeni: number };
export type Yorum = { id: string; slug: string; yazar: string; metin: string };

const bekle = (ms: number) => new Promise((r) => setTimeout(r, ms));

const yazilar: Yazi[] = [
  {
    slug: "server-components",
    baslik: "Server Components'e Giriş",
    icerik: "Bileşenlerin bir kısmı sunucuda çalışıyor ve kodları tarayıcıya hiç gitmiyor...",
    tarih: "2026-10-20",
    begeni: 12,
  },
  {
    slug: "react-19-3",
    baslik: "React 19.3 Yenilikleri",
    icerik: "View Transitions ve Fragment refs artık kararlı...",
    tarih: "2026-10-22",
    begeni: 8,
  },
];

const yorumlar: Yorum[] = [
  { id: "1", slug: "server-components", yazar: "Ayşe", metin: "Çok net anlatım, teşekkürler!" },
  { id: "2", slug: "server-components", yazar: "Mehmet", metin: "use client sınırı şimdi oturdu." },
];

function slugOlustur(baslik: string) {
  return baslik
    .toLocaleLowerCase("tr")
    .replace(/ç/g, "c").replace(/ğ/g, "g").replace(/ı/g, "i")
    .replace(/ö/g, "o").replace(/ş/g, "s").replace(/ü/g, "u")
    .replace(/[^a-z0-9]+/g, "-")
    .replace(/(^-|-$)/g, "");
}

export const db = {
  async yazilariGetir() {
    await bekle(200);
    return yazilar.toSorted((a, b) => b.tarih.localeCompare(a.tarih));
  },
  async yaziGetir(slug: string) {
    await bekle(100);
    return yazilar.find((y) => y.slug === slug) ?? null;
  },
  async yorumlariGetir(slug: string) {
    await bekle(1500);   // bilerek yavaş: streaming'i görmek için
    return yorumlar.filter((y) => y.slug === slug);
  },
  async yaziEkle(veri: { baslik: string; icerik: string }) {
    await bekle(300);
    const yazi: Yazi = {
      slug: slugOlustur(veri.baslik),
      ...veri,
      tarih: new Date().toISOString().slice(0, 10),
      begeni: 0,
    };
    yazilar.push(yazi);
    return yazi;
  },
  async begen(slug: string) {
    await bekle(400);
    const yazi = yazilar.find((y) => y.slug === slug);
    if (!yazi) throw new Error("Yazı bulunamadı");
    yazi.begeni += 1;
    return yazi.begeni;
  },
};
```

### Önbellekli okuma fonksiyonları

```ts
// ============ lib/veri.ts ============
import "server-only";
import { cacheLife, cacheTag } from "next/cache";
import { db } from "./db";

export async function yazilariGetir() {
  "use cache";
  cacheLife("hours");
  cacheTag("yazilar");
  return db.yazilariGetir();
}

export async function yaziGetir(slug: string) {
  "use cache";
  cacheLife("hours");
  cacheTag("yazilar", `yazi-${slug}`);
  return db.yaziGetir(slug);
}
```

### Server Functions

```ts
// ============ app/blog/actions.ts ============
"use server";

import { z } from "zod";
import { redirect } from "next/navigation";
import { updateTag } from "next/cache";
import { db } from "@/lib/db";

const YaziSemasi = z.object({
  baslik: z.string().trim().min(3, "Başlık en az 3 karakter olmalı."),
  icerik: z.string().trim().min(10, "İçerik en az 10 karakter olmalı."),
});

export type YaziFormDurumu = {
  hatalar?: { baslik?: string[]; icerik?: string[] };
  degerler?: { baslik?: string; icerik?: string };
};

export async function yaziOlustur(
  _onceki: YaziFormDurumu,
  formData: FormData
): Promise<YaziFormDurumu> {
  // Gerçek uygulamada: önce oturum ve yetki kontrolü!
  const degerler = {
    baslik: String(formData.get("baslik") ?? ""),
    icerik: String(formData.get("icerik") ?? ""),
  };
  const sonuc = YaziSemasi.safeParse(degerler);
  if (!sonuc.success) {
    return { hatalar: z.flattenError(sonuc.error).fieldErrors, degerler };
  }

  const yazi = await db.yaziEkle(sonuc.data);
  updateTag("yazilar");              // liste önbelleği hemen geçersiz
  redirect(`/blog/${yazi.slug}`);
}

export async function begen(slug: string): Promise<number> {
  const yeni = await db.begen(slug);
  updateTag(`yazi-${slug}`);         // bu yazının önbelleği hemen geçersiz
  return yeni;
}
```

### Kök düzen ve ana sayfa

```tsx
// ============ app/layout.tsx ============
import type { ReactNode } from "react";
import type { Metadata } from "next";
import Link from "next/link";
import { Inter } from "next/font/google";
import "./globals.css";

const inter = Inter({ subsets: ["latin", "latin-ext"] });

export const metadata: Metadata = {
  title: { default: "Notlarım", template: "%s · Notlarım" },
  description: "Next.js ve Server Components ile blog",
};

export default function RootLayout({ children }: { children: ReactNode }) {
  return (
    <html lang="tr">
      <body className={`${inter.className} min-h-dvh bg-slate-50 text-slate-900`}>
        <header className="border-b bg-white">
          <nav className="mx-auto flex max-w-3xl items-center gap-4 p-4">
            <Link href="/" className="font-bold">📝 Notlarım</Link>
            <Link href="/blog" className="text-slate-600 hover:text-slate-900">Blog</Link>
            <Link href="/blog/yeni" className="ml-auto rounded bg-indigo-600 px-3 py-1.5 text-white">
              Yeni yazı
            </Link>
          </nav>
        </header>
        <main className="mx-auto max-w-3xl p-4">{children}</main>
      </body>
    </html>
  );
}
```

```tsx
// ============ app/page.tsx ============
import { redirect } from "next/navigation";

export default function AnaSayfa() {
  redirect("/blog");
}
```

### Blog listesi (Server Component, önbellekli)

```tsx
// ============ app/blog/page.tsx ============
import Link from "next/link";
import type { Metadata } from "next";
import { yazilariGetir } from "@/lib/veri";

export const metadata: Metadata = { title: "Blog" };

export default async function BlogSayfasi() {
  const yazilar = await yazilariGetir();

  return (
    <section>
      <h1 className="mb-4 text-3xl font-bold">Blog</h1>
      <ul className="space-y-3">
        {yazilar.map((y) => (
          <li key={y.slug} className="rounded-lg border bg-white p-4">
            <Link href={`/blog/${y.slug}`} className="text-lg font-semibold hover:underline">
              {y.baslik}
            </Link>
            <p className="text-sm text-slate-500">
              {new Date(y.tarih).toLocaleDateString("tr-TR", { dateStyle: "long" })} · ❤️ {y.begeni}
            </p>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

### Yazı detayı

```tsx
// ============ app/blog/[slug]/loading.tsx ============
export default function Yukleniyor() {
  return (
    <div className="animate-pulse space-y-3">
      <div className="h-8 w-2/3 rounded bg-slate-200" />
      <div className="h-4 w-full rounded bg-slate-200" />
      <div className="h-4 w-5/6 rounded bg-slate-200" />
    </div>
  );
}
```

```tsx
// ============ app/blog/[slug]/page.tsx ============
import { Suspense } from "react";
import { notFound } from "next/navigation";
import { connection } from "next/server";
import type { Metadata } from "next";
import { yaziGetir } from "@/lib/veri";
import { db } from "@/lib/db";
import { BegenButonu } from "./BegenButonu";

type SayfaProps = { params: Promise<{ slug: string }> };

export async function generateMetadata({ params }: SayfaProps): Promise<Metadata> {
  const { slug } = await params;
  const yazi = await yaziGetir(slug);
  if (!yazi) return { title: "Bulunamadı" };
  return { title: yazi.baslik, description: yazi.icerik.slice(0, 150) };
}

export default async function YaziSayfasi({ params }: SayfaProps) {
  const { slug } = await params;
  const yazi = await yaziGetir(slug);
  if (!yazi) notFound();

  return (
    <article className="space-y-4">
      <h1 className="text-3xl font-bold">{yazi.baslik}</h1>
      <p className="text-sm text-slate-500">
        {new Date(yazi.tarih).toLocaleDateString("tr-TR", { dateStyle: "long" })}
      </p>
      <p className="leading-relaxed">{yazi.icerik}</p>

      <BegenButonu slug={yazi.slug} begeni={yazi.begeni} />

      <section>
        <h2 className="mb-2 text-xl font-semibold">Yorumlar</h2>
        <Suspense fallback={<p className="text-slate-500">Yorumlar yükleniyor...</p>}>
          <Yorumlar slug={yazi.slug} />
        </Suspense>
      </section>
    </article>
  );
}

// Dinamik, önbelleksiz Server Component: her istekte güncel, streaming ile gelir
async function Yorumlar({ slug }: { slug: string }) {
  await connection();
  const yorumlar = await db.yorumlariGetir(slug);
  if (yorumlar.length === 0) return <p className="text-slate-500">Henüz yorum yok.</p>;
  return (
    <ul className="space-y-2">
      {yorumlar.map((y) => (
        <li key={y.id} className="rounded bg-white p-3 text-sm">
          <strong>{y.yazar}:</strong> {y.metin}
        </li>
      ))}
    </ul>
  );
}
```

```tsx
// ============ app/blog/[slug]/not-found.tsx ============
import Link from "next/link";

export default function Bulunamadi() {
  return (
    <div className="py-12 text-center">
      <h1 className="text-2xl font-bold">Yazı bulunamadı</h1>
      <Link href="/blog" className="mt-4 inline-block text-indigo-600 underline">Bloga dön</Link>
    </div>
  );
}
```

### İyimser beğeni (Client Component + Server Function)

```tsx
// ============ app/blog/[slug]/BegenButonu.tsx ============
"use client";

import { useOptimistic, useState, startTransition } from "react";
import { begen } from "../actions";

export function BegenButonu({ slug, begeni }: { slug: string; begeni: number }) {
  const [sayi, setSayi] = useState(begeni);
  const [iyimserSayi, iyimserEkle] = useOptimistic(sayi, (mevcut, artis: number) => mevcut + artis);

  function handleBegen() {
    startTransition(async () => {
      iyimserEkle(1);                        // anında +1
      const yeni = await begen(slug);        // Server Function: sunucuda çalışır
      startTransition(() => setSayi(yeni));  // gerçek değer
    });
  }

  return (
    <button
      onClick={handleBegen}
      className="rounded-full border px-4 py-1.5 hover:bg-rose-50"
      aria-label={`Beğen (${iyimserSayi})`}
    >
      ❤️ {iyimserSayi}
    </button>
  );
}
```

### Yeni yazı formu

```tsx
// ============ app/blog/yeni/page.tsx ============
import type { Metadata } from "next";
import { YaziFormu } from "./YaziFormu";

export const metadata: Metadata = { title: "Yeni yazı" };

export default function YeniYaziSayfasi() {
  return (
    <section>
      <h1 className="mb-4 text-2xl font-bold">Yeni yazı</h1>
      <YaziFormu />
    </section>
  );
}
```

```tsx
// ============ app/blog/yeni/YaziFormu.tsx ============
"use client";

import { useActionState } from "react";
import { useFormStatus } from "react-dom";
import { yaziOlustur, type YaziFormDurumu } from "../actions";

function YayinlaButonu() {
  const { pending } = useFormStatus();
  return (
    <button disabled={pending} className="rounded bg-indigo-600 px-4 py-2 text-white disabled:opacity-50">
      {pending ? "Yayınlanıyor..." : "Yayınla"}
    </button>
  );
}

export function YaziFormu() {
  const [durum, formAction] = useActionState<YaziFormDurumu, FormData>(yaziOlustur, {});

  return (
    <form action={formAction} className="space-y-3">
      <label className="block">
        <span className="font-medium">Başlık</span>
        <input
          name="baslik"
          defaultValue={durum.degerler?.baslik}
          className="mt-1 w-full rounded border px-3 py-2"
        />
        {durum.hatalar?.baslik && (
          <span className="text-sm text-red-600">{durum.hatalar.baslik[0]}</span>
        )}
      </label>

      <label className="block">
        <span className="font-medium">İçerik</span>
        <textarea
          name="icerik"
          rows={8}
          defaultValue={durum.degerler?.icerik}
          className="mt-1 w-full rounded border px-3 py-2"
        />
        {durum.hatalar?.icerik && (
          <span className="text-sm text-red-600">{durum.hatalar.icerik[0]}</span>
        )}
      </label>

      <YayinlaButonu />
    </form>
  );
}
```

### Dene ve gözlemle

| Deneme | Gözlenen | Kavram |
|---|---|---|
| `/blog` → "Kaynağı Görüntüle" | Başlıklar HTML'in içinde | Sunucuda render, SEO |
| Network sekmesi | Sadece `BegenButonu` ve `YaziFormu` kodu iner | Server Components (sıfır JS) |
| Bir yazıya gir | İçerik hemen, yorumlar 1,5 sn sonra | Streaming + `<Suspense>` |
| Yazılar arası geçiş | İskelet görünür | `loading.tsx` |
| Liste/yazıyı yenile | Veritabanı tekrar sorgulanmaz | `"use cache"` |
| Kalbe tıkla | Sayı anında artar; önbellek temizlenir | `useOptimistic` + Server Function + `updateTag` |
| Kısa başlıkla gönder | Zod hataları sunucudan gelir, yazılanlar kalır | Server Function + `useActionState` |
| Geçerli yazı gönder | Yeni yazıya yönlendirme; listede görünür | `redirect` + `updateTag("yazilar")` |
| `/blog/olmayan-yazi` | Segmentin `not-found.tsx`'i | `notFound()` |
| `BegenButonu.tsx`'e `db` import et | Build hatası | `server-only` |

> **Hiç `useEffect` yok, istemcide hiç `fetch` yok, hiç API uç noktası yazılmadı.** Veri sunucuda okundu ve değiştirildi; istemciye yalnızca iki küçük etkileşimli bileşen gitti.

---

## 14. Kapanış: Bu Yolculukta Neler Öğrendin?

| Bölüm | Konular |
|---|---|
| **Temeller (0–9)** | JSX, bileşenler, props, state, olaylar, koşullu render, listeler, formlar — "UI = f(state)" |
| **Orta seviye (10–16a)** | Effect'ler, ref'ler, state'in yeri, context, reducer, custom Hook'lar, stil yöntemleri, Tailwind |
| **İleri seviye (17–23)** | Yönlendirme, sunucu verisi ve önbellek, performans ve React Compiler, React 19 Actions, test, TypeScript, Server Components |

### Bundan sonra: Proje fikirleri (kolaydan zora)

1. **Kişisel bütçe takipçisi** — Vite + `useReducer` + `localStorage` + bir grafik kütüphanesi *(Konu 5–15)*
2. **Film/kitap keşif uygulaması** — React Router + TanStack Query + açık bir API; sonsuz kaydırma, önceden yükleme, URL'de filtreler *(Konu 17–19)*
3. **Kanban panosu** — TypeScript + reducer + context + sürükle-bırak + testler *(Konu 14, 21, 22)*
4. **Tam yığın blog / mini e-ticaret** — Next.js + veritabanı (PostgreSQL + Drizzle/Prisma) + kimlik doğrulama *(Konu 23)*

> En iyi kaynak: **react.dev** — saflık, "effect'e ihtiyacın olmayabilir", state yapısı gibi kavramların en iyi açıklamaları orada.

---

## 15. Özet

- **CSR**'de sunucu boş HTML gönderir; ilk ekran yavaş, şelaleler var, SEO zayıf, her şey tarayıcıya gider.
- Render stratejileri: **CSR, SSR, SSG, önbellekli render, streaming**; Next.js bunları tek sayfada karıştırır.
- **Hydration:** Sunucu HTML'ine istemcide etkileşim bağlanır; ilk render'lar eşleşmelidir.
- **Next.js App Router:** Klasör yapısı = URL; `page`, `layout`, `loading`, `error`, `not-found`, `route` özel dosyaları; `[slug]`, `[...yol]`, `(grup)`; `params` asenkron; gezinme `next/link` ve `next/navigation`.
- **Server Components** varsayılandır: `async` olabilir, veriyi doğrudan okur, kodu tarayıcıya gitmez; Hook, olay ve tarayıcı API'si kullanamaz; `server-only` ile korunur.
- **`"use client"`** bir modül sınırıdır; Client Components da SSR edilir ama kodları tarayıcıya gider. Etkileşim **yapraklara** itilir; Server Component'ler Client'a `children` olarak verilebilir; props serileştirilebilir olmalı; provider'lar bir `"use client"` dosyasında.
- Sunucuda veri: doğrudan `await`, `Promise.all` ile paralel, `notFound()` / `redirect()`.
- **Streaming:** `<Suspense>` ve `loading.tsx` ile hazır kısımlar hemen, yavaşlar sonra gelir; Promise sunucuda başlatılıp istemcide `use` ile okunabilir.
- **Server Functions** (`"use server"`): sunucuda çalışan, istemciden çağrılan fonksiyonlar; `useActionState`/`useFormStatus`/`useOptimistic` ile; **herkese açık uç noktalardır** → doğrulama ve yetki kontrolü şart.
- **Next.js 16 önbellek:** açık (opt-in); `cacheComponents: true`, `"use cache"`, `cacheLife`, `cacheTag`; `updateTag` (anında) / `revalidateTag` (arka planda); **kısmi ön-render** ile statik iskelet + dinamik delikler; `connection()` ile istek anına zorlamak.
- **Metadata:** `export const metadata` / `generateMetadata`; sunucuda `<head>`'e yazılır.
- **Diğer:** Route Handlers (`route.ts`) dış API'ler için; **`proxy.ts`** (eski `middleware.ts`) istek öncesi kontroller; `next/image`, `next/font`; `NEXT_PUBLIC_` önekli değişkenler tarayıcıya gider; `error.tsx` Client Component.
- **Seçim:** Giriş arkasındaki SPA → Vite + React Router + TanStack Query; içerik/SEO/tam yığın → Next.js; alternatifler: React Router Framework, TanStack Start, Astro.

---

⬅️ Önceki: [22 · TypeScript ile React](./22-typescript.md) · ➡️ Sonraki: [24 · Mülakat Soruları](./24-mulakat-sorulari.md)
