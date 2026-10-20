# 08 · Listeler ve `key`

> **Konu özeti:** `map` ile liste render etme, liste elemanı bileşenleri, `key`'in ne olduğu ve React'in onu nasıl kullandığı, index'i key yapmanın tehlikesi, iyi key kaynakları, key kuralları, Fragment ile key, filtreleme/sıralama/gruplama, iç içe listeler, boş liste, key ile state sıfırlama ve büyük listeler.

---

## İçindekiler

1. [Listeleri Render Etmek](#1-listeleri-render-etmek)
2. [Nesne Dizileri ve Liste Elemanı Bileşenleri](#2-nesne-dizileri-ve-liste-elemanı-bileşenleri)
3. [`key` Nedir, Neden Gerekli?](#3-key-nedir-neden-gerekli-)
4. [React `key`'i Nasıl Kullanır?](#4-react-keyi-nasıl-kullanır)
5. [Index'i `key` Olarak Kullanmak](#5-️-indexi-key-olarak-kullanmak)
6. [İyi Bir `key` Nereden Gelir?](#6-iyi-bir-key-nereden-gelir)
7. [`key` Kuralları](#7-key-kuralları)
8. [Fragment ile `key`](#8-fragment-ile-key)
9. [Filtrelemek, Sıralamak, Gruplamak](#9-filtrelemek-sıralamak-gruplamak)
10. [İç İçe Listeler](#10-iç-içe-listeler)
11. [Boş Liste](#11-boş-liste)
12. [`key` ile State'i Sıfırlamak](#12-key-ile-statei-sıfırlamak)
13. [Büyük Listeler ve Performans](#13-büyük-listeler-ve-performans)
14. [Her Şey Bir Arada: Okuma Listesi](#14-her-şey-bir-arada-okuma-listesi)
15. [Özet](#15-özet)

---

## 1. Listeleri Render Etmek

React **dizileri** render edebilir. Bir **veri dizisi**, `map` ile bir **JSX dizisine** dönüştürülür:

```jsx
const meyveler = ["Elma", "Armut", "Kiraz"];

function MeyveListesi() {
  return (
    <ul>
      {meyveler.map((meyve) => (
        <li key={meyve}>{meyve}</li>
      ))}
    </ul>
  );
}
```

```
["Elma", "Armut", "Kiraz"]
        │ map(meyve => <li>{meyve}</li>)
        ▼
[<li>Elma</li>, <li>Armut</li>, <li>Kiraz</li>]
        │ React render eder
        ▼
• Elma  • Armut  • Kiraz
```

### Neden `map`?

| Yöntem | JSX içinde çalışır mı? | Sebep |
|---|---|---|
| `map` | ✅ | Yeni dizi döndüren bir **ifade** |
| `forEach` | ❌ | Hiçbir şey döndürmez |
| `for` döngüsü | ❌ | Bir **deyim** |

### ⚠️ Süslü parantezde `return` unutulmamalı

```jsx
{meyveler.map((m) => <li key={m}>{m}</li>)}             // ✅ otomatik return
{meyveler.map((m) => { <li key={m}>{m}</li> })}         // ❌ hiçbir şey dönmez → boş liste
{meyveler.map((m) => { return <li key={m}>{m}</li>; })} // ✅
```

---

## 2. Nesne Dizileri ve Liste Elemanı Bileşenleri

```jsx
const kitaplar = [
  { id: "k1", baslik: "Suç ve Ceza", yazar: "Dostoyevski", yil: 1866 },
  { id: "k2", baslik: "Tutunamayanlar", yazar: "Oğuz Atay", yil: 1972 },
  { id: "k3", baslik: "Kürk Mantolu Madonna", yazar: "Sabahattin Ali", yil: 1943 },
];

function KitapListesi() {
  return (
    <ul>
      {kitaplar.map((kitap) => (
        <li key={kitap.id}>
          <strong>{kitap.baslik}</strong> — {kitap.yazar} ({kitap.yil})
        </li>
      ))}
    </ul>
  );
}
```

### Satırı bileşene çıkarmak

```jsx
function KitapSatiri({ kitap }) {
  return (
    <li className="kitap">
      <strong>{kitap.baslik}</strong>
      <span>{kitap.yazar}</span>
      <small>{kitap.yil}</small>
    </li>
  );
}

function KitapListesi({ kitaplar }) {
  return (
    <ul>
      {kitaplar.map((kitap) => (
        <KitapSatiri key={kitap.id} kitap={kitap} />
      ))}
    </ul>
  );
}
```

### ⚠️ `key` nereye yazılır?

**`map`'in doğrudan döndürdüğü elemente.** Bileşene çıkarıldıysa key bileşenin **kullanıldığı yerde** verilir.

```jsx
// ❌ key bileşenin içinde
function KitapSatiri({ kitap }) {
  return <li key={kitap.id}>...</li>;
}
{kitaplar.map((k) => <KitapSatiri kitap={k} />)}

// ✅ key map'in döndürdüğü elemanda
{kitaplar.map((k) => <KitapSatiri key={k.id} kitap={k} />)}
```

> **Kural:** Key, dizinin elemanlarına verilir. Dizinin elemanları `<KitapSatiri>`'lerdir, içlerindeki `<li>`'ler değil.

---

## 3. `key` Nedir, Neden Gerekli? ⭐

Key verilmezse konsolda şu uyarı görülür:

> *Warning: Each child in a list should have a unique "key" prop.*

**`key`, bir liste elemanının React için kimliğidir.** Elemanın dizideki *konumundan* bağımsız olarak "bu hangi eleman?" sorusunu cevaplar.

### Neden gerekli?

State değişince React yeni ağacı eskisiyle karşılaştırır (reconciliation). Liste değiştiğinde anlaması gerekenler:

- Hangi eleman **yeni eklendi**?
- Hangisi **silindi**?
- Hangisi **yer değiştirdi**?
- Hangisi **aynı kaldı** ama içeriği değişti?

> **Benzetme:** Klasördeki dosyalar ada göre sıralanınca `rapor.pdf`'in yeri değişir ama hâlâ `rapor.pdf`'tir. Key, liste elemanının "dosya adı"dır; sıra değişse bile React elemanı tanır.

---

## 4. React `key`'i Nasıl Kullanır?

Listenin **başına** eleman eklendiğinde:

```
Önce:  [Ayşe, Mehmet]
Sonra: [Zeynep, Ayşe, Mehmet]
```

### Key yoksa (veya index ise) — sırayla eşleştirme

```
Konum 0: Ayşe   → Zeynep   "içeriği değişmiş, güncelle"
Konum 1: Mehmet → Ayşe     "içeriği değişmiş, güncelle"
Konum 2: (yok)  → Mehmet   "yeni, oluştur"
```

- 3 gereksiz işlem.
- Ayşe'nin satırındaki **state** artık Zeynep'e ait.

### Anlamlı key varsa — kimlikle eşleştirme

```
key="zeynep" → önceden yok     "yeni, oluştur ve başa ekle"
key="ayse"   → önceden vardı   "aynı, dokunma"
key="mehmet" → önceden vardı   "aynı, dokunma"
```

- Tek işlem; her state doğru elemanda kalır.

### State ile bağlantı

React state'i **ağaçtaki konuma** göre eşler; listelerde konum bilgisini **key** sağlar.

| Durum | Sonuç |
|---|---|
| Aynı key | Aynı bileşen → state korunur |
| Farklı key | Farklı bileşen → state sıfırlanır |

---

## 5. ⚠️ Index'i `key` Olarak Kullanmak

```jsx
{gorevler.map((gorev, index) => (
  <GorevSatiri key={index} gorev={gorev} />
))}
```

- Uyarı kaybolur ama sorun **gizlenir.**
- Key verilmezse React zaten index kullanır; index vermek "key yok" ile aynıdır.

### Hatayı gözle görmek

```jsx
import { useState } from "react";

function Satir({ ad }) {
  const [not, setNot] = useState("");   // her satırın kendi state'i
  return (
    <li>
      <strong>{ad}</strong>{" "}
      <input value={not} onChange={(e) => setNot(e.target.value)} placeholder="not..." />
    </li>
  );
}

export default function App() {
  const [kisiler, setKisiler] = useState([
    { id: 1, ad: "Ayşe" },
    { id: 2, ad: "Mehmet" },
  ]);

  function basaEkle() {
    setKisiler([{ id: Date.now(), ad: "Zeynep" }, ...kisiler]);
  }

  return (
    <>
      <button onClick={basaEkle}>Başa Zeynep ekle</button>

      <h3>key = index ❌</h3>
      <ul>{kisiler.map((k, i) => <Satir key={i} ad={k.ad} />)}</ul>

      <h3>key = id ✅</h3>
      <ul>{kisiler.map((k) => <Satir key={k.id} ad={k.ad} />)}</ul>
    </>
  );
}
```

Ayşe'ye "doktor", Mehmet'e "mühendis" yazıp butona basılınca:

| | `key = index` ❌ | `key = id` ✅ |
|---|---|---|
| Zeynep | doktor | (boş) |
| Ayşe | mühendis | doktor |
| Mehmet | (boş) | mühendis |

- Index listesinde notlar **yanlış kişilere kayar**: index 0'daki bileşen (state'i "doktor") artık Zeynep'i gösterir.
- Aynı hata **sıralama, filtreleme ve ortadan silmede** de olur.
- Etkilenenler: input değerleri, checkbox işaretleri, açık/kapalı durumları, animasyonlar, odak.

### Index ne zaman kabul edilebilir?

Şu üçü **birden** doğruysa:

1. Liste **statik**: sıralanmıyor, filtrelenmiyor, araya/başa eklenmiyor, ortadan silinmiyor.
2. Elemanların **kendi state'i yok**.
3. Elemanların **benzersiz bir kimliği yok** ve üretmek mantıksız.

> Şüphedeysen index kullanma.

---

## 6. İyi Bir `key` Nereden Gelir?

### 1. Verinin kendi kimliği (en iyisi)

```jsx
<UrunKarti key={urun.id} />
<Yorum key={yorum._id} />            // MongoDB
<Kullanici key={kullanici.email} />  // benzersizse
```

### 2. Yerel veride, oluşturma anında üretmek

```jsx
function handleEkle(metin) {
  setGorevler([...gorevler, { id: crypto.randomUUID(), metin, tamam: false }]);
}
```

- `crypto.randomUUID()` modern tarayıcılarda yerleşiktir (güvenli bağlamda: `https` veya `localhost`).
- `Date.now()` basit durumlarda iş görür ama aynı milisaniyede eklenen elemanlarda çakışır.
- Alternatif: `nanoid` kütüphanesi.

### 3. Doğal olarak benzersiz değer

Tekrarsız string listesinde string'in kendisi: `key={meyve}`. Tekrar ihtimali varsa kullanılmaz.

### 4. Bileşik (composite) key

```jsx
<Ders key={`${ders.gun}-${ders.saat}`} ders={ders} />
```

### ❌ Asla render sırasında üretme

```jsx
{gorevler.map((g) => <GorevSatiri key={Math.random()} gorev={g} />)}        // ❌
{gorevler.map((g) => <GorevSatiri key={crypto.randomUUID()} gorev={g} />)}  // ❌
```

- Her render'da yeni key → React **tüm elemanları silip yeniden oluşturur.**
- State'ler sıfırlanır, input'lar her tuşta odağı kaybeder, performans çöker.
- **Index'ten bile kötüdür.**

> Kimlik **veriyle birlikte** ve **bir kez** oluşturulur; render'da değil.

---

## 7. `key` Kuralları

### 1. Kardeşler arasında benzersiz olmalı

Tüm uygulamada değil, **aynı dizideki elemanlar arasında**:

```jsx
<ul>{kategoriler.map((k) => <li key={k.id}>{k.ad}</li>)}</ul>
<ul>{urunler.map((u) => <li key={u.id}>{u.ad}</li>)}</ul>
// kategori id=1 ile ürün id=1 çakışmaz
```

Aynı listede tekrarlayan key → uyarı ve elemanların karışma riski.

### 2. Kararlı (stable) olmalı

Aynı veri elemanı her render'da aynı key'i almalı.

### 3. Bileşene prop olarak ulaşmaz

```jsx
<GorevSatiri key={g.id} id={g.id} gorev={g} />   // içeride lazımsa ayrıca gönderilir
```

### 4. String veya sayı olmalı

React key'leri string'e çevirir: `1` ile `"1"` aynı key sayılır.

---

## 8. Fragment ile `key`

Her eleman için birden fazla kardeş element döndürülecekse `<>...</>` kısa yazımı **key alamaz**; uzun hali kullanılır:

```jsx
import { Fragment } from "react";

function Sozluk({ terimler }) {
  return (
    <dl>
      {terimler.map((t) => (
        <Fragment key={t.id}>
          <dt>{t.terim}</dt>
          <dd>{t.tanim}</dd>
        </Fragment>
      ))}
    </dl>
  );
}
```

> `<dl>` içinde `<dt>` ve `<dd>` kardeş olmak zorundadır; araya `<div>` koymak HTML'i bozar.

---

## 9. Filtrelemek, Sıralamak, Gruplamak

> Gösterilen liste **state değil, türetilmiş değerdir.** Orijinal veri state'te tutulur, gösterilecek hali render sırasında hesaplanır.

### Filtre + sıralama zinciri

```jsx
function UrunListesi({ urunler }) {
  const [arama, setArama] = useState("");
  const [sadeceStokta, setSadeceStokta] = useState(false);
  const [siralama, setSiralama] = useState("ad"); // "ad" | "fiyatArtan" | "fiyatAzalan"

  const gorunenler = urunler
    .filter((u) => u.ad.toLocaleLowerCase("tr").includes(arama.toLocaleLowerCase("tr")))
    .filter((u) => !sadeceStokta || u.stok > 0)
    .toSorted((a, b) => {
      if (siralama === "fiyatArtan") return a.fiyat - b.fiyat;
      if (siralama === "fiyatAzalan") return b.fiyat - a.fiyat;
      return a.ad.localeCompare(b.ad, "tr");
    });

  return (
    <>
      {/* arama ve filtre kontrolleri... */}
      <ul>
        {gorunenler.map((u) => (
          <li key={u.id}>{u.ad} — {u.fiyat} ₺</li>
        ))}
      </ul>
    </>
  );
}
```

### Dikkat edilecekler

- `sort` orijinal diziyi **değiştirir.** Props/state üzerinde `toSorted` veya `[...dizi].sort()` kullanılır.
- **Türkçe metinler:** `toLowerCase` yerine `toLocaleLowerCase("tr")`, sıralamada `localeCompare(..., "tr")`. Aksi halde İ/i, I/ı, Ş, Ç gibi harflerde arama ve sıralama hatalı olur.
- Filtrelemede key'ler değişmez; `u.id` elemanla birlikte gider. Index kullanılsaydı filtrelemede de karışıklık yaşanırdı.

### Gruplamak

```jsx
const gruplar = Object.groupBy(urunler, (u) => u.kategori);
// { "Elektronik": [...], "Kitap": [...] }

return Object.entries(gruplar).map(([kategori, liste]) => (
  <section key={kategori}>
    <h2>{kategori}</h2>
    <ul>
      {liste.map((u) => (
        <li key={u.id}>{u.ad}</li>
      ))}
    </ul>
  </section>
));
```

> `Object.groupBy` tüm modern tarayıcılarda desteklenir; desteklenmeyen ortamda `reduce` ile aynı sonuç elde edilir.

---

## 10. İç İçe Listeler

```jsx
const menu = [
  {
    id: "c1",
    kategori: "Kahveler",
    urunler: [
      { id: "p1", ad: "Türk Kahvesi", fiyat: 90 },
      { id: "p2", ad: "Latte", fiyat: 120 },
    ],
  },
  {
    id: "c2",
    kategori: "Tatlılar",
    urunler: [{ id: "p3", ad: "Baklava", fiyat: 150 }],
  },
];

function Menu() {
  return (
    <div>
      {menu.map((bolum) => (
        <section key={bolum.id}>
          <h2>{bolum.kategori}</h2>
          <ul>
            {bolum.urunler.map((urun) => (
              <li key={urun.id}>
                {urun.ad} — {urun.fiyat} ₺
              </li>
            ))}
          </ul>
        </section>
      ))}
    </div>
  );
}
```

- Her seviyenin **kendi** key'i vardır.
- İç listedeki key'ler yalnızca kendi kardeşleri arasında benzersiz olmalıdır.
- İki seviyeden derin `map`'lerde iç kısım bileşene çıkarılır (`<MenuBolumu bolum={bolum} />`).

---

## 11. Boş Liste

```jsx
function YorumListesi({ yorumlar }) {
  if (yorumlar.length === 0) {
    return <p className="bos">Henüz yorum yok. İlk yorumu sen yaz!</p>;
  }

  return (
    <ul>
      {yorumlar.map((y) => <Yorum key={y.id} yorum={y} />)}
    </ul>
  );
}
```

### "Veri yok" ile "sonuç yok" ayrımı

```jsx
if (urunler.length === 0) return <p>Henüz ürün eklenmemiş.</p>;
if (gorunenler.length === 0) return <p>"{arama}" için sonuç bulunamadı.</p>;
```

---

## 12. `key` ile State'i Sıfırlamak

`key` yalnızca listeler için değil, **herhangi bir bileşene** verilebilir:

> "Bu bileşenin kimliği bu. Kimlik değişirse eskisini at, yenisini kur."

```jsx
function MesajPaneli() {
  const [aliciId, setAliciId] = useState(1);

  return (
    <>
      <KisiListesi seciliId={aliciId} onSec={setAliciId} />
      <MesajYazmaKutusu key={aliciId} aliciId={aliciId} />
    </>
  );
}
```

- Key olmasaydı, bir kişiye yarım yazılan taslak kişi değişince kutuda kalır ve yanlış kişiye gidebilirdi.
- `key={aliciId}` ile alıcı değişince kutu sıfırlanır.

> "Prop değişince state'i sıfırla" ihtiyacının **en temiz çözümüdür.** `useEffect` ile aynı şeyi yapmaya çalışmaktan daha iyidir *(Konu 10)*.

---

## 13. Büyük Listeler ve Performans

- Birkaç yüz elemana kadar `map` ile render genelde sorun değildir.
- Binlerce satırlık listelerde (loglar, büyük tablolar) tüm satırları DOM'a koymak yavaşlatır.

### Sanallaştırma (virtualization / windowing)

- Sadece ekranda **görünen** satırlar render edilir, kaydırdıkça değiştirilir.
- 10.000 elemanlı listede DOM'da aynı anda ~20 satır olur.
- Kütüphaneler: **TanStack Virtual**, **react-window**, **react-virtuoso**.

> İlk tercih bu değildir; önce **sayfalama** veya **"daha fazla yükle"** düşünülür. *(Performans: Konu 19)*

---

## 14. Her Şey Bir Arada: Okuma Listesi

```jsx
import { useState } from "react";

const BASLANGIC = [
  { id: crypto.randomUUID(), baslik: "Suç ve Ceza", yazar: "Dostoyevski", tur: "Roman", okundu: true },
  { id: crypto.randomUUID(), baslik: "Tutunamayanlar", yazar: "Oğuz Atay", tur: "Roman", okundu: false },
  { id: crypto.randomUUID(), baslik: "Sapiens", yazar: "Yuval Noah Harari", tur: "Tarih", okundu: false },
  { id: crypto.randomUUID(), baslik: "İnce Memed", yazar: "Yaşar Kemal", tur: "Roman", okundu: true },
];

function KitapSatiri({ kitap, onToggle, onSil, onYukari, onAsagi, ilk, son }) {
  const [notAcik, setNotAcik] = useState(false);   // satıra özel state
  const [not, setNot] = useState("");

  return (
    <li className={kitap.okundu ? "okundu" : ""}>
      <label>
        <input
          type="checkbox"
          checked={kitap.okundu}
          onChange={() => onToggle(kitap.id)}
        />
        <strong>{kitap.baslik}</strong> — {kitap.yazar}
      </label>

      <button type="button" onClick={() => onYukari(kitap.id)} disabled={ilk}>↑</button>
      <button type="button" onClick={() => onAsagi(kitap.id)} disabled={son}>↓</button>
      <button type="button" onClick={() => setNotAcik(!notAcik)}>📝</button>
      <button type="button" onClick={() => onSil(kitap.id)}>🗑️</button>

      {notAcik && (
        <input value={not} onChange={(e) => setNot(e.target.value)} placeholder="Not..." />
      )}
    </li>
  );
}

export default function App() {
  const [kitaplar, setKitaplar] = useState(BASLANGIC);
  const [arama, setArama] = useState("");
  const [yeniBaslik, setYeniBaslik] = useState("");

  // --- Türetilmiş değerler ---
  const aramaKucuk = arama.toLocaleLowerCase("tr");
  const filtreli = kitaplar.filter(
    (k) =>
      k.baslik.toLocaleLowerCase("tr").includes(aramaKucuk) ||
      k.yazar.toLocaleLowerCase("tr").includes(aramaKucuk)
  );
  const gruplar = Object.groupBy(filtreli, (k) => k.tur);
  const okunan = kitaplar.filter((k) => k.okundu).length;

  // --- Olay yöneticileri ---
  function handleEkle(e) {
    e.preventDefault();
    const baslik = yeniBaslik.trim();
    if (!baslik) return;
    setKitaplar((onceki) => [
      { id: crypto.randomUUID(), baslik, yazar: "Bilinmiyor", tur: "Diğer", okundu: false },
      ...onceki,               // BAŞA ekleme: key'ler sayesinde state'ler karışmaz
    ]);
    setYeniBaslik("");
  }

  function handleToggle(id) {
    setKitaplar((onceki) =>
      onceki.map((k) => (k.id === id ? { ...k, okundu: !k.okundu } : k))
    );
  }

  function handleSil(id) {
    setKitaplar((onceki) => onceki.filter((k) => k.id !== id));
  }

  function tasi(id, yon) {
    setKitaplar((onceki) => {
      const i = onceki.findIndex((k) => k.id === id);
      const j = i + yon;
      if (j < 0 || j >= onceki.length) return onceki;
      const yeni = [...onceki];
      [yeni[i], yeni[j]] = [yeni[j], yeni[i]];   // kopya üzerinde yer değiştir
      return yeni;
    });
  }

  return (
    <main>
      <h1>Okuma Listem</h1>
      <p>{okunan} / {kitaplar.length} kitap okundu</p>

      <form onSubmit={handleEkle}>
        <input
          value={yeniBaslik}
          onChange={(e) => setYeniBaslik(e.target.value)}
          placeholder="Kitap adı"
        />
        <button type="submit">Başa ekle</button>
      </form>

      <input
        value={arama}
        onChange={(e) => setArama(e.target.value)}
        placeholder="Başlık veya yazar ara..."
      />

      {kitaplar.length === 0 ? (
        <p>Listen boş. Bir kitap ekle!</p>
      ) : filtreli.length === 0 ? (
        <p>"{arama}" için sonuç yok.</p>
      ) : (
        Object.entries(gruplar).map(([tur, liste]) => (
          <section key={tur}>
            <h2>{tur} ({liste.length})</h2>
            <ul>
              {liste.map((k) => (
                <KitapSatiri
                  key={k.id}
                  kitap={k}
                  onToggle={handleToggle}
                  onSil={handleSil}
                  onYukari={(id) => tasi(id, -1)}
                  onAsagi={(id) => tasi(id, 1)}
                  ilk={kitaplar[0].id === k.id}
                  son={kitaplar.at(-1).id === k.id}
                />
              ))}
            </ul>
          </section>
        ))
      )}
    </main>
  );
}
```

> **Dene:** Birkaç kitabın not alanını açıp bir şeyler yaz; sonra başa kitap ekle, sırala, ara, sil. Notlar ve açık/kapalı durumları **doğru kitaplarda kalır.** `key={k.id}` yerine `key={index}` yazılırsa karıştıkları görülür.

> **Not:** ↑/↓ okları ana listedeki (`kitaplar`) sırayı değiştirir; `ilk`/`son` da ana listeye göre hesaplanır. Gruplu görünümde sıra değişikliği yalnızca aynı türdeki kitaplar arasında gözlemlenir.

### Bu örnekte kullanılanlar

- `map` ile bileşen listesi
- `crypto.randomUUID()` ile oluşturma anında kimlik
- Satıra özel state
- Başa ekleme / silme / yer değiştirme (immutability ile)
- Türetilmiş filtreli ve gruplanmış liste
- İç içe `map` ve her seviyede key
- Türkçe uyumlu arama
- İki farklı boş durum mesajı

---

## 15. Özet

- Listeler, veri dizisinin `map` ile **JSX dizisine** dönüştürülmesiyle render edilir; `forEach` ve `for` JSX içinde çalışmaz.
- Süslü parantezli arrow function'da `return` unutulmamalı.
- **Key, `map`'in doğrudan döndürdüğü elemente** verilir; bileşene çıkarıldıysa kullanıldığı yerde.
- **Key, liste elemanının React için kimliğidir**; reconciliation sırasında ekleme, silme ve yer değişimini anlamayı sağlar.
- Aynı key → state korunur; farklı key → state sıfırlanır.
- **Index'i key yapmak**, başa ekleme, sıralama, filtreleme ve silmede state'in yanlış elemanlara kaymasına yol açar; yalnızca statik, state'siz listelerde kabul edilebilir.
- İyi key kaynakları: verinin **kendi id'si**, oluşturma anında üretilen **`crypto.randomUUID()`**, benzersiz doğal değerler, bileşik key'ler.
- Key **asla render sırasında** üretilmez (`Math.random()` ❌).
- Key'ler kardeşler arasında **benzersiz** ve **kararlı** olmalı; bileşene prop olarak ulaşmaz.
- Birden fazla kardeş element için **`<Fragment key>`** kullanılır.
- Filtrelenmiş/sıralanmış liste **türetilmiş değerdir**; `sort` yerine `toSorted`, Türkçe metinlerde `toLocaleLowerCase("tr")` ve `localeCompare(..., "tr")`.
- Gruplama için `Object.groupBy`; iç içe listelerde her seviyenin kendi key'i olur.
- Boş liste ve "sonuç yok" durumları ayrı ele alınır.
- **Key ile state sıfırlama:** Prop değişince bileşenin sıfırlanması gerekiyorsa `key={prop}` verilir.
- Çok büyük listelerde sayfalama veya **sanallaştırma** kullanılır.

---

⬅️ Önceki: [07 · Koşullu Render](./07-kosullu-render.md) · ➡️ Sonraki: [09 · Formlar](./09-formlar.md)
