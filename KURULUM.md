# Soru Defteri → PWA kurulum

## 1) Dosyaları depoya ekleyin
Bu klasördeki her şeyi `index.html` ile **aynı klasöre** koyun:

```
Sorudefteri/
├── index.html          (mevcut)
├── manifest.webmanifest
├── sw.js
└── icons/
    ├── icon-192.png
    ├── icon-512.png
    ├── icon-maskable-512.png
    └── apple-touch-icon.png
```

## 2) `index.html` içindeki `<head>` bölümüne ekleyin
`<title>` satırının hemen altına yapıştırın:

```html
<link rel="manifest" href="manifest.webmanifest">
<meta name="theme-color" content="#0f766e">
<link rel="apple-touch-icon" href="icons/apple-touch-icon.png">
<meta name="mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-title" content="Soru Defteri">
<meta name="apple-mobile-web-app-status-bar-style" content="default">
```

## 3) `</body>` kapanışından hemen önce ekleyin

```html
<script>
// Service worker kaydı (çevrimdışı çalışma)
if ('serviceWorker' in navigator) {
  window.addEventListener('load', () => navigator.serviceWorker.register('sw.js').catch(console.error));
}
// Yan menüye "Uygulamayı Yükle" öğesi (Android/Chrome)
let deferredInstall;
window.addEventListener('beforeinstallprompt', e => {
  e.preventDefault(); deferredInstall = e;
  const menu = document.querySelector('.nav-menu');
  if (!menu || document.getElementById('btnInstallApp')) return;
  const a = document.createElement('a');
  a.className = 'nav-item'; a.id = 'btnInstallApp';
  a.innerHTML = '<span class="icon">📲</span> Uygulamayı Yükle';
  a.onclick = async () => { deferredInstall.prompt(); await deferredInstall.userChoice; deferredInstall = null; a.remove(); };
  menu.appendChild(a);
});
</script>
```

## 4) HTTPS ile yayınlayın
PWA yalnızca HTTPS'te (veya localhost) kurulur. En kolayı **GitHub Pages**:
Depo → Settings → Pages → Branch: `main` / `(root)` → Save.
Adres şu olur: `https://MAVI2-PSH.github.io/Sorudefteri/`

## 5) Telefona kurun
- **Android (Chrome):** sayfayı açın → menüden "Uygulamayı Yükle" ya da Chrome ⋮ → "Ana ekrana ekle / Uygulamayı yükle".
- **iPhone (Safari):** Paylaş → "Ana Ekrana Ekle".

## Güncelleme yaparken
`index.html` değiştirdiğinizde `sw.js` içindeki `VERSION` değerini (`sorudefteri-v1` → `v2`) artırın; böylece eski önbellek temizlenir.

## Dikkat
- Çevrimdışı modda arayüz açılır; giriş ve veri senkronizasyonu için internet gerekebilir (bu, `index.html` içindeki JavaScript'in veriyi nereye kaydettiğine bağlıdır).
- Service worker yalnızca kendi dosyalarınızı ve flatpickr/Google Fonts gibi CDN kaynaklarını önbelleğe alır; giriş/veritabanı istekleri her zaman doğrudan ağa gider.
