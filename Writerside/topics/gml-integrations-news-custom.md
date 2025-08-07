# Импорт новостей с внешнего ресурса

В данном разделе описывается API для импорта новостей с внешнего ресурса. Этот API предоставляет доступ к новостям через
REST интерфейс.

## API Endpoint

### GET /api/news

Этот эндпоинт возвращает список новостей.

#### Параметры запроса

| Параметр | Тип    | Обязательный | Описание                                     |
|----------|--------|--------------|----------------------------------------------|
| limit    | number | Нет          | Ограничение количества возвращаемых новостей |
| offset   | number | Нет          | Смещение для пагинации                       |

#### Формат ответа

```json
[
  {
    "id": 1,
    "title": "Заголовок новости",
    "description": "Содержание новости",
    "createdAt": "2024-01-01T12:00:00.000Z"
  },
  {
    "id": 2,
    "title": "Другая новость",
    "description": "Текст другой новости",
    "createdAt": "2024-01-02T15:30:00.000Z"
  }
]
```

#### Описание полей ответа

| Поле        | Тип    | Описание                         |
|-------------|--------|----------------------------------|
| id          | number | Уникальный идентификатор новости |
| title       | string | Заголовок новости                |
| description | string | Полный текст новости             |
| createdAt   | string | Дата публикации в формате ISO    |

## Примеры использования

### JavaScript (Fetch API)

```javascript
// Пример получения новостей с использованием Fetch API
async function getNews(limit = 10, offset = 0) {
    try {
        const response = await fetch(`https://your-api.com/api/news?limit=${limit}&offset=${offset}`);

        if (!response.ok) {
            throw new Error(`HTTP error! Status: ${response.status}`);
        }

        const newsItems = await response.json();

        console.log(`Получено ${newsItems.length} новостей`);

        // Обработка полученных новостей
        newsItems.forEach(news => {
            console.log(`${news.title} (${new Date(news.createdAt).toLocaleDateString()})`);
            console.log(news.description);
            console.log('---');
        });

        return newsItems;
    } catch (error) {
        console.error('Ошибка при получении новостей:', error);
        return null;
    }
}

// Использование
getNews(5, 0).then(result => {
    // Дополнительная обработка результата
});
```

### PHP с базой данных

```php
<?php
// Пример получения и сохранения новостей в базу данных

// Функция для получения новостей из API
function fetchNews($limit = 10, $offset = 0) {
    $url = "https://your-api.com/api/news?limit={$limit}&offset={$offset}";

    $ch = curl_init($url);
    curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
    curl_setopt($ch, CURLOPT_TIMEOUT, 30);

    $response = curl_exec($ch);

    if (curl_errno($ch)) {
        echo 'Ошибка cURL: ' . curl_error($ch);
        return null;
    }

    curl_close($ch);

    return json_decode($response, true);
}

// Функция для сохранения новостей в базу данных
function saveNewsToDatabase($newsItems) {
    // Параметры подключения к базе данных
    $host = 'localhost';
    $dbname = 'news_database';
    $username = 'root';
    $password = 'password';

    try {
        // Подключение к базе данных
        $pdo = new PDO("mysql:host={$host};dbname={$dbname}", $username, $password);
        $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

        // Подготовка запроса для вставки или обновления новостей
        $stmt = $pdo->prepare("
            INSERT INTO news (external_id, title, description, created_at)
            VALUES (:external_id, :title, :description, :created_at)
            ON DUPLICATE KEY UPDATE
            title = :title,
            description = :description,
            created_at = :created_at
        ");

        // Обработка каждой новости
        $count = 0;
        foreach ($newsItems as $news) {
            $stmt->execute([
                ':external_id' => $news['id'],
                ':title' => $news['title'],
                ':description' => $news['description'],
                ':created_at' => $news['createdAt']
            ]);
            $count++;
        }

        echo "Успешно сохранено {$count} новостей\n";
        return true;

    } catch (PDOException $e) {
        echo "Ошибка базы данных: " . $e->getMessage() . "\n";
        return false;
    }
}

// Создание таблицы (выполняется один раз при настройке)
function createNewsTable() {
    $host = 'localhost';
    $dbname = 'news_database';
    $username = 'root';
    $password = 'password';

    try {
        $pdo = new PDO("mysql:host={$host};dbname={$dbname}", $username, $password);
        $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);

        $pdo->exec("
            CREATE TABLE IF NOT EXISTS news (
                id INT AUTO_INCREMENT PRIMARY KEY,
                external_id INT UNIQUE,
                title VARCHAR(255) NOT NULL,
                description TEXT NOT NULL,
                created_at DATETIME NOT NULL,
                updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP
            )
        ");

        echo "Таблица новостей успешно создана\n";
        return true;

    } catch (PDOException $e) {
        echo "Ошибка создания таблицы: " . $e->getMessage() . "\n";
        return false;
    }
}

// Использование
createNewsTable(); // Выполняется один раз при настройке
$newsItems = fetchNews(20, 0);

if ($newsItems && is_array($newsItems)) {
    saveNewsToDatabase($newsItems);
} else {
    echo "Не удалось получить данные новостей\n";
}
?>
```

### C#

```csharp
using System;
using System.Collections.Generic;
using System.Net.Http;
using System.Text.Json;
using System.Threading.Tasks;

namespace NewsApiClient
{
    // Класс для десериализации ответа API
    public class NewsItem
    {
        public int Id { get; set; }
        public string Title { get; set; }
        public string Description { get; set; }
        public DateTime CreatedAt { get; set; }
    }

    // Класс для работы с API новостей
    public class NewsApiClient
    {
        private readonly HttpClient _httpClient;
        private readonly string _baseUrl;

        public NewsApiClient(string baseUrl)
        {
            _httpClient = new HttpClient();
            _baseUrl = baseUrl;
        }

        public async Task<List<NewsItem>> GetNewsAsync(int limit = 10, int offset = 0)
        {
            try
            {
                string url = $"{_baseUrl}/api/news?limit={limit}&offset={offset}";
                HttpResponseMessage response = await _httpClient.GetAsync(url);

                response.EnsureSuccessStatusCode();

                string responseBody = await response.Content.ReadAsStringAsync();
                var options = new JsonSerializerOptions
                {
                    PropertyNameCaseInsensitive = true
                };

                List<NewsItem> newsItems = JsonSerializer.Deserialize<List<NewsItem>>(responseBody, options);
                return newsItems;
            }
            catch (HttpRequestException e)
            {
                Console.WriteLine($"Ошибка HTTP запроса: {e.Message}");
                return null;
            }
            catch (JsonException e)
            {
                Console.WriteLine($"Ошибка десериализации JSON: {e.Message}");
                return null;
            }
            catch (Exception e)
            {
                Console.WriteLine($"Общая ошибка: {e.Message}");
                return null;
            }
        }
    }

    // Пример использования
    class Program
    {
        static async Task Main(string[] args)
        {
            string apiUrl = "https://your-api.com";
            var client = new NewsApiClient(apiUrl);

            var newsItems = await client.GetNewsAsync(5, 0);

            if (newsItems != null && newsItems.Count > 0)
            {
                Console.WriteLine($"Получено {newsItems.Count} новостей");

                foreach (var news in newsItems)
                {
                    Console.WriteLine($"ID: {news.Id}");
                    Console.WriteLine($"Заголовок: {news.Title}");
                    Console.WriteLine($"Дата: {news.CreatedAt.ToLocalTime()}");
                    Console.WriteLine($"Содержание: {news.Description}");
                    Console.WriteLine(new string('-', 50));
                }
            }
            else
            {
                Console.WriteLine("Не удалось получить новости");
            }
        }
    }
}
```

## Интеграция с GML

Для интеграции API новостей с GML:

1. Убедитесь, что ваш API запущен и доступен по сети
2. В панели управления GML перейдите в раздел интеграций
3. Выберите "Импорт новостей с внешнего ресурса"
4. Укажите URL вашего API, который возвращает новости в формате, описанном выше
5. Сохраните настройки

После успешной интеграции новости будут автоматически отображаться в лаунчере GML. Обратите внимание, что GML ожидает
получить массив объектов с полями `id`, `title`, `description` и `createdAt`. Если ваш API возвращает данные в другом
формате, вам потребуется создать промежуточный слой для преобразования данных.
