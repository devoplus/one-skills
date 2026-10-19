---
name: one-access-requests
description: Bir Devoplus One uygulaması kayıt, portala kurulum, ERP (DEXA) sorgusu ya da işlemi, uygulama izni ataması veya yeni bir yetki alanı gerektirdiğinde kullanın. Talebi Devoplus One MCP aracıyla (ya da one request komutuyla) yöneticiye gönderir ve kullanıcıya sade dille bildirir.
---

# Devoplus One: erişim talebi

Eksik bir izni kullanıcıya teknik adım olarak anlatmayın; **talebi kendiniz gönderin**. Talep izni kendiliğinden
vermez; yönetici Admin Console'da onaylar, sonuç e-postayla bildirilir. Ayrıntı: `/docs/access-requests.md`.

## Nasıl

- MCP bağlıysa: `one_request_access` (gönder), `one_list_my_requests` ve `one_get_request` (izle),
  `one_portal_context` (oturumdaki portalın adı ve var olan kısa adı, salt okuma).
- Değilse: bir kez `one login`, sonra `one request <tür> ...`, durumu izlemek için `one requests`, portal için
  `one context`.

| Tür | Ne zaman | Zorunlu |
|---|---|---|
| `registration` | Uygulama henüz kayıtlı değil (onaylanınca gönderen sahip olur) | `titleTr` (+ isteğe bağlı `additionalPermissions`) |
| `installation` | Uygulama bir portalda görünsün | yok (`portalKey` isteğe bağlı) |
| `dexa-query` | ERP'den hazır bir sorgu okunsun | `queryKey` |
| `dexa-command` | ERP'ye hazır bir işlemle yazılsın | `queryKey` |
| `app-permission` | Uygulama içi rol bir gruba atansın | `permissionCode` |
| `scope` | Uygulamaya yeni bir yetki alanı | `scope` |
| `new-query` | İstenen ERP verisi için sorgu yok | `description` (+ öneri `draftSql`, çalıştırılmaz) |

Her talepte `appSlug` (manifest `app.slug`) ve kullanıcının iş diliyle kısa bir `justification` verin.

## Kim onaylar, neye dikkat edilir

- Kayıt, kurulum, sorgu ve izin taleplerini talebin portalının **şirket yöneticisi** onaylar. Yalnız yetki alanı
  (`scope`) talepleri platform yöneticisine düşer.
- **Ek izinler kayıt talebinde istenir.** Uygulama içinde rol gerekiyorsa (manifest `permissions`), hepsini
  `additionalPermissions` olarak kayıtla birlikte gönderin: `[{ code: "One.<Uygulama>.<Eylem>", nameTr, nameEn }]`,
  en çok 16. Kayıttan sonra eklenecek izin için platform yöneticisi gerekir.
- `app-permission` talebinin kodu uygulamanın kaydında tanımlı olmalıdır; kayıtta olmayan izin için talep reddedilir.
- **Kurulumda kullanıcıya portal adresini ya da anahtarını sormayın.** `portalKey` isteğe bağlıdır: verilmezse
  portalın var olan anahtarı kullanılır, portalda henüz kurulum yoksa yönetici onayda belirler. Merak ederseniz
  `one_portal_context` ile okuyun.

## Kullanıcıya ne söylenir

- Gönderince: "<konu> için yöneticinizden onay istedim. Onaylanınca e-postayla haber verilecek; bu arada diğer işlere
  devam ediyorum."
- Aynı talep zaten bekliyorsa yenisini göndermeyin; durumunu bildirin.
- Reddedilirse yöneticinin notunu olduğu gibi iletin ve birlikte başka bir yol düşünün.
- Talep yalnız uygulamanın sahibi tarafından gönderilebilir (kayıt hariç) ve Vekalet / Yerine Görüntüle oturumunda
  gönderilemez.
