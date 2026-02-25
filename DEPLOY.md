# Настройка деплоя в Azure Web App через GitHub Actions

## Ошибка «Publish profile is invalid»

Она возникает, когда секрет `AZURE_WEBAPP_PUBLISH_PROFILE` в GitHub не совпадает с приложением **alex** или скопирован не полностью.

---

## Шаг 1: Получить профиль публикации (Publish Profile)

### Вариант A: Azure CLI (уже настроен у вас)

```bash
az webapp deployment list-publishing-profiles \
  --name alex \
  --resource-group AlexResourceGroup \
  --xml
```

Вывод — это **целиком** ваш publish profile. Его нужно сохранить в секрет (см. шаг 2).

Чтобы сохранить в файл и потом скопировать:

```bash
az webapp deployment list-publishing-profiles \
  --name alex \
  --resource-group AlexResourceGroup \
  --xml > publishProfile.xml
```

Далее откройте `publishProfile.xml` в редакторе и скопируйте **весь** текст от первой до последней строки.

### Вариант B: Azure Portal

1. Откройте [portal.azure.com](https://portal.azure.com).
2. Перейдите в **App Services** → выберите приложение **alex**.
3. На странице приложения нажмите **Get publish profile** (или **Download publish profile**).
4. Откроется файл `.PublishSettings`. Откройте его в блокноте и скопируйте **весь** XML (от `<publishData>` до `</publishData>` или весь файл).

---

## Шаг 2: Добавить секрет в GitHub

1. Откройте репозиторий на GitHub.
2. **Settings** → **Secrets and variables** → **Actions**.
3. Нажмите **New repository secret** (или **Update** у существующего).
4. **Name:** `AZURE_WEBAPP_PUBLISH_PROFILE` (точно так, как в workflow).
5. **Value:** вставьте **полное** содержимое publish profile:
   - весь XML из `publishProfile.xml` или из скачанного `.PublishSettings`;
   - без удаления строк и без лишнего текста до/после XML;
   - можно одной строкой или с переносами — главное, ничего не обрезать.

Сохраните секрет.

---

## Шаг 3: Проверить workflow

В `.github/workflows/main_mynoderissimo.yml` должны быть:

- `app-name: 'alex'` — имя вашего Web App в Azure;
- `publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}`;
- `slot-name: 'production'` — слот (обычно production).

Имя приложения в Azure и `app-name` в workflow должны **совпадать**.

---

## Шаг 4: Запустить деплой

- Сделайте коммит и пуш в ветку **main**, или  
- **Actions** → выберите workflow run → **Re-run all jobs**.

---

## Частые ошибки

| Проблема | Решение |
|----------|--------|
| Секрет создан от другого приложения | Скачайте профиль именно от приложения **alex** и обновите секрет. |
| Секрет обрезан при вставке | Копируйте весь XML целиком, без удаления начала/конца. |
| Другое имя приложения в Azure | Либо поменяйте `app-name` в workflow на реальное имя приложения, либо скачайте профиль от **alex**. |
| Не залогинены в Azure CLI | Выполните `az login` перед командой `list-publishing-profiles`. |

---

## Альтернатива: OIDC (без секрета с паролем)

Можно настроить вход в Azure через OIDC (Federated Credentials). Тогда не нужен publish profile, но потребуется создать App Registration и настроить trust между GitHub и Azure. Если понадобится такой вариант — можно описать отдельно.
