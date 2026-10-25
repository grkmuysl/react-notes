# 24 · React Mülakat Soruları ve Cevapları

> **Amaç:** Frontend / React mülakatlarında sık sorulan soruları, sektörde en çok kullanılan konulara odaklanarak toplamak. Cevaplar kısa ve mülakatta söylenebilecek netlikte tutuldu; detaylar için ilgili konu notlarına bağlantı verildi.
>
> **Kullanım önerisi:** Önce soruyu oku, cevabı kendin sesli anlat, sonra açılır bölümden kontrol et.
>
> **Zorluk:** 🟢 Temel · 🟡 Orta · 🔴 İleri

---

## İçindekiler

1. [React İçin JavaScript](#1-react-için-javascript)
2. [React Temelleri ve JSX](#2-react-temelleri-ve-jsx)
3. [Bileşenler ve Props](#3-bileşenler-ve-props)
4. [State ve Render Süreci](#4-state-ve-render-süreci)
5. [Listeler, Key'ler ve Koşullu Render](#5-listeler-keyler-ve-koşullu-render)
6. [Olaylar ve Formlar](#6-olaylar-ve-formlar)
7. [Hook'lar](#7-hooklar)
8. [useEffect ve Yan Etkiler](#8-useeffect-ve-yan-etkiler)
9. [State Yönetimi ve Mimari](#9-state-yönetimi-ve-mimari)
10. [Performans](#10-performans)
11. [Yönlendirme ve Veri Çekme](#11-yönlendirme-ve-veri-çekme)
12. [React 19 ve Modern Özellikler](#12-react-19-ve-modern-özellikler)
13. [SSR, Server Components ve Next.js](#13-ssr-server-components-ve-nextjs)
14. [TypeScript ile React](#14-typescript-ile-react)
15. [Test](#15-test)
16. [Stil, Erişilebilirlik ve Güvenlik](#16-stil-erişilebilirlik-ve-güvenlik)
17. [Kod Okuma ve "Çıktı Ne Olur?" Soruları](#17-kod-okuma-ve-çıktı-ne-olur-soruları)
18. [Kodlama Soruları (Canlı Kodlama)](#18-kodlama-soruları-canlı-kodlama)
19. [Davranışsal ve Deneyim Soruları](#19-davranışsal-ve-deneyim-soruları)
20. [Mülakat İpuçları](#20-mülakat-ipuçları)

---

## 1. React İçin JavaScript

<details>
<summary>🟢 <b>1.1 <code>var</code>, <code>let</code> ve <code>const</code> arasındaki farklar nelerdir?</b></summary>

- `var`: Fonksiyon kapsamlıdır, hoisting ile `undefined` olarak yukarı taşınır, yeniden tanımlanabilir. Modern kodda kullanılmaz.
- `let`: Blok kapsamlıdır, yeniden atanabilir.
- `const`: Blok kapsamlıdır, **yeniden atanamaz**; ama gösterdiği nesnenin içi değiştirilebilir (`const kisi = {}; kisi.ad = "A"` geçerli).

React'te neredeyse her şey `const` ile tanımlanır. → [Konu 0](./00-react-icin-javascript.md)
</details>

<details>
<summary>🟢 <b>1.2 <code>==</code> ile <code>===</code> farkı nedir?</b></summary>

`==` karşılaştırmadan önce tip dönüşümü yapar (`0 == "0"` → `true`), `===` hem tipi hem değeri karşılaştırır (`0 === "0"` → `false`). Öngörülebilirlik için her zaman `===` kullanılır.
</details>

<details>
<summary>🟢 <b>1.3 Primitive ve referans tipleri arasındaki fark nedir? React için neden önemlidir?</b></summary>

- Primitive'ler (string, number, boolean, null, undefined...) **değerle** karşılaştırılır.
- Nesne, dizi ve fonksiyonlar **referansla** karşılaştırılır: `{} === {}` → `false`.

React; state değişimini, `memo` props karşılaştırmasını ve effect bağımlılıklarını `Object.is` ile, yani referansla kontrol eder. Bu yüzden state'i mutasyona uğratmak değişikliği görünmez yapar, her render'da yeni oluşturulan nesne/fonksiyon ise `memo`'yu ve effect'leri bozar. → [Konu 5.7](./05-state.md)
</details>

<details>
<summary>🟢 <b>1.4 Spread (<code>...</code>) ve rest arasındaki fark nedir? Spread derin kopya mı yapar?</b></summary>

Aynı sözdizimi: **spread** açar/kopyalar (`[...dizi, x]`, `{ ...nesne, a: 1 }`), **rest** kalanları toplar (`const { a, ...kalan } = nesne`). Spread **sığ (shallow)** kopya yapar; iç içe nesneler aynı referansı paylaşır. Derin güncellemede her seviye ayrıca kopyalanmalıdır. → [Konu 0.5](./00-react-icin-javascript.md)
</details>

<details>
<summary>🟡 <b>1.5 Closure (kapanış) nedir? React'teki "stale closure" problemi nasıl oluşur?</b></summary>

Closure, bir fonksiyonun tanımlandığı kapsamdaki değişkenleri hatırlamasıdır. React'te her render kendi props/state değerleriyle yeni fonksiyonlar oluşturur. Bir effect, zamanlayıcı veya callback **eski bir render'ın** fonksiyonunu tutarsa, eski (bayat) değerleri görür:

```jsx
useEffect(() => {
  const id = setInterval(() => setSayi(sayi + 1), 1000);   // sayi hep ilk değer
  return () => clearInterval(id);
}, []);
```

Çözümler: updater fonksiyon (`setSayi(s => s + 1)`), bağımlılıkları doğru vermek, `useEffectEvent` veya ref kullanmak. → [Konu 10.5](./10-useeffect.md)
</details>

<details>
<summary>🟢 <b>1.6 <code>map</code>, <code>filter</code>, <code>reduce</code> ne yapar? Neden React'te sık kullanılırlar?</b></summary>

Üçü de orijinal diziyi değiştirmeden yeni bir değer döndürür: `map` dönüştürür, `filter` süzer, `reduce` tek değere indirger. React immutability ilkesine uygun oldukları için listeleri render etmede (`map`), silmede (`filter`), güncellemede (`map`) ve toplam hesaplamada (`reduce`) kullanılır. `push`, `splice`, `sort` ise orijinali değiştirdiği için state'te kullanılmaz.
</details>

<details>
<summary>🟢 <b>1.7 <code>||</code>, <code>??</code> ve <code>?.</code> operatörleri ne işe yarar?</b></summary>

- `a || b`: `a` herhangi bir falsy değerse (`0`, `""`, `false`, `null`...) `b`.
- `a ?? b`: Sadece `a` `null`/`undefined` ise `b`. `0` ve `""` korunur.
- `a?.b`: `a` `null`/`undefined` ise hata vermeden `undefined` döner.
</details>

<details>
<summary>🟡 <b>1.8 Promise, <code>async/await</code> ve <code>Promise.all</code> nedir?</b></summary>

Promise, gelecekte tamamlanacak bir işlemi temsil eder (pending → fulfilled/rejected). `async/await` Promise'leri senkron kod gibi okunabilir kılar; `async` fonksiyon her zaman Promise döndürür. `Promise.all` birden çok Promise'i **paralel** çalıştırır, hepsi bitince sonuç dizisi döner; biri reddedilirse tamamı reddedilir. Bağımsız veri isteklerinde şelaleyi önlemek için kullanılır.
</details>

<details>
<summary>🟡 <b>1.9 <code>fetch</code> HTTP 404 veya 500 döndüğünde hata fırlatır mı?</b></summary>

Hayır. `fetch` sadece ağ hatalarında reddedilir. 404/500 gibi cevaplarda Promise başarılı çözülür; `res.ok` (veya `res.status`) kontrol edilip gerekirse elle hata fırlatılmalıdır.
</details>

<details>
<summary>🟡 <b>1.10 Debounce ile throttle arasındaki fark nedir?</b></summary>

- **Debounce:** Olaylar durduktan belirli bir süre sonra **bir kez** çalışır (arama kutusu: kullanıcı yazmayı bırakınca ara).
- **Throttle:** Olay ne kadar sık gelirse gelsin, belirli aralıklarla **en fazla bir kez** çalışır (kaydırma, pencere boyutlandırma).
</details>

<details>
<summary>🟡 <b>1.11 Event loop nedir? Microtask ve macrotask farkı nedir?</b></summary>

JavaScript tek iş parçacıklıdır; event loop, çağrı yığını boşaldığında kuyruklardaki işleri sırayla çalıştırır. **Microtask**'lar (Promise `then`, `queueMicrotask`) her makro görevden sonra tamamen boşaltılır; **macrotask**'lar (`setTimeout`, olaylar) ise birer birer çalışır. Bu yüzden `Promise.resolve().then(...)` her zaman `setTimeout(..., 0)`'dan önce çalışır.
</details>

---

## 2. React Temelleri ve JSX

<details>
<summary>🟢 <b>2.1 React nedir? Kütüphane mi framework mü?</b></summary>

React, kullanıcı arayüzleri oluşturmak için bir JavaScript **kütüphanesidir**. Sadece görünüm katmanıyla ilgilenir; yönlendirme, veri çekme, build gibi konularda yöntem dayatmaz. Next.js veya React Router Framework modu gibi araçlar React'in üzerine bu parçaları ekleyen **framework**'lerdir. → [Konu 1](./01-react-nedir.md)
</details>

<details>
<summary>🟢 <b>2.2 Deklaratif ve imperatif programlama arasındaki fark nedir?</b></summary>

İmperatifte sonuca **nasıl** ulaşılacağı adım adım yazılır (elementi bul, metnini değiştir). Deklaratifte belirli bir durumda ekranın **ne** göstermesi gerektiği tarif edilir; güncellemeyi React yapar. React'in formülü: `UI = f(state)`.
</details>

<details>
<summary>🟢 <b>2.3 Virtual DOM nedir? Gerçekten her zaman daha mı hızlıdır?</b></summary>

Virtual DOM, arayüzün sade JavaScript nesnelerinden oluşan bir temsilidir. State değişince React yeni bir sanal ağaç üretir, eskisiyle karşılaştırır (**reconciliation / diffing**) ve sadece farkları gerçek DOM'a uygular.

"Her zaman daha hızlı" bir efsanedir: Elle optimize edilmiş DOM işlemleri daha hızlı olabilir. Asıl kazanç, geliştiricinin DOM'u elle yönetmek zorunda kalmaması ve bunun makul bir performansla yapılmasıdır.
</details>

<details>
<summary>🟡 <b>2.4 Reconciliation nasıl çalışır? React iki ağacı nasıl karşılaştırır?</b></summary>

React, O(n) karmaşıklıkta çalışan sezgisel bir algoritma kullanır ve iki varsayıma dayanır:
1. **Farklı türde** elementler (ör. `<div>` → `<span>`, `ComponentA` → `ComponentB`) farklı ağaçlar üretir; eski alt ağaç tamamen yıkılıp yenisi kurulur (state'ler silinir).
2. Listelerde **`key`**, elemanların kimliğini belirler; aynı key'li elemanlar eşleştirilir.

Aynı türdeki elementlerde sadece değişen nitelikler güncellenir.
</details>

<details>
<summary>🔴 <b>2.5 React Fiber nedir?</b></summary>

Fiber, React 16 ile gelen yeniden yazılmış uzlaştırma (reconciliation) motorudur. Render işini küçük birimlere (fiber'lara) bölerek **kesintiye uğratılabilir** hale getirir: React işi duraklatabilir, önceliklendirebilir, yarıda bırakabilir veya yeniden başlatabilir. `useTransition`, `useDeferredValue`, Suspense gibi eşzamanlı (concurrent) özelliklerin temelidir.
</details>

<details>
<summary>🟢 <b>2.6 JSX nedir? Tarayıcı JSX'i doğrudan anlar mı?</b></summary>

JSX, JavaScript içinde HTML benzeri sözdizimi sağlayan bir uzantıdır. Tarayıcı anlamaz; build aracı onu `_jsx("div", props)` gibi fonksiyon çağrılarına derler. Bu çağrılar **React elementi** denen sade nesneler döndürür. → [Konu 2](./02-jsx.md)
</details>

<details>
<summary>🟢 <b>2.7 Bir bileşen neden iki kardeş element döndüremez? Çözüm nedir?</b></summary>

Her JSX elementi bir fonksiyon çağrısıdır ve bir fonksiyon tek değer döndürebilir. Çözüm, elementleri tek bir ebeveynle sarmaktır; DOM'a fazladan element eklememek için **Fragment** (`<>...</>`) kullanılır.
</details>

<details>
<summary>🟢 <b>2.8 JSX'te <code>class</code> yerine neden <code>className</code> yazılır?</b></summary>

JSX JavaScript'e derlenir ve nitelikler nesne anahtarı olur; `class` ve `for` JavaScript'te ayrılmış kelimelerdir. Bu yüzden `className` ve `htmlFor` kullanılır; diğer nitelikler de camelCase yazılır (`onClick`, `tabIndex`).
</details>

<details>
<summary>🟡 <b>2.9 <code>{liste.length && &lt;Liste /&gt;}</code> ifadesinin sorunu nedir?</b></summary>

Liste boşsa `0 && ...` ifadesi `0` döner ve React `0`'ı ekrana basar (`false`, `null`, `undefined` basılmaz ama `0` basılır). Çözüm: sol tarafı boolean yapmak (`liste.length > 0 && ...`) veya ternary kullanmak. → [Konu 7.6](./07-kosullu-render.md)
</details>

<details>
<summary>🟢 <b>2.10 <code>React.StrictMode</code> ne işe yarar? Neden bileşenler iki kez render oluyor?</b></summary>

Yalnızca geliştirme ortamında çalışan bir yardımcıdır. Saf olmayan bileşenleri ve eksik effect temizliklerini yakalamak için bileşenleri **bilerek iki kez** render eder ve effect'leri setup → cleanup → setup şeklinde çalıştırır. Production'da bu olmaz.
</details>

---

## 3. Bileşenler ve Props

<details>
<summary>🟢 <b>3.1 Props ve state arasındaki fark nedir?</b></summary>

| | Props | State |
|---|---|---|
| Kaynak | Ebeveynden gelir | Bileşenin kendisine ait |
| Kim değiştirir | Ebeveyn (yenisini verir) | Bileşen (`set` fonksiyonu ile) |
| Bileşen içinde | Salt okunur | Salt okunur; değişim `set` ile |
| Değişince | Yeniden render | Yeniden render |

Props bileşenin **girdisi**, state ise **hafızasıdır**. → [Konu 4](./04-props.md), [Konu 5](./05-state.md)
</details>

<details>
<summary>🟢 <b>3.2 Bir bileşen kendi props'unu değiştirebilir mi?</b></summary>

Hayır. Props salt okunurdur; değiştirmek saflığı bozar ve nesneler referansla paylaşıldığı için ebeveynin verisini de bozar. Değer değişmeliyse sahibi (genelde ebeveyn) state olarak tutar ve yenisini props ile verir; çocuk değişiklik isteğini callback ile bildirir.
</details>

<details>
<summary>🟢 <b>3.3 Tek yönlü veri akışı (one-way data flow) nedir?</b></summary>

Veri yalnızca ebeveynden çocuğa (yukarıdan aşağıya) props ile akar. Çocuk, ebeveynin verdiği fonksiyonu çağırarak olay bildirir. Kısaca: **veri aşağı, olaylar yukarı.** Verinin kaynağı her zaman bellidir, hata ayıklama kolaylaşır.
</details>

<details>
<summary>🟢 <b>3.4 <code>children</code> prop'u nedir? Ne için kullanılır?</b></summary>

Bir bileşenin açılış ve kapanış etiketleri arasına yazılan içeriktir. İçeriğini bilmeyen "çerçeve" bileşenler (Kart, Modal, Layout) yazmak için kullanılır. Kompozisyonun temel aracıdır ve prop drilling'i azaltmada da kullanılır.
</details>

<details>
<summary>🟡 <b>3.5 Prop drilling nedir? Nasıl çözülür?</b></summary>

Bir verinin, onu kullanmayan ara bileşenlerden sırf aktarılmak için geçirilmesidir. Çözümler sırasıyla:
1. Birkaç seviyeyse sorun değildir; açık ve izlenebilirdir.
2. **Kompozisyon** (`children` / slot props) ile ara bileşenleri aradan çıkarmak.
3. Uygulama genelindeki veriler (tema, oturum, dil) için **Context**.
4. Karmaşık global state için Zustand/Redux gibi kütüphaneler.
</details>

<details>
<summary>🟡 <b>3.6 Bileşenin içinde başka bir bileşen tanımlamak neden hatalıdır?</b></summary>

```jsx
function Liste() {
  function Satir() { return <li>...</li>; }   // ❌
  return <Satir />;
}
```

Her render'da `Satir` **yeni bir fonksiyon** (yeni bir bileşen türü) olur. React bunu farklı bir tür sayar, alt ağacı yıkıp sıfırdan kurar: state silinir, input'lar odağını kaybeder, performans düşer. Bileşenler her zaman en üst seviyede tanımlanır.
</details>

<details>
<summary>🟡 <b>3.7 Saf (pure) bileşen ne demektir? Neden önemlidir?</b></summary>

Aynı props/state/context için her zaman aynı JSX'i döndüren ve render sırasında dışarıdaki hiçbir şeyi değiştirmeyen bileşendir. React; render'ı kesip yeniden başlatabilmek, gereksiz render'ları atlamak (memo, React Compiler) ve sunucuda render etmek için bu varsayıma dayanır. Yan etkiler olay yöneticilerinde veya effect'lerde yapılır.
</details>

<details>
<summary>🟢 <b>3.8 Controlled ve uncontrolled bileşen (veya input) farkı nedir?</b></summary>

- **Controlled:** Değer React state'inde tutulur (`value` + `onChange`); state tek doğruluk kaynağıdır. Anlık doğrulama, girdi dönüştürme için idealdir.
- **Uncontrolled:** Değer DOM'da tutulur (`defaultValue`), gerektiğinde ref veya `FormData` ile okunur. Daha az kod ve render.

Aynı ayrım kendi bileşenlerin için de geçerlidir: bilgi kendi state'indeyse uncontrolled, props'tan geliyorsa controlled. → [Konu 9](./09-formlar.md), [Konu 12.3](./12-bilesenler-arasi-iletisim.md)
</details>

<details>
<summary>🟡 <b>3.9 Class bileşenleri ile fonksiyon bileşenleri arasındaki fark nedir? Yaşam döngüsü metotlarının Hook karşılıkları nelerdir?</b></summary>

Fonksiyon bileşenleri daha sade, Hook'larla mantığı paylaşmaya uygun ve React'in yeni özelliklerinin hedefidir. Class bileşenleri eski API'dir (Error Boundary hariç).

| Class | Fonksiyon |
|---|---|
| `this.state` / `setState` | `useState`, `useReducer` |
| `componentDidMount` | `useEffect(fn, [])` |
| `componentDidUpdate` | `useEffect(fn, [bagimliliklar])` |
| `componentWillUnmount` | `useEffect` cleanup fonksiyonu |
| `shouldComponentUpdate` / `PureComponent` | `memo` |

Not: Effect'leri yaşam döngüsü olarak değil **senkronizasyon** olarak düşünmek önerilir.
</details>

<details>
<summary>🟡 <b>3.10 Higher-Order Component (HOC), render props ve custom Hook arasındaki fark nedir?</b></summary>

Üçü de mantığı bileşenler arasında paylaşmanın yollarıdır:
- **HOC:** Bir bileşeni alıp yeni bir bileşen döndüren fonksiyon (`withAuth(Sayfa)`). Props çakışmaları ve "sarmalayıcı cehennemi" sorunları vardır.
- **Render props:** Ne render edileceğini fonksiyon olarak alan bileşen (`<Fare>{(konum) => ...}</Fare>`).
- **Custom Hook:** `use` ile başlayan, içinde Hook çağıran fonksiyon. Günümüzde mantık paylaşmanın **tercih edilen** yoludur; daha sade ve iç içe sarmalayıcı yaratmaz.
</details>

<details>
<summary>🟡 <b>3.11 Portal nedir? Ne zaman kullanılır?</b></summary>

`createPortal(jsx, domDugumu)` bir bileşenin çıktısını, React ağacındaki konumundan bağımsız olarak DOM'un başka bir yerine (ör. `document.body`) render eder. Modallar, tooltip'ler ve açılır menüler gibi, ebeveynin `overflow: hidden` veya `z-index` kısıtlarından kaçması gereken elemanlar için kullanılır. Olaylar DOM'daki yerine göre değil, **React ağacındaki** yerine göre yayılır.
</details>

<details>
<summary>🟡 <b>3.12 Error Boundary nedir? Hangi hataları yakalamaz?</b></summary>

Alt ağacındaki render hatalarını yakalayıp yedek bir arayüz gösteren bileşendir (hâlâ class ile yazılır veya `react-error-boundary` kullanılır; React Router ve Next.js kendi mekanizmalarını sunar).

Yakalamadıkları: olay yöneticilerindeki hatalar, asenkron koddaki hatalar (`setTimeout`, Promise'ler; state'e konup render'da fırlatılmadıkça), sunucu tarafı render hataları ve kendi içindeki hatalar.
</details>

---

## 4. State ve Render Süreci

<details>
<summary>🟢 <b>4.1 Bir bileşen ne zaman yeniden render olur?</b></summary>

1. Kendi state'i değiştiğinde.
2. Ebeveyni render olduğunda (props değişmese bile).
3. Okuduğu bir context değeri değiştiğinde.

Render, bileşen fonksiyonunun çalışması demektir; DOM'a sadece değişen kısımlar uygulanır.
</details>

<details>
<summary>🟢 <b>4.2 "Render" ile "commit" arasındaki fark nedir?</b></summary>

- **Render:** Bileşen fonksiyonları çalışır, yeni sanal ağaç üretilir ve öncekiyle karşılaştırılır. Saf olmalıdır; kesilebilir.
- **Commit:** Farklar gerçek DOM'a uygulanır, ref'ler ayarlanır, ardından effect'ler çalışır.
</details>

<details>
<summary>🟡 <b>4.3 <code>setState</code> neden hemen güncellenmiş değeri vermez?</b></summary>

State her render için bir **anlık görüntüdür (snapshot)**. `setSayi(sayi + 1)` mevcut değişkeni değiştirmez, bir sonraki render için istek bırakır. Bu yüzden aynı olay içinde `console.log(sayi)` eski değeri gösterir. Yeni değer gerekiyorsa bir değişkende hesaplanır.
</details>

<details>
<summary>🟡 <b>4.4 Updater fonksiyon (<code>setSayi(s =&gt; s + 1)</code>) ne zaman gereklidir?</b></summary>

Yeni değer önceki değere dayanıyorsa ve aynı olayda birden çok güncelleme, zamanlayıcı veya async kod varsa. `setSayi(sayi + 1)` üç kez çağrılınca sonuç 1 artar (hepsi aynı snapshot'ı görür); `setSayi(s => s + 1)` üç kez çağrılınca 3 artar (kuyrukta sırayla uygulanır).
</details>

<details>
<summary>🟡 <b>4.5 Batching nedir?</b></summary>

React'in aynı olaydaki birden çok state güncellemesini toplayıp **tek bir render** yapmasıdır. React 18'den beri `setTimeout`, Promise ve native olay yöneticilerinde de otomatiktir. Nadiren hemen DOM güncellemesi gerekirse `flushSync` kullanılır.
</details>

<details>
<summary>🟢 <b>4.6 State neden doğrudan değiştirilmemelidir (immutability)?</b></summary>

React değişikliği referans karşılaştırmasıyla (`Object.is`) anlar. `kisi.yas = 26; setKisi(kisi)` aynı referansı verdiği için React değişiklik görmez ve render'ı atlayabilir. Ayrıca önceki render'ın verisi bozulur. Doğru yol yeni nesne/dizi oluşturmaktır: `setKisi({ ...kisi, yas: 26 })`, `setListe([...liste, yeni])`.
</details>

<details>
<summary>🟡 <b>4.7 Hangi değerler state olmamalıdır?</b></summary>

- Zaman içinde değişmeyenler (sabitler).
- Props'tan gelenler (props'u state'e kopyalamak bayat veri yaratır).
- Mevcut state/props'tan **hesaplanabilenler** (`tamAd`, filtrelenmiş liste, toplam): bunlar render sırasında türetilir.

State minimal tutulur; ayrıca seçili bir nesnenin kopyası yerine **id'si** saklanır.
</details>

<details>
<summary>🟡 <b>4.8 State'i nasıl sıfırlarsın? <code>key</code> bu konuda nasıl kullanılır?</b></summary>

React state'i bileşenin ağaçtaki **konumu ve türüyle** eşler. Bir bileşene farklı bir `key` verildiğinde React onu tamamen yeni bir bileşen sayar ve state'ini sıfırlar: `<Profil key={kullaniciId} />`. "Prop değişince state'i sıfırla" ihtiyacının effect'ten daha temiz çözümüdür. → [Konu 8.12](./08-listeler-ve-key.md)
</details>

<details>
<summary>🟡 <b>4.9 "Lifting state up" (state'i yukarı taşıma) nedir?</b></summary>

İki kardeş bileşen aynı veriyi paylaşacaksa, state en yakın ortak ebeveyne taşınır; ebeveyn değeri props ile verir ve değiştirme fonksiyonunu callback olarak geçirir. Tek doğruluk kaynağı ilkesinin uygulamasıdır. → [Konu 12](./12-bilesenler-arasi-iletisim.md)
</details>

---

## 5. Listeler, Key'ler ve Koşullu Render

<details>
<summary>🟢 <b>5.1 <code>key</code> nedir, neden gereklidir?</b></summary>

`key`, liste elemanının React için **kimliğidir**. Reconciliation sırasında React, eleman eklendiğinde, silindiğinde veya yer değiştirdiğinde hangisinin hangisi olduğunu key ile anlar; böylece doğru elemanların state'ini korur ve gereksiz DOM işlemlerini önler. Kardeşler arasında benzersiz ve kararlı olmalıdır.
</details>

<details>
<summary>🟡 <b>5.2 Index'i key olarak kullanmak neden sorunludur? Ne zaman kabul edilebilir?</b></summary>

Liste başına ekleme, sıralama, filtreleme veya ortadan silme yapıldığında index'ler kayar; React elemanları yanlış eşleştirir ve satırların state'i (input değerleri, checkbox'lar, odak) **yanlış elemanlara** geçer. Liste statikse, elemanların state'i yoksa ve benzersiz kimlik yoksa kabul edilebilir.
</details>

<details>
<summary>🟡 <b>5.3 <code>key={Math.random()}</code> veya <code>key={crypto.randomUUID()}</code> neden yanlıştır?</b></summary>

Her render'da yeni key üretildiği için React tüm elemanları silip baştan oluşturur: state'ler sıfırlanır, input'lar her tuşta odağı kaybeder, performans çöker. Kimlik veriyle birlikte, **oluşturulduğu anda bir kez** üretilmelidir.
</details>

<details>
<summary>🟢 <b>5.4 Koşullu render yöntemleri nelerdir?</b></summary>

Erken `return` (guard clause), `null` döndürmek, değişkene atama, ternary (`a ? <X/> : <Y/>`), mantıksal `&&` ve çoklu durumlar için `switch` / nesne eşleme. İç içe ternary'den kaçınılır. → [Konu 7](./07-kosullu-render.md)
</details>

<details>
<summary>🟡 <b>5.5 Bir bileşeni koşullu render ile gizlemek ile CSS ile gizlemek arasındaki fark nedir?</b></summary>

- Koşullu render (`{acik && <X />}`): Bileşen ağaçtan çıkar, **state'i silinir**, effect'leri temizlenir.
- CSS (`display: none` / `hidden`): Bileşen ağaçta kalır, **state korunur**, ama effect'leri çalışmaya devam eder.
- React 19.2 `<Activity mode="hidden">`: State korunur, effect'ler temizlenir, güncellemeler düşük öncelikli olur.
</details>

---

## 6. Olaylar ve Formlar

<details>
<summary>🟢 <b>6.1 <code>onClick={handleClick}</code> ile <code>onClick={handleClick()}</code> arasındaki fark nedir?</b></summary>

İlki fonksiyonu **verir**; React tıklanınca çağırır. İkincisi fonksiyonu **render sırasında çağırır** ve dönüş değerini (`undefined`) verir. State güncelleyen bir fonksiyonsa sonsuz render döngüsüne ("Too many re-renders") yol açar. Argüman gerekiyorsa: `onClick={() => handleSil(id)}`.
</details>

<details>
<summary>🟡 <b>6.2 SyntheticEvent nedir? React olayları nasıl bağlar?</b></summary>

SyntheticEvent, tarayıcının yerel olay nesnesini saran ve tüm tarayıcılarda aynı davranan bir sarmalayıcıdır (`e.nativeEvent` ile yerele ulaşılır). React olay dinleyicilerini her elemente değil, **kök elemente** tek bir dinleyiciyle bağlar (event delegation); React 17'den beri `document` yerine kök container'a.
</details>

<details>
<summary>🟢 <b>6.3 <code>preventDefault</code> ile <code>stopPropagation</code> farkı nedir?</b></summary>

- `preventDefault`: Tarayıcının **varsayılan davranışını** durdurur (form gönderiminde sayfa yenilenmesi, link gidişi).
- `stopPropagation`: Olayın ağaçta **yukarı yayılmasını** durdurur (ör. modal içindeki tıklama arka planı kapatmasın).
</details>

<details>
<summary>🟢 <b>6.4 <code>e.target</code> ile <code>e.currentTarget</code> farkı nedir?</b></summary>

`e.target` olayın başladığı en içteki elementtir; `e.currentTarget` olay yöneticisinin bağlı olduğu elementtir. Yayılım nedeniyle farklı olabilirler.
</details>

<details>
<summary>🟡 <b>6.5 React'te <code>onChange</code>, DOM'daki <code>change</code> olayından nasıl farklıdır?</b></summary>

DOM'daki `change` olayı input odaktan çıkınca tetiklenir; React'in `onChange`'i **her tuş vuruşunda** tetiklenir (DOM'daki `input` olayı gibi). Bu sayede değer anında state'e yansıtılabilir.
</details>

<details>
<summary>🟡 <b>6.6 Bir formda doğrulama hatalarını ne zaman göstermek gerekir?</b></summary>

Kullanıcı daha yazmadan hata göstermek kötü bir deneyimdir. Yaygın yaklaşım: hatayı alan odaktan çıktıktan (**blur / touched**) veya form gönderilmeye çalışıldıktan sonra göstermek. Hatalar form değerlerinden **türetilir**; "hangi alana dokunuldu" bilgisi state'tir. Erişilebilirlik için `aria-invalid` ve `aria-describedby` kullanılır. İstemci doğrulaması UX içindir; sunucu her zaman tekrar doğrular.
</details>

<details>
<summary>🟡 <b>6.7 Büyük formları yönetmek için hangi araçları kullanırsın?</b></summary>

Küçük formlarda `useState` veya `FormData` yeterlidir. Büyük formlarda **React Hook Form** (uncontrolled, az render) ve şema tabanlı doğrulama için **Zod** yaygındır. React 19'da `<form action>`, `useActionState` ve `useFormStatus` bekleme ve hata durumunu yerleşik olarak yönetir.
</details>

---

## 7. Hook'lar

<details>
<summary>🟢 <b>7.1 Hook nedir? Hook kuralları nelerdir?</b></summary>

Hook'lar, fonksiyon bileşenlerinin React özelliklerini (state, effect, context, ref...) kullanmasını sağlayan `use` ile başlayan fonksiyonlardır. İki kural:
1. Sadece **en üst seviyede** çağrılır: döngü, koşul, iç içe fonksiyon veya erken `return` sonrasında çağrılmaz.
2. Sadece **React fonksiyon bileşenlerinde veya custom Hook'larda** çağrılır.

(İstisna: `use` API'si koşul içinde çağrılabilir.)
</details>

<details>
<summary>🟡 <b>7.2 Hook'lar neden koşul içinde çağrılamaz?</b></summary>

React, bir bileşenin Hook'larını isimleriyle değil **çağrılma sırasıyla** tanır (ilk `useState` → 1. kutu, ikinci → 2. kutu). Koşul nedeniyle bir render'da bir Hook atlanırsa sıra kayar ve state'ler yanlış değişkenlere eşlenir.
</details>

<details>
<summary>🟢 <b>7.3 <code>useRef</code> ile <code>useState</code> arasındaki fark nedir?</b></summary>

İkisi de render'lar arasında değer saklar. Fark: State değişimi **yeniden render tetikler**, ref değişimi (`ref.current = x`) tetiklemez. Ref, ekranda gösterilmeyen değerler (zamanlayıcı id'si, önceki değer, DOM elemanı) için kullanılır. Render sırasında `ref.current` okunmaz/yazılmaz. → [Konu 11](./11-useref.md)
</details>

<details>
<summary>🟡 <b>7.4 <code>useMemo</code> ile <code>useCallback</code> farkı nedir?</b></summary>

`useMemo` bir hesaplamanın **sonucunu**, `useCallback` bir **fonksiyonun kendisini** render'lar arasında saklar. `useCallback(fn, deps)` ≡ `useMemo(() => fn, deps)`. Kullanım alanları: ölçülebilir şekilde pahalı hesaplamalar ve `memo`'lu bileşenlere veya bağımlılık dizilerine verilen referansları sabitlemek. React Compiler açıkken çoğu durumda gerekmez. → [Konu 19](./19-performans.md)
</details>

<details>
<summary>🟡 <b>7.5 <code>useReducer</code>'ı ne zaman <code>useState</code>'e tercih edersin?</b></summary>

- Birbirine bağlı birden çok state parçası birlikte değişiyorsa,
- Sonraki state birden fazla farklı olaya göre karmaşık kurallarla hesaplanıyorsa,
- Güncelleme mantığını bileşenden ayırıp **test etmek** istiyorsan,
- State'i Context ile alt bileşenlere dağıtacaksan (`dispatch` sabittir).

Basit, bağımsız değerler için `useState` yeterlidir. → [Konu 14](./14-usereducer.md)
</details>

<details>
<summary>🟡 <b>7.6 Custom Hook nedir? İki bileşen aynı custom Hook'u kullanınca state'i paylaşırlar mı?</b></summary>

`use` ile başlayan ve içinde başka Hook'lar çağıran fonksiyondur; tekrar eden **mantığı** paylaşır. State'i **paylaşmazlar**: Her bileşenin Hook çağrısı kendi bağımsız state'ini oluşturur. State paylaşımı için state'i yukarı taşımak veya context kullanmak gerekir. → [Konu 15](./15-custom-hooks.md)
</details>

<details>
<summary>🟡 <b>7.7 <code>useContext</code> nasıl çalışır? Varsayılan değer ne zaman kullanılır?</b></summary>

`useContext(Ctx)` ağaçta yukarı doğru en yakın sağlayıcının (`<Ctx value>`) değerini okur ve değer değişince bileşeni yeniden render eder. Varsayılan değer (`createContext(varsayilan)`) yalnızca ağaçta **hiç sağlayıcı yoksa** kullanılır. Yaygın kalıp: varsayılanı `null` yapıp, sağlayıcı yoksa hata fırlatan bir custom Hook yazmak.
</details>

<details>
<summary>🔴 <b>7.8 <code>useLayoutEffect</code> ne zaman <code>useEffect</code>'e tercih edilir?</b></summary>

`useEffect` tarayıcı ekranı çizdikten sonra çalışır. `useLayoutEffect` DOM güncellendikten sonra ama **ekran çizilmeden önce**, senkron çalışır. Bir elemanın ölçüsünü okuyup ona göre konum hesaplamak (tooltip yerleşimi) gibi, kullanıcının titreme (flicker) görmemesi gereken durumlarda kullanılır. Ekranı bloke ettiği için nadiren kullanılmalıdır.
</details>

<details>
<summary>🟡 <b>7.9 <code>useId</code> ne işe yarar?</b></summary>

Sunucu ve istemci render'larında **tutarlı**, benzersiz id'ler üretir. `<label htmlFor>` ile input'u veya `aria-describedby` ile açıklamayı bağlamak için kullanılır. Liste key'i üretmek için **kullanılmaz**.
</details>

<details>
<summary>🔴 <b>7.10 <code>useSyncExternalStore</code> nedir?</b></summary>

React dışındaki bir veri kaynağına (tarayıcı API'leri, global store kütüphaneleri) güvenle abone olmak için kullanılır. `subscribe` ve `getSnapshot` fonksiyonları alır; eşzamanlı render sırasında tutarsız değer (tearing) oluşmasını önler. Zustand, Redux gibi kütüphaneler içeride bunu kullanır.
</details>

---

## 8. useEffect ve Yan Etkiler

<details>
<summary>🟢 <b>8.1 <code>useEffect</code> ne işe yarar?</b></summary>

Bileşeni React dışındaki bir sistemle **senkronize etmek** için kullanılır: ağ bağlantıları, abonelikler, zamanlayıcılar, tarayıcı API'leri, React dışı kütüphaneler. Render'dan sonra (commit sonrası) çalışır. → [Konu 10](./10-useeffect.md)
</details>

<details>
<summary>🟢 <b>8.2 Bağımlılık dizisinin üç hali nasıl davranır?</b></summary>

- Dizi yok: Her render'dan sonra çalışır.
- `[]`: Sadece bileşen ekrana geldiğinde bir kez (ve cleanup ekrandan kalkınca).
- `[a, b]`: İlk render'da ve `a` veya `b` değiştiğinde.

Bağımlılıklar "ne zaman çalışsın" tercihi değil, effect'in **kullandığı reaktif değerlerin** listesidir; `exhaustive-deps` lint kuralına uyulmalıdır.
</details>

<details>
<summary>🟢 <b>8.3 Cleanup fonksiyonu nedir? Ne zaman çalışır?</b></summary>

Effect'in döndürdüğü fonksiyondur; kurulanı geri alır (aboneliği bitirir, zamanlayıcıyı temizler, bağlantıyı kapatır). Bileşen ekrandan kalkarken ve effect **bir sonraki kez çalışmadan önce** (bağımlılık değişince) çalışır.
</details>

<details>
<summary>🟡 <b>8.4 StrictMode'da effect'ler neden iki kez çalışıyor? Nasıl düzeltirsin?</b></summary>

Geliştirmede React, eksik cleanup'ları ortaya çıkarmak için effect'i setup → cleanup → setup sırasıyla çalıştırır. Çözüm iki kez çalışmayı engellemek değil, **doğru cleanup** yazmaktır: bağlantıyı kapatmak, isteği `AbortController` ile iptal etmek, zamanlayıcıyı temizlemek. Doğru cleanup varsa kullanıcı farkı görmez.
</details>

<details>
<summary>🟡 <b>8.5 Effect içinde sonsuz döngü nasıl oluşur?</b></summary>

- Effect bir state'i güncelliyor ve o state bağımlılıklarda yer alıyorsa.
- Bağımlılık her render'da yeni oluşturulan bir nesne/dizi/fonksiyon ise.
- Bağımlılık dizisi verilmemiş ve effect state güncelliyorsa.

Çözümler: updater fonksiyon, nesneleri effect içine taşımak, primitive bağımlılıklar kullanmak, değeri effect yerine render'da türetmek.
</details>

<details>
<summary>🟡 <b>8.6 Effect içinde veri çekerken yarış durumu (race condition) nasıl önlenir? Effect fonksiyonu neden doğrudan <code>async</code> olamaz?</b></summary>

Effect fonksiyonu sadece cleanup fonksiyonu döndürebilir; `async` fonksiyon Promise döndürür. Bu yüzden içeride async bir fonksiyon tanımlanıp çağrılır.

Yarış durumu: Kullanıcı hızlıca A'dan B'ye geçerse A'nın geç gelen cevabı B'nin üzerine yazılabilir. Çözüm: cleanup'ta `AbortController` ile isteği iptal etmek veya bir `iptal` bayrağıyla eski cevabı yok saymak. Gerçek projelerde bunu TanStack Query veya router loader'ları halleder.

```jsx
useEffect(() => {
  const ctrl = new AbortController();
  (async () => {
    try {
      const res = await fetch(`/api/kullanici/${id}`, { signal: ctrl.signal });
      setKullanici(await res.json());
    } catch (err) {
      if (err.name !== "AbortError") setHata(err);
    }
  })();
  return () => ctrl.abort();
}, [id]);
```
</details>

<details>
<summary>🟡 <b>8.7 Hangi durumlarda effect'e ihtiyaç yoktur?</b></summary>

- Props/state'ten **hesaplanabilen** değerler için (render sırasında hesapla).
- Kullanıcı eylemine bağlı işler için (olay yöneticisinde yap: form gönderimi, API çağrısı, bildirim).
- Prop değişince state'i sıfırlamak için (`key` kullan).
- Ebeveyne veri bildirmek için (olay yöneticisinde ikisini birlikte güncelle veya state'i yukarı taşı).

Kural: Effect, sadece bileşen **ekranda olduğu için** gereken senkronizasyonlar içindir. → [Konu 10.10](./10-useeffect.md)
</details>

<details>
<summary>🔴 <b>8.8 <code>useEffectEvent</code> hangi problemi çözer?</b></summary>

Effect'in içinde güncel bir değeri **okumak** istediğin ama o değer değişince effect'in **yeniden çalışmasını istemediğin** durumlar için (ör. sohbet odasına bağlanırken güncel temayı okuyup bildirim göstermek). `useEffectEvent` ile tanımlanan fonksiyon her zaman en güncel props/state'i görür ama reaktif değildir; bağımlılık dizisine eklenmez. React 19.2'de kararlı hale geldi.
</details>

---

## 9. State Yönetimi ve Mimari

<details>
<summary>🟡 <b>9.1 State nerede tutulmalı? Colocation nedir?</b></summary>

State, onu kullanan **en alttaki ortak ebeveynde** tutulmalıdır (colocation). Gereğinden yukarıda tutmak gereksiz render'lara ve karmaşıklığa yol açar; aşağıda tutmak paylaşımı imkânsız kılar. Sıralama: yerel state → yukarı taşıma → kompozisyon → context → global store.
</details>

<details>
<summary>🟡 <b>9.2 Context bir state yönetim aracı mıdır? Context'in performans sorunu nedir?</b></summary>

Context aslında bir **taşıma (dependency injection)** mekanizmasıdır; state'i `useState`/`useReducer` tutar, context onu ağaca dağıtır. Sorun: Sağlayıcının değeri değişince o context'i okuyan **tüm** bileşenler yeniden render olur ve seçici abonelik (sadece bir alana abone olmak) yoktur. Çözümler: değeri `useMemo` ile sabitlemek (veya React Compiler), context'leri bölmek (state ve dispatch ayrı), sık değişen verileri context'e koymamak.
</details>

<details>
<summary>🟡 <b>9.3 Context, Redux (Toolkit) ve Zustand arasında nasıl seçim yaparsın?</b></summary>

- **Context + useState/useReducer:** Nadiren değişen global veriler (tema, dil, oturum). Ek bağımlılık yok.
- **Zustand / Jotai:** Basit API, seçici abonelik (sadece kullanılan alan değişince render), az kod. Orta ölçek için popüler.
- **Redux Toolkit:** Büyük ekipler, katı yapı, gelişmiş DevTools (zaman yolculuğu), middleware; RTK Query ile veri çekme.

Önemli nokta: **Sunucu verisi** (API'den gelenler) için bunların yerine TanStack Query gibi araçlar kullanılır; global store'a genellikle sadece istemci state'i kalır.
</details>

<details>
<summary>🟡 <b>9.4 Sunucu state'i ile istemci state'i arasındaki fark nedir?</b></summary>

- **İstemci state'i:** Tarayıcıya ait, senkron, her zaman güncel (modal açık mı, seçili sekme, form taslağı).
- **Sunucu state'i:** Asıl sahibi uzaktaki sunucu; elindeki bir kopyadır, bayatlayabilir, asenkron ve başarısız olabilir. Önbellek, yeniden doğrulama, tekrarlanan isteklerin birleştirilmesi gibi ihtiyaçları vardır → TanStack Query, SWR, RTK Query. → [Konu 18](./18-veri-cekme.md)
</details>

<details>
<summary>🟡 <b>9.5 State yapısını tasarlarken hangi ilkelere dikkat edersin?</b></summary>

- Birlikte değişen değerleri gruplamak.
- Çelişkili state'lerden kaçınmak (`yukleniyor` + `hata` + `basarili` booleanları yerine tek bir `durum`).
- Gereksiz/türetilebilir state tutmamak.
- Tekrarlanan veri tutmamak (nesne yerine id).
- Derin iç içe yapılar yerine **normalize** etmek (id → nesne haritası).
</details>

<details>
<summary>🟡 <b>9.6 Büyük bir React projesinde klasör yapısını nasıl kurarsın?</b></summary>

Tek doğru yok; yaygın ve ölçeklenen yaklaşım **özellik (feature) bazlı** yapıdır:

```
src/
├── features/
│   ├── auth/      (components, hooks, api, types)
│   └── sepet/
├── components/    (paylaşılan UI: Buton, Modal)
├── hooks/         (paylaşılan Hook'lar)
├── lib/           (api istemcisi, yardımcılar)
└── routes/ veya app/
```

Bir özellikle ilgili her şeyin bir arada olması, değişikliklerin dağınık olmasını önler.
</details>

---

## 10. Performans

<details>
<summary>🟢 <b>10.1 Bir React uygulamasının yavaş olduğunu fark ettin. Nasıl ilerlersin?</b></summary>

1. **Ölç:** Production build'de, CPU yavaşlatmayla; React DevTools Profiler ("why did this render?"), Chrome Performance paneli, Lighthouse / Web Vitals.
2. **Sorunu sınıflandır:** Gereksiz render mı, pahalı hesaplama mı, büyük liste mi, büyük bundle mı, ağ mı?
3. **Ucuz çözümlerle başla:** State'i aşağı taşı, içeriği `children` ile yukarı kaldır.
4. Gerekirse: `memo`/`useMemo`/`useCallback` (veya React Compiler), `useTransition`/`useDeferredValue`, sanallaştırma, code splitting, görsel optimizasyonu, önbellek.
5. Tekrar ölç. → [Konu 19](./19-performans.md)
</details>

<details>
<summary>🟡 <b>10.2 <code>React.memo</code> ne yapar? Ne zaman işe yaramaz?</b></summary>

Bileşeni sarar; props'lar **sığ olarak** aynıysa, ebeveyn render olsa bile yeniden render'ı atlar. İşe yaramadığı durumlar: Her render'da yeni oluşturulan nesne/dizi/fonksiyon props'u (referans değişir), `children` almak (JSX her render'da yenidir), bileşenin kendi state'i veya okuduğu context değişmesi.
</details>

<details>
<summary>🟡 <b>10.3 Her şeyi <code>useMemo</code> / <code>useCallback</code> ile sarmak neden iyi bir fikir değildir?</b></summary>

Memoizasyon bedava değildir: her render'da bağımlılık karşılaştırması, bellekte saklama, daha karmaşık kod ve yanlış bağımlılıkta bayat değer hataları. Çoğu hesaplama mikrosaniyeler sürer. Önce ölçülür; sadece pahalı hesaplamalarda ve referansın önemli olduğu yerlerde kullanılır.
</details>

<details>
<summary>🟡 <b>10.4 React Compiler nedir?</b></summary>

Kodu build sırasında analiz edip `memo`, `useMemo`, `useCallback`'in yaptığı işi **otomatik** ekleyen derleyicidir (1.0 kararlı sürüm Ekim 2025). Elle memoizasyondan daha ince taneli çalışır. Şartı: Kodun React kurallarına uyması (saf bileşenler, mutasyon yok, Hook kuralları). Pahalı algoritmaları, büyük DOM'u, bundle boyutunu veya ağı çözmez.
</details>

<details>
<summary>🟡 <b>10.5 <code>useTransition</code> ve <code>useDeferredValue</code> ne işe yarar? Debounce'tan farkı nedir?</b></summary>

İkisi de güncellemeleri **acil** (yazmak, tıklamak) ve **acil olmayan** (sonuç listesi, sekme içeriği) olarak ayırır; acil olmayan render yeni bir güncelleme gelirse yarıda bırakılabilir, böylece arayüz donmaz.
- `useTransition`: Bir state güncellemesini işaretler, `isPending` verir.
- `useDeferredValue`: Bir değerin gecikmeli kopyasını verir.

Debounce sabit bir süre bekler; bunlar cihaz hızına uyum sağlar. Ama ağ isteği sayısını azaltmaz; API isteklerini azaltmak için debounce hâlâ gereklidir.
</details>

<details>
<summary>🟡 <b>10.6 10.000 satırlık bir listeyi nasıl performanslı gösterirsin?</b></summary>

Önce gerçekten gerekli mi diye sorulur: sayfalama, filtreleme/arama veya "daha fazla yükle" çoğu zaman daha iyidir. Gerekliyse **sanallaştırma** (TanStack Virtual, react-window): sadece görünen satırlar DOM'da tutulur. Satır bileşenleri `memo`'lanır ve props'ları sabitlenir; filtreleme `useDeferredValue` ile ertelenebilir.
</details>

<details>
<summary>🟡 <b>10.7 Code splitting nedir? React'te nasıl yapılır?</b></summary>

Bundle'ı parçalara bölüp kodun sadece gerektiğinde indirilmesidir. `const Grafik = lazy(() => import("./Grafik"))` + `<Suspense fallback>` ile bileşen düzeyinde; router'larda rota düzeyinde (`lazy` rota özelliği); ağır kütüphaneler olay içinde `await import("xlsx")` ile yapılır. Bundle analizi için `vite-bundle-visualizer` kullanılır.
</details>

<details>
<summary>🟡 <b>10.8 Core Web Vitals nelerdir?</b></summary>

- **LCP** (Largest Contentful Paint): Ana içeriğin görünme süresi (≤ 2,5 sn).
- **INP** (Interaction to Next Paint): Etkileşime tepki süresi (≤ 200 ms); React render performansıyla en ilişkili olanı.
- **CLS** (Cumulative Layout Shift): Yüklenirken sayfa kayması (≤ 0,1); görsellere boyut vermek önemli.
</details>

---

## 11. Yönlendirme ve Veri Çekme

<details>
<summary>🟢 <b>11.1 İstemci tarafı yönlendirme nasıl çalışır? Neden <code>&lt;a&gt;</code> yerine <code>&lt;Link&gt;</code> kullanılır?</b></summary>

Linke tıklanınca sayfa yenilemesi engellenir, URL History API ile güncellenir ve React yeni URL'ye karşılık gelen bileşenleri render eder; uygulama state'i korunur. `<a>` tam sayfa yenilemesi yapar ve state'i kaybettirir; `<Link>` ise yine `<a>` üretir ama tıklamayı yakalar. → [Konu 17](./17-react-router.md)
</details>

<details>
<summary>🟡 <b>11.2 Hangi state'ler URL'de tutulmalıdır?</b></summary>

Paylaşılabilir ve yenilemede korunması gereken her şey: filtreler, sıralama, sayfa numarası, arama sorgusu, seçili sekme. Ölçüt: "Bu linki birine gönderdiğimde aynı ekranı görmeli mi?" Modal açık/kapalı, hover gibi geçici durumlar `useState`'te kalır.
</details>

<details>
<summary>🟡 <b>11.3 React Router'da loader ve action nedir? Effect ile veri çekmekten farkı nedir?</b></summary>

- **Loader:** Rota render edilmeden **önce** veriyi yükler; tüm eşleşen rotaların loader'ları paralel çalışır (şelale yok), bileşen `useLoaderData` ile hazır veriyi okur.
- **Action:** `<Form>` gönderimlerini işler; sonrasında loader'lar otomatik yenilenir.

Effect ile çekmeye göre: şelale yok, yükleniyor/hata state'leri router tarafından yönetilir, yarış durumları otomatik çözülür. Eksiği: önbellek tutmaz.
</details>

<details>
<summary>🟡 <b>11.4 TanStack Query neden kullanılır? Ne sağlar?</b></summary>

Sunucu verisini merkezi bir **önbellekte** yönetir: aynı veriyi isteyen bileşenler tek istek atar; bayat veri gösterilirken arka planda yenilenir; pencere odağında ve yeniden bağlanmada tazelenir; başarısız istekler tekrar denenir; sayfalama, sonsuz kaydırma, mutasyon sonrası geçersiz kılma ve iyimser güncellemeler hazır gelir. Elle yazılan `useEffect` + `useState` kodunun yerini alır.
</details>

<details>
<summary>🟡 <b>11.5 <code>staleTime</code> ile <code>gcTime</code> farkı nedir?</b></summary>

- `staleTime` (varsayılan 0): Veri ne kadar süre **taze** sayılır; tazeyken yeni istek atılmaz.
- `gcTime` (varsayılan 5 dk): Kullanılmayan verinin önbellekte **ne kadar saklanacağı**.

Bayat veri yine gösterilir ama belirli olaylarda (mount, pencere odağı, yeniden bağlanma) arka planda yenilenir.
</details>

<details>
<summary>🟡 <b>11.6 Query key nedir? Neden önemlidir?</b></summary>

Önbellekteki verinin kimliğidir (dizi): aynı key aynı veriyi paylaşır; key'deki bir değer değişince yeni sorgu başlar. `queryFn`'de kullanılan her değişken key'de olmalıdır. Genelden özele yazılan key'ler (`["urunler", "detay", 5]`) toplu geçersiz kılmayı kolaylaştırır.
</details>

<details>
<summary>🔴 <b>11.7 İyimser güncelleme (optimistic update) nedir? Nasıl uygulanır?</b></summary>

Sunucu cevabını beklemeden arayüzü değişiklik başarılı olmuş gibi güncellemek, hata olursa geri almaktır. Uygulama yolları:
- React 19 `useOptimistic`: Action bitince iyimser değer otomatik atılır.
- TanStack Query: `onMutate`'te sorguları iptal et ve yedek al → önbelleğe yaz; `onError`'da yedeği geri yükle; `onSettled`'da geçersiz kıl. Ya da basitçe mutasyonun `variables`'ını göstermek.

Hızlı ve nadiren başarısız olan işlemler (beğeni, favori) için uygundur; ödeme gibi kritik işlemlerde kullanılmaz.
</details>

<details>
<summary>🟡 <b>11.8 Korumalı rotaları nasıl uygularsın?</b></summary>

Router düzeyinde (loader'da oturumu kontrol edip `redirect`, Next.js'te `proxy.ts` veya sayfada kontrol) veya düzen bileşeninde `<Navigate>` ile. Giriş sonrası kullanıcıyı geldiği sayfaya döndürmek için hedef URL query parametresinde taşınır. **Önemli:** İstemci tarafı koruma sadece arayüz içindir; gerçek yetki kontrolü sunucuda, veriye erişilen yerde yapılmalıdır.
</details>

---

## 12. React 19 ve Modern Özellikler

<details>
<summary>🟡 <b>12.1 React 19'daki "Actions" kavramı nedir?</b></summary>

Async fonksiyonların transition içinde çalıştırılmasıdır (`startTransition(async () => ...)`). Bekleme durumu (`isPending`), hatalar, formun sıfırlanması ve iyimser güncellemeler React tarafından yönetilir. `<form action={fn}>` ile formlar doğrudan bir fonksiyona (`FormData` alarak) gönderilebilir. → [Konu 20](./20-react-19.md)
</details>

<details>
<summary>🟡 <b>12.2 <code>useActionState</code>, <code>useFormStatus</code> ve <code>useOptimistic</code> ne işe yarar?</b></summary>

- `useActionState(fn, baslangic)` → `[durum, formAction, isPending]`: Action'ın son sonucunu (doğrulama hataları, mesajlar) ve bekleme durumunu tutar; fn `(oncekiDurum, formData)` alır.
- `useFormStatus()` (`react-dom`): İçinde bulunduğu formun `pending` durumunu props'suz okur (tekrar kullanılabilir gönder butonları).
- `useOptimistic(state, birlestir)`: Action sürerken iyimser değer gösterir, bitince gerçek değere döner.
</details>

<details>
<summary>🟡 <b>12.3 <code>use</code> API'si nedir? Diğer Hook'lardan farkı nedir?</b></summary>

Render sırasında bir Promise'i veya Context'i okur. Promise bekliyorsa bileşen Suspense ile askıya alınır, reddedilirse Error Boundary yakalar. Diğer Hook'lardan farklı olarak **koşul içinde** çağrılabilir. Promise render sırasında oluşturulmamalıdır (her render'da yeni Promise sonsuz döngü yaratır); genellikle sunucudan, loader'dan veya bir önbellekten gelir.
</details>

<details>
<summary>🟡 <b>12.4 Suspense nedir?</b></summary>

Alt ağacındaki bir bileşen "hazır değilim" dediğinde (tembel yüklenen kod, `use` ile bekleyen Promise, `useSuspenseQuery`) yedek bir arayüz (`fallback`) gösteren sınırdır. Yükleme durumlarını bileşenin içinden çıkarıp **deklaratif** olarak ağaçta yerleştirmeyi sağlar; sunucuda streaming'in de temelidir.
</details>

<details>
<summary>🟡 <b>12.5 React 19 ile <code>forwardRef</code> neden gereksizleşti?</b></summary>

React 19'da fonksiyon bileşenleri `ref`'i **normal bir prop** olarak alabilir: `function Input({ ref, ...props }) { return <input ref={ref} {...props} /> }`. `forwardRef` hâlâ çalışır ama kullanımdan kalkmaktadır. Benzer şekilde `<Context.Provider>` yerine `<Context value>` yazılabilir.
</details>

<details>
<summary>🔴 <b>12.6 <code>&lt;Activity&gt;</code> ve <code>&lt;ViewTransition&gt;</code> ne işe yarar?</b></summary>

- `<Activity mode="visible|hidden">` (19.2): İçeriği gizlerken **state'ini korur** ama effect'lerini temizler ve güncellemelerini düşük önceliğe alır; sekmeler ve önceden render için.
- `<ViewTransition>` (19.3): Tarayıcının View Transition API'siyle elemanların giriş, çıkış, güncelleme ve paylaşımlı geçişlerini animasyonlar; sadece transition olarak işaretlenmiş güncellemelerde tetiklenir.
</details>

---

## 13. SSR, Server Components ve Next.js

<details>
<summary>🟢 <b>13.1 CSR, SSR ve SSG arasındaki farklar nelerdir?</b></summary>

| | Ne zaman, nerede render | Artı | Eksi |
|---|---|---|---|
| **CSR** | Tarayıcıda, her ziyarette | Basit, statik barındırma | Yavaş ilk ekran, SEO zayıf |
| **SSR** | Sunucuda, her istekte | Hızlı ilk ekran, güncel veri | Her istekte sunucu maliyeti |
| **SSG** | Build sırasında bir kez | En hızlı (CDN) | Veri değişince yeniden build |

Modern framework'ler bunları sayfa, hatta bileşen bazında karıştırır (önbellekli render, kısmi ön-render). → [Konu 23](./23-nextjs.md)
</details>

<details>
<summary>🟡 <b>13.2 Hydration nedir? Hydration uyuşmazlığı neden olur?</b></summary>

Sunucudan gelen HTML'e istemcide React'in bağlanıp olay yöneticilerini eklemesi ve sayfayı etkileşimli yapmasıdır. Uyuşmazlık, sunucuda üretilen HTML ile istemcideki ilk render farklı olduğunda olur: `Date.now()`/`Math.random()`, `window`/`localStorage` kontrolüne göre farklı dallar, saat dilimi/yerel ayar farkları, geçersiz HTML iç içeliği (`<p>` içinde `<div>`). Çözümler: deterministik render, tarayıcıya özgü kısmı effect'e veya React 19.3 `browser()` ile yalnızca istemciye taşımak.
</details>

<details>
<summary>🟡 <b>13.3 React Server Components (RSC) nedir? SSR ile aynı şey mi?</b></summary>

Aynı şey değildir:
- **SSR:** Bileşenlerin ilk HTML'ini sunucuda üretmek; tüm bileşenlerin kodu yine tarayıcıya gider ve hydrate olur.
- **Server Components:** **Sadece sunucuda** çalışan bileşenler; kodları tarayıcıya hiç gitmez, `async` olabilir, veritabanına doğrudan erişebilir. State, effect ve olay yöneticisi kullanamazlar.

İkisi birlikte kullanılır: Server Components ağacı üretir, Client Components SSR ile HTML'e dönüşüp tarayıcıda hydrate olur.
</details>

<details>
<summary>🟡 <b>13.4 <code>"use client"</code> ne yapar? "Client Component sadece tarayıcıda çalışır" doğru mu?</b></summary>

`"use client"` bir **modül sınırı** belirler: o dosya ve import ettiği her şey istemci paketine girer. "Sadece tarayıcıda çalışır" yanlıştır; Client Component'ler de ilk HTML için sunucuda render edilir, fark kodlarının tarayıcıya gönderilip hydrate olmasıdır. İyi pratik: etkileşimi yapraklara itmek, sayfanın büyük kısmını Server Component bırakmak.
</details>

<details>
<summary>🟡 <b>13.5 Server Component'ten Client Component'e hangi props verilebilir?</b></summary>

Serileştirilebilir değerler: string, number, boolean, null, düz nesne/dizi, `Date`, `Map`, `Set`, JSX elementleri, Promise'ler ve Server Function'lar. Normal fonksiyonlar (ör. `onClick` callback'i) ve sınıf örnekleri verilemez. Client Component bir Server Component'i import edemez ama onu `children` olarak alabilir.
</details>

<details>
<summary>🟡 <b>13.6 Server Actions / Server Functions nedir? Güvenlik açısından nelere dikkat edilir?</b></summary>

`"use server"` ile işaretlenen, sunucuda çalışan ama istemciden normal fonksiyon gibi çağrılabilen fonksiyonlardır; formlarla doğrudan kullanılabilir ve ayrı API uç noktası yazma ihtiyacını kaldırır. **Herkese açık HTTP uç noktalarıdır**: her birinde girdi doğrulaması (Zod) ve yetki kontrolü yapılmalı; istemcideki doğrulamaya veya gizlenmiş butonlara güvenilmemelidir.
</details>

<details>
<summary>🟡 <b>13.7 Streaming nedir? Next.js'te nasıl sağlanır?</b></summary>

Sunucunun sayfanın hazır kısımlarını hemen gönderip, yavaş kısımları hazır oldukça aynı bağlantı üzerinden akıtmasıdır. `<Suspense fallback>` sınırları ve segment düzeyinde `loading.tsx` ile sağlanır. Yavaş bir sorgu bütün sayfayı bekletmez.
</details>

<details>
<summary>🔴 <b>13.8 Next.js 16'da önbellekleme nasıl çalışır?</b></summary>

Önbellek **açık (opt-in)** hale geldi: varsayılan olarak her şey dinamiktir. `cacheComponents: true` ile `"use cache"` direktifi bir fonksiyonun veya bileşenin sonucunu önbelleğe alır; `cacheLife` süreyi, `cacheTag` etiketi belirler. Veri değişince `updateTag` (anında) veya `revalidateTag` (arka planda) ile geçersiz kılınır. `"use cache"` ve `<Suspense>` birlikte **kısmi ön-render** sağlar: statik iskelet anında, dinamik kısımlar streaming ile gelir.
</details>

<details>
<summary>🟡 <b>13.9 Next.js mi, Vite + React Router mı? Nasıl karar verirsin?</b></summary>

- **Vite + React Router + TanStack Query:** Giriş arkasındaki SPA'lar (paneller, SaaS, iç araçlar); SEO önemsiz; statik barındırma; daha basit zihinsel model.
- **Next.js:** İçerik ağırlıklı ve SEO'nun kritik olduğu siteler (e-ticaret, blog, pazarlama), ilk açılış hızı önemli, frontend ve backend aynı projede.

Alternatifler: React Router Framework modu, TanStack Start, Astro (çoğunlukla statik içerik).
</details>

---

## 14. TypeScript ile React

<details>
<summary>🟢 <b>14.1 TypeScript'in React projelerine faydaları nelerdir?</b></summary>

Yanlış/eksik props ve `undefined` erişimleri gibi hataları kod çalışmadan yakalar; otomatik tamamlama ve güvenli yeniden adlandırma sağlar; props tipleri bileşenin belgesi olur. Önemli: Vite tip kontrolü yapmaz; kontrol editörde, `tsc -b` (build) ve CI'da yapılır. → [Konu 22](./22-typescript.md)
</details>

<details>
<summary>🟢 <b>14.2 Props nasıl tiplenir? <code>React.FC</code> kullanmalı mıyız?</b></summary>

Fonksiyonun parametresi tiplenir: `function Kart({ baslik, children }: KartProps)`. `React.FC` gerekli değildir; eskiden `children`'ı otomatik eklemesi kafa karıştırıyordu ve generic bileşenlerle uyumu zayıftır. `children` için `ReactNode`, HTML elementini saran bileşenler için `ComponentProps<"button">` kullanılır.
</details>

<details>
<summary>🟡 <b>14.3 <code>type</code> ile <code>interface</code> farkı nedir?</b></summary>

React'te çoğu durumda birbirinin yerine kullanılabilir. `interface` genişletilebilir (declaration merging) ve `extends` ile kalıtım okunaklıdır; `type` birleşimler (`A | B`), kesişimler ve tip türetmede daha esnektir. Önemli olan projede tutarlılıktır.
</details>

<details>
<summary>🟡 <b>14.4 <code>useState</code> ne zaman açık generic ister?</b></summary>

Başlangıç değeri tipi tam anlatmadığında: `useState<Kullanici | null>(null)`, `useState<Urun[]>([])` (aksi halde `never[]` çıkarılır), literal birleşimler `useState<Durum>("bosta")`. Diğer durumlarda tip başlangıç değerinden çıkarılır.
</details>

<details>
<summary>🟡 <b>14.5 Discriminated union (ayırt edici birleşim) React'te nerede işe yarar?</b></summary>

- Reducer action'larında: Her `type` için gerekli alanlar tiplenir, yanlış `dispatch` yazılamaz; `never` ile tüm durumların ele alındığı kontrol edilir.
- Birbirine bağlı props'larda: `{ tur: "onay"; onOnayla: () => void } | { tur: "bilgi" }` gibi; geçersiz prop kombinasyonları derlenmez.
- Async durumlarda: `{ durum: "yukleniyor" } | { durum: "basarili"; veri: T } | { durum: "hata"; hata: string }`.
</details>

<details>
<summary>🟡 <b>14.6 API'den gelen veriye TypeScript tipi vermek yeterli mi?</b></summary>

Hayır. Tipler çalışma zamanında silinir; `res.json()` sonucunu `Urun` olarak işaretlemek veriyi kontrol etmez. Uygulamanın sınırında gelen veri `unknown` kabul edilip **çalışma zamanında doğrulanmalıdır** (Zod gibi şema kütüphaneleri; `z.infer` ile tip de şemadan üretilir).
</details>

<details>
<summary>🟡 <b>14.7 <code>any</code> ile <code>unknown</code> farkı nedir?</b></summary>

`any` tip kontrolünü tamamen kapatır, her işleme izin verir. `unknown` "ne olduğunu bilmiyorum" der; kullanmadan önce daraltma (`typeof`, `instanceof`, şema doğrulama) zorunludur. Bilinmeyen veri için her zaman `unknown` tercih edilir.
</details>

<details>
<summary>🟡 <b>14.8 Olay ve ref tipleri nasıl yazılır?</b></summary>

```tsx
function handleChange(e: React.ChangeEvent<HTMLInputElement>) { /* ... */ }
function handleSubmit(e: React.FormEvent<HTMLFormElement>) { /* ... */ }
const inputRef = useRef<HTMLInputElement>(null);
```

Inline handler'larda (`onChange={(e) => ...}`) tip otomatik çıkarılır.
</details>

---

## 15. Test

<details>
<summary>🟢 <b>15.1 Unit, integration ve E2E testleri arasındaki fark nedir? Hangisinden daha çok yazarsın?</b></summary>

- **Unit:** Tek bir fonksiyon/reducer/Hook; hızlı.
- **Integration:** Bileşeni alt bileşenleri ve state'iyle birlikte render edip kullanıcı gibi etkileşmek.
- **E2E:** Gerçek tarayıcıda tam kullanıcı akışı (Playwright, Cypress); yavaş ama gerçekçi.

Yaygın görüş ("test kupası"): En çok **entegrasyon** testi yazılır; statik analiz (TypeScript, ESLint) temeli oluşturur. → [Konu 21](./21-test.md)
</details>

<details>
<summary>🟡 <b>15.2 React Testing Library'nin felsefesi nedir?</b></summary>

"Testler yazılımın kullanılma şekline ne kadar benzerse, o kadar güven verir." Bileşenin iç yapısı (state adları, Hook'lar) değil, kullanıcının gördüğü ve yaptığı test edilir. Böylece uygulama yeniden yazılsa bile davranış aynıysa testler geçer.
</details>

<details>
<summary>🟡 <b>15.3 <code>getBy</code>, <code>queryBy</code> ve <code>findBy</code> arasındaki fark nedir? Hangi sorgu önceliklidir?</b></summary>

- `getBy`: Bulamazsa hata fırlatır (eleman olmalı).
- `queryBy`: Bulamazsa `null` döner (eleman olmamalı kontrolü).
- `findBy`: Asenkron bekler (eleman gelecek).

Öncelik: `getByRole` > `getByLabelText` > `getByPlaceholderText` > `getByText` > ... > `getByTestId` (son çare). `getByRole` erişilebilirliği de dolaylı olarak test eder.
</details>

<details>
<summary>🟡 <b>15.4 API çağrısı yapan bir bileşeni nasıl test edersin?</b></summary>

Ağ isteklerini **MSW (Mock Service Worker)** ile ağ düzeyinde taklit ederim: kod gerçek `fetch` yapar, MSW sahte cevap döner. Başarılı, hatalı ve gecikmeli senaryolar test başına `server.use(...)` ile ayarlanır. Asenkron sonuçlar `findBy`/`waitFor` ile beklenir. TanStack Query kullanılıyorsa her test için `retry: false` ayarlı yeni bir `QueryClient` oluşturulur.
</details>

<details>
<summary>🟡 <b>15.5 Bir custom Hook'u nasıl test edersin?</b></summary>

`renderHook(() => useHookum())` ile; sonuç `result.current`'tan okunur, state güncelleyen çağrılar `act` ile sarılır, props değişimi `rerender` ile, zaman `vi.useFakeTimers` ile kontrol edilir. Provider gerekiyorsa `wrapper` seçeneği kullanılır. Çoğu zaman Hook'u kullanan bileşen üzerinden test etmek daha anlamlıdır.
</details>

<details>
<summary>🟡 <b>15.6 Kırılgan (flaky) testlerin yaygın nedenleri nelerdir?</b></summary>

Beklenmeyen asenkron işlemler (`await` / `findBy` eksikliği), testler arası paylaşılan state (önbellek, mock'lar, MSW handler'ları sıfırlanmaması), gerçek zamanlayıcılar ve ağ, uygulama detaylarına bağlı sorgular.
</details>

---

## 16. Stil, Erişilebilirlik ve Güvenlik

<details>
<summary>🟢 <b>16.1 React'te stil verme yöntemleri nelerdir? Tercihin nedir?</b></summary>

Global CSS, inline stil, CSS Modules, Sass, utility-first CSS (**Tailwind**), CSS-in-JS (styled-components, Emotion; çalışma zamanı maliyeti nedeniyle zero-runtime çözümlere kayış var) ve bileşen kütüphaneleri (MUI, shadcn/ui, Radix). Bugün yeni projelerde Tailwind + headless bileşenler (shadcn/ui) çok yaygındır. Cevapta takımın ihtiyacına göre gerekçelendirmek önemlidir. → [Konu 16](./16-stil-yontemleri.md)
</details>

<details>
<summary>🟡 <b>16.2 Tailwind'de sınıf adlarını dinamik olarak (<code>bg-${renk}-500</code>) oluşturmak neden çalışmaz?</b></summary>

Tailwind kaynak kodu **metin olarak tarar** ve sadece kodda tam haliyle geçen sınıfları üretir. Dinamik birleştirilen sınıflar bulunamaz. Çözüm: tam sınıf adlarını içeren bir eşleme nesnesi (`{ mavi: "bg-blue-500" }`), varyant sistemleri (`cva`) veya CSS değişkenleri.
</details>

<details>
<summary>🟢 <b>16.3 React uygulamalarında erişilebilirlik için nelere dikkat edersin?</b></summary>

Anlamsal HTML (`button`, `nav`, `label`), tıklanabilir `div` yerine `button`, input'ları `label` ile bağlamak, ikon butonlarına `aria-label`, klavye ile kullanılabilirlik ve görünür odak (`focus-visible`), modallarda odak yönetimi, yeterli renk kontrastı, durum bildirimleri için `role="status"`/`aria-live`, hata mesajları için `aria-invalid`/`aria-describedby`, `prefers-reduced-motion`.
</details>

<details>
<summary>🟡 <b>16.4 React XSS'e karşı nasıl korur? Hangi durumlarda risk vardır?</b></summary>

React, JSX'teki `{}` içine yazılan değerleri otomatik olarak **kaçışlar (escape)**; `<script>` metni HTML olarak değil düz metin olarak basılır. Riskler: `dangerouslySetInnerHTML` (kullanılacaksa DOMPurify gibi bir kütüphaneyle temizlenmeli), kullanıcı girdisini `href`'e koymak (`javascript:` URL'leri), `eval`. React 19.3 ayrıca tarayıcının Trusted Types API'siyle uyumlu çalışır.
</details>

<details>
<summary>🟡 <b>16.5 Token'ları (JWT) tarayıcıda nerede saklamalı?</b></summary>

`localStorage` JavaScript ile okunabildiği için XSS saldırısında çalınabilir. Daha güvenli yaklaşım: kimlik bilgisini **`HttpOnly`, `Secure`, `SameSite`** bayraklı bir çerezde tutmak (JS erişemez); CSRF'e karşı `SameSite` ve gerekirse CSRF token'ı kullanmak. Kısa ömürlü access token bellekte, refresh token `HttpOnly` çerezde tutmak da yaygındır.
</details>

<details>
<summary>🟢 <b>16.6 <code>NEXT_PUBLIC_</code> / <code>VITE_</code> önekli ortam değişkenlerine API anahtarı koymak neden tehlikelidir?</b></summary>

Bu önekli değişkenler build sırasında JavaScript bundle'ına gömülür; tarayıcıda herkes görebilir. Gizli anahtarlar sadece sunucuda (önek olmadan) tutulmalı, istemci gizli anahtar gerektiren servislere kendi sunucun üzerinden erişmelidir.
</details>

---

## 17. Kod Okuma ve "Çıktı Ne Olur?" Soruları

<details>
<summary>🟡 <b>17.1 Butona bir kez tıklanınca ekranda ne yazar, konsolda ne görünür?</b></summary>

```jsx
function Sayac() {
  const [sayi, setSayi] = useState(0);
  function handleClick() {
    setSayi(sayi + 1);
    setSayi(sayi + 1);
    setSayi(sayi + 1);
    console.log(sayi);
  }
  return <button onClick={handleClick}>{sayi}</button>;
}
```

**Cevap:** Ekranda **1**, konsolda **0**. Üç çağrı da aynı snapshot'taki `sayi = 0`'ı görür ve "1 yap" ister; `console.log` mevcut render'ın değerini yazar.
</details>

<details>
<summary>🟡 <b>17.2 Aynı örnekte <code>setSayi(s =&gt; s + 1)</code> üç kez çağrılırsa?</b></summary>

Ekranda **3**, konsolda yine **0**. Updater fonksiyonlar kuyrukta sırayla bir önceki sonuca uygulanır (0 → 1 → 2 → 3), ama mevcut render'daki `sayi` değişkeni değişmez.
</details>

<details>
<summary>🟡 <b>17.3 Sonuç kaç olur?</b></summary>

```jsx
setSayi(sayi + 5);
setSayi((s) => s + 1);
// sayi başlangıçta 0
```

**Cevap:** **6.** İlki kuyruğa "5 yap" ekler, ikincisi o değere 1 ekler.
</details>

<details>
<summary>🟡 <b>17.4 Geliştirme ortamında (StrictMode açık) konsolda hangi sırayla ne görünür? Production'da?</b></summary>

```jsx
function Bilesen() {
  console.log("render");
  useEffect(() => {
    console.log("effect");
    return () => console.log("cleanup");
  }, []);
  return null;
}
```

**Cevap:** - **Geliştirme:** `render`, `render` (ikincisi çift render), `effect`, `cleanup`, `effect`.
- **Production:** `render`, `effect`.
</details>

<details>
<summary>🔴 <b>17.5 Ebeveyn ve çocuk bileşende log'ların sırası nedir?</b></summary>

```jsx
function Cocuk() {
  console.log("Çocuk render");
  useEffect(() => console.log("Çocuk effect"));
  return null;
}
function Ebeveyn() {
  console.log("Ebeveyn render");
  useEffect(() => console.log("Ebeveyn effect"));
  return <Cocuk />;
}
```

**Cevap:** `Ebeveyn render` → `Çocuk render` → `Çocuk effect` → `Ebeveyn effect`. Render yukarıdan aşağı, effect'ler ise **aşağıdan yukarı** (önce çocuklar) çalışır; ebeveynin effect'i çalıştığında çocukların DOM'u hazırdır.
</details>

<details>
<summary>🟢 <b>17.6 Bu kodda neden ekran güncellenmiyor?</b></summary>

```jsx
function handleEkle() {
  liste.push(yeniOge);
  setListe(liste);
}
```

**Cevap:** Dizi mutasyona uğratıldı ve **aynı referans** verildi; React `Object.is` ile değişiklik görmez ve render'ı atlar. Doğrusu: `setListe([...liste, yeniOge])`.
</details>

<details>
<summary>🟡 <b>17.7 Kullanıcı "Göster"e tıklayıp 3 saniye içinde sayacı iki kez artırırsa uyarıda hangi değer görünür?</b></summary>

```jsx
function Bilesen() {
  const [sayi, setSayi] = useState(0);
  function goster() {
    setTimeout(() => alert(sayi), 3000);
  }
  return (
    <>
      <button onClick={() => setSayi(sayi + 1)}>Artır ({sayi})</button>
      <button onClick={goster}>Göster</button>
    </>
  );
}
```

**Cevap:** **Tıklama anındaki** değer (ör. 0). Zamanlayıcı, kurulduğu render'ın closure'ındaki `sayi`'yı görür (snapshot). En güncel değer isteniyorsa değer bir ref'te tutulur.
</details>

<details>
<summary>🟡 <b>17.8 Bu effect neden sonsuz döngüye giriyor?</b></summary>

```jsx
function Liste({ kategori }) {
  const [urunler, setUrunler] = useState([]);
  const filtre = { kategori, limit: 20 };

  useEffect(() => {
    urunleriGetir(filtre).then(setUrunler);
  }, [filtre]);
}
```

**Cevap:** `filtre` her render'da **yeni bir nesne**; bağımlılık her render'da "değişmiş" sayılır. Effect çalışır → `setUrunler` yeni dizi verir → render → yeni `filtre` → effect tekrar... Çözüm: nesneyi effect içinde oluşturmak ve primitive'lere bağımlı olmak (`[kategori]`), veya veriyi TanStack Query ile çekmek.
</details>

<details>
<summary>🟡 <b>17.9 Listenin başına yeni eleman eklenince, kullanıcının input'lara yazdıkları neden yanlış satırlarda görünüyor?</b></summary>

```jsx
{gorevler.map((g, i) => (
  <li key={i}>
    {g.ad} <input placeholder="Not ekle" />
  </li>
))}
```

**Cevap:** `key` olarak index kullanılmış. Başa ekleme yapılınca tüm index'ler kayar; React elemanları index'e göre eşleştirdiği için input'ların DOM'u ve değerleri yanlış görevlere kalır. Çözüm: kararlı kimlik `key={g.id}`.
</details>

<details>
<summary>🟢 <b>17.10 <code>sepet.length</code> 0 iken ekranda ne görünür?</b></summary>

```jsx
<div>{sepet.length && <SepetOzeti />}</div>
```

**Cevap:** Ekranda **0** görünür. Düzeltme: `{sepet.length > 0 && <SepetOzeti />}`.
</details>

<details>
<summary>🟡 <b>17.11 Bu bileşendeki hata nedir?</b></summary>

```jsx
function Profil({ kullanici }) {
  if (!kullanici) return <p>Yükleniyor...</p>;
  const [duzenle, setDuzenle] = useState(false);
  // ...
}
```

**Cevap:** Hook erken `return`'den **sonra** çağrılıyor; `kullanici` ilk render'da `null`, sonra dolu olursa Hook sayısı değişir ve React hata verir ("Rendered more hooks than during the previous render"). Tüm Hook'lar koşullu dönüşlerden önce çağrılmalıdır.
</details>

<details>
<summary>🟡 <b>17.12 Bu kodun sorunu nedir?</b></summary>

```jsx
function Arama({ sorgu }) {
  const [sonuclar, setSonuclar] = useState([]);
  const [filtrelenmis, setFiltrelenmis] = useState([]);

  useEffect(() => {
    setFiltrelenmis(sonuclar.filter((s) => s.ad.includes(sorgu)));
  }, [sonuclar, sorgu]);
}
```

**Cevap:** `filtrelenmis` türetilebilen bir değerdir; ayrı state + effect gereksiz bir render ve senkronizasyon hatası riski yaratır. Doğrusu render sırasında hesaplamak: `const filtrelenmis = sonuclar.filter((s) => s.ad.includes(sorgu));` (gerçekten pahalıysa `useMemo`).
</details>

---

## 18. Kodlama Soruları (Canlı Kodlama)

<details>
<summary>🟢 <b>18.1 Sınırları olan bir sayaç bileşeni yaz (min 0, max 10; sınırda butonlar devre dışı).</b></summary>

```jsx
function SinirliSayac({ min = 0, max = 10 }) {
  const [sayi, setSayi] = useState(min);
  return (
    <div>
      <button onClick={() => setSayi((s) => s - 1)} disabled={sayi <= min} aria-label="Azalt">−</button>
      <output aria-live="polite">{sayi}</output>
      <button onClick={() => setSayi((s) => s + 1)} disabled={sayi >= max} aria-label="Artır">+</button>
    </div>
  );
}
```

Dikkat çekilecek noktalar: updater fonksiyon, `disabled` ile sınır kontrolü, erişilebilir isimler.
</details>

<details>
<summary>🟡 <b>18.2 Saf JavaScript ile bir <code>debounce</code> fonksiyonu yaz.</b></summary>

```js
function debounce(fn, gecikme) {
  let zamanlayici;
  return function (...args) {
    clearTimeout(zamanlayici);
    zamanlayici = setTimeout(() => fn.apply(this, args), gecikme);
  };
}

const ara = debounce((q) => console.log("Arıyor:", q), 300);
```

Her çağrıda önceki zamanlayıcı iptal edilir; sadece son çağrıdan `gecikme` ms sonra çalışır. Closure ile `zamanlayici` korunur.
</details>

<details>
<summary>🟡 <b>18.3 <code>useDebounce</code> Hook'u yaz.</b></summary>

```jsx
function useDebounce(deger, gecikme = 400) {
  const [gecikmeli, setGecikmeli] = useState(deger);

  useEffect(() => {
    const id = setTimeout(() => setGecikmeli(deger), gecikme);
    return () => clearTimeout(id);   // değer değişirse önceki zamanlayıcı iptal
  }, [deger, gecikme]);

  return gecikmeli;
}

// Kullanım
const [arama, setArama] = useState("");
const q = useDebounce(arama, 400);
```
</details>

<details>
<summary>🟡 <b>18.4 Yükleniyor/hata durumlarını yöneten ve yarış durumunu önleyen bir <code>useFetch</code> Hook'u yaz.</b></summary>

```jsx
function useFetch(url) {
  const [durum, setDurum] = useState({ veri: null, hata: null, yukleniyor: true });

  useEffect(() => {
    if (!url) return;
    const ctrl = new AbortController();
    setDurum((d) => ({ ...d, yukleniyor: true, hata: null }));

    fetch(url, { signal: ctrl.signal })
      .then((res) => {
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        return res.json();
      })
      .then((veri) => setDurum({ veri, hata: null, yukleniyor: false }))
      .catch((hata) => {
        if (hata.name !== "AbortError") setDurum({ veri: null, hata, yukleniyor: false });
      });

    return () => ctrl.abort();   // url değişirse veya bileşen kalkarsa eski istek iptal
  }, [url]);

  return durum;
}
```

Bahsedilecek noktalar: `res.ok` kontrolü, `AbortController` ile yarış durumu ve iptal, `AbortError`'ın yok sayılması; gerçek projede TanStack Query tercih edileceği.
</details>

<details>
<summary>🟡 <b>18.5 <code>useLocalStorage</code> Hook'u yaz.</b></summary>

```jsx
function useLocalStorage(anahtar, baslangic) {
  const [deger, setDeger] = useState(() => {
    try {
      const kayit = localStorage.getItem(anahtar);
      return kayit !== null ? JSON.parse(kayit) : baslangic;
    } catch {
      return baslangic;
    }
  });

  useEffect(() => {
    try {
      localStorage.setItem(anahtar, JSON.stringify(deger));
    } catch {
      /* kota dolu veya erişim yok */
    }
  }, [anahtar, deger]);

  return [deger, setDeger];
}
```

Bahsedilecek noktalar: lazy initial state (her render'da `localStorage` okunmaz), `try/catch` (gizli mod, bozuk JSON), SSR'de `localStorage` olmadığı.
</details>

<details>
<summary>🟡 <b>18.6 Ekleme, tamamlama, silme ve filtreleme yapılabilen bir yapılacaklar listesi yaz.</b></summary>

```jsx
function Yapilacaklar() {
  const [gorevler, setGorevler] = useState([]);
  const [metin, setMetin] = useState("");
  const [filtre, setFiltre] = useState("hepsi");   // "hepsi" | "aktif" | "tamam"

  function ekle(e) {
    e.preventDefault();
    const temiz = metin.trim();
    if (!temiz) return;
    setGorevler((g) => [...g, { id: crypto.randomUUID(), metin: temiz, tamam: false }]);
    setMetin("");
  }

  const toggle = (id) =>
    setGorevler((g) => g.map((x) => (x.id === id ? { ...x, tamam: !x.tamam } : x)));
  const sil = (id) => setGorevler((g) => g.filter((x) => x.id !== id));

  const gorunenler = gorevler.filter((g) =>
    filtre === "aktif" ? !g.tamam : filtre === "tamam" ? g.tamam : true
  );   // türetilmiş

  return (
    <div>
      <form onSubmit={ekle}>
        <label>
          Yeni görev <input value={metin} onChange={(e) => setMetin(e.target.value)} />
        </label>
        <button>Ekle</button>
      </form>

      <div role="group" aria-label="Filtre">
        {["hepsi", "aktif", "tamam"].map((f) => (
          <button key={f} onClick={() => setFiltre(f)} aria-pressed={filtre === f}>{f}</button>
        ))}
      </div>

      <ul>
        {gorunenler.map((g) => (
          <li key={g.id}>
            <label>
              <input type="checkbox" checked={g.tamam} onChange={() => toggle(g.id)} />
              {g.metin}
            </label>
            <button onClick={() => sil(g.id)} aria-label={`${g.metin} görevini sil`}>Sil</button>
          </li>
        ))}
      </ul>
      <p>{gorevler.filter((g) => !g.tamam).length} görev kaldı</p>
    </div>
  );
}
```

Bahsedilecek noktalar: immutability, kararlı `key`, filtrelenmiş listenin state değil türetilmiş olması, erişilebilir form, büyürse `useReducer`'a geçilebileceği.
</details>

<details>
<summary>🟡 <b>18.7 Önceki değeri döndüren bir <code>usePrevious</code> Hook'u yaz.</b></summary>

```jsx
function usePrevious(deger) {
  const ref = useRef();
  useEffect(() => {
    ref.current = deger;   // render'dan SONRA güncellenir
  }, [deger]);
  return ref.current;      // bu render sırasında hâlâ önceki değer
}
```

Not: Render sırasında `ref.current` okumak React kurallarına göre önerilmez; React dokümantasyonu bu ihtiyaç için önceki değeri **state'te** tutup render sırasında karşılaştırmayı önerir:

```jsx
const [onceki, setOnceki] = useState(deger);
if (onceki !== deger) {
  setOnceki(deger);
  // değişime tepki ver
}
```
</details>

<details>
<summary>🔴 <b>18.8 Escape tuşuyla ve arka plana tıklayınca kapanan, odağı yöneten bir Modal bileşeni yaz.</b></summary>

```jsx
import { useEffect, useRef } from "react";
import { createPortal } from "react-dom";

function Modal({ acik, onKapat, baslik, children }) {
  const panelRef = useRef(null);

  useEffect(() => {
    if (!acik) return;
    const oncekiOdak = document.activeElement;
    panelRef.current?.focus();

    function tus(e) {
      if (e.key === "Escape") onKapat();
    }
    document.addEventListener("keydown", tus);
    document.body.style.overflow = "hidden";   // arka plan kaymasın

    return () => {
      document.removeEventListener("keydown", tus);
      document.body.style.overflow = "";
      oncekiOdak?.focus();                    // odağı geri ver
    };
  }, [acik, onKapat]);

  if (!acik) return null;

  return createPortal(
    <div className="arka-plan" onClick={onKapat}>
      <div
        ref={panelRef}
        role="dialog"
        aria-modal="true"
        aria-labelledby="modal-baslik"
        tabIndex={-1}
        onClick={(e) => e.stopPropagation()}   // içeri tıklama kapatmasın
      >
        <h2 id="modal-baslik">{baslik}</h2>
        {children}
        <button onClick={onKapat}>Kapat</button>
      </div>
    </div>,
    document.body
  );
}
```

Bahsedilecek noktalar: portal, effect cleanup, `stopPropagation`, odak yönetimi ve geri verme, `role="dialog"` + `aria-modal`. Gerçek projede odak tuzağı (focus trap) eklenir veya yerel `<dialog>` elementi / Radix gibi bir kütüphane kullanılır.
</details>

<details>
<summary>🟡 <b>18.9 Bir API'den kullanıcıları çeken, arama kutusuyla filtreleyen bir bileşen yaz.</b></summary>

```jsx
function KullaniciArama() {
  const [kullanicilar, setKullanicilar] = useState([]);
  const [durum, setDurum] = useState("yukleniyor");   // "yukleniyor" | "hata" | "hazir"
  const [arama, setArama] = useState("");

  useEffect(() => {
    const ctrl = new AbortController();
    fetch("https://jsonplaceholder.typicode.com/users", { signal: ctrl.signal })
      .then((r) => {
        if (!r.ok) throw new Error();
        return r.json();
      })
      .then((veri) => {
        setKullanicilar(veri);
        setDurum("hazir");
      })
      .catch((e) => e.name !== "AbortError" && setDurum("hata"));
    return () => ctrl.abort();
  }, []);

  const q = arama.trim().toLocaleLowerCase("tr");
  const sonuclar = kullanicilar.filter((k) => k.name.toLocaleLowerCase("tr").includes(q));

  if (durum === "yukleniyor") return <p>Yükleniyor...</p>;
  if (durum === "hata") return <p role="alert">Kullanıcılar yüklenemedi.</p>;

  return (
    <div>
      <label>
        Ara <input type="search" value={arama} onChange={(e) => setArama(e.target.value)} />
      </label>
      {sonuclar.length === 0 ? (
        <p>Sonuç yok.</p>
      ) : (
        <ul>
          {sonuclar.map((k) => <li key={k.id}>{k.name} — {k.email}</li>)}
        </ul>
      )}
    </div>
  );
}
```

Bahsedilecek noktalar: tek bir `durum` değişkeni, iptal, filtrenin türetilmesi, boş durum; liste büyükse `useDeferredValue`, sunucu tarafı arama gerekiyorsa debounce ve TanStack Query.
</details>

---

## 19. Davranışsal ve Deneyim Soruları

Teknik mülakatların çoğunda deneyime dair sorular da gelir. Cevaplarda **STAR** yöntemi işe yarar: **S**ituation (durum) → **T**ask (görev) → **A**ction (ne yaptın) → **R**esult (sonuç, mümkünse ölçülebilir).

<details>
<summary>🟢 <b>19.1 Üzerinde çalıştığın ve gurur duyduğun bir React projesini anlat.</b></summary>

Hazırlanacaklar: Projenin amacı, kullandığın teknolojiler ve **neden** seçtiğin, senin sorumluluğun, karşılaştığın en zor teknik problem ve nasıl çözdüğün, öğrendiklerin. Bu notlardaki örnek uygulamalardan birini kendi projen haline getirip GitHub'da canlı bir demo ile sunmak çok etkilidir.
</details>

<details>
<summary>🟡 <b>19.2 Çözmekte zorlandığın bir hatayı anlat. Nasıl ayıkladın?</b></summary>

İyi bir cevap sistematik bir yaklaşım gösterir: hatayı tekrar üretmek, kapsamı daraltmak (DevTools, Profiler, Network, `console`/breakpoint), hipotez kurup test etmek, kök nedeni bulmak, düzeltip **tekrar etmemesi için test eklemek**. Örnek konular: stale closure, yarış durumu, hydration uyuşmazlığı, gereksiz render'lar.
</details>

<details>
<summary>🟡 <b>19.3 Bir kod incelemesinde (code review) nelere dikkat edersin?</b></summary>

Doğruluk ve kenar durumlar; okunabilirlik ve isimlendirme; bileşen sorumluluklarının ayrılığı; state'in doğru yerde ve minimal olması; gereksiz effect'ler; erişilebilirlik; testlerin varlığı; güvenlik (XSS, gizli bilgiler); performans (ama erken optimizasyon olmadan). Geri bildirimi yapıcı ve gerekçeli vermek.
</details>

<details>
<summary>🟡 <b>19.4 Teknik bir konuda takım arkadaşınla anlaşamadığında ne yaparsın?</b></summary>

Kişisel değil teknik argümanlara odaklanmak, iki tarafın gerekçelerini ve ödünleşimleri (trade-off) açıkça yazmak, mümkünse küçük bir prototip veya ölçümle karar vermek, gerekirse ekibin teknik lideriyle birlikte karar almak ve karar alındıktan sonra uygulamaya destek vermek.
</details>

<details>
<summary>🟢 <b>19.5 Yeni teknolojileri nasıl takip ediyorsun?</b></summary>

Resmi kaynaklar (react.dev blog, framework sürüm notları), haftalık bültenler (This Week in React gibi), açık kaynak projeleri incelemek ve küçük deneme projeleri yapmak. Yeni bir aracı benimsemeden önce olgunluğunu ve takıma faydasını değerlendirmek.
</details>

---

## 20. Mülakat İpuçları

**Teknik sorularda:**
- Cevaba **kısa bir tanımla** başla, sonra "neden önemli"yi ve küçük bir örneği ekle.
- Ödünleşimleri (trade-off) belirt: "X'i Y durumunda kullanırım çünkü...; ama Z durumunda şunu tercih ederim."
- Bilmediğin bir şey sorulursa dürüstçe söyle ve nasıl öğreneceğini/araştıracağını anlat.

**Canlı kodlamada:**
- Kodlamaya başlamadan önce **gereksinimleri netleştir** (kenar durumlar, erişilebilirlik beklentisi, hangi API'ler kullanılabilir).
- **Sesli düşün**: Ne yaptığını ve neden yaptığını anlat.
- Önce çalışan basit bir sürüm yap, sonra iyileştir (doğrulama, hata durumları, erişilebilirlik, performans).
- Bitirince kenar durumları kendin test et: boş girdi, hızlı tıklama, ağ hatası, boş liste.

**Sıkça gözden kaçan ama puan kazandıran detaylar:**
- Liste key'leri, immutability, updater fonksiyonlar.
- Effect cleanup ve `AbortController`.
- Türetilebilen değerleri state'te tutmamak.
- `label`, `button`, `aria-*` ile erişilebilirlik.
- `res.ok` kontrolü ve hata durumları.
- "Gerçek projede bunu TanStack Query / React Hook Form / Zod ile yapardım" gibi ekosistem farkındalığı.

**Hazırlık:**
- Bu notlardaki "Her Şey Bir Arada" örneklerini **sıfırdan, bakmadan** yazmayı dene.
- GitHub'da düzenli bir portföy: README'li, canlı demolu, testli 2–3 proje.
- Başvurduğun şirketin kullandığı teknolojilere (Next.js mi, SPA mı, hangi state kütüphanesi) göre ilgili bölümleri tekrar et.

---

⬅️ Önceki: [23 · Next.js ve Server Components](./23-nextjs.md)
