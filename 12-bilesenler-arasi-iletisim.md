# 12 · Bileşenler Arası İletişim ve State'i Yukarı Taşıma

> **Konu özeti:** Kardeş bileşenlerin veri paylaşması, state'i yukarı taşıma adımları, controlled ve uncontrolled bileşen tasarımı, state'in nerede yaşaması gerektiği ve colocation, tek doğruluk kaynağı, iletişim kalıpları, callback tasarımı, state yapılandırma ilkeleri (normalizasyon dahil), "React'te Düşünmek" yöntemi ve prop drilling'e karşı kompozisyon.

---

## İçindekiler

1. [Problem: Kardeşler Aynı Veriyi Paylaşmalı](#1-problem-kardeşler-aynı-veriyi-paylaşmalı)
2. [State'i Yukarı Taşımak: Adım Adım](#2-statei-yukarı-taşımak-adım-adım-)
3. [Controlled ve Uncontrolled Bileşenler](#3-controlled-ve-uncontrolled-bileşenler-)
4. [State Nerede Yaşamalı?](#4-state-nerede-yaşamalı-)
5. [Tek Doğruluk Kaynağı](#5-tek-doğruluk-kaynağı-single-source-of-truth)
6. [İletişim Kalıpları](#6-iletişim-kalıpları)
7. [Callback Tasarımı: Ham Setter mı, Niyet mi?](#7-callback-tasarımı-ham-setter-mı-niyet-mi)
8. [State'i Yapılandırma İlkeleri](#8-statei-yapılandırma-ilkeleri)
9. [React'te Düşünmek: 5 Adım](#9-reactte-düşünmek-5-adım-)
10. [Prop Drilling'e Karşı Kompozisyon](#10-prop-drillinge-karşı-kompozisyon)
11. [Her Şey Bir Arada: Ürün Tablosu, Detay ve Sepet](#11-her-şey-bir-arada-ürün-tablosu-detay-ve-sepet)
12. [Özet](#12-özet)

---

## 1. Problem: Kardeşler Aynı Veriyi Paylaşmalı

Her panelin kendi açık/kapalı state'ini tuttuğu bir akordeon:

```jsx
function Panel({ baslik, children }) {
  const [acik, setAcik] = useState(false);
  return (
    <section className="panel">
      <h3>{baslik}</h3>
      {acik ? <p>{children}</p> : <button onClick={() => setAcik(true)}>Göster</button>}
    </section>
  );
}

function SSS() {
  return (
    <>
      <Panel baslik="Kargo ne kadar sürer?">2-4 iş günü içinde teslim edilir.</Panel>
      <Panel baslik="İade nasıl yapılır?">14 gün içinde ücretsiz iade edebilirsiniz.</Panel>
    </>
  );
}
```

**Yeni gereksinim:** Aynı anda yalnızca **bir** panel açık olabilir.

- Panel A, Panel B'nin state'ini göremez ve değiştiremez.
- Kardeşler arasında doğrudan veri yolu yoktur.

→ Çözüm: **State'i yukarı taşımak (lifting state up).**

---

## 2. State'i Yukarı Taşımak: Adım Adım ⭐

### Adım 1: State'i çocuktan kaldır

```jsx
function Panel({ baslik, children, acik }) {
  // const [acik, setAcik] = useState(false);  ← silindi, artık props'tan geliyor
}
```

### Adım 2: Ortak ebeveynde state tanımla, props ile aşağı ver

```jsx
function SSS() {
  const [aktifIndex, setAktifIndex] = useState(0);
  return (
    <>
      <Panel baslik="..." acik={aktifIndex === 0}>...</Panel>
      <Panel baslik="..." acik={aktifIndex === 1}>...</Panel>
    </>
  );
}
```

### Adım 3: Çocuğa, değişiklik istediğinde çağıracağı fonksiyonu ver

```jsx
function Panel({ baslik, children, acik, onGoster }) {
  return (
    <section className="panel">
      <h3>{baslik}</h3>
      {acik ? <p>{children}</p> : <button onClick={onGoster}>Göster</button>}
    </section>
  );
}

function SSS() {
  const [aktifIndex, setAktifIndex] = useState(0);

  return (
    <>
      <Panel
        baslik="Kargo ne kadar sürer?"
        acik={aktifIndex === 0}
        onGoster={() => setAktifIndex(0)}
      >
        2-4 iş günü içinde teslim edilir.
      </Panel>
      <Panel
        baslik="İade nasıl yapılır?"
        acik={aktifIndex === 1}
        onGoster={() => setAktifIndex(1)}
      >
        14 gün içinde ücretsiz iade edebilirsiniz.
      </Panel>
    </>
  );
}
```

### Akış

```
           SSS  (aktifIndex: 0)
          /    \
  acik={true}   acik={false}
  Panel A       Panel B
                    │ tıklandı → onGoster() → setAktifIndex(1)
           SSS  (aktifIndex: 1)
          /    \
  acik={false}  acik={true}
```

> **Veri aşağı (props), olaylar yukarı (callback).** Bu akışa **ters veri akışı (inverse data flow)** da denir.

> 💡 State `aktifIndex` olarak tasarlandı, `[true, false]` gibi bir boolean dizisi olarak değil. "İki panel aynı anda açık" durumu böylece **temsil bile edilemez.**

---

## 3. Controlled ve Uncontrolled Bileşenler ⭐

Input'lardaki ayrım, **kendi bileşenler** için de geçerlidir.

| | Uncontrolled | Controlled |
|---|---|---|
| Önemli bilgi nerede | Bileşenin kendi state'inde | Props'ta (ebeveynde) |
| Kullanımı | Kolay; az ayar | Ebeveyn state yönetmeli |
| Esneklik | Ebeveyn davranışı değiştiremez | Ebeveyn her şeyi koordine edebilir |
| Örnek | Kendi açılıp kapanan basit `Ipucu` | Birbirine bağlı paneller, sekme grubu |

- Karar **her bilgi parçası için ayrı** verilir.
- Örn. `TarihSecici`: seçili tarih controlled (ebeveyn bilmeli), takvim popup'ının açık olması uncontrolled (ebeveyni ilgilendirmez).

### İkisini birden destekleyen bileşen

`<input value>` / `<input defaultValue>` gibi: Ebeveyn `acik` verirse controlled, vermezse bileşen kendi state'ini kullanır.

```jsx
function Akordeon({ acik: disAcik, varsayilanAcik = false, onDegis, baslik, children }) {
  const [icAcik, setIcAcik] = useState(varsayilanAcik);

  const kontrollu = disAcik !== undefined;
  const acik = kontrollu ? disAcik : icAcik;     // tek doğruluk kaynağı seçimi

  function handleToggle() {
    const yeni = !acik;
    if (!kontrollu) setIcAcik(yeni);             // uncontrolled ise kendi state'i
    onDegis?.(yeni);                             // her durumda ebeveyne haber ver
  }

  return (
    <section>
      <button onClick={handleToggle} aria-expanded={acik}>{baslik}</button>
      {acik && <div>{children}</div>}
    </section>
  );
}

// Uncontrolled kullanım
<Akordeon baslik="Detaylar" varsayilanAcik>...</Akordeon>

// Controlled kullanım
<Akordeon baslik="Detaylar" acik={acikMi} onDegis={setAcikMi}>...</Akordeon>
```

> Uygulama kodunda genelde gerekmez; tasarım sistemlerinde ve kütüphanelerde çok yaygındır.

---

## 4. State Nerede Yaşamalı? ⭐

### Karar adımları

1. Bu state'e göre bir şey **render eden** tüm bileşenleri belirle.
2. Bunların **en yakın ortak ebeveynini** bul.
3. State'i oraya (veya hemen üstüne) koy.
4. Mantıklı bir yer yoksa, sadece bu state'i tutacak yeni bir bileşen oluştur ve ortak ebeveynin üstüne yerleştir.

```
App                     ← sepet state'i burada
├── Header
│   └── SepetRozeti     ← sepet sayısını gösteriyor
└── Main
    ├── UrunListesi
    │   └── UrunKarti   ← "Sepete ekle" butonu
    └── Kenar
        └── SepetOzeti  ← sepet içeriğini listeliyor
```

### Colocation: State'i mümkün olduğunca aşağıda tut

Her state'i "ne olur ne olmaz" diye en üste taşımak hatadır:

- Üstteki state değişince **altındaki tüm ağaç** yeniden render olur.
- Okunabilirlik düşer.
- Prop drilling artar.

> **Kural:** State, onu kullanan tüm bileşenleri kapsayan **en alttaki** yerde tutulur.

> State'in yeri sabit değildir; uygulama geliştikçe yukarı-aşağı hareket eder.

---

## 5. Tek Doğruluk Kaynağı (Single Source of Truth)

> **Her benzersiz state parçası tek bir bileşene "aittir".**

```jsx
// ❌ Aynı bilgi iki yerde
function App() {
  const [sepet, setSepet] = useState([]);
  return <Header sepetSayisi={sepet.length} />;
}
function Header({ sepetSayisi }) {
  const [sayi, setSayi] = useState(sepetSayisi);   // ❌ prop'un kopyası
  return <span>{sayi}</span>;                       // sepet değişince güncellenmez
}

// ✅ Tek kaynak, props ile iner
function Header({ sepetSayisi }) {
  return <span>{sepetSayisi}</span>;
}
```

> State paylaşmak gerektiğinde çözüm **kopyalamak değil, yukarı taşımaktır.**

---

## 6. İletişim Kalıpları

| Yön | Yöntem | Örnek |
|---|---|---|
| Ebeveyn → Çocuk | Props | `<Panel acik={...} />` |
| Çocuk → Ebeveyn | Callback props | `<Panel onGoster={...} />` |
| Kardeş ↔ Kardeş | State'i ortak ebeveyne taşı; props + callback | Akordeon panelleri |
| Uzak bileşenler (derin ağaç) | Kompozisyon (`children`) veya **Context** | Tema, oturum bilgisi |
| Ebeveyn → Çocuğa "komut" | Ref + `useImperativeHandle` (kaçış yolu) | `inputRef.current.focus()` |

> Kardeşler arası iletişimin cevabı her zaman **ortak ebeveyndir.** Global olay yayını (event bus) gibi mekanizmalara gerek yoktur; veri akışını izlemeyi zorlaştırır.

---

## 7. Callback Tasarımı: Ham Setter mı, Niyet mi?

### ❌ Ham setter'ı aşağı vermek

```jsx
<SepetSatiri urun={u} setSepet={setSepet} />

function SepetSatiri({ urun, setSepet }) {
  return (
    <button onClick={() => setSepet((s) => s.filter((x) => x.id !== urun.id))}>
      Sil
    </button>
  );
}
```

Sorunlar:

- Çocuk, sepetin **iç yapısını** bilmek zorunda.
- Sepet mantığı birçok bileşene **dağılır**; hata ayıklamak zorlaşır.
- Bileşen farklı veri yapısıyla tekrar kullanılamaz.

### ✅ Niyet bildiren callback'ler

```jsx
<SepetSatiri
  urun={u}
  onSil={() => handleSil(u.id)}
  onAdetDegis={(adet) => handleAdet(u.id, adet)}
/>

function SepetSatiri({ urun, onSil, onAdetDegis }) {
  return (
    <>
      <button onClick={onSil}>Sil</button>
      <input
        type="number"
        value={urun.adet}
        onChange={(e) => onAdetDegis(Number(e.target.value))}
      />
    </>
  );
}
```

- Çocuk **ne olduğunu** söyler ("silindi", "adet değişti").
- **Nasıl** işleneceğine state'in sahibi karar verir; tüm mantık tek yerde toplanır.

> Bu ayrım Konu 14'te (`useReducer`) daha da netleşecek.

---

## 8. State'i Yapılandırma İlkeleri

### 1. İlişkili state'i grupla

```jsx
const [x, setX] = useState(0);
const [y, setY] = useState(0);
// hep birlikte değişiyorsa ↓
const [konum, setKonum] = useState({ x: 0, y: 0 });
```

### 2. Çelişkili state'ten kaçın

```jsx
const [gonderiliyor, setGonderiliyor] = useState(false);
const [gonderildi, setGonderildi] = useState(false);   // ikisi aynı anda true olabilir mi?
// ↓
const [durum, setDurum] = useState("yaziyor");         // "yaziyor" | "gonderiliyor" | "gonderildi"
```

### 3. Gereksiz state'ten kaçın

```jsx
const [tamAd, setTamAd] = useState("");   // ❌
const tamAd = `${ad} ${soyad}`;           // ✅ türetilmiş
```

### 4. Kopyalamaktan kaçın

```jsx
const [seciliUrun, setSeciliUrun] = useState(urunler[0]);   // ❌ bayatlar
const [seciliId, setSeciliId] = useState(urunler[0].id);    // ✅
const seciliUrun = urunler.find((u) => u.id === seciliId);
```

### 5. Derin iç içelikten kaçın: düzleştir (normalize)

```jsx
// ❌ İç içe: bir yorumu güncellemek için 3 seviye kopyalama
const blog = {
  yazilar: [
    { id: 1, baslik: "React", yorumlar: [{ id: 11, metin: "..." } /* ... */] },
  ],
};
```

```jsx
// ✅ Düz (normalize): her varlık kendi "tablosunda", id ile bağlantı
const yazilar = {
  1: { id: 1, baslik: "React", yorumIdleri: [11, 12] },
};
const yorumlar = {
  11: { id: 11, yaziId: 1, metin: "..." },
  12: { id: 12, yaziId: 1, metin: "..." },
};

// Bir yorumu güncellemek tek seviye:
setYorumlar((y) => ({ ...y, 11: { ...y[11], metin: "Güncellendi" } }));
```

> Veritabanı tabloları gibi düşünülür. Büyük uygulamalarda ve Redux gibi kütüphanelerde çok yaygındır.

---

## 9. React'te Düşünmek: 5 Adım ⭐

Örnek: **Filtrelenebilir ürün tablosu**

```js
const URUNLER = [
  { id: 1, kategori: "Meyve", ad: "Elma", fiyat: 25, stokta: true },
  { id: 2, kategori: "Meyve", ad: "Ejder meyvesi", fiyat: 90, stokta: true },
  { id: 3, kategori: "Meyve", ad: "Çarkıfelek", fiyat: 120, stokta: false },
  { id: 4, kategori: "Sebze", ad: "Ispanak", fiyat: 30, stokta: true },
  { id: 5, kategori: "Sebze", ad: "Kabak", fiyat: 40, stokta: false },
  { id: 6, kategori: "Sebze", ad: "Bezelye", fiyat: 35, stokta: true },
];
```

```
┌─────────────────────────────────┐
│ [Ara...                       ] │
│ ☐ Sadece stoktakileri göster    │
│                                 │
│ Ad              Fiyat           │
│ MEYVE                           │
│ Elma            25 ₺            │
│ Ejder meyvesi   90 ₺            │
│ Çarkıfelek      120 ₺  (kırmızı)│
│ SEBZE                           │
│ ...                             │
└─────────────────────────────────┘
```

### Adım 1: Arayüzü bileşen hiyerarşisine böl

```
FiltreliUrunTablosu        ← tüm uygulama
├── AramaCubugu            ← kullanıcı girdisi
└── UrunTablosu            ← filtrelenmiş liste
    ├── KategoriSatiri     ← kategori başlığı
    └── UrunSatiri         ← her ürün
```

### Adım 2: Önce statik (state'siz) sürüm

Sadece props ile veriyi ekrana bas; etkileşim yok.

```jsx
import { Fragment } from "react";

function UrunSatiri({ urun }) {
  return (
    <tr>
      <td style={{ color: urun.stokta ? "inherit" : "crimson" }}>{urun.ad}</td>
      <td>{urun.fiyat} ₺</td>
    </tr>
  );
}

function KategoriSatiri({ kategori }) {
  return (
    <tr>
      <th colSpan={2}>{kategori}</th>
    </tr>
  );
}

function UrunTablosu({ urunler }) {
  const gruplar = Object.groupBy(urunler, (u) => u.kategori);
  return (
    <table>
      <thead>
        <tr><th>Ad</th><th>Fiyat</th></tr>
      </thead>
      <tbody>
        {Object.entries(gruplar).map(([kategori, liste]) => (
          <Fragment key={kategori}>
            <KategoriSatiri kategori={kategori} />
            {liste.map((u) => <UrunSatiri key={u.id} urun={u} />)}
          </Fragment>
        ))}
      </tbody>
    </table>
  );
}

function AramaCubugu() {
  return (
    <form>
      <input placeholder="Ara..." />
      <label><input type="checkbox" /> Sadece stoktakileri göster</label>
    </form>
  );
}

function FiltreliUrunTablosu({ urunler }) {
  return (
    <div>
      <AramaCubugu />
      <UrunTablosu urunler={urunler} />
    </div>
  );
}
```

### Adım 3: Minimal ve eksiksiz state'i belirle

| Veri | Değişiyor mu? | Props'tan mı? | Hesaplanabilir mi? | State mi? |
|---|---|---|---|---|
| Orijinal ürün listesi | — | ✅ | — | ❌ |
| Arama metni | ✅ | ❌ | ❌ | ✅ |
| Checkbox değeri | ✅ | ❌ | ❌ | ✅ |
| Filtrelenmiş liste | — | — | ✅ | ❌ |

→ Yalnızca iki state: `aramaMetni`, `sadeceStokta`.

### Adım 4: State'in nerede yaşayacağını belirle

- `UrunTablosu` listeyi bu değerlere göre filtreler.
- `AramaCubugu` bu değerleri gösterir.
- En yakın ortak ebeveyn → **`FiltreliUrunTablosu`.**

```jsx
function FiltreliUrunTablosu({ urunler }) {
  const [aramaMetni, setAramaMetni] = useState("");
  const [sadeceStokta, setSadeceStokta] = useState(false);

  return (
    <div>
      <AramaCubugu aramaMetni={aramaMetni} sadeceStokta={sadeceStokta} />
      <UrunTablosu urunler={urunler} aramaMetni={aramaMetni} sadeceStokta={sadeceStokta} />
    </div>
  );
}
```

> Bu aşamada input'lara yazılamaz ("value var ama onChange yok" uyarısı); veri henüz sadece aşağı akıyor.

### Adım 5: Ters veri akışını ekle

```jsx
function AramaCubugu({ aramaMetni, sadeceStokta, onAramaMetniDegis, onSadeceStoktaDegis }) {
  return (
    <form onSubmit={(e) => e.preventDefault()}>
      <input
        value={aramaMetni}
        onChange={(e) => onAramaMetniDegis(e.target.value)}
        placeholder="Ara..."
      />
      <label>
        <input
          type="checkbox"
          checked={sadeceStokta}
          onChange={(e) => onSadeceStoktaDegis(e.target.checked)}
        />
        Sadece stoktakileri göster
      </label>
    </form>
  );
}
```

> **Önce yapı, sonra minimal state, sonra state'in yeri, en son etkileşim.**

---

## 10. Prop Drilling'e Karşı Kompozisyon

```jsx
// ❌ Layout sepeti kullanmıyor ama aşağı aktarmak zorunda
function App() {
  const [sepet, setSepet] = useState([]);
  return <Layout sepet={sepet} onSepeteEkle={/* ... */} />;
}
function Layout({ sepet, onSepeteEkle }) {
  return (
    <>
      <Header sepet={sepet} />
      <UrunListesi onSepeteEkle={onSepeteEkle} />
    </>
  );
}
```

```jsx
// ✅ Layout sadece "yuva" sağlıyor, içeriği App veriyor
function App() {
  const [sepet, setSepet] = useState([]);
  return (
    <Layout header={<Header sepet={sepet} />}>
      <UrunListesi onSepeteEkle={/* ... */} />
    </Layout>
  );
}
function Layout({ header, children }) {
  return (
    <>
      {header}
      <main>{children}</main>
    </>
  );
}
```

- `Layout` sepetten tamamen habersiz.
- Kompozisyon yetmediğinde (çok derin ağaç, çok sayıda kullanıcı bileşen) → **Context** *(Konu 13)*.

---

## 11. Her Şey Bir Arada: Ürün Tablosu, Detay ve Sepet

```jsx
import { Fragment, useState } from "react";

const URUNLER = [
  { id: 1, kategori: "Meyve", ad: "Elma", fiyat: 25, stokta: true, aciklama: "Amasya elması, 1 kg" },
  { id: 2, kategori: "Meyve", ad: "Ejder meyvesi", fiyat: 90, stokta: true, aciklama: "Adet" },
  { id: 3, kategori: "Meyve", ad: "Çarkıfelek", fiyat: 120, stokta: false, aciklama: "250 g" },
  { id: 4, kategori: "Sebze", ad: "Ispanak", fiyat: 30, stokta: true, aciklama: "Demet" },
  { id: 5, kategori: "Sebze", ad: "Kabak", fiyat: 40, stokta: false, aciklama: "1 kg" },
  { id: 6, kategori: "Sebze", ad: "Bezelye", fiyat: 35, stokta: true, aciklama: "500 g" },
];

// ---------- Saf yardımcı ----------
function filtrele(urunler, aramaMetni, sadeceStokta) {
  const q = aramaMetni.toLocaleLowerCase("tr");
  return urunler.filter(
    (u) => u.ad.toLocaleLowerCase("tr").includes(q) && (!sadeceStokta || u.stokta)
  );
}

// ---------- Sunum bileşenleri (state'siz, controlled) ----------
function AramaCubugu({ aramaMetni, sadeceStokta, onAramaMetniDegis, onSadeceStoktaDegis }) {
  return (
    <form onSubmit={(e) => e.preventDefault()}>
      <input
        value={aramaMetni}
        onChange={(e) => onAramaMetniDegis(e.target.value)}
        placeholder="Ara..."
      />
      <label>
        <input
          type="checkbox"
          checked={sadeceStokta}
          onChange={(e) => onSadeceStoktaDegis(e.target.checked)}
        />
        Sadece stoktakileri göster
      </label>
    </form>
  );
}

function UrunSatiri({ urun, secili, onSec }) {
  return (
    <tr onClick={onSec} className={secili ? "secili" : ""} style={{ cursor: "pointer" }}>
      <td style={{ color: urun.stokta ? "inherit" : "crimson" }}>{urun.ad}</td>
      <td>{urun.fiyat} ₺</td>
    </tr>
  );
}

function UrunTablosu({ urunler, seciliId, onSec }) {
  if (urunler.length === 0) return <p>Eşleşen ürün yok.</p>;

  const gruplar = Object.groupBy(urunler, (u) => u.kategori);
  return (
    <table>
      <thead>
        <tr><th>Ad</th><th>Fiyat</th></tr>
      </thead>
      <tbody>
        {Object.entries(gruplar).map(([kategori, liste]) => (
          <Fragment key={kategori}>
            <tr><th colSpan={2}>{kategori}</th></tr>
            {liste.map((u) => (
              <UrunSatiri
                key={u.id}
                urun={u}
                secili={u.id === seciliId}
                onSec={() => onSec(u.id)}
              />
            ))}
          </Fragment>
        ))}
      </tbody>
    </table>
  );
}

function UrunDetay({ urun, sepetteki, onSepeteEkle }) {
  if (!urun) return <aside><p>Detay için bir ürün seçin.</p></aside>;
  return (
    <aside>
      <h3>{urun.ad}</h3>
      <p>{urun.aciklama}</p>
      <p>{urun.fiyat} ₺</p>
      <button type="button" disabled={!urun.stokta} onClick={onSepeteEkle}>
        {urun.stokta ? `Sepete ekle${sepetteki ? ` (${sepetteki})` : ""}` : "Tükendi"}
      </button>
    </aside>
  );
}

function SepetSatiri({ urun, adet, onAdetDegis, onSil }) {
  return (
    <li>
      {urun.ad}
      <input
        type="number"
        min={1}
        value={adet}
        onChange={(e) => onAdetDegis(Number(e.target.value) || 1)}
        style={{ width: 50 }}
      />
      × {urun.fiyat} ₺
      <button type="button" onClick={onSil}>🗑️</button>
    </li>
  );
}

function Sepet({ satirlar, toplam, onAdetDegis, onSil, onBosalt }) {
  return (
    <section>
      <h3>Sepet ({satirlar.length})</h3>
      {satirlar.length === 0 ? (
        <p>Sepetin boş.</p>
      ) : (
        <>
          <ul>
            {satirlar.map(({ urun, adet }) => (
              <SepetSatiri
                key={urun.id}
                urun={urun}
                adet={adet}
                onAdetDegis={(yeni) => onAdetDegis(urun.id, yeni)}
                onSil={() => onSil(urun.id)}
              />
            ))}
          </ul>
          <p>Toplam: <strong>{toplam} ₺</strong></p>
          <button type="button" onClick={onBosalt}>Sepeti boşalt</button>
        </>
      )}
    </section>
  );
}

// ---------- State'in sahibi: tek doğruluk kaynağı ----------
export default function Magaza() {
  // Minimal state
  const [aramaMetni, setAramaMetni] = useState("");
  const [sadeceStokta, setSadeceStokta] = useState(false);
  const [seciliId, setSeciliId] = useState(null);   // nesne değil, kimlik
  const [sepet, setSepet] = useState({});            // normalize: { [urunId]: adet }

  // Türetilmiş değerler
  const gorunenler = filtrele(URUNLER, aramaMetni, sadeceStokta);
  const seciliUrun = URUNLER.find((u) => u.id === seciliId) ?? null;
  const sepetSatirlari = Object.entries(sepet).map(([id, adet]) => ({
    urun: URUNLER.find((u) => u.id === Number(id)),
    adet,
  }));
  const toplam = sepetSatirlari.reduce((t, s) => t + s.urun.fiyat * s.adet, 0);
  const sepetAdedi = sepetSatirlari.reduce((t, s) => t + s.adet, 0);

  // Niyet bildiren olay yöneticileri: tüm sepet mantığı burada
  function handleSepeteEkle(id) {
    setSepet((s) => ({ ...s, [id]: (s[id] ?? 0) + 1 }));
  }
  function handleAdetDegis(id, adet) {
    setSepet((s) => ({ ...s, [id]: adet }));
  }
  function handleSil(id) {
    setSepet((s) => {
      const { [id]: _, ...kalan } = s;   // rest ile anahtarı çıkar
      return kalan;
    });
  }

  return (
    <div className="magaza">
      <header>
        <h1>Manav</h1>
        <span>🛒 {sepetAdedi}</span>
      </header>

      <div className="duzen">
        <main>
          <AramaCubugu
            aramaMetni={aramaMetni}
            sadeceStokta={sadeceStokta}
            onAramaMetniDegis={setAramaMetni}
            onSadeceStoktaDegis={setSadeceStokta}
          />
          <UrunTablosu urunler={gorunenler} seciliId={seciliId} onSec={setSeciliId} />
        </main>

        <div className="kenar">
          <UrunDetay
            urun={seciliUrun}
            sepetteki={seciliUrun ? sepet[seciliUrun.id] : 0}
            onSepeteEkle={() => handleSepeteEkle(seciliUrun.id)}
          />
          <Sepet
            satirlar={sepetSatirlari}
            toplam={toplam}
            onAdetDegis={handleAdetDegis}
            onSil={handleSil}
            onBosalt={() => setSepet({})}
          />
        </div>
      </div>
    </div>
  );
}
```

> **Not:** Tıklanabilir `<tr>` örneği kısa tutmak içindir. Gerçek projede erişilebilirlik için satırın içine bir `<button>` konulması önerilir (Konu 6.12).

### Bu örnekte kullanılanlar

- "React'te Düşünmek" adımlarıyla tasarlanmış tablo
- **Dört kardeş bölge** (arama, tablo, detay, sepet) aynı state'i paylaşıyor → state ortak ebeveynde (`Magaza`)
- Minimal state + türetilmiş değerler
- Nesne yerine kimlik saklama (`seciliId`)
- Normalize edilmiş sepet (`{ [id]: adet }`)
- State'siz, tamamen controlled sunum bileşenleri
- Niyet bildiren callback'ler (`onSil`, `onAdetDegis`)
- Tüm sepet mantığı tek yerde

> Dört farklı bileşen **tek bir kaynaktan** besleniyor ve birbirlerinden habersiz. Ağaç derinleştikçe props geçirmek zahmetli hale gelir → **Context API** *(Konu 13)*.

---

## 12. Özet

- Kardeşler state'i doğrudan paylaşamaz; state **en yakın ortak ebeveyne** taşınır.
- Yukarı taşıma: çocuktan state'i kaldır → ebeveynde tanımla ve props ile ver → çocuğa callback ver.
- **Veri aşağı (props), olaylar yukarı (callback)** — ters veri akışı.
- State, imkânsız durumları temsil edemeyecek şekilde tasarlanır (ör. boolean dizisi yerine `aktifIndex`).
- **Uncontrolled bileşen** bilgiyi kendi state'inde, **controlled bileşen** props'ta tutar; karar her bilgi parçası için ayrı verilir. Kütüphanelerde ikisini birden destekleyen kalıp yaygındır.
- State, onu kullanan tüm bileşenleri kapsayan **en alttaki** yerde tutulur (colocation); yeri uygulama geliştikçe değişebilir.
- **Tek doğruluk kaynağı:** Her state parçası tek bir bileşene aittir; props state'e kopyalanmaz.
- Kardeş iletişimi için event bus değil, ortak ebeveyn kullanılır.
- Çocuklara ham setter yerine **niyet bildiren callback'ler** verilir; mantık state'in sahibinde toplanır.
- **State yapılandırma ilkeleri:** ilişkiliyi grupla, çelişkiden kaçın, gereksizi türet, kopyalama (id sakla), iç içeliği **normalize et.**
- **React'te Düşünmek:** 1) bileşenlere böl, 2) statik sürüm, 3) minimal state, 4) state'in yeri, 5) ters veri akışı.
- Prop drilling'e karşı önce **kompozisyon**, yetmezse **Context.**

---

⬅️ Önceki: [11 · useRef](./11-useref.md) · ➡️ Sonraki: [13 · Context API](./13-context-api.md)
