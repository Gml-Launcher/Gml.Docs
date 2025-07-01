# Разработка бэкенда

В этом руководстве описан процесс разработки бэкенд-части модуля с использованием Gml.Web.Api.EndpointSDK. Этот SDK
предоставляет инструменты для создания эндпоинтов, обработки HTTP-запросов и работы с данными в рамках экосистемы GML.

## Установка SDK

Для начала работы с Gml.Web.Api.EndpointSDK необходимо добавить соответствующий NuGet-пакет в ваш проект:

```bash
dotnet add package Gml.Web.Api.EndpointSDK
```

Или через NuGet Package Manager в Visual Studio:

1. Щелкните правой кнопкой мыши на проекте в Solution Explorer
2. Выберите "Manage NuGet Packages..."
3. Перейдите на вкладку "Browse"
4. Найдите "Gml.Web.Api.EndpointSDK"
5. Нажмите "Install"

## Создание эндпоинта

Для создания эндпоинта необходимо создать класс, реализующий интерфейс `IPluginEndpoint`. Имя класса может быть любым,
но рекомендуется выбирать имя, отражающее функциональность эндпоинта.

### Базовая структура эндпоинта

```c#
using System.Threading.Tasks;
using Gml.Web.Api.EndpointSDK;
using GmlCore.Interfaces;
using Microsoft.AspNetCore.Http;

namespace YourNamespace;

[Path("get", "/api/v1/your/endpoint/path", true)]
public class YourEndpoint : IPluginEndpoint
{
    public async Task Execute(HttpContext context, IGmlManager gmlManager)
    {
        // Ваш код обработки запроса
        var endpoint = new EndpointHelper();
        await endpoint.Ok(context, new { Message = "Hello, World!" }, "Успешно");
    }
}
```

### Атрибут Path

Атрибут `Path` используется для определения маршрута эндпоинта и имеет следующую структуру:

```c#
[Path(string? method, string? path, bool needAuth)]
```

Параметры:

- `method` - HTTP-метод (get, post, put, delete и т.д.)
- `path` - URL-путь эндпоинта
- `needAuth` — флаг, указывающий, требуется ли аутентификация для доступа к эндпоинту

Пример:

```c#
[Path("get", "/api/v1/plugins/template", true)]
```

## Работа с запросами и ответами

SDK предоставляет класс `EndpointHelper`, который упрощает обработку запросов и формирование ответов.

### Получение данных из запроса

Для получения данных из тела запроса используйте метод `ParseDto<T>`:

```c#
var model = await _endpointHelper.ParseDto<YourModel>(context);
```

Где `YourModel` — класс, соответствующий структуре данных в запросе.

### Формирование ответов

`EndpointHelper` предоставляет несколько методов для формирования HTTP-ответов:

```c#
// Успешный ответ (200 OK)
await endpoint.Ok(context, data, message);

// Созданный ресурс (201 Created)
await endpoint.Created(context, data, message);

// Ошибка запроса (400 Bad Request)
await endpoint.BadRequest(context, data, message);

// Ресурс не найден (404 Not Found)
await endpoint.NotFound(context, message);
```

Параметры:

- `context` - объект HttpContext
- `data` — данные для включения в ответ (может быть null)
- `message` — сообщение для пользователя

## Примеры использования

### Пример 1: Получение данных

```c#
using System.Threading.Tasks;
using Gml.Web.Api.EndpointSDK;
using GmlCore.Interfaces;
using Microsoft.AspNetCore.Http;
using Newtonsoft.Json;

namespace Gml.Web.Api.Plugins.Template;

[Path("get", "/api/v1/plugins/template", true)]
public class TemplateEndpoint : IPluginEndpoint
{
    public async Task Execute(HttpContext context, IGmlManager gmlManager)
    {
        var profiles = await gmlManager.Profiles.GetProfiles();
        var endpoint = new EndpointHelper();

        await endpoint.Ok(context, profiles, "Супер");
    }
}
```

### Пример 2: Работа с хранилищем данных

```c#
using Gml.Web.Api.EndpointSDK;
using Gml.Web.Api.OAuth2.Storage;
using GmlCore.Interfaces;
using Microsoft.AspNetCore.Http;

namespace Gml.Web.Api.OAuth2;

[Path("get", "/api/v1/plugins/gamervii/oauth", true)]
public class OAuthEndpointCallback : IPluginEndpoint
{
    private EndpointHelper _endpointHelper = new();

    public async Task Execute(HttpContext context, IGmlManager gmlManager)
    {
        // Получение данных из хранилища с установкой значения по умолчанию, если данные отсутствуют
        var configuration = await gmlManager.Storage.GetAsync<OmniAuthSettings?>(StorageConstants.AuthSettingsKey) ?? new OmniAuthSettings
        {
            ApplicationId = "RecloudID",
            AppSecret = "RecloudSecret",
            Icon = "https://mirror.recloud.tech/mail/logo.png",
            Label = "Войти через Recloud ID",
            Options = new ClientOptions
            {
                Site = "https://oauth.recloud.tech",
                AuthorizeUrl = "/connect/authorize",
                UserInfoUrl = "/connect/userinfo",
                TokenUrl = "/connect/token",
            },
            Structure = new ResponseStructure
            {
                IdPath = ["sub"],
                Attributes = new Dictionary<string, string>
                {
                    {"email", "email"},
                    {"name", "name"},
                }
            },
            AuthorizeParameters = new AuthorizeParameters
            {
                Scope = "openid profile email"
            }
        };

        await _endpointHelper.Ok(context, configuration, string.Empty);
    }
}
```

### Пример 3: Обновление данных и обработка ошибок

```c#
using Gml.Web.Api.EndpointSDK;
using Gml.Web.Api.OAuth2.Storage;
using GmlCore.Interfaces;
using Microsoft.AspNetCore.Http;

namespace Gml.Web.Api.OAuth2;

[Path("put", "/api/v1/plugins/gamervii/oauth", true)]
public class OAuthEndpointUpdateCallback : IPluginEndpoint
{
    private EndpointHelper _endpointHelper = new();

    public async Task Execute(HttpContext context, IGmlManager gmlManager)
    {
        try
        {
            // Получение данных из запроса
            var model = await _endpointHelper.ParseDto<OmniAuthSettings>(context);

            // Сохранение данных в хранилище
            await gmlManager.Storage.SetAsync(StorageConstants.AuthSettingsKey, model);

            // Отправка успешного ответа
            await _endpointHelper.Ok(context, model, string.Empty);
        }
        catch (Exception exception)
        {
            // Логирование ошибки
            gmlManager.BugTracker.CaptureException(exception);

            // Отправка ответа с ошибкой
            await _endpointHelper.Ok(context, null, exception.Message);
        }
    }
}
```

## Работа с хранилищем данных

GML предоставляет простой механизм для хранения и получения данных через интерфейс `IGmlManager.Storage`.

### Сохранение данных

```c#
await gmlManager.Storage.SetAsync(key, value);
```

### Получение данных

```c#
var data = await gmlManager.Storage.GetAsync<T>(key);
```

Рекомендуется хранить ключи в константах для удобства использования:

```c#
namespace YourNamespace.Storage;

internal abstract class StorageConstants
{
    public const string YourSettingsKey = "YOUR_SETTINGS_KEY";
}
```

## Валидация данных

`EndpointHelper` предоставляет метод для валидации данных с использованием FluentValidation:

```c#
if (!_endpointHelper.IsValidDto<YourModel, YourModelValidator>(model, out var validationResult))
{
    await EndpointHelper.BadRequest(context, validationResult.Errors, "Ошибка валидации");
    return;
}
```

## Обработка ошибок

Для обработки ошибок рекомендуется использовать блок try-catch и логировать ошибки с помощью `gmlManager.BugTracker`:

```c#
try
{
    // Ваш код
}
catch (Exception exception)
{
    gmlManager.BugTracker.CaptureException(exception);
    await _endpointHelper.Ok(context, null, exception.Message);
}
```

## Лучшие практики

1. **Структурируйте код**: Разделяйте логику на отдельные методы для улучшения читаемости и поддерживаемости.
2. **Используйте константы**: Храните строковые ключи и пути в константах.
3. **Обрабатывайте ошибки**: Всегда оборачивайте код в блок try-catch для обработки исключений.
4. **Валидируйте входные данные**: Используйте валидацию для проверки входных данных.
5. **Документируйте API**: Добавляйте комментарии к коду и документируйте публичные API.
6. **Следуйте соглашениям об именовании**: Используйте понятные имена для классов, методов и переменных.

## Заключение

Gml.Web.Api.EndpointSDK предоставляет мощный и гибкий инструментарий для разработки бэкенд-части модулей GML. Следуя
рекомендациям и примерам из этого руководства, вы сможете быстро и эффективно создавать собственные эндпоинты для
расширения функциональности GML.
