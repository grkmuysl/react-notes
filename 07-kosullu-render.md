# 07 · Koşullu Render

> **Konu özeti:** Duruma göre farklı arayüz gösterme yöntemleri: erken `return` ve guard clause'lar, `null`, değişkene atama, ternary, `&&` ve tuzakları, koşullu nitelik/class/stil, çoklu durumlar, yükleniyor/hata/boş/başarılı kalıbı, gizleme ile kaldırma arasındaki fark ve okunabilirlik kuralları.

---

## İçindekiler

1. [Koşullu Render Nedir?](#1-koşullu-render-nedir)
2. [`if` / `else` ile Erken Return](#2-if--else-ile-erken-return)
3. [Hiçbir Şey Göstermemek: `null`](#3-hiçbir-şey-göstermemek-null)
4. [Değişkene Atama](#4-değişkene-atama)
5. [Ternary Operatörü](#5-ternary-operatörü--)
6. [Mantıksal `&&`](#6-mantıksal--)
7. [Koşullu Nitelikler, Props ve Stiller](#7-koşullu-nitelikler-props-ve-stiller)
8. [İkiden Fazla Durum](#8-ikiden-fazla-durum)
9. [Yükleniyor / Hata / Boş / Başarılı Kalıbı](#9-yükleniyor--hata--boş--başarılı-kalıbı-)
10. [Gizlemek mi, Kaldırmak mı?](#10-gizlemek-mi-kaldırmak-mı-)
11. [Okunabilirlik İçin Pratik Kurallar](#11-okunabilirlik-için-pratik-kurallar)
12. [Her Şey Bir Arada: Kullanıcı Profili](#12-her-şey-bir-arada-kullanıcı-profili)
13. [Özet](#13-özet)

---

## 1. Koşullu Render Nedir?

Uygulamalar duruma göre farklı şeyler gösterir: giriş yapan kullanıcıya profil, yapmayana giriş butonu; yüklenirken spinner, hata varsa mesaj, liste boşsa bilgi yazısı…

- React'te özel bir sözdizimi yoktur; **sıradan JavaScript** kullanılır: `if`, ternary, `&&`, `switch`.
- `UI = f(state)` formülündeki dallanmadan ibarettir.

| Araç | Türü | Nerede kullanılır |
|---|---|---|
| `if`, `switch` | Deyim | JSX'in **dışında** |
| Ternary (`? :`), `&&` | İfade | JSX'in **içinde** |

---

## 2. `if` / `else` ile Erken Return

Bileşenin **tamamen** farklı bir şey göstermesi gerektiğinde en okunaklı yöntem:

```jsx
function Panel({ kullanici }) {
  if (!kullanici) {
    return <GirisEkrani />;
  }

  return (
    <div>
      <h1>Hoş geldin, {kullanici.ad}</h1>
      <Istatistikler />
    </div>
  );
}
```

### Guard clause (koruma koşulu)

Özel durumlar baştan ele alınıp erken çıkılır; asıl içerik girintisiz ve okunaklı kalır.

```jsx
function SiparisDetay({ siparis, yukleniyor, hata }) {
  if (yukleniyor) return <Spinner />;
  if (hata) return <HataMesaji mesaj={hata} />;
  if (!siparis) return <p>Sipariş bulunamadı.</p>;

  // Buradan sonra siparis'in var olduğu kesin
  return (
    <article>
      <h2>Sipariş #{siparis.no}</h2>
      <p>Durum: {siparis.durum}</p>
    </article>
  );
}
```

> ⚠️ Hook'lar erken `return`'den **önce** çağrılır: önce tüm `useState`'ler, sonra guard clause'lar.

---

## 3. Hiçbir Şey Göstermemek: `null`

```jsx
function Bildirim({ mesaj }) {
  if (!mesaj) return null;
  return <div className="bildirim">{mesaj}</div>;
}
```

### Çoğu zaman daha iyisi: kararı ebeveyne bırakmak

```jsx
{mesaj && <Bildirim mesaj={mesaj} />}
```

- Bir bileşenin ekranda olup olmadığı, ebeveyn kodundan okunabilir hale gelir.
- Bileşenin kendini gizlemesi, ona özgü ve doğal bir davranışsa uygundur (ör. kapalı `Modal`'ın `null` döndürmesi).

---

## 4. Değişkene Atama

JSX'in bir **kısmı** koşula bağlıysa ve koşul karmaşıksa:

```jsx
function Gorev({ metin, tamam, oncelik }) {
  let icerik = metin;
  if (tamam) {
    icerik = <del>{metin} ✓</del>;
  } else if (oncelik === "yuksek") {
    icerik = <strong>🔥 {metin}</strong>;
  }

  return <li className="gorev">{icerik}</li>;
}
```

- JSX bir değer olduğu için değişkene atanabilir.
- En esnek yöntemdir; ternary iç içe girmeye başladığında buraya geçilir.

---

## 5. Ternary Operatörü (`? :`)

**İki seçenekten biri** gösterilecekse:

```jsx
function GirisButonu({ girisYapti, onGiris, onCikis }) {
  return (
    <header>
      <Logo />
      {girisYapti ? (
        <button onClick={onCikis}>Çıkış yap</button>
      ) : (
        <button onClick={onGiris}>Giriş yap</button>
      )}
    </header>
  );
}
```

- Çok satırlı dallar **parantezle** sarılır.

### Küçük metin farkları

```jsx
<p>Durum: {aktif ? "Çevrimiçi" : "Çevrimdışı"}</p>
<span>{adet} ürün{adet === 1 ? "" : "ler"}</span>
```

> 💡 "Hiçbir şey" dalı için `null`: `{kosul ? <A /> : null}` — `&&`'nin güvenli alternatifidir.

---

## 6. Mantıksal `&&` ⭐

**"Koşul doğruysa göster, değilse hiçbir şey gösterme":**

```jsx
function Gelen({ okunmamis }) {
  return (
    <div>
      <h2>Gelen kutusu</h2>
      {okunmamis.length > 0 && (
        <p>{okunmamis.length} okunmamış mesajın var.</p>
      )}
    </div>
  );
}
```

### Nasıl çalışır?

`a && b`: `a` truthy ise `b`'yi, falsy ise **`a`'nın kendisini** döndürür.

| İfade | Sonuç | Ekranda |
|---|---|---|
| `true && <p/>` | `<p/>` | Gösterilir |
| `false && <p/>` | `false` | Hiçbir şey |
| `0 && <p/>` | `0` | **`0`** ⚠️ |
| `NaN && <p/>` | `NaN` | **`NaN`** ⚠️ |

### ⚠️ Sayı tuzağı

```jsx
{okunmamis.length && <p>Yeni mesajların var</p>}
// Dizi boşsa ekrana "0" basılır
```

- Sayfada nereden geldiği belli olmayan bir **"0"** görülürse sebep neredeyse her zaman budur.
- React Native'de uygulama çöker.

### Çözüm: Sol taraf her zaman gerçek boolean

```jsx
{okunmamis.length > 0 && <p>...</p>}        // ✅ karşılaştırma (en okunaklı)
{Boolean(okunmamis.length) && <p>...</p>}   // ✅
{!!okunmamis.length && <p>...</p>}          // ✅ çift değil
{okunmamis.length ? <p>...</p> : null}      // ✅ ternary
```

### `&&` mı ternary mi?

| Durum | Tercih |
|---|---|
| Göster / gösterme | `&&` (sol taraf boolean) |
| A'yı göster / B'yi göster | Ternary |
| Sol taraf sayı veya belirsiz tip | `> 0` ile boolean'a çevir veya ternary + `null` |

---

## 7. Koşullu Nitelikler, Props ve Stiller

### className

```jsx
<button className={aktif ? "sekme aktif" : "sekme"}>...</button>
<li className={`gorev ${tamam ? "tamam" : ""} ${secili ? "secili" : ""}`}>...</li>
```

#### Çok sayıda koşullu class: dizi + `filter(Boolean)`

```jsx
const siniflar = ["gorev", tamam && "tamam", secili && "secili", hata && "hata"]
  .filter(Boolean)
  .join(" ");

<li className={siniflar}>...</li>
```

#### `clsx` kütüphanesi

```bash
npm install clsx
```

```jsx
import clsx from "clsx";

<li className={clsx("gorev", { tamam, secili, hata: !!hata })}>...</li>
```

### Inline stil

```jsx
<p style={{ color: hata ? "crimson" : "inherit", opacity: pasif ? 0.5 : 1 }}>...</p>
```

### Niteliği tamamen kaldırmak

`undefined` (veya `null`) verilen nitelik DOM'a hiç eklenmez:

```jsx
<a href={link || undefined}>...</a>              // link yoksa href yok
<input aria-invalid={hata ? true : undefined} />
<button disabled={!gecerli}>Gönder</button>      // boolean nitelik: false ise eklenmez
```

### Koşullu props aktarmak

```jsx
<Buton
  varyant={tehlikeli ? "kirmizi" : "varsayilan"}
  {...(yukleniyor && { disabled: true, "aria-busy": true })}
/>
```

- `false`'u spread etmek hiçbir şey eklemez.
- Okunabilirlik açısından çoğu zaman açık yazım daha iyidir: `disabled={yukleniyor}`.

---

## 8. İkiden Fazla Durum

### ❌ İç içe ternary'den kaçın

```jsx
{durum === "yukleniyor" ? <Spinner /> : durum === "hata" ? <Hata /> : durum === "bos" ? <Bos /> : <Liste />}
```

### Seçenek 1: Guard clause'lar

Bileşenin tamamı duruma bağlıysa en iyisi (bkz. [Bölüm 2](#2-if--else-ile-erken-return)).

### Seçenek 2: Yardımcı fonksiyon + `switch`

```jsx
function SiparisDurumu({ durum }) {
  function rozet() {
    switch (durum) {
      case "hazirlaniyor": return <span className="sari">⏳ Hazırlanıyor</span>;
      case "kargoda":      return <span className="mavi">🚚 Kargoda</span>;
      case "teslim":       return <span className="yesil">✅ Teslim edildi</span>;
      case "iptal":        return <span className="kirmizi">❌ İptal</span>;
      default:             return null;
    }
  }

  return <div className="siparis">Durum: {rozet()}</div>;
}
```

> `rozet()` bir bileşen değil, JSX döndüren sıradan bir yardımcı fonksiyondur; Hook kullanmadığı sürece çağrılması sorun değildir.

### Seçenek 3: Nesne eşleme (lookup map)

Durumlar veri gibi duruyorsa çok temizdir:

```jsx
const DURUMLAR = {
  hazirlaniyor: { ikon: "⏳", metin: "Hazırlanıyor",  renk: "sari" },
  kargoda:      { ikon: "🚚", metin: "Kargoda",       renk: "mavi" },
  teslim:       { ikon: "✅", metin: "Teslim edildi", renk: "yesil" },
  iptal:        { ikon: "❌", metin: "İptal",          renk: "kirmizi" },
};

function SiparisRozeti({ durum }) {
  const d = DURUMLAR[durum];
  if (!d) return null;
  return <span className={d.renk}>{d.ikon} {d.metin}</span>;
}
```

#### Bileşen eşleme

```jsx
const SEKMELER = {
  profil: ProfilSekmesi,
  ayarlar: AyarlarSekmesi,
  bildirimler: BildirimSekmesi,
};

function Sekmeler({ aktif }) {
  const AktifSekme = SEKMELER[aktif];   // değişken adı BÜYÜK harfle başlamalı
  return AktifSekme ? <AktifSekme /> : null;
}
```

### Seçenek 4: Ayrı bileşene çıkarmak

Koşullu kısım büyüyorsa kendi bileşenine taşınır; koşullar oranın guard clause'ları olur.

---

## 9. Yükleniyor / Hata / Boş / Başarılı Kalıbı ⭐

Veriyle çalışan her ekranın dört hali vardır:

```jsx
function UrunListesi({ durum, urunler, hata, onTekrarDene }) {
  if (durum === "yukleniyor") {
    return <p>Yükleniyor...</p>;
  }

  if (durum === "hata") {
    return (
      <div role="alert">
        <p>Bir şeyler ters gitti: {hata}</p>
        <button onClick={onTekrarDene}>Tekrar dene</button>
      </div>
    );
  }

  if (urunler.length === 0) {
    return <p>Henüz ürün yok.</p>;
  }

  return (
    <ul>
      {urunler.map((u) => <li key={u.id}>{u.ad}</li>)}
    </ul>
  );
}
```

> Boş durum unutulmamalı; boş beyaz ekran kullanıcıya bir şeylerin bozuk olduğunu düşündürür.

### Boolean'lar yerine tek bir durum değişkeni

```jsx
// ❌ İmkânsız kombinasyonlara açık
const [yukleniyor, setYukleniyor] = useState(false);
const [hata, setHata] = useState(null);
const [basarili, setBasarili] = useState(false);
```

- Üç boolean = 8 kombinasyon; çoğu **imkânsız** olmalı (`yukleniyor` ve `basarili` aynı anda?).
- Birini güncelleyip diğerini sıfırlamayı unutmak kolaydır.

```jsx
// ✅ Her an tam olarak tek bir durum
const [durum, setDurum] = useState("bosta");  // "bosta" | "yukleniyor" | "hata" | "basarili"
const [hata, setHata] = useState(null);

const yukleniyor = durum === "yukleniyor";    // gerekirse türetilir
```

> Bu yaklaşım Konu 14 (`useReducer`) ve Konu 18'de (Veri Çekme) temel olacak.

---

## 10. Gizlemek mi, Kaldırmak mı? ⭐

### Koşullu render ile kaldırmak

Bileşen ağaçtan **tamamen çıkar** (unmount); state silinir.

```jsx
{acik && <YorumFormu />}
// Formu doldur → kapat → aç: yazılanlar gitmiş
```

### CSS ile gizlemek

Bileşen ağaçta **kalır**, sadece görünmez; state korunur.

```jsx
<div style={{ display: acik ? "block" : "none" }}>
  <YorumFormu />
</div>

// veya HTML hidden niteliği
<div hidden={!acik}>
  <YorumFormu />
</div>
```

### Karşılaştırma

| | Koşullu render (`&&`, ternary) | CSS ile gizleme |
|---|---|---|
| DOM'da | Yok | Var (görünmez) |
| State | Silinir | Korunur |
| İlk yükleme maliyeti | Sadece gösterilince | Hepsi baştan render edilir |
| Ne zaman | Varsayılan tercih; içerik pahalıysa veya sıfırlanması istenirse | Sık aç/kapa yapılan ve durumu korunması gereken parçalar (sekmeler, akordeon) |

> 📌 **React 19.2 — `<Activity mode="visible" | "hidden">`:** Gizlenen kısım ekrandan kalkar ama state'i korunur, efektleri durdurulur ve arka planda düşük öncelikle güncellenebilir. *(Konu 20)*

### ⚠️ Tuzak: Aynı konumda aynı bileşen

```jsx
function SkorTablosu() {
  const [aliOynuyor, setAliOynuyor] = useState(true);

  return (
    <div>
      {aliOynuyor ? <Sayac isim="Ali" /> : <Sayac isim="Ayşe" />}
      <button onClick={() => setAliOynuyor(!aliOynuyor)}>Sıradaki oyuncu</button>
    </div>
  );
}
```

- Ali'nin sayacı 5'e getirilip oyuncu değiştirilince Ayşe'nin sayacı da **5** gösterir.
- Sebep: React state'i **konum + bileşen türü** ile eşler. İki dalda da aynı konumda aynı tür (`Sayac`) var; sadece prop değişiyor.

### Çözümler

```jsx
// 1. Farklı key → farklı bileşen sayılır, state sıfırlanır
{aliOynuyor ? <Sayac key="ali" isim="Ali" /> : <Sayac key="ayse" isim="Ayşe" />}

// 2. Farklı konumlar
{aliOynuyor && <Sayac isim="Ali" />}
{!aliOynuyor && <Sayac isim="Ayşe" />}
```

> İkinci yöntemde her `&&` ifadesi ağaçta ayrı bir "yuva" kaplar; `false` olsa bile yuvası durur. İki sayaç asla aynı konumu paylaşmaz.

---

## 11. Okunabilirlik İçin Pratik Kurallar

| Durum | Yöntem |
|---|---|
| Bileşenin **tamamı** duruma bağlı | Guard clause / erken `return` |
| **Küçük bir parça** göster/gösterme | `&&` (sol taraf boolean) |
| **İki seçenek** | Ternary (çok satırlıysa parantez) |
| **Üç ve üzeri seçenek** | `switch` / nesne eşleme / ayrı bileşen — iç içe ternary yok |
| JSX içindeki mantık karmaşık | Hesaplamayı `return`'den önce değişkene çıkar |

```jsx
const indirimVar = urun.eskiFiyat && urun.eskiFiyat > urun.fiyat;
const sonStok = urun.stok > 0 && urun.stok <= 3;

return (
  <div>
    {indirimVar && <s>{urun.eskiFiyat} ₺</s>}
    {sonStok && <span>Son {urun.stok} ürün!</span>}
  </div>
);
```

> İyi adlandırılmış bir değişken, koşulun **ne anlama geldiğini** anlatır.

---

## 12. Her Şey Bir Arada: Kullanıcı Profili

```jsx
import { useState } from "react";
import clsx from "clsx"; // npm install clsx

const ROLLER = {
  admin:  { etiket: "Yönetici", ikon: "🛡️" },
  editor: { etiket: "Editör",   ikon: "✏️" },
  uye:    { etiket: "Üye",      ikon: "👤" },
};

function RolRozeti({ rol }) {
  const r = ROLLER[rol];
  if (!r) return null;
  return <span className="rozet">{r.ikon} {r.etiket}</span>;
}

function Profil({ kullanici }) {
  const [detayAcik, setDetayAcik] = useState(false);

  const dogrulanmis = kullanici.emailDogrulandi;
  const takipciVar = kullanici.takipci > 0;

  return (
    <article className={clsx("profil", { cevrimici: kullanici.cevrimici })}>
      <h2>
        {kullanici.ad} <RolRozeti rol={kullanici.rol} />
      </h2>

      <p>{kullanici.cevrimici ? "🟢 Çevrimiçi" : "⚪ Çevrimdışı"}</p>

      {!dogrulanmis && (
        <p className="uyari" role="alert">
          E-posta adresin doğrulanmamış.
        </p>
      )}

      {takipciVar && <p>{kullanici.takipci} takipçi</p>}

      <button type="button" onClick={() => setDetayAcik(!detayAcik)}>
        {detayAcik ? "Detayları gizle ▲" : "Detayları göster ▼"}
      </button>

      {/* CSS ile gizleme: içerik ağaçta kalır */}
      <div hidden={!detayAcik}>
        <p>Şehir: {kullanici.sehir ?? "Belirtilmemiş"}</p>
        <p>Üyelik: {kullanici.uyelikYili}</p>
      </div>

      {kullanici.rol === "admin" && (
        <button type="button" className="tehlike">Yönetim paneli</button>
      )}
    </article>
  );
}

function ProfilSayfasi({ durum, kullanici, hata, onTekrarDene }) {
  if (durum === "yukleniyor") return <p>Profil yükleniyor...</p>;

  if (durum === "hata") {
    return (
      <div role="alert">
        <p>Profil yüklenemedi: {hata}</p>
        <button onClick={onTekrarDene}>Tekrar dene</button>
      </div>
    );
  }

  if (!kullanici) return <p>Kullanıcı bulunamadı.</p>;

  return <Profil kullanici={kullanici} />;
}

const ORNEK = {
  ad: "Ayşe Yılmaz",
  rol: "admin",
  cevrimici: true,
  emailDogrulandi: false,
  takipci: 0,
  sehir: null,
  uyelikYili: 2023,
};

export default function App() {
  const [durum, setDurum] = useState("basarili"); // "yukleniyor" | "hata" | "basarili"

  return (
    <main>
      <div>
        {["yukleniyor", "hata", "basarili"].map((d) => (
          <button key={d} onClick={() => setDurum(d)} disabled={durum === d}>
            {d}
          </button>
        ))}
      </div>

      <ProfilSayfasi
        durum={durum}
        kullanici={ORNEK}
        hata="Sunucuya ulaşılamadı"
        onTekrarDene={() => setDurum("basarili")}
      />
    </main>
  );
}
```

### Bu örnekte kullanılanlar

- Guard clause'larla yükleniyor / hata / boş durumları
- Tek bir `durum` değişkeni
- Nesne eşleme (`ROLLER`) ve `null` döndüren bileşen
- Ternary (çevrimiçi durumu, buton metni)
- Boolean'a çevrilmiş `&&` (`takipciVar`)
- `??` ile varsayılan metin
- `clsx` ile koşullu class
- `hidden` ile state'i koruyarak gizleme
- Role göre içerik gösterme

---

## 13. Özet

- Koşullu render için özel sözdizimi yoktur; **sıradan JavaScript** kullanılır.
- Deyimler (`if`, `switch`) JSX dışında, ifadeler (ternary, `&&`) JSX içinde kullanılır.
- Bileşenin tamamı duruma bağlıysa **guard clause / erken `return`**; Hook'lar return'lerden önce.
- Hiçbir şey göstermemek için `null`; çoğu zaman göster/gösterme kararı ebeveyne bırakılır.
- Karmaşık koşullarda JSX önceden **değişkene atanır.**
- İki seçenek için **ternary**; "hiçbir şey" dalı `null`.
- **`&&`'de sol taraf boolean olmalı:** `0` ve `NaN` ekrana basılır → `length > 0 &&`.
- Koşullu class için template literal, `filter(Boolean)` veya **`clsx`**; `undefined` verilen nitelik DOM'a eklenmez.
- Üç ve üzeri durumda iç içe ternary yerine `switch`, **nesne eşleme** veya ayrı bileşen.
- Veriyle çalışan ekranlarda **yükleniyor / hata / boş / başarılı** durumlarının hepsi ele alınır.
- Birden çok boolean yerine **tek bir durum değişkeni** imkânsız kombinasyonları önler.
- Koşullu render bileşeni **kaldırır (state silinir)**; CSS ile gizlemek **state'i korur.** React 19.2'de `<Activity>` ara bir yol sunar.
- Aynı konumda aynı türde bileşen state'ini paylaşır; farklı **`key`** veya farklı konum ile ayrılır.
- Karmaşık koşullar iyi adlandırılmış değişkenlere çıkarılır.

---

⬅️ Önceki: [06 · Olay Yönetimi](./06-olay-yonetimi.md) · ➡️ Sonraki: [08 · Listeler ve key](./08-listeler-ve-key.md)
