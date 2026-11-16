<!--
Бланк звіту до лабораторної роботи «TLS та сертифікати».
Скопіювати цей файл у власний публічний репозиторій звіту як README.md і заповнити.
Вивід команд вставляти текстом у блоки коду, а не знімками екрана.
Порожні поля не допускаються: якщо щось не вдалося, записати, що саме і чому.
-->

# Лабораторна робота 2. TLS та сертифікати

| Поле | Значення |
|---|---|
| ПІБ | Василіогло Микола Сергійович|
| Група |F5-2.01 |
| Ідентифікатор |s04 |
| Дата виконання | 16.11.2026 |
| Середовище | GitHub Codespaces |
| Версія OpenSSL | OpenSSL 3.5.8 25 Aug 2026 |

## Параметри варіанта

<!-- Переписати з картки stand/cards/<id>.md. -->

| Параметр | Значення |
|---|---|
| Віддалений сервер | edge.s04.lab|
| Імена для сертифіката | `www.s04.lab`, `shop.s04.lab` |
| Ваш сервер (IP) | 10.53.1.1 |

<!-- Якщо стенд перезапускався під час роботи, вказати, після якого завдання. -->

## Завдання 1. HTTP і HTTPS у дампі трафіку


Вивід кроку 1.1:

```text
[1] 41
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
server: web
Host: edge.s04.lab
SNI: 
protocol: HTTP/1.1 
client: 10.53.1.1
10 packets captured
10 packets received by filter
0 packets dropped by kernel
[1]+  Exit 124                timeout 8 tcpdump -ni eth0 -A -s0 -l 'host 10.53.0.80 and tcp port 80' > t1-http.txt
14:13:16.699919 IP 10.53.1.1.50844 > 10.53.0.80.80: Flags [P.], seq 1:76, ack 1, win 502, options [nop,nop,TS val 990543285 ecr 3830633205], length 75: HTTP: GET / HTTP/1.1
}..R..GET / HTTP/1.1
Host: edge.s04.lab
server: web
Host: edge.s04.lab
root@s04:/lab/work# 

```

Вивід кроку 1.2:

```text
[1] 97
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
server: web
Host: edge.s04.lab
SNI: edge.s04.lab
protocol: HTTP/2.0 TLSv1.3
client: 10.53.1.1
21 packets captured
21 packets received by filter
0 packets dropped by kernel
[1]+  Exit 124                timeout 8 tcpdump -ni eth0 -A -s0 -l 'host 10.53.0.80 and tcp port 443' > t1-https.txt
0
1
```

| Що шукали в дампі | HTTP | HTTPS |
|---|---|---|
| рядок запиту `GET /` | GET / HTTP/1.1 | не видно, зашифровано |
| заголовок `Host` | edge.s04.lab | не видно зашифровано |
| тіло відповіді (`server: web`) | server: web | не видно зашифровао |
| ім'я віддаленого сервера (кількість входжень) | 2 | 1 |

1. У якому повідомленні і в якому полі передається ім'я сервера, яке видно в дампі HTTPS?
Ім'я сервера передається у першому ж повідомленні від клієнта до сервера ClientHello. Воно записане відкритим текстом у розширенні SNI, у полі server_name
2. Чому рядок `SNI:` у відповіді сервера порожній для HTTP і заповнений для HTTPS?
Бо SNI від протоколу TLS, а не самого HTTP. При звичайному HTTP-запиті ніякого TLSрукостискання немає, тому SNI порожній, а сервер дізнається потрібний домен із HTTP-заголовка Host. У HTTPS спочатку йде TLS, де клієнт через SNI одразу каже серверу, до якого саме сайту стукає, щоб сервер знав, який сертифікат показати.
3. Рядок `protocol:` показує HTTP/1.1 для запиту по HTTP і HTTP/2.0 для запиту по HTTPS, хоча обидва запити виконано однією командою `curl` без додаткових ключів. Що визначило версію HTTP у другому випадку?
Версію протоколу визначило розширення Application-Layer Protocol Negotiation під час TLSрукостискання. curl у ClientHello запропонував серверу на вибір h2 та http/1.1, а сервер підтримав більш сучасний h2. У звичайному HTTP такого узгодження немає, тому там за замовчуванням працює HTTP1.1.

## Завдання 2. Рукостискання по записах

Вивід кроку 2.1 (TLS 1.3):

```text
Sent Record
  Content Type = Handshake (22)
    ClientHello, Length=323
Received Record
  Content Type = Handshake (22)
    ServerHello, Length=118
Received Record
  Content Type = ChangeCipherSpec (20)
Received Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    EncryptedExtensions, Length=15
Received Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    Certificate, Length=975
Received Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    CertificateVerify, Length=74
Received Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    Finished, Length=48
Sent Record
  Content Type = ChangeCipherSpec (20)
Sent Record
  Content Type = ApplicationData (23)
  Inner Content Type = Handshake (22)
    Finished, Length=48
Sent Record
  Content Type = ApplicationData (23)
  Inner Content Type = Alert (21)
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
ALPN protocol: h2
Verify return code: 0 (ok)
```

Вивід кроку 2.2 (TLS 1.2):

```text
Sent Record
  Content Type = Handshake (22)
    ClientHello, Length=218
Received Record
  Content Type = Handshake (22)
    ServerHello, Length=74
Received Record
  Content Type = Handshake (22)
    Certificate, Length=970
Received Record
  Content Type = Handshake (22)
    ServerKeyExchange, Length=112
Received Record
  Content Type = Handshake (22)
    ServerHelloDone, Length=0
Sent Record
  Content Type = Handshake (22)
    ClientKeyExchange, Length=33
Sent Record
  Content Type = ChangeCipherSpec (20)
Sent Record
  Content Type = Handshake (22)
    Finished, Length=12
Received Record
  Content Type = Handshake (22)
    NewSessionTicket, Length=214
Received Record
  Content Type = ChangeCipherSpec (20)
Received Record
  Content Type = Handshake (22)
    Finished, Length=12
Received Record
  Content Type = ApplicationData (23)
Sent Record
  Content Type = Alert (21)
New, TLSv1.2, Cipher is ECDHE-ECDSA-AES256-GCM-SHA384
ALPN protocol: h2
    Verify return code: 0 (ok)
```

<!-- Один рядок на кожен запис з виводу 2.1, у тому самому порядку. Напрям: К→С або С→К. -->

| № | Напрям | `Content Type` | `Inner Content Type` | Повідомлення | `Length` |
|---|---|---|---|---|---|
| 1 | К→С | Handshake (22) | — | ClientHello | 323 |
| 2 | С→К | Handshake (22) | — | ServerHello | 118 |
| 3 | С→К | ChangeCipherSpec (20) | — | — | — |
| 4 | С→К | ApplicationData (23) | Handshake (22) | EncryptedExtensions | 15 |
| 5 | С→К | ApplicationData (23) | Handshake (22) | Certificate | 975 |
| 6 | С→К | ApplicationData (23) | Handshake (22) | CertificateVerify | 74 |
| 7 | С→К | ApplicationData (23) | Handshake (22) | Finished | 48 |
| 8 | К→С | ChangeCipherSpec (20) | — | — | — |
| 9 | К→С | ApplicationData (23) | Handshake (22) | Finished | 48 |
| 10 | К→С | ApplicationData (23) | Alert (21) | — | — |

Вивід кроків 2.3 і 2.4:

```text
 extension_type=server_name(0), length=17
          0000 - 00 0f 00 00 0c 65 64 67-65 2e 73 30 34 2e 6c   .....edge.s04.l
          000f - 61 62                                          ab
        extension_type=ec_point_formats(11), length=4

[1] 82
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
21 packets captured
21 packets received by filter
0 packets dropped by kernel
[1]+  Exit 124                timeout 6 tcpdump -ni eth0 -A -s0 -l 'host 10.53.0.80 and tcp port 443' > t2-dump-$v.txt
TLS 1.3: 0
[1] 88
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
21 packets captured
21 packets received by filter
0 packets dropped by kernel
[1]+  Exit 124                timeout 6 tcpdump -ni eth0 -A -s0 -l 'host 10.53.0.80 and tcp port 443' > t2-dump-$v.txt
TLS 1.2: 2
```
| Показник | TLS 1.3 | TLS 1.2 |
|---|---|---|
| Довжина ClientHello, байт | 323 | 218 |
| Номер першого запису, тип якого спостерігач не бачить | 4 | — (усі типи записів у TLS 1.2 залишаються відкритими) |
| Кількість RTT від ClientHello до першого HTTP-запиту | 1 | 2 |
| Чи видно сертифікат у дампі (результат 2.4) | Ні (`TLS 1.3: 0`) | Так (`TLS 1.2: 2`) |
| Обраний протокол ALPN | `h2` | `h2` |

1. За якими рядками виводу 2.1 визначено, з якого запису починається шифрування? Що бачить спостерігач замість типу кожного наступного запису?
Шифрування починається з 4го запису . У виводі це видно по тому, що у записів з'являється поле Inner Content Type = Handshake (22), заховане всередині зовнішнього типу Content Type = ApplicationData (23). Сторонній спостерігач тепер не бачить, чи це триває рукостискання, чи йдуть дані сайту, чи закриття з'єднання  — для всіх наступних записів зовні він бачить лише ApplicationData (23)
2. Чому ClientHello у TLS 1.3 довший, ніж у TLS 1.2? Назвіть розширення, яке за рахунок цього скорочує рукостискання на один RTT.
У нашому замірі ClientHello для TLS 1.3 займає 323 байти, а для TLS 1.2 218 байт. У TLS 1.3 клієнт не чекає поки сервер обере алгоритм і надішле свої параметри, а одразу в першому ж пакеті кидає свою половину ключа шифрування наперед. За це відповідає розширення key_share, завдяки якому рукостискання скорочується з 2 RTT до 1 RTT.
3. Довжина розширення `server_name` у виводі 2.3 становить 17 байт для імені з 12 символів. З яких частин складається це значення і яким воно буде для вашого імені?

З байтів у дампі 00 0f 00 00 0c 65 64 67 65 2e 73 30 34 2e 6c 61 62 видно таку структуру:   00 0f (2 байти) — загальна довжина списку імен (15 байт);   00 (1 байт) — тип імені (host_name);   00 0c (2 байти) — довжина самого доменного імені (12 байт);   решта 12 байт — саме ім'я edge.s04.lab у кодуванні ASCII.
У сумі виходить 17 байт. Оскільки ім'я edge.s04.lab якраз має 12 символів, для нього значення дорівнює 17 байт.  

4. Навіщо в TLS 1.3 передається запис ChangeCipherSpec, якщо в рукостисканні він нічого не змінює?

Тільки заради сумісності зі старим мережевим залізом  яке заточене під TLS 1.2. Деякі старі проміжні пристрої просто обривають з'єднання, якщо після ServerHello не бачать пакета ChangeCipherSpec, тому в TLS 1.3 його залишили як формальну пустишку щоб трафік не блокувався по дорозі.

## Завдання 3. Сертифікат для власного сервера

Вивід кроків 3.1 і 3.2:

```text
збережено /lab/tls/acme-root.crt
subject=CN = Pebble Root CA 0e91e4
notBefore=Nov 16 14:44:07 2026 GMT
notAfter=Nov 16 14:44:07 2056 GMT

curl: (60) SSL certificate problem: self-signed certificate
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
```

Вивід кроку 3.3 (останні рядки `lego` і рядки журналу nginx):

```text
…
2026/11/16 14:47:39 [INFO] [www.s04.lab] acme: Trying to solve HTTP-01
2026/11/16 14:47:42 [INFO] [www.s04.lab] The server validated our request
2026/11/16 14:47:42 [INFO] [shop.s04.lab] acme: Trying to solve HTTP-01
2026/11/16 14:47:49 [INFO] [shop.s04.lab] The server validated our request
2026/11/16 14:47:49 [INFO] [www.s04.lab, shop.s04.lab] acme: Validations succeeded; requesting certificates
2026/11/16 14:47:49 [INFO] Wait for certificate [timeout: 30s, interval: 500ms]
2026/11/16 14:47:49 [INFO] [www.s04.lab] Server responded with a certificate.
10.53.0.10 - - [16/Nov/2026:14:47:39 +0000] "GET /.well-known/acme-challenge/6rn-SAYh7aO0-CSWbmGiuMmqh4MK49jVw0q2NY4huJk HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [16/Nov/2026:14:47:39 +0000] "GET /.well-known/acme-challenge/6rn-SAYh7aO0-CSWbmGiuMmqh4MK49jVw0q2NY4huJk HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [16/Nov/2026:14:47:39 +0000] "GET /.well-known/acme-challenge/6rn-SAYh7aO0-CSWbmGiuMmqh4MK49jVw0q2NY4huJk HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [16/Nov/2026:14:47:42 +0000] "GET /.well-known/acme-challenge/7BHgprmmY75ONjo-BUquJxoc5Cp-cpsHJosTDzcQLEE HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [16/Nov/2026:14:47:42 +0000] "GET /.well-known/acme-challenge/7BHgprmmY75ONjo-BUquJxoc5Cp-cpsHJosTDzcQLEE HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"
10.53.0.10 - - [16/Nov/2026:14:47:42 +0000] "GET /.well-known/acme-challenge/7BHgprmmY75ONjo-BUquJxoc5Cp-cpsHJosTDzcQLEE HTTP/1.1" 200 87 "-" "LetsEncrypt-Pebble-VA (linux; amd64)"

```

Вивід кроку 3.4:

```text
subject=CN = www.s04.lab
issuer=CN = Pebble Intermediate CA 173394
notBefore=Nov 16 14:47:49 2026 GMT
notAfter=Nov 16 14:47:48 2031 GMT
serial=20436C11DD952F94
X509v3 Subject Alternative Name: 
    DNS:www.s04.lab, DNS:shop.s04.lab
subject=CN = Pebble Intermediate CA 173394
issuer=CN = Pebble Root CA 0e91e4
root@s04:/lab/work/lego/certificates# 
```

Вивід кроку 3.5:

```text
nginx: the configuration file /lab/nginx.conf syntax is ok
nginx: configuration file /lab/nginx.conf test is successful
2026/11/16 14:50:49 [notice] 48#48: signal process started
curl: (60) SSL certificate problem: unable to get local issuer certificate
More details here: https://curl.se/docs/sslcerts.html

curl failed to verify the legitimacy of the server and therefore could not
establish a secure connection to it. To learn more about this situation and
how to fix it, please visit the web page mentioned above.
Certificate chain
 0 s:CN = www.s04.lab
   i:CN = Pebble Intermediate CA 173394
   a:PKEY: id-ecPublicKey, 256 (bit); sigalg: RSA-SHA256
   v:NotBefore: Nov 16 14:47:49 2026 GMT; NotAfter: Nov 16 14:47:48 2031 GMT
---
Verify return code: 21 (unable to verify the first certificate)
root@s04:/lab/work/lego/certificates# 
```

Вивід кроку 3.6:

```text
nginx: the configuration file /lab/nginx.conf syntax is ok
nginx: configuration file /lab/nginx.conf test is successful
2026/11/16 14:51:44 [notice] 56#56: signal process started

server: s04 (https)
Host: www.s04.lab
SNI: www.s04.lab
protocol: TLSv1.3
Certificate chain
 0 s:CN = www.s04.lab
   i:CN = Pebble Intermediate CA 173394
   a:PKEY: id-ecPublicKey, 256 (bit); sigalg: RSA-SHA256
   v:NotBefore: Nov 16 14:47:49 2026 GMT; NotAfter: Nov 16 14:47:48 2031 GMT
 1 s:CN = Pebble Intermediate CA 173394
   i:CN = Pebble Root CA 0e91e4
   a:PKEY: rsaEncryption, 2048 (bit); sigalg: RSA-SHA256
   v:NotBefore: Nov 16 14:44:07 2026 GMT; NotAfter: Nov 16 14:44:07 2056 GMT
---
Verify return code: 0 (ok)
server: s04 (https)
Host: shop.s04.lab
SNI: shop.s04.lab
protocol: TLSv1.3
```

| Параметр | Значення |
|---|---|
| Видавець сертифіката сервера | `CN = Pebble Intermediate CA 173394` |
| Видавець сертифіката видавця | `CN = Pebble Root CA 0e91e4` |
| Імена в SAN | `DNS:www.s04.lab`, `DNS:shop.s04.lab` |
| Строк дії (від – до) | `Nov 16 14:47:49 2026 GMT` – `Nov 16 14:47:48 2031 GMT` |
| Тип ключа сервера і алгоритм підпису | `id-ecPublicKey` (256 біт) і `RSA-SHA256` |
| Кількість сертифікатів, які надсилає сервер, у кроці 3.5 і в кроці 3.6 | У кроці 3.5 — 1 сертифікат; у кроці 3.6 — 2 сертифікати |

1. Хто і як перевірив, що ви контролюєте обидва імена? Вкажіть рядки виводу, які це підтверджують.

Перевірку виконав центр сертифікації Pebble за протоколом ACME методом HTTP-01. Утиліта lego поклала спеціальні токени у корінь нашого вебсервера а центр сертифікації зробив HTTPзапити по порту 80 на кожен із доменів і переконався, що файли доступні.
Це підтверджують рядки з журналу nginx та lego у кроці 3.3

2. Чому в кроці 3.5 перевірка не пройшла, хоча корінь центру сертифікації вказано в `--cacert`, а сертифікат виданий правильно?

Тому що наш сертифікат підписаний не напряму кореневим центром Pebble Root, а проміжним центром Pebble Intermediate. У кроці 3.5 ми дали серверу тільки його власний сертифікат, тому клієнт отримав ланцюжок лише з однієї ланки і не зміг побудувати місток до довіреного кореня

3. У виводі ланцюжка для сертифіката сервера вказано `id-ecPublicKey` і `RSA-SHA256`. Що означає кожне з цих значень і чому вони не суперечать одне одному?

id-ecPublicKey — це тип відкритого ключа самого нашого сервера, який згенерувала утиліта lego.   
RSA-SHA256 — це алгоритм, яким центр сертифікації підписав наш сертифікат своїм ключем.
Вони не суперечать одне одному, бо це ключі різних сторін: алгоритм ключа власника сертифіката може бути яким завгодно , а алгоритм цифрового підпису залежить виключно від ключа центру сертифікації, який цей сертифікат завірив.

4. Що станеться з перевіркою вашого сервера після перезапуску стенда і чому? Що в реальній інфраструктурі відповідає цій ситуації?
Після перезапуску стенда перевірка впаде з помилкою недовіреного центру сертифікації, тому що тестовий центр Pebble при кожному запуску генерує нову кореневу та проміжну пару ключів з нуля, і старий сертифікат перестає збігатися з новим коренем.
У реальній інфраструктурі цьому відповідає ситуація, коли кореневий сертифікат центру сертифікації скомпрометовано і його відкликають/замінюють на новий, або коли у клієнта в системі встановлений застарілий список довірених кореневих сертифікатів, який не знає про новий корінь ЦС.


## Завдання 4. Діагностика несправностей

Вивід кроку 4.1:

```text
=== f1.s04.lab
curl: (60) SSL certificate problem: unable to get local issuer certificate
Certificate chain
 0 s:CN = f1.s04.lab
   i:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   v:NotBefore: Nov 11 06:24:01 2026 GMT; NotAfter: Feb  9 06:24:01 2027 GMT
---
Verify return code: 21 (unable to verify the first certificate)
=== f2.s04.lab
curl: (60) SSL certificate problem: certificate has expired
Certificate chain
 0 s:CN = f2.s04.lab
   i:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   v:NotBefore: Apr 25 06:24:01 2026 GMT; NotAfter: Oct 22 06:24:01 2026 GMT
 1 s:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   i:O = OPISM Lab, CN = OPISM Stand Root CA
   v:NotBefore: Nov 11 06:24:01 2026 GMT; NotAfter: Nov 10 06:24:01 2031 GMT
---
Verify return code: 10 (certificate has expired)
=== f3.s04.lab
curl: (60) SSL certificate problem: unable to get local issuer certificate
Certificate chain
 0 s:CN = f3.s04.lab
   i:O = Foreign Org, CN = Foreign Intermediate CA
   v:NotBefore: Nov 11 06:24:01 2026 GMT; NotAfter: Feb  9 06:24:01 2027 GMT
 1 s:O = Foreign Org, CN = Foreign Intermediate CA
   i:O = Foreign Org, CN = Foreign Root CA
   v:NotBefore: Nov 11 06:24:01 2026 GMT; NotAfter: Nov 10 06:24:01 2031 GMT
---
Verify return code: 20 (unable to get local issuer certificate)
=== f4.s04.lab
curl: (60) SSL certificate problem: self-signed certificate
Certificate chain
 0 s:CN = f4.s04.lab
   i:CN = f4.s04.lab
   v:NotBefore: Nov 11 06:24:01 2026 GMT; NotAfter: Feb  9 06:24:01 2027 GMT
---
Verify return code: 18 (self-signed certificate)
=== f5.s04.lab
server: web, f5
Certificate chain
 0 s:CN = f5.s04.lab
   i:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   v:NotBefore: Nov 11 06:24:02 2026 GMT; NotAfter: Feb  9 06:24:02 2027 GMT
 1 s:O = OPISM Lab, CN = OPISM Stand Intermediate CA
   i:O = OPISM Lab, CN = OPISM Stand Root CA
   v:NotBefore: Nov 11 06:24:01 2026 GMT; NotAfter: Nov 10 06:24:01 2031 GMT
---
Verify return code: 0 (ok)
```

| Сервер | Помилка `curl` | `Verify return code` | Яка перевірка не пройшла | Рядок виводу, що це доводить | Що має змінити власник сервера |
|---|---|---|---|---|---|
| f1 | `unable to get local issuer certificate` | `21 (unable to verify the first certificate)` | Сервер не надсилає проміжний сертифікат | У `Certificate chain` лише 1 запис (`0 s:CN = f1.s04.lab`), немає `1 s:O = OPISM Lab, CN = OPISM Stand Intermediate CA` | Додати проміжний сертифікат видавця у файл ланцюжка сертифіката на сервері |
| f2 | `certificate has expired` | `10 (certificate has expired)` | Прострочений сертифікат | `v:NotBefore: Apr 25 06:24:01 2026 GMT; NotAfter: Oct 22 06:24:01 2026 GMT` (дата вже минула) | Перевипустити (оновити) сертифікат у центрі сертифікації та перезавантажити сервер |
| f3 | `unable to get local issuer certificate` | `20 (unable to get local issuer certificate)` | Сертифікат виданий центром, якому клієнт не довіряє | `i:O = Foreign Org, CN = Foreign Root CA` (клієнт довіряє лише `OPISM Stand Root CA`) | Отримати сертифікат у довіреному центрі сертифікації (`OPISM Stand CA`) або додати `Foreign Root CA` у довірені на клієнті |
| f4 | `self-signed certificate` | `18 (self-signed certificate)` | Самопідписаний сертифікат | `0 s:CN = f4.s04.lab` та `i:CN = f4.s04.lab` (суб'єкт і видавець збігаються) | Замінити самопідписаний сертифікат на сертифікат, підписаний довіреним ЦС |
| f5 | Немає (`server: web, f5`) | `0 (ok)` | Усі перевірки пройшли успішно | `Verify return code: 0 (ok)` | Нічого, сервер налаштований правильно |

1. Для сервера з помилкою `unable to get local issuer certificate` визначте, яка це несправність: сервер не надсилає проміжний сертифікат чи сертифікат виданий центром, якому клієнт не довіряє. Чим ці дві несправності відрізняються у виводі `openssl s_client`?

є обидві ці несправності: на сервері f1 не надіслано проміжний сертифікат, а на сервері f3 сертифікат підписаний чужим кореневим центром якому клієнт не довіряє.
У виводі openssl s_client вони відрізняються так:  
 Коли немає проміжного сертифіката f1: у ланцюжку передається лише один сертифікат 0, його видавцем i вказаний відомий проміжний центр, якого просто немає в ланцюжку, а код повернення — 21 .   
Коли недовірений центр f3: сервер надсилає повний ланцюжок 0 і 1, але на самому верху вказаний невідомий корінь i:O = Foreign Org, CN = Foreign Root CA, якого немає у клієнта, і код повернення — 20 unable to get local issuer certificate.  

2. Якщо серед ваших серверів є сервер з невідповідним ім'ям, чому `openssl s_client` повертає для нього `Verify return code: 0 (ok)`, а `curl` повідомляє про помилку? Якщо такого сервера немає, дайте відповідь для загального випадку.

Серед моїх серверів такого немає, але в загальному випадку так відбувається тому, що openssl s_client за замовчуванням перевіряє лише криптографічний ланцюжок довіри (чи підписаний сертифікат довіреним коренем і чи не вийшов строк дії), але не звіряє ім'я хоста з полем Subject Alternative Name для цього йому потрібен окремий ключ -verify_hostname. Натомість curl як повноцінний HTTPклієнт обов'язково перевіряє, чи збігається домен із запиту з іменами в SAN сертифіката, і відхиляє з'єднання при розбіжності.

3. Які з п'яти серверів відповіли б на запит `curl -k`? Що при цьому втрачається порівняно з `--cacert`?

На запит curl -k (--insecure) відповіли б усі п'ять серверів (f1–f5), оскільки ключ -k наказує клієнту ігнорувати будь-які помилки перевірки сертифіката (строк дії, ланцюжок, невідомий ЦС, самопідписаність, невідповідність імені).
Порівняно з --cacert повністю втрачається автентифікація сервера: трафік усе ще шифрується, але клієнт не має жодної гарантії, що спілкується саме зі справжнім сервером, а не зі зловмисником посередині (атака Man-in-the-Middle), який підсунув свій фальшивий сертифікат.

## Завдання 5. Ціна нового з'єднання

**Прогноз** (записати до вимірювання). Скільки RTT мине до завершення кожного етапу:

| Етап | TLS 1.3, нове з'єднання | TLS 1.2, нове з'єднання | Повторне використання з'єднання |
|---|---|---|---|
| `time_connect` | 1 | 1 | 0 |
| `time_appconnect` |2 |3 | 0|
| `time_starttransfer` |3 |4 |1 |

Вивід кроків 5.2 і 5.3:

```text
0.060961 0.091251 0.125245 0.155874 0.155940 2 1
0.030458 0.060591 0.094322 0.124649 0.124710 2 1
0.030472 0.060655 0.094416 0.124754 0.124812 2 1
0.030486 0.060637 0.124487 0.154783 0.154848 2 1
0.030477 0.060649 0.124713 0.155051 0.155113 2 1
0.030502 0.060656 0.125824 0.156757 0.156817 2 1

0.030430 0.060760 0.094620 0.124970 0.125026 2 1
0.000018 0.000000 0.000000 0.030400 0.030456 2 0
0.000018 0.000000 0.000000 0.030321 0.030478 2 0
```

<!-- Значення в мілісекундах. Для нових з'єднань — медіана трьох вимірювань. -->

| Запит | `namelookup` | `connect` | `appconnect` | `starttransfer` | `total` | `num_connects` |
|---|---|---|---|---|---|---|
| TLS 1.3, нове | 30.47 | 60.66 | 94.42 | 124.75 | 124.81 | 1 |
| TLS 1.2, нове | 30.48 | 60.65 | 124.71 | 155.05 | 155.11 | 1 |
| 5.3, перший запит | 30.43 | 60.76 | 94.62 | 124.97 | 125.03 | 1 |
| 5.3, другий запит | 0.02 | 0.00 | 0.00 | 30.40 | 30.46 | 0 |
| 5.3, третій запит | 0.02 | 0.00 | 0.00 | 30.32 | 30.48 | 0 |

RTT, визначений вимірюванням, і з якого значення його визначено:

RTT = 30 мс, точніше ~30.2–30.4 мс.

Найточніше його видно з різниці етапів TCP-рукостискання connect - namelookup = 60.66 - 30.47 = 30.19 мс, часу одного закешованого DNS-запиту namelookup ≈ 30.47 мс та часу отримання відповіді у вже відкритому з'єднанні starttransfer для 2го і 3го запитів у кроці 5.3 = 30.40 та 30.32 мс.

1. Виразіть кожне виміряне значення через RTT. Де вимірювання відрізняється від цілого числа RTT і чим це пояснюється?

TLS 1.3, нове з'єднання: namelookup ≈ 1 RTT 30.5 мс, connect ≈ 2 RTT 60.7 мс = 1 DNS + 1 TCP, appconnect ≈ 3 RTT + ~3.8 мс (94.4 мс = 1 DNS + 1 TCP + 1 TLS), starttransfer ≈ 4 RTT + ~3.8 мс (124.8 мс).

TLS 1.2, нове з'єднання: namelookup ≈ 1 RTT (30.5 мс), connect ≈ 2 RTT (60.7 мс), appconnect ≈ 4 RTT + ~3.6 мс (124.7 мс = 1 DNS + 1 TCP + 2 TLS), starttransfer ≈ 5 RTT + ~3.6 мс (155.1 мс).

5.3, перший запит: збігається з новим з'єднанням TLS 1.3 (1 RTT, 2 RTT, 3 RTT + ~3.8 мс, 4 RTT + ~3.8 мс).

5.3, другий і третій запити: namelookup ≈ 0 RTT (0.02 мс, IP взято з локального кешу curl), connect = 0 RTT, appconnect = 0 RTT, starttransfer ≈ 1 RTT (30.4 мс — лише сам HTTP-запит і відповідь)

на етапі appconnect додається близько 3.6–3.8 мс понад ціле число RTT. Це час на локальні криптографічні обчислення CPU під час TLSрукостискання
2. Чому `time_namelookup` не близький до нуля, хоча резолвер уже має ім'я в кеші? За яких умов він був би приблизно вдвічі більшим?

Тому що кеш знаходиться не всередині нашого контейнера, а на окремому вузлі resolver-a, а мережева затримка 30 мс повішена на вихідний інтерфейс eth0 нашого контейнера. Тому навіть закешована відповідь вимагає одного повного мережевого запиту UDP до резолвера і назад.
Він був би приблизно вдвічі більшим, якби запис у кеші resolver-a ще не з'явився або встиг застаріти, і резолверу довелося б додатково робити запит до вищого DNS-сервера.   

3. Сервіс виконує 200 послідовних запитів до цього сервера. Розрахуйте, скільки часу займе лише очікування мережі, якщо кожен запит встановлює нове з'єднання TLS 1.3, нове з'єднання TLS 1.2 і якщо всі запити йдуть через одне з'єднання.

Кожен запит — нове з'єднання TLS 1.3 (3 RTT на з'єднання: 1 TCP + 1 TLS + 1 HTTP; якщо з DNS щоразу — 4 RTT): без днс 18000 мс  з днс 24000мс

Кожен запит — нове з'єднання TLS 1.2 (4 RTT на з'єднання: 1 TCP + 2 TLS + 1 HTTP; з DNS — 5 RTT):
без днс 24000мс с днс 30000мс

Усі 200 запитів через одне з'єднання (1-й запит витрачає 3 RTT на встановлення TCP + TLS 1.3 + HTTP, а решта 199 запитів ідуть по готовому каналу по 1 RTT кожен):

без днс 6060мс с 1днс на початку 6090 мс

## Висновки

**Що виявилося несподіваним.** Результат, який не збігся з очікуванням, і рядок виводу, де це видно:

у кроці 5.2 для першого запиту TLS 1.3 значення time_namelookup склало 0.060961 с, тоді як для всіх наступних вимірів воно стабільно дорівнювало ~30.5 мс. Це пояснюється тим, що на момент першого запуску запис встиг вийти з кешу резолвера ( бо закінчився TTL), тому резолвер виконав додатковий рекурсивний запит, а далі вже віддавав відповідь із кешу за 1 RTT.

**Яке питання залишилося без відповіді.** Що саме не вдалося пояснити; до якого рядка виводу це стосується; що вже перевірено і що з'ясувати не вдалося:
У кроці 5.2 на етапі appconnect виникає додаткова затримка ~3.6–3.8 мс понад ціле число RTT (наприклад, 94.42 мс замість очікуваних ~90 мс для TLS 1.3), тоді як розбір записів між starttransfer та total у кроці 5.3 займає лише ~0.06–0.16 мс.

## Декларація використання ШІ

Використовувалися інструменти ШІ: <!-- так / ні --> так

<!-- Якщо так, заповнити таблицю: один рядок на кожне звернення, що вплинуло на звіт. Промпт наводити дослівно. -->

| № | Інструмент | Промпт (дослівно) | Що використано у звіті і що змінено самостійно |
|---|---|---|---|
| 1 | Gemini |відповіси на запитання у 4 та 5 завданні| дослідив відповіді |сформулвав своїми словами|
| 2 | Gemini | згенеруй мені готові езаповнені макети таблиці у форматі мд для 2 3 4 5 завданнь | вставив пусті макети таблиць |таблиці були заповнені самостійно|
| 3 | Gemini | перевірь| перевірено виконання лаби |переконався. нічого не міняв|

