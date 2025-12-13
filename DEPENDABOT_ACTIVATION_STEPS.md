# Пошаговая инструкция: Активация Dependabot

## Где найти настройки Dependabot

### Шаг 1: Перейдите в Settings репозитория

1. Откройте репозиторий: `https://github.com/DevSecOps-F25/DevSecOps`
2. Нажмите на вкладку **Settings** (вверху справа, рядом с Insights)
3. В левом меню найдите раздел **Security**
4. Нажмите на **Advanced Security**

### Шаг 2: Включите Dependabot

В разделе **Advanced Security** вы увидите несколько опций. Найдите секцию **Dependabot** и включите:

#### 2.1. Dependency graph (опционально, но рекомендуется)
- Нажмите кнопку **Enable** рядом с "Dependency graph"
- Это поможет Dependabot лучше понимать зависимости

#### 2.2. Dependabot alerts (ОБЯЗАТЕЛЬНО)
- Найдите "Dependabot alerts"
- Нажмите кнопку **Enable** справа
- Это включит сканирование уязвимостей

#### 2.3. Dependabot security updates (рекомендуется)
- Найдите "Dependabot security updates"
- Нажмите кнопку **Enable** справа
- Это позволит Dependabot автоматически создавать PR для исправления уязвимостей

#### 2.4. Dependabot version updates (опционально)
- Если хотите, чтобы Dependabot обновлял все зависимости (не только уязвимые)
- Нажмите **Configure** и настройте через `.github/dependabot.yml`

---

## Альтернативный путь (если не видите Advanced Security)

### Вариант 1: Прямая ссылка

Попробуйте эту ссылку (замените на ваш репозиторий):
```
https://github.com/DevSecOps-F25/DevSecOps/settings/security_analysis
```

### Вариант 2: Через меню

1. Репозиторий → **Settings**
2. В левом меню найдите **Security**
3. Если не видите "Advanced Security", попробуйте:
   - **Code security and analysis** (старое название)
   - Или просто **Security** → прокрутите вниз

### Вариант 3: Через Security tab

1. Перейдите на вкладку **Security** (вверху репозитория)
2. Там должна быть ссылка "Enable Dependabot" или "Configure Dependabot"

---

## Проверка, что Dependabot включён

### Через интерфейс GitHub:

1. Перейдите: `https://github.com/DevSecOps-F25/DevSecOps/security`
2. Должна быть вкладка **Dependabot**
3. Если вкладка есть - Dependabot включён!

### Через API (в workflow):

Workflow автоматически проверит наличие alerts через GitHub API.

---

## Что делать, если не можете найти настройки

### Возможные причины:

1. **Нет прав администратора** - нужно быть владельцем или иметь права администратора репозитория
2. **Репозиторий приватный** - некоторые функции могут быть недоступны
3. **Старая версия интерфейса** - попробуйте обновить страницу

### Решение:

1. Убедитесь, что вы администратор репозитория
2. Попробуйте другой браузер
3. Очистите кэш браузера
4. Попробуйте прямую ссылку: `https://github.com/DevSecOps-F25/DevSecOps/settings/security_analysis`

---

## После включения

1. **Подождите 5-10 минут** - Dependabot начнёт сканирование
2. **Проверьте результаты:**
   - `https://github.com/DevSecOps-F25/DevSecOps/security/dependabot`
3. **Запустите workflow** - job `dependabot-check` покажет количество alerts

---

## Скриншот того, что нужно включить

Судя по вашему скриншоту, в разделе **Advanced Security** вы должны увидеть:

```
Dependabot
├── Dependabot alerts [Enable]
├── Dependabot security updates [Enable]
├── Grouped security updates [Enable]
└── Dependabot version updates [Configure]
```

**Включите хотя бы "Dependabot alerts"** - этого достаточно для работы!

