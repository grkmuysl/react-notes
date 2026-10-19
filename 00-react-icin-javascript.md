# 00 · React İçin JavaScript

> **Konu özeti:** React'te sürekli karşılaşılan modern JavaScript özellikleri ve her birinin React'te nerede kullanıldığı.

---

## İçindekiler

1. [`let` ve `const`](#1-let-ve-const)
2. [Arrow Function](#2-arrow-function)
3. [Template Literal](#3-template-literal)
4. [Destructuring](#4-destructuring-)
5. [Spread ve Rest](#5-spread-ve-rest-)
6. [Dizi Metotları](#6-dizi-metotları-)
7. [Koşullu İfadeler](#7-koşullu-ifadeler)
8. [Modüller: `import` / `export`](#8-modüller-import--export)
9. [Promise ve `async` / `await`](#9-promise-ve-async--await)
10. [Kendini Kontrol Et (Cevaplı)](#10-kendini-kontrol-et-cevaplı)
11. [Özet](#11-özet)

> ⭐ ile işaretli konular React'te en sık kullanılanlardır.

---

## 1. `let` ve `const`

```js
const ad = "Ayşe";   // yeniden atanamaz
let yas = 25;        // yeniden atanabilir
yas = 26;            // ✅
// ad = "Ali";       // ❌ TypeError

const kisi = { ad: "Ayşe" };
kisi.ad = "Ali";     // ✅ nesnenin içi değişebilir
// kisi = {};        // ❌ değişkene yeni değer atanamaz
```

- `const` **değişkenin yeniden atanmasını** engeller, değerin içeriğini dondurmaz.
- `var` kullanılmaz: blok kapsamı (block scope) yoktur, beklenmedik davranışlara yol açar.
- Varsayılan tercih `const`; değer gerçekten yeniden atanacaksa `let`.

**React'te:** Neredeyse her şey `const` ile tanımlanır, state dahil:

```jsx
const [sayi, setSayi] = useState(0);
```

---

## 2. Arrow Function

```js
// Klasik fonksiyon
function topla(a, b) {
  return a + b;
}

// Arrow function
const topla = (a, b) => a + b;              // tek ifade → return otomatik
const kare = x => x * x;                     // tek parametre → parantez opsiyonel
const selamla = () => {                      // süslü parantezli gövde → return elle yazılır
  console.log("Selam");
};
const nesneDon = () => ({ ad: "Ayşe" });     // nesne döndürürken parantez şart!
```

| Yazım | Return |
|---|---|
| `x => x * 2` | Otomatik (implicit) |
| `x => { return x * 2; }` | Elle (explicit) |
| `() => ({ a: 1 })` | Nesne döndürmek için `()` ile sarılır |

**React'te:** Olay yöneticileri ve kısa callback'ler:

```jsx
<button onClick={() => setSayi(sayi + 1)}>Artır</button>
```

---

## 3. Template Literal

Backtick (`` ` ``) ile yazılan, içine `${}` ile ifade gömülebilen string'ler.

```js
const ad = "Ayşe";
const mesaj = `Merhaba ${ad}, ${2 + 3} yeni mesajın var`;

const cokSatir = `
  Birinci satır
  İkinci satır
`;
```

**React'te:** Dinamik class isimleri:

```jsx
<button className={`buton ${aktif ? "aktif" : ""}`}>Tıkla</button>
```

---

## 4. Destructuring ⭐

Nesne veya diziden değerleri ayrı değişkenlere "parçalayarak" alma.

### Nesne destructuring

```js
const kullanici = { ad: "Ayşe", yas: 25, sehir: "İstanbul" };

const { ad, yas } = kullanici;               // isimle eşleşir
const { sehir: yasadigiYer } = kullanici;    // yeniden adlandırma
const { ulke = "Türkiye" } = kullanici;      // varsayılan değer
```

### Dizi destructuring

```js
const renkler = ["kırmızı", "yeşil", "mavi"];

const [ilk, ikinci] = renkler;               // sıraya göre eşleşir
const [, , ucuncu] = renkler;                // eleman atlama
```

### Fonksiyon parametresinde

```js
function kart({ baslik, aciklama = "Açıklama yok" }) {
  return `${baslik}: ${aciklama}`;
}

kart({ baslik: "React" }); // "React: Açıklama yok"
```

**React'te:**

```jsx
// useState → dizi destructuring
const [deger, setDeger] = useState(0);

// Props → nesne destructuring
function Kart({ baslik, aciklama }) {
  return <h2>{baslik}</h2>;
}
```

> 💡 `useState` dizi döndürür çünkü dizi destructuring'de isimleri **sen** seçersin: `[sayi, setSayi]`, `[ad, setAd]`...

---

## 5. Spread ve Rest ⭐

Aynı `...` sözdizimi, kullanıldığı yere göre iki farklı iş yapar.

### Spread: aç / kopyala

```js
const a = [1, 2];
const b = [...a, 3];                    // [1, 2, 3]
const birlesik = [...a, ...b];          // [1, 2, 1, 2, 3]

const kisi = { ad: "Ayşe", yas: 25 };
const guncel = { ...kisi, yas: 26 };    // { ad: "Ayşe", yas: 26 }
```

> Nesnelerde **sonra yazılan özellik öncekini ezer.**

### Rest: kalanları topla

```js
const { ad, ...geriKalan } = kisi;      // geriKalan = { yas: 25 }

const topla = (...sayilar) => sayilar.reduce((t, s) => t + s, 0);
topla(1, 2, 3); // 6
```

### Sığ (shallow) kopya uyarısı

Spread sadece **ilk seviyeyi** kopyalar; iç içe nesneler hâlâ aynı referansı paylaşır.

```js
const k1 = { ad: "Ayşe", adres: { sehir: "İstanbul" } };
const k2 = { ...k1 };
k2.adres.sehir = "Ankara";
console.log(k1.adres.sehir); // "Ankara" ❗

// Doğrusu: iç nesneyi de kopyala
const k3 = { ...k1, adres: { ...k1.adres, sehir: "Ankara" } };
```

**React'te — immutability (değişmezlik):** State asla doğrudan değiştirilmez, yeni bir kopya oluşturulur:

```js
setKisi({ ...kisi, yas: 26 });          // ✅
// kisi.yas = 26;                        // ❌ React değişikliği fark etmez

setListe([...liste, yeniEleman]);       // ✅ diziye ekleme
// liste.push(yeniEleman);               // ❌
```

---

## 6. Dizi Metotları ⭐

Aşağıdaki metotlar **yeni bir değer döndürür, orijinal diziyi değiştirmez** — bu yüzden React için idealdir.

```js
const urunler = [
  { id: 1, ad: "Kalem", fiyat: 10 },
  { id: 2, ad: "Defter", fiyat: 40 },
  { id: 3, ad: "Çanta", fiyat: 300 },
];
```

| Metot | Ne döndürür | Örnek | Sonuç |
|---|---|---|---|
| `map` | Dönüştürülmüş yeni dizi | `urunler.map(u => u.ad)` | `["Kalem", "Defter", "Çanta"]` |
| `filter` | Koşulu sağlayanlar | `urunler.filter(u => u.fiyat < 50)` | İlk iki ürün |
| `find` | İlk eşleşen **tek eleman** | `urunler.find(u => u.id === 2)` | `{ id: 2, ... }` |
| `findIndex` | İlk eşleşenin indeksi | `urunler.findIndex(u => u.id === 2)` | `1` |
| `reduce` | Tek bir değer | `urunler.reduce((t, u) => t + u.fiyat, 0)` | `350` |
| `some` | En az biri sağlıyor mu | `urunler.some(u => u.fiyat > 100)` | `true` |
| `every` | Hepsi sağlıyor mu | `urunler.every(u => u.fiyat > 5)` | `true` |
| `includes` | Eleman var mı | `[1, 2].includes(2)` | `true` |

### React'te sık kalıplar

```jsx
// Listeyi ekrana basmak
urunler.map(u => <li key={u.id}>{u.ad}</li>)

// Listeden silmek
setUrunler(urunler.filter(u => u.id !== silinecekId));

// Listede tek elemanı güncellemek
setUrunler(urunler.map(u => (u.id === 2 ? { ...u, fiyat: 45 } : u)));

// Listeye eklemek
setUrunler([...urunler, { id: 4, ad: "Silgi", fiyat: 5 }]);
```

### ⚠️ Orijinali değiştiren (mutating) metotlar

State üzerinde **kullanılmamalı**:

| Değiştiren ❌ | Değiştirmeyen alternatif ✅ |
|---|---|
| `push`, `unshift` | `[...dizi, x]`, `[x, ...dizi]` |
| `pop`, `shift`, `splice` | `filter`, `slice`, `toSpliced()` |
| `sort` | `toSorted()` veya `[...dizi].sort()` |
| `reverse` | `toReversed()` veya `[...dizi].reverse()` |
| `dizi[i] = x` | `map` veya `with(i, x)` |

---

## 7. Koşullu İfadeler

### Ternary (`? :`)

```js
const yas = 20;
const durum = yas >= 18 ? "Yetişkin" : "Çocuk";
```

### Mantıksal AND (`&&`)

Sol taraf truthy ise sağ tarafı, değilse sol tarafı döndürür.

```js
true && "Hoş geldin";   // "Hoş geldin"
false && "Hoş geldin";  // false
```

### `||` ve `??` farkı

```js
0 || 10;           // 10 → 0 falsy sayılır
0 ?? 10;           // 0  → ?? sadece null/undefined'da devreye girer
"" || "Misafir";   // "Misafir"
"" ?? "Misafir";   // ""
null ?? "Misafir"; // "Misafir"
```

> Falsy değerler: `false`, `0`, `""`, `null`, `undefined`, `NaN`

### Optional chaining (`?.`)

```js
const k = { adres: null };
k.adres?.sehir;          // undefined — hata fırlatmaz
k.adres.sehir;           // ❌ TypeError
k.selamla?.();           // fonksiyon yoksa çağırmaz
```

**React'te — koşullu render:**

```jsx
{girisYapti ? <Panel /> : <GirisFormu />}
{hata && <p>{hata}</p>}
<p>{kullanici?.ad ?? "Misafir"}</p>
```

> ⚠️ **Tuzak:** `{liste.length && <Liste />}` → liste boşsa ekrana **`0`** basılır.
> Doğrusu: `{liste.length > 0 && <Liste />}`

---

## 8. Modüller: `import` / `export`

```js
// matematik.js
export const PI = 3.14;                          // named export (birden fazla olabilir)
export function topla(a, b) { return a + b; }
export default function carp(a, b) {             // default export (dosyada en fazla bir tane)
  return a * b;
}
```

```js
// app.js
import carp, { PI, topla } from "./matematik.js";
import { topla as ekle } from "./matematik.js";  // yeniden adlandırma
import * as Mat from "./matematik.js";           // hepsini tek nesnede topla → Mat.PI
```

| | Named export | Default export |
|---|---|---|
| Dosyada adet | Sınırsız | En fazla 1 |
| Import yazımı | `{ }` içinde, aynı isimle | `{ }` olmadan, istenen isimle |

**React'te:**

```jsx
import { useState, useEffect } from "react";  // Hook'lar → named
import Navbar from "./components/Navbar";     // bileşenler genelde → default
```

---

## 9. Promise ve `async` / `await`

**Promise:** Gelecekte tamamlanacak (ya da başarısız olacak) bir işlemi temsil eden nesne. Durumları: `pending` → `fulfilled` / `rejected`.

### `.then` zinciri

```js
fetch("https://jsonplaceholder.typicode.com/users")
  .then(res => res.json())
  .then(data => console.log(data))
  .catch(err => console.error(err));
```

### `async` / `await` (daha okunaklı)

```js
async function kullanicilariGetir() {
  try {
    const res = await fetch("https://jsonplaceholder.typicode.com/users");
    if (!res.ok) throw new Error("İstek başarısız");
    const data = await res.json();
    return data;
  } catch (err) {
    console.error(err);
  }
}
```

- `await` sadece `async` fonksiyon içinde (veya modül en üst seviyesinde) kullanılır.
- `async` fonksiyon **her zaman Promise döndürür.**
- `fetch` HTTP hata kodlarında (404, 500) hata **fırlatmaz**; `res.ok` kontrolü bu yüzden gerekir.

### Paralel istekler

```js
const [kullanicilar, gonderiler] = await Promise.all([
  fetch("/api/users").then(r => r.json()),
  fetch("/api/posts").then(r => r.json()),
]);
```

**React'te:** API'den veri çekerken (`useEffect`, Veri Çekme ve React 19 `use` konularında).

---

## 10. Kendini Kontrol Et (Cevaplı)

<details>
<summary><b>1. <code>const</code> ile tanımlanmış bir nesnenin özelliği değiştirilebilir mi? Neden?</b></summary>

Evet. `const` yalnızca **değişkenin başka bir değere yeniden atanmasını** engeller. Değişken bir nesneyi (referansı) tutar; nesnenin kendisi dondurulmaz. Bu yüzden `kisi.ad = "Ali"` çalışır ama `kisi = {}` hata verir. Nesneyi gerçekten dondurmak için `Object.freeze()` kullanılır (o da sığdır).

</details>

<details>
<summary><b>2. <code>||</code> ile <code>??</code> arasındaki fark nedir?</b></summary>

`||` sol taraf **herhangi bir falsy** değerse (`0`, `""`, `false`, `null`, `undefined`, `NaN`) sağ tarafı döndürür.
`??` sadece sol taraf **`null` veya `undefined`** ise sağ tarafı döndürür.

`0` veya `""` gibi geçerli değerlerin korunması gerektiğinde `??` kullanılır: `adet ?? 1` → adet `0` ise `0` kalır, `adet || 1` ise `1` olur.

</details>

<details>
<summary><b>3. <code>{ ...kisi, yas: 26 }</code> ile <code>{ yas: 26, ...kisi }</code> aynı sonucu verir mi?</b></summary>

Hayır. Nesnelerde **sonra yazılan özellik öncekini ezer.**

- `{ ...kisi, yas: 26 }` → önce `kisi` kopyalanır, sonra `yas` 26 ile ezilir → `yas: 26`.
- `{ yas: 26, ...kisi }` → önce `yas: 26` yazılır, sonra `kisi` içindeki `yas` bunu ezer → `yas: 25`.

Güncelleme yaparken spread **önce**, yeni değerler **sonra** yazılır.

</details>

<details>
<summary><b>4. React'te state'i neden <code>push</code> ile değil de spread ile güncelleriz?</b></summary>

React, state'in değişip değişmediğini **referans karşılaştırmasıyla** (`Object.is`) anlar. `push` aynı diziyi yerinde değiştirir; referans aynı kaldığı için React "değişiklik yok" sanır ve bileşeni yeniden render etmeyebilir.

Spread (`[...liste, yeni]`) **yeni bir dizi** oluşturur, referans değişir ve React güncellemeyi algılar. Ayrıca eski değerin bozulmaması hata ayıklamayı ve performans optimizasyonlarını (ör. `memo`) mümkün kılar. Bu ilkeye **immutability** denir.

</details>

---

## 11. Özet

- Varsayılan `const`, gerekirse `let`; `var` kullanılmaz. `const` içeriği dondurmaz.
- Arrow function'da tek ifade otomatik döner; nesne döndürmek için `({ })`.
- **Destructuring:** `useState` (dizi) ve props (nesne) için temel sözdizimi.
- **Spread** kopyalar/açar, **rest** toplar. Spread sığ kopyadır.
- `map`, `filter`, `find`, `reduce` yeni değer döndürür → React için uygun.
- `push`, `splice`, `sort`, `reverse` orijinali değiştirir → state'te kullanılmaz.
- `??` sadece `null`/`undefined`'a, `||` tüm falsy değerlere tepki verir.
- `&&` ile koşullu render'da sayı tuzağına dikkat: `length > 0 &&`.
- Hook'lar named import, bileşenler genelde default export.
- `async` fonksiyon her zaman Promise döndürür; `fetch` hata kodlarında hata fırlatmaz.
- **Immutability:** State'i değiştirme, yeni kopya oluştur.

---

➡️ Sonraki: [01 · React Nedir ve Neden Var?](./01-react-nedir.md)
