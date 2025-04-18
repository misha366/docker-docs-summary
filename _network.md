1. Запрос отправляется от клиента на IP адрес нашей машины; например, 93.184.216.34 и порт (например, 80)
2. Запрос идёт по маршрутизаторам и доходит до нашего компьютера
3. Сетевая карта принимает Enthernet frame - блок данных
4. Из фрейма извлекается IP-пакет — форматированный блок информации, передаваемый по компьютерной сети, структура которого определена протоколом IP
5. Из пакета извлекается TCP сегмент (порт назначение; например, номер порта - 80)
    - порт - абстракция - его не существует физически - просто числовой маркер, который отслеживает ОС. После того как мы обращаемся к какому либо порту, ос передаёт данные в нужный процесс. Порты своего рода **маркеры**, мы договорились о том на какие порты, какие данные передавать, мы будем передавать http запрос только на 80 порт, потому что только на 80 порту будет процесс который этот пакет сможет обработать. *Если бы не было портов, ОС бы не знала в какой процесс передавать пакеты*;
    - другими словами - порт - заранее обговорённое число, за которым лежит нужный нам процесс, который обработает наши данные так, как мы этого ожидаем; без портов нам было бы очень сложно находить этот процесс
    
    - для sql нужен отдельный порт, потому что sql сервер это отдельный процесс, что слушает сетевой порт; Backend общается с БД через сетевой протокол (даже локально). Другими словами sql это всегда отдельный сервер, даже если он расположен на той же машине что и бэкенд.

6. ОС смотрит какой процесс слушает 80. Например, если nginx заранее зарегистрировал порт 80, то ос передаст данные ему. В нашем случае докер проводит манипуляции, при помощи которых контейнер получает пакет так, будто он пришёл сразу на порт к контейнеру, а не приходил до этого к нам на машину
    - все входящие запросы отслеживаються ядром ос (а не приложением, в нашем случае nginx). Под каждое соединение создаётся **сокет**, который хранится в ядре и представляет собой наше соединение с клиентом, хранит в себе информацию про клиента и данные. Соединения можно увидеть в реальном времени - `ss -tanp`
    - Сам nginx особенный, он использует event систему, благодаря которой всё работает внутри одного потока/процесса; где даже несколько одновременных событий, **соединений** управляються в одном потоке
7. Nginx будет дёргать php-fpm для формирования ответа. Php-fpm - несколько работающих процессов, которые ждут задач от веб сервера.
    - всё настраивается в `www.conf`
    - если например, макс. число потоков 10, а пхп запросов 20, то 10 обработаются сразу, а 10 будут ждать своей очереди
```plaintext
[ Клиенты ] 
    ↓ ↓ ↓
┌───────────────┐
│    NGINX      │
│ (epoll model) │  ← 1 воркер обрабатывает 1000 сокетов
└─────┬─────────┘
      ↓ FastCGI
┌───────────────┐
│   PHP-FPM     │
│ [pool: www]   │
│ proc  proc    │ ← пул: заранее запущенные процессы
│  #1    #2     │
│  #3   ...     │
└───────────────┘
```
8. php-fpm сформирует запросы и отдаст nginx, а тот в свою очередь отправит их обратно клиентам
