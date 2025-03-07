# Arhc Installation Guide + Basic Setup

## 1. Подготовка
### 1.1. Загрузка ISO
Скачайте последнюю версию Arch Linux с [официального сайта](https://archlinux.org/download/).

### 1.2. Создание загрузочной флешки
Используйте `dd`, Ventoy или Rufus:
sudo dd if=/path/to/archlinux.iso of=/dev/sdX bs=4M status=progress

Где `/dev/sdX` — ваша флешка (проверьте через `lsblk`).

### 1.3. Настройка BIOS/UEFI
- Отключите Secure Boot.
- Выберите загрузку в UEFI-режиме.
- Настройте порядок загрузки с флешки.

---
## 2. Установка Arch Linux
### 2.1. Разметка диска
Загрузитесь с флешки и проверьте диски:
lsblk

Если Windows уже установлена, уменьшите её раздел с помощью `gparted` или `diskmgmt.msc` в Windows.

Создайте разделы (замените `/dev/nvme0n1` на ваш диск):
fdisk /dev/nvme0n1

Создайте:
- 512M EFI (`EF00`)
- остальное — `/` (Linux `8304`)

Форматируйте разделы:
mkfs.fat -F32 /dev/nvme0n1p1
mkfs.ext4 /dev/nvme0n1p2

Монтируем их:
mount /dev/nvme0n1p2 /mnt
mkdir -p /mnt/boot/efi
mount /dev/nvme0n1p1 /mnt/boot/efi

### 2.2. Установка системы
pacstrap /mnt base linux linux-firmware

Генерация fstab:
genfstab -U /mnt >> /mnt/etc/fstab

Переход в систему:
arch-chroot /mnt

### 2.3. Базовая настройка
echo "archlinux" > /etc/hostname
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
hwclock --systohc

Настройка локалей (`/etc/locale.gen`):
sed -i 's/#en_US.UTF-8 UTF-8/en_US.UTF-8 UTF-8/' /etc/locale.gen
locale-gen

Добавьте в `/etc/locale.conf`:
echo "LANG=en_US.UTF-8" > /etc/locale.conf

### 2.4. Установка загрузчика
pacman -S grub efibootmgr os-prober

Установите GRUB:
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB

Включите поддержку Windows в GRUB:
echo "GRUB_DISABLE_OS_PROBER=false" >> /etc/default/grub
os-prober

Сгенерируйте конфиг:
grub-mkconfig -o /boot/grub/grub.cfg

### 2.5. Установка сети
pacman -S networkmanager
systemctl enable NetworkManager

Установите пароль для root:
passwd

### 2.6. Создание нового пользователя и предоставление прав администратора
useradd -m -G wheel -s /bin/bash myuser
passwd myuser

Разрешите пользователям группы `wheel` использовать sudo:
pacman -S sudo
EDITOR=nano visudo

Раскомментируйте строку:
%wheel ALL=(ALL:ALL) ALL

Перезагрузите систему:
exit
umount -R /mnt
reboot

---
## 3. Настройка окружения Hyprland
### 3.1. Установка Hyprland и связанных компонентов
pacman -S hyprland waybar rofi alacritty thunar polkit-gnome \
    hyprpaper hyprlock hypridle xdg-desktop-portal-hyprland hyprpolkitagent \
    hyprsysteminfo hyprland-qt-support hyprcursor hyprutils hyprlang \
    hyprwayland-scanner aquamarine hyprgraphics hyprland-qtutils \
    git curl wget bibata-cursor-theme swaync

### 3.2. Установка AUR-менеджера yay
git clone https://aur.archlinux.org/yay-bin.git
cd yay-bin
makepkg -si
cd .. && rm -rf yay-bin

### 3.3. Установка звука
pacman -S pulseaudio pavucontrol

### 3.4. Установка тем и иконок
pacman -S gruvbox-gtk-theme papirus-icon-theme lxappearance

Настройка GTK (`~/.gtkrc-2.0` и `~/.config/gtk-3.0/settings.ini`):
[Settings]
gtk-theme-name=Gruvbox
icon-theme-name=Papirus

### 3.5. Установка и настройка курсора Bibata
echo "XCURSOR_THEME=Bibata-Modern-Ice" >> ~/.profile
mkdir -p ~/.icons/default
echo -e "[Icon Theme]\nInherits=Bibata-Modern-Ice" > ~/.icons/default/index.theme

### 3.6. Установка SDDM и его настройка
pacman -S sddm
systemctl enable sddm

Для настройки темы SDDM:
nano /etc/sddm.conf.d/theme.conf

Добавьте:
[Theme]
Current=gruvbox

### 3.7. Утилиты для монтирования и работы с архивами
pacman -S udisks2 udiskie file-roller

### 3.8. Автозапуск Hyprland
Создайте `~/.xinitrc`:
echo "exec Hyprland" > ~/.xinitrc

Перезагрузитесь и войдите в Hyprland!

---
## 4. Финальная настройка
- Настройте Waybar (`~/.config/waybar/config.json`)
- Настройте Rofi (`~/.config/rofi/config.rasi`)
- Убедитесь, что автомонтирование работает (`udiskie &`)

