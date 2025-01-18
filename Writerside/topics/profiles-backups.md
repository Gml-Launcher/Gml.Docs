# Бекапирование и перенос

В данной статье рассказывается, как перенести установленный проект Gml на другой сервер.
Первым делом, необходимо определить и найти директорию установки, куда производилась инсталяция Gml.

Содержимое папки выглядит следующим образом:

![profiles-backups-1.png](profiles-backups-1.png)

## 1. Остановка сервиса

Перед началом работы вам необходимо остановить сервис Gml, если вы следовали
установке [из установщика ](server-install-from-script.md),
то в корневой директории Gml достаточно прописать команду:

```bash
docker compose stop
```

В случае, если вы [устанавливали на Windows](server-install-from-script-windows.md) необходимо остановить или убить
процесс

```Bash
kill PID
```

Где `PID` - ID процесса Gml

## 2. Упаковка файла

Вам необходимо запаковать папку `data`, которую **вы и должны перенести** на другой сервер, для этого воспользуйтесь
командой
в корневой папке Gml.

<tabs>
    <tab id="Linux-install" title="Linux">
<code-block lang="Bash" >
zip -r data.zip ./data
</code-block>

Данная команда создаст архив `data.zip` в текущей директории, содержащий файлы из папки `data`.
</tab>
<tab id="Windows-install" title="Windows">

<code-block lang="PowerShell" >
Compress-Archive -Path .\data -DestinationPath data.zip
</code-block>

Эта команда создаст архив `data.zip` в текущей директории с файлами из папки `data`.
</tab>
<tab id="MacOS-install" title="MacOS">

<code-block lang="Bash" >
zip -r data.zip ./data
</code-block>

Команда создаст архив `data.zip` в текущей директории, содержащий файлы из папки `data`.
</tab>
</tabs>

## 3. Перенос на другой сервер

После того как вы сделали `data.zip` файл, вам необходимо выполнить [установку Gml](server-install-home.topic) на новом
сервере,
после успешной установки серверной части остановите сервер командой:

```bash
docker compose stop
```

В случае, если вы [устанавливали на Windows](server-install-from-script-windows.md) необходимо просто остановить или
убить процесс

```Bash
kill PID
```

и поместить папку data на место старой.

## 4. Запуск

После того как вы подменили папку `data`, запустите Gml.Backend командой:

```bash
docker compose up -d
```

В случае, если вы [устанавливали на Windows](server-install-from-script-windows.md) необходимо запустить приложение
штатными средствами Windows