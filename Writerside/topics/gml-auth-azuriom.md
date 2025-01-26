# Авторизация Azuriom

Так как Gml эмитирует полную работу лицензионной версии игры, а
Azuriom генерирует неверные UUID для пользователей, то необходимо произвести некоторые действия:

- Откройте файл ```/папка_с_сайтом/app/Games/Minecraft/MinecraftOfflineGame.php```, и найдите в нем следующий код:

```php
public function getUserUniqueId(string $name): ?string
    {
        $factory = new UuidFactory();
        $factory->setNameGenerator(new class implements NameGeneratorInterface
        {
            public function generate(UuidInterface $ns, string $name, string $hashAlgorithm): string
            {
                return md5($name, true);
            }
        });
        $uuid = $factory->uuid3(Uuid::NIL, 'OfflinePlayer:'.$name)->toString();

        return Str::remove('-', $uuid);  // < --- Вот эту строчку
    }
```

- Замените ```return Str::remove('-', $uuid);``` на ```return $uuid;```.

> **ВНИМАНИЕ:** После этой процедуры необходимо пересоздать все уже имеющиеся аккаунты, чтобы сгенерировались новые и
> корректные UUID.
> Для этого необходимо удалить аккаунты пользователей.