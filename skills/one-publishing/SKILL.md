---
name: one-publishing
description: Bir Devoplus One uygulamasını yayına hazırlarken, kodu Devoplus'a yüklerken, test ortamına yayınlayıp önizleme adresi verirken, sürümü bir sonraki ortama taşırken ya da önceki sürüme geri alırken, hattın (CI) bir kapısı durduğunda ve kullanıcı yayındaki uygulamada bir sorun bildirdiğinde kullanın. Yükleme kurallarını, kapıları, yayın araçlarını (one_source_upload, one_publish, one_pipeline_status, one_deployments, one_app_health, one_dependency_patch) ve kullanıcıya nasıl bildirileceğini anlatır.
---

# Devoplus One: yayın

Kod geliştiricinin makinesinden yayımlanmaz. Uygulamanın deposu Devoplus'ın dışa kapalı GitLab'ındadır ve size kapalıdır:
kodu Devoplus API'si depoya yazar, platformun hattı kapılardan geçirir ve yayımlar. Ayrıntı: `/docs/publishing.md`.

## Kurallar

- Kodu yüklemenin iki yolu var: komut satırında uygulamanın kök dizininde `one publish` (projenin tam görüntüsünü yükler
  ve test ortamına yayınlar) ya da dosya sisteminiz yoksa `one_source_upload` (değişen dosyalar, `patch`). Yükleme yalnız
  test ortamına yapılır; kullanıcıdan GitLab'a girmesini istemeyin.
- Depoya CI dosyası (`.gitlab-ci.yml`) ve sır dosyası (`.env`, `.dev.vars`) gönderilmez; platformun hattı koşar.
  `pnpm-lock.yaml` depoya girer, kurulum betiği (`postinstall`) eklenmez.
- Her sürümde `one.manifest.json` içindeki `app.version` artırılır.
- Yüklemeden önce yerelde `pnpm test` ve `one build` çalıştırın.

## Yayın, önizleme, terfi, geri alma

Yayını kullanıcıya GitLab'da yaptırmayın; siz isteyin (ayrıntı `/docs/publishing.md`):

- **Test ortamına yayın:** kodu yükleyin (`one publish` ya da `one_source_upload`); yayın aynı çağrıda başlar. Depodaki
  kodu yüklemeden yeniden yayınlamak için `one_publish` (`action: "deploy"`). Hat bitince kullanıcıya önizleme adresini
  verin: *"Önizleme adresiniz şu: `<adres>`. Test Portal'ından açıp bakabilirsiniz."*
- **Hat düşerse:** `one_pipeline_status` (`one pipeline`) son başarısız işin günlüğünü gösterir; nedeni okuyup düzeltin ve
  yeniden yükleyin.
- **Hat hiç açılmadan `409` gelirse:** platform yayın ve terfide manifesti hattan önce denetler; incelenecek hat işi yoktur
  (bu denetim uygulamalar arası okuma güncellemesiyle gelir; o yayına alınana kadar hata hatta görünür).
  `ONE-APP-E-164` manifest yok, başka uygulamaya ait ya da `exports` / `imports` geçersiz: düzeltip yeniden yükleyin.
  `ONE-APP-E-163` aynı ad ve sürümdeki dışa açımın sözleşmesi değişmiş: yeni `version` verin, eskisini birlikte sunun.
  `ONE-APP-E-162` aynı portalda onaylı okunan bir sürüm kaldırılmış: sürümü geri ekleyin, alan uygulama geçene kadar
  ikisini birlikte sunun. Ayrıntı `/docs/publishing.md` ve `/docs/manifest.md` "Yayın kuralları".
- **Terfi:** yalnız test'ten uat'a ve yalnız kullanıcının açık onayıyla (`action: "promote"` uat oturumuyla / `one promote`).
  Canlıya terfi ve canlıda geri alma insan onaylıdır: uygulamanın sahibi Portal'daki "Devoplus One Uygulamalarım"
  ekranından, şirket yöneticisi Devoplus Console'dan yapar.
- **Geri alma:** `action: "rollback"` + `confirm: true` / `one rollback --yes` (test ve uat). Dönülen noktadan sonra
  kaydedilen veriler kaybolabilir; kullanıcıya bunu söyleyip açık onay almadan göndermeyin. Kod `v<sürüm>` etiketinden
  alınır; etiketi Devoplus her terfide kendisi yazar, hiç terfi etmemiş sürüme dönülemez.
- **Onay kartı:** ortamınız seçenekli soru kartı gösterebiliyorsa (örneğin Claude Code'da `AskUserQuestion`) terfi ve
  geri alma onayını kartla alın; uyarıyı önce metinle yazın, "Evet" şıkkını önerilen olarak işaretlemeyin.
- **Durum:** `one_deployments` / `one deployments`.
- Yanıt "yayın hattı bu ortamda yapılandırılmamış" ise kullanıcıya yalnız bunu söyleyin.

## Sorun bildirildiğinde

Kullanıcı "çalışmıyor / hata veriyor / yavaş" derse kodu değiştirmeden önce `one_app_health` (ya da `one status`) ile
kullanım ve hata özetine bakın (`/docs/health.md`). Hata iletilerini kullanıcıya teknik dille aktarmayın; sorunu ve ne
yaptığınızı sade dille anlatın, düzeltmeyi test ortamına yayınlayıp önizlemede denemesini isteyin.

## Bağımlılık açığı

Günlük bağımlılık taraması bilinen açığı olan bir paket bulursa (hat bağımlılık kapısında durduğunda da) önce
`one_dependency_patch` (`one deps patch`) ile bulguları ve platformun önerdiği yamayı okuyun. Yama kendiliğinden
uygulanmaz: yerel kopyada uygulayın (komut satırında `one deps patch --apply`; kaydedilmemiş değişiklikte ya da yama
uymadığında hiçbir dosyaya dokunmaz), testleri ve `one build`u çalıştırın, sonra her zamanki yoldan yükleyin. Yama
yoksa ya da uymuyorsa paketi düzeltildiği sürüme elle yükseltin. Sonuç yoksa notlardaki nedeni söyleyin, bulgu
uydurmayın. Ayrıntı: `/docs/publishing.md`.

## Kapılar

Hat şunlarda durur: geçersiz manifest ya da eksik `README.md`/`AGENTS.md`; satır içi ya da dış betik; depoya girmiş
sır; yüksek önemli güvenlik bulgusu; açığı bilinen bağımlılık; uygulanmış migration'ın değişmesi ya da yıkıcı SQL;
düşük sürüm; (platform açtıysa) uat'ta doğrulanmamış sürümün canlıya gitmesi.

Hat yayını durdurmadan **uyarı** da yazar. `⚠ SDK:` satırı, kurulu `@devoplus/one-sdk` asgari sürümden (bugün
`0.2.0-next.158`) eski ya da sürümü okunamıyor demektir: SDK'yı güncelleyin (`pnpm add @devoplus/one-sdk@next`) ve
`pnpm-lock.yaml`'ı da yükleyin. Platform bu denetimi ileride zorlayıcı yapabilir; uyarıyı kalıcı bırakmayın.

Gerçek olmayan bir sır ya da güvenlik bulgusu `.one-secrets-ignore` / `.one-sast-ignore` dosyasına parmak izi +
**gerekçe** ile eklenir; gerekçesiz satır kapıyı geçmez. Sır bir kez depoya girdiyse silmek yetmez: iptal edilip
yenilenmesi gerekir. Kullanıcıya bunu açıkça söyleyin.

## Kullanıcıya

Teknik ayrıntı değil sonuç anlatılır: "Güvenlik kontrolü bir sorun buldu, düzeltip yeniden gönderdim", "Test
ortamına yüklendi; önizleme adresiniz şu, test Portal'ından açıp bakabilirsiniz", "Portal'da görmek için kurulum
onayı bekleniyor".
