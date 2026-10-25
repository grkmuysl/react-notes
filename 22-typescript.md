# 22 · TypeScript ile React

> **Konu özeti:** TypeScript'in ne olduğu ve React'e kattıkları, kurulum ve `tsconfig`, Vite'ın tip kontrolü yapmaması, React için yeterli TypeScript temelleri, props / `children` / `ComponentProps` / ayırt edici birleşim props, Hook'ları tiplemek (`useState`, `useReducer` + kapsamlılık kontrolü, `useRef`, context, custom Hook'lar), olaylar ve form verisi, Zod ile dış veriyi doğrulamak, TanStack Query ve React Router tipleri, generic bileşenler, yardımcı tipler, yaygın hatalar ve kademeli geçiş.

---

## İçindekiler

1. [Sezgisel Bakış](#1-sezgisel-bakış)
2. [Kurulum ve Yapılandırma](#2-kurulum-ve-yapılandırma)
3. [React İçin Yeterli TypeScript Temelleri](#3-react-için-yeterli-typescript-temelleri)
4. [Props'u Tiplemek](#4-propsu-tiplemek-)
5. [Hook'ları Tiplemek](#5-hookları-tiplemek-)
6. [Olayları Tiplemek](#6-olayları-tiplemek)
7. [Dış Veriyi Tiplemek: API'ler ve Zod](#7-dış-veriyi-tiplemek-apiler-ve-zod-)
8. [Generic Bileşenler](#8-generic-bileşenler)
9. [Yardımcı Tipler](#9-faydalı-yardımcı-tipler-utility-types)
10. [Yaygın Hatalar ve Kaçış Kapıları](#10-yaygın-hatalar-ve-kaçış-kapıları)
11. [JavaScript Projesini TypeScript'e Geçirmek](#11-mevcut-bir-javascript-projesini-typescripte-geçirmek)
12. [Her Şey Bir Arada: Tipli Ürün Yönetimi](#12-her-şey-bir-arada-tipli-ürün-yönetimi)
13. [Özet](#13-özet)

---

## 1. Sezgisel Bakış

> **TypeScript = JavaScript + tipler.** Değerlerin türü belirtilir; TypeScript kodu **çalıştırmadan**, yazarken kontrol eder. Derlemede tipler silinir, geriye düz JavaScript kalır.

### Örnek: JavaScript'te gizli kalan hatalar

```jsx
function UrunKarti({ urun, onSepeteEkle }) {
  return (
    <div>
      <h3>{urun.baslk}</h3>                  {/* yazım hatası */}
      <p>{urun.fiyat.toFixed(2)} ₺</p>       {/* fiyat bazen string geliyorsa? */}
      <button onClick={onSepeteEkle(urun)}>  {/* fonksiyonu çağırdık, vermedik */}
        Sepete ekle
      </button>
    </div>
  );
}
```

JavaScript'te bu hatalar **ancak tarayıcıda** ortaya çıkar. TypeScript ile **editörde**, kaydetmeden önce:

```tsx
type Urun = { id: number; baslik: string; fiyat: number };

type UrunKartiProps = {
  urun: Urun;
  onSepeteEkle: (urun: Urun) => void;
};

function UrunKarti({ urun, onSepeteEkle }: UrunKartiProps) {
  return (
    <div>
      <h3>{urun.baslk}</h3>
      {/*       ~~~~~ Property 'baslk' does not exist on type 'Urun'. Did you mean 'baslik'? */}
      <button onClick={onSepeteEkle(urun)}>
        {/*           ~~~~~~~~~~~~~~~~~~ Type 'void' is not assignable to type 'MouseEventHandler' */}
        Sepete ekle
      </button>
    </div>
  );
}
```

### Üç büyük fayda

1. **Hatalar erken yakalanır:** yanlış/eksik prop, `undefined` erişimi, yanlış tipte argüman.
2. **Editör seni tanır:** otomatik tamamlama, props listesi, güvenli yeniden adlandırma.
3. **Kod kendini belgeler:** `type Props` bileşenin kullanım kılavuzudur; PropTypes'ın yerini alır.

> **Benzetme:** JavaScript yazım denetimi olmayan bir kelime işlemci; TypeScript yazım denetimini açar.

---

## 2. Kurulum ve Yapılandırma

### Yeni proje

```bash
npm create vite@latest my-app -- --template react-ts
```

- `.tsx` (JSX içeren) ve `.ts` (JSX içermeyen) dosyalar; kökte `tsconfig.json`.

### TypeScript sürümü

- **TypeScript 7.0**: Derleyiciyi **Go diline** taşıyan ilk kararlı sürüm; tam derlemelerde genellikle **8–12 kat** hızlı.
- **TypeScript 6.0**: JavaScript tabanlı son sürüm; geçiş için **köprü.**
- 7.0'da `strict` ve `esnext` **varsayılan**; 6.0'ın kullanımdan kaldırdıkları **sert hata.**
- 7.0'da henüz kararlı **programatik API yok** (7.1'de bekleniyor) → typescript-eslint gibi araçlar şimdilik kullanamıyor; birçok proje bir süre 6.x ve 7.x'i yan yana kullanacak.

> Tip **sözdizimi** her iki sürümde aynıdır; fark derleyici hızı ve bazı ayarlardadır.

### `tsconfig.json`'da önemli ayarlar

```jsonc
{
  "compilerOptions": {
    "strict": true,                    // TÜM sıkı kontrolleri aç (mutlaka)
    "jsx": "react-jsx",                // yeni JSX dönüşümü
    "noUncheckedIndexedAccess": true,  // dizi[i] → T | undefined (önerilir)
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "verbatimModuleSyntax": true,      // tip importlarında "import type" zorunlu
    "skipLibCheck": true
  }
}
```

> `strict: true` kapalıysa TypeScript'in faydasının yarısı kaybolur.

### ⚠️ Vite tip kontrolü YAPMAZ

- Vite derlerken **sadece tipleri siler**; tip hatalı kod `npm run dev` ile çalışmaya devam eder.
- Tip kontrolü: **editörde**, **build'de** (`tsc -b`), **CI'da** (`tsc --noEmit`).

```json
{
  "scripts": {
    "dev": "vite",
    "build": "tsc -b && vite build",
    "typecheck": "tsc --noEmit"
  }
}
```

---

## 3. React İçin Yeterli TypeScript Temelleri

### Temel tipler ve çıkarım

```ts
let ad: string = "Ayşe";
let yas: number = 25;
let aktif: boolean = true;
let etiketler: string[] = ["react", "ts"];
let bos: null = null;

// Çoğu zaman tip yazmaya GEREK YOK
let sehir = "İstanbul";   // string çıkarıldı
sehir = 34;               // ❌ Type 'number' is not assignable to type 'string'
```

> **Kural:** Çıkarılabilen yerde tip yazma. Tipleri **sınırlarda** yaz: parametreler, props, API cevapları.

### Nesne tipleri: `type` ve `interface`

```ts
type Kullanici = {
  id: number;
  ad: string;
  email: string;
  telefon?: string;              // opsiyonel
  readonly kayitTarihi: string;  // değiştirilemez
};

interface Kullanici2 {
  id: number;
  ad: string;
}
```

> React'te ikisi neredeyse aynı işi görür; bu notlarda **`type`** kullanılır. Tutarlılık önemlidir.

### Birleşim (union) ve literal tipler ⭐

```ts
type Durum = "bosta" | "yukleniyor" | "hata" | "basarili";
let durum: Durum = "yukleniyor";
durum = "tamam";   // ❌

type Id = string | number;
type Belki<T> = T | null;
```

> "Tek bir durum değişkeni" fikri tip düzeyinde garanti altına alınır.

### Daraltma (narrowing)

```ts
function goster(deger: string | number | null) {
  if (deger === null) return "Yok";          // null
  if (typeof deger === "number") {
    return deger.toFixed(2);                 // number
  }
  return deger.toUpperCase();                // string
}
```

### Fonksiyon tipleri

```ts
function topla(a: number, b: number): number {
  return a + b;
}

const selamla = (ad: string, unvan?: string) => `Merhaba ${unvan ?? ""} ${ad}`;

type Karsilastirici = (a: Urun, b: Urun) => number;
```

### Generic'ler

```ts
function ilk<T>(dizi: T[]): T | undefined {
  return dizi[0];
}

const sayi = ilk([1, 2, 3]);     // number | undefined
const isim = ilk(["a", "b"]);    // string | undefined
```

### `any` ve `unknown`

```ts
let a: any = cevap;       // ❌ TypeScript'i KAPATIR
let b: unknown = cevap;   // ✅ kullanmadan önce kontrol ZORUNLU
if (typeof b === "string") b.toUpperCase();
```

### `as const` ve `satisfies`

```ts
const ROLLER = ["admin", "editor", "uye"] as const;
type Rol = (typeof ROLLER)[number];                   // "admin" | "editor" | "uye"

const RENKLER = {
  birincil: "#3b82f6",
  tehlike: "#dc2626",
} satisfies Record<string, string>;   // şekli kontrol eder, anahtarları korur
type RenkAdi = keyof typeof RENKLER;  // "birincil" | "tehlike"
```

> Değerden tip **türetmek**, değeri ve tipi tek yerde tutar.

---

## 4. Props'u Tiplemek ⭐

```tsx
type ProfilKartiProps = {
  ad: string;
  unvan: string;
  avatar?: string;
  pro?: boolean;
  yetenekler?: string[];
  onMesaj: (kime: string) => void;
};

function ProfilKarti({
  ad,
  unvan,
  avatar = "/varsayilan.png",   // içeride string
  pro = false,
  yetenekler = [],
  onMesaj,
}: ProfilKartiProps) {
  return (/* ... */);
}
```

```tsx
<ProfilKarti ad="Ayşe" unvan="Geliştirici" onMesaj={(k) => console.log(k)} />   // ✅
<ProfilKarti ad="Ayşe" />                       // ❌ 'unvan' ve 'onMesaj' eksik
<ProfilKarti ad="Ayşe" unvan="..." onMesaj={...} yas={25} />   // ❌ 'yas' yok
```

- `onMesaj={(k) => ...}`: `k`'nın tipi props'tan çıkarılır (bağlamsal çıkarım).

> 💡 **`React.FC` kullanılmaz.** Fonksiyonun **parametresi** tiplenir; daha sade ve generic bileşenlerle uyumludur.

### `children` tipi

```tsx
import type { ReactNode } from "react";

type KartProps = {
  baslik: string;
  children: ReactNode;
};

function Kart({ baslik, children }: KartProps) {
  return (
    <section>
      <h2>{baslik}</h2>
      {children}
    </section>
  );
}
```

| Tip | Ne kabul eder |
|---|---|
| `ReactNode` | Render edilebilen her şey (`children` için en yaygın) |
| `ReactElement` / `JSX.Element` | Tek bir JSX elementi |
| `string` | Sadece metin |
| `(deger: T) => ReactNode` | Render prop |

> Sadece tip olarak kullanılanlar **`import type`** ile alınır (`verbatimModuleSyntax`).

### HTML elementini saran bileşenler: `ComponentProps` ⭐

```tsx
import type { ComponentProps } from "react";

type ButonProps = ComponentProps<"button"> & {
  varyant?: "birincil" | "ikincil" | "tehlike";
  boyut?: "kucuk" | "orta" | "buyuk";
};

function Buton({ varyant = "birincil", boyut = "orta", className, ...rest }: ButonProps) {
  return <button className={cn(stiller({ varyant, boyut }), className)} {...rest} />;
}

<Buton varyant="tehlike" onClick={handleSil} disabled={yukleniyor} type="submit">Sil</Buton>   // ✅
<Buton varyant="kirmizi">Sil</Buton>   // ❌
```

- `ComponentProps<"button">`: `<button>`'ın tüm props'u (React 19'da `ref` dahil).
- `&` (kesişim): iki tipi birleştirir.
- Bileşen props'u: `ComponentProps<typeof ProfilKarti>`.
- Nitelik çıkarmak: `Omit<ComponentProps<"input">, "size"> & { size?: "sm" | "lg" }`.

### Ayırt edici birleşim (discriminated union) props

```tsx
type BildirimProps =
  | { tur: "bilgi"; mesaj: string }
  | { tur: "onay"; mesaj: string; onOnayla: () => void; onIptal: () => void }
  | { tur: "hata"; mesaj: string; tekrarDene?: () => void };

function Bildirim(props: BildirimProps) {
  switch (props.tur) {
    case "bilgi":
      return <p className="bilgi">{props.mesaj}</p>;
    case "onay":
      return (
        <div>
          <p>{props.mesaj}</p>
          <button onClick={props.onOnayla}>Evet</button>   {/* kesin var */}
          <button onClick={props.onIptal}>Hayır</button>
        </div>
      );
    case "hata":
      return (
        <p role="alert">
          {props.mesaj}{" "}
          {props.tekrarDene && <button onClick={props.tekrarDene}>Tekrar dene</button>}
        </p>
      );
  }
}

<Bildirim tur="onay" mesaj="Silinsin mi?" />                     // ❌ onOnayla eksik
<Bildirim tur="bilgi" mesaj="Kaydedildi" onOnayla={() => {}} />  // ❌ bilinmeyen prop
```

> **Ayırt edici alan** (`tur`) geçerli seçeneği belirler; yanlış prop kombinasyonu **yazılamaz.**

---

## 5. Hook'ları Tiplemek ⭐

### `useState`

```tsx
const [sayi, setSayi] = useState(0);         // number
const [ad, setAd] = useState("");            // string
const [acik, setAcik] = useState(false);     // boolean
```

Başlangıç değeri tipi tam anlatmıyorsa **generic**:

```tsx
const [kullanici, setKullanici] = useState<Kullanici | null>(null);
const [urunler, setUrunler] = useState<Urun[]>([]);
const [durum, setDurum] = useState<Durum>("bosta");

kullanici.ad;                 // ❌ 'kullanici' is possibly 'null'
kullanici?.ad;                // ✅
if (kullanici) kullanici.ad;  // ✅
```

> ⚠️ `useState([])` → `never[]`; boş dizi ve `null` başlangıçlarında **mutlaka** generic verilir.

### `useReducer` + ayırt edici birleşim action'lar ⭐

```tsx
type Gorev = { id: string; metin: string; tamam: boolean };

type GorevAction =
  | { type: "eklendi"; id: string; metin: string }
  | { type: "toggle"; id: string }
  | { type: "silindi"; id: string }
  | { type: "tamamlananlar_temizlendi" };

function assertNever(x: never): never {
  throw new Error(`Beklenmeyen değer: ${JSON.stringify(x)}`);
}

function gorevReducer(gorevler: Gorev[], action: GorevAction): Gorev[] {
  switch (action.type) {
    case "eklendi":
      return [...gorevler, { id: action.id, metin: action.metin, tamam: false }];
    case "toggle":
      return gorevler.map((g) => (g.id === action.id ? { ...g, tamam: !g.tamam } : g));
    case "silindi":
      return gorevler.filter((g) => g.id !== action.id);
    case "tamamlananlar_temizlendi":
      return gorevler.filter((g) => !g.tamam);
    default:
      return assertNever(action);   // tüm durumlar ele alındı mı? (kapsamlılık kontrolü)
  }
}

const [gorevler, dispatch] = useReducer(gorevReducer, []);

dispatch({ type: "eklendi", id: "1", metin: "TS öğren" });   // ✅
dispatch({ type: "eklendi", id: "1" });                       // ❌ 'metin' eksik
dispatch({ type: "slindi", id: "1" });                        // ❌ yazım hatası
```

- Her action türü için gerekli alanlar tiplenir; yanlış `dispatch` **yazılamaz.**
- **`assertNever`**: Yeni bir action türü ekleyip `case`'i unutursan **derleme hatası**; çalışma zamanında da hata fırlatır.

### `useRef`

```tsx
// 1. DOM ref'i: başlangıç null, element tipi verilir
const inputRef = useRef<HTMLInputElement>(null);
inputRef.current?.focus();   // HTMLInputElement | null
<input ref={inputRef} />

// 2. Değer saklama: başlangıçtan çıkarılır
const sayacRef = useRef(0);
const zamanlayiciRef = useRef<ReturnType<typeof setTimeout> | null>(null);
```

**Sık element tipleri:** `HTMLInputElement`, `HTMLButtonElement`, `HTMLDivElement`, `HTMLFormElement`, `HTMLTextAreaElement`, `HTMLSelectElement`, `HTMLVideoElement`, `HTMLUListElement`

### Context

```tsx
import { createContext, useContext, useState, type ReactNode } from "react";

type Tema = "acik" | "koyu";

type TemaContextDegeri = {
  tema: Tema;
  temaDegistir: () => void;
};

const TemaContext = createContext<TemaContextDegeri | null>(null);

export function TemaProvider({ children }: { children: ReactNode }) {
  const [tema, setTema] = useState<Tema>("acik");
  const temaDegistir = () => setTema((t) => (t === "acik" ? "koyu" : "acik"));
  return <TemaContext value={{ tema, temaDegistir }}>{children}</TemaContext>;
}

export function useTema(): TemaContextDegeri {
  const deger = useContext(TemaContext);
  if (!deger) throw new Error("useTema, <TemaProvider> içinde kullanılmalı.");
  return deger;   // tip: TemaContextDegeri (null değil)
}
```

> Hata kontrolü aynı zamanda **tip daraltma** sağlar; kullananlar her yerde `?.` yazmak zorunda kalmaz.

### Custom Hook dönüş tipleri: `as const`

```tsx
function useToggle(baslangic = false) {
  const [deger, setDeger] = useState(baslangic);
  const toggle = () => setDeger((d) => !d);
  return [deger, toggle];            // (boolean | (() => void))[]  ❌
}

function useToggle(baslangic = false) {
  // ...
  return [deger, toggle] as const;   // readonly [boolean, () => void]  ✅
}
```

> Tuple döndüren Hook'larda `as const`; nesne döndürenlerde gerek yok.

---

## 6. Olayları Tiplemek

### Inline: otomatik çıkarım

```tsx
<input onChange={(e) => setAd(e.target.value)} />   // e: ChangeEvent<HTMLInputElement>
```

### Ayrı fonksiyon: tip belirtilir

```tsx
import type { ChangeEvent, FormEvent, KeyboardEvent, MouseEvent } from "react";

function handleChange(e: ChangeEvent<HTMLInputElement>) {
  const { name, value, type, checked } = e.target;
}

function handleSubmit(e: FormEvent<HTMLFormElement>) {
  e.preventDefault();
  const formData = new FormData(e.currentTarget);
}

function handleKeyDown(e: KeyboardEvent<HTMLTextAreaElement>) {
  if (e.key === "Enter" && !e.shiftKey) { /* ... */ }
}

function handleClick(e: MouseEvent<HTMLButtonElement>) {
  e.stopPropagation();
}
```

> 💡 Tipi hatırlamıyorsan: önce inline yaz, `e`'nin üzerine gel, VS Code gösterir.

### Olay prop'ları

```tsx
type AramaKutusuProps = {
  onAra: (sorgu: string) => void;                        // uygulamaya özgü
  onChange?: (e: ChangeEvent<HTMLInputElement>) => void; // DOM olayını geçirmek
};
```

### FormData ve form action'ları

- `formData.get()` → `FormDataEntryValue | null` (string, dosya veya null).

```tsx
async function kayitAction(onceki: KayitDurumu, formData: FormData): Promise<KayitDurumu> {
  const ad = String(formData.get("ad") ?? "").trim();
  // ...
}

const [durum, formAction, isPending] = useActionState<KayitDurumu, FormData>(
  kayitAction,
  { durum: "bosta" }
);
```

---

## 7. Dış Veriyi Tiplemek: API'ler ve Zod ⭐

### Tipler bir söz, kontrol değil

```ts
async function urunGetir(id: number): Promise<Urun> {
  const res = await fetch(`/api/urunler/${id}`);
  return res.json();   // any → TypeScript "sana güveniyorum" der
}
```

- Tipler çalışırken **silinir**; API'ye tip yapıştırmak veriyi kontrol etmez.
- **Uygulamanın sınırında** (API, `localStorage`, URL, form) gelen veri `unknown` kabul edilip **doğrulanmalıdır.**

### Zod: Şemadan hem doğrulama hem tip

```bash
npm install zod
```

```ts
import { z } from "zod";

const UrunSemasi = z.object({
  id: z.number(),
  title: z.string(),
  price: z.number().nonnegative(),
  category: z.string(),
  thumbnail: z.url(),
  rating: z.number().min(0).max(5).optional(),
});

type Urun = z.infer<typeof UrunSemasi>;

async function urunGetir(id: number, signal?: AbortSignal): Promise<Urun> {
  const res = await fetch(`https://dummyjson.com/products/${id}`, { signal });
  if (!res.ok) throw new Error(`HTTP ${res.status}`);
  return UrunSemasi.parse(await res.json());   // uymazsa net hata
}
```

- Tip elle yazılmaz; **tek doğruluk kaynağı** şemadır.
- Bozuk veri, uygulamaya **girdiği yerde** anlaşılır bir mesajla yakalanır.

### Form doğrulama

```ts
const KayitSemasi = z.object({
  ad: z.string().trim().min(3, "Ad en az 3 karakter olmalı."),
  email: z.email("Geçerli bir e-posta girin."),
  yas: z.coerce.number().int().min(18, "18 yaşından büyük olmalısınız."),   // string → number
});

const sonuc = KayitSemasi.safeParse(Object.fromEntries(formData));
if (!sonuc.success) {
  const hatalar = z.flattenError(sonuc.error).fieldErrors;   // { ad?: string[], ... }
}
```

### TanStack Query

```tsx
const urunSorgusu = (id: number) =>
  queryOptions({
    queryKey: ["urunler", id],
    queryFn: ({ signal }) => urunGetir(id, signal),   // Promise<Urun>
  });

const { data } = useQuery(urunSorgusu(5));           // Urun | undefined
const { data } = useSuspenseQuery(urunSorgusu(5));   // Urun
```

- Veri tipi `queryFn`'den **otomatik** çıkarılır; `queryOptions` key ile tipi bağlar.

### React Router

```tsx
import { data, useLoaderData, type LoaderFunctionArgs } from "react-router";

export async function loader({ params }: LoaderFunctionArgs) {
  const not = await notGetir(params.notId!);
  if (!not) throw data("Bulunamadı", { status: 404 });
  return { not };
}

export default function NotDetay() {
  const { not } = useLoaderData<typeof loader>();   // loader'dan çıkarıldı
}
```

> Framework modu her rota için tipleri otomatik üretir (`Route.LoaderArgs`, `Route.ComponentProps`).

---

## 8. Generic Bileşenler

```tsx
import type { ReactNode, Key } from "react";

type ListeProps<T> = {
  ogeler: readonly T[];
  anahtar: (oge: T) => Key;
  render: (oge: T) => ReactNode;
  bosMesaj?: string;
};

function Liste<T>({ ogeler, anahtar, render, bosMesaj = "Gösterilecek öğe yok." }: ListeProps<T>) {
  if (ogeler.length === 0) return <p>{bosMesaj}</p>;
  return (
    <ul>
      {ogeler.map((oge) => (
        <li key={anahtar(oge)}>{render(oge)}</li>
      ))}
    </ul>
  );
}

// T otomatik olarak Kullanici
<Liste
  ogeler={kullanicilar}
  anahtar={(k) => k.id}
  render={(k) => <span>{k.ad} — {k.email}</span>}
/>
```

### Generic seçici

```tsx
type SeciciProps<T> = {
  secenekler: readonly T[];
  deger: T;
  onDegis: (yeni: T) => void;
  etiket: (s: T) => string;
  anahtar: (s: T) => string;
};

function Secici<T>({ secenekler, deger, onDegis, etiket, anahtar }: SeciciProps<T>) {
  return (
    <select
      value={anahtar(deger)}
      onChange={(e) => {
        const secilen = secenekler.find((s) => anahtar(s) === e.target.value);
        if (secilen) onDegis(secilen);
      }}
    >
      {secenekler.map((s) => (
        <option key={anahtar(s)} value={anahtar(s)}>{etiket(s)}</option>
      ))}
    </select>
  );
}
```

> `.tsx`'te arrow function ile generic: `const Liste = <T,>(props: ListeProps<T>) => ...` (virgül şart). `function` sözdizimi daha rahattır.

---

## 9. Faydalı Yardımcı Tipler (Utility Types)

```ts
type Urun = { id: number; ad: string; fiyat: number; aciklama: string; stok: number };
```

| Yardımcı | Ne yapar | Örnek |
|---|---|---|
| `Partial<T>` | Tüm alanlar opsiyonel | `urunGuncelle(id, d: Partial<Urun>)` |
| `Required<T>` | Tüm alanlar zorunlu | |
| `Pick<T, K>` | Seçilen alanlar | `Pick<Urun, "id" \| "ad">` |
| `Omit<T, K>` | Seçilenler hariç | `Omit<Urun, "id">` → yeni ürün formu |
| `Record<K, V>` | Anahtar → değer haritası | `Record<Durum, string>` |
| `Readonly<T>` | Değiştirilemez | |
| `NonNullable<T>` | `null`/`undefined` çıkarılır | |
| `ReturnType<F>` | Dönüş tipi | `ReturnType<typeof useGorevler>` |
| `Awaited<T>` | Promise'in çözüldüğü tip | `Awaited<ReturnType<typeof urunGetir>>` |
| `Parameters<F>` | Parametre tipleri | |
| `ComponentProps<C>` | Bileşen / HTML elementi props'u | |

```ts
const DURUM_RENKLERI: Record<Durum, string> = {
  bosta: "gray",
  yukleniyor: "blue",
  hata: "red",
  basarili: "green",
  // bir durum unutulursa → ❌
};
```

> `Record<Durum, ...>` eşleme nesnelerini **eksiksiz** yapar; `Durum`'a yeni değer eklenince tüm eşlemeler hata verir.

---

## 10. Yaygın Hatalar ve Kaçış Kapıları

| Hata | Neden kötü | Alternatif |
|---|---|---|
| `any` | TypeScript'i o noktada kapatır | `unknown` + doğrulama |
| `veri as Kullanici` | "Bana güven" der; kontrol yok | Daraltma veya Zod |
| `ref.current!` | Yanılırsan çalışma zamanı hatası | `?.` veya açık kontrol (`getElementById("root")!` gibi kesin durumlar hariç) |
| `// @ts-ignore` | Hatayı gizler | Çöz; gerekirse açıklamalı `// @ts-expect-error` |
| `enum` | JS'te karşılığı olmayan kod üretir | Literal birleşim veya `as const` nesne |
| Her şeyi tiplemek | Gürültü | Çıkarıma güven |

```tsx
const [sayi, setSayi] = useState<number>(0);                                    // gereksiz
const toplam: number = urunler.reduce((t: number, u: Urun) => t + u.fiyat, 0);   // gereksiz
```

---

## 11. Mevcut Bir JavaScript Projesini TypeScript'e Geçirmek

1. Kur: `npm install -D typescript @types/react @types/react-dom`
2. `tsconfig.json` ekle; **`"allowJs": true`** ile `.js` ve `.ts` yan yana yaşasın.
3. Dosyaları birer birer `.jsx` → `.tsx` yap; **yapraklardan** başla: yardımcılar, tipler, API → küçük bileşenler → büyük sayfalar.
4. Mümkünse baştan **`strict: true`** ile geç.
5. Tipi olmayan eski kütüphaneler için `@types/kutuphane-adi` (DefinitelyTyped); modern kütüphaneler tiplerini içinde getirir.

---

## 12. Her Şey Bir Arada: Tipli Ürün Yönetimi

Zod şemaları, ayırt edici birleşim action'lı reducer + kapsamlılık kontrolü, tipli context, `ComponentProps` ile buton, generic liste, ayırt edici birleşim props, tipli React 19 form action'ı ve TanStack Query.

### Şemalar ve tipler

```ts
// ============ src/tipler.ts ============
import { z } from "zod";

export const KATEGORILER = ["elektronik", "kitap", "giyim"] as const;
export type Kategori = (typeof KATEGORILER)[number];

export const UrunSemasi = z.object({
  id: z.string(),
  ad: z.string().min(1),
  fiyat: z.number().nonnegative(),
  kategori: z.enum(KATEGORILER),
  stok: z.number().int().nonnegative(),
});
export type Urun = z.infer<typeof UrunSemasi>;

// Form: id sunucuda üretilir; FormData string'leri sayıya çevrilir (coerce)
export const YeniUrunSemasi = z.object({
  ad: z.string().trim().min(3, "Ürün adı en az 3 karakter olmalı."),
  fiyat: z.coerce.number().positive("Fiyat sıfırdan büyük olmalı."),
  kategori: z.enum(KATEGORILER, { error: "Bir kategori seçin." }),
  stok: z.coerce.number().int().min(0, "Stok negatif olamaz."),
});
export type YeniUrun = z.infer<typeof YeniUrunSemasi>;

export const KATEGORI_ETIKETLERI: Record<Kategori, string> = {
  elektronik: "Elektronik",
  kitap: "Kitap",
  giyim: "Giyim",
};

export function assertNever(x: never): never {
  throw new Error(`Beklenmeyen değer: ${JSON.stringify(x)}`);
}
```

### Doğrulanmış sahte API

```ts
// ============ src/api.ts ============
import { z } from "zod";
import { UrunSemasi, type Urun, type YeniUrun } from "./tipler";

const bekle = (ms: number) => new Promise<void>((r) => setTimeout(r, ms));

let veritabani: unknown = [   // dışarıdan gelen veri: unknown
  { id: "1", ad: "Mekanik Klavye", fiyat: 2499, kategori: "elektronik", stok: 12 },
  { id: "2", ad: "Suç ve Ceza", fiyat: 180, kategori: "kitap", stok: 0 },
  { id: "3", ad: "Kapüşonlu Sweatshirt", fiyat: 799, kategori: "giyim", stok: 4 },
];

export async function urunleriGetir(): Promise<Urun[]> {
  await bekle(400);
  return z.array(UrunSemasi).parse(veritabani);   // sınırda doğrula
}

export async function urunEkle(yeni: YeniUrun): Promise<Urun> {
  await bekle(600);
  const mevcut = z.array(UrunSemasi).parse(veritabani);
  const urun: Urun = { id: crypto.randomUUID(), ...yeni };
  veritabani = [...mevcut, urun];
  return urun;
}
```

### Tipli UI bileşenleri

```tsx
// ============ src/ui.tsx ============
import type { ComponentProps, Key, ReactNode } from "react";

// --- ComponentProps ile genişletilmiş buton ---
type ButonProps = ComponentProps<"button"> & {
  varyant?: "birincil" | "ikincil" | "tehlike";
};

const VARYANT_SINIFLARI: Record<NonNullable<ButonProps["varyant"]>, string> = {
  birincil: "bg-indigo-600 text-white hover:bg-indigo-700",
  ikincil: "bg-gray-100 hover:bg-gray-200",
  tehlike: "bg-red-600 text-white hover:bg-red-700",
};

export function Buton({ varyant = "birincil", className = "", ...rest }: ButonProps) {
  return (
    <button
      className={`rounded px-3 py-1.5 font-medium disabled:opacity-50 ${VARYANT_SINIFLARI[varyant]} ${className}`}
      {...rest}
    />
  );
}

// --- Generic liste ---
type ListeProps<T> = {
  ogeler: readonly T[];
  anahtar: (oge: T) => Key;
  render: (oge: T) => ReactNode;
  bosMesaj?: string;
};

export function Liste<T>({
  ogeler,
  anahtar,
  render,
  bosMesaj = "Gösterilecek öğe yok.",
}: ListeProps<T>) {
  if (ogeler.length === 0) return <p className="text-gray-500">{bosMesaj}</p>;
  return (
    <ul className="divide-y rounded border">
      {ogeler.map((oge) => (
        <li key={anahtar(oge)} className="px-3 py-2">{render(oge)}</li>
      ))}
    </ul>
  );
}

// --- Ayırt edici birleşim props ---
export type BildirimProps =
  | { tur: "basarili"; mesaj: string }
  | { tur: "hata"; mesaj: string; tekrarDene?: () => void };

export function Bildirim(props: BildirimProps) {
  if (props.tur === "basarili") {
    return <p role="status" className="text-green-700">✓ {props.mesaj}</p>;
  }
  return (
    <p role="alert" className="text-red-600">
      {props.mesaj}{" "}
      {props.tekrarDene && (
        <Buton varyant="ikincil" onClick={props.tekrarDene}>Tekrar dene</Buton>
      )}
    </p>
  );
}
```

### Reducer + tipli context

```tsx
// ============ src/sepet.tsx ============
import { createContext, useContext, useReducer, type Dispatch, type ReactNode } from "react";
import { assertNever, type Urun } from "./tipler";

type SepetSatiri = { urun: Urun; adet: number };

type SepetAction =
  | { type: "eklendi"; urun: Urun }
  | { type: "adet_degisti"; id: string; adet: number }
  | { type: "silindi"; id: string }
  | { type: "bosaltildi" };

function sepetReducer(satirlar: SepetSatiri[], action: SepetAction): SepetSatiri[] {
  switch (action.type) {
    case "eklendi": {
      const mevcut = satirlar.find((s) => s.urun.id === action.urun.id);
      return mevcut
        ? satirlar.map((s) => (s === mevcut ? { ...s, adet: s.adet + 1 } : s))
        : [...satirlar, { urun: action.urun, adet: 1 }];
    }
    case "adet_degisti":
      return satirlar.map((s) =>
        s.urun.id === action.id ? { ...s, adet: Math.max(1, action.adet) } : s
      );
    case "silindi":
      return satirlar.filter((s) => s.urun.id !== action.id);
    case "bosaltildi":
      return [];
    default:
      return assertNever(action);   // yeni action eklenip unutulursa derleme hatası
  }
}

type SepetDegeri = {
  satirlar: SepetSatiri[];
  toplam: number;
  dispatch: Dispatch<SepetAction>;
};

const SepetContext = createContext<SepetDegeri | null>(null);

export function SepetProvider({ children }: { children: ReactNode }) {
  const [satirlar, dispatch] = useReducer(sepetReducer, []);
  const toplam = satirlar.reduce((t, s) => t + s.urun.fiyat * s.adet, 0);   // türetilmiş
  return <SepetContext value={{ satirlar, toplam, dispatch }}>{children}</SepetContext>;
}

export function useSepet(): SepetDegeri {
  const deger = useContext(SepetContext);
  if (!deger) throw new Error("useSepet, <SepetProvider> içinde kullanılmalı.");
  return deger;
}
```

### Uygulama

```tsx
// ============ src/UrunYonetimi.tsx ============
import { useActionState } from "react";
import { useFormStatus } from "react-dom";
import { queryOptions, useMutation, useQuery, useQueryClient } from "@tanstack/react-query";
import { z } from "zod";
import { urunEkle, urunleriGetir } from "./api";
import {
  KATEGORILER, KATEGORI_ETIKETLERI, YeniUrunSemasi,
  type Urun, type YeniUrun,
} from "./tipler";
import { Bildirim, Buton, Liste } from "./ui";
import { SepetProvider, useSepet } from "./sepet";

const urunlerSorgusu = queryOptions({
  queryKey: ["urunler"],
  queryFn: urunleriGetir,   // data: Urun[] | undefined
});

// --- Form action'ının durum tipi: ayırt edici birleşim ---
type FormDegerleri = Partial<Record<keyof YeniUrun, string>>;

type FormDurumu =
  | { durum: "bosta" }
  | {
      durum: "hata";
      alanHatalari: Partial<Record<keyof YeniUrun, string[]>>;
      degerler: FormDegerleri;   // hata durumunda yazılanlar kaybolmasın
      genelHata?: string;
    }
  | { durum: "basarili"; eklenen: string };

function KaydetButonu() {
  const { pending } = useFormStatus();
  return (
    <Buton type="submit" disabled={pending}>
      {pending ? "Ekleniyor..." : "Ürün ekle"}
    </Buton>
  );
}

function YeniUrunFormu() {
  const queryClient = useQueryClient();
  const ekle = useMutation({
    mutationFn: urunEkle,
    onSuccess: () => queryClient.invalidateQueries({ queryKey: urunlerSorgusu.queryKey }),
  });

  const [sonuc, formAction] = useActionState<FormDurumu, FormData>(
    async (_onceki, formData) => {
      const degerler = Object.fromEntries(formData) as FormDegerleri;
      const ayristirma = YeniUrunSemasi.safeParse(degerler);

      if (!ayristirma.success) {
        return {
          durum: "hata",
          alanHatalari: z.flattenError(ayristirma.error).fieldErrors,
          degerler,
        };
      }
      try {
        const urun = await ekle.mutateAsync(ayristirma.data);   // ayristirma.data: YeniUrun
        return { durum: "basarili", eklenen: urun.ad };
      } catch (err) {
        return {
          durum: "hata",
          alanHatalari: {},
          degerler,
          genelHata: err instanceof Error ? err.message : "Bilinmeyen hata",
        };
      }
    },
    { durum: "bosta" }
  );

  // Daraltma: sadece "hata" durumunda alan hatası ve değer okunabilir
  const hata = (alan: keyof YeniUrun) =>
    sonuc.durum === "hata" ? sonuc.alanHatalari[alan]?.[0] : undefined;
  const deger = (alan: keyof YeniUrun) =>
    sonuc.durum === "hata" ? sonuc.degerler[alan] : undefined;

  return (
    <form action={formAction} className="space-y-3 rounded border p-4">
      <h2 className="font-semibold">Yeni ürün</h2>

      <label className="block">
        Ad
        <input name="ad" defaultValue={deger("ad")} className="w-full rounded border px-2 py-1" />
        {hata("ad") && <small className="text-red-600">{hata("ad")}</small>}
      </label>

      <div className="grid grid-cols-2 gap-3">
        <label>
          Fiyat
          <input
            name="fiyat"
            type="number"
            step="0.01"
            defaultValue={deger("fiyat")}
            className="w-full rounded border px-2 py-1"
          />
          {hata("fiyat") && <small className="text-red-600">{hata("fiyat")}</small>}
        </label>
        <label>
          Stok
          <input
            name="stok"
            type="number"
            defaultValue={deger("stok") ?? "0"}
            className="w-full rounded border px-2 py-1"
          />
          {hata("stok") && <small className="text-red-600">{hata("stok")}</small>}
        </label>
      </div>

      <label className="block">
        Kategori
        <select
          name="kategori"
          defaultValue={deger("kategori") ?? ""}
          className="w-full rounded border px-2 py-1"
        >
          <option value="" disabled>Seçin</option>
          {KATEGORILER.map((k) => (
            <option key={k} value={k}>{KATEGORI_ETIKETLERI[k]}</option>
          ))}
        </select>
        {hata("kategori") && <small className="text-red-600">{hata("kategori")}</small>}
      </label>

      {sonuc.durum === "basarili" && (
        <Bildirim tur="basarili" mesaj={`${sonuc.eklenen} eklendi.`} />
      )}
      {sonuc.durum === "hata" && sonuc.genelHata && (
        <Bildirim tur="hata" mesaj={sonuc.genelHata} />
      )}

      <KaydetButonu />
    </form>
  );
}

function UrunListesi() {
  const { data: urunler = [], isPending, isError, refetch } = useQuery(urunlerSorgusu);
  const { dispatch } = useSepet();

  if (isPending) return <p>Yükleniyor...</p>;
  if (isError) {
    return <Bildirim tur="hata" mesaj="Ürünler yüklenemedi." tekrarDene={() => refetch()} />;
  }

  return (
    <Liste<Urun>
      ogeler={urunler}
      anahtar={(u) => u.id}
      bosMesaj="Henüz ürün yok."
      render={(u) => (
        <div className="flex items-center justify-between">
          <span>
            {u.ad} <small className="text-gray-500">{KATEGORI_ETIKETLERI[u.kategori]}</small>
          </span>
          <span className="flex items-center gap-2">
            <span className="tabular-nums">{u.fiyat.toLocaleString("tr-TR")} ₺</span>
            <Buton
              varyant="ikincil"
              disabled={u.stok === 0}
              onClick={() => dispatch({ type: "eklendi", urun: u })}
            >
              {u.stok === 0 ? "Tükendi" : "Sepete ekle"}
            </Buton>
          </span>
        </div>
      )}
    />
  );
}

function Sepet() {
  const { satirlar, toplam, dispatch } = useSepet();

  return (
    <section className="rounded border p-4">
      <h2 className="mb-2 font-semibold">Sepet</h2>
      <Liste
        ogeler={satirlar}
        anahtar={(s) => s.urun.id}
        bosMesaj="Sepetin boş."
        render={(s) => (
          <div className="flex items-center justify-between gap-2">
            <span className="truncate">{s.urun.ad}</span>
            <input
              type="number"
              min={1}
              value={s.adet}
              onChange={(e) =>
                dispatch({ type: "adet_degisti", id: s.urun.id, adet: e.target.valueAsNumber || 1 })
              }
              className="w-16 rounded border px-1"
              aria-label={`${s.urun.ad} adedi`}
            />
            <Buton
              varyant="tehlike"
              onClick={() => dispatch({ type: "silindi", id: s.urun.id })}
              aria-label={`${s.urun.ad} ürününü sil`}
            >
              ✕
            </Buton>
          </div>
        )}
      />
      <p className="mt-3 font-bold">Toplam: {toplam.toLocaleString("tr-TR")} ₺</p>
      {satirlar.length > 0 && (
        <Buton varyant="ikincil" onClick={() => dispatch({ type: "bosaltildi" })}>
          Sepeti boşalt
        </Buton>
      )}
    </section>
  );
}

export default function UrunYonetimi() {
  return (
    <SepetProvider>
      <main className="mx-auto grid max-w-5xl gap-6 p-6 md:grid-cols-[1fr_320px]">
        <div className="space-y-6">
          <h1 className="text-2xl font-bold">Ürün Yönetimi</h1>
          <UrunListesi />
          <YeniUrunFormu />
        </div>
        <Sepet />
      </main>
    </SepetProvider>
  );
}
```

### Editörde dene

| Deneme | TypeScript'in tepkisi |
|---|---|
| `dispatch({ type: "eklendi" })` | `urun` eksik |
| Reducer'a yeni action türü ekle, `case` yazma | `assertNever` satırında hata |
| `KATEGORILER`'e `"oyuncak"` ekle | `KATEGORI_ETIKETLERI` eksik anahtar; Zod şeması ve `<select>` otomatik güncellenir |
| `<Buton varyant="kirmizi">` | Geçersiz varyant; yerel `<button>` props'ları otomatik tamamlanır |
| `<Bildirim tur="basarili" tekrarDene={...}>` | "basarili" türünde `tekrarDene` yok |
| `Liste`'nin `render`'ında `u.` | `Urun` alanları listelenir |
| `hata("fiyt")` | `keyof YeniUrun` değil |
| Sahte API'de `fiyat`'ı `"2499"` yap | Zod, veri uygulamaya girerken net hata fırlatır |

> Tipler çoğunlukla **Zod şemalarından**, **`as const` dizilerden** ve **çıkarımdan** geliyor; açıkça yalnızca **sınırlarda** (props, action'lar, context) tanımlandı.

---

## 13. Özet

- **TypeScript = JavaScript + tipler**; hataları yazarken yakalar, editör desteği sağlar, kodu belgeler.
- Kurulum: `react-ts` şablonu; **`strict: true`** şart; `verbatimModuleSyntax` ile **`import type`.**
- **TypeScript 7.0** Go tabanlı, 8–12 kat hızlı; 6.0 köprü sürüm; 7.0'da kararlı API henüz yok.
- **Vite tip kontrolü yapmaz**; kontrol editörde, `tsc -b` (build) ve `tsc --noEmit` (CI) ile.
- Temeller: çıkarıma güven, sınırlarda tip yaz; literal birleşimler, daraltma, generic'ler; `any` yerine `unknown`; `as const` ve `satisfies` ile değerden tip türet.
- **Props:** fonksiyon parametresi tiplenir (`React.FC` gereksiz); `children: ReactNode`; HTML saran bileşenlerde **`ComponentProps<"button">`**; bağlı props için **ayırt edici birleşim.**
- **`useState`**: boş dizi ve `null` başlangıçlarında generic; **`useReducer`**: ayırt edici birleşim action'lar + **`assertNever`** ile kapsamlılık kontrolü.
- **`useRef<HTMLInputElement>(null)`** DOM için; context'te `null` varsayılan + hata kontrollü custom Hook tipi daraltır; tuple döndüren Hook'larda `as const`.
- Olaylar inline'da otomatik çıkarılır; ayrı fonksiyonlarda `ChangeEvent`, `FormEvent`, `KeyboardEvent`, `MouseEvent`; `formData.get()` → `FormDataEntryValue | null`.
- **Tipler kontrol değildir:** Sınırdaki veri **Zod** ile doğrulanır; `z.infer` ile tip şemadan üretilir; form doğrulamada `safeParse` + `flattenError`.
- TanStack Query tipi `queryFn`'den çıkarır; React Router'da `useLoaderData<typeof loader>()`.
- **Generic bileşenler** (`Liste<T>`, `Secici<T>`) veri tipini korur.
- Yardımcı tipler: `Partial`, `Pick`, `Omit`, `Record`, `ReturnType`, `Awaited`, `NonNullable`, `ComponentProps`; `Record<Durum, ...>` eşlemeleri eksiksiz yapar.
- Kaçınılacaklar: `any`, gereksiz `as`, `!`, `@ts-ignore`, `enum`, aşırı tip yazımı.
- Kademeli geçiş: `allowJs`, yapraklardan başla, baştan `strict`.

---

⬅️ Önceki: [21 · Test](./21-test.md) · ➡️ Sonraki: [23 · Next.js ve Server Components](./23-nextjs.md)
