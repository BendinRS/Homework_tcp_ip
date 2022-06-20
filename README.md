# Домашняя работа "Разворачиваем сетевую лабораторию"  


<details>
  <summary>Введение </summary>

Сеть — очень важная составляющая в работе серверов. По сети сервера взаимодействуют между собой.
В данном домашнем задании мы рассмотрим технологии маршрутизации и NAT.  
**Маршрутизация** — выбор оптимального пути передачи пакетов. Для маршрутизации используется таблица маршрутизации. Основная задача маршрутизации — доставить пакет по указанному IP-адресу.  
Если одно устройство имеет сразу несколько подсетей, например в сервере есть 2 порта с адресами:  
+ 192.168.1.10/24
+ 10.10.12.72/24
то, такие сети называются непосредственно подключенными (Directly connected networks). Маршрутизация между Directrly Connected сетями происходит автоматически. Дополнительная настройка не потребуется. Если необходимая сеть удалена, маршрутизатор будет искать через какой порт она будет доступна, если такой порт не найден, то трафик уйдет на шлюз по умолчанию.  

**Маршрутизация бывает статическая и динамическая.**

При использовании статической маршрутизации администратор сам создаёт правила для маршрутов. Плюсом данного метода будет являться безопасность, так как статические маршруты не обновляются по сети, а минусом — сложности при работе с сетями больших объёмов.  
Динамическая маршрутизация подразумевает построение маршрутов автоматически с помощью различных протоколов (RIP,OSPF,BGP, и.т.д.). Маршрутизаторы сами обмениваются друг с другом информацией о сетях и автоматически прописывают маршруты.  

**NAT** — это процесс, используемый для преобразования сетевых адресов.
Основные цели NAT:  
+ Экономия публичных IPv4-адресов
+ Повышение степени конфиденциальности и безопасности сети.
NAT обычно работает на границе, где локальная сеть соединяется с сетью Интернет. Когда устройству сети потребуется подключение к устройству вне его сети (например в Интернете), пакет пересылается маршрутизатору с NAT, а маршрутизатор преобразовывает его внутренний адрес в публичный.  
</details>

<details>
<summary>Цели</summary>

+ Создать домашнюю сетевую лабораторию.   
+ Научиться менять базовые сетевые настройки в Linux-based системах.
</details>


<details>
<summary>Описание</summary>

1. Скачать и развернуть Vagrant-стенд (https://github.com/erlong15/otus-linux/tree/network)  

2. Построить следующую сетевую архитектуру:  
Сеть office1  
- 192.168.2.0/26 - dev
- 192.168.2.64/26 - test servers
- 192.168.2.128/26 - managers
- 192.168.2.192/26 - office hardware
Сеть office2  
- 192.168.1.0/25 - dev
- 192.168.1.128/26 - test servers
- 192.168.1.192/26 - office hardware
Сеть central  
- 192.168.0.0/28 - directors
- 192.168.0.32/28 - office hardware
- 192.168.0.64/26 - wifi
Итого должны получиться следующие сервера:  
● inetRouter  
● centralRouter  
● office1Router  
● office2Router  
● centralServer  
● office1Server  
● office2Server  

Задание состоит из 2-х частей: теоретической и практической.  

В теоретической части требуется:
● Найти свободные подсети  
● Посчитать количество узлов в каждой подсети, включая свободные  
● Указать Broadcast-адрес для каждой подсети  
● Проверить, нет ли ошибок при разбиении  
В практической части требуется:  
● Соединить офисы в сеть согласно логической схеме и настроить роутинг  
● Интернет-трафик со всех серверов должен ходить через inetRouter  
● Все сервера должны видеть друг друга (должен проходить ping)  
● У всех новых серверов отключить дефолт на NAT (eth0), который vagrant поднимает для связи  
● Добавить дополнительные сетевые интерфейсы, если потребуется  

Рекомендуется использовать Vagrant + Ansible для настройки данной схемы.  

</details>

<details>
<summary>Теоретическая часть</summary>

+ Найти свободные подсети
+ Посчитать количество узлов в каждой подсети, включая свободные
+ Указать Broadcast-адрес для каждой подсети
+ Проверить, нет ли ошибок при разбиении

| Name  |Network|Netmask| N  | Hostmin  | Hostmax  |  Broadcast |
|---|---|---|---|---|---|---|
|   |   |   | **Central Network**   |   |   |   |
| Directors  | 192.168.0.0/28  | 255.255.255.240   | 14  | 192.168.0.1   | 192.168.0.14   | 192.168.0.15  |
| Office hardware  | 192.168.0.32/28   | 255.255.255.240   | 14  | 192.168.0.33   | 192.168.0.46   | 192.168.0.47  |
| Wifi  | 192.168.0.64/26  | 255.255.255.192   | 62  | 192.168.0.65   | 192.168.0.126   | 192.168.0.127  |
|   |   |   | **Office 1 network**  |   |   |   |
|  Dev | 192.168.2.0/26   | 255.255.255.192   | 62  | 192.168.2.1   | 192.168.2.62   | 192.168.2.63  |
| Test  | 192.168.2.64/26   | 255.255.255.192  |  62 | 192.168.2.65   |  192.168.2.126  | 192.168.2.127  |
|Managers   | 192.168.2.128/26   | 255.255.255.192  | 62  | 192.168.2.129   | 192.168.2.190   | 192.168.2.191  |
| Office hardware  | 192.168.2.192/26   | 255.255.255.192  | 62  | 192.168.2.193   |  192.168.2.254  | 192.168.2.255  |
|   |   |   | **Office 2 network**  |   |   |   |
| Dev  | 192.168.1.0/25   | 255.255.255.128  | 126  | 192.168.1.1   | 192.168.1.126  | 192.168.1.127  |
| Test  | 192.168.1.128/26   | 255.255.255.192  | 62  |  192.168.1.129  |  192.168.1.190 | 192.168.1.191  |
|  Office | 192.168.1.192/26   | 255.255.255.192  | 62  | 192.168.1.193   | 192.168.1.254  | 192.168.1.255  |
|   |   |   |  **InetRouter — CentralRouter network** |   |   |   |
| Inet-central  | 192.168.255.0/30  | 255.255.255.252   |  2 |  192.168.255.1  |  192.168.255.2  | 192.168.255.3  |

Свободные сети:  

+ 192.168.0.16/28
+ 192.168.0.48/28
+ 192.168.0.128/25
+ 192.168.255.64/26
+ 192.168.255.32/27
+ 192.168.255.16/28
+ 192.168.255.8/29
+ 192.168.255.4/30

</details>