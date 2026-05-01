# Домашнее задание к занятию 4 «Работа с roles» `Ефимов Вячеслав`

---

# Ansible Playbook: ClickHouse + Vector + Lighthouse (Roles)

Автоматическое развертывание ClickHouse, Vector и Lighthouse с использованием Ansible ролей.

## Архитектура

[Vector] ---логи---> [ClickHouse] <---запросы--- [Lighthouse]
:9000 :8123 :80

## Структура проекта

```text
ansible-04-role/
├── playbook/
│ ├── inventory/
│ │ └── prod.yml # Инвентарь хостов
│ ├── group_vars/
│ │ ├── clickhouse/vars.yml # Переменные ClickHouse
│ │ ├── vector/vars.yml # Переменные Vector
│ │ └── lighthouse/vars.yml # Переменные Lighthouse
│ ├── requirements.yml # Зависимости ролей
│ ├── site.yml # Основной playbook
│ └── .ansible-lint # Настройки линтера
└── README.md
```

## Требования

- Ansible 2.15+
- Terraform 1.0+ (для создания инфраструктуры в Yandex Cloud)
- SSH-ключ для доступа к ВМ
- Три хоста на Fedora 37+ или AlmaLinux 10+

## Роли

Playbook использует три роли:

| Роль | Репозиторий | Описание |
|------|-------------|----------|
| `clickhouse-role` | [GitHub](https://github.com/IthnHuitn/clickhouse-role) | Установка ClickHouse |
| `vector-role` | [GitHub](https://github.com/IthnHuitn/vector-role) | Установка Vector |
| `lighthouse-role` | [GitHub](https://github.com/IthnHuitn/lighthouse-role) | Установка Lighthouse + Nginx |

### 1. Настройка инвентаря под вашу инфраструктуру

#### - Обновите inventory/prod.yml IP-адресами из вашей инфраструктуры
#### - Укажите свой SSH-ключ

```yml
---
clickhouse:
  hosts:
    clickhouse-01:
      ansible_host: # clickhouse host IP
      ansible_user: fedora
      ansible_ssh_private_key_file: /home/user/.ssh/ssh-key
      ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
      become: true
      become_user: root

vector:
  hosts:
    vector-01:
      ansible_host: # vector host IP
      ansible_user: fedora
      ansible_ssh_private_key_file: /home/user/.ssh/ssh-key
      ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
      become: true
      become_user: root

lighthouse:
  hosts:
    lighthouse-01:
      ansible_host: # lighthouse host IP
      ansible_user: fedora
      ansible_ssh_private_key_file: /home/user/.ssh/ssh-key
      ansible_ssh_common_args: '-o StrictHostKeyChecking=no'
      become: true
      become_user: root
```

### 2. Установка ролей

```bash
ansible-galaxy install -r requirements.yml
```

### 3. Проверка подключения

```bash
ansible -i inventory/prod.yml all -m ping
```

### 4. Запуск playbook

```bash
# Проверка синтаксиса
ansible-lint site.yml

# Запуск
ansible-playbook -i inventory/prod.yml site.yml --diff

# Проверка идемпотентности
ansible-playbook -i inventory/prod.yml site.yml --diff
```

## Использование тегов

```bash
# Только ClickHouse
ansible-playbook -i inventory/prod.yml site.yml --tags clickhouse

# Только Vector
ansible-playbook -i inventory/prod.yml site.yml --tags vector

# Только Lighthouse
ansible-playbook -i inventory/prod.yml site.yml --tags lighthouse
```

## Переменные

```text
ClickHouse (group_vars/clickhouse/vars.yml)
Переменная	        Значение	        Описание
clickhouse_version	26.3.9.8	        Версия для установки

Vector (group_vars/vector/vars.yml)
Переменная	        Значение	        Описание
vector_version	    0.39.0	          Версия Vector

Lighthouse (group_vars/lighthouse/vars.yml)
Переменная	        Значение	        Описание
clickhouse_host	    IP ClickHouse	    Адрес для подключения
```

## Проверка работы

```bash
# Статус сервисов
ansible all -i inventory/prod.yml -m shell -a "systemctl is-active clickhouse-server vector nginx" --become

# Данные в ClickHouse
ansible clickhouse -i inventory/prod.yml -m shell -a "clickhouse-client -q 'SELECT count() FROM logs.vector_logs;'"

# Lighthouse
curl -s -o /dev/null -w "%{http_code}" http://<LIGHTHOUSE_IP>
```

## Автор
## `IthnHuitn`

---

## Подготовка к выполнению

1. Развернул три ВМ в Яндекс.Облаке с помощью Terraform.
2. Из-за постоянной ошибки отсутствия подписи для современных ОС создал свой публичный репозиторий с ролью Clickhouse(установкa через прямые ссылки на RPM-пакеты) и два публичных репозитория Vector и Lighthouse.

  [Clickhouse-role](https://github.com/IthnHuitn/clickhouse-role) [Vector-role](https://github.com/IthnHuitn/vector-role) [Lighthouse-role](https://github.com/IthnHuitn/lighthouse-role)

### Ошибка
![Ошибка](https://github.com/IthnHuitn/ansible-04-playbook/blob/master/screens/clickhouseGPGfail.png)

## Основная часть

![Role1](https://github.com/IthnHuitn/ansible-04-playbook/blob/master/screens/ansible1-1.png)

### Повторный запуск
![Role2](https://github.com/IthnHuitn/ansible-04-playbook/blob/master/screens/ansible1-2.png)


---
