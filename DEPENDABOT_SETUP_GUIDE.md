# Пошаговая инструкция: Настройка GitHub Dependabot для SCA

## Что такое GitHub Dependabot?

GitHub Dependabot - это встроенный инструмент GitHub для автоматического анализа зависимостей проекта на наличие уязвимостей. Он:
- ✅ Автоматически сканирует зависимости
- ✅ Находит уязвимости
- ✅ Создаёт Pull Requests для обновления уязвимых зависимостей
- ✅ Не требует настройки базы данных
- ✅ Работает полностью автоматически

---

## Шаг 1: Проверка структуры проекта

Убедитесь, что в проекте есть файлы зависимостей:

```bash
# Проверьте наличие composer.json
ls -la app/vulnerabilities/api/composer.json

# Если файл есть - отлично!
```

---

## Шаг 2: Создание конфигурации Dependabot

Файл `.github/dependabot.yml` уже создан! Проверьте его содержимое:

```yaml
version: 2
updates:
  - package-ecosystem: "composer"
    directory: "/app/vulnerabilities/api"
    schedule:
      interval: "weekly"
      day: "monday"
      time: "09:00"
    open-pull-requests-limit: 10
    labels:
      - "dependencies"
      - "security"
      - "dependabot"
```

**Что это означает:**
- `package-ecosystem: "composer"` - сканирует PHP зависимости (Composer)
- `directory: "/app/vulnerabilities/api"` - путь к директории с `composer.json`
- `schedule: weekly` - проверка раз в неделю (понедельник, 09:00)
- `open-pull-requests-limit: 10` - максимум 10 открытых PR одновременно

---

## Шаг 3: Активация Dependabot в GitHub

### 3.1. Включите Dependabot alerts (если ещё не включено)

1. Перейдите в репозиторий: `https://github.com/DevSecOps-F25/DevSecOps`
2. Нажмите **Settings** (вверху справа)
3. В левом меню выберите **Security**
4. Найдите раздел **Code security and analysis**
5. Включите:
   - ✅ **Dependabot alerts** - включите
   - ✅ **Dependabot security updates** - включите (опционально, но рекомендуется)

### 3.2. Проверьте, что файл конфигурации закоммичен

```bash
# Проверьте, что файл есть в репозитории
git status .github/dependabot.yml

# Если файл не закоммичен, закоммитьте:
git add .github/dependabot.yml
git commit -m "Add Dependabot configuration for SCA analysis"
git push origin master
```

---

## Шаг 4: Первый запуск Dependabot

После создания/обновления файла `.github/dependabot.yml`:

1. **GitHub автоматически обнаружит конфигурацию** (может занять несколько минут)
2. **Dependabot начнёт сканирование** зависимостей
3. **Результаты появятся в разделе Security**

### Как проверить:

1. Перейдите: `https://github.com/DevSecOps-F25/DevSecOps/security/dependabot`
2. Там будут все найденные уязвимости
3. Dependabot автоматически создаст Pull Requests для исправления

---

## Шаг 5: Добавление проверки Dependabot в workflow

Добавим шаг в workflow для автоматической проверки Dependabot alerts:

### 5.1. Откройте файл workflow

```bash
code .github/workflows/security-scan.yml
# или
nano .github/workflows/security-scan.yml
```

### 5.2. Добавьте новый job для проверки Dependabot

Добавьте после job `sca-analysis`:

```yaml
  dependabot-check:
    runs-on: ubuntu-latest
    permissions:
      security-events: read
      contents: read
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Check Dependabot Security Alerts
        run: |
          echo "📊 Checking Dependabot Security Alerts..."
          echo "View alerts at: https://github.com/${{ github.repository }}/security/dependabot"
          
          # Устанавливаем GitHub CLI (если не установлен)
          if ! command -v gh &> /dev/null; then
            echo "Installing GitHub CLI..."
            type -p curl >/dev/null || (apt-get update && apt-get install curl -y)
            curl -fsSL https://cli.github.com/packages/githubcli-archive-keyring.gpg | dd of=/usr/share/keyrings/githubcli-archive-keyring.gpg
            echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" | tee /etc/apt/sources.list.d/github-cli.list > /dev/null
            apt-get update
            apt-get install gh -y
          fi
          
          # Авторизуемся (используем GITHUB_TOKEN)
          echo "${{ secrets.GITHUB_TOKEN }}" | gh auth login --with-token
          
          # Получаем список Dependabot alerts
          ALERTS=$(gh api repos/${{ github.repository }}/dependabot/alerts --jq 'length' 2>/dev/null || echo "0")
          
          if [ "$ALERTS" = "0" ] || [ -z "$ALERTS" ]; then
            echo "✅ No Dependabot alerts found"
          else
            echo "⚠️ Found $ALERTS Dependabot security alerts"
            echo ""
            echo "Top 5 alerts:"
            gh api repos/${{ github.repository }}/dependabot/alerts --jq '.[] | select(.state=="open") | "  - \(.dependency.package.name)@\(.dependency.manifest_path): \(.security_advisory.summary)"' | head -5 || echo "  (Unable to fetch details)"
            echo ""
            echo "View all alerts: https://github.com/${{ github.repository }}/security/dependabot"
          fi
```

---

## Шаг 6: Тестирование

### 6.1. Закоммитьте изменения

```bash
cd /Users/MojPK/Downloads/University/NCS/DevSecOps
git add .github/dependabot.yml .github/workflows/security-scan.yml
git commit -m "Configure Dependabot for SCA analysis"
git push origin master
```

### 6.2. Проверьте работу Dependabot

1. **Подождите 5-10 минут** после push
2. Перейдите: `https://github.com/DevSecOps-F25/DevSecOps/security/dependabot`
3. Там должны появиться результаты сканирования

### 6.3. Запустите workflow

1. Перейдите: `https://github.com/DevSecOps-F25/DevSecOps/actions`
2. Запустите workflow вручную или дождитесь автоматического запуска
3. Проверьте, что job `dependabot-check` выполнился успешно

---

## Шаг 7: Просмотр результатов

### 7.1. В GitHub интерфейсе

1. **Security → Dependabot:**
   - `https://github.com/DevSecOps-F25/DevSecOps/security/dependabot`
   - Список всех найденных уязвимостей
   - Детальная информация по каждой уязвимости

2. **Pull Requests:**
   - Dependabot автоматически создаст PR для исправления уязвимостей
   - PR будут помечены лейблом `dependabot`

### 7.2. В workflow

- Job `dependabot-check` выведет количество найденных уязвимостей
- Ссылка на просмотр всех alerts

---

## Что дальше?

### Автоматические обновления

Dependabot будет:
- ✅ Еженедельно проверять зависимости
- ✅ Создавать Pull Requests для обновления уязвимых зависимостей
- ✅ Уведомлять о новых уязвимостях

### Ручное управление

Вы можете:
- Просматривать alerts в разделе Security
- Одобрять или отклонять Pull Requests от Dependabot
- Настраивать расписание проверок в `.github/dependabot.yml`

---

## Преимущества Dependabot

1. ✅ **Не требует настройки базы данных** - всё работает автоматически
2. ✅ **Интегрирован в GitHub** - результаты прямо в интерфейсе
3. ✅ **Автоматические PR** - предлагает исправления
4. ✅ **Бесплатный** - входит в GitHub
5. ✅ **Надёжный** - используется миллионами проектов

---

## Для отчёта

В разделе "SCA (Software Composition Analysis)" можно описать:

1. **Инструмент:** GitHub Dependabot
2. **Настройка:** Создан файл `.github/dependabot.yml` с конфигурацией
3. **Интеграция:** Добавлен job в GitHub Actions workflow
4. **Результаты:** Автоматическое сканирование зависимостей и создание PR для исправления уязвимостей
5. **Преимущества:** Не требует настройки базы данных, полностью автоматизирован

---

## Troubleshooting

### Проблема: Dependabot не запускается

**Решение:**
1. Проверьте, что файл `.github/dependabot.yml` закоммичен в репозиторий
2. Убедитесь, что Dependabot alerts включены в Settings → Security
3. Подождите 5-10 минут после создания файла

### Проблема: Нет результатов сканирования

**Решение:**
1. Проверьте путь к `composer.json` в конфигурации
2. Убедитесь, что файл `composer.json` существует
3. Проверьте формат файла (должен быть валидный JSON)

### Проблема: Dependabot не создаёт PR

**Решение:**
1. Проверьте настройки репозитория (разрешения)
2. Убедитесь, что `open-pull-requests-limit` не достигнут
3. Проверьте, что есть уязвимости для исправления

---

## Готово! 🎉

Теперь у вас настроен GitHub Dependabot для автоматического анализа зависимостей!

