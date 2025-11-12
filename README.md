# Лабораторная работа №5  
## Облачные базы данных: Amazon RDS и DynamoDB  

**Автор:** Васильева (Новак) Светлана

**Тема:** Развертывание реляционной базы данных в AWS и подключение CRUD-приложения через Amazon EC2  

**Сервис:** Amazon RDS (MySQL), Read Replica, Amazon DynamoDB *(ознакомительно)*  

---

### Цель работы

Целью лабораторной работы является изучение сервисов Amazon RDS и DynamoDB, освоение создания и настройки реляционных баз данных в облаке AWS, понимание концепции Read Replica для повышения отказоустойчивости и производительности систем, а также реализация подключения веб-приложения, развернутого на Amazon EC2, к базе данных RDS с выполнением базовых операций CRUD и ознакомление с принципами хранения данных в NoSQL DynamoDB.

---

### Шаг 1. Подготовка среды (VPC, подсети, Security Groups)

1. В консоли AWS создала **VPC project-vpc** с CIDR `10.21.0.0/16`.  
2. Добавила **две публичные** и **две приватные подсети** в разных зонах доступности (AZ):  
   - `public-subnet-a (10.21.1.0/24)` — `eu-central-1a`  
   - `public-subnet-b (10.21.2.0/24)` — `eu-central-1b`  
   - `private-subnet-a (10.21.3.0/24)` — `eu-central-1a`  
   - `private-subnet-b (10.21.4.0/24)` — `eu-central-1b`  
3. Подключила **Internet Gateway** для публичных подсетей и **NAT Gateway** (через Elastic IP) для доступа приватных.  
4. Настроила две **Security Groups**:
   - `web-security-group` — разрешён вход по HTTP (80) и SSH (22);  
   - `db-mysql-security-group` — разрешён MySQL (3306) только от `web-security-group`.

<img width="468" height="119" alt="image" src="https://github.com/user-attachments/assets/68f8426b-9f1d-4802-b703-5d850eb6b8e3" />         

5. Добавила в `web-security-group` исходящее правило для порта `3306` (MySQL) в сторону `db-mysql-security-group`.  

<img width="468" height="157" alt="image" src="https://github.com/user-attachments/assets/8ebe6ee0-cb2c-4b29-8230-112cb9baa48f" />         

---

### Шаг 2. Развертывание Amazon RDS

В консоли **Amazon RDS** я создала **Subnet Group** под названием `project-rds-subnet-group`, выбрав ранее созданную VPC и добавив две приватные подсети из разных зон доступности (AZ).  

**Subnet Group** — это логическая группа подсетей, используемая RDS для размещения базы данных в приватных сетях разных зон, чтобы обеспечить отказоустойчивость и безопасность.  

<img width="468" height="225" alt="image" src="https://github.com/user-attachments/assets/e9982263-fe90-4aef-ab79-b22a142210d0" />\
<img width="468" height="191" alt="image" src="https://github.com/user-attachments/assets/6f5c0560-cc91-49d6-af0f-981b7cc58375" />\
<img width="468" height="68" alt="image" src="https://github.com/user-attachments/assets/4a37897b-b805-4945-8d98-ce8858b4a5c8" />\
<img width="468" height="107" alt="image" src="https://github.com/user-attachments/assets/8dd54525-635f-4f5e-b84c-94488fdd8bdd" />             


Далее я развернула экземпляр базы данных **MySQL** со следующими параметрами (полностью следуя инструкции по данной лабораторной работе):
- **DB identifier:** `project-rds-mysql-prod`  
- **Engine version:** MySQL 8.0.42  
- **Instance class:** db.t3.micro (Free Tier)  
- **Storage:** gp3, 20 GB, с автоскейлингом до 100 GB  
- **VPC:** project-vpc  
- **DB subnet group:** project-rds-subnet-group  
- **Public access:** No  
- **Security group:** db-mysql-security-group  
- **Master username:** admin  
- **Database name:** project_db  
- **Backup:** включён, **encryption** и **auto minor version upgrade** отключены  

После создания дождалась статуса **Available** и сохранила endpoint базы данных для дальнейшего подключения.

<img width="468" height="73" alt="image" src="https://github.com/user-attachments/assets/09d79fbe-7c81-4da2-803f-46061595c09a" />         

Скопировала **Endpoint** базы данных (он понадобится для подключения): `project-rds-mysql-prod.cf4so28yocpy.eu-central-1.rds.amazonaws.com`

---

### Шаг 3. Создание виртуальной машины EC2

Для подключения к базе данных я создала виртуальную машину **EC2 (Amazon Linux 2)** в публичной подсети с именем `project-db-client`.  
Использовала группу безопасности `web-security-group`, разрешающую подключение по SSH (22) и HTTP (80).  

При инициализации добавила скрипт для установки MySQL-клиента:
```bash
#!/bin/bash
dnf update -y
dnf install -y mariadb105
```

<img width="468" height="208" alt="image" src="https://github.com/user-attachments/assets/53f25667-6480-49c5-a07a-54054c36f25a" />         

### Шаг 4. Подключение к базе данных и выполнение базовых операций

После запуска EC2-инстанса я подключилась по SSH:  
```bash
ssh -i student-key-k21.pem ec2-user@18.156.78.56
```
<img width="468" height="232" alt="image" src="https://github.com/user-attachments/assets/a0a45a8b-5162-42ec-9158-fc97eeb03449" />         


Далее подключилась к базе данных Amazon RDS с помощью установленного MySQL-клиента:  
```bash
mysql -h project-rds-mysql-prod.cf4so28yocpy.eu-central-1.rds.amazonaws.com -u admin -p
```

<img width="468" height="145" alt="image" src="https://github.com/user-attachments/assets/a43d2525-9ce4-4ad1-a6bc-5a2ee9f762ca" />         

После ввода пароля выполнила выбор базы данных:
```sql
USE project_db;
```

<img width="224" height="31" alt="image" src="https://github.com/user-attachments/assets/b344cab7-602d-4d8a-8285-3fed4066cde2" />         


Создала две связанные таблицы с отношением **один ко многим** (1 → N):  
```sql
CREATE TABLE categories (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100) NOT NULL
);

CREATE TABLE todos (
    id INT AUTO_INCREMENT PRIMARY KEY,
    title VARCHAR(200) NOT NULL,
    status ENUM('open','done') DEFAULT 'open',
    category_id INT NOT NULL,
    FOREIGN KEY (category_id) REFERENCES categories(id)
);
```

<img width="444" height="251" alt="image" src="https://github.com/user-attachments/assets/f122c523-ae09-48ed-9d78-fa0de0112c89" />         


Добавила по три записи в каждую таблицу:
```sql
INSERT INTO categories (name) VALUES ('Work'), ('Study'), ('Home');
INSERT INTO todos (title, category_id) VALUES
('Write report', 1), ('Read AWS docs', 2), ('Clean room', 3);
```

<img width="468" height="183" alt="image" src="https://github.com/user-attachments/assets/d37d494e-45d6-4019-b51f-a8f9542486ff" />         


Выполнила выборку с использованием JOIN:
```sql
SELECT todos.title, categories.name
FROM todos
JOIN categories ON todos.category_id = categories.id;
```

<img width="468" height="368" alt="image" src="https://github.com/user-attachments/assets/47630c55-ede7-4359-b8f6-eb65dbf3bbaf" />\
<img width="457" height="209" alt="image" src="https://github.com/user-attachments/assets/43e58639-d578-4fd7-85ed-ac9ab83a0cae" />\
<img width="468" height="337" alt="image" src="https://github.com/user-attachments/assets/f67b55f4-3955-4cf1-899e-d142e73e35cd" />         

Результат показал корректное соединение таблиц и отображение категорий для каждого задания.

---

### Шаг 5. Создание Read Replica

В консоли RDS выбрала основную базу данных и создала **Read Replica**:  

<img width="468" height="116" alt="image" src="https://github.com/user-attachments/assets/ba002f17-1a88-4ce2-9e39-db162e3d5931" />\
<img width="468" height="222" alt="image" src="https://github.com/user-attachments/assets/9f173a89-59a0-4a86-ab2b-1e12fd99190d" />         

- **DB identifier:** `project-rds-mysql-read-replica`  
- **Class:** db.t3.micro  
- **Storage:** gp3  
- **Public access:** No  
- **Security Group:** db-mysql-security-group  

После создания дождалась статуса **Available** и подключилась к реплике (Endpoint реплики: `project-rds-mysql-read-replica.cf4so28yocpy.eu-central-1.rds.amazonaws.com`):  

<img width="468" height="87" alt="image" src="https://github.com/user-attachments/assets/49dde4aa-9af1-4edd-a736-02f009a73999" />         

```bash
mysql -h project-rds-mysql-read-replica.cf4so28yocpy.eu-central-1.rds.amazonaws.com -u admin -p
```

#### Проверка работы реплики:
1. Запросы **SELECT** выполняются успешно — данные полностью совпадают с основной базой.

<img width="342" height="340" alt="image" src="https://github.com/user-attachments/assets/4ac36572-24e1-445f-a1a8-5552eebdb938" />         

2. При попытке **INSERT/UPDATE** система не разрешает выполнение записи, так как реплика предназначена только для чтения.

<img width="468" height="45" alt="image" src="https://github.com/user-attachments/assets/2ac2ba6c-42ea-4492-b522-bdefae888cdb" />         

3. После добавления новой записи в таблицу на основном экземпляре она отобразилась на реплике, что подтверждает успешную синхронизацию.

<img width="468" height="208" alt="image" src="https://github.com/user-attachments/assets/7b64c055-fda9-4200-b2ff-c12e963c482a" />\
<img width="468" height="233" alt="image" src="https://github.com/user-attachments/assets/d97c7597-692e-4f49-95fe-af2a85754044" />         

**Вывод:** Read Replica используется для масштабирования нагрузки на базу данных — чтение выполняется с реплики, а запись с мастер-экземпляра. Это повышает производительность и отказоустойчивость системы.

---

### Шаг 6. Развертывание CRUD-приложения

На виртуальной машине **EC2** я развернула простое веб-приложение на **Node.js (Express)**, которое подключается к базе данных **Amazon RDS** и выполняет базовые операции **CRUD** (создание, чтение, обновление, удаление).

#### 6.1. Установка Node.js и npm  
Подключившись по SSH к серверу EC2, я установила Node.js и npm:  
```bash
sudo dnf install -y nodejs npm
node -v
npm -v
```

<img width="468" height="205" alt="image" src="https://github.com/user-attachments/assets/38f4d4d4-fb14-4854-9ec7-c1ab26ec3a24" />         

#### 6.2. Загрузка проекта на сервер  
Я создала директорию `project-crud` и загрузила в неё файлы проекта (например, с помощью `scp`):  
```bash
scp -i student-key.pem -r ./project-crud ec2-user@18.156.78.56:~/project-crud
```

<img width="468" height="66" alt="image" src="https://github.com/user-attachments/assets/f8b0e4b7-5513-4b94-b641-24379286c205" />         


Перешла в папку проекта:  
```bash
cd project-crud
```

#### 6.3. Настройка переменных окружения  
Создала файл `.env` с настройками подключения к базе данных:  
```bash
nano .env
```
Вставила в него:
```
DB_WRITE_HOST=project-rds-mysql-prod.xxxxxx.eu-central-1.rds.amazonaws.com
DB_READ_HOST=project-rds-mysql-read-replica.xxxxxx.eu-central-1.rds.amazonaws.com
DB_USER=admin
DB_PASS=StrongPassword123!
DB_NAME=project_db
```

#### 6.4. Установка зависимостей и запуск через PM2  
Установила зависимости проекта и менеджер процессов **PM2**:  
```bash
npm install
sudo npm install -g pm2
```

Запустила приложение:  
```bash
pm2 start app.js --name project-crud
```

Проверила статус процесса:  
```bash
pm2 list
```

<img width="468" height="121" alt="image" src="https://github.com/user-attachments/assets/6a74f53e-1c93-4002-8dfd-74f424c7b1da" />         

#### 6.5. Проверка работы сайта  
Приложение стало доступно по адресу:  
```
http://18.156.78.56
```

<img width="468" height="304" alt="image" src="https://github.com/user-attachments/assets/bd6efe3d-28d4-44eb-bb81-c48301e9ebb3" />         

- **Создание и изменение записей** выполняется через **основной экземпляр базы данных (master)**.  
- **Чтение данных** выполняется через **реплику (read replica)**.  

## Пример работы приложения

#### Добавление записи  
Пользователь заполняет форму на сайте, и новая запись добавляется в базу данных — подтверждая корректное соединение с **RDS master instance**.  

До:

<img width="468" height="111" alt="image" src="https://github.com/user-attachments/assets/c5ba1d45-4375-4ff7-b7da-e46aa05b3cc0" />         

Добавление:

<img width="206" height="274" alt="image" src="https://github.com/user-attachments/assets/5cb6a666-4b35-48a5-8bfd-64577adb145b" />         

После:

<img width="468" height="132" alt="image" src="https://github.com/user-attachments/assets/3755cffe-e794-44c0-b2c1-065c2cc9eb02" />         


#### Изменение статуса  
Приложение отправляет запрос на обновление данных (например, изменение статуса задачи), и запись успешно изменяется в базе данных.  

<img width="340" height="294" alt="image" src="https://github.com/user-attachments/assets/19f2b6a4-4a16-40cf-a7f1-c20251713fea" />\
<img width="468" height="50" alt="image" src="https://github.com/user-attachments/assets/6362a7a4-503e-4107-8ce1-1cc020b23b38" />         


#### Удаление записи  
При нажатии на кнопку удаления запись удаляется из таблицы базы данных, что демонстрирует корректную работу операции **DELETE**.  
Проверила подключение — сайт открывается, данные из базы корректно отображаются и обновляются.  

<img width="312" height="293" alt="image" src="https://github.com/user-attachments/assets/4f4e333a-8b69-4fe3-8665-24a282fcb920" />\
<img width="468" height="119" alt="image" src="https://github.com/user-attachments/assets/9e912a64-37e0-4da9-90cf-f7eed7f3181f" />         


---

### Итог

Я реализовала полную инфраструктуру AWS для работы с облачной базой данных:  
- Настроила MySQL в RDS и Read Replica;  
- Подключила EC2 как клиент;  
- Создала таблицы и протестировала репликацию;  
- Развернула веб-приложение для CRUD-операций;  
- Проверила работу реплики и синхронизацию данных.

Работа завершена успешно.

