# STM32 Reset Reason
 
Eine in C implementierte Bibliothek für STM32-Mikrocontroller, die beim Systemstart den
Grund des letzten Resets aus dem RCC-Statusregister ausliest und über UART ausgibt. Sie
ist besonders nützlich zur Diagnose von unerwarteten Neustarts, z. B. durch
Watchdog-Timeouts oder Spannungseinbrüche.
 
## Beschreibung
 
Nach jedem Reset speichert der STM32 im RCC-Register die Ursache des Resets als Bitfeld.
Die Bibliothek liest dieses Register aus, kombiniert alle gesetzten Flags zu einem
`reset_reason`-Wert und gibt die Ursachen verständlich über UART aus. Anschließend
werden die Flags automatisch gelöscht, damit beim nächsten Start nur der aktuelle Reset
ausgewertet wird.
 
Da sich mehrere Reset-Ursachen gleichzeitig ergeben können (z. B. gleichzeitiger
Power-On-Reset und PIN-Reset), werden alle gesetzten Flags einzeln ausgewertet und
ausgegeben.
 
## Dateien
 
| Datei            | Beschreibung                                                     |
|------------------|------------------------------------------------------------------|
| `resetreason.h`  | Header mit `reset_reason`-Enum und Funktionsdeklarationen        |
| `resetreason.c`  | Implementierung der Auslesung und UART-Ausgabe                   |
 
## Reset-Ursachen
 
Die Bibliothek erkennt folgende Reset-Ursachen:
 
| Enum-Wert    | Wert   | Beschreibung                                                       |
|--------------|--------|--------------------------------------------------------------------|
| `STARTUP`    | 0x00   | Normaler Start, kein besonderer Reset-Grund                        |
| `IWDGRST1`   | 0x01   | Reset durch den unabhängigen Watchdog (IWDG)                       |
| `WWDGRST1`   | 0x02   | Reset durch den Fenster-Watchdog (WWDG)                            |
| `LPWRRST1`   | 0x04   | Reset durch Low-Power-Ereignis (CPU)                               |
| `BORRST1`    | 0x08   | Reset durch Brown-Out (Spannungseinbruch) *(nur STM32F7)*          |
| `OBLRST1`    | 0x08   | Reset durch den Option-Byte-Loader *(nur STM32G0)*                 |
| `PORRST1`    | 0x10   | Reset durch Power-On oder Power-Down                               |
| `SFTRST1`    | 0x20   | Reset durch Software (`HAL_NVIC_SystemReset`)                      |
| `PINRST1`    | 0x40   | Reset durch den externen Reset-Pin (NRST)                          |
 
## API
 
```c
reset_reason readResetSource(void);                        // Liest Reset-Flags aus und löscht sie anschließend
void printResetSource(reset_reason reset_flags);           // Gibt die Reset-Ursachen über UART aus
```
 
## Verwendung
 
Die beiden Funktionen werden typischerweise direkt nach der Systeminitialisierung
aufgerufen, bevor die Hauptschleife startet:
 
```c
#include "resetreason.h"
 
// Nach HAL_Init() und Peripherie-Initialisierung:
reset_reason reason = readResetSource();
printResetSource(reason);
```
 
`readResetSource()` gibt einen `reset_reason`-Wert zurück, der aus den gesetzten Flags
zusammengesetzt ist. Dieser Wert kann anschließend auch programmatisch ausgewertet werden,
um auf bestimmte Reset-Ursachen zu reagieren:
 
```c
reset_reason reason = readResetSource();
 
if (reason & IWDGRST1) {
    // Auf Watchdog-Reset reagieren
}
 
if (reason & SFTRST1) {
    // Auf Software-Reset reagieren
}
```
 
## Unterstützte STM32-Familien
 
Die Bibliothek unterstützt verschiedene STM32-Serien über Präprozessor-Direktiven:
 
| Familie    | Besonderheit                                                              |
|------------|---------------------------------------------------------------------------|
| STM32F7    | Unterstützt zusätzlich Brown-Out-Reset (`BORRST1`)                        |
| STM32G0    | Verwendet `RCC_FLAG_PWRRST` statt `RCC_FLAG_PORRST`; unterstützt `OBLRST1` |
| andere     | Grundlegende Reset-Ursachen werden auf allen STM32-Familien unterstützt   |
 
## Abhängigkeiten
 
- `main.h` – STM32 HAL
- [`basicuart.h`](https://github.com/Diveturtle93/STM32_Basicuart) – UART-Sende- und Empfangsfunktionen

## Lizenz
 
Dieses Projekt steht unter der [GPL-3.0 Lizenz](LICENSE).
