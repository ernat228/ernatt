# Lab 3: Teardown and OS Comparison

## 1. The Machine
* **Manufacturer:** Acer
* **Model:** Nitro V 15 ANV15-52
* **How I found it:** Ran `Get-CimInstance Win32_ComputerSystemProduct | Format-List *` on Windows host.
* **Manual Link / Source:** Acer не предоставляет публичные сервисные руководства (Service Manual) с детальной схемой разборки для этой серии. Согласно правилам лабораторной работы, в качестве источника использовалось официальное руководство пользователя, спецификации с сайта производителя и скриншот из видео-разборки (teardown) аналогичной модели в интернете, что дает достаточное представление о внутренних компонентах.

## 2. Components
*(Смотрите файл `evidence/manual-figures/teardown-annotated.jpg` с расставленными номерами)*

* **1 — RAM (Оперативная память)**
  * **What:** Слот SO-DIMM, DDR4 (установлен модуль Kingston 16 GB).
  * **Connects to:** Контроллер памяти процессора на материнской плате.
  * **Limits:** Форм-фактор SO-DIMM; тип DDR4-3200 (согласно данным AIDA64).

* **2 — Storage (Накопитель)**
  * **What:** Слот M.2 с установленным SSD накопителем.
  * **Connects to:** Шина PCIe на материнской плате.
  * **Limits:** Размер M.2 2280, интерфейс PCIe NVMe.

* **3 — Wi-Fi Card**
  * **What:** Сетевая карта Wi-Fi / Bluetooth.
  * **Connects to:** Слот M.2 (Key E) на материнской плате и антенны в крышке.
  * **Limits:** Форм-фактор M.2 2230.

* **4 — Battery (Батарея)**
  * **What:** Литий-ионный аккумулятор.
  * **Connects to:** Разъем питания на материнской плате.
  * **Limits:** Физические габариты корпуса и номинальная емкость (Li-ion).

* **5 — Cooling System (Система охлаждения)**
  * **What:** Два вентилятора и медные тепловые трубки.
  * **Connects to:** Кристаллы процессора (CPU) и видеокарты (GPU).
  * **Limits:** Максимальный теплоотвод (TDP), ограничивающий пиковую частоту CPU.

## 3. Upgrade Verdict

| Component | Manual / Teardown says | OS says | Verdict |
| :--- | :--- | :--- | :--- |
| **Memory** | 2 слота SO-DIMM, съемная | FormFactor: 12 (SO-DIMM) | **Yes (Partly):** Память не распаяна, можно заменить модули SO-DIMM DDR4. |
| **Storage** | Слот M.2 2280, съемный | BusType: NVMe | **Yes:** Можно установить любой накопитель M.2 2280 NVMe. |
| **CPU** | Распаян на плате под системой охлаждения | SocketDesignation: BGA / FCBGA | **No:** Процессор припаян (BGA), замене не подлежит. |

*Note on disagreement:* OS берет данные о максимальном объеме памяти (MaxCapacity) из таблиц прошивки (SMBIOS), которые производители часто заполняют небрежно. Если ОС показывает 128 ГБ, а спецификация Acer — 32 ГБ, нужно верить спецификации производителя (Manual).

## 4. Three-OS Table

| Fact | Windows (Host) | Linux (Ubuntu VM) |
| :--- | :--- | :--- |
| **Logical CPUs** | `Win32_Processor` NumberOfLogicalProcessors = 12 | `lscpu` CPU(s) = 2 |
| **Total memory** | `Win32_ComputerSystem` TotalPhysicalMemory = 16852574208 | `free -h` total = 1.6Gi |
| **Disk interface** | `Get-PhysicalDisk` BusType = NVMe | `lsblk` TRAN = (пусто, так как это VM) |
| **CPU model** | `Win32_Processor` Name = 13th Gen Intel(R) Core(TM) i5-13420H | `lscpu` Model name = 13th Gen Intel(R) Core(TM) i5-13420H |
| **Motherboard** | `Win32_BaseBoard` Product = Sportage_RTH | `dmidecode -t baseboard` Product Name = 440BX Desktop Reference Platform |

*Сравнение:* Значения в Linux отличаются, так как он запущен в виртуальной машине (VM). Команды в Linux показывают виртуальное оборудование, выделенное гипервизором, в то время как Windows показывает реальное физическое железо хоста. Разница в Total memory вызвана тем, что Windows показывает байты (отнимая часть под систему), а Linux показывает в гибибайтах (GiB).