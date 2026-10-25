# ⚛️ Sıfırdan React — Türkçe Ders Notları

Sıfırdan başlayıp modern React ekosistemine kadar ilerleyen, Türkçe hazırlanmış ders notları. Her konu ayrı bir Markdown dosyasında; açıklamalar, kod örnekleri, sık yapılan hatalar ve özet tabloları içerir. Dosyaların sonundaki **⬅️ Önceki / ➡️ Sonraki** bağlantılarıyla seriyi baştan sona takip edebilirsin.

## 🧰 Kapsanan Teknolojiler

| Alan | Araç / Sürüm |
| --- | --- |
| Kütüphane | React 19 (+ React Compiler) |
| Geliştirme ortamı | Vite |
| Stil | Tailwind CSS v4, CSS Modules |
| Yönlendirme | React Router v8 |
| Sunucu verisi | TanStack Query v5 |
| Test | Vitest, React Testing Library, MSW, Playwright |
| Tip güvenliği | TypeScript, Zod |
| Tam yığın | Next.js 16 (App Router, Server Components) |

## 📚 İçindekiler

### 🟢 Bölüm 1 — Temeller

| # | Konu | Ne öğreneceksin? |
| --- | --- | --- |
| 00 | [React İçin JavaScript](./00-react-icin-javascript.md) | Arrow function, destructuring, spread, map/filter, modüller, async/await |
| 01 | [React Nedir?](./01-react-nedir.md) | Bileşen mantığı, bildirimsel yaklaşım, Vite ile proje kurulumu |
| 02 | [JSX](./02-jsx.md) | JSX kuralları, ifadeler, `className`, Fragment |
| 03 | [Bileşenler](./03-bilesenler.md) | Fonksiyon bileşenleri, bileşen ağacı, dosya düzeni |
| 04 | [Props](./04-props.md) | Veri aktarımı, `children`, varsayılan değerler |
| 05 | [State](./05-state.md) | `useState`, yeniden render, değişmezlik (immutability) |
| 06 | [Olay Yönetimi](./06-olay-yonetimi.md) | `onClick`, `onChange`, olay nesnesi, fonksiyon geçirme |
| 07 | [Koşullu Render](./07-kosullu-render.md) | `&&`, ternary, erken `return` |
| 08 | [Listeler ve Key](./08-listeler-ve-key.md) | `map` ile listeleme, `key` neden önemli |
| 09 | [Formlar](./09-formlar.md) | Kontrollü / kontrolsüz input, form gönderimi, doğrulama |

### 🟡 Bölüm 2 — Hook'lar ve Mimari

| # | Konu | Ne öğreneceksin? |
| --- | --- | --- |
| 10 | [useEffect ve Yan Etkiler](./10-useeffect.md) | Bağımlılık dizisi, temizleme (cleanup), gereksiz effect'ten kaçınma |
| 11 | [useRef](./11-useref.md) | DOM erişimi, render tetiklemeyen değerler |
| 12 | [Bileşenler Arası İletişim](./12-bilesenler-arasi-iletisim.md) | State'i yukarı taşıma, tek doğruluk kaynağı |
| 13 | [Context API](./13-context-api.md) | Prop drilling'e çözüm, Provider, `use(Context)` |
| 14 | [useReducer](./14-usereducer.md) | Karmaşık state, action ve reducer deseni |
| 15 | [Custom Hook'lar](./15-custom-hooks.md) | Mantığı yeniden kullanılabilir Hook'lara ayırma |

### 🎨 Bölüm 3 — Stil

| # | Konu | Ne öğreneceksin? |
| --- | --- | --- |
| 16 | [Stil Yöntemleri](./16-stil-yontemleri.md) | Inline stil, CSS dosyaları, CSS Modules, karşılaştırma |
| 16a | [Tailwind CSS (Detaylı)](./16a-tailwind-css.md) | Tailwind v4 kurulumu, utility sınıfları, responsive, dark mode, `@theme` |

### 🔵 Bölüm 4 — Ekosistem

| # | Konu | Ne öğreneceksin? |
| --- | --- | --- |
| 17 | [React Router](./17-react-router.md) | Data mode, `loader` / `action`, dinamik ve iç içe rotalar |
| 18 | [Veri Çekme ve TanStack Query](./18-veri-cekme.md) | `queryKey`, önbellek, `staleTime`, mutation, iyimser güncelleme |
| 19 | [Performans](./19-performans.md) | `memo`, `useMemo`, `useCallback`, React Compiler, code splitting |
| 20 | [React 19 Yenilikleri](./20-react-19.md) | Actions, `useActionState`, `useOptimistic`, `use`, `<Activity>` |

### 🔴 Bölüm 5 — Profesyonel Geliştirme

| # | Konu | Ne öğreneceksin? |
| --- | --- | --- |
| 21 | [Test](./21-test.md) | Vitest, Testing Library, user-event, MSW, Playwright |
| 22 | [TypeScript ile React](./22-typescript.md) | Props tipleri, generic bileşenler, discriminated union, Zod |
| 23 | [Next.js ve Server Components](./23-nextjs.md) | App Router, `"use client"`, Server Functions, önbellekleme |

### 🎯 Bölüm 6 — Mülakat Hazırlığı

| # | Konu | Ne öğreneceksin? |
| --- | --- | --- |
| 24 | [Mülakat Soruları ve Cevapları](./24-mulakat-sorulari.md) | 155 soru: teori, "çıktı ne olur?", canlı kodlama, davranışsal sorular |

## 🗺️ Önerilen Çalışma Sırası

1. **00 → 09:** Temelleri sırayla bitir. Her konudaki örnekleri kendi Vite projende yeniden yaz.
2. **10 → 15:** Hook'ları öğrendikten sonra küçük bir proje yap (ör. todo uygulaması veya not defteri).
3. **16 → 16a:** Projene Tailwind ile stil ver.
4. **17 → 20:** Projeye rotalar ve gerçek bir API ekle, ardından performansı ve React 19 özelliklerini dene.
5. **21 → 23:** Projeyi TypeScript'e geçir, test yaz; sonra Next.js ile tam yığın bir sürüm kur.
6. **24:** Mülakat öncesinde soruları önce cevaplara bakmadan çözmeye çalış.

## 💡 Nasıl Kullanılır?

- Dosyaları sırayla oku; her dosya bir öncekinin üzerine kurulur.
- Kod örneklerini kopyalamak yerine **elle yazmaya** çalış.
- Takıldığın yerde ilgili dosyadaki "Sık Yapılan Hatalar" ve "Özet" bölümlerine dön.
- Mülakat dosyasındaki cevaplar açılır-kapanır kutular içindedir; önce kendin düşün, sonra aç.

## 🔗 Faydalı Kaynaklar

- [react.dev](https://react.dev) — Resmî React dokümantasyonu
- [vite.dev](https://vite.dev) — Vite
- [tailwindcss.com](https://tailwindcss.com) — Tailwind CSS
- [reactrouter.com](https://reactrouter.com) — React Router
- [tanstack.com/query](https://tanstack.com/query) — TanStack Query
- [vitest.dev](https://vitest.dev) — Vitest
- [testing-library.com](https://testing-library.com) — Testing Library
- [typescriptlang.org](https://www.typescriptlang.org) — TypeScript
- [nextjs.org](https://nextjs.org) — Next.js

---

⭐ Faydalı bulduysan depoya yıldız vermeyi unutma. İyi çalışmalar!
