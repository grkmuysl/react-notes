# 04 · Props

> **Konu özeti:** Props'un ne olduğu, gönderme ve alma yöntemleri, gönderilebilen değer tipleri, varsayılan değerler, `children`, salt okunurluk, tek yönlü veri akışı, fonksiyon props'ları, spread ve `...rest`, özel props (`key`, `ref`), prop drilling ve tip kontrolü.

---

## İçindekiler

1. [Props Nedir?](#1-props-nedir)
2. [Props Göndermek ve Almak](#2-props-göndermek-ve-almak)
3. [Hangi Değerler Gönderilebilir?](#3-hangi-değerler-gönderilebilir)
4. [Varsayılan (Default) Değerler](#4-varsayılan-default-değerler)
5. [`children` Prop'u](#5-children-propu-)
6. [Props Salt Okunurdur](#6-props-salt-okunurdur-read-only-)
7. [Tek Yönlü Veri Akışı](#7-tek-yönlü-veri-akışı)
8. [Fonksiyonları Prop Olarak Göndermek](#8-fonksiyonları-prop-olarak-göndermek)
9. [Spread ile Aktarma ve `...rest`](#9-spread-ile-aktarma-ve-rest)
10. [Özel Props: `key` ve `ref`](#10-özel-props-key-ve-ref)
11. [Props Değişince Ne Olur?](#11-props-değişince-ne-olur)
12. [Prop Drilling](#12-prop-drilling)
13. [Props'un Tipini Kontrol Etmek](#13-propsun-tipini-kontrol-etmek)
14. [Her Şey Bir Arada: Ekip Sayfası](#14-her-şey-bir-arada-ekip-sayfası)
15. [Özet](#15-özet)

---

## 1. Props Nedir?

**Props (properties)**, bir ebeveyn bileşenin çocuğuna veri aktarma yoludur. HTML niteliklerine benzer:

```html
<img src="kedi.jpg" alt="Kedi" />                <!-- HTML: etikete nitelik -->
```

```jsx
<ProfilKarti ad="Ayşe" unvan="Geliştirici" />    // React: bileşene prop
```

> **Props, bileşen fonksiyonunun parametresidir.** Fonksiyonlar parametre alarak esnekleşir; bileşenler de props alarak tekrar kullanılabilir hale gelir.

```js
function selamla(ad) { return `Merhaba ${ad}`; }                     // fonksiyon + parametre
```

```jsx
function Selam(props) { return <h1>Merhaba {props.ad}</h1>; }       // bileşen + props
```

---

## 2. Props Göndermek ve Almak

### Gönderen (ebeveyn)

```jsx
function App() {
  return (
    <>
      <ProfilKarti ad="Ayşe Yılmaz" unvan="Frontend Geliştirici" />
      <ProfilKarti ad="Mehmet Kaya" unvan="Backend Geliştirici" />
      <ProfilKarti ad="Zeynep Ak" unvan="UI Tasarımcı" />
    </>
  );
}
```

### Alan (çocuk) — `props` nesnesi

React, verilen tüm nitelikleri **tek bir nesnede** toplayıp bileşene ilk parametre olarak verir.

```jsx
function ProfilKarti(props) {
  console.log(props); // { ad: "Ayşe Yılmaz", unvan: "Frontend Geliştirici" }
  return (
    <div>
      <h3>{props.ad}</h3>
      <p>{props.unvan}</p>
    </div>
  );
}
```

> `<ProfilKarti ad="Ayşe" />` → `_jsx(ProfilKarti, { ad: "Ayşe" })`. İkinci parametredeki nesne, `props`'un kendisidir.

### Alan (çocuk) — destructuring ile ✅ önerilen

```jsx
function ProfilKarti({ ad, unvan }) {
  return (
    <div>
      <h3>{ad}</h3>
      <p>{unvan}</p>
    </div>
  );
}
```

- Daha kısa ve okunaklı.
- Bileşenin hangi props'ları beklediği ilk satırdan anlaşılır.

---

## 3. Hangi Değerler Gönderilebilir?

**Her şey.** String dışındaki tüm değerler `{}` ile gönderilir.

```jsx
<Urun
  ad="Kulaklık"                              // string → tırnak
  fiyat={1299.9}                             // sayı
  stokta={true}                              // boolean
  indirimli                                  // değersiz yazım = true
  etiketler={["kablosuz", "bluetooth"]}      // dizi
  satici={{ ad: "TeknoShop", puan: 4.8 }}    // nesne
  onSepeteEkle={() => alert("Eklendi")}      // fonksiyon
  ikon={<span>🎧</span>}                     // JSX
/>
```

```jsx
function Urun({ ad, fiyat, stokta, indirimli, etiketler, satici, onSepeteEkle, ikon }) {
  return (
    <div className="urun">
      <h3>{ikon} {ad}</h3>
      <p>
        {fiyat.toLocaleString("tr-TR")} ₺ {indirimli && <strong>İndirimde!</strong>}
      </p>
      <p>Satıcı: {satici.ad} (⭐ {satici.puan})</p>
      <p>{etiketler.join(", ")}</p>
      <button disabled={!stokta} onClick={onSepeteEkle}>
        {stokta ? "Sepete Ekle" : "Tükendi"}
      </button>
    </div>
  );
}
```

> ⚠️ `fiyat="1299"` yazılırsa değer **string** olur: `fiyat + 1` → `"12991"`. Sayı için `fiyat={1299}`.

### Değişkenlerle göndermek

```jsx
const kullanici = {
  ad: "Ayşe Yılmaz",
  unvan: "Frontend Geliştirici",
  avatar: "https://i.pravatar.cc/80?img=1",
};

// Ayrı ayrı
<ProfilKarti ad={kullanici.ad} unvan={kullanici.unvan} avatar={kullanici.avatar} />

// Tek nesne olarak
<ProfilKarti kullanici={kullanici} />

function ProfilKarti({ kullanici }) {
  return <h3>{kullanici.ad}</h3>;
}
```

| Yöntem | Avantajı |
|---|---|
| Ayrı ayrı göndermek | Bileşenin neye ihtiyaç duyduğu net |
| Nesne olarak göndermek | Kalabalık veride pratik |

---

## 4. Varsayılan (Default) Değerler

Gönderilmeyen prop'un değeri `undefined` olur. Varsayılan değer destructuring içinde verilir:

```jsx
function Avatar({ src, boyut = 64, alt = "Kullanıcı" }) {
  return <img src={src} width={boyut} height={boyut} alt={alt} />;
}

<Avatar src="a.jpg" />               // boyut = 64
<Avatar src="a.jpg" boyut={120} />   // boyut = 120
<Avatar src="a.jpg" boyut={null} />  // boyut = null ❗
```

> Varsayılan değer **sadece `undefined`** durumunda devreye girer; `null` gönderilirse kullanılmaz.

> 📜 **Eski yöntem:** `Avatar.defaultProps = { boyut: 64 }` — fonksiyon bileşenlerinde **React 19'da kaldırıldı.** Varsayılan parametre yazımı kullanılır.

---

## 5. `children` Prop'u ⭐

Açılış ve kapanış etiketi arasına yazılan her şey `children` adlı özel prop olarak gelir.

```jsx
<Kart>
  <h2>Başlık</h2>
  <p>İçerik</p>
</Kart>

// Yukarıdakiyle tamamen aynı:
<Kart children={<><h2>Başlık</h2><p>İçerik</p></>} />
```

- `children` metin, tek element, birden çok element veya hiçbir şey olabilir.
- İçeriğini bilmeyen, sadece **çerçeve sunan** bileşenler yazmayı sağlar.
- Kompozisyonun en güçlü aracıdır.

### Sarmalayıcı bileşen örnekleri

```jsx
function Kart({ baslik, children }) {
  return (
    <section className="kart">
      {baslik && <h2 className="kart-baslik">{baslik}</h2>}
      <div className="kart-govde">{children}</div>
    </section>
  );
}

function Modal({ acik, children }) {
  if (!acik) return null;
  return (
    <div className="modal-arkaplan">
      <div className="modal">{children}</div>
    </div>
  );
}

function SayfaDuzeni({ children }) {
  return (
    <>
      <Header />
      <main className="container">{children}</main>
      <Footer />
    </>
  );
}

function App() {
  return (
    <SayfaDuzeni>
      <Kart baslik="Hoş geldin">
        <p>Bugün 3 yeni mesajın var.</p>
      </Kart>
      <Kart baslik="Siparişlerin">
        <ul>
          <li>#1042 — Kargoda</li>
          <li>#1039 — Teslim edildi</li>
        </ul>
      </Kart>
    </SayfaDuzeni>
  );
}
```

### Birden fazla yuva (slot)

JSX bir değer olduğu için normal prop olarak da gönderilebilir:

```jsx
function Panel({ baslik, altKisim, children }) {
  return (
    <div className="panel">
      <div className="panel-ust">{baslik}</div>
      <div className="panel-icerik">{children}</div>
      {altKisim && <div className="panel-alt">{altKisim}</div>}
    </div>
  );
}

<Panel
  baslik={<h2>⚙️ Ayarlar</h2>}
  altKisim={<button>Kaydet</button>}
>
  <p>Bildirimleri aç/kapat</p>
</Panel>
```

---

## 6. Props Salt Okunurdur (Read-only) ⭐

> **Bir bileşen kendi props'unu asla değiştirmez.**

```jsx
function Sayac({ deger }) {
  deger = deger + 1;        // ❌ props'u değiştirmek
  return <p>{deger}</p>;
}

function Kullanici({ kullanici }) {
  kullanici.ad = "Ali";     // ❌ ebeveynin nesnesini değiştiriyor!
  return <p>{kullanici.ad}</p>;
}
```

### Neden?

- **Saflık kuralı:** Props bileşenin girdisidir; girdiyi değiştirmek yan etkidir.
- **Referans paylaşımı:** Nesne ve diziler referansla aktarılır. Çocuk değiştirirse ebeveynin verisi de bozulur ve React bundan haberdar olmaz.

> Props bir **anlık görüntü (snapshot)** gibidir: bileşen o render için verilen değerleri okur ve gösterir.

### Değerin değişmesi gerekirse?

- Değiştirme yetkisi **verinin sahibi olan bileşendedir** (genelde ebeveyn).
- Ebeveyn değeri **state** olarak tutar ve değiştirir; yeni değer props ile tekrar aşağı iner.
- Çocuğun değişiklik istemesi gerekiyorsa ebeveyn ona bir **fonksiyon** gönderir (bkz. [Bölüm 8](#8-fonksiyonları-prop-olarak-göndermek)).

---

## 7. Tek Yönlü Veri Akışı

Props yalnızca **yukarıdan aşağıya** akar: ebeveyn → çocuk.

```
       App  (veri burada: kullanici)
        │  props: kullanici
        ▼
    ProfilKarti
        │  props: src, boyut
        ▼
      Avatar
```

- Çocuk, ebeveynine props gönderemez.
- Kardeş bileşenler birbirine doğrudan veri gönderemez.
- Bir verinin kaynağı için ağaçta **yukarı doğru** bakmak yeterlidir → öngörülebilirlik, kolay hata ayıklama.
- Kardeşlerin veri paylaşması gerekirse veri **ortak ebeveyne** taşınır (*lifting state up*, Konu 12).

---

## 8. Fonksiyonları Prop Olarak Göndermek

Veri aşağı akar; çocuk ebeveynine haber vermek için ebeveynin gönderdiği **fonksiyonu çağırır.**

```jsx
function SilButonu({ onSil }) {
  return <button onClick={onSil}>🗑️ Sil</button>;
}

function Yorum({ metin }) {
  function yorumuSil() {
    alert(`"${metin}" silinecek`);
  }

  return (
    <div>
      <p>{metin}</p>
      <SilButonu onSil={yorumuSil} />
    </div>
  );
}
```

- `SilButonu` silmenin ne anlama geldiğini bilmez; sadece "tıklandım" der.
- Ne olacağına ebeveyn karar verir → buton her yerde tekrar kullanılabilir.

### Argümanla çağırmak

```jsx
function Yildizlar({ onSec }) {
  return (
    <div>
      {[1, 2, 3, 4, 5].map((n) => (
        <button key={n} onClick={() => onSec(n)}>⭐</button>
      ))}
    </div>
  );
}

<Yildizlar onSec={(puan) => console.log("Seçilen puan:", puan)} />
```

### İsimlendirme geleneği

| Nerede | Önek | Örnek |
|---|---|---|
| Prop adı | `on` | `onSil`, `onSec`, `onChange` |
| Ebeveyndeki fonksiyon | `handle` | `handleSil`, `handleSec` |

```jsx
<SilButonu onSil={handleSil} />
```

---

## 9. Spread ile Aktarma ve `...rest`

### Nesnenin tüm alanlarını prop olarak açmak

```jsx
const kullanici = { ad: "Ayşe", unvan: "Geliştirici", avatar: "a.jpg" };

<ProfilKarti {...kullanici} />
// = <ProfilKarti ad="Ayşe" unvan="Geliştirici" avatar="a.jpg" />
```

### Bir kısmını kullanıp kalanını aktarmak

```jsx
function Buton({ varyant = "birincil", children, ...rest }) {
  return (
    <button className={`btn btn-${varyant}`} {...rest}>
      {children}
    </button>
  );
}

<Buton varyant="tehlike" onClick={handleSil} disabled={yukleniyor} type="submit">
  Sil
</Buton>
```

`Buton` yalnızca `varyant` ve `children` ile ilgilenir; `onClick`, `disabled`, `type`, `aria-label` gibi diğer her şeyi `...rest` ile gerçek `<button>`'a geçirir. HTML elementlerini saran bileşenlerde (Button, Input, Link) çok yaygın bir kalıptır.

### Sıra önemlidir

```jsx
<button className="btn" {...rest} />   // rest içinde className varsa "btn" ezilir
<button {...rest} className="btn" />   // her zaman "btn" kalır
```

### `className` birleştirme

```jsx
function Buton({ className = "", ...rest }) {
  return <button className={`btn ${className}`} {...rest} />;
}
```

> ⚠️ Spread ölçülü kullanılmalı. `<Bilesen {...props} />` ile her şeyi körü körüne aktarmak bileşenin ne aldığını belirsizleştirir ve DOM'a anlamsız nitelikler sızdırabilir.

---

## 10. Özel Props: `key` ve `ref`

| Prop | Görevi | Bileşene ulaşır mı? | Konu |
|---|---|---|---|
| `key` | Listelerde elemanları ayırt etmek için React'in kullandığı kimlik | ❌ `props.key` her zaman `undefined` | Konu 8 |
| `ref` | DOM elementine doğrudan erişim | ✅ React 19'dan itibaren normal prop gibi alınır | Konu 11 |

```jsx
// key değerine bileşen içinde de ihtiyaç varsa ayrı prop olarak gönderilir
<UrunKarti key={urun.id} id={urun.id} ad={urun.ad} />
```

> 📜 Eski kodlarda `ref` almak için `forwardRef` sarmalayıcısı kullanılırdı; React 19'da fonksiyon bileşenleri için gerekmiyor.

---

## 11. Props Değişince Ne Olur?

Bileşen props'u değiştirmez; **ebeveyn yenisini verir.**

```
Ebeveyn state'i değişir
   → Ebeveyn render olur
   → Çocuğa yeni props gider
   → Çocuk yeni props ile render olur
   → Değişen kısım DOM'a yansır
```

> Her render bir film karesi gibidir: her kare kendi props'uyla çizilir.

---

## 12. Prop Drilling

Derindeki bir bileşene lazım olan verinin, onu kullanmayan ara bileşenlerden geçirilmesi:

```jsx
function App() {
  const kullanici = { ad: "Ayşe" };
  return <Sayfa kullanici={kullanici} />;
}
function Sayfa({ kullanici }) {
  return <Header kullanici={kullanici} />;           // kullanmıyor, sadece aktarıyor
}
function Header({ kullanici }) {
  return <KullaniciMenusu kullanici={kullanici} />;  // kullanmıyor, sadece aktarıyor
}
function KullaniciMenusu({ kullanici }) {
  return <span>👤 {kullanici.ad}</span>;             // asıl kullanan
}
```

- Birkaç seviyede sorun değildir; açık ve okunabilirdir.
- Çok derinleşince zahmetli hale gelir.

### Çözümler

**1. Kompozisyon (`children`)** — çoğu zaman yeterli, ilk denenecek yöntem:

```jsx
function App() {
  const kullanici = { ad: "Ayşe" };
  return (
    <Sayfa>
      <Header>
        <KullaniciMenusu kullanici={kullanici} />
      </Header>
    </Sayfa>
  );
}
```

`Sayfa` ve `Header` artık `kullanici`'yı hiç bilmiyor.

**2. Context API** — uygulamanın her yerinde lazım olan veriler için (tema, oturum açmış kullanıcı, dil). *(Konu 13)*

---

## 13. Props'un Tipini Kontrol Etmek

JavaScript'te yanlış tipte prop göndermek hata vermez, sadece garip davranışlara yol açar.

| Yöntem | Durum |
|---|---|
| **PropTypes** (`prop-types` paketi) | 📜 Eski yöntem; React 19 bu kontrolleri artık yapmıyor. Eski kodda tanınmalı, yeni kodda kullanılmamalı. |
| **TypeScript** | ✅ Günümüz standardı; hataları kod yazarken, editörde yakalar. |

```tsx
type UrunProps = {
  ad: string;
  fiyat: number;
  stokta?: boolean;            // ? = opsiyonel
  onSepeteEkle: () => void;
};

function Urun({ ad, fiyat, stokta = true, onSepeteEkle }: UrunProps) {
  // ...
}
```

*(Konu 22'de detaylı.)*

---

## 14. Her Şey Bir Arada: Ekip Sayfası

```jsx
// src/components/Avatar.jsx
export default function Avatar({ src, ad, boyut = 64 }) {
  return (
    <img
      className="avatar"
      src={src}
      alt={`${ad} fotoğrafı`}
      width={boyut}
      height={boyut}
      style={{ borderRadius: "50%" }}
    />
  );
}
```

```jsx
// src/components/Rozet.jsx
export default function Rozet({ renk = "gri", children }) {
  return <span className={`rozet rozet-${renk}`}>{children}</span>;
}
```

```jsx
// src/components/Kart.jsx
export default function Kart({ children, className = "", ...rest }) {
  return (
    <div className={`kart ${className}`} {...rest}>
      {children}
    </div>
  );
}
```

```jsx
// src/components/ProfilKarti.jsx
import Avatar from "./Avatar";
import Rozet from "./Rozet";
import Kart from "./Kart";

export default function ProfilKarti({
  ad,
  unvan,
  avatar,
  pro = false,
  yetenekler = [],
  onMesaj,
}) {
  return (
    <Kart className="profil-karti">
      <Avatar src={avatar} ad={ad} boyut={80} />
      <h3>{ad}</h3>
      <p>{unvan}</p>

      {pro && <Rozet renk="altin">Pro Üye</Rozet>}

      {yetenekler.length > 0 && (
        <ul>
          {yetenekler.map((y) => (
            <li key={y}>{y}</li>
          ))}
        </ul>
      )}

      <button onClick={() => onMesaj(ad)}>💬 Mesaj gönder</button>
    </Kart>
  );
}
```

```jsx
// src/App.jsx
import ProfilKarti from "./components/ProfilKarti";

const ekip = [
  { id: 1, ad: "Ayşe Yılmaz", unvan: "Frontend Geliştirici", avatar: "https://i.pravatar.cc/80?img=1", pro: true, yetenekler: ["React", "TypeScript"] },
  { id: 2, ad: "Mehmet Kaya", unvan: "Backend Geliştirici", avatar: "https://i.pravatar.cc/80?img=3", yetenekler: ["Node.js", "PostgreSQL"] },
  { id: 3, ad: "Zeynep Ak", unvan: "UI Tasarımcı", avatar: "https://i.pravatar.cc/80?img=5" },
];

function App() {
  function handleMesaj(kime) {
    alert(`${kime} kişisine mesaj penceresi açılıyor...`);
  }

  return (
    <main>
      <h1>Ekibimiz</h1>
      {ekip.map((kisi) => (
        <ProfilKarti key={kisi.id} {...kisi} onMesaj={handleMesaj} />
      ))}
    </main>
  );
}

export default App;
```

### Bu örnekte kullanılanlar

- String, boolean, dizi ve fonksiyon props
- Varsayılan değerler (`pro = false`, `yetenekler = []`, `boyut = 64`)
- `children` (`Rozet`, `Kart`)
- `...rest` ve `className` birleştirme (`Kart`)
- Spread ile aktarma (`{...kisi}`)
- Fonksiyon prop'u ve isimlendirme geleneği (`onMesaj` / `handleMesaj`)
- `key` ile liste

> Her kart artık farklı kişiyi gösteriyor, ama hiçbir şey **değişmiyor.** Zamanla değişen veriler (sayaç, "takip ediliyor" durumu vb.) için props yetmez, çünkü props değiştirilemez. Bileşenin **kendi hafızasına** ihtiyacı var → **State** (Konu 5).

---

## 15. Özet

- **Props**, ebeveynden çocuğa veri aktarır; bileşen fonksiyonunun parametresidir.
- Tüm props tek bir nesnede gelir; **destructuring** ile almak önerilen yazımdır.
- String tırnakla, diğer her şey (sayı, boolean, dizi, nesne, fonksiyon, JSX) `{}` ile gönderilir.
- Değersiz yazılan prop `true` demektir.
- Varsayılan değerler destructuring içinde verilir; sadece `undefined`'da devreye girer. `defaultProps` React 19'da fonksiyon bileşenleri için kaldırıldı.
- **`children`**, etiketler arasındaki içeriktir; sarmalayıcı (çerçeve) bileşenlerin temelidir.
- Birden fazla yuva için JSX normal prop olarak da gönderilebilir.
- **Props salt okunurdur**; bileşen kendi props'unu asla değiştirmez.
- Veri **tek yönlü** akar: ebeveyn → çocuk.
- Çocuk, ebeveynin gönderdiği **fonksiyonu çağırarak** haber verir (`onX` / `handleX`).
- `{...nesne}` ile props açılır; `...rest` ile kalan props alt elemente aktarılır. Sıra önemlidir.
- `key` bileşene ulaşmaz; `ref` React 19'da normal prop gibi alınır.
- Props değişince bileşen yeni props ile yeniden render olur.
- **Prop drilling** için önce kompozisyon, gerekirse Context kullanılır.
- Tip kontrolü için PropTypes yerine **TypeScript** tercih edilir.

---

⬅️ Önceki: [03 · Bileşenler](./03-bilesenler.md) · ➡️ Sonraki: [05 · State ve useState](./05-state.md)
