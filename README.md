# Lirix rules

Каталог правил сайтов для приложения [Lirix](https://github.com/Asm-o-Dan/lirix): где на сайте искать песню и какой блок страницы — текст или аккорды. Приложение скачивает только `index.json`, проверяет его подпись и ставит правило лишь после того, как оно сработало на песнях из библиотеки пользователя.

*English below.*

## Как устроено

| Файл | Что это |
|---|---|
| `rules/<домен>/<LYRICS\|CHORDS>.json` | одно правило: селекторы, шаблон поиска, версия, проверочные песни |
| `index.json` | список правил с SHA-256 каждого файла |
| `index.json.sig` | подпись `index.json` (ECDSA P-256, SHA-256, base64 DER) |
| `keys/catalog-keys.txt` | открытые ключи, которыми может быть подписан каталог; те же ключи зашиты в приложение |
| `tools/lirix-rules-cli.jar` | проверка и подпись каталога (исходники — `shared/src/desktopMain/kotlin/com/lirix/app/rules/` в репозитории Lirix) |

### Безопасность

- Индекс подписывает владелец **на своём компьютере**. Закрытого ключа нет ни в репозитории, ни в секретах GitHub, поэтому взлом CI не даёт подписать каталог.
- Приложение принимает только подписанный индекс и файл правила, у которого SHA-256 совпадает с индексом.
- Проверка запрещает: поиск на другом сайте, IP-адреса и `http`, собственные заголовки (cookie, токены), опасные селекторы.
- Правило из каталога включается только после успешной проверки на песнях пользователя. Выключенное пользователем правило не включается обновлением.

## Добавить или исправить правило

1. Создайте `rules/<домен>/<LYRICS|CHORDS>.json` по образцу `rules/amdm.ru/CHORDS.json`.
   - `version` — с 1. При любом изменении файла версию нужно увеличить.
   - `checks` — не меньше двух разных песен, которые есть на сайте.
   - `language` — языки песен на сайте (`ru`, `en`, …).
2. Проверьте локально (нужна Java 21):
   ```
   java -jar tools/lirix-rules-cli.jar validate .
   ```
3. Откройте pull request. Менять можно только файлы в `rules/`; `index.json` не трогайте — его пересоберёт и подпишет владелец.

Не присылайте правила для сайтов, которые запрещают автоматический доступ, требуют входа или защищены CAPTCHA.

## Для владельца: выпуск каталога

```
java -jar tools/lirix-rules-cli.jar validate .
java -jar tools/lirix-rules-cli.jar index . <время в миллисекундах>
java -jar tools/lirix-rules-cli.jar sign . ~/lirix-keys/catalog-primary.pk8
java -jar tools/lirix-rules-cli.jar verify . keys/catalog-keys.txt
```

Затем коммит `index.json` и `index.json.sig`. Запасной ключ хранится отдельно и нужен только для смены основного.

---

## English

A catalogue of site rules for Lirix: where a site's search is and which block of a song page holds the lyrics or chords. The app downloads `index.json`, checks its signature, and installs a rule only after it worked on songs from the user's own library.

- The index is signed by the owner on their own machine; no private key exists in this repository or in GitHub secrets.
- To contribute, add `rules/<domain>/<LYRICS|CHORDS>.json` (see `rules/amdm.ru/CHORDS.json`), give it `version` ≥ 1 (raise it on every change), at least two `checks` songs and its `language`, run `java -jar tools/lirix-rules-cli.jar validate .`, and open a pull request that changes only files under `rules/`.
- Do not submit rules for sites that forbid automated access, need a login or use CAPTCHA.
