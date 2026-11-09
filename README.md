# Проверка публичных Docker-образов

Три независимых GitHub Actions workflow проверяют образы:

- `ghcr.io/ton-blockchain/localton:latest`
- `ghcr.io/ton-blockchain/verifier:latest`
- `ghcr.io/ton-blockchain/faucet:latest`
- `ghcr.io/ton-blockchain/actonscan-backend:latest`

| Workflow | Инструмент | Проверки | Отчёт |
| --- | --- | --- | --- |
| [Trivy](.github/workflows/trivy.yml) | `aquasecurity/trivy-action` | Уязвимости пакетов ОС и зависимостей | SARIF |
| [Grype](.github/workflows/grype.yml) | `anchore/scan-action` | Уязвимости пакетов ОС и зависимостей | SARIF |
| [Dockle](.github/workflows/dockle.yml) | `goodwithtech/dockle` Docker action | Настройки образа и практики сборки | JSON |

Готовые actions сами устанавливают и запускают сканеры. Dockle запускается прямо
из готового контейнера. Собственных скриптов и шагов `run` нет.

## Запуск

1. Отправить файлы в основную ветку GitHub-репозитория.
2. Открыть **Actions**, выбрать нужный workflow и нажать **Run workflow**.
3. Посмотреть логи четырёх jobs и скачать отчёты в **Artifacts** на странице запуска.

Каждый workflow также запускается при push изменения его YAML. Все четыре образа
проверяются независимо: ошибка одного не останавливает остальные.

Артефакты называются `trivy-localton`, `grype-localton`,
`dockle-localton` и аналогично для других образов. Хранятся 30 дней.
SARIF — отчёт в формате JSON с идентификаторами уязвимостей, уровнями серьёзности,
пакетами и описаниями находок.

Найденные проблемы не делают запуск красным. Ошибки самого сканера или скачивания
образа завершают соответствующий job с ошибкой. Отчёты сохраняются и при ошибках,
если сканер успел их создать.

## Авторизация

Все образы публичные: вход в GHCR и отдельный токен GitHub не нужны.
Trivy, Grype и Dockle запускаются без дополнительных secrets.

## Особенности результатов

Trivy и Grype могут давать разные результаты из-за покрытия, баз уязвимостей
и оценки серьёзности. Dockle проверяет конфигурацию образа: например, пользователя,
права файлов и кэши пакетов. Его замечания не являются CVE.

Все проверки используют `latest`. Для точного сравнения одной сборки можно заменить
тег на один и тот же digest образа в workflow.

Для Trivy выбраны `trivy-action` версии `0.35.0` (закреплена на полном commit SHA)
и Trivy `v0.69.3` из
[рекомендации Aqua Security](https://www.aquasec.com/blog/trivy-supply-chain-attack-what-you-need-to-know/).

Документация: [Trivy](https://trivy.dev/docs/latest/guide/target/container_image/),
[Grype](https://oss.anchore.com/docs/guides/vulnerability/getting-started/),
[Dockle](https://hub.docker.com/r/goodwithtech/dockle).
