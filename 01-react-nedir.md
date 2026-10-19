# 01 · React Nedir ve Neden Var?

> **Konu özeti:** React'in ne olduğu, hangi problemi çözdüğü, temel fikirleri, render süreci ve Vite ile proje kurulumu.
> **React sürümü:** 19.2.x

---

## İçindekiler

1. [Tanım](#1-tanım)
2. [React Hangi Problemi Çözüyor?](#2-react-hangi-problemi-çözüyor)
3. [Üç Temel Fikir](#3-üç-temel-fikir)
4. [Virtual DOM ve Render Süreci](#4-virtual-dom-ve-render-süreci)
5. [Ortam Kurulumu](#5-ortam-kurulumu)
6. [Proje Yapısı](#6-proje-yapısı)
7. [Ön Koşul: React İçin JavaScript](#7-ön-koşul-react-için-javascript)
8. [Özet](#10-özet)

---

## 1. Tanım

React, kullanıcı arayüzleri (UI) oluşturmak için kullanılan bir **JavaScript kütüphanesidir**.

- Meta (Facebook) tarafından geliştirildi, 2013'te açık kaynak oldu.
- Yalnızca **görünüm (view) katmanıyla** ilgilenir.
- Routing, veri çekme, form yönetimi gibi işler için ek kütüphaneler ya da React üzerine kurulu framework'ler (ör. Next.js) kullanılır.

---

## 2. React Hangi Problemi Çözüyor?

### İmperatif yaklaşım (saf JavaScript)

```html
<p id="sayac">0</p>
<button id="btn">Artır</button>

<script>
  let sayi = 0;
  const p = document.getElementById("sayac");
  document.getElementById("btn").addEventListener("click", () => {
    sayi++;
    p.textContent = sayi; // DOM'u elle güncelliyoruz
  });
</script>
```

Hem **veriyi** hem de **ekranı** sen yönetiyorsun. Uygulama büyüdükçe "veri değişti, ekranda hangi yerleri güncellemem gerekiyor?" sorusu yönetilemez hale gelir.

### Deklaratif yaklaşım (React)

```jsx
import { useState } from "react";

function Sayac() {
  const [sayi, setSayi] = useState(0);

  return (
    <>
      <p>{sayi}</p>
      <button onClick={() => setSayi(sayi + 1)}>Artır</button>
    </>
  );
}
```

Sadece ekranın veriye göre **nasıl görünmesi gerektiğini** tarif ediyorsun. Veri değişince React ekranı kendisi günceller.

| İmperatif | Deklaratif |
|---|---|
| *Nasıl* yapılacağını adım adım söylersin | *Ne* istediğini söylersin |
| DOM'u elle değiştirirsin | DOM'u React değiştirir |
| Veri ile ekranı senkron tutmak senin işin | Senkronizasyon React'in işi |

> 💡 **Temel formül:** `UI = f(state)` — arayüz, verinin bir fonksiyonudur.

---

## 3. Üç Temel Fikir

### a) Bileşen (Component) tabanlı yapı
Arayüz küçük, bağımsız ve tekrar kullanılabilir parçalara bölünür: `Navbar`, `Kart`, `Buton`... Her bileşen kendi görünümünü ve mantığını taşır; LEGO parçaları gibi birleştirilir.

```
App
├── Navbar
├── UrunListesi
│   ├── UrunKarti
│   ├── UrunKarti
│   └── UrunKarti
└── Footer
```

### b) Deklaratif yaklaşım
Yukarıda anlatıldı: ne görmek istediğini tarif edersin, güncellemeyi React yapar.

### c) Tek yönlü veri akışı
Veri **yukarıdan aşağıya** (ebeveyn → çocuk) akar. Verinin nereden geldiği her zaman bellidir, bu da hata ayıklamayı kolaylaştırır. *(Props konusunda detaylanacak.)*

---

## 4. Virtual DOM ve Render Süreci

Gerçek DOM'u değiştirmek görece maliyetlidir. React'in güncelleme döngüsü:

1. **Tetikleme:** State değişir.
2. **Render:** İlgili bileşen fonksiyonu yeniden çalışır ve yeni bir **sanal ağaç** (Virtual DOM — sıradan JS nesneleri) üretir.
3. **Reconciliation (uzlaştırma / diffing):** React eski ağaçla yenisini karşılaştırır.
4. **Commit:** Yalnızca **değişen kısımlar** gerçek DOM'a uygulanır.

```
state değişti → render (yeni sanal ağaç) → karşılaştır (diff) → gerçek DOM'a sadece farkı uygula
```

> ⚠️ **Yaygın efsane:** "Virtual DOM her zaman daha hızlıdır." Değil. Asıl kazanç, DOM'u elle yönetme yükünden kurtulman ve bunun makul bir performansla yapılmasıdır.

---

## 5. Ortam Kurulumu

### Gereksinim
[Node.js](https://nodejs.org) (LTS sürümü önerilir).

```bash
node -v
npm -v
```

### Vite ile proje oluşturma

```bash
npm create vite@latest ilk-react-app -- --template react
cd ilk-react-app
npm install
npm run dev
```

Uygulama `http://localhost:5173` adresinde açılır.

> ⚠️ **`create-react-app` kullanma.** Eski eğitimlerde geçer ama resmi olarak kullanımdan kaldırıldı. Yeni projeler için **Vite** gibi bir build aracı ya da **Next.js** gibi bir framework önerilir.

### Faydalı npm komutları

| Komut | Görevi |
|---|---|
| `npm run dev` | Geliştirme sunucusunu başlatır (HMR ile) |
| `npm run build` | Yayına hazır, optimize edilmiş çıktıyı `dist/` klasörüne üretir |
| `npm run preview` | Build çıktısını yerelde önizler |

---

## 6. Proje Yapısı

```
ilk-react-app/
├── index.html        ← tek HTML sayfası, içinde <div id="root">
├── package.json      ← bağımlılıklar ve scriptler
├── vite.config.js    ← Vite ayarları
└── src/
    ├── main.jsx      ← giriş noktası: React'i #root'a bağlar
    └── App.jsx       ← ana (kök) bileşen
```

### `main.jsx`

```jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App.jsx";

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <App />
  </StrictMode>
);
```

- `createRoot(...)`: `#root` elementini React'in yöneteceği kök olarak işaretler.
- `.render(<App />)`: Kök bileşeni bu alana çizer.
- Tüm uygulama bu tek noktadan büyür → **SPA (Single Page Application)**.

### `StrictMode`
- Yalnızca **geliştirme ortamında** çalışır, production'a etkisi yoktur.
- Olası hataları yakalamak için bileşenleri **bilerek iki kez** render eder.
- Konsolda logların iki kez görünmesinin sebebi budur.

### `react` ve `react-dom` farkı
- `react`: Bileşen, Hook, JSX gibi çekirdek mantık (platformdan bağımsız).
- `react-dom`: Bu mantığı **tarayıcı DOM'una** bağlayan paket. (Mobil için React Native ayrı bir "renderer" kullanır.)

---

## 7. Ön Koşul: React İçin JavaScript

React'te sık kullanılan JS özellikleri:

- `let` / `const`
- Arrow function
- Template literal
- Destructuring (nesne ve dizi)
- Spread / rest (`...`)
- Dizi metotları: `map`, `filter`, `find`, `reduce`
- `import` / `export` (ES modülleri)
- Ternary (`? :`), `&&`, `??`, `?.`
- Promise ve `async` / `await`

> Bu konular ayrı bir notta işlenecek: `00-react-icin-javascript.md`


---

## 8. Özet

- React, UI oluşturmak için bir **kütüphanedir**; sadece görünüm katmanıyla ilgilenir.
- **Deklaratif:** Ne görmek istediğini tarif edersin → `UI = f(state)`.
- **Bileşen tabanlı:** Arayüz küçük, tekrar kullanılabilir parçalara bölünür.
- **Tek yönlü veri akışı:** Veri ebeveynden çocuğa akar.
- **Güncelleme döngüsü:** tetikleme → render → reconciliation → commit.
- Yeni projeler **Vite** (veya Next.js) ile başlatılır; `create-react-app` kullanılmaz.
- `main.jsx` giriş noktasıdır; `createRoot` ile uygulamayı `#root`'a bağlar.
- `StrictMode` sadece geliştirmede çalışır ve bileşenleri bilerek iki kez render eder.

---

⬅️ Önceki: [00 · React İçin JavaScript](./00-react-icin-javascript.md) · ➡️ Sonraki: [02 · JSX](./02-jsx.md)
