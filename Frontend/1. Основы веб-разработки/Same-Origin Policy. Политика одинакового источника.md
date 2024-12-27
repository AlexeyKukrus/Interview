**Same-Origin Policy (SOP)** — это механизм безопасности в веб-браузерах, который ограничивает взаимодействие между веб-страницами, чтобы предотвратить несанкционированный доступ к данным. Этот механизм предотвращает выполнение потенциально опасных действий между ресурсами с разных источников.

SOP запрещает JavaScript на веб-странице взаимодействовать с ресурсами, если они не принадлежат одному источнику. Это касается доступа к DOM, чтения и модификации данных, выполнения запросов и использования API.

**Исключения:**
 1. **CORS (Cross-Origin Resource Sharing):**
	[[CORS (Cross-Origin Resource Sharing)|Подробнее про CORS]]
	Сервер может явно разрешить доступ из другого источника, добавив в ответ HTTP-заголовки, такие как:
	`Access-Control-Allow-Origin: *` или `Access-Control-Allow-Origin: https://example.com`.
	**Пример:**
```http
HTTP/1.1 200 OK Access-Control-Allow-Origin: https://another.com
```

2. **WebSockets** 
	[[Websocket|Подробнее про WebSocket]]
	WebSockets не подчиняются SOP, но требуют проверки происхождения при установке соединения.
	Серверы WebSocket используют HTTP-запрос для установления соединения (`Handshake`), и этот запрос содержит заголовок `Origin`.
	Сервер должен самостоятельно проверять заголовок `Origin`, чтобы определить, разрешено ли устанавливать соединение с данным клиентом. Это важно для предотвращения атак, таких как Cross-Site WebSocket Hijacking (CSWSH).
	**Пример:**
```JavaScript
const WebSocket = require('ws');
const server = new WebSocket.Server({ port: 8080 });

server.on('connection', (ws, req) => {
    const origin = req.headers.origin;

    if (origin !== 'https://trusted-origin.com') {
        ws.close(4001, 'Unauthorized origin');
        return;
    }

    ws.on('message', (message) => {
        console.log(`Received: ${message}`);
        ws.send('Hello!');
    });
});

```
