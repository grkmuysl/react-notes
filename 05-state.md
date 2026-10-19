# 05 · State ve `useState`

> **Konu özeti:** Normal değişkenlerin neden yetmediği, `useState` kullanımı, Hook kuralları, state'in anlık görüntü (snapshot) doğası, updater fonksiyon ve batching, nesne ve dizilerde immutability, state'in izolasyonu, başlangıç değeri, state tasarımı ve state'in ağaçtaki konuma bağlılığı.

---

## İçindekiler

1. [Problem: Normal Değişkenler Neden Yetmiyor?](#1-problem-normal-değişkenler-neden-yetmiyor)
2. [`useState` Temelleri](#2-usestate-temelleri)
3. [Hook'lar ve Kuralları](#3-hooklar-ve-kuralları)
4. [State Bir Anlık Görüntüdür](#4-state-bir-anlık-görüntüdür-snapshot-)
5. [Güncelleyici (Updater) Fonksiyon ve Batching](#5-güncelleyici-updater-fonksiyon-ve-batching)
6. [Birden Fazla State](#6-birden-fazla-state)
7. [State'te Nesneler: Immutability](#7-statete-nesneler-immutability-)
8. [State'te Diziler](#8-statete-diziler-)
9. [State Bileşene Özeldir ve İzoledir](#9-state-bileşene-özeldir-ve-izoledir)
10. [Başlangıç Değeri](#10-başlangıç-değeri)
11. [Aynı Değeri Set Etmek](#11-aynı-değeri-set-etmek)
12. [State'e Ne Konmalı?](#12-statee-ne-konmalı-)
13. [State Ağaçtaki Konuma Bağlıdır](#13-state-ağaçtaki-konuma-bağlıdır)
14. [Her Şey Bir Arada: Görev Listesi](#14-her-şey-bir-arada-görev-listesi)
15. [Özet](#15-özet)

---

## 1. Problem: Normal Değişkenler Neden Yetmiyor?

```jsx
function Sayac() {
  let sayi = 0;

  function handleArtir() {
    sayi = sayi + 1;
    console.log(sayi); // konsolda artıyor
  }

  return (
    <>
      <p>{sayi}</p>              {/* ekranda hep 0 */}
      <button onClick={handleArtir}>Artır</button>
    </>
  );
}
```

### İki sebep

1. **Değişiklik render'ı tetiklemiyor.** React yerel bir değişkenin değiştiğinden habersizdir.
2. **Yerel değişkenler render'lar arasında korunmuyor.** Bileşen yeniden çalışsa bile `let sayi = 0` değeri sıfırlar.

### İhtiyaç

- Değeri render'lar arasında **hatırlamak**
- Değer değişince React'e **"yeniden render et"** demek

→ İkisini birden sağlayan araç: **`useState`**

---

## 2. `useState` Temelleri

```jsx
import { useState } from "react";

function Sayac() {
  const [sayi, setSayi] = useState(0);

  function handleArtir() {
    setSayi(sayi + 1);
  }

  return (
    <>
      <p>{sayi}</p>
      <button onClick={handleArtir}>Artır</button>
    </>
  );
}
```

### Sözdizimi

```jsx
const [sayi, setSayi] = useState(0);
//      │        │               │
//      │        │               └─ başlangıç değeri (sadece ilk render'da kullanılır)
//      │        └─ setter: değeri güncelleyen ve render'ı tetikleyen fonksiyon
//      └─ mevcut değer (bu render'daki)
```

- `useState` iki elemanlı bir **dizi** döndürür; dizi destructuring ile alınır.
- İsimlendirme geleneği: `[birSey, setBirSey]`
- Başlangıç değeri her türden olabilir:

```jsx
const [ad, setAd] = useState("");
const [acik, setAcik] = useState(false);
const [kullanici, setKullanici] = useState(null);
const [urunler, setUrunler] = useState([]);
const [form, setForm] = useState({ email: "", sifre: "" });
```

### Tıklandığında ne olur?

```
1. Kullanıcı butona tıklar
2. setSayi(0 + 1) çağrılır → React'e "sayi artık 1, yeniden render et" denir
3. React Sayac fonksiyonunu tekrar çalıştırır
4. useState(0) bu kez hatırlanan 1'i döndürür
5. JSX'te <p>1</p> oluşur → fark DOM'a uygulanır
```

> `set` fonksiyonu, "tetikleme → render → commit" döngüsünün tetikleyicisidir.

---

## 3. Hook'lar ve Kuralları

**Hook:** `use` ile başlayan, fonksiyon bileşenlerinin React özelliklerine (state, yaşam döngüsü, context…) erişmesini sağlayan özel fonksiyonlar. `useState` ilk Hook'tur.

### Kurallar

1. **Sadece en üst seviyede çağrılır.** `if`, döngü, iç içe fonksiyon veya erken `return` sonrasında değil.
2. **Sadece React bileşenlerinde veya custom Hook'larda çağrılır.**

```jsx
// ❌ YANLIŞ
function Profil({ girisYapti }) {
  if (girisYapti) {
    const [ad, setAd] = useState("");   // koşul içinde
  }

  if (!girisYapti) return null;
  const [yas, setYas] = useState(0);    // erken return'den sonra
}
```

```jsx
// ✅ DOĞRU
function Profil({ girisYapti }) {
  const [ad, setAd] = useState("");
  const [yas, setYas] = useState(0);

  if (!girisYapti) return null;
  // ...
}
```

### Neden bu kural var?

React state'leri isimleriyle değil, **çağrılma sırasıyla** tutar:

```
Profil bileşeninin state listesi:
  [0] → "Ayşe"   (1. useState çağrısı)
  [1] → 25       (2. useState çağrısı)
```

Bir Hook koşula bağlı olursa sıra kayar ve değerler yanlış değişkenlere eşleşir. Hook'lar her render'da **aynı sırayla** çağrılmalıdır.

> 💡 `eslint-plugin-react-hooks` (Vite şablonunda hazır gelir) bu kuralları otomatik kontrol eder.

---

## 4. State Bir Anlık Görüntüdür (Snapshot) ⭐

```jsx
function Sayac() {
  const [sayi, setSayi] = useState(0);

  function handleUcArtir() {
    setSayi(sayi + 1);
    setSayi(sayi + 1);
    setSayi(sayi + 1);
  }

  return <button onClick={handleUcArtir}>{sayi}</button>;
}
```

Bir tıklamada sonuç **1** olur, 3 değil.

### Sebep

`setSayi` değişkeni o anda değiştirmez; **bir sonraki render için** istek bırakır. Bu render boyunca `sayi` sabittir:

```js
setSayi(0 + 1);
setSayi(0 + 1);
setSayi(0 + 1);
```

> **Her render, kendi state değerleriyle çekilmiş bir fotoğraftır.** O render'daki tüm kod (JSX, olay yöneticileri) o fotoğraftaki değerleri görür.

### `set` sonrası okumak eski değeri verir

```jsx
function handleTikla() {
  setSayi(sayi + 1);
  console.log(sayi);      // hâlâ ESKİ değer (0)
}

// Yeni değer gerekiyorsa değişkende hesapla
function handleTikla() {
  const yeniSayi = sayi + 1;
  setSayi(yeniSayi);
  console.log(yeniSayi);  // 1
}
```

### Zaman aşımında da aynı

```jsx
function handleGonder() {
  setSayi(sayi + 5);
  setTimeout(() => {
    alert(sayi);   // 3 saniye sonra bile tıklandığı andaki değer
  }, 3000);
}
```

Fonksiyon, oluşturulduğu render'ın değerlerini hatırlar (JS closure). Bu bilinçli bir tasarımdır: olay yöneticisi, tıklandığı anki ekranla tutarlı çalışır.

---

## 5. Güncelleyici (Updater) Fonksiyon ve Batching

### Updater fonksiyon

Yeni değer **önceki değere dayanıyorsa** `set`'e değer yerine **fonksiyon** verilir:

```jsx
function handleUcArtir() {
  setSayi(s => s + 1);
  setSayi(s => s + 1);
  setSayi(s => s + 1);
}
// Sonuç: 3 ✅
```

React bu fonksiyonları **kuyruğa** koyar ve sırayla uygular:

| Kuyruk | `s` (gelen) | Döndürülen |
|---|---|---|
| `s => s + 1` | 0 | 1 |
| `s => s + 1` | 1 | 2 |
| `s => s + 1` | 2 | 3 |

### Karışık kullanım

```jsx
setSayi(sayi + 5);    // "değeri 5 yap"   (sayi = 0) → 5
setSayi(s => s + 1);  // "önceki + 1"                → 6
setSayi(42);          // "değeri 42 yap"             → 42
```

### Ne zaman hangisi?

| Durum | Tercih |
|---|---|
| Basit tek güncelleme (`setAd(e.target.value)`, `setAcik(true)`) | Doğrudan değer |
| Önceki değere göre güncelleme (`+1`, `!acik`, listeye ekleme) | Updater fonksiyon |
| Aynı olayda birden çok güncelleme, zamanlayıcı, async kod | Updater fonksiyon |

```jsx
setAcik(a => !a);                              // toggle kalıbı
setUrunler(onceki => [...onceki, yeniUrun]);  // listeye ekleme
```

- Parametre adı genelde state'in ilk harfi veya `onceki` olur.
- Updater fonksiyon **saf** olmalı: içinde başka `set` çağrılmaz, API isteği atılmaz. StrictMode bunu da iki kez çalıştırır.

### Batching (toplu güncelleme)

- React, bir olay yöneticisindeki tüm `set` çağrılarını **toplar ve tek bir render** yapar.
- Üç `setSayi` → üç değil, **bir** render.
- React 18'den beri `setTimeout`, Promise ve diğer async kodlarda da otomatik.

> Garsonun her sipariş için mutfağa koşmayıp hepsini alıp tek seferde gitmesi gibi.

---

## 6. Birden Fazla State

```jsx
// Ayrı state'ler
function KayitFormu() {
  const [ad, setAd] = useState("");
  const [email, setEmail] = useState("");
  const [kabulEtti, setKabulEtti] = useState(false);
}

// Tek nesne
const [form, setForm] = useState({ ad: "", email: "", kabulEtti: false });
```

| Durum | Tercih |
|---|---|
| Birbirinden bağımsız değişen değerler | Ayrı state |
| Hep birlikte değişen değerler (`x` ve `y` konumu, form alanları) | Tek nesne |

> ⚠️ `useState`'in `set` fonksiyonu nesneleri **birleştirmez, tamamen değiştirir.** `setForm({ ad: "Ayşe" })` yazılırsa `email` ve `kabulEtti` kaybolur. (Class bileşenlerdeki `this.setState` birleştiriyordu.)

---

## 7. State'te Nesneler: Immutability ⭐

> **State asla doğrudan değiştirilmez; yeni bir kopya oluşturulur.**

```jsx
const [kisi, setKisi] = useState({ ad: "Ayşe", yas: 25 });

// ❌ Mutasyon
kisi.yas = 26;
setKisi(kisi);              // aynı nesne → React değişiklik görmez

// ✅ Yeni nesne
setKisi({ ...kisi, yas: 26 });
```

### Neden?

- React yeni değeri eskisiyle **`Object.is`** ile, yani **referansla** karşılaştırır.
- Aynı nesne değiştirilip geri verilirse referans aynıdır → React render'ı atlar.
- Ayrıca önceki render'ın "fotoğrafı" da bozulur.

> **Kural:** State'teki nesne ve dizileri salt okunur kabul et. Kopyala → kopyayı değiştir → kopyayı `set` et.

### Tek alan güncelleyen form kalıbı

```jsx
function ProfilFormu() {
  const [form, setForm] = useState({ ad: "", email: "", sehir: "" });

  function handleChange(e) {
    setForm({ ...form, [e.target.name]: e.target.value });
    //                  └ computed property: name="email" ise { email: ... }
  }

  return (
    <>
      <input name="ad" value={form.ad} onChange={handleChange} />
      <input name="email" value={form.email} onChange={handleChange} />
      <input name="sehir" value={form.sehir} onChange={handleChange} />
    </>
  );
}
```

### İç içe nesneler

Spread sığ kopya yapar; **değişen her seviye** ayrıca kopyalanmalıdır.

```jsx
const [kullanici, setKullanici] = useState({
  ad: "Ayşe",
  adres: { sehir: "İstanbul", ilce: "Kadıköy" },
});

// ❌ İç nesneyi mutasyona uğratır
kullanici.adres.sehir = "Ankara";
setKullanici({ ...kullanici });

// ✅ Her seviye yeni
setKullanici({
  ...kullanici,
  adres: { ...kullanici.adres, sehir: "Ankara" },
});
```

### Derin yapılar için çözümler

- **State'i düzleştirmek** (daha az iç içe tasarlamak) — genelde en iyisi.
- **Immer** (`use-immer`): mutasyon gibi yazılır, arka planda kopya üretilir.

```jsx
updateKullanici(taslak => {
  taslak.adres.sehir = "Ankara";
});
```

---

## 8. State'te Diziler ⭐

```jsx
const [gorevler, setGorevler] = useState([
  { id: 1, metin: "React çalış", tamam: false },
  { id: 2, metin: "Notları GitHub'a yükle", tamam: true },
]);
```

| İşlem | ❌ Mutasyon | ✅ Yeni dizi |
|---|---|---|
| Sona ekle | `push` | `[...dizi, yeni]` |
| Başa ekle | `unshift` | `[yeni, ...dizi]` |
| Sil | `splice` | `filter` |
| Güncelle | `dizi[i] = x` | `map` |
| Araya ekle | `splice` | `slice` + spread / `toSpliced` |
| Sırala / ters çevir | `sort` / `reverse` | `toSorted` / `toReversed` |

```jsx
// Ekle
setGorevler([...gorevler, { id: Date.now(), metin: "Yeni görev", tamam: false }]);

// Sil
setGorevler(gorevler.filter(g => g.id !== silinecekId));

// Güncelle (toggle)
setGorevler(gorevler.map(g =>
  g.id === hedefId ? { ...g, tamam: !g.tamam } : g
));

// Araya ekle (index 1'e)
setGorevler([...gorevler.slice(0, 1), yeniGorev, ...gorevler.slice(1)]);

// Sırala
setGorevler(gorevler.toSorted((a, b) => a.metin.localeCompare(b.metin, "tr")));
```

> ⚠️ `map` içinde nesne de kopyalanmalı. `g.tamam = !g.tamam; return g;` yazılırsa dizi yeni olur ama içindeki nesne eski render'la paylaşılır → yine mutasyon.

```jsx
// Önceki değere dayandığı için updater formu da sık kullanılır
setGorevler(onceki => [...onceki, yeniGorev]);
```

---

## 9. State Bileşene Özeldir ve İzoledir

### İzolasyon

```jsx
function App() {
  return (
    <>
      <Sayac />   {/* kendi sayi'si */}
      <Sayac />   {/* bambaşka bir sayi */}
    </>
  );
}
```

State, fonksiyona değil **ekrandaki o bileşen örneğine (instance)** bağlıdır. Birine tıklamak diğerini etkilemez.

### Gizlilik

Ebeveyn, çocuğunun state'ini okuyamaz ve değiştiremez.

### Props vs State

| | Props | State |
|---|---|---|
| Nereden gelir | Ebeveynden | Bileşenin kendisinden |
| Kim değiştirir | Ebeveyn (yenisini verir) | Bileşenin kendisi (`set` ile) |
| Bileşen içinde | Salt okunur | Salt okunur (değişim `set` ile) |
| Değişince | Yeniden render | Yeniden render |

### State paylaşmak: Lifting State Up

İki bileşen aynı state'i paylaşacaksa state **ortak ebeveyne taşınır**, props ile aşağı verilir:

```jsx
function SayacButonu({ sayi, onArtir }) {
  return <button onClick={onArtir}>Tıklandı: {sayi}</button>;
}

function App() {
  const [sayi, setSayi] = useState(0);   // state ebeveynde
  const handleArtir = () => setSayi(s => s + 1);

  return (
    <>
      <SayacButonu sayi={sayi} onArtir={handleArtir} />
      <SayacButonu sayi={sayi} onArtir={handleArtir} />
    </>
  );
}
```

> **Veri aşağı (props), olay yukarı (fonksiyon).** *(Detayı Konu 12'de.)*

---

## 10. Başlangıç Değeri

### 1. Sadece ilk render'da kullanılır

```jsx
function Karsilama({ baslangicAdi }) {
  const [ad, setAd] = useState(baslangicAdi);
  // baslangicAdi sonradan değişirse ad GÜNCELLENMEZ
}
```

- Props'u state'e kopyalamak genelde hatadır; prop doğrudan kullanılır:

```jsx
function Karsilama({ ad }) {
  return <h1>Merhaba {ad}</h1>;   // ✅ prop değişince otomatik güncellenir
}
```

- Bilinçli olarak "sadece başlangıç değeri" kastediliyorsa adlandırmayla belirtilir: `baslangicAdi`, `varsayilanDeger`.

### 2. Tembel (lazy) başlatma

```jsx
// ❌ hesaplaGorevler() HER render'da çalışır, sonuç ilk render dışında çöpe gider
const [gorevler, setGorevler] = useState(hesaplaGorevler());

// ✅ Fonksiyonun kendisi verilir → React sadece ilk render'da çağırır
const [gorevler, setGorevler] = useState(hesaplaGorevler);
const [gorevler, setGorevler] = useState(
  () => JSON.parse(localStorage.getItem("gorevler")) ?? []
);
```

> Fark parantezlerde: `fn()` sonucunu verir, `fn` fonksiyonun kendisini verir.

---

## 11. Aynı Değeri Set Etmek

Yeni değer `Object.is` ile eskisine eşitse React render'ı **atlar**:

```jsx
const [sayi, setSayi] = useState(0);
setSayi(0);   // değişiklik yok → render yok
```

Bu bir optimizasyondur ve mutasyon hatalarının neden "sessizce" çalışmadığını açıklar.

---

## 12. State'e Ne Konmalı? ⭐

> **State minimal olmalı. Hesaplanabilen hiçbir şey state'e konmaz.**

### ❌ Gereksiz (türetilmiş) state

```jsx
const [ad, setAd] = useState("Ayşe");
const [soyad, setSoyad] = useState("Yılmaz");
const [tamAd, setTamAd] = useState("Ayşe Yılmaz");   // ❌ hesaplanabilir

function handleAdDegis(e) {
  setAd(e.target.value);
  setTamAd(e.target.value + " " + soyad);   // senkron tutmak zorunda, unutulursa hata
}
```

### ✅ Render sırasında hesapla

```jsx
const [ad, setAd] = useState("Ayşe");
const [soyad, setSoyad] = useState("Yılmaz");
const tamAd = `${ad} ${soyad}`;   // her render'da güncel
```

```jsx
const [gorevler, setGorevler] = useState([/* ... */]);
const [filtre, setFiltre] = useState("hepsi");

// ❌ const [gorunenGorevler, setGorunenGorevler] = useState(...)
// ✅
const gorunenGorevler = gorevler.filter(g =>
  filtre === "hepsi" ? true : filtre === "tamam" ? g.tamam : !g.tamam
);
const kalanSayisi = gorevler.filter(g => !g.tamam).length;
```

### Kontrol listesi

Bir değer için sor:

1. Zaman içinde **değişmiyor** mu? → Sabit, state değil.
2. Ebeveynden **props** ile mi geliyor? → State değil.
3. Mevcut state veya props'tan **hesaplanabiliyor** mu? → State değil, render'da hesapla.

Üçüne de **"hayır"** → o değer state'tir.

### Veri kopyası tutma

```jsx
// ❌ Seçili ürünün kendisini kopyalamak
const [seciliUrun, setSeciliUrun] = useState(urunler[0]);
// urunler güncellenirse seciliUrun eski kopyayı gösterir

// ✅ Sadece kimliği tut, ürünü hesapla
const [seciliId, setSeciliId] = useState(urunler[0].id);
const seciliUrun = urunler.find(u => u.id === seciliId);
```

> 💡 Her render'da hesaplamak çoğu durumda ucuzdur. Gerçekten pahalıysa `useMemo` kullanılır (Konu 19). **Önce doğruluk, sonra optimizasyon.**

---

## 13. State Ağaçtaki Konuma Bağlıdır

React state'i bileşenin **ağaçtaki konumuyla** eşleştirir:

| Durum | State |
|---|---|
| Aynı bileşen aynı konumda render edilmeye devam ediyor | **Korunur** |
| Bileşen ağaçtan kaldırılıyor (koşullu render ile gizleniyor) | **Silinir**; geri gelince baştan başlar |
| Aynı konumda farklı türde bileşen render ediliyor | **Silinir** |

```jsx
{goster && <Sayac />}   // goster false olunca Sayac'ın state'i kaybolur
```

### `key` ile bilinçli sıfırlama

```jsx
<SohbetPenceresi key={kisiId} kisiId={kisiId} />
// kisiId değişince React bunu yepyeni bir bileşen sayar → state sıfırlanır
```

> Konu 3.4'teki "bileşen içinde bileşen tanımlama" hatasının state'i silmesinin sebebi de budur: her render'da farklı türde bir bileşen oluşur. *(`key`'in bu rolü Konu 8'de tekrar ele alınacak.)*

---

## 14. Her Şey Bir Arada: Görev Listesi

```jsx
import { useState } from "react";

function GorevSatiri({ gorev, onToggle, onSil }) {
  return (
    <li>
      <label style={{ textDecoration: gorev.tamam ? "line-through" : "none" }}>
        <input
          type="checkbox"
          checked={gorev.tamam}
          onChange={() => onToggle(gorev.id)}
        />
        {gorev.metin}
      </label>
      <button onClick={() => onSil(gorev.id)}>🗑️</button>
    </li>
  );
}

const BASLANGIC = [
  { id: 1, metin: "useState'i öğren", tamam: true },
  { id: 2, metin: "Immutability'yi kavra", tamam: false },
  { id: 3, metin: "05-state.md'yi GitHub'a yükle", tamam: false },
];

export default function App() {
  // --- State (minimal) ---
  const [gorevler, setGorevler] = useState(BASLANGIC);
  const [yeniMetin, setYeniMetin] = useState("");
  const [filtre, setFiltre] = useState("hepsi"); // "hepsi" | "aktif" | "tamam"

  // --- Türetilmiş değerler (state değil!) ---
  const gorunenler = gorevler.filter((g) =>
    filtre === "aktif" ? !g.tamam : filtre === "tamam" ? g.tamam : true
  );
  const kalan = gorevler.filter((g) => !g.tamam).length;

  // --- Olay yöneticileri ---
  function handleEkle() {
    const metin = yeniMetin.trim();
    if (!metin) return;
    setGorevler((onceki) => [...onceki, { id: Date.now(), metin, tamam: false }]);
    setYeniMetin("");
  }

  function handleToggle(id) {
    setGorevler((onceki) =>
      onceki.map((g) => (g.id === id ? { ...g, tamam: !g.tamam } : g))
    );
  }

  function handleSil(id) {
    setGorevler((onceki) => onceki.filter((g) => g.id !== id));
  }

  function handleTamamlananlariTemizle() {
    setGorevler((onceki) => onceki.filter((g) => !g.tamam));
  }

  return (
    <main>
      <h1>Görevlerim</h1>

      <div>
        <input
          value={yeniMetin}
          onChange={(e) => setYeniMetin(e.target.value)}
          placeholder="Yeni görev..."
        />
        <button onClick={handleEkle}>Ekle</button>
      </div>

      <div>
        {["hepsi", "aktif", "tamam"].map((f) => (
          <button
            key={f}
            onClick={() => setFiltre(f)}
            style={{ fontWeight: filtre === f ? "bold" : "normal" }}
          >
            {f}
          </button>
        ))}
      </div>

      {gorunenler.length > 0 ? (
        <ul>
          {gorunenler.map((g) => (
            <GorevSatiri
              key={g.id}
              gorev={g}
              onToggle={handleToggle}
              onSil={handleSil}
            />
          ))}
        </ul>
      ) : (
        <p>Gösterilecek görev yok 🎉</p>
      )}

      <p>
        {kalan} görev kaldı{" "}
        {gorevler.length - kalan > 0 && (
          <button onClick={handleTamamlananlariTemizle}>
            Tamamlananları temizle
          </button>
        )}
      </p>
    </main>
  );
}
```

### Bu örnekte kullanılanlar

- Farklı tiplerde birden çok state
- Minimal state + türetilmiş değerler (`gorunenler`, `kalan`)
- Dizide ekle / sil / güncelle (immutability)
- Updater fonksiyonlar
- State'in ebeveynde tutulup props ve fonksiyonlarla çocuğa verilmesi (`GorevSatiri`)
- Controlled input'a ilk bakış (`value` + `onChange`)

> `onClick`, `onChange`, `e.target.value` sezgisel olarak kullanıldı; detayları **Konu 6: Olay Yönetimi**'nde.

---

## 15. Özet

- Normal değişkenler render'lar arasında korunmaz ve değişimleri render tetiklemez → **`useState`** kullanılır.
- `const [deger, setDeger] = useState(baslangic)`; `set` çağrısı yeniden render'ı tetikler.
- **Hook kuralları:** Sadece en üst seviyede ve sadece bileşenlerde/custom Hook'larda çağrılır; React state'leri çağrı sırasıyla eşleştirir.
- **State bir anlık görüntüdür:** `set` değişkeni hemen değiştirmez, bir sonraki render için istek bırakır.
- Önceki değere dayalı güncellemelerde **updater fonksiyon** (`s => s + 1`) kullanılır.
- **Batching:** Bir olaydaki tüm güncellemeler tek render'da uygulanır.
- `set` nesneleri birleştirmez, tamamen değiştirir.
- **Immutability:** Nesne ve diziler kopyalanarak güncellenir; React referansla (`Object.is`) karşılaştırır.
- İç içe nesnelerde değişen her seviye kopyalanır; derin yapılarda düzleştirme veya Immer.
- Dizilerde `push`/`splice`/`sort` yerine spread, `filter`, `map`, `toSorted` kullanılır.
- State her bileşen örneğine **özel ve izoledir**; paylaşmak için ortak ebeveyne taşınır.
- Başlangıç değeri sadece ilk render'da kullanılır; props state'e kopyalanmaz. Pahalı başlangıçta **lazy initializer**.
- Aynı değer set edilirse render atlanır.
- **State minimal tutulur:** Hesaplanabilen değerler render sırasında türetilir; veri kopyası yerine kimlik (`id`) saklanır.
- State ağaçtaki **konuma** bağlıdır; bileşen kaldırılınca silinir, `key` değişince sıfırlanır.

---

⬅️ Önceki: [04 · Props](./04-props.md) · ➡️ Sonraki: [06 · Olay Yönetimi](./06-olay-yonetimi.md)
