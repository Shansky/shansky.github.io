---
layout: post
title: Logitech k810 + t650 + ubuntu 14.04
---

Wyglądają mniej więcej tak:

[Logitech k810](http://i.i.cbsi.com/cnwk.1d/i/tim/2012/11/28/35509443_logitech_keyboard_01.jpg)

[Logitech t650](http://www.logitech.com/assets/46161/3/t650-wireless-rechargeable-touchpad.jpg)

są totalnie bezprzewodowe i nie mają bateryjek (better then apple :D )

### K810
Niestety operuje na bluetooth a w zestawie nie ma adaptera usb, więc trzeba sobie dokupić (w moim przypadku - *gembird usb bluetooth dongle*).
Te paczki wystarczyły, aby go uruchomić:
```
$ apt-get install bluez-hcidump bluez-utils blueman
```
Aby sparować urządzenia odpalić ``` blueman```, k810 w tryb parowania (przycisk z tyłu urządzenia + przypisany F1-F2-F3), następnie w terminalu ``` hcitool scan``` co zwróci adres urządzenia w formacie **xx:xx:xx:xx:xx:xx**.

Teraz wystarczy podsłuchać jaki wysyła kod parowania ``` sudo hcidump -at | grep pass ``` (niestety parowanie przez *blueman* nie powiodło się - w moim przypadku).

W innym terminalu ``` sudo bluez-simple-agent hci0 xx:xx:xx:xx:xx:xx ```, wpisać kod uzyskany dzięki *hcidump* na klawiaturze. Teraz w *blueman* dodać do zaufanych i to wszystko.

Klawisze specjalne działają z marszu.

### t650
Tutaj niestety pierwszy kontakt i pierwsze zgryzoty - t650 przy firmware oferowanym z "pudełka" nie współpracuje z ubuntu. To znaczy nie obsługuje kliknięcia dwoma palcami, a obsługa wielu palców i gestów to powód istnienia tego typu urządzeń. Należy podnieść firmware urządzenia do najwyższej dostępnej (chyba sierpień 2014) ale żeby to zrobić trzeba mieć Windowsa - nie próbowałem podnosić po linuksem :)

Gdy już ma się najnowszy firmware, sprzęt działa bez zarzutu. Jedyne co może trochę denerwować to przewijanie, ale i na to są sposoby.

Backup ustawień:
```
$ xinput get-button-map "Logitech Unifying Device. Wireless PID:4101"
1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24
```

Poprawki na prawy klik - 3 palce:
```
$ xinput set-button-map "Logitech Unifying Device. Wireless PID:4101" 1 3 1
```

Odwrócone przewijanie:
```
$ xinput set-button-map "Logitech Unifying Device. Wireless PID:4101" 1 3 1 5 4 7 6
```

Poprawki na gesty:
```
$ sudo /lib/udev/keymap -i input/event15
```

```
Press ESC to finish, or Control-C if this device is not your primary keyboard
scan code: 0x700E3   key code: leftmeta
scan code: 0x70007   key code: d

scan code: 0x700E0   key code: leftctrl
scan code: 0x700E3   key code: leftmeta
scan code: 0x7002A   key code: backspace

scan code: 0x700E0   key code: leftctrl
scan code: 0x700E3   key code: leftmeta
scan code: 0x70072   key code: f23

scan code: 0x700E2   key code: leftalt
scan code: 0x700E3   key code: leftmeta
scan code: 0x70072   key code: f23
```

Przewijanie trzema palcami, przewijanie wzg. lewej krawędzi, przewijanie wzg. górnej krawędzi i przewijanie wzg. prawej krawędzi:

```
/lib/udev/keymaps/logitech-t650
```

```
0x70007 w
0x70072 leftmeta
0x700E2 leftmeta
0x700E0 unknown
0x7002A unknown
```

Zatwierdzenie zmian:
```
$ sudo /lib/udev/keymap input/event15 /lib/udev/keymaps/logitech-t650
```

Dodanie reguł udev:
```
/etc/udev/rules.d/85-logitech-t650.rules
```

```
# Logitech Wireless Touchpad T650 (keymap)
ENV{ID_VENDOR}=="Logitech*", ATTRS{name}=="Logitech Unifying Device. Wireless PID:4101", RUN+="keymap $name logitech-t650"
```

Do kontroli urządzeń łączących się przy pomocy Logitech Unifying używam **Solaar**a

-----
##### Sources

1. [Christian M. Schmid - k810 and ubuntu](http://blog.chschmid.com/?p=1537)
2. [\#\!/bin/strube](http://franklinstrube.com/blog/logitech-t650-wireless-touchpad-ubuntu/)
3. [Map scancodes to keycodes](https://wiki.archlinux.org/index.php/Map_scancodes_to_keycodes)
4. [How to change keymap of device](http://askubuntu.com/questions/69804/how-do-i-change-the-keymap-of-a-single-device-logitech-presenter)
5. [Solaar](https://github.com/pwr/Solaar)