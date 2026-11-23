# Вход через Microsoft: как подключить

Лаунчер входит в Microsoft «по коду»: игрок открывает microsoft.com/link, вводит короткий код
и входит в свою учётную запись. Для этого нужно **одно приложение Azure** на весь лаунчер —
его Client ID вшивается в `launcher/__init__.py` (`MS_CLIENT_ID`). Это публичный идентификатор,
не пароль: секрет (client secret) не нужен и не используется.

## 1. Создать приложение в Azure (≈5 минут, бесплатно)
1. Открой <https://portal.azure.com> и войди учётной записью Microsoft (подойдёт обычная личная).
   Если портал просит создать «каталог/тенант» — согласись, это бесплатно.
2. Найди **Microsoft Entra ID** → **App registrations** → **New registration**.
3. Заполни:
   - **Name**: `Tikhon Launcher`
   - **Supported account types**: *Personal Microsoft accounts only*
   - **Redirect URI**: платформа *Public client/native (mobile & desktop)*,
     адрес `https://login.microsoftonline.com/common/oauth2/nativeclient`
4. Нажми **Register**. На странице приложения скопируй **Application (client) ID** — это
   строка вида `1a2b3c4d-....`.
5. Слева **Authentication** → внизу **Allow public client flows** → **Yes** → **Save**.
   (Без этого вход по коду не работает.)

## 2. Подать заявку Mojang на доступ к Minecraft
С 2023 года Mojang пускает новые приложения к Minecraft только после проверки.
1. Открой форму <https://aka.ms/mce-reviewappid>.
2. Укажи Client ID из шага 1, название `Tikhon`, кратко опиши: «Сторонний лаунчер Minecraft:
   Java Edition, вход по коду устройства, токены хранятся только на компьютере игрока».
   Ссылка на проект: <https://github.com/vdvfdr-lab/tikhon>.
3. Ответ приходит на почту — обычно от нескольких дней до пары недель.

Пока заявка не одобрена, вход дойдёт до Microsoft, но Minecraft ответит отказом — лаунчер
покажет понятное сообщение «заявка ещё не одобрена». Офлайн и Ely.by при этом работают как обычно.

## 3. Вшить Client ID
Впиши ID в `launcher/__init__.py`:
```python
MS_CLIENT_ID = "1a2b3c4d-...."
```
и выпусти новую версию — вход заработает у всех, кто обновится. Проверить до релиза можно,
вписав ID в **Настройки → Сервисы → Свой Microsoft Client ID**.

## Как это устроено
`launcher/auth.py`: код устройства (`/consumers/oauth2/v2.0/devicecode`) → токен Microsoft →
Xbox Live → XSTS → `api.minecraftservices.com` → профиль. В `data\accounts.json` хранится
только refresh-токен; пароль лаунчер не видит. Понятные ошибки: нет профиля Xbox, детская
учётная запись, игра не куплена, нет ника, приложение не одобрено Mojang.
