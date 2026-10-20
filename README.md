# RetailCRM plugin

Официальный плагин для работы с RetailCRM из OpenAI Codex и Claude Code. Позволяет подключать AI-агента к RetailCRM через MCP и помогает безопасно находить, проверять и выполнять GraphQL-запросы и мутации.

Один репозиторий, устанавливаемый и в OpenAI Codex, и в Claude Code, объединяет:

1. **RetailCRM MCP server** — удалённый MCP-сервер по адресу `https://mcp.rcrm-tech.ru/mcp`, предоставляющий инструменты для работы с API RetailCRM.
2. **Skill `use-retailcrm`** — инструкции для агента, которые помогают найти подходящие операции, получить необходимую часть схемы, построить и проверить GraphQL-документ, а затем безопасно выполнить его. Перед выполнением мутаций skill требует отдельного явного подтверждения пользователя.

MCP-сервер предоставляет инструменты и доступ к данным, а skill объясняет агенту, как корректно ими пользоваться.

Официальные выпуски: [retailcrm/mcp-plugin](https://github.com/retailcrm/mcp-plugin). Этот репозиторий обновляется автоматически при выпуске новой версии.

Исходная версия перенесена из [ilyavlasoff/RetailCRM-Plugin](https://github.com/ilyavlasoff/RetailCRM-Plugin).

## Установка плагина

### OpenAI Codex

Откройте **File → Settings → Integrations → Plugins**, нажмите **Add → Add a marketplace** и заполните форму:

- **Source:** `https://github.com/retailcrm/mcp-plugin.git`;
- **Git ref:** `main`.

Нажмите **Add marketplace**. После добавления:

1. Откройте вкладку **Personal** в разделе **Plugins**.
2. Выберите **Created by you → RetailCRM**.
3. Нажмите **Install Plugin**, затем **Try Now** или создайте новый чат.

Marketplace также можно добавить из командной строки:

```bash
codex plugin marketplace add https://github.com/retailcrm/mcp-plugin.git
```

Затем установите плагин:

```bash
codex plugin add retailcrm@retailcrm
```

Вместо второй команды можно открыть `/plugins`, выбрать marketplace `retailcrm` и установить плагин `retailcrm` интерактивно. После установки создайте новый тред.

### Claude Code

Добавьте marketplace и установите плагин:

```bash
claude plugin marketplace add https://github.com/retailcrm/mcp-plugin.git
claude plugin install retailcrm@retailcrm-plugin
```

После установки запустите новую сессию Claude Code, чтобы плагин стал доступен.

Установка плагина не требует OAuth-входа. Для обращения к RetailCRM MCP необходимо отдельно настроить персональный токен.

## Аутентификация и подключение MCP

### Получение персонального токена

1. Откройте страницу создания токена в системе:

   ```text
   https://<домен-вашей-системы>/profile/access-tokens/new
   ```

2. Укажите название токена, например `Codex — работа с CRM` или `Claude Code — работа с CRM`.
3. При необходимости задайте срок действия.
4. Выдайте только те права, которые нужны для планируемых операций через MCP.
5. Нажмите **Сохранить** и скопируйте токен. После создания он показывается только один раз.

Используйте отдельный токен с минимально необходимыми правами. Не добавляйте значение токена в репозиторий, конфигурационные файлы или сообщения агенту.

Сохраните токен в переменной окружения до запуска OpenAI Codex или Claude Code:

```bash
export RETAILCRM_TOKEN="ваш-персональный-токен"
```

Для текущей сессии PowerShell:

```powershell
$env:RETAILCRM_TOKEN = "ваш-персональный-токен"
```

### OpenAI Codex

Добавьте сервер в `~/.codex/config.toml`:

```toml
[mcp_servers.retailcrm_mcp]
url = "https://mcp.rcrm-tech.ru/mcp"
bearer_token_env_var = "RETAILCRM_TOKEN"
enabled = true
enabled_tools = [
  "GqlSearchOperations",
  "GqlDescribeOperations",
  "GqlDescribeTypes",
  "GqlExecuteOperation",
]
```

Перезапустите Codex и проверьте подключение командой `/mcp`.

### Claude Code

Создайте или дополните файл `.mcp.json` в корне проекта:

```json
{
  "mcpServers": {
    "retailcrm_mcp": {
      "type": "http",
      "url": "https://mcp.rcrm-tech.ru/mcp",
      "headers": {
        "Authorization": "Bearer ${RETAILCRM_TOKEN}"
      }
    }
  }
}
```

Перезапустите Claude Code и проверьте подключение командой `/mcp`. В репозиторий можно добавлять конфигурацию со ссылкой на переменную окружения, но не само значение токена.

## Standalone-подключение MCP

RetailCRM MCP можно использовать без установки плагина:

1. Получите персональный токен и сохраните его в `RETAILCRM_TOKEN`, как описано выше.
2. Добавьте сервер в OpenAI Codex:

   ```bash
   codex mcp add retailcrm_mcp \
     --url https://mcp.rcrm-tech.ru/mcp \
     --bearer-token-env-var RETAILCRM_TOKEN
   ```

3. Добавьте сервер в Claude Code:

   ```bash
   claude mcp add --transport http retailcrm_mcp https://mcp.rcrm-tech.ru/mcp \
     --header 'Authorization: Bearer ${RETAILCRM_TOKEN}'
   ```

4. Перезапустите клиент и проверьте доступность `retailcrm_mcp` через `/mcp`.

В standalone-режиме инструменты MCP будут доступны, но skill `use-retailcrm` и его правила построения, проверки и безопасного выполнения операций установлены не будут.

## Обновление

Обновите каталог `retailcrm` в Codex или `retailcrm-plugin` в Claude Code, затем обновите установленный плагин через меню управления плагинами клиента.

Версия пакета и происхождение выпуска указаны в `release-manifest.json`. Архив соответствующей версии доступен в [GitHub Releases](https://github.com/retailcrm/mcp-plugin/releases).

## Лицензия

Проект распространяется по лицензии [MIT](LICENSE).
