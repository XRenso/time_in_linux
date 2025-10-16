
Когда вы работаете с Linux, кажется, что «время» одно: вот `date` показывает дату, и более ничего.  

На самом деле, в системе существует несколько независимых представлений времени, каждое из которых обслуживает свой уровень — аппаратный, системный или пользовательский.

Это разделение нужно, чтобы одновременно обеспечивать:
- точное представление календарного времени (для журналов, файлов и пользователей),
- стабильное, монотонное измерение интервалов (для планировщика, таймеров, профилирования),
- независимость от ручных/сетевых коррекций времени.


Виды:
1) RTC (Real-Time Clock) 
2) CLOCK_REALTIME
3) CLOCK_MONOTONIC, CLOCK_MONOTONIC_RAW, CLOCK_BOOTTIME
4) Jiffies, tick, внутренние счётчики
5) clocksource, clockevent
6) VDSO (virtual dynamic shared object)
7) CLOCK_TAI, CLOCK_REALTIME_COARSE



# RTC (Real-Time Clock) - аппаратные часы

Что это:
- Небольшой чип на материнской плате, питающийся от батарейки.
- Сохраняет время даже при выключенном питании.
- Работает независимо от ОС; часто управляется через BIOS/UEFI.

Зачем:
- Система при старте знает «какое сейчас время».
- RTC - первичный источник времени, от которого ядро «запускает» системные часы при загрузке.
- Для поддержания функций запуска по таймеру.

Работа с RTC:
```bash 
sudo hwclock --show   # выведет текущее время
```

Синхронизация:
```bash
sudo hwclock --hctosys   # из RTC в системное
sudo hwclock --systohc   # из системного в RTC
```

> [!warning] Как время хранится в RTC
> Обычно это UTC (Linux, MacOS и т.д), но есть системы которые хранят локальное время (Windows по умолчанию). Это важно для случаев dual-boot систем



```mermaid
flowchart TD
    rtc["Аппаратные часы (RTC)"] --> kernel["Ядро (kernel timekeeping)"]

    subgraph clocks [Часы ядра]
        realtime["CLOCK_REALTIME<br/>Календарное системное время"]
        monotonic["CLOCK_MONOTONIC<br/>Время с момента загрузки"]
        raw["CLOCK_MONOTONIC_RAW<br/>Без коррекции NTP"]
        boottime["CLOCK_BOOTTIME<br/>Учитывает время сна"]
        tai["CLOCK_TAI<br/>Атомное время (TAI)"]
        coarse1["CLOCK_REALTIME_COARSE<br/>Грубое системное время"]
        coarse2["CLOCK_MONOTONIC_COARSE<br/>Грубое монотонное время"]
    end

    kernel --> clocks
    clocks --> vdso["VDSO / системные вызовы"]
    vdso --> apps["Приложения и библиотеки<br/>(date, cron, systemd, glibc)"]

    %% Оформление
    style rtc fill:#e3f2fd,stroke:#90caf9,stroke-width:2px
    style kernel fill:#e8f5e9,stroke:#81c784,stroke-width:2px
    style clocks fill:#fffde7,stroke:#fdd835,stroke-width:2px
    style vdso fill:#f3e5f5,stroke:#ba68c8,stroke-width:2px
    style apps fill:#ffe0b2,stroke:#ffb74d

```
