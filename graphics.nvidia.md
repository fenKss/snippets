# Установка драйверов Intel + Nvidia
## Установка intel
### Установка драйвера intel и VA-API и vulkan.
```sh
sudo pacman -S xf86-video-intel libva libva-utils libva-intel-driver vulkan-intel && 
sudo pacman -S lib32-libva lib32-libva-intel-driver lib32-vulkan-intel
```
Настройка Kernel Mode Setting
Во первых необходимо установить linux-headers, для установленного ядра linux.
```
sudo pacman -S linux-headers     # для ядра linux
sudo pacman -S linux-lts-headers # для ядра linux-lts
```
Еще добавьте модуль i915 в строку MODULES в файле `/etc/mkinitcpio.conf`:
```
MODULES=i915
```
Также необходимо установить параметры ядра.

В GRUB Отредактируйте /etc/default/grub и добавьте параметры ядра i915.modeset=1 между кавычками в строке GRUB_CMDLINE_LINUX_DEFAULT:
```
GRUB_CMDLINE_LINUX_DEFAULT="i915.modeset=1"
```
А затем автоматически заново сгенерируйте файл grub.cfg с помощью команды:
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
Если используете UEFI без GRUB то добавьте параметры ядра в строку options, в `/boot/loader/entries/arch.conf`
```
options root=/dev/sda3 rw i915.modeset=1
```
И собрать RAM linux
```
sudo mkinitcpio -p linux
```
Настройка xorg intel
Остается только настроить xorg, создаем файл:
```
/etc/X11/xorg.conf.d/20-intel.conf
Section "Device"
Identifier  "Intel Graphics"
Driver      "intel"
Option      "TearFree"    "true"    # Убирает тиринг
EndSection
```
P.S. Настраивать xorg для intel только в случае если nvidia настраиваться не будет.

## Установка nVidia
Для карт GeForce 620-900 и Quadro/Tesla/Tegra серии K и новее [семейства NVE0, NV110 и NV130 примерно из 2010-2019], установите пакет nvidia, nvidia-lts или nvidia-dkms.
```
sudo pacman -S nvidia nvidia-utils nvidia-settings nvidia-dkms opencl-nvidia libvdpau
sudo pacman -S lib32-nvidia-utils lib32-opencl-nvidia lib32-libvdpau
```
Все остальные драйвера вынесены в AUR. Чтобы узнать актуальную версию для своей видеокарты переходите на сайт nVidia.

Пример установки 390 драйверов nVidia из AUR с использование pacaur.
```sh
pacaur -S nvidia-390xx nvidia-390xx-utils nvidia-390xx-settings nvidia-390xx-dkms opencl-nvidia-390xx
pacaur -S lib32-nvidia-390xx-utils lib32-opencl-nvidia-390xx
```
также необходимо установить для dkms системы.
```sh
sudo pacman -S dkms
```
###  Kernel Mode Setting
Необходимо установить linux-headers для ядра linux.

Для активации добавьте `nvidia-drm.modeset=1` в параметры ядра, а также добавьте `nvidia, nvidia_modeset, nvidia_uvm, nvidia_drm` в `initramfs` в соответствии с `mkinitcpio.conf`.

<br>`/etc/mkinitcpio.conf`
```
MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm ...)
```

<br>`/etc/default/grub` для GRUB
```
GRUB_CMDLINE_LINUX_DEFAULT="nvidia-drm.modeset=1"
```
<br>`/boot/loader/entries/arch.conf` для UEFI без GRUB

```
options root=/dev/sda3 rw nvidia-drm.modeset=1
```
И собрать RAM linux

```sh
sudo mkinitcpio -p linux
```
Отключение intel использование только видеокарты nVidia
При использовании менеджеров входа, создайте или отредактируйте скрипт настройки. Так же для работы требуется установить xorg-xrandr

#### LightDM

Для LightDM:

`/etc/lightdm/display_setup.sh`
```
#!/bin/sh
xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
```
Сделайте этот скрипт выполняемым:
```sh
chmod +x /etc/lightdm/display_setup.sh
```
Теперь настройте LightDM для запуска скрипта, отредактировав раздел [Seat:*] в `/etc/lightdm/lightdm.conf`:

`/etc/lightdm/lightdm.conf`
```
[Seat:*]
display-setup-script=/etc/lightdm/display_setup.sh
Теперь перезагрузитесь и DM запуститься.
```
#### SDDM

`/usr/share/sddm/scripts/Xsetup`
```sh
xrandr --setprovideroutputsource modesetting NVIDIA-0
xrandr --auto
```
#### GDM

Для GDM создайте новый файл .desktop:

`/usr/share/grm/greeter/autostart/optimus.desktop`
```
[Desktop Entry]
Type=Application
Name=Optimus
Exec=sh -c "xrandr --setprovideroutputsource modesetting NVIDIA-0; xrandr --auto"
NoDisplay=true
X-GNOME-Autostart-Phase=DisplayServer
```
Удостоверьтесь, что GDM использует X как стандартный бэкенд.

#### KDM

Для KDM, добавьте строки xrandr в файл `/usr/share/config/kdm/Xsetup.`

Настройка xorg
`/etc/X11/xorg.conf.d/20-nvidia.conf`

```
Section "ServerLayout"
Identifier "layout"
Screen   0 "nvidia"
Inactive   "intel"
EndSection
Section "Device"
Identifier "nvidia"
Driver     "nvidia"
BusID      "PCI:1:0:0"
EndSection
Section "Screen"
Identifier "nvidia"
Device     "nvidia"
Option     "AllowEmptyInitialConfiguration" "Yes"
EndSection
​
Section "Device"
Identifier "intel"
Driver     "modesetting"
BusID      "PCI:0:2:0"
Option     "AccelMethod" "none"
EndSection
​
Section "Screen"
Identifier "intel"
Device     "intel"
EndSection
```
После выполнения всех настроек, необходимо перезагрузить компьютер.

##### Проверка 3D
Для проверки работает ли чип Nvidia установите mesa-demos и запустите:

`glxinfo | grep NVIDIA`<br>
### Настройка nVidia используя PRIME Render Offload<br>
ВНИМАНИЕ! PRIME Render Offload на драйверах nvidia-390xx не работает.<br>
ВНИМАНИЕ! KMS настраиваем и для intel и для nVidia

`/etc/mkinitcpio.conf`
```
MODULES=(nvidia nvidia_modeset nvidia_uvm nvidia_drm i915)
```
`/etc/default/grub` для GRUB
```
GRUB_CMDLINE_LINUX_DEFAULT="nvidia-drm.modeset=1 i915.modeset=1"
```
`/boot/loader/entries/arch.conf `для UEFI без GRUB
```
options root=/dev/sda3 rw nvidia-drm.modeset=1 i915.modeset=1
```
ВНИМАНИЕ! Не забываем заново сгенерировать файл grub.cfg и собирать RAM диск с помощью команды:
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
sudo mkinitcpio -p linux
```
Настройка менеджера входа выполняется аналогично как при отключении intel, но оставить только:

`xrandr --auto`<br>
Далее создать и отредактировать файл
`/etc/X11/xorg.conf.d/20-nvidia.conf`

```
Section "ServerLayout"
Identifier     "Layout0"
Option         "AllowNVIDIAGPUScreens"
Screen      0  "iGPU" 0 0
EndSection
Section "Device"
Identifier     "iGPU"
Driver         "modesetting"
BusID          "PCI:0:2:0"
Option         "TearFree" "true"
EndSection
​
Section "Device"
Identifier     "dGPU"
Driver         "nvidia"
BusID          "PCI:1:0:0"
EndSection
​
Section "Screen"
Identifier     "iGPU"
Device         "iGPU"
DefaultDepth    24
SubSection     "Display"
Viewport        0 0
EndSubSection
EndSection
​
Section "OutputClass"
Identifier     "iGPU"
MatchDriver    "i915"
Driver         "modesetting"
EndSection
​
Section "OutputClass"
Identifier     "dGPU"
MatchDriver    "nvidia-drm"
Driver         "nvidia"
Option         "AllowEmptyInitialConfiguration"
Option         "PrimaryGPU" "yes"
ModulePath     "/usr/lib/nvidia/xorg"
ModulePath     "/usr/lib/xorg/modules"
EndSection
​
Section "ServerFlags"
Option         "IgnoreABI" "1"
EndSection
```
Проверьте BusID, получить информацию об оборудовании можно командой `lspci -k | grep -A 2 -E "(VGA|3D)"`

Перезагрузите компьютер и проверьте, что загрузились оба модуля видеокарт:

`xrandr --listproviders`
```
Providers: number : 2
Provider 0: id: 0x45 cap: 0xf, Source Output, Sink Output, Source Offload, Sink Offload crtcs: 3 outputs: 3 associated providers: 0 name:modesetting
Provider 1: id: 0x240 cap: 0x0 crtcs: 0 outputs: 0 associated providers: 0 name:NVIDIA-G0
```
К примеру, запуск `vkcube`:

`__NV_PRIME_RENDER_OFFLOAD=1 __GLX_VENDOR_LIBRARY_NAME="nvidia" __VK_LAYER_NV_optimus="NVIDIA_only" vkcube`
Чтобы облегчить использование длинной команды, доступен пакет `nvidia-prime`. Пример использования:

`prime-run glxinfo | grep vendor`
```
server glx vendor string: NVIDIA Corporation
client glx vendor string: NVIDIA Corporation
OpenGL vendor string: NVIDIA Corporation
```
