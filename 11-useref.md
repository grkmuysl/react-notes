# 11 · `useRef`

> **Konu özeti:** Render'ı tetiklemeden değer hatırlamak, `useRef`'in state ve normal değişkenden farkı, render sırasında ref kullanımı kuralları, zamanlayıcı id'leri ve debounce, DOM elementlerine erişim, ref'in ne zaman dolduğu, kendi bileşenine ref verme (React 19), `useImperativeHandle`, ref callback'leri, tembel başlatma, DOM'u güvenli kullanma ve `flushSync`.

---

## İçindekiler

1. [Problem: Render'ı Tetiklemeden Hatırlamak](#1-problem-renderı-tetiklemeden-hatırlamak)
2. [`useRef` Temelleri](#2-useref-temelleri)
3. [Render Sırasında `ref.current`](#3-️-render-sırasında-refcurrent)
4. [Kullanım 1: Zamanlayıcı ID'leri](#4-kullanım-1-zamanlayıcı-idleri)
5. [Kullanım 2: DOM Elementlerine Erişmek](#5-kullanım-2-dom-elementlerine-erişmek-)
6. [Ref Ne Zaman Dolar?](#6-ref-ne-zaman-dolar)
7. [Kendi Bileşenine Ref Vermek](#7-kendi-bileşenine-ref-vermek)
8. [`useImperativeHandle`](#8-useimperativehandle-sınırlı-bir-api-açmak)
9. [Ref Callback'leri](#9-ref-callbackleri)
10. [Tembel Başlatma](#10-tembel-başlatma-lazy-initialization)
11. [DOM'u Ref ile Değiştirirken Dikkat](#11-domu-ref-ile-değiştirirken-dikkat-)
12. [`flushSync`](#12-flushsync-state-güncellemesinden-hemen-sonra-doma-erişmek)
13. [Her Şey Bir Arada: Mesajlaşma Ekranı](#13-her-şey-bir-arada-mesajlaşma-ekranı)
14. [Özet](#14-özet)

---

## 1. Problem: Render'ı Tetiklemeden Hatırlamak

| Hafıza türü | Davranış |
|---|---|
| Normal değişken (`let x`) | Her render'da sıfırlanır |
| State (`useState`) | Korunur, değişince **render tetikler** |
| **İhtiyaç** | Korunsun ama değişince **render tetiklemesin** |

### Örnek ihtiyaçlar

- `setInterval` id'si: durdurmak için saklanmalı, ekranda gösterilmiyor.
- DOM elementinin kendisi: odaklanmak, kaydırmak, ölçmek için.
- Önceki değer, zaman damgası, sayaç: hesaplamada lazım, ekranı değiştirmiyor.

→ Çözüm: **`useRef`**

---

## 2. `useRef` Temelleri

```jsx
import { useRef } from "react";

function Ornek() {
  const tiklamaRef = useRef(0);

  function handleClick() {
    tiklamaRef.current = tiklamaRef.current + 1;
    console.log(`${tiklamaRef.current} kez tıklandı`);
  }

  return <button onClick={handleClick}>Tıkla</button>;
}
```

`useRef(baslangic)` bir **nesne** döndürür:

```js
{ current: 0 }
```

- `ref.current` ile okunur ve **doğrudan değiştirilir** (mutasyon serbest).
- Nesne bileşenin ömrü boyunca **aynıdır**; her render'da aynı referans döner.
- `ref.current`'ı değiştirmek **render'ı tetiklemez**; React haberdar olmaz.

### Karşılaştırma

| | Normal değişken | `useRef` | `useState` |
|---|---|---|---|
| Render'lar arasında korunur | ❌ | ✅ | ✅ |
| Değişince render tetikler | ❌ | ❌ | ✅ |
| Nasıl değiştirilir | Atama | `ref.current = x` (mutasyon) | `setX(yeni)` |
| Değişiklik ne zaman görülür | Hemen | **Hemen** | Bir sonraki render'da (snapshot) |
| Render sırasında okunabilir | ✅ | ⚠️ Okunmamalı/yazılmamalı | ✅ |
| Kullanım | Render içi geçici hesaplama | Ekranı etkilemeyen hafıza, DOM erişimi | Ekranda görünen veri |

> **Kural:** Değer **ekranda görünüyor** veya ekranı etkiliyorsa → **state.** Ekranla ilgisi yoksa, sadece olay yöneticilerinde/effect'lerde kullanılıyorsa → **ref.**

> 💡 `useRef`'i şöyle bir `useState` olarak düşünebilirsin:
> ```js
> const [ref] = useState({ current: baslangic });
> ```
> Setter'ı hiç kullanılmayan, içi mutasyonla değiştirilen bir state nesnesi.

---

## 3. ⚠️ Render Sırasında `ref.current`

```jsx
function Sayac() {
  const ref = useRef(0);
  ref.current = ref.current + 1;                  // ❌ render sırasında yazmak
  return <p>{ref.current} kez render edildi</p>;  // ❌ render sırasında okumak
}
```

| Hata | Sonuç |
|---|---|
| Render sırasında **yazmak** | Render saflığı bozulur; StrictMode'da tutarsız sonuç |
| Render sırasında **okuyup göstermek** | Ref değişince render olmadığı için ekran **eski değerde takılır** |

### Kural

- `ref.current` **olay yöneticilerinde** ve **effect'lerde** okunur/yazılır.
- Ekranda gösterilecekse **state** kullanılır.
- Tek istisna: tembel başlatma (bkz. [Bölüm 10](#10-tembel-başlatma-lazy-initialization)).

---

## 4. Kullanım 1: Zamanlayıcı ID'leri

```jsx
import { useRef, useState } from "react";

function Kronometre() {
  const [baslangic, setBaslangic] = useState(null);
  const [simdi, setSimdi] = useState(null);
  const intervalRef = useRef(null);   // id ekranda görünmüyor → ref

  function handleBaslat() {
    setBaslangic(Date.now());
    setSimdi(Date.now());

    clearInterval(intervalRef.current);   // zaten çalışıyorsa önce durdur
    intervalRef.current = setInterval(() => {
      setSimdi(Date.now());
    }, 10);
  }

  function handleDurdur() {
    clearInterval(intervalRef.current);
  }

  const gecen = baslangic && simdi ? (simdi - baslangic) / 1000 : 0;  // türetilmiş

  return (
    <>
      <h1>{gecen.toFixed(2)} sn</h1>
      <button onClick={handleBaslat}>Başlat</button>
      <button onClick={handleDurdur}>Durdur</button>
    </>
  );
}
```

| Değer | Tür | Neden |
|---|---|---|
| `intervalRef` | Ref | Ekranda görünmüyor; ama `handleDurdur`'da lazım (normal değişken her render'da sıfırlanırdı) |
| `baslangic`, `simdi` | State | Değişince ekran güncellenmeli |

### Debounce kalıbı

Kullanıcı yazmayı bıraktıktan sonra işlem yapmak:

```jsx
function AramaKutusu({ onAra }) {
  const [metin, setMetin] = useState("");
  const zamanlayiciRef = useRef(null);

  function handleChange(e) {
    const yeni = e.target.value;
    setMetin(yeni);

    clearTimeout(zamanlayiciRef.current);
    zamanlayiciRef.current = setTimeout(() => {
      onAra(yeni);          // son tuştan 400 ms sonra ara
    }, 400);
  }

  return <input value={metin} onChange={handleChange} placeholder="Ara..." />;
}
```

- Her tuşta önceki zamanlayıcı iptal edilir; kullanıcı durunca arama yapılır.
- API'ye her harfte istek atılması önlenir.

> Bileşen kaldırılırken bekleyen zamanlayıcıyı temizlemek iyi pratiktir:
> ```jsx
> useEffect(() => () => clearTimeout(zamanlayiciRef.current), []);
> ```

---

## 5. Kullanım 2: DOM Elementlerine Erişmek ⭐

React DOM'u yönetir; ama bazı işlemler doğrudan DOM API'si gerektirir:

| İşlem | DOM API |
|---|---|
| Odaklanmak | `focus()` |
| Kaydırmak | `scrollIntoView()`, `scrollTo()` |
| Ölçmek | `getBoundingClientRect()` |
| Medya kontrolü | `play()`, `pause()` |
| Metni seçmek | `select()` |
| React dışı kütüphaneye düğüm vermek | Harita, grafik vb. |

### Temel kullanım

```jsx
import { useRef } from "react";

function AramaFormu() {
  const inputRef = useRef(null);

  function handleOdaklan() {
    inputRef.current.focus();
  }

  return (
    <>
      <input ref={inputRef} placeholder="Ara..." />
      <button onClick={handleOdaklan}>Aramaya odaklan</button>
    </>
  );
}
```

1. `useRef(null)` ile boş ref oluştur.
2. `ref={inputRef}` ile elemente bağla.
3. React DOM düğümünü oluşturunca `inputRef.current`'a **gerçek DOM elementini** koyar.
4. Olay yöneticisinde/effect'te DOM API'leri çağrılır.

### Örnekler

```jsx
// Kaydırma
const bolumRef = useRef(null);

<button onClick={() => bolumRef.current.scrollIntoView({ behavior: "smooth" })}>
  İletişim'e git
</button>
<section ref={bolumRef}>...</section>
```

```jsx
// Video kontrolü
const videoRef = useRef(null);
const [oynuyor, setOynuyor] = useState(false);

function handleOynatDurdur() {
  const sonraki = !oynuyor;
  setOynuyor(sonraki);
  sonraki ? videoRef.current.play() : videoRef.current.pause();
}

<video
  ref={videoRef}
  src="tanitim.mp4"
  onPlay={() => setOynuyor(true)}
  onPause={() => setOynuyor(false)}
/>
<button onClick={handleOynatDurdur}>{oynuyor ? "Durdur" : "Oynat"}</button>
```

```jsx
// Ölçüm
const kutuRef = useRef(null);

function handleOlc() {
  const { width, height } = kutuRef.current.getBoundingClientRect();
  console.log(width, height);
}
```

### Sayfa yüklenince otomatik odaklanma

```jsx
useEffect(() => {
  inputRef.current.focus();
}, []);

// Basit durumlarda HTML niteliği de yeterli
<input autoFocus />
```

---

## 6. Ref Ne Zaman Dolar?

```
Render   → ref.current henüz null (DOM yok ya da güncellenmedi)
Commit   → React DOM'u günceller ve ref.current'ı ayarlar
Effect'ler ve olaylar → ref.current kullanılabilir
Unmount  → React ref.current'ı tekrar null yapar
```

- **Render sırasında** `ref.current`'a dokunulmaz; ilk render'da zaten `null`'dır.
- DOM ref'leri **olay yöneticilerinde** ve **effect'lerde** kullanılır.
- Koşullu render edilen elementlerde ref `null` olabilir: `inputRef.current?.focus()`.

---

## 7. Kendi Bileşenine Ref Vermek

**React 19'da** fonksiyon bileşenleri `ref`'i **normal bir prop gibi** alır ve istediği DOM elementine iletir:

```jsx
function OzelInput({ etiket, ref, ...rest }) {
  return (
    <label className="ozel-input">
      {etiket}
      <input ref={ref} {...rest} />
    </label>
  );
}

function Form() {
  const inputRef = useRef(null);
  return (
    <>
      <OzelInput etiket="E-posta" ref={inputRef} type="email" />
      <button onClick={() => inputRef.current.focus()}>Odaklan</button>
    </>
  );
}
```

- Bileşen ref'i iletmezse ebeveynin `ref.current`'ı `null` kalır.
- Bileşenler DOM'larını varsayılan olarak **kapsüllenmiş** tutar; sadece düşük seviyeli bileşenler (input, buton) açar.

> 📜 **React 18 ve öncesi:** Ref normal prop olarak geçirilemiyordu; `forwardRef` gerekiyordu.
> ```jsx
> const OzelInput = forwardRef(function OzelInput(props, ref) {
>   return <input ref={ref} {...props} />;
> });
> ```
> React 19'da hâlâ çalışır ama yeni kodda gerekmez; ileride kaldırılması planlanıyor.

---

## 8. `useImperativeHandle`: Sınırlı Bir API Açmak

Ebeveyne DOM elementinin tamamı yerine **yalnızca belirli işlemler** verilir:

```jsx
import { useImperativeHandle, useRef } from "react";

function VideoOynatici({ src, ref }) {
  const videoRef = useRef(null);   // gerçek DOM ref'i içeride kalır

  useImperativeHandle(ref, () => ({
    oynat() {
      videoRef.current.play();
    },
    durdur() {
      videoRef.current.pause();
    },
    basaSar() {
      videoRef.current.currentTime = 0;
    },
  }), []);

  return <video ref={videoRef} src={src} />;
}

function Sayfa() {
  const oynaticiRef = useRef(null);
  return (
    <>
      <VideoOynatici ref={oynaticiRef} src="tanitim.mp4" />
      <button onClick={() => oynaticiRef.current.oynat()}>▶️</button>
      <button onClick={() => oynaticiRef.current.durdur()}>⏸</button>
      <button onClick={() => oynaticiRef.current.basaSar()}>⏮</button>
    </>
  );
}
```

- `oynaticiRef.current` artık DOM elementi değil, tanımlanan `{ oynat, durdur, basaSar }` nesnesidir.

> ⚠️ Bu bir **kaçış yoludur.** "Ebeveyn çocuğa komut veriyor" kalıbı, "veri props ile aşağı akar" modeline aykırıdır. `<Modal acik={acik} />`, `modalRef.current.ac()`'tan neredeyse her zaman daha iyidir. `useImperativeHandle`; odak, kaydırma, medya oynatma gibi **doğası gereği emir kipi** olan işlemler içindir.

---

## 9. Ref Callback'leri

`ref` niteliğine **fonksiyon** verilebilir. React, DOM düğümü oluşunca fonksiyonu düğümle çağırır:

```jsx
<div ref={(dugum) => {
  if (dugum) console.log("Eklendi:", dugum);
}} />
```

### Liste elemanlarına ref vermek

Hook'lar döngü içinde çağrılamaz:

```jsx
{urunler.map((u) => {
  const ref = useRef(null);   // ❌ Hook kuralı ihlali
  return <li ref={ref}>...</li>;
})}
```

Çözüm: Tek ref'te bir **`Map`** tutulur, ref callback'iyle doldurulur:

```jsx
function KediGalerisi({ kediler }) {
  const dugumlerRef = useRef(new Map());

  function kediyeKaydir(id) {
    dugumlerRef.current.get(id)?.scrollIntoView({
      behavior: "smooth",
      block: "nearest",
      inline: "center",
    });
  }

  return (
    <>
      <nav>
        {kediler.map((k) => (
          <button key={k.id} onClick={() => kediyeKaydir(k.id)}>{k.ad}</button>
        ))}
      </nav>
      <ul className="galeri">
        {kediler.map((k) => (
          <li
            key={k.id}
            ref={(dugum) => {
              dugumlerRef.current.set(k.id, dugum);
              return () => dugumlerRef.current.delete(k.id);   // React 19: cleanup
            }}
          >
            <img src={k.foto} alt={k.ad} />
          </li>
        ))}
      </ul>
    </>
  );
}
```

### React 19 yeniliği: Ref callback cleanup

- Ref callback'i bir **cleanup fonksiyonu döndürebilir**; eleman kaldırılınca React onu çağırır.
- Cleanup döndürüldüğünde React callback'i artık `null` ile çağırmaz.
- 📜 React 19 öncesinde eleman kaldırılınca callback `null` ile çağrılırdı.

> Ref callback'leri, bir element DOM'a eklendiği **anda** bir şey yapmak için de kullanışlıdır (dinamik eklenen input'a odaklanmak, ölçmeye başlamak).

---

## 10. Tembel Başlatma (Lazy Initialization)

```jsx
const isleyiciRef = useRef(new PahaliVideoIsleyici());   // ❌ her render'da yeni nesne oluşturulur
```

`useRef` fonksiyon kabul etmez; kalıp şöyledir:

```jsx
function VideoIsleyici() {
  const isleyiciRef = useRef(null);

  if (isleyiciRef.current === null) {
    isleyiciRef.current = new PahaliVideoIsleyici();   // sadece ilk render'da
  }

  // ...
}
```

> Render sırasında `ref.current` yazmanın **tek kabul edilen istisnasıdır**: sonuç her render'da aynıdır, saflığı bozmaz.

---

## 11. DOM'u Ref ile Değiştirirken Dikkat ⭐

Ref, React'in yönetiminden **kaçış yoludur (escape hatch).**

### Güvenli işlemler (yıkıcı olmayan)

Odak, kaydırma, seçim, medya oynatma, ölçüm — React'in yönetmediği şeyler.

### Tehlikeli: React'in yönettiği DOM'u elle değiştirmek

```jsx
function Tehlikeli() {
  const [goster, setGoster] = useState(true);
  const ref = useRef(null);

  return (
    <>
      <button onClick={() => setGoster(!goster)}>Göster/Gizle (React ile)</button>
      <button onClick={() => ref.current.remove()}>Sil (DOM ile)</button>   {/* ❌ */}
      {goster && <p ref={ref}>Merhaba</p>}
    </>
  );
}
```

"Sil (DOM ile)" → "Göster/Gizle" sırasıyla basılınca uygulama **çöker**: React, artık var olmayan bir düğümü kaldırmaya çalışır.

### Kurallar

- React'in render ettiği elementlerin **içeriği, nitelikleri veya varlığı** ref ile değiştirilmez → state ve JSX kullanılır.
- ❌ `ref.current.style.color = ...`, `innerHTML = ...`, `classList.add(...)`, `remove()` (React'in render ettiği şeyler için).
- ✅ İçi React tarafından **hiç yönetilmeyen** boş bir kap (`<div ref={kapRef} />`) üçüncü parti kütüphaneye verilebilir.

> Ref'e başvurmadan önce sor: *"Bunu state ve props ile deklaratif olarak yapabilir miyim?"*

---

## 12. `flushSync`: State Güncellemesinden Hemen Sonra DOM'a Erişmek

### Sorun

```jsx
function handleGonder() {
  setMesajlar([...mesajlar, yeniMesaj]);
  listeRef.current.lastChild.scrollIntoView();   // ❌ yeni mesaj henüz DOM'da değil
}
```

`setState` DOM'u hemen güncellemez (batching); bir **önceki** son mesaja kaydırılır.

### Çözüm 1: Effect (genelde daha iyi)

```jsx
useEffect(() => {
  listeRef.current.lastChild?.scrollIntoView({ behavior: "smooth" });
}, [mesajlar.length]);
```

### Çözüm 2: `flushSync` (nadir durumlar)

```jsx
import { flushSync } from "react-dom";

function handleGonder() {
  flushSync(() => {
    setMesajlar([...mesajlar, yeniMesaj]);   // DOM'a HEMEN yansır
  });
  listeRef.current.lastChild.scrollIntoView();   // ✅
}
```

> `flushSync` batching'i zorla bozar ve performansı düşürebilir; sadece "bu güncellemenin DOM'a yansımasını şimdi beklemeliyim" durumlarında kullanılır.

---

## 13. Her Şey Bir Arada: Mesajlaşma Ekranı

```jsx
import { useEffect, useImperativeHandle, useRef, useState } from "react";

// --- Düşük seviyeli input: sınırlı bir API açıyor ---
function MesajInput({ deger, onDegis, onGonder, ref }) {
  const inputRef = useRef(null);

  useImperativeHandle(ref, () => ({
    odaklan: () => inputRef.current.focus(),
    temizleVeOdaklan: () => {
      onDegis("");
      inputRef.current.focus();
    },
  }), [onDegis]);

  return (
    <input
      ref={inputRef}
      value={deger}
      onChange={(e) => onDegis(e.target.value)}
      onKeyDown={(e) => e.key === "Enter" && onGonder()}
      placeholder="Mesaj yaz..."
    />
  );
}

const BASLANGIC = [
  { id: 1, kimden: "Ayşe", metin: "Selam! Notları yükledin mi?" },
  { id: 2, kimden: "Ben", metin: "10. konuya kadar hazır 🎉" },
  { id: 3, kimden: "Ayşe", metin: "Süper, useRef'e geçtin mi?" },
];

export default function Sohbet() {
  const [mesajlar, setMesajlar] = useState(BASLANGIC);
  const [metin, setMetin] = useState("");
  const [yaziyor, setYaziyor] = useState(false);

  const listeRef = useRef(null);            // DOM: kaydırma kabı
  const inputApiRef = useRef(null);         // kendi bileşenimizin API'si
  const mesajDugumleri = useRef(new Map()); // liste elemanlarına ref'ler
  const yaziyorZamanlayici = useRef(null);  // zamanlayıcı id'si
  const gonderilenSayisi = useRef(0);       // ekranda gösterilmeyen istatistik

  // Sayfa açılınca input'a odaklan
  useEffect(() => {
    inputApiRef.current.odaklan();
  }, []);

  // Yeni mesaj gelince en alta kaydır
  useEffect(() => {
    const liste = listeRef.current;
    liste.scrollTo({ top: liste.scrollHeight, behavior: "smooth" });
  }, [mesajlar.length]);

  // Bileşen kalkarken bekleyen zamanlayıcıyı temizle
  useEffect(() => () => clearTimeout(yaziyorZamanlayici.current), []);

  function handleGonder() {
    const temiz = metin.trim();
    if (!temiz) return;

    setMesajlar((m) => [...m, { id: Date.now(), kimden: "Ben", metin: temiz }]);
    inputApiRef.current.temizleVeOdaklan();
    gonderilenSayisi.current += 1;
    console.log(`Bu oturumda ${gonderilenSayisi.current} mesaj gönderildi`);

    // Karşı taraf "yazıyor..." ve 1,5 sn sonra cevap veriyor
    setYaziyor(true);
    clearTimeout(yaziyorZamanlayici.current);
    yaziyorZamanlayici.current = setTimeout(() => {
      setYaziyor(false);
      setMesajlar((m) => [...m, { id: Date.now(), kimden: "Ayşe", metin: "👍" }]);
    }, 1500);
  }

  function ilkMesajaGit() {
    const ilkId = mesajlar[0]?.id;
    const dugum = mesajDugumleri.current.get(ilkId);
    dugum?.scrollIntoView({ behavior: "smooth", block: "center" });
    dugum?.animate(
      [{ background: "#fde68a" }, { background: "transparent" }],
      { duration: 1200 }
    );
  }

  return (
    <div className="sohbet">
      <header>
        <strong>Ayşe</strong> {yaziyor && <small>yazıyor...</small>}
        <button type="button" onClick={ilkMesajaGit}>⬆ İlk mesaj</button>
      </header>

      <ul ref={listeRef} className="mesajlar" style={{ height: 300, overflowY: "auto" }}>
        {mesajlar.map((m) => (
          <li
            key={m.id}
            className={m.kimden === "Ben" ? "benim" : "onun"}
            ref={(dugum) => {
              mesajDugumleri.current.set(m.id, dugum);
              return () => mesajDugumleri.current.delete(m.id);
            }}
          >
            {m.metin}
          </li>
        ))}
      </ul>

      <footer>
        <MesajInput
          ref={inputApiRef}
          deger={metin}
          onDegis={setMetin}
          onGonder={handleGonder}
        />
        <button type="button" onClick={handleGonder}>Gönder</button>
      </footer>
    </div>
  );
}
```

### Bu örnekte kullanılanlar

| Ref | Kullanım |
|---|---|
| `listeRef` | DOM elementine erişim (kaydırma) |
| `inputApiRef` + `MesajInput` | React 19'da ref'i prop olarak alma + `useImperativeHandle` ile sınırlı API |
| `mesajDugumleri` | Ref callback'i + `Map` ile liste elemanlarına ref, React 19 cleanup'ı |
| `yaziyorZamanlayici` | Zamanlayıcı id'si saklama |
| `gonderilenSayisi` | Ekranda gösterilmeyen değeri render tetiklemeden tutma |

- Ekranda görünen her şey (`mesajlar`, `metin`, `yaziyor`) **state.**
- Kaydırma effect ile yapılıyor; DOM içeriği hiçbir yerde ref ile elle değiştirilmiyor (`animate` geçici bir görsel efekt).

---

## 14. Özet

- **`useRef`**, render'lar arasında korunan ama değişince **render tetiklemeyen** bir değer saklar: `{ current: deger }`.
- `ref.current` doğrudan değiştirilir (mutasyon); değişiklik hemen görülür.
- Ekranda görünen/ekranı etkileyen değer → **state**; ekranla ilgisi olmayan hafıza ve DOM erişimi → **ref.**
- Render sırasında `ref.current` **okunmaz ve yazılmaz**; olay yöneticilerinde ve effect'lerde kullanılır (istisna: tembel başlatma).
- Zamanlayıcı id'leri ref'te saklanır; **debounce** kalıbı ref + `setTimeout` ile yapılır.
- `ref={inputRef}` ile DOM elementine erişilir: `focus`, `scrollIntoView`, `play/pause`, `getBoundingClientRect`.
- Ref **commit** aşamasında dolar, unmount'ta `null` olur; koşullu elementlerde `?.` kullanılır.
- **React 19:** Fonksiyon bileşenleri `ref`'i normal prop olarak alır; `forwardRef` artık gerekmez.
- **`useImperativeHandle`** ile ebeveyne DOM yerine sınırlı bir API açılır; emir kipi işlemler için kaçış yoludur.
- **Ref callback'leri** ile liste elemanlarına ref verilir (`Map` ile); React 19'da callback cleanup döndürebilir.
- `useRef` için tembel başlatma: `if (ref.current === null) ref.current = new X()`.
- Ref ile **yıkıcı olmayan** işlemler yapılır; React'in yönettiği DOM'un içeriği, nitelikleri veya varlığı elle değiştirilmez.
- State güncellemesinden hemen sonra DOM'a erişmek için tercihen **effect**, gerekirse **`flushSync`.**

---

⬅️ Önceki: [10 · useEffect ve Yan Etkiler](./10-useeffect.md) · ➡️ Sonraki: [12 · Bileşenler Arası İletişim](./12-bilesenler-arasi-iletisim.md)
