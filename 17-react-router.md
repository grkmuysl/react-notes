# 17 · React Router

> **Konu özeti:** İstemci tarafı yönlendirme, React Router v8 kurulumu ve kullanım modları, router tanımı, `Link` / `NavLink`, iç içe rotalar ve düzenler, dinamik segmentler, 404 ve hata sınırları, programla yönlendirme, URL arama parametreleri, `useLocation`, loader ile veri yükleme, action / `<Form>` / `useFetcher` ile veri değiştirme, korumalı rotalar, tembel yükleme, kaydırma ve sayfa başlıkları, Declarative ve Framework modları.
> **Sürüm:** React Router v8 (Haziran 2026)

---

## İçindekiler

1. [Yönlendirme (Routing) Nedir?](#1-yönlendirme-routing-nedir)
2. [Kurulum, Sürüm ve Modlar](#2-kurulum-sürüm-ve-modlar)
3. [İlk Router](#3-ilk-router)
4. [`Link` ve `NavLink`](#4-link-ve-navlink)
5. [İç İçe Rotalar ve Düzenler](#5-iç-içe-rotalar-ve-düzenler-)
6. [Dinamik Segmentler ve `useParams`](#6-dinamik-segmentler-ve-useparams-)
7. [404 ve Hata Sayfaları](#7-404-ve-hata-sayfaları)
8. [Programla Yönlendirme](#8-programla-yönlendirme)
9. [Arama Parametreleri: URL'deki State](#9-arama-parametreleri-urldeki-state-)
10. [`useLocation` ve Konum State'i](#10-uselocation-ve-konum-statei)
11. [Veri Yükleme: `loader`](#11-veri-yükleme-loader-)
12. [Veri Değiştirme: `action`, `<Form>`, `useFetcher`](#12-veri-değiştirme-action-form-usefetcher-)
13. [Korumalı Rotalar](#13-korumalı-rotalar)
14. [Tembel Yükleme (Code Splitting)](#14-tembel-yükleme-code-splitting)
15. [Diğer Faydalı Parçalar](#15-diğer-faydalı-parçalar)
16. [Declarative Mod, Framework Modu ve Alternatifler](#16-declarative-mod-framework-modu-ve-alternatifler)
17. [Her Şey Bir Arada: Not Defteri Uygulaması](#17-her-şey-bir-arada-not-defteri-uygulaması)
18. [Özet](#18-özet)

---

## 1. Yönlendirme (Routing) Nedir?

Gerçek uygulamalarda birden çok sayfa ve her birinin kendi **URL**'si vardır:

```
/                    → Ana sayfa
/urunler             → Ürün listesi
/urunler/42          → 42 numaralı ürün
/urunler?kategori=kitap&sirala=fiyat
/profil/ayarlar
```

| Geleneksel site | SPA (React) |
|---|---|
| Her link sunucudan **yeni HTML** ister | **İstemci tarafı yönlendirme** |
| Ekran beyazlar, her şey baştan yüklenir | Sayfa yenilenmez, anında geçiş |
| JS state'i kaybolur | State korunur |

### İstemci tarafı yönlendirme

1. Kullanıcı linke tıklar.
2. Tarayıcının sayfa yenilemesi **engellenir.**
3. URL, History API ile güncellenir.
4. React yeni URL'ye karşılık gelen bileşenleri render eder.

- Geri/ileri butonları çalışır, URL'ler paylaşılabilir, yenileyince aynı ekran açılır.

> 💡 **URL de bir state'tir.** Sayfa, seçili öğe, filtreler URL'de tutulduğunda paylaşılabilir ve kalıcı olur.

React'te yerleşik yönlendirme yoktur; en yaygın çözüm **React Router**'dır.

---

## 2. Kurulum, Sürüm ve Modlar

### Sürüm

- **React Router v8**, 17 Haziran 2026'da yayınlandı.
- Minimum gereksinimler: **Node 22.22.0+, React 19.2.7+, Vite 7+**.
- Kırıcı değişiklikler az; hepsi v7'de de yapılabiliyordu → v7 içerikleri büyük ölçüde geçerli.
- React Router v6 ve Remix v2 artık **End of Life** (güvenlik güncellemesi almıyor).

```bash
npm install react-router
```

### ⚠️ `react-router-dom` kaldırıldı

```jsx
// ❌ Eski eğitimlerde
import { Link, useParams } from "react-router-dom";

// ✅ Güncel
import { Link, useParams } from "react-router";
import { RouterProvider } from "react-router/dom";   // DOM'a özgü olanlar
```

> İnternette `react-router-dom` görürsen `react-router` olarak oku.

### Üç kullanım modu

| Mod | Ne sunar | Ne zaman |
|---|---|---|
| **Declarative** | `<BrowserRouter>`, `<Routes>`, `<Route>`; sadece URL ↔ bileşen eşlemesi | En basit kullanım; mevcut/eski projeler |
| **Data** | `createBrowserRouter` + **loader**, **action**, bekleme durumları | ✅ Yeni SPA projeleri için önerilen |
| **Framework** | Dosya tabanlı rotalar, SSR, kod bölme, tip üretimi | Next.js benzeri tam yığın uygulamalar |

> Modlar arasında aşamalı geçiş yapılabilir (Declarative → Data → Framework). Bu notlarda **Data modu** esas alınmıştır.

---

## 3. İlk Router

```jsx
// src/main.jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { createBrowserRouter } from "react-router";
import { RouterProvider } from "react-router/dom";
import "./index.css";

import Kok from "./routes/Kok";
import AnaSayfa from "./routes/AnaSayfa";
import Hakkimizda from "./routes/Hakkimizda";

const router = createBrowserRouter([
  {
    path: "/",
    Component: Kok,                                  // tüm sayfaları saran düzen
    children: [
      { index: true, Component: AnaSayfa },          // "/"
      { path: "hakkimizda", Component: Hakkimizda }, // "/hakkimizda"
    ],
  },
]);

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <RouterProvider router={router} />
  </StrictMode>
);
```

```jsx
// src/routes/Kok.jsx
import { Link, Outlet } from "react-router";

export default function Kok() {
  return (
    <>
      <header>
        <nav>
          <Link to="/">Ana Sayfa</Link> | <Link to="/hakkimizda">Hakkımızda</Link>
        </nav>
      </header>
      <main>
        <Outlet />   {/* eşleşen alt rota burada render edilir */}
      </main>
      <footer>© 2026</footer>
    </>
  );
}
```

| Parça | Görevi |
|---|---|
| `createBrowserRouter([...])` | Rotaları **nesne dizisi** olarak tanımlar; modül seviyesinde **bir kez** oluşturulur |
| `RouterProvider` | Router'ı uygulamaya bağlar (`react-router/dom`'dan) |
| `path` | Eşleşilecek URL parçası |
| `Component` (veya `element`) | Eşleşince render edilecek bileşen |
| `children` | İç içe rotalar |
| `index: true` | Ebeveynin yolunda varsayılan çocuk |
| `<Outlet />` | Çocuk rotanın render edileceği yer tutucu |

### Klasör düzeni önerisi

```
src/
├── routes/          ← sayfa (rota) bileşenleri
│   ├── Kok.jsx
│   ├── AnaSayfa.jsx
│   └── urunler/
│       ├── UrunListesi.jsx
│       └── UrunDetay.jsx
├── components/      ← tekrar kullanılan bileşenler
└── main.jsx         ← router tanımı (büyüyünce router.jsx'e taşınabilir)
```

---

## 4. `Link` ve `NavLink`

### Neden `<a>` değil?

```jsx
<a href="/hakkimizda">Hakkımızda</a>      // ❌ Sayfa yenilenir, state kaybolur
<Link to="/hakkimizda">Hakkımızda</Link>  // ✅ İstemci tarafı geçiş
```

- `<Link>` DOM'da yine `<a href>` üretir: erişilebilirlik, sağ tık, Ctrl+tık çalışır.
- Dış siteler için normal `<a>` kullanılır.

### `NavLink`: Aktif linki vurgulamak

```jsx
import { NavLink } from "react-router";

<NavLink
  to="/"
  end
  className={({ isActive }) => (isActive ? "font-bold text-blue-600" : "text-gray-600")}
>
  Ana Sayfa
</NavLink>

<NavLink
  to="/urunler"
  className={({ isActive, isPending }) =>
    cn("text-gray-600", isActive && "font-bold text-blue-600", isPending && "opacity-50")
  }
>
  Ürünler
</NavLink>
```

- `className`/`style` fonksiyon alabilir: `isActive`, `isPending`.
- Aktif link otomatik **`aria-current="page"`** alır → Tailwind ile:

```jsx
<NavLink to="/urunler" className="text-gray-600 aria-[current=page]:font-bold aria-[current=page]:text-blue-600">
```

- **`end`**: `/urunler` linki `/urunler/42`'de de aktif sayılır (istenen davranış). `/` linkinin her sayfada aktif olmaması için `end` kullanılır.

### Göreli (relative) linkler

```jsx
// /urunler/42 sayfasındayken:
<Link to="yorumlar">     → /urunler/42/yorumlar
<Link to="..">           → /urunler
<Link to="/sepet">       → /sepet (mutlak)
```

### Diğer özellikler

```jsx
<Link to="/giris" replace>                      {/* geçmişe yeni kayıt eklemez */}
<Link to="/urunler?kategori=kitap">             {/* arama parametreleri */}
<Link to="/odeme" state={{ kaynak: "sepet" }}>  {/* URL'de görünmeyen geçici veri */}
```

---

## 5. İç İçe Rotalar ve Düzenler ⭐

> **URL'nin parçaları, iç içe bileşenlerle eşleşir.**

```
URL:   /panel/siparisler/1042

       ┌───────────────── Kok (header, footer) ────────────────────┐
       │  ┌────────────── PanelDuzeni (kenar menü) ──────────────┐ │
       │  │  ┌────────── SiparisDetay ─────────────┐             │ │
       │  │  │  Sipariş #1042                     │             │ │
       │  │  └────────────────────────────────────┘             │ │
       │  └─────────────────────────────────────────────────────┘ │
       └───────────────────────────────────────────────────────────┘
```

```jsx
const router = createBrowserRouter([
  {
    path: "/",
    Component: Kok,
    children: [
      { index: true, Component: AnaSayfa },
      {
        path: "panel",
        Component: PanelDuzeni,
        children: [
          { index: true, Component: PanelOzet },                 // /panel
          { path: "siparisler", Component: Siparisler },         // /panel/siparisler
          { path: "siparisler/:no", Component: SiparisDetay },   // /panel/siparisler/1042
          { path: "ayarlar", Component: Ayarlar },               // /panel/ayarlar
        ],
      },
    ],
  },
]);
```

```jsx
function PanelDuzeni() {
  return (
    <div className="grid grid-cols-[220px_1fr] gap-6">
      <aside>
        <NavLink to="/panel" end>Özet</NavLink>
        <NavLink to="/panel/siparisler">Siparişler</NavLink>
        <NavLink to="/panel/ayarlar">Ayarlar</NavLink>
      </aside>
      <section>
        <Outlet />
      </section>
    </div>
  );
}
```

- Çocuk `path`'leri ebeveyninkine **eklenir.**
- Alt sayfalar arasında geçerken düzen bileşenleri **yeniden oluşturulmaz**; state'leri korunur, sadece `<Outlet />` içeriği değişir.

### Yolsuz düzen rotası (pathless layout)

URL'ye parça eklemeden düzen paylaşmak:

```jsx
{
  Component: OrtalanmisDuzen,   // path YOK
  children: [
    { path: "giris", Component: Giris },   // /giris
    { path: "kayit", Component: Kayit },   // /kayit
  ],
}
```

### Önek rotası (bileşensiz)

Düzen olmadan URL öneki paylaşmak:

```jsx
{
  path: "yardim",               // Component YOK
  children: [
    { path: "sss", Component: Sss },            // /yardim/sss
    { path: "iletisim", Component: Iletisim },  // /yardim/iletisim
  ],
}
```

---

## 6. Dinamik Segmentler ve `useParams` ⭐

```jsx
{ path: "urunler/:urunId", Component: UrunDetay }
```

```jsx
import { useParams } from "react-router";

function UrunDetay() {
  const { urunId } = useParams();   // /urunler/42 → { urunId: "42" }
  return <h1>Ürün #{urunId}</h1>;
}
```

> ⚠️ Parametreler **her zaman string**'dir; gerekirse `Number(urunId)`.

| Tür | Tanım | Örnek |
|---|---|---|
| Birden çok parametre | `kategoriler/:kategori/urunler/:urunId` | `{ kategori: "kitap", urunId: "42" }` |
| Opsiyonel (`?`) | `:dil?/hakkimizda` | `/hakkimizda` → `dil: undefined`, `/en/hakkimizda` → `dil: "en"` |
| Splat (`*`) | `dosyalar/*` | `params["*"] = "belgeler/2026/rapor.pdf"` |

### ⚠️ Aynı bileşen, farklı parametre

`/urunler/1` → `/urunler/2` geçişinde bileşen **yeniden oluşturulmaz**; içindeki state önceki üründen kalır. Çözüm: `key`.

```jsx
function UrunDetay() {
  const { urunId } = useParams();
  return <UrunIcerigi key={urunId} urunId={urunId} />;   // ürün değişince state sıfırlanır
}
```

---

## 7. 404 ve Hata Sayfaları

### Tanımsız URL'ler: `*` rotası

```jsx
children: [
  { index: true, Component: AnaSayfa },
  { path: "urunler", Component: UrunListesi },
  { path: "*", Component: Bulunamadi },   // hiçbiri eşleşmezse
]
```

- Rotalar **en spesifik olandan** başlayarak eşlenir; sıra önemli değildir.

### Hata sınırı: `ErrorBoundary`

```jsx
import { isRouteErrorResponse, useRouteError, Link } from "react-router";

function HataSayfasi() {
  const hata = useRouteError();

  if (isRouteErrorResponse(hata)) {
    // Bilinçli fırlatılan HTTP benzeri hatalar (404, 401...)
    return (
      <div>
        <h1>{hata.status} {hata.status === 404 ? "Sayfa bulunamadı" : "Hata"}</h1>
        <p>{hata.data}</p>
        <Link to="/">Ana sayfaya dön</Link>
      </div>
    );
  }

  // Beklenmeyen hatalar
  return (
    <div>
      <h1>Bir şeyler ters gitti 😕</h1>
      <p>{hata instanceof Error ? hata.message : "Bilinmeyen hata"}</p>
    </div>
  );
}

const router = createBrowserRouter([
  {
    path: "/",
    Component: Kok,
    ErrorBoundary: HataSayfasi,   // veya errorElement: <HataSayfasi />
    children: [/* ... */],
  },
]);
```

- Render, loader veya action hataları **en yakın** `ErrorBoundary`'ye çıkar.
- Kök `ErrorBoundary` tüm sayfayı değiştirir; iç rotalara ayrı `ErrorBoundary` verilirse menü/düzen yerinde kalır:

```jsx
{ path: "urunler/:urunId", Component: UrunDetay, ErrorBoundary: UrunHatasi }
```

---

## 8. Programla Yönlendirme

### `useNavigate`

```jsx
import { useNavigate } from "react-router";

function CikisButonu() {
  const navigate = useNavigate();

  async function handleCikis() {
    await cikisYap();
    navigate("/giris", { replace: true });
  }

  return <button onClick={handleCikis}>Çıkış yap</button>;
}
```

```jsx
navigate("/urunler");                               // git
navigate(-1);                                       // geri
navigate(1);                                        // ileri
navigate("..");                                     // bir üst rota
navigate("/odeme", { state: { kaynak: "sepet" } });
navigate("/giris", { replace: true });
```

> 💡 Kullanıcı eylemine bağlı yönlendirme **olay yöneticisinde** (veya action'da) yapılır, effect'te değil.

### `<Navigate>` bileşeni

```jsx
function EskiSayfa() {
  return <Navigate to="/yeni-sayfa" replace />;
}
```

> Data modunda yönlendirmeler tercihen **loader/action içinde `redirect()`** ile yapılır; yanlış sayfa hiç render edilmez.

---

## 9. Arama Parametreleri: URL'deki State ⭐

```
/urunler?kategori=kitap&sirala=fiyat&sayfa=2
```

### Neden URL'de?

- Link paylaşılınca karşı taraf **aynı filtreleri** görür.
- Sayfa yenilense de filtreler kaybolmaz.
- Geri tuşu önceki filtreye döner.

```jsx
import { useSearchParams } from "react-router";

function UrunListesi() {
  const [aramaParametreleri, setAramaParametreleri] = useSearchParams();

  // Okuma: değerler string veya null
  const kategori = aramaParametreleri.get("kategori") ?? "hepsi";
  const sirala = aramaParametreleri.get("sirala") ?? "ad";
  const arama = aramaParametreleri.get("q") ?? "";

  // Yazma: tek parametreyi güncelle, diğerlerini koru
  function parametreDegistir(anahtar, deger, secenekler) {
    setAramaParametreleri((onceki) => {
      const yeni = new URLSearchParams(onceki);
      if (deger) yeni.set(anahtar, deger);
      else yeni.delete(anahtar);              // boşsa URL'den kaldır
      return yeni;
    }, secenekler);
  }

  const gorunenler = URUNLER
    .filter((u) => kategori === "hepsi" || u.kategori === kategori)
    .filter((u) => u.ad.toLocaleLowerCase("tr").includes(arama.toLocaleLowerCase("tr")))
    .toSorted(siralamaFonksiyonu(sirala));   // türetilmiş değer

  return (
    <>
      <input
        value={arama}
        onChange={(e) => parametreDegistir("q", e.target.value, { replace: true })}
        placeholder="Ara..."
      />
      <select
        value={kategori}
        onChange={(e) => parametreDegistir("kategori", e.target.value === "hepsi" ? "" : e.target.value)}
      >
        <option value="hepsi">Tümü</option>
        <option value="kitap">Kitap</option>
        <option value="elektronik">Elektronik</option>
      </select>
      {/* ... liste */}
    </>
  );
}
```

### Dikkat edilecekler

- Okuma tarafı standart **`URLSearchParams`**: `get`, `getAll`, `has`.
- Tüm değerler **string**: `Number(aramaParametreleri.get("sayfa") ?? 1)`.
- Sık değişen girdilerde **`{ replace: true }`**; aksi halde her harf geçmişe ayrı kayıt olur.
- Varsayılan değerler URL'den **kaldırılır** (`delete`) → temiz URL'ler.

### Hangi state URL'de olmalı?

| URL'de (search params) | `useState`'te |
|---|---|
| Filtreler, sıralama, sayfa numarası | Açık/kapalı menü, hover durumu |
| Arama sorgusu | Yarım yazılmış form alanları |
| Seçili sekme (paylaşılabilir olacaksa) | Animasyon durumu |
| Harita konumu/yakınlık | Geçici bildirimler |

> **Ölçüt:** "Bu linki birine gönderdiğimde aynı ekranı görmeli mi?" Evet → URL.

---

## 10. `useLocation` ve Konum State'i

```jsx
import { useLocation } from "react-router";

const location = useLocation();
// { pathname: "/urunler/42", search: "?renk=mavi", hash: "#yorumlar", state: {...}, key: "abc123" }
```

### Analitik (dış sisteme bildirim → effect)

```jsx
useEffect(() => {
  analitikGonder(location.pathname);
}, [location.pathname]);
```

### Geçiş state'i

```jsx
// Gönderen
navigate("/giris", { state: { geriDon: location.pathname } });

// Alan
const { state } = useLocation();
const geriDon = state?.geriDon ?? "/";
```

- Tarayıcı geçmişinde korunur ama linki **paylaşan kişiye gitmez.**
- Sayfanın çalışması ona bağımlı olmamalı; `?.` ve varsayılan değerle okunur.

---

## 11. Veri Yükleme: `loader` ⭐

Effect ile veri çekmenin sorunları: tekrar eden yükleniyor/hata kodu, yarış durumları, **ağ şelaleleri.**

**Loader:** Rota render edilmeden **önce** çalışan veri yükleme fonksiyonu.

```jsx
// src/routes/urunler/UrunDetay.jsx
import { data, useLoaderData } from "react-router";

export async function loader({ params, request }) {
  const res = await fetch(`https://dummyjson.com/products/${params.urunId}`, {
    signal: request.signal,          // sayfadan ayrılınca istek iptal
  });
  if (res.status === 404) {
    throw data("Bu ürün bulunamadı.", { status: 404 });   // → ErrorBoundary
  }
  if (!res.ok) throw new Error("Ürün yüklenemedi");
  return res.json();
}

export default function UrunDetay() {
  const urun = useLoaderData();      // her zaman hazır veri
  return (
    <article>
      <h1>{urun.title}</h1>
      <p>{urun.description}</p>
      <strong>{urun.price} $</strong>
    </article>
  );
}
```

```jsx
// router tanımı
import UrunDetay, { loader as urunDetayLoader } from "./routes/urunler/UrunDetay";

{ path: "urunler/:urunId", Component: UrunDetay, loader: urunDetayLoader, ErrorBoundary: UrunHatasi }
```

### Kazanımlar

- Bileşende `useEffect`, veri için `useState`, yükleniyor/hata kontrolü **yok.**
- Hatalar `ErrorBoundary`'ye; 404 için `data(..., { status: 404 })`.
- `request.signal` ile yarış durumu ve iptal otomatik.
- **Şelale yok:** Eşleşen tüm rotaların loader'ları **paralel** çalışır.

### Loader argümanları

- `params`: URL parametreleri
- `request`: Standart `Request`; arama parametreleri buradan okunur:

```jsx
export async function loader({ request }) {
  const url = new URL(request.url);
  const q = url.searchParams.get("q") ?? "";
  const res = await fetch(`https://dummyjson.com/products/search?q=${encodeURIComponent(q)}`);
  return res.json();
}
```

> Arama parametreleri değişince loader **otomatik yeniden çalışır.**

### Bir loader'da paralel istekler

```jsx
export async function loader({ params }) {
  const [kullanici, siparisler] = await Promise.all([
    fetch(`/api/kullanicilar/${params.id}`).then((r) => r.json()),
    fetch(`/api/kullanicilar/${params.id}/siparisler`).then((r) => r.json()),
  ]);
  return { kullanici, siparisler };
}
```

### Bekleme durumu: `useNavigation`

Loader çalışırken kullanıcı **eski sayfayı** görmeye devam eder.

```jsx
import { useNavigation } from "react-router";

function Kok() {
  const navigation = useNavigation();
  const yukleniyor = navigation.state === "loading";   // "idle" | "loading" | "submitting"

  return (
    <>
      {yukleniyor && <div className="fixed inset-x-0 top-0 h-1 animate-pulse bg-blue-600" />}
      <main className={yukleniyor ? "opacity-60 transition-opacity" : ""}>
        <Outlet />
      </main>
    </>
  );
}
```

### İlk yükleme: `HydrateFallback`

```jsx
{
  path: "/",
  Component: Kok,
  HydrateFallback: () => <p className="p-8">Uygulama yükleniyor...</p>,
  children: [/* ... */],
}
```

> 💡 Loader'lar render döngüsü dışında çalışır; içinde **Hook kullanılamaz.** Sıradan async fonksiyonlar oldukları için kolay test edilir.

> 📌 Loader'lar **önbellek tutmaz**; gelişmiş önbellekleme için TanStack Query ile birleştirilebilir *(Konu 18)*.

---

## 12. Veri Değiştirme: `action`, `<Form>`, `useFetcher` ⭐

Loader **okur**, action **değiştirir** (oluştur, güncelle, sil).

### `action` + `<Form>`

```jsx
// src/routes/notlar/YeniNot.jsx
import { Form, redirect, useActionData, useNavigation } from "react-router";

export async function action({ request }) {
  const formData = await request.formData();
  const baslik = String(formData.get("baslik") ?? "").trim();
  const icerik = String(formData.get("icerik") ?? "").trim();

  // Doğrulama: hata varsa döndür
  const hatalar = {};
  if (baslik.length < 3) hatalar.baslik = "Başlık en az 3 karakter olmalı.";
  if (!icerik) hatalar.icerik = "İçerik boş olamaz.";
  if (Object.keys(hatalar).length > 0) return { hatalar };

  const not = await notOlustur({ baslik, icerik });
  return redirect(`/notlar/${not.id}`);   // başarılı → yeni nota git
}

export default function YeniNot() {
  const actionVerisi = useActionData();
  const navigation = useNavigation();
  const gonderiliyor = navigation.state === "submitting";

  return (
    <Form method="post" className="flex flex-col gap-3">
      <label>
        Başlık
        <input name="baslik" aria-invalid={actionVerisi?.hatalar?.baslik ? true : undefined} />
      </label>
      {actionVerisi?.hatalar?.baslik && <p className="text-red-600">{actionVerisi.hatalar.baslik}</p>}

      <label>
        İçerik
        <textarea name="icerik" rows={5} />
      </label>
      {actionVerisi?.hatalar?.icerik && <p className="text-red-600">{actionVerisi.hatalar.icerik}</p>}

      <button type="submit" disabled={gonderiliyor}>
        {gonderiliyor ? "Kaydediliyor..." : "Kaydet"}
      </button>
    </Form>
  );
}
```

```jsx
{ path: "notlar/yeni", Component: YeniNot, action: yeniNotAction }
```

### Akış

1. `<Form method="post">` gönderilir; sayfa yenilenmez, veri rotanın **action**'ına gider.
2. Action `FormData` ile okur; input'lar **uncontrolled**, `name` yeterli.
3. Hata → nesne döndürülür → **`useActionData()`** ile okunur.
4. Başarı → **`redirect()`**.
5. `useNavigation().state === "submitting"` → gönderim durumu.
6. **Otomatik revalidation:** Action sonrası sayfadaki **tüm loader'lar yeniden çalışır**; ekrandaki tüm veriler kendiliğinden güncellenir.

> 💡 React 19'un `<form action={fn}>` fikrinin rota düzeyindeki karşılığı; üstüne yönlendirme ve otomatik veri yenileme ekler.

### Aynı formda birden çok işlem

```jsx
<Form method="post">
  <button name="islem" value="arsivle">Arşivle</button>
  <button name="islem" value="sil">Sil</button>
</Form>

export async function action({ request, params }) {
  const formData = await request.formData();
  switch (formData.get("islem")) {
    case "arsivle": await notArsivle(params.notId); return null;
    case "sil": await notSil(params.notId); return redirect("/notlar");
    default: throw data("Bilinmeyen işlem", { status: 400 });
  }
}
```

### Gezinmeden veri değiştirmek: `useFetcher`

Favori, beğeni, "tamamlandı" işaretleme gibi sayfada kalınan işlemler için:

```jsx
import { useFetcher } from "react-router";

function FavoriButonu({ not }) {
  const fetcher = useFetcher();

  // İyimser arayüz: sunucu cevabını beklemeden yeni değeri göster
  const favori = fetcher.formData
    ? fetcher.formData.get("favori") === "true"
    : not.favori;

  return (
    <fetcher.Form method="post" action={`/notlar/${not.id}/favori`}>
      <button
        name="favori"
        value={favori ? "false" : "true"}
        aria-label={favori ? "Favoriden çıkar" : "Favorilere ekle"}
      >
        {favori ? "★" : "☆"}
      </button>
    </fetcher.Form>
  );
}
```

```jsx
// Sadece action'ı olan, bileşensiz rota
{
  path: "notlar/:notId/favori",
  action: async ({ params, request }) => {
    const formData = await request.formData();
    await notGuncelle(params.notId, { favori: formData.get("favori") === "true" });
    return null;
  },
}
```

| Özellik | Açıklama |
|---|---|
| `fetcher.Form` | Sayfa değiştirmez, URL'ye dokunmaz |
| Bağımsız durum | Her fetcher'ın kendi `state`, `data`, `formData`'sı var |
| Revalidation | İşlem sonrası loader'lar yine otomatik yenilenir |
| İyimser UI | `fetcher.formData` ile gönderilen değer hemen gösterilir; hata olursa gerçek değer geri gelir |
| `fetcher.load(url)` | Bir rotanın loader'ını gezinmeden çağırır (otomatik tamamlama vb.) |

> React 19'un `useOptimistic` Hook'u aynı fikri genel olarak sunar *(Konu 20)*.

---

## 13. Korumalı Rotalar

En temiz yöntem: **loader içinde kontrol edip yönlendirmek** → korumalı sayfa hiç render edilmez, verisi hiç yüklenmez.

```jsx
// src/auth.js
import { redirect } from "react-router";

export function mevcutKullanici() {
  const kayit = localStorage.getItem("kullanici");
  return kayit ? JSON.parse(kayit) : null;
}

export function girisZorunlu(request) {
  const kullanici = mevcutKullanici();
  if (!kullanici) {
    const nereye = new URL(request.url).pathname;
    throw redirect(`/giris?geriDon=${encodeURIComponent(nereye)}`);
  }
  return kullanici;
}
```

```jsx
// Düzen rotasına konur → altındaki tüm sayfalar korunur
{
  path: "panel",
  Component: PanelDuzeni,
  loader: ({ request }) => {
    const kullanici = girisZorunlu(request);
    return { kullanici };
  },
  children: [
    { index: true, Component: PanelOzet },
    { path: "ayarlar", Component: Ayarlar },
  ],
}
```

```jsx
// Giriş action'ı: geldiği yere geri gönder
export async function action({ request }) {
  const formData = await request.formData();
  const kullanici = await girisYap(formData.get("email"), formData.get("sifre"));
  if (!kullanici) return { hata: "E-posta veya şifre hatalı." };

  localStorage.setItem("kullanici", JSON.stringify(kullanici));
  const geriDon = new URL(request.url).searchParams.get("geriDon") ?? "/panel";
  return redirect(geriDon);
}
```

**Akış:** `/panel/ayarlar` → giriş yok → `/giris?geriDon=/panel/ayarlar` → giriş → `/panel/ayarlar`

> ⚠️ İstemci tarafı korumalar yalnızca **arayüz** içindir; gerçek güvenlik **sunucuda** sağlanır.

> 📌 React Router v8'de **middleware** varsayılan olarak etkin: tüm loader/action'lardan önce çalışan ortak kod (kimlik kontrolü, loglama). Özellikle Framework modunda kullanışlıdır; SPA'larda düzen-loader yaklaşımı yeterlidir.

---

## 14. Tembel Yükleme (Code Splitting)

Rotanın kodu yalnızca o rotaya **gidildiğinde** indirilir:

```jsx
{
  path: "yonetim",
  lazy: () => import("./routes/yonetim/Yonetim"),
}
```

```jsx
// src/routes/yonetim/Yonetim.jsx — rota özellikleri isimli export olarak
export async function loader() { /* ... */ }
export function Component() { /* ... */ }
export function ErrorBoundary() { /* ... */ }
```

- Vite, `import()` gördüğü yerde kodu ayrı bir dosyaya (chunk) böler.
- Ağır ve nadir kullanılan sayfalar (raporlar, ayarlar, grafikli sayfalar) için idealdir *(Konu 19)*.

---

## 15. Diğer Faydalı Parçalar

### Kaydırma konumu: `<ScrollRestoration />`

```jsx
import { Outlet, ScrollRestoration } from "react-router";

function Kok() {
  return (
    <>
      <Header />
      <Outlet />
      <ScrollRestoration />   {/* yeni sayfada en üste, geri gelince eski konuma */}
    </>
  );
}
```

### Sayfa başlığı: React 19 `<title>`

React 19'da `<title>` ve `<meta>` bileşen içinde render edilir; React onları `<head>`'e taşır:

```jsx
function UrunDetay() {
  const urun = useLoaderData();
  return (
    <>
      <title>{`${urun.title} · Mağaza`}</title>
      <meta name="description" content={urun.description} />
      <h1>{urun.title}</h1>
    </>
  );
}
```

> Konu 15'teki `useBelgeBasligi` Hook'una artık gerek yok.

### Yayınlama (deploy) notu

- `/urunler/42`'yi doğrudan açınca veya yenileyince istek **sunucuya** gider; dosya olmadığı için 404 döner.
- Sunucu, bilinmeyen tüm yollar için **`index.html`** döndürmelidir (rewrite kuralı).
- Netlify, Vercel, Cloudflare Pages'te genelde bir ayar veya tek satırlık yapılandırmadır; Vite geliştirme sunucusu bunu otomatik yapar.

---

## 16. Declarative Mod, Framework Modu ve Alternatifler

### Declarative mod

```jsx
import { BrowserRouter, Routes, Route } from "react-router";

createRoot(document.getElementById("root")).render(
  <BrowserRouter>
    <Routes>
      <Route path="/" element={<Kok />}>
        <Route index element={<AnaSayfa />} />
        <Route path="urunler" element={<UrunListesi />} />
        <Route path="urunler/:urunId" element={<UrunDetay />} />
        <Route path="*" element={<Bulunamadi />} />
      </Route>
    </Routes>
  </BrowserRouter>
);
```

- **Çalışanlar:** `Link`, `NavLink`, `Outlet`, `useParams`, `useNavigate`, `useSearchParams`, `useLocation`
- **Çalışmayanlar:** `loader`, `action`, action'a gönderen `<Form>`, `useLoaderData`, `useActionData`, `useNavigation`, `useFetcher`, `ErrorBoundary`

| Data modu | Declarative mod |
|---|---|
| `createBrowserRouter([...])` + `<RouterProvider>` | `<BrowserRouter>` + `<Routes>` |
| `{ path, Component, children }` | İç içe `<Route path element>` |
| `loader` + `useLoaderData` | `useEffect` veya TanStack Query |
| `action` + `<Form>` | `onSubmit` + `fetch` + `navigate` |
| `ErrorBoundary` | Kendi hata sınırın |

### Framework modu

Data modunun her şeyi, üstüne:

- **Dosya tabanlı rotalar** (isteğe bağlı)
- **Sunucu tarafı render (SSR)** ve statik ön-render (SEO, ilk yükleme hızı)
- Loader/action'lar **sunucuda** çalışabilir (veritabanı, gizli anahtarlar)
- Otomatik kod bölme, tip üretimi

> Eskiden **Remix** olarak bilinen framework'ün bugünkü hali. SSR ve Server Components → *Konu 23*.

### Alternatif: TanStack Router

- Güçlü **TypeScript tip güvenliği** (rota ve arama parametreleri dahil)
- Aynı temel fikirler: iç içe rotalar, loader'lar, arama parametreleri

---

## 17. Her Şey Bir Arada: Not Defteri Uygulaması

Düzenler, `NavLink`, dinamik segmentler, URL'de arama/sıralama, loader'lar, form action'ları ve doğrulama, `useFetcher` ile iyimser favori, silme, korumalı rotalar, 404 ve hata sınırları, bekleme göstergesi, tembel sayfa ve sayfa başlıkları.

### Sahte sunucu

```js
// ============ src/api.js ============
const GECIKME = 300;
const bekle = (ms = GECIKME) => new Promise((r) => setTimeout(r, ms));

function oku() {
  try { return JSON.parse(localStorage.getItem("notlar")) ?? baslangic(); }
  catch { return baslangic(); }
}
function yaz(notlar) {
  localStorage.setItem("notlar", JSON.stringify(notlar));
}
function baslangic() {
  const notlar = [
    { id: "1", baslik: "React Router", icerik: "Loader ve action'lar harika.", favori: true, tarih: "2026-10-20" },
    { id: "2", baslik: "Tailwind", icerik: "@theme ile tasarım sistemi.", favori: false, tarih: "2026-10-21" },
    { id: "3", baslik: "Alışveriş", icerik: "Süt, ekmek, peynir.", favori: false, tarih: "2026-10-22" },
  ];
  yaz(notlar);
  return notlar;
}

export async function notlariGetir({ q = "", sirala = "yeni" } = {}) {
  await bekle();
  const ara = q.toLocaleLowerCase("tr");
  return oku()
    .filter(
      (n) =>
        n.baslik.toLocaleLowerCase("tr").includes(ara) ||
        n.icerik.toLocaleLowerCase("tr").includes(ara)
    )
    .toSorted((a, b) =>
      sirala === "baslik" ? a.baslik.localeCompare(b.baslik, "tr")
      : sirala === "eski" ? a.tarih.localeCompare(b.tarih)
      : b.tarih.localeCompare(a.tarih)
    );
}

export async function notGetir(id) {
  await bekle();
  return oku().find((n) => n.id === id) ?? null;
}

export async function notOlustur({ baslik, icerik }) {
  await bekle(500);
  const not = {
    id: crypto.randomUUID(),
    baslik,
    icerik,
    favori: false,
    tarih: new Date().toISOString().slice(0, 10),
  };
  yaz([not, ...oku()]);
  return not;
}

export async function notGuncelle(id, degisiklikler) {
  await bekle();
  yaz(oku().map((n) => (n.id === id ? { ...n, ...degisiklikler } : n)));
}

export async function notSil(id) {
  await bekle();
  yaz(oku().filter((n) => n.id !== id));
}

// --- Basit oturum ---
export function mevcutKullanici() {
  try { return JSON.parse(localStorage.getItem("kullanici")); } catch { return null; }
}
export async function girisYap(ad) {
  await bekle();
  const kullanici = { ad };
  localStorage.setItem("kullanici", JSON.stringify(kullanici));
  return kullanici;
}
export function cikisYap() {
  localStorage.removeItem("kullanici");
}
```

### Kök düzen

```jsx
// ============ src/routes/Kok.jsx ============
import { Form, Link, NavLink, Outlet, ScrollRestoration, useLoaderData, useNavigation } from "react-router";
import { mevcutKullanici } from "../api";

export function loader() {
  return { kullanici: mevcutKullanici() };
}

const menuSinifi =
  "rounded px-3 py-1.5 text-slate-600 hover:bg-slate-100 " +
  "aria-[current=page]:bg-slate-900 aria-[current=page]:text-white";

export default function Kok() {
  const { kullanici } = useLoaderData();
  const navigation = useNavigation();
  const mesgul = navigation.state !== "idle";

  return (
    <div className="min-h-dvh bg-slate-50 text-slate-900">
      {mesgul && (
        <div className="fixed inset-x-0 top-0 z-50 h-1 animate-pulse bg-indigo-600" aria-hidden="true" />
      )}

      <header className="border-b border-slate-200 bg-white">
        <div className="mx-auto flex max-w-5xl items-center gap-2 px-4 py-3">
          <Link to="/" className="mr-4 font-bold">📝 Notlarım</Link>
          <nav className="flex gap-1">
            <NavLink to="/" end className={menuSinifi}>Ana Sayfa</NavLink>
            <NavLink to="/notlar" className={menuSinifi}>Notlar</NavLink>
            <NavLink to="/istatistik" className={menuSinifi}>İstatistik</NavLink>
          </nav>
          <div className="ml-auto text-sm">
            {kullanici ? (
              <Form method="post" action="/cikis" className="flex items-center gap-2">
                <span>👤 {kullanici.ad}</span>
                <button className="rounded border px-2 py-1 hover:bg-slate-100">Çıkış</button>
              </Form>
            ) : (
              <Link to="/giris" className="rounded bg-indigo-600 px-3 py-1.5 text-white">Giriş</Link>
            )}
          </div>
        </div>
      </header>

      <main className={`mx-auto max-w-5xl px-4 py-6 transition-opacity ${mesgul ? "opacity-60" : ""}`}>
        <Outlet />
      </main>

      <ScrollRestoration />
    </div>
  );
}
```

### Ana sayfa

```jsx
// ============ src/routes/AnaSayfa.jsx ============
import { Link } from "react-router";

export default function AnaSayfa() {
  return (
    <section className="py-12 text-center">
      <title>Notlarım</title>
      <h1 className="text-4xl font-bold">Notlarını düzenle</h1>
      <p className="mt-2 text-slate-500">React Router ile yapılmış küçük bir not defteri.</p>
      <Link to="/notlar" className="mt-6 inline-block rounded-lg bg-indigo-600 px-5 py-2.5 text-white">
        Notlara git →
      </Link>
    </section>
  );
}
```

### Notlar düzeni (korumalı, liste + arama)

```jsx
// ============ src/routes/notlar/NotlarDuzeni.jsx ============
import { Form, Link, NavLink, Outlet, redirect, useLoaderData, useSearchParams, useSubmit } from "react-router";
import { mevcutKullanici, notlariGetir } from "../../api";

export async function loader({ request }) {
  if (!mevcutKullanici()) {
    const nereye = new URL(request.url).pathname;
    throw redirect(`/giris?geriDon=${encodeURIComponent(nereye)}`);
  }
  const url = new URL(request.url);
  const q = url.searchParams.get("q") ?? "";
  const sirala = url.searchParams.get("sirala") ?? "yeni";
  const notlar = await notlariGetir({ q, sirala });
  return { notlar, q, sirala };
}

export default function NotlarDuzeni() {
  const { notlar, q, sirala } = useLoaderData();
  const [aramaParametreleri] = useSearchParams();
  const submit = useSubmit();
  const sorgu = aramaParametreleri.toString() ? `?${aramaParametreleri}` : "";

  return (
    <div className="grid gap-6 md:grid-cols-[280px_1fr]">
      <aside className="space-y-3">
        {/* GET formu: alanlar URL arama parametrelerine dönüşür, loader yeniden çalışır */}
        <Form method="get" role="search" className="space-y-2">
          <input
            type="search"
            name="q"
            defaultValue={q}
            placeholder="Notlarda ara..."
            className="w-full rounded border border-slate-300 px-3 py-2"
            onChange={(e) => submit(e.currentTarget.form, { replace: q !== "" })}
          />
          <select
            name="sirala"
            defaultValue={sirala}
            onChange={(e) => submit(e.currentTarget.form)}
            className="w-full rounded border border-slate-300 px-3 py-2"
          >
            <option value="yeni">En yeni</option>
            <option value="eski">En eski</option>
            <option value="baslik">Başlığa göre</option>
          </select>
        </Form>

        <Link to={`yeni${sorgu}`} className="block rounded bg-indigo-600 px-3 py-2 text-center text-white">
          + Yeni not
        </Link>

        {notlar.length === 0 ? (
          <p className="text-sm text-slate-500">{q ? `"${q}" için sonuç yok.` : "Henüz not yok."}</p>
        ) : (
          <ul className="divide-y divide-slate-200 rounded border border-slate-200 bg-white">
            {notlar.map((n) => (
              <li key={n.id}>
                <NavLink
                  to={`${n.id}${sorgu}`}
                  className={({ isActive, isPending }) =>
                    `flex items-center justify-between px-3 py-2 hover:bg-slate-50 ${
                      isActive ? "bg-indigo-50 font-medium text-indigo-700" : ""
                    } ${isPending ? "opacity-50" : ""}`
                  }
                >
                  <span className="truncate">{n.baslik}</span>
                  {n.favori && <span aria-label="favori">★</span>}
                </NavLink>
              </li>
            ))}
          </ul>
        )}
      </aside>

      <section className="min-h-64 rounded border border-slate-200 bg-white p-6">
        <Outlet />
      </section>
    </div>
  );
}
```

> **Not:** Arama input'u uncontrolled (`defaultValue`) olduğundan, tarayıcının geri/ileri tuşlarıyla `q` değiştiğinde input içeriği eski kalabilir. Gerekirse bir effect ile `input.value = q` senkronize edilebilir. Input'a `key={q}` vermek **önerilmez**: her harfte input yeniden oluşur ve odak kaybolur.

### Not detayı (favori + silme + hata sınırı)

```jsx
// ============ src/routes/notlar/NotDetay.jsx ============
import {
  data, Form, isRouteErrorResponse, Link, redirect,
  useFetcher, useLoaderData, useRouteError,
} from "react-router";
import { notGetir, notGuncelle, notSil } from "../../api";

export async function loader({ params }) {
  const not = await notGetir(params.notId);
  if (!not) throw data("Bu not bulunamadı ya da silinmiş.", { status: 404 });
  return { not };
}

export async function action({ request, params }) {
  const formData = await request.formData();
  switch (formData.get("islem")) {
    case "favori":
      await notGuncelle(params.notId, { favori: formData.get("favori") === "true" });
      return null;
    case "sil": {
      await notSil(params.notId);
      const url = new URL(request.url);
      return redirect(`/notlar${url.search}`);
    }
    default:
      throw data("Bilinmeyen işlem", { status: 400 });
  }
}

function FavoriButonu({ not }) {
  const fetcher = useFetcher();
  const favori = fetcher.formData
    ? fetcher.formData.get("favori") === "true"   // iyimser
    : not.favori;

  return (
    <fetcher.Form method="post">
      <input type="hidden" name="islem" value="favori" />
      <button
        name="favori"
        value={favori ? "false" : "true"}
        className="text-2xl text-amber-500"
        aria-label={favori ? "Favorilerden çıkar" : "Favorilere ekle"}
      >
        {favori ? "★" : "☆"}
      </button>
    </fetcher.Form>
  );
}

export default function NotDetay() {
  const { not } = useLoaderData();

  return (
    <article>
      <title>{`${not.baslik} · Notlarım`}</title>
      <div className="flex items-start justify-between gap-4">
        <div>
          <h1 className="text-2xl font-bold">{not.baslik}</h1>
          <p className="text-sm text-slate-500">{new Date(not.tarih).toLocaleDateString("tr-TR")}</p>
        </div>
        <FavoriButonu not={not} />
      </div>

      <p className="mt-4 whitespace-pre-wrap">{not.icerik}</p>

      <Form
        method="post"
        className="mt-6"
        onSubmit={(e) => {
          if (!confirm("Bu not silinsin mi?")) e.preventDefault();
        }}
      >
        <button
          name="islem"
          value="sil"
          className="rounded border border-red-300 px-3 py-1.5 text-red-700 hover:bg-red-50"
        >
          Sil
        </button>
      </Form>
    </article>
  );
}

export function ErrorBoundary() {
  const hata = useRouteError();
  return (
    <div className="text-center">
      <h2 className="text-xl font-semibold">
        {isRouteErrorResponse(hata) ? `${hata.status} · ${hata.data}` : "Not yüklenirken hata oluştu."}
      </h2>
      <Link to="/notlar" className="mt-3 inline-block text-indigo-600 underline">Notlara dön</Link>
    </div>
  );
}
```

### Yeni not (action + doğrulama)

```jsx
// ============ src/routes/notlar/YeniNot.jsx ============
import { Form, redirect, useActionData, useNavigation } from "react-router";
import { notOlustur } from "../../api";

export async function action({ request }) {
  const formData = await request.formData();
  const baslik = String(formData.get("baslik") ?? "").trim();
  const icerik = String(formData.get("icerik") ?? "").trim();

  const hatalar = {};
  if (baslik.length < 3) hatalar.baslik = "Başlık en az 3 karakter olmalı.";
  if (!icerik) hatalar.icerik = "İçerik boş olamaz.";
  if (Object.keys(hatalar).length) return { hatalar, degerler: { baslik, icerik } };

  const not = await notOlustur({ baslik, icerik });
  return redirect(`/notlar/${not.id}`);
}

export default function YeniNot() {
  const sonuc = useActionData();
  const navigation = useNavigation();
  const kaydediliyor = navigation.state === "submitting";

  return (
    <Form method="post" className="flex flex-col gap-4">
      <title>Yeni not · Notlarım</title>
      <h1 className="text-xl font-bold">Yeni not</h1>

      <label className="flex flex-col gap-1">
        <span className="font-medium">Başlık</span>
        <input
          name="baslik"
          defaultValue={sonuc?.degerler?.baslik}
          aria-invalid={sonuc?.hatalar?.baslik ? true : undefined}
          className="rounded border border-slate-300 px-3 py-2 aria-invalid:border-red-500"
          autoFocus
        />
        {sonuc?.hatalar?.baslik && <span className="text-sm text-red-600">{sonuc.hatalar.baslik}</span>}
      </label>

      <label className="flex flex-col gap-1">
        <span className="font-medium">İçerik</span>
        <textarea
          name="icerik"
          rows={6}
          defaultValue={sonuc?.degerler?.icerik}
          aria-invalid={sonuc?.hatalar?.icerik ? true : undefined}
          className="rounded border border-slate-300 px-3 py-2 aria-invalid:border-red-500"
        />
        {sonuc?.hatalar?.icerik && <span className="text-sm text-red-600">{sonuc.hatalar.icerik}</span>}
      </label>

      <button
        type="submit"
        disabled={kaydediliyor}
        className="self-start rounded bg-indigo-600 px-4 py-2 text-white disabled:opacity-50"
      >
        {kaydediliyor ? "Kaydediliyor..." : "Kaydet"}
      </button>
    </Form>
  );
}
```

### Giriş ve çıkış

```jsx
// ============ src/routes/Giris.jsx ============
import { Form, redirect, useActionData, useNavigation, useSearchParams } from "react-router";
import { cikisYap, girisYap, mevcutKullanici } from "../api";

export function loader() {
  if (mevcutKullanici()) throw redirect("/notlar");   // zaten girişliyse
  return null;
}

export async function action({ request }) {
  const formData = await request.formData();
  const ad = String(formData.get("ad") ?? "").trim();
  if (ad.length < 2) return { hata: "Adınızı girin (en az 2 karakter)." };

  await girisYap(ad);
  const geriDon = new URL(request.url).searchParams.get("geriDon") ?? "/notlar";
  return redirect(geriDon);
}

export default function Giris() {
  const sonuc = useActionData();
  const navigation = useNavigation();
  const [aramaParametreleri] = useSearchParams();
  const geriDon = aramaParametreleri.get("geriDon");

  return (
    <div className="mx-auto max-w-sm rounded border border-slate-200 bg-white p-6">
      <title>Giriş · Notlarım</title>
      <h1 className="mb-1 text-xl font-bold">Giriş yap</h1>
      {geriDon && <p className="mb-4 text-sm text-slate-500">Devam etmek için giriş yapmalısın.</p>}
      <Form method="post" className="flex flex-col gap-3">
        <input name="ad" placeholder="Adın" className="rounded border border-slate-300 px-3 py-2" autoFocus />
        {sonuc?.hata && <p className="text-sm text-red-600">{sonuc.hata}</p>}
        <button
          disabled={navigation.state === "submitting"}
          className="rounded bg-indigo-600 px-4 py-2 text-white disabled:opacity-50"
        >
          Giriş
        </button>
      </Form>
    </div>
  );
}

export function cikisAction() {
  cikisYap();
  return redirect("/");
}
```

### Tembel yüklenen sayfa

```jsx
// ============ src/routes/Istatistik.jsx ============
import { useLoaderData } from "react-router";
import { notlariGetir } from "../api";

export async function loader() {
  const notlar = await notlariGetir();
  const toplamKelime = notlar.reduce(
    (t, n) => t + n.icerik.split(/\s+/).filter(Boolean).length,
    0
  );
  return { toplam: notlar.length, favori: notlar.filter((n) => n.favori).length, toplamKelime };
}

export function Component() {
  const { toplam, favori, toplamKelime } = useLoaderData();
  return (
    <section>
      <title>İstatistik · Notlarım</title>
      <h1 className="mb-4 text-2xl font-bold">İstatistik</h1>
      <dl className="grid grid-cols-3 gap-4 text-center">
        {[["Not", toplam], ["Favori", favori], ["Kelime", toplamKelime]].map(([etiket, deger]) => (
          <div key={etiket} className="rounded border border-slate-200 bg-white p-4">
            <dt className="text-sm text-slate-500">{etiket}</dt>
            <dd className="text-3xl font-bold tabular-nums">{deger}</dd>
          </div>
        ))}
      </dl>
    </section>
  );
}
```

### Kök hata sayfası

```jsx
// ============ src/routes/Hata.jsx ============
import { isRouteErrorResponse, Link, useRouteError } from "react-router";

export default function Hata() {
  const hata = useRouteError();
  const baslik = isRouteErrorResponse(hata)
    ? `${hata.status} · ${hata.status === 404 ? "Sayfa bulunamadı" : hata.statusText}`
    : "Beklenmeyen bir hata oluştu";

  return (
    <div className="grid min-h-dvh place-items-center bg-slate-50 p-8 text-center">
      <div>
        <title>{baslik}</title>
        <h1 className="text-3xl font-bold">{baslik}</h1>
        {hata instanceof Error && <p className="mt-2 text-slate-500">{hata.message}</p>}
        <Link to="/" className="mt-6 inline-block rounded bg-indigo-600 px-4 py-2 text-white">
          Ana sayfa
        </Link>
      </div>
    </div>
  );
}
```

### Router tanımı

```jsx
// ============ src/main.jsx ============
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { createBrowserRouter, data } from "react-router";
import { RouterProvider } from "react-router/dom";
import "./index.css";

import Kok, { loader as kokLoader } from "./routes/Kok";
import AnaSayfa from "./routes/AnaSayfa";
import Hata from "./routes/Hata";
import Giris, { loader as girisLoader, action as girisAction, cikisAction } from "./routes/Giris";
import NotlarDuzeni, { loader as notlarLoader } from "./routes/notlar/NotlarDuzeni";
import NotDetay, {
  loader as notLoader,
  action as notAction,
  ErrorBoundary as NotHatasi,
} from "./routes/notlar/NotDetay";
import YeniNot, { action as yeniNotAction } from "./routes/notlar/YeniNot";

const router = createBrowserRouter([
  {
    path: "/",
    Component: Kok,
    loader: kokLoader,
    ErrorBoundary: Hata,
    HydrateFallback: () => <p className="p-8">Yükleniyor...</p>,
    children: [
      { index: true, Component: AnaSayfa },
      { path: "giris", Component: Giris, loader: girisLoader, action: girisAction },
      { path: "cikis", action: cikisAction },                        // bileşensiz rota
      {
        path: "notlar",
        Component: NotlarDuzeni,
        loader: notlarLoader,                                          // korumalı + liste
        children: [
          {
            index: true,
            element: <p className="text-slate-500">Soldan bir not seç ya da yeni not oluştur.</p>,
          },
          { path: "yeni", Component: YeniNot, action: yeniNotAction },
          {
            path: ":notId",
            Component: NotDetay,
            loader: notLoader,
            action: notAction,
            ErrorBoundary: NotHatasi,
          },
        ],
      },
      { path: "istatistik", lazy: () => import("./routes/Istatistik") },   // ayrı chunk
      { path: "*", loader: () => { throw data("Sayfa bulunamadı", { status: 404 }); } },
    ],
  },
]);

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <RouterProvider router={router} />
  </StrictMode>
);
```

### Dene ve gözlemle

| Deneme | Gözlenen |
|---|---|
| Giriş yapmadan `/notlar/1`'e git | `/giris?geriDon=/notlar/1`'e yönlendirme; girişten sonra o nota dönüş |
| Arama kutusuna yaz | URL `?q=...` olur, loader yeniden çalışır; yenileme/paylaşımda aynı sonuç |
| Arama varken başka nota geç | `sorgu` linklere eklendiği için filtre korunur |
| Yeni not formunu boş gönder | Hatalar görünür, yazılanlar silinmez (`degerler`) |
| Geçerli not kaydet | Yeni nota yönlendirme; **kenar liste kendiliğinden güncellenir** (revalidation) |
| Yıldıza tıkla | Anında dolar (iyimser), sayfa değişmez (`useFetcher`) |
| Notu sil | Listeye dönülür, arama korunur |
| `/notlar/olmayan-id` | Sadece sağ panelde 404; menü ve liste yerinde (iç içe `ErrorBoundary`) |
| `/rastgele-bir-yol` | Kök hata sayfası |
| İlk kez "İstatistik" | Network sekmesinde ayrı bir JS dosyası iner (`lazy`) |
| Her geçişte | Üstte ince yükleniyor çubuğu; sekme başlığı sayfaya göre değişir |

> Bu uygulamada **hiç `useEffect` yok**, veri için **hiç `useState` yok**; yükleniyor ve hata durumlarının neredeyse tamamını router yönetiyor.

---

## 18. Özet

- **İstemci tarafı yönlendirme:** Sayfa yenilenmeden URL değişir, eşleşen bileşenler render edilir; **URL de bir state'tir.**
- **React Router v8** (Haziran 2026): Node 22.22+, React 19.2.7+, Vite 7+; **`react-router-dom` kaldırıldı** → `react-router` ve `react-router/dom`.
- Üç mod: **Declarative** (basit eşleme), **Data** (loader/action; yeni SPA'lar için önerilen), **Framework** (dosya tabanlı rotalar, SSR).
- `createBrowserRouter([...])` + `<RouterProvider>`; rota nesneleri: `path`, `Component`, `children`, `index`, `loader`, `action`, `ErrorBoundary`, `lazy`.
- **`<Link>`** sayfa yenilemeden gezinir; **`<NavLink>`** aktif durumu bilir (`isActive`, `aria-current`, `end`).
- **İç içe rotalar** URL parçalarını iç içe bileşenlerle eşler; düzenler **`<Outlet />`** ile çocuklarını gösterir; yolsuz düzen ve önek rotaları yapılabilir.
- **Dinamik segmentler** (`:id`, `?`, `*`) `useParams` ile okunur; değerler string'dir. Aynı rotada parametre değişince state sıfırlamak için `key`.
- **404** için `*` rotası; render/loader/action hataları en yakın **`ErrorBoundary`**'ye gider (`useRouteError`, `isRouteErrorResponse`).
- Programla yönlendirme: **`useNavigate`** (olay yöneticilerinde), **`<Navigate>`**, loader/action'da **`redirect()`**.
- **`useSearchParams`**: Filtre, sıralama, arama gibi paylaşılabilir state URL'de; sık değişimlerde `{ replace: true }`.
- **`useLocation`**: konum bilgisi ve URL'de görünmeyen geçiş state'i.
- **`loader`**: Rota render edilmeden önce veri yükler; `useLoaderData`; `request.signal` ile iptal; loader'lar paralel çalışır, şelale yoktur; `useNavigation` ile bekleme durumu; `HydrateFallback` ile ilk yükleme.
- **`action` + `<Form>`**: Form verisi action'a gider; `useActionData` ile doğrulama hataları, `redirect` ile yönlendirme; action sonrası tüm loader'lar **otomatik yenilenir.**
- **`useFetcher`**: Gezinmeden veri değiştirme; bağımsız durum; `fetcher.formData` ile **iyimser arayüz.**
- **Korumalı rotalar**: Düzen rotasının loader'ında kontrol + `redirect`; gerçek güvenlik sunucudadır. v8'de middleware varsayılan olarak etkin.
- **`lazy`** ile rotalar ayrı dosyalara bölünür (code splitting).
- **`<ScrollRestoration />`** kaydırma konumunu yönetir; React 19'da **`<title>`** bileşen içinde render edilir.
- SPA yayınlanırken sunucu bilinmeyen yollar için **`index.html`** döndürmelidir.
- Alternatif: **TanStack Router** (güçlü tip güvenliği).

---

⬅️ Önceki: [16a · Tailwind CSS (Detaylı)](./16a-tailwind-css.md) · ➡️ Sonraki: [18 · Veri Çekme](./18-veri-cekme.md)
