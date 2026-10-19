# 03 · Bileşenler (Components)

> **Konu özeti:** Bileşenin tanımı, yazım kuralları, kompozisyon ve bileşen ağacı, dosyalara ayırma, saflık (purity), render zamanları, arayüzü bileşenlere bölme, `children`'a giriş, class bileşenleri ve React DevTools.

---

## İçindekiler

1. [Bileşen Nedir?](#1-bileşen-nedir)
2. [Bileşen Yazma Kuralları](#2-bileşen-yazma-kuralları)
3. [Kompozisyon ve Bileşen Ağacı](#3-kompozisyon-ve-bileşen-ağacı)
4. [Bileşeni Bileşenin İçinde Tanımlama](#4-️-bileşeni-bileşenin-içinde-tanımlama)
5. [Bileşenleri Ayrı Dosyalara Taşımak](#5-bileşenleri-ayrı-dosyalara-taşımak)
6. [Bileşenler Saf (Pure) Olmalı](#6-bileşenler-saf-pure-olmalı-)
7. [Bileşen Ne Zaman Render Olur?](#7-bileşen-ne-zaman-render-olur)
8. [Arayüzü Bileşenlere Bölmek](#8-arayüzü-bileşenlere-bölmek)
9. [`children`'a Kısa Bir Bakış](#9-childrena-kısa-bir-bakış)
10. [Class Bileşenleri (Tarihçe)](#10-class-bileşenleri-tarihçe)
11. [React Developer Tools](#11-react-developer-tools)
12. [Her Şey Bir Arada](#12-her-şey-bir-arada)
13. [Özet](#13-özet)

---

## 1. Bileşen Nedir?

Bileşen, arayüzün **kendi görünümünü ve mantığını taşıyan, tekrar kullanılabilir bir parçasıdır.**

> **React bileşeni = JSX döndüren, adı büyük harfle başlayan bir JavaScript fonksiyonu.**

```jsx
function Selam() {
  return <h1>Merhaba!</h1>;
}

function App() {
  return (
    <div>
      <Selam />
      <Selam />
      <Selam />
    </div>
  );
}
```

- Bir kez tanımlanır, istenildiği kadar kullanılır.
- `<Selam />` derlenince `_jsx(Selam, {})` olur; React fonksiyonu **kendisi çağırır** ve dönen JSX'i ağaca yerleştirir.

> ⚠️ Bileşen asla `Selam()` şeklinde elle çağrılmaz, her zaman `<Selam />` yazılır. Elle çağrıldığında React onu bileşen olarak tanımaz; Hook'lar ve state düzgün çalışmaz.

---

## 2. Bileşen Yazma Kuralları

### 1. Adı büyük harfle başlar (PascalCase)

`UrunKarti`, `Navbar`, `KullaniciProfili`. Küçük harfle başlayan etiketler HTML etiketi sanılır.

### 2. Bir şey döndürür

JSX, string, sayı, dizi veya hiçbir şey göstermemek için `null`.

```jsx
function Uyari({ goster }) {
  if (!goster) return null; // hiçbir şey render etme
  return <p className="uyari">Dikkat!</p>;
}
```

> `undefined` döndürmek React 18'den beri hata vermez, ama niyeti açık belirtmek için `null` tercih edilir.

### 3. Tanımlama biçimi fark etmez

```jsx
function Kart() { return <div>Kart</div>; }   // function declaration
const Kart = () => <div>Kart</div>;            // arrow function
```

- İkisi de geçerlidir.
- Bileşenlerde `function` daha yaygındır (React dokümantasyonu da bunu kullanır).
- Proje içinde tutarlı olunmalı.

---

## 3. Kompozisyon ve Bileşen Ağacı

Küçük bileşenler birleştirilerek büyük bileşenler, onlar birleştirilerek de uygulamanın tamamı oluşturulur.

```jsx
function Logo() {
  return <span className="logo">⚛️ ReactShop</span>;
}

function Menu() {
  return (
    <nav>
      <a href="/">Ana Sayfa</a>
      <a href="/urunler">Ürünler</a>
      <a href="/sepet">Sepet</a>
    </nav>
  );
}

function Header() {
  return (
    <header>
      <Logo />
      <Menu />
    </header>
  );
}

function Footer() {
  return <footer>© 2026 ReactShop</footer>;
}

function App() {
  return (
    <>
      <Header />
      <main>İçerik burada</main>
      <Footer />
    </>
  );
}
```

### Oluşan bileşen ağacı

```
App
├── Header
│   ├── Logo
│   └── Menu
├── (main)
└── Footer
```

| Terim | Anlamı |
|---|---|
| **Ebeveyn (parent)** | Başka bir bileşeni render eden bileşen (`Header`, `Logo`'nun ebeveyni) |
| **Çocuk (child)** | Render edilen bileşen |
| **Kök bileşen** | Ağacın tepesindeki bileşen (`App`); `createRoot(...).render(<App />)` ile çizilir |

> Bileşen ağacı; veri akışını (props), state'in nerede tutulacağını ve performansı anlamanın temelidir.

---

## 4. ⚠️ Bileşeni Bileşenin İçinde Tanımlama

```jsx
// ❌ YANLIŞ
function UrunListesi() {
  function UrunKarti() {        // her render'da YENİ bir fonksiyon oluşur
    return <div>Ürün</div>;
  }
  return <UrunKarti />;
}
```

```jsx
// ✅ DOĞRU: en üst seviyede tanımla
function UrunKarti() {
  return <div>Ürün</div>;
}

function UrunListesi() {
  return <UrunKarti />;
}
```

**Neden yanlış?**
- Ebeveyn her render olduğunda iç bileşen **yeni bir fonksiyon** olarak yeniden oluşturulur.
- React bunu "tamamen farklı bir bileşen" olarak yorumlar ve sıfırdan kurar.
- Sonuç: **state silinir**, input'taki yazı kaybolur, performans düşer.

> **Kural:** Bileşenler her zaman dosyanın en üst seviyesinde tanımlanır. Ebeveynden veri gerekiyorsa props ile aktarılır.

---

## 5. Bileşenleri Ayrı Dosyalara Taşımak

```jsx
// src/components/Header.jsx
import Logo from "./Logo";
import Menu from "./Menu";

function Header() {
  return (
    <header>
      <Logo />
      <Menu />
    </header>
  );
}

export default Header;
```

```jsx
// src/App.jsx
import Header from "./components/Header";
import Footer from "./components/Footer";

function App() {
  return (
    <>
      <Header />
      <main>İçerik</main>
      <Footer />
    </>
  );
}

export default App;
```

### Default export mu, named export mu?

```jsx
// Default export
export default function Header() { /* ... */ }
import Header from "./Header";                    // istenen isimle import edilebilir

// Named export
export function Button() { /* ... */ }
export function IconButton() { /* ... */ }
import { Button, IconButton } from "./Buttons";   // isim birebir aynı olmalı
```

| Durum | Yaygın tercih |
|---|---|
| Dosya başına bir ana bileşen | Default export |
| Birbiriyle ilgili küçük bileşenler tek dosyada | Named export |

> Bazı ekipler yeniden adlandırma karmaşasını önlemek için her yerde named export kullanır. İkisi de geçerlidir; önemli olan tutarlılıktır.

### İsimlendirme alışkanlıkları

- Dosya adı = bileşen adı: `UrunKarti.jsx` → `UrunKarti`
- JSX içeren dosyalar `.jsx` uzantılıdır (Vite, `.js` dosyasında JSX'i varsayılan olarak derlemez).
- Import'ta uzantı yazmak opsiyoneldir: `import Header from "./Header"`

### Örnek klasör yapısı (küçük–orta projeler)

```
src/
├── components/          ← tekrar kullanılan, genel bileşenler
│   ├── Button.jsx
│   ├── Header.jsx
│   └── Footer.jsx
├── pages/               ← sayfa düzeyindeki bileşenler
│   ├── HomePage.jsx
│   └── CartPage.jsx
├── App.jsx
└── main.jsx
```

### Bileşen başına klasör (büyüyen projeler)

```
components/
└── UrunKarti/
    ├── UrunKarti.jsx
    ├── UrunKarti.module.css
    └── index.js         ← export { default } from "./UrunKarti";
```

> React bir klasör yapısı dayatmaz. Başta basit tutulur, ihtiyaç doğdukça düzenlenir.

---

## 6. Bileşenler Saf (Pure) Olmalı ⭐

### Saf fonksiyon nedir?

Matematikteki `y = 2x` gibi:

1. **Aynı girdiye her zaman aynı çıktıyı verir.**
2. **Kendi dışındaki hiçbir şeyi değiştirmez** (yan etki yoktur).

React, bileşenlerin render sırasında böyle davranacağını varsayar:

```
girdiler (props, state, context)  →  bileşen  →  çıktı (JSX)
```

### Saf olmayan bileşen

```jsx
let misafirSayisi = 0;           // bileşenin DIŞINDA bir değişken

function Bardak() {
  misafirSayisi = misafirSayisi + 1;   // ❌ render sırasında dışarıyı değiştiriyor
  return <h2>{misafirSayisi} numaralı misafirin çay bardağı</h2>;
}

function App() {
  return (
    <>
      <Bardak />
      <Bardak />
      <Bardak />
    </>
  );
}
```

- Beklenen: **1, 2, 3**
- Geliştirme ortamında görülen: **2, 4, 6**
- Sebep: `StrictMode` bileşenleri **bilerek iki kez** çalıştırır. Amacı saf olmayan bileşenleri görünür kılmaktır.
- Production'da da React'in render'ı ne zaman ve kaç kez çalıştıracağının garantisi yoktur.

### Saf hali

```jsx
function Bardak({ misafir }) {
  return <h2>{misafir} numaralı misafirin çay bardağı</h2>;
}

function App() {
  return (
    <>
      <Bardak misafir={1} />
      <Bardak misafir={2} />
      <Bardak misafir={3} />
    </>
  );
}
```

### Render sırasında yapılmaması gerekenler

- Bileşen dışındaki değişkenleri veya nesneleri değiştirmek
- DOM'a doğrudan dokunmak (`document.title = ...`, `document.getElementById(...)`)
- API isteği atmak, `setTimeout` kurmak, `localStorage`'a yazmak
- Props'u değiştirmek (`props.ad = "x"` ❌)

### Yan etkiler (side effects) nerede yapılır?

| Yer | Ne zaman çalışır | Konu |
|---|---|---|
| **Olay yöneticileri** (tıklama, form gönderme) | Kullanıcı bir şey yaptığında; saf olmaları gerekmez | Konu 6 |
| **`useEffect`** | Render tamamlandıktan sonra; son çare | Konu 10 |

### Yerel değişiklik serbesttir

Render sırasında, **o render'da oluşturulan** değişkenler değiştirilebilir:

```jsx
function Liste() {
  const elemanlar = [];                         // bu render'da oluşturuldu
  for (let i = 1; i <= 3; i++) {
    elemanlar.push(<li key={i}>Madde {i}</li>); // ✅ yerel değişiklik
  }
  return <ul>{elemanlar}</ul>;
}
```

### Saflık neden önemli?

React şu özelliklerini bu varsayıma dayandırır:

- Render'ı yarıda kesip yeniden başlatabilmek
- Gereksiz render'ları atlayabilmek (`memo`, React Compiler)
- Sunucuda render edebilmek

Saf bileşenler **öngörülebilir, test edilebilir ve hatasızdır.**

---

## 7. Bileşen Ne Zaman Render Olur?

Bir bileşen şu durumlarda yeniden çalışır:

1. **İlk yükleme:** `createRoot(...).render(<App />)` ile.
2. **Kendi state'i değiştiğinde.**
3. **Ebeveyni render olduğunda:** Varsayılan olarak ebeveyn render olunca tüm çocukları da render olur.

> Render ≠ DOM güncellemesi. Bileşen fonksiyonu çalışır, ama DOM'a yalnızca fark uygulanır. Saf bileşenlerde bu ucuzdur; gerektiğinde optimizasyon yolları vardır (Konu 19).

---

## 8. Arayüzü Bileşenlere Bölmek

### Pratik kurallar

- **Tek sorumluluk ilkesi:** Bir bileşen ideal olarak tek bir iş yapar. Birden fazla iş yapıyorsa parçalanır.
- **Tekrar eden şey bileşendir:** Ürün kartı, yorum, liste satırı gibi tekrar eden yapılar bileşen adayıdır.
- **Görsel bloklara bak:** Tasarımın etrafına kutular çiz; her kutu bir bileşen, iç içe kutular iç içe bileşenler.
- **Aşırıya kaçma:** Her `<span>` için bileşen açmak okunabilirliği düşürür. Ölçüt: *"Bunu ayırınca kod daha anlaşılır mı oluyor?"*

### Örnek: e-ticaret ürün sayfası

```
UrunSayfasi
├── Header
├── UrunDetay
│   ├── UrunGaleri
│   ├── UrunBilgi
│   │   ├── Fiyat
│   │   └── SepeteEkleButonu
│   └── Yorumlar
│       └── YorumKarti (×N)
└── Footer
```

---

## 9. `children`'a Kısa Bir Bakış

Açılış ve kapanış etiketi arasına yazılan her şey, `children` adlı özel bir prop olarak bileşene gelir.

```jsx
function Kart({ children }) {
  return <div className="kart">{children}</div>;
}

function App() {
  return (
    <Kart>
      <h2>Başlık</h2>
      <p>Bu içerik Kart'ın içine yerleşir.</p>
    </Kart>
  );
}
```

- `Kart`, içinde ne olduğunu bilmeden bir "çerçeve" sunar.
- Layout, modal, panel gibi sarmalayıcı bileşenler bu şekilde yapılır.
- *(Detayı Konu 4: Props'ta.)*

---

## 10. Class Bileşenleri (Tarihçe)

```jsx
import { Component } from "react";

class Selam extends Component {
  render() {
    return <h1>Merhaba, {this.props.ad}</h1>;
  }
}
```

- 2019'da Hook'lar gelmeden önce state ve yaşam döngüsü (lifecycle) yalnızca class bileşenlerde kullanılabiliyordu.
- Bugün **yeni kod fonksiyon bileşenleriyle yazılır**; React dokümantasyonu class bileşenleri "eski API" olarak listeler.
- Bilinmesi gereken: Eski kodlarda karşılaşıldığında tanıyabilmek.
- **İstisna:** Error Boundary'ler (render hatalarını yakalayan bileşenler) hâlâ class ile yazılır veya `react-error-boundary` gibi bir kütüphaneyle kullanılır.

---

## 11. React Developer Tools

Tarayıcı eklentisi (Chrome / Firefox / Edge). Geliştirici araçlarına iki sekme ekler:

| Sekme | İşlevi |
|---|---|
| **Components** | Bileşen ağacını, her bileşenin props'unu, state'ini ve Hook'larını gösterir; değerler canlı değiştirilebilir |
| **Profiler** | Hangi bileşenin ne zaman, neden ve ne kadar sürede render olduğunu ölçer |

> 💡 Ayarlardan **"Highlight updates when components render"** açılırsa render olan bileşenler ekranda yanıp söner. "Ebeveyn render olunca çocuklar da render olur" kuralı gözle görülebilir.

---

## 12. Her Şey Bir Arada

```jsx
// src/components/Avatar.jsx
export default function Avatar() {
  return (
    <img
      className="avatar"
      src="https://i.pravatar.cc/80"
      alt="Kullanıcı fotoğrafı"
    />
  );
}
```

```jsx
// src/components/ProfilKarti.jsx
import Avatar from "./Avatar";

function Rozet() {               // sadece bu dosyada kullanılan küçük bileşen
  return <span className="rozet">Pro Üye</span>;
}

export default function ProfilKarti() {
  return (
    <div className="profil-karti">
      <Avatar />
      <div>
        <h3>Ayşe Yılmaz</h3>
        <Rozet />
      </div>
    </div>
  );
}
```

```jsx
// src/App.jsx
import ProfilKarti from "./components/ProfilKarti";

function App() {
  return (
    <main>
      <h1>Ekibimiz</h1>
      <ProfilKarti />
      <ProfilKarti />
      <ProfilKarti />
    </main>
  );
}

export default App;
```

> Üç kart da **aynı kişiyi** gösteriyor: bileşen tekrar kullanılabilir ama içindeki veri sabit. Her kartın farklı veri göstermesi için bileşene dışarıdan veri verilmesi gerekir → **Props** (Konu 4).

---

## 13. Özet

- Bileşen = JSX döndüren, adı **büyük harfle** başlayan bir fonksiyon.
- Bileşenler `<Bilesen />` şeklinde kullanılır, asla `Bilesen()` diye elle çağrılmaz.
- Hiçbir şey göstermemek için `null` döndürülür.
- Küçük bileşenler birleştirilerek **bileşen ağacı** oluşturulur (kompozisyon).
- Bileşenler **asla başka bir bileşenin içinde tanımlanmaz**; aksi halde state her render'da sıfırlanır.
- Genelde her bileşen kendi `.jsx` dosyasındadır; default veya named export tutarlı kullanılır.
- Bileşenler **saf** olmalı: aynı girdiye aynı çıktı, render sırasında yan etki yok.
- Yan etkiler **olay yöneticilerinde** veya son çare olarak **`useEffect`**'te yapılır.
- `StrictMode`, saf olmayan bileşenleri yakalamak için render'ı iki kez çalıştırır.
- Bileşen; ilk yüklemede, kendi state'i değişince ve ebeveyni render olunca yeniden render olur.
- Arayüz; tek sorumluluk, tekrar ve görsel bloklar gözetilerek bileşenlere bölünür.
- `children`, etiketler arasına yazılan içeriği taşıyan özel prop'tur.
- Class bileşenleri eski API'dir; yeni kod fonksiyon bileşenleriyle yazılır.
- React DevTools ile bileşen ağacı, props, state ve render'lar incelenir.

---

⬅️ Önceki: [02 · JSX](./02-jsx.md) · ➡️ Sonraki: [04 · Props](./04-props.md)
