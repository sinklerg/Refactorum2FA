# Refactorum2FA

Refactorum2FA — Velocity-плагин для двух-факторной авторизации Minecraft аккаунтов через Telegram-бота. В этом архиве также есть встроенный Paper/Purpur helper для hub, который замораживает игрока во время активного 2FA-запроса.

Плагин рассчитан на схему сети:

```text
Игрок -> Velocity -> hub (/login или /register) -> Telegram 2FA -> towny
```

Основной плагин ставится на **Velocity**. Чтобы блокировать движение, команды, NPC и GUI в `hub` во время ожидания Telegram 2FA, этот же jar дополнительно ставится в `hub/plugins/` как Paper/Purpur helper. На `server` jar ставить не нужно.

Автор: **sinkler**

---

## Что изменено в версии 1.2.6

- Исправлена причина, из-за которой игрок мог кликнуть NPC/GUI и занять очередь до подтверждения 2FA: Paper/Purpur helper теперь блокирует не только движение и команды, но и взаимодействия с NPC/сущностями/блоками, открытие и клики в инвентарях, drag/drop/swap предметов.
- Freeze-сообщение с Velocity на hub теперь дублируется несколько раз, пока 2FA-запрос активен. Это закрывает тайминги, когда первое plugin-message сообщение могло прийти до полной готовности backend-helper.
- Интеграция с ajQueue стала устойчивее к изоляции classloader Velocity: плагин ищет API через classloader самого ajQueue/ajQueuePlus, а не только через текущий classloader.
- Очистка очереди ajQueue теперь использует официальный `QueueManager#clear(AdaptedPlayer)`, а не ручной обход очередей.
- Добавлена блокировка `/move` и slash-команды защищённого сервера, например `/server`, если ajQueue включает slash server aliases.
- Добавлена защита от блокировки очереди ajQueue: linked-игрок без подтверждённой 2FA-сессии больше не может занять очередь через `/queue`, `/joinqueue`, `/joinq`, `/move`, `/server`, `/ajqueue`, `/ajq`, а также slash-команду protected-сервера вроде `/server`.
- Добавлен runtime-hook в ajQueue `PreQueueEvent`: если очередь запускается не командой, а NPC/GUI/API, постановка в очередь тоже отменяется до подтверждения Telegram 2FA.
- 2FA больше не стартует при попытке перейти на `server`: теперь запрос создаётся именно после `/login` или `/l` на `hub` для уже привязанного аккаунта.
- До подтверждения Telegram игрок остаётся замороженным в `hub`; команды блокируются и переходы на `server` запрещаются, включая попытки от ajQueue.
- При `Запретить`, таймауте или ошибке fail-closed игрок отключается, чтобы он не мог двигаться без подтверждения.
- Сообщение `/2fatg` в Minecraft стало удобным для копирования:
  - отдельно копируется команда `/link КОД`;
  - отдельно копируется сам код;
  - отдельно копируется ссылка на Telegram-бота.
- Переделан внешний вид Telegram-сообщений: аккуратные блоки, статусы, иконки, понятные кнопки.
- Добавлено полноценное техническое меню в Telegram для администраторов по Telegram ID.
- Обычные пользователи не видят тех-часть: на `/tech` они получают только `Неизвестная команда`.
- Все пользовательские сообщения и подписи кнопок вынесены в `messages.yml`.
- Сохранён ручной HTTP CONNECT для HTTP-прокси с Basic-авторизацией, который помог обойти ошибку `407 Proxy Authentication Required`.
- После первой привязки Telegram отправляет справку по Minecraft/Telegram-командам и предупреждает, что при блокировке через Telegram нужно обращаться к тех.админу.
- Во время активного 2FA-запроса все команды на Velocity блокируются.
- Добавлен Paper/Purpur helper в этом же jar: при установке jar на `hub` он блокирует движение, команды, NPC/GUI-взаимодействия и инвентарь игрока до завершения активного 2FA-запроса.
- Если в Telegram нажать «Запретить» на запросе входа, игрок отключается от сервера.

---

## Возможности

- Привязка Minecraft аккаунта к Telegram через одноразовый код.
- Предложение подключить 2FA только на `server`, а не на `hub`.
- Telegram 2FA запускается сразу после команды `/login` или `/l` на `hub`, если аккаунт привязан.
- Пока Telegram 2FA не подтверждена, игрок заморожен на `hub`, все команды блокируются, а переходы на `server` запрещаются на уровне Velocity.
- ajQueue не сможет занять игроком очередь до подтверждения: блокируются queue-команды и дополнительно отменяется `PreQueueEvent`, если очередь создаётся через NPC/GUI/API.
- ajQueue не сможет протолкнуть игрока на `towny` до подтверждения: любой `ServerPreConnect` на protected-server отклоняется, пока нет авторизованной 2FA-сессии.
- Автоматический перевод после `Разрешить` отключён: подтверждение только открывает доступ в текущей прокси-сессии.
- Telegram-меню игрока:
  - статус аккаунта;
  - отвязка;
  - кик своего аккаунта;
  - блокировка.
- Самостоятельная разблокировка через Telegram отключена: разблокировать может только тех-администратор.
- Тех-меню Telegram:
  - список всех привязанных аккаунтов;
  - карточка аккаунта;
  - UUID, Telegram ID, ник Telegram;
  - онлайн-статус;
  - количество отыгранного времени;
  - последний вход;
  - IP последнего входа;
  - последний backend-сервер;
  - блокировка и разблокировка;
  - кик;
  - удаление привязки;
  - reload конфигов.
- Поддержка HTTP/SOCKS прокси для Telegram API.
- YAML-файлы: `config.yml`, `messages.yml`, `data.yml`.

---

## Установка

1. Соберите проект:

```bash
./gradlew build
```

Если Gradle установлен отдельно:

```bash
gradle build
```

2. Скопируйте jar:

```text
build/libs/Refactorum2FA-1.2.6.jar
```

в папку Velocity:

```text
velocity/plugins/
```

3. Запустите Velocity один раз, чтобы создалась папка:

```text
velocity/plugins/refactorum2fa/
```

4. Остановите Velocity.
5. Настройте:

```text
velocity/plugins/refactorum2fa/config.yml
velocity/plugins/refactorum2fa/messages.yml
```

6. Запустите Velocity заново.

---

## Куда ставить jar

Обязательно:

```text
velocity/plugins/Refactorum2FA-1.2.6.jar
```

Для блокировки движения и команд на hub во время активного 2FA-запроса дополнительно поставьте этот же jar на Paper/Purpur hub:

```text
hub/plugins/Refactorum2FA-1.2.6.jar
```

На `server` jar ставить не нужно.

Один и тот же jar содержит две части:

```text
Velocity: основной Telegram 2FA gate.
Paper/Purpur hub: helper, который слушает канал refactorum2fa:freeze и замораживает игрока, блокируя движение, команды, NPC/GUI и инвентарь.
```

---

## Настройка под сеть velocity / hub / towny

В `velocity.toml` имена серверов должны совпадать с `config.yml` плагина.

Пример:

```toml
[servers]
hub = "127.0.0.1:25566"
towny = "127.0.0.1:25567"

try = [
  "hub"
]
```

В `config.yml`:

```yaml
network:
  auth-server: "hub"
  target-server: "server"
  protected-servers:
    - "towny"
  gate-only-from-auth-server: true
  redirect-initial-protected-to-auth-server: false
  auto-connect-after-approval: false

join-suggestion:
  enabled: true
  delay-seconds: 3
  show-only-unlinked: true
  servers:
    - "server"
```

Логика работы:

```text
1. Игрок заходит на Velocity.
2. Velocity отправляет игрока на hub.
3. Игрок вводит /login или /l на hub.
4. Auth-плагин пытается отправить игрока на server.
5. Если 2FA не подключена — игрок проходит на server.
6. Если 2FA подключена — переход отменяется, игрок остаётся на hub.
7. Бот отправляет запрос Разрешить / Запретить.
8. После Разрешить доступ к towny открывается в текущей сессии, но плагин никуда не переносит игрока автоматически.
```

Если auth-плагин не переводит игрока на `server` автоматически, игрок может использовать на `hub`:

```text
/2fa
```

---

## Telegram-бот

В `config.yml` нужно указать токен от BotFather:

```yaml
telegram:
  bot-token: "PUT_TELEGRAM_BOT_TOKEN_HERE"
  bot-username: "RefactorumBot"
```

В `bot-token` указывается только сам токен. Слово `bot` перед ним добавлять не нужно.

---

## Прокси для Telegram API

SOCKS:

```yaml
telegram:
  proxy:
    enabled: true
    type: "SOCKS"
    host: "127.0.0.1"
    port: 1080
    username: ""
    password: ""
```

HTTP с логином и паролем:

```yaml
telegram:
  proxy:
    enabled: true
    type: "HTTP"
    host: "185.168.251.46"
    port: 8000
    username: "login"
    password: "password"
```

Если в консоли появляется:

```text
407 Proxy Authentication Required
```

значит прокси требует авторизацию, указан неверный тип прокси или выбран не тот порт. В этой версии HTTP-прокси с авторизацией обрабатывается вручную через HTTPS CONNECT, поэтому корректный HTTP-прокси должен работать так же, как проверка через `curl -x`.

---

## Команды Minecraft

| Команда | Где используется | Описание |
|---|---|---|
| `/2fatg` | Обычно на `towny` | Создаёт код привязки Telegram. |
| `/2fatg reload` | Velocity/игра | Перезагружает `config.yml` и `messages.yml`. |
| `/2fa` | На `hub` после `/login` | Отправляет запрос подтверждения. После одобрения открывает доступ, но не переносит на `target-server`. |
| `/2faverify` | На `hub` | Алиас `/2fa`. |

---

## Permissions

| Permission | Описание |
|---|---|
| `refactorum2fa.reload` | Доступ к `/2fatg reload`. |
| `refactorum2fa.bypass` | Обход 2FA, если `auth.allow-permission-bypass: true`. По умолчанию выключено. |

---

## Команды Telegram для игрока

После привязки игрок получает личное меню с кнопками. Также доступны команды:

| Команда | Описание |
|---|---|
| `/status` | Показывает онлайн, отыгранное время, последний вход, IP, последний сервер и блокировку. |
| `/unlink` | Удаляет привязку 2FA. |
| `/kick` | Кикает Minecraft аккаунт с сервера. |
| `/block` | Блокирует переходы на защищённый сервер. |
| `/unblock` | Самостоятельная разблокировка отключена. Если аккаунт заблокирован через Telegram, нужно обращаться к тех.админу. |

---

## Техническое меню Telegram

Тех-администраторы добавляются в `config.yml` по Telegram ID:

```yaml
telegram:
  tech-admin-ids:
    - 123456789
```

Обычный пользователь при вводе `/tech` получит:

```text
Неизвестная команда
```

Администратору доступны:

| Команда | Описание |
|---|---|
| `/tech` | Открывает техническое меню с inline-кнопками. |
| `/tech list [page]` | Показывает список привязанных аккаунтов. |
| `/tech info <player\|uuid\|telegramId>` | Показывает карточку аккаунта. |
| `/tech block <player\|uuid\|telegramId>` | Блокирует аккаунт. |
| `/tech unblock <player\|uuid\|telegramId>` | Разблокирует аккаунт. |
| `/tech kick <player\|uuid\|telegramId>` | Кикает аккаунт, если он онлайн. |
| `/tech unlink <player\|uuid\|telegramId>` | Удаляет привязку. |
| `/tech reload` | Перезагружает `config.yml` и `messages.yml`. |

---

## Пример config.yml

```yaml
telegram:
  bot-token: "PUT_TELEGRAM_BOT_TOKEN_HERE"
  bot-username: "RefactorumBot"
  tech-admin-ids:
    - 123456789
  api:
    base-url: "https://api.telegram.org"
    connect-timeout-ms: 10000
    read-timeout-ms: 35000
    polling-timeout-seconds: 25
    retry-delay-ms: 3000
  proxy:
    enabled: false
    type: "SOCKS"
    host: "127.0.0.1"
    port: 1080
    username: ""
    password: ""

network:
  auth-server: "hub"
  target-server: "server"
  protected-servers:
    - "server"
  gate-only-from-auth-server: true
  redirect-initial-protected-to-auth-server: false
  auto-connect-after-approval: false

auth:
  login-timeout-seconds: 45
  allow-permission-bypass: false
  fail-closed: true
  link-code:
    length: 6
    expires-seconds: 300
    alphabet: "ABCDEFGHJKLMNPQRSTUVWXYZ23456789"
  allow-same-telegram-for-multiple-accounts: false
  block-commands-while-pending: true
  block-all-commands-while-pending: true
  allowed-commands-while-pending:
    - "login"
    - "l"
    - "register"
    - "reg"
    - "2fa"
    - "2fatg"
  block-queue-commands-until-2fa: true
  queue-commands:
    - "queue"
    - "joinqueue"
    - "joinq"
    - "move"
    - "server"
    - "ajqueue"
    - "ajq"
  block-online-player:
    kick: true
    delay-seconds: 2

join-suggestion:
  enabled: true
  delay-seconds: 3
  show-only-unlinked: true
  servers:
    - "server"

storage:
  file: "data.yml"
  autosave-after-change: true

permissions:
  reload: "refactorum2fa.reload"
  bypass: "refactorum2fa.bypass"
```

---

## messages.yml

Все пользовательские сообщения вынесены в `messages.yml`. Там же находятся:

- сообщения Minecraft;
- сообщения Telegram;
- подписи кнопок Telegram;
- иконки статусов;
- слова для форматирования статуса: `да`, `нет`, `онлайн`, `не онлайн`, `никогда`, `неизвестно`.

После изменения `messages.yml` можно выполнить:

```text
/2fatg reload
```

или в Telegram, если вы тех-администратор:

```text
/tech reload
```

---

## Советы по настройке

- Backend-порты `hub` и `server` лучше закрыть firewall’ом от прямого входа игроков. Игроки должны заходить только через Velocity.
- В `protected-servers` указывайте реальные имена из `velocity.toml`, а не IP и не порт.
- На Paper/Purpur ставьте jar только на `hub`, если нужна заморозка движения/NPC/GUI во время 2FA. На `server` helper не нужен.
- Если стоит ajQueue, дополнительная интеграция не нужна: плагин блокирует queue-команды и дополнительно цепляется к `PreQueueEvent`, поэтому игрок без подтверждения 2FA не займёт место в очереди.
- После замены jar лучше полностью перезапускать Velocity, а не делать hot-reload.
- Токен Telegram-бота не публикуйте. Если токен был показан кому-то, перевыпустите его через BotFather.

---

## Сборка

Требования:

- Java 17+
- Gradle 9.3+ или запуск через приложенный ./gradlew

Команда:

```bash
./gradlew build
```

Итоговый jar:

```text
build/libs/Refactorum2FA-1.2.6.jar
```

---

## Структура проекта

```text
src/main/java/ru/refactorum/refactorum2fa/
├── Refactorum2FAPlugin.java
├── command/
│   ├── TwoFaAuthCommand.java
│   └── TwoFaCommand.java
├── config/
├── listener/
│   ├── PlayerActivityListener.java
│   └── ServerGateListener.java
├── model/
├── service/
├── telegram/
├── paper/
│   └── Refactorum2FAHubPlugin.java
└── util/

src/main/resources/
├── config.yml
├── messages.yml
└── plugin.yml
```

---

## Содержимое архива

В ZIP лежит готовый Gradle-проект:

```text
src/
build.gradle
settings.gradle
gradle.properties
gradlew
gradlew.bat
plugin.yml
config.yml
messages.yml
README.md
.gitignore
```
