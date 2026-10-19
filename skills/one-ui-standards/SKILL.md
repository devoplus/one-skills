---
name: one-ui-standards
description: Bir Devoplus One uygulamasının sayfasını, ekranını, menüsünü, bildirimlerini ya da görünümünü yazarken kullanın. Devoplus görsel dili (/one-ui.css), Portal köprüsü (window.one), sekmeler, tema/dil ve yayını durduran arayüz kurallarını anlatır.
---

# Devoplus One — arayüz standartları

Uygulama Portal'ın içinde bir çerçevede açılır; kullanıcı onu Portal'ın bir parçası gibi görmelidir. Ayrıntı:
`/docs/sdk.md` ("Portal köprüsü ve görsel dil") ve `/docs/app-rules.md`.

## Kurallar

- **Hazır bileşenler önce:** liste, kayıt formu, süzgeç, kişi seçimi, dosya yükleme ya da grafik gerekiyorsa
  `/one-components.js` bileşenlerini kullanın (`<one-table>` · `<one-form>` · `<one-filter>` · `<one-person-picker>` ·
  `<one-file-upload>` · `<one-chart>`); elle tablo/form yazmayın. Betik köprüden SONRA yüklenir; kolon ve alan
  tanımları `/shell.js`'te JavaScript özelliği olarak atanır, etiketler `{ tr, en }`. Liste ucu
  `{ items, nextCursor }` döndürür; kişi seçici için sunucuda `peopleSearchHandler` (+ `orgschema.read`). Ayrıntı:
  `/docs/ui-components.md`.
- **Görsel dil:** sayfa `/one-ui.css`'i yükler; renk, yazı tipi ve bileşenler (düğme, alan, tablo, kart, mesaj, sekme)
  oradan gelir. Açık/koyu tema ve Portal'ın tasarım değerleri köprüyle uygulanır.
- **Köprü:** Portal ile her konuşma `window.one` (`/one-bridge.js`) üzerinden; `postMessage` doğrudan yazılmaz.
  İstekler `one.fetch`, bildirim `one.toast`, onay `one.confirm`, indirme `one.download`, dış bağlantı
  `one.openExternal`, kaydedilmemiş değişiklik `one.setDirty`, alt başlık `one.setTitle`.
- **Menü çizilmez.** Görünümler manifest `navigation` içinde beyan edilir; Portal bunları sekme olarak çizer. Bir sekme
  yalnız belli bir izni olanlara görünecekse öğeye `"permission": "One.<Uygulama>.<Eylem>"` yazın ve sayfa oturumu
  okuyunca `one.setNavigationAccess({ permissions, admin })` çağırın; veriyi sunucu ucunda ayrıca koruyun.
  Uygulama kendi üst bilgisini, sol menüsünü ya da sabit konumlu çubuğunu çizmez.
- **Betik kuralı (yayını durdurur):** satır içi `<script>`, başka adresten betik, `onclick="…"` gibi satır içi olay
  işleyicisi ve `javascript:` adresi YAZILMAZ. Betik uygulamayla birlikte sunulur (`/shell.js`), olaylar betikte bağlanır.
- **Dil:** her kullanıcı metni Türkçe + İngilizce, resmî "siz"; Portal dili değişince sayfa da değişir.
- **Yükseklik:** kök öğelerde `100vh` / `height:100%` kullanılmaz; sayfa yüksekliğini köprü Portal'a bildirir.
- **Vekalet / Yerine Görüntüle:** bu oturumlar varsayılan salt-okunurdur; yazma düğmelerini kapatıp nedenini yazın.
