# cli_tool_catalog — ekleme standardı

Bu depoya eklenen HER araç bu standarda uyar. Uyum **otomatik denetlenir**:

```console
team-agent catalog lint --dir .        # ihlal varsa çıkış kodu 1; --json makine okunur çıktı verir
```

Lint kırmızıysa girdi yayımlanmaz. Tek doğruluk kaynağı MindAlert ürününün kendi katalog ayrıştırıcısıdır; kurallar burada ve orada AYNIDIR.
(Karar kaydı: MindAlert deposunda `docs/decisions/D89.md`.)

## 1. Bir araç bir CLI'dir
- Tek çalıştırılabilir; argv ile sürülür; **etkileşimsizdir** (soru sormaz, TTY istemez).
- Veri üreten komutlar stdout'a **JSON ya da JSONL** basar; hata stderr'e JSON olarak gider; çıkış kodları belgelidir.
- **Secret asla argv'den verilmez**; yalnız ortam değişkeninden (`credential_env_vars` ADLARI). Lint: `secret_in_argv`.
- CLI olmayan bir şey (kütüphane, servis, GUI) önce CLI'ye dönüştürülür. Kendi araçlarımız `mindalert-cli-tools` deposunda stdlib-only zipapp kalıbıyla yazılır (tek JSON stdout, çıkış kodu tablosu).

## 2. Lisans
- İzin verilenler: MIT, Apache-2.0, BSD-2-Clause, BSD-3-Clause, ISC, PostgreSQL. Diğerleri → `license_not_allowed`.
- GPL/AGPL/LGPL gibi lisanslar YALNIZ `license_exception: {decision: "D<n>[.<m>]", isolation: separate_process}` ile girer (sahip onayı kararı + araç ayrı süreç olarak çalışır; kütüphane olarak bağlanmaz). Biçim bozuksa `license_exception_invalid`.

## 3. Kurulum
- `install_method: archive|pip`: `source.version` **sabit** olmalı (`latest`, `main`, `master`, `*` yasak) ve `source.sha256` 64 hex olmalı → aksi halde `install_not_pinned`. Arşiv `source.asset` `https` olmalı → `asset_not_https`.
- `install_method: system`: sürüm/sha istenmez ama `detect` ve `verify_command` zorunludur.
- Kullanıcı düzeyinde kurulur (root gerekmez), etkileşimsiz; `detect` aracı müşteri sunucusunda tanır; `verify_command` sağlıklıysa 0 ile çıkar ve ilk sözcüğü aracın kendi ikilisidir.
- Girdi ürün ayrıştırıcısıyla geçersizse `entry_invalid` (neden `detail`'de).

## 4. İzleme HAZIR gelir
Her girdi `monitoring_class` taşır (yoksa `monitoring_class_missing`):
- **A** (aracın/hedef sistemin günlüğü var): `log_source_hint` `file` ya da `syslog_identifier` (+ isteğe bağlı seviye kuralları, `critical_rules`, `discover`/`{yer_tutucu}`: tanım MÜŞTERİYE özel değil GENEL olur; müşteriye özel değerler kurulumda keşfedilir).
- **B** (günlüğü yok, sağlığı yoklanır): `monitoring.probe` (komut + aralık); araca özel sorun toplama için `monitoring.collectors[]` (JSON/JSONL, `sample`, `fingerprint_fields`, `verified`).
- **C** (izlenecek günlük/servis yok): `log_source_hint: {kind: none, reason: "..."}`; gerekçe ≥ 40 karakter.
`verified: true` yalnız aracın GERÇEK çıktısıyla doğrulanmış toplayıcılara yazılır; doğrulanmamışlar `verified: false` (varsayılan) kalır.

## 5. Kimlik ve açıklama
- `credential_env_vars` yalnız ortam değişkeni ADLARIdır (`^[A-Z][A-Z0-9_]*$`).
- `description` ≥ 20 karakter (`description_too_short`); `capabilities_hint` boş olamaz (`capabilities_hint_empty`).

## 6. Ekleme kontrol listesi
1. Araç bir CLI mi? Değilse dönüştür (bölüm 1).
2. Lisans izinli mi / istisna kararı var mı? (bölüm 2)
3. Sürüm ve sha256 sabitlendi mi? (bölüm 3)
4. `monitoring_class` ve gereken izleme tanımı hazır mı? (bölüm 4)
5. `team-agent catalog lint --dir .` temiz mi?
6. (Varsa) toplayıcı tanımı gerçek çıktıyla doğrulandı mı? (`verified`)

## Kural adları (lint çıktısı)
`entry_invalid`, `license_not_allowed`, `license_exception_invalid`, `install_not_pinned`, `asset_not_https`, `description_too_short`, `capabilities_hint_empty`, `monitoring_class_missing`, `secret_in_argv`.
