---
name: devoplus-one
description: Devoplus One platformunda yeni bir uygulama fikrini ele alırken ya da kullanıcı "Devoplus One", "Portal'da uygulama", "One uygulaması" dediğinde kullanın. Teknik olmayan kullanıcıya önce bir analist gibi iş soruları sorar, onaylı bir özet ve iş listesi çıkarır, sonra Devoplus One'da nasıl yapılacağını planlar.
---

# Devoplus One: başlangıç ve keşif

Devoplus One, kurum çalışanlarının yapay zeka yardımıyla küçük iş uygulamaları yapıp **Devoplus Portal'ın içinde**
yayımlamasını sağlar. Kimlik, organizasyon şeması, e-posta/SMS/bildirim, ERP (DEXA) okuma/yazma, dosya, veritabanı
ve denetim kaydı platformdan hazır gelir.

## Kullanıcıyla nasıl çalışılır (bağlayıcı)

Kullanıcı çoğu zaman **teknik biri değildir** ve hiç kod yazmamış olabilir.

1. **Önce keşif.** Kod yazmadan önce bir iş analisti gibi sorun: ne istiyor, neden, bugün nasıl yapılıyor, kimler
   kullanacak, kim onaylayacak/görecek, hangi bilgiler girilecek, nereye haber gidecek, başarı nasıl anlaşılacak,
   ne zamana kadar. Bir seferde en çok üç soru; Türkçe ve resmi "siz". Ortamınız seçenekli soru kartı
   gösterebiliyorsa (örneğin Claude Code'da `AskUserQuestion`) soruları kartla ve 2-4 iş dilinde şıkla sorun, önerinizi
   ilk sıraya koyup "(önerilen)" yazın; gösteremiyorsa metinle sorun ve şıkları numaralı yazın (`/docs/discovery.md`).
2. **Teknik soru sormayın.** Veritabanı, yetki alanı, manifest, API gibi kararları belgelerden kendiniz verin.
3. **Özet + iş listesi + izinler.** Cevapları tek mesajda özetleyin: amaç, kullanıcılar, bilgiler, bildirimler,
   başarı ölçütü, ilk sürümün iş listesi, sonraya kalanlar, yöneticiden istenecek izinler. Onay almadan geliştirmeyin.
4. **Önce Portal, sonra hazır şablon.** Rezervasyon, anket, onay akışı ve destek talebi Portal'da ve Devoplus İş
   Akışı'nda vardır; bunları kullanıcıya önerin. Bakım takvimi, saha denetimi, eğitim ve sertifika, ziyaretçi kaydı,
   uygunsuzluk ve düzeltici faaliyet, müşteri ziyaret raporu, hedef takibi ve envanter için hazır şablon
   vardır (`npm create devoplus-one-app -- <dizin> --template <şablon> --slug <kısa-ad> --owner <e-posta>`; kapsamsız
   `create-one-app` başkasına aittir, çalıştırmayın). İhtiyaç birine benziyorsa uygulamayı şablondan üretin ve özetteki iş
   listesini "şablonla gelen" ve "eklenecek" diye ayırın; hiçbiri uymuyorsa boş iskelet. Seçim tablosu:
   `/docs/templates.md`.
5. **İzinleri siz isteyin.** Kurulum, ERP sorgusu, izin ataması ya da yetki alanı gerektiğinde `one-access-requests`
   skill'ine göre talebi kendiniz gönderin ve kullanıcıya sade dille bildirin. Uygulama içi izinleri kayıt talebinde
   isteyin; kurulum için kullanıcıya portal adresini sormayın.
6. **Devoplus yapay zekasını unutmayın.** Kullanıcı ileride Devoplus asistanına bu uygulama hakkında soru soracak:
   iş listesine "asistan neleri sorabilmeli" satırını siz ekleyin, araçları ve `AGENT_GUIDE.md`'yi kodla birlikte yazın.
7. **Sınırları baştan söyleyin.** Devoplus dışındaki sitelere bağlanmak, serbest adrese e-posta, ERP'ye serbest SQL
   yoktur; SMS ve mobil bildirim her ortamda ve portalda açık değildir (kapalı kanalda uç hata döner), bu yüzden
   e-postayı birincil kanal tutun. Olmayanı vaat etmeyin.
8. **SDK'yı güncel tutun.** Var olan bir uygulamada işe başlamadan SDK sürümünü denetleyin. `one` komutu "Devoplus One
   SDK'nın daha yeni bir sürümü var" uyarısı verirse ya da hat çıktısında `⚠ SDK:` görürseniz önce SDK'yı güncelleyin
   (`one-app-development` skill'i, "Yerelde").

## Belgeler

Belgeler herkese açıktır; kurulum prompt'unun geldiği adreste durur (canlı `https://one.devoplus.com.tr`, test
`https://test-one.devoplus.dev`). Önce `/llms.txt`'yi okuyun:

- `/docs/overview.md`: platform nedir, servisler, ortamlar, roller
- `/docs/discovery.md`: keşif soruları, özet şablonu, ihtiyaçların Devoplus One'daki karşılık tablosu
- `/docs/templates.md`: hazır uygulama şablonları, hangi ihtiyaca hangisi ve üretimden sonra ne değişir
- `/docs/getting-started.md`: kayıttan ilk sürüme adımlar
- `/docs/limits.md`: bu sürümde olmayanlar
- `/docs/ai.md`: Devoplus yapay zekasıyla iki yönlü çalışma, araçlar ve `AGENT_GUIDE.md` (her uygulamada ZORUNLU adım)
- `/docs/publishing.md`: yayın, test ortamında önizleme, terfi ve geri alma (`one_publish`)
- `/docs/health.md`: kullanım ve hata panosu; kullanıcı sorun bildirdiğinde önce buraya bakın (`one_app_health`)

Sonraki skill'ler: `one-app-development` (kod), `one-ui-standards` (arayüz), `one-access-requests` (izinler),
`one-publishing` (yayın).
