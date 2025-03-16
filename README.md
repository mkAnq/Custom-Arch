# Arhc Installation Guide + Basic Setup

# Подключаем к компьютеру флешку с  загруженным образом Arch, важно проверить подключение к сети ( ping -c 4 google.com ), узнать на какой диск будет ставится Arch ( fdisk -l ) - в моем случае это /dev/sdc

# Форматирование разделов
```bash 
mkfs.ext4 /dev/sdc2
mkfs.fat -F32 /dev/sdc1
```

# Монтирование разделов
```bash
mount /dev/sdc2 /mnt
mount --mkdir /dev/sdc1 /mnt/boot
```

### Установка системы
```bash
pacstrap /mnt base linux linux-firmware nano networkmanager
```

Генерация fstab:
```bash
genfstab -U /mnt >> /mnt/etc/fstab
```

Переход в систему:
```bash
arch-chroot /mnt
```

### Базовая настройка
```bash
echo "archlinux" > /etc/hostname
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc

```

Настройка локалей (`/etc/locale.gen`):
```bash
nano /etc/localegen
locale-gen
```

Добавьте в `/etc/locale.conf`:
```bash
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

Настройка /etc/hosts
```bash
127.0.0.1    localhost
::1          localhost
127.0.0.1    archlinux.localdomain    archlinux
```

### Установка загрузчика
```bash
pacman -S grub efibootmgr os-prober
```

Установите GRUB:
```bash
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB
```

При установке Linux рядом с Windows может возникнуть проблема, что диск, на который был установлен Linux, не будет отображаться в качестве загрузочного
Мне помогло следующее решение:
```bash
grub-install --efi-directory=/boot --removable
grub-install --efi-directory=/boot --target=x86_64-efi --bootloader-id=arch --recheck
```

Включите поддержку Windows в GRUB:
```bash
echo "GRUB_DISABLE_OS_PROBER=false" >> /etc/default/grub
os-prober
```

Сгенерируйте конфиг:
```bash
grub-mkconfig -o /boot/grub/grub.cfg
```

### Настройка сети
```bash
systemctl enable NetworkManager
```

Установите пароль для root:
```bash
passwd
```

### Создание нового пользователя и предоставление прав администратора
```bash
useradd -m -G wheel -s /bin/bash myuser
passwd myuser
```

Разрешите пользователям группы `wheel` использовать sudo:
```bash
pacman -S sudo
nano /etc/sudoers # Раскомментируйте строку "%wheel ALL=(ALL:ALL) ALL"
```

Перезагрузите систему:
```bash
exit
umount -R /mnt
reboot
```

---
## Настройка окружения Hyprland
### Установка Hyprland и связанных компонентов
```bash
pacman -S hyprland waybar rofi alacritty thunar polkit-gnome \
    hyprpaper hyprlock hypridle xdg-desktop-portal-hyprland hyprpolkitagent \
    hyprsysteminfo hyprland-qt-support hyprcursor hyprutils hyprlang \
    hyprwayland-scanner aquamarine hyprgraphics hyprland-qtutils \
    git curl wget swaync
```

### Установка AUR-менеджера yay
```bash
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si
cd .. && rm -rf yay-bin
```

### Установка звука
```bash
pacman -S pulseaudio pavucontrol wireplumber
```

### Установка тем и иконок
```bash
pacman -S papirus-icon-theme lxappearance kvantum nwg-look
```

### Установка SDDM и его настройка
```bash
pacman -S sddm qt6-svg qt6-virtualkeyboard qt6-multimedia-ffmpeg # Если не планируете устанавливать кастомную тему для sddm, то можно ограничиться установкой sddm
systemctl enable sddm
```

Для настройки темы SDDM:
```bash
nano /etc/sddm.conf.d/theme.conf
```

Добавьте:
```ini
[Theme]
Current="Выбранная тема"
```

### Утилиты для монтирования и работы с архивами
```bash
pacman -S udisks2 udiskie file-roller
```

Перезагрузитесь и войдите в Hyprland!

---

# Установка кастомной темы для sddm
### Переместите папку sddm-astronaut-theme в дирректорию /usr/share/sddm/themes
### Выполните команду: 
```bash echo "[Theme]
Current=sddm-astronaut-theme" | sudo tee /etc/sddm.conf
```

# Установка конфигов для Hyprland, rofi, waybar, alacritty
### Переместите соответствующие папки в дирректорию ~/.config
