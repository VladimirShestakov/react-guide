# Сервер SSR

Сервер для рендера реализован в скрипте [`/server.js`](https://github.com/ylabio/react-skeleton/blob/master/server.js) 
по классическому шаблону [express](https://expressjs.com/ru) приложения. 

К express приложению добавлена типовая логика отдачи статических файлов из сборки для веба `'/dist/web'` (на продакшене их отдаёт Nginx), 
и парсер куков. В куке может передаваться ключ состояния, хотя можно учитывать и токены авторизации, чтобы 
рендерить персональные страницы.

*server.js*
```js
const app = express();

// Отдача файлов кроме index.html
app.use(express.static(path.resolve('./dist/web'), { index: false, /*dotfiles: 'allow'*/ }));
app.use(cookieParser());
```

Основная задача сервера рендера - обработать запросы к страницам сайта, отрендерить страницу фронтенд приложения и вернуть 
html страницы в ответ на запрос. Для этого реализован общий роутер на все страницы. 

*server.js*
```js
app.get('/*', async (req, res) => {
  try {
    // Ключи для временного хранения состояния
    const stateKey = uniqid();
    const stateSecret = uniqid(stateKey);

    // Запуск ренедра с передачей параметров запроса
    const result = await render({
      method: req.method,
      url: req.url,
      headers: req.headers,
      body: req.body,
      cookies: req.cookies,
      stateKey: stateKey,
    });

    // Запоминаем состояние на 15сек, чтобы его успел запросить клиент
    stateStorage[stateKey] = { [stateSecret]: result.state };
    setTimeout(() => {
      if (stateKey in stateStorage) {
        delete stateStorage[stateKey];
      }
    }, 15000);

    // В куке второй ключ состояния для защиты
    res.cookie('stateSecret', stateSecret, { expires: false, httpOnly: true/*, secure: true*/ });

    // Отправляем сгенерированную страницу к кодом статуса, обычно 200
    res.writeHead(result.status, { 'Content-Type': 'text/html; charset=utf-8' });
    res.end(result.out);
  } catch (e) {
    // Если были ошибки рендера, отправляем код 500
    console.error(e);
    res.writeHead(500, { 'Content-Type': 'text/html; charset=utf-8' });
    res.end(`ERROR ${e.toString()}`);
  }
});
// Запуск сервера - прослушивание запросов на указанном хосте и порте
app.listen(config.ssr.port, config.ssr.host);
```

В обработчике запроса вызывается функция `render()` с передачей адреса страницы и других параметров запроса. 
В ней же запускается собранное для Node.js фронтенд приложение из `/dist/node/main.js`. 
Для запуска используется Worker (дочерний процесс) - фактически на каждый запрос запускается отдельное изолированное приложение.

> Дело в том, что архитектура фронтенд приложения не учитывает множественный запуск, в нем глобальные объекты store, 
> слоя апи, объекта навигации и другие - их нельзя изолировать через замыкание. Запуск через Worker позволяет сохранить 
> привычную архитектуру фронтенда.

*server.js*
```js
function render(params) {
  return new Promise((resolve, reject) => {
    const worker = new Worker('./dist/node/main.js', { workerData: params, stdout: false, stderr: false });
    worker.on('message', resolve);
    //...
  });
}
```

В воркер передаются параметры запроса и сгенерированный ключ для состояния. Ключ потом просто добавляется в html разметку. 



Воркер вернет результат рендера в виде строки с html `result.out` и http код ответа `result.status`. Кроме этого будет 
возарщен объект состания, с которым выполнялся рендер `result.state`. Это объект redux хранилища. 
Объект состояния запоминается по случайном уникальному ключу на 15 секунд. 
В рендере html будет указан ключ состояния, по нему фронтенд сможет запросить объект начального состояния. 
Учитывается ещё второй уникальный ключ состояния в куках для защиты от несанкионированного доступа. 

Чтобы сервер рендера отдавал объект состояния по запросу с ключем в адресе и в куке, реализован роутер `/ssr/state/:key`

*server.js*
```js
let stateStorage = {};

app.get('/ssr/state/:key', async (req, res) => {
  const stateKey = req.params.key;
  const stateSecret = req.cookies.stateSecret;
  if (stateStorage[stateKey] && stateStorage[stateKey][stateSecret]) {
    res.json(stateStorage[stateKey][stateSecret]);
    delete stateStorage[stateKey];
  } else {
    res.json({});
  }
});
```


В рамках серверного приложения реализован прокси сервер запросов к АПИ. АПИ обычно находится на другом сервере. 
При этом фронтенд приложение шлет запросы на тот же домен, где и развернуто. Запросы идут на свой домен, чтобы миновать 
проблемы кросс доменных запросов, но уже на сервере запросы нужно проксировать к домену апи. 
Лучше, если запросы проксируются через Nginx, но если приложение запущено локально без Nginx, то запросы будут проксироваться серверным рендером. 

*server.js*
```js
// Прокси на внешний сервер по конфигу (обычно для апи)
const proxy = httpProxy.createProxyServer({});
for (const path of Object.keys(config.api.proxy)) {
  console.log(`Proxy ${path} => ${config.api.proxy[path].target}`);
  app.all(path, async (req, res) => {
    try {
      proxy.web(req, res, config.api.proxy[path]);
    } catch (e) {
      console.error(e);
      res.send(500);
    }
  });
}
```

Используется тот же конфиг путей прокси, что и вебпаком. Вебпак тоже умеет поднимать прокси сервер, но он используется 
локально в режиме разработки без SSR. 

*src/config.js*
```js
let config = {
  // Параметры сервера рендера
  ssr: {
    host: 'localhost',
    port: 8132,
  }, 
  //..
  api: {
    //..
    // Прокси на апи, используется в webpack-dev-server и в server.js
    proxy: {
      '/api/**': {
        target: 'http://example.front.ylab.io',
        secure: true,
        changeOrigin: true,
      },
    },
  }
}
```
