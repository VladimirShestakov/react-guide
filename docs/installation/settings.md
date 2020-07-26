# Настройка

Параметры сборки определяются в файле `webpack.config.js`. По умолчанию учтены режимы разработки/продакшена, сборки для серверного рендера, импорта файлов с jsx, less, картинок, шрифтов, прокси для апи (обхода CORS).

Конфигурация самого приложения определяется в файле `src/config.js`. По умолчанию определены параметры АПИ, роутинга. Здесь же параметры сервера разработки и рендера. В config.js можно добавлять свои параметры приложения.

Кроме настройки приложения и его сборки для продакшена потребуется настроить http сервер. Рекомендуется использовать nginx.
Nginx должен отдавать статичные файлы из `./dist/web`, а если url не на существующий файл, то отдавать `./dist/web/index.html`.

Пример настройки nginx

```
server {
    listen 80;
    server_name react-skeleton.com;
    location / {
        root /home/user/react-skeleton/dist/web;
        try_files $uri /index.html;
    }
}
```

Если планируется использовать серверный рендер (SSR), то потребуется запустить приложение на node.js и вместо отдачи index.html из nginx проксировать запросы в серверное приложение.

```
server {
    listen 80;
    server_name react-skeleton.com;
    location / {
        root /home/user/react-skeleton/dist/web;
        try_files $uri @ssr;
    }

    location @ssr {
        proxy_redirect off;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP         $remote_addr;
        proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Frame-Options   SAMEORIGIN;
        proxy_pass http://127.0.0.1:8132;
    }
}
```
Для постоянной работы приложения рендера воспользуетесь менеджером процессов pm2. 
```
pm2 start process.json 
```

## Алиасы путей

В файле `package.js` кроме типовых свойств проекта прописаны алиасы на директории, чтобы импортировать их из любого файла через @ вместо указания относительных путей. Алиасы также поймет IDE WebStorm.

```
"_moduleAliases": {
    "@api": "./src/api/",
    "@app": "./src/app/",
    "@components": "./src/components/",
    "@containers": "./src/containers/",
    "@store": "./src/store/",
    "@theme": "./src/theme",
    "@utils": "./src/utils/"
}

import Button from "@components/button"
```
