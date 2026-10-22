# 16 · Stil Yöntemleri

> **Konu özeti:** React'te bileşenleri stillendirme yöntemleri: global CSS, inline stil, CSS değişkenleri, CSS Modules, Sass ve modern CSS, Tailwind CSS v4, CSS-in-JS, bileşen kütüphaneleri, tema ve koyu mod, animasyonlar ve yöntem seçimi.

---

## İçindekiler

1. [Genel Bakış](#1-genel-bakış)
2. [Global CSS Dosyaları](#2-global-css-dosyaları)
3. [Inline Stil](#3-inline-stil)
4. [CSS Değişkenleri: Dinamik Stillerin En Temiz Yolu](#4-css-değişkenleri-dinamik-stillerin-en-temiz-yolu-)
5. [CSS Modules](#5-css-modules-)
6. [Sass / SCSS ve Modern CSS](#6-sass--scss-ve-modern-css)
7. [Tailwind CSS](#7-tailwind-css-)
8. [CSS-in-JS](#8-css-in-js)
9. [Bileşen Kütüphaneleri](#9-bileşen-kütüphaneleri)
10. [Tema ve Koyu Mod](#10-tema-ve-koyu-mod)
11. [Animasyon ve Geçişler](#11-animasyon-ve-geçişler)
12. [Hangisini Seçmeli?](#12-hangisini-seçmeli)
13. [Her Şey Bir Arada: Aynı Ürün Kartı, İki Yöntem](#13-her-şey-bir-arada-aynı-ürün-kartı-iki-yöntem)
14. [Özet](#14-özet)

---

## 1. Genel Bakış

React stil konusunda **hiçbir yöntem dayatmaz**; HTML'i stillendirebilen her yöntem React ile de çalışır.

### Karşılaştırma kriterleri

1. **Kapsam (scope):** Bir bileşenin stili başkalarını yanlışlıkla etkileyebilir mi?
2. **Dinamik stiller:** Props/state'e göre stil değiştirmek ne kadar kolay?
3. **Performans:** Stil derleme anında mı, çalışma anında mı üretiliyor?
4. **Geliştirici deneyimi:** Yazmak, okumak, bakımı kolay mı?

| Yöntem | Özet |
|---|---|
| Global CSS dosyaları | Klasik CSS, `className` ile |
| Inline stil | `style={{ ... }}` |
| CSS değişkenleri | Dinamik değerler için köprü |
| CSS Modules | Otomatik kapsamlı CSS dosyaları |
| Sass / SCSS | CSS'e değişken, iç içe yazım vb. ekleyen dil |
| Tailwind CSS | Hazır yardımcı (utility) sınıflar |
| CSS-in-JS | Stilleri JavaScript içinde yazmak |
| Bileşen kütüphaneleri | Hazır, stilli veya stilsiz bileşenler |

---

## 2. Global CSS Dosyaları

```css
/* src/components/Kart.css */
.kart {
  padding: 16px;
  border-radius: 8px;
  background: white;
  box-shadow: 0 1px 3px rgb(0 0 0 / 0.1);
}

.kart .baslik {
  font-size: 1.25rem;
  font-weight: 600;
}
```

```jsx
// src/components/Kart.jsx
import "./Kart.css";

export default function Kart({ baslik, children }) {
  return (
    <div className="kart">
      <h3 className="baslik">{baslik}</h3>
      {children}
    </div>
  );
}
```

### Sorun: Her şey global

- `Kart.css`'i sadece `Kart.jsx` import etse de `.baslik` **tüm sayfada** geçerlidir.
- Aynı sınıf adı başka yerde kullanılırsa stiller **çakışır**; kazanan kural yüklenme sırasına ve özgüllüğe bağlıdır.

### Geleneksel çözüm: BEM isimlendirmesi

```css
.kart { }              /* Block */
.kart__baslik { }      /* Element */
.kart--vurgulu { }     /* Modifier */
```

> İşe yarar ama tamamen insan disiplinine dayanır.

### Ne zaman uygun?

- CSS reset, temel tipografi, `body` stilleri gibi gerçekten **global** kurallar
- Tema değişkenlerinin (`:root { --renk-ana: ... }`) tanımlanması
- Küçük projeler

---

## 3. Inline Stil

```jsx
<div style={{ padding: 16, backgroundColor: secili ? "#eef" : "white" }}>...</div>
```

### Kısıtlar

- `:hover`, `:focus`, `::before` gibi **pseudo-class/element** yok
- **Medya sorguları** yok → responsive tasarım yapılamaz
- **`@keyframes`** animasyon yok
- Tekrar kullanımı zor; yüksek özgüllük nedeniyle dışarıdan ezmek zor

> **Ne zaman?** Yalnızca **çalışma anında hesaplanan** değerler (ilerleme yüzdesi, sürüklenen elemanın konumu, kullanıcının seçtiği renk) için — ve bunun için bile CSS değişkenleri daha iyidir.

---

## 4. CSS Değişkenleri: Dinamik Stillerin En Temiz Yolu ⭐

**Değer** inline verilir, **nasıl kullanılacağı** CSS'te tanımlanır:

```css
/* IlerlemeCubugu.css */
.ilerleme {
  height: 8px;
  background: #e5e7eb;
  border-radius: 999px;
  overflow: hidden;
}
.ilerleme__dolgu {
  height: 100%;
  width: var(--yuzde);                  /* değer dışarıdan */
  background: var(--renk, #3b82f6);     /* yedek değerli */
  transition: width 0.3s ease;
}
.ilerleme__dolgu:hover {               /* inline stilde imkânsız */
  filter: brightness(1.1);
}
@media (prefers-reduced-motion: reduce) {
  .ilerleme__dolgu { transition: none; }
}
```

```jsx
function IlerlemeCubugu({ yuzde, renk }) {
  return (
    <div
      className="ilerleme"
      role="progressbar"
      aria-valuenow={yuzde}
      aria-valuemin={0}
      aria-valuemax={100}
    >
      <div
        className="ilerleme__dolgu"
        style={{ "--yuzde": `${yuzde}%`, "--renk": renk }}
      />
    </div>
  );
}
```

- React, `--` ile başlayan anahtarları olduğu gibi CSS değişkeni olarak ayarlar.
- Dinamik değer JS'te; hover, medya sorgusu, animasyon CSS'te kalır.
- Tema sistemlerinin de temelidir.

> Bu kalıp, aşağıdaki **tüm** yöntemlerle birlikte kullanılabilir.

---

## 5. CSS Modules ⭐

Normal CSS yazılır ama **sınıf adları otomatik olarak benzersiz** yapılır. Vite'ta kurulum gerekmez; dosya adı `.module.css` ile bitmelidir.

```css
/* src/components/Kart.module.css */
.kart {
  padding: 16px;
  border-radius: 8px;
  background: white;
  box-shadow: 0 1px 3px rgb(0 0 0 / 0.1);
}

.baslik {
  font-size: 1.25rem;
  font-weight: 600;
}

.vurgulu {
  border: 2px solid #3b82f6;
}
```

```jsx
// src/components/Kart.jsx
import styles from "./Kart.module.css";

export default function Kart({ baslik, vurgulu, children }) {
  return (
    <div className={vurgulu ? `${styles.kart} ${styles.vurgulu}` : styles.kart}>
      <h3 className={styles.baslik}>{baslik}</h3>
      {children}
    </div>
  );
}
```

### Ne oluyor?

```js
console.log(styles);
// { kart: "_kart_x7k2p_1", baslik: "_baslik_x7k2p_8", vurgulu: "_vurgulu_x7k2p_13" }
```

- DOM'da `<div class="_kart_x7k2p_1">` görülür.
- Başka dosyadaki `.baslik` farklı ad alır → **çakışma imkânsız.**

### İsimlendirme

| CSS'te | JSX'te |
|---|---|
| `.kartBaslik` (camelCase) | `styles.kartBaslik` ✅ pratik |
| `.kart-baslik` (tireli) | `styles["kart-baslik"]` |

### Birden fazla ve koşullu sınıf: `clsx`

```jsx
import clsx from "clsx";
import styles from "./Buton.module.css";

function Buton({ varyant = "birincil", boyut = "orta", tamGenislik, className, ...rest }) {
  return (
    <button
      className={clsx(
        styles.buton,
        styles[varyant],              // dinamik anahtar
        styles[boyut],
        tamGenislik && styles.tamGenislik,
        className                     // dışarıdan ek sınıf
      )}
      {...rest}
    />
  );
}
```

```css
/* Buton.module.css */
.buton { border: none; border-radius: 6px; cursor: pointer; font-weight: 500; }
.buton:disabled { opacity: 0.5; cursor: not-allowed; }
.buton:focus-visible { outline: 2px solid #3b82f6; outline-offset: 2px; }

.birincil { background: #3b82f6; color: white; }
.birincil:hover:not(:disabled) { background: #2563eb; }
.ikincil { background: #e5e7eb; color: #111827; }
.tehlike { background: #dc2626; color: white; }

.kucuk { padding: 4px 10px; font-size: 0.875rem; }
.orta { padding: 8px 16px; }
.buyuk { padding: 12px 24px; font-size: 1.125rem; }

.tamGenislik { width: 100%; }
```

### Global sınıflara kaçış

```css
.kart :global(.leaflet-container) { border-radius: 8px; }
```

### Artılar / Eksiler

| ✅ Artılar | ❌ Eksiler |
|---|---|
| Saf CSS bilgisi yeterli, sıfır kurulum | CSS ve JSX ayrı dosyalarda |
| Çalışma anı maliyeti yok, çakışma yok | `styles.xyz` tekrarı |
| Pseudo-class, medya sorgusu, animasyonun tamamı | Tasarım tutarlılığı için ayrıca bir sistem (CSS değişkenleri) gerekir |

---

## 6. Sass / SCSS ve Modern CSS

**Sass**, CSS'e değişken, iç içe yazım, mixin ve fonksiyon ekler.

```bash
npm install -D sass
```

```scss
// Kart.module.scss
$golge: 0 1px 3px rgb(0 0 0 / 0.1);

.kart {
  padding: 16px;
  box-shadow: $golge;

  &:hover { box-shadow: 0 4px 12px rgb(0 0 0 / 0.15); }

  .baslik {
    font-weight: 600;
  }

  @media (max-width: 640px) {
    padding: 12px;
  }
}
```

- CSS Modules ile birlikte çalışır (`.module.scss`).

### Modern CSS hızla yetişti

| Sass özelliği | Modern CSS karşılığı |
|---|---|
| `$değişken` | CSS custom properties (`--renk`), çalışma anında değiştirilebilir |
| İç içe yazım | **Native CSS nesting** (tüm modern tarayıcılar) |
| Renk fonksiyonları | `color-mix()`, `oklch()` |

```css
/* Native CSS nesting */
.kart {
  padding: 16px;
  &:hover { box-shadow: 0 4px 12px rgb(0 0 0 / 0.15); }
  .baslik { font-weight: 600; }
}
```

> Yeni projelerde Sass'a ihtiyaç azalıyor; mevcut Sass kod tabanlarında sorunsuz kullanılmaya devam ediyor.

---

## 7. Tailwind CSS ⭐

**Felsefe:** CSS yazmak yerine, her biri **tek bir iş** yapan hazır sınıfları (utility) birleştirmek.

```jsx
function Kart({ baslik, children }) {
  return (
    <div className="rounded-lg bg-white p-4 shadow-sm">
      <h3 className="text-xl font-semibold">{baslik}</h3>
      {children}
    </div>
  );
}
```

| Sınıf | CSS |
|---|---|
| `p-4` | `padding: 1rem` |
| `rounded-lg` | `border-radius: 0.5rem` |
| `bg-white` | `background-color: white` |
| `text-xl` | büyük yazı boyutu |
| `font-semibold` | `font-weight: 600` |
| `shadow-sm` | hafif gölge |

- Tailwind derleme sırasında kodu tarar ve **sadece kullanılan sınıfların** CSS'ini üretir.

### Kurulum (Tailwind v4 + Vite)

> Güncel sürüm **v4** serisi. v3'e göre çok sade: `tailwind.config.js` ve PostCSS ayarı çoğu durumda gerekmez. İnternetteki birçok eğitim hâlâ v3 kurulumunu gösterir.

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
/* src/index.css — v3'teki üç @tailwind satırı yerine tek satır */
@import "tailwindcss";
```

> 💡 VS Code için **Tailwind CSS IntelliSense** eklentisi: otomatik tamamlama ve sınıfın ürettiği CSS'i gösterir.

### Önekler (variants): durumlar, responsive, koyu mod

```jsx
<button
  className="rounded-md bg-blue-600 px-4 py-2 text-white
             hover:bg-blue-700
             focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-blue-600
             disabled:cursor-not-allowed disabled:opacity-50"
>
  Kaydet
</button>
```

| Önek | Anlamı |
|---|---|
| `hover:`, `focus:`, `focus-visible:`, `active:`, `disabled:` | Durumlar |
| `sm:`, `md:`, `lg:`, `xl:` | Ekran bu genişlikten **büyükse** (mobile-first) |
| `dark:` | Koyu mod |
| `group-hover:` | `group` sınıflı ebeveyn üzerine gelinince |
| `first:`, `last:`, `odd:` | Liste elemanları |
| `aria-expanded:`, `data-[durum=acik]:` | ARIA ve data nitelikleri |
| `motion-reduce:` | `prefers-reduced-motion` |

#### Mobile-first responsive

```jsx
<div className="grid grid-cols-1 gap-4 sm:grid-cols-2 lg:grid-cols-4">
  {/* telefonda 1, tablette 2, masaüstünde 4 sütun */}
</div>
```

#### Grup

```jsx
<a href="#" className="group block rounded-lg p-4 hover:bg-gray-50">
  <h3 className="font-semibold group-hover:text-blue-600">Başlık</h3>
  <p className="text-gray-500">Kartın üzerine gelince başlık maviye döner</p>
</a>
```

### Kendi tasarım sistemi: `@theme`

v4'te özelleştirme **CSS içinde** yapılır:

```css
@import "tailwindcss";

@theme {
  --color-marka: oklch(0.6 0.2 260);
  --color-marka-koyu: oklch(0.5 0.2 260);
  --font-govde: "Inter", sans-serif;
  --breakpoint-3xl: 1920px;
}
```

- Otomatik sınıflar: `bg-marka`, `text-marka-koyu`, `font-govde`, `3xl:grid-cols-6`
- Normal CSS değişkeni olarak da kullanılabilir: `var(--color-marka)`

### ⚠️ Dinamik sınıf tuzağı

Tailwind sınıfları kaynak kodu **metin olarak** tarayarak bulur; sınıf adları parçalardan birleştirilemez:

```jsx
// ❌ "bg-red-500" kodda geçmiyor → CSS üretilmez
<div className={`bg-${renk}-500`} />

// ✅ Tam sınıf adları kodda geçmeli
const RENKLER = {
  kirmizi: "bg-red-500",
  yesil: "bg-green-500",
  mavi: "bg-blue-500",
};
<div className={RENKLER[renk]} />
```

Gerçekten çalışma anında hesaplanan değerler için CSS değişkeni:

```jsx
<div className="bg-(--renk)" style={{ "--renk": kullaniciRengi }} />
```

### Koşullu sınıflar ve `cn` yardımcısı

`clsx` tek başına çakışan Tailwind sınıflarını çözmez:

```jsx
function Buton({ className, ...rest }) {
  return <button className={clsx("px-4 py-2 bg-blue-600", className)} {...rest} />;
}

<Buton className="bg-red-600" />
// class="px-4 py-2 bg-blue-600 bg-red-600" → hangisi kazanır? Öngörülemez
```

**tailwind-merge** çakışan sınıfları birleştirir (sonra geleni tutar):

```bash
npm install clsx tailwind-merge
```

```js
// src/lib/cn.js
import { clsx } from "clsx";
import { twMerge } from "tailwind-merge";

export function cn(...girdiler) {
  return twMerge(clsx(girdiler));
}
```

```jsx
<button className={cn("px-4 py-2 bg-blue-600", aktif && "ring-2", className)} />
// cn("bg-blue-600", "bg-red-600") → "bg-red-600"
```

### Tekrarı azaltmak: Bileşen çıkarmak

> React'te stil tekrarının çözümü CSS'te değil, **bileşen** düzeyindedir.

```jsx
const VARYANTLAR = {
  birincil: "bg-blue-600 text-white hover:bg-blue-700",
  ikincil: "bg-gray-100 text-gray-900 hover:bg-gray-200",
  tehlike: "bg-red-600 text-white hover:bg-red-700",
};

const BOYUTLAR = {
  kucuk: "px-2.5 py-1 text-sm",
  orta: "px-4 py-2",
  buyuk: "px-6 py-3 text-lg",
};

export function Buton({ varyant = "birincil", boyut = "orta", className, ...rest }) {
  return (
    <button
      className={cn(
        "inline-flex items-center justify-center gap-2 rounded-md font-medium transition-colors",
        "focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-blue-600",
        "disabled:pointer-events-none disabled:opacity-50",
        VARYANTLAR[varyant],
        BOYUTLAR[boyut],
        className
      )}
      {...rest}
    />
  );
}

// Kullanım
<Buton varyant="tehlike">Sil</Buton>
```

- Bu varyant kalıbını düzenleyen yaygın kütüphane: **cva** (class-variance-authority).
- `@apply` mümkündür ama React projelerinde bileşen çıkarmak daha esnektir.

### Artılar / Eksiler

| ✅ Artılar | ❌ Eksiler |
|---|---|
| İsim bulma ve dosyalar arası gidip gelme yok | JSX'te uzun sınıf listeleri |
| Hazır, tutarlı tasarım sistemi (boşluk, renk, yazı ölçekleri) | Öğrenilmesi gereken bir "sözlük" |
| Kullanılmayan CSS üretilmez | Dinamik sınıf kısıtı |
| Stil elemanın yanında; ölü CSS yok | Karmaşık seçicilerde CSS dosyası kadar rahat değil |
| Çalışma anı maliyeti yok, Server Components uyumlu | Takımca benimsenmesi gerekir |

> Bugün piyasadaki React projelerinin büyük kısmında karşına çıkar; okuyabilecek kadar bilmek neredeyse zorunludur.

---

## 8. CSS-in-JS

Stiller **JavaScript içinde** yazılır. En bilinenleri **styled-components** ve **Emotion**:

```jsx
import styled from "styled-components";

const Buton = styled.button`
  padding: 8px 16px;
  border-radius: 6px;
  border: none;
  background: ${(props) => (props.$tehlike ? "#dc2626" : "#3b82f6")};
  color: white;

  &:hover { filter: brightness(1.1); }
`;

<Buton $tehlike onClick={handleSil}>Sil</Buton>
```

### Ekosistem neden uzaklaşıyor?

- **Çalışma anı maliyeti:** Stiller tarayıcıda, render sırasında JS ile üretilip enjekte edilir.
- **React Server Components ile uyumsuz** *(Konu 23)*.
- styled-components bakımcıları 2025'te kütüphaneyi **bakım moduna** aldıklarını duyurdu; yeni projeler için önermiyorlar.

### Sıfır çalışma anı (zero-runtime) alternatifleri

Stiller JS/TS'te yazılır, **derleme sırasında** normal CSS'e dönüştürülür:

- **vanilla-extract**
- **Panda CSS**
- **StyleX** (Meta)

> Yeni projede klasik runtime CSS-in-JS için güçlü bir sebep yok; ama mevcut projelerde (özellikle Emotion tabanlı **MUI** kullananlarda) karşına çıkar.

---

## 9. Bileşen Kütüphaneleri

Özellikle **erişilebilirlik** (klavye gezintisi, odak yönetimi, ARIA) doğru yapılması zor bir iştir; kütüphaneler bunu hazır sunar.

### 1. Stilli (styled) kütüphaneler

Bileşenler hazır görünümle gelir, tema ile özelleştirilir.

- **MUI (Material UI)** — Material Design, kapsamlı, kurumsal projelerde yaygın
- **Mantine**, **Chakra UI**, **Ant Design**

> Hızlı başlanır; kendi tasarımına uydurmak zorlaşabilir.

### 2. Stilsiz (headless) kütüphaneler

Sadece **davranış ve erişilebilirlik**; görünüm tamamen senin.

- **Radix UI**, **React Aria** (Adobe), **Headless UI** (Tailwind ekibi), **Base UI**, **Ark UI**

```jsx
// Radix: davranış hazır, stil Tailwind ile
import * as Dialog from "@radix-ui/react-dialog";

<Dialog.Root>
  <Dialog.Trigger className="rounded bg-blue-600 px-4 py-2 text-white">Aç</Dialog.Trigger>
  <Dialog.Portal>
    <Dialog.Overlay className="fixed inset-0 bg-black/40" />
    <Dialog.Content className="fixed left-1/2 top-1/2 -translate-x-1/2 -translate-y-1/2 rounded-lg bg-white p-6">
      <Dialog.Title className="text-lg font-semibold">Emin misin?</Dialog.Title>
      <Dialog.Close className="mt-4 rounded bg-gray-100 px-3 py-1">Kapat</Dialog.Close>
    </Dialog.Content>
  </Dialog.Portal>
</Dialog.Root>
```

Esc ile kapanma, odağı modal içinde tutma, kapanınca odağı geri verme, ekran okuyucu duyuruları hazırdır.

### 3. shadcn/ui: "Kütüphane değil, kod kopyalama"

- Bileşenler npm paketi olarak değil, **kaynak kodu olarak projene kopyalanır.**
- Altta Radix (veya benzeri) + Tailwind; `cn` ve varyant kalıplarıyla yazılmıştır.
- Kod senin olduğu için istediğin gibi değiştirilir.

```bash
npx shadcn@latest add button dialog
# src/components/ui/button.jsx ve dialog.jsx projene eklenir
```

### Seçim rehberi

| Durum | Öneri |
|---|---|
| Kurumsal yönetim paneli, hız öncelikli | MUI / Mantine |
| Kendi tasarım dilin var | Headless + Tailwind veya shadcn/ui |

---

## 10. Tema ve Koyu Mod

Tema sistemlerinin temeli **CSS değişkenleridir**:

```css
/* index.css */
:root {
  --arkaplan: #ffffff;
  --yazi: #111827;
  --kart: #f9fafb;
  --vurgu: #2563eb;
}

:root[data-tema="koyu"] {
  --arkaplan: #0b1120;
  --yazi: #e5e7eb;
  --kart: #1f2937;
  --vurgu: #60a5fa;
}

/* Kullanıcı tercih etmediyse sistem ayarını izle */
@media (prefers-color-scheme: dark) {
  :root:not([data-tema="acik"]) {
    --arkaplan: #0b1120;
    --yazi: #e5e7eb;
    --kart: #1f2937;
    --vurgu: #60a5fa;
  }
}

body { background: var(--arkaplan); color: var(--yazi); }
```

- Bileşenler yalnızca değişkenleri kullanır (`background: var(--kart)`).
- Tema değişince `<html data-tema="...">` değişir (Konu 13.11'deki `TemaProvider`) ve **hiçbir bileşen yeniden render olmadan** sayfa güncellenir.

### Tailwind'de koyu mod

- `dark:` varsayılan olarak sistem tercihini izler.
- Kendi tema düğmenle kontrol etmek için:

```css
@import "tailwindcss";
@custom-variant dark (&:where([data-tema="koyu"], [data-tema="koyu"] *));
```

```jsx
<div className="bg-white text-gray-900 dark:bg-gray-900 dark:text-gray-100">...</div>
```

---

## 11. Animasyon ve Geçişler

### 1. CSS geçişleri

```jsx
<div className={clsx(styles.panel, acik && styles.acik)}>...</div>
```

```css
.panel { max-height: 0; opacity: 0; overflow: hidden; transition: all 0.3s ease; }
.panel.acik { max-height: 500px; opacity: 1; }
```

- Tailwind'de: `transition-all duration-300` + koşullu sınıflar.
- **Kısıt:** Koşullu render ile ağaçtan **kaldırılan** eleman için çıkış animasyonu yapılamaz; React onu hemen siler.

### 2. Motion (eski adıyla Framer Motion)

React için en yaygın animasyon kütüphanesi: çıkış animasyonları, jestler, sürükleme, layout animasyonları.

```jsx
import { AnimatePresence, motion } from "motion/react";

<AnimatePresence>
  {acik && (
    <motion.div
      initial={{ opacity: 0, y: -10 }}
      animate={{ opacity: 1, y: 0 }}
      exit={{ opacity: 0, y: -10 }}      // kaldırılırken çalışır
    >
      Bildirim
    </motion.div>
  )}
</AnimatePresence>
```

### 3. Erişilebilirlik

`prefers-reduced-motion`'a saygı gösterilmeli (CSS medya sorgusu; Tailwind'de `motion-reduce:`).

---

## 12. Hangisini Seçmeli?

| Yöntem | Kapsam | Dinamik stil | Çalışma anı maliyeti | Öğrenme | Not |
|---|---|---|---|---|---|
| Global CSS | ❌ Global | CSS değişkenleri ile | Yok | Kolay | Reset, tema değişkenleri, küçük projeler |
| Inline stil | ✅ Elemana özel | ✅ Kolay | Düşük | Kolay | Hover/medya yok |
| **CSS Modules** | ✅ Otomatik | CSS değişkenleri + koşullu sınıf | Yok | Kolay | Saf CSS seven için güvenli varsayılan |
| Sass | Modules ile ✅ | Aynı | Yok | Orta | Modern CSS ile ihtiyaç azalıyor |
| **Tailwind** | ✅ (utility) | Tanımlı sınıflar + CSS değişkenleri | Yok | Orta | Bugün en yaygın tercih |
| Runtime CSS-in-JS | ✅ Otomatik | ✅ Çok kolay | ⚠️ Var | Orta | Yeni projede önerilmez; RSC uyumsuz |
| Zero-runtime CSS-in-JS | ✅ Otomatik | Sınırlı | Yok | Orta | Tip güvenli tasarım sistemleri |

### Pratik öneri

- Yeni proje: **Tailwind** (+ shadcn/ui veya headless kütüphane) ya da **CSS Modules** (+ CSS değişkenleri ile tema).
- Global CSS yalnızca reset ve tema değişkenleri için.
- Dinamik değerler için **CSS değişkenleri.**
- Projede **tek bir ana yöntem** seçilir; birden çok stil sistemini karıştırmak bakımı zorlaştırır.

---

## 13. Her Şey Bir Arada: Aynı Ürün Kartı, İki Yöntem

### Ortak veri ve tema değişkenleri

```css
/* src/index.css — her iki yöntemde ortak */
:root {
  --arkaplan: #f8fafc; --kart: #ffffff; --yazi: #0f172a; --soluk: #64748b;
  --vurgu: #2563eb; --vurgu-hover: #1d4ed8; --kenar: #e2e8f0;
}
:root[data-tema="koyu"] {
  --arkaplan: #020617; --kart: #0f172a; --yazi: #e2e8f0; --soluk: #94a3b8;
  --vurgu: #3b82f6; --vurgu-hover: #60a5fa; --kenar: #1e293b;
}
body {
  margin: 0;
  background: var(--arkaplan);
  color: var(--yazi);
  font-family: system-ui, sans-serif;
}
```

```js
// src/urunler.js
export const URUNLER = [
  { id: 1, ad: "Mekanik Klavye", fiyat: 2499, eskiFiyat: 2999, stok: 12, renk: "#8b5cf6" },
  { id: 2, ad: "Kablosuz Mouse", fiyat: 899, eskiFiyat: null, stok: 0, renk: "#10b981" },
  { id: 3, ad: "USB-C Hub", fiyat: 649, eskiFiyat: 799, stok: 3, renk: "#f59e0b" },
  { id: 4, ad: "4K Monitör", fiyat: 11999, eskiFiyat: null, stok: 5, renk: "#ef4444" },
];
```

### Yöntem A: CSS Modules

```css
/* src/modules/UrunKarti.module.css */
.izgara {
  display: grid;
  gap: 16px;
  grid-template-columns: repeat(auto-fill, minmax(220px, 1fr));
}

.kart {
  position: relative;
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: 16px;
  border: 1px solid var(--kenar);
  border-radius: 12px;
  background: var(--kart);
  transition: transform 0.2s ease, box-shadow 0.2s ease;

  &:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 24px rgb(0 0 0 / 0.08);
  }
}

.gorsel {
  aspect-ratio: 4 / 3;
  border-radius: 8px;
  background: color-mix(in oklch, var(--urun-rengi) 20%, transparent);   /* dinamik renk */
  display: grid;
  place-items: center;
  font-size: 2.5rem;
}

.rozet {
  position: absolute;
  top: 12px;
  right: 12px;
  padding: 2px 8px;
  border-radius: 999px;
  background: #dc2626;
  color: white;
  font-size: 0.75rem;
  font-weight: 600;
}

.ad { margin: 0; font-size: 1rem; font-weight: 600; }
.fiyatlar { display: flex; align-items: baseline; gap: 8px; }
.fiyat { font-size: 1.125rem; font-weight: 700; }
.eskiFiyat { color: var(--soluk); text-decoration: line-through; font-size: 0.875rem; }
.stok { color: var(--soluk); font-size: 0.875rem; }
.stokAz { color: #d97706; }

.buton {
  margin-top: auto;
  padding: 8px 12px;
  border: none;
  border-radius: 8px;
  background: var(--vurgu);
  color: white;
  font-weight: 500;
  cursor: pointer;

  &:hover:not(:disabled) { background: var(--vurgu-hover); }
  &:focus-visible { outline: 2px solid var(--vurgu); outline-offset: 2px; }
  &:disabled { opacity: 0.5; cursor: not-allowed; }
}

@media (prefers-reduced-motion: reduce) {
  .kart { transition: none; }
  .kart:hover { transform: none; }
}
```

```jsx
// src/modules/UrunKarti.jsx
import clsx from "clsx";
import styles from "./UrunKarti.module.css";

export function UrunKarti({ urun, onEkle }) {
  const indirim = urun.eskiFiyat ? Math.round((1 - urun.fiyat / urun.eskiFiyat) * 100) : 0;
  const tukendi = urun.stok === 0;
  const azKaldi = urun.stok > 0 && urun.stok <= 3;

  return (
    <article className={styles.kart} style={{ "--urun-rengi": urun.renk }}>
      {indirim > 0 && <span className={styles.rozet}>%{indirim}</span>}
      <div className={styles.gorsel} aria-hidden="true">📦</div>
      <h3 className={styles.ad}>{urun.ad}</h3>
      <div className={styles.fiyatlar}>
        <span className={styles.fiyat}>{urun.fiyat.toLocaleString("tr-TR")} ₺</span>
        {urun.eskiFiyat && (
          <span className={styles.eskiFiyat}>{urun.eskiFiyat.toLocaleString("tr-TR")} ₺</span>
        )}
      </div>
      <p className={clsx(styles.stok, azKaldi && styles.stokAz)}>
        {tukendi ? "Tükendi" : azKaldi ? `Son ${urun.stok} ürün!` : "Stokta"}
      </p>
      <button
        type="button"
        className={styles.buton}
        disabled={tukendi}
        onClick={() => onEkle(urun)}
      >
        Sepete ekle
      </button>
    </article>
  );
}

export function UrunIzgarasi({ urunler, onEkle }) {
  return (
    <div className={styles.izgara}>
      {urunler.map((u) => <UrunKarti key={u.id} urun={u} onEkle={onEkle} />)}
    </div>
  );
}
```

### Yöntem B: Tailwind CSS

```css
/* src/index.css (Tailwind sürümü) — en üste ekle, ortak değişkenler altında kalır */
@import "tailwindcss";
@custom-variant dark (&:where([data-tema="koyu"], [data-tema="koyu"] *));

@theme {
  --color-vurgu: #2563eb;
  --color-vurgu-hover: #1d4ed8;
}
```

```jsx
// src/tailwind/UrunKarti.jsx
import { cn } from "../lib/cn";

export function UrunKarti({ urun, onEkle }) {
  const indirim = urun.eskiFiyat ? Math.round((1 - urun.fiyat / urun.eskiFiyat) * 100) : 0;
  const tukendi = urun.stok === 0;
  const azKaldi = urun.stok > 0 && urun.stok <= 3;

  return (
    <article
      style={{ "--urun-rengi": urun.renk }}
      className={cn(
        "relative flex flex-col gap-2 rounded-xl border p-4",
        "border-slate-200 bg-white dark:border-slate-800 dark:bg-slate-900",
        "transition hover:-translate-y-0.5 hover:shadow-lg",
        "motion-reduce:transition-none motion-reduce:hover:translate-y-0"
      )}
    >
      {indirim > 0 && (
        <span className="absolute right-3 top-3 rounded-full bg-red-600 px-2 py-0.5 text-xs font-semibold text-white">
          %{indirim}
        </span>
      )}

      <div
        aria-hidden="true"
        className="grid aspect-4/3 place-items-center rounded-lg bg-(--urun-rengi)/20 text-4xl"
      >
        📦
      </div>

      <h3 className="font-semibold">{urun.ad}</h3>

      <div className="flex items-baseline gap-2">
        <span className="text-lg font-bold">{urun.fiyat.toLocaleString("tr-TR")} ₺</span>
        {urun.eskiFiyat && (
          <span className="text-sm text-slate-500 line-through dark:text-slate-400">
            {urun.eskiFiyat.toLocaleString("tr-TR")} ₺
          </span>
        )}
      </div>

      <p
        className={cn(
          "text-sm text-slate-500 dark:text-slate-400",
          azKaldi && "text-amber-600 dark:text-amber-500"
        )}
      >
        {tukendi ? "Tükendi" : azKaldi ? `Son ${urun.stok} ürün!` : "Stokta"}
      </p>

      <button
        type="button"
        disabled={tukendi}
        onClick={() => onEkle(urun)}
        className={cn(
          "mt-auto rounded-lg bg-vurgu px-3 py-2 font-medium text-white",
          "hover:bg-vurgu-hover focus-visible:outline-2 focus-visible:outline-offset-2 focus-visible:outline-vurgu",
          "disabled:cursor-not-allowed disabled:opacity-50"
        )}
      >
        Sepete ekle
      </button>
    </article>
  );
}

export function UrunIzgarasi({ urunler, onEkle }) {
  return (
    <div className="grid grid-cols-[repeat(auto-fill,minmax(220px,1fr))] gap-4">
      {urunler.map((u) => <UrunKarti key={u.id} urun={u} onEkle={onEkle} />)}
    </div>
  );
}
```

### Uygulama

```jsx
// src/App.jsx
import { useState } from "react";
import { URUNLER } from "./urunler";
import { UrunIzgarasi } from "./tailwind/UrunKarti";   // veya "./modules/UrunKarti"
import { useTema } from "./context/TemaContext";       // Konu 13

export default function App() {
  const { tema, temaDegistir } = useTema();
  const [sepet, setSepet] = useState(0);

  return (
    <main style={{ maxWidth: 1000, margin: "0 auto", padding: 24 }}>
      <header style={{ display: "flex", justifyContent: "space-between", alignItems: "center" }}>
        <h1>Mağaza</h1>
        <div>
          🛒 {sepet}{" "}
          <button type="button" onClick={temaDegistir}>{tema === "acik" ? "🌙" : "☀️"}</button>
        </div>
      </header>
      <UrunIzgarasi urunler={URUNLER} onEkle={() => setSepet((s) => s + 1)} />
    </main>
  );
}
```

### Karşılaştırma

| | CSS Modules | Tailwind |
|---|---|---|
| Stil nerede | Ayrı dosyada, anlamlı sınıf adları (`styles.eskiFiyat`) | Doğrudan elemanın üzerinde |
| CSS dosyası | Bileşen başına bir tane | Yok (sadece giriş dosyası) |
| Koşullu sınıf | `clsx` | `cn` (`clsx` + `tailwind-merge`) |
| Ürüne özel renk | `--urun-rengi` + `color-mix()` | `--urun-rengi` + `bg-(--urun-rengi)/20` |

**İkisinde de ortak:**

- Tema renkleri CSS değişkenlerinden gelir; tema değişince hiçbir bileşen render olmaz.
- Responsive ızgara medya sorgusu olmadan `auto-fill` ile kurulur.
- Hover, focus, disabled ve `prefers-reduced-motion` desteklenir.

---

## 14. Özet

- React stil yöntemi dayatmaz; kriterler: **kapsam, dinamik stil, performans, geliştirici deneyimi.**
- **Global CSS** her şeyi global yapar; reset, tipografi ve tema değişkenleri için uygundur.
- **Inline stil** pseudo-class, medya sorgusu ve animasyon desteklemez; yalnızca hesaplanan değerler için.
- **CSS değişkenleri** dinamik değerleri JS'ten CSS'e taşımanın en temiz yoludur (`style={{ "--renk": x }}`); tema sistemlerinin temelidir.
- **CSS Modules** (`.module.css`) sınıf adlarını otomatik benzersiz yapar; sıfır kurulum, çalışma anı maliyeti yok; koşullu sınıflar için `clsx`.
- **Sass** hâlâ kullanılır ama native CSS nesting, custom properties ve `color-mix()` ile ihtiyaç azalıyor.
- **Tailwind CSS v4**: utility sınıfları; `@tailwindcss/vite` + `@import "tailwindcss"` ile kurulum; özelleştirme `@theme` ile CSS içinde.
- Tailwind önekleri: durumlar (`hover:`), responsive (`md:`, mobile-first), `dark:`, `group-hover:`, `motion-reduce:`.
- Tailwind sınıfları **parçalardan birleştirilemez**; tam sınıf adları eşleme nesnesinde tutulur veya CSS değişkeni kullanılır.
- Çakışan Tailwind sınıfları için **`cn` = `clsx` + `tailwind-merge`**; tekrar eden sınıflar **bileşen**e çıkarılır (varyant kalıbı, cva).
- **Runtime CSS-in-JS** (styled-components, Emotion) çalışma anı maliyeti ve RSC uyumsuzluğu nedeniyle yeni projelerde önerilmez; zero-runtime alternatifler: vanilla-extract, Panda CSS, StyleX.
- **Bileşen kütüphaneleri:** stilli (MUI, Mantine), headless (Radix, React Aria, Headless UI, Base UI), kod kopyalama (**shadcn/ui**).
- **Koyu mod**: CSS değişkenleri + `<html data-tema>` → bileşenler render olmadan tema değişir; Tailwind'de `@custom-variant dark`.
- Animasyonlar: CSS geçişleri; çıkış animasyonları için **Motion**; `prefers-reduced-motion`'a saygı.
- Projede **tek bir ana stil yöntemi** seçilir.

---

⬅️ Önceki: [15 · Custom Hook'lar](./15-custom-hooks.md) · ➡️ Sonraki: [16a · Tailwind CSS (Detaylı)](./16a-tailwind-css.md)
