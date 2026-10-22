# 16a · Tailwind CSS (Detaylı)

> **Konu özeti:** Utility-first felsefesi, kurulum ve araçlar, Tailwind'in çalışma mantığı ve Preflight, ölçek sistemi ve sözdizimi, kategori kategori sınıflar, varyantlar (durumlar, `group`, `peer`, `has-*`, `aria-*`, `data-*`, `starting:`), responsive tasarım ve container queries, koyu mod ve anlamsal renkler, `@theme` ile tema özelleştirme, kendi CSS'ini ekleme (`@utility`, `@custom-variant`, `@apply`, `@reference`, `@plugin`), React kalıpları (`cn`, `cva`), erişilebilirlik ve v3 → v4 farkları.
> **Sürüm:** Tailwind CSS v4.x (yazım anında v4.3)

---

## İçindekiler

1. [Utility-First Felsefesi](#1-utility-first-felsefesi)
2. [Kurulum ve Araçlar](#2-kurulum-ve-araçlar)
3. [Tailwind Nasıl Çalışır?](#3-tailwind-nasıl-çalışır)
4. [Temel Sözdizimi ve Ölçek Sistemi](#4-temel-sözdizimi-ve-ölçek-sistemi-)
5. [Kategori Kategori Sık Kullanılan Sınıflar](#5-kategori-kategori-sık-kullanılan-sınıflar)
6. [Varyantlar (Önekler)](#6-varyantlar-önekler-)
7. [Responsive Tasarım ve Container Queries](#7-responsive-tasarım-ve-container-queries-)
8. [Koyu Mod](#8-koyu-mod)
9. [Tema Özelleştirme: `@theme`](#9-tema-özelleştirme-theme-)
10. [Kendi CSS'ini Eklemek](#10-kendi-cssini-eklemek)
11. [React ile Tailwind Kalıpları](#11-react-ile-tailwind-kalıpları-)
12. [Erişilebilirlik](#12-erişilebilirlik)
13. [v3'ten v4'e Önemli Farklar](#13-v3ten-v4e-önemli-farklar)
14. [Her Şey Bir Arada: Responsive Ürün Paneli](#14-her-şey-bir-arada-responsive-ürün-paneli)
15. [Özet](#15-özet)

---

## 1. Utility-First Felsefesi

**Geleneksel CSS:** Önce isim bul, sonra o isme stil yaz.

```html
<div class="profil-karti">...</div>
```

```css
.profil-karti { display: flex; align-items: center; gap: 1rem; padding: 1.5rem; border-radius: 0.75rem; background: white; }
```

**Tailwind:** İsim yok; stil doğrudan elemana yazılır.

```html
<div class="flex items-center gap-4 rounded-xl bg-white p-6">...</div>
```

Her sınıf **tek bir CSS özelliğine** (bazen birkaçına) karşılık gelir → **utility-first.**

### "Bu inline stil değil mi?"

| | Inline stil | Tailwind |
|---|---|---|
| Değerler | Herhangi bir değer | Bir **tasarım sisteminden** → tutarlılık |
| Hover, focus | ❌ | ✅ `hover:`, `focus:` |
| Responsive | ❌ | ✅ `md:`, `lg:` |
| Koyu mod | ❌ | ✅ `dark:` |
| Animasyon | ❌ | ✅ `animate-spin`, `transition` |
| Performans | Her elemanda tekrar | Ortak sınıflar, tek CSS dosyası |

### Neden popüler?

- **İsim bulma yorgunluğu yok**
- **Bağlam değiştirme yok** (JSX ↔ CSS dosyası)
- **Ölü CSS yok** — bileşen silinince stili de gider
- **CSS büyümez** — `p-4` CSS'te bir kez tanımlanır
- **Tasarım tutarlılığı** — herkes aynı ölçeği kullanır

> **Eleştiri:** Uzun sınıf listeleri ve öğrenilmesi gereken bir "sözlük." React'te çözüm: **bileşenler** (bkz. [Bölüm 11](#11-react-ile-tailwind-kalıpları-)).

---

## 2. Kurulum ve Araçlar

```bash
npm install tailwindcss @tailwindcss/vite
```

```js
// vite.config.js
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import tailwindcss from "@tailwindcss/vite";

export default defineConfig({
  plugins: [react(), tailwindcss()],
});
```

```css
/* src/index.css */
@import "tailwindcss";
```

### Mutlaka kurulacak araçlar

**1. Tailwind CSS IntelliSense** (VS Code): Otomatik tamamlama, üretilen CSS'i gösterme, renk önizleme, hatalı sınıf uyarısı.

**2. prettier-plugin-tailwindcss:** Sınıfları resmi önerilen sıraya dizer.

```bash
npm install -D prettier prettier-plugin-tailwindcss
```

```json
// .prettierrc
{ "plugins": ["prettier-plugin-tailwindcss"] }
```

---

## 3. Tailwind Nasıl Çalışır?

1. **Kaynak kodu tarar:** Proje dosyalarını **düz metin** olarak okur, sınıf adına benzeyen her şeyi toplar. `.gitignore`'daki dosyaları ve `node_modules`'ü atlar.
2. **Sadece kullanılanları üretir:** Çıktı CSS'i yalnızca kodda geçen sınıfları içerir.
3. **Ek kaynak belirtmek** (`@source`):

```css
@import "tailwindcss";
@source "../node_modules/@sirketim/ui-kit";
```

### Katmanlar (layers)

```
theme       → tema değişkenleri (--color-blue-500 vb.)
base        → Preflight (sıfırlama) ve temel stiller
components  → bileşen sınıfları
utilities   → yardımcı sınıflar (p-4, flex, ...) — en yüksek öncelik
```

### Preflight (CSS sıfırlama)

- Başlık (`h1`–`h6`) ve paragrafların varsayılan boyut/boşlukları **kaldırılır** → boyutu sen verirsin (`text-3xl font-bold`).
- Liste madde işaretleri kaldırılır (`list-disc` ile geri eklenir).
- `<img>` blok element olur.
- Kenarlık rengi varsayılan olarak `currentColor`.
- **`<button>` imleci `cursor: default`** → tıklanabilir hissi için `cursor-pointer` eklenir.

> "Başlıklarım neden küçüldü?" — bilinçli: hiçbir eleman sen istemeden stil almaz.

---

## 4. Temel Sözdizimi ve Ölçek Sistemi ⭐

### Boşluk ölçeği

Taban değer: `--spacing: 0.25rem` (4px).

```
p-1   → 4px
p-2   → 8px
p-4   → 16px
p-6   → 24px
p-8   → 32px
p-13  → 52px    ← v4'te HER tam sayı geçerli
p-px  → 1px
p-0.5 → 2px
```

> **Akılda tutma:** sayı × 4 = piksel. `gap-6` = 24px, `mt-10` = 40px, `w-64` = 256px.

### Kesirler ve özel boyutlar

```
w-1/2   → 50%          w-2/3 → 66.67%
w-full  → 100%         w-screen → 100vw
h-dvh   → 100dvh (mobil tarayıcı çubuklarını hesaba katar)
size-10 → width + height = 40px
w-fit, w-min, w-max
max-w-md, max-w-2xl, max-w-prose
```

### Renkler

Tonlar `50` (en açık) – `950` (en koyu):

```
slate, gray, zinc, neutral, stone                 ← griler
red, orange, amber, yellow, lime, green,
emerald, teal, cyan, sky, blue, indigo,
violet, purple, fuchsia, pink, rose               ← renkler
mauve, olive, mist, taupe                         ← v4.2 ile eklenen nötrler
```

- Varsayılan renkler `oklch` renk uzayında tanımlıdır.
- Her renk özelliğinde aynı isimle kullanılır:

```
bg-blue-500  text-blue-500  border-blue-500  ring-blue-500  outline-blue-500
fill-blue-500  stroke-blue-500  decoration-blue-500  divide-blue-500
accent-blue-500  caret-blue-500
```

### Opaklık değiştirici: `/`

```
bg-black/50        → %50 opak siyah
text-blue-600/75
border-white/10
```

> 📜 v3'teki `bg-opacity-50` gibi sınıflar v4'te kaldırıldı.

### Rastgele değerler: `[...]`

```
w-[327px]   top-[117px]   bg-[#1da1f2]
grid-cols-[200px_1fr]                     ← boşluk yerine alt çizgi
text-[clamp(1rem,2.5vw,2rem)]
```

**Rastgele özellik:**

```
[mask-type:luminance]   [scrollbar-gutter:stable]
```

### CSS değişkeni kısayolu: `(...)`

```jsx
<div
  className="bg-(--marka-rengi) w-(--genislik)"
  style={{ "--marka-rengi": renk, "--genislik": "40%" }}
/>
```

> 📜 v3: `bg-[--marka-rengi]` → v4: `bg-(--marka-rengi)`

### Önem (important): `!`

```
bg-red-500!    → !important
```

> 📜 v3'te başa yazılıyordu (`!bg-red-500`).

### Negatif değerler

```
-mt-4   -translate-x-1/2
```

---

## 5. Kategori Kategori Sık Kullanılan Sınıflar

### Yerleşim (Layout)

| Sınıf | CSS |
|---|---|
| `block`, `inline-block`, `inline`, `hidden` | `display` |
| `flex`, `inline-flex`, `grid`, `contents` | `display` |
| `relative`, `absolute`, `fixed`, `sticky` | `position` |
| `inset-0`, `top-4`, `left-1/2`, `inset-x-0` | Konum |
| `z-10`, `z-50` | `z-index` |
| `overflow-hidden`, `overflow-auto`, `overflow-x-scroll` | Taşma |
| `aspect-square`, `aspect-video`, `aspect-4/3` | En-boy oranı |
| `object-cover`, `object-contain` | Görsel yerleşimi |
| `sr-only` | Görsel olarak gizli, ekran okuyucuda okunur |

### Flexbox

```jsx
<div className="flex items-center justify-between gap-4">
  <Logo />
  <nav className="flex gap-2">...</nav>
</div>
```

| Sınıf | Anlamı |
|---|---|
| `flex-row`, `flex-col`, `flex-wrap` | Yön ve sarma |
| `items-start / center / end / stretch / baseline` | Çapraz eksen |
| `justify-start / center / end / between / around / evenly` | Ana eksen |
| `gap-4`, `gap-x-2`, `gap-y-6` | Elemanlar arası boşluk |
| `flex-1` | Kalan alanı doldur |
| `shrink-0`, `grow` | Küçülme / büyüme |
| `self-end`, `order-first` | Tek elemanı ayarlama |

### Grid

```jsx
<div className="grid grid-cols-3 gap-6">
  <div className="col-span-2">Ana içerik</div>
  <aside>Kenar çubuğu</aside>
</div>
```

| Sınıf | Anlamı |
|---|---|
| `grid-cols-3`, `grid-cols-12` | Eşit sütunlar |
| `grid-cols-[250px_1fr]` | Özel sütun yapısı |
| `col-span-2`, `col-span-full`, `col-start-2` | Sütun kaplama |
| `grid-rows-3`, `row-span-2` | Satırlar |
| `place-items-center` | Yatay + dikey ortalama |
| `grid-flow-col`, `auto-rows-fr` | Akış |

> 💡 Tam ortalamak: `grid place-items-center` veya `flex items-center justify-center`.

### Boşluk (Spacing)

| Sınıf | Anlamı |
|---|---|
| `p-4`, `px-4`, `py-2`, `pt-6`, `pr-2`, `pb-0`, `pl-8` | Padding |
| `m-4`, `mx-auto`, `my-8`, `mt-2` | Margin (`mx-auto` yatay ortalar) |
| `ps-4`, `pe-4`, `ms-2`, `me-2` | Mantıksal (sağdan sola dillere uyumlu) |
| `space-y-4`, `space-x-2` | Çocuklar arası boşluk |

> 💡 Flex/grid içinde `space-*` yerine `gap` tercih edilir.

### Boyutlandırma (Sizing)

| Sınıf | Anlamı |
|---|---|
| `w-64`, `w-full`, `w-1/3`, `w-screen`, `w-auto` | Genişlik |
| `h-12`, `h-full`, `h-dvh`, `min-h-svh` | Yükseklik |
| `size-8` | Genişlik + yükseklik |
| `min-w-0` | Flex içinde metnin küçülmesine izin ver (`truncate` için gerekli!) |
| `max-w-sm … max-w-7xl`, `max-w-prose` | Maksimum genişlik |

### Tipografi

| Sınıf | Anlamı |
|---|---|
| `text-xs / sm / base / lg / xl / 2xl … 9xl` | Boyut (uygun satır yüksekliğiyle) |
| `text-lg/7` | Boyut + satır yüksekliği birlikte |
| `font-normal / medium / semibold / bold` | Kalınlık |
| `font-sans`, `font-serif`, `font-mono` | Aile |
| `leading-tight`, `leading-relaxed`, `leading-6` | Satır yüksekliği |
| `tracking-tight`, `tracking-wide` | Harf aralığı |
| `text-left / center / right / justify` | Hizalama |
| `uppercase`, `lowercase`, `capitalize` | Dönüşüm |
| `underline`, `line-through`, `no-underline`, `underline-offset-4` | Dekorasyon |
| `truncate` | Tek satır + `...` |
| `line-clamp-3` | 3 satırdan sonra `...` |
| `text-balance`, `text-pretty` | Dengeli satır kırılımı |
| `whitespace-nowrap`, `break-words` | Sarma |
| `tabular-nums` | Sabit genişlikli rakamlar |

### Arka plan ve gradyanlar

```jsx
<div className="bg-linear-to-r from-indigo-500 via-purple-500 to-pink-500" />
<div className="bg-linear-45 from-sky-400 to-blue-700" />   {/* açılı */}
<div className="bg-radial from-white to-gray-200" />
<div className="bg-conic from-red-500 to-blue-500" />
```

> 📜 v3: `bg-gradient-to-r` → v4: `bg-linear-to-r`

Diğer: `bg-cover`, `bg-center`, `bg-no-repeat`, `bg-fixed`, `bg-[url(/desen.svg)]`

### Kenarlık, köşe, halka, dış çizgi

| Sınıf | Anlamı |
|---|---|
| `border`, `border-2`, `border-t`, `border-x-0` | Kalınlık |
| `border-gray-200` | Renk (v4'te varsayılan `currentColor` → açıkça ver) |
| `border-dashed`, `border-dotted` | Stil |
| `rounded-xs / sm / md / lg / xl / 2xl / 3xl / full`, `rounded-t-lg` | Köşe |
| `divide-y divide-gray-200` | Çocuklar arası çizgi |
| `ring-2 ring-blue-500 ring-offset-2` | Halka (box-shadow) |
| `outline-2 outline-offset-2 outline-blue-600` | Dış çizgi (odak) |
| `outline-hidden` | Odak çizgisini gizle (dikkatli kullan) |

### Efektler

| Sınıf | Anlamı |
|---|---|
| `shadow-2xs / xs / sm / md / lg / xl / 2xl` | Gölge |
| `inset-shadow-sm`, `inset-ring-1` | İç gölge / halka (v4) |
| `shadow-blue-500/30` | Renkli gölge |
| `opacity-50` | Opaklık |
| `mix-blend-multiply` | Karışım modu |
| `text-shadow-sm`, `mask-b-from-50%` | Metin gölgesi, maskeleme (v4.1+) |

### Filtreler

`blur-sm`, `brightness-110`, `contrast-125`, `grayscale`, `sepia`, `drop-shadow-lg`, `backdrop-blur-md`

> Cam efekti: `bg-white/70 backdrop-blur-md`

### Dönüşümler

```
scale-95   scale-110   rotate-45   -rotate-12
translate-x-4   -translate-y-1/2   skew-x-6   origin-top-left
rotate-x-45   perspective-normal   transform-3d      ← 3D (v4)
```

> v4'te ayrıca `transform` sınıfı gerekmez.

### Geçişler ve animasyonlar

| Sınıf | Anlamı |
|---|---|
| `transition` | Renk, opaklık, gölge, dönüşüm gibi yaygın özelliklere geçiş |
| `transition-colors / opacity / transform / all` | Belirli özellikler |
| `duration-150 / 300 / 500` | Süre |
| `ease-in`, `ease-out`, `ease-in-out` | Zamanlama |
| `delay-100` | Gecikme |
| `animate-spin` | Döndürme (yükleniyor) |
| `animate-ping` | Dalgalanma (bildirim noktası) |
| `animate-pulse` | Nabız (iskelet ekranı) |
| `animate-bounce` | Zıplama |

```jsx
// İskelet yükleme ekranı
<div className="animate-pulse space-y-3">
  <div className="h-4 w-3/4 rounded bg-gray-200" />
  <div className="h-4 w-1/2 rounded bg-gray-200" />
</div>
```

### Etkileşim

| Sınıf | Anlamı |
|---|---|
| `cursor-pointer`, `cursor-not-allowed`, `cursor-grab` | İmleç |
| `select-none`, `select-all` | Metin seçimi |
| `pointer-events-none` | Tıklamaları geçir |
| `resize-none`, `resize-y` | Textarea boyutlandırma |
| `scroll-smooth`, `snap-x snap-mandatory`, `snap-center` | Kaydırma |
| `accent-blue-600` | Checkbox/radio rengi |
| `caret-pink-500` | Yazma imleci rengi |
| `field-sizing-content` | Textarea içeriğe göre büyür (v4) |
| `appearance-none` | Tarayıcı varsayılan görünümünü kaldır |

### v4.2 – v4.3 yenilikleri (örnekler)

- **Scrollbar** sınıfları (ör. `scrollbar-thin`) — v4.3
- **`zoom-*`** ve **`tab-*`** sınıfları — v4.3
- Mantıksal özellik sınıfları (`pbs-*`, `mbe-*` vb.) ve **`font-features-*`** — v4.2

> Yeni eklemeler için resmi dokümantasyon takip edilmelidir.

---

## 6. Varyantlar (Önekler) ⭐

Varyant, sınıfın **ne zaman** uygulanacağını belirler: `hover:bg-blue-700`.

### Durumlar (pseudo-class)

```
hover:  focus:  focus-visible:  focus-within:  active:  visited:
disabled:  enabled:  checked:  indeterminate:  required:  invalid:
user-invalid:  ← kullanıcı etkileşiminden SONRA geçersizse
read-only:  placeholder-shown:  autofill:  open:  inert:  target:
```

```jsx
<input
  type="email"
  required
  className="rounded border border-gray-300 px-3 py-2
             focus:border-blue-500 focus:outline-2 focus:outline-blue-500/30
             user-invalid:border-red-500"
/>
```

> 💡 `focus-visible:` yalnızca klavyeyle odaklanınca çalışır; butonlarda `focus:` yerine tercih edilir.

### Pseudo-element'ler

```
before:  after:  placeholder:  file:  marker:  selection:  first-line:  first-letter:  backdrop:
```

```jsx
<label className="after:ml-0.5 after:text-red-500 after:content-['*']">E-posta</label>
<input
  type="file"
  className="file:mr-4 file:rounded file:border-0 file:bg-blue-50 file:px-4 file:py-2 file:text-blue-700"
/>
<ul className="list-disc marker:text-blue-500">...</ul>
```

### Yapısal

```
first:  last:  only:  odd:  even:
nth-3:  nth-last-2:  nth-[3n+1]:  first-of-type:  empty:
*:     ← doğrudan tüm çocuklara
**:    ← tüm torunlara
```

```jsx
<ul className="divide-y *:py-3">...</ul>
<tbody className="*:odd:bg-gray-50">...</tbody>
```

### Ebeveyne göre: `group`

```jsx
<a href="#" className="group flex items-center gap-3 rounded-lg p-3 hover:bg-gray-50">
  <img className="size-10 rounded-full" />
  <span className="font-medium group-hover:text-blue-600">Ayşe Yılmaz</span>
  <span className="ml-auto opacity-0 transition group-hover:opacity-100">→</span>
</a>
```

**İsimli gruplar:**

```jsx
<div className="group/kart">
  <div className="group/baslik">
    <span className="group-hover/kart:underline group-hover/baslik:text-blue-600">...</span>
  </div>
</div>
```

### Kardeşe göre: `peer`

```jsx
<label className="flex flex-col gap-1">
  <span>E-posta</span>
  <input type="email" required className="peer rounded border px-3 py-2" />
  <p className="hidden text-sm text-red-600 peer-user-invalid:block">
    Geçerli bir e-posta adresi girin.
  </p>
</label>
```

- JS/state olmadan, hata mesajı kullanıcı geçersiz bir şey yazdıktan sonra görünür.

> ⚠️ `peer` yalnızca **sonraki** kardeşleri etkiler; `peer` sınıflı eleman önce gelmelidir.

### İçeriğe göre: `has-*`

```jsx
<label className="flex items-center gap-2 rounded-lg border p-4 has-checked:border-blue-500 has-checked:bg-blue-50">
  <input type="radio" name="plan" value="pro" />
  Pro Plan
</label>
```

- `group-has-*` ve `peer-has-*` de vardır.

### `not-*` ve `in-*`

```
not-first:mt-4
not-disabled:hover:bg-blue-700
in-focus:opacity-100     ← herhangi bir atası odaklıysa (group gerekmeden)
```

### ARIA ve data nitelikleri ⭐

State niteliğe yansıtılır, stil nitelikten okunur:

```jsx
<button aria-expanded={acik} className="flex items-center gap-2 aria-expanded:font-semibold">
  Detaylar
  <span className="transition aria-expanded:rotate-180">▾</span>
</button>
```

```jsx
<div
  data-durum={durum}
  className="rounded p-3 data-[durum=hata]:bg-red-50 data-[durum=basarili]:bg-green-50"
>
  ...
</div>

// Değersiz nitelik
<div data-aktif={aktif || undefined} className="opacity-50 data-aktif:opacity-100" />
```

- Headless kütüphanelerle (Radix vb.) uyumlu: `data-[state=open]:bg-gray-100`.

> 💡 `aria-*` hem stil hem **erişilebilirlik** sağlar: aynı nitelik ekran okuyucuya da durumu bildirir.

### `starting:`: CSS ile giriş animasyonu

```jsx
{acik && (
  <div className="opacity-100 transition-opacity duration-300 starting:opacity-0">
    Bildirim
  </div>
)}
```

- CSS `@starting-style` kullanır; DOM'a eklenen eleman `opacity-0`'dan başlar.
- Çıkış animasyonu için hâlâ Motion gibi bir kütüphane gerekir.

### Rastgele varyantlar: `[...]`

```
[&>li]:py-2          ← doğrudan li çocuklarına
[&_a]:underline      ← içindeki tüm a'lara
[&:nth-child(3)]:font-bold
```

### Varyant zincirleme

**Soldan sağa** okunur:

```
dark:md:hover:bg-gray-700         → koyu modda, md+ ekranda, hover'da
group-hover:disabled:opacity-50
```

> 📜 v3'te bazı zincirler sağdan sola uygulanıyordu.

---

## 7. Responsive Tasarım ve Container Queries ⭐

### Kırılma noktaları

| Önek | Minimum genişlik | Tipik cihaz |
|---|---|---|
| (yok) | 0 | Telefon |
| `sm:` | 40rem (640px) | Büyük telefon |
| `md:` | 48rem (768px) | Tablet |
| `lg:` | 64rem (1024px) | Dizüstü |
| `xl:` | 80rem (1280px) | Masaüstü |
| `2xl:` | 96rem (1536px) | Büyük ekran |

### Mobile-first

Öneksiz sınıf **her ekranda**, önekli sınıf o genişlikten **itibaren** geçerlidir.

```jsx
// ❌ "sm: = küçük ekran" sanmak
<div className="sm:text-center">            // Telefonda ORTALANMAZ

// ✅ Önce mobil, sonra büyük ekranlar
<div className="text-center sm:text-left">  // telefonda ortalı, 640px+ sola yaslı
```

```jsx
<div className="flex flex-col gap-4 md:flex-row md:items-center">...</div>
<h1 className="text-2xl font-bold md:text-4xl lg:text-5xl">Başlık</h1>
<nav className="hidden md:flex">...</nav>       {/* sadece tablet+ */}
<button className="md:hidden">☰</button>       {/* sadece mobil */}
```

### Aralıklar ve `max-*`

```
max-md:hidden       ← md'nin ALTINDA gizle
md:max-lg:flex      ← sadece md ile lg arasında
```

### Özel kırılma noktaları

```css
@theme {
  --breakpoint-xs: 30rem;
  --breakpoint-3xl: 120rem;
}
```

Tek seferlik: `min-[900px]:grid-cols-3`, `max-[600px]:hidden`

### Container Queries (v4'te yerleşik)

Medya sorguları **ekranın**, container queries **kapsayıcının** genişliğine bakar.

```jsx
function ProfilKarti() {
  return (
    <div className="@container">
      <div className="flex flex-col gap-3 @md:flex-row @md:items-center">
        <img className="size-16 rounded-full @md:size-20" />
        <div>
          <h3 className="font-semibold @lg:text-lg">Ayşe Yılmaz</h3>
          <p className="hidden text-gray-500 @sm:block">Frontend Geliştirici</p>
        </div>
      </div>
    </div>
  );
}

<main className="w-2/3"><ProfilKarti /></main>   {/* geniş: yan yana */}
<aside className="w-64"><ProfilKarti /></aside>  {/* dar: alt alta */}
```

| Önek | Kapsayıcı genişliği ≥ |
|---|---|
| `@3xs:` | 16rem (256px) |
| `@2xs:` | 18rem |
| `@xs:` | 20rem |
| `@sm:` | 24rem (384px) |
| `@md:` | 28rem (448px) |
| `@lg:` | 32rem (512px) |
| `@xl:` … `@7xl:` | 36rem … 80rem |

- `@max-md:` → kapsayıcı md'den küçükse.
- **İsimli kapsayıcı:** `@container/kenar` ve `@md/kenar:flex`.

> **Sayfa düzeni** için medya sorguları, **yeniden kullanılabilir bileşen içi düzen** için container queries.

---

## 8. Koyu Mod

### Varsayılan: Sistem tercihi

```jsx
<div className="bg-white text-gray-900 dark:bg-gray-900 dark:text-gray-100">
  <p className="text-gray-600 dark:text-gray-400">...</p>
</div>
```

### Elle kontrol (tema düğmesi)

```css
@import "tailwindcss";
@custom-variant dark (&:where([data-tema="koyu"], [data-tema="koyu"] *));

/* veya sınıf tabanlı */
@custom-variant dark (&:where(.dark, .dark *));
```

### Yöntem 1: Her yerde `dark:` yazmak

Basit ama her elemanda iki renk; değişiklikler onlarca yere dağılır.

### Yöntem 2: Anlamsal renk değişkenleri ⭐

Renkler **anlamlarına** göre adlandırılır; koyu modda değişken değerleri değişir; bileşenlerde `dark:` gerekmez.

```css
@import "tailwindcss";

:root {
  --yuzey: var(--color-white);
  --yuzey-ikincil: var(--color-gray-50);
  --metin: var(--color-gray-900);
  --metin-soluk: var(--color-gray-500);
  --kenar: var(--color-gray-200);
  --vurgu: var(--color-blue-600);
}

[data-tema="koyu"] {
  --yuzey: var(--color-gray-900);
  --yuzey-ikincil: var(--color-gray-800);
  --metin: var(--color-gray-100);
  --metin-soluk: var(--color-gray-400);
  --kenar: var(--color-gray-700);
  --vurgu: var(--color-blue-400);
}

@theme inline {
  --color-yuzey: var(--yuzey);
  --color-yuzey-ikincil: var(--yuzey-ikincil);
  --color-metin: var(--metin);
  --color-metin-soluk: var(--metin-soluk);
  --color-kenar: var(--kenar);
  --color-vurgu: var(--vurgu);
}
```

```jsx
<div className="border border-kenar bg-yuzey text-metin">
  <p className="text-metin-soluk">Koyu modda otomatik değişir</p>
  <button className="bg-vurgu text-white">Kaydet</button>
</div>
```

> **`@theme inline` kuralı:** Tema değişkeni başka bir CSS değişkenine referans veriyorsa `inline` kullanılır; böylece sınıf doğru değişkeni okur.

> shadcn/ui bu yaklaşımı kullanır (`bg-background`, `text-foreground`, `bg-primary`).

### `color-scheme`

Tarayıcının çizdiği kaydırma çubukları ve form elemanları için:

```jsx
<html className="scheme-light dark:scheme-dark">
```

---

## 9. Tema Özelleştirme: `@theme` ⭐

Her değişken bir **ad alanına** aittir; ad alanı hangi sınıfların üretileceğini belirler.

| Ad alanı | Ürettiği sınıflar | Örnek |
|---|---|---|
| `--color-*` | `bg-*`, `text-*`, `border-*`, `fill-*`… | `--color-marka: #5b21b6;` → `bg-marka` |
| `--font-*` | `font-*` (aile) | `--font-baslik: "Poppins";` → `font-baslik` |
| `--text-*` | `text-*` (boyut) | `--text-dev: 5rem;` → `text-dev` |
| `--font-weight-*` | `font-*` (kalınlık) | `--font-weight-extrablack: 950;` |
| `--spacing` | Tüm boşluk/boyut ölçeği | `--spacing: 0.25rem;` |
| `--breakpoint-*` | Responsive önekler | `--breakpoint-3xl: 120rem;` → `3xl:` |
| `--container-*` | Container önekleri, `max-w-*` | `--container-8xl: 96rem;` |
| `--radius-*` | `rounded-*` | `--radius-kart: 1.25rem;` → `rounded-kart` |
| `--shadow-*` | `shadow-*` | `--shadow-yumusak: ...` |
| `--ease-*` | `ease-*` | `--ease-yay: cubic-bezier(.34,1.56,.64,1);` |
| `--animate-*` | `animate-*` | aşağıda |

```css
@import "tailwindcss";

@theme {
  /* Marka paleti */
  --color-marka-50: oklch(0.97 0.02 290);
  --color-marka-500: oklch(0.55 0.22 290);
  --color-marka-700: oklch(0.42 0.2 290);

  /* Fontlar */
  --font-govde: "Inter", ui-sans-serif, system-ui, sans-serif;
  --font-baslik: "Poppins", var(--font-govde);

  /* Köşe ve gölge */
  --radius-kart: 1rem;
  --shadow-kart: 0 1px 2px rgb(0 0 0 / 0.04), 0 8px 24px rgb(0 0 0 / 0.06);

  /* Özel animasyon */
  --animate-sallan: sallan 0.4s ease-in-out;

  @keyframes sallan {
    0%, 100% { transform: translateX(0); }
    25% { transform: translateX(-4px); }
    75% { transform: translateX(4px); }
  }
}
```

```jsx
<h1 className="font-baslik text-marka-700">...</h1>
<div className="rounded-kart shadow-kart">...</div>
<input className={cn("...", hata && "animate-sallan border-red-500")} />
```

### Genişletmek mi, değiştirmek mi?

- Normal yazım → varsayılan tema **korunur**, yeni değerler **eklenir.**
- Bir ad alanını tamamen değiştirmek:

```css
@theme {
  --color-*: initial;        /* tüm varsayılan renkleri kaldır */
  --color-beyaz: #fff;
  --color-siyah: #0a0a0a;
  --color-marka: #5b21b6;
}
```

> Katı tasarım sistemlerinde rastgele renk kullanımını engeller.

### Tema değişkenleri gerçek CSS değişkenleridir

```jsx
<div style={{ boxShadow: "0 0 0 4px var(--color-marka-500)" }} />
```

```css
.grafik-cizgisi { stroke: var(--color-marka-500); }
```

```js
// JS'te okumak (ör. grafik kütüphaneleri)
getComputedStyle(document.documentElement).getPropertyValue("--color-marka-500");
```

---

## 10. Kendi CSS'ini Eklemek

### `@layer base`: Temel stiller

```css
@layer base {
  h1 { @apply text-3xl font-bold; }
  a { @apply text-vurgu underline-offset-4 hover:underline; }
  ::selection { @apply bg-marka-500/20; }
}
```

### `@utility`: Kendi yardımcı sınıfların

```css
@utility icerik-otomatik {
  content-visibility: auto;
}

@utility kaydirma-cubugu-gizli {
  scrollbar-width: none;
  &::-webkit-scrollbar { display: none; }
}
```

```jsx
<section className="icerik-otomatik">...</section>
<div className="overflow-x-auto md:kaydirma-cubugu-gizli">...</div>
```

- Tüm varyantlarla çalışır (`hover:`, `md:`); yalnızca kullanılırsa CSS'e eklenir.

#### Değer alan (fonksiyonel) utility

```css
@utility ust-bosluk-* {
  padding-top: calc(var(--spacing) * --value(integer));
}
/* ust-bosluk-4 → padding-top: 1rem */
```

### `@custom-variant`: Kendi varyantların

```css
@custom-variant yazdir (@media print);
@custom-variant dokunmatik (@media (pointer: coarse));
```

```jsx
<nav className="yazdir:hidden">...</nav>
<button className="px-3 py-1 dokunmatik:px-5 dokunmatik:py-3">...</button>
```

### `@apply`: Ne zaman?

```css
.btn-birincil {
  @apply rounded-md bg-blue-600 px-4 py-2 text-white hover:bg-blue-700;
}
```

- **React projelerinde genellikle önerilmez**; tekrar bileşenlerle azaltılır.
- **Uygun yerler:** `@layer base` içindeki global stiller; kontrol edilemeyen HTML (CMS/Markdown içeriği, üçüncü parti kütüphane elemanları).

### `@reference`: Ayrı CSS dosyalarında `@apply`

```css
/* Kart.module.css */
@reference "../index.css";

.baslik {
  @apply text-xl font-semibold text-marka-700;
}
```

- Ana CSS'i **kopyalamadan** referans alır.

### `@plugin`: Eklentiler (Typography)

```bash
npm install -D @tailwindcss/typography
```

```css
@import "tailwindcss";
@plugin "@tailwindcss/typography";
```

```jsx
<article className="prose lg:prose-lg dark:prose-invert">
  <div dangerouslySetInnerHTML={{ __html: blogYazisiHtml }} />
</article>
```

- Markdown/CMS'ten gelen HTML'deki başlık, paragraf, liste, kod ve alıntılar otomatik stillenir.

---

## 11. React ile Tailwind Kalıpları ⭐

### 1. `cn` yardımcısı

```js
// src/lib/cn.js
import { clsx } from "clsx";
import { twMerge } from "tailwind-merge";
export const cn = (...girdiler) => twMerge(clsx(girdiler));
```

> Dışarıdan `className` alan **her bileşende** `cn` kullanılır; dışarıdan gelen sınıflar güvenle ezer.

### 2. Tam sınıf adları kodda olmalı

```jsx
// ❌ Tailwind bu sınıfları bulamaz
<div className={`bg-${renk}-500 text-${renk}-50`} />

// ✅ Eşleme nesnesi
const RENK_SINIFLARI = {
  blue: "bg-blue-500 text-blue-50",
  green: "bg-green-500 text-green-50",
  red: "bg-red-500 text-red-50",
};
<div className={RENK_SINIFLARI[renk]} />
```

**Önceden bilinemeyen sınıfları üretmek (safelist):**

```css
@source inline("bg-{red,green,blue,amber}-{100,500}");
```

### 3. Varyant kalıbı: `cva`

```bash
npm install class-variance-authority
```

```jsx
import { cva } from "class-variance-authority";
import { cn } from "../lib/cn";

const butonStilleri = cva(
  // Temel sınıflar
  "inline-flex cursor-pointer items-center justify-center gap-2 rounded-md font-medium transition-colors " +
  "focus-visible:outline-2 focus-visible:outline-offset-2 " +
  "disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      varyant: {
        birincil: "bg-blue-600 text-white hover:bg-blue-700 focus-visible:outline-blue-600",
        ikincil: "bg-gray-100 text-gray-900 hover:bg-gray-200 focus-visible:outline-gray-400",
        cerceve: "border border-gray-300 bg-transparent hover:bg-gray-50",
        tehlike: "bg-red-600 text-white hover:bg-red-700 focus-visible:outline-red-600",
        hayalet: "hover:bg-gray-100",
      },
      boyut: {
        kucuk: "h-8 px-3 text-sm",
        orta: "h-10 px-4",
        buyuk: "h-12 px-6 text-lg",
        ikon: "size-10",
      },
    },
    compoundVariants: [
      { varyant: "cerceve", boyut: "kucuk", class: "border-gray-200" },
    ],
    defaultVariants: {
      varyant: "birincil",
      boyut: "orta",
    },
  }
);

export function Buton({ varyant, boyut, className, ...rest }) {
  return <button className={cn(butonStilleri({ varyant, boyut }), className)} {...rest} />;
}
```

```jsx
<Buton>Kaydet</Buton>
<Buton varyant="tehlike" boyut="kucuk">Sil</Buton>
<Buton varyant="hayalet" boyut="ikon" aria-label="Kapat">✕</Buton>
<Buton className="w-full">Tam genişlik</Buton>
```

> shadcn/ui bileşenleri bu kalıpla yazılmıştır.

### 4. State'i niteliğe yansıt, stili nitelikten oku

```jsx
// Koşullu sınıflar: kalabalık
<li className={cn("rounded p-2", secili && "bg-blue-50 font-medium", pasif && "opacity-50")}>

// Nitelik + varyant: daha temiz ve erişilebilir
<li
  aria-selected={secili}
  aria-disabled={pasif}
  className="rounded p-2 aria-selected:bg-blue-50 aria-selected:font-medium aria-disabled:opacity-50"
>
```

### 5. Uzun sınıf listelerini düzenlemek

```jsx
<div
  className={cn(
    // yerleşim
    "flex flex-col gap-4 md:flex-row",
    // görünüm
    "rounded-xl border border-kenar bg-yuzey p-6 shadow-sm",
    // etkileşim
    "transition hover:shadow-md",
    // durum
    secili && "ring-2 ring-vurgu"
  )}
/>
```

> 20-30 sınıfa ulaşan bir eleman muhtemelen ayrı bir bileşen olmayı hak ediyordur.

### 6. Ne zaman düz CSS yazmalı?

- Karmaşık `@keyframes` (→ `@theme` içinde)
- Üçüncü parti kütüphanelerin iç elemanları
- Çok karmaşık seçiciler

---

## 12. Erişilebilirlik

| Konu | Sınıf | Not |
|---|---|---|
| Görsel olarak gizli ama okunur metin | `sr-only` | İkon butonlarında etiket |
| Klavye odağı | `focus-visible:outline-2 focus-visible:outline-offset-2` | `outline-hidden`'ı yerine bir şey koymadan **kullanma** |
| Hareket hassasiyeti | `motion-safe:animate-bounce`, `motion-reduce:transition-none` | Animasyonu yalnızca izin varsa çalıştır |
| Kontrast | Açık gri üstüne açık gri kullanma | Metin için en az `gray-600` |
| Dokunma hedefi | `min-h-11 min-w-11` (44px) | Mobilde küçük butonlar zor tıklanır |
| Yüksek kontrast modu | `forced-colors:` | Windows yüksek kontrast modu |

```jsx
<button className="rounded-full p-2 hover:bg-gray-100 focus-visible:outline-2 focus-visible:outline-blue-600">
  <XIkonu className="size-5" aria-hidden="true" />
  <span className="sr-only">Pencereyi kapat</span>
</button>
```

---

## 13. v3'ten v4'e Önemli Farklar

| v3 | v4 |
|---|---|
| `tailwind.config.js` | CSS içinde `@theme` |
| `@tailwind base; @tailwind components; @tailwind utilities;` | `@import "tailwindcss";` |
| `content: [...]` ayarı | Otomatik tarama (+ gerekirse `@source`) |
| PostCSS + autoprefixer | `@tailwindcss/vite` (dahili) |
| `bg-opacity-50`, `text-opacity-75` | `bg-black/50`, `text-blue-600/75` |
| `bg-gradient-to-r` | `bg-linear-to-r` |
| `shadow-sm` / `shadow` | `shadow-xs` / `shadow-sm` |
| `rounded-sm` / `rounded` | `rounded-xs` / `rounded-sm` |
| `blur-sm` / `blur` | `blur-xs` / `blur-sm` |
| `ring` (3px) | `ring-3` (`ring` artık 1px) |
| `outline-none` | `outline-hidden` |
| `flex-shrink-0`, `flex-grow` | `shrink-0`, `grow` |
| `!bg-red-500` | `bg-red-500!` |
| `bg-[--degisken]` | `bg-(--degisken)` |
| Varsayılan kenarlık rengi `gray-200` | `currentColor` |
| Butonlarda `cursor: pointer` | `cursor: default` |
| Container queries eklentisi | Yerleşik (`@container`, `@md:`) |
| `darkMode: "class"` | `@custom-variant dark (...)` |
| Varyant zinciri bazı durumlarda sağdan sola | Soldan sağa |

### Otomatik yükseltme

```bash
npx @tailwindcss/upgrade
```

---

## 14. Her Şey Bir Arada: Responsive Ürün Paneli

```css
/* ============ src/index.css ============ */
@import "tailwindcss";
@custom-variant dark (&:where([data-tema="koyu"], [data-tema="koyu"] *));

/* Anlamsal tema değişkenleri */
:root {
  --yuzey: var(--color-white);
  --yuzey-ikincil: var(--color-slate-50);
  --metin: var(--color-slate-900);
  --metin-soluk: var(--color-slate-500);
  --kenar: var(--color-slate-200);
  --vurgu: var(--color-indigo-600);
  --vurgu-hover: var(--color-indigo-700);
}
[data-tema="koyu"] {
  --yuzey: var(--color-slate-900);
  --yuzey-ikincil: var(--color-slate-950);
  --metin: var(--color-slate-100);
  --metin-soluk: var(--color-slate-400);
  --kenar: var(--color-slate-800);
  --vurgu: var(--color-indigo-400);
  --vurgu-hover: var(--color-indigo-300);
}

@theme inline {
  --color-yuzey: var(--yuzey);
  --color-yuzey-ikincil: var(--yuzey-ikincil);
  --color-metin: var(--metin);
  --color-metin-soluk: var(--metin-soluk);
  --color-kenar: var(--kenar);
  --color-vurgu: var(--vurgu);
  --color-vurgu-hover: var(--vurgu-hover);
}

@theme {
  --font-govde: "Inter", ui-sans-serif, system-ui, sans-serif;
  --radius-kart: 1rem;
  --animate-sallan: sallan 0.35s ease-in-out;
  @keyframes sallan {
    0%, 100% { transform: translateX(0); }
    25% { transform: translateX(-4px); }
    75% { transform: translateX(4px); }
  }
}

@layer base {
  html { @apply scheme-light dark:scheme-dark; }
  body { @apply bg-yuzey-ikincil font-govde text-metin antialiased; }
}
```

```js
// ============ src/lib/cn.js ============
import { clsx } from "clsx";
import { twMerge } from "tailwind-merge";
export const cn = (...girdiler) => twMerge(clsx(girdiler));
```

```jsx
// ============ src/ui/Buton.jsx ============
import { cva } from "class-variance-authority";
import { cn } from "../lib/cn";

const butonStilleri = cva(
  "inline-flex cursor-pointer items-center justify-center gap-2 rounded-lg font-medium transition-colors " +
  "focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-vurgu " +
  "disabled:pointer-events-none disabled:opacity-50",
  {
    variants: {
      varyant: {
        birincil: "bg-vurgu text-white hover:bg-vurgu-hover dark:text-slate-950",
        cerceve: "border border-kenar bg-yuzey hover:bg-yuzey-ikincil",
        hayalet: "hover:bg-yuzey-ikincil",
      },
      boyut: {
        kucuk: "h-8 px-3 text-sm",
        orta: "h-10 px-4",
        ikon: "size-10",
      },
    },
    defaultVariants: { varyant: "birincil", boyut: "orta" },
  }
);

export function Buton({ varyant, boyut, className, ...rest }) {
  return <button className={cn(butonStilleri({ varyant, boyut }), className)} {...rest} />;
}
```

```jsx
// ============ src/UrunPaneli.jsx ============
import { useState } from "react";
import { cn } from "./lib/cn";
import { Buton } from "./ui/Buton";

const URUNLER = [
  { id: 1, ad: "Mekanik Klavye", kategori: "Aksesuar", fiyat: 2499, stok: 12, yeni: true },
  { id: 2, ad: "Kablosuz Mouse", kategori: "Aksesuar", fiyat: 899, stok: 0, yeni: false },
  { id: 3, ad: "4K Monitör", kategori: "Ekran", fiyat: 11999, stok: 3, yeni: true },
  { id: 4, ad: "USB-C Hub", kategori: "Aksesuar", fiyat: 649, stok: 25, yeni: false },
  { id: 5, ad: "Dizüstü Standı", kategori: "Ergonomi", fiyat: 549, stok: 8, yeni: false },
  { id: 6, ad: "Web Kamerası", kategori: "Aksesuar", fiyat: 1299, stok: 2, yeni: false },
];

const STOK_ROZETI = {
  tukendi: "bg-red-100 text-red-700 dark:bg-red-950 dark:text-red-300",
  az: "bg-amber-100 text-amber-800 dark:bg-amber-950 dark:text-amber-300",
  var: "bg-emerald-100 text-emerald-800 dark:bg-emerald-950 dark:text-emerald-300",
};

function stokDurumu(stok) {
  if (stok === 0) return { anahtar: "tukendi", metin: "Tükendi" };
  if (stok <= 3) return { anahtar: "az", metin: `Son ${stok}` };
  return { anahtar: "var", metin: "Stokta" };
}

// --- Container query ile kendini uyarlayan kart ---
function UrunKarti({ urun, favori, onFavori, onEkle }) {
  const stok = stokDurumu(urun.stok);

  return (
    <article className="@container">
      <div
        data-tukendi={urun.stok === 0 || undefined}
        className={cn(
          "group relative flex flex-col gap-4 rounded-kart border border-kenar bg-yuzey p-4",
          "@sm:flex-row @sm:items-center",
          "transition hover:-translate-y-0.5 hover:shadow-lg",
          "motion-reduce:transition-none motion-reduce:hover:translate-y-0",
          "data-tukendi:opacity-60"
        )}
      >
        <div className="grid aspect-square w-full shrink-0 place-items-center rounded-xl bg-linear-to-br from-indigo-100 to-violet-100 text-4xl @sm:w-24 dark:from-indigo-950 dark:to-violet-950">
          <span aria-hidden="true">📦</span>
        </div>

        <div className="min-w-0 flex-1">
          <div className="flex items-center gap-2">
            <h3 className="truncate font-semibold group-hover:text-vurgu">{urun.ad}</h3>
            {urun.yeni && (
              <span className="shrink-0 rounded-full bg-vurgu/10 px-2 py-0.5 text-xs font-medium text-vurgu">
                Yeni
              </span>
            )}
          </div>
          <p className="text-sm text-metin-soluk">{urun.kategori}</p>
          <div className="mt-2 flex items-center gap-2">
            <span className="text-lg font-bold tabular-nums">
              {urun.fiyat.toLocaleString("tr-TR")} ₺
            </span>
            <span className={cn("rounded-md px-1.5 py-0.5 text-xs font-medium", STOK_ROZETI[stok.anahtar])}>
              {stok.metin}
            </span>
          </div>
        </div>

        <div className="flex gap-2 @sm:flex-col">
          <Buton
            varyant="hayalet"
            boyut="ikon"
            aria-pressed={favori}
            onClick={onFavori}
            className="text-xl aria-pressed:text-rose-500"
          >
            <span aria-hidden="true">{favori ? "♥" : "♡"}</span>
            <span className="sr-only">{favori ? "Favorilerden çıkar" : "Favorilere ekle"}</span>
          </Buton>
          <Buton
            boyut="kucuk"
            disabled={urun.stok === 0}
            onClick={onEkle}
            className="flex-1 @sm:flex-none"
          >
            Sepete ekle
          </Buton>
        </div>
      </div>
    </article>
  );
}

// --- İskelet (yükleniyor) ---
function UrunIskeleti() {
  return (
    <div
      className="flex animate-pulse flex-col gap-3 rounded-kart border border-kenar bg-yuzey p-4"
      aria-hidden="true"
    >
      <div className="aspect-square w-full rounded-xl bg-kenar" />
      <div className="h-4 w-3/4 rounded bg-kenar" />
      <div className="h-4 w-1/3 rounded bg-kenar" />
    </div>
  );
}

// --- Filtre: has-checked ile vurgulanan seçenekler ---
function KategoriFiltresi({ kategoriler, secili, onDegis }) {
  return (
    <fieldset className="flex flex-wrap gap-2">
      <legend className="sr-only">Kategori</legend>
      {["Tümü", ...kategoriler].map((k) => (
        <label
          key={k}
          className={cn(
            "cursor-pointer rounded-full border border-kenar px-3 py-1.5 text-sm transition",
            "hover:bg-yuzey-ikincil",
            "has-checked:border-vurgu has-checked:bg-vurgu has-checked:text-white dark:has-checked:text-slate-950",
            "has-focus-visible:outline-2 has-focus-visible:outline-offset-2 has-focus-visible:outline-vurgu"
          )}
        >
          <input
            type="radio"
            name="kategori"
            value={k}
            checked={secili === k}
            onChange={() => onDegis(k)}
            className="sr-only"
          />
          {k}
        </label>
      ))}
    </fieldset>
  );
}

// --- Arama: peer ile CSS-only uyarı, hata durumunda sallanma ---
function AramaKutusu({ deger, onDegis }) {
  const cokKisa = deger.length > 0 && deger.length < 2;
  return (
    <div className="relative w-full sm:max-w-xs">
      <input
        type="search"
        value={deger}
        onChange={(e) => onDegis(e.target.value)}
        placeholder="Ürün ara..."
        aria-invalid={cokKisa || undefined}
        className={cn(
          "peer h-10 w-full rounded-lg border border-kenar bg-yuzey px-3 placeholder:text-metin-soluk",
          "focus:border-vurgu focus:outline-2 focus:outline-vurgu/20",
          "aria-invalid:animate-sallan aria-invalid:border-red-500"
        )}
      />
      <p className="mt-1 hidden text-xs text-red-600 peer-aria-invalid:block">
        En az 2 karakter yazın.
      </p>
    </div>
  );
}

// --- Bildirim: starting: ile giriş animasyonu ---
function Bildirim({ mesaj }) {
  return (
    <div
      role="status"
      className={cn(
        "fixed right-4 bottom-4 z-50 rounded-lg bg-slate-900 px-4 py-3 text-sm text-white shadow-xl",
        "dark:bg-white dark:text-slate-900",
        "translate-y-0 opacity-100 transition duration-300",
        "starting:translate-y-4 starting:opacity-0"
      )}
    >
      {mesaj}
    </div>
  );
}

// --- Ana panel ---
export default function UrunPaneli() {
  const [tema, setTema] = useState("acik");
  const [arama, setArama] = useState("");
  const [kategori, setKategori] = useState("Tümü");
  const [favoriler, setFavoriler] = useState(new Set());
  const [bildirim, setBildirim] = useState(null);
  const [yukleniyor, setYukleniyor] = useState(false);

  const kategoriler = [...new Set(URUNLER.map((u) => u.kategori))];
  const q = arama.toLocaleLowerCase("tr");
  const gorunenler = URUNLER.filter(
    (u) =>
      (kategori === "Tümü" || u.kategori === kategori) &&
      (q.length < 2 || u.ad.toLocaleLowerCase("tr").includes(q))
  );

  function temaDegistir() {
    const yeni = tema === "acik" ? "koyu" : "acik";
    setTema(yeni);
    document.documentElement.dataset.tema = yeni;   // gerçek projede TemaProvider (Konu 13)
  }

  function favoriDegistir(id) {
    setFavoriler((onceki) => {
      const yeni = new Set(onceki);
      yeni.has(id) ? yeni.delete(id) : yeni.add(id);
      return yeni;
    });
  }

  function sepeteEkle(urun) {
    setBildirim(`${urun.ad} sepete eklendi`);
    setTimeout(() => setBildirim(null), 2500);
  }

  function yenile() {
    setYukleniyor(true);
    setTimeout(() => setYukleniyor(false), 1200);
  }

  return (
    <div className="mx-auto max-w-6xl px-4 py-8 sm:px-6 lg:px-8">
      {/* Başlık: mobilde alt alta, sm+ yan yana */}
      <header className="mb-6 flex flex-col gap-4 sm:flex-row sm:items-center sm:justify-between">
        <div>
          <h1 className="text-2xl font-bold tracking-tight text-balance md:text-3xl">Ürün Paneli</h1>
          <p className="text-sm text-metin-soluk">
            {gorunenler.length} ürün · {favoriler.size} favori
          </p>
        </div>
        <div className="flex gap-2">
          <Buton varyant="cerceve" boyut="kucuk" onClick={yenile}>
            <span className={cn(yukleniyor && "animate-spin")} aria-hidden="true">↻</span>
            Yenile
          </Buton>
          <Buton varyant="cerceve" boyut="ikon" onClick={temaDegistir}>
            <span aria-hidden="true">{tema === "acik" ? "🌙" : "☀️"}</span>
            <span className="sr-only">Temayı değiştir</span>
          </Buton>
        </div>
      </header>

      {/* Filtre çubuğu */}
      <div className="mb-6 flex flex-col gap-4 rounded-kart border border-kenar bg-yuzey p-4 md:flex-row md:items-start md:justify-between">
        <AramaKutusu deger={arama} onDegis={setArama} />
        <KategoriFiltresi kategoriler={kategoriler} secili={kategori} onDegis={setKategori} />
      </div>

      {/* Izgara: 1 → 2 → 3 sütun */}
      {yukleniyor ? (
        <div className="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-3">
          {Array.from({ length: 6 }, (_, i) => <UrunIskeleti key={i} />)}
        </div>
      ) : gorunenler.length === 0 ? (
        <div className="grid place-items-center rounded-kart border border-dashed border-kenar py-16 text-center">
          <p className="text-4xl" aria-hidden="true">🔍</p>
          <p className="mt-2 font-medium">Sonuç bulunamadı</p>
          <p className="text-sm text-metin-soluk">Farklı bir arama veya kategori deneyin.</p>
        </div>
      ) : (
        <div className="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-3">
          {gorunenler.map((u) => (
            <UrunKarti
              key={u.id}
              urun={u}
              favori={favoriler.has(u.id)}
              onFavori={() => favoriDegistir(u.id)}
              onEkle={() => sepeteEkle(u)}
            />
          ))}
        </div>
      )}

      {/* Geniş alan: aynı kart container query ile yatay düzene geçer */}
      <section className="mt-10">
        <h2 className="mb-3 text-lg font-semibold">Öne çıkan</h2>
        <UrunKarti
          urun={URUNLER[2]}
          favori={favoriler.has(3)}
          onFavori={() => favoriDegistir(3)}
          onEkle={() => sepeteEkle(URUNLER[2])}
        />
      </section>

      {bildirim && <Bildirim key={bildirim} mesaj={bildirim} />}
    </div>
  );
}
```

### Bu örnekte kullanılanlar

| Teknik | Nerede |
|---|---|
| **Anlamsal renkler** (`bg-yuzey`, `text-metin-soluk`, `border-kenar`) | Bileşenlerde neredeyse hiç `dark:` yok |
| **Container queries** | `UrunKarti` ızgarada dikey, "Öne çıkan"da yatay — prop değişmeden |
| **Mobile-first responsive** | `grid-cols-1 sm:grid-cols-2 lg:grid-cols-3`, `flex-col sm:flex-row` |
| **`cva` + `cn`** | Buton varyantları tek yerde; dışarıdan `className` güvenle ekleniyor |
| **`group-hover:`** | Kart üzerine gelince başlık rengi |
| **`has-checked:`, `has-focus-visible:`** | Seçili kategori etiketi |
| **`peer-aria-invalid:`** | State'siz uyarı mesajı |
| **`aria-pressed:`** | Favori kalbi |
| **`data-tukendi:`** | Tükenen ürün soluk |
| **`aria-invalid:animate-sallan`** | `@theme`'de tanımlanan özel animasyon |
| **`starting:`** | Bildirimin giriş animasyonu (JS kütüphanesi olmadan) |
| **`animate-pulse`** | İskelet ekranı |
| **Erişilebilirlik** | `sr-only`, `focus-visible`, `motion-reduce`, gizli ama klavyeyle erişilebilir radio'lar, `role="status"` |
| **Tipografi detayları** | `truncate` + `min-w-0`, `tabular-nums`, `text-balance` |

---

## 15. Özet

- **Utility-first:** Stil doğrudan elemana, tek işlevli sınıflarla yazılır; inline stilden farkı tasarım sistemi, varyantlar ve tek CSS dosyasıdır.
- Kurulum: `@tailwindcss/vite` + `@import "tailwindcss"`; **IntelliSense** ve **prettier-plugin-tailwindcss** kurulmalı.
- Tailwind kaynak kodu **metin olarak tarar** ve yalnızca kullanılan sınıfları üretir; ek kaynaklar `@source` ile eklenir.
- **Preflight** başlık, liste ve buton varsayılanlarını sıfırlar; butonlara `cursor-pointer` eklenmelidir.
- Boşluk ölçeği `--spacing` (0.25rem) katlarıdır: **sayı × 4 = piksel**; v4'te her tam sayı geçerlidir.
- Renkler `50`–`950` tonlarında, `oklch` tabanlıdır; opaklık için `/` (`bg-black/50`).
- Rastgele değer `[...]`, CSS değişkeni `(--x)`, important `sınıf!`.
- **Varyantlar:** durumlar, pseudo-element'ler, yapısal (`*:`, `odd:`), `group`, `peer`, `has-*`, `not-*`, `aria-*`, `data-*`, `starting:`, rastgele (`[&>li]:`); soldan sağa zincirlenir.
- **Mobile-first:** Öneksiz sınıflar her ekranda, `sm:`/`md:`/`lg:` o genişlikten itibaren; `max-*` ile aralıklar.
- **Container queries** (`@container`, `@md:`) bileşeni kapsayıcısının genişliğine göre uyarlar.
- **Koyu mod:** `dark:` veya `@custom-variant dark`; büyük projelerde **anlamsal renk değişkenleri** + `@theme inline`.
- **`@theme`** ad alanlarıyla (`--color-*`, `--font-*`, `--radius-*`, `--animate-*`...) tasarım sistemini tanımlar; `--color-*: initial` ile ad alanı sıfırlanır.
- Kendi CSS'i: `@layer base`, `@utility`, `@custom-variant`, `@reference`, `@plugin` (Typography); `@apply` React'te idareli kullanılır.
- **React kalıpları:** `cn` (`clsx` + `tailwind-merge`), tam sınıf adları / eşleme nesneleri, **`cva`** ile varyantlar, state'i `aria-*`/`data-*` ile yansıtmak, uzun listeleri gruplamak veya bileşene çıkarmak.
- **Erişilebilirlik:** `sr-only`, `focus-visible`, `motion-reduce`, kontrast, dokunma hedefi.
- **v3 → v4:** CSS-first yapılandırma, isim değişiklikleri (`shadow-xs`, `rounded-xs`, `outline-hidden`, `bg-linear-*`), `npx @tailwindcss/upgrade`.

---

⬅️ Önceki: [16 · Stil Yöntemleri](./16-stil-yontemleri.md) · ➡️ Sonraki: [17 · React Router](./17-react-router.md)
