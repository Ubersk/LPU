# ЛПУ (Личный Помощник Участника)

## Оглавление
1. [Общее описание проекта](#общее-описание-проекта)
2. [Архитектура системы](#архитектура-системы)
3. [Роли пользователей](#роли-пользователей)
4. [Технологический стек](#технологический-стек)
5. [Структура проекта](#структура-проекта)
6. [API спецификация](#api-спецификация)
7. [База данных](#база-данных)
8. [Аутентификация и безопасность](#аутентификация-и-безопасность)
9. [Развертывание и DevOps](#развертывание-и-devops)
10. [План разработки](#план-разработки)
11. [Дорожная карта (Roadmap)](#дорожная-карта-roadmap)
12. [Команда и контакты](#команда-и-контакты)

---

## Общее описание проекта

**ЛПУ (Личный Помощник Участника)** — это многофункциональная веб-платформа для систематизации и контроля образовательных процессов в частных учебных учреждениях. Приложение предоставляет персонализированные рабочие пространства для всех участников образовательного процесса с учетом их ролей и потребностей.

### Ключевые особенности
- **Мобильно-ориентированный дизайн**: PWA-приложение, адаптированное для использования на смартфонах
- **Многоуровневая ролевая модель**: Индивидуальные интерфейсы для каждой категории пользователей
- **Микросервисная архитектура**: Масштабируемая и поддерживаемая система
- **Интеграционная платформа**: Поддержка подключения внешних сервисов
- **Аналитика в реальном времени**: Мониторинг и отчетность для администрации

### Целевая аудитория
- Частные учебные заведения (школы, курсы, тренинговые центры)
- Административный персонал учебных заведений
- Преподаватели и методисты
- Учащиеся и их родители

---

## Архитектура системы

### Высокоуровневая схема
```
┌─────────────────────────────────────────────────────────────┐
│                    Клиентские устройства                     │
│   (мобильные телефоны, планшеты, десктопы)                  │
└───────────────────────────┬─────────────────────────────────┘
                            │ HTTPS/WebSocket
┌───────────────────────────▼─────────────────────────────────┐
│                    API Gateway (Nginx/Kong)                 │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌──────────┐      │
│  │  Auth    │ │  Routing │ │  Rate    │ │  Logging │      │
│  │  Check   │ │          │ │  Limit   │ │          │      │
│  └──────────┘ └──────────┘ └──────────┘ └──────────┘      │
└─────────────────┬──────────────┬──────────────┬────────────┘
                  │              │              │
      ┌───────────▼───┐  ┌──────▼──────┐  ┌───▼──────────┐
      │   Сервис      │  │   Сервис    │  │   Сервис     │
      │ Пользователей │  │ Образования │  │ Уведомлений  │
      │  (Django)     │  │  (Django)   │  │  (Node.js)   │
      └──────┬────────┘  └──────┬──────┘  └──────┬───────┘
             │                  │                 │
      ┌──────▼──────────────────▼─────────────────▼──────┐
      │              Базы данных и кэш                   │
      │   ┌────────────┐ ┌──────────┐ ┌────────────┐    │
      │   │PostgreSQL  │ │  Redis   │ │   Redis    │    │
      │   │ (Основная) │ │  (Кэш)   │ │(Очереди)   │    │
      │   └────────────┘ └──────────┘ └────────────┘    │
      └─────────────────────────────────────────────────┘
```

### Микросервисная структура
1. **User Management Service** (Django) - управление пользователями, аутентификация, роли
2. **Education Service** (Django) - учебный процесс, расписание, задания, оценки
3. **Communication Service** (Node.js) - чаты, комментарии, обсуждения
4. **Analytics Service** (Django + Celery) - аналитика, отчеты, дашборды
5. **Notification Service** (Node.js) - email, SMS, push-уведомления
6. **File Storage Service** (Django) - управление файлами и документами

---

## Роли пользователей

### Глобальные роли (системные)
- **Супер-администратор** - полный доступ ко всей системе
- **Модератор** - управление контентом, мониторинг активности

### Учебное учреждение №1
- **Директор** - управление учреждением, аналитика, отчетность
- **Администрация** - оперативное управление, расписание, кадры
- **Работники** - преподаватели, методисты, вспомогательный персонал
- **Родители** - мониторинг успеваемости, коммуникация с учителями
- **Ученики** - учебные материалы, задания, прогресс, коммуникация

### Учебное учреждение №2
- *(Аналогичная структура ролей)*

### Особенности ролевой модели
- Пользователь может принадлежать к нескольким учреждениям
- Иерархическое наследование прав (Директор → Администрация → Работники)
- Контекстное переключение между ролями в интерфейсе
- Временное делегирование прав

---

## Технологический стек

### Backend
- **Django 4.2+** - основной фреймворк
  - Django REST Framework - API
  - Django Channels - WebSocket для реального времени
  - Django Celery - асинхронные задачи
  - Django Guardian - расширенные права доступа
- **PostgreSQL 15+** - основная база данных
- **Redis 7+** - кэширование и очереди задач
- **JWT** - аутентификация (djangorestframework-simplejwt)
- **Python 3.11+** - язык программирования

### Frontend
- **React 18+** - UI библиотека
  - React Router DOM - маршрутизация
  - Redux Toolkit - управление состоянием
  - React Query - кэширование API запросов
  - Formik + Yup - формы и валидация
- **Tailwind CSS 3+** - утилитарный CSS фреймворк
- **TypeScript** - типизированный JavaScript
- **Vite** - сборка и дев-сервер
- **PWA** - прогрессивное веб-приложение
  - Workbox - service workers
  - Web App Manifest

### DevOps и инфраструктура
- **Docker + Docker Compose** - контейнеризация
- **Nginx** - веб-сервер и API Gateway
- **GitHub Actions** - CI/CD
- **Sentry** - мониторинг ошибок
- **Prometheus + Grafana** - мониторинг системы

### Внешние интеграции
- **Google Calendar API** - синхронизация расписания
- **Telegram Bot API** - уведомления и коммуникация
- **SMTP сервисы** - email рассылки (SendGrid/Mailgun)
- **SMS шлюзы** - SMS уведомления
- **Сервисы аналитики** - метрики использования

---

## Структура проекта

```
lpu-platform/
├── backend/                    # Бэкенд сервисы
│   ├── user-service/          # Сервис пользователей
│   │   ├── src/
│   │   │   ├── users/
│   │   │   ├── auth/
│   │   │   ├── roles/
│   │   │   └── institutions/
│   │   ├── Dockerfile
│   │   └── requirements.txt
│   │
│   ├── education-service/     # Образовательный сервис
│   │   ├── src/
│   │   │   ├── schedule/
│   │   │   ├── courses/
│   │   │   ├── assignments/
│   │   │   └── grades/
│   │   └── Dockerfile
│   │
│   └── shared/               # Общие библиотеки
│       ├── models/
│       ├── utils/
│       └── exceptions/
│
├── frontend/                  # Фронтенд приложение
│   ├── src/
│   │   ├── components/       # Переиспользуемые компоненты
│   │   │   ├── common/       # Кнопки, формы, модалки
│   │   │   ├── layout/       # Шапка, сайдбар, футер
│   │   │   └── features/     # Бизнес-компоненты
│   │   │
│   │   ├── pages/           # Страницы приложения
│   │   │   ├── auth/        # Авторизация
│   │   │   ├── dashboard/   # Дашборды по ролям
│   │   │   ├── schedule/    # Расписание
│   │   │   ├── assignments/ # Задания
│   │   │   └── analytics/   # Аналитика
│   │   │
│   │   ├── store/           # Состояние приложения
│   │   │   ├── slices/      # Redux слайсы
│   │   │   └── hooks/       # Кастомные хуки
│   │   │
│   │   ├── services/        # API клиенты
│   │   │   ├── api/
│   │   │   └── websocket/
│   │   │
│   │   ├── utils/           # Вспомогательные функции
│   │   └── types/           # TypeScript типы
│   │
│   ├── public/              # Статические файлы
│   │   ├── manifest.json    # PWA манифест
│   │   └── icons/           # Иконки приложения
│   │
│   ├── Dockerfile
│   └── package.json
│
├── infrastructure/           # Инфраструктура
│   ├── docker-compose.yml
│   ├── nginx/
│   │   └── nginx.conf
│   ├── prometheus/
│   └── grafana/
│
├── docs/                    # Документация
│   ├── api/                # API спецификация
│   ├── architecture/       # Архитектурные решения
│   └── deployment/         # Инструкции по развертыванию
│
├── tests/                  # Тесты
│   ├── unit/
│   ├── integration/
│   └── e2e/
│
└── scripts/               # Вспомогательные скрипты
    ├── deploy.sh
    ├── backup.sh
    └── migrate.sh
```

---

## API спецификация

### Базовый URL
```
https://api.lpu-platform.com/v1
```

### Аутентификация
Все запросы (кроме аутентификации) требуют JWT токен в заголовке:
```
Authorization: Bearer <access_token>
```

### Основные эндпоинты

#### 1. Аутентификация
```http
POST /auth/login/
Content-Type: application/json

{
  "email": "user@example.com",
  "password": "password123"
}

Response:
{
  "access": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
  "refresh": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9...",
  "user": {
    "id": 1,
    "email": "user@example.com",
    "roles": ["student", "parent"],
    "institutions": [1, 2]
  }
}
```

#### 2. Пользователи
```http
GET /users/me/ - текущий пользователь
GET /users/{id}/ - информация о пользователе
PUT /users/{id}/ - обновление профиля
GET /users/{id}/roles/ - роли пользователя
```

#### 3. Учебные учреждения
```http
GET /institutions/ - список учреждений
POST /institutions/ - создание учреждения (админ)
GET /institutions/{id}/ - детали учреждения
GET /institutions/{id}/members/ - участники учреждения
```

#### 4. Расписание
```http
GET /schedule/ - расписание пользователя
GET /schedule/teacher/{teacher_id}/ - расписание преподавателя
GET /schedule/group/{group_id}/ - расписание группы
POST /schedule/lessons/ - создание занятия (админ)
PUT /schedule/lessons/{id}/ - изменение занятия
```

#### 5. Задания и оценки
```http
GET /assignments/ - задания пользователя
POST /assignments/ - создание задания (преподаватель)
GET /assignments/{id}/submit/ - отправить выполнение
GET /grades/ - оценки пользователя
POST /grades/ - выставить оценку (преподаватель)
```

### WebSocket соединения
```
wss://api.lpu-platform.com/ws/
```

Каналы:
- `notifications.{user_id}` - уведомления
- `chat.{room_id}` - чат комнаты
- `updates.{institution_id}` - обновления учреждения

---

## База данных

### Основные сущности

```sql
-- Учебные учреждения
CREATE TABLE institutions (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    description TEXT,
    settings JSONB DEFAULT '{}',
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Пользователи
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE NOT NULL,
    password_hash VARCHAR(255) NOT NULL,
    first_name VARCHAR(100),
    last_name VARCHAR(100),
    phone VARCHAR(20),
    avatar_url VARCHAR(500),
    is_active BOOLEAN DEFAULT TRUE,
    last_login TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Связь пользователей с учреждениями
CREATE TABLE user_institutions (
    user_id INTEGER REFERENCES users(id),
    institution_id INTEGER REFERENCES institutions(id),
    role VARCHAR(50) NOT NULL, -- 'director', 'teacher', 'student', 'parent'
    permissions JSONB DEFAULT '{}',
    PRIMARY KEY (user_id, institution_id)
);

-- Расписание
CREATE TABLE schedule_lessons (
    id SERIAL PRIMARY KEY,
    institution_id INTEGER REFERENCES institutions(id),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    teacher_id INTEGER REFERENCES users(id),
    group_id INTEGER,
    start_time TIMESTAMP NOT NULL,
    end_time TIMESTAMP NOT NULL,
    room VARCHAR(50),
    recurring_pattern JSONB, -- для повторяющихся занятий
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Задания
CREATE TABLE assignments (
    id SERIAL PRIMARY KEY,
    lesson_id INTEGER REFERENCES schedule_lessons(id),
    title VARCHAR(255) NOT NULL,
    description TEXT,
    due_date TIMESTAMP,
    max_score INTEGER,
    attachments JSONB DEFAULT '[]',
    created_by INTEGER REFERENCES users(id),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Оценки
CREATE TABLE grades (
    id SERIAL PRIMARY KEY,
    assignment_id INTEGER REFERENCES assignments(id),
    student_id INTEGER REFERENCES users(id),
    score INTEGER,
    feedback TEXT,
    graded_by INTEGER REFERENCES users(id),
    graded_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Уведомления
CREATE TABLE notifications (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    type VARCHAR(50) NOT NULL, -- 'assignment', 'grade', 'message', 'system'
    title VARCHAR(255) NOT NULL,
    message TEXT,
    data JSONB DEFAULT '{}',
    is_read BOOLEAN DEFAULT FALSE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### Индексы для оптимизации
```sql
CREATE INDEX idx_user_institutions_user ON user_institutions(user_id);
CREATE INDEX idx_user_institutions_institution ON user_institutions(institution_id);
CREATE INDEX idx_schedule_institution ON schedule_lessons(institution_id, start_time);
CREATE INDEX idx_assignments_due_date ON assignments(due_date) WHERE due_date > NOW();
CREATE INDEX idx_notifications_user_unread ON notifications(user_id) WHERE is_read = FALSE;
```

---

## Аутентификация и безопасность

### JWT реализация
```python
# settings.py
SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME': timedelta(minutes=60),
    'REFRESH_TOKEN_LIFETIME': timedelta(days=7),
    'ROTATE_REFRESH_TOKENS': True,
    'BLACKLIST_AFTER_ROTATION': True,
    'ALGORITHM': 'HS256',
    'SIGNING_KEY': SECRET_KEY,
    'AUTH_HEADER_TYPES': ('Bearer',),
}
```

### RBAC (Role-Based Access Control)
```python
# Пример проверки прав
class IsInstitutionDirector(permissions.BasePermission):
    def has_permission(self, request, view):
        institution_id = view.kwargs.get('institution_id')
        return request.user.has_institution_role(institution_id, 'director')

class IsTeacherOrAbove(permissions.BasePermission):
    def has_permission(self, request, view):
        institution_id = view.kwargs.get('institution_id')
        user_roles = request.user.get_institution_roles(institution_id)
        return any(role in user_roles for role in ['teacher', 'admin', 'director'])
```

### Шифрование данных
- Пароли: bcrypt с солью
- Чувствительные данные: AES-256 шифрование
- SSL/TLS: обязательное использование HTTPS
- Заголовки безопасности: CSP, HSTS, X-Frame-Options

---

## Развертывание и DevOps

### Docker конфигурация
```yaml
# docker-compose.yml
version: '3.8'

services:
  postgres:
    image: postgres:15
    environment:
      POSTGRES_DB: lpu
      POSTGRES_USER: lpu_user
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
    volumes:
      - redis_data:/data

  user-service:
    build: ./backend/user-service
    ports:
      - "8001:8000"
    environment:
      DATABASE_URL: postgresql://lpu_user:${DB_PASSWORD}@postgres/lpu
      REDIS_URL: redis://redis:6379/0
    depends_on:
      - postgres
      - redis

  nginx:
    image: nginx:alpine
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - ./infrastructure/nginx/nginx.conf:/etc/nginx/nginx.conf
      - ./infrastructure/ssl:/etc/nginx/ssl
    depends_on:
      - user-service
      - education-service

volumes:
  postgres_data:
  redis_data:
```

### CI/CD пайплайн
```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Run tests
        run: |
          docker-compose run --rm backend pytest
          docker-compose run --rm frontend npm test

  build-and-push:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Build and push Docker images
        run: |
          docker build -t lpu-backend:latest ./backend
          docker build -t lpu-frontend:latest ./frontend

  deploy:
    needs: build-and-push
    runs-on: ubuntu-latest
    steps:
      - name: Deploy to server
        run: |
          ssh user@server "cd /opt/lpu && docker-compose pull && docker-compose up -d"
```

---

## План разработки

### Фаза 1: MVP (4-6 недель)
1. **Неделя 1-2**: Базовая настройка проекта
   - Инициализация Django + React
   - Базовая аутентификация (JWT)
   - Docker окружение

2. **Неделя 3-4**: Модели и API
   - Модели пользователей и учреждений
   - RBAC система
   - Базовые API эндпоинты

3. **Неделя 5-6**: Фронтенд основа
   - Layout и навигация
   - Страница входа/регистрации
   - Базовый дашборд

### Фаза 2: Основной функционал (8-10 недель)
1. Управление учреждениями
2. Расписание занятий
3. Система заданий
4. Оценки и прогресс
5. Коммуникации (чаты, комментарии)

### Фаза 3: Расширение (6-8 недель)
1. Мобильная PWA
2. Интеграции с внешними сервисами
3. Расширенная аналитика
4. Геймификация

---

## Дорожная карта (Roadmap)

### Q1 2024: Основа
- [ ] MVP с базовым функционалом
- [ ] Тестирование в пилотных учреждениях
- [ ] Сбор обратной связи

### Q2 2024: Расширение
- [ ] Мобильное PWA приложение
- [ ] Интеграция с календарями
- [ ] Система уведомлений
- [ ] Базовая аналитика

### Q3 2024: Оптимизация
- [ ] Улучшение производительности
- [ ] Расширение аналитики
- [ ] Геймификация (баллы, рейтинги)
- [ ] Массовые операции

### Q4 2024: Масштабирование
- [ ] Мультитенантная архитектура
- [ ] Marketplace расширений
- [ ] API для сторонних разработчиков
- [ ] Мобильные приложения (iOS/Android)

---

## Команда и контакты

### Требуемые специалисты
1. **Backend разработчик (Python/Django)**
   - Опыт работы с Django 3+ лет
   - Знание PostgreSQL, Redis
   - Понимание микросервисной архитектуры

2. **Frontend разработчик (React/TypeScript)**
   - Опыт с React 3+ лет
   - TypeScript, Redux Toolkit
   - PWA, адаптивная верстка

3. **DevOps инженер**
   - Docker, Kubernetes
   - CI/CD настройка
   - Мониторинг и логирование

4. **UI/UX дизайнер**
   - Дизайн мобильных интерфейсов
   - Прототипирование Figma
   - Дизайн-системы

5. **QA инженер**
   - Автоматизированное тестирование
   - E2E тесты (Cypress/Playwright)
   - Нагрузочное тестирование

### Контакты
- **Репозиторий проекта**: [github.com/Ubersk/LPU](https://github.com/Ubersk/LPU)
- **Документация**: [docs.lpu-platform.com](https://docs.lpu-platform.com)
- **Трекер задач**: [lpu-platform.atlassian.net](https://lpu-platform.atlassian.net)
- **Slack команды**: [lpu-platform.slack.com](https://lpu-platform.slack.com)

### Процесс разработки
1. **Git Flow** ветвление
2. **Code Review** обязателен для всех PR
3. **Тестирование** покрытие не менее 80%
4. **Документирование** всех API и компонентов
5. **Еженедельные демо** и планирование спринтов

---

## Лицензия
© 2025 ЛПУ Платформа. Все права защищены.
Для внутреннего использования разработческой командой.

*Последнее обновление: 15 марта 2024*
