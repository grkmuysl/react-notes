# 15 · Custom Hook'lar

> **Konu özeti:** Custom Hook'un ne olduğu ve neden var olduğu, tekrar eden mantığı Hook'a çıkarmak, "state değil mantık paylaşılır" ilkesi, Hook kuralları ve isimlendirme, parametreler ve dönüş tasarımı, olay yöneticilerini Hook'a vermek, sık kullanılan Hook örnekleri, `useSyncExternalStore`, ne zaman Hook yazılmalı, dosya düzeni ve hazır kütüphaneler.

---

## İçindekiler

1. [Sezgisel Bakış](#1-sezgisel-bakış)
2. [Problem: Aynı Mantık İki Yerde](#2-problem-aynı-mantık-iki-yerde)
3. [İlk Custom Hook](#3-ilk-custom-hook)
4. [State Değil, Mantık Paylaşılır](#4-state-değil-mantık-paylaşılır-)
5. [Custom Hook Kuralları](#5-custom-hook-kuralları)
6. [Parametreler ve Dönüş Değerleri](#6-parametreler-ve-dönüş-değerleri)
7. [Olay Yöneticilerini Custom Hook'a Vermek](#7-olay-yöneticilerini-custom-hooka-vermek)
8. [Sık Kullanılan Custom Hook Örnekleri](#8-sık-kullanılan-custom-hook-örnekleri-)
9. [`useSyncExternalStore`](#9-usesyncexternalstore-dış-kaynaklara-abone-olmak)
10. [Ne Zaman Custom Hook Yazmalı?](#10-ne-zaman-custom-hook-yazmalı)
11. [Dosya Düzeni ve Hazır Kütüphaneler](#11-dosya-düzeni-ve-hazır-kütüphaneler)
12. [Her Şey Bir Arada: Kitap Arama Uygulaması](#12-her-şey-bir-arada-kitap-arama-uygulaması)
13. [Özet](#13-özet)

---

## 1. Sezgisel Bakış

> **Custom Hook = Adı `use` ile başlayan ve içinde başka Hook'lar çağıran sıradan bir JavaScript fonksiyonu.**

| Araç | Neyi tekrar kullanılabilir yapar? | Örnek |
|---|---|---|
| **Bileşen** | Bir **arayüz parçasını** | `Buton`, `Kart` |
| **Custom Hook** | Bir **davranışı** | "Çevrimiçi miyim?", "Pencere ne kadar geniş?", "Bu değeri localStorage'da sakla" |

> **Benzetme:** İki restoran şubesi aynı **tarifi** kullanır ama her şube kendi **malzemesiyle** pişirir. Custom Hook bir tariftir: mantık paylaşılır, her bileşen **kendi state'ini** alır.

---

## 2. Problem: Aynı Mantık İki Yerde

```jsx
function DurumCubugu() {
  const [cevrimici, setCevrimici] = useState(navigator.onLine);

  useEffect(() => {
    const cevrimiciOl = () => setCevrimici(true);
    const cevrimdisiOl = () => setCevrimici(false);
    window.addEventListener("online", cevrimiciOl);
    window.addEventListener("offline", cevrimdisiOl);
    return () => {
      window.removeEventListener("online", cevrimiciOl);
      window.removeEventListener("offline", cevrimdisiOl);
    };
  }, []);

  return <p>{cevrimici ? "✅ Çevrimiçi" : "❌ Bağlantı yok"}</p>;
}

function KaydetButonu() {
  const [cevrimici, setCevrimici] = useState(navigator.onLine);
  useEffect(() => {
    // ... yukarıdakinin AYNISI
  }, []);
  return <button disabled={!cevrimici}>{cevrimici ? "Kaydet" : "Bağlanılıyor..."}</button>;
}
```

- Aynı kod iki yerde; hata düzeltmesi iki yerde yapılmalı.
- Normal JS'te tekrar eden kod fonksiyona çıkarılır; **Hook içeren kod da custom Hook'a çıkarılır.**

---

## 3. İlk Custom Hook

```jsx
// src/hooks/useCevrimici.js
import { useEffect, useState } from "react";

export function useCevrimici() {
  const [cevrimici, setCevrimici] = useState(navigator.onLine);

  useEffect(() => {
    const cevrimiciOl = () => setCevrimici(true);
    const cevrimdisiOl = () => setCevrimici(false);
    window.addEventListener("online", cevrimiciOl);
    window.addEventListener("offline", cevrimdisiOl);
    return () => {
      window.removeEventListener("online", cevrimiciOl);
      window.removeEventListener("offline", cevrimdisiOl);
    };
  }, []);

  return cevrimici;
}
```

```jsx
import { useCevrimici } from "./hooks/useCevrimici";

function DurumCubugu() {
  const cevrimici = useCevrimici();
  return <p>{cevrimici ? "✅ Çevrimiçi" : "❌ Bağlantı yok"}</p>;
}

function KaydetButonu() {
  const cevrimici = useCevrimici();
  return <button disabled={!cevrimici}>{cevrimici ? "Kaydet" : "Bağlanılıyor..."}</button>;
}
```

- Bileşenler **ne** istediklerini söyler, **nasıl** öğrenileceğini değil.
- Effect'i amacını anlatan bir Hook'a sarmak bileşeni daha **bildirimsel** yapar.

> 💡 Bileşende çıplak `useEffect` görülen yerlerde, onu amacını anlatan bir custom Hook'a sarmak düşünülmelidir.

---

## 4. State Değil, Mantık Paylaşılır ⭐

```jsx
function useSayac(baslangic = 0) {
  const [sayi, setSayi] = useState(baslangic);
  const artir = () => setSayi((s) => s + 1);
  return [sayi, artir];
}

function A() {
  const [sayi, artir] = useSayac();
  return <button onClick={artir}>A: {sayi}</button>;
}

function B() {
  const [sayi, artir] = useSayac();
  return <button onClick={artir}>B: {sayi}</button>;
}
```

A'ya tıklamak B'yi **etkilemez**; sayaçlar tamamen bağımsızdır.

```
useSayac (tarif)
   │
   ├── A içinde çağrıldı → A'nın kendi "sayi" state'i
   └── B içinde çağrıldı → B'nin kendi "sayi" state'i
```

- Custom Hook sadece bir fonksiyondur; içindeki `useState`, **çağıran bileşenin** state'ini oluşturur.
- Sanki Hook'un kodu kopyalanıp her bileşene yapıştırılmış gibidir.

| İstediğin | Araç |
|---|---|
| Aynı **davranışı** birden çok bileşende kullanmak | Custom Hook |
| Aynı **veriyi** birden çok bileşende paylaşmak | State'i yukarı taşımak / Context |
| İkisi birden | Context'i okuyan bir custom Hook (`useTema` gibi) |

> Konu 13'teki `useTema` aynı değeri döndürüyordu çünkü state `TemaProvider`'da **tek yerde** duruyordu; `useTema` sadece okuyordu.

---

## 5. Custom Hook Kuralları

### 1. Ad `use` + büyük harf ile başlar

`useCevrimici`, `useLocalStorage`, `useDebounce`

- Linter Hook kurallarını bu isme bakarak kontrol eder.
- Okuyan kişi `use` görünce içinde state/effect olabileceğini anlar.

### 2. Hook kuralları aynen geçerlidir

- Sadece bileşenlerin veya başka Hook'ların **en üst seviyesinde** çağrılır.
- Koşul, döngü veya iç içe fonksiyon içinde çağrılmaz.

### 3. İçinde Hook çağırmayan fonksiyona `use` adı verilmez

```js
// ❌ İçinde Hook yok
function useSiralanmisListe(liste) {
  return liste.toSorted();
}

// ✅ Normal yardımcı fonksiyon
function siralanmisListe(liste) {
  return liste.toSorted();
}
```

> Hook olmayan fonksiyon koşullu çağrılabilir; `use` adı bu özgürlüğü gereksiz yere kısıtlar.

### 4. Custom Hook başka custom Hook'ları kullanabilir

Küçük Hook'lar birleştirilerek büyükleri kurulur.

---

## 6. Parametreler ve Dönüş Değerleri

### Dönüş tasarımı

```jsx
const cevrimici = useCevrimici();                                    // tek değer
const [acik, toggle] = useToggle();                                  // dizi (tuple)
const [tema, setTema] = useLocalStorage("tema", "acik");
const { veri, yukleniyor, hata, tekrarDene } = useVeriGetir(url);    // nesne
```

| Dönüş | Ne zaman |
|---|---|
| Tek değer | Hook tek bir şey veriyorsa |
| Dizi `[a, b]` | İki (en fazla üç) değer; aynı Hook bir bileşende birden çok kez kullanılacaksa (isimler serbest) |
| Nesne `{ a, b, c }` | Üç ve daha fazla değer; kullanıcı sadece bir kısmını alacaksa |

### Hook'lar her render'da yeniden çalışır

Custom Hook'un kodu bileşenin her render'ında tekrar çalışır; verilen props ve state **her zaman güncel**dir:

```jsx
function SohbetOdasi({ odaId }) {
  const [sunucuUrl, setSunucuUrl] = useState("wss://sohbet.ornek.com");
  useSohbetBaglantisi({ sunucuUrl, odaId });   // her render'da güncel değerlerle
}

function useSohbetBaglantisi({ sunucuUrl, odaId }) {
  useEffect(() => {
    const baglanti = baglan(sunucuUrl, odaId);
    baglanti.baglan();
    return () => baglanti.kes();
  }, [sunucuUrl, odaId]);
}
```

> Konu 10'daki tüm effect kuralları Hook'un içinde aynen geçerlidir.

---

## 7. Olay Yöneticilerini Custom Hook'a Vermek

```jsx
function SohbetOdasi({ odaId, tema }) {
  useSohbetBaglantisi({
    odaId,
    onMesaj(mesaj) {
      bildirimGoster(`Yeni mesaj: ${mesaj}`, tema);
    },
  });
}
```

**Sorun:** `onMesaj` her render'da yeni fonksiyon.

- Bağımlılığa eklenirse → her render'da yeniden bağlanır.
- Eklenmezse → bayat `tema`.

### Çözüm: `useEffectEvent`

```jsx
import { useEffect, useEffectEvent } from "react";

function useSohbetBaglantisi({ odaId, onMesaj }) {
  const onMesajEvent = useEffectEvent(onMesaj);   // her zaman güncel onMesaj'ı çağırır

  useEffect(() => {
    const baglanti = baglan(odaId);
    baglanti.on("mesaj", (m) => onMesajEvent(m));
    baglanti.baglan();
    return () => baglanti.kes();
  }, [odaId]);   // ✅ onMesaj bağımlılık değil
}
```

> Bileşen istediği gibi inline fonksiyon verebilir; bağlantı sadece `odaId` değişince yeniden kurulur.

---

## 8. Sık Kullanılan Custom Hook Örnekleri ⭐

### `useToggle`: Aç/kapa

```jsx
import { useCallback, useState } from "react";

export function useToggle(baslangic = false) {
  const [deger, setDeger] = useState(baslangic);
  const toggle = useCallback(() => setDeger((d) => !d), []);
  return [deger, toggle, setDeger];
}

// Kullanım
const [menuAcik, menuToggle] = useToggle();
const [sifreGorunur, sifreToggle] = useToggle();
```

### `useLocalStorage`: Kalıcı state

```jsx
import { useEffect, useState } from "react";

export function useLocalStorage(anahtar, baslangic) {
  const [deger, setDeger] = useState(() => {
    try {
      const kayitli = localStorage.getItem(anahtar);
      return kayitli !== null ? JSON.parse(kayitli) : baslangic;
    } catch {
      return baslangic;   // bozuk JSON veya erişim engeli
    }
  });

  useEffect(() => {
    try {
      localStorage.setItem(anahtar, JSON.stringify(deger));
    } catch {
      // kota doldu veya gizli mod: sessizce geç
    }
  }, [anahtar, deger]);

  return [deger, setDeger];
}

// Kullanım: useState ile birebir aynı imza
const [tema, setTema] = useLocalStorage("tema", "acik");
const [favoriler, setFavoriler] = useLocalStorage("favoriler", []);
```

> `useState` ile aynı arayüz → değiştirmek tek kelimelik değişiklik. İyi Hook tasarımının işareti.

### `useDebounce`: Değeri geciktirmek

```jsx
import { useEffect, useState } from "react";

export function useDebounce(deger, gecikme = 400) {
  const [gecikmeli, setGecikmeli] = useState(deger);

  useEffect(() => {
    const id = setTimeout(() => setGecikmeli(deger), gecikme);
    return () => clearTimeout(id);   // değer tekrar değişirse önceki zamanlayıcı iptal
  }, [deger, gecikme]);

  return gecikmeli;
}

// Kullanım
const [arama, setArama] = useState("");
const gecikmeliArama = useDebounce(arama, 500);
// arama her tuşta değişir; gecikmeliArama sadece kullanıcı 500 ms durunca değişir
```

> Debounce mantığının tamamı cleanup'ın gücüne dayanır.

### `useVeriGetir`: Veri çekme

```jsx
import { useCallback, useEffect, useReducer, useState } from "react";

function veriReducer(state, action) {
  switch (action.type) {
    case "basladi":  return { ...state, yukleniyor: true, hata: null };   // önceki veri korunur
    case "basarili": return { veri: action.veri, yukleniyor: false, hata: null };
    case "hata":     return { ...state, yukleniyor: false, hata: action.mesaj };
    default: throw new Error(`Bilinmeyen action: ${action.type}`);
  }
}

export function useVeriGetir(url) {
  const [state, dispatch] = useReducer(veriReducer, {
    veri: null,
    yukleniyor: !!url,
    hata: null,
  });
  const [deneme, setDeneme] = useState(0);
  const tekrarDene = useCallback(() => setDeneme((d) => d + 1), []);

  useEffect(() => {
    if (!url) return;
    const controller = new AbortController();
    dispatch({ type: "basladi" });

    fetch(url, { signal: controller.signal })
      .then((res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then((veri) => dispatch({ type: "basarili", veri }))
      .catch((err) => {
        if (err.name !== "AbortError") dispatch({ type: "hata", mesaj: err.message });
      });

    return () => controller.abort();
  }, [url, deneme]);

  return { ...state, tekrarDene };
}

// Kullanım
function KullaniciProfili({ id }) {
  const { veri, yukleniyor, hata, tekrarDene } = useVeriGetir(
    `https://jsonplaceholder.typicode.com/users/${id}`
  );

  if (yukleniyor) return <p>Yükleniyor...</p>;
  if (hata) return <p>Hata: {hata} <button onClick={tekrarDene}>Tekrar dene</button></p>;
  return <h2>{veri.name}</h2>;
}
```

- `useReducer`, `useState`, `useCallback`, `useEffect` bir arada; bileşen 3 satırla veri çekiyor.
- `deneme` sayacı değişince effect yeniden çalışır → "tekrar dene".
- Önbellekleme gibi özellikler için gerçek projelerde **TanStack Query** tercih edilir *(Konu 18)*.

### `useDisariTiklama`: Dışarı tıklanınca kapat

```jsx
import { useEffect, useEffectEvent } from "react";

export function useDisariTiklama(ref, onDisariTiklandi, aktif = true) {
  const onDisari = useEffectEvent(onDisariTiklandi);

  useEffect(() => {
    if (!aktif) return;
    function handlePointerDown(e) {
      if (ref.current && !ref.current.contains(e.target)) onDisari(e);
    }
    document.addEventListener("pointerdown", handlePointerDown);
    return () => document.removeEventListener("pointerdown", handlePointerDown);
  }, [ref, aktif]);
}

// Kullanım
function AcilirMenu() {
  const [acik, toggle, setAcik] = useToggle();
  const menuRef = useRef(null);
  useDisariTiklama(menuRef, () => setAcik(false), acik);

  return (
    <div ref={menuRef}>
      <button onClick={toggle}>Menü ▾</button>
      {acik && <ul className="menu">...</ul>}
    </div>
  );
}
```

### `useMedyaSorgusu`: Ekran boyutuna tepki

```jsx
import { useCallback, useSyncExternalStore } from "react";

export function useMedyaSorgusu(sorgu) {
  const abone = useCallback((bildir) => {
    const mql = window.matchMedia(sorgu);
    mql.addEventListener("change", bildir);
    return () => mql.removeEventListener("change", bildir);
  }, [sorgu]);   // sorgu değişmedikçe aynı fonksiyon → yeniden abone olunmaz

  return useSyncExternalStore(
    abone,
    () => window.matchMedia(sorgu).matches,   // güncel değer
    () => false                                // sunucuda render için yedek
  );
}

// Kullanım
const mobil = useMedyaSorgusu("(max-width: 640px)");
const koyuTercih = useMedyaSorgusu("(prefers-color-scheme: dark)");
```

---

## 9. `useSyncExternalStore`: Dış Kaynaklara Abone Olmak

React dışındaki bir veri kaynağına (tarayıcı API'si, üçüncü parti store) abone olmak için özel Hook:

```js
const deger = useSyncExternalStore(subscribe, getSnapshot, getServerSnapshot?);
```

| Parametre | Görevi |
|---|---|
| `subscribe(bildir)` | Kaynağa abone olur; değişiklikte `bildir()` çağırır; abonelikten çıkan fonksiyon döndürür |
| `getSnapshot()` | Kaynağın **o anki** değerini döndürür |
| `getServerSnapshot()` | (Opsiyonel) Sunucuda render için değer |

### `useCevrimici`'nin daha doğru hali

```jsx
import { useSyncExternalStore } from "react";

function abone(bildir) {
  window.addEventListener("online", bildir);
  window.addEventListener("offline", bildir);
  return () => {
    window.removeEventListener("online", bildir);
    window.removeEventListener("offline", bildir);
  };
}

export function useCevrimici() {
  return useSyncExternalStore(
    abone,
    () => navigator.onLine,
    () => true   // sunucuda "çevrimiçi" varsay
  );
}
```

### `useState` + `useEffect`'e göre avantajları

- İlk render ile effect arasındaki değişiklikler **kaçırılmaz.**
- Eşzamanlı (concurrent) render ile güvenlidir; ekranın farklı yerlerinde aynı kaynağın farklı değerlerinin görünmesini (**tearing**) engeller.
- Sunucu tarafı render'ı destekler.

> ⚠️ `subscribe` bileşen içinde her render'da yeni tanımlanırsa React her render'da yeniden abone olur → **dışarıda tanımla** veya `useCallback` ile sabitle.
> ⚠️ `getSnapshot` aynı değer için **aynı sonucu** döndürmeli; her çağrıda yeni nesne döndürürse sonsuz render olur.

> Kendi uygulama state'in için `useState`/`useReducer` yeterlidir. Zustand, Redux gibi kütüphaneler arka planda bu Hook'u kullanır.

---

## 10. Ne Zaman Custom Hook Yazmalı?

### ✅ Custom Hook'a çıkar

- Aynı Hook kombinasyonu birden çok bileşende **tekrar ediyorsa**.
- Bileşende **dış sistemle** senkronizasyon yapan bir `useEffect` varsa (`useSohbetBaglantisi`, `useBelgeBasligi`).
- Bileşenin içindeki mantık, "ne gösterdiğini" anlamayı zorlaştıracak kadar büyüdüyse.
- Mantığı ayrı **test etmek** istiyorsan *(Konu 21)*.

### ❌ Gerek yok

- Gerçekten tekrar etmeyen tek bir `useState` için ince sarmalayıcılar.
- "İleride lazım olur" diye erken soyutlama. Önce iki-üç kez tekrar etsin.

### ⚠️ "Yaşam döngüsü" Hook'larından kaçın

```jsx
// ❌ Ne yaptığını değil, NE ZAMAN çalıştığını söylüyor
function useMount(fn) {
  useEffect(() => { fn(); }, []);   // linter: fn bağımlılık olmalı
}

useMount(() => {
  baglan(odaId);   // odaId değişirse yeniden bağlanmaz!
});
```

- `useMount`, `useUpdate`, `useUnmount` class bileşen düşüncesini geri getirir ve bağımlılıkları gizleyerek bayat değer hatalarına yol açar.
- Hook'lar **somut amaca** göre adlandırılır: `useSohbetBaglantisi(odaId)`, `useBelgeBasligi(baslik)`, `usePencereBoyutu()`.

> **Test:** Hook'un adını duyan biri, içine bakmadan ne yaptığını tahmin edebiliyor mu? `useCevrimici` ✅, `useMount` ❌.

---

## 11. Dosya Düzeni ve Hazır Kütüphaneler

### Dosya düzeni

```
src/
├── hooks/                  ← genel amaçlı, her yerde kullanılan Hook'lar
│   ├── useCevrimici.js
│   ├── useDebounce.js
│   ├── useLocalStorage.js
│   └── useMedyaSorgusu.js
├── context/
│   └── TemaContext.jsx     ← context'e özel Hook'lar kendi dosyasında (useTema)
└── features/
    └── sepet/
        └── useSepet.js     ← bir özelliğe ait Hook'lar o özelliğin yanında
```

- JSX içermeyen Hook dosyaları `.js` uzantılı olabilir.

### Hazır kütüphaneler

| Kütüphane | Not |
|---|---|
| **usehooks-ts** | TypeScript dostu, geniş koleksiyon |
| **@uidotdev/usehooks** | Popüler, sade |
| **ahooks**, **react-use** | Kapsamlı koleksiyonlar |

> Kendi Hook'unu yazmayı bilmek önemlidir; ama `useLocalStorage`, `useMedyaSorgusu` gibi yaygın Hook'ları kütüphaneden almak da makuldür.

---

## 12. Her Şey Bir Arada: Kitap Arama Uygulaması

Gerçek bir API (Open Library) ile bu konudaki Hook'ların birleşimi:

```jsx
// ============ src/hooks/index.js ============
import {
  useCallback, useEffect, useEffectEvent, useReducer, useState, useSyncExternalStore,
} from "react";

// --- useToggle ---
export function useToggle(baslangic = false) {
  const [deger, setDeger] = useState(baslangic);
  const toggle = useCallback(() => setDeger((d) => !d), []);
  return [deger, toggle, setDeger];
}

// --- useLocalStorage ---
export function useLocalStorage(anahtar, baslangic) {
  const [deger, setDeger] = useState(() => {
    try {
      const k = localStorage.getItem(anahtar);
      return k !== null ? JSON.parse(k) : baslangic;
    } catch {
      return baslangic;
    }
  });

  useEffect(() => {
    try { localStorage.setItem(anahtar, JSON.stringify(deger)); } catch { /* yok say */ }
  }, [anahtar, deger]);

  return [deger, setDeger];
}

// --- useDebounce ---
export function useDebounce(deger, gecikme = 400) {
  const [gecikmeli, setGecikmeli] = useState(deger);
  useEffect(() => {
    const id = setTimeout(() => setGecikmeli(deger), gecikme);
    return () => clearTimeout(id);
  }, [deger, gecikme]);
  return gecikmeli;
}

// --- useVeriGetir ---
function veriReducer(state, action) {
  switch (action.type) {
    case "basladi":  return { ...state, yukleniyor: true, hata: null };
    case "basarili": return { veri: action.veri, yukleniyor: false, hata: null };
    case "hata":     return { ...state, yukleniyor: false, hata: action.mesaj };
    case "sifirla":  return { veri: null, yukleniyor: false, hata: null };
    default: throw new Error(`Bilinmeyen action: ${action.type}`);
  }
}

export function useVeriGetir(url) {
  const [state, dispatch] = useReducer(veriReducer, { veri: null, yukleniyor: false, hata: null });
  const [deneme, setDeneme] = useState(0);
  const tekrarDene = useCallback(() => setDeneme((d) => d + 1), []);

  useEffect(() => {
    if (!url) {
      dispatch({ type: "sifirla" });
      return;
    }
    const controller = new AbortController();
    dispatch({ type: "basladi" });

    fetch(url, { signal: controller.signal })
      .then((res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then((veri) => dispatch({ type: "basarili", veri }))
      .catch((err) => {
        if (err.name !== "AbortError") dispatch({ type: "hata", mesaj: err.message });
      });

    return () => controller.abort();
  }, [url, deneme]);

  return { ...state, tekrarDene };
}

// --- useMedyaSorgusu ---
export function useMedyaSorgusu(sorgu) {
  const abone = useCallback((bildir) => {
    const mql = window.matchMedia(sorgu);
    mql.addEventListener("change", bildir);
    return () => mql.removeEventListener("change", bildir);
  }, [sorgu]);

  return useSyncExternalStore(abone, () => window.matchMedia(sorgu).matches, () => false);
}

// --- useDisariTiklama ---
export function useDisariTiklama(ref, onDisariTiklandi, aktif = true) {
  const onDisari = useEffectEvent(onDisariTiklandi);
  useEffect(() => {
    if (!aktif) return;
    function handle(e) {
      if (ref.current && !ref.current.contains(e.target)) onDisari(e);
    }
    document.addEventListener("pointerdown", handle);
    return () => document.removeEventListener("pointerdown", handle);
  }, [ref, aktif]);
}

// --- useKlavyeKisayolu ---
export function useKlavyeKisayolu(tus, onBasildi, { ctrl = false } = {}) {
  const onBas = useEffectEvent(onBasildi);
  useEffect(() => {
    function handle(e) {
      const ctrlUygun = !ctrl || e.ctrlKey || e.metaKey;
      if (ctrlUygun && e.key.toLowerCase() === tus.toLowerCase()) {
        e.preventDefault();
        onBas(e);
      }
    }
    window.addEventListener("keydown", handle);
    return () => window.removeEventListener("keydown", handle);
  }, [tus, ctrl]);
}

// --- useBelgeBasligi ---
export function useBelgeBasligi(baslik) {
  useEffect(() => {
    const onceki = document.title;
    document.title = baslik;
    return () => { document.title = onceki; };
  }, [baslik]);
}
```

```jsx
// ============ src/KitapArama.jsx ============
import { useRef, useState } from "react";
import {
  useToggle, useLocalStorage, useDebounce, useVeriGetir,
  useMedyaSorgusu, useDisariTiklama, useKlavyeKisayolu, useBelgeBasligi,
} from "./hooks";

function KitapKarti({ kitap, favori, onFavori }) {
  const kapak = kitap.cover_i
    ? `https://covers.openlibrary.org/b/id/${kitap.cover_i}-M.jpg`
    : null;

  return (
    <li className="kitap-karti">
      {kapak ? <img src={kapak} alt="" width={60} /> : <div className="kapak-yok">📕</div>}
      <div>
        <strong>{kitap.title}</strong>
        <p>{kitap.author_name?.join(", ") ?? "Yazar bilinmiyor"}</p>
        <small>{kitap.first_publish_year ?? "—"}</small>
      </div>
      <button type="button" onClick={onFavori} aria-pressed={favori} aria-label="Favori">
        {favori ? "★" : "☆"}
      </button>
    </li>
  );
}

function FavorilerMenusu({ favoriler, onKaldir }) {
  const [acik, toggle, setAcik] = useToggle();
  const kapRef = useRef(null);
  useDisariTiklama(kapRef, () => setAcik(false), acik);
  useKlavyeKisayolu("Escape", () => setAcik(false));

  return (
    <div ref={kapRef} className="favoriler">
      <button type="button" onClick={toggle} aria-expanded={acik}>
        ★ Favoriler ({favoriler.length})
      </button>
      {acik && (
        <ul className="acilir">
          {favoriler.length === 0 ? (
            <li>Henüz favori yok.</li>
          ) : (
            favoriler.map((f) => (
              <li key={f.key}>
                {f.title}
                <button type="button" onClick={() => onKaldir(f.key)} aria-label="Kaldır">✕</button>
              </li>
            ))
          )}
        </ul>
      )}
    </div>
  );
}

export default function KitapArama() {
  const [arama, setArama] = useState("");
  const [favoriler, setFavoriler] = useLocalStorage("kitap-favoriler", []);
  const gecikmeliArama = useDebounce(arama.trim(), 500);
  const mobil = useMedyaSorgusu("(max-width: 640px)");
  const aramaRef = useRef(null);

  const url =
    gecikmeliArama.length >= 3
      ? `https://openlibrary.org/search.json?q=${encodeURIComponent(gecikmeliArama)}` +
        `&limit=${mobil ? 5 : 10}&fields=key,title,author_name,first_publish_year,cover_i`
      : null;

  const { veri, yukleniyor, hata, tekrarDene } = useVeriGetir(url);
  const kitaplar = veri?.docs ?? [];

  useBelgeBasligi(gecikmeliArama ? `"${gecikmeliArama}" · Kitap Arama` : "Kitap Arama");
  useKlavyeKisayolu("k", () => aramaRef.current?.focus(), { ctrl: true });

  const favoriAnahtarlari = new Set(favoriler.map((f) => f.key));   // türetilmiş

  function handleFavori(kitap) {
    setFavoriler((onceki) =>
      onceki.some((f) => f.key === kitap.key)
        ? onceki.filter((f) => f.key !== kitap.key)
        : [...onceki, { key: kitap.key, title: kitap.title }]
    );
  }

  return (
    <main className={mobil ? "mobil" : "masaustu"}>
      <header>
        <h1>📚 Kitap Arama</h1>
        <FavorilerMenusu
          favoriler={favoriler}
          onKaldir={(key) => setFavoriler((o) => o.filter((f) => f.key !== key))}
        />
      </header>

      <input
        ref={aramaRef}
        type="search"
        value={arama}
        onChange={(e) => setArama(e.target.value)}
        placeholder={mobil ? "Ara..." : "Kitap veya yazar ara (Ctrl+K)"}
      />

      {arama.trim().length > 0 && arama.trim().length < 3 && <p>En az 3 karakter yazın.</p>}
      {yukleniyor && <p>Aranıyor...</p>}
      {hata && (
        <p role="alert">
          Arama başarısız: {hata}{" "}
          <button type="button" onClick={tekrarDene}>Tekrar dene</button>
        </p>
      )}
      {!yukleniyor && !hata && url && kitaplar.length === 0 && <p>Sonuç bulunamadı.</p>}

      <ul className="kitap-listesi" style={{ opacity: yukleniyor ? 0.5 : 1 }}>
        {kitaplar.map((k) => (
          <KitapKarti
            key={k.key}
            kitap={k}
            favori={favoriAnahtarlari.has(k.key)}
            onFavori={() => handleFavori(k)}
          />
        ))}
      </ul>
    </main>
  );
}
```

### Bu örnekte kullanılanlar

| Hook | Görevi |
|---|---|
| `useDebounce` | Her tuşta değil, kullanıcı durunca ara |
| `useVeriGetir` | Yükleniyor / hata / yarış durumu / tekrar dene (reducer + effect + AbortController) |
| `useLocalStorage` | Favoriler sayfa yenilense de kalıcı (`useState` ile aynı imza) |
| `useMedyaSorgusu` | `useSyncExternalStore` ile ekran boyutuna göre sonuç sayısı ve placeholder |
| `useToggle` + `useDisariTiklama` + `useKlavyeKisayolu("Escape")` | Dışarı tıklayınca veya Esc ile kapanan menü |
| `useKlavyeKisayolu("k", ..., { ctrl: true })` | Ctrl+K ile aramaya odaklanma |
| `useBelgeBasligi` | Sekme başlığında arama terimi |

- `KitapArama` bileşeninde **hiç `useEffect` yok**; tüm effect'ler amacını anlatan Hook'ların içinde.
- `FavorilerMenusu` ve `KitapArama` aynı `useKlavyeKisayolu` Hook'unu kullanıyor ama **birbirinden bağımsız** dinleyicilere sahipler.

---

## 13. Özet

- **Custom Hook**, adı `use` ile başlayan ve içinde Hook çağıran sıradan bir fonksiyondur; bileşenler arayüzü, Hook'lar **davranışı** tekrar kullanılabilir yapar.
- Tekrar eden Hook mantığı custom Hook'a çıkarılır; bileşenler **ne** istediğini söyler, **nasıl** olduğunu değil.
- **State değil, mantık paylaşılır:** Hook'u kullanan her bileşen kendi bağımsız state'ini alır. Veri paylaşmak için state yukarı taşınır veya context kullanılır.
- İsim `use` + büyük harf ile başlar; Hook kuralları aynen geçerlidir; içinde Hook olmayan fonksiyona `use` adı verilmez.
- Hook'lar istediği parametreyi alır; tek değer, dizi (2-3 değer) veya nesne (çok değer) döndürür.
- Hook'lar her render'da yeniden çalışır; aldıkları değerler her zaman günceldir.
- Hook'a verilen olay yöneticileri **`useEffectEvent`** ile sarılarak effect bağımlılığı olmaktan çıkarılır.
- Yaygın Hook'lar: `useToggle`, `useLocalStorage`, `useDebounce`, `useVeriGetir`, `useDisariTiklama`, `useMedyaSorgusu`, `useKlavyeKisayolu`, `useBelgeBasligi`.
- **`useSyncExternalStore`**, tarayıcı API'leri ve dış store'lara güvenli abonelik sağlar; `subscribe` sabit olmalı, `getSnapshot` tutarlı değer döndürmelidir.
- Bileşendeki çıplak `useEffect`'ler amacını anlatan Hook'lara sarılabilir; `useMount` gibi yaşam döngüsü Hook'larından kaçınılır.
- Genel Hook'lar `hooks/` klasöründe, özelliğe ait Hook'lar özelliğin yanında tutulur; hazır kütüphaneler (usehooks-ts vb.) de kullanılabilir.

---

⬅️ Önceki: [14 · useReducer](./14-usereducer.md) · ➡️ Sonraki: [16 · Stil Yöntemleri](./16-stil-yontemleri.md)
