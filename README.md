
# 📚 Простое серверное приложение на Python с использованием Flask и SQLite
Это простое веб-приложение на Flask для управления пользователями с использованием базы данных SQLite.

## Установка зависимостей

 Выполните установку необходимых библиотек:

  ```bash
   pip install flask flask_sqlalchemy passlib

## Создание базы данных

### Установка необходимых библиотек

```bash
from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy
from passlib.hash import pbkdf2_sha256```

### Cоздаем экземпляр Flask приложения

``` app = Flask(__name__)```

### Настройки базы данных SQLite

```bash
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///users.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False```

### Инициализируем SQLAlchemy с нашим Flask приложением
```db = SQLAlchemy(app)```

### 2.5 Определяем модель User для взаимодействия с пользовательскими данными

```bash
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    password_hash = db.Column(db.String(120), nullable=False)```

#### 2.5.1 Метод для установки хэша пароля

```bash
def set_password(self, password):
        self.password_hash = pbkdf2_sha256.hash(password)```

#### 2.5.2 Метод для проверки пароля

```bash
def check_password(self, password):
        return pbkdf2_sha256.verify(password, self.password_hash)```

### 2.6 Создание всех необходимых таблиц в базе данных

```bash
with app.app_context():
    db.create_all()```

## 3. Реализация функций регистрации и аутентификации пользователей

### 3.1 Регистрация пользователя
#### 3.1.1 Создание маршрута для регистрации нового пользователя

```bash
@app.route('/register', methods=['POST'])
def register():
    data = request.get_json()
    username = data['username']
    password = data['password'] ```

#### 3.1.2 Проверка, существует ли уже такой пользователь

```bash
if User.query.filter_by(username=username).first():
        return jsonify({"message": "User already exists"}), 400```

#### 3.1.3 Создание нового пользователя

```bash
new_user = User(username=username)
new_user.set_password(password)
db.session.add(new_user)
db.session.commit()
return jsonify({"message": "User created successfully"}), 201```

### 3.2 Аутентификация пользователя

#### 3.2.1 Создание маршрута для аутентификации пользователя

```bash
@app.route('/login', methods=['POST'])
def login():
    data = request.get_json()
    username = data['username']
    password = data['password']```

#### 3.2.2 Проверка, существует ли пользователь и корректны ли у него данные

```bash
user = User.query.filter_by(username=username).first()
    if user and user.check_password(password):
        return jsonify({"message": "Login successful"}), 200
    else:
        return jsonify({"message": "Invalid credentials"}), 401```

## 4. Обработка запросов для CRUD операций с пользователями

### 4.1 Создание маршрута для получения списка всех пользователей

```bash
@app.route('/users', methods=['GET'])
def get_users():
    users = User.query.all()
    users_list = [{"id": user.id, "username": user.username} for user in users]
    return jsonify(users_list), 200```

### 4.2 Добавление нового пользователя

#### 4.2.1 Создание маршрута для добавления нового пользователя
```bash
@app.route('/users', methods=['POST'])
def add_user():
    data = request.get_json()
    username = data['username']
    password = data['password'] ```

#### 4.2.1 Проверяем, существует ли уже такой пользователь

```bash
if User.query.filter_by(username=username).first():
        return jsonify({"message": "User already exists"}), 400``` 


#### 4.2.2 Создание нового пользователя 

```bash
new_user = User(username=username)
new_user.set_password(password)
db.session.add(new_user)
db.session.commit()
return jsonify({"message": "User added successfully"}), 201``` 


### 4.3 Обновление информации о пользователе

```bash
user.username = data.get('username', user.username)
if 'password' in data:
    user.set_password(data['password'])

db.session.commit()
return jsonify({"message": "User updated successfully"}), 200``` 

#### 4.3.1 Создание маршрута для обновления информации о пользователе
```bash
@app.route('/users/<int:user_id>', methods=['PUT'])
def update_user(user_id):
    data = request.get_json()
    user = User.query.get(user_id)``` 

#### 4.3.2 Проверка, существует ли пользователь

```bash
if not user:
    return jsonify({"message": "User not found"}), 404``` 


#### 4.3.2 Обновление информации о пользователе

```bash
user.username = data.get('username', user.username)
if 'password' in data:
    user.set_password(data['password'])
db.session.commit()
return jsonify({"message": "User updated successfully"}), 200``` 


### 4.4 Удаление пользователя

#### 4.4.1 Создание маршрута для удаления пользователя

```bash
user.username = data.get('username', user.username)
if 'password' in data:
    user.set_password(data['password'])
db.session.commit()
return jsonify({"message": "User updated successfully"}), 200``` 


#### 4.4.2 Проверка, существует ли пользователь

```bash
if not user:
    return jsonify({"message": "User not found"}), 404``` 


#### 4.4.3 Удаление пользователя

```bash
db.session.delete(user)
db.session.commit()
return jsonify({"message": "User deleted successfully"}), 200``` 
 
Отправьте DELETE запрос на `/users/{user_id}`.

## 5. Маршруты

- `/register` - Регистрация нового пользователя.
- `/login` - Аутентификация пользователя.
- `/users` - Получение списка всех пользователей и добавление новых.
- `/users/{user_id}` - Обновление и удаление пользователя по его ID.

## 6. Защита приложения

Приложение защищено от SQL-инъекций с использованием SQLAlchemy ORM и от хэширования паролей с помощью библиотеки passlib.

## ⚙️ Запуск

Запустите приложение командой:

```bash
python app.py
```

Приложение будет доступно по адресу `http://localhost:5000`.

## 📦 Зависимости

- Flask
- Flask-SQLAlchemy
- passlib

