# SELinux
H/W SELinux - когда все запрещено 
1. **Запуск nginx на нестандартном порту 3-мя разными способами**
2.   ``` vagrant up ```  разворачиваем виртуальную машину
3.   Проверяем nginx  и режим работы SELinux. Должен отображаться режим Enforcing. Данный режим означает, что SELinux будет блокировать запрещенную активность.   
4.    ![alt text](./Pictures/1.png)
5. Заходим в файл конфигурации nginx и меняем порт на 4881 -  ``` nano /etc/nginx/nginx.conf ```  Запустить на таком порту не получается.
6. Далее выполняем команду  ``` audit2why < /var/log/audit/audit.log  ```, для того чтобы понять что включать.
7. Утилита audit2why отображает комманду для выполнения  ``` setsebool -P nis_enabled 1  ```   
8.   ![alt text](./Pictures/2.png)
9.   После этого сервер nginx запускается
10.    ![alt text](./Pictures/3.png)
11.    ![alt text](./Pictures/4.png)
12.  Вернем запрет работы nginx на 4881 порту   ``` setsebool -P nis_enabled off ```   После этого сервер nginx не запустится.
13.  **Теперь разрешим в SELinux работу nginx на порту TCP 4881 c помощью добавления нестандартного порта в имеющийся тип**
14.   Выполним комманду  ``` semanage port -l | grep http  ``` что бы узнать какие порты открыты для http
15.   ![alt text](./Pictures/5.png)
16.   Добавляем наш не стандартный порт  ``` semanage port -a -t http_port_t -p tcp 4881  ```
17.   Запускаем ngnix и проверяем что все работает
18.   ![alt text](./Pictures/6.png)
19.    Удалим наш не стандартный порт  ``` semanage port -d -t http_port_t -p tcp 4881  ```
20. **Разрешим в SELinux работу nginx на порту TCP 4881 c помощью формирования и установки модуля SELinux**
21.    ``` systemctl start nginx  ``` -  не запускается , так как SELinux продолжает его блокировать. Посмотрим логи SELinux, которые относятся к nginx
22. Посмотрим логи SELinux, которые относятся к nginx   ``` grep nginx /var/log/audit/audit.log  ```
23.  Воспользуемся утилитой audit2allow для того, чтобы на основе логов SELinux сделать модуль, разрешающий работу nginx на нестандартном порту
24.   ``` grep nginx /var/log/audit/audit.log | audit2allow -M nginx  ```
25.   Audit2allow сформировала команду  ``` semodule -i nginx.pp  ``` после выполнения которой можно запускать nginx и проверять status
26.    ![alt text](./Pictures/7.png)
## Обеспечение работоспособности приложения при включенном SELinux
1.  Выполним клонирование репозитория:  ``` git clone https://github.com/mbfx/otus-linux-adm.git  ```  **vagrant up** из данного каталога развернет стенд с 2-мя виртуальными машинами
2.  Подключаемся к клиенту ``` vagrant ssh client ```
3.   Попробуем внести изменения в зону: ``` nsupdate -k /etc/named.zonetransfer.key ```

```nsupdate -k /etc/named.zonetransfer.key
nsupdate -k /etc/named.zonetransfer.key
server 192.168.50.10
zone ddns.lab 
update add www.ddns.lab. 60 A 192.168.50.15
send 
```

