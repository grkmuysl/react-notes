# 19 · Performans

> **Konu özeti:** Performansın sezgisel çerçevesi, render ne zaman ve ne kadar pahalı, ölçüm araçları ve Core Web Vitals, yapısal optimizasyonlar (colocation, children), `memo`, `useMemo`, `useCallback` ve ne zaman kullanılacakları, React Compiler, `useTransition` ve `useDeferredValue`, sanallaştırma, code splitting ve bundle analizi, görseller ve kaynak ipuçları, sık karşılaşılan sorunlar ve kontrol listesi.

---

## İçindekiler

1. [Sezgisel Bakış](#1-sezgisel-bakış)
2. [Ne Zaman Render Olur, Ne Kadar Pahalı?](#2-ne-zaman-render-olur-ne-kadar-pahalı)
3. [Önce Ölç](#3-önce-ölç-)
4. [En Ucuz Optimizasyon: Yapıyı Düzeltmek](#4-en-ucuz-optimizasyon-yapıyı-düzeltmek-)
5. [`memo`](#5-memo-props-değişmediyse-render-etme)
6. [`useMemo`](#6-usememo-hesaplamanın-sonucunu-saklamak)
7. [`useCallback`](#7-usecallback-fonksiyonu-saklamak)
8. [Ne Zaman Memoize Etmeli?](#8-ne-zaman-memoize-etmeli)
9. [React Compiler](#9-react-compiler-otomatik-memoizasyon-)
10. [`useTransition` ve `useDeferredValue`](#10-eşzamanlı-özellikler-usetransition-ve-usedeferredvalue-)
11. [Büyük Listeler: Sanallaştırma](#11-büyük-listeler-sanallaştırma)
12. [Yükleme Performansı: Code Splitting](#12-yükleme-performansı-code-splitting-ve-bundle-boyutu)
13. [Görseller ve Ağ](#13-görseller-ve-ağ)
14. [Sık Karşılaşılan Sorunlar](#14-sık-karşılaşılan-sorunlar)
15. [Performans Kontrol Listesi](#15-performans-kontrol-listesi)
16. [Her Şey Bir Arada: Hızlı Ürün Gezgini](#16-her-şey-bir-arada-hızlı-ürün-gezgini)
17. [Özet](#17-özet)

---

## 1. Sezgisel Bakış

> **Performans optimizasyonu = Gereksiz iş yapmamak.**

React'te "iş" üç yerde yapılır:

1. **Render:** Bileşen fonksiyonlarının çalışması
2. **Commit:** Değişikliklerin DOM'a uygulanması, ekranın yeniden çizilmesi
3. **Ağ ve yükleme:** JS, görsel ve veri indirmek

### Mutfak benzetmesi

| Sorun | Benzetme | Araç |
|---|---|---|
| Gereksiz render | Sipariş yokken yemeği baştan pişirmek | `memo`, state'i doğru yere koymak, React Compiler |
| Pahalı hesaplama | Her siparişte aynı sosu sıfırdan yapmak | `useMemo` |
| Acil olmayan iş | Önce suyu getir, tatlıyı sonra hazırla | `useTransition`, `useDeferredValue` |
| Çok büyük liste | 10.000 kişilik menüyü tek seferde masaya koymak | Sanallaştırma |
| Ağır yükleme | Açılışta tüm malzemeleri getirmek | Code splitting, lazy loading |

> **Önce ölç, sonra optimize et.** React varsayılan olarak hızlıdır; çoğu uygulamada optimizasyona gerek yoktur. Her yere `useMemo` serpiştirmek kodu karmaşıklaştırır, çoğu zaman bir şey kazandırmaz.

---

## 2. Ne Zaman Render Olur, Ne Kadar Pahalı?

### Render tetikleyicileri

1. Kendi **state**'i değişince
2. **Ebeveyni** render olunca (props değişmese bile!) ← en sık sorun kaynağı
3. Okuduğu **context** değişince

### Render çoğu zaman ucuzdur

Render ≠ DOM güncellemesi. Değişmeyen kısımlar DOM'a dokunulmaz; birkaç yüz basit bileşenin render'ı milisaniyeler sürer.

### Render ne zaman sorun olur?

- Bileşende **pahalı bir hesaplama** varsa (binlerce elemanı filtrelemek/sıralamak)
- Çok **sayıda** bileşen aynı anda render oluyorsa (büyük tablolar, uzun listeler)
- Render **çok sık** tetikleniyorsa (her tuş, her fare hareketi, her animasyon karesi)

> Kullanıcı tarafında: input'un takılması, tıklamaya geç tepki, kaydırmanın kasması.

---

## 3. Önce Ölç ⭐

### React DevTools: Profiler

1. Kayıt (⏺) → 2. Yavaş işlemi yap → 3. Durdur

| Görünüm | Ne gösterir |
|---|---|
| **Flame graph** | Her commit'te hangi bileşenlerin render olduğu ve süresi (gri = render olmadı) |
| **"Why did this render?"** | Ayarlardan açılır: "Props changed", "Parent rendered", "Hook changed" |
| **Ranked chart** | En yavaş bileşenler |

> **"Highlight updates when components render"** açılırsa render olan bileşenler yanıp söner. Yazarken sayfanın yarısı yanıp sönüyorsa gereksiz render vardır.

### Tarayıcı araçları

- **Chrome DevTools → Performance:** Ana iş parçacığı ve uzun görevler (50 ms+)
- **Lighthouse:** Genel puan ve öneriler

### Core Web Vitals

| Ölçüt | Ne ölçer | İyi değer |
|---|---|---|
| **LCP** (Largest Contentful Paint) | Ana içeriğin görünme süresi | ≤ 2.5 sn |
| **INP** (Interaction to Next Paint) | Etkileşimlere tepki süresi | ≤ 200 ms |
| **CLS** (Cumulative Layout Shift) | Yüklenirken içeriğin kayması | ≤ 0.1 |

> INP, React performansıyla en doğrudan ilişkili ölçüttür.

### ⚠️ Doğru ölçüm

- `npm run dev` modunda her şey daha yavaştır (ek kontroller, StrictMode çift render).
- Gerçekçi ölçüm: **`npm run build && npm run preview`**
- Chrome DevTools'ta **CPU throttling** (4x–6x) ile orta seviye telefonu simüle et.

---

## 4. En Ucuz Optimizasyon: Yapıyı Düzeltmek ⭐

Gereksiz render sorunlarının çoğu **state'in yanlış yerde olmasından** kaynaklanır ve hiçbir özel API kullanmadan çözülür.

### Sorunlu örnek

```jsx
function App() {
  const [arama, setArama] = useState("");

  return (
    <div>
      <input value={arama} onChange={(e) => setArama(e.target.value)} />
      <AramaSonucu metin={arama} />
      <YavasGrafik />     {/* 200 ms süren, aramayla ilgisiz bileşen */}
      <YavasTablo />
    </div>
  );
}
```

Her harfte `App` → `YavasGrafik` ve `YavasTablo` da render olur → input takılır.

### Çözüm 1: State'i aşağı taşı (colocation)

```jsx
function AramaBolumu() {
  const [arama, setArama] = useState("");   // state buraya taşındı
  return (
    <>
      <input value={arama} onChange={(e) => setArama(e.target.value)} />
      <AramaSonucu metin={arama} />
    </>
  );
}

function App() {
  return (
    <div>
      <AramaBolumu />
      <YavasGrafik />     {/* artık arama değişince render OLMAZ */}
      <YavasTablo />
    </div>
  );
}
```

### Çözüm 2: İçeriği yukarı kaldır (children)

State, pahalı bileşenleri **saran** bileşende olmak zorundaysa:

```jsx
// ❌ Kaydırdıkça YavasIcerik de render olur
function KaydirmaKabi() {
  const [kaydirildi, setKaydirildi] = useState(false);
  return (
    <div className={kaydirildi ? "golgeli" : ""} onScroll={/* ... */}>
      <YavasIcerik />
    </div>
  );
}
```

```jsx
// ✅ Pahalı içerik children olarak dışarıdan geliyor
function KaydirmaKabi({ children }) {
  const [kaydirildi, setKaydirildi] = useState(false);
  return (
    <div className={kaydirildi ? "golgeli" : ""} onScroll={/* ... */}>
      {children}
    </div>
  );
}

function App() {
  return (
    <KaydirmaKabi>
      <YavasIcerik />   {/* App'te oluşturuldu; KaydirmaKabi render olsa da render OLMAZ */}
    </KaydirmaKabi>
  );
}
```

- `KaydirmaKabi` kendi state'iyle render olduğunda `children` **aynı element nesnesidir**; React o alt ağacı yeniden render etmez.

> Bu iki teknik çoğu sorunu kodu **daha temiz** hale getirerek çözer; `memo`'dan önce düşünülmelidir.

---

## 5. `memo`: Props Değişmediyse Render Etme

```jsx
import { memo } from "react";

const UrunSatiri = memo(function UrunSatiri({ urun, secili }) {
  return <li className={secili ? "secili" : ""}>{urun.ad}</li>;
});
```

- Props'lar **sığ (shallow)** karşılaştırılır (her prop `Object.is` ile).
- Hepsi aynıysa render **atlanır.**

```jsx
function UrunListesi({ urunler }) {
  const [seciliId, setSeciliId] = useState(null);
  return (
    <ul>
      {urunler.map((u) => (
        <UrunSatiri key={u.id} urun={u} secili={u.id === seciliId} />
      ))}
    </ul>
  );
}
```

> Bir ürün seçilince 1.000 satırın değil, sadece `secili` prop'u değişen **iki** satırın render'ı yapılır.

### ⚠️ `memo`'yu bozan şey: Her render'da yeni nesne/fonksiyon

```jsx
<UrunSatiri
  key={u.id}
  urun={u}
  secili={u.id === seciliId}
  onSec={() => setSeciliId(u.id)}   // ❌ her render'da YENİ fonksiyon
  stil={{ padding: 8 }}              // ❌ her render'da YENİ nesne
/>
```

`Object.is` false döner → `memo` her seferinde "değişti" der → kazanç yok, üstüne karşılaştırma maliyeti.

### Çözümler

- Sabit nesneleri bileşen **dışına** taşı: `const SATIR_STILI = { padding: 8 };`
- Fonksiyonları **`useCallback`** ile sabitle
- Çocuğa id'yi ver, **sabit** bir fonksiyon geçir:

```jsx
const handleSec = useCallback((id) => setSeciliId(id), []);
<UrunSatiri urun={u} onSec={handleSec} />

// UrunSatiri içinde:
<li onClick={() => onSec(urun.id)}>
```

- Mümkünse nesne yerine **primitive** props ver (`ust={...}`, `yukseklik={...}`) ve nesneyi çocuğun içinde oluştur.

> `children` da bir prop'tur ve JSX her render'da yeni element üretir; `children` alan bileşeni `memo` ile sarmak genelde işe yaramaz.

---

## 6. `useMemo`: Hesaplamanın Sonucunu Saklamak

```jsx
import { useMemo } from "react";

function UrunTablosu({ urunler, filtre, siralama }) {
  const gorunenler = useMemo(() => {
    return urunler
      .filter((u) => u.ad.toLocaleLowerCase("tr").includes(filtre.toLocaleLowerCase("tr")))
      .toSorted(siralamaFonksiyonu(siralama));
  }, [urunler, filtre, siralama]);
  // başka state'ler değişse bile urunler/filtre/siralama aynıysa tekrar hesaplanmaz
}
```

```js
const deger = useMemo(() => hesapla(a, b), [a, b]);
//                     │                    │
//                     │                    └ bağımlılıklar (effect kurallarıyla aynı)
//                     └ saf bir hesaplama fonksiyonu
```

### İki kullanım amacı

#### 1. Gerçekten pahalı hesaplamalar

```jsx
console.time("filtrele");
const sonuc = urunler.filter(/* ... */);
console.timeEnd("filtrele");   // 0.3ms → gereksiz; 15ms → gerekli olabilir
```

> Kabaca: **1 ms'den uzun** süren ve sık render olan bileşendeki hesaplama adaydır.

#### 2. Referansı sabitlemek

```jsx
const secenekler = useMemo(() => ({ sunucuUrl, odaId }), [sunucuUrl, odaId]);
<SohbetPaneli secenekler={secenekler} />   // memo'lu çocuk gereksiz render olmaz
```

> ⚠️ `useMemo` bir **optimizasyon ipucudur**, garanti değil. Kodun doğruluğu ona bağlı olmamalı; içinde **yan etki yapılmaz.**

---

## 7. `useCallback`: Fonksiyonu Saklamak

```jsx
const handleSec = useCallback((id) => {
  setSeciliId(id);
}, []);   // bağımlılık yok → fonksiyon hiç değişmez
```

```js
useCallback(fn, bagimliliklar)  ===  useMemo(() => fn, bagimliliklar)
```

### Ne zaman?

Sadece fonksiyonun **referansının** önemli olduğu yerlerde:

- `memo`'lu bir çocuğa prop olarak veriliyorsa
- Bir effect'in veya başka bir `useMemo`/`useCallback`'in bağımlılığıysa
- Custom Hook'tan döndürülüyorsa

```jsx
// ❌ Gereksiz: sıradan <button>'a veriliyor
const handleClick = useCallback(() => setAcik(true), []);
<button onClick={handleClick}>Aç</button>
```

---

## 8. Ne Zaman Memoize Etmeli?

`memo`, `useMemo`, `useCallback` **bedava değildir**: karşılaştırma maliyeti, bellek, daha uzun kod, yanlış bağımlılıkta bayat değer hataları.

| Durum | Öneri |
|---|---|
| Hesaplama ölçülebilir şekilde yavaş (1 ms+) | `useMemo` ✅ |
| Sık render olan, pahalı, props'u çoğu zaman aynı bileşen | `memo` ✅ (+ props sabitlenir) |
| Değer `memo`'lu bileşene veya effect bağımlılığına gidiyor | `useMemo` / `useCallback` ✅ |
| Basit bileşen, ucuz render | ❌ |
| Sıradan DOM elemanına verilen fonksiyon | ❌ |
| "Belki ileride yavaşlar" | ❌ Önce ölç |
| Yapı düzeltilebiliyorsa | ❌ Yapıyı düzelt |

---

## 9. React Compiler: Otomatik Memoizasyon ⭐

**React Compiler**, kodu **derleme sırasında** analiz edip `memo`/`useMemo`/`useCallback` işini **otomatik** yapar. **1.0 sürümü Ekim 2025'te kararlı** olarak yayınlandı.

```jsx
// Sen bunu yazarsın:
function UrunListesi({ urunler, filtre }) {
  const gorunenler = urunler.filter((u) => u.ad.includes(filtre));
  const handleSec = (id) => console.log(id);
  return gorunenler.map((u) => <UrunSatiri key={u.id} urun={u} onSec={handleSec} />);
}

// Compiler kabaca şunun eşdeğerini üretir:
//  - gorunenler: urunler/filtre değişmedikçe tekrar hesaplanmaz
//  - handleSec: render'lar arasında aynı referans
//  - her <UrunSatiri> elementi: props'u değişmedikçe yeniden oluşturulmaz
```

- Elle memoizasyondan **daha ince taneli**: JSX'in parçalarını ayrı ayrı, koşullu dallardan sonra bile saklayabilir.
- Compiler açıkken çoğu durumda `memo`, `useMemo`, `useCallback` yazmaya **gerek kalmaz.**

### Kurulum (Vite)

> Kurulum yöntemi araç sürümlerine göre değişti; eski eğitimlerdeki yapılandırmalar çalışmayabilir. **En güvenilir kaynak resmi dokümantasyondur** (react.dev, Vite plugin-react).

#### Vite 8 + `@vitejs/plugin-react` v6 (Babel tabanlı derleyici)

`@vitejs/plugin-react` v6, Babel yerine **oxc** kullanır; eski `react({ babel: {...} })` kalıbı Vite 8'de çalışmaz. Babel tabanlı derleyici için `@rolldown/plugin-babel` eklenir:

```bash
npm i -D babel-plugin-react-compiler @rolldown/plugin-babel
```

```js
// vite.config.js
import { defineConfig } from "vite";
import react, { reactCompilerPreset } from "@vitejs/plugin-react";
import babel from "@rolldown/plugin-babel";

export default defineConfig({
  plugins: [
    babel({ presets: [reactCompilerPreset()] }),   // react()'tan ÖNCE
    react(),
  ],
});
```

#### Vite 8 + `@vitejs/plugin-react` 6.1+ (Oxc ile yerleşik, deneysel)

- Ağustos 2026'dan itibaren React Compiler, Vite 8'de **Oxc üzerinden** yerleşik olarak kullanılabiliyor.
- `react({ compiler: true })` ayarıyla açılır; Babel'den çok daha hızlıdır.
- Henüz **deneysel**; kullanmadan önce dokümantasyon kontrol edilmeli.

### Compiler'ın koşulu: React kurallarına uymak

- Bileşenler ve Hook'lar **saf** (render'da yan etki yok)
- Props ve state **mutasyona uğratılmaz**
- Hook'lar sadece **en üst seviyede**
- Render sırasında **`ref.current`** okunmaz/yazılmaz

- Kural ihlali olan bileşenleri compiler **atlar** (optimize etmez ama bozmaz).
- İhlalleri görmek için compiler lint kuralları `eslint-plugin-react-hooks` içindedir:

```bash
npm install -D eslint-plugin-react-hooks@latest
```

### Kaçış kapısı

```jsx
function SorunluBilesen() {
  "use no memo";   // bu bileşeni derleme
  // ...
}
```

### Compiler neyi çözmez?

| Sorun | Çözüm |
|---|---|
| Gerçekten pahalı algoritmalar (bir kez bile yavaş) | Algoritmayı iyileştir |
| Çok büyük listelerin DOM maliyeti | Sanallaştırma |
| Bundle boyutu | Code splitting |
| Ağ yavaşlığı | Önbellek, önceden yükleme |
| Acil/acil olmayan güncellemeler | `useTransition`, `useDeferredValue` |

> **Yeni projede:** Compiler'ı aç, `memo`/`useMemo`/`useCallback`'i varsayılan olarak yazma, React kurallarına uy. Yine de bu API'leri bilmek gerekir: compiler kullanmayan projelerde ve eski kodlarda her yerdeler.

---

## 10. Eşzamanlı Özellikler: `useTransition` ve `useDeferredValue` ⭐

Bazen render azaltılamaz (10.000 satırı filtrelemek zaman alır). Ama **yazılan harfin** görünmesi, listenin güncellenmesinden daha **acildir.**

| Güncelleme türü | Örnek | Davranış |
|---|---|---|
| **Acil** | Yazmak, tıklamak | Anında |
| **Acil olmayan (transition)** | Sonuç listesi, grafik, sekme içeriği | Gecikebilir; yeni güncelleme gelirse **yarıda bırakılabilir** |

### `useDeferredValue`: Bir değerin gecikmeli kopyası

```jsx
import { memo, useDeferredValue, useState } from "react";

function Arama() {
  const [arama, setArama] = useState("");
  const gecikmeliArama = useDeferredValue(arama);   // acil olmayan kopya
  const eskiMi = arama !== gecikmeliArama;          // liste henüz güncellenmedi mi?

  return (
    <>
      <input value={arama} onChange={(e) => setArama(e.target.value)} />   {/* anında */}
      <div style={{ opacity: eskiMi ? 0.5 : 1 }}>
        <YavasListe sorgu={gecikmeliArama} />                              {/* gecikmeli */}
      </div>
    </>
  );
}

const YavasListe = memo(function YavasListe({ sorgu }) {
  const sonuclar = BUYUK_VERI.filter((x) => x.includes(sorgu));   // pahalı
  return <ul>{sonuclar.map((x) => <li key={x}>{x}</li>)}</ul>;
});
```

#### Akış

1. "a" yazılır → **acil** render: `arama = "a"`, `gecikmeliArama` hâlâ `""` → `YavasListe` atlanır (memo), input anında güncellenir.
2. Arka planda **düşük öncelikli** render: `gecikmeliArama = "a"` → liste filtrelenir.
3. Bu sırada "ab" yazılırsa yarım render **bırakılır**, yeniden başlar.

> ⚠️ Yavaş bileşen **`memo`'lu** olmalı (veya Compiler optimize etmeli); yoksa acil render'da da render olur, kazanç kaybolur.

> 💡 **`useDebounce` ile farkı:** Debounce sabit süre bekler; `useDeferredValue` cihazın hızına uyum sağlar ve bekleyen işi iptal edebilir. Ama **ağ isteği sayısını azaltmaz**; API isteklerini azaltmak için debounce hâlâ doğru araçtır.

### `useTransition`: Bir güncellemeyi "acil değil" olarak işaretlemek

```jsx
import { useState, useTransition } from "react";

function Sekmeler() {
  const [sekme, setSekme] = useState("ozet");
  const [isPending, startTransition] = useTransition();

  function sekmeSec(yeni) {
    startTransition(() => {
      setSekme(yeni);   // bu güncelleme acil değil
    });
  }

  return (
    <>
      <nav>
        {["ozet", "raporlar", "ayarlar"].map((s) => (
          <button key={s} onClick={() => sekmeSec(s)} aria-pressed={sekme === s}>
            {s}
          </button>
        ))}
        {isPending && <span>⏳</span>}
      </nav>
      <div style={{ opacity: isPending ? 0.6 : 1 }}>
        {sekme === "ozet" && <Ozet />}
        {sekme === "raporlar" && <AgirRaporlar />}   {/* 500 ms süren render */}
        {sekme === "ayarlar" && <Ayarlar />}
      </div>
    </>
  );
}
```

- `startTransition` içindeki güncellemeler **düşük öncelikli** render edilir.
- Ağır sekme render olurken başka tıklama gelirse yarım iş bırakılır; arayüz donmaz.
- Eski içerik geçiş bitene kadar ekranda kalır.

> ⚠️ Kontrollü input güncellemesi transition'a **konmaz**; input acil olmalıdır. Pahalı kısım `useDeferredValue` ile ertelenir.

### Karşılaştırma

| | `useTransition` | `useDeferredValue` |
|---|---|---|
| Neyi kontrol eder | Bir state **güncellemesini** | Bir **değeri** |
| Ne zaman | Güncellemeyi sen tetikliyorsan (tıklama, sekme) | Değer props'tan/başka Hook'tan geliyorsa |
| Gösterge | `isPending` | `deger !== gecikmeliDeger` |

> 📌 React Router geçişleri ve React 19 Actions da arka planda transition'ları kullanır *(Konu 20)*.

---

## 11. Büyük Listeler: Sanallaştırma

10.000 satırda sorun genellikle **DOM'un kendisidir**; `memo` bunu çözmez.

**Sanallaştırma:** Sadece ekranda **görünen** satırları render et, kaydırdıkça değiştir.

```bash
npm install @tanstack/react-virtual
```

```jsx
import { useRef } from "react";
import { useVirtualizer } from "@tanstack/react-virtual";

function SanalListe({ satirlar }) {
  const kapRef = useRef(null);

  const sanallastirici = useVirtualizer({
    count: satirlar.length,                  // toplam satır
    getScrollElement: () => kapRef.current,  // kaydırılan kap
    estimateSize: () => 36,                  // tahmini satır yüksekliği (px)
    overscan: 8,                             // görünürün dışına fazladan render
  });

  return (
    <div ref={kapRef} style={{ height: 500, overflowY: "auto" }}>
      {/* Toplam yükseklik: kaydırma çubuğu doğru görünsün */}
      <div style={{ height: sanallastirici.getTotalSize(), position: "relative" }}>
        {sanallastirici.getVirtualItems().map((sanal) => (
          <div
            key={sanal.key}
            style={{
              position: "absolute",
              top: 0,
              left: 0,
              width: "100%",
              height: sanal.size,
              transform: `translateY(${sanal.start}px)`,
            }}
          >
            {satirlar[sanal.index].ad}
          </div>
        ))}
      </div>
    </div>
  );
}
```

- 10.000 satırda DOM'da aynı anda ~25 satır bulunur.
- Değişken yükseklik, yatay listeler ve ızgaralar desteklenir.

> Önce sor: **"Kullanıcı gerçekten 10.000 satırı kaydıracak mı?"** Sayfalama, arama/filtre veya "daha fazla yükle" çoğu zaman daha basit ve daha iyidir.

---

## 12. Yükleme Performansı: Code Splitting ve Bundle Boyutu

Büyük bundle → indirme, ayrıştırma, çalıştırma gecikir → **LCP** kötüleşir (özellikle mobilde).

### `lazy` + `Suspense`: Bileşen düzeyinde bölme

```jsx
import { lazy, Suspense, useState } from "react";

const GrafikPaneli = lazy(() => import("./GrafikPaneli"));   // default export olmalı

function Panel() {
  const [grafikAcik, setGrafikAcik] = useState(false);

  return (
    <>
      <button onClick={() => setGrafikAcik(true)}>Grafikleri göster</button>
      {grafikAcik && (
        <Suspense fallback={<p>Grafik yükleniyor...</p>}>
          <GrafikPaneli />   {/* kodu sadece ilk gösterildiğinde indirilir */}
        </Suspense>
      )}
    </>
  );
}
```

- Kod ayrı bir dosyaya (chunk) bölünür; indirme sürerken `<Suspense>` fallback'i gösterilir.
- `lazy` bileşen **dışında**, modül seviyesinde çağrılır.

**İyi adaylar:** Grafik kütüphaneleri, zengin metin/kod editörleri, haritalar, PDF görüntüleyiciler, nadir açılan modallar ve ayar sayfaları.

### Üzerine gelince önceden yükleme

```jsx
const grafikYukle = () => import("./GrafikPaneli");
const GrafikPaneli = lazy(grafikYukle);

<button onMouseEnter={grafikYukle} onFocus={grafikYukle} onClick={() => setGrafikAcik(true)}>
```

### Ağır kütüphaneleri ihtiyaç anında yüklemek

```jsx
async function handleDisaAktar() {
  const { utils } = await import("xlsx");   // sadece butona basılınca indirilir
  const sayfa = utils.json_to_sheet(veriler);
  // ...
}
```

### Bundle'ı incelemek

```bash
npm run build
npx vite-bundle-visualizer
```

**Sık sürprizler:**

- Bütün bir yardımcı kütüphaneyi import etmek (`import _ from "lodash"`)
- İkon setlerini toptan import etmek
- Tarih kütüphaneleri ve yerelleştirme dosyaları (çoğu zaman `Intl` yeterli: `toLocaleDateString("tr-TR")`)

> 💡 Paket eklemeden önce boyutu: **bundlephobia.com**, **pkg-size.dev**

---

## 13. Görseller ve Ağ

Çoğu sayfada en büyük yük **görsellerdir.**

```jsx
<img
  src="/urun-800.webp"
  srcSet="/urun-400.webp 400w, /urun-800.webp 800w, /urun-1600.webp 1600w"
  sizes="(max-width: 640px) 100vw, 400px"
  width={800}
  height={600}
  alt="Kırmızı koşu ayakkabısı"
  loading="lazy"
  decoding="async"
/>
```

| Teknik | Faydası |
|---|---|
| `width`/`height` (veya `aspect-ratio`) | Yer ayrılır, sayfa kaymaz (**CLS**) |
| `loading="lazy"` | Ekran dışı görseller yaklaşınca indirilir |
| `srcSet` + `sizes` | Cihaza uygun boyut |
| WebP, AVIF | Aynı kalitede çok daha küçük dosya |
| Ana görselde `fetchPriority="high"` ve lazy **olmaması** | **LCP** iyileşir |

> ⚠️ Açılışta hemen görünen ana görsele `loading="lazy"` **verilmez.**

### React 19 kaynak ipuçları

```jsx
import { preconnect, preload, prefetchDNS } from "react-dom";

function UrunSayfasi() {
  preconnect("https://cdn.ornek.com");                       // bağlantıyı önceden kur
  preload("/fontlar/inter.woff2", { as: "font" });           // fontu önceden indir
  preload("/hero.webp", { as: "image", fetchPriority: "high" });
  // ...
}
```

- React bunları `<head>`'e uygun `<link>` etiketleri olarak ekler ve tekrarları birleştirir.

### Veri tarafı

Konu 17 ve 18'deki teknikler de performans teknikleridir: loader'larla şelaleleri önlemek, TanStack Query önbelleği, önceden yükleme, `keepPreviousData`.

---

## 14. Sık Karşılaşılan Sorunlar

| Sorun | Çözüm | Konu |
|---|---|---|
| Context değişince her şeyin render olması | `useMemo` ile değer sabitleme (veya Compiler), context'leri bölme, TanStack Query `select` | 13.8, 18 |
| Bileşen içinde bileşen tanımlamak | Bileşenleri en üst seviyede tanımla (performans **ve** doğruluk hatası) | 3.4 |
| Render'da oluşturulan key'ler (`Math.random()`) | Kimliği veriyle birlikte bir kez oluştur | 8.6 |
| Gereksiz effect zincirleri | Türetilebilen değeri render'da hesapla | 10.10 |
| `scroll`/`resize`/`mousemove`'da her seferinde state | Ref, `requestAnimationFrame`, CSS çözümleri, `IntersectionObserver` | 11 |
| Animasyonları JS state'iyle yapmak | CSS geçişleri veya Motion; `transform` ve `opacity` en ucuzudur | 16.11 |

---

## 15. Performans Kontrol Listesi

### Önce

- [ ] Gerçek bir sorun var mı? Production build + CPU yavaşlatmayla ölçtüm mü?
- [ ] Profiler'da hangi bileşenin, neden render olduğunu gördüm mü?

### Render performansı

- [ ] State, onu kullanan en alttaki yerde mi?
- [ ] Pahalı içerik `children` olarak yukarıdan verilebilir mi?
- [ ] React Compiler açık mı?
- [ ] (Compiler yoksa) Pahalı bileşen `memo`'lu ve props'u sabit mi?
- [ ] Gerçekten pahalı hesaplamalar `useMemo`'da mı?
- [ ] Acil olmayan güncellemeler `useTransition`/`useDeferredValue` ile ayrıldı mı?
- [ ] Binlerce satır varsa sayfalama veya sanallaştırma kullanıldı mı?

### Yükleme performansı

- [ ] Rotalar ve ağır bileşenler `lazy` ile bölündü mü?
- [ ] Bundle analizinde beklenmedik büyük paket var mı?
- [ ] Görsellerin boyutu, formatı, `width`/`height`'i, `loading`'i doğru mu?
- [ ] Veri için önbellek ve önceden yükleme kullanılıyor mu?

---

## 16. Her Şey Bir Arada: Hızlı Ürün Gezgini

20.000 ürünlük yerel veride: anında tepki veren arama (`useDeferredValue`), sanallaştırılmış liste, `useTransition` ile ağır sekme geçişi, tembel yüklenen istatistik paneli ve yapısal optimizasyonlar.

> React Compiler kullanılmadığı varsayılarak `memo`/`useCallback` elle yazılmıştır; Compiler açıksa bunlar silinebilir.

### Veri

```js
// ============ src/veri.js — 20.000 sahte ürün ============
const KATEGORILER = ["Elektronik", "Kitap", "Giyim", "Ev", "Spor", "Oyuncak"];
const SIFATLAR = ["Akıllı", "Klasik", "Hafif", "Profesyonel", "Mini", "Ekonomik", "Lüks", "Kablosuz"];
const ISIMLER = ["Saat", "Lamba", "Çanta", "Kulaklık", "Defter", "Ayakkabı", "Kupa", "Top", "Klavye", "Mont"];

export const URUNLER = Array.from({ length: 20000 }, (_, i) => ({
  id: i + 1,
  ad: `${SIFATLAR[i % SIFATLAR.length]} ${ISIMLER[(i * 7) % ISIMLER.length]} #${i + 1}`,
  kategori: KATEGORILER[(i * 3) % KATEGORILER.length],
  fiyat: Math.round(((i * 37) % 4900) + 100),
  puan: ((i * 13) % 50) / 10,
}));
// Veri modül seviyesinde BİR KEZ oluşturulur, render sırasında değil.
```

### Ana bileşen

```jsx
// ============ src/UrunGezgini.jsx ============
import {
  lazy, memo, Suspense, useCallback, useDeferredValue,
  useMemo, useRef, useState, useTransition,
} from "react";
import { useVirtualizer } from "@tanstack/react-virtual";
import { URUNLER } from "./veri";

const IstatistikPaneli = lazy(() => import("./IstatistikPaneli"));   // ağır, nadir açılan

// --- Saf yardımcı ---
function filtrele(urunler, sorgu, kategori) {
  const q = sorgu.toLocaleLowerCase("tr");
  return urunler.filter(
    (u) =>
      (kategori === "Tümü" || u.kategori === kategori) &&
      u.ad.toLocaleLowerCase("tr").includes(q)
  );
}

// --- Satır: memo + yalnızca primitive props + sabit callback ---
const UrunSatiri = memo(function UrunSatiri({ urun, secili, onSec, ust, yukseklik }) {
  return (
    <div
      style={{
        position: "absolute",
        top: 0,
        left: 0,
        width: "100%",
        height: yukseklik,
        transform: `translateY(${ust}px)`,
      }}
      className={`flex items-center justify-between border-b px-3 ${
        secili ? "bg-indigo-50 font-medium" : ""
      }`}
    >
      <button type="button" onClick={() => onSec(urun.id)} className="truncate text-left">
        {urun.ad}
      </button>
      <span className="shrink-0 text-sm tabular-nums text-gray-500">
        {urun.kategori} · {urun.fiyat.toLocaleString("tr-TR")} ₺
      </span>
    </div>
  );
});

// --- Sanallaştırılmış liste: memo'lu, gecikmeli sorguyla çalışır ---
const SanalUrunListesi = memo(function SanalUrunListesi({ sorgu, kategori, seciliId, onSec }) {
  const sonuclar = useMemo(() => filtrele(URUNLER, sorgu, kategori), [sorgu, kategori]);   // pahalı
  const kapRef = useRef(null);

  const sanal = useVirtualizer({
    count: sonuclar.length,
    getScrollElement: () => kapRef.current,
    estimateSize: () => 40,
    overscan: 10,
  });

  return (
    <>
      <p className="mb-2 text-sm text-gray-500">
        {sonuclar.length.toLocaleString("tr-TR")} sonuç
      </p>
      <div ref={kapRef} className="h-[480px] overflow-y-auto rounded border">
        <div style={{ height: sanal.getTotalSize(), position: "relative" }}>
          {sanal.getVirtualItems().map((v) => {
            const urun = sonuclar[v.index];
            return (
              <UrunSatiri
                key={urun.id}
                urun={urun}
                secili={urun.id === seciliId}
                onSec={onSec}
                ust={v.start}
                yukseklik={v.size}
              />
            );
          })}
        </div>
      </div>
    </>
  );
});

// --- Arama bölümü: state burada (colocation) ---
function AramaBolumu({ seciliId, onSec }) {
  const [arama, setArama] = useState("");
  const [kategori, setKategori] = useState("Tümü");
  const gecikmeliArama = useDeferredValue(arama);
  const gecikmeliKategori = useDeferredValue(kategori);
  const eski = arama !== gecikmeliArama || kategori !== gecikmeliKategori;

  return (
    <section>
      <div className="mb-3 flex gap-2">
        <input
          value={arama}
          onChange={(e) => setArama(e.target.value)}   // acil: input asla takılmaz
          placeholder="20.000 ürün içinde ara..."
          className="flex-1 rounded border px-3 py-2"
        />
        <select
          value={kategori}
          onChange={(e) => setKategori(e.target.value)}
          className="rounded border px-2"
        >
          {["Tümü", "Elektronik", "Kitap", "Giyim", "Ev", "Spor", "Oyuncak"].map((k) => (
            <option key={k}>{k}</option>
          ))}
        </select>
      </div>
      <div style={{ opacity: eski ? 0.6 : 1, transition: "opacity .15s" }}>
        <SanalUrunListesi
          sorgu={gecikmeliArama}
          kategori={gecikmeliKategori}
          seciliId={seciliId}
          onSec={onSec}
        />
      </div>
    </section>
  );
}

// --- Detay ---
function UrunDetay({ id }) {
  const urun = URUNLER[id - 1];
  if (!urun) return <p className="text-gray-500">Bir ürün seç.</p>;
  return (
    <div className="space-y-1">
      <h2 className="text-lg font-bold">{urun.ad}</h2>
      <p>{urun.kategori}</p>
      <p className="tabular-nums">
        {urun.fiyat.toLocaleString("tr-TR")} ₺ · ⭐ {urun.puan}
      </p>
    </div>
  );
}

// --- Ana bileşen ---
export default function UrunGezgini() {
  const [seciliId, setSeciliId] = useState(null);
  const [sekme, setSekme] = useState("liste");
  const [isPending, startTransition] = useTransition();

  const handleSec = useCallback((id) => setSeciliId(id), []);   // sabit → memo'lu satırlar korunur

  function sekmeSec(yeni) {
    startTransition(() => setSekme(yeni));                       // ağır sekme: acil değil
  }

  const istatistikYukle = () => import("./IstatistikPaneli");   // üzerine gelince önceden indir

  const sekmeSinifi =
    "rounded px-3 py-1 aria-pressed:bg-gray-900 aria-pressed:text-white";

  return (
    <div className="mx-auto max-w-5xl p-4">
      <header className="mb-4 flex items-center gap-2">
        <h1 className="mr-auto text-2xl font-bold">Ürün Gezgini</h1>
        <button
          onClick={() => sekmeSec("liste")}
          aria-pressed={sekme === "liste"}
          className={sekmeSinifi}
        >
          Liste
        </button>
        <button
          onClick={() => sekmeSec("istatistik")}
          onMouseEnter={istatistikYukle}
          onFocus={istatistikYukle}
          aria-pressed={sekme === "istatistik"}
          className={sekmeSinifi}
        >
          İstatistik
        </button>
        {isPending && <span aria-label="Yükleniyor">⏳</span>}
      </header>

      <div style={{ opacity: isPending ? 0.6 : 1 }}>
        {sekme === "liste" ? (
          <div className="grid gap-6 md:grid-cols-[1fr_260px]">
            <AramaBolumu seciliId={seciliId} onSec={handleSec} />
            <aside className="rounded border p-4">
              <UrunDetay id={seciliId} />
            </aside>
          </div>
        ) : (
          <Suspense fallback={<p>İstatistik paneli yükleniyor...</p>}>
            <IstatistikPaneli urunler={URUNLER} />
          </Suspense>
        )}
      </div>
    </div>
  );
}
```

### Tembel yüklenen panel

```jsx
// ============ src/IstatistikPaneli.jsx — ayrı chunk, ağır hesaplama ============
import { useMemo } from "react";

export default function IstatistikPaneli({ urunler }) {
  const istatistik = useMemo(() => {
    const kategoriler = Object.groupBy(urunler, (u) => u.kategori);
    return Object.entries(kategoriler)
      .map(([ad, liste]) => ({
        ad,
        adet: liste.length,
        ortalamaFiyat: Math.round(liste.reduce((t, u) => t + u.fiyat, 0) / liste.length),
        ortalamaPuan: (liste.reduce((t, u) => t + u.puan, 0) / liste.length).toFixed(2),
      }))
      .toSorted((a, b) => b.adet - a.adet);
  }, [urunler]);

  const enYuksek = Math.max(...istatistik.map((k) => k.adet));

  return (
    <section>
      <h2 className="mb-4 text-xl font-bold">Kategori İstatistikleri</h2>
      <ul className="space-y-3">
        {istatistik.map((k) => (
          <li key={k.ad}>
            <div className="flex justify-between text-sm">
              <span className="font-medium">{k.ad}</span>
              <span className="tabular-nums text-gray-500">
                {k.adet.toLocaleString("tr-TR")} ürün · ort. {k.ortalamaFiyat} ₺ · ⭐ {k.ortalamaPuan}
              </span>
            </div>
            <div className="mt-1 h-2 rounded bg-gray-100">
              <div
                className="h-2 rounded bg-indigo-500"
                style={{ width: `${(k.adet / enYuksek) * 100}%` }}
              />
            </div>
          </li>
        ))}
      </ul>
    </section>
  );
}
```

### Dene ve gözlemle (CPU throttling 4x açıkken)

| Deneme | Gözlenen | Teknik |
|---|---|---|
| Hızlı hızlı yaz | Harfler anında görünür; liste soluklaşıp biraz sonra güncellenir | `useDeferredValue` + `memo` |
| `useDeferredValue`'yu kaldır | Input takılır | — |
| Elements sekmesine bak | 20.000 sonuçta DOM'da ~25 satır; kaydırma akıcı | Sanallaştırma |
| Bir ürüne tıkla | Profiler'da sadece **iki satır** render olur | `memo` + `useCallback` + primitive props |
| `handleSec`'i `useCallback`'siz yaz | Görünür satırların hepsi render olur | — |
| Arama yaz | Başlık ve detay paneli render olmaz | Colocation |
| "İstatistik"in üzerine gel | Network'te ayrı JS dosyası iner | Önceden yükleme + `lazy` |
| "İstatistik"e tıkla | ⏳ belirir, liste kalır, panel hazır olunca geçilir | `useTransition` + `Suspense` |
| İstatistik paneli | Hesaplama bir kez yapılır | `useMemo` |

---

## 17. Özet

- **Performans = gereksiz iş yapmamak:** render, commit ve ağ/yükleme.
- **Önce ölç:** React DevTools Profiler ("why did this render?", highlight updates), Chrome Performance, Lighthouse; production build + CPU throttling.
- **Core Web Vitals:** LCP (ana içerik), INP (etkileşim tepkisi), CLS (kayma).
- Bileşen state değişince, ebeveyn render olunca ve context değişince render olur; render çoğu zaman ucuzdur.
- **En ucuz optimizasyon yapısaldır:** state'i aşağı taşı (colocation), pahalı içeriği `children` olarak yukarıdan ver.
- **`memo`**: props sığ karşılaştırılır; her render'da yeni nesne/fonksiyon onu bozar → sabitle, dışarı taşı, primitive props ver.
- **`useMemo`**: ölçülebilir şekilde pahalı hesaplamalar ve referans sabitleme; yan etki yok.
- **`useCallback`**: fonksiyon referansı önemliyse (memo'lu çocuk, bağımlılık, Hook dönüşü).
- Memoizasyon bedava değildir; gereksiz yere kullanılmaz.
- **React Compiler** (1.0, Ekim 2025): Derleme sırasında otomatik memoizasyon; React kurallarına uyan kod gerektirir; lint kuralları `eslint-plugin-react-hooks`'ta; Vite 8'de kurulum değişti (Babel eklentisi veya deneysel Oxc desteği).
- **`useDeferredValue`**: değerin gecikmeli kopyası; **`useTransition`**: güncellemeyi acil olmayan olarak işaretler (`isPending`); input'lar acil kalır.
- **Sanallaştırma** (TanStack Virtual): binlerce satırda sadece görüneni render eder; önce sayfalama/filtre düşünülür.
- **Code splitting**: `lazy` + `Suspense`, önceden yükleme, ağır kütüphaneleri `import()` ile ihtiyaç anında yükleme; bundle analizi.
- **Görseller**: `width`/`height`, `loading="lazy"` (ana görsel hariç), `srcSet`, modern formatlar; React 19 `preload`/`preconnect`.
- Sık sorunlar: context, bileşen içinde bileşen, rastgele key, effect zincirleri, sık olaylarda state, JS ile animasyon.

---

⬅️ Önceki: [18 · Veri Çekme ve TanStack Query](./18-veri-cekme.md) · ➡️ Sonraki: [20 · React 19 Yenilikleri](./20-react-19.md)
