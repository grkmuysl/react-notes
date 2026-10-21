# 14 · `useReducer`

> **Konu özeti:** `useReducer`'ın neden var olduğu, reducer / action / dispatch kavramları, `useState`'ten `useReducer`'a geçiş, action tasarımı, reducer kuralları, birden çok state'i tek reducer'da yönetmek, `useState` ile karşılaştırma, tembel başlatma, `dispatch`'in özellikleri, Immer, reducer testleri, durum makineleri ve reducer + context ile ölçeklenebilir state yönetimi.

---

## İçindekiler

1. [Sezgisel Bakış: Restoran Benzetmesi](#1-sezgisel-bakış-restoran-benzetmesi)
2. [Problem: Dağınık State Güncellemeleri](#2-problem-dağınık-state-güncellemeleri)
3. [Temel Kavramlar: Reducer, Action, Dispatch](#3-temel-kavramlar-reducer-action-dispatch)
4. [`useState`'ten `useReducer`'a: Adım Adım](#4-usestateten-usereducera-adım-adım-)
5. [Action Tasarımı](#5-action-tasarımı)
6. [Reducer Kuralları](#6-reducer-kuralları-)
7. [Birden Çok State'i Tek Reducer'da Yönetmek](#7-birden-çok-statei-tek-reducerda-yönetmek)
8. [`useState` mi, `useReducer` mı?](#8-usestate-mi-usereducer-mı)
9. [Başlangıç State'i ve Tembel Başlatma](#9-başlangıç-statei-ve-tembel-başlatma)
10. [`dispatch` Hakkında İki Detay](#10-dispatch-hakkında-iki-detay)
11. [Immer ile Daha Kısa Reducer'lar](#11-immer-ile-daha-kısa-reducerlar)
12. [Reducer'ı Test Etmek](#12-reducerı-test-etmek)
13. [Reducer ile Durum Makinesi](#13-reducer-ile-durum-makinesi-state-machine)
14. [Reducer + Context](#14-reducer--context-ölçeklenebilir-state-yönetimi-)
15. [Her Şey Bir Arada: Geri Alınabilir Görev Panosu](#15-her-şey-bir-arada-geri-alınabilir-görev-panosu)
16. [Özet](#16-özet)

---

## 1. Sezgisel Bakış: Restoran Benzetmesi

### Üç parçanın görevi

| Parça | Ne yapar | Soru |
|---|---|---|
| **Action** | Ne olduğunu anlatan bir not | "Ne oldu?" |
| **Dispatch** | Bu notu reducer'a **götüren** fonksiyon | "Notu kime vereyim?" |
| **Reducer** | Notu okuyup state'in nasıl değişeceğine karar verir | "Bu olunca state ne olmalı?" |

### Restoran

- **Müşteri** (bileşen): "Bir mercimek çorbası" der. Nasıl yapılacağını bilmez, bilmesi de gerekmez.
- **Sipariş fişi** (action): `{ type: "corba_istendi", cesit: "mercimek" }`
- **Garson** (dispatch): Fişi mutfağa götürür. Yemek yapmaz, sadece **iletir.**
- **Mutfak** (reducer): Fişi okur, tarifi bilir, yemeği hazırlar. **Yemek = yeni state.**

> `dispatch` bir şey "göstermez"; sadece **postacıdır.** Bileşen `dispatch(action)` der → React action'ı reducer'a verir → reducer yeni state'i döndürür → ekran güncellenir.

### Aynı şeyin iki yazımı

```jsx
// useState ile: bileşen state'i NASIL değiştireceğini kendisi biliyor
function Sayac() {
  const [sayi, setSayi] = useState(0);

  return (
    <>
      <p>{sayi}</p>
      <button onClick={() => setSayi(sayi + 1)}>+</button>
      <button onClick={() => setSayi(sayi - 1)}>−</button>
      <button onClick={() => setSayi(0)}>Sıfırla</button>
    </>
  );
}
```

```jsx
// useReducer ile: bileşen sadece NE OLDUĞUNU söylüyor
function sayacReducer(sayi, action) {          // mutfak: tüm kurallar burada
  switch (action.type) {
    case "artirildi":  return sayi + 1;
    case "azaltildi":  return sayi - 1;
    case "sifirlandi": return 0;
    default: throw new Error(`Bilinmeyen action: ${action.type}`);
  }
}

function Sayac() {
  const [sayi, dispatch] = useReducer(sayacReducer, 0);

  return (
    <>
      <p>{sayi}</p>
      <button onClick={() => dispatch({ type: "artirildi" })}>+</button>
      <button onClick={() => dispatch({ type: "azaltildi" })}>−</button>
      <button onClick={() => dispatch({ type: "sifirlandi" })}>Sıfırla</button>
    </>
  );
}
```

> Bu kadar basit bir sayaç için `useReducer` **gereksizdir**; yalnızca mekanizmayı göstermek için yazıldı.

### Neden var?

**Karmaşık state'leri düzenli yönetmek için.** "Karmaşık" şu üç durumu kapsar:

1. **Kurallar her yere dağılıyorsa:** Reducer'da "state nasıl değişebilir?" sorusunun cevabı **tek bir `switch`'te** görünür.
2. **Bir değişiklik birkaç state'i birden etkiliyorsa:** Kural reducer'a bir kez yazılır, her zaman uygulanır.
3. **Mantığı test etmek istiyorsan:** Reducer sıradan bir fonksiyondur; React'e dokunmadan test edilir.

> Reducer, **hangi olayda hangi state'in nasıl değişeceğini** tek bir yerde gösteren bir **kural listesidir.**

> `useState` bileşenin kendi yemeğini pişirmesi, `useReducer` siparişi mutfağa bırakmasıdır. Tek kişilik kahvaltı için mutfak kurmaya gerek yok; kalabalık bir restoranda ise herkesin kendi yemeğini pişirmesi kaosa yol açar.

---

## 2. Problem: Dağınık State Güncellemeleri

```jsx
function GorevUygulamasi() {
  const [gorevler, setGorevler] = useState(BASLANGIC);

  function handleEkle(metin) {
    setGorevler([...gorevler, { id: crypto.randomUUID(), metin, tamam: false }]);
  }
  function handleToggle(id) {
    setGorevler(gorevler.map((g) => (g.id === id ? { ...g, tamam: !g.tamam } : g)));
  }
  function handleDuzenle(id, metin) {
    setGorevler(gorevler.map((g) => (g.id === id ? { ...g, metin } : g)));
  }
  function handleSil(id) {
    setGorevler(gorevler.filter((g) => g.id !== id));
  }
  function handleTamamlananlariTemizle() {
    setGorevler(gorevler.filter((g) => !g.tamam));
  }
  // ...
}
```

### Sorunlar

- "Görevler **nasıl** değişebilir?" sorusu için bileşenin tamamını taramak gerekir.
- Güncelleme mantığı render koduyla iç içedir.
- Test etmek için bileşeni render etmek gerekir.
- Birbirine bağlı birden çok state'i tutarlı güncellemek giderek zorlaşır.

→ **`useReducer`**, güncelleme mantığını bileşenin **dışında, tek bir fonksiyonda** toplar.

---

## 3. Temel Kavramlar: Reducer, Action, Dispatch

### Reducer

Mevcut state'i ve bir action'ı alıp **yeni state'i döndüren** saf fonksiyon:

```js
function gorevReducer(state, action) {
  // state + action → yeni state
}
```

### Action

"Ne oldu?" sorusunun cevabı olan sıradan bir nesne; gelenek olarak `type` alanı içerir:

```js
{ type: "eklendi", metin: "React çalış" }
{ type: "toggle", id: "a1b2" }
{ type: "silindi", id: "a1b2" }
```

### Dispatch

Action'ı React'e gönderen fonksiyon; olay yöneticilerinde çağrılır:

```js
dispatch({ type: "silindi", id: gorev.id });
```

### Akış

```
Kullanıcı tıklar
  → dispatch({ type: "silindi", id: 3 })
  → React reducer'ı çağırır: gorevReducer(mevcutState, action)
  → reducer yeni state'i döndürür
  → React yeni state ile yeniden render eder
```

### Neden "reducer"?

`Array.prototype.reduce`'tan gelir: Elemanları sırayla bir biriktiriciye uygulayıp tek değere "indirger."

```js
const toplam = [1, 2, 3].reduce((birikmis, sayi) => birikmis + sayi, 0);   // 6
```

React reducer'ı da **zaman içindeki action'ları** başlangıç state'inden başlayarak tek tek uygular:

```js
const actionlar = [
  { type: "eklendi", id: 1, metin: "A" },
  { type: "eklendi", id: 2, metin: "B" },
  { type: "silindi", id: 1 },
];
const sonState = actionlar.reduce(gorevReducer, []);   // [{ id: 2, metin: "B", ... }]
```

---

## 4. `useState`'ten `useReducer`'a: Adım Adım ⭐

### Adım 1: State güncellemelerini `dispatch` çağrılarına çevir

Olay yöneticileri artık **ne olduğunu** bildirir:

```jsx
function handleEkle(metin) {
  dispatch({ type: "eklendi", id: crypto.randomUUID(), metin });
}
function handleToggle(id) {
  dispatch({ type: "toggle", id });
}
function handleDuzenle(id, metin) {
  dispatch({ type: "duzenlendi", id, metin });
}
function handleSil(id) {
  dispatch({ type: "silindi", id });
}
function handleTamamlananlariTemizle() {
  dispatch({ type: "tamamlananlar_temizlendi" });
}
```

> `id` reducer'da değil olay yöneticisinde üretilir: `crypto.randomUUID()` her çağrıda farklı sonuç verir, reducer ise **saf** olmalıdır. Rastgelelik ve zaman (`Date.now()`) action'a **veri olarak** konur.

### Adım 2: Reducer fonksiyonunu yaz

Genellikle bileşen dışında, hatta ayrı bir dosyada:

```jsx
function gorevReducer(gorevler, action) {
  switch (action.type) {
    case "eklendi":
      return [...gorevler, { id: action.id, metin: action.metin, tamam: false }];

    case "toggle":
      return gorevler.map((g) => (g.id === action.id ? { ...g, tamam: !g.tamam } : g));

    case "duzenlendi":
      return gorevler.map((g) => (g.id === action.id ? { ...g, metin: action.metin } : g));

    case "silindi":
      return gorevler.filter((g) => g.id !== action.id);

    case "tamamlananlar_temizlendi":
      return gorevler.filter((g) => !g.tamam);

    default:
      throw new Error(`Bilinmeyen action: ${action.type}`);
  }
}
```

### Adım 3: Bileşende `useReducer` kullan

```jsx
import { useReducer } from "react";

function GorevUygulamasi() {
  const [gorevler, dispatch] = useReducer(gorevReducer, BASLANGIC);
  // ... handler'lar ve JSX
}
```

```js
const [state, dispatch] = useReducer(reducer, baslangicState);
//      │       │                       │          │
//      │       │                       │          └ başlangıç değeri
//      │       │                       └ reducer fonksiyonu
//      │       └ action gönderen fonksiyon
//      └ mevcut state
```

> `useState` ile neredeyse aynı imza; fark, değiştiricinin yeni değer değil **action** alması.

---

## 5. Action Tasarımı

> **Action, kullanıcının ne yaptığını anlatmalı; state'in nasıl değişeceğini değil.**

```js
// ❌ Setter gibi: "nasıl değişeceğini" söylüyor
{ type: "set_gorevler", gorevler: yeniDizi }
{ type: "set_tamam", id: 3, deger: true }

// ✅ "Ne olduğunu" söylüyor
{ type: "toggle", id: 3 }
{ type: "silindi", id: 3 }
{ type: "form_sifirlandi" }
```

- "Set" tarzı action'lar mantığı tekrar bileşenlere dağıtır.
- Action'lar bir **kullanıcı eylem günlüğü** gibi okunabilmelidir.
- **Tek etkileşim = tek action:** Beş alanı temizleyen buton beş `set_alan` değil, tek bir `form_sifirlandi` gönderir.

### İsimlendirme tarzları

```js
{ type: "eklendi" }          // geçmiş zaman: "ne oldu" (React dokümantasyonu)
{ type: "gorev/eklendi" }    // "alan/olay" (Redux Toolkit)
{ type: "GOREV_EKLE" }       // SCREAMING_CASE (eski Redux kodları)
```

> Önemli olan tutarlılıktır.

### Veri alanları

```js
{ type: "duzenlendi", id: 3, metin: "Yeni" }                 // alanlar doğrudan
{ type: "duzenlendi", payload: { id: 3, metin: "Yeni" } }    // payload (Redux kökenli)
```

---

## 6. Reducer Kuralları ⭐

### 1. Reducer saf olmalı

Aynı state + action → her zaman aynı sonuç; dışarıda hiçbir şey değişmez.

**Reducer'da yapılmaz:**

- API isteği, `setTimeout`, `localStorage`'a yazma
- `Math.random()`, `Date.now()`, `crypto.randomUUID()` → veri olarak action'a konur

- Reducer'lar render sırasında çalışır; StrictMode geliştirmede **iki kez** çağırır.
- Yan etkiler **olay yöneticilerinde** veya **effect'lerde** yapılır.

### 2. Mutasyon yok; yeni state döndür

```js
case "eklendi":
  state.push({ /* ... */ });   // ❌ mutasyon
  return state;                // ❌ aynı referans

case "eklendi":
  return [...state, { /* ... */ }];   // ✅
```

### 3. Bilinmeyen action için hata fırlat

- `default: return state` yaygındır ama `type`'taki yazım hatalarını (`"slindi"`) gizler.
- Hata fırlatmak yanlışlığı anında gösterir.

### 4. `const` kullanan `case`'leri süslü parantezle sar

```js
case "duzenlendi": {
  const hedef = state.find((g) => g.id === action.id);
  if (!hedef) return state;
  return state.map((g) => (g === hedef ? { ...g, metin: action.metin } : g));
}
```

### 5. `switch` zorunlu değil

`if/else` veya nesne eşleme de kullanılabilir; `switch` en okunaklı ve yaygın olanıdır.

---

## 7. Birden Çok State'i Tek Reducer'da Yönetmek

`useReducer`'ın asıl parladığı yer: **birbirine bağlı birden çok state parçası.**

```jsx
const baslangic = {
  gorevler: [],
  filtre: "hepsi",       // "hepsi" | "aktif" | "tamam"
  duzenlenenId: null,
};

function uygulamaReducer(state, action) {
  switch (action.type) {
    case "eklendi":
      return {
        ...state,
        gorevler: [...state.gorevler, { id: action.id, metin: action.metin, tamam: false }],
      };

    case "silindi":
      return {
        ...state,
        gorevler: state.gorevler.filter((g) => g.id !== action.id),
        // Düzenlenen görev silinirse düzenleme modu da kapanır: TUTARLILIK
        duzenlenenId: state.duzenlenenId === action.id ? null : state.duzenlenenId,
      };

    case "duzenleme_basladi":
      return { ...state, duzenlenenId: action.id };

    case "duzenleme_kaydedildi":
      return {
        ...state,
        gorevler: state.gorevler.map((g) =>
          g.id === state.duzenlenenId ? { ...g, metin: action.metin } : g
        ),
        duzenlenenId: null,
      };

    case "filtre_degisti":
      return { ...state, filtre: action.filtre };

    default:
      throw new Error(`Bilinmeyen action: ${action.type}`);
  }
}
```

- `"silindi"`: Kural tek yerde, garanti altında. Ayrı `useState`'lerle her silme noktasında hatırlamak gerekirdi.

> Hesaplanabilen değerler (`gorunenGorevler`, `kalanSayisi`) yine **state'e konmaz**; reducer state'i de minimal olmalı.

---

## 8. `useState` mi, `useReducer` mı?

İkisi de aynı işi yapar (`useState` React içinde basit bir reducer ile uygulanmıştır). Seçim **okunabilirlik ve organizasyon** meselesidir.

| | `useState` | `useReducer` |
|---|---|---|
| Kod miktarı | Az; hemen başlar | Reducer + action'lar için ön yazım |
| Basit değerler (boolean, input metni) | ✅ İdeal | Gereksiz tören |
| Birçok farklı şekilde güncellenen state | Mantık handler'lara dağılır | ✅ Tek yerde toplanır |
| Birbirine bağlı birden çok alan | Senkron tutmak zor | ✅ Tek action'da tutarlı güncelleme |
| Okunabilirlik | Basit durumda daha iyi | Karmaşık durumda daha iyi |
| Hata ayıklama | "Nerede yanlış set edildi?" | Reducer'a log, her action izlenir |
| Test | Bileşeni render etmek gerekir | ✅ Saf fonksiyon, tek başına test |

### Pratik karar kuralı

- State basitse → **`useState`**
- Aynı state'i 3-4'ten fazla farklı şekilde değiştiriyorsan ya da "bunu değiştirirken şunu da güncellemeyi unutmayayım" diye düşünüyorsan → **`useReducer`**

> Aynı bileşende ikisi birlikte kullanılabilir: iş verisi reducer'da, modalın açık/kapalı durumu `useState`'te.

---

## 9. Başlangıç State'i ve Tembel Başlatma

Üçüncü argüman: başlangıç state'ini hesaplayan **`init`** fonksiyonu.

```jsx
function baslangicOlustur(kayitliAnahtar) {
  const kayitli = localStorage.getItem(kayitliAnahtar);
  return {
    gorevler: kayitli ? JSON.parse(kayitli) : [],
    filtre: "hepsi",
    duzenlenenId: null,
  };
}

function GorevUygulamasi() {
  const [state, dispatch] = useReducer(uygulamaReducer, "gorevler", baslangicOlustur);
  // React sadece ilk render'da baslangicOlustur("gorevler") çağırır
}
```

- `useReducer(reducer, baslangicOlustur("gorevler"))` yazılsaydı fonksiyon **her render'da** çalışırdı.
- Aynı `init` fonksiyonu bir sıfırlama action'ında da kullanılabilir (saflık için gerekli veri action'a konmalı).

---

## 10. `dispatch` Hakkında İki Detay

### 1. `dispatch` sabittir

Kimliği bileşenin ömrü boyunca **değişmez** (`set` fonksiyonu gibi):

- Effect bağımlılıklarına yazmak gerekmez.
- Context ile aşağı geçirildiğinde tüketicileri boşuna render etmez.
- `useCallback` ile sarmalamaya gerek yoktur.

### 2. State yine bir anlık görüntüdür

```jsx
function handleTikla() {
  dispatch({ type: "artirildi" });
  console.log(state.sayi);   // hâlâ ESKİ değer
}
```

- `dispatch` bir sonraki render için istek bırakır.
- Aynı olaydaki birden çok `dispatch` kuyruğa girer, tek render'da işlenir (batching).
- Her action bir öncekinin **sonucu** üzerine uygulanır → "üç kez artır ama 1 olur" sorunu yaşanmaz.

---

## 11. Immer ile Daha Kısa Reducer'lar

```bash
npm install immer use-immer
```

```jsx
import { useImmerReducer } from "use-immer";

function gorevReducer(taslak, action) {
  switch (action.type) {
    case "eklendi":
      taslak.push({ id: action.id, metin: action.metin, tamam: false });   // mutasyon gibi
      break;
    case "toggle": {
      const g = taslak.find((g) => g.id === action.id);
      g.tamam = !g.tamam;                                                   // doğrudan değiştir
      break;
    }
    case "silindi":
      return taslak.filter((g) => g.id !== action.id);   // yeni değer döndürmek de serbest
    default:
      throw new Error(`Bilinmeyen action: ${action.type}`);
  }
}

const [gorevler, dispatch] = useImmerReducer(gorevReducer, BASLANGIC);
```

- `taslak` (draft) özel bir kopyadır; Immer değişiklikleri kaydedip değişmez yeni state üretir.
- Redux Toolkit de arka planda Immer kullanır.
- **Ne zaman?** State derin iç içeyse ve spread'ler okunabilirliği bozuyorsa.

---

## 12. Reducer'ı Test Etmek

Reducer saf fonksiyon olduğu için **React'e dokunmadan** test edilir:

```js
// gorevReducer.test.js (Vitest ile — Konu 21)
import { describe, it, expect } from "vitest";
import { gorevReducer } from "./gorevReducer";

describe("gorevReducer", () => {
  const baslangic = [
    { id: "1", metin: "A", tamam: false },
    { id: "2", metin: "B", tamam: true },
  ];

  it("görev ekler", () => {
    const sonuc = gorevReducer(baslangic, { type: "eklendi", id: "3", metin: "C" });
    expect(sonuc).toHaveLength(3);
    expect(sonuc[2]).toEqual({ id: "3", metin: "C", tamam: false });
  });

  it("tamamlananları temizler", () => {
    const sonuc = gorevReducer(baslangic, { type: "tamamlananlar_temizlendi" });
    expect(sonuc).toEqual([{ id: "1", metin: "A", tamam: false }]);
  });

  it("orijinal state'i değiştirmez", () => {
    const kopya = structuredClone(baslangic);
    gorevReducer(baslangic, { type: "toggle", id: "1" });
    expect(baslangic).toEqual(kopya);
  });

  it("bilinmeyen action'da hata fırlatır", () => {
    expect(() => gorevReducer(baslangic, { type: "yok" })).toThrow();
  });
});
```

> Kritik iş mantığını (sepet hesapları, oyun kuralları, form akışları) reducer'a koymanın en büyük kazancı: hızlı, güvenilir, bileşenden bağımsız testler.

---

## 13. Reducer ile Durum Makinesi (State Machine)

Reducer yalnızca durumları değil, **durumlar arası geçişleri** de kontrol edebilir.

```
          ┌──── "getir" ────┐
          ▼                 │
  ┌──────────┐  "basarili" ┌──────────┐
  │yukleniyor│────────────►│ basarili │
  └──────────┘             └──────────┘
        │  "hata"                ▲
        ▼                        │ "getir"
  ┌──────────┐ "tekrar_dene"     │
  │   hata   │───────────────────┘ (yukleniyor'a döner)
  └──────────┘
```

```jsx
const baslangic = { durum: "yukleniyor", veri: null, hata: null };

function veriReducer(state, action) {
  switch (state.durum) {
    case "yukleniyor":
      switch (action.type) {
        case "basarili": return { durum: "basarili", veri: action.veri, hata: null };
        case "hata":     return { durum: "hata", veri: null, hata: action.mesaj };
        default:         return state;   // yüklenirken diğer action'lar yok sayılır
      }

    case "basarili":
    case "hata":
      switch (action.type) {
        case "getir":
        case "tekrar_dene":
          return { ...state, durum: "yukleniyor", hata: null };
        default:
          return state;
      }

    default:
      throw new Error(`Bilinmeyen durum: ${state.durum}`);
  }
}
```

- Yüklenirken tekrar "getir" gelirse ikinci yükleme başlamaz.
- Yükleme bitmişken yanlışlıkla gelen `"basarili"` yok sayılır.
- Geçiş kuralları kodda **açıkça** yazılıdır.

> Karmaşık akışlarda (ödeme adımları, sihirbaz formlar, oyunlar) çok değerlidir. Büyük durum makineleri için **XState** gibi kütüphaneler vardır.

---

## 14. Reducer + Context: Ölçeklenebilir State Yönetimi ⭐

- **Reducer:** State'in nasıl değiştiğini tek yerde toplar.
- **Context:** State'i ve `dispatch`'i ağacın derinliklerine props'suz iletir.

`dispatch` sabit olduğu için "veri ve işlemleri ayrı context'lere böl" önerisi burada çok doğaldır:

```jsx
// src/context/GorevlerContext.jsx
import { createContext, useContext, useReducer } from "react";
import { gorevReducer, BASLANGIC } from "./gorevReducer";

const GorevlerContext = createContext(null);
const GorevlerDispatchContext = createContext(null);

export function GorevlerProvider({ children }) {
  const [gorevler, dispatch] = useReducer(gorevReducer, BASLANGIC);

  return (
    <GorevlerContext value={gorevler}>
      <GorevlerDispatchContext value={dispatch}>
        {children}
      </GorevlerDispatchContext>
    </GorevlerContext>
  );
}

export function useGorevler() {
  const d = useContext(GorevlerContext);
  if (d === null) throw new Error("useGorevler, <GorevlerProvider> içinde kullanılmalı.");
  return d;
}

export function useGorevlerDispatch() {
  const d = useContext(GorevlerDispatchContext);
  if (d === null) throw new Error("useGorevlerDispatch, <GorevlerProvider> içinde kullanılmalı.");
  return d;
}
```

```jsx
// Herhangi bir derinlikte, props almadan:
function GorevEkle() {
  const dispatch = useGorevlerDispatch();   // gorevler değişince render OLMAZ
  const [metin, setMetin] = useState("");

  function handleSubmit(e) {
    e.preventDefault();
    dispatch({ type: "eklendi", id: crypto.randomUUID(), metin });
    setMetin("");
  }
  // ...
}

function GorevListesi() {
  const gorevler = useGorevler();           // gorevler değişince render olur
  return <ul>{gorevler.map((g) => <GorevSatiri key={g.id} gorev={g} />)}</ul>;
}

function GorevSatiri({ gorev }) {
  const dispatch = useGorevlerDispatch();
  return (
    <li>
      <input
        type="checkbox"
        checked={gorev.tamam}
        onChange={() => dispatch({ type: "toggle", id: gorev.id })}
      />
      {gorev.metin}
      <button onClick={() => dispatch({ type: "silindi", id: gorev.id })}>🗑️</button>
    </li>
  );
}
```

### Kazanımlar

- Tek bir `<GorevlerProvider>`; ara bileşenler hiçbir şey taşımaz.
- Görev mantığı `gorevReducer`'da, test edilebilir.
- Sadece `dispatch` kullanan bileşenler (form, butonlar) görevler değişince render olmaz.

### Action'ları gizlemek: niyet fonksiyonları

```jsx
const islemler = useMemo(() => ({
  ekle: (metin) => dispatch({ type: "eklendi", id: crypto.randomUUID(), metin }),
  toggle: (id) => dispatch({ type: "toggle", id }),
  sil: (id) => dispatch({ type: "silindi", id }),
}), []);   // dispatch sabit → islemler de sabit
```

Bileşenler `const { ekle } = useGorevIslemleri()` yazar; action yapısını bilmez. İki yaklaşım da yaygındır.

> 💡 Bu yapı, **Redux**'ın temel fikrinin (store, action'lar, reducer'lar) React'in kendi araçlarıyla küçük ölçekli bir uygulamasıdır.

---

## 15. Her Şey Bir Arada: Geri Alınabilir Görev Panosu

Geri al / yinele (undo / redo): `useState` ile zahmetli, reducer ile birkaç satır.

```jsx
// ============ src/pano/panoReducer.js ============
export const SUTUNLAR = [
  { id: "yapilacak", baslik: "📝 Yapılacak" },
  { id: "suruyor", baslik: "🚧 Sürüyor" },
  { id: "bitti", baslik: "✅ Bitti" },
];

export const BASLANGIC_KARTLAR = [
  { id: "k1", metin: "useReducer'ı öğren", sutun: "suruyor" },
  { id: "k2", metin: "14-usereducer.md'yi yaz", sutun: "yapilacak" },
  { id: "k3", metin: "Context API notları", sutun: "bitti" },
];

// --- Saf iş mantığı: sadece kartlar ---
export function kartReducer(kartlar, action) {
  switch (action.type) {
    case "kart_eklendi":
      return [...kartlar, { id: action.id, metin: action.metin, sutun: "yapilacak" }];

    case "kart_tasindi":
      return kartlar.map((k) => (k.id === action.id ? { ...k, sutun: action.hedef } : k));

    case "kart_duzenlendi":
      return kartlar.map((k) => (k.id === action.id ? { ...k, metin: action.metin } : k));

    case "kart_silindi":
      return kartlar.filter((k) => k.id !== action.id);

    case "bitenler_temizlendi":
      return kartlar.filter((k) => k.sutun !== "bitti");

    default:
      throw new Error(`Bilinmeyen action: ${action.type}`);
  }
}

// --- Geçmiş sarmalayıcısı: herhangi bir reducer'a undo/redo ekler ---
export function gecmisliReducer(icReducer) {
  return function (state, action) {
    const { gecmis, simdi, gelecek } = state;

    switch (action.type) {
      case "geri_al": {
        if (gecmis.length === 0) return state;
        return {
          gecmis: gecmis.slice(0, -1),
          simdi: gecmis.at(-1),
          gelecek: [simdi, ...gelecek],
        };
      }
      case "yinele": {
        if (gelecek.length === 0) return state;
        return {
          gecmis: [...gecmis, simdi],
          simdi: gelecek[0],
          gelecek: gelecek.slice(1),
        };
      }
      default: {
        const yeni = icReducer(simdi, action);
        if (yeni === simdi) return state;          // değişiklik yoksa geçmişe ekleme
        return {
          gecmis: [...gecmis, simdi].slice(-50),   // en fazla 50 adım
          simdi: yeni,
          gelecek: [],                             // yeni işlem, yinele geçmişini siler
        };
      }
    }
  };
}

export const panoReducer = gecmisliReducer(kartReducer);

export function baslangicOlustur(kartlar) {
  return { gecmis: [], simdi: kartlar, gelecek: [] };
}
```

```jsx
// ============ src/pano/PanoContext.jsx ============
import { createContext, useContext, useMemo, useReducer } from "react";
import { panoReducer, baslangicOlustur, BASLANGIC_KARTLAR } from "./panoReducer";

const PanoContext = createContext(null);
const PanoIslemleriContext = createContext(null);

export function PanoProvider({ children }) {
  const [state, dispatch] = useReducer(panoReducer, BASLANGIC_KARTLAR, baslangicOlustur);

  // Veri: değiştikçe yeni nesne
  const deger = useMemo(() => ({
    kartlar: state.simdi,
    geriAlinabilir: state.gecmis.length > 0,
    yinelenebilir: state.gelecek.length > 0,
  }), [state]);

  // İşlemler: dispatch sabit olduğu için hiç değişmez
  const islemler = useMemo(() => ({
    ekle: (metin) => dispatch({ type: "kart_eklendi", id: crypto.randomUUID(), metin }),
    tasi: (id, hedef) => dispatch({ type: "kart_tasindi", id, hedef }),
    duzenle: (id, metin) => dispatch({ type: "kart_duzenlendi", id, metin }),
    sil: (id) => dispatch({ type: "kart_silindi", id }),
    bitenleriTemizle: () => dispatch({ type: "bitenler_temizlendi" }),
    geriAl: () => dispatch({ type: "geri_al" }),
    yinele: () => dispatch({ type: "yinele" }),
  }), []);

  return (
    <PanoContext value={deger}>
      <PanoIslemleriContext value={islemler}>{children}</PanoIslemleriContext>
    </PanoContext>
  );
}

export function usePano() {
  const d = useContext(PanoContext);
  if (!d) throw new Error("usePano, <PanoProvider> içinde kullanılmalı.");
  return d;
}

export function usePanoIslemleri() {
  const d = useContext(PanoIslemleriContext);
  if (!d) throw new Error("usePanoIslemleri, <PanoProvider> içinde kullanılmalı.");
  return d;
}
```

```jsx
// ============ src/pano/Pano.jsx ============
import { useEffect, useState } from "react";
import { SUTUNLAR } from "./panoReducer";
import { PanoProvider, usePano, usePanoIslemleri } from "./PanoContext";

function KartEkle() {
  const { ekle } = usePanoIslemleri();   // kartlar değişince render OLMAZ
  const [metin, setMetin] = useState("");

  function handleSubmit(e) {
    e.preventDefault();
    if (!metin.trim()) return;
    ekle(metin.trim());
    setMetin("");
  }

  return (
    <form onSubmit={handleSubmit}>
      <input value={metin} onChange={(e) => setMetin(e.target.value)} placeholder="Yeni kart..." />
      <button type="submit">Ekle</button>
    </form>
  );
}

function Kart({ kart }) {
  const { tasi, duzenle, sil } = usePanoIslemleri();
  const [duzenleniyor, setDuzenleniyor] = useState(false);   // yerel UI state'i
  const [taslak, setTaslak] = useState(kart.metin);

  const sutunIndex = SUTUNLAR.findIndex((s) => s.id === kart.sutun);
  const onceki = SUTUNLAR[sutunIndex - 1];
  const sonraki = SUTUNLAR[sutunIndex + 1];

  function handleKaydet(e) {
    e.preventDefault();
    if (taslak.trim() && taslak !== kart.metin) duzenle(kart.id, taslak.trim());
    setDuzenleniyor(false);
  }

  return (
    <li className="kart">
      {duzenleniyor ? (
        <form onSubmit={handleKaydet}>
          <input
            autoFocus
            value={taslak}
            onChange={(e) => setTaslak(e.target.value)}
            onBlur={handleKaydet}
          />
        </form>
      ) : (
        <span onDoubleClick={() => { setTaslak(kart.metin); setDuzenleniyor(true); }}>
          {kart.metin}
        </span>
      )}
      <div className="kart-butonlar">
        {onceki && (
          <button type="button" onClick={() => tasi(kart.id, onceki.id)} aria-label="Sola taşı">◀</button>
        )}
        {sonraki && (
          <button type="button" onClick={() => tasi(kart.id, sonraki.id)} aria-label="Sağa taşı">▶</button>
        )}
        <button type="button" onClick={() => sil(kart.id)} aria-label="Sil">🗑️</button>
      </div>
    </li>
  );
}

function Sutun({ sutun }) {
  const { kartlar } = usePano();
  const buSutundakiler = kartlar.filter((k) => k.sutun === sutun.id);   // türetilmiş

  return (
    <section className="sutun">
      <h2>{sutun.baslik} ({buSutundakiler.length})</h2>
      {buSutundakiler.length === 0 ? (
        <p className="bos">Boş</p>
      ) : (
        <ul>{buSutundakiler.map((k) => <Kart key={k.id} kart={k} />)}</ul>
      )}
    </section>
  );
}

function AracCubugu() {
  const { kartlar, geriAlinabilir, yinelenebilir } = usePano();
  const { geriAl, yinele, bitenleriTemizle } = usePanoIslemleri();
  const bitenVar = kartlar.some((k) => k.sutun === "bitti");

  // Klavye kısayolları: Ctrl/Cmd+Z, Ctrl/Cmd+Shift+Z
  useEffect(() => {
    function handleKeyDown(e) {
      if (!(e.ctrlKey || e.metaKey) || e.key.toLowerCase() !== "z") return;
      if (e.target.tagName === "INPUT") return;
      e.preventDefault();
      e.shiftKey ? yinele() : geriAl();
    }
    window.addEventListener("keydown", handleKeyDown);
    return () => window.removeEventListener("keydown", handleKeyDown);
  }, [geriAl, yinele]);   // ikisi de sabit; effect bir kez kurulur

  return (
    <div className="arac-cubugu">
      <button type="button" onClick={geriAl} disabled={!geriAlinabilir}>↶ Geri al</button>
      <button type="button" onClick={yinele} disabled={!yinelenebilir}>↷ Yinele</button>
      <button type="button" onClick={bitenleriTemizle} disabled={!bitenVar}>Bitenleri temizle</button>
    </div>
  );
}

export default function Pano() {
  return (
    <PanoProvider>
      <h1>Görev Panosu</h1>
      <AracCubugu />
      <KartEkle />
      <div
        className="sutunlar"
        style={{ display: "grid", gridTemplateColumns: "repeat(3, 1fr)", gap: 16 }}
      >
        {SUTUNLAR.map((s) => <Sutun key={s.id} sutun={s} />)}
      </div>
      <small>İpucu: Karta çift tıklayarak düzenle · Ctrl+Z / Ctrl+Shift+Z</small>
    </PanoProvider>
  );
}
```

### Bu örnekte kullanılanlar

- Tüm iş mantığı saf reducer'da (`kartReducer`)
- "Ne oldu" tarzı action'lar
- Rastgele id reducer dışında üretilip action'a veri olarak konuyor
- **Reducer'ı saran reducer** ile geri al/yinele (`gecmisliReducer` herhangi bir reducer'a uygulanabilir)
- `init` fonksiyonu ile başlangıç state'i
- Reducer + context ile ölçeklenebilir yapı
- Veri ve işlemler için ayrı context'ler (`dispatch` sabit → işlemler hiç değişmez)
- Bileşenler action yapısını bilmeden niyet fonksiyonları kullanıyor
- Yerel UI state'i için hâlâ `useState` (`duzenleniyor`, `taslak`)
- Türetilmiş değerler (`buSutundakiler`, `bitenVar`)
- Sabit işlemler sayesinde bir kez kurulan klavye kısayolu effect'i

---

## 16. Özet

- **Action** "ne oldu?", **dispatch** action'ı reducer'a götüren postacı, **reducer** "bu olunca state ne olmalı?" sorusunun cevabıdır.
- `useReducer`, state güncelleme mantığını bileşen dışında **tek bir saf fonksiyonda** toplar.
- `const [state, dispatch] = useReducer(reducer, baslangic, init?)`
- Olay yöneticileri state'i değiştirmez, **action gönderir.**
- Action'lar **ne olduğunu** anlatır (`"silindi"`), nasıl değişeceğini değil (`"set_..."`); tek etkileşim = tek action.
- Reducer **saf** olmalı: yan etki, rastgelelik ve zaman yok; bunlar action'a veri olarak konur.
- Reducer **mutasyon yapmaz**, yeni state döndürür; bilinmeyen action için hata fırlatır.
- Birbirine bağlı state'ler tek reducer'da tutarlı biçimde güncellenir; türetilmiş değerler yine state'e konmaz.
- Basit state → `useState`; çok yönlü veya birbirine bağlı state → `useReducer`. İkisi birlikte kullanılabilir.
- Pahalı başlangıç için üçüncü argüman olan **`init`** fonksiyonu kullanılır.
- **`dispatch` sabittir**; state yine bir anlık görüntüdür, action'lar kuyrukta sırayla uygulanır.
- Derin state'lerde **Immer** (`useImmerReducer`) reducer'ı kısaltır.
- Reducer'lar React'ten bağımsız, kolayca **test edilir.**
- Reducer ile **durum makineleri** kurulabilir; geçerli geçişler kodda açıkça yazılır.
- **Reducer + context**: state ve `dispatch` ayrı context'lerde sağlanır; ölçeklenebilir, Redux benzeri bir yapı oluşur.
- Reducer'ları saran reducer'larla **geri al / yinele** gibi özellikler kolayca eklenir.

---

⬅️ Önceki: [13 · Context API](./13-context-api.md) · ➡️ Sonraki: [15 · Custom Hook'lar](./15-custom-hooks.md)
