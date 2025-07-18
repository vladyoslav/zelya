# Настройка VirtualBox для работы с Linux

## Содержание
1. [Установка Guest Additions](#установка-guest-additions)
2. [Настройка общей папки](#настройка-общей-папки)
3. [Настройка буфера обмена](#настройка-буфера-обмена)
4. [Устранение проблем](#устранение-проблем)
5. [Сброс root пароля в Linux](#сброс-root-пароля-в-linux)

## Установка Guest Additions

1. В VirtualBox выберите вашу виртуальную машину
2. В меню выберите `Устройства` -> `Установить дополнения гостевой ОС`
3. В Linux выполните:
```bash
# Для Ubuntu/Debian
sudo apt update
sudo apt install -y build-essential dkms linux-headers-$(uname -r)
sudo mount /dev/cdrom /mnt
cd /mnt
sudo ./VBoxLinuxAdditions.run

# Для Fedora
sudo dnf install -y gcc kernel-devel kernel-headers dkms make bzip2 perl
sudo mount /dev/cdrom /mnt
cd /mnt
sudo ./VBoxLinuxAdditions.run
```

4. Перезагрузите виртуальную машину:
```bash
sudo reboot
```

## Настройка общей папки

### Через графический интерфейс VirtualBox

1. Выключите виртуальную машину
2. В настройках виртуальной машины перейдите в раздел `Общие папки`
3. Нажмите на иконку добавления новой папки
4. Укажите:
   - Путь к папке на Windows (например, `C:\Shared`)
   - Имя папки в Linux (например, `shared`)
   - Отметьте `Автоподключение`
   - Отметьте `Создать постоянную папку`

### Через командную строку

1. Создайте папку на Windows (например, `C:\Shared`)
2. В Linux выполните:
```bash
# Создание точки монтирования
sudo mkdir -p /mnt/shared

# Монтирование папки
sudo mount -t vboxsf shared /mnt/shared

# Для автоматического монтирования при загрузке добавьте в /etc/fstab:
echo "shared /mnt/shared vboxsf defaults 0 0" | sudo tee -a /etc/fstab
```

## Настройка буфера обмена

### Двунаправленный буфер обмена

1. В VirtualBox выберите вашу виртуальную машину
2. В меню выберите `Устройства` -> `Буфер обмена` -> `Двунаправленный`

### Через командную строку

```bash
# Проверка статуса буфера обмена
VBoxClient --clipboard

# Запуск сервиса буфера обмена
VBoxClient --clipboard

# Автозапуск при входе в систему
echo "VBoxClient --clipboard" >> ~/.xinitrc
```

## Устранение проблем

### Проблемы с общей папкой

1. Проверка прав доступа:
```bash
# Изменение владельца папки
sudo chown -R $USER:$USER /mnt/shared

# Изменение прав доступа
sudo chmod -R 755 /mnt/shared
```

2. Проверка монтирования:
```bash
# Просмотр смонтированных файловых систем
mount | grep vboxsf

# Принудительное размонтирование
sudo umount -f /mnt/shared

# Повторное монтирование
sudo mount -t vboxsf shared /mnt/shared
```

### Проблемы с буфером обмена

1. Перезапуск сервиса:
```bash
# Остановка сервиса
killall VBoxClient

# Запуск сервиса
VBoxClient --clipboard
```

2. Проверка статуса Guest Additions:
```bash
# Проверка загруженных модулей
lsmod | grep vbox

# Переустановка Guest Additions
sudo /opt/VBoxGuestAdditions-*/init/vboxadd setup
```

### Общие рекомендации

1. Всегда устанавливайте последнюю версию Guest Additions
2. Регулярно обновляйте VirtualBox
3. Используйте последнюю версию Linux
4. При проблемах с производительностью:
   - Увеличьте объем оперативной памяти
   - Включите виртуализацию в BIOS
   - Установите драйверы виртуализации для Windows

## Сброс root пароля в Linux

### Ubuntu/Debian

1. Выключите виртуальную машину
2. В настройках VirtualBox:
   - Выберите вашу виртуальную машину
   - Перейдите в `Настройки` -> `Система`
   - В разделе `Порядок загрузки` переместите `CD/DVD-ROM` на первое место

3. Запустите виртуальную машину
4. При загрузке нажмите `Shift` для входа в меню GRUB
5. Выберите строку с ядром Linux и нажмите `e`
6. Найдите строку, начинающуюся с `linux` и добавьте в конец:
   ```
   init=/bin/bash
   ```
7. Нажмите `Ctrl+X` для загрузки
8. После загрузки выполните:
   ```bash
   # Перемонтирование корневой файловой системы в режиме записи
   mount -o remount,rw /

   # Сброс пароля
   passwd root

   # Перезагрузка
   exec /sbin/init
   ```

### Fedora/RHEL/CentOS

1. Выключите виртуальную машину
2. В настройках VirtualBox:
   - Выберите вашу виртуальную машину
   - Перейдите в `Настройки` -> `Система`
   - В разделе `Порядок загрузки` переместите `CD/DVD-ROM` на первое место

3. Запустите виртуальную машину
4. При загрузке нажмите `e` для входа в меню GRUB
5. Найдите строку, начинающуюся с `linux` и добавьте в конец:
   ```
   rd.break
   ```
6. Нажмите `Ctrl+X` для загрузки
7. После загрузки выполните:
   ```bash
   # Перемонтирование корневой файловой системы в режиме записи
   mount -o remount,rw /sysroot
   chroot /sysroot

   # Сброс пароля
   passwd root

   # Обновление SELinux контекста
   touch /.autorelabel

   # Выход из chroot
   exit

   # Перезагрузка
   reboot
   ```

### Важные замечания

1. После сброса пароля:
   - Верните настройки загрузки в исходное состояние
   - Уберите `CD/DVD-ROM` с первого места в порядке загрузки
   - Сохраните новый пароль в надежном месте

2. Безопасность:
   - Используйте сложный пароль
   - Включите двухфакторную аутентификацию, если возможно
   - Регулярно меняйте пароль

3. Если не удается войти в GRUB:
   - Попробуйте нажать `Shift` или `Esc` при загрузке
   - Убедитесь, что виртуальная машина не запущена в режиме быстрого запуска
   - Проверьте настройки BIOS в VirtualBox

4. Альтернативные методы:
   - Использование live CD/USB
   - Восстановление из резервной копии
   - Использование утилит восстановления системы