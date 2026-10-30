---
name: one-app-development
description: Bir Devoplus One uygulamasının kodunu yazarken ya da değiştirirken kullanın (one.manifest.json, @devoplus/one-sdk, src/index.ts, src/agent.ts, migrations, AGENTS.md, AGENT_GUIDE.md). Kimlik, veritabanı, dosya, önbellek, organizasyon şeması, e-posta/SMS/bildirim, DEXA, denetim kaydı ve Devoplus yapay zekasıyla iki yönlü çalışmanın doğru kullanımını anlatır.
---

# Devoplus One: uygulama geliştirme

Uygulama deposunda **ilk iş `AGENTS.md`'yi okuyun**; kurallar bağlayıcıdır. Ayrıntılı başvurular (kurulum
prompt'unun geldiği adreste): `/docs/sdk.md`, `/docs/manifest.md`, `/docs/app-rules.md`.

Yeni uygulama **hazır şablondan** başlar (`npm create devoplus-one-app -- <dizin> --template <şablon>`; şablonlar
`bakim-takvimi`, `saha-denetimi`, `egitim-sertifika`, `ziyaretci-kaydi`, `uygunsuzluk-dof`, `musteri-ziyaret`, `hedef-takibi`,
`envanter`; seçim: `/docs/templates.md`). Şablondan üretilmiş uygulamada `AGENTS.md`'nin
en üstündeki şablon bölümü dosya haritasını ve bozulmaması gereken kuralları (sunucuda hesaplanan alanlar, durum akışı,
eksi stok, kişisel verinin silinmesi) yazar: değiştirin,
baştan yazmayın.

## Değişmez kurallar

- **Kimlik platformdan gelir.** `readContext(request.headers)` ile okunur; kendi oturum, çerez ya da belirteç
  mekanizması yazılmaz. `X-One-*` başlıkları yalnız okunur.
- **Her kurulumun kendi veritabanı vardır.** Tabloya portal sütunu eklenmez; kişiye ait satır `ctx.user.sub` ile
  bağlanır. Şema yalnız yeni numaralı `migrations/NNNN_*.sql` dosyasıyla ilerler; var olan dosya değiştirilmez,
  `DROP TABLE` / `ALTER ... DROP/RENAME` / WHERE'siz `DELETE` yazılmaz.
- **Dışarıya yalnız Devoplus API'si.** `BackendClient` ile ve kullanıcının belirteciyle; başka adres reddedilir.
  Sır, API anahtarı, parola koda ya da depoya yazılmaz.
- **Yetki alanı önce manifest'te.** Kullanılan her yetki alanı `one.manifest.json` dosyasının `scopes` listesinde olur; kodda
  `requireScope(ctx, '<scope>')` ile korunur.
- **Uç, alan, sorgu anahtarı uydurulmaz.** DEXA sorgusunun anahtarını, parametrelerini ve sütunlarını
  `one_dexa_queries` (ya da `one dexa queries`) listesinden alın; yoksa kullanıcıya iş diliyle sorun ya da `new-query`
  talebi açın.
- **Kendi yetki modeli kurulmaz.** Rol gerekiyorsa manifest `permissions` + `requirePermission` / `hasPermission`;
  atamayı portal yöneticisi yapar.
- **Devoplus yapay zekası uygulamaya erişebilmeli (zorunlu adım).** Veri ya da işlem sunan her özellik için:
  manifest `agent.tools[]` beyanı + `src/agent.ts` işleyicisi (`defineAgentTools`) + `AGENT_GUIDE.md` (dört başlık:
  "Uygulama ne yapar", "Veri", "Araçlar", "Örnek istekler"; her araç adı geçer). Önce salt okunur araçlar; değiştiren araç
  `readOnly: false`. Kullanıcıya teknik soru sorulmaz; hangi soruların bu uygulamaya ait olduğunu siz çıkarırsınız.
  Ayrıntı: `/docs/ai.md`.

## Servisler (SDK)

| İhtiyaç | Kullanın |
|---|---|
| Kullanıcı | `readContext`, `requireUser`, `requireScope`, `requirePermission` |
| Kayıt | Önce veri modeli: `one.data.json` + `one data generate` + `createCrudHandler` (skill `one-data-and-exports`); özel sorgu için `env.DB` |
| Excel / CSV / yazdırma | `toXlsx`, `toCsv`, `exportResponse`, `renderPrintableReport` (skill `one-data-and-exports`) |
| Dosya | `services.files` + `OneFiles` (`readBoundedBody`, `sanitizeFileName`, kullanıcı başına ön ek) |
| Önbellek | `services.cache` + `OneCache` (TTL zorunlu; kalıcı veri için değil) |
| Organizasyon | `getMyOrgContext`, `getOrgUsersWithManagers`, `approvalChain` (`orgschema.read`) |
| Bildirim | `notify`, `sendMail`, `sendSms`, `sendPush` (alıcı kullanıcı kimliği; serbest adres yok) |
| ERP okuma | `queryDexa` (`dexa.read` + yöneticinin izin verdiği sorgu anahtarı); bağlam kodları aşağıda |
| ERP yazma | `runDexaCommand` (`dexa.write` + `Idempotency-Key`; sonuç belirsizse yeni anahtarla yeniden gönderilmez) |
| Denetim | `recordAudit` (`audit.write`); kişisel veri ya da sır yazılmaz |
| Zamanlanmış / arka plan işi | manifest `jobs` + `defineJobs`, `handleJobRequest`, `enqueue` (ayrıntı: `/docs/jobs.md`) |
| Günlük | `logEvent`, `logError`; hata yanıtı `problemResponse` |
| Yapay zeka kullanma | `completeAi` (`ai.use`): özet, sınıflandırma, alan çıkarma, metin; model sayı hesaplamaz, kayıt yazmaz |
| Yapay zekaya açma | `defineAgentTools` + `handleAgentRequest` + `agentManifestResponse` (manifest `agent.tools`, `AGENT_GUIDE.md`) |

Zamanlanmış ya da kuyruk işinde aynı iş birden çok kez gelebilir: kalıcı yazımı `job.id` ile tekrarsız yapın
(`INSERT OR IGNORE`); kalıcı hatayı 4xx `OneError` ile bitirin. Kendi zamanlayıcınızı yazmayın; `/__one/` yolunda uç
açmayın. İşte Devoplus servisleri uygulamanın kimliğiyle çağrılır ve yalnız manifest `jobs.services` ile beyan edilenler
(e-posta, SMS, mobil bildirim, DEXA okuma, yapay zeka) kullanılabilir: arka plan işi servisleri kurulumda yönetici
onayıyla açılır, DEXA'ya yazma bu yolla açılmaz.

DEXA okuma sorgusunun bağlam kodları: `LangCode` platformundur, gönderilmez. `UserCode`, `CompanyCode`,
`CostCenterCode` gönderilmezse kişi/ajan kipinde oturumdaki kişiden çözülür; arka plan işinde sorgu reddedilir.
Uygulamanın bu kodları göndermesi kurulumda yöneticinin "kodları uygulama belirler" onayını ister (yoksa reddedilir);
onaylıysa her kipte gönderilebilir, kişi ve şirket kodu portala karşı doğrulanır. Boş değer ERP'ye gitmez
(`ONE-SDK-E-140`); kodu bilmiyorsanız göndermeyin, uydurmayın.

Yapay zekaya açılan araç Devoplus servislerini kişi adına çağırır; süreli onay gerektiren scope'lar (e-posta, SMS, mobil
bildirim, DEXA'ya yazma) bu yolda kullanılamaz. Araç bağlamında `ctx.admin` ve `ctx.actor` doludur.

E-posta, SMS, mobil bildirim ve ERP'ye yazma kullanıcının Portal'da ayrıca onay vermesini ister; onay yoksa
`awaitsUserConsent(ctx, scope)` ile düğmeyi kapatın ve kullanıcıya Portal'daki "İzinleri incele" düğmesini söyleyin.

## Yerelde

**Önce SDK sürümü.** Var olan bir uygulamada işe başlamadan kurulu sürümü (`pnpm list @devoplus/one-sdk`) kanaldaki en
yeni sürümle (`npm view @devoplus/one-sdk dist-tags`, ön sürüm kuruluysa `next`) karşılaştırın. `one` komutu kurulu SDK
eskiyse "Devoplus One SDK'nın daha yeni bir sürümü var" uyarısını ve güncelleme komutunu yazar; komut durmaz. Uyarıyı
görürseniz başka bir işe geçmeden güncelleyin (`pnpm add @devoplus/one-sdk@latest` (kurulu SDK ön sürümse `@next`), ardından `npx one upgrade` (sürümü tam sabitler, `one.manifest.json` `sdkVersion` alanını ve `AGENTS.md`'nin platform kuralları bölümünü yazar)), testleri ve
`one build`'i çalıştırın, `package.json`, `pnpm-lock.yaml`, `one.manifest.json`, `one.platform-state.json` ve `AGENTS.md`'yi
kodla birlikte yükleyin.

**Platform kuralları.** `AGENTS.md`'nin sonundaki "Platform kuralları" bölümünü `npx one agents sync` yazar; elle
düzenlemeyin. Commit'ten önce `npx one doctor` çalıştırın: uygulamayı bu kurallara karşı tarar, zorunlu bulguda çıkış
kodu 1'dir.

Sıra: `one migrate`, `one dev`, `one build`. `one dev` uygulamayı yerel Portal içinde açar (`http://127.0.0.1:8787/`);
kimlik başlıkları elle yazılmaz, kişi `one dev --as <kişi>` ya da sayfadaki listeyle seçilir. Kişiler, kullanıcı grupları
ve DEXA örnek yanıtları uygulamanın kökündeki `one.dev.json` dosyasındadır (yoksa iki örnek kişi). E-posta ve SMS
gönderilmez, yerel posta kutusuna yazılır (`/__one-dev/posta-kutusu`); mobil bildirim platformun varsayılanı gibi
kapalıdır (`ONE-APP-E-029`; `"channels": { "push": "open" }` ile posta kutusuna yazılır). DEXA ve organizasyon şeması
örnek veridendir; tanımsız DEXA anahtarı platformdaki gibi reddedilir. DEXA anahtarını örnek veriye uydurmayın,
`one_dexa_queries` listesinden alın. Yapay zeka ve arka plan işinin servis çağrıları yerelde yoktur. Komutları
kullanıcıya yazdırmayın, kendiniz çalıştırın.
