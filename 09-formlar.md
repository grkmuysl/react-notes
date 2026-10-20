# 09 · Formlar

> **Konu özeti:** Controlled ve uncontrolled yaklaşımlar, tüm input türlerinin React'te kullanımı, çok alanlı formlarda tek handler, etiketler ve `useId`, form gönderimi, doğrulama ve hataların doğru zamanda / erişilebilir gösterilmesi, `FormData`, React 19 form actions'a giriş ve form kütüphaneleri.

---

## İçindekiler

1. [İki Yaklaşım: Controlled ve Uncontrolled](#1-iki-yaklaşım-controlled-ve-uncontrolled)
2. [Controlled Input](#2-controlled-input-)
3. [Input Türleri](#3-input-türleri)
4. [Çok Alanlı Formlar: Tek Handler](#4-çok-alanlı-formlar-tek-handler)
5. [Etiketler, Erişilebilirlik ve `useId`](#5-etiketler-erişilebilirlik-ve-useid)
6. [Formu Göndermek](#6-formu-göndermek)
7. [Doğrulama (Validation)](#7-doğrulama-validation-)
8. [Uncontrolled Formlar ve `FormData`](#8-uncontrolled-formlar-ve-formdata)
9. [React 19: Form Actions'a İlk Bakış](#9-react-19-form-actionsa-ilk-bakış)
10. [Form Kütüphaneleri](#10-form-kütüphaneleri)
11. [Her Şey Bir Arada: Kayıt Formu](#11-her-şey-bir-arada-kayıt-formu)
12. [Özet](#12-özet)

---

## 1. İki Yaklaşım: Controlled ve Uncontrolled

HTML'de form elemanları değerlerini **kendileri** (DOM'da) tutar. React'te bu değeri kimin yöneteceğine göre iki yaklaşım vardır:

| | **Controlled** (kontrollü) | **Uncontrolled** (kontrolsüz) |
|---|---|---|
| Değeri kim tutar | React state'i | DOM (tarayıcı) |
| Değere erişim | Her an state'ten | Gerektiğinde DOM'dan (gönderimde `FormData` veya ref) |
| Başlangıç değeri | `value` / `checked` | `defaultValue` / `defaultChecked` |
| Her tuşta render | Evet | Hayır |
| Ne zaman | Anlık doğrulama, alanlar arası bağımlılık, girdiyi dönüştürme, koşullu alanlar | Basit formlar, değer sadece gönderimde lazımsa, dosya input'u |

---

## 2. Controlled Input ⭐

```jsx
function AdFormu() {
  const [ad, setAd] = useState("");

  return (
    <>
      <input value={ad} onChange={(e) => setAd(e.target.value)} />
      <p>Merhaba, {ad || "yabancı"}!</p>
    </>
  );
}
```

### Döngü

```
Kullanıcı "A" yazar
  → onChange tetiklenir → setAd("A")
  → yeniden render → <input value="A" />
  → React DOM'u state ile eşitler
```

> Input'un gösterdiği değer **her zaman state'in değeridir** → **tek doğruluk kaynağı (single source of truth).**

### Girdiyi dönüştürmek / sınırlamak

```jsx
// Sadece rakam, en fazla 11 hane
<input
  value={tcNo}
  onChange={(e) => setTcNo(e.target.value.replace(/\D/g, "").slice(0, 11))}
  inputMode="numeric"
/>

// Her zaman büyük harf (Türkçe uyumlu)
<input
  value={plaka}
  onChange={(e) => setPlaka(e.target.value.toLocaleUpperCase("tr"))}
/>
```

State'e yazılmayan karakter ekranda da görünmez.

### ⚠️ Uyarı 1: `value` var, `onChange` yok

```jsx
<input value={ad} />
```

> *Warning: You provided a `value` prop to a form field without an `onChange` handler…*

Input donar. Çözümler:

| Niyet | Çözüm |
|---|---|
| Kullanıcı yazabilsin | `onChange` ekle |
| Sadece gösterilsin | `readOnly` |
| Sadece başlangıç değeri | `defaultValue` |

### ⚠️ Uyarı 2: Controlled ↔ uncontrolled geçişi

```jsx
const [ad, setAd] = useState();   // undefined!
<input value={ad} onChange={...} />
```

> *Warning: A component is changing an uncontrolled input to be controlled…*

- `value={undefined}` veya `value={null}` → React'e göre "kontrolsüz".
- Çözüm: Metin alanlarında başlangıç değeri **her zaman boş string**.

```jsx
const [ad, setAd] = useState("");                     // ✅
<input value={kullanici.ad ?? ""} onChange={...} />   // ✅ veri null olabiliyorsa
```

---

## 3. Input Türleri

### Metin, e-posta, şifre

```jsx
<input type="text" value={ad} onChange={(e) => setAd(e.target.value)} />
<input type="email" value={email} onChange={(e) => setEmail(e.target.value)} />
<input type="password" value={sifre} onChange={(e) => setSifre(e.target.value)} />
```

### Sayı

> ⚠️ `type="number"` olsa bile `e.target.value` **her zaman string**dir.

```jsx
onChange={(e) => setAdet(e.target.value)}          // ❌ "3" → adet + 1 = "31"
onChange={(e) => setAdet(Number(e.target.value))}  // ✅
onChange={(e) => setAdet(e.target.valueAsNumber)}  // ✅ (boşsa NaN)
```

**Sorun:** Alan silinince `Number("")` → `0`; input'a anında "0" yazılır, silmek imkânsızlaşır.

**Önerilen kalıp:** State'te string tut, kullanırken sayıya çevir.

```jsx
const [adetMetin, setAdetMetin] = useState("1");
const adet = Number(adetMetin) || 0;   // türetilmiş değer

<input type="number" value={adetMetin} onChange={(e) => setAdetMetin(e.target.value)} />
```

### Textarea

İçerik etiketler arasına değil, **`value` prop'una** yazılır:

```jsx
// HTML: <textarea>Merhaba</textarea>
<textarea value={mesaj} onChange={(e) => setMesaj(e.target.value)} rows={4} />
```

### Select

`selected` yerine `<select>`'e **`value`** verilir:

```jsx
const SEHIRLER = [
  { kod: "34", ad: "İstanbul" },
  { kod: "06", ad: "Ankara" },
  { kod: "35", ad: "İzmir" },
];

<select value={sehirKodu} onChange={(e) => setSehirKodu(e.target.value)}>
  <option value="" disabled>Şehir seçin</option>
  {SEHIRLER.map((s) => (
    <option key={s.kod} value={s.kod}>{s.ad}</option>
  ))}
</select>
```

#### Çoklu seçim (`multiple`) → `value` bir dizi

```jsx
const [diller, setDiller] = useState(["js"]);

<select
  multiple
  value={diller}
  onChange={(e) => setDiller(Array.from(e.target.selectedOptions, (o) => o.value))}
>
  <option value="js">JavaScript</option>
  <option value="ts">TypeScript</option>
  <option value="py">Python</option>
</select>
```

> Çoklu select kullanıcı açısından rahat değildir; çoğu zaman checkbox grubu tercih edilir.

### Checkbox

`value` değil **`checked`**; okurken `e.target.checked`:

```jsx
<label>
  <input type="checkbox" checked={abone} onChange={(e) => setAbone(e.target.checked)} />
  Bültene abone ol
</label>
```

#### Checkbox grubu

```jsx
const ILGI_ALANLARI = ["React", "Node.js", "Tasarım", "DevOps"];
const [ilgiler, setIlgiler] = useState([]);

function handleIlgi(alan, isaretli) {
  setIlgiler((onceki) =>
    isaretli ? [...onceki, alan] : onceki.filter((x) => x !== alan)
  );
}

{ILGI_ALANLARI.map((alan) => (
  <label key={alan}>
    <input
      type="checkbox"
      checked={ilgiler.includes(alan)}
      onChange={(e) => handleIlgi(alan, e.target.checked)}
    />
    {alan}
  </label>
))}
```

### Radio

Grup aynı `name`'i paylaşır; `checked` state ile **karşılaştırılarak** hesaplanır:

```jsx
const [teslimat, setTeslimat] = useState("standart");

{[
  { deger: "standart", etiket: "Standart (3-5 gün)" },
  { deger: "hizli", etiket: "Hızlı (1-2 gün)" },
  { deger: "magaza", etiket: "Mağazadan teslim" },
].map((s) => (
  <label key={s.deger}>
    <input
      type="radio"
      name="teslimat"
      value={s.deger}
      checked={teslimat === s.deger}
      onChange={(e) => setTeslimat(e.target.value)}
    />
    {s.etiket}
  </label>
))}
```

### Range, tarih, renk

```jsx
<input type="range" min={0} max={100} value={ses} onChange={(e) => setSes(Number(e.target.value))} />
<input type="date" value={tarih} onChange={(e) => setTarih(e.target.value)} />   // "2026-10-20"
<input type="color" value={renk} onChange={(e) => setRenk(e.target.value)} />    // "#3b82f6"
```

- Tarih input'u değeri her zaman `"YYYY-MM-DD"` formatında **string** olarak verir/alır.

### Dosya

`<input type="file">` **her zaman uncontrolled**'dır; güvenlik nedeniyle değeri JavaScript ile ayarlanamaz, sadece okunur:

```jsx
const [dosya, setDosya] = useState(null);

<input type="file" accept="image/*" onChange={(e) => setDosya(e.target.files[0] ?? null)} />
{dosya && <p>{dosya.name} ({Math.round(dosya.size / 1024)} KB)</p>}
```

### Özet tablo

| Element | Değer prop'u | Okunan değer | Not |
|---|---|---|---|
| `input` (text, email…) | `value` | `e.target.value` | |
| `input type="number"` | `value` | `e.target.value` / `valueAsNumber` | Değer string gelir |
| `textarea` | `value` | `e.target.value` | İçerik çocuk olarak yazılmaz |
| `select` | `value` (`<select>`'te) | `e.target.value` | `multiple` ise dizi |
| `checkbox` | `checked` | `e.target.checked` | |
| `radio` | `checked` (karşılaştırma) | `e.target.value` | Aynı `name` |
| `input type="file"` | — | `e.target.files` | Hep uncontrolled |

---

## 4. Çok Alanlı Formlar: Tek Handler

```jsx
function ProfilFormu() {
  const [form, setForm] = useState({
    ad: "",
    email: "",
    sehir: "",
    bulten: false,
  });

  function handleChange(e) {
    const { name, value, type, checked } = e.target;
    setForm((onceki) => ({
      ...onceki,
      [name]: type === "checkbox" ? checked : value,
    }));
  }

  return (
    <form>
      <input name="ad" value={form.ad} onChange={handleChange} />
      <input name="email" type="email" value={form.email} onChange={handleChange} />
      <select name="sehir" value={form.sehir} onChange={handleChange}>
        <option value="">Seçin</option>
        <option value="istanbul">İstanbul</option>
        <option value="ankara">Ankara</option>
      </select>
      <label>
        <input name="bulten" type="checkbox" checked={form.bulten} onChange={handleChange} />
        Bülten
      </label>
    </form>
  );
}
```

- `name` niteliği, state nesnesindeki **anahtarla aynı** olmalı.
- Checkbox için `checked`, diğerleri için `value` okunur.
- Updater fonksiyon kullanmak güvenlidir.

---

## 5. Etiketler, Erişilebilirlik ve `useId`

Her form alanının bir **`<label>`**'ı olmalı.

> Placeholder etiket değildir: yazmaya başlayınca kaybolur ve ekran okuyucular tarafından güvenilir şekilde okunmaz.

### İki yöntem

```jsx
// 1. Sarmalama: id gerekmez
<label>
  E-posta
  <input type="email" value={email} onChange={...} />
</label>

// 2. htmlFor + id
<label htmlFor="email">E-posta</label>
<input id="email" type="email" value={email} onChange={...} />
```

Etikete tıklamak input'a odaklanır, checkbox'larda işaretler; tıklama alanı büyür.

### `useId` ile benzersiz id'ler

Bileşen sayfada birden çok kez kullanılırsa sabit `id` çakışır. Çözüm: **`useId`** Hook'u.

```jsx
import { useId } from "react";

function FormAlani({ etiket, ...inputProps }) {
  const id = useId();   // benzersiz ve kararlı id
  return (
    <div className="alan">
      <label htmlFor={id}>{etiket}</label>
      <input id={id} {...inputProps} />
    </div>
  );
}

<FormAlani etiket="Ad" value={ad} onChange={(e) => setAd(e.target.value)} />
<FormAlani etiket="Soyad" value={soyad} onChange={(e) => setSoyad(e.target.value)} />
```

> ⚠️ `useId` liste **key'i üretmek için değildir**; key'ler veriden gelir. `useId`, erişilebilirlik niteliklerini (`id`, `htmlFor`, `aria-describedby`) birbirine bağlamak içindir.

---

## 6. Formu Göndermek

```jsx
function IletisimFormu() {
  const [form, setForm] = useState({ ad: "", mesaj: "" });
  const [durum, setDurum] = useState("bosta"); // "bosta" | "gonderiliyor" | "basarili" | "hata"

  async function handleSubmit(e) {
    e.preventDefault();
    setDurum("gonderiliyor");
    try {
      const res = await fetch("/api/iletisim", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(form),
      });
      if (!res.ok) throw new Error("Gönderilemedi");
      setDurum("basarili");
      setForm({ ad: "", mesaj: "" });   // formu sıfırla
    } catch {
      setDurum("hata");
    }
  }

  if (durum === "basarili") return <p>Teşekkürler, mesajın alındı! ✉️</p>;

  const gonderiliyor = durum === "gonderiliyor";

  return (
    <form onSubmit={handleSubmit}>
      {/* alanlar... */}
      <button type="submit" disabled={gonderiliyor}>
        {gonderiliyor ? "Gönderiliyor..." : "Gönder"}
      </button>
      {durum === "hata" && <p role="alert">Bir hata oluştu, tekrar dene.</p>}
    </form>
  );
}
```

### İyi bir gönderim akışı

- Gönderim sürerken butonu **devre dışı bırak** → çift gönderimi önler.
- Kullanıcıya **geri bildirim** ver: buton metni, başarı/hata mesajı.
- Başarıdan sonra formu **sıfırla** (state'i başlangıç değerine döndür).
- Hata durumunda kullanıcının girdiklerini **silme.**
- Formu göndermemesi gereken butonlara `type="button"`.

---

## 7. Doğrulama (Validation) ⭐

### Tarayıcının yerleşik doğrulaması

```jsx
<input type="email" required />
<input minLength={8} maxLength={64} required />
<input type="number" min={18} max={120} />
<input pattern="[0-9]{5}" title="5 haneli posta kodu" />
```

- Ücretsiz ve hızlı bir ilk savunma hattıdır.
- Eksiği: Görünüm ve mesajlar üzerinde kontrol azdır, tarayıcıya göre farklı görünür.
- Kendi doğrulaman varsa tarayıcınınki **`<form noValidate>`** ile kapatılır.

### Kendi doğrulaman: hatalar türetilmiş değerdir

```jsx
function dogrula(form) {
  const hatalar = {};
  if (!form.ad.trim()) hatalar.ad = "Ad zorunlu.";
  if (!/^\S+@\S+\.\S+$/.test(form.email)) hatalar.email = "Geçerli bir e-posta girin.";
  if (form.sifre.length < 8) hatalar.sifre = "Şifre en az 8 karakter olmalı.";
  if (form.sifreTekrar !== form.sifre) hatalar.sifreTekrar = "Şifreler eşleşmiyor.";
  return hatalar;
}

// bileşen içinde
const hatalar = dogrula(form);
const gecerli = Object.keys(hatalar).length === 0;
```

- `dogrula` **saf** bir fonksiyondur: bileşen dışında tanımlanır, test etmesi kolaydır.

### Hataları ne zaman göstermeli? `touched` state'i

- Hemen gösterilirse kullanıcı bir şey yazmadan kırmızı uyarılarla karşılaşır.
- İyi deneyim: Hata, kullanıcı **alandan çıktıktan (blur)** veya **göndermeye çalıştıktan** sonra gösterilir.
- Hangi alana dokunulduğu hesaplanamaz → **state**'tir.

```jsx
const [dokunuldu, setDokunuldu] = useState({});      // { email: true, ... }
const [gonderildi, setGonderildi] = useState(false);

function handleBlur(e) {
  setDokunuldu((onceki) => ({ ...onceki, [e.target.name]: true }));
}

function hataGoster(alan) {
  return (dokunuldu[alan] || gonderildi) && hatalar[alan];
}

function handleSubmit(e) {
  e.preventDefault();
  setGonderildi(true);
  if (!gecerli) return;    // geçersizse gönderme, hatalar artık görünür
  // ... gönder
}
```

### Hataları erişilebilir göstermek

```jsx
function Alan({ etiket, hata, ...inputProps }) {
  const id = useId();
  const hataId = `${id}-hata`;

  return (
    <div className="alan">
      <label htmlFor={id}>{etiket}</label>
      <input
        id={id}
        aria-invalid={hata ? true : undefined}
        aria-describedby={hata ? hataId : undefined}
        {...inputProps}
      />
      {hata && <p id={hataId} className="hata">{hata}</p>}
    </div>
  );
}
```

| Nitelik | Görevi |
|---|---|
| `aria-invalid` | Ekran okuyucuya alanın hatalı olduğunu söyler; CSS'te `[aria-invalid="true"]` ile stillendirilebilir |
| `aria-describedby` | Hata mesajını alana bağlar; odaklanınca mesaj da okunur |

> ⚠️ İstemci tarafı doğrulama **kullanıcı deneyimi** içindir, güvenlik için değil. **Sunucu her zaman tekrar doğrulamalıdır.**

---

## 8. Uncontrolled Formlar ve `FormData`

Değer sadece **gönderimde** lazımsa DOM'a bırakılabilir:

```jsx
function BultenFormu() {
  function handleSubmit(e) {
    e.preventDefault();
    const formData = new FormData(e.currentTarget);
    const veri = Object.fromEntries(formData);
    console.log(veri);          // { email: "ayse@ornek.com", siklik: "haftalik", kvkk: "on" }
    e.currentTarget.reset();    // DOM formunu sıfırla
  }

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" type="email" required defaultValue="" />
      <select name="siklik" defaultValue="haftalik">
        <option value="gunluk">Günlük</option>
        <option value="haftalik">Haftalık</option>
      </select>
      <label>
        <input name="kvkk" type="checkbox" defaultChecked={false} required />
        KVKK metnini okudum
      </label>
      <button type="submit">Abone ol</button>
    </form>
  );
}
```

### Dikkat edilecekler

- Başlangıç değeri: `value` yerine **`defaultValue`**, `checked` yerine **`defaultChecked`**. Sadece ilk render'da uygulanır.
- `FormData` yalnızca **`name`'i olan** alanları toplar.
- İşaretsiz checkbox `FormData`'da yer almaz; işaretliyse değeri `"on"` olur.
- Aynı `name`'li birden çok değer: `formData.getAll("ilgiler")`.
- Tüm değerler **string**'dir (dosyalar hariç).
- Değere gönderim dışında erişmek için **ref** kullanılır *(Konu 11)*.

### Hangisini seçmeli?

| Durum | Yaklaşım |
|---|---|
| Değer yazarken bir şeyi etkiliyor (canlı önizleme, anlık doğrulama, alanlar arası bağımlılık, karakter sayacı) | **Controlled** |
| Değer sadece gönderimde lazım | **Uncontrolled + `FormData`** (daha az kod, daha az render) |

> Aynı formda ikisi karıştırılabilir.

---

## 9. React 19: Form Actions'a İlk Bakış

`<form>`'un `action` prop'una URL yerine **fonksiyon** verilebilir:

```jsx
function BultenFormu() {
  async function abonelikAction(formData) {
    const email = formData.get("email");
    await fetch("/api/bulten", { method: "POST", body: JSON.stringify({ email }) });
  }

  return (
    <form action={abonelikAction}>
      <input name="email" type="email" required />
      <button type="submit">Abone ol</button>
    </form>
  );
}
```

### Farklar

- `preventDefault` gerekmez.
- Fonksiyon doğrudan `FormData` alır.
- Başarılı gönderimden sonra uncontrolled alanlar otomatik sıfırlanır.
- `useActionState` (sonuç ve hata durumu) ve `useFormStatus` (gönderiliyor bilgisi) ile manuel durum yönetiminin çoğunu React üstlenir.

> Detaylar **Konu 20**'de (Actions, `useActionState`, `useFormStatus`, `useOptimistic`). `onSubmit` yaklaşımı her yerde geçerli ve en yaygın yöntem olmaya devam ediyor.

---

## 10. Form Kütüphaneleri

Büyük formlarda (onlarca alan, iç içe/dinamik alanlar, karmaşık doğrulama):

| Kütüphane | Özellik |
|---|---|
| **React Hook Form** | En popüler; uncontrolled yaklaşımı temel alır, çok az render yapar |
| **Zod** | Şema tabanlı doğrulama; kurallar bir kez tanımlanır, hem istemcide hem sunucuda kullanılır. React Hook Form ile birlikte çok yaygın |
| **TanStack Form** | Tip güvenliği güçlü, daha yeni bir alternatif |

> Kütüphaneler bu konudaki kavramların (controlled/uncontrolled, doğrulama, erişilebilirlik) üzerine kuruludur; önce temeller oturtulmalı.

---

## 11. Her Şey Bir Arada: Kayıt Formu

```jsx
import { useId, useState } from "react";

// ---------- Saf doğrulama fonksiyonu ----------
function dogrula(f) {
  const h = {};
  if (!f.ad.trim()) h.ad = "Ad soyad zorunlu.";
  if (!/^\S+@\S+\.\S+$/.test(f.email)) h.email = "Geçerli bir e-posta girin.";
  if (f.sifre.length < 8) h.sifre = "En az 8 karakter olmalı.";
  else if (!/\d/.test(f.sifre)) h.sifre = "En az bir rakam içermeli.";
  if (f.sifreTekrar !== f.sifre) h.sifreTekrar = "Şifreler eşleşmiyor.";
  if (!f.sehir) h.sehir = "Şehir seçin.";
  if (f.ilgiler.length === 0) h.ilgiler = "En az bir ilgi alanı seçin.";
  if (!f.sozlesme) h.sozlesme = "Devam etmek için onaylamalısınız.";
  return h;
}

// ---------- Yeniden kullanılabilir alan (render prop) ----------
function Alan({ etiket, hata, children }) {
  const id = useId();
  const hataId = `${id}-hata`;
  // children: (id, a11yProps) => input elementi
  return (
    <div className="alan">
      <label htmlFor={id}>{etiket}</label>
      {children(id, {
        "aria-invalid": hata ? true : undefined,
        "aria-describedby": hata ? hataId : undefined,
      })}
      {hata && <p id={hataId} className="hata">{hata}</p>}
    </div>
  );
}

const SEHIRLER = ["İstanbul", "Ankara", "İzmir", "Bursa", "Kocaeli"];
const ILGILER = ["Frontend", "Backend", "Mobil", "Tasarım"];

const BOS_FORM = {
  ad: "",
  email: "",
  sifre: "",
  sifreTekrar: "",
  sehir: "",
  deneyim: "baslangic",
  ilgiler: [],
  hakkinda: "",
  sozlesme: false,
};

export default function KayitFormu() {
  const [form, setForm] = useState(BOS_FORM);
  const [dokunuldu, setDokunuldu] = useState({});
  const [gonderildi, setGonderildi] = useState(false);
  const [durum, setDurum] = useState("bosta"); // "bosta" | "gonderiliyor" | "basarili"

  // --- Türetilmiş değerler ---
  const hatalar = dogrula(form);
  const gecerli = Object.keys(hatalar).length === 0;
  const hata = (alan) => ((dokunuldu[alan] || gonderildi) ? hatalar[alan] : undefined);
  const kalanKarakter = 200 - form.hakkinda.length;

  // --- Olay yöneticileri ---
  function handleChange(e) {
    const { name, value, type, checked } = e.target;
    setForm((o) => ({ ...o, [name]: type === "checkbox" ? checked : value }));
  }

  function handleBlur(e) {
    setDokunuldu((o) => ({ ...o, [e.target.name]: true }));
  }

  function handleIlgi(alan, isaretli) {
    setForm((o) => ({
      ...o,
      ilgiler: isaretli ? [...o.ilgiler, alan] : o.ilgiler.filter((x) => x !== alan),
    }));
    setDokunuldu((o) => ({ ...o, ilgiler: true }));
  }

  async function handleSubmit(e) {
    e.preventDefault();
    setGonderildi(true);
    if (!gecerli) return;

    setDurum("gonderiliyor");
    await new Promise((r) => setTimeout(r, 1000)); // sahte API isteği
    console.log("Gönderilen:", { ...form, sifre: "***", sifreTekrar: undefined });
    setDurum("basarili");
  }

  function handleYeniKayit() {
    setForm(BOS_FORM);
    setDokunuldu({});
    setGonderildi(false);
    setDurum("bosta");
  }

  if (durum === "basarili") {
    return (
      <div>
        <h2>Hoş geldin, {form.ad.split(" ")[0]}! 🎉</h2>
        <button type="button" onClick={handleYeniKayit}>Yeni kayıt</button>
      </div>
    );
  }

  const gonderiliyor = durum === "gonderiliyor";

  return (
    <form onSubmit={handleSubmit} noValidate>
      <h2>Kayıt Ol</h2>

      <Alan etiket="Ad Soyad" hata={hata("ad")}>
        {(id, a11y) => (
          <input id={id} name="ad" value={form.ad}
            onChange={handleChange} onBlur={handleBlur}
            autoComplete="name" {...a11y} />
        )}
      </Alan>

      <Alan etiket="E-posta" hata={hata("email")}>
        {(id, a11y) => (
          <input id={id} name="email" type="email" value={form.email}
            onChange={handleChange} onBlur={handleBlur}
            autoComplete="email" {...a11y} />
        )}
      </Alan>

      <Alan etiket="Şifre" hata={hata("sifre")}>
        {(id, a11y) => (
          <input id={id} name="sifre" type="password" value={form.sifre}
            onChange={handleChange} onBlur={handleBlur}
            autoComplete="new-password" {...a11y} />
        )}
      </Alan>

      <Alan etiket="Şifre (tekrar)" hata={hata("sifreTekrar")}>
        {(id, a11y) => (
          <input id={id} name="sifreTekrar" type="password" value={form.sifreTekrar}
            onChange={handleChange} onBlur={handleBlur}
            autoComplete="new-password" {...a11y} />
        )}
      </Alan>

      <Alan etiket="Şehir" hata={hata("sehir")}>
        {(id, a11y) => (
          <select id={id} name="sehir" value={form.sehir}
            onChange={handleChange} onBlur={handleBlur} {...a11y}>
            <option value="" disabled>Seçin</option>
            {SEHIRLER.map((s) => (
              <option key={s} value={s}>{s}</option>
            ))}
          </select>
        )}
      </Alan>

      <fieldset>
        <legend>Deneyim</legend>
        {[
          { deger: "baslangic", etiket: "Başlangıç" },
          { deger: "orta", etiket: "Orta" },
          { deger: "ileri", etiket: "İleri" },
        ].map((d) => (
          <label key={d.deger}>
            <input type="radio" name="deneyim" value={d.deger}
              checked={form.deneyim === d.deger} onChange={handleChange} />
            {d.etiket}
          </label>
        ))}
      </fieldset>

      <fieldset aria-invalid={hata("ilgiler") ? true : undefined}>
        <legend>İlgi alanları</legend>
        {ILGILER.map((alan) => (
          <label key={alan}>
            <input type="checkbox" checked={form.ilgiler.includes(alan)}
              onChange={(e) => handleIlgi(alan, e.target.checked)} />
            {alan}
          </label>
        ))}
        {hata("ilgiler") && <p className="hata">{hata("ilgiler")}</p>}
      </fieldset>

      <Alan etiket="Hakkında (opsiyonel)">
        {(id) => (
          <>
            <textarea id={id} name="hakkinda" rows={3} maxLength={200}
              value={form.hakkinda} onChange={handleChange} />
            <small style={{ color: kalanKarakter < 20 ? "crimson" : "inherit" }}>
              {kalanKarakter} karakter kaldı
            </small>
          </>
        )}
      </Alan>

      <label>
        <input type="checkbox" name="sozlesme" checked={form.sozlesme}
          onChange={handleChange} onBlur={handleBlur} />
        Kullanım koşullarını kabul ediyorum
      </label>
      {hata("sozlesme") && <p className="hata">{hata("sozlesme")}</p>}

      <button type="submit" disabled={gonderiliyor}>
        {gonderiliyor ? "Kaydediliyor..." : "Kayıt ol"}
      </button>

      {gonderildi && !gecerli && (
        <p role="alert">Lütfen işaretli alanları düzeltin.</p>
      )}
    </form>
  );
}
```

### Bu örnekte kullanılanlar

- Tek state nesnesi ve `name` tabanlı tek handler
- text / email / password / select / radio / checkbox grubu / textarea
- Saf ve türetilmiş doğrulama
- `touched` + `gonderildi` ile doğru zamanda hata gösterme
- `useId` ve `aria-*` ile erişilebilir hatalar
- `fieldset` / `legend` ile gruplama
- `noValidate` ile tarayıcı doğrulamasını kapatma
- Karakter sayacı (controlled'ın gücü)
- `autoComplete` ipuçları
- Gönderim ve başarı durumu

### Render prop kalıbı

`Alan` bileşenine `children` olarak **fonksiyon** verildi. Ebeveyn, çocuğun içinde ne render edileceğini, çocuğun ürettiği değerlerle (`id`, `a11y`) belirler. `children` fikrinin bir uzantısıdır.

---

## 12. Özet

- **Controlled:** Değer state'te (`value` + `onChange`); **uncontrolled:** değer DOM'da (`defaultValue`, gönderimde `FormData`).
- Controlled input'ta state **tek doğruluk kaynağıdır**; girdi yazılırken dönüştürülebilir veya sınırlanabilir.
- `value` verip `onChange` vermemek input'u dondurur → `onChange`, `readOnly` veya `defaultValue`.
- `value={undefined/null}` uncontrolled sayılır; metin alanlarında başlangıç değeri `""`.
- `type="number"` değeri **string** gelir; state'te string tutup kullanırken sayıya çevirmek pratiktir.
- `textarea` ve `select`'te değer **`value` prop'u** ile verilir; `multiple` select'te dizi.
- Checkbox'ta `checked` / `e.target.checked`; radio'da `checked={state === deger}` ve ortak `name`.
- Tarih input'u `"YYYY-MM-DD"` string'i; dosya input'u **hep uncontrolled**.
- Çok alanlı formlarda tek state nesnesi + `[e.target.name]` ile **tek handler.**
- Her alanın bir `<label>`'ı olmalı; placeholder etiket değildir. Benzersiz id'ler için **`useId`** (key için değil).
- Gönderimde: `onSubmit` + `preventDefault`, butonu devre dışı bırakma, geri bildirim, başarıda sıfırlama.
- Hatalar **türetilmiş değerdir** (saf `dogrula` fonksiyonu); hangi alana dokunulduğu ise **state**'tir.
- Hatalar blur veya gönderim denemesinden sonra gösterilir; `aria-invalid` ve `aria-describedby` ile erişilebilir yapılır.
- İstemci doğrulaması UX içindir; **sunucu her zaman tekrar doğrular.**
- `FormData` yalnızca `name`'li alanları toplar; değerler string, işaretsiz checkbox yer almaz.
- React 19'da `<form action={fn}>` ile form actions mümkündür *(Konu 20)*.
- Büyük formlar için **React Hook Form**, **Zod**, **TanStack Form**.

---

⬅️ Önceki: [08 · Listeler ve key](./08-listeler-ve-key.md) · ➡️ Sonraki: [10 · useEffect ve Yan Etkiler](./10-useeffect.md)
