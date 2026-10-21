# 13 · Context API

> **Konu özeti:** Derin ağaçlarda prop drilling sorunu, context oluşturma / sağlama / okuma, React 19 `<Context value>` yazımı, varsayılan değer, en yakın sağlayıcı kuralı ve iç içe context'ler, context + state, sağlayıcı bileşeni ve custom hook ile düzenleme, birden fazla context, performans (memoizasyon ve context bölme), React 19 `use(Context)`, context'in ne zaman kullanılıp kullanılmayacağı.

---

## İçindekiler

1. [Problem: Derin Ağaçlarda Prop Drilling](#1-problem-derin-ağaçlarda-prop-drilling)
2. [Context Kullanımı: Üç Adım](#2-context-kullanımı-üç-adım)
3. [Varsayılan Değer](#3-varsayılan-değer)
4. [En Yakın Sağlayıcı Kazanır](#4-en-yakın-sağlayıcı-kazanır-iç-içe-contextler)
5. [Context + State: Değişen Değerler](#5-context--state-değişen-değerler-)
6. [Sağlayıcı Bileşeni ve Custom Hook](#6-contexti-düzenlemek-sağlayıcı-bileşeni-ve-custom-hook-)
7. [Birden Fazla Context](#7-birden-fazla-context)
8. [Performans: Context ve Yeniden Render](#8-performans-context-ve-yeniden-render-)
9. [React 19: `use(Context)`](#9-react-19-usecontext)
10. [Ne Zaman Context Kullanmalı?](#10-ne-zaman-context-kullanmalı-ve-kullanmamalı)
11. [Her Şey Bir Arada: Tema, Oturum ve Dil](#11-her-şey-bir-arada-tema-oturum-ve-dil)
12. [Özet](#12-özet)

---

## 1. Problem: Derin Ağaçlarda Prop Drilling

```
App                      tema="koyu", kullanici={...}
└── Sayfa                ← tema, kullanici (kullanmıyor, aktarıyor)
    └── Layout           ← tema, kullanici (kullanmıyor, aktarıyor)
        ├── Header       ← tema, kullanici (kullanmıyor, aktarıyor)
        │   └── Profil   ← kullanici ✅ kullanıyor
        └── Icerik       ← tema (kullanmıyor, aktarıyor)
            └── Kart     ← tema (kullanmıyor, aktarıyor)
                └── Buton ← tema ✅ kullanıyor
```

Bazı veriler uygulamanın **her yerinde**, **birçok bileşen** tarafından gerekir: tema, oturum açmış kullanıcı, dil, para birimi, saat dilimi…

**Context**, bir ebeveynin altındaki **tüm ağaca**, ne kadar derin olursa olsun, props'suz veri sağlar:

```
App  ── sağlar: tema, kullanici ──────────────┐
└── Sayfa                                     │
    └── Layout                                │  (aradakiler hiçbir şey bilmiyor)
        ├── Header                            │
        │   └── Profil  ← "kullanici" okuyor ◄─┤
        └── Icerik                            │
            └── Kart                          │
                └── Buton ← "tema" okuyor ◄───┘
```

---

## 2. Context Kullanımı: Üç Adım

### Adım 1: Context oluştur

```jsx
// TemaContext.js
import { createContext } from "react";

export const TemaContext = createContext("acik");   // "acik" = varsayılan değer
```

- `createContext` bir **context nesnesi** döndürür: bileşen veya veri değil, bir "kanal".
- Bileşen dışında, modül seviyesinde oluşturulur ve export edilir.

### Adım 2: Değeri sağla (provide)

```jsx
// App.jsx
import { TemaContext } from "./TemaContext";

export default function App() {
  return (
    <TemaContext value="koyu">
      <Sayfa />
    </TemaContext>
  );
}
```

### Adım 3: Değeri oku (consume)

```jsx
// Buton.jsx
import { useContext } from "react";
import { TemaContext } from "./TemaContext";

export default function Buton({ children }) {
  const tema = useContext(TemaContext);   // "koyu"
  return <button className={`btn btn-${tema}`}>{children}</button>;
}
```

- `useContext`, ağaçta **yukarı doğru** gidip bu context'i sağlayan **en yakın** bileşenin değerini verir.
- Aradaki bileşenler hiçbir şeyden haberdar olmaz.

> 📜 **Eski yazımlar:**
> - React 18 ve öncesi: `<TemaContext.Provider value="koyu">` — hâlâ çalışır, ileride kaldırılması planlanıyor.
> - Hook'lardan önce: `<TemaContext.Consumer>{(tema) => ...}</TemaContext.Consumer>` render prop'u.

---

## 3. Varsayılan Değer

`createContext`'e verilen değer, bileşenin **üstünde hiç sağlayıcı yoksa** kullanılır:

```jsx
export const TemaContext = createContext("acik");

function App() {
  return <Buton>Tıkla</Buton>;   // sağlayıcı yok → "acik"
}
```

- Sağlayıcı varsa onun `value`'su kullanılır; varsayılan değer onu **tamamlamaz veya birleştirmez.**

> ⚠️ **Yanılgı:** "Varsayılan değeri değiştirince context güncellenir." Hayır; varsayılan değer sabit bir yedektir. Context değeri, sağlayıcının `value`'su (genelde state ile) değiştirilerek güncellenir.

### Varsayılan değer seçimi

| Varsayılan | Anlamı |
|---|---|
| Anlamlı bir yedek (ör. `"acik"`) | Sağlayıcı olmasa da çalışabilir |
| `null` | "Mutlaka bir sağlayıcı içinde kullanılmalı" — custom hook'ta hata kontrolüyle birleştirilir |

---

## 4. En Yakın Sağlayıcı Kazanır: İç İçe Context'ler

```jsx
function App() {
  return (
    <TemaContext value="acik">
      <Header />                          {/* "acik" */}
      <main>
        <Kart />                          {/* "acik" */}
        <TemaContext value="koyu">
          <Kart />                        {/* "koyu" — en yakın sağlayıcı */}
          <Footer />                      {/* "koyu" */}
        </TemaContext>
      </main>
    </TemaContext>
  );
}
```

> CSS kalıtımına benzer: bir alt ağaçta değer yeniden tanımlanırsa oradan aşağısı yeni değeri alır.

### Kendi kendini besleyen context: Başlık seviyeleri

```jsx
const SeviyeContext = createContext(0);

function Bolum({ children }) {
  const seviye = useContext(SeviyeContext);   // üstteki seviyeyi oku
  return (
    <section>
      <SeviyeContext value={seviye + 1}>       {/* bir artırıp alt ağaca sağla */}
        {children}
      </SeviyeContext>
    </section>
  );
}

function Baslik({ children }) {
  const seviye = useContext(SeviyeContext);
  const Etiket = `h${Math.min(seviye, 6)}`;    // "h1", "h2", ...
  return <Etiket>{children}</Etiket>;
}

function Makale() {
  return (
    <Bolum>
      <Baslik>React Notları</Baslik>           {/* h1 */}
      <Bolum>
        <Baslik>Context API</Baslik>           {/* h2 */}
        <Bolum>
          <Baslik>Varsayılan Değer</Baslik>    {/* h3 */}
        </Bolum>
      </Bolum>
    </Bolum>
  );
}
```

- `Baslik` seviye prop'u almadan, bulunduğu yere göre doğru seviyeyi bulur.
- Bileşenler taşındığında seviyeler otomatik uyum sağlar.

---

## 5. Context + State: Değişen Değerler ⭐

```jsx
export default function App() {
  const [tema, setTema] = useState("acik");

  return (
    <TemaContext value={tema}>
      <label>
        <input
          type="checkbox"
          checked={tema === "koyu"}
          onChange={(e) => setTema(e.target.checked ? "koyu" : "acik")}
        />
        Koyu tema
      </label>
      <Sayfa />
    </TemaContext>
  );
}
```

- `tema` değişince sağlayıcıya yeni `value` gider ve bu context'i **okuyan tüm bileşenler** yeniden render olur.
- Aradaki bileşenler yeni props almasa da tüketiciler güncellenir.

### Derindeki bileşen değeri değiştirmek isterse

Context'e state ile birlikte **onu değiştiren fonksiyon** da konur:

```jsx
export default function App() {
  const [tema, setTema] = useState("acik");
  const temaDegistir = () => setTema((t) => (t === "acik" ? "koyu" : "acik"));

  return (
    <TemaContext value={{ tema, temaDegistir }}>
      <Sayfa />
    </TemaContext>
  );
}

// Ağacın derinliklerinde:
function TemaButonu() {
  const { tema, temaDegistir } = useContext(TemaContext);
  return (
    <button onClick={temaDegistir}>
      {tema === "acik" ? "🌙 Koyu" : "☀️ Açık"}
    </button>
  );
}
```

> **Context, state'in kendisi değildir; sadece bir taşıma mekanizmasıdır.** Değeri tutan ve değiştiren hâlâ `useState` (veya `useReducer`).

---

## 6. Context'i Düzenlemek: Sağlayıcı Bileşeni ve Custom Hook ⭐

Önerilen yapı: **Her context için bir dosya** → context + sağlayıcı bileşeni + custom hook.

```jsx
// src/context/TemaContext.jsx
import { createContext, useContext, useState } from "react";

const TemaContext = createContext(null);   // dışarı export edilmiyor

export function TemaProvider({ children }) {
  const [tema, setTema] = useState(() => localStorage.getItem("tema") ?? "acik");

  function temaDegistir() {
    setTema((t) => (t === "acik" ? "koyu" : "acik"));
  }

  return (
    <TemaContext value={{ tema, temaDegistir }}>
      {children}
    </TemaContext>
  );
}

export function useTema() {
  const deger = useContext(TemaContext);
  if (deger === null) {
    throw new Error("useTema, bir <TemaProvider> içinde kullanılmalı.");
  }
  return deger;
}
```

```jsx
// src/main.jsx
import { TemaProvider } from "./context/TemaContext";

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <TemaProvider>
      <App />
    </TemaProvider>
  </StrictMode>
);
```

```jsx
// Herhangi bir bileşende
import { useTema } from "../context/TemaContext";

function TemaButonu() {
  const { tema, temaDegistir } = useTema();
  return <button onClick={temaDegistir}>{tema === "acik" ? "🌙" : "☀️"}</button>;
}
```

### Kazanımlar

| Kazanım | Açıklama |
|---|---|
| **Kapsülleme** | Tema mantığı tek dosyada; bileşenler nasıl saklandığını bilmez |
| **Temiz API** | Bileşenler `useTema()` yazar; context nesnesini import etmez |
| **Güvenlik** | Sağlayıcı unutulursa sessiz `null` yerine açık bir hata mesajı |

> `useTema` gibi `use` ile başlayan ve içinde Hook çağıran fonksiyonlara **custom Hook** denir *(Konu 15)*.

> 📌 `localStorage`'a yazmak gibi yan etkiler updater içinde değil, `useEffect` ile `tema` değiştiğinde yapılmalıdır (bkz. [Bölüm 11](#11-her-şey-bir-arada-tema-oturum-ve-dil)).

---

## 7. Birden Fazla Context

Her bağımsız veri **kendi context'ine** sahip olmalı; tek dev "AppContext" kötü bir fikirdir (bkz. [Bölüm 8](#8-performans-context-ve-yeniden-render-)).

```jsx
// main.jsx
<TemaProvider>
  <AuthProvider>
    <SepetProvider>
      <App />
    </SepetProvider>
  </AuthProvider>
</TemaProvider>
```

- Sıra **bağımlılıklara** göre belirlenir: `SepetProvider` kullanıcı bilgisine ihtiyaç duyuyorsa `AuthProvider`'ın **içinde** olmalı.

### Sağlayıcıları toplamak

```jsx
export function Providers({ children }) {
  return (
    <TemaProvider>
      <AuthProvider>
        <SepetProvider>{children}</SepetProvider>
      </AuthProvider>
    </TemaProvider>
  );
}
```

---

## 8. Performans: Context ve Yeniden Render ⭐

Context değeri değişince, onu okuyan **her bileşen** yeniden render olur.

### 1. Her render'da yeni nesne tuzağı

```jsx
function TemaProvider({ children }) {
  const [tema, setTema] = useState("acik");
  const temaDegistir = () => setTema((t) => (t === "acik" ? "koyu" : "acik"));

  return (
    <TemaContext value={{ tema, temaDegistir }}>   {/* her render'da YENİ nesne */}
      {children}
    </TemaContext>
  );
}
```

- Sağlayıcı başka bir sebeple render olursa `tema` değişmese bile **tüm tüketiciler** render olur (`Object.is` yeni nesneyi farklı sayar).

#### Çözüm: `useMemo` ve `useCallback`

```jsx
import { useCallback, useMemo, useState } from "react";

function TemaProvider({ children }) {
  const [tema, setTema] = useState("acik");

  const temaDegistir = useCallback(() => {
    setTema((t) => (t === "acik" ? "koyu" : "acik"));
  }, []);

  const deger = useMemo(() => ({ tema, temaDegistir }), [tema, temaDegistir]);

  return <TemaContext value={deger}>{children}</TemaContext>;
}
```

- "Bağımlılıkları değişmedikçe aynı nesneyi/fonksiyonu geri ver" *(detay: Konu 19)*.

> 💡 **React Compiler** etkinse bu memoizasyon çoğu zaman otomatik yapılır.

> 💡 `children` prop olarak geldiği için, sağlayıcı kendi state'i yüzünden render olduğunda `children` elementleri yeniden oluşturulmaz; yalnızca **tüketiciler** render olur. Sağlayıcıyı ayrı bileşene çıkarmanın bir faydası da budur.

### 2. Context'leri bölmek

```jsx
// ❌ Tek dev context
<AppContext value={{ tema, kullanici, sepet, bildirimler /* ... */ }}>
// sepet her değiştiğinde, sadece tema'yı okuyan bileşenler de render olur
```

- İlgisiz veriler **ayrı context'lere** bölünür.
- Bir adım ileri: Aynı veri için **değer** ve **işlemler** ayrı context'lerde:

```jsx
const SepetContext = createContext(null);           // veri: sık değişir
const SepetIslemleriContext = createContext(null);  // fonksiyonlar: hiç değişmez

export function SepetProvider({ children }) {
  const [sepet, setSepet] = useState({});

  const islemler = useMemo(() => ({
    ekle: (id) => setSepet((s) => ({ ...s, [id]: (s[id] ?? 0) + 1 })),
    sil: (id) => setSepet(({ [id]: _, ...kalan }) => kalan),
    bosalt: () => setSepet({}),
  }), []);   // setSepet sabittir → islemler hiç değişmez

  return (
    <SepetContext value={sepet}>
      <SepetIslemleriContext value={islemler}>
        {children}
      </SepetIslemleriContext>
    </SepetContext>
  );
}

export const useSepet = () => useContext(SepetContext);
export const useSepetIslemleri = () => useContext(SepetIslemleriContext);
```

- Sadece "Sepete ekle" butonu olan ürün kartları `useSepetIslemleri()` okur → sepet değişince **render olmaz.**
- Sepet içeriğini gösteren birkaç bileşen `useSepet()` okur → güncellenir.

> Çoğu uygulamada bu optimizasyonlara gerek yoktur. Önce doğru yapıyı kur; sorun **ölçerek** (React DevTools Profiler) görünürse optimize et.

---

## 9. React 19: `use(Context)`

```jsx
import { use } from "react";

function Buton({ children }) {
  const { tema } = use(TemaContext);
  return <button className={`btn btn-${tema}`}>{children}</button>;
}
```

- `use`, "sadece en üst seviyede çağır" kuralına **tabi değildir**; koşul içinde veya erken `return` sonrasında çağrılabilir:

```jsx
function Baslik({ children, temaliMi }) {
  if (!temaliMi) {
    return <h2>{children}</h2>;          // erken return
  }
  const { tema } = use(TemaContext);     // ✅ use ile mümkün, useContext ile ❌
  return <h2 className={`baslik-${tema}`}>{children}</h2>;
}
```

| | `useContext` | `use` |
|---|---|---|
| Context okuma | ✅ | ✅ |
| Koşullu / erken return sonrası | ❌ | ✅ |
| Promise okuma (Suspense) | ❌ | ✅ *(Konu 18, 20)* |

> Context için `useContext` hâlâ tamamen geçerli ve en yaygın yöntemdir.

---

## 10. Ne Zaman Context Kullanmalı (ve Kullanmamalı)?

### Gereksiz kullanımın zararları

- Veri akışını takip etmek zorlaşır.
- Bileşenler belirli bir sağlayıcıya **bağımlı** olur; tekrar kullanım ve test zorlaşır.

### Context'ten önce dene

1. **Props ile geçirmek** — birkaç seviye props geçirmek kötü değildir; açık ve izlenebilirdir.
2. **Kompozisyon** — aradaki bileşenler veriyi kullanmıyorsa JSX'i `children` / slot props ile aktar.

### Uygun kullanım alanları

| Kullanım | Neden uygun |
|---|---|
| **Tema** | Ağacın her yerine dağılmış çok sayıda bileşen kullanır |
| **Oturum açmış kullanıcı** | Birçok yerde "kim giriş yaptı?" bilgisi lazım |
| **Dil / yerelleştirme** | Her metin bileşeni çeviriye ihtiyaç duyar |
| **Yönlendirme (routing)** | Router kütüphaneleri mevcut sayfayı context ile sağlar *(Konu 17)* |
| **Bileşen grubunun iç iletişimi** | `<Sekmeler>` ve `<Sekme>` gibi birlikte çalışan bileşenler |
| **Global, nadiren değişen ayarlar** | Para birimi, özellik bayrakları (feature flags) |

### Context'in yetmediği yerler

| Durum | Araç |
|---|---|
| Çok sık değişen, karmaşık, her yerden güncellenen global state | **Zustand**, **Redux Toolkit**, **Jotai** (seçici abonelik sunar) |
| Sunucudan gelen veri (aslında önbellek) | **TanStack Query** *(Konu 18)* |

> **İlk refleks:** props → kompozisyon → context → kütüphane.

---

## 11. Her Şey Bir Arada: Tema, Oturum ve Dil

```jsx
// ============ src/context/TemaContext.jsx ============
import { createContext, useCallback, useContext, useEffect, useMemo, useState } from "react";

const TemaContext = createContext(null);

export function TemaProvider({ children }) {
  const [tema, setTema] = useState(() => localStorage.getItem("tema") ?? "acik");

  // Dış sistemlerle senkronizasyon: <html> niteliği ve localStorage
  useEffect(() => {
    document.documentElement.dataset.tema = tema;
    localStorage.setItem("tema", tema);
  }, [tema]);

  const temaDegistir = useCallback(() => {
    setTema((t) => (t === "acik" ? "koyu" : "acik"));
  }, []);

  const deger = useMemo(() => ({ tema, temaDegistir }), [tema, temaDegistir]);
  return <TemaContext value={deger}>{children}</TemaContext>;
}

export function useTema() {
  const d = useContext(TemaContext);
  if (!d) throw new Error("useTema, <TemaProvider> içinde kullanılmalı.");
  return d;
}
```

```jsx
// ============ src/context/DilContext.jsx ============
import { createContext, useContext, useMemo, useState } from "react";

const CEVIRILER = {
  tr: { hosgeldin: "Hoş geldin", giris: "Giriş yap", cikis: "Çıkış yap", misafir: "Misafir", tema: "Tema" },
  en: { hosgeldin: "Welcome", giris: "Sign in", cikis: "Sign out", misafir: "Guest", tema: "Theme" },
};

const DilContext = createContext(null);

export function DilProvider({ children }) {
  const [dil, setDil] = useState("tr");

  const deger = useMemo(() => ({
    dil,
    setDil,
    t: (anahtar) => CEVIRILER[dil][anahtar] ?? anahtar,   // çeviri fonksiyonu
  }), [dil]);

  return <DilContext value={deger}>{children}</DilContext>;
}

export function useDil() {
  const d = useContext(DilContext);
  if (!d) throw new Error("useDil, <DilProvider> içinde kullanılmalı.");
  return d;
}
```

```jsx
// ============ src/context/AuthContext.jsx ============
import { createContext, useCallback, useContext, useMemo, useState } from "react";

const AuthContext = createContext(null);

export function AuthProvider({ children }) {
  const [kullanici, setKullanici] = useState(null);
  const [durum, setDurum] = useState("bosta"); // "bosta" | "yukleniyor"

  const girisYap = useCallback(async (ad) => {
    setDurum("yukleniyor");
    await new Promise((r) => setTimeout(r, 600));   // sahte API
    setKullanici({ ad, rol: ad === "admin" ? "admin" : "uye" });
    setDurum("bosta");
  }, []);

  const cikisYap = useCallback(() => setKullanici(null), []);

  const deger = useMemo(
    () => ({ kullanici, yukleniyor: durum === "yukleniyor", girisYap, cikisYap }),
    [kullanici, durum, girisYap, cikisYap]
  );

  return <AuthContext value={deger}>{children}</AuthContext>;
}

export function useAuth() {
  const d = useContext(AuthContext);
  if (!d) throw new Error("useAuth, <AuthProvider> içinde kullanılmalı.");
  return d;
}
```

```jsx
// ============ src/Providers.jsx ============
import { TemaProvider } from "./context/TemaContext";
import { DilProvider } from "./context/DilContext";
import { AuthProvider } from "./context/AuthContext";

export function Providers({ children }) {
  return (
    <TemaProvider>
      <DilProvider>
        <AuthProvider>{children}</AuthProvider>
      </DilProvider>
    </TemaProvider>
  );
}
```

```jsx
// ============ src/App.jsx ============
import { Providers } from "./Providers";
import { useTema } from "./context/TemaContext";
import { useDil } from "./context/DilContext";
import { useAuth } from "./context/AuthContext";

function TemaButonu() {
  const { tema, temaDegistir } = useTema();
  const { t } = useDil();
  return (
    <button type="button" onClick={temaDegistir} aria-label={t("tema")}>
      {tema === "acik" ? "🌙" : "☀️"}
    </button>
  );
}

function DilSecici() {
  const { dil, setDil } = useDil();
  return (
    <select value={dil} onChange={(e) => setDil(e.target.value)}>
      <option value="tr">🇹🇷 TR</option>
      <option value="en">🇬🇧 EN</option>
    </select>
  );
}

function KullaniciMenusu() {
  const { kullanici, yukleniyor, girisYap, cikisYap } = useAuth();
  const { t } = useDil();

  if (!kullanici) {
    return (
      <button type="button" onClick={() => girisYap("Ayşe")} disabled={yukleniyor}>
        {yukleniyor ? "..." : t("giris")}
      </button>
    );
  }

  return (
    <span>
      👤 {kullanici.ad}{" "}
      <button type="button" onClick={cikisYap}>{t("cikis")}</button>
    </span>
  );
}

function Header() {
  // Header hiçbir context değerini kullanmıyor, sadece bileşenleri yerleştiriyor
  return (
    <header style={{ display: "flex", gap: 12, alignItems: "center" }}>
      <strong>⚛️ ReactNotlar</strong>
      <div style={{ marginLeft: "auto", display: "flex", gap: 8 }}>
        <DilSecici />
        <TemaButonu />
        <KullaniciMenusu />
      </div>
    </header>
  );
}

function Karsilama() {
  const { kullanici } = useAuth();
  const { t } = useDil();
  return <h1>{t("hosgeldin")}, {kullanici?.ad ?? t("misafir")}!</h1>;
}

function Panel() {
  return (
    <main>
      <Karsilama />
      <section>
        <p>Bu bölüm prop almadan üç context'e de erişebilen bileşenler içeriyor.</p>
      </section>
    </main>
  );
}

export default function App() {
  return (
    <Providers>
      <Header />
      <Panel />
    </Providers>
  );
}
```

```css
/* TemaProvider <html data-tema="..."> niteliğini ayarlıyor */
:root[data-tema="acik"] { --arkaplan: #ffffff; --yazi: #1f2937; }
:root[data-tema="koyu"] { --arkaplan: #111827; --yazi: #f9fafb; }
body { background: var(--arkaplan); color: var(--yazi); }
```

### Bu örnekte kullanılanlar

- Her context için ayrı dosya: sağlayıcı bileşeni + hata kontrollü custom hook
- React 19 `<Context value>` yazımı
- `null` varsayılan değer
- State + context birleşimi
- Context'e fonksiyon koyma (`temaDegistir`, `girisYap`, `t`)
- `useMemo` / `useCallback` ile sabit context değerleri
- Sağlayıcı içinde `useEffect` ile dış sistem senkronizasyonu (`<html>` niteliği, `localStorage`)
- Birbirinden bağımsız üç context ve `Providers` ile toplama
- `Header`, `Panel` gibi ara bileşenler hiçbir prop taşımıyor

---

## 12. Özet

- **Context**, bir ebeveynin altındaki tüm ağaca props'suz veri sağlar; derin prop drilling'i çözer.
- Üç adım: `createContext(varsayilan)` → `<Context value={...}>` ile sağla → `useContext(Context)` ile oku.
- **React 19:** Context doğrudan sağlayıcı olarak kullanılır; `<Context.Provider>` eski yazımdır.
- Varsayılan değer yalnızca **sağlayıcı yoksa** kullanılır; `null` vermek "sağlayıcı zorunlu" niyetini belirtir.
- `useContext` **en yakın** sağlayıcıyı bulur; alt ağaçlarda değer geçersiz kılınabilir.
- Context **state değildir**, taşıma mekanizmasıdır; değişen değerler için state ile birleştirilir, güncelleyici fonksiyonlar da context'e konur.
- Önerilen yapı: context + **sağlayıcı bileşeni** + hata kontrollü **custom hook** (`useTema`) tek dosyada.
- Her bağımsız veri kendi context'inde; sağlayıcıların sırası bağımlılıklara göre belirlenir.
- Context değeri değişince **tüm tüketiciler** render olur; nesne değerleri `useMemo`, fonksiyonlar `useCallback` ile sabitlenir (veya React Compiler).
- Sık değişen ve nadir değişen veriler, gerekirse **değer** ve **işlemler**, ayrı context'lere bölünür.
- **`use(Context)`** (React 19) koşullu olarak da çağrılabilir.
- Önce props ve kompozisyon denenir; context tema, kullanıcı, dil, routing gibi gerçekten global veriler içindir. Karmaşık global state için Zustand/Redux, sunucu verisi için TanStack Query.

---

⬅️ Önceki: [12 · Bileşenler Arası İletişim](./12-bilesenler-arasi-iletisim.md) · ➡️ Sonraki: [14 · useReducer](./14-usereducer.md)
