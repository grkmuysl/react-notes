# 06 · Olay Yönetimi (Events)

> **Konu özeti:** React'te olaylara tepki verme, handler tanımlama ve argüman geçirme, fonksiyonu vermek ile çağırmak arasındaki fark, olay nesnesi, sık kullanılan olaylar, `onChange` davranışı, `preventDefault`, olay yayılımı ve `stopPropagation`, kendi bileşenlerinde olay props'ları, klavye olayları, erişilebilirlik ve React'in olay sistemi.

---

## İçindekiler

1. [Temeller](#1-temeller)
2. [Fonksiyonu Vermek vs Çağırmak](#2-️-fonksiyonu-vermek-vs-çağırmak)
3. [Handler Tanımlama Yöntemleri](#3-handler-tanımlama-yöntemleri)
4. [Olay Yöneticileri Yan Etki İçerebilir](#4-olay-yöneticileri-yan-etki-içerebilir)
5. [Olay Nesnesi (Event Object)](#5-olay-nesnesi-event-object)
6. [Sık Kullanılan Olaylar](#6-sık-kullanılan-olaylar)
7. [`onChange` React'te Farklı Çalışır](#7-onchange-reactte-farklı-çalışır-)
8. [Varsayılan Davranışı Engellemek](#8-varsayılan-davranışı-engellemek-preventdefault)
9. [Olay Yayılımı (Propagation)](#9-olay-yayılımı-propagation--bubbling-)
10. [Kendi Bileşenlerinde Olay Props'ları](#10-kendi-bileşenlerinde-olay-propsları)
11. [Klavye Olayları](#11-klavye-olayları)
12. [Erişilebilirlik: Doğru Elementi Kullan](#12-erişilebilirlik-doğru-elementi-kullan)
13. [Arka Planda: React Olay Sistemi](#13-arka-planda-react-olay-sistemi)
14. [Her Şey Bir Arada: Mağaza ve Sepet](#14-her-şey-bir-arada-mağaza-ve-sepet)
15. [Özet](#15-özet)

---

## 1. Temeller

**Olay (event):** Kullanıcının veya tarayıcının yaptığı bir şey — tıklama, tuşa basma, yazı yazma, form gönderme, fareyi üzerine getirme…

React'te olaylara, JSX içinde elemente bir **olay yöneticisi (event handler)** fonksiyonu verilerek tepki verilir.

```jsx
function Buton() {
  function handleClick() {
    alert("Tıklandın!");
  }

  return <button onClick={handleClick}>Tıkla</button>;
}
```

### HTML ile farklar

| HTML | React |
|---|---|
| `onclick="handleClick()"` | `onClick={handleClick}` |
| Küçük harf | **camelCase** (`onClick`, `onMouseEnter`, `onKeyDown`) |
| Değer **string** (çalıştırılacak kod) | Değer **fonksiyonun kendisi** |
| `return false` ile varsayılan davranış engellenir | `e.preventDefault()` açıkça çağrılır |

---

## 2. ⚠️ Fonksiyonu Vermek vs Çağırmak

```jsx
<button onClick={handleClick}>     // ✅ fonksiyonu VER: tıklanınca React çağırır
<button onClick={handleClick()}>   // ❌ fonksiyonu ÇAĞIR: render sırasında hemen çalışır
```

- İkinci satırda fonksiyon render sırasında çalışır, **dönüş değeri** (`undefined`) `onClick`'e verilir.
- Sonuç: Sayfa açılınca alert çıkar, tıklayınca hiçbir şey olmaz.

### State değiştiren handler'da

```jsx
<button onClick={setSayi(sayi + 1)}>   // ❌
```

Render → `setSayi` → yeni render → `setSayi` → … → **"Too many re-renders"** hatası.

### Argüman gerekiyorsa arrow function ile sar

```jsx
<button onClick={() => setSayi(sayi + 1)}>   // ✅
<button onClick={() => handleSil(urun.id)}>  // ✅
```

| Yazım | Anlamı | Doğru mu? |
|---|---|---|
| `onClick={handleClick}` | Fonksiyonu ver | ✅ |
| `onClick={() => handleClick()}` | Fonksiyonu çağıran fonksiyon ver | ✅ |
| `onClick={() => handleSil(id)}` | Argümanla çağıran fonksiyon ver | ✅ |
| `onClick={handleClick()}` | Render'da çağır, sonucu ver | ❌ |
| `onClick={handleSil(id)}` | Render'da çağır, sonucu ver | ❌ |

---

## 3. Handler Tanımlama Yöntemleri

### 1. Adlandırılmış fonksiyon (birden fazla satırlık işler için)

```jsx
function SepeteEkle({ urun }) {
  const [adet, setAdet] = useState(1);

  function handleEkle() {
    console.log(`${adet} adet ${urun.ad} eklendi`);
    setAdet(1);
  }

  return <button onClick={handleEkle}>Sepete Ekle</button>;
}
```

### 2. Inline arrow function (kısa, tek satırlık işler için)

```jsx
<button onClick={() => setAcik(!acik)}>Aç / Kapat</button>
```

### 3. Arrow function olarak tanımlanmış handler

```jsx
const handleEkle = () => { /* ... */ };
```

- Hepsi geçerlidir.
- Handler'lar **bileşenin içinde** tanımlanır → props ve state'e erişebilir (closure).

> 💡 Inline fonksiyonların her render'da yeniden oluşması neredeyse her durumda önemsizdir. Nadir optimizasyon gerektiren durumlar: `useCallback` (Konu 19).

---

## 4. Olay Yöneticileri Yan Etki İçerebilir

Render saf olmalıdır (Konu 3.6), ama olay yöneticileri render sırasında değil, **kullanıcı bir şey yaptığında** çalışır. Bu yüzden saf olmaları gerekmez; **yan etkilerin asıl yeri** burasıdır.

```jsx
async function handleKaydet() {
  setKaydediliyor(true);                                   // state değiştir
  await fetch("/api/profil", {                             // API isteği
    method: "POST",
    body: JSON.stringify(form),
  });
  localStorage.setItem("sonKayit", Date.now());            // tarayıcı depolaması
  document.title = "Kaydedildi ✓";                         // DOM'a dokun
  setKaydediliyor(false);
}
```

> Yan etki bir olaya bağlanabiliyorsa (tıklandığında kaydet, gönderildiğinde istek at) **doğru yer olay yöneticisidir, `useEffect` değil.** *(Konu 10'da önemli olacak.)*

---

## 5. Olay Nesnesi (Event Object)

React handler'a otomatik olarak bir **olay nesnesi** verir (genelde `e` veya `event`).

```jsx
function handleClick(e) {
  console.log(e.type);                // "click"
  console.log(e.target);              // tıklanan element
  console.log(e.clientX, e.clientY);  // fare konumu
}

<button onClick={handleClick}>Tıkla</button>
```

### SyntheticEvent

- React'in tarayıcının yerel olay nesnesini sardığı, **tüm tarayıcılarda aynı davranan** sarmalayıcı.
- Standart DOM olay API'siyle aynı özellik ve metotlara sahiptir.
- Yerel olaya `e.nativeEvent` ile ulaşılır.

### En çok kullanılan özellik ve metotlar

| Özellik / Metot | Ne verir |
|---|---|
| `e.target` | Olayın **başladığı** element (en içteki) |
| `e.currentTarget` | Handler'ın **bağlı olduğu** element |
| `e.target.value` | Input'un güncel değeri |
| `e.target.checked` | Checkbox'ın işaretli olup olmadığı |
| `e.target.name` | Elementin `name` niteliği |
| `e.key` | Basılan tuş (`"Enter"`, `"Escape"`, `"a"`) |
| `e.shiftKey`, `e.ctrlKey`, `e.metaKey`, `e.altKey` | Değiştirici tuşlar basılı mı |
| `e.preventDefault()` | Tarayıcının varsayılan davranışını engeller |
| `e.stopPropagation()` | Olayın yukarı yayılmasını durdurur |

### `target` vs `currentTarget`

```jsx
<div onClick={(e) => {
  console.log(e.target);        // <span> → tıklanan en içteki element
  console.log(e.currentTarget); // <div>  → handler'ın bağlı olduğu element
}}>
  <span>Bana tıkla</span>
</div>
```

### Olay nesnesi + kendi argümanın

```jsx
<input onChange={(e) => handleAlanDegis("email", e.target.value)} />

function handleAlanDegis(alan, deger) {
  setForm({ ...form, [alan]: deger });
}
```

> 📜 **Event pooling:** React 16 ve öncesinde olay nesnesi handler bitince sıfırlanıyordu, async kodda `e.persist()` gerekiyordu. React 17'de kaldırıldı; `e.persist()` artık gereksiz.

---

## 6. Sık Kullanılan Olaylar

| Kategori | Olaylar | Notlar |
|---|---|---|
| **Fare / tıklama** | `onClick`, `onDoubleClick`, `onContextMenu` | `onContextMenu` = sağ tık |
| | `onMouseEnter`, `onMouseLeave` | Yayılmaz; hover için ideal |
| | `onMouseOver`, `onMouseOut` | Yayılır; çocuklara girip çıkınca da tetiklenir |
| | `onMouseDown`, `onMouseUp`, `onMouseMove` | |
| **Pointer** | `onPointerDown`, `onPointerMove`, `onPointerUp` | Fare, dokunmatik ve kalem tek API'de |
| **Klavye** | `onKeyDown`, `onKeyUp` | `onKeyPress` kullanımdan kalktı |
| **Form** | `onChange` | Input, select, textarea, checkbox |
| | `onSubmit` | `<form>` üzerinde |
| | `onInput`, `onReset`, `onInvalid` | |
| **Odak** | `onFocus`, `onBlur` | React'te yayılırlar (DOM'daki `focusin`/`focusout` gibi) |
| **Kaydırma** | `onScroll`, `onWheel` | `onScroll` React 17'den beri yayılmaz |
| **Pano** | `onCopy`, `onPaste`, `onCut` | |
| **Sürükle-bırak** | `onDragStart`, `onDragOver`, `onDrop`, … | |
| **Medya** | `onLoad`, `onError` | `<img>`, `<video>` vb. |
| **Animasyon** | `onAnimationEnd`, `onTransitionEnd` | CSS animasyonları |

---

## 7. `onChange` React'te Farklı Çalışır ⭐

| | Tetiklenme anı |
|---|---|
| Saf DOM `change` olayı | Input **odaktan çıkınca** |
| React `onChange` | **Her tuş vuruşunda** (DOM'daki `input` olayı gibi) |

Bilinçli bir tercihtir: değer anında state'e yansıtılabilir.

```jsx
function AramaKutusu() {
  const [arama, setArama] = useState("");

  return (
    <>
      <input
        value={arama}
        onChange={(e) => setArama(e.target.value)}
        placeholder="Ara..."
      />
      <p>Aranıyor: {arama}</p>   {/* her harfte güncellenir */}
    </>
  );
}
```

- `value` + `onChange` ile değeri React state'inin kontrol ettiği input → **controlled input** *(Konu 9)*.

### Checkbox'ta `checked` okunur

```jsx
<input
  type="checkbox"
  checked={kabul}
  onChange={(e) => setKabul(e.target.checked)}
/>
```

---

## 8. Varsayılan Davranışı Engellemek: `preventDefault`

### Varsayılan davranışı olan elementler

| Element / Olay | Varsayılan davranış |
|---|---|
| `<form>` gönderimi | Sayfa **yenilenir** |
| `<a href>` tıklaması | Sayfaya **gider** |
| Sağ tık | Tarayıcı menüsü açılır |

### Form örneği

```jsx
function GirisFormu() {
  const [email, setEmail] = useState("");

  function handleSubmit(e) {
    e.preventDefault();          // sayfa yenilenmesini engelle
    console.log("Giriş:", email);
  }

  return (
    <form onSubmit={handleSubmit}>
      <input value={email} onChange={(e) => setEmail(e.target.value)} />
      <button type="submit">Giriş yap</button>
    </form>
  );
}
```

- `preventDefault` unutulursa sayfa yenilenir ve **tüm state kaybolur.**
- Form, butonun `onClick`'iyle değil **`onSubmit`** ile dinlenir → Enter ile gönderim de otomatik çalışır.

> 📌 React 19'da `<form action={fonksiyon}>` yöntemi de var; varsayılan davranışı React yönetir *(Konu 20)*. `onSubmit` + `preventDefault` hâlâ geçerli ve yaygındır.

### Sağ tık menüsünü özelleştirme

```jsx
<div onContextMenu={(e) => { e.preventDefault(); setMenuAcik(true); }}>
  ...
</div>
```

---

## 9. Olay Yayılımı (Propagation / Bubbling) ⭐

Olay sadece tıklanan elementte kalmaz; **ağaçta yukarı doğru, tüm ebeveynlere yayılır.**

```jsx
function AracCubugu() {
  return (
    <div onClick={() => alert("Araç çubuğuna tıklandı")}>
      <button onClick={() => alert("Oynat")}>▶️ Oynat</button>
      <button onClick={() => alert("Durdur")}>⏸️ Durdur</button>
    </div>
  );
}
```

"Oynat"a tıklanınca: önce **"Oynat"**, sonra **"Araç çubuğuna tıklandı"**.

```
     div  ← 2. yayılır, buradaki handler da çalışır
      ▲
   button ← 1. tıklanan, önce bu çalışır
```

### Yayılımı durdurmak: `stopPropagation`

```jsx
<button onClick={(e) => {
  e.stopPropagation();   // olay buradan yukarı gitmez
  alert("Oynat");
}}>
```

### Gerçek hayat örneği: Modal

```jsx
function Modal({ onKapat, children }) {
  return (
    <div className="arkaplan" onClick={onKapat}>                     {/* dışarı tıklayınca kapan */}
      <div className="modal" onClick={(e) => e.stopPropagation()}>  {/* içeride tıklama kapatmasın */}
        {children}
      </div>
    </div>
  );
}
```

### Alternatif: Olayın kaynağını kontrol etmek

```jsx
<div className="arkaplan" onClick={(e) => {
  if (e.target === e.currentTarget) onKapat();  // sadece arka plana doğrudan tıklandıysa
}}>
```

> Genelde daha temizdir; `stopPropagation` başka yerlerde dinlenen olayları (analytics, dışarı tıklama dinleyicileri) da kesebilir.

### `stopPropagation` vs `preventDefault`

| Metot | Durdurduğu şey |
|---|---|
| `stopPropagation()` | Olayın **yukarı yayılmasını** |
| `preventDefault()` | Tarayıcının **varsayılan davranışını** |

Biri diğerini yapmaz.

### Yakalama (capture) aşaması

Olaylar önce yukarıdan aşağı iner (**capture**), sonra aşağıdan yukarı çıkar (**bubble**). Capture aşamasında dinlemek için olay adına `Capture` eklenir:

```jsx
<div onClickCapture={() => console.log("Çocuklardan ÖNCE çalışırım")}>
```

- Çocuk `stopPropagation` yapsa bile çalışır.
- Loglama ve analytics için kullanılır; nadiren gerekir.

---

## 10. Kendi Bileşenlerinde Olay Props'ları

| Element türü | `onClick` gibi adlar |
|---|---|
| **HTML elementleri** (`<button>`, `<input>`) | Gerçek tarayıcı olaylarına bağlanır |
| **Kendi bileşenlerin** (`<Buton>`, `<Oynatici>`) | Sıradan prop adıdır; bileşen onu bir HTML elementine iletmeli veya kendisi çağırmalıdır |

```jsx
function Buton({ onClick, children }) {
  return <button onClick={onClick}>{children}</button>;   // ileterek anlam kazandırılır
}
```

### Uygulamaya özgü olay adları

```jsx
function Oynatici({ onOynat, onDurdur }) {
  return (
    <div>
      <button onClick={onOynat}>▶️</button>
      <button onClick={onDurdur}>⏸️</button>
    </div>
  );
}

function App() {
  return (
    <Oynatici
      onOynat={() => console.log("Video başladı")}
      onDurdur={() => console.log("Video durdu")}
    />
  );
}
```

- `onOynat` tarayıcının bildiği bir olay değil, bileşenin tanımladığı bir sözleşmedir.
- Gelenek: prop `onX`, handler `handleX`.

### Olayla birlikte veri iletmek

Ebeveyn olay nesnesiyle uğraşmak zorunda kalmaz:

```jsx
function AramaKutusu({ onAra }) {
  const [metin, setMetin] = useState("");

  function handleSubmit(e) {
    e.preventDefault();
    onAra(metin.trim());     // ebeveyne sadece temiz veri iletilir
  }

  return (
    <form onSubmit={handleSubmit}>
      <input value={metin} onChange={(e) => setMetin(e.target.value)} />
      <button>Ara</button>
    </form>
  );
}

<AramaKutusu onAra={(sorgu) => setSonuclar(ara(sorgu))} />
```

### Yayılım yerine handler'ı açıkça çağırmak

```jsx
function TakipButonu({ onClick }) {
  return (
    <button onClick={(e) => {
      e.stopPropagation();
      console.log("Takip butonu tıklandı");  // kendi işi
      onClick?.();                           // ebeveyninkini de çağır (varsa)
    }}>
      Takip et
    </button>
  );
}
```

> Örtük yayılıma güvenmek yerine akış kodda görünür kılınır.

---

## 11. Klavye Olayları

```jsx
function MesajKutusu({ onGonder }) {
  const [mesaj, setMesaj] = useState("");

  function handleKeyDown(e) {
    if (e.key === "Enter" && !e.shiftKey) {   // Enter gönderir, Shift+Enter yeni satır
      e.preventDefault();
      onGonder(mesaj);
      setMesaj("");
    }
    if (e.key === "Escape") {
      setMesaj("");
    }
  }

  return (
    <textarea
      value={mesaj}
      onChange={(e) => setMesaj(e.target.value)}
      onKeyDown={handleKeyDown}
      placeholder="Mesaj yaz (Enter: gönder, Shift+Enter: yeni satır)"
    />
  );
}
```

- `e.key` okunabilir tuş adı verir: `"Enter"`, `"Escape"`, `"ArrowUp"`, `" "` (boşluk), `"a"`.
- 📜 `e.keyCode === 13` kullanımdan kalktı; `e.key` kullanılır.
- IME / otomatik tamamlama sırasındaki Enter'ı ayırt etmek için `e.nativeEvent.isComposing` kontrol edilebilir.

> Pencerenin tamamında kısayol dinlemek (ör. her yerde `Ctrl+K`) JSX olaylarıyla değil, `useEffect` içinde `window.addEventListener` ile yapılır *(Konu 10)*.

---

## 12. Erişilebilirlik: Doğru Elementi Kullan

```jsx
// ❌ Tıklanabilir div
<div onClick={handleSil}>Sil</div>

// ✅ Gerçek buton
<button type="button" onClick={handleSil}>Sil</button>
```

### `<div onClick>` sorunları

- Klavyeyle (Tab) odaklanılamaz, Enter/Space ile tetiklenemez.
- Ekran okuyucular onu buton olarak tanımaz.

`<button>` bunların hepsini kendiliğinden sağlar.

| Davranış | Element |
|---|---|
| Tıklanınca **bir şey yapar** | `<button>` |
| Tıklanınca **bir yere gider** | `<a>` (veya router'ın `Link`'i) |

### Mecburen div kullanılırsa

```jsx
<div
  role="button"
  tabIndex={0}
  onClick={handleSil}
  onKeyDown={(e) => {
    if (e.key === "Enter" || e.key === " ") handleSil();
  }}
>
  Sil
</div>
```

> Zahmetlidir; `<button>` kullanıp CSS ile görünümünü değiştirmek neredeyse her zaman daha iyidir.

> ⚠️ `<form>` içindeki `<button>`'ın varsayılan tipi **`submit`**'tir. Formu göndermemesi gereken butonlara `type="button"` yazılır.

---

## 13. Arka Planda: React Olay Sistemi

- React her elemente ayrı `addEventListener` eklemez.
- Olayları **kök elemente (`#root`) tek bir dinleyici** ile bağlar; olay gelince hangi bileşenin handler'ını çağıracağını kendisi bulur → **event delegation**.
- Binlerce elementli listelerde bile dinleyici sayısı sabit kalır.
- 📜 React 17 öncesinde dinleyiciler `document`'a bağlanıyordu; 17'den beri kök elemente bağlanır. Aynı sayfada birden fazla React sürümü veya React dışı kodla çakışma azaldı.

---

## 14. Her Şey Bir Arada: Mağaza ve Sepet

```jsx
import { useState } from "react";

const URUNLER = [
  { id: 1, ad: "Mekanik Klavye", fiyat: 2499 },
  { id: 2, ad: "Kablosuz Mouse", fiyat: 899 },
  { id: 3, ad: "USB-C Hub", fiyat: 649 },
];

function AdetSecici({ adet, onDegis }) {
  return (
    <div className="adet" onClick={(e) => e.stopPropagation()}>
      <button type="button" onClick={() => onDegis(Math.max(1, adet - 1))}>−</button>
      <span>{adet}</span>
      <button type="button" onClick={() => onDegis(adet + 1)}>+</button>
    </div>
  );
}

function UrunKarti({ urun, secili, onSec, onSepeteEkle }) {
  const [adet, setAdet] = useState(1);

  function handleEkle(e) {
    e.stopPropagation();          // kartın onSec'ini tetiklemesin
    onSepeteEkle(urun, adet);
    setAdet(1);
  }

  return (
    <div
      className={`kart ${secili ? "secili" : ""}`}
      onClick={() => onSec(urun.id)}
    >
      <h3>{urun.ad}</h3>
      <p>{urun.fiyat.toLocaleString("tr-TR")} ₺</p>
      <AdetSecici adet={adet} onDegis={setAdet} />
      <button type="button" onClick={handleEkle}>Sepete ekle</button>
    </div>
  );
}

function KuponFormu({ onUygula }) {
  const [kod, setKod] = useState("");

  function handleSubmit(e) {
    e.preventDefault();
    if (!kod.trim()) return;
    onUygula(kod.trim().toUpperCase());
    setKod("");
  }

  return (
    <form onSubmit={handleSubmit}>
      <input
        value={kod}
        onChange={(e) => setKod(e.target.value)}
        onKeyDown={(e) => e.key === "Escape" && setKod("")}
        placeholder="Kupon kodu"
      />
      <button type="submit">Uygula</button>
    </form>
  );
}

export default function App() {
  const [seciliId, setSeciliId] = useState(null);
  const [sepet, setSepet] = useState([]);        // [{ id, ad, fiyat, adet }]
  const [kupon, setKupon] = useState(null);

  const toplam = sepet.reduce((t, s) => t + s.fiyat * s.adet, 0);
  const indirimli = kupon === "REACT10" ? toplam * 0.9 : toplam;

  function handleSepeteEkle(urun, adet) {
    setSepet((onceki) => {
      const mevcut = onceki.find((s) => s.id === urun.id);
      if (mevcut) {
        return onceki.map((s) =>
          s.id === urun.id ? { ...s, adet: s.adet + adet } : s
        );
      }
      return [...onceki, { ...urun, adet }];
    });
  }

  function handleKuponUygula(kod) {
    if (kod === "REACT10") setKupon(kod);
    else alert("Geçersiz kupon");
  }

  return (
    <main>
      <h1>Mağaza</h1>
      <div className="urunler">
        {URUNLER.map((u) => (
          <UrunKarti
            key={u.id}
            urun={u}
            secili={u.id === seciliId}
            onSec={setSeciliId}
            onSepeteEkle={handleSepeteEkle}
          />
        ))}
      </div>

      <h2>Sepet ({sepet.length})</h2>
      <ul>
        {sepet.map((s) => (
          <li key={s.id}>
            {s.ad} × {s.adet}
            <button
              type="button"
              onClick={() => setSepet(sepet.filter((x) => x.id !== s.id))}
            >
              🗑️
            </button>
          </li>
        ))}
      </ul>

      <KuponFormu onUygula={handleKuponUygula} />
      <p>
        Toplam: <strong>{indirimli.toLocaleString("tr-TR")} ₺</strong>
        {kupon && <span> (%10 indirim uygulandı)</span>}
      </p>
    </main>
  );
}
```

### Bu örnekte kullanılanlar

- Adlandırılmış ve inline handler'lar
- Argümanla handler çağırma
- Olay nesnesi
- `stopPropagation` (kart içindeki butonlar kartı seçmesin)
- `preventDefault` ile form gönderimi
- `onKeyDown` ile Escape tuşu
- `type="button"` / `type="submit"` ayrımı
- Kendi olay props'ları (`onSec`, `onSepeteEkle`, `onDegis`, `onUygula`)
- Olayların state güncellemesiyle birleşimi

> Not: Kartın tamamının tıklanabilir `div` olması örneği kısa tutmak içindir; gerçek projede erişilebilirlik için kartın içine bir `<button>` (ör. "Seç") koymak veya Bölüm 12'deki eksikleri tamamlamak gerekir.

---

## 15. Özet

- Olaylar JSX'te **camelCase** prop'larla (`onClick`, `onChange`) dinlenir; değer **fonksiyonun kendisidir.**
- `onClick={fn}` ✅ fonksiyonu verir; `onClick={fn()}` ❌ render sırasında çağırır.
- Argüman gerekiyorsa arrow function ile sarılır: `onClick={() => handleSil(id)}`.
- Handler'lar bileşen içinde tanımlanır; props ve state'e erişir.
- Olay yöneticileri saf olmak zorunda değildir; **yan etkilerin asıl yeri** burasıdır.
- Handler'a **SyntheticEvent** gelir: `e.target`, `e.currentTarget`, `e.key`, `e.target.value`, `e.target.checked`.
- `e.target` olayın başladığı, `e.currentTarget` handler'ın bağlı olduğu elementtir.
- React'te `onChange` **her tuş vuruşunda** tetiklenir.
- `e.preventDefault()` varsayılan davranışı (form yenilemesi, link gidişi) engeller; formlar `onSubmit` ile dinlenir.
- Olaylar ağaçta **yukarı yayılır**; `e.stopPropagation()` ile durdurulur veya `e.target === e.currentTarget` ile kaynak kontrol edilir.
- `onXCapture` ile olay capture aşamasında dinlenir.
- Kendi bileşenlerinde `onX` adlı props uygulamaya özgü olay sözleşmeleri tanımlar; veriyi temizleyip ebeveyne iletmek iyi bir kalıptır.
- Klavyede `e.key` kullanılır; `keyCode` ve `onKeyPress` kullanımdan kalktı.
- Tıklanabilir öğeler için `<div>` değil **`<button>`**; form içindeki butonlarda `type` belirtilir.
- React olayları kök elemente bağlı tek dinleyiciyle yönetir (**event delegation**).

---

⬅️ Önceki: [05 · State ve useState](./05-state.md) · ➡️ Sonraki: [07 · Koşullu Render](./07-kosullu-render.md)
