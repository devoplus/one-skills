---
name: one-access-requests
description: Bir Devoplus One uygulaması kayıt, portala kurulum, ERP (DEXA) sorgusu ya da işlemi, uygulama izni ataması veya yeni bir yetki alanı gerektirdiğinde kullanın. Talebi Devoplus One MCP aracıyla (ya da one request komutuyla) yöneticiye gönderir ve kullanıcıya sade dille bildirir.
---

# Devoplus One — erişim talebi

Eksik bir izni kullanıcıya teknik adım olarak anlatmayın; **talebi kendiniz gönderin**. Talep izni kendiliğinden
vermez; yönetici Admin Console'da onaylar, sonuç e-postayla bildirilir. Ayrıntı: `/docs/access-requests.md`.

## Nasıl

- MCP bağlıysa: `one_request_access` (gönder), `one_list_my_requests` ve `one_get_request` (izle).
- Değilse: `one login` (bir kez) → `one request <tür> …` → `one requests`.

| Tür | Ne zaman | Zorunlu |
|---|---|---|
| `registration` | Uygulama henüz kayıtlı değil (onaylanınca gönderen sahip olur) | `titleTr` |
| `installation` | Uygulama bir portalda görünsün | `portalKey` |
| `dexa-query` | ERP'den hazır bir sorgu okunsun | `queryKey` |
| `dexa-command` | ERP'ye hazır bir işlemle yazılsın | `queryKey` |
| `app-permission` | Uygulama içi rol bir gruba atansın | `permissionCode` |
| `scope` | Uygulamaya yeni bir yetki alanı | `scope` |
| `new-query` | İstenen ERP verisi için sorgu yok | `description` (+ öneri `draftSql`, çalıştırılmaz) |

Her talepte `appSlug` (manifest `app.slug`) ve kullanıcının iş diliyle kısa bir `justification` verin.

## Kullanıcıya ne söylenir

- Gönderince: "… için yöneticinizden onay istedim. Onaylanınca e-postayla haber verilecek; bu arada diğer işlere
  devam ediyorum."
- Aynı talep zaten bekliyorsa yenisini göndermeyin; durumunu bildirin.
- Reddedilirse yöneticinin notunu olduğu gibi iletin ve birlikte başka bir yol düşünün.
- Talep yalnız uygulamanın sahibi tarafından gönderilebilir (kayıt hariç) ve Vekalet / Yerine Görüntüle oturumunda
  gönderilemez.
