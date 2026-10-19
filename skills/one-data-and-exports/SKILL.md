---
name: one-data-and-exports
description: Bir Devoplus One uygulaması kayıt tutacaksa (talep, rezervasyon, envanter, anket cevabı …) ya da kullanıcı Excel, CSV, çıktı veya PDF isterse kullanın. one.data.json veri modeli, one data generate, createCrudHandler ile hazır kayıt uçları ve toXlsx / toCsv / renderPrintableReport ile dışa aktarmayı anlatır.
---

# Devoplus One — veri modeli, kayıt işlemleri ve dışa aktarma

Ayrıntılı belgeler (kurulum prompt'unun geldiği adreste): `/docs/data-model.md` · `/docs/exports.md` · `/docs/sdk.md`.

## Kayıt tutan her uygulamada sıra

1. Keşif görüşmesinde kayıt türlerini, alanlarını, zorunlu ve tekil alanları, seçimli alanların **bütün** seçeneklerini
   ve kayıtları kimin göreceğini (herkes mi, yalnız ekleyen mi) kullanıcıyla netleştirin. Teknik ad sormayın.
2. `one.data.json` yazın (varlık adı ve alan adı küçük harf + alt çizgi; başlıklar `{ "tr", "en" }`).
3. `one data generate` çalıştırın; üretilen migration, `src/data/types.ts` ve `migrations/.one-data-snapshot.json`
   dosyalarını depoya ekleyin. Komut bir değişikliği reddederse nedenini kullanıcıya sade dille anlatın.
4. Uçları `createCrudHandler({ model: dataModel, entity, records, canWrite })` ile bağlayın; tabloyu, tipleri ve
   kayıt uçlarını elle yazmayın.
5. `one migrate` → `one dev` ile deneyin.

## Değişmez kurallar

- **Model yalnız büyür.** Alan/varlık silme, ad veya tür değiştirme, zorunlu ↔ isteğe bağlı, tekillik değişikliği ve
  seçim değeri çıkarma üretilmez; seçim alanına yeni değer eklemek serbesttir. Var olan kayıt türüne zorunlu alan
  eklenecekse `default` verin.
- Arama Türkçe katlanmıştır (`<alan>_fold` arama kolonu); bir metin alanını elle yazdığınız SQL ile güncellerseniz arama
  kolonunu da `foldSearchText(değer)` ile yazın ya da `NULL` yapın.
- Üretilmiş migration dosyası değiştirilmez; anlık görüntü dosyası elle düzenlenmez.
- `records` her zaman bilinçli seçilir: `shared` (uygulamayı kullanan herkes görür) ya da `own` (yalnız ekleyen).
- Rol gerekiyorsa uygulama izni (`canWrite: 'One.<Uygulama>.<Eylem>'`) — e-posta listesiyle yetki yazılmaz.
- Para alanı kuruş hassasiyetindedir; üçüncü ondalık reddedilir, yuvarlanmaz.
- Vekalet / Yerine Görüntüle oturumunda kayıtlar varsayılan olarak yalnız görüntülenir.

## Dışa aktarma

- Veri modeli varsa hazır: `/api/<varlık>/export.xlsx` · `export.csv` · `report.html` (liste süzgeçleriyle aynı).
- Kendi rapor verisi için `toXlsx` / `toCsv` + `exportResponse`; yazdırma için `renderPrintableReport` + `reportResponse`.
- Arayüzde dosya `one.fetch` ile alınır ve `one.download` ile Portal'a verilir (çerçevede doğrudan indirme kapalıdır).
- PDF sunucuda üretilmez; yazdırma sayfası açılır ve kullanıcı "PDF olarak kaydet"i seçer. Bunu kullanıcıya baştan söyleyin.
