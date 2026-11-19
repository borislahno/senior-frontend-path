A

https://developer.mozilla.org/en-US/docs/Web/HTTP?utm_source=x#headers

| Схема | Хост                                                             | Порт                | Шлях                 | Query        | Фрагмент | Піде на сервер | Не піде на сервер |
|-------|------------------------------------------------------------------|---------------------|----------------------|--------------|----------|-----------|-------------------|
| HTTPS | Домен:mozilla.org<br/>Піддомен:developer<br/>TLD:org | 443 за замовчуваням | /en-US/docs/Web/HTTP | utm_source=x |   headers       |      https://developer.mozilla.org/en-US/docs/Web/HTTP?utm_source=x    |    headers               |
Ми купуємо тут домен mozilla.org, org ми купити не можемо. Піддомен створюємо

http://localhost:5173/dashboard?tab=billing

| Схема | Хост                                                 | Порт         | Шлях | Query | Фрагмент | Піде на сервер | Не піде на сервер |
|------|------------------------------------------------------|--------------|------|-------|----------|-----------|-------------------|
| HTTP | тут у нас локальний хост, немає доменів і піддоменів | 5173 - явний |   /dashboard  |   tab=billing    | -        |      http://localhost:5173/dashboard?tab=billing   | -                 |
Тут у нас локальний хост, нічого не купуємо і не реєструємо

https://api.example.co.uk:8443/v1/users/42?fields=name,email#top

| Схема | Хост                                            | Порт               | Шлях                 | Query         | Фрагмент | Піде на сервер | Не піде на сервер |
|-------|-------------------------------------------------|--------------------|----------------------|---------------|----------|-----------|-------------------|
| HTTPS | Домен:example.co.uk<br/>Піддомен:apі<br/>TLD:uk | 8443 явно вказаний | /v1/users/42 | fields=name,email |   top      |     https://api.example.co.uk:8443/v1/users/42?fields=name,email   |    top              |
Ми купуємо тут домен example.co.uk, co.uk ми купити не можемо.

https://user:pass@example.com/a/b/../c?x=1&x=2

| Схема | Хост                          | Порт                 | Шлях                 | Query         | Фрагмент | Піде на сервер | Не піде на сервер |
|-------|-------------------------------|----------------------|----------------------|---------------|----------|-----------|-------------------|
| HTTPS | Домен:example.com<br/>TLD:com | 443 за замовчуванням |/a/b/../c | x=1&x=2 | -        |     https://example.com/a/c?x=1&x=2  |    user:pass            |
тут є три особливості:
 - user:pass - так передаються облікові дані користувача для авторизації, це застарілий і небезпечний спосіб. Але ці надін не передаються в url на сервер, браузер їх обрізає.
 - /../ - це означає на папку вище, сегмент b і наступний за ним .. взаємно знищуються, тому на сервер піде /a/c
 - Два однакових параметри, тут все сервер визначає відносно своєї логіки або обирає один з них, а інший відкидає, або формує масив

HTTPS://Example.COM:443/Path?Q=1

| Схема | Хост                                             | Порт              | Шлях                 | Query        | Фрагмент | Піде на сервер | Не піде на сервер |
|-------|--------------------------------------------------|-------------------|----------------------|--------------|----------|-----------|-------------------|
| HTTPS | Домен:example.com<br/>Піддомен:немає<br/>TLD:сom | 443 явно вказаний | /Path | Q=1 | -        |    https://example.com/Path?Q=1    | -                 |
Ми купуємо тут домен example.com, com ми купити не можемо.
Браузер приведе протокол і домен і tld до нижнього регістру (загалом вони не залежні від регістру). Назву параметра і шлях браузер не буде рухати, вон є регістрозалежні
Також порт 443 є портом по замовчуванню для https тому браузер теж його відкине
Після нормалізації браузера https://example.com/Path?Q=1

https://shop.example.com/catalog/items/index.html - база
https://shop.example.com/assets/logo.png
https://shop.example.com/catalog/img/a.png
//cdn.example.com/lib.js - тут не впевнений, можливо ось так https://cdn.example.com/lib.js, але краще поясни
https://shop.example.com/catalog/items/index.html?page=2
https://shop.example.com/catalog/items/index.html#reviews - тут фрагмент не піде на сервер

B
hothawk.ai
- dig +noall +answer hothawk.ai A
Тут я спитав утиліту DNS big про те які ipv4 адреси має домен hothawk.ai
![img.png](img.png)
Ось яку відповідь я отримав, домен має дві ipv4 адреси, TTl для обох 377 - це час скільки вони будуть жити в кеші.
Колонка IN - це клас запису Internet.
A- це тип запису в нашому випадку це Ipv4, далі йдуть дві IP адреси які доступні, задля розподілу наванатаження.

- dig +noall +answer hothawk.ai AAAA
Тут я запитую те саме але вже для ipv6 і також отримую два записи
![img_1.png](img_1.png)

- dig +noall +answer hothawk.ai NS
![img_2.png](img_2.png)
Тут змінився TTL він став більшим, через те що сервери імен змінюються рідко, а NS - це запис про те, що ми запитуємо які сервери імен тримабть записи про цей домен

- dig +noall +answer www.hothawk.ai
![img_3.png](img_3.png)
Тут у мене нічого не змінилось, ip ті самі і ведуть на ту саме мережу Cloudflare. Тип запису на вказаний тому за замовчуванням береться А
  www — це окремий запис у DNS-зоні він не успадковується від основного домену автоматично. У панелі Cloudflare він може бути налаштований як A-запис або як CNAME на hothawk.ai. Через увімкнений проксі Cloudflare в обох випадках назовні віддає лише свої IP-адреси

- dig +noall +answer hothawk.ai A - якщо повторити такі записи з інтервалом в декілька секунд то ми побачимо що ttl зменшується, а це означає що resolver бере дані з кеша, а не запитує з dns серверів.
Коли ttl закінчиться то новий запит знову піде в мережу і знову запишеться в кеш зі своїм ttl

-  dig @1.1.1.1 +noall +answer hothawk.ai A
![img_4.png](img_4.png)
Через те що ми додали @1.1.1.1 - запит йде не до нашого стандартного DNS сервера, а до резолвера якиого ми вказали.

- dig @8.8.8.8 +noall +answer hothawk.ai A
![img_5.png](img_5.png)
Тут отримав таку саму відповідь використовуючи публчіний резолвер Google, це означає що резолвер Cloudflare та Google бачать мій сайт однаково

- dig +trace hothawk.ai
![img_6.png](img_6.png)
тут ми використали trace яка змушує утиліту не питати резолвер, а пройти весь шлях самому від кореня DNS то авторитетних серверів самому

- dig nema-takoho-12345.hothawk.ai
![img_7.png](img_7.png)
Тут ми бачимо негативну відповідь NXDOMAIN про те, що такого домену не існує

```
dig +noall +answer hothawk.ai A
hothawk.ai.		300	IN	A	172.67.178.240
hothawk.ai.		300	IN	A	104.21.31.178
mac@Macs-MacBook-Pro senior-frontend-path % dig +noall +answer hothawk.ai A
hothawk.ai.		296	IN	A	172.67.178.240
hothawk.ai.		296	IN	A	104.21.31.178
mac@Macs-MacBook-Pro senior-frontend-path % dig +noall +answer hothawk.ai A
hothawk.ai.		291	IN	A	172.67.178.240
hothawk.ai.		291	IN	A	104.21.31.178
```
Я тричі виконав запит з інтервалом в 5 секунд і видно що з кожним разом TTL зменшується і все коректно

```
mac@Macs-MacBook-Pro senior-frontend-path % dig @pola.ns.cloudflare.com +noall +answer hothawk.ai A
hothawk.ai.		300	IN	A	172.67.178.240
hothawk.ai.		300	IN	A	104.21.31.178
```
Виконав цю команду і TTL тут завжди оданковий і не зменшується тому що тут ми опиуємо не свій резолвер, а авторитетний сервер і TTl у ньому не для зменшення, а це запис який вказує для ресолвера скільки часу можна кешувати відповідь.

github.com
```
dig +noall +answer github.com A
github.com.		60	IN	A	140.82.121.3
```
Тут я спитав утиліту DNS big про те які ipv4 адреси має домен github.com
З результату видно що TTl для github.com є набагато менший ніж для hothawk.ai

```
dig +noall +answer github.com AAAA
```
Тут нічого не повретається, можлов ipv6 для цього домена немає

```
dig +noall +answer github.com NS
github.com.		900	IN	NS	ns-421.awsdns-52.com.
github.com.		900	IN	NS	dns1.p08.nsone.net.
github.com.		900	IN	NS	ns-1283.awsdns-32.org.
github.com.		900	IN	NS	dns2.p08.nsone.net.
github.com.		900	IN	NS	dns4.p08.nsone.net.
github.com.		900	IN	NS	dns3.p08.nsone.net.
github.com.		900	IN	NS	ns-520.awsdns-01.net.
github.com.		900	IN	NS	ns-1707.awsdns-21.co.uk.
```
В результаті ми бачимо які сервери імен тримають запис про цей домен, тут теж ttl більший ніж в попередніх запитах

```
dig +noall +answer www.github.com
www.github.com.		2843	IN	CNAME	github.com.
github.com.		25	IN	A	140.82.121.4
```
Тут у нас вже є CNAME який має більший TTl ніж звичайни запис, бо він не так часто змінюється. Тут ми можемо побачити відмінність від hothawk.ai.
В hothawk для такого ж запиту у нас було два різних записи з своїми ip адресами - це через проксі Cloudflare, а тут у нас теж два запити, але вони ведуть на ту саму ip адресу бо один з них є CNAME

```
dig +noall +answer github.com A
github.com.		60	IN	A	140.82.121.3
```
```
dig +noall +answer github.com A
github.com.		50	IN	A	140.82.121.3
```
Два запити з проміжко в 10 секунд і ми можемо побачити що ttl зменшився відповідно.

```
dig @1.1.1.1 +noall +answer github.com A
github.com.		19	IN	A	140.82.121.4
```

```
dig @8.8.8.8 +noall +answer github.com A
github.com.		60	IN	A	140.82.121.3
```
Може виникнути питання, чому тут повертаються різні ip адреси. Насправді ці дві адреси належать серврам github, але github  чергує їх щоб змінювати навантаження серверів і в разі виходу одного з ним замінити його на інший. Різні адреси в деяких командах, але водночас і однакові в інших, це лише співпадіння по ttl, тобто запит виконувався коли в кеші було те саме значення. Якщо вконати один запит, а через 60 сек інший то ми побачимо що кожного разу ip міняється.

```
dig +trace github.com

; <<>> DiG 9.10.6 <<>> +trace github.com
;; global options: +cmd
.			76765	IN	NS	i.root-servers.net.
.			76765	IN	NS	h.root-servers.net.
.			76765	IN	NS	m.root-servers.net.
.			76765	IN	NS	f.root-servers.net.
.			76765	IN	NS	b.root-servers.net.
.			76765	IN	NS	e.root-servers.net.
.			76765	IN	NS	j.root-servers.net.
.			76765	IN	NS	d.root-servers.net.
.			76765	IN	NS	k.root-servers.net.
.			76765	IN	NS	l.root-servers.net.
.			76765	IN	NS	a.root-servers.net.
.			76765	IN	NS	g.root-servers.net.
.			76765	IN	NS	c.root-servers.net.
.			76873	IN	RRSIG	NS 8 0 518400 20261202130000 20261119120000 8763 . CDu//CtM/t5CTHdBman6HfiptYSPXflI5bP/yTkERmWEd/ROHYgGaZ9L AT0gMacf9alRF4+GUDs2mJKelctEmf4LM0N5YHkJoYYiFBdzFBHxN1+I JgmjxwJjtzCIFdh0T0umZL6/wXwD78yvc2pTIYA9KiTyXFY8Bu4kHXpZ q8SW1sIG0SzirNBCCLcMEp5XBVDMHHRNfU0hBXvOr4fTCT0xj/EjnxiC VwRFSbKiXzT1CgW3Wd8c3FGhwZU26JLQVF7E7oUwOMSK5Lc2XR0EIC5e HaRNm+nafNN4EUv0+5Ohyk3uzHaAyMTsKLvCvWTDjyWRtmtja3fSWGly zo0wjg==
;; Received 1097 bytes from 77.121.24.7#53(77.121.24.7) in 221 ms

com.			172800	IN	NS	a.gtld-servers.net.
com.			172800	IN	NS	b.gtld-servers.net.
com.			172800	IN	NS	c.gtld-servers.net.
com.			172800	IN	NS	d.gtld-servers.net.
com.			172800	IN	NS	e.gtld-servers.net.
com.			172800	IN	NS	f.gtld-servers.net.
com.			172800	IN	NS	g.gtld-servers.net.
com.			172800	IN	NS	h.gtld-servers.net.
com.			172800	IN	NS	i.gtld-servers.net.
com.			172800	IN	NS	j.gtld-servers.net.
com.			172800	IN	NS	k.gtld-servers.net.
com.			172800	IN	NS	l.gtld-servers.net.
com.			172800	IN	NS	m.gtld-servers.net.
com.			86400	IN	DS	19718 13 2 8ACBB0CD28F41250A80A491389424D341522D946B0DA0C0291F2D3D7 71D7805A
com.			86400	IN	RRSIG	DS 8 1 86400 20261202170000 20261119160000 8763 . acwgg93h89J0zc5cD8cCsBrBXrzyL8djDLd/C5MmfQvoSds3F5u/VeaE dJhfUpcW82VEMxc76cU1fDzjCpIz+Khu9w4N6BjV9dptbEFE1Dk9OTRA QXaD5NLFaRXscIlHxNLdtYx8lPL2Q/swvBNWaFTnPwT5dFXZ7AexnfFT TpWtw+q1wbwCAy9VnpeTw6piQXPWng0+eOWQnceMslhh93LVyrLiioYU 6bOFme81NcrkeVLgjRDjsYqE4UU9x0EKVOI24xBOorw07btjSzEIjvgr g1qnZ41gdneXXc860M2aDWGosBLVGwcjBmAIaDmC/GbpXzQWoIDM57Ll 3oWyIg==
;; Received 1170 bytes from 192.203.230.10#53(e.root-servers.net) in 186 ms

github.com.		172800	IN	NS	ns-520.awsdns-01.net.
github.com.		172800	IN	NS	ns-421.awsdns-52.com.
github.com.		172800	IN	NS	ns-1707.awsdns-21.co.uk.
github.com.		172800	IN	NS	ns-1283.awsdns-32.org.
github.com.		172800	IN	NS	dns1.p08.nsone.net.
github.com.		172800	IN	NS	dns2.p08.nsone.net.
github.com.		172800	IN	NS	dns3.p08.nsone.net.
github.com.		172800	IN	NS	dns4.p08.nsone.net.
CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 900 IN NSEC3 1 1 0 - CK0Q35HRS4H76G6CHNB9414CJN6S5UPL  NS SOA RRSIG DNSKEY NSEC3PARAM
CK0POJMG874LJREF7EFN8430QVIT8BSM.com. 900 IN RRSIG NSEC3 13 2 900 20261126002640 20261118231640 65344 com. B7Yrt+EyOBChfhURhB3ti52cnZS4118tPepxRpCHqD5KvY3rmtKp83Dg zuHkkOJO8wgvKlla0/TjE3Vo4W89gg==
4KB4DFS71LEP8G8P8VT4CCUSQNL4CNCS.com. 900 IN NSEC3 1 1 0 - 4KB4V9IAMJL29MC1VBJ8NDO7E6SI2B8O  NS DS RRSIG
4KB4DFS71LEP8G8P8VT4CCUSQNL4CNCS.com. 900 IN RRSIG NSEC3 13 2 900 20261123024350 20261116013350 65344 com. HxnYCKr6MQZSwA6M+k3NwHOAi0VDXHKKxL9T2O3tgle2zLy332ZNqiwb VSiTrCrp7KkUcO01fZ5ppV94Bru18Q==
;; Received 663 bytes from 192.48.79.30#53(j.gtld-servers.net) in 33 ms

github.com.		60	IN	A	140.82.121.3
;; Received 55 bytes from 198.51.44.8#53(dns1.p08.nsone.net) in 37 ms
```
тут видно трасування для визначення ip яке починаєтсья з кореневого домену, далі сервери tld com, далі сервери які відповідають за зону github.com і вони вже мають запис про ip, в цьому випадку ми запитали сервер зони dns1.p08.nsone.net який видав нам запис про ip домену

```
dig nema-takoho-12345.github.com

; <<>> DiG 9.10.6 <<>> nema-takoho-12345.github.com
;; global options: +cmd
;; Got answer:
;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 34015
;; flags: qr rd ra; QUERY: 1, ANSWER: 0, AUTHORITY: 1, ADDITIONAL: 1

;; OPT PSEUDOSECTION:
; EDNS: version: 0, flags:; udp: 1232
;; QUESTION SECTION:
;nema-takoho-12345.github.com.	IN	A

;; AUTHORITY SECTION:
github.com.		3600	IN	SOA	dns1.p08.nsone.net. hostmaster.nsone.net. 1656468023 43200 7200 1209600 3600

;; Query time: 44 msec
;; SERVER: 77.121.24.7#53(77.121.24.7)
;; WHEN: Thu Nov 19 21:39:04 EET 2026
;; MSG SIZE  rcvd: 122
```
Тут ми отримали статус NOERROR, нам його повернув наш резолвер, але йому відпові сервер зони dns1.p08.nsone.net.
Тут помилка нам вказує на те що ім'я існує, але запису потрібного типу для нього немає, бо ANSWER в нас 0

Відповіді на 4 зпитання:
1. TTL між двома однаковими запитами зменшився, це лоігчно бо після кожного нового запиту в кеші збергіається запис, щоб потім знову не йти в мережі і кожні наступні запити будуть брати дані з кеш і там буде показано менший ttl. Коли ttl закінчиться, то настпуний запит знову піде в мережу і запишеться в кеш з новим ttl. І я повторно позапускав dig на тип A та AAAA і +trace і в обох випадках при першому запиті у мене ttl 300, не розумію чому спочатку він був 377.
2. Так збігаються. Якщо не збігаються то це нормально і означає що один з публчіних резолверів закешував неправильні дані які хтось колись в нього запитав і ttl невалідного кешу ще не закінчився. Окрім того порядок доменів у відповіді може змінюватись, а також може бути присутнє геобалансування.
3. Ось шлях: 1.Спершу dig запитав мій resolver про те які сервери обслуговують корінь і ми отримали відповід. 2. Далі dig запитав один з кореневих серверів який відповів йому які сервери обслуговують зону '.ai'. 3. Далі dig запитав один з tld серверів який відповів йому що не знає ip, але видав список серверів які делеговано серверам Cloudflare. 4. Далі dig  запитав один з цих серверів і той повренув йому список з IP адресами V4
4. Статус відповіді для неіснуючого піддомена це NXDOMAIN і його доставив мій ресолвер (швидше за все роутер), а резолверу його повідмови авторитетний сервер

С
![img_9.png](img_9.png)
Ось схема як визначається ip адрес з моменту введення домену і кліку Enter

Бонусне завдання:
Загалом ці дві команди призначені для різних речей. Команда ping - для перевірки того, чи доступний компютер в мережі, а команда dig для визгачення ip адреси домена.
Файл /etc/hosts - це файл в якому записані відношення домену до ip адреси.
dig не те саме що браузер, тому що браузер спочатку дивиться свій кеш, далі в в файл /etc/host, якщо там немає то браузер дивиться в кеш ОС, а далі запит вже йде до ресолвера і DNS
dig не дивиться в цей файл і оминає кеш браузера та ОС. dig показує стан dns а не те що бачить браузер то на локалхост він кине помилку NXDOMAIN

Відповіді на питання для самоперевірки:
1. Ip адреса - це числове представлення адреси машини в мережі, домен - це словесне представлення адреси сайте, було прийнято для зручності людей, так як словесну адресу простіше запамятати ніж цифрову, а URL це вже повна адреса до ресурсу, яка скаладється з протоклу (http або https), домену, шляху до ресурсу і не обовязкових параметрів запиту та фрагментів.
По суті без порта та IP адреси не можливо ставновити зєдання так як це логічна адреса машини, домен і  URL було придумано для людей. Хоча без протоколу http/s також не можливо вставновити зєднання. Тому так чи інакше це все повинно працювати в сукупності.
2. Таке можливо в випадку коли в клієнтів закешувався запис який містить стару адресу. Це залежить від TTL і записів на проміжних вузлах (кеш браузера,кеш ос та кеші резолверів). Така проблема триватиме доки всі кеші поступовно не протухнуть і запит знвоу не побудується від кореня.  Щоб цього уникнути потрібно тримати старий сервер робочим поки не протухнуть всі ttl і запис не оновиться на нові сервери, або знизити TTl заздалегіть або змінити запис. Важливий нюасн,я якщо в файлі /etc/hosts є запис який стосується потрібного домену то він завжли буде посилатись на старйи сервер допкои ми не змінимо його вурчну або не видалимо
3. Запитання рівня співбесіди: 
Перше введення домену на машині:
hothawk.ai в браузері -> кеш браузера (відсутній) -> кеш ОС (відсутній) -> кеш резолвер(роутер)(відсутній) -> кеш резорвела провайдер (відсутній) -> кореневі сервери -> сервер зони .ai ->  сервери зони hothawk.ai -> Повернення IP адреси

Повторне введення домену:
hothawk.ai в браузері -> кеш браузера (якщо пристуній то повертаємо, якщо ні то йдемо далі) -> кеш ОС (якщо пристуній то повертаємо, якщо ні то йдемо далі) -> кеш резолвер(роутер)(якщо пристуній то повертаємо, якщо ні то йдемо далі) -> кеш резорвела провайдер (якщо пристуній то повертаємо, якщо ні то йдемо далі) -> кореневі сервери -> сервер зони .ai ->  сервери зони hothawk.ai -> Повернення IP адреси

Браузер робить рекурсивний запит до резолвреа, а він вже робить ітеративні запити до серверів. Якщо відкрити ту саму адресу через 10 секунд, то TTL запиису про неї ще не протух і ip візьметься з кеша браузера

Висновок:
З цього уроку я дізнався багато новго про те що таке DNS і як він працює. Повторив що таке Ip та url, що таке шляхи та теги. Насправді я не знав що теги не відправляються на сервер. Дізнався про утиліту dig з якою я до цього часу не працював.
Також я дізнався як проходить весь шлях визначення ip адреси по домену і як працюють кеші на резолверах. Частоко я мав уявлення як працює DNS, але сьогодні розібрався в усьому більше детально.
Також новим для мене було CNAME і його призначення.
Одне завдання щодо //cdn.example.com/lib.js мені не вдалось вирішити, сподіваюсь на пояснення від тебе, як правильно це зробити.
Очікую від тебе на детальний звіт по тому що добре, що може бути але варто підтягнути і що взагалі не знаю і варто вивчити. Але наведи мені в звіті датільну інформацію з детальним поясненням тем, на яких я плутаюсь, чи провалився чи тем яких я взагалі не знаю. І не тільки список тем, а саме поясненн кожної деталі в якій я гублюсь чи помиляюсь.
