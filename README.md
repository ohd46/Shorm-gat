# Shorm-gat
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
