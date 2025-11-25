# Локальный стенд CI/CD (Docker Compose)

Минимальный набор сервисов для локальной проверки GitLab CI/CD пайплайнов: GitLab CE, GitLab Runner (Docker executor), SonarQube, Nexus OSS и локальный Docker Registry.

## Требования
- Docker и Docker Compose v2 (команда `docker compose`)
- Свободные порты: 8080, 2222, 9000, 8081, 5000

## Быстрый старт
1. Скопируйте переменные окружения и при необходимости отредактируйте:
   ```bash
   cp .env.example .env
   ```
2. Запустите стек:
   ```bash
   docker compose up -d
   ```
3. Проверьте, что контейнеры поднялись:
   ```bash
   docker compose ps
   ```
4. Откройте сервисы:
   - GitLab: http://localhost:8080 (SSH: порт 2222)
   - SonarQube: http://localhost:9000
   - Nexus: http://localhost:8081
   - Registry: localhost:5000 (без аутентификации по умолчанию)

## Первые входы и пароли
- **GitLab**: первый пароль `root` лежит в контейнере:  
  ```bash
  docker exec -it gitlab grep 'Password' /etc/gitlab/initial_root_password
  ```
- **SonarQube**: `admin` / `admin` (принудительно смените при первом входе).
- **Nexus**: пароль администратора внутри контейнера:  
  ```bash
  docker exec -it nexus cat /nexus-data/admin.password
  ```

## Регистрация GitLab Runner (Docker executor)
1. В GitLab откройте **Admin > Runners** и скопируйте токен регистрации.
2. Выполните регистрацию в контейнере (подставьте свой токен и описание):
   ```bash
   docker exec -it gitlab-runner gitlab-runner register \
     --non-interactive \
     --url "$GITLAB_URL" \
     --registration-token "$GITLAB_RUNNER_REGISTRATION_TOKEN" \
     --executor docker \
     --docker-image docker:24.0 \
     --docker-privileged \
     --description "local-docker-runner"
   ```
   - Docker socket уже смонтирован, поэтому Docker executor сможет запускать job-контейнеры.

## Полезные команды
- Остановить стек: `docker compose down`
- Остановить и удалить данные: `docker compose down -v` (удалит тома)
- Просмотреть логи конкретного сервиса: `docker compose logs -f gitlab`
