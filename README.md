### A tutorial about installing Arch Linux ARM on Android TV Box 
### 在安卓盒子安装Arch Linux ARM的教程

~~今后或许会有其他发行版的教程~~

~~Probably there will be a tutorial about other distros afterwards~~

Utils 包含一些实用功能. 在基本系统配置完成后运行来下载脚本:

Utils contains some utility function. After the base system is configured, run this to download script:

`sudo pacman -S dialog && sudo wget https://github.com/Scirese/alarm/raw/main/utils/alarm_utils -O /usr/bin/alarm_utils`

然后使用 `sudo alarm_utils` 运行。

Then run with `sudo alarm_utils`.

[中文](alarm.md) [English](alarm_en.md)
