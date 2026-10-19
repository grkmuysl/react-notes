# 02 · JSX

> **Konu özeti:** JSX'in ne olduğu, neye dönüştüğü, yazım kuralları, `{}` ile JavaScript kullanımı, render edilen değerler, stil, güvenlik ve diğer ayrıntılar.

---

## İçindekiler

1. [JSX Nedir?](#1-jsx-nedir)
2. [JSX Neye Dönüşür?](#2-jsx-neye-dönüşür)
3. [JSX'in Kuralları](#3-jsxin-kuralları)
4. [Süslü Parantez `{ }`](#4-süslü-parantez---jsx-içinde-javascript)
5. [Hangi Değerler Ekrana Basılır?](#5-hangi-değerler-ekrana-basılır)
6. [Inline Stil](#6-inline-stil-style-nesne-alır)
7. [Yorum Satırları](#7-yorum-satırları)
8. [Küçük Harf mi Büyük Harf mi?](#8-küçük-harf-mi-büyük-harf-mi)
9. [Güvenlik: Otomatik Kaçış](#9-güvenlik-otomatik-kaçış-escaping)
10. [Diğer Ayrıntılar](#10-diğer-ayrıntılar)
11. [Her Şey Bir Arada](#11-her-şey-bir-arada)
12. [Özet](#13-özet)

---

## 1. JSX Nedir?

JSX (**J**avaScript **X**ML), JavaScript içinde HTML'e benzeyen bir sözdizimiyle arayüz tarif etmeyi sağlayan bir **sözdizimi uzantısıdır**.

```jsx
const baslik = <h1 className="baslik">Merhaba React</h1>;
```

- JSX ne string ne de HTML'dir.
- Tarayıcı JSX'i doğrudan anlamaz; build aracı (Vite) onu sıradan JavaScript'e derler.
- JSX zorunlu değildir ama ağaç yapısını okunabilir kıldığı için standart hale gelmiştir.

---

## 2. JSX Neye Dönüşür?

```jsx
// Yazdığımız
const baslik = <h1 className="baslik">Merhaba React</h1>;
```

```js
// Derlenmiş hali (kabaca)
import { jsx as _jsx } from "react/jsx-runtime";

const baslik = _jsx("h1", { className: "baslik", children: "Merhaba React" });
```

Bu çağrı bir **React elementi** döndürür; ekranda ne olması gerektiğini tarif eden sade bir JS nesnesi:

```js
{
  type: "h1",
  props: { className: "baslik", children: "Merhaba React" },
  // ...React'in iç alanları
}
```

### Çıkarımlar

- **JSX = fonksiyon çağrısı.** Değişkene atanabilir, fonksiyondan döndürülebilir, dizide tutulabilir.
- **Element ≠ DOM düğümü.** Element hafif bir tarif nesnesidir; sanal ağaç bu nesnelerden oluşur.
- Yeni JSX dönüşümüyle her dosyaya `import React from "react"` yazmak **artık gerekmez.** (Eski sistemde JSX `React.createElement(...)`'e dönüştüğü için gerekiyordu.)

---

## 3. JSX'in Kuralları

### Kural 1: Tek bir kök element döndür

```jsx
// ❌ Hata
function Profil() {
  return (
    <h1>Ayşe</h1>
    <p>Geliştirici</p>
  );
}
```

Bir fonksiyon iki değer döndüremez. Çözüm: **Fragment** ile sar.

```jsx
// ✅
function Profil() {
  return (
    <>
      <h1>Ayşe</h1>
      <p>Geliştirici</p>
    </>
  );
}
```

- `<>...</>` DOM'a fazladan element eklemez.
- `key` vermek gerektiğinde uzun hali kullanılır:

```jsx
import { Fragment } from "react";

<Fragment key={id}>...</Fragment>
```

> Çok satırlı JSX `return`'den sonra **parantezle** sarılır. Aksi halde JavaScript satır sonuna otomatik noktalı virgül ekler ve fonksiyon `undefined` döndürür.

### Kural 2: Tüm etiketler kapatılmalı

```jsx
<img src="logo.png" alt="Logo" />
<br />
<input type="text" />
<div />   // içi boşsa her etiket kendini kapatabilir
```

### Kural 3: Nitelikler camelCase yazılır

| HTML | JSX | Neden |
|---|---|---|
| `class` | `className` | `class` JS'te ayrılmış kelime |
| `for` | `htmlFor` | `for` JS'te ayrılmış kelime |
| `onclick` | `onClick` | camelCase |
| `tabindex` | `tabIndex` | camelCase |
| `maxlength` | `maxLength` | camelCase |
| `stroke-width` (SVG) | `strokeWidth` | Tire, JS anahtarında sorun çıkarır |

**İstisna:** `data-*` ve `aria-*` nitelikleri tireli kalır.

```jsx
<button data-id="42" aria-label="Kapat">×</button>
```

---

## 4. Süslü Parantez `{ }` — JSX İçinde JavaScript

`{ }`, JSX'ten JavaScript'e açılan penceredir.

```jsx
const ad = "Ayşe";
const yas = 25;
const avatar = "https://i.pravatar.cc/100";

function Kart() {
  return (
    <div>
      <h2>{ad.toUpperCase()}</h2>
      <p>Yaş: {yas}, gelecek yıl: {yas + 1}</p>
      <img src={avatar} alt={`${ad} profil fotoğrafı`} />
    </div>
  );
}
```

### Kullanılabildiği iki yer

1. **Etiketler arasında içerik olarak:** `<h2>{ad}</h2>`
2. **Nitelik değeri olarak:** `src={avatar}`

```jsx
<img src="logo.png" />     // sabit string → tırnak
<img src={logoUrl} />       // dinamik değer → süslü parantez
<img src="{logoUrl}" />     // ❌ düz "{logoUrl}" yazısı gider
```

### İfade evet, deyim hayır

`{ }` içine yalnızca **değer üreten ifadeler** yazılabilir, çünkü içerik bir nesne özelliğinin değerine dönüşür.

| ✅ İfade | ❌ Deyim |
|---|---|
| `{ad}` | `{if (x) {...}}` |
| `{a + b}` | `{for (...) {...}}` |
| `{x ? "A" : "B"}` | `{const y = 5}` |
| `{giris && <Panel />}` | `{switch (...) {...}}` |
| `{liste.map(...)}` | |
| `{fonk()}` | |

`if` veya döngü gerekiyorsa JSX'in **dışında**, `return`'den önce kullanılır:

```jsx
function Mesaj({ girisYapti }) {
  let icerik;
  if (girisYapti) {
    icerik = <p>Hoş geldin!</p>;
  } else {
    icerik = <p>Lütfen giriş yap.</p>;
  }
  return <div>{icerik}</div>;
}
```

---

## 5. Hangi Değerler Ekrana Basılır?

| Değer | Ekranda |
|---|---|
| `"metin"` | metin |
| `42` | 42 |
| **`0`** | **0** ⚠️ |
| `NaN` | NaN |
| `true` / `false` | hiçbir şey |
| `null` / `undefined` | hiçbir şey |
| Dizi (`[<li/>, <li/>]`) | elemanlar sırayla |
| JSX elementi | element |
| **Düz nesne (`{ ad: "Ayşe" }`)** | **❌ Hata** |

```jsx
const kullanici = { ad: "Ayşe" };

<p>{kullanici}</p>     // ❌ "Objects are not valid as a React child"
<p>{kullanici.ad}</p>  // ✅
```

- Boolean, `null`, `undefined` görünmez → `&&` ile koşullu render bu sayede çalışır.
- `0` görünür → `{liste.length && ...}` tuzağının sebebi. Doğrusu: `{liste.length > 0 && ...}`
- Diziler render edilebilir → `map` ile liste oluşturmanın temeli:

```jsx
const meyveler = ["Elma", "Armut", "Kiraz"];

<ul>
  {meyveler.map(m => <li key={m}>{m}</li>)}
</ul>
```

> `key`: listedeki her elemana verilen benzersiz kimlik. *(Konu 8'de detaylanacak.)*

---

## 6. Inline Stil: `style` Nesne Alır

```jsx
// HTML: <p style="color: red; font-size: 20px;">
<p style={{ color: "red", fontSize: 20, backgroundColor: "#eee" }}>Uyarı</p>
```

- **Çift süslü parantez** özel bir sözdizimi değildir: dıştaki `{}` "JavaScript'e geç", içteki `{}` nesnenin kendisi.
- CSS özellik adları **camelCase**: `font-size` → `fontSize`.
- Sayısal değerlere otomatik `px` eklenir: `fontSize: 20` → `20px`.
- Başka birimler string ile: `width: "50%"`, `margin: "1rem"`.

```jsx
const kartStili = { padding: 16, borderRadius: 8 };

<div style={kartStili}>...</div>
```

### CSS dosyası kullanımı

```jsx
import "./App.css";

<div className="kart">...</div>
```

> Inline stil küçük, dinamik değerler için uygundur. Büyük projelerde CSS dosyaları, CSS Modules veya Tailwind tercih edilir. *(Konu 16)*

---

## 7. Yorum Satırları

```jsx
return (
  <div>
    {/* JSX içinde yorum böyle yazılır */}
    <h1>Başlık</h1>
  </div>
);
```

`<!-- -->` (HTML yorumu) JSX'te çalışmaz.

---

## 8. Küçük Harf mi Büyük Harf mi?

```jsx
<div />    // küçük harf → HTML etiketi  → _jsx("div", ...)   (string)
<Kart />   // büyük harf → React bileşeni → _jsx(Kart, ...)   (fonksiyon referansı)
```

- **Bileşen adları her zaman büyük harfle başlar.**
- `<kart />` yazılırsa React bunu bilinmeyen bir HTML etiketi sanır; bileşen fonksiyonu hiç çalışmaz.
- Nokta notasyonu kullanılabilir: `<Form.Input />`

---

## 9. Güvenlik: Otomatik Kaçış (Escaping)

```jsx
const yorum = '<img src=x onerror="alert(\'hack\')" />';

<p>{yorum}</p>  // Metin olarak görünür, ÇALIŞTIRILMAZ
```

React, `{}` içindeki string'leri HTML olarak yorumlamaz. Bu, **XSS (Cross-Site Scripting)** saldırılarına karşı varsayılan korumadır.

### Bilinçli HTML basmak

```jsx
<div dangerouslySetInnerHTML={{ __html: guvenliHtml }} />
```

> ⚠️ Kullanıcı girdisini **asla** temizlemeden (ör. DOMPurify ile) buraya verme. İsimdeki "dangerously" bilinçli bir uyarıdır.

---

## 10. Diğer Ayrıntılar

### Boolean nitelikler

```jsx
<input disabled />           // disabled={true}
<input disabled={false} />   // nitelik hiç eklenmez
```

### Prop'ları spread ile aktarma

```jsx
const inputAyar = { type: "email", placeholder: "E-posta", required: true };

<input {...inputAyar} />
```

### Boşluk (whitespace) davranışı

JSX, satır sonlarındaki boşlukları ve girintileri siler.

```jsx
<p>
  Merhaba
  {ad}
</p>
// Ekranda: "MerhabaAyşe" — boşluk yok!

<p>Merhaba {ad}</p>        // ✅ aynı satırda boşluk korunur
<p>Merhaba{" "}{ad}</p>    // ✅ açık boşluk
```

### JSX bir değerdir

```jsx
const ikon = <span>⭐</span>;
const etiket = (metin) => <strong>{metin}</strong>;

<p>{ikon} {etiket("Favori")}</p>
```

---

## 11. Her Şey Bir Arada

```jsx
import "./App.css";

const kullanici = {
  ad: "Ayşe Yılmaz",
  unvan: "Frontend Geliştirici",
  avatar: "https://i.pravatar.cc/120",
  yetenekler: ["React", "TypeScript", "CSS"],
  aktif: true,
};

function App() {
  const yetenekSayisi = kullanici.yetenekler.length;

  return (
    <div className="kart">
      {/* Profil başlığı */}
      <img
        src={kullanici.avatar}
        alt={`${kullanici.ad} fotoğrafı`}
        style={{ borderRadius: "50%", width: 120 }}
      />
      <h2>{kullanici.ad}</h2>
      <p className="unvan">{kullanici.unvan}</p>

      <span className={kullanici.aktif ? "rozet aktif" : "rozet"}>
        {kullanici.aktif ? "Çevrimiçi" : "Çevrimdışı"}
      </span>

      <h3>Yetenekler ({yetenekSayisi})</h3>
      {yetenekSayisi > 0 && (
        <ul>
          {kullanici.yetenekler.map((y) => (
            <li key={y}>{y}</li>
          ))}
        </ul>
      )}

      <label htmlFor="mesaj">Mesaj gönder</label>
      <input id="mesaj" type="text" placeholder="Yaz..." />
    </div>
  );
}

export default App;
```

Bu örnekte kullanılan kurallar: tek kök element, `className`, `htmlFor`, `{}` ile ifadeler, template literal, stil nesnesi, ternary, güvenli `&&`, `map` ile liste, kapatılmış `<img />` ve `<input />`, JSX yorumu.

---

## 13. Özet

- JSX, JavaScript'e HTML benzeri sözdizimi ekleyen bir uzantıdır; derlenerek `_jsx(...)` çağrılarına dönüşür.
- Her JSX ifadesi bir **React elementi** (sade bir JS nesnesi) üretir.
- **Tek kök:** Kardeş elementler Fragment (`<>...</>`) ile sarılır.
- **Tüm etiketler kapatılır:** `<img />`, `<input />`, `<br />`.
- **camelCase nitelikler:** `className`, `htmlFor`, `onClick`; `data-*` ve `aria-*` hariç.
- `{ }` içine yalnızca **ifadeler** yazılır; `if`/`for` JSX dışında kullanılır.
- `true`/`false`/`null`/`undefined` görünmez, **`0` görünür**, düz nesne hata verir, diziler render edilir.
- `style` bir nesne alır: camelCase özellikler, sayılara otomatik `px`.
- JSX yorumu: `{/* ... */}`
- Bileşen adları **büyük harfle** başlar.
- React string'leri otomatik kaçışlar (XSS koruması); `dangerouslySetInnerHTML` dikkatle kullanılır.
- JSX satır sonu boşluklarını siler; gerekirse `{" "}` kullanılır.

---

⬅️ Önceki: [01 · React Nedir ve Neden Var?](./01-react-nedir.md) · ➡️ Sonraki: [03 · Bileşenler](./03-bilesenler.md)
