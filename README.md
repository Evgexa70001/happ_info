# Подключение к Happ и VPN (ROSCOM)

Краткая инструкция: подписка от автора, автоматическая маршрутизация + дополнительно Discord и режим **TUN**.

---

## 1. Подписка от автора — обязательный первый шаг

1. Получите у автора **ссылку на подписку** (subscription URL).
2. Выберите сервер и подключитесь.

---

## 2. Маршрутизация: добавляется автоматически

**Важно:** после подключения подписки **правила маршрутизации подставляются сами** — отдельно вручную «собирать» базовую маршрутизацию под Happ обычно **не нужно**. Она приходит в составе конфигурации, которую вы загружаете вместе с подпиской.

- Вручную что-то менять в маршрутах имеет смысл только если вы **осознанно** донастраиваете клиент под себя (и понимаете последствия).
- Если автор позже обновит подписку, набор правил может обновиться — снова обновите подписку в клиенте.

### Что подставляется по умолчанию: пресет «RoscomVPN»

**Название в JSON:** `RoscomVPN` (срабатывает через механизм Happ `routing/onadd` — отдельную ссылку в документе не приводим).

**Суть:** включён **глобальный прокси** (`GlobalProxy`), списки правил грузятся **чанками** (`UseChunkFiles`). DNS разведены: «удалённый» резолв через **8.8.8.8** (DoH), «домашний» — через **77.88.8.8** (DoH, типичный сценарий для РФ). Для личных кабинетов **ФНС** зафиксированы записи **DnsHosts** на `lkfl2.nalog.ru` и `lknpd.nalog.ru`. Файлы **geoip** и **geosite** берутся из **своих сборок ROSCOM VPN** на CDN (не из стандартного набора Loyalsoldier). Стратегия доменов: `IPIfNonMatch`, FakeDNS выключен.

**Разбивка трафика** (порядок обработки `block-proxy-direct`):

- **Напрямую (direct):** частные сети (`geoip:private`, `geoip:direct`); по `geosite` — в том числе **private**, **category-ru**, **whitelist**, экосистемы **Microsoft**, **Apple**, **Google Play**, игровые сервисы и лаунчеры из пресета (**Epic Games**, **Riot**, **Steam**, **Origin**, **Escape from Tarkov** и др.), **Twitch** (основной контент), **Pinterest**, **Faceit** и др. из списка — то есть типичный «российский и игровой» и локальный трафик остаётся вне VPN-туннеля там, где это заложено правилами.
- **Через прокси:** **GitHub**, реклама **Twitch** (`twitch-ads`), **YouTube**, **Telegram**.
- **Блокировка:** категории **win-spy**, **torrent**, реклама (**category-ads**).

Итого: базовый профиль **RoscomVPN** балансирует «своё» и игры в direct, часть зарубежных сервисов — через прокси, шпионское/торренты/лишняя реклама — в block.

---

## 3. Дополнительное правило маршрутизации для Discord (Happ)

По желанию можно добавить **готовый пресет маршрутизации** через deep link клиента **Happ** (схема `happ://`). Подписка от автора по-прежнему задаёт базовую конфигурацию; это правило — **отдельный профиль** с тонкой разбивкой трафика.

**Название пресета в JSON:** «Для Дискорда без пинга в играх».

**Суть:** в **прокси** уходят категории `geosite` для **Discord**, OpenAI, YouTube и Telegram; **напрямую (direct)** — частные сети, трафик по **geoip:ru**, а также ряд `geosite` (в т.ч. VK, зона `.ru`, детект по гео и т.д. по списку в правиле). Порядок обработки: `block-proxy-direct`. Списки `geoip`/`geosite` подтягиваются с репозитория [Loyalsoldier/v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat) (как указано в пресете).

**Как добавить:** откройте ссылку на устройстве, где установлен Happ (или вставьте её в поддерживаемый импорт), подтвердите добавление правила маршрутизации.

```
happ://routing/add/ewogICAgIkJsb2NrSXAiOiBbCiAgICBdLAogICAgIkJsb2NrU2l0ZXMiOiBbCiAgICBdLAogICAgIkRpcmVjdElwIjogWwogICAgICAgICIxMC4wLjAuMC84IiwKICAgICAgICAiMTcyLjE2LjAuMC8xMiIsCiAgICAgICAgIjE5Mi4xNjguMC4wLzE2IiwKICAgICAgICAiMTY5LjI1NC4wLjAvMTYiLAogICAgICAgICIyMjQuMC4wLjAvNCIsCiAgICAgICAgIjI1NS4yNTUuMjU1LjI1NSIsCiAgICAgICAgImdlb2lwOnJ1IgogICAgXSwKICAgICJEaXJlY3RTaXRlcyI6IFsKICAgICAgICAiZ2Vvc2l0ZTpDQVRFR09SWS1JUC1HRU8tREVURUNUIiwKICAgICAgICAiZ2Vvc2l0ZTp0bGQtcnUiLAogICAgICAgICJnZW9zaXRlOnZrIiwKICAgICAgICAiZ2Vvc2l0ZTp0ZWxlZ3JhbSIKICAgIF0sCiAgICAiRG5zSG9zdHMiOiB7CiAgICAgICAgImNsb3VkZmxhcmUtZG5zLmNvbSI6ICIxLjEuMS4xIiwKICAgICAgICAiZG5zLmdvb2dsZSI6ICI4LjguOC44IgogICAgfSwKICAgICJEb21haW5TdHJhdGVneSI6ICJJUElmTm9uTWF0Y2giLAogICAgIkRvbWVzdGljRE5TRG9tYWluIjogImh0dHBzOi8vZG5zLmdvb2dsZS9kbnMtcXVlcnkiLAogICAgIkRvbWVzdGljRE5TSVAiOiAiOC44LjguOCIsCiAgICAiRG9tZXN0aWNETlNUeXBlIjogIkRvSCIsCiAgICAiRmFrZUROUyI6ICJmYWxzZSIsCiAgICAiR2VvaXB1cmwiOiAiaHR0cHM6Ly9naXRodWIuY29tL0xveWFsc29sZGllci92MnJheS1ydWxlcy1kYXQvcmVsZWFzZXMvbGF0ZXN0L2Rvd25sb2FkL2dlb2lwLmRhdCIsCiAgICAiR2Vvc2l0ZXVybCI6ICJodHRwczovL2dpdGh1Yi5jb20vTG95YWxzb2xkaWVyL3YycmF5LXJ1bGVzLWRhdC9yZWxlYXNlcy9sYXRlc3QvZG93bmxvYWQvZ2Vvc2l0ZS5kYXQiLAogICAgIkdsb2JhbFByb3h5IjogImZhbHNlIiwKICAgICJMYXN0VXBkYXRlZCI6IDE3NzUwMzc0MzgsCiAgICAiTmFtZSI6ICLQlNC70Y8g0JTQuNGB0LrQvtGA0LTQsCDQsdC10Lcg0L/QuNC90LPQsCDQsiDQuNCz0YDQsNGFIiwKICAgICJQcm94eUlwIjogWwogICAgXSwKICAgICJQcm94eVNpdGVzIjogWwogICAgICAgICJnZW9zaXRlOkRJU0NPUkQiLAogICAgICAgICJnZW9zaXRlOk9QRU5BSSIsCiAgICAgICAgImdlb3NpdGU6WU9VVFVCRSIsCiAgICAgICAgImdlb3NpdGU6VEVMRUdSQU0iCiAgICBdLAogICAgIlJlbW90ZUROU0RvbWFpbiI6ICJodHRwczovL2Nsb3VkZmxhcmUtZG5zLmNvbS9kbnMtcXVlcnkiLAogICAgIlJlbW90ZUROU0lQIjogIjEuMS4xLjEiLAogICAgIlJlbW90ZUROU1R5cGUiOiAiRG9IIiwKICAgICJSb3V0ZU9yZGVyIjogImJsb2NrLXByb3h5LWRpcmVjdCIKfQo=
```

Если клиент не реагирует на `happ://`, обновите Happ или добавьте правило вручную, импортировав тот же JSON (декодируется из части ссылки после `/add/`).

---

## 4. Discord (общие советы)

Discord использует много адресов и CDN: опирайтесь на **правила из подписки**, при необходимости — на **пресет из §3**, и на режим **TUN** (§5).

Если Discord ведёт себя нестабильно (часть соединений мимо туннеля), чаще всего помогает режим **TUN**.

---

## 5. Рекомендация: для Discord лучше TUN-режим

**TUN** перехватывает трафик на уровне IP, что лучше подходит для сервисов с динамическими адресами и несколькими соединениями.

**Рекомендация:** включите маршрутизацию "Для Дискорда без пинга в играх" режим **TUN** (если в настройках есть выбор TUN / TAP / другое). Так проще согласовать работу с маршрутизацией из подписки и с пресетом Happ (§3), в том числе для Discord.

После смены режима может понадобиться переподключение VPN или перезапуск клиента; на Windows иногда нужны права администратора.

---

## 6. Краткий чеклист

| Шаг | Действие |
|-----|----------|
| 1 | Добавить **подписку** от автора |
| 2 | Подключиться к серверу — **маршрутизация из подписки применится автоматически** |
| 3 | По желанию — пресет **Happ** для Discord: ссылка `happ://` из §3 |
| 4 | Для Discord — по возможности режим **TUN** |
| 5 | При проблемах — обновить подписку, переподключить VPN |

---

## 7. Если что-то не работает

- Подписка **просрочена или ссылка недействительна** — запросите новую у автора.
- **Обновите подписку** в клиенте и переподключитесь.
- Проверьте режим **TUN** и права администратора в Windows при необходимости.

---
