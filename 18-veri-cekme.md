# 18 · Veri Çekme ve TanStack Query

> **Konu özeti:** Sunucu state'i ile istemci state'inin farkı, API katmanı, TanStack Query kurulumu, `useQuery` ve durum alanları, query key'ler ve `queryOptions`, önbellek yaşam döngüsü (`staleTime`, `gcTime`), koşullu/bağımlı/paralel sorgular, sayfalama ve sonsuz kaydırma, `useMutation` ve önbellek güncelleme, iyimser güncellemeler, önceden yükleme ve React Router ile birlikte kullanım, Suspense, `select` ve diğer özellikler, alternatifler.
> **Sürüm:** `@tanstack/react-query` v5.x

---

## İçindekiler

1. [Sezgisel Bakış](#1-sezgisel-bakış)
2. [Sunucu State'i ve İstemci State'i](#2-sunucu-statei-ve-istemci-statei-)
3. [Önce Temel: Bir API Katmanı](#3-önce-temel-bir-api-katmanı)
4. [Kurulum](#4-kurulum)
5. [`useQuery`: Veri Okumak](#5-usequery-veri-okumak-)
6. [Query Key'ler ve `queryOptions`](#6-query-keyler-ve-queryoptions-)
7. [Önbellek Yaşam Döngüsü: `staleTime` ve `gcTime`](#7-önbellek-yaşam-döngüsü-staletime-ve-gctime-)
8. [Koşullu, Bağımlı ve Paralel Sorgular](#8-koşullu-bağımlı-ve-paralel-sorgular)
9. [Sayfalama ve Sonsuz Kaydırma](#9-sayfalama-ve-sonsuz-kaydırma)
10. [Veri Değiştirme: `useMutation`](#10-veri-değiştirme-usemutation-)
11. [İyimser Güncellemeler](#11-iyimser-güncellemeler-optimistic-updates)
12. [Önceden Yükleme ve React Router ile Kullanım](#12-önceden-yükleme-ve-react-router-ile-kullanım)
13. [Suspense ile Veri Çekme](#13-suspense-ile-veri-çekme)
14. [Faydalı Diğer Özellikler](#14-faydalı-diğer-özellikler)
15. [Alternatifler ve Karar Rehberi](#15-alternatifler-ve-karar-rehberi)
16. [Her Şey Bir Arada: Ürün Kataloğu](#16-her-şey-bir-arada-ürün-kataloğu)
17. [Özet](#17-özet)

---

## 1. Sezgisel Bakış

### TanStack Query kullanmazsak ne olur?

Uygulama yine çalışır; ama aynı işler her bileşende **elle** yazılır ve bazı şeyler hiç olmaz.

**Senaryo:** Ürün listesi → bir ürüne tıkla → geri dön.

| TanStack Query olmadan (`useState` + `useEffect`) | TanStack Query ile |
|---|---|
| Geri dönünce liste **tekrar yüklenir** (boş ekran, yeni istek); bileşen kalkınca state silinmişti | Veri **ortak önbellekte** durur; liste **anında** görünür, gerekirse arka planda tazelenir |
| Aynı veriyi gösteren 3 bileşen → **3 ayrı istek** | Aynı veriyi isteyen bileşenler **tek istek** atar, veriyi paylaşır |
| Favoriye ekleyince başlıktaki sayaç güncellenmez; elle haber vermek gerekir | `invalidateQueries` → o veriyi gösteren **her yer** kendiliğinden güncellenir |
| Her bileşende yükleniyor / hata / iptal kodu tekrar yazılır | Yükleniyor, hata, iptal, yeniden deneme hazır gelir |

### "Veri state yönetimi" ne demek?

Sunucudan gelen veriyi her bileşen kendi `useState`'inde ayrı ayrı tutmaz; TanStack Query onu **tek bir merkezi depoda** tutar ve bileşenler oradan okur.

> Context'e benzer; üstüne **"bu veri ne zaman eskidi, ne zaman yeniden çekilmeli"** mantığı eklenmiş hali.

> Küçük, tek sayfalık uygulamada kullanmamak pek fark etmez. Veri birçok sayfada paylaşılıyor ve sık değişiyorsa, ya bu işler elle yazılır ya da kullanıcı sürekli "Yükleniyor..." görür.

### `queryKey` nedir?

> **`queryKey`, önbellekteki verinin adıdır** — dolaptaki çekmecelerin üzerindeki etiket gibi.

```jsx
useQuery({ queryKey: ["urunler", 5], queryFn: () => urunGetir(5) });
```

"`['urunler', 5]` etiketli çekmeceye bak. Veri varsa onu ver. Yoksa `queryFn`'i çalıştır, gelen veriyi bu çekmeceye koy."

**Üç işi var:**

**a) Paylaşım:** Aynı key'i kullanan bileşenler **aynı çekmeceye** bakar → tek istek.

**b) Değişkene göre ayrı veri:** Key'deki değer değişince **farklı çekmece** açılır.

```jsx
["urunler", 5]   // 5 numaralı ürünün çekmecesi
["urunler", 6]   // 6 numaralı ürünün çekmecesi, ayrı veri
```

→ `queryFn` içinde kullanılan her değişken (id, arama metni, sayfa) key'de olmalıdır; yoksa id değişse de hep aynı çekmeceye bakılır.

**c) Toplu işlem:** Genelden özele yazılan key'lerle bir grup birden hedeflenir.

```js
queryClient.invalidateQueries({ queryKey: ["urunler"] });
// ["urunler", 5], ["urunler", 6], ["urunler", "liste", ...] hepsi "eskidi" olarak işaretlenir
```

> **`queryKey`** = "hangi veri?" · **`queryFn`** = "o veri nasıl getirilir?"

---

## 2. Sunucu State'i ve İstemci State'i ⭐

| | **İstemci state'i** | **Sunucu state'i** |
|---|---|---|
| Örnek | Modal açık mı, seçili sekme, form taslağı, tema | Ürün listesi, profil, siparişler, yorumlar |
| Asıl sahibi | Tarayıcı | **Uzaktaki sunucu** |
| Güncel mi? | Her zaman | Elindeki bir **kopya**; başkası değiştirmiş olabilir |
| Erişim | Senkron, anında | Asenkron, gecikmeli, başarısız olabilir |
| Araçlar | `useState`, `useReducer`, context | TanStack Query vb. |

### Sunucu state'ini elle yönetirken çözülmesi gerekenler

1. **Önbellek** — sayfaya dönünce neden tekrar yükleniyor?
2. **Tekrarlanan istekler** — aynı veri için birden çok istek
3. **Bayatlık** — veri ne zaman eski sayılır?
4. **Arka planda yenileme** — eskiyi gösterirken yenisini çekmek
5. **Yeniden deneme** — ağ kopunca
6. **Sayfalama / sonsuz kaydırma** — sayfa değişirken ekranın boşalmaması
7. **Mutasyon sonrası senkronizasyon** — değişikliği gösteren tüm listelerin güncellenmesi
8. **İyimser güncellemeler** — beklemeden göster, hata olursa geri al

> TanStack Query bir "veri çekme kütüphanesi" değildir; veriyi sen çekersin (`fetch`, axios...). TanStack Query, **çekilen sunucu verisinin önbelleğini ve yaşam döngüsünü** yönetir.

---

## 3. Önce Temel: Bir API Katmanı

`fetch` çağrıları bileşenlere dağıtılmaz; küçük bir API katmanında toplanır.

```js
// src/api/istemci.js
const TABAN_URL = "https://dummyjson.com";

export class ApiHatasi extends Error {
  constructor(mesaj, durum) {
    super(mesaj);
    this.durum = durum;
  }
}

export async function istek(yol, { signal, ...secenekler } = {}) {
  const res = await fetch(`${TABAN_URL}${yol}`, {
    headers: { "Content-Type": "application/json" },
    signal,
    ...secenekler,
  });

  if (!res.ok) {
    // fetch 404/500'de hata FIRLATMAZ → kendimiz fırlatıyoruz
    throw new ApiHatasi(`İstek başarısız: ${res.status}`, res.status);
  }
  return res.json();
}
```

```js
// src/api/urunler.js
import { istek } from "./istemci";

export function urunleriGetir({ q = "", sayfa = 1, limit = 12 }, signal) {
  const skip = (sayfa - 1) * limit;
  return istek(`/products/search?q=${encodeURIComponent(q)}&limit=${limit}&skip=${skip}`, { signal });
}

export function urunGetir(id, signal) {
  return istek(`/products/${id}`, { signal });
}
```

### Kazanımlar

- Hata kontrolü tek yerde (`res.ok` unutulmaz)
- Taban URL ve ortak başlıklar (token vb.) tek yerde
- Bileşenler ve TanStack Query "API nasıl çağrılır"ı bilmez

> 💡 **axios** da yaygındır; ama modern tarayıcılarda `fetch` + küçük bir sarmalayıcı çoğu ihtiyaç için yeterlidir.

---

## 4. Kurulum

```bash
npm install @tanstack/react-query
npm install -D @tanstack/react-query-devtools
```

```jsx
// src/main.jsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";
import App from "./App";

const queryClient = new QueryClient();   // bileşen DIŞINDA, bir kez

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <QueryClientProvider client={queryClient}>
      <App />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  </StrictMode>
);
```

| Parça | Görevi |
|---|---|
| `QueryClient` | Önbelleğin kendisi ve ayarların sahibi; **bileşen dışında** oluşturulur |
| `QueryClientProvider` | Context ile tüm uygulamaya sağlar |
| `ReactQueryDevtools` | Önbellekteki sorguları, durumlarını ve verilerini gösterir; sadece geliştirmede |

---

## 5. `useQuery`: Veri Okumak ⭐

```jsx
import { useQuery } from "@tanstack/react-query";
import { urunGetir } from "../api/urunler";

function UrunDetay({ id }) {
  const { data, isPending, isError, error } = useQuery({
    queryKey: ["urunler", id],
    queryFn: ({ signal }) => urunGetir(id, signal),
  });

  if (isPending) return <p>Yükleniyor...</p>;
  if (isError) return <p role="alert">Hata: {error.message}</p>;

  return (
    <article>
      <h1>{data.title}</h1>
      <p>{data.description}</p>
    </article>
  );
}
```

| Parametre | Görevi |
|---|---|
| `queryKey` | Verinin önbellekteki **benzersiz kimliği** |
| `queryFn` | Veriyi getiren, **Promise döndüren** fonksiyon; hata durumunda **hata fırlatmalı**; `signal` ile iptal desteklenir |

> Konu 10.9'daki ~30 satırlık effect kodu 4 satıra indi; yarış durumu, iptal, önbellek, yeniden deneme hazır.

### `status`: Elimde veri var mı?

| `status` | Kısayol | Anlamı |
|---|---|---|
| `"pending"` | `isPending` | Henüz hiç veri yok |
| `"error"` | `isError` | Hata (`error` dolu) |
| `"success"` | `isSuccess` | Veri var (`data` dolu) |

### `fetchStatus`: Şu an istek atılıyor mu?

| `fetchStatus` | Kısayol | Anlamı |
|---|---|---|
| `"fetching"` | `isFetching` | İstek sürüyor |
| `"paused"` | `isPaused` | İnternet yok, bağlanınca devam edecek |
| `"idle"` | — | Bir şey yapılmıyor |

### En önemli kombinasyon: `isSuccess` + `isFetching`

"Veri var, gösteriyorum; arka planda yenisini çekiyorum."

```jsx
const { data, isPending, isFetching } = useQuery({ /* ... */ });

if (isPending) return <Iskelet />;                     // ilk yükleme: veri yok

return (
  <div className={isFetching ? "opacity-70" : ""}>     {/* arka plan yenilemesi */}
    {isFetching && <small>Güncelleniyor...</small>}
    <UrunListesi urunler={data} />
  </div>
);
```

### Diğer alanlar

| Alan | Anlamı |
|---|---|
| `refetch()` | Elle yeniden çek |
| `dataUpdatedAt` | Verinin alındığı zaman |
| `isLoading` | `isPending && isFetching` (ilk yüklemede gerçekten istek atılıyor mu) |
| `isPlaceholderData` | Gösterilen veri geçici/önceki veri mi |

---

## 6. Query Key'ler ve `queryOptions` ⭐

### İki görev

**1. Önbellek kimliği:** Aynı key = aynı veri. Üç bileşen aynı key'i kullanırsa **tek istek** atılır (*deduplication*).

**2. Bağımlılık dizisi:** `queryFn`'de kullanılan **her değişken** key'de olmalı; key değişince yeni sorgu başlar.

```jsx
// ❌ id key'de yok → hep ilk ürünün verisi
useQuery({ queryKey: ["urun"], queryFn: () => urunGetir(id) });

// ✅
useQuery({ queryKey: ["urunler", id], queryFn: () => urunGetir(id) });

// ✅ Birden çok değişken: nesne okunaklı
useQuery({
  queryKey: ["urunler", "liste", { q, sayfa, kategori }],
  queryFn: ({ signal }) => urunleriGetir({ q, sayfa, kategori }, signal),
});
```

- Key'deki nesnelerde anahtar sırası önemsizdir; key'ler **derinlemesine** karşılaştırılır (her render'da yeni nesne sorun değil).

> 💡 `@tanstack/eslint-plugin-query`, key'e eklenmemiş değişkenleri yakalar.

### Hiyerarşik key'ler (genelden özele)

```js
["urunler"]                                    // ürünlerle ilgili HER ŞEY
["urunler", "liste"]                           // tüm ürün listeleri
["urunler", "liste", { q: "telefon", sayfa: 1 }]
["urunler", "liste", { q: "telefon", sayfa: 2 }]
["urunler", "detay", 5]                        // 5 numaralı ürün
["urunler", "detay", 5, "yorumlar"]
```

### Key fabrikası ve `queryOptions`

Key'ler ve sorgu tanımları **tek yerde** toplanır:

```js
// src/api/urunSorgulari.js
import { queryOptions } from "@tanstack/react-query";
import { urunGetir, urunleriGetir } from "./urunler";

export const urunAnahtarlari = {
  hepsi: ["urunler"],
  listeler: () => [...urunAnahtarlari.hepsi, "liste"],
  liste: (filtre) => [...urunAnahtarlari.listeler(), filtre],
  detaylar: () => [...urunAnahtarlari.hepsi, "detay"],
  detay: (id) => [...urunAnahtarlari.detaylar(), id],
};

export const urunSorgulari = {
  liste: (filtre) =>
    queryOptions({
      queryKey: urunAnahtarlari.liste(filtre),
      queryFn: ({ signal }) => urunleriGetir(filtre, signal),
    }),
  detay: (id) =>
    queryOptions({
      queryKey: urunAnahtarlari.detay(id),
      queryFn: ({ signal }) => urunGetir(id, signal),
      staleTime: 5 * 60 * 1000,
    }),
};
```

```jsx
// Bileşenlerde
const { data } = useQuery(urunSorgulari.detay(id));
const { data } = useQuery({ ...urunSorgulari.liste({ q, sayfa }), placeholderData: keepPreviousData });
```

> `queryOptions` aynı tanımı `useQuery`, `prefetchQuery`, `ensureQueryData`, `getQueryData` gibi her yerde kullanmayı sağlar; TypeScript'te key ile veri tipini bağlar.

---

## 7. Önbellek Yaşam Döngüsü: `staleTime` ve `gcTime` ⭐

```
                 istek başarılı
   [ yok ] ───────────────────► [ TAZE (fresh) ]
                                      │
                                      │ staleTime doldu
                                      ▼
                                [ BAYAT (stale) ] ◄─── veri hâlâ gösterilir!
                                      │
                    son kullanan bileşen ekrandan kalktı
                                      ▼
                               [ PASİF (inactive) ]
                                      │
                                      │ gcTime doldu
                                      ▼
                                [ silindi ]
```

### `staleTime`: Veri ne kadar "taze"?

- **Varsayılan: `0`** → veri geldiği anda bayat sayılır.
- Taze veri için **asla** istek atılmaz; önbellekten anında verilir.
- Bayat veri **yine gösterilir**, ama belirli olaylarda arka planda yenilenir.

### `gcTime`: Kullanılmayan veri ne kadar saklanır?

- **Varsayılan: 5 dakika.**
- Hiçbir bileşen kullanmayınca sayaç başlar; dolunca silinir.
- Süre dolmadan dönülürse veri **anında** gösterilir.
- 📜 v4 ve öncesinde adı `cacheTime`.

### Bayat veri ne zaman otomatik yenilenir?

| Tetikleyici | Ayar | Varsayılan |
|---|---|---|
| Sorguyu kullanan **yeni bileşen** ekrana gelince | `refetchOnMount` | ✅ |
| **Sekmeye/pencereye geri dönünce** | `refetchOnWindowFocus` | ✅ |
| **İnternet bağlantısı** gelince | `refetchOnReconnect` | ✅ |
| Belirli aralıklarla | `refetchInterval` | kapalı |

- Başarısız istekler istemcide **3 kez** tekrar denenir (`retry`).

> Varsayılanlar **agresif güncellik** içindir: Sekme değiştirip dönünce Network'te yeni istekler görmek normaldir.

### `staleTime`'ı veriye göre ayarlamak

```jsx
useQuery({ ...sorgu, staleTime: 0 });                // borsa fiyatı
useQuery({ ...sorgu, staleTime: 60 * 1000 });        // ürün listesi: 1 dk
useQuery({ ...sorgu, staleTime: 10 * 60 * 1000 });   // kategoriler
useQuery({ ...sorgu, staleTime: Infinity });         // ülke listesi
```

### Global varsayılanlar

```js
const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 60 * 1000,
      retry: 1,
      refetchOnWindowFocus: true,
    },
  },
});
```

> 💡 **`staleTime`** = "Bu veri ne kadar süre değişmeyecek diye güvenebilirim?" · **`gcTime`** = "Kullanıcı dönerse anında göstermek için ne kadar saklayayım?" Genelde asıl ayarlanan `staleTime`'dır.

---

## 8. Koşullu, Bağımlı ve Paralel Sorgular

### Koşullu: `enabled`

```jsx
const { data } = useQuery({
  ...aramaSorgusu(q),
  enabled: q.length >= 2,   // en az 2 harf
});
```

> `enabled: false` iken `isPending` true'dur ama istek yoktur; ilk yükleme göstergesi için `isLoading` daha doğrudur.

### Bağımlı sorgular

```jsx
function KullaniciSiparisleri({ email }) {
  const { data: kullanici } = useQuery({
    queryKey: ["kullanici", email],
    queryFn: () => kullaniciBul(email),
  });

  const { data: siparisler } = useQuery({
    queryKey: ["siparisler", kullanici?.id],
    queryFn: () => siparisleriGetir(kullanici.id),
    enabled: !!kullanici?.id,   // kullanıcı gelmeden başlama
  });
}
```

> ⚠️ Bağımlı sorgular doğası gereği **şelaledir**; mümkünse API tek istekte ikisini döndürmeli.

### `skipToken`

```jsx
import { skipToken } from "@tanstack/react-query";

useQuery({
  queryKey: ["siparisler", kullanici?.id],
  queryFn: kullanici?.id ? () => siparisleriGetir(kullanici.id) : skipToken,
});
```

- `enabled` alternatifi; TypeScript'te daha güvenli.

### Paralel sorgular

```jsx
const urunler = useQuery(urunSorgulari.liste({ sayfa: 1 }));
const kategoriler = useQuery(kategoriSorgusu());   // aynı anda başlarlar
```

### Değişken sayıda sorgu: `useQueries`

```jsx
const sonuclar = useQueries({
  queries: secilenIdler.map((id) => urunSorgulari.detay(id)),
});
const hepsiYuklendi = sonuclar.every((s) => s.isSuccess);
```

---

## 9. Sayfalama ve Sonsuz Kaydırma

### Sayfalama: `keepPreviousData` ⭐

```jsx
import { keepPreviousData, useQuery } from "@tanstack/react-query";

function UrunSayfasi() {
  const [sayfa, setSayfa] = useState(1);

  const { data, isPending, isFetching, isPlaceholderData } = useQuery({
    ...urunSorgulari.liste({ sayfa }),
    placeholderData: keepPreviousData,   // yeni sayfa gelene kadar eskisini göster
  });

  if (isPending) return <Iskelet />;

  const toplamSayfa = Math.ceil(data.total / 12);

  return (
    <>
      <ul style={{ opacity: isPlaceholderData ? 0.5 : 1 }}>
        {data.products.map((u) => <li key={u.id}>{u.title}</li>)}
      </ul>

      <button onClick={() => setSayfa((s) => s - 1)} disabled={sayfa === 1}>Önceki</button>
      <span>{sayfa} / {toplamSayfa}</span>
      <button
        onClick={() => setSayfa((s) => s + 1)}
        disabled={isPlaceholderData || sayfa >= toplamSayfa}
      >
        Sonraki
      </button>
      {isFetching && <small>Yükleniyor...</small>}
    </>
  );
}
```

- `placeholderData: keepPreviousData` → ekran hiç boşalmaz.
- `isPlaceholderData` → gösterilen veri önceki sayfanın mı?
- Önceki sayfaya dönüş önbellekten **anında.**

> 💡 `sayfa` tercihen URL'de tutulur (`useSearchParams`, Konu 17.9).

### Sonsuz kaydırma: `useInfiniteQuery`

```jsx
import { useInfiniteQuery } from "@tanstack/react-query";

const LIMIT = 10;

function YorumAkisi() {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage, isPending } = useInfiniteQuery({
    queryKey: ["yorumlar"],
    queryFn: ({ pageParam, signal }) =>
      istek(`/comments?limit=${LIMIT}&skip=${pageParam}`, { signal }),
    initialPageParam: 0,
    getNextPageParam: (sonSayfa, tumSayfalar) => {
      const yuklenen = tumSayfalar.length * LIMIT;
      return yuklenen < sonSayfa.total ? yuklenen : undefined;   // undefined → bitti
    },
  });

  if (isPending) return <p>Yükleniyor...</p>;

  const yorumlar = data.pages.flatMap((sayfa) => sayfa.comments);

  return (
    <>
      <ul>
        {yorumlar.map((y) => <li key={y.id}>{y.body}</li>)}
      </ul>
      <button onClick={() => fetchNextPage()} disabled={!hasNextPage || isFetchingNextPage}>
        {isFetchingNextPage ? "Yükleniyor..." : hasNextPage ? "Daha fazla" : "Hepsi bu kadar"}
      </button>
    </>
  );
}
```

| Alan | Anlamı |
|---|---|
| `data.pages` | Yüklenen her sayfanın cevabı (dizi) |
| `initialPageParam` | İlk sayfanın parametresi |
| `getNextPageParam` | Sonraki sayfanın parametresi; `undefined` → sayfa kalmadı |
| `fetchNextPage`, `hasNextPage`, `isFetchingNextPage` | Sonraki sayfa kontrolü |

### Kaydırınca otomatik yükleme (IntersectionObserver)

```jsx
const sonRef = useRef(null);

useEffect(() => {
  const el = sonRef.current;
  if (!el || !hasNextPage) return;
  const gozlemci = new IntersectionObserver(([giris]) => {
    if (giris.isIntersecting && !isFetchingNextPage) fetchNextPage();
  });
  gozlemci.observe(el);
  return () => gozlemci.disconnect();
}, [hasNextPage, isFetchingNextPage, fetchNextPage]);

// liste sonunda:
<div ref={sonRef} aria-hidden="true" />
```

---

## 10. Veri Değiştirme: `useMutation` ⭐

```jsx
import { useMutation, useQueryClient } from "@tanstack/react-query";

function YorumFormu({ urunId }) {
  const queryClient = useQueryClient();
  const [metin, setMetin] = useState("");

  const yorumEkle = useMutation({
    mutationFn: (yeniYorum) =>
      istek("/comments/add", { method: "POST", body: JSON.stringify(yeniYorum) }),

    onSuccess: () => {
      queryClient.invalidateQueries({ queryKey: ["urunler", "detay", urunId, "yorumlar"] });
      setMetin("");
    },
  });

  function handleSubmit(e) {
    e.preventDefault();
    yorumEkle.mutate({ body: metin, postId: urunId, userId: 1 });
  }

  return (
    <form onSubmit={handleSubmit}>
      <textarea value={metin} onChange={(e) => setMetin(e.target.value)} />
      <button disabled={yorumEkle.isPending}>
        {yorumEkle.isPending ? "Gönderiliyor..." : "Gönder"}
      </button>
      {yorumEkle.isError && <p role="alert">Hata: {yorumEkle.error.message}</p>}
    </form>
  );
}
```

### `useQuery`'den farkları

- **Otomatik çalışmaz**; `mutate(degiskenler)` ile (genelde olay yöneticisinde) tetiklenir.
- Önbelleğe kaydedilmez, `queryKey`'i yoktur.
- Durumlar: `isIdle`, `isPending`, `isSuccess`, `isError`; alanlar: `data`, `error`, `variables`, `reset()`.
- Varsayılan olarak **tekrar denenmez.**

### `mutate` vs `mutateAsync`

```jsx
yorumEkle.mutate(veri);                                        // callback'lerle (önerilen)
yorumEkle.mutate(veri, { onSuccess: () => navigate("/") });   // çağrıya özel callback

const sonuc = await yorumEkle.mutateAsync(veri);              // Promise; try/catch gerekir
```

### Mutasyondan sonra önbelleği güncellemek

#### 1. Geçersiz kılma: `invalidateQueries` (en basit, en güvenli)

```js
queryClient.invalidateQueries({ queryKey: ["urunler"] });
```

- Eşleşen sorgular **bayat** işaretlenir; ekrandakiler **hemen** yeniden çekilir.
- **Önek eşleşmesi:** `["urunler"]` tüm alt key'leri kapsar.

```js
queryClient.invalidateQueries({ queryKey: urunAnahtarlari.listeler() });  // sadece listeler
queryClient.invalidateQueries({ queryKey: urunAnahtarlari.detay(5) });    // sadece 5. ürün
```

#### 2. Doğrudan yazma: `setQueryData`

```js
onSuccess: (guncelUrun) => {
  queryClient.setQueryData(urunAnahtarlari.detay(guncelUrun.id), guncelUrun);
  queryClient.invalidateQueries({ queryKey: urunAnahtarlari.listeler() });
},
```

> ⚠️ `setQueryData`'da **immutability** geçerlidir:
> ```js
> queryClient.setQueryData(key, (eski) => ({ ...eski, baslik: "Yeni" }));
> ```

> React Router action'larında tüm loader'lar otomatik yenilenir; TanStack Query'de **neyin** yenileneceğini `invalidateQueries` ile sen belirlersin.

---

## 11. İyimser Güncellemeler (Optimistic Updates)

### Yöntem 1: `variables` ile (basit, önerilen)

Önbelleğe dokunmadan, mutasyon sürerken gönderilen değeri göster:

```jsx
function GorevListesi() {
  const { data: gorevler } = useQuery(gorevSorgusu());
  const gorevEkle = useMutation({
    mutationFn: gorevOlustur,
    onSettled: () => queryClient.invalidateQueries({ queryKey: ["gorevler"] }),
  });

  return (
    <ul>
      {gorevler.map((g) => <li key={g.id}>{g.metin}</li>)}
      {gorevEkle.isPending && (
        <li className="opacity-50">{gorevEkle.variables.metin}</li>   {/* henüz sunucuda değil */}
      )}
      {gorevEkle.isError && (
        <li className="text-red-600">
          {gorevEkle.variables.metin}
          <button onClick={() => gorevEkle.mutate(gorevEkle.variables)}>Tekrar dene</button>
        </li>
      )}
    </ul>
  );
}
```

- Hata olursa geri alınacak bir şey yoktur; geçici satır kendiliğinden kaybolur.

### Yöntem 2: Önbelleği güncelle, hata olursa geri al

Değişiklik birçok yerde anında görünmeliyse:

```jsx
const favoriDegistir = useMutation({
  mutationFn: ({ id, favori }) => favoriGuncelle(id, favori),

  // 1. İstek gitmeden ÖNCE
  onMutate: async ({ id, favori }) => {
    await queryClient.cancelQueries({ queryKey: ["favoriler"] });   // yarışan yenilemeleri iptal
    const onceki = queryClient.getQueryData(["favoriler"]);          // yedek
    queryClient.setQueryData(["favoriler"], (eski = []) =>
      favori ? [...eski, id] : eski.filter((x) => x !== id)
    );
    return { onceki };                                                // onError'a iletilir
  },

  // 2. Hata olursa: yedeği geri yükle
  onError: (hata, degiskenler, sonuc) => {
    queryClient.setQueryData(["favoriler"], sonuc.onceki);
  },

  // 3. Her durumda: sunucuyla senkronize ol
  onSettled: () => {
    queryClient.invalidateQueries({ queryKey: ["favoriler"] });
  },
});
```

> **Kalıp:** iptal et + yedekle → iyimser yaz → hata olursa geri al → her durumda tazele.

> Hızlı ve nadiren başarısız işlemler (favori, beğeni, işaretleme) için kullanılır; ödeme gibi kritik işlemlerde sunucu cevabı beklenir.

---

## 12. Önceden Yükleme ve React Router ile Kullanım

### Üzerine gelince önceden yükle

```jsx
function UrunLinki({ urun }) {
  const queryClient = useQueryClient();

  function onceYukle() {
    queryClient.prefetchQuery(urunSorgulari.detay(urun.id));
  }

  return (
    <Link to={`/urunler/${urun.id}`} onMouseEnter={onceYukle} onFocus={onceYukle}>
      {urun.title}
    </Link>
  );
}
```

- Veri önbellekte ve tazeyse `prefetchQuery` hiçbir şey yapmaz → güvenle sık çağrılır.

### React Router loader'ları + TanStack Query ⭐

| | Görevi |
|---|---|
| **Loader** | Rota açılmadan veriyi başlatır (şelale yok) |
| **TanStack Query** | Önbellekte varsa tekrar istemez; arka planda tazeler; mutasyonlarla senkronize eder |

```jsx
// src/queryClient.js — router'dan da erişilebilsin diye ayrı modülde
import { QueryClient } from "@tanstack/react-query";
export const queryClient = new QueryClient({
  defaultOptions: { queries: { staleTime: 30 * 1000 } },
});
```

```jsx
// src/routes/UrunDetay.jsx
import { useSuspenseQuery } from "@tanstack/react-query";
import { useParams } from "react-router";
import { queryClient } from "../queryClient";
import { urunSorgulari } from "../api/urunSorgulari";

export async function loader({ params }) {
  // Önbellekte taze veri varsa anında döner; yoksa çeker ve önbelleğe koyar
  await queryClient.ensureQueryData(urunSorgulari.detay(params.urunId));
  return null;
}

export default function UrunDetay() {
  const { urunId } = useParams();
  const { data: urun } = useSuspenseQuery(urunSorgulari.detay(urunId));   // veri kesin hazır
  return <h1>{urun.title}</h1>;
}
```

```jsx
// main.jsx
<QueryClientProvider client={queryClient}>
  <RouterProvider router={router} />
</QueryClientProvider>
```

- Loader veriyi önbelleğe **koyar**; bileşen aynı `queryOptions` ile **okur.**
- `useSuspenseQuery` sayesinde `data` asla `undefined` olmaz.
- Action'larda veya `useMutation`'da `invalidateQueries` ile neyin tazeleneceği seçilir.

---

## 13. Suspense ile Veri Çekme

Yükleme/hata kontrolleri bileşenin **dışına**, deklaratif olarak taşınır:

```jsx
import { Suspense } from "react";
import { useSuspenseQuery } from "@tanstack/react-query";
import { ErrorBoundary } from "react-error-boundary";   // npm install react-error-boundary

function UrunDetay({ id }) {
  const { data } = useSuspenseQuery(urunSorgulari.detay(id));
  // isPending yok, isError yok: data HER ZAMAN hazır
  return <h1>{data.title}</h1>;
}

function UrunSayfasi({ id }) {
  return (
    <ErrorBoundary fallback={<p>Ürün yüklenemedi.</p>}>
      <Suspense fallback={<UrunIskeleti />}>
        <UrunDetay id={id} />
      </Suspense>
    </ErrorBoundary>
  );
}
```

### Nasıl çalışır?

1. Veri hazır değilse `useSuspenseQuery` render'ı **askıya alır.**
2. React en yakın **`<Suspense>`**'in `fallback`'ini gösterir.
3. Veri gelince bileşen render edilir.
4. Hata olursa en yakın **Error Boundary** yakalar.

> Yükleme ve hata durumları bileşenin içinde değil, **onu kullanan yerde** belirlenen bir yerleşim kararı olur.

### Hata sonrası "Tekrar dene"

```jsx
import { QueryErrorResetBoundary } from "@tanstack/react-query";

<QueryErrorResetBoundary>
  {({ reset }) => (
    <ErrorBoundary
      onReset={reset}
      fallbackRender={({ resetErrorBoundary }) => (
        <div>
          <p>Bir hata oluştu.</p>
          <button onClick={resetErrorBoundary}>Tekrar dene</button>
        </div>
      )}
    >
      <Suspense fallback={<Iskelet />}>
        <UrunDetay id={id} />
      </Suspense>
    </ErrorBoundary>
  )}
</QueryErrorResetBoundary>
```

> 📌 React 19'un **`use`** API'si bir Promise'i de okuyup Suspense ile çalışır (`const urun = use(urunPromise)`); ama önbellek, tazeleme, mutasyon yönetimi sunmaz. Genellikle framework'lerin sunucu verisini istemciye aktarmasında kullanılır *(Konu 20, 23)*.

---

## 14. Faydalı Diğer Özellikler

### `select`: Veriyi dönüştürmek

```jsx
// Sadece ürün adları
const { data: adlar } = useQuery({
  ...urunSorgulari.liste({ sayfa: 1 }),
  select: (veri) => veri.products.map((u) => u.title),
});

// Sadece sepetteki ürün sayısı
const { data: adet } = useQuery({
  ...sepetSorgusu(),
  select: (sepet) => sepet.urunler.length,
});
```

- Bileşen yalnızca `select`'in **sonucu** değişince render olur (context performans sorununun çözümü).

### Global hata yönetimi

```js
import { QueryCache, QueryClient } from "@tanstack/react-query";

const queryClient = new QueryClient({
  queryCache: new QueryCache({
    onError: (hata, sorgu) => {
      // Sadece arka plan yenilemelerinde (ekranda veri varken) bildirim
      if (sorgu.state.data !== undefined) {
        bildirimGoster(`Veriler güncellenemedi: ${hata.message}`);
      }
    },
  }),
});
```

### Yeniden deneme ayarı

```js
retry: (denemeSayisi, hata) => {
  if (hata.durum === 404) return false;   // bulunamadıysa tekrar deneme
  return denemeSayisi < 3;
},
```

### Listedeki veriden ilk veri (`initialData`)

```js
useQuery({
  ...urunSorgulari.detay(id),
  initialData: () =>
    queryClient
      .getQueryData(urunAnahtarlari.liste({ sayfa: 1 }))
      ?.products.find((u) => u.id === id),
  initialDataUpdatedAt: () =>
    queryClient.getQueryState(urunAnahtarlari.liste({ sayfa: 1 }))?.dataUpdatedAt,
});
```

---

## 15. Alternatifler ve Karar Rehberi

| Araç | Not |
|---|---|
| **TanStack Query** | En yaygın; REST, GraphQL, her Promise ile çalışır; Vue, Solid, Svelte, Angular sürümleri de var |
| **SWR** (Vercel) | Daha küçük, basit API; aynı "stale-while-revalidate" fikri |
| **RTK Query** | Redux Toolkit projelerinde, Redux store'una entegre |
| **Apollo Client**, **urql**, **Relay** | GraphQL'e özel, normalize önbellek |
| **React Router loader'ları** | Rota düzeyinde veri; önbellek yok; basit uygulamalar için yeterli |
| **Server Components** | Veri sunucuda çekilir *(Konu 23)* |

### Karar rehberi

| Durum | Öneri |
|---|---|
| Küçük uygulama, az sayfa, veri nadiren değişiyor | React Router loader'ları |
| Veri birçok sayfada paylaşılıyor, sık güncelleniyor, mutasyonlar, sayfalama | **TanStack Query** |
| Hem hızlı rota açılışı hem önbellek | **İkisi birlikte** |

> TanStack Query ile **sunucu verilerinin** çoğu `useState`/`useReducer`/context'ten çıkar; context'te sadece gerçekten istemciye ait şeyler (tema, oturum, UI tercihleri) kalır.

---

## 16. Her Şey Bir Arada: Ürün Kataloğu

Gerçek API (DummyJSON) ile liste, arama, sayfalama, detay, önceden yükleme, sonsuz yorum akışı; sahte gecikmeli bir "sunucu"da saklanan favoriler ve iyimser mutasyonlar.

### API istemcisi

```js
// ============ src/api/istemci.js ============
export class ApiHatasi extends Error {
  constructor(mesaj, durum) {
    super(mesaj);
    this.durum = durum;
  }
}

export async function istek(url, { signal, ...secenekler } = {}) {
  const res = await fetch(url, {
    headers: { "Content-Type": "application/json" },
    signal,
    ...secenekler,
  });
  if (!res.ok) throw new ApiHatasi(`İstek başarısız (${res.status})`, res.status);
  return res.json();
}
```

### Sorgu tanımları

```js
// ============ src/api/katalog.js ============
import { infiniteQueryOptions, queryOptions } from "@tanstack/react-query";
import { istek } from "./istemci";

const API = "https://dummyjson.com";
export const SAYFA_BOYUTU = 12;

// --- Ham API fonksiyonları ---
function urunleriGetir({ q, sayfa }, signal) {
  const skip = (sayfa - 1) * SAYFA_BOYUTU;
  const alanlar = "id,title,price,thumbnail,rating,category";
  return istek(
    `${API}/products/search?q=${encodeURIComponent(q)}&limit=${SAYFA_BOYUTU}&skip=${skip}&select=${alanlar}`,
    { signal }
  );
}

function urunGetir(id, signal) {
  return istek(`${API}/products/${id}`, { signal });
}

function yorumlariGetir(skip, signal) {
  return istek(`${API}/comments?limit=5&skip=${skip}`, { signal });
}

// --- Sahte "sunucu" favorileri (gecikmeli, ara sıra hata veriyor) ---
const bekle = (ms) => new Promise((r) => setTimeout(r, ms));

async function favorileriGetir() {
  await bekle(300);
  return JSON.parse(localStorage.getItem("favoriler") ?? "[]");
}

export async function favoriGuncelle({ id, favori }) {
  await bekle(600);
  if (Math.random() < 0.15) throw new Error("Sunucu hatası (simülasyon)");   // %15 hata
  const mevcut = JSON.parse(localStorage.getItem("favoriler") ?? "[]");
  const yeni = favori ? [...new Set([...mevcut, id])] : mevcut.filter((x) => x !== id);
  localStorage.setItem("favoriler", JSON.stringify(yeni));
  return yeni;
}

// --- Key fabrikası ---
export const anahtarlar = {
  urunler: ["urunler"],
  liste: (filtre) => ["urunler", "liste", filtre],
  detay: (id) => ["urunler", "detay", id],
  yorumlar: ["yorumlar"],
  favoriler: ["favoriler"],
};

// --- Sorgu tanımları ---
export const sorgular = {
  liste: (filtre) =>
    queryOptions({
      queryKey: anahtarlar.liste(filtre),
      queryFn: ({ signal }) => urunleriGetir(filtre, signal),
      staleTime: 60 * 1000,
    }),

  detay: (id) =>
    queryOptions({
      queryKey: anahtarlar.detay(id),
      queryFn: ({ signal }) => urunGetir(id, signal),
      staleTime: 5 * 60 * 1000,
    }),

  yorumlar: () =>
    infiniteQueryOptions({
      queryKey: anahtarlar.yorumlar,
      queryFn: ({ pageParam, signal }) => yorumlariGetir(pageParam, signal),
      initialPageParam: 0,
      getNextPageParam: (son, hepsi) => {
        const yuklenen = hepsi.reduce((t, s) => t + s.comments.length, 0);
        return yuklenen < son.total ? yuklenen : undefined;
      },
    }),

  favoriler: () =>
    queryOptions({
      queryKey: anahtarlar.favoriler,
      queryFn: favorileriGetir,
      staleTime: Infinity,   // sadece biz değiştiriyoruz; mutasyondan sonra tazeliyoruz
    }),
};
```

### İyimser favori Hook'u

```jsx
// ============ src/katalog/useFavori.js ============
import { useMutation, useQuery, useQueryClient } from "@tanstack/react-query";
import { anahtarlar, favoriGuncelle, sorgular } from "../api/katalog";

export function useFavoriler() {
  return useQuery(sorgular.favoriler());
}

export function useFavoriDegistir() {
  const queryClient = useQueryClient();

  return useMutation({
    mutationFn: favoriGuncelle,
    onMutate: async ({ id, favori }) => {
      await queryClient.cancelQueries({ queryKey: anahtarlar.favoriler });
      const onceki = queryClient.getQueryData(anahtarlar.favoriler);
      queryClient.setQueryData(anahtarlar.favoriler, (eski = []) =>
        favori ? [...eski, id] : eski.filter((x) => x !== id)
      );
      return { onceki };
    },
    onError: (_hata, _degiskenler, sonuc) => {
      queryClient.setQueryData(anahtarlar.favoriler, sonuc?.onceki);
    },
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: anahtarlar.favoriler });
    },
  });
}
```

### Katalog bileşenleri

```jsx
// ============ src/katalog/Katalog.jsx ============
import { Suspense, useEffect, useRef, useState } from "react";
import {
  keepPreviousData, useInfiniteQuery, useQuery, useQueryClient, useSuspenseQuery,
} from "@tanstack/react-query";
import { SAYFA_BOYUTU, sorgular } from "../api/katalog";
import { useFavoriDegistir, useFavoriler } from "./useFavori";
import { useDebounce } from "../hooks/useDebounce";   // Konu 15

// --- Favori butonu (iyimser) ---
function FavoriButonu({ id }) {
  const { data: favoriler = [] } = useFavoriler();
  const degistir = useFavoriDegistir();
  const favori = favoriler.includes(id);

  return (
    <button
      type="button"
      onClick={() => degistir.mutate({ id, favori: !favori })}
      aria-pressed={favori}
      aria-label={favori ? "Favorilerden çıkar" : "Favorilere ekle"}
      className="text-xl text-rose-500"
    >
      {favori ? "♥" : "♡"}
    </button>
  );
}

// --- Ürün kartı: üzerine gelince detayı önceden yükle ---
function UrunKarti({ urun, onSec }) {
  const queryClient = useQueryClient();
  const onceYukle = () => queryClient.prefetchQuery(sorgular.detay(urun.id));

  return (
    <li className="flex flex-col gap-2 rounded-lg border p-3">
      <button
        type="button"
        onClick={() => onSec(urun.id)}
        onMouseEnter={onceYukle}
        onFocus={onceYukle}
        className="flex flex-col gap-2 text-left"
      >
        <img
          src={urun.thumbnail}
          alt=""
          className="aspect-square w-full rounded object-cover"
          loading="lazy"
        />
        <span className="line-clamp-1 font-medium">{urun.title}</span>
      </button>
      <div className="flex items-center justify-between">
        <span className="font-bold tabular-nums">{urun.price} $</span>
        <FavoriButonu id={urun.id} />
      </div>
    </li>
  );
}

// --- Liste: arama + sayfalama + keepPreviousData ---
function UrunListesi({ onSec }) {
  const [arama, setArama] = useState("");
  const [sayfa, setSayfa] = useState(1);
  const q = useDebounce(arama.trim(), 400);

  const { data, isPending, isError, error, isFetching, isPlaceholderData, refetch } = useQuery({
    ...sorgular.liste({ q, sayfa }),
    placeholderData: keepPreviousData,
  });

  const toplamSayfa = data ? Math.max(1, Math.ceil(data.total / SAYFA_BOYUTU)) : 1;

  return (
    <section>
      <div className="mb-3 flex items-center gap-3">
        <input
          type="search"
          value={arama}
          onChange={(e) => {
            setArama(e.target.value);
            setSayfa(1);   // yeni aramada ilk sayfaya dön
          }}
          placeholder="Ürün ara..."
          className="w-full rounded border px-3 py-2"
        />
        {isFetching && <span className="text-sm text-gray-500">⏳</span>}
      </div>

      {isPending ? (
        <p>Yükleniyor...</p>
      ) : isError ? (
        <p role="alert">
          {error.message} <button onClick={() => refetch()}>Tekrar dene</button>
        </p>
      ) : data.products.length === 0 ? (
        <p>"{q}" için sonuç yok.</p>
      ) : (
        <>
          <ul
            className="grid grid-cols-2 gap-3 sm:grid-cols-3 lg:grid-cols-4"
            style={{ opacity: isPlaceholderData ? 0.5 : 1, transition: "opacity .2s" }}
          >
            {data.products.map((u) => <UrunKarti key={u.id} urun={u} onSec={onSec} />)}
          </ul>

          <nav className="mt-4 flex items-center justify-center gap-3">
            <button onClick={() => setSayfa((s) => s - 1)} disabled={sayfa === 1}>
              ← Önceki
            </button>
            <span className="tabular-nums">{sayfa} / {toplamSayfa}</span>
            <button
              onClick={() => setSayfa((s) => s + 1)}
              disabled={isPlaceholderData || sayfa >= toplamSayfa}
            >
              Sonraki →
            </button>
          </nav>
        </>
      )}
    </section>
  );
}

// --- Detay: Suspense ile ---
function UrunDetay({ id }) {
  const { data: urun } = useSuspenseQuery(sorgular.detay(id));
  return (
    <article className="space-y-2">
      <title>{`${urun.title} · Katalog`}</title>
      <img src={urun.thumbnail} alt="" className="w-full rounded" />
      <div className="flex items-start justify-between gap-2">
        <h2 className="text-xl font-bold">{urun.title}</h2>
        <FavoriButonu id={urun.id} />
      </div>
      <p className="text-sm text-gray-600">{urun.description}</p>
      <p className="font-bold">{urun.price} $ · ⭐ {urun.rating}</p>
    </article>
  );
}

// --- Sonsuz yorum akışı ---
function YorumAkisi() {
  const { data, fetchNextPage, hasNextPage, isFetchingNextPage, isPending } =
    useInfiniteQuery(sorgular.yorumlar());
  const sonRef = useRef(null);

  useEffect(() => {
    const el = sonRef.current;
    if (!el || !hasNextPage) return;
    const gozlemci = new IntersectionObserver(([giris]) => {
      if (giris.isIntersecting && !isFetchingNextPage) fetchNextPage();
    });
    gozlemci.observe(el);
    return () => gozlemci.disconnect();
  }, [hasNextPage, isFetchingNextPage, fetchNextPage]);

  if (isPending) return <p>Yorumlar yükleniyor...</p>;
  const yorumlar = data.pages.flatMap((s) => s.comments);

  return (
    <section>
      <h3 className="mb-2 font-semibold">Son yorumlar</h3>
      <ul className="max-h-64 space-y-2 overflow-y-auto">
        {yorumlar.map((y) => (
          <li key={y.id} className="rounded bg-gray-50 p-2 text-sm">
            <strong>{y.user.username}:</strong> {y.body}
          </li>
        ))}
        <li ref={sonRef} aria-hidden="true" />
      </ul>
      <p className="mt-1 text-xs text-gray-500">
        {isFetchingNextPage ? "Yükleniyor..." : hasNextPage ? "Aşağı kaydır" : "Hepsi bu kadar"}
      </p>
    </section>
  );
}

// --- Favori sayacı: select ile sadece sayıya abone ---
function FavoriSayaci() {
  const { data: adet = 0 } = useQuery({ ...sorgular.favoriler(), select: (f) => f.length });
  return <span>♥ {adet}</span>;
}

// --- Ana sayfa ---
export default function Katalog() {
  const [seciliId, setSeciliId] = useState(null);

  return (
    <div className="mx-auto max-w-6xl p-4">
      <header className="mb-4 flex items-center justify-between">
        <h1 className="text-2xl font-bold">Ürün Kataloğu</h1>
        <FavoriSayaci />
      </header>

      <div className="grid gap-6 lg:grid-cols-[1fr_320px]">
        <UrunListesi onSec={setSeciliId} />

        <aside className="space-y-6">
          {seciliId ? (
            <Suspense fallback={<div className="h-80 animate-pulse rounded bg-gray-100" />}>
              <UrunDetay id={seciliId} />
            </Suspense>
          ) : (
            <p className="text-gray-500">Detay için bir ürün seç.</p>
          )}
          <YorumAkisi />
        </aside>
      </div>
    </div>
  );
}
```

### Giriş noktası

```jsx
// ============ src/main.jsx ============
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { QueryClient, QueryClientProvider, QueryCache } from "@tanstack/react-query";
import { ReactQueryDevtools } from "@tanstack/react-query-devtools";
import Katalog from "./katalog/Katalog";
import "./index.css";

const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 30 * 1000,
      retry: (sayi, hata) => hata?.durum !== 404 && sayi < 2,
    },
  },
  queryCache: new QueryCache({
    onError: (hata, sorgu) => {
      if (sorgu.state.data !== undefined) {
        console.warn("Arka plan yenilemesi başarısız:", hata.message);
      }
    },
  }),
});

createRoot(document.getElementById("root")).render(
  <StrictMode>
    <QueryClientProvider client={queryClient}>
      <Katalog />
      <ReactQueryDevtools initialIsOpen={false} />
    </QueryClientProvider>
  </StrictMode>
);
```

### Dene ve Devtools'ta izle

| Deneme | Gözlenen |
|---|---|
| "Sonraki"ye bas | Liste soluklaşır ama kaybolmaz (`keepPreviousData`); geri dönünce anında gelir |
| Arama kutusuna yaz | Her harfte değil, durunca istek (`useDebounce`); aynı arama tekrar → istek yok |
| Ürün kartının üzerine gel | Devtools'ta detay sorgusu belirir; tıklayınca detay anında açılır |
| Önceden yüklenmemiş ürüne hızlıca tıkla | Kenarda iskelet görünür (Suspense) |
| Kalbe tıkla | Kart, detay ve sayaç **aynı anda** güncellenir (aynı `["favoriler"]` sorgusu) |
| Arada simüle hata (%15) | Kalp kısa süre dolu kalır, sonra geri alınır (`onError`) |
| Yorumlarda aşağı kaydır | Yeni yorumlar otomatik eklenir (`useInfiniteQuery` + IntersectionObserver) |
| Başka sekmeye geç, 30 sn+ bekle, dön | Bayat sorgular arka planda tazelenir |
| Favori listesi değişip sayı aynı kalırsa | `FavoriSayaci` render olmaz (`select`) |

> Bu uygulamada **effect ile veri çekme yok**, "yükleniyor" state'i elle tutulmuyor; tüm sunucu verisi, önbelleği ve senkronizasyonu TanStack Query'de.

---

## 17. Özet

- **İstemci state'i** (tema, modal) ile **sunucu state'i** (ürünler, siparişler) farklıdır; sunucu state'i uzakta yaşar, elindeki bir kopyadır.
- **TanStack Query**, sunucu verisini **merkezi bir önbellekte** tutar: paylaşım, tekrar isteklerin birleştirilmesi, bayatlık takibi, arka planda yenileme, yeniden deneme, mutasyon senkronizasyonu.
- `fetch` çağrıları bir **API katmanında** toplanır; `res.ok` kontrolü ile hatalar fırlatılır.
- Kurulum: `QueryClient` (bileşen dışında) + `QueryClientProvider` + Devtools.
- **`useQuery({ queryKey, queryFn })`**; `queryFn` Promise döndürür, hata fırlatır, `signal` alır.
- **`status`** (pending/error/success) veri var mı, **`fetchStatus`** (fetching/paused/idle) istek sürüyor mu; `isSuccess && isFetching` = veri gösterilirken arka planda yenileme.
- **`queryKey`** = önbellekteki verinin adı; `queryFn`'deki her değişken key'de olmalı; key'ler **genelden özele** yazılır.
- **`queryOptions`** ve key fabrikalarıyla sorgu tanımları tek yerde toplanır.
- **`staleTime`** (varsayılan 0): veri ne kadar taze; **`gcTime`** (varsayılan 5 dk): kullanılmayan veri ne kadar saklanır. Bayat veri mount, pencere odağı ve yeniden bağlanmada otomatik yenilenir.
- **`enabled`** / **`skipToken`** ile koşullu ve bağımlı sorgular; **`useQueries`** ile değişken sayıda paralel sorgu.
- **`placeholderData: keepPreviousData`** sayfa geçişlerinde ekranı boşaltmaz; **`useInfiniteQuery`** sonsuz kaydırma sağlar.
- **`useMutation`** yazma işlemleri içindir; sonrasında **`invalidateQueries`** (önek eşleşmeli) veya **`setQueryData`** ile önbellek güncellenir.
- **İyimser güncellemeler:** basitçe `variables` ile, ya da `onMutate` (iptal + yedek + yaz) → `onError` (geri al) → `onSettled` (tazele).
- **`prefetchQuery`** ile üzerine gelince önceden yükleme; React Router loader'larında **`ensureQueryData`** + bileşende **`useSuspenseQuery`.**
- **Suspense** + Error Boundary ile yükleme/hata durumları bileşen dışına taşınır; `QueryErrorResetBoundary` ile "tekrar dene."
- **`select`** ile veri dönüştürülür ve gereksiz render'lar önlenir; `QueryCache.onError` ile global hata yönetimi.
- Alternatifler: SWR, RTK Query, Apollo/urql (GraphQL), React Router loader'ları, Server Components.

---

⬅️ Önceki: [17 · React Router](./17-react-router.md) · ➡️ Sonraki: [19 · Performans](./19-performans.md)
