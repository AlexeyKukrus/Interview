#### Авторизация на основе JWT (JSON Web Token)
***Основной процесс***:
    - Клиент отправляет учетные данные (например, логин и пароль) на сервер.
    - Сервер проверяет их и, если успешны, генерирует JWT, содержащий информацию о пользователе и время истечения токена (Access Token).
    - Токен отправляется клиенту, который сохраняет его (лучше всего использовать`sessionStorage` потому что у него короткое время жизни и его не обязательно хранить длительное время, а так же легко записывать и извлекать по аналогии с `localStorage`)
    - Для каждого последующего запроса клиент добавляет токен в заголовок `Authorization: Bearer <token>`.
    - Для короткоживущих токенов (`Access Token`) сервер возвращает дополнительный `Refresh Token` (его лучше хранить в `HttpOnly coockie` так как это позволяет защитить пользователя от атак).
	- `Access Token` используется для запросов, а `Refresh Token` — для получения нового `Access Token` после его истечения

***Использование JWT на фронтенде:***
```javascript
// Авторизация пользователя 
async function login(username: string, password: string): Promise<void> { 
	const response = await fetch('/api/auth/login', { 
		method: 'POST', 
		headers: { 'Content-Type': 'application/json' }, 
		body: JSON.stringify({ username, password }), 
	}); 
	if (!response.ok) throw new Error('Login failed'); 
	const { accessToken, refreshToken } = await response.json(); 
	
	// Хранение токенов 
	sessionStorage.setItem('accessToken', accessToken); // Access Token в сессии
	document.cookie = `refreshToken=${refreshToken}; HttpOnly; Secure; SameSite=Strict`;
};

// Обновление Access токена
async function refreshAccessToken(): Promise<void> {
  const response = await fetch('/api/auth/refresh', {
    method: 'POST',
    credentials: 'include', // Отправляет HttpOnly cookies с запросом
  });

  if (!response.ok) throw new Error('Failed to refresh token');
  const { accessToken } = await response.json();

  sessionStorage.setItem('accessToken', accessToken); // Обновляем токен в хранилище
}

//Использование Access токена
async function fetchWithAuth(url: string, options: RequestInit = {}): Promise<Response>{
  const accessToken = sessionStorage.getItem('accessToken');

  const response = await fetch(url, {
    ...options,
    headers: {
      ...options.headers,
      Authorization: `Bearer ${accessToken}`,
    },
  });

  // Если Access Token истёк, обновляем его
  if (response.status === 401) {
    await refreshAccessToken();
    return fetchWithAuth(url, options); // Повторяем запрос с новым токеном
  }

  return response;
}

//Выход из системы и удаление токенов
async function logout(): Promise<void> { 
	await fetch('/api/auth/logout', { 
		method: 'POST', 
		credentials: 'include', // Удаляет HttpOnly cookies на сервере 
	}); 
	
	sessionStorage.removeItem('accessToken'); // Удаляем токен из памяти 
	document.cookie = 'refreshToken=; Max-Age=0;'; // Удаляем cookie 
}
```

#### Что такое OAuth 2.0?

OAuth 2.0 — это способ дать приложению доступ данным пользователя на другом сайте, **не передавая пароль**.

***Основные участники OAuth 2.0:***
1. **Ресурсный владелец (Resource Owner)**: 
    - Пользователь, чьи данные защищены.
    - Пример: Пользователь, который разрешает приложению доступ к своим данным на Google.
2. **Клиент (Client)**:
    - Приложение, которое запрашивает доступ к данным пользователя.
    - Пример: Веб-приложение, которое хочет получить доступ к контактам пользователя.
3. **Сервер авторизации (Authorization Server)**: 
    - Утверждает запросы доступа от клиента и выдает токены.
    - Пример: Сервер Google OAuth.
4. **Ресурсный сервер (Resource Server)**:
    - Содержит защищенные данные, доступ к которым контролируется через токены.
    - Пример: Google API для получения списка контактов.

***Основной процесс***
1.  Сайт просит разрешение авторизоваться с помощью аккаунта на стороннем ресурсе
2.  Google спрашивает разрешение предоставить доступ к определенным персональным данным. Если дать согласие, Google перенаправляет пользователя обратно на сайт с временным кодом:
```
https://мойсайт.ру/callback?code=AUTH_CODE
```
3. Сайт получает токен, отправив запрос на сервер Google
```
// Запрос
POST https://accounts.google.com/o/oauth2/token {   
	client_id: 12345,   
	client_secret: секрет_сайта,   
	code: AUTH_CODE,   
	redirect_uri: https://мойсайт.ру/callback,   
	grant_type: authorization_code 
}

// Ответ
{   
	"access_token": "ACCESS_TOKEN", // действует 1 час
	"refresh_token": "REFRESH_TOKEN",
	"expires_in": 3600 
}
```
4. Далее происходит стандартное взаимодействие с использованием `Access token` и его обновлением каждый час с помощью `Refresh token`