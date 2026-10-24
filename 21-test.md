# 21 · Test

> **Konu özeti:** Otomatik testin amacı ve test türleri, "kullanıcı gibi test et" ilkesi, Vitest + Testing Library kurulumu, saf fonksiyon testleri, eşleştiriciler, bileşen render'ı ve sorgu öncelikleri, `user-event` ile etkileşim, asenkron testler, mock'lama (fonksiyonlar, modüller, zamanlayıcılar, MSW ile ağ), provider'larla özel `render`, router testleri, Hook ve reducer testleri, form/Action testleri, snapshot'lar, kapsam, Playwright ile E2E ve neyin test edilmesi gerektiği.
> **Sürüm:** Vitest 5.x (Eylül 2026)

---

## İçindekiler

1. [Sezgisel Bakış](#1-sezgisel-bakış)
2. [Araçlar ve Kurulum](#2-araçlar-ve-kurulum)
3. [İlk Test: Saf Bir Fonksiyon](#3-ilk-test-saf-bir-fonksiyon)
4. [Eşleştiriciler (Matchers)](#4-eşleştiriciler-matchers)
5. [Bileşen Testi: Render ve Sorgular](#5-bileşen-testi-render-ve-sorgular-)
6. [Kullanıcı Etkileşimi: `user-event`](#6-kullanıcı-etkileşimi-user-event)
7. [Asenkron Davranışlar](#7-asenkron-davranışları-test-etmek)
8. [Mock'lamak](#8-mocklamak-fonksiyonlar-zamanlayıcılar-ve-ağ)
9. [Provider'larla Test: Özel `render`](#9-providerlarla-test-özel-render-)
10. [Hook, Reducer ve Context Testleri](#10-hookları-reducerları-ve-contexti-test-etmek)
11. [Formlar ve React 19 Action'ları](#11-formları-ve-react-19-actionlarını-test-etmek)
12. [Snapshot Testleri](#12-snapshot-testleri-ölçülü-kullan)
13. [Kapsam ve E2E Testleri](#13-kapsam-coverage-ve-e2e-testleri)
14. [Neyi Test Etmeli?](#14-neyi-test-etmeli-neyi-etmemeli)
15. [Her Şey Bir Arada: Görev Uygulamasının Test Takımı](#15-her-şey-bir-arada-görev-uygulamasının-test-takımı)
16. [Özet](#16-özet)

---

## 1. Sezgisel Bakış

**Elle test** (tarayıcıda tıkla, bak) küçük projelerde işe yarar; ama her değişiklikte eski özellikleri tek tek denemek gerekir, kenar durumlar unutulur, başkasının neyi bozduğu bilinmez.

**Otomatik test** = "tıkla ve bak" işini kodla yazmak. Bir kez yaz, her değişiklikte saniyeler içinde yüzlerce kontrol çalıştır.

### Test türleri (araba benzetmesi)

| Tür | Benzetme | React'te |
|---|---|---|
| **Statik analiz** | Montajdan önce parçaların ölçüsü | TypeScript, ESLint |
| **Birim (unit)** | Fren balatası doğru sürtünüyor mu? | Fonksiyon, reducer, Hook |
| **Entegrasyon** | Pedala basınca araba duruyor mu? | Bileşeni alt bileşenleri ve state'iyle render edip kullanıcı gibi etkileşmek |
| **Uçtan uca (E2E)** | Gerçek yolda gerçek sürüş | Gerçek tarayıcı + sunucu: giriş → ürün ekle → ödeme |

> **"Test kupası" yaklaşımı:** En çok **entegrasyon testi** yaz. Birim testleri hızlı ama gerçek kullanımı az yansıtır; E2E gerçekçi ama yavaş ve kırılgan.

### Altın kural

> **"Testlerin yazılımın kullanılma şekline ne kadar benzerse, sana o kadar çok güven verir."**

- Bileşenin **iç yapısını** (state adı, Hook, `div` sayısı) değil, **kullanıcının gördüğünü ve yaptığını** test et.
- İç yapı değişse bile davranış aynıysa testler geçmeye devam eder.

---

## 2. Araçlar ve Kurulum

| Araç | Görevi |
|---|---|
| **Vitest** | Test çalıştırıcı; `expect` ve mock araçları |
| **jsdom** (veya happy-dom) | Node.js içinde sahte tarayıcı DOM'u |
| **React Testing Library** | Bileşenleri render etmek, ekranda **kullanıcı gibi** eleman bulmak |
| **user-event** | Gerçekçi kullanıcı etkileşimleri |
| **jest-dom** | DOM'a özel eşleştiriciler (`toBeInTheDocument`, `toBeDisabled`...) |
| **MSW** | Ağ isteklerini taklit etmek |
| **Playwright** | Gerçek tarayıcıda E2E |

### Neden Vitest?

- Vite ile aynı yapılandırma ve dönüştürme hattı: JSX, TypeScript, CSS importları ek ayarsız çalışır.
- Jest ile neredeyse aynı API.
- **Vitest 5.0** — 3 Eylül 2026; performans odaklı; **Node.js 22.12.0+ ve Vite 6.4.0+** gerektirir.

### Kurulum

```bash
npm install -D vitest jsdom @testing-library/react @testing-library/dom @testing-library/user-event @testing-library/jest-dom
```

```js
// vite.config.js
/// <reference types="vitest/config" />
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";

export default defineConfig({
  plugins: [react()],
  test: {
    environment: "jsdom",               // sahte tarayıcı
    globals: true,                      // describe/it/expect/vi import etmeden
    setupFiles: "./src/test/kurulum.js",
    css: true,                          // CSS importlarını işle
  },
});
```

```js
// src/test/kurulum.js — her test dosyasından önce çalışır
import "@testing-library/jest-dom/vitest";
```

```json
// package.json
{
  "scripts": {
    "test": "vitest",
    "test:run": "vitest run",
    "test:coverage": "vitest run --coverage"
  }
}
```

| Komut | Davranış |
|---|---|
| `npm test` | **İzleme modu**: dosya kaydettikçe ilgili testler anında çalışır |
| `npm run test:run` | Bir kez çalıştır ve çık (CI) |

### Test dosyalarının yeri

`.test.js(x)` veya `.spec.js(x)` uzantılı dosyalar; genelde test edilen dosyanın **yanına**:

```
src/
├── components/
│   ├── Sayac.jsx
│   └── Sayac.test.jsx
├── utils/
│   ├── fiyat.js
│   └── fiyat.test.js
```

> 💡 VS Code **Vitest** eklentisi: testleri editörden tek tıkla çalıştırır.

---

## 3. İlk Test: Saf Bir Fonksiyon

```js
// src/utils/fiyat.js
export function fiyatFormatla(tutar) {
  return `${tutar.toLocaleString("tr-TR", { minimumFractionDigits: 2, maximumFractionDigits: 2 })} ₺`;
}

export function indirimUygula(fiyat, yuzde) {
  if (yuzde < 0 || yuzde > 100) throw new Error("Geçersiz indirim oranı");
  return Math.round(fiyat * (1 - yuzde / 100) * 100) / 100;
}
```

```js
// src/utils/fiyat.test.js
import { describe, it, expect } from "vitest";   // globals: true ise opsiyonel
import { fiyatFormatla, indirimUygula } from "./fiyat";

describe("fiyatFormatla", () => {
  it("Türk lirası biçiminde formatlar", () => {
    expect(fiyatFormatla(1299.9)).toBe("1.299,90 ₺");
  });

  it("tam sayılara kuruş ekler", () => {
    expect(fiyatFormatla(50)).toBe("50,00 ₺");
  });
});

describe("indirimUygula", () => {
  it("yüzde indirimi uygular", () => {
    expect(indirimUygula(200, 25)).toBe(150);
  });

  it("geçersiz oranda hata fırlatır", () => {
    expect(() => indirimUygula(100, 150)).toThrow("Geçersiz indirim oranı");
  });
});
```

| Parça | Görevi |
|---|---|
| `describe(ad, fn)` | İlgili testleri gruplar |
| `it(ad, fn)` / `test(ad, fn)` | Tek bir test; adı davranışı bir cümle gibi anlatmalı |
| `expect(gercek).eslestirici(beklenen)` | Kontrol (assertion) |

> 💡 **AAA kalıbı:** **Arrange** (hazırla) → **Act** (eylem) → **Assert** (kontrol).

---

## 4. Eşleştiriciler (Matchers)

### Vitest temel eşleştiricileri

| Eşleştirici | Ne kontrol eder | Örnek |
|---|---|---|
| `toBe(x)` | Tam eşitlik (`Object.is`); primitive'ler | `expect(2 + 2).toBe(4)` |
| `toEqual(x)` | **İçerik** eşitliği; nesne/dizi | `expect({ a: 1 }).toEqual({ a: 1 })` |
| `toStrictEqual(x)` | `toEqual` + `undefined` alanlar ve tipler | |
| `toBeTruthy()` / `toBeFalsy()` | Doğru/yanlış sayılan değer | |
| `toBeNull()`, `toBeUndefined()`, `toBeDefined()` | | |
| `toContain(x)` | Dizi/string içeriyor mu | `expect(["a", "b"]).toContain("a")` |
| `toHaveLength(n)` | Uzunluk | |
| `toMatch(/regex/)` | Desen | |
| `toMatchObject(x)` | Nesnenin bir kısmı eşleşiyor mu | |
| `toBeGreaterThan(n)`, `toBeCloseTo(n)` | Sayısal karşılaştırma | |
| `toThrow(mesaj?)` | Hata fırlatıyor mu | `expect(() => f()).toThrow()` |
| `toHaveBeenCalled()`, `toHaveBeenCalledWith(...)`, `toHaveBeenCalledTimes(n)` | Mock çağrıları | |
| `.not` | Tersini kontrol eder | `expect(x).not.toBe(0)` |
| `.resolves` / `.rejects` | Promise sonucu | `await expect(p).resolves.toEqual(...)` |

> ⚠️ `expect({ a: 1 }).toBe({ a: 1 })` **başarısız** (farklı nesneler); `toEqual` başarılı.

### jest-dom eşleştiricileri

| Eşleştirici | Ne kontrol eder |
|---|---|
| `toBeInTheDocument()` | Eleman sayfada var mı |
| `toBeVisible()` | Görünür mü |
| `toBeDisabled()` / `toBeEnabled()` | Devre dışı mı |
| `toHaveValue(x)` | Input/select değeri |
| `toBeChecked()` | Checkbox/radio işaretli mi |
| `toHaveTextContent(x)` | Metin içeriği (string/regex) |
| `toHaveAttribute(ad, deger?)` | Nitelik |
| `toHaveClass(...)` | CSS sınıfı |
| `toHaveFocus()` | Odakta mı |
| `toHaveAccessibleName(x)` | Ekran okuyucunun okuyacağı isim |
| `toBeInvalid()` | `aria-invalid` veya geçersiz alan |

---

## 5. Bileşen Testi: Render ve Sorgular ⭐

```jsx
// src/components/Sayac.jsx
import { useState } from "react";

export default function Sayac({ baslangic = 0 }) {
  const [sayi, setSayi] = useState(baslangic);
  return (
    <div>
      <p>Değer: {sayi}</p>
      <button onClick={() => setSayi((s) => s + 1)}>Artır</button>
      <button onClick={() => setSayi(baslangic)} disabled={sayi === baslangic}>
        Sıfırla
      </button>
    </div>
  );
}
```

```jsx
// src/components/Sayac.test.jsx
import { render, screen } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import Sayac from "./Sayac";

describe("Sayac", () => {
  it("başlangıç değerini gösterir", () => {
    render(<Sayac baslangic={5} />);
    expect(screen.getByText("Değer: 5")).toBeInTheDocument();
  });

  it("Artır'a tıklayınca değer bir artar", async () => {
    const user = userEvent.setup();
    render(<Sayac />);

    await user.click(screen.getByRole("button", { name: "Artır" }));
    await user.click(screen.getByRole("button", { name: "Artır" }));

    expect(screen.getByText("Değer: 2")).toBeInTheDocument();
  });

  it("Sıfırla başta devre dışı, değer değişince etkin olur", async () => {
    const user = userEvent.setup();
    render(<Sayac />);
    const sifirla = screen.getByRole("button", { name: "Sıfırla" });

    expect(sifirla).toBeDisabled();
    await user.click(screen.getByRole("button", { name: "Artır" }));
    expect(sifirla).toBeEnabled();

    await user.click(sifirla);
    expect(screen.getByText("Değer: 0")).toBeInTheDocument();
  });
});
```

- **`render(<Bilesen />)`**: jsdom'a render eder; her testten sonra otomatik temizlenir.
- **`screen`**: Sayfadaki elemanları bulmak için sorgular.
- Test `useState`'ten veya `sayi`'dan **hiç bahsetmiyor**; sadece görünenlere bakıyor.

### Sorgu türleri

| Önek | Bulamazsa | Birden çok bulursa | Ne zaman |
|---|---|---|---|
| `getBy...` | **Hata** | Hata | Elemanın **var olmasını** beklerken |
| `queryBy...` | `null` | Hata | Elemanın **olmadığını** kontrol ederken |
| `findBy...` | Bekler, süre dolunca hata | Hata | Eleman **asenkron** gelecekse (`await`) |
| `getAllBy` / `queryAllBy` / `findAllBy` | | Dizi | Birden çok eleman |

```jsx
expect(screen.queryByText("Hata")).not.toBeInTheDocument();
```

### Sorgu öncelik sırası ⭐

| Öncelik | Sorgu | Örnek |
|---|---|---|
| 1 | **`getByRole`** | `getByRole("button", { name: "Kaydet" })`, `getByRole("heading", { level: 1 })` |
| 2 | `getByLabelText` | `getByLabelText("Şifre")` |
| 3 | `getByPlaceholderText` | Etiket yoksa |
| 4 | `getByText` | Etkileşimsiz metinler |
| 5 | `getByDisplayValue` | Input'un mevcut değeri |
| 6 | `getByAltText` | Görseller |
| 7 | `getByTitle` | `title` niteliği |
| 8 | `getByTestId` | **Son çare** (`data-testid`) |

### `getByRole` neden ilk sırada?

Ekran okuyucular sayfayı **roller** üzerinden gezer. `getByRole` ile bulunamayan eleman büyük ihtimalle erişilebilir değildir → **test yazmak erişilebilirliği de test eder.**

```jsx
// ❌ getByRole("button") ile BULUNAMAZ
<div onClick={handleSil}>Sil</div>

// ✅
<button onClick={handleSil}>Sil</button>

// ✅ İkon butonu
<button aria-label="Pencereyi kapat">✕</button>
// screen.getByRole("button", { name: "Pencereyi kapat" })
```

### Yaygın roller

| HTML | Rol |
|---|---|
| `<button>` | `button` |
| `<a href>` | `link` |
| `<h1>`–`<h6>` | `heading` (`{ level: 2 }`) |
| `<input type="text">`, `<textarea>` | `textbox` |
| `<input type="checkbox">` | `checkbox` |
| `<input type="radio">` | `radio` |
| `<select>` | `combobox` |
| `<ul>` / `<li>` | `list` / `listitem` |
| `<img alt>` | `img` |
| `role="alert"` / `role="status"` | `alert` / `status` |
| `<nav>` | `navigation` |
| `<dialog>` | `dialog` |

> 💡 `screen.debug()` → HTML'i yazdırır; `screen.logTestingPlaygroundURL()` → sorgu önerileri. `getByRole` başarısız olunca hata mesajı sayfadaki tüm rolleri listeler.

### `within`: Belirli bir alanda aramak

```jsx
import { within } from "@testing-library/react";

const sepet = screen.getByRole("region", { name: "Sepet" });
expect(within(sepet).getAllByRole("listitem")).toHaveLength(2);
```

---

## 6. Kullanıcı Etkileşimi: `user-event`

```jsx
import userEvent from "@testing-library/user-event";

it("...", async () => {
  const user = userEvent.setup();   // her testin başında
  render(<Form />);

  await user.click(eleman);
  await user.dblClick(eleman);
  await user.type(input, "Merhaba");        // harf harf yazar
  await user.clear(input);
  await user.selectOptions(select, "ankara");
  await user.keyboard("{Enter}");           // {Escape}, {Tab}, {ArrowDown}, {Shift>}A{/Shift}
  await user.tab();
  await user.hover(eleman);
  await user.upload(dosyaInput, new File(["içerik"], "rapor.pdf", { type: "application/pdf" }));
});
```

- Tüm etkileşimler **async**; `await` gerekir.
- `user.type`, gerçek kullanıcı gibi önce tıklar, her harf için `keydown`/`input`/`keyup` tetikler.

> 📌 Eski ve daha düşük seviyeli **`fireEvent`** tek bir DOM olayı tetikler; `user-event` gerçek davranışı taklit eder. Varsayılan tercih **`user-event`**.

---

## 7. Asenkron Davranışları Test Etmek

```jsx
// Eleman GELECEK: findBy (varsayılan 1 sn bekler)
expect(await screen.findByText("Ayşe Yılmaz")).toBeInTheDocument();

// Bir şeyin OLMASINI bekle
await waitFor(() => {
  expect(screen.getByRole("button", { name: "Kaydet" })).toBeEnabled();
});

// Eleman KAYBOLACAK
await waitForElementToBeRemoved(() => screen.queryByText("Yükleniyor..."));
```

```jsx
it("kullanıcıyı yükler ve gösterir", async () => {
  render(<KullaniciProfili id={1} />);

  expect(screen.getByText("Yükleniyor...")).toBeInTheDocument();
  expect(await screen.findByRole("heading", { name: "Leanne Graham" })).toBeInTheDocument();
  expect(screen.queryByText("Yükleniyor...")).not.toBeInTheDocument();
});
```

### Kurallar

- Asenkron bekleme için en okunaklısı **`findBy`.**
- `waitFor` içinde **yan etki yapılmaz** (tıklama, yazma); sadece `expect`.
- **`await` unutulmaz.** Vitest 5, `await` edilmemiş asenkron kontrolleri olan testleri **başarısız** sayar.

> 💡 **`act(...)` uyarısı:** Test bittikten sonra state güncellemesi olduğunu gösterir. Çözüm neredeyse her zaman sonucu `findBy`/`waitFor` ile **beklemektir**; `render` ve `user-event` zaten `act` kullanır.

---

## 8. Mock'lamak: Fonksiyonlar, Zamanlayıcılar ve Ağ

**Mock** = gerçek bağımlılığın yerine geçen sahte sürüm. Testler **hızlı**, **tekrarlanabilir** ve **dış dünyadan bağımsız** olmalı.

### Mock fonksiyonlar: `vi.fn()`

```jsx
it("Sil'e tıklanınca onSil'i ürün id'siyle çağırır", async () => {
  const user = userEvent.setup();
  const onSil = vi.fn();

  render(<SepetSatiri urun={{ id: 7, ad: "Kalem" }} onSil={onSil} />);
  await user.click(screen.getByRole("button", { name: /sil/i }));

  expect(onSil).toHaveBeenCalledTimes(1);
  expect(onSil).toHaveBeenCalledWith(7);
});
```

```js
const fn = vi.fn();
const fn = vi.fn(() => 42);                  // dönüş değeriyle
fn.mockReturnValue(10);
fn.mockResolvedValue({ id: 1 });             // Promise.resolve
fn.mockRejectedValue(new Error("Hata"));     // Promise.reject
```

### Var olan fonksiyonu izlemek: `vi.spyOn`

```js
const uyari = vi.spyOn(console, "error").mockImplementation(() => {});
// ... test
expect(uyari).toHaveBeenCalled();
uyari.mockRestore();
```

### Modülleri mock'lamak: `vi.mock`

```js
import { kayitOl } from "../api";

vi.mock("../api", () => ({
  kayitOl: vi.fn(),
}));

it("...", async () => {
  kayitOl.mockResolvedValue({ id: "1", ad: "Ayşe" });
  // ...
});
```

> ⚠️ **Vitest 5**: mock'lar her testten önce varsayılan olarak **sıfırlanır**; mock davranışı her testte (veya `beforeEach`'te) yeniden ayarlanmalıdır.

### Sahte zamanlayıcılar

```jsx
it("bildirim 3 saniye sonra kaybolur", async () => {
  vi.useFakeTimers();
  const user = userEvent.setup({ advanceTimers: vi.advanceTimersByTime });

  render(<BildirimDemo />);
  await user.click(screen.getByRole("button", { name: "Kaydet" }));
  expect(screen.getByRole("status")).toHaveTextContent("Kaydedildi");

  await vi.advanceTimersByTimeAsync(3000);   // zamanı ileri sar
  expect(screen.queryByRole("status")).not.toBeInTheDocument();

  vi.useRealTimers();
});
```

### Ağ istekleri: MSW ⭐

- `fetch`'i doğrudan mock'lamak kırılgandır.
- **MSW** ağ düzeyinde araya girer: kod gerçek `fetch` yapar, MSW cevabı taklit eder.

```bash
npm install -D msw
```

```js
// src/test/sunucu.js
import { http, HttpResponse } from "msw";
import { setupServer } from "msw/node";

export const handlers = [
  http.get("https://jsonplaceholder.typicode.com/users/:id", ({ params }) => {
    return HttpResponse.json({ id: Number(params.id), name: "Leanne Graham", email: "leanne@ornek.com" });
  }),

  http.post("/api/kayit", async ({ request }) => {
    const govde = await request.json();
    return HttpResponse.json({ id: "yeni-1", ...govde }, { status: 201 });
  }),
];

export const sunucu = setupServer(...handlers);
```

```js
// src/test/kurulum.js
import "@testing-library/jest-dom/vitest";
import { sunucu } from "./sunucu";

beforeAll(() => sunucu.listen({ onUnhandledRequest: "error" }));   // tanımsız istek = hata
afterEach(() => sunucu.resetHandlers());                            // testlerin özel ayarlarını temizle
afterAll(() => sunucu.close());
```

#### Test başına farklı cevap

```jsx
import { http, HttpResponse } from "msw";
import { sunucu } from "../test/sunucu";

it("sunucu hatasında hata mesajı gösterir", async () => {
  sunucu.use(
    http.get("https://jsonplaceholder.typicode.com/users/:id", () =>
      new HttpResponse(null, { status: 500 })
    )
  );

  render(<KullaniciProfili id={1} />);
  expect(await screen.findByRole("alert")).toHaveTextContent(/hata/i);
});
```

> Aynı handler'lar **tarayıcıda** da (backend hazır değilken geliştirme sırasında) kullanılabilir.

---

## 9. Provider'larla Test: Özel `render` ⭐

```jsx
// src/test/yardimcilar.jsx
import { render } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { TemaProvider } from "../context/TemaContext";

function testQueryClient() {
  return new QueryClient({
    defaultOptions: {
      queries: { retry: false, gcTime: Infinity },
      mutations: { retry: false },
    },
  });
}

export function renderUygulama(ui, { queryClient = testQueryClient(), ...secenekler } = {}) {
  function Sarmalayici({ children }) {
    return (
      <QueryClientProvider client={queryClient}>
        <TemaProvider>{children}</TemaProvider>
      </QueryClientProvider>
    );
  }
  return {
    user: userEvent.setup(),
    queryClient,
    ...render(ui, { wrapper: Sarmalayici, ...secenekler }),
  };
}

export * from "@testing-library/react";
```

```jsx
import { renderUygulama, screen } from "../test/yardimcilar";

it("...", async () => {
  const { user } = renderUygulama(<UrunKatalogu />);
  // ...
});
```

- **Her test için yeni `QueryClient`**: önbellek testler arasında sızmasın.
- **`retry: false`**: hata testleri yeniden denemelerle zaman aşımına uğramasın.

### React Router ile test

```jsx
import { createMemoryRouter, RouterProvider } from "react-router";
import { render, screen } from "@testing-library/react";
import NotDetay, { loader as notLoader } from "./NotDetay";

it("URL'deki nota ait başlığı gösterir", async () => {
  const router = createMemoryRouter(
    [{ path: "/notlar/:notId", Component: NotDetay, loader: notLoader }],
    { initialEntries: ["/notlar/1"] }
  );

  render(<RouterProvider router={router} />);
  expect(await screen.findByRole("heading", { name: "React Router" })).toBeInTheDocument();
});
```

- `createMemoryRouter`: bellekte geçmiş tutar; `initialEntries` ile başlangıç URL'si.
- Yönlendirme sonrası: `router.state.location.pathname`.
- `createRoutesStub`: özellikle Framework modunda rota modüllerini test etmek için.

---

## 10. Hook'ları, Reducer'ları ve Context'i Test Etmek

### Reducer'lar: saf fonksiyon

```js
import { gorevReducer } from "./gorevReducer";

it("görev ekler", () => {
  const sonuc = gorevReducer([], { type: "eklendi", id: "1", metin: "Test yaz" });
  expect(sonuc).toEqual([{ id: "1", metin: "Test yaz", tamam: false }]);
});
```

### Custom Hook'lar: `renderHook`

```jsx
import { renderHook, act } from "@testing-library/react";
import { useToggle } from "./useToggle";

describe("useToggle", () => {
  it("başlangıç değeriyle başlar ve tersine çevirir", () => {
    const { result } = renderHook(() => useToggle(false));
    expect(result.current[0]).toBe(false);

    act(() => result.current[1]());
    expect(result.current[0]).toBe(true);
  });
});
```

- `result.current`: Hook'un **en son** döndürdüğü değer.
- State güncelleyen çağrılar **`act`** ile sarılır.

### Prop değişimi ve zaman

```jsx
import { useDebounce } from "./useDebounce";

it("değeri gecikmeyle günceller", async () => {
  vi.useFakeTimers();
  const { result, rerender } = renderHook(({ deger }) => useDebounce(deger, 500), {
    initialProps: { deger: "a" },
  });

  rerender({ deger: "ab" });
  expect(result.current).toBe("a");          // henüz değişmedi

  await act(() => vi.advanceTimersByTimeAsync(500));
  expect(result.current).toBe("ab");
  vi.useRealTimers();
});
```

### Provider gerektiren Hook'lar

```jsx
renderHook(() => useTema(), { wrapper: TemaProvider });
```

> 💡 Hook'u kullanan bileşen üzerinden test etmek çoğu zaman daha anlamlıdır; `renderHook`'u tekrar kullanılabilir, mantığı yoğun Hook'lar için sakla.

---

## 11. Formları ve React 19 Action'larını Test Etmek

```jsx
it("geçersiz e-postada hata gösterir, geçerlide kaydeder", async () => {
  const user = userEvent.setup();
  render(<KayitFormu />);

  const email = screen.getByLabelText("E-posta");
  const gonder = screen.getByRole("button", { name: "Kayıt ol" });

  // 1. Hatalı gönderim
  await user.type(email, "gecersiz");
  await user.click(gonder);
  expect(await screen.findByText("Geçerli bir e-posta girin.")).toBeInTheDocument();
  expect(email).toHaveValue("gecersiz");   // yazılan kaybolmadı

  // 2. Doğru gönderim
  await user.clear(email);
  await user.type(email, "ayse@ornek.com");
  await user.type(screen.getByLabelText("Şifre"), "gizli12345");
  await user.click(gonder);

  expect(await screen.findByRole("button", { name: "Kaydediliyor..." })).toBeDisabled();
  expect(await screen.findByText(/hoş geldin/i)).toBeInTheDocument();
});
```

> Test; `useActionState`, `useFormStatus` veya `onSubmit` + `useState` hangisi kullanılırsa kullanılsın **aynı kalır.**

---

## 12. Snapshot Testleri (Ölçülü Kullan)

```jsx
it("rozeti doğru render eder", () => {
  const { container } = render(<Rozet renk="yesil">Aktif</Rozet>);
  expect(container.firstChild).toMatchInlineSnapshot(`
    <span class="rozet rozet-yesil">Aktif</span>
  `);
});
```

### Tuzaklar

- Büyük snapshot'lar gerçekten okunmaz.
- Küçük değişikliklerde kırılır; "güncelle" deyip geçilince test bir şey kontrol etmemiş olur.

> **Kural:** Küçük, kararlı çıktılar için `toMatchInlineSnapshot`; büyük ağaçlar için davranış testleri.

---

## 13. Kapsam (Coverage) ve E2E Testleri

### Kod kapsamı

```bash
npm install -D @vitest/coverage-v8
npm run test:coverage
```

- `coverage/index.html` raporu: hangi satır/dal/fonksiyonun çalıştırıldığı.

> ⚠️ **%100 kapsamı hedefleme.** Kapsam bir satırın **çalıştığını** söyler, **doğru test edildiğini** değil. Test edilmemiş önemli bölgeleri bulmak için bir **harita** olarak kullan.

### Uçtan uca: Playwright

```bash
npm init playwright@latest
```

```js
// e2e/sepet.spec.js
import { test, expect } from "@playwright/test";

test("kullanıcı ürünü sepete ekleyip ödemeye geçebilir", async ({ page }) => {
  await page.goto("http://localhost:5173/");
  await page.getByRole("link", { name: "Ürünler" }).click();
  await page.getByRole("button", { name: "Sepete ekle" }).first().click();
  await expect(page.getByRole("status")).toContainText("sepete eklendi");

  await page.getByRole("link", { name: /sepet/i }).click();
  await expect(page).toHaveURL(/\/sepet/);
  await expect(page.getByRole("listitem")).toHaveCount(1);
});
```

- Sorgu felsefesi aynı (`getByRole`).
- E2E testleri **kritik kullanıcı yolculukları** (kayıt, giriş, satın alma) için sınırlı tutulur.

> 📌 **Vitest Browser Mode**: bileşen testlerini jsdom yerine **gerçek tarayıcıda** çalıştırır (gerçek layout, CSS, `IntersectionObserver`, View Transitions); Vitest 5 ile yerleşik **Trace View** de eklendi.

---

## 14. Neyi Test Etmeli, Neyi Etmemeli?

### ✅ Test et

- Kullanıcının **gördüğü** sonuçlar: metinler, buton durumları, hata mesajları, listeler
- Kullanıcı **etkileşimleri** → beklenen sonuç
- **Kenar durumlar**: boş liste, yükleniyor, hata, yetkisiz kullanıcı, uzun metin
- Kritik **iş mantığı**: fiyat, indirim, doğrulama, reducer'lar
- **Erişilebilirlik**: roller ve isimler (`getByRole` ile otomatik)

### ❌ Test etme

- **Uygulama detayları**: state adları/değerleri, Hook çağrı sayısı, iç fonksiyonlar, görsel davranış temsil etmeyen sınıf adları
- **Kütüphanelerin kendisi**: React, TanStack Query zaten test edilmiş
- **Üçüncü parti servisler**: gerçek API'ler, ödeme sistemi (mock'la)

### Kırılgan testlerin işaretleri

| Belirti | Sebep | Çözüm |
|---|---|---|
| Yeniden düzenlemede (davranış aynı) testler kırılıyor | Uygulama detayı test ediliyor | Kullanıcının gördüğünü test et |
| Her yerde `getByTestId` | Elemanlar erişilebilir değil | Rol/etiket ekle, `getByRole` kullan |
| Testler bazen geçiyor bazen kalıyor (flaky) | Beklenmeyen async işlem, paylaşılan state | `findBy`/`waitFor`, her testte yeni QueryClient, MSW `resetHandlers` |
| Testler yavaş | Gerçek ağ, gerçek zamanlayıcı | MSW, sahte zamanlayıcılar |

---

## 15. Her Şey Bir Arada: Görev Uygulamasının Test Takımı

Bir görev uygulaması ve **tüm test katmanları**: reducer (birim), Hook (`renderHook`), bileşen (entegrasyon, MSW), form ve hata durumları.

### Test edilecek kod

```js
// ============ src/gorevler/gorevReducer.js ============
export function gorevReducer(gorevler, action) {
  switch (action.type) {
    case "yuklendi":
      return action.gorevler;
    case "eklendi":
      return [...gorevler, action.gorev];
    case "toggle":
      return gorevler.map((g) => (g.id === action.id ? { ...g, tamam: !g.tamam } : g));
    case "silindi":
      return gorevler.filter((g) => g.id !== action.id);
    default:
      throw new Error(`Bilinmeyen action: ${action.type}`);
  }
}

export function kalanSayisi(gorevler) {
  return gorevler.filter((g) => !g.tamam).length;
}
```

```js
// ============ src/gorevler/api.js ============
export async function gorevleriGetir() {
  const res = await fetch("/api/gorevler");
  if (!res.ok) throw new Error("Görevler yüklenemedi");
  return res.json();
}

export async function gorevEkle(metin) {
  const res = await fetch("/api/gorevler", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify({ metin }),
  });
  if (!res.ok) throw new Error("Görev eklenemedi");
  return res.json();
}
```

```js
// ============ src/gorevler/useGorevler.js ============
import { useEffect, useReducer, useState } from "react";
import { gorevReducer } from "./gorevReducer";
import { gorevEkle, gorevleriGetir } from "./api";

export function useGorevler() {
  const [gorevler, dispatch] = useReducer(gorevReducer, []);
  const [durum, setDurum] = useState("yukleniyor");   // "yukleniyor" | "hata" | "hazir"

  useEffect(() => {
    let iptal = false;
    gorevleriGetir()
      .then((veri) => {
        if (iptal) return;
        dispatch({ type: "yuklendi", gorevler: veri });
        setDurum("hazir");
      })
      .catch(() => !iptal && setDurum("hata"));
    return () => { iptal = true; };
  }, []);

  async function ekle(metin) {
    const gorev = await gorevEkle(metin);
    dispatch({ type: "eklendi", gorev });
  }

  return {
    gorevler,
    durum,
    ekle,
    toggle: (id) => dispatch({ type: "toggle", id }),
    sil: (id) => dispatch({ type: "silindi", id }),
  };
}
```

```jsx
// ============ src/gorevler/GorevUygulamasi.jsx ============
import { useActionState } from "react";
import { useFormStatus } from "react-dom";
import { useGorevler } from "./useGorevler";
import { kalanSayisi } from "./gorevReducer";

function EkleButonu() {
  const { pending } = useFormStatus();
  return <button disabled={pending}>{pending ? "Ekleniyor..." : "Ekle"}</button>;
}

export default function GorevUygulamasi() {
  const { gorevler, durum, ekle, toggle, sil } = useGorevler();

  const [sonuc, ekleAction] = useActionState(async (_onceki, formData) => {
    const metin = String(formData.get("metin") ?? "").trim();
    if (metin.length < 3) return { hata: "Görev en az 3 karakter olmalı." };
    try {
      await ekle(metin);
      return { hata: null };
    } catch (err) {
      return { hata: err.message };
    }
  }, { hata: null });

  if (durum === "yukleniyor") return <p>Yükleniyor...</p>;
  if (durum === "hata") return <p role="alert">Görevler yüklenemedi.</p>;

  return (
    <main>
      <h1>Görevlerim</h1>

      <form action={ekleAction}>
        <label>
          Yeni görev
          <input name="metin" />
        </label>
        <EkleButonu />
        {sonuc.hata && <p role="alert">{sonuc.hata}</p>}
      </form>

      {gorevler.length === 0 ? (
        <p>Henüz görev yok.</p>
      ) : (
        <ul aria-label="Görevler">
          {gorevler.map((g) => (
            <li key={g.id}>
              <label>
                <input type="checkbox" checked={g.tamam} onChange={() => toggle(g.id)} />
                {g.metin}
              </label>
              <button onClick={() => sil(g.id)} aria-label={`${g.metin} görevini sil`}>🗑️</button>
            </li>
          ))}
        </ul>
      )}

      <p role="status">{kalanSayisi(gorevler)} görev kaldı</p>
    </main>
  );
}
```

### Test altyapısı (MSW)

```js
// ============ src/test/sunucu.js ============
import { http, HttpResponse } from "msw";
import { setupServer } from "msw/node";

export const ORNEK_GOREVLER = [
  { id: "1", metin: "Testleri yaz", tamam: false },
  { id: "2", metin: "Kahve iç", tamam: true },
];

export const handlers = [
  http.get("/api/gorevler", () => HttpResponse.json(ORNEK_GOREVLER)),
  http.post("/api/gorevler", async ({ request }) => {
    const { metin } = await request.json();
    return HttpResponse.json({ id: crypto.randomUUID(), metin, tamam: false }, { status: 201 });
  }),
];

export const sunucu = setupServer(...handlers);
```

```js
// ============ src/test/kurulum.js ============
import "@testing-library/jest-dom/vitest";
import { sunucu } from "./sunucu";

beforeAll(() => sunucu.listen({ onUnhandledRequest: "error" }));
afterEach(() => sunucu.resetHandlers());
afterAll(() => sunucu.close());
```

### 1. Birim testleri: reducer ve yardımcı

```js
// ============ src/gorevler/gorevReducer.test.js ============
import { gorevReducer, kalanSayisi } from "./gorevReducer";

const baslangic = [
  { id: "1", metin: "A", tamam: false },
  { id: "2", metin: "B", tamam: true },
];

describe("gorevReducer", () => {
  it("görev ekler", () => {
    const yeni = { id: "3", metin: "C", tamam: false };
    expect(gorevReducer(baslangic, { type: "eklendi", gorev: yeni })).toHaveLength(3);
  });

  it("toggle sadece hedef görevi değiştirir", () => {
    const sonuc = gorevReducer(baslangic, { type: "toggle", id: "1" });
    expect(sonuc[0].tamam).toBe(true);
    expect(sonuc[1]).toBe(baslangic[1]);   // değişmeyen nesne AYNI referans
  });

  it("orijinal diziyi değiştirmez", () => {
    const kopya = structuredClone(baslangic);
    gorevReducer(baslangic, { type: "silindi", id: "1" });
    expect(baslangic).toEqual(kopya);
  });

  it("bilinmeyen action'da hata fırlatır", () => {
    expect(() => gorevReducer(baslangic, { type: "yok" })).toThrow(/bilinmeyen/i);
  });
});

describe("kalanSayisi", () => {
  it("tamamlanmamış görevleri sayar", () => {
    expect(kalanSayisi(baslangic)).toBe(1);
    expect(kalanSayisi([])).toBe(0);
  });
});
```

### 2. Hook testi: `renderHook` + MSW

```js
// ============ src/gorevler/useGorevler.test.js ============
import { renderHook, waitFor, act } from "@testing-library/react";
import { http, HttpResponse } from "msw";
import { sunucu, ORNEK_GOREVLER } from "../test/sunucu";
import { useGorevler } from "./useGorevler";

describe("useGorevler", () => {
  it("görevleri yükler", async () => {
    const { result } = renderHook(() => useGorevler());
    expect(result.current.durum).toBe("yukleniyor");

    await waitFor(() => expect(result.current.durum).toBe("hazir"));
    expect(result.current.gorevler).toEqual(ORNEK_GOREVLER);
  });

  it("sunucu hatasında hata durumuna geçer", async () => {
    sunucu.use(http.get("/api/gorevler", () => new HttpResponse(null, { status: 500 })));
    const { result } = renderHook(() => useGorevler());

    await waitFor(() => expect(result.current.durum).toBe("hata"));
  });

  it("yeni görev ekler", async () => {
    const { result } = renderHook(() => useGorevler());
    await waitFor(() => expect(result.current.durum).toBe("hazir"));

    await act(() => result.current.ekle("Yeni görev"));
    expect(result.current.gorevler.at(-1)).toMatchObject({ metin: "Yeni görev", tamam: false });
  });
});
```

### 3. Entegrasyon testleri: bileşen, kullanıcı gibi

```jsx
// ============ src/gorevler/GorevUygulamasi.test.jsx ============
import { render, screen, within } from "@testing-library/react";
import userEvent from "@testing-library/user-event";
import { http, HttpResponse, delay } from "msw";
import { sunucu } from "../test/sunucu";
import GorevUygulamasi from "./GorevUygulamasi";

async function hazirRender() {
  const user = userEvent.setup();
  render(<GorevUygulamasi />);
  await screen.findByRole("heading", { name: "Görevlerim" });   // yükleme bitsin
  return { user };
}

describe("GorevUygulamasi", () => {
  it("önce yükleniyor gösterir, sonra görevleri listeler", async () => {
    render(<GorevUygulamasi />);
    expect(screen.getByText("Yükleniyor...")).toBeInTheDocument();

    const liste = await screen.findByRole("list", { name: "Görevler" });
    expect(within(liste).getAllByRole("listitem")).toHaveLength(2);
    expect(screen.getByRole("status")).toHaveTextContent("1 görev kaldı");
  });

  it("yükleme hatasında uyarı gösterir", async () => {
    sunucu.use(http.get("/api/gorevler", () => new HttpResponse(null, { status: 500 })));
    render(<GorevUygulamasi />);
    expect(await screen.findByRole("alert")).toHaveTextContent("Görevler yüklenemedi.");
  });

  it("görev tamamlanınca kalan sayısı azalır", async () => {
    const { user } = await hazirRender();
    await user.click(screen.getByRole("checkbox", { name: "Testleri yaz" }));

    expect(screen.getByRole("checkbox", { name: "Testleri yaz" })).toBeChecked();
    expect(screen.getByRole("status")).toHaveTextContent("0 görev kaldı");
  });

  it("görevi siler", async () => {
    const { user } = await hazirRender();
    await user.click(screen.getByRole("button", { name: "Kahve iç görevini sil" }));

    expect(screen.queryByText("Kahve iç")).not.toBeInTheDocument();
    expect(screen.getAllByRole("listitem")).toHaveLength(1);
  });

  it("kısa görev metninde doğrulama hatası gösterir", async () => {
    const { user } = await hazirRender();
    await user.type(screen.getByLabelText("Yeni görev"), "ab");
    await user.click(screen.getByRole("button", { name: "Ekle" }));

    expect(await screen.findByRole("alert")).toHaveTextContent("en az 3 karakter");
    expect(screen.getAllByRole("listitem")).toHaveLength(2);   // eklenmedi
  });

  it("yeni görev ekler; eklerken buton devre dışı kalır", async () => {
    sunucu.use(
      http.post("/api/gorevler", async ({ request }) => {
        await delay(100);   // bekleme durumunu görebilmek için
        const { metin } = await request.json();
        return HttpResponse.json({ id: "yeni", metin, tamam: false }, { status: 201 });
      })
    );
    const { user } = await hazirRender();

    await user.type(screen.getByLabelText("Yeni görev"), "Notları gözden geçir");
    await user.click(screen.getByRole("button", { name: "Ekle" }));

    expect(await screen.findByRole("button", { name: "Ekleniyor..." })).toBeDisabled();
    expect(await screen.findByText("Notları gözden geçir")).toBeInTheDocument();
    expect(screen.getByRole("status")).toHaveTextContent("2 görev kaldı");
    expect(screen.getByLabelText("Yeni görev")).toHaveValue("");   // form sıfırlandı
  });

  it("sunucu ekleme hatasını gösterir", async () => {
    sunucu.use(http.post("/api/gorevler", () => new HttpResponse(null, { status: 500 })));
    const { user } = await hazirRender();

    await user.type(screen.getByLabelText("Yeni görev"), "Başarısız görev");
    await user.click(screen.getByRole("button", { name: "Ekle" }));

    expect(await screen.findByRole("alert")).toHaveTextContent("Görev eklenemedi");
  });
});
```

### Bu test takımında

| Özellik | Açıklama |
|---|---|
| **Katmanlar** | Reducer/yardımcı → birim; Hook → `renderHook`; bileşen → entegrasyon |
| **Uygulama detayı yok** | Bileşen testleri `useReducer`, `useActionState`, `dispatch`'ten habersiz; bileşen TanStack Query ile yeniden yazılsa da testler geçer |
| **Erişilebilir sorgular** | `getByRole`, `getByLabelText`; hiç `data-testid` yok |
| **MSW** | Gerçek `fetch`; hata ve gecikme `sunucu.use` ile test başına; `resetHandlers` ile izolasyon |
| **Asenkron** | `findBy` ile yükleme, `waitFor` ile Hook durumu, `delay` ile bekleme durumu |
| **Kenar durumlar** | Yükleniyor, yükleme hatası, doğrulama hatası, ekleme hatası, form sıfırlama |

> **Dene:** `kalanSayisi`'nda `!g.tamam` yerine `g.tamam` yaz → hem birim hem entegrasyon testleri kırmızıya döner ve neyin bozulduğunu gösterir.

---

## 16. Özet

- **Otomatik test**, elle "tıkla ve bak" işini kodla yazmaktır; değişikliklerde hızlı ve güvenli geri bildirim sağlar.
- Test türleri: statik analiz, birim, **entegrasyon** (en çok yazılan), E2E.
- **Altın kural:** Uygulama detayını değil, kullanıcının gördüğünü ve yaptığını test et.
- Araçlar: **Vitest 5** (Node 22.12+, Vite 6.4+), jsdom, **React Testing Library**, **user-event**, **jest-dom**, **MSW**, **Playwright.**
- Yapı: `describe` / `it` / `expect`; AAA (hazırla → eylem → kontrol).
- `toBe` referans/primitive, `toEqual` içerik eşitliği; jest-dom ile DOM eşleştiricileri.
- `render` + `screen`; sorgular: `getBy` (olmalı), `queryBy` (olmamalı), `findBy` (gelecek).
- **Sorgu önceliği:** `getByRole` > `getByLabelText` > ... > `getByTestId` (son çare); `getByRole` erişilebilirliği de test eder.
- **`user-event`** gerçekçi etkileşimler sağlar; hepsi `await` edilir.
- Asenkron: `findBy`, `waitFor` (sadece kontrol), `waitForElementToBeRemoved`; `act` uyarısının çözümü beklemektir.
- Mock: `vi.fn`, `vi.spyOn`, `vi.mock`; sahte zamanlayıcılar (`vi.useFakeTimers` + `advanceTimers`); Vitest 5'te mock'lar her testten önce sıfırlanır.
- **MSW** ağ düzeyinde taklit eder; `onUnhandledRequest: "error"`, `resetHandlers`, test başına `sunucu.use`.
- Provider'lar için **özel `render`**; her testte yeni `QueryClient` ve `retry: false`; router için `createMemoryRouter` + `initialEntries`.
- Reducer'lar saf fonksiyon olarak, Hook'lar **`renderHook`** + `act` ile test edilir.
- Form/Action testleri uygulamadan bağımsızdır (`useActionState` mi `onSubmit` mi fark etmez).
- Snapshot'lar ölçülü (küçük, inline); kapsam bir harita, hedef değil.
- **Playwright** kritik akışlar için E2E; Vitest Browser Mode gerçek tarayıcıda bileşen testi.
- Kırılgan test belirtileri: uygulama detayı testi, `getByTestId` bolluğu, flaky testler, yavaşlık.

---

⬅️ Önceki: [20 · React 19 Yenilikleri](./20-react-19.md) · ➡️ Sonraki: [22 · TypeScript ile React](./22-typescript.md)
