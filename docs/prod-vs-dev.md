# Отличия dev и prod

## Реплики
**dev:**
- Все сервисы (frontend, bff, user-service, message-service) = 1 реплика

**prod:**
- Все сервисы (frontend, bff, user-service, message-service) = 3 реплики

## Ресурсы (Requests/Limits)
**dev:**
- Базовые/минимальные значения (например, frontend: 50m CPU / 64Mi RAM)

**prod:**
- Увеличенные значения для стабильности (например, frontend: 200m CPU / 256Mi RAM)

## Images
**dev:**
- Используется тег `latest` 

**prod:**
- Фиксированный тег `v1.0.0/stable`

## Namespace и имена ресурсов
**dev:**
- Namespace: `messager-dev`
- К именам ресурсов добавляется суффикс `-dev`

**prod:**
- Namespace: `messager-prod`
- К именам ресурсов добавляется суффикс `-prod`

## Ingress (Маршрутизация)
**dev:**
- Host: `dev.messager.local`

**prod:**
- Host: `messager.example.com`