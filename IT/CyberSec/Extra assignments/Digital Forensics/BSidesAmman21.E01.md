---
tags:
  - IT/CS/CTF
  - IT/Digital-Forensics
Date and time: 2026-08-23T22:38:00
Source:
  - https://archive.org/download/BSidesAmman21.E01
Topic: Dead-box Forensic Analysis
---

# Description

> **Scenario:** You have been given a system that has been used for some illegal activity where a user accessed confidential files they were not authorized to access. The system has two user accounts that are the main subjects of the investigation: `joker` and `IEUser`.
> 
> Your task is to identify the confidential files and provide evidence of unauthorized access.

SHA-256 хеш розпакованого зліпку: `2b830de50a198b50bdd677098331270956ba41633710629923131cf8e1fbd02a`
# Acquisition
Оскільки постановкою задачі не зазначено які саме файли вважаються конфіденційними, нам доведеться шукати ці файли вручну. Першим що спадає на думку є перегляд LNK файлів, prefetch та Jump lists, які утворюються автоматично при відкритті файлів (prefetch для програм). 

Шлях до шуканих Jump List файлів: `C:/Users/_username/AppData/Roaming/Microsoft/Windows/Recent/AutomaticDestinations/`.

Результати пошуку для користувача `IEUser` не дали значних результатів окрім декількох файлів із трохи підозрілими назвами:

![Pasted image 20260824231326.png](../../../../Cache/IMGs/Pasted%20image%2020260824231326.png)

Попередньо відзначимо, що нічого підозрілого поки не знайдено. Зробимо ту саму перевірку для `joker`:

![Pasted image 20260824221413.png](../../../../Cache/IMGs/Pasted%20image%2020260824221413.png)

Тут відразу бачимо п'ять різних за хешем файлів що містять *"Confidential"* у назві. Ймовірно, це і є конфіденційні файли. Відзначимо, що MIME тип цих файлів позначається як `application/x-msoffice`. Це, у комбінації із `.docx` розширеннями, означає, що автоматично створенні при відкритті файли посилань ( ті самі, завдяки яким, при відкритті застосунків програмного пакету MS Office, можна побачити нещодавно відкриті файли серед запропонованих на головній сторінці додатку ) дійсно відносяться до файлів MS Office. Це означає що потенційно ефективним вектором є пошук програм що можуть відкривати подібні файли (MS Office, WinWord тощо).

![Pasted image 20260824230843.png](../../../../Cache/IMGs/Pasted%20image%2020260824230843.png)

Спробуймо отримати інформацію про відкриття цих файлів завдяки кущу реєстру `NTUSER.DAT`. Цей файл завжди знаходиться у домашній директорії користувача. З нього можемо дослідити, наприклад відкриття файлів за допомогою Explorer (чи інших стандартних діалогових вікон), переходячи за таким шляхом: `/Software/Microsoft/Windows/CurrentVersion/Explorer/RecentDocs/`. Вказана директорія міститиме артефакти, що вказують на відкриття користувачем файлів за допомогою системної утиліти Explorer:

![Pasted image 20260824233312.png](../../../../Cache/IMGs/Pasted%20image%2020260824233312.png)

Таким чином було підтверджено відкриття чотирьох конфіденційних файлів: 
- `Confidential.rtf`
- `Confidential_02.docx`
- `Confidential_03.docx`
- `Confidential_04.docx`

Відкриття п'ятого файлу, тобто однойменного `Confidential.rtf` таким чином не підтверджено. Але це можна довести перехресним чином проаналізувавши prefetch файл програми WordPad (як виявилось, MS Word не був встановлений):

![Pasted image 20260825004420.png](../../../../Cache/IMGs/Pasted%20image%2020260825004420.png)

Відтак можемо підтвердити час відкриття файлів, кількість разів, що програма була відкрита ( 5 ) несистемним користувачем, а також розділ, з якого було отримано доступ ( домашня директорія користувача `joker` ):

![Pasted image 20260825004939.png](../../../../Cache/IMGs/Pasted%20image%2020260825004939.png)

До того ж, ми можемо підтвердити розташування цих файлів з того ж Jump list:

![Pasted image 20260824215545.png](../../../../Cache/IMGs/Pasted%20image%2020260824215545.png)

Для перехресного підтвердження також продемонструймо LINK файли:

![Pasted image 20260824221749.png](../../../../Cache/IMGs/Pasted%20image%2020260824221749.png)

Можемо викачати бажаний Jump List та проаналізувати:

```
F:\stuff\JLECmd>JLECmd.exe -f 469e4a7982cea4d4.automaticDestinations-ms
JLECmd version 2026.5.0

Author: Eric Zimmerman (saericzimmerman@gmail.com)
https://github.com/EricZimmerman/JLECmd

Command line: -f 469e4a7982cea4d4.automaticDestinations-ms

Warning: Administrator privileges not found!

Processing F:\stuff\JLECmd\469e4a7982cea4d4.automaticDestinations-ms

Source file: F:\stuff\JLECmd\469e4a7982cea4d4.automaticDestinations-ms

--- AppId information ---
  AppID: 469e4a7982cea4d4
  Description: Windows Wordpad

--- DestList information ---
  Expected DestList entries:  5
  Actual DestList entries:    5
  DestList version:           4


--- DestList entries ---
Entry #: 5
  MRU: 0
  Path: \\192.168.70.128\SharedJJ\docs\Confidential_04.docx
  Pinned: False
  Created on:    2019-02-15 03:52:56
  Last modified: 2019-02-15 05:03:45
  Hostname:
  Mac Address: 00:0c:29:a1:dc:26
  Interaction count: 1

--- Lnk information ---
   (lnk file not present)


Entry #: 4
  MRU: 1
  Path: \\192.168.70.128\SharedJJ\docs\Confidential_03.docx
  Pinned: False
  Created on:    2019-02-15 03:52:56
  Last modified: 2019-02-15 05:03:39
  Hostname:
  Mac Address: 00:0c:29:a1:dc:26
  Interaction count: 1

--- Lnk information ---
   (lnk file not present)


Entry #: 3
  MRU: 2
  Path: \\192.168.70.128\SharedJJ\docs\Confidential_02.docx
  Pinned: False
  Created on:    2019-02-15 03:52:56
  Last modified: 2019-02-15 05:03:34
  Hostname:
  Mac Address: 00:0c:29:a1:dc:26
  Interaction count: 1

--- Lnk information ---
   (lnk file not present)


Entry #: 2
  MRU: 3
  Path: \\192.168.70.128\SharedJJ\docs\Confidential.rtf
  Pinned: False
  Created on:    2019-02-15 03:52:56
  Last modified: 2019-02-15 05:03:25
  Hostname:
  Mac Address: 00:0c:29:a1:dc:26
  Interaction count: 1

--- Lnk information ---
   (lnk file not present)


Entry #: 1
  MRU: 4
  Path: C:\Users\Joker\Confidential.rtf
  Pinned: False
  Created on:    2019-02-15 04:49:40
  Last modified: 2019-02-15 05:02:56
  Hostname: msedgewin10
  Mac Address: 00:0c:29:bc:95:21
  Interaction count: 1

--- Lnk information ---
  Absolute path: This PC\C:\@shell32.dll,-21813\Joker\Confidential.rtf



** JumpList has serialized property store(s)! View its contents via -f for details **

---------- Processed F:\stuff\JLECmd\469e4a7982cea4d4.automaticDestinations-ms in 0,27253670 seconds ----------
```

Бачимо 5 сутностей: 

| сутність | адреса                                                | час останнього відкриття (модифікації) |
| -------- | ----------------------------------------------------- | -------------------------------------- |
| 1        | `C:\Users\Joker\Confidential.rtf`                     | 2019-02-15 05:02:56                    |
| 2        | `\\192.168.70.128\SharedJJ\docs\Confidential.rtf`     | 2019-02-15 05:03:25                    |
| 3        | `\\192.168.70.128\SharedJJ\docs\Confidential_02.docx` | 2019-02-15 05:03:34                    |
| 4        | `\\192.168.70.128\SharedJJ\docs\Confidential_03.docx` | 2019-02-15 05:03:39                    |
| 5        | `\\192.168.70.128\SharedJJ\docs\Confidential_04.docx` | 2019-02-15 05:03:45                    |
# Conclusion
4 з 5 файлів відкривалися через мережеву папку SMB (`\\192.168.70.128\SharedJJ\docs\`). IP-адреса належить до приватного простору **RFC 1918**  ( CIDR `192.168.0.0/16` ), що вказує на несанкціонований доступ до мережевого ресурсу всередині корпоративної або приватної мережі. Зв'язок із користувачем `joker` підтверджується наявністю відповідних записів у його персональному Jump List (AppID WordPad) та кущі `NTUSER.DAT\Software\Microsoft\Windows\CurrentVersion\Explorer\RecentDocs`.

## Chain of custody 

Загальний огляд кейсу:

| **Параметр**                       | **Значення**                                                                                     |
| ---------------------------------- | ------------------------------------------------------------------------------------------------ |
| **Case / Challenge ID**            | BSidesAmman21.E01 (Forensics Scenario)                                                           |
| **Evidence ID**                    | EVD-2026-0823-01                                                                                 |
| **Evidence Description**           | Forensic Image / Raw Disk Dump of Target Workstation (`msedgewin10`)                             |
| **Source URL**                     | [https://archive.org/download/BSidesAmman21.E01](https://archive.org/download/BSidesAmman21.E01) |
| **Original File Name**             | `BSidesAmman21.E01`                                                                              |
| **SHA-256 (Unpacked / Raw Image)** | `2b830de50a198b50bdd677098331270956ba41633710629923131cf8e1fbd02a`                               |
| **Acquisition Date / Time**        | 2026-08-23 22:38:00 UTC                                                                          |

Ланцюжок збереження доказів:

| **Дата / Час (UTC)** | **Від кого (Relinquished by)** | **Кому (Received by)** | **Мета передачі / Дія**                                                                                             | **Стан / Хеш валідація**                                                |
| -------------------- | ------------------------------ | ---------------------- | ------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------- |
| **2026-08-23 22:38** | Archive.org Repository         | Forensic Examiner      | Завантаження вихідного судово-медичного образу                                                                      | Цілісність перевірена, образ поміщено у захищене сховище                |
| **2026-08-23 23:00** | Forensic Examiner              | Analysis Workstation   | Монтування образу у режимі **Read-Only** для парсингу артефактів                                                    | Read-Only блокування активне, модифікація вихідного дампу виключена     |
| **2026-08-23 23:14** | Analysis Workstation           | Examiner Repo          | Експорт артефактів користувача `joker`: LNK, Jump Lists (`469e4a7982cea4d4.automaticDestinations-ms`), `NTUSER.DAT` | Вилучені артефакти збережені у локальній робочій директорії дослідження |
| **2026-08-23 23:44** | Analysis Workstation           | Examiner Repo          | Експорт системних артефактів: Prefetch (`WORDPAD.EXE-*.pf`)                                                         | Перевірка збігу часових міток і вивантаження виводу `JLECmd`            |
| **2026-08-24 01:30** | Analysis Workstation           | Secure Evidence Store  | Демонтування образу, фіксація звіту дослідження                                                                     | Хеш вихідного образу валідований повторно: **MATCH**                    |
Усі аналітичні дії проводилися виключно на робочій копії або через монтування у режимі тільки для читання (Read-Only), що гарантує незмінність оригінального образу `BSidesAmman21.E01`.

## Methods and tools used

**Tools**:
- Autopsy — [4.23.1](https://github.com/sleuthkit/autopsy/releases/download/autopsy-4.23.1/autopsy-4.23.1-64bit.msi)
- [Eric Zimmerman's Tools](https://ericzimmerman.github.io/):
	- JLECmd — [2026.5.0](https://download.ericzimmermanstools.com/JLECmd.zip);
	- PECmd — [2026.5.0](https://download.ericzimmermanstools.com/RBCmd.zip);
	- Registry Explorer — [2026.5.0](https://download.ericzimmermanstools.com/net9/RegistryExplorer.zip).

**Methods**:
1. **Dead-box Forensic Analysis** — дослідження статичного образу накопичувача без завантаження цільової ОС. 
2.  **Evidence Integrity Verification** — перевірка цілісності вхідних даних за допомогою криптографічного хешування SHA-256. 
3.  **Artifact-driven Attribution** — перехресне зіставлення дій користувача за допомогою зв'язки артефактів: *Jump Lists (MRU) + Windows Registry (RecentDocs) + Windows Prefetch (Execution evidence)* для точного встановлення фактів несанкціонованого доступу до файлів