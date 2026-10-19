# Быстрый старт - afin

## 🚀 Быстрый запуск с Docker (рекомендуется)

### ⚠️ ВАЖНО: Перед запуском убедитесь, что Docker Desktop запущен!

Если видите ошибку "The system cannot find the file specified" - запустите Docker Desktop и дождитесь его полной загрузки.

### Windows (PowerShell):

```powershell
# 1. Убедитесь, что вы в корневой директории проекта
# cd afin-ui-backend/afin-frontend

# 2. Создайте файл backend/.env (если еще не создан)
if (-not (Test-Path backend\.env)) {
    Copy-Item backend\.env.example backend\.env
}

# 3. Запустите все сервисы
docker-compose up --build

# 4. В другом терминале PowerShell выполните миграции и seed данных
docker exec -it afin-backend npx prisma migrate deploy
docker exec -it afin-backend npm run db:seed

# 5. Откройте браузер
# Frontend: http://localhost:3000
# Backend API: http://localhost:5000
```

### Linux/Mac:

```bash
# 1. Создайте файл backend/.env
cd backend
cp .env.example .env
cd ..

# 2. Запустите все сервисы
docker-compose up --build

# 3. В другом терминале выполните миграции и seed данных
docker exec -it afin-backend npx prisma migrate deploy
docker exec -it afin-backend npm run db:seed

# 4. Откройте браузер
# Frontend: http://localhost:3000
# Backend API: http://localhost:5000
```

## 📝 Тестовые учетные записи

После выполнения seed:

- **Email:** `admin@afin.ru` / **Пароль:** `password123`
- **Email:** `user@afin.ru` / **Пароль:** `password123`

## 🛠️ Локальный запуск (без Docker)

```bash
# 1. Установите зависимости
npm run install:all

# 2. Настройте PostgreSQL и создайте базу данных
# Создайте файл backend/.env с настройками БД

# 3. Настройте Prisma
cd backend
npx prisma generate
npx prisma migrate dev
npx prisma db seed
cd ..

# 4. Запустите приложение
npm run dev

# Или отдельно:
# Terminal 1: cd backend && npm run dev
# Terminal 2: cd frontend && npm run dev
```

## 📚 Полная документация

См. [README.md](./README.md) для подробной документации.

