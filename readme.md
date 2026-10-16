# Домашняя лаборатория: Развертывание инфраструктуры Active Directory

![VirtualBox](https://img.shields.io/badge/VirtualBox-7.2.20-2F61B4?logo=virtualbox&logoColor=white)
![Windows Server](https://img.shields.io/badge/Windows_Server-2025-0078D4?logo=windows&logoColor=white)
![Windows 11](https://img.shields.io/badge/Windows_11-Enterprise-0078D4?logo=windows&logoColor=white)
![Active Directory](https://img.shields.io/badge/Active_Directory-AD_DS-0078D4?logo=microsoft&logoColor=white)
![DNS](https://img.shields.io/badge/DNS-Service-4B8BBE?logo=cloudflare&logoColor=white)
![Homelab](https://img.shields.io/badge/Homelab-Sysadmin-2C3E50?logo=linux&logoColor=white)

## 📋 Описание

**Цель:** Построение домашней лаборатории Active Directory для практического изучения администрирования Windows Server.

**Стек технологий:**
- **Серверная ОС:** Windows Server 2025
- **Клиентская ОС:** Windows 11 Enterprise
- **Службы:** Active Directory Domain Services (AD DS), DNS
- **Домен:** ToonWrld.local

## 🖥️ Хостовая система (Test Environment)

| Компонент       | Характеристики                          |
|-----------------|-----------------------------------------|
| **Процессор**   | AMD Ryzen 5 3600 (6 ядер, 12 потоков)   |
| **ОЗУ**         | 64 ГБ DDR4                              |
| **Диски**       | 1 ТБ HDD (Samsung HD103UJ) + 500 ГБ NVMe SSD (Samsung 970) |
| **ОС**          | Windows 10 Pro (64-bit)                 |
| **Гипервизор**  | Oracle VirtualBox 7.2.20                |

## 🌐 Схема сети

| Узел             | IP-адрес                    | Примечание                                                        |
|------------------|-----------------------------|-------------------------------------------------------------------|
| Виртуальная сеть | `10.0.2.0/24`               | NAT Network `AD_HOME_LAB` в VirtualBox                            |
| Шлюз             | `10.0.2.1`                  | Шлюз VirtualBox NAT Network                                       |
| DHCP-сервер      | `10.0.2.2`                  | Встроенный DHCP VirtualBox (раздаёт IP клиенту)                   |
| **DC01**         | `10.0.2.3` (статический)    | Контроллер домена, DNS-сервер. DNS: `127.0.0.1`                   |
| **Client01**     | по DHCP (например, `10.0.2.4`) | DNS вручную указан `10.0.2.3` для поиска домена                |
| Домен            | `ToonWrld.local`            |                                                                   |

## 🚀 Проделанные шаги

### Подготовка
- [x] Скачивание **VirtualBox** (VirtualBox-7.2.20-175154-Win.exe)
- [x] Скачивание расширения **VirtualBox Extension Pack** (Oracle_VirtualBox_Extension_Pack-7.2.20.vbox-extpack)
- [x] Скачивание образа установки **Windows 11 Enterprise** (26200.6584.250915-1905.25h2_ge_release_svc_refresh_CLIENTENTERPRISEEVAL_OEMRET_x64FRE_en-us.iso)
- [x] Скачивание образа установки **Windows Server 2025** (26100.32230.260111-0550.lt_release_svc_refresh_SERVER_EVAL_x64FRE_en-us.iso)
- [x] Установка **VirtualBox**
- [x] Создана **NAT-сеть** с названием `AD_HOME_LAB` (10.0.2.0/24, DHCP enabled)

### Создание виртуальной машины DC01 (контроллер домена)
- [x] Создана ВМ: RAM — 8 ГБ, CPU — 2, вирт. диск 50 ГБ, сеть NAT: `AD_HOME_LAB`
- [x] Установлена и настроена **Windows Server 2025**
- [x] Установлена поддержка **Guest Additions**
- [x] В **Server Manager** установлена роль **Active Directory Domain Services**
- [x] Система повышена до **контроллера домена**
- [x] Назначено доменное имя **ToonWrld.local**

### Создание виртуального клиента Client01
- [x] Создана ВМ: RAM — 8 ГБ, CPU — 2, вирт. диск 50 ГБ, сеть NAT: `AD_HOME_LAB`
- [x] Установлена и настроена **Windows 11 Enterprise**
- [x] Установлена поддержка **Guest Additions**
- [x] Система присоединена к домену **ToonWrld.local**

### Ручное администрирование AD (базовые операции)
- [x] Создана корневая OU **Departments** и три подразделения: **Engineering**, **Compression**, **Business**.
- [x] Созданы группы безопасности: **Engineers**, **CompressionTeam**, **BizOps**.
- [x] Созданы учётные записи первых пользователей вручную (через ADUC).
- [x] Пользователи добавлены в соответствующие группы безопасности.
- [x] Компьютер **CLIENT01** перемещён в OU **Engineering**.
- [x] Отработан сброс пароля пользователя (с учётом правила «сначала вход, потом смена»).
- [x] Создана и прилинкована GPO с баннером безопасности, проверена на клиенте.

### Автоматизация AD (PowerShell + CSV)
- [x] Написан/использован скрипт `AD-Admin-Console.ps1` для меню-ориентированного администрирования.
- [x] Массовое создание пользователей через CSV-импорт (`New-Users-FromCsv`).
- [x] Автоматическое добавление пользователей в группы.
- [x] Диагностирован и исправлен баг с перезаписью `$_` во вложенном `ForEach-Object`.

Исправленный скрипт меню-ориентированного администрирования: [AD-Admin-Console-fixed.ps](Scripts/AD-Admin-Console-fixed.ps)

## 🛠️ Список решенных проблем (Troubleshooting Log)

### 1. Аппаратная виртуализация (BIOS/UEFI)

**Проблема**: Ошибка AMD-V is disabled при запуске виртуальной машины.

**Решение**: Зашел в BIOS материнской платы (Gigabyte B450), нашел и активировал режим SVM Mode (аппаратная виртуализация AMD).

### 2. Восстановление загрузчика ОС (Операционная система)

**Проблема**: После прерывания установки Windows Server не загружалась из-за отсутствия загрузочных записей.

**Решение**: Загрузился в среду восстановления (WinRE). Через diskpart назначил букву скрытому системному разделу (100 МБ). С помощью утилиты bcdboot вручную восстановил загрузчик (bcdboot D:\Windows /s Z: /f UEFI).

### 3. Службы разрешения имен (Уровень сети/DNS)

**Проблема**: Ошибка An Active Directory Domain Controller (AD DC) for the domain could not be contacted при попытке ввести клиент в домен.

**Решение**: Выявил, что клиент использовал публичные DNS (1.1.1.1). Вручную изменил настройки TCP/IPv4 на клиенте, указав в качестве предпочитаемого DNS-сервера IP-адрес контроллера домена (10.0.2.3).

### 4. Автоматизация AD: пользователи создавались, но не добавлялись в группы (PowerShell)
**Проблема:** При массовом импорте пользователей из CSV (`New-Users-FromCsv`) учётные записи создавались корректно, но добавление в группы безопасности не выполнялось. Скрипт выдавал предупреждение:
`WARNING: Group add failed: Cannot validate argument on parameter 'Members'. The argument is null or empty.`

**Причина:** Вложенный `ForEach-Object`, который разбивал строку групп на отдельные имена, перезаписывал автоматическую переменную `$_`. В результате внутри вложенного цикла `$_` содержал уже **имя группы**, а не объект пользователя. Выражение `$_.SamAccountName` возвращало `$null`, и командлет `Add-ADGroupMember` получал пустой аргумент `-Members`.

**Решение:** Перед вложенным циклом сохранил `SamAccountName` пользователя в отдельную переменную `$sam`, а имя группы — в `$groupName`. Это исключило конфликт имён и позволило корректно добавлять пользователей в группы.

## 📈 Планы по развитию

- [ ] Развернуть роль DHCP Server на DC01.
- [ ] Настроить File Server и Home Folders с маппингом сетевого диска через GPO.
- [ ] Расширить PowerShell-скрипт: логирование, создание домашних папок, проверки на существование объектов.
- 
## 📸 Скриншоты

### VirtualBox
![VirtualBox](Img/VirtualBox.jpg)

### Виртуальные машины
![Setup](Img/Setup.jpg)

### Настройка групп и пользователей 
![Setup](Img/Users.jpg)