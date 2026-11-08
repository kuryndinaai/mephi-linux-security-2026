# operating-systems

Номер зачетной книжки М265259
Пояснения к работе.

### Раздел 1. Создание пользователя

**1.1. Создание пользователя `user1`, добавление в группу и настройка срока действия пароля**

```bash
groupadd students
useradd -u 1234 user1
usermod -aG students user1
passwd user1
chage -M 90 user1
```

**Зачем:** создан пользователь `user1` с заданным UID, добавлен в группу `students`, установлен пароль и срок его действия ограничен 90 днями.

Проверка:

```bash
id user1
getent passwd user1
getent group students
chage -l user1
```

---

### Раздел 2. Исследование идентификаторов и set-UID

**2.1. Поиск файлов с установленным set-UID**

```bash
sudo find / -type f -perm -4000 -print 2>/dev/null
```

**Зачем:** найдены все обычные файлы в системе, для которых установлен бит set-UID. Такие программы могут выполняться с эффективными правами владельца файла.

---

**2.2. Поиск процессов с RUID обычного пользователя и EUID = root**

```bash
ps -e -o pid,ruid,euid,user,ruser,command
```

Фильтрация:

```bash
ps -e -o pid,ruid,euid,user,ruser,command | awk '$2 != 0 && $3 == 0'
```

**Зачем:** определены процессы, запущенные обычным пользователем, но выполняющиеся с эффективными правами `root`. Для демонстрации использовался процесс `passwd`.

---

### Раздел 3. Изучение механизма set-UID

**3.1. Проверка операции, требующей прав root**

```bash
which cat
ls -l /etc/shadow
cat /etc/shadow
sudo cat /etc/shadow
```

**Зачем:** проверено, что обычный пользователь не может прочитать `/etc/shadow`, а выполнение `cat` через `sudo` позволяет получить доступ с правами `root`.

---

**3.2. Создание копии `cat` и установка set-UID**

```bash
cp /usr/bin/cat ~/cat
sudo chown root:root ~/cat
sudo chmod u+s ~/cat
ls -l ~/cat
```

Проверка:

```bash
~/cat /etc/shadow
```

**Зачем:** на копии утилиты `cat` установлен бит set-UID и владельцем назначен `root`. Проверено, что программа может выполнять операцию с привилегиями владельца файла.

---

### Раздел 4. Изучение механизма Linux capabilities

**4.1. Выбор операции, требующей root**

```bash
which chown
touch ~/test_chown
chown root:root ~/test_chown
```

**Зачем:** проверено, что обычный пользователь не может изменить владельца файла на `root`.

---

**4.2. Назначение capability `CAP_CHOWN`**

```bash
cp /usr/bin/chown ~/chown
sudo setcap cap_chown+ep ~/chown
getcap ~/chown
```

Проверка:

```bash
~/chown root:root ~/test_chown
ls -l ~/test_chown
```

**Зачем:** утилите `chown` предоставлена конкретная привилегия `CAP_CHOWN`, позволяющая изменять владельца файла без предоставления программе полного набора прав `root`.

После эксперимента capability была удалена:

```bash
sudo setcap -r ~/chown
```

---

### Раздел 5. Изучение механизма `sudo`

**5.1. Настройка разрешений для `user1`**

Конфигурация:

```text
user1 ALL=(root) /usr/bin/timedatectl set-ntp false
user1 ALL=(root) /usr/bin/timedatectl set-time *
```

Проверка разрешений:

```bash
sudo -l
```

**Зачем:** пользователю `user1` предоставлено ограниченное право выполнять конкретные операции с системным временем с привилегиями `root`, без предоставления полного доступа через `sudo`.

---

**5.2. Проверка изменения системного времени**

```bash
sudo /usr/bin/timedatectl set-ntp false
sudo /usr/bin/timedatectl set-time "12:00:00"
date
```

**Зачем:** подтверждена возможность пользователя `user1` изменять системное время через `sudo`.

После проверки:

```bash
sudo /usr/bin/timedatectl set-ntp true
```

**Зачем:** восстановлена автоматическая синхронизация времени.

---

