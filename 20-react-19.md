# 20 · React 19 Yenilikleri

> **Konu özeti:** React 19 serisinin (19.0 – 19.3) getirdikleri: Actions ve async transition'lar, form action'ları, `useActionState`, `useFormStatus`, `useOptimistic`, `use`, belge metadata'sı ve kaynaklar, küçük iyileştirmeler ve hata raporlama, `<Activity>`, React Performance Tracks, `<ViewTransition>` ve `addTransitionType`, Fragment refs, `browser()`, Trusted Types, kaldırılan ve kullanımdan kalkan API'ler.

---

## İçindekiler

1. [Sezgisel Bakış](#1-sezgisel-bakış)
2. [Actions: Async Transition'lar](#2-actions-async-transitionlar-)
3. [Form Actions](#3-form-actions-form-actionfn-)
4. [`useActionState`](#4-useactionstate-actionın-sonucunu-ve-durumunu-tutmak-)
5. [`useFormStatus`](#5-useformstatus-formun-durumunu-alt-bileşenden-okumak)
6. [`useOptimistic`](#6-useoptimistic-iyimser-arayüz-)
7. [`use`: Promise ve Context](#7-use-promise-ve-context-okumak)
8. [Metadata, Stil Dosyaları ve Script'ler](#8-belge-metadatası-stil-dosyaları-ve-scriptler)
9. [Küçük İyileştirmeler ve Hata Raporlama](#9-küçük-ama-önemli-iyileştirmeler)
10. [React 19.2: `<Activity>` ve Performance Tracks](#10-react-192-activity-useeffectevent-ve-performance-tracks)
11. [React 19.3: View Transitions, Fragment Refs, `browser()`](#11-react-193-view-transitions-fragment-refs-ve-browser-)
12. [Kaldırılan ve Kullanımdan Kalkan API'ler](#12-kaldırılan-ve-kullanımdan-kalkan-apiler)
13. [Her Şey Bir Arada: Etkinlik Kayıt Uygulaması](#13-her-şey-bir-arada-etkinlik-kayıt-uygulaması)
14. [Özet](#14-özet)

---

## 1. Sezgisel Bakış

### Actions'ın çözdüğü problem

```jsx
const [durum, setDurum] = useState("bosta");
const [hata, setHata] = useState(null);

async function handleSubmit(e) {
  e.preventDefault();
  setDurum("gonderiliyor");
  try {
    await kaydet(form);
    setDurum("basarili");
  } catch (err) {
    setHata(err.message);
    setDurum("hata");
  }
}
```

Her formda aynı tören. React 19 bu kalıbı **çerçeveye** taşıdı:

| Senin elle yazdığın | React 19'da kim yapıyor |
|---|---|
| `isPending` state'i | `useTransition`, `useActionState`, `useFormStatus` |
| Hata/sonuç state'i | `useActionState` |
| `e.preventDefault()` + `FormData` okumak | `<form action={fn}>` |
| Formu sıfırlamak | Otomatik |
| İyimser güncelleme ve geri alma | `useOptimistic` |

> **Benzetme:** Eskiden her garson fişi kendisi tasarlıyor, mutfağa kendisi taşıyor, durumu kendisi takip ediyordu. React 19 restorana standart bir **sipariş sistemi** kurdu.

### Sürüm zaman çizelgesi

| Sürüm | Tarih | Öne çıkanlar |
|---|---|---|
| **19.0** | Aralık 2024 | Actions, `useActionState`, `useFormStatus`, `useOptimistic`, `use`, `ref` prop, `<Context>` sağlayıcı, metadata, kaynak ipuçları, Server Components |
| **19.1** | Mart 2025 | Owner stack (hata ayıklama), Suspense iyileştirmeleri |
| **19.2** | Ekim 2025 | `<Activity>`, `useEffectEvent`, Performance Tracks, `cacheSignal` |
| **19.3** | 9 Eylül 2026 | `<ViewTransition>`, Fragment refs, `browser()`, Trusted Types |

> 📌 React artık Linux Foundation bünyesindeki **React Foundation** altında barınıyor (Şubat 2026).

---

## 2. Actions: Async Transition'lar ⭐

React 19 ile `startTransition`'a **async** fonksiyon verilebilir. Async bir transition'a **Action** denir.

```jsx
import { useState, useTransition } from "react";

function IsimGuncelle() {
  const [isim, setIsim] = useState("");
  const [hata, setHata] = useState(null);
  const [isPending, startTransition] = useTransition();

  function handleKaydet() {
    startTransition(async () => {
      const sonuc = await isimGuncelle(isim);
      if (sonuc.hata) {
        setHata(sonuc.hata);
        return;
      }
      startTransition(() => {   // await SONRASI güncelleme
        setIsim("");
      });
    });
  }

  return (
    <>
      <input value={isim} onChange={(e) => setIsim(e.target.value)} />
      <button onClick={handleKaydet} disabled={isPending}>
        {isPending ? "Kaydediliyor..." : "Kaydet"}
      </button>
      {hata && <p role="alert">{hata}</p>}
    </>
  );
}
```

- `isPending`, async fonksiyon **tamamen bitene kadar** `true` kalır.
- Arayüz donmaz; transition olduğu için diğer güncellemeler önceliklidir.

> ⚠️ **Bilinen kısıt:** `await`'ten **sonraki** state güncellemeleri otomatik olarak transition'a dahil edilmez; tekrar `startTransition` ile sarılmalıdır. `useActionState` bu sorunu çözer.

> **İsimlendirme geleneği:** Action olarak kullanılan fonksiyonların sonuna `Action` eklenir: `kaydetAction`, `silAction`.

---

## 3. Form Actions: `<form action={fn}>` ⭐

```jsx
function BultenFormu() {
  async function abonelikAction(formData) {
    const email = formData.get("email");
    await bulteneKaydet(email);
  }

  return (
    <form action={abonelikAction}>
      <input name="email" type="email" required />
      <button type="submit">Abone ol</button>
    </form>
  );
}
```

- Fonksiyon otomatik olarak bir **Action** (transition) içinde çalışır.
- `preventDefault` yok; fonksiyon doğrudan **`FormData`** alır.
- Başarılı gönderimden sonra **uncontrolled** alanlar otomatik sıfırlanır (controlled alanlar sıfırlanmaz).
- Butonlar kendi action'ını alabilir: `<button formAction={taslakKaydetAction}>`.

### Elle sıfırlama

```jsx
import { requestFormReset } from "react-dom";

startTransition(() => requestFormReset(formElemani));
```

---

## 4. `useActionState`: Action'ın Sonucunu ve Durumunu Tutmak ⭐

```jsx
import { useActionState } from "react";

async function kayitAction(oncekiDurum, formData) {
  const email = String(formData.get("email") ?? "").trim();
  const sifre = String(formData.get("sifre") ?? "");

  // Doğrulama
  const hatalar = {};
  if (!email.includes("@")) hatalar.email = "Geçerli bir e-posta girin.";
  if (sifre.length < 8) hatalar.sifre = "Şifre en az 8 karakter olmalı.";
  if (Object.keys(hatalar).length) {
    return { durum: "hata", hatalar, degerler: { email } };
  }

  // Sunucu
  try {
    await kayitOl(email, sifre);
    return { durum: "basarili", mesaj: `Hoş geldin, ${email}!` };
  } catch (err) {
    return { durum: "hata", hatalar: { genel: err.message }, degerler: { email } };
  }
}

function KayitFormu() {
  const [sonuc, formAction, isPending] = useActionState(kayitAction, { durum: "bosta" });

  if (sonuc.durum === "basarili") return <p>{sonuc.mesaj}</p>;

  return (
    <form action={formAction}>
      <input name="email" defaultValue={sonuc.degerler?.email} />
      {sonuc.hatalar?.email && <p className="hata">{sonuc.hatalar.email}</p>}

      <input name="sifre" type="password" />
      {sonuc.hatalar?.sifre && <p className="hata">{sonuc.hatalar.sifre}</p>}

      {sonuc.hatalar?.genel && <p role="alert">{sonuc.hatalar.genel}</p>}

      <button disabled={isPending}>{isPending ? "Kaydediliyor..." : "Kayıt ol"}</button>
    </form>
  );
}
```

```js
const [durum, formAction, isPending] = useActionState(actionFn, baslangicDurumu);
//       │         │          │                        │
//       │         │          │                        └ fn(oncekiDurum, formData) → yeniDurum
//       │         │          └ action çalışıyor mu?
//       │         └ <form action> veya <button formAction>'a verilir
//       └ action'ın son döndürdüğü değer
```

### Dikkat edilecekler

- Action **iki** parametre alır: `(oncekiDurum, formData)` → reducer'a benzer, ama async olabilir ve yan etki yapabilir.
- Döndürülen değer bir sonraki render'da `durum` olur; `await` sonrası güncelleme derdi yoktur.
- Hata durumunda kullanıcının yazdıkları kaybolmasın diye değerler sonuçta döndürülüp `defaultValue` ile geri verilir.
- Form dışında elle çağırmak: `startTransition(() => formAction(veri))`.

> 📌 Eski kaynaklarda adı **`useFormState`** idi ve `react-dom`'dan import ediliyordu.

> 💡 Server Components kullanan framework'lerde action bir **Server Function** olabilir; form JavaScript yüklenmeden bile çalışır *(Konu 23)*.

---

## 5. `useFormStatus`: Formun Durumunu Alt Bileşenden Okumak

```jsx
import { useFormStatus } from "react-dom";

function GonderButonu({ children, bekleyenMetin = "Gönderiliyor..." }) {
  const { pending } = useFormStatus();
  return (
    <button type="submit" disabled={pending} aria-busy={pending}>
      {pending ? bekleyenMetin : children}
    </button>
  );
}

// Herhangi bir formda, props geçirmeden:
<form action={kayitAction}>
  {/* ... */}
  <GonderButonu>Kayıt ol</GonderButonu>
</form>
```

| Alan | Anlamı |
|---|---|
| `pending` | Form gönderiliyor mu |
| `data` | Gönderilen `FormData` |
| `method` | `get` / `post` |
| `action` | Formun action'ı |

> ⚠️ Yalnızca bir `<form>`'un **içinde render edilen** bileşenlerde çalışır; formu render eden bileşende değil.

```jsx
function Form() {
  const { pending } = useFormStatus();   // ❌ her zaman false
  return <form action={/* ... */}>...</form>;
}
```

---

## 6. `useOptimistic`: İyimser Arayüz ⭐

```jsx
import { useOptimistic, useState, startTransition } from "react";

function MesajListesi() {
  const [mesajlar, setMesajlar] = useState([]);

  const [iyimserMesajlar, iyimserEkle] = useOptimistic(
    mesajlar,                                          // gerçek state
    (mevcut, yeniMetin) => [                           // iyimser birleştirme
      ...mevcut,
      { id: `gecici-${Date.now()}`, metin: yeniMetin, gonderiliyor: true },
    ]
  );

  async function gonderAction(formData) {
    const metin = formData.get("mesaj");
    iyimserEkle(metin);                                // anında göster
    const kayitli = await mesajGonder(metin);          // sunucu
    startTransition(() => {
      setMesajlar((onceki) => [...onceki, kayitli]);   // gerçek veri
    });
  }

  return (
    <>
      <ul>
        {iyimserMesajlar.map((m) => (
          <li key={m.id} style={{ opacity: m.gonderiliyor ? 0.5 : 1 }}>
            {m.metin} {m.gonderiliyor && <small>(gönderiliyor...)</small>}
          </li>
        ))}
      </ul>
      <form action={gonderAction}>
        <input name="mesaj" required />
        <button>Gönder</button>
      </form>
    </>
  );
}
```

### Nasıl çalışır?

- `useOptimistic(gercekState, birlestir)` → `[gosterilecekState, iyimserGuncelle]`
- Action **sürerken**: gerçek state + iyimser değişiklikler gösterilir.
- Action **bitince**: iyimser değer atılır, ekran gerçek state'e döner.
  - Gerçek state güncellendiyse değişiklik kalıcıdır.
  - Hata olduysa iyimser değişiklik **kendiliğinden kaybolur** → elle geri alma kodu yok.
- `iyimserGuncelle` yalnızca bir **Action/transition içinde** çağrılabilir.

### Birleştirme fonksiyonsuz kullanım

```jsx
const [iyimserFavori, setIyimserFavori] = useOptimistic(favori);

async function favoriAction() {
  setIyimserFavori(!favori);
  await favoriGuncelle(!favori);
  // ... gerçek state'i güncelle
}
```

---

## 7. `use`: Promise ve Context Okumak

```jsx
import { use, Suspense } from "react";

function Yorumlar({ yorumlarPromise }) {
  const yorumlar = use(yorumlarPromise);   // Promise çözülene kadar askıya alınır
  return yorumlar.map((y) => <p key={y.id}>{y.metin}</p>);
}

function Sayfa({ yorumlarPromise }) {
  return (
    <Suspense fallback={<p>Yorumlar yükleniyor...</p>}>
      <Yorumlar yorumlarPromise={yorumlarPromise} />
    </Suspense>
  );
}
```

- Bekleyen Promise → bileşen **askıya alınır**, en yakın `<Suspense>` fallback'i görünür.
- Reddedilen Promise → en yakın **Error Boundary** yakalar.
- **Koşul içinde** ve erken `return` sonrasında çağrılabilir.
- `try/catch` içinde **çağrılmaz.**
- Context de okur: `use(TemaContext)` *(Konu 13.9)*.

### ⚠️ Promise render sırasında oluşturulmaz

```jsx
// ❌ Her render'da YENİ promise → sonsuz döngü
function Yorumlar() {
  const yorumlar = use(fetch("/api/yorumlar").then((r) => r.json()));
}
```

### Promise nereden gelebilir?

1. **Sunucu / framework:** Server Component oluşturup prop olarak geçer *(Konu 23)* — en yaygın kullanım.
2. **Router loader'ı:** Loader Promise'i `await` etmeden döndürür.
3. **Olay yöneticisinde oluşturulup state'e konarak:**

```jsx
const [sonucPromise, setSonucPromise] = useState(null);

function handleAra() {
  setSonucPromise(araApi(sorgu));   // promise bir kez, olayda oluştu
}

{sonucPromise && (
  <Suspense fallback="Aranıyor...">
    <Sonuclar promise={sonucPromise} />
  </Suspense>
)}
```

4. **Bir önbellekten** (aynı anahtar → aynı Promise).

> `use` bir **okuma** mekanizmasıdır; önbellek, yeniden doğrulama, mutasyon yönetimi sunmaz. İstemci tarafı veri çekme için TanStack Query (`useSuspenseQuery`) hâlâ daha pratiktir.

---

## 8. Belge Metadata'sı, Stil Dosyaları ve Script'ler

### Metadata

```jsx
function UrunSayfasi({ urun }) {
  return (
    <article>
      <title>{`${urun.ad} · Mağaza`}</title>
      <meta name="description" content={urun.ozet} />
      <link rel="canonical" href={`https://magaza.com/urun/${urun.id}`} />
      <h1>{urun.ad}</h1>
    </article>
  );
}
```

- React bu etiketleri `<head>`'e taşır.

### Stil dosyaları

```jsx
<link rel="stylesheet" href="/grafik.css" precedence="default" />
```

- `precedence` ile React dosyayı `<head>`'e doğru sırayla ekler, tekrarları birleştirir ve **yüklenene kadar** içeriği göstermez (stilsiz içerik titremesi olmaz).

### Async script'ler

- `<script async src="...">` bileşen içinde render edilebilir; birden çok bileşen render etse bile bir kez yüklenir.

### Kaynak ipuçları

`preload`, `preconnect`, `prefetchDNS`, `preinit` *(Konu 19.13)*

---

## 9. Küçük Ama Önemli İyileştirmeler

| Özellik | Eskisi | React 19 | Konu |
|---|---|---|---|
| Fonksiyon bileşenine ref | `forwardRef(...)` | `ref` normal prop | 11.7 |
| Ref callback temizliği | `null` ile tekrar çağrılır | Cleanup döndürür | 11.9 |
| Context sağlayıcı | `<Context.Provider value>` | `<Context value>` | 13.2 |
| Ertelenen değerin ilk değeri | — | `useDeferredValue(deger, ilkDeger)` | 19.10 |
| Hydration hataları | Belirsiz uyarılar | Neyin uyuşmadığını gösteren tek bir fark mesajı | — |
| Üçüncü parti `<head>`/`<body>` etiketleri | Hydration hatası | Beklenmedik etiketler atlanır | — |
| Custom Elements (Web Components) | Kısmi destek | Tam destek | — |

### Hata raporlama

```jsx
createRoot(document.getElementById("root"), {
  onUncaughtError: (hata, bilgi) => {
    // Hiçbir Error Boundary'nin yakalamadığı hatalar
    hataServisi.raporla(hata, bilgi.componentStack);
  },
  onCaughtError: (hata, bilgi) => {
    // Bir Error Boundary'nin yakaladığı hatalar
    hataServisi.raporla(hata, bilgi.componentStack, { yakalandi: true });
  },
  onRecoverableError: (hata) => {
    // React'in kendiliğinden toparladığı hatalar (ör. hydration uyuşmazlığı)
    console.warn(hata);
  },
}).render(<App />);
```

> Sentry gibi servislere merkezi hata gönderimi için idealdir.

---

## 10. React 19.2: `<Activity>`, `useEffectEvent` ve Performance Tracks

### `<Activity>`: Gizle ama unutma ⭐

```jsx
import { Activity, useState } from "react";

function Sekmeler() {
  const [sekme, setSekme] = useState("profil");

  return (
    <>
      <nav>
        <button onClick={() => setSekme("profil")}>Profil</button>
        <button onClick={() => setSekme("mesajlar")}>Mesajlar</button>
        <button onClick={() => setSekme("ayarlar")}>Ayarlar</button>
      </nav>

      <Activity mode={sekme === "profil" ? "visible" : "hidden"}>
        <Profil />
      </Activity>
      <Activity mode={sekme === "mesajlar" ? "visible" : "hidden"}>
        <Mesajlar />   {/* yarım yazılmış mesaj, kaydırma konumu korunur */}
      </Activity>
      <Activity mode={sekme === "ayarlar" ? "visible" : "hidden"}>
        <Ayarlar />
      </Activity>
    </>
  );
}
```

#### `mode="hidden"` olduğunda

- Çocuklar gizlenir (`display: none`).
- **State ve DOM korunur.**
- **Effect'ler temizlenir** (cleanup çalışır); görünür olunca yeniden kurulur.
- Güncellemeler **düşük öncelikle** işlenir.
- Henüz gösterilmemiş gizli içerik arka planda **önceden render** edilebilir.

| | `{kosul && <X />}` | CSS `hidden` | `<Activity mode="hidden">` |
|---|---|---|---|
| State | Silinir | Korunur | Korunur |
| Effect'ler | Temizlenir | Çalışmaya devam eder | Temizlenir |
| Arka plan render'ları | Yok | Normal öncelik | Düşük öncelik |
| Önceden render | ❌ | ✅ | ✅ |

### `useEffectEvent`

Effect içinde güncel değerleri okuyan ama reaktif olmayan mantık *(Konu 10.11, 15.7)*.

### React Performance Tracks

Chrome DevTools **Performance** panelinde React'e özel izler:

- **Scheduler:** Hangi güncelleme hangi öncelikte, ne zaman çalıştı; transition'lar, bloklanmalar
- **Components:** Hangi bileşenler render oldu, effect'ler ne kadar sürdü

> React Profiler ile tarayıcının performans verisi tek ekranda.

---

## 11. React 19.3: View Transitions, Fragment Refs ve `browser()` ⭐

### `<ViewTransition>`: Animasyonlu geçişler

Tarayıcının **View Transition API**'sini kullanarak elemanları belirirken, kaybolurken, yer değiştirirken veya boyut değiştirirken animasyonla geçirir.

```jsx
import { ViewTransition, startTransition, useState } from "react";

function Bildirim() {
  const [goster, setGoster] = useState(false);

  return (
    <>
      <button onClick={() => startTransition(() => setGoster((g) => !g))}>
        {goster ? "Gizle" : "Göster"}
      </button>

      {goster && (
        <ViewTransition>
          <div className="bildirim">Yeni mesajın var!</div>
        </ViewTransition>
      )}
    </>
  );
}
```

> **Animasyonlar yalnızca transition'larla tetiklenir:** `startTransition`, `<Suspense>` açılması, `useDeferredValue`. Acil güncellemeler animasyon tetiklemez.

#### Animasyon türleri

| Tür | Ne zaman |
|---|---|
| **enter** | `<ViewTransition>` eklendi |
| **exit** | `<ViewTransition>` kaldırıldı |
| **update** | Çocukların stili veya içeriği değişti |
| **share** | İsimli bir `<ViewTransition>` bir yerden kaldırılıp başka yere eklendi |

#### Özelleştirme

- Varsayılan: yumuşak **çapraz geçiş** (cross-fade).
- CSS ile: View Transition sınıfı verilir.
- JS ile: `onEnter`, `onExit`, `onShare`, `onUpdate` olay prop'ları + Web Animations API.

```jsx
<ViewTransition enter="kaydir-gir" exit="kaydir-cik">
  <Panel />
</ViewTransition>
```

```css
::view-transition-new(.kaydir-gir) { animation: 250ms ease-out both kayarak-gir; }
::view-transition-old(.kaydir-cik) { animation: 200ms ease-in both kayarak-cik; }

@keyframes kayarak-gir { from { opacity: 0; transform: translateY(12px); } }
@keyframes kayarak-cik { to   { opacity: 0; transform: translateY(-12px); } }

@media (prefers-reduced-motion: reduce) {
  ::view-transition-group(*),
  ::view-transition-old(*),
  ::view-transition-new(*) { animation: none !important; }
}
```

> ⚠️ React, `prefers-reduced-motion` için animasyonları **otomatik kapatmaz**; CSS'te eklenmelidir.

#### `addTransitionType`: Aynı güncelleme, farklı animasyon

```jsx
import { addTransitionType, startTransition, ViewTransition } from "react";

function ileri() {
  startTransition(() => {
    addTransitionType("ileri");
    setSayfa((s) => s + 1);
  });
}
function geri() {
  startTransition(() => {
    addTransitionType("geri");
    setSayfa((s) => s - 1);
  });
}

<ViewTransition
  key={sayfa}
  enter={{ ileri: "sagdan-gir", geri: "soldan-gir" }}
  exit={{ ileri: "sola-cik", geri: "saga-cik" }}
>
  <Sayfa no={sayfa} />
</ViewTransition>
```

- `key={sayfa}` değişince eski `<ViewTransition>` **çıkar**, yenisi **girer** → enter/exit birlikte çalışır.

#### Suspense ile

```jsx
<ViewTransition update="auto" default="none">
  <Suspense fallback={<Iskelet />}>
    <Icerik />
  </Suspense>
</ViewTransition>
```

**React ekibinin önerisi:**

- Fallback **animasyonsuz** hemen görünmeli
- Fallback → gerçek içerik geçişi **animasyonlu** olmalı
- Askıya alınmayan içerik **animasyonsuz** görünmeli

> `<ViewTransition>` içindeki **görseller ve fontlar** yüklenirken Suspense tetiklenebilir → koordineli yükleme sıraları.

> Şimdilik yalnızca DOM'da (web) çalışır; React Native desteği üzerinde çalışılıyor.

### Fragment Refs

`<Fragment>`'a doğrudan ref verilir; ref bir **`FragmentInstance`** döndürür:

```jsx
import { Fragment, useEffect, useRef } from "react";

function GorunurlukIzleyici({ onDegis, children }) {
  const ref = useRef(null);

  useEffect(() => {
    const fragment = ref.current;
    const gozlemci = new IntersectionObserver((girisler) => {
      onDegis(girisler.some((g) => g.isIntersecting));
    });
    fragment.observeUsing(gozlemci);              // tüm çocukları gözle
    return () => fragment.unobserveUsing(gozlemci);
  }, [onDegis]);

  return <Fragment ref={ref}>{children}</Fragment>;   // fazladan <div> YOK
}
```

| Metot grubu | Görevi |
|---|---|
| `addEventListener`, `removeEventListener`, `dispatchEvent` | Birinci seviye çocukların olayları |
| `focus`, `focusLast`, `blur` | Odağı çocuklar arasında derinlik öncelikli taşır |
| `observeUsing`, `unobserveUsing` | `IntersectionObserver` / `ResizeObserver` bağlar |
| `getClientRects`, `getRootNode`, `compareDocumentPosition`, `scrollIntoView` | Ölçüm ve kaydırma |

> Çocukların `ref` açmasına veya sarmalayıcı `<div>` eklemeye gerek kalmadan davranış eklenir; üçüncü parti bileşenler için çok kullanışlıdır.

### `browser()`: Sadece tarayıcıda render

SSR yapan uygulamalarda, sunucuda anlamlı çıktı üretemeyen bileşenler (`localStorage`, yerel saat dilimi) için:

```jsx
import { Suspense, use } from "react";
import { browser } from "react-dom";

function SaatDilimi() {
  use(browser());   // sunucuda askıya al
  const dilim = new Intl.DateTimeFormat().resolvedOptions().timeZone;
  return <p>{dilim}</p>;
}

<Suspense fallback="Yükleniyor...">
  <SaatDilimi />
</Suspense>
```

- Sunucuda Suspense'i tetikler (HTML'de fallback görünür), istemcide tetiklemez.
- Eski çözümlerin (`mounted` state + effect, `typeof window` kontrolü) yerini alır.
- `use` olduğu için koşul içinde çağrılabilir.

> Saf istemci tarafı Vite uygulamalarında gerekmez; SSR yapan ortamlarda (Next.js vb.) kullanılır *(Konu 23)*.

### Diğer 19.3 değişiklikleri

- **Trusted Types desteği:** Tarayıcının Trusted Types API'siyle entegrasyon; DOM tabanlı XSS saldırılarına karşı ek güvenlik katmanı.
- **Bağımsız transition'lar:** Yavaş bir transition ilgisiz diğerlerini bekletmez.
- **Server Components** `'use client'` modülünden aktarılan Context'i sarmalayıcı olmadan doğrudan render edebilir.

---

## 12. Kaldırılan ve Kullanımdan Kalkan API'ler

### React 19'da kaldırılanlar

| Kaldırılan | Yerine | Konu |
|---|---|---|
| `ReactDOM.render`, `ReactDOM.hydrate` | `createRoot(...).render()`, `hydrateRoot` | 1.6 |
| `ReactDOM.unmountComponentAtNode` | `root.unmount()` | — |
| `ReactDOM.findDOMNode` | Ref'ler | 11 |
| Fonksiyon bileşenlerinde `propTypes` | TypeScript | 4.13 |
| Fonksiyon bileşenlerinde `defaultProps` | Varsayılan parametreler | 4.4 |
| String ref'ler (`ref="input"`) | `useRef` / ref callback | 11 |
| Eski context API (`contextTypes`, `getChildContext`) | `createContext` | 13 |
| `react-dom/test-utils` içindeki `act` | `react`'ten `act` / Testing Library | 21 |

### Kullanımdan kalkanlar (çalışır ama yeni kodda kullanılmaz)

- `forwardRef` → `ref` prop
- `<Context.Provider>` → `<Context>`
- `element.ref` erişimi
- `react-test-renderer` → Testing Library

### Otomatik yükseltme

```bash
npx codemod@latest react/19/migration-recipe
```

---

## 13. Her Şey Bir Arada: Etkinlik Kayıt Uygulaması

Ek kütüphane olmadan: Actions, `useActionState`, `useFormStatus`, `useOptimistic`, `use`, `<Activity>`, `<ViewTransition>` + `addTransitionType`, Fragment ref ve metadata.

### Sahte sunucu

```js
// ============ src/api.js ============
const bekle = (ms) => new Promise((r) => setTimeout(r, ms));

let katilimcilar = [
  { id: "1", ad: "Ayşe Yılmaz", email: "ayse@ornek.com" },
  { id: "2", ad: "Mehmet Kaya", email: "mehmet@ornek.com" },
];

export async function etkinlikGetir() {
  await bekle(800);
  return {
    baslik: "React İstanbul Buluşması",
    tarih: "2026-11-14T19:00:00+03:00",
    yer: "Kadıköy, İstanbul",
    kontenjan: 30,
    aciklama: "React 19.3 yenilikleri, View Transitions ve Actions üzerine sohbet.",
  };
}

export async function katilimcilariGetir() {
  await bekle(400);
  return [...katilimcilar];
}

export async function kayitOl({ ad, email }) {
  await bekle(1000);
  if (katilimcilar.some((k) => k.email === email)) {
    throw new Error("Bu e-posta ile zaten kayıt olunmuş.");
  }
  if (Math.random() < 0.15) throw new Error("Sunucu yanıt vermedi, tekrar dene.");   // %15 hata
  const yeni = { id: crypto.randomUUID(), ad, email };
  katilimcilar = [...katilimcilar, yeni];
  return yeni;
}

export async function kayitSil(id) {
  await bekle(600);
  katilimcilar = katilimcilar.filter((k) => k.id !== id);
}
```

### View Transition animasyonları

```css
/* ============ src/gecisler.css ============ */
::view-transition-new(.listeye-gir) { animation: 300ms ease-out both listeye-gir; }
::view-transition-old(.listeden-cik) { animation: 200ms ease-in both listeden-cik; }

::view-transition-new(.sagdan-gir) { animation: 250ms ease-out both sagdan-gir; }
::view-transition-old(.sola-cik) { animation: 200ms ease-in both sola-cik; }
::view-transition-new(.soldan-gir) { animation: 250ms ease-out both soldan-gir; }
::view-transition-old(.saga-cik) { animation: 200ms ease-in both saga-cik; }

@keyframes listeye-gir { from { opacity: 0; transform: translateY(-8px) scale(0.98); } }
@keyframes listeden-cik { to { opacity: 0; transform: translateX(24px); } }
@keyframes sagdan-gir { from { opacity: 0; transform: translateX(32px); } }
@keyframes sola-cik { to { opacity: 0; transform: translateX(-32px); } }
@keyframes soldan-gir { from { opacity: 0; transform: translateX(-32px); } }
@keyframes saga-cik { to { opacity: 0; transform: translateX(32px); } }

@media (prefers-reduced-motion: reduce) {
  ::view-transition-group(*),
  ::view-transition-old(*),
  ::view-transition-new(*) { animation: none !important; }
}
```

### Uygulama

```jsx
// ============ src/EtkinlikUygulamasi.jsx ============
import {
  Activity, addTransitionType, Fragment, Suspense, startTransition,
  use, useActionState, useEffect, useOptimistic, useRef, useState, ViewTransition,
} from "react";
import { useFormStatus } from "react-dom";
import { etkinlikGetir, katilimcilariGetir, kayitOl, kayitSil } from "./api";
import "./gecisler.css";

// Promise'ler render sırasında DEĞİL, modül yüklenirken bir kez oluşturuluyor
const etkinlikPromise = etkinlikGetir();
const ilkKatilimcilarPromise = katilimcilariGetir();

// ---------- Tekrar kullanılabilir gönder butonu (useFormStatus) ----------
function GonderButonu({ children, bekleyenMetin }) {
  const { pending } = useFormStatus();
  return (
    <button
      type="submit"
      disabled={pending}
      aria-busy={pending}
      className="rounded bg-indigo-600 px-4 py-2 text-white disabled:opacity-50"
    >
      {pending ? bekleyenMetin : children}
    </button>
  );
}

// ---------- Etkinlik bilgisi (use + Suspense + metadata) ----------
function EtkinlikBilgisi() {
  const etkinlik = use(etkinlikPromise);
  const tarih = new Date(etkinlik.tarih).toLocaleString("tr-TR", {
    dateStyle: "full",
    timeStyle: "short",
  });

  return (
    <header className="mb-6">
      <title>{`${etkinlik.baslik} · Kayıt`}</title>
      <meta name="description" content={etkinlik.aciklama} />
      <h1 className="text-3xl font-bold">{etkinlik.baslik}</h1>
      <p className="text-gray-600">📅 {tarih} · 📍 {etkinlik.yer}</p>
      <p className="mt-2">{etkinlik.aciklama}</p>
    </header>
  );
}

// ---------- Kayıt formu (useActionState + form action) ----------
function KayitFormu({ onKayit }) {
  const [sonuc, formAction] = useActionState(
    async (onceki, formData) => {
      const ad = String(formData.get("ad") ?? "").trim();
      const email = String(formData.get("email") ?? "").trim().toLocaleLowerCase("tr");

      const hatalar = {};
      if (ad.length < 3) hatalar.ad = "Ad soyad en az 3 karakter olmalı.";
      if (!/^\S+@\S+\.\S+$/.test(email)) hatalar.email = "Geçerli bir e-posta girin.";
      if (Object.keys(hatalar).length) return { durum: "hata", hatalar, degerler: { ad, email } };

      const hataMesaji = await onKayit({ ad, email });   // iyimser ekleme + sunucu
      if (hataMesaji) {
        return { durum: "hata", hatalar: { genel: hataMesaji }, degerler: { ad, email } };
      }
      return { durum: "basarili", mesaj: `${ad}, kaydın alındı! 🎉` };
    },
    { durum: "bosta" }
  );

  return (
    <form action={formAction} className="space-y-3 rounded-lg border p-4">
      <h2 className="text-lg font-semibold">Kayıt ol</h2>

      <label className="block">
        <span className="text-sm font-medium">Ad soyad</span>
        <input
          name="ad"
          defaultValue={sonuc.degerler?.ad}
          className="mt-1 w-full rounded border px-3 py-2"
        />
        {sonuc.hatalar?.ad && <span className="text-sm text-red-600">{sonuc.hatalar.ad}</span>}
      </label>

      <label className="block">
        <span className="text-sm font-medium">E-posta</span>
        <input
          name="email"
          type="email"
          defaultValue={sonuc.degerler?.email}
          className="mt-1 w-full rounded border px-3 py-2"
        />
        {sonuc.hatalar?.email && <span className="text-sm text-red-600">{sonuc.hatalar.email}</span>}
      </label>

      {sonuc.hatalar?.genel && (
        <p role="alert" className="text-sm text-red-600">{sonuc.hatalar.genel}</p>
      )}
      {sonuc.durum === "basarili" && (
        <p role="status" className="text-sm text-green-700">{sonuc.mesaj}</p>
      )}

      <GonderButonu bekleyenMetin="Kaydediliyor...">Kayıt ol</GonderButonu>
    </form>
  );
}

// ---------- Katılımcı listesi (ViewTransition + Fragment ref) ----------
function KatilimciListesi({ katilimcilar, onSil }) {
  const listeRef = useRef(null);
  const [gorunur, setGorunur] = useState(true);

  // Fragment ref: sarmalayıcı eklemeden tüm satırların görünürlüğünü izle
  useEffect(() => {
    const fragment = listeRef.current;
    if (!fragment) return;
    const gozlemci = new IntersectionObserver((girisler) =>
      setGorunur(girisler.some((g) => g.isIntersecting))
    );
    fragment.observeUsing(gozlemci);
    return () => fragment.unobserveUsing(gozlemci);
  }, []);

  return (
    <section>
      <h2 className="mb-2 text-lg font-semibold">
        Katılımcılar ({katilimcilar.length}){" "}
        {!gorunur && <small className="text-gray-500">— liste ekran dışında</small>}
      </h2>
      <ul className="divide-y rounded-lg border">
        <Fragment ref={listeRef}>
          {katilimcilar.map((k) => (
            // key = e-posta: iyimser satır gerçeğiyle değişince tekrar animasyon olmasın
            <ViewTransition key={k.email} enter="listeye-gir" exit="listeden-cik">
              <li
                className="flex items-center justify-between px-3 py-2"
                style={{ opacity: k.bekliyor ? 0.5 : 1 }}
              >
                <span>
                  {k.ad} <small className="text-gray-500">{k.email}</small>
                  {k.bekliyor && <small className="ml-2 text-indigo-600">kaydediliyor...</small>}
                </span>
                {!k.bekliyor && (
                  <form action={() => onSil(k.id)}>
                    <button
                      className="text-sm text-red-600 hover:underline"
                      aria-label={`${k.ad} kaydını sil`}
                    >
                      Sil
                    </button>
                  </form>
                )}
              </li>
            </ViewTransition>
          ))}
        </Fragment>
      </ul>
    </section>
  );
}

// ---------- Katılım bölümü (useOptimistic + Actions) ----------
function KatilimBolumu() {
  const ilk = use(ilkKatilimcilarPromise);
  const [katilimcilar, setKatilimcilar] = useState(ilk);

  const [gorunenler, iyimserGuncelle] = useOptimistic(katilimcilar, (mevcut, islem) =>
    islem.tur === "ekle"
      ? [...mevcut, { ...islem.katilimci, bekliyor: true }]
      : mevcut.filter((k) => k.id !== islem.id)
  );

  async function handleKayit(veri) {
    iyimserGuncelle({ tur: "ekle", katilimci: { id: `gecici-${veri.email}`, ...veri } });
    try {
      const yeni = await kayitOl(veri);
      startTransition(() => setKatilimcilar((o) => [...o, yeni]));
      return null;
    } catch (err) {
      return err.message;   // iyimser satır action bitince kendiliğinden kaybolur
    }
  }

  async function handleSil(id) {
    iyimserGuncelle({ tur: "sil", id });
    await kayitSil(id);
    startTransition(() => setKatilimcilar((o) => o.filter((k) => k.id !== id)));
  }

  return (
    <div className="grid gap-6 md:grid-cols-2">
      <KayitFormu onKayit={handleKayit} />
      <KatilimciListesi katilimcilar={gorunenler} onSil={handleSil} />
    </div>
  );
}

// ---------- Program (addTransitionType ile yön bilgili geçiş) ----------
const PROGRAM = [
  { saat: "19:00", baslik: "Açılış ve tanışma" },
  { saat: "19:20", baslik: "React 19.3: View Transitions" },
  { saat: "20:00", baslik: "Actions ile formlar" },
  { saat: "20:40", baslik: "Soru-cevap ve networking" },
];

function Program() {
  const [adim, setAdim] = useState(0);

  function git(yon) {
    startTransition(() => {
      addTransitionType(yon);
      setAdim((a) =>
        yon === "ileri" ? Math.min(a + 1, PROGRAM.length - 1) : Math.max(a - 1, 0)
      );
    });
  }

  const oturum = PROGRAM[adim];

  return (
    <section className="rounded-lg border p-4">
      <div className="mb-3 flex items-center justify-between">
        <button onClick={() => git("geri")} disabled={adim === 0}>← Önceki</button>
        <span className="text-sm text-gray-500">{adim + 1} / {PROGRAM.length}</span>
        <button onClick={() => git("ileri")} disabled={adim === PROGRAM.length - 1}>
          Sonraki →
        </button>
      </div>
      <ViewTransition
        key={adim}
        enter={{ ileri: "sagdan-gir", geri: "soldan-gir" }}
        exit={{ ileri: "sola-cik", geri: "saga-cik" }}
      >
        <div className="py-6 text-center">
          <p className="text-3xl font-bold tabular-nums">{oturum.saat}</p>
          <p className="text-lg">{oturum.baslik}</p>
        </div>
      </ViewTransition>
    </section>
  );
}

// ---------- Ana uygulama (Activity ile sekmeler) ----------
export default function EtkinlikUygulamasi() {
  const [sekme, setSekme] = useState("kayit");

  return (
    <main className="mx-auto max-w-4xl p-6">
      <Suspense fallback={<div className="mb-6 h-24 animate-pulse rounded bg-gray-100" />}>
        <EtkinlikBilgisi />
      </Suspense>

      <nav className="mb-4 flex gap-2" role="tablist">
        {[["kayit", "Kayıt"], ["program", "Program"]].map(([id, etiket]) => (
          <button
            key={id}
            role="tab"
            aria-selected={sekme === id}
            onClick={() => startTransition(() => setSekme(id))}
            className="rounded px-3 py-1.5 aria-selected:bg-gray-900 aria-selected:text-white"
          >
            {etiket}
          </button>
        ))}
      </nav>

      {/* Activity: gizli sekmenin state'i (yarım form, program adımı) korunur */}
      <Activity mode={sekme === "kayit" ? "visible" : "hidden"}>
        <Suspense fallback={<p>Katılımcılar yükleniyor...</p>}>
          <KatilimBolumu />
        </Suspense>
      </Activity>

      <Activity mode={sekme === "program" ? "visible" : "hidden"}>
        <Program />
      </Activity>
    </main>
  );
}
```

### Dene ve gözlemle

| Deneme | Gözlenen | Özellik |
|---|---|---|
| Sayfayı aç | Etkinlik bilgisi ve katılımcılar bağımsız yüklenir; sekme başlığı değişir | `use` + Suspense + `<title>` |
| Formu boş gönder | Hatalar görünür, yazılanlar kaybolmaz | `useActionState` |
| Gönderirken | Buton "Kaydediliyor..." olur (props almadan) | `useFormStatus` |
| Geçerli kayıt gönder | Satır anında soluk eklenir, sunucu cevabıyla kalıcı olur | `useOptimistic` |
| Simüle hata (%15) | Satır kendiliğinden kaybolur, formda hata mesajı | `useOptimistic` (otomatik geri alma) |
| Kayıt ekle / sil | Satırlar kayarak girer / çıkar | `<ViewTransition>` |
| Program'da ileri / geri | Animasyon yönleri farklı | `addTransitionType` |
| Formda yaz → Program'a geç → geri dön | Yazılanlar ve program adımı yerinde | `<Activity>` |
| Listeyi ekran dışına kaydır | Başlıkta "liste ekran dışında" | Fragment ref |
| İşletim sisteminde "hareketi azalt" | Animasyonlar kapanır | `prefers-reduced-motion` |

> Bu uygulamada **effect ile veri çekme yok**, elle `isPending`/`yukleniyor` state'i yok, iyimser güncelleme için **kütüphane yok.**

---

## 14. Özet

- React 19, form gönderimi ve async işlemlerdeki tekrar eden kalıpları (bekleme, hata, sıfırlama, iyimser güncelleme) **çerçeveye** taşıdı.
- **Action** = async transition; `startTransition(async () => ...)` ile `isPending` bitene kadar `true`; `await` sonrası güncellemeler tekrar `startTransition` ile sarılır.
- **`<form action={fn}>`**: `preventDefault` yok, fonksiyon `FormData` alır, uncontrolled alanlar otomatik sıfırlanır; `formAction`, `requestFormReset`.
- **`useActionState(fn, baslangic)`** → `[durum, formAction, isPending]`; `fn(oncekiDurum, formData)`; doğrulama hataları ve sonuçlar için ideal (eski adı `useFormState`).
- **`useFormStatus`** (`react-dom`): form içindeki bileşenlerin formun `pending`, `data` bilgisini props'suz okumasını sağlar.
- **`useOptimistic(state, birlestir)`**: Action sürerken iyimser değer, bitince gerçek değer; hata olursa otomatik geri döner.
- **`use`**: Promise ve Context okur; koşullu çağrılabilir; Suspense ve Error Boundary ile çalışır; **Promise render'da oluşturulmaz.**
- `<title>`, `<meta>`, `<link>` bileşen içinde render edilir; `precedence`'li stil dosyaları yüklenene kadar içerik bekletilir; async script'ler tekilleştirilir.
- **Küçük iyileştirmeler:** `ref` prop, ref cleanup, `<Context value>`, `useDeferredValue` başlangıç değeri, daha iyi hydration hataları; `onCaughtError`/`onUncaughtError` ile merkezi hata raporlama.
- **19.2 — `<Activity>`**: gizlenen içerik state'ini korur, effect'leri temizlenir, düşük öncelikle güncellenir ve önceden render edilebilir; **Performance Tracks** DevTools'a React izleri ekler.
- **19.3 — `<ViewTransition>`**: enter/exit/update/share animasyonları, yalnızca transition'larla tetiklenir; `addTransitionType` ile yöne göre animasyon; Suspense ile koordineli yükleme; `prefers-reduced-motion` CSS'te desteklenmeli.
- **19.3 — Fragment refs**: `<Fragment ref>` → `FragmentInstance` (olaylar, odak, observer'lar, ölçüm) sarmalayıcısız.
- **19.3 — `browser()`**: `use(browser())` ile bileşen sunucu render'ından çıkar; **Trusted Types** desteği; bağımsız transition'lar.
- Kaldırılanlar: `ReactDOM.render`, `findDOMNode`, fonksiyonlarda `propTypes`/`defaultProps`, string ref'ler, eski context; kullanımdan kalkanlar: `forwardRef`, `<Context.Provider>`, `react-test-renderer`; codemod ile otomatik yükseltme.

---

⬅️ Önceki: [19 · Performans](./19-performans.md) · ➡️ Sonraki: [21 · Test](./21-test.md)
