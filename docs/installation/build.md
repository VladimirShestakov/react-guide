# Сборка

Для публикации проекта на сервере сначала выполняется сборка приложения командой:
```
npm run build
```
Собранные и минимизированные файлы приложения помещаются в папку `/dist/web`.  На http сервере директория `/dist/web` должны быть публичной. 
Основные файлы в ней - это `index.html` и `main.js` Ниже есть пример настройки сервера nginx.

## Сборка для SSR
SSR - это рендер приложения на стороне сервера, чтобы клиент по первому запросу получал готовый html с данными. В первую очередь для обработки поисковиками. В последнюю для сокращения времени первого отображения. 

Чтобы фронтенд приложение запускать на сервере, его нужно собрать специально для ssr командой:  
```
npm run build:ssr
```

Сборка для SSR не отменяет обычной сборки, так как сервер все равно должен отдавать фронтенд приложение клиенту. Поэтому можно воспользоваться командой сборки всего:
```
npm run build:all
```
 
В директории ./dist/node будут созданы файлы серверного приложения. Приложение на сервере можно запустить командой:
```
node ./server.js 
```
В server.js подключаются собранные файлы из `./dist/node`.

Можно одной командой выполнить сборку всего и запуск приложения с ssr:
```
npm run start:ssr
```
Сборка и запуск SSR по умолчанию работает в режиме продакшена. Как именно устроен SSR подробно рассматривается в соответствующем разделе документации.