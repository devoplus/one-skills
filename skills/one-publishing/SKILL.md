---
name: one-publishing
description: Bir Devoplus One uygulamasını yayına hazırlarken, Devoplus GitLab'a gönderirken, test ortamına yayınlayıp önizleme adresi verirken, sürümü bir sonraki ortama taşırken ya da önceki sürüme geri alırken, hattın (CI) bir kapısı durduğunda ve kullanıcı yayındaki uygulamada bir sorun bildirdiğinde kullanın. Dal ve commit kurallarını, kapıları, yayın araçlarını (one_publish · one_deployments · one_app_health) ve kullanıcıya nasıl bildirileceğini anlatır.
---

# Devoplus One — yayın

Kod geliştiricinin makinesinden yayımlanmaz. Uygulamanın deposu Devoplus'ın dışa kapalı GitLab'ındadır; platformun
hattı kapılardan geçirir ve yayımlar. Ayrıntı: `/docs/publishing.md`.

## Kurallar

- İş `ai/<konu>` dalında yapılır ve birleştirme isteğiyle girer; yapay zekâ ile yazılan commit iletisine
  `AI-Assisted: <araç adı>` satırı eklenir.
- Depoya CI dosyası (`.gitlab-ci.yml`) yazılmaz; platformun hattı koşar. `pnpm-lock.yaml` depoya girer, kurulum
  betiği (`postinstall`) eklenmez.
- Her sürümde `one.manifest.json` → `app.version` artırılır.
- Göndermeden önce yerelde `pnpm test` ve `one build` çalıştırın.

## Yayın, önizleme, terfi, geri alma

Yayını kullanıcıya GitLab'da yaptırmayın; siz isteyin (ayrıntı `/docs/publishing.md`):

- **Test ortamına yayın:** `one_publish` (`action: "deploy"`) ya da `one publish`. Hat bitince kullanıcıya önizleme
  adresini verin: *"Önizleme adresiniz şu: … — test Portal'ından açıp bakabilirsiniz."*
- **Terfi:** `action: "promote"` / `one promote` — kullanıcının açık onayıyla.
- **Geri alma:** `action: "rollback"` + `confirm: true` / `one rollback --yes`. Dönülen noktadan sonra kaydedilen veriler
  kaybolabilir; kullanıcıya bunu söyleyip açık onay almadan göndermeyin. Kod `v<sürüm>` git etiketinden alınır: yayına
  giren her sürümü etiketleyin (`git tag v<app.version>` + `git push origin v<app.version>`); etiketsiz sürüme dönülemez.
- **Durum:** `one_deployments` / `one deployments`.
- Yanıt "yayın hattı bu ortamda yapılandırılmamış" ise kullanıcıya yalnız bunu söyleyin.

## Sorun bildirildiğinde

Kullanıcı "çalışmıyor / hata veriyor / yavaş" derse kodu değiştirmeden önce `one_app_health` (ya da `one status`) ile
kullanım ve hata özetine bakın (`/docs/health.md`). Hata iletilerini kullanıcıya teknik dille aktarmayın; sorunu ve ne
yaptığınızı sade dille anlatın, düzeltmeyi test ortamına yayınlayıp önizlemede denemesini isteyin.

## Kapılar

Hat şunlarda durur: geçersiz manifest ya da eksik `README.md`/`AGENTS.md` · satır içi ya da dış betik · depoya girmiş
sır · yüksek önemli güvenlik bulgusu · açığı bilinen bağımlılık · uygulanmış migration'ın değişmesi ya da yıkıcı SQL ·
düşük sürüm · (platform açtıysa) uat'ta doğrulanmamış sürümün canlıya gitmesi.

Gerçek olmayan bir sır ya da güvenlik bulgusu `.one-secrets-ignore` / `.one-sast-ignore` dosyasına parmak izi +
**gerekçe** ile eklenir; gerekçesiz satır kapıyı geçmez. Sır bir kez depoya girdiyse silmek yetmez: iptal edilip
yenilenmesi gerekir — kullanıcıya bunu açıkça söyleyin.

## Kullanıcıya

Teknik ayrıntı değil sonuç anlatılır: "Güvenlik kontrolü bir sorun buldu, düzeltip yeniden gönderdim" · "Test
ortamına yüklendi; önizleme adresiniz şu, test Portal'ından açıp bakabilirsiniz" · "Portal'da görmek için kurulum
onayı bekleniyor".
