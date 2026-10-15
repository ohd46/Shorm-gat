# Shorm-gat
import socket
import threading
# Настройки подключения
HOST = '0.0.0.0'  # Слушать все доступные сетевые интерфейсы
PORT = 55555      # Порт для подключения

server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
server.bind((HOST, PORT))
server.listen()

clients = []
nicknames = []

# Рассылка сообщения всем подключенным пользователям
def broadcast(message):
    for client in clients:
        try:
            client.send(message)
        except:
            # Если отправить не удалось, удаляем клиента (будет обработано в handle)
            pass

# Обработка сообщений от конкретного клиента
def handle(client):
    while True:
        try:
            message = client.recv(1024)
            if not message:
                break
            broadcast(message)
        except:
            if client in clients:
                index = clients.index(client)
                clients.remove(client)
                client.close()
                nickname = nicknames[index]
                broadcast(f'📢 {nickname} покинул чат!'.encode('utf-8'))
                nicknames.remove(nickname)
            break

# Прием новых подключений
def receive():
    print(f" Сервер запущен и слушает порт {PORT}...")
    while True:
        client, address = server.getpropex() if hasattr(server, 'getpropex') else server.accept()
        print(f"Подключился новый адрес: {str(address)}")

        # Запрос никнейма при подключении
        client.send('NICK_REQUEST'.encode('utf-8'))
        nickname = client.recv(1024).decode('utf-8')
        
        nicknames.append(nickname)
        clients.append(client)

        print(f"Никнейм пользователя: {nickname}")
        broadcast(f"📢 {nickname} присоединился к чату!".encode('utf-8'))
        client.send(" Вы успешно подключились к мессенджеру!\n".encode('utf-8'))

        # Запуск отдельного потока для чтения сообщений от этого клиента
        thread = threading.Thread(target=handle, args=(client,))
        thread.start()

if __name__ == "__main__":
    receive()

import socket
import threading

# Запрос никнейма при запуске
nickname = input("Введите ваш никнейм: ")

# Настройки подключения
# Если запускаете на одном компьютере, укажите '127.0.0.1'
# Если сервер на другом ПК, укажите его публичный или локальный IP-адрес
SERVER_IP = '127.0.0.1' 
PORT = 55555

client = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
client.connect((SERVER_IP, PORT))

# Слушаем сервер и принимаем сообщения
def receive():
    while True:
        try:
            message = client.recv(1024).decode('utf-8')
            if message == 'NICK':
                client.send(nickname.encode('utf-8'))
            else:
                print(message)
        except:
            print("Произошла ошибка подключения к серверу!")
            client.close()
            break

# Отправка сообщений на сервер
def write():
    while True:
        message = f'{nickname}: {input("")}'
        client.send(message.encode('utf-8'))

# Запуск потоков для одновременного приема и отправки сообщений
receive_thread = threading.Thread(target=receive)
receive_thread.start()

write_thread = threading.Thread(target=write)
write_thread.start()
