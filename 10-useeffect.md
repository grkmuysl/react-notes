# 10 · `useEffect` ve Yan Etkiler

> **Konu özeti:** Effect'in ne olduğu ve render/olay yöneticisinden farkı, `useEffect` kullanımı, bağımlılık dizisi, cleanup ve StrictMode davranışı, sık kullanılan effect'ler, senkronizasyon bakış açısı, bağımlılıkları doğru belirleme, async kullanım, effect ile veri çekme ve race condition, effect'e ihtiyaç olmayan durumlar, `useEffectEvent` ve `useLayoutEffect`.

---

## İçindekiler

1. [Effect Nedir? Üç Tür Kod](#1-effect-nedir-üç-tür-kod)
2. [Temel Kullanım](#2-temel-kullanım)
3. [Bağımlılık Dizisi](#3-bağımlılık-dizisi-)
4. [Temizleme (Cleanup) Fonksiyonu](#4-temizleme-cleanup-fonksiyonu-)
5. [Sık Kullanılan Effect Örnekleri](#5-sık-kullanılan-effect-örnekleri)
6. [Yaşam Döngüsü Değil, Senkronizasyon](#6-yaşam-döngüsü-değil-senkronizasyon)
7. [Bağımlılıkları Doğru Belirlemek](#7-bağımlılıkları-doğru-belirlemek-)
8. [Async Fonksiyonlar ve Effect](#8-async-fonksiyonlar-ve-effect)
9. [Effect ile Veri Çekmek](#9-effect-ile-veri-çekmek-)
10. [Effect'e İhtiyacın Olmayabilir](#10-effecte-ihtiyacın-olmayabilir-)
11. [`useEffectEvent`: Reaktif Olmayan Mantık](#11-useeffectevent-reaktif-olmayan-mantık)
12. [`useLayoutEffect`](#12-uselayouteffect)
13. [Her Şey Bir Arada: Pomodoro Sayacı](#13-her-şey-bir-arada-pomodoro-sayacı)
14. [Özet](#14-özet)

---

## 1. Effect Nedir? Üç Tür Kod

Bazı yan etkiler ne render sırasında yapılabilir (saflığı bozar) ne de belirli bir olaya bağlanabilir:

- Sohbet ekranı **görünür olduğu sürece** sunucuya bağlı kalmalı.
- Sayfa başlığı her zaman okunmamış mesaj sayısını **yansıtmalı.**
- Bileşen ekrandayken pencere boyutu **dinlenmeli.**

Bunlar bir tıklamadan değil, **bileşenin ekranda olmasından** kaynaklanır → **effect.**

| Kod türü | Ne zaman çalışır | Yan etki? | Örnek |
|---|---|---|---|
| Render | Her render'da | ❌ Saf olmalı | `const tamAd = ad + " " + soyad` |
| Olay yöneticisi | Kullanıcı bir şey yapınca | ✅ | Sepete ekle, formu gönder |
| **Effect** | Render ekrana yansıdıktan **sonra** | ✅ | Sunucuya bağlan, başlığı güncelle, olay dinle |

> **Effect'lerin amacı: Bileşeni React dışındaki bir sistemle senkronize etmek.**

**Dış sistem örnekleri:** Tarayıcı API'leri (`document`, `window`, `localStorage`, zamanlayıcılar), ağ bağlantıları, üçüncü parti kütüphaneler (harita, grafik, video oynatıcı), React olmayan kod.

> Effect yazmadan önce sor: *"Senkronize ettiğim bir dış sistem var mı?"* Yoksa büyük ihtimalle effect'e ihtiyaç yoktur (bkz. [Bölüm 10](#10-effecte-ihtiyacın-olmayabilir-)).

---

## 2. Temel Kullanım

```jsx
import { useState, useEffect } from "react";

function Sayac() {
  const [sayi, setSayi] = useState(0);

  useEffect(() => {
    document.title = `${sayi} kez tıklandı`;
  });

  return <button onClick={() => setSayi(sayi + 1)}>Tıkla: {sayi}</button>;
}
```

### Çalışma sırası

```
State değişir
  → Render (bileşen fonksiyonu çalışır, JSX üretilir)   ← saf
  → Commit (DOM güncellenir)
  → Tarayıcı ekranı boyar
  → Effect çalışır                                         ← yan etki burada
```

- Effect, render tamamlanıp ekran güncellendikten **sonra** çalışır; kullanıcının ekranı görmesini geciktirmez.
- Render sırasında yasak olan `document.title = ...` gibi kodun **meşru yeri** effect'tir.

---

## 3. Bağımlılık Dizisi ⭐

```jsx
useEffect(() => { /* ... */ });          // 1. Dizi YOK → her render'dan sonra
useEffect(() => { /* ... */ }, []);      // 2. BOŞ dizi → sadece ilk render'dan sonra (mount)
useEffect(() => { /* ... */ }, [a, b]);  // 3. Dolu dizi → ilk render + a veya b değiştiğinde
```

| Bağımlılık | Ne zaman çalışır |
|---|---|
| Yok | Her render'dan sonra |
| `[]` | Sadece bileşen ekrana geldiğinde (bir kez) |
| `[a, b]` | İlk render + `a` veya `b` öncekinden farklıysa |

```jsx
function Profil({ kullaniciId }) {
  const [tema, setTema] = useState("acik");

  useEffect(() => {
    console.log("Kullanıcı değişti:", kullaniciId);
  }, [kullaniciId]);   // tema değişince çalışmaz
}
```

- React her değeri öncekiyle **`Object.is`** ile karşılaştırır.

> ⚠️ Bağımlılık dizisi **seçilmez**; effect içinde kullanılan değerlere göre **belirlenir** (bkz. [Bölüm 7](#7-bağımlılıkları-doğru-belirlemek-)).

---

## 4. Temizleme (Cleanup) Fonksiyonu ⭐

### Sorun: Sızıntı

```jsx
useEffect(() => {
  const baglanti = sohbetSunucusunaBaglan(odaId);
  baglanti.baglan();
}, [odaId]);
```

Oda her değiştiğinde yeni bağlantı açılır, eskisi açık kalır → **sızıntı (leak).**

### Çözüm: Cleanup döndürmek

```jsx
useEffect(() => {
  const baglanti = sohbetSunucusunaBaglan(odaId);
  baglanti.baglan();

  return () => {
    baglanti.kes();   // cleanup
  };
}, [odaId]);
```

### React cleanup'ı ne zaman çağırır?

1. Effect **tekrar çalışmadan önce** (bağımlılık değişince)
2. Bileşen **ekrandan kaldırılınca** (unmount)

```
Bileşen ekrana gelir (odaId="genel")
  → setup: "genel"e bağlan

odaId "genel" → "react" olur
  → cleanup: "genel"den çık       (önceki render'ın değerleriyle!)
  → setup: "react"e bağlan

Bileşen ekrandan kalkar
  → cleanup: "react"ten çık
```

> Cleanup, **kendi render'ının** değerlerini görür (snapshot).

### Setup / cleanup eşleşmeleri

| Setup | Cleanup |
|---|---|
| Bağlan | Bağlantıyı kes |
| `addEventListener` | `removeEventListener` |
| `setInterval` / `setTimeout` | `clearInterval` / `clearTimeout` |
| Abone ol (subscribe) | Abonelikten çık |
| Animasyon başlat | Durdur / başa sar |
| Veri isteği gönder | Yanıtı yok say / isteği iptal et |

### StrictMode: Effect neden iki kez çalışıyor?

```jsx
useEffect(() => {
  console.log("✅ Bağlanıldı");
  return () => console.log("❌ Bağlantı kesildi");
}, []);
```

Geliştirme ortamında çıktı:

```
✅ Bağlanıldı
❌ Bağlantı kesildi
✅ Bağlanıldı
```

- StrictMode, bileşeni ekrana getirdikten hemen sonra **bir kez kaldırıp tekrar ekler** (setup → cleanup → setup).
- Amacı: **Cleanup'ı unutulmuş effect'leri yakalamak.**
- Cleanup doğruysa kullanıcı açısından fark yoktur (bağlan-kes-bağlan = bağlı).
- Production'da bu çift çalıştırma olmaz.

> ❌ Çözüm StrictMode'u kapatmak veya `useRef` ile "zaten çalıştı mı?" kontrolü eklemek **değildir.** Çözüm doğru cleanup yazmaktır.

---

## 5. Sık Kullanılan Effect Örnekleri

### 1. Tarayıcı olaylarını dinlemek

```jsx
function PencereBoyutu() {
  const [genislik, setGenislik] = useState(window.innerWidth);

  useEffect(() => {
    function handleResize() {
      setGenislik(window.innerWidth);
    }
    window.addEventListener("resize", handleResize);
    return () => window.removeEventListener("resize", handleResize);
  }, []);

  return <p>Pencere genişliği: {genislik}px</p>;
}
```

> `removeEventListener`'a **aynı fonksiyon referansı** verilmelidir.

### 2. Global klavye kısayolu

```jsx
function KomutPaleti() {
  const [acik, setAcik] = useState(false);

  useEffect(() => {
    function handleKeyDown(e) {
      if ((e.ctrlKey || e.metaKey) && e.key === "k") {
        e.preventDefault();
        setAcik((a) => !a);
      }
      if (e.key === "Escape") setAcik(false);
    }
    window.addEventListener("keydown", handleKeyDown);
    return () => window.removeEventListener("keydown", handleKeyDown);
  }, []);

  return acik ? <div className="palet">🔍 Komut ara...</div> : null;
}
```

### 3. Zamanlayıcı

```jsx
function Kronometre() {
  const [saniye, setSaniye] = useState(0);
  const [calisiyor, setCalisiyor] = useState(false);

  useEffect(() => {
    if (!calisiyor) return;              // çalışmıyorsa hiçbir şey kurma

    const id = setInterval(() => {
      setSaniye((s) => s + 1);           // updater!
    }, 1000);

    return () => clearInterval(id);
  }, [calisiyor]);

  return (
    <div>
      <p>{saniye} sn</p>
      <button onClick={() => setCalisiyor(!calisiyor)}>
        {calisiyor ? "Durdur" : "Başlat"}
      </button>
      <button onClick={() => setSaniye(0)}>Sıfırla</button>
    </div>
  );
}
```

> **Neden updater?** Interval fonksiyonu kurulduğu render'ın `saniye`'sini görür; `saniye + 1` hep `0 + 1` olur ve sayaç 1'de takılır. Updater güncel değeri alır ve `saniye`'yi bağımlılık yapma zorunluluğunu kaldırır.

### 4. `localStorage` senkronizasyonu

```jsx
function NotDefteri() {
  const [not, setNot] = useState(() => localStorage.getItem("not") ?? "");  // lazy init

  useEffect(() => {
    localStorage.setItem("not", not);
  }, [not]);

  return <textarea value={not} onChange={(e) => setNot(e.target.value)} />;
}
```

- **Okuma:** İlk render'da lazy initializer ile.
- **Yazma:** Her değişiklikte effect ile.

### 5. React dışı kütüphane ile senkronizasyon

```jsx
function Harita({ merkez, yakinlik }) {
  const kapRef = useRef(null);       // DOM elementine erişim (Konu 11)
  const haritaRef = useRef(null);

  // Haritayı bir kez oluştur
  useEffect(() => {
    haritaRef.current = new HaritaKutuphanesi(kapRef.current);
    return () => haritaRef.current.destroy();
  }, []);

  // Props değişince haritayı güncelle
  useEffect(() => {
    haritaRef.current.setView(merkez, yakinlik);
  }, [merkez, yakinlik]);

  return <div ref={kapRef} style={{ height: 400 }} />;
}
```

> Effect'in en "ders kitabı" kullanımı: React dışı bir kütüphaneyi React state'iyle senkron tutmak.

---

## 6. Yaşam Döngüsü Değil, Senkronizasyon

- 📜 Class bileşenlerde "mount / update / unmount" (`componentDidMount`, `componentDidUpdate`, `componentWillUnmount`) diye düşünülürdü.
- Effect'in bakış açısında **yalnızca iki şey** vardır: senkronizasyonu **başlatmak** (setup) ve **durdurmak** (cleanup).

> *"`odaId` şu değerdeyken şu odaya bağlı olmalısın. `odaId` değişirse eskisini kapat, yenisini aç."*

Sorulacak tek soru: **"Bu effect neyi senkronize ediyor ve nasıl durdurulur?"**

### Her effect tek bir iş yapmalı

```jsx
// ❌ İki bağımsız iş tek effect'te
useEffect(() => {
  logSayfaGoruntulenme(sayfa);
  const baglanti = baglan(odaId);
  baglanti.baglan();
  return () => baglanti.kes();
}, [sayfa, odaId]);   // sayfa değişince bağlantı da gereksiz yere yeniden kurulur
```

```jsx
// ✅ Ayrı effect'ler
useEffect(() => {
  logSayfaGoruntulenme(sayfa);
}, [sayfa]);

useEffect(() => {
  const baglanti = baglan(odaId);
  baglanti.baglan();
  return () => baglanti.kes();
}, [odaId]);
```

---

## 7. Bağımlılıkları Doğru Belirlemek ⭐

> **Kural: Effect içinde kullanılan her reaktif değer bağımlılık dizisinde olmalı.**

**Reaktif değer:** Render sırasında değişebilen her şey — props, state ve bunlardan bileşen gövdesinde hesaplanan değişkenler ve fonksiyonlar.

```jsx
function SohbetOdasi({ odaId }) {
  const [sunucuUrl, setSunucuUrl] = useState("wss://sohbet.ornek.com");

  useEffect(() => {
    const baglanti = baglan(sunucuUrl, odaId);
    baglanti.baglan();
    return () => baglanti.kes();
  }, [sunucuUrl, odaId]);   // ✅ ikisi de kullanılıyor, ikisi de listede
}
```

- Biri eksik olsaydı, o değer değişince effect çalışmaz ve **bayat değerle** kalınır (stale closure).
- `eslint-plugin-react-hooks` → `react-hooks/exhaustive-deps` kuralı eksikleri yakalar.
- Uyarıyı `// eslint-disable-next-line` ile susturmak sorunu gizler, çözmez.

> Bağımlılıklar **seçilmez, kod belirler.** Bir bağımlılığı istemiyorsan diziden silme; **kodu değiştirerek** bağımlılık olmaktan çıkar.

### Bağımlılık olmaktan çıkarmanın yolları

#### 1. Reaktif olmayan değeri bileşenin dışına taşı

```jsx
const SUNUCU_URL = "wss://sohbet.ornek.com";   // bileşen dışında: hiç değişmez

function SohbetOdasi({ odaId }) {
  useEffect(() => {
    const baglanti = baglan(SUNUCU_URL, odaId);
    baglanti.baglan();
    return () => baglanti.kes();
  }, [odaId]);   // ✅
}
```

#### 2. State güncellemesinde updater kullan

```jsx
setMesajlar([...mesajlar, yeni]);   // mesajlar bağımlılık olur
setMesajlar((m) => [...m, yeni]);   // ✅ gerekmez
```

#### 3. Nesne ve fonksiyonları effect'in içine taşı

```jsx
// ❌ Her render'da YENİ nesne → her render'da yeniden bağlanır
function SohbetOdasi({ odaId }) {
  const secenekler = { sunucuUrl: SUNUCU_URL, odaId };

  useEffect(() => {
    const baglanti = baglan(secenekler);
    baglanti.baglan();
    return () => baglanti.kes();
  }, [secenekler]);
}
```

```jsx
// ✅ Nesne effect içinde oluşturulur
useEffect(() => {
  const secenekler = { sunucuUrl: SUNUCU_URL, odaId };
  const baglanti = baglan(secenekler);
  baglanti.baglan();
  return () => baglanti.kes();
}, [odaId]);   // sadece primitive bağımlılık
```

Nesne props olarak geliyorsa primitive değerler dışarıda çıkarılır:

```jsx
function SohbetOdasi({ secenekler }) {
  const { sunucuUrl, odaId } = secenekler;   // string'ler
  useEffect(() => {
    const baglanti = baglan({ sunucuUrl, odaId });
    baglanti.baglan();
    return () => baglanti.kes();
  }, [sunucuUrl, odaId]);   // ✅
}
```

#### 4. Okunacak ama tepki verilmeyecek mantık → `useEffectEvent`

Bkz. [Bölüm 11](#11-useeffectevent-reaktif-olmayan-mantık).

### ⚠️ Sonsuz döngü

```jsx
const [veri, setVeri] = useState([]);

useEffect(() => {
  setVeri([...veri, "yeni"]);
});   // dizi yok → her render'dan sonra çalışır
// Render → effect → setVeri → render → effect → ... ♾️
```

- Effect içinde, kendisinin (doğrudan veya dolaylı) bağlı olduğu state güncellenirse döngü oluşur.
- "Her render'da yeni nesne" bağımlılığı + `setState` de sonsuz döngüye yol açar.

> Effect içinde `setState` görüyorsan iki kez düşün; çoğu zaman [Bölüm 10](#10-effecte-ihtiyacın-olmayabilir-)'daki durumlardan biridir.

---

## 8. Async Fonksiyonlar ve Effect

```jsx
useEffect(async () => {     // ❌
  const veri = await getir();
}, []);
```

- `async` fonksiyon her zaman **Promise** döndürür; React effect'ten ya hiçbir şey ya da **cleanup fonksiyonu** bekler.

```jsx
// ✅ İçeride async fonksiyon tanımlayıp çağırmak
useEffect(() => {
  async function veriGetir() {
    const res = await fetch(url);
    const veri = await res.json();
    setVeri(veri);
  }
  veriGetir();
}, [url]);
```

---

## 9. Effect ile Veri Çekmek ⭐

### Naif hali ve gizli hatası

```jsx
function KullaniciProfili({ kullaniciId }) {
  const [kullanici, setKullanici] = useState(null);

  useEffect(() => {
    fetch(`https://jsonplaceholder.typicode.com/users/${kullaniciId}`)
      .then((res) => res.json())
      .then((veri) => setKullanici(veri));
  }, [kullaniciId]);
}
```

- Kullanıcı hızlıca 1 → 2 → 3'e geçerse üç istek aynı anda gider.
- Cevapların sırası belli değildir; 1'in cevabı en son gelirse ekranda **yanlış kullanıcı** kalır.
- Buna **yarış durumu (race condition)** denir.

### Çözüm 1: `ignore` bayrağı

```jsx
useEffect(() => {
  let ignore = false;

  async function getir() {
    const res = await fetch(`https://jsonplaceholder.typicode.com/users/${kullaniciId}`);
    const veri = await res.json();
    if (!ignore) setKullanici(veri);   // eski isteğin cevabını yok say
  }
  getir();

  return () => {
    ignore = true;
  };
}, [kullaniciId]);
```

- Her effect çalışması kendi `ignore` değişkenine sahiptir (closure).

### Çözüm 2: `AbortController`

İsteği gerçekten iptal eder, ağ trafiğini de keser:

```jsx
useEffect(() => {
  const controller = new AbortController();

  fetch(url, { signal: controller.signal })
    .then((res) => res.json())
    .then(setVeri)
    .catch((err) => {
      if (err.name !== "AbortError") setHata(err.message);   // iptal bir hata değildir
    });

  return () => controller.abort();
}, [url]);
```

### Yükleniyor ve hata durumlarıyla tam hali

```jsx
function KullaniciProfili({ kullaniciId }) {
  const [kullanici, setKullanici] = useState(null);
  const [durum, setDurum] = useState("yukleniyor");   // "yukleniyor" | "hata" | "basarili"
  const [hata, setHata] = useState(null);

  useEffect(() => {
    const controller = new AbortController();
    setDurum("yukleniyor");

    async function getir() {
      try {
        const res = await fetch(
          `https://jsonplaceholder.typicode.com/users/${kullaniciId}`,
          { signal: controller.signal }
        );
        if (!res.ok) throw new Error(`HTTP ${res.status}`);
        const veri = await res.json();
        setKullanici(veri);
        setDurum("basarili");
      } catch (err) {
        if (err.name === "AbortError") return;
        setHata(err.message);
        setDurum("hata");
      }
    }
    getir();

    return () => controller.abort();
  }, [kullaniciId]);

  if (durum === "yukleniyor") return <p>Yükleniyor...</p>;
  if (durum === "hata") return <p role="alert">Hata: {hata}</p>;
  return (
    <div>
      <h2>{kullanici.name}</h2>
      <p>{kullanici.email}</p>
    </div>
  );
}
```

### Effect ile veri çekmenin dezavantajları

- Her bileşende yükleniyor/hata/race condition kodu tekrar yazılır.
- **Önbellek (cache) yoktur**: sayfaya dönünce aynı veri yeniden çekilir.
- **Ağ şelaleleri (waterfall)**: ebeveyn veriyi çeker, render olur, sonra çocuk kendi verisini çeker; istekler paralel yerine sırayla gider.
- Sunucu tarafında render'da çalışmaz.

### Gerçek projelerde

- **TanStack Query** (React Query) veya **SWR**
- **Next.js** gibi framework'lerin veri çekme mekanizmaları
- React 19'un **`use`** API'si ve Suspense

> *(Konu 18 ve Konu 20.)* Effect ile veri çekmeyi bilmek, bu araçların neyi çözdüğünü anlamayı sağlar.

---

## 10. Effect'e İhtiyacın Olmayabilir ⭐⭐

Gereksiz effect'ler kodu **yavaşlatır** (ekstra render), **karmaşıklaştırır** ve **hataya açık** hale getirir.

> **Effect, dış bir sistemle senkronizasyon içindir.** Dış sistem yoksa effect'e de ihtiyaç yoktur.

### 1. Türetilebilen değerler → render sırasında hesapla

```jsx
// ❌ Ekstra render: önce eski tamAd ile, sonra effect, sonra yeni tamAd ile
const [tamAd, setTamAd] = useState("");
useEffect(() => {
  setTamAd(`${ad} ${soyad}`);
}, [ad, soyad]);

// ✅
const tamAd = `${ad} ${soyad}`;
```

### 2. Pahalı hesaplamalar → düz hesaplama veya `useMemo`

```jsx
// ❌
const [gorunenler, setGorunenler] = useState([]);
useEffect(() => {
  setGorunenler(gorevler.filter(/* ... */));
}, [gorevler, filtre]);

// ✅ (gerçekten pahalıysa useMemo — Konu 19)
const gorunenler = gorevler.filter(/* ... */);
```

### 3. Prop değişince state'i sıfırlamak → `key`

```jsx
// ❌
function ProfilSayfasi({ kullaniciId }) {
  const [yorum, setYorum] = useState("");
  useEffect(() => {
    setYorum("");
  }, [kullaniciId]);
}

// ✅
<ProfilSayfasi key={kullaniciId} kullaniciId={kullaniciId} />
```

### 4. Kullanıcı eylemine bağlı mantık → olay yöneticisi

```jsx
// ❌ Sayfa yenilenince veya sayfaya her girişte bildirim tekrar çıkar
useEffect(() => {
  if (urun.sepette) bildirimGoster(`${urun.ad} sepete eklendi`);
}, [urun]);

// ✅ Eylemin olduğu yerde
function handleSepeteEkle() {
  sepeteEkle(urun);
  bildirimGoster(`${urun.ad} sepete eklendi`);
}
```

> Sor: *"Bu kod, bileşen ekranda göründüğü için mi çalışmalı, yoksa kullanıcı bir şey yaptığı için mi?"* İkincisiyse → olay yöneticisi.

### 5. Form gönderimi (POST) → olay yöneticisi

```jsx
// ❌
const [gonderilecek, setGonderilecek] = useState(null);
useEffect(() => {
  if (gonderilecek) {
    fetch("/api/kayit", { method: "POST", body: JSON.stringify(gonderilecek) });
  }
}, [gonderilecek]);

// ✅
async function handleSubmit(e) {
  e.preventDefault();
  await fetch("/api/kayit", { method: "POST", body: JSON.stringify(form) });
}
```

> İstisna: "Sayfa görüntülendi" analitiği gibi, bileşen **ekranda göründüğü için** gönderilen istekler effect'e aittir.

### 6. Effect zincirleri → tek olay yöneticisinde hesapla

```jsx
// ❌ Her adım ekstra render, akışı takip etmek zor
useEffect(() => { if (kart?.altin) setAltinSayisi((s) => s + 1); }, [kart]);
useEffect(() => { if (altinSayisi > 3) { setTur((t) => t + 1); setAltinSayisi(0); } }, [altinSayisi]);
useEffect(() => { if (tur > 5) setOyunBitti(true); }, [tur]);
```

```jsx
// ✅ Sonraki durum olay anında hesaplanır
function handleKartOyna(kart) {
  setKart(kart);
  if (kart.altin) {
    if (altinSayisi < 3) {
      setAltinSayisi(altinSayisi + 1);
    } else {
      setAltinSayisi(0);
      setTur(tur + 1);
      if (tur === 5) alert("Oyun bitti!");
    }
  }
}

const oyunBitti = tur > 5;   // türetilmiş
```

### 7. Ebeveyni bilgilendirmek → aynı olay yöneticisinde

```jsx
// ❌
function Toggle({ onDegis }) {
  const [acik, setAcik] = useState(false);
  useEffect(() => { onDegis(acik); }, [acik, onDegis]);
  return <button onClick={() => setAcik(!acik)}>...</button>;
}

// ✅
function Toggle({ onDegis }) {
  const [acik, setAcik] = useState(false);
  function handleClick() {
    const yeni = !acik;
    setAcik(yeni);
    onDegis(yeni);
  }
  return <button onClick={handleClick}>...</button>;
}
```

> Çoğu zaman daha iyisi: state'i tamamen ebeveyne taşımak (controlled bileşen, Konu 12).

### 8. Uygulama başlangıcında bir kez → bileşen dışında

```jsx
// ❌ StrictMode'da iki kez çalışır, her App mount'unda tekrarlanır
useEffect(() => {
  analitikBaslat();
}, []);

// ✅ Modül seviyesinde, uygulama yüklenirken bir kez
if (typeof window !== "undefined") {
  analitikBaslat();
}
```

### Karar akışı

```
Bu kod neden çalışmalı?
├── Bir değer hesaplamak için          → Render sırasında hesapla (gerekirse useMemo)
├── Kullanıcı bir şey yaptığı için     → Olay yöneticisi
├── Prop değişince state sıfırlansın   → key
└── Bileşen ekranda olduğu için,
    dış bir sistemle senkron kalmak    → ✅ useEffect
```

---

## 11. `useEffectEvent`: Reaktif Olmayan Mantık

Effect içinde bir değeri **okumak** isteyip, o değer değişince effect'in **yeniden çalışmasını istemediğin** durumlar için.

### Sorun

```jsx
function SohbetOdasi({ odaId, tema }) {
  useEffect(() => {
    const baglanti = baglan(odaId);
    baglanti.on("baglandi", () => {
      bildirimGoster("Bağlanıldı!", tema);
    });
    baglanti.baglan();
    return () => baglanti.kes();
  }, [odaId, tema]);   // tema değişince bağlantı kopup yeniden kuruluyor!
}
```

- Tema bir görüntü tercihi; bağlantıyla ilgisi yok.
- `tema`'yı diziden silmek ise bayat değer hatası yaratır.

### Çözüm (React 19.2 ile kararlı)

```jsx
import { useEffect, useEffectEvent } from "react";

function SohbetOdasi({ odaId, tema }) {
  const onBaglandi = useEffectEvent(() => {
    bildirimGoster("Bağlanıldı!", tema);   // her zaman güncel tema'yı görür
  });

  useEffect(() => {
    const baglanti = baglan(odaId);
    baglanti.on("baglandi", () => onBaglandi());
    baglanti.baglan();
    return () => baglanti.kes();
  }, [odaId]);   // ✅ tema yok; onBaglandi de gerekmez
}
```

- `useEffectEvent` ile tanımlanan fonksiyon bir **effect olayıdır**: çağrıldığı anda props ve state'in **en güncel** değerlerini görür ama **reaktif değildir.**
- Bağımlılık dizisine yazılmaz.

### Kurallar

- Yalnızca **effect içinden** çağrılır; JSX'te olay yöneticisi olarak veya render sırasında kullanılmaz.
- Başka bileşenlere prop olarak geçirilmez.
- Bağımlılık uyarılarını susturmak için değil, gerçekten "bu mantık bir olay gibi davranmalı" durumları için kullanılır.

### Bir başka örnek

```jsx
const onZiyaret = useEffectEvent((url) => {
  logZiyaret(url, sepettekiUrunSayisi);   // sayı değişince yeniden loglanmasın
});

useEffect(() => {
  onZiyaret(url);
}, [url]);   // sadece sayfa değişince logla
```

> 📜 React 19.2 öncesinde `experimental_useEffectEvent` adıyla deneysel sürümlerdeydi.

---

## 12. `useLayoutEffect`

| Hook | Ne zaman çalışır | Boyamayı engeller mi? |
|---|---|---|
| `useEffect` | Tarayıcı ekranı **boyadıktan sonra** | Hayır |
| `useLayoutEffect` | DOM güncellendikten sonra, **boyamadan önce** | **Evet** |

Ekrana çizilmeden önce DOM'u ölçüp ona göre state güncellemek gerektiğinde (ör. tooltip konumu) kullanılır; aksi halde kullanıcı bir anlık yanlış konumu ve "zıplamayı" görebilir (titreme).

```jsx
import { useLayoutEffect, useRef, useState } from "react";

function Tooltip({ children }) {
  const ref = useRef(null);
  const [yukseklik, setYukseklik] = useState(0);

  useLayoutEffect(() => {
    setYukseklik(ref.current.getBoundingClientRect().height);   // boyamadan önce ölç
  }, []);

  // yukseklik'e göre konum hesapla...
  return <div ref={ref}>{children}</div>;
}
```

> **Kural:** Varsayılan `useEffect`. Sadece görsel titreme sorunu varsa `useLayoutEffect`.

---

## 13. Her Şey Bir Arada: Pomodoro Sayacı

```jsx
import { useEffect, useEffectEvent, useState } from "react";

const SURELER = { calisma: 25 * 60, mola: 5 * 60 };

function formatla(sn) {
  const dk = String(Math.floor(sn / 60)).padStart(2, "0");
  const s = String(sn % 60).padStart(2, "0");
  return `${dk}:${s}`;
}

// --- Veri çekme: race condition korumalı ---
function GununSozu({ yenile }) {
  const [soz, setSoz] = useState(null);
  const [durum, setDurum] = useState("yukleniyor");

  useEffect(() => {
    const controller = new AbortController();
    setDurum("yukleniyor");

    fetch("https://dummyjson.com/quotes/random", { signal: controller.signal })
      .then((res) => {
        if (!res.ok) throw new Error();
        return res.json();
      })
      .then((veri) => {
        setSoz(veri);
        setDurum("basarili");
      })
      .catch((err) => {
        if (err.name !== "AbortError") setDurum("hata");
      });

    return () => controller.abort();
  }, [yenile]);

  if (durum === "yukleniyor") return <p>Söz yükleniyor...</p>;
  if (durum === "hata") return <p>Söz yüklenemedi.</p>;
  return (
    <blockquote>
      “{soz.quote}” — <cite>{soz.author}</cite>
    </blockquote>
  );
}

export default function Pomodoro() {
  const [mod, setMod] = useState("calisma");           // "calisma" | "mola"
  const [kalan, setKalan] = useState(SURELER.calisma);
  const [calisiyor, setCalisiyor] = useState(false);
  const [tamamlanan, setTamamlanan] = useState(
    () => Number(localStorage.getItem("pomodoro-tamamlanan")) || 0
  );
  const [sesAcik, setSesAcik] = useState(true);

  // --- Türetilmiş değer (effect DEĞİL) ---
  const yuzde = 100 - (kalan / SURELER[mod]) * 100;

  // --- Olay yöneticileri ---
  function handleBaslatDurdur() {
    setCalisiyor((c) => !c);
  }

  function handleSifirla() {
    setCalisiyor(false);
    setKalan(SURELER[mod]);
  }

  function handleModDegistir(yeniMod) {
    setMod(yeniMod);
    setKalan(SURELER[yeniMod]);
    setCalisiyor(false);
  }

  // --- Effect olayı: süre bitince (sesAcik ve mod'u güncel okur, onlara tepki vermez) ---
  const onSureBitti = useEffectEvent(() => {
    if (sesAcik) new Audio("/bip.mp3").play().catch(() => {});
    if (mod === "calisma") setTamamlanan((t) => t + 1);
    const sonraki = mod === "calisma" ? "mola" : "calisma";
    setMod(sonraki);
    setKalan(SURELER[sonraki]);
    setCalisiyor(false);
  });

  // 1) Zamanlayıcı (dış sistem: setInterval)
  useEffect(() => {
    if (!calisiyor) return;
    const id = setInterval(() => {
      setKalan((k) => k - 1);
    }, 1000);
    return () => clearInterval(id);
  }, [calisiyor]);

  // 2) Süre bitti mi? (zamanlayıcının sonucuna tepki)
  useEffect(() => {
    if (kalan <= 0) onSureBitti();
  }, [kalan]);

  // 3) Sekme başlığı (dış sistem: document)
  useEffect(() => {
    const eskiBaslik = document.title;
    document.title = `${formatla(kalan)} · ${mod === "calisma" ? "Çalışma" : "Mola"}`;
    return () => {
      document.title = eskiBaslik;
    };
  }, [kalan, mod]);

  // 4) Kalıcı kayıt (dış sistem: localStorage)
  useEffect(() => {
    localStorage.setItem("pomodoro-tamamlanan", String(tamamlanan));
  }, [tamamlanan]);

  // 5) Klavye kısayolları (dış sistem: window olayları)
  const onKisayol = useEffectEvent((e) => {
    if (e.target.tagName === "INPUT" || e.target.tagName === "TEXTAREA") return;
    if (e.code === "Space") {
      e.preventDefault();
      handleBaslatDurdur();
    }
    if (e.key === "r" || e.key === "R") handleSifirla();
  });

  useEffect(() => {
    function handleKeyDown(e) {
      onKisayol(e);              // effect olayı, effect içindeki fonksiyondan çağrılır
    }
    window.addEventListener("keydown", handleKeyDown);
    return () => window.removeEventListener("keydown", handleKeyDown);
  }, []);

  return (
    <main className={`pomodoro ${mod}`}>
      <div>
        <button onClick={() => handleModDegistir("calisma")} disabled={mod === "calisma"}>
          Çalışma
        </button>
        <button onClick={() => handleModDegistir("mola")} disabled={mod === "mola"}>
          Mola
        </button>
      </div>

      <h1>{formatla(kalan)}</h1>
      <progress value={yuzde} max={100} />

      <div>
        <button onClick={handleBaslatDurdur}>{calisiyor ? "⏸ Durdur" : "▶️ Başlat"}</button>
        <button onClick={handleSifirla}>↺ Sıfırla</button>
        <label>
          <input
            type="checkbox"
            checked={sesAcik}
            onChange={(e) => setSesAcik(e.target.checked)}
          />
          Ses
        </label>
      </div>

      <p>Bugün tamamlanan: {"🍅".repeat(tamamlanan) || "—"}</p>
      <small>Kısayollar: Boşluk = başlat/durdur, R = sıfırla</small>

      <GununSozu yenile={tamamlanan} />
    </main>
  );
}
```

### Bu örnekte kullanılanlar

- Her effect **tek bir dış sistemle** senkronize: zamanlayıcı, sekme başlığı, `localStorage`, klavye olayları, ağ
- Her effect'in cleanup'ı var
- Zamanlayıcıda updater → `kalan` bağımlılık değil
- `useEffectEvent` → `sesAcik`, `mod` güncel okunur ama onlara tepki verilmez; klavye dinleyicisi yalnızca bir kez eklenir
- `yuzde` effect değil, **türetilmiş değer**
- Butonlar effect değil, **olay yöneticisi** kullanıyor
- Veri çekme `AbortController` ile race condition'a karşı korumalı; `yenile` prop'u değişince yeni söz çekiliyor
- Lazy initializer ile `localStorage`'dan okuma

---

## 14. Özet

- **Effect**, bileşeni React dışındaki bir sistemle **senkronize etmek** içindir; render saf, olay yöneticileri eyleme bağlı, effect'ler ise bileşenin ekranda olmasına bağlı yan etkilerin yeridir.
- Effect, render ve commit'ten **sonra**, tarayıcı boyadıktan sonra çalışır.
- **Bağımlılık dizisi:** yok → her render; `[]` → sadece mount; `[a, b]` → ilk render + değişimlerde (`Object.is`).
- **Cleanup**, effect tekrar çalışmadan önce ve unmount'ta çağrılır; setup ne başlatıyorsa cleanup onu durdurur.
- **StrictMode** geliştirmede setup → cleanup → setup yapar; çözüm doğru cleanup yazmaktır.
- Yaşam döngüsü değil **senkronizasyon** düşünülür: "Neyi senkronize ediyor, nasıl durdurulur?" Her effect tek bir iş yapar.
- Effect içinde kullanılan **her reaktif değer** bağımlılıktır; linter dinlenir, bağımlılıklar kodu değiştirerek azaltılır.
- Bağımlılıktan kurtulma yolları: sabitleri dışarı taşımak, **updater** kullanmak, nesne/fonksiyonları effect **içine** almak, `useEffectEvent`.
- Her render'da yeni oluşan nesne/fonksiyon bağımlılıkları ve effect içi `setState` **sonsuz döngüye** yol açabilir.
- Effect fonksiyonu `async` olamaz; içinde async fonksiyon tanımlanıp çağrılır.
- Effect ile veri çekerken **race condition** `ignore` bayrağı veya **`AbortController`** ile önlenir; gerçek projelerde TanStack Query, framework'ler veya `use` tercih edilir.
- **Effect'e ihtiyaç olmayan durumlar:** türetilmiş değerler, pahalı hesaplamalar, prop değişince sıfırlama (`key`), kullanıcı eylemleri, form gönderimi, effect zincirleri, ebeveyni bilgilendirme, uygulama başlangıcı.
- **`useEffectEvent`** (React 19.2): effect içinde güncel değerleri okuyan ama reaktif olmayan mantık; sadece effect içinden çağrılır.
- **`useLayoutEffect`** boyamadan önce çalışır; yalnızca görsel titreme sorunlarında kullanılır.

---

⬅️ Önceki: [09 · Formlar](./09-formlar.md) · ➡️ Sonraki: [11 · useRef](./11-useref.md)
