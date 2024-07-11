## Простое серверное приложение на Python с использованием Flask и SQLite
Шаги:
  1.Установка зависимостей
  2.Создание базы данных
    2.1 Установка необходимых библиотек
    2.2 Создание экземпляра Flask приложения
    2.3 Настройки базы данных SQLite
    2.4 Инициализация SQLAlchemy с нашим Flask приложением
    2.5 Определение модели User для взаимодействия с пользовательскими данными
      2.5.1 Определение модели User для взаимодействия с пользовательскими данными
      2.5.2 Установка хэша пароля
      2.5.3 Проверка пароля
    2.6 Создание всех необходимых таблиц в базе данных  
  3. Реализация функций регистрации и аутентификации пользователей
    3.1 Регистрация пользователя
      3.1.1 Создание маршрута для регистрации нового пользователя
      3.1.2 Проверка, существует ли уже такой пользователь
      3.1.3 Создание нового пользователя
    3.2 Аутентификация пользователя 
      3.2.1 Создание маршрута для аутентификации пользователя 
      3.2.2 Проверка, существует ли пользователь и корректны ли у него данные
  4. Обработка запросов для CRUD операций с пользователями
    4.1Создание маршрута для получения списка всех пользователей  
    4.2 Добавление нового пользователя
      4.2.1 Создание маршрута для добавления нового пользователя
      4.2.2 Проверка, существует ли уже такой пользователь
      4.2.3 Создание нового пользователя
    4.3 Обновление информации о пользователе 
      4.3.1 Создание маршрута для обновления информации о пользователе
      4.3.2 Проверка, существует ли пользователь
      4.3.3 Обновление информации о пользователе
    4.4 Создание маршрута для удаления пользователя 
  5. Обеспечение минимальной защиты


### 1. Установка зависимостей
Выполните установку необходимых библиотек:

pip install flask flask_sqlalchemy passlib

### 2. Создание базы данных
   
from flask import Flask, request, jsonify
from flask_sqlalchemy import SQLAlchemy
from passlib.hash import pbkdf2_sha256


     # Cоздаем экземпляр Flask приложения
app = Flask(__name__)

#Настройки базы данных SQLite
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///users.db'
app.config['SQLALCHEMY_TRACK_MODIFICATIONS'] = False

    # Инициализируем SQLAlchemy с нашим Flask приложением
db = SQLAlchemy(app)

    # Определяем модель User для взаимодействия с пользовательскими данными
class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    password_hash = db.Column(db.String(120), nullable=False)


    # Метод для установки хэша пароля
    def set_password(self, password):
        self.password_hash = pbkdf2_sha256.hash(password)

    # Метод для проверки пароля
    def check_password(self, password):
        return pbkdf2_sha256.verify(password, self.password_hash)

    # Создаем все необходимые таблицы в базе данных
with app.app_context():
    db.create_all()


### 3. Реализация функций регистрации и аутентификации пользователей

    # Маршрут для регистрации нового пользователя
@app.route('/register', methods=['POST'])
def register():
    data = request.get_json()
    username = data['username']
    password = data['password']

    # Проверяем, существует ли уже такой пользователь
    if User.query.filter_by(username=username).first():
        return jsonify({"message": "User already exists"}), 400

    # Создаем нового пользователя
    new_user = User(username=username)
    new_user.set_password(password)
    db.session.add(new_user)
    db.session.commit()

    return jsonify({"message": "User created successfully"}), 201

    # Маршрут для аутентификации пользователя
@app.route('/login', methods=['POST'])
def login():
    data = request.get_json()
    username = data['username']
    password = data['password']

    # Проверяем, существует ли пользователь и корректны ли у него данные
    user = User.query.filter_by(username=username).first()
    if user and user.check_password(password):
        return jsonify({"message": "Login successful"}), 200
    else:
        return jsonify({"message": "Invalid credentials"}), 401


### 4. Обработка запросов для CRUD операций с пользователями

    # Маршрут для получения списка всех пользователей
@app.route('/users', methods=['GET'])
def get_users():
    users = User.query.all()
    users_list = [{"id": user.id, "username": user.username} for user in users]
    return jsonify(users_list), 200

    # Маршрут для добавления нового пользователя
@app.route('/users', methods=['POST'])
def add_user():
    data = request.get_json()
    username = data['username']
    password = data['password']

    # Проверяем, существует ли уже такой пользователь
    if User.query.filter_by(username=username).first():
        return jsonify({"message": "User already exists"}), 400

    # Создаем нового пользователя
    new_user = User(username=username)
    new_user.set_password(password)
    db.session.add(new_user)
    db.session.commit()

    return jsonify({"message": "User added successfully"}), 201

    # Маршрут для обновления информации о пользователе
@app.route('/users/<int:user_id>', methods=['PUT'])
def update_user(user_id):
    data = request.get_json()
    user = User.query.get(user_id)

    # Проверяем, существует ли пользователь
    if not user:
        return jsonify({"message": "User not found"}), 404

    # Обновляем информацию о пользователе
    user.username = data.get('username', user.username)
    if 'password' in data:
        user.set_password(data['password'])

    db.session.commit()
    return jsonify({"message": "User updated successfully"}), 200

    # Маршрут для удаления пользователя
@app.route('/users/<int:user_id>', methods=['DELETE'])
def delete_user(user_id):
    user = User.query.get(user_id)

    # Проверяем, существует ли пользователь
    if not user:
        return jsonify({"message": "User not found"}), 404

    # Удаляем пользователя
    db.session.delete(user)
    db.session.commit()
    return jsonify({"message": "User deleted successfully"}), 200
### 5. Обеспечение минимальной защиты
Использование параметризованных запросов (производится автоматически с использованием SQLAlchemy).
Хэширование паролей (реализовано с помощью passlib).
