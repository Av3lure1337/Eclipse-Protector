# Eclipse-Protector 1.0.0
EXE-DLL Protector
<img width="1521" height="204" alt="image" src="https://github.com/user-attachments/assets/af46f727-435e-4a72-b223-f7219a881ed5" />
RU:

Активные функции:

1. Anti-Debug:
реализована двухэтапная эшелонированная система защиты от отладки
Защита на уровне загрузчика (EXE-стаб):
Классические PEB-проверки: Анализируются стандартные флаги отладки в структуре Process Environment Block
Проверка флагов кучи (Heap Flags)
Продвинутые трюки: Библиотека активно использует syscall'ы, проверки на основе обработки исключений (SEH/VEH) и замеры времени выполнения блоков кода (Timing attacks через rdtsc), чтобы обнаружить скрытые отладчики или реверс-инженера, идущего по коду "шагами" (Step-by-Step).
Защита на уровне загрузчика (DLL-стаб):
Внедряется секция со специальным x64 шеллкодом. Поскольку это DLL, стандартный antidbg туда не повесишь — он может конфликтовать со средой процесса, в который DLL внедряется. Поэтому там написан кастомный, легкий и очень злой ассемблерный шеллкод

2. Anti-VM:
Каждая виртуальная машина оставляет уникальные «цифровые отпечатки» в операционной системе, которые библиотека собирает и анализирует:

    VMware: Ищется знаменитый «магический порт» VMware (специфический интерфейс связи между гостевой ОС и гипервизором через ассемблерные инструкции), а также проверяются характерные строки в реестре и системных устройствах.

    VirtualBox: Проверяется наличие специфических виртуальных устройств (драйверы графики, сетевые адаптеры VBOX, гостевые дополнения VirtualBox) и ключи реестра, содержащие подстроки VBOX.

    Hyper-V & KVM: Сканируются системные таблицы (SMBIOS/DMI), где производители виртуального железа вынуждены указывать свои идентификаторы.

3. Obfuscation Strings:
   Обфускация строк, тут без обьяснений.

4. Import Resolve:
   Обычные протекторы защищают импорты поверхностно: они просто мутируют, скрывают или рандомизируют имена DLL в заголовках. Но сами названия функций (например, VirtualAlloc, CreateProcessW) остаются нетронутыми в структурах IMAGE_IMPORT_BY_NAME, и любой реверсер, открыв файл в IDA Pro, сразу видит всю подноготную программы через вкладку Imports.
   ( Основная часть логики и смысл были взяты из Astral-PE ( DosX )

5. Fake Section:
   Если опция включена и имя введено: Протектор берёт строку пользователя (например, твой фирменный F0rexRater), обрезает её до разрешённых PE-форматом 8 байт и зануляет остаток.

Если опция выключена (или поле пустое): Протектор автоматически применяет фирменный почерк и выставляет дефолтное имя .eclipse.

ENG: 

Active functions:

1. Anti-Debug:
A two-stage, layered anti-debugging protection system has been implemented.
Loader-level protection (EXE stub):
Classic PEB checks: Standard debugging flags within the Process Environment Block (PEB) structure are analyzed.
Heap flag checks.
Advanced techniques: The library actively employs syscalls, exception-handling-based checks (SEH/VEH), and code block execution timing measurements (timing attacks using `rdtsc`) to detect hidden debuggers or a reverse engineer stepping through the code.
Bootloader-level protection (DLL stub):
A section containing specialized x64 shellcode is injected. Since it is a DLL, standard anti-debugging techniques cannot be used, as they might conflict with the environment of the process into which the DLL is injected. Therefore, custom, lightweight, and highly aggressive assembly shellcode is implemented instead.

2. Anti-VM:
Each virtual machine leaves unique "digital fingerprints" in the operating system, which the library collects and analyzes:

    VMware: It searches for the well-known VMware "magic port" (a specific communication interface between the guest OS and the hypervisor using assembly instructions) and checks for characteristic strings in the registry and system devices.

    VirtualBox: It checks for the presence of specific virtual devices (graphics drivers, VBOX network adapters, VirtualBox Guest Additions) and registry keys containing "VBOX" substrings.

    Hyper-V & KVM: It scans system tables (SMBIOS/DMI), where virtual hardware vendors are required to list their identifiers.

   3. Obfuscation Strings:
   String obfuscation—no explanation needed here.

4. Import Resolve:
Regular protectors only superficially protect imports: they simply mutate, hide, or randomize DLL names in headers. However, the function names themselves (e.g., VirtualAlloc, CreateProcessW) remain intact in the IMAGE_IMPORT_BY_NAME structures, and any reverse engineer, opening the file in IDA Pro, immediately sees the program's full context in the Imports tab.
(The main part of the logic and meaning was taken from Astral-PE (DosX)

5. Fake Section:
   If the option is enabled and a name is entered: The protector takes the user-provided string (e.g., your custom "F0rexRater"), truncates it to the 8 bytes permitted by the PE format, and null-pads the remainder.

If the option is disabled (or the field is empty): The protector automatically applies its signature style and sets the default name to ".eclipse".
