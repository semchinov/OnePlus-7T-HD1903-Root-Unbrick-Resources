Root на OnePlus 7T (HD1903) под macOS

Инструкция проверена на OxygenOS 10.0.15 HD65BA.
Все команды adb/fastboot выполняются с Mac (❄️ macOS 13),
сервисная прошивка через MSM Download Tool – с Windows-ПК.

⸻

## Предупреждения
	•	Разблокировка загрузчика стирает данные (factory reset).
Сделайте резервную копию заранее.
	•	Любые модификации прошивки лишают гарантии.
Вы действуете на свой страх и риск.
	•	Сохраняйте шильдик IMEI и Serial → понадобятся при обращении в сервис.

⸻

## 0 Установка необходимого ПО на Mac

# ставим adb и fastboot
```bash
brew install --cask android-platform-tools
```

# проверяем версии
```bash
adb  version
```
```bash
fastboot --version
```
Если какая-то из команд проверки завершилась с ошибкой - нужно установить ПО повторно


> **Важно:** после установки переподключите телефон, чтобы macOS выдала запрос доверия устройству.


⸻

## 1  Подготовка телефона

1. **Включите режим разработчика** — *Настройки ▸ О телефоне (About phone) ▸ Номер сборки (Build number)* — нажмите 7 раз, пока не появится «Вы стали разработчиком!».
2. *Настройки ▸ Система (System) ▸ Параметры разработчика (Developer options)*:
   - **Разблокировка OEM (OEM unlocking)** → **включите**.
   - **Отладка по USB (USB debugging)** → включите; при первом подключении подтвердите RSA‑ключ от Mac.
   - *(Необязательно)* **Расширенная перезагрузка (Advanced reboot)** — добавит пункт *Bootloader* в меню питания.
3. **Подключите кабель USB‑C.** Убедитесь, что `adb devices` показывает устройство как `device`, а не `unauthorized`.
4. **Разблокируйте загрузчик** (если ещё не):
   ```bash
   adb reboot bootloader
   fastboot oem unlock           # подтвердите на экране; данные будут стёрты
   fastboot reboot
   ```
   После перезапуска снова включите *Отладку по USB*.

---

⸻

3. Разблокировка загрузчика (на Mac)

adb reboot bootloader            # телефон в fastboot
fastboot oem unlock              # на экране — подтверждаем

Телефон сбросится ещё раз. После настройки → Settings ▸ System ▸ Developer options → включить USB debugging и OEM unlocking.

⸻

4. Извлечение стокового boot.img

На Mac:

mkdir -p ~/OnePlus/root && cd ~/OnePlus/root
cp ~/Downloads/OnePlus7TOxygen_14.O.21_OTA*.zip .
# извлекаем boot.img + dtbo + vbmeta
payload-dumper-go OnePlus7TOxygen_14.O.21_*.zip --partitions boot,dtbo,vbmeta,vbmeta_system --out extracted

Файл extracted/boot.img – сток-ядро 10.0.15.

⸻

5. Патч ядра через Magisk

adb push extracted/boot.img /sdcard/
adb push ~/Downloads/Magisk-v28.1.apk /sdcard/

На телефоне:
	1.	Установите Magisk APK.
	2.	Magisk ▸ Install ▸ Select and patch a file → выберите boot.img.
Через ≈10 с получите magisk_patched-XXXX.img в Download.

⸻

6. Прошивка патченного boot

adb pull /sdcard/Download/magisk_patched-*.img .
adb reboot bootloader
fastboot flash boot magisk_patched-*.img
fastboot reboot

Первый запуск может идти до 90 сек.

⸻

7. Проверка root

Откройте Magisk → должно быть «Installed – v28.1».

adb shell
$ su
# id

Если получили prompt # — root готов.

⸻

8. Настройка раздачи VPN (опционально)
	1.	Tethering hardware acceleration: Settings ▸ System ▸ Developer options → OFF.
	2.	Установите VPN Hotspot (Mygod, F-Droid). Дайте root‐доступ.
	3.	Запустите Red Shield VPN. В VPN Hotspot нажмите Share VPN ▸ Wi-Fi Hotspot.

⸻

9. Как вернуться в сток (OTA/сервис)
	•	Либо через MSM Download Tool (повторить раздел 2).
	•	Либо прошить стоковый boot.img и запустить Settings ▸ System ▸ System updates.

⸻

10. Частые ошибки

Симптом	Лечение
Бесконечный Fastboot Mode после прошивки ядра	Убедитесь, что прошиваете оба слота или активный slot совпадает с прошитым. `fastboot –set-active=a
Qualcomm CrashDump Mode	Прошить стоковый boot/dtbo/vbmeta → если не помогает, возврат через MSM.
FAILED (remote: '(system_a_b) No such partition')	Команду даёте из старого fastboot-bootloader. Перейдите в fastbootd: fastboot reboot fastboot.
OTA не ставится (signature mismatch)	Удалите Magisk Modules → Install to Inactive Slot в Magisk → reboot.


⸻

© 2025 BearColonel.
Лицензия: MIT
