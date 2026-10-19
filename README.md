# Devoplus One: yapay zeka ajanları için skill paketi

Devoplus One uygulamalarını geliştiren yapay zeka ajanlarına (Claude Code, Codex, Cursor ve diğerleri) platformun
kurallarını öğreten skill'ler. Kurulum için Devoplus One kurulum sayfasındaki prompt'u ajanınıza verin; ajan bu paketi
kendisi kurar.

| Skill | Ne zaman |
|---|---|
| `devoplus-one` | Yeni bir uygulama fikri: önce keşif görüşmesi, özet ve iş listesi |
| `one-app-development` | Uygulama kodu (manifest, SDK, veritabanı, servisler) |
| `one-ui-standards` | Sayfa, menü, bildirim, görünüm |
| `one-data-and-exports` | Kayıt tutan ekranlar (veri modeli, hazır kayıt uçları) ve Excel/CSV/yazdırma |
| `one-access-requests` | Kayıt, kurulum, ERP sorgusu, izin ataması, yetki alanı talebi |
| `one-publishing` | Yayın, GitLab hattı ve kapıları |

## Elle kurulum

```bash
# Claude Code
claude plugin marketplace add <bu depo>
claude plugin install devoplus-one@devoplus-one

# Diğer ajanlar
npx -y skills add <bu depo> --skill '*' --yes --global
```

Bu depo `devoplus/devoplus-one` deposundaki `agent-kit/` klasöründen otomatik eşitlenir; değişiklik orada yapılır.
