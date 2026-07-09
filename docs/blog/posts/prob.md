---
title: "Prob: construire un capteur de température local avec micro:bit, Bluetooth et dashboard"
date: 2026-07-09
author: "Sebastien Tadiello"
description: >
    Construire un capteur de température local avec un micro:bit et Raspberry Pi, Bluetooth et dashboard.
categories: ["Raspberry Pi","Bluetooth","Dashboard","IoT","Python","C++","Micro:bit","Embedded Systems"]
languages: ["fr"]
draft: false
---

# Prob: construire un capteur de température local avec micro:bit, Bluetooth et dashboard

![alt text](image.png)

## Introduction

Depuis quelque temps je m'intéresse de plus en plus aux systèmes embarqués et aux capteurs intelligents. Il est vrai que quand on y réfléchit il y a une multitude de données qui sont produites dans notre environnement, mais qui ne sont pas captées, stockées ou analysées. Mais dès que l'on commence à vouloir toucher à de l'embarqué, on se rend compte que ce n'est pas si simple. Il y a beaucoup de contraintes à prendre en compte : le matériel, l'énergie, la connectivité, la robustesse, la visualisation, etc. C'est pourquoi dans ce projet, l’objectif était volontairement simple : construire une première solution de captation locale, autonome et peu coûteuse, capable de récupérer des mesures depuis un capteur, de les stocker proprement, puis de les afficher dans un petit dashboard accessible en local. Le but est réellement de découvrir les contraintes d’un projet de capteur, c'est pourquoi cet article sera plus orienté sur les problèmes rencontrés et les solutions mises en place que sur le code ou l'architecture finale, même si absolument tout le code est disponible sur mon repo GitHub.

## Méthodologie et outils

Dans ce projet j'ai utilisé les outils que j'avais déjà à disposition, ce n'est peut être pas le plus adapté mais le but était aussi d'utiliser du hardware qui ne me servait plus, ici en l’occurrence le [Raspberry Pi 3 B+](https://www.raspberrypi.com/products/raspberry-pi-3-model-b-plus/) et le [micro:bit v2](https://microbit.org/buy/bbc-microbit-go/). 
J'ai pour avancer dans ce projet utilisé les outils suivants :

1. Les docs du micro:bit pour la partie capteur et Bluetooth, le site officiel de Raspberry Pi :

[BBC micro:bit Bluetooth Profile](https://lancaster-university.github.io/microbit-docs/ble/profile/#reference-documentation)

[Bluetooth Low Energy protocol stack](https://www.nordicsemi.com/Products/Development-software/s113)

[Raspberry Pi computer hardware](https://www.raspberrypi.com/documentation/computers/raspberry-pi.html#customer-mac-addresses-on-bcm2712-devices)

2. ChatGPT model 5.5 Thinking au travers de l'application ChatGPT, pour la fouille de documentation, la recherche de solutions et la génération de code (relecture et modification du code généré par l'IA pour matcher à mes patterns et mes habitudes).

---

## Architecture générale

La solution actuelle repose sur trois blocs principaux.

### 1. Le capteur

Le capteur est basé sur une carte micro:bit v2 alimentée par deux piles AAA.
Il mesure périodiquement la température et la luminosité, et expose les données via Bluetooth Low Energy.

Le capteur a un identifiant simple, par exemple :

```txt
DEVICE_ID = "T01"
```

L’objectif est de pouvoir déployer plusieurs capteurs par la suite, chacun avec son propre identifiant.

```txt
T01 = salon
T02 = extérieur
T03 = chambre
```

Le fonctionnement reste minimaliste :

* réveil périodique ;
* lecture de la température et luminosité ;
* exposition ou envoi de la valeur via Bluetooth ;
* retour à un comportement basse consommation autant que possible.

---

### 2. Le Raspberry Pi

Le Raspberry Pi 3b+ joue le rôle de serveur local.

Il récupère les données émises par le capteur en Bluetooth, puis les enregistre dans des fichiers CSV.
Ce choix du CSV est volontaire : ce n’est pas la solution la plus élégante à long terme, mais c’est robuste, lisible, facile à debugger et compatible avec tous les outils Python.

À ce stade du projet, je préfère une solution simple et observable.

Le Raspberry Pi sert donc à la fois de :

* collecteur Bluetooth ;
* stockage local ;
* serveur web local ;
* point d’accès au dashboard.

---

### 3. Le dashboard local

Le dashboard est une petite application Python accessible depuis le réseau local.

Il permet de visualiser :

* l'état du capteur ;
* la dernière température reçue ;
* l’historique des mesures ;
* un petit graphique ;
* une sélection par période ou par jour.

---

## Le capteur micro:bit

Ici je vous présente le code du micro:bit tel qu'il est actuellement déployé sur le capteur, donc il est déjà optimisé pour la consommation d'énergie et ne présente sa connexion Bluetooth que toute les 5min en sortie de sa `deep sleep`. 

[main.cpp](https://github.com/stadiello/microbit_prob/blob/main/source/main.cpp)

```c++
#include "MicroBit.h"
#include "MicroBitUARTService.h"

using namespace codal;

MicroBit uBit;
static MicroBitUARTService *sensorUart;

// -----------------------------------------------------------------------------
// Configuration capteur
// -----------------------------------------------------------------------------
#define DEVICE_ID           "T01"
#define DEBUG_LED           1           // 1 = petit point visuel, 0 = aucune LED
#define SLEEP_INTERVAL_MS 300000 // 300_000 = 5 min.

// Fenêtre pendant laquelle le micro:bit est visible en BLE
#define BLE_WINDOW_MS              20000
// Répétition de l'envoi pendant la fenêtre BLE
#define BLE_SEND_EVERY_MS          2000

static int seq = 0;

static void blink_debug()
{
#if DEBUG_LED
    uBit.display.image.setPixelValue(2, 2, 80);
    uBit.sleep(100);
    uBit.display.image.setPixelValue(2, 2, 0);
#endif
}

static ManagedString make_payload()
{
    seq++;

    int temp_c = uBit.thermometer.getTemperature();
    uBit.sleep(50);
    int light = uBit.display.readLightLevel();

    // Microphone micro:bit v2: volontairement laissé à 0 dans cette V0.
    // On valide d'abord BLE -> Raspberry Pi -> CSV, puis on ajoutera l'API son CODAL exacte.
    int sound = 0;

    ManagedString payload =
        ManagedString(DEVICE_ID) +
        "," + ManagedString(seq) +
        "," + ManagedString((int)system_timer_current_time()) +
        "," + ManagedString(temp_c) +
        "," + ManagedString(light) +
        "," + ManagedString(sound) +
        "\n";

    return payload;
}

static void send_measurement(ManagedString payload)
{
    // Envoie via le service Bluetooth UART.
    // Si aucun central BLE n'est connecté, cette V0 ne bloque pas.
    // Le Raspberry Pi peut se connecter et recevoir les lignes périodiques.
    sensorUart->send(payload);

    // Indication très courte optionnelle.
    blink_debug();
    uBit.sleep(BLE_SEND_EVERY_MS);
}

static void advertise_and_send_window()

{
    ManagedString payload = make_payload();

    // Le micro:bit devient visible/connectable en BLE.
    uBit.bleManager.advertise();
    uint64_t start = system_timer_current_time();
    while ((system_timer_current_time() - start) < BLE_WINDOW_MS)
    {
        // Si le Raspberry est connecté et abonné à l'UART,
        // il recevra la ligne. Si personne n'est connecté,
        // send() ne bloque pas durablement.
        send_measurement(payload);
    }
    // Arrête les annonces BLE après la fenêtre.
    // Si aucun Raspberry ne s'est connecté, le micro:bit redevient invisible.
    uBit.bleManager.stopAdvertising();
}

static void go_to_sleep()
{
    // Éteint visuellement la matrice.
    uBit.display.clear();
    // deep sleep pr objectif basse conso.
    uBit.power.deepSleep(SLEEP_INTERVAL_MS);
}

int main()
{
    uBit.init();

    uBit.display.print("S");
    uBit.sleep(1000);
    uBit.display.clear();

    // Service UART BLE micro:bit.
    // Buffer TX/RX assez large pour des lignes CSV courtes.
    sensorUart = new MicroBitUARTService(*uBit.ble, 64, 64);

    uBit.display.print("B_A");
    uBit.sleep(1000);
    uBit.display.clear();

    while (true)
    {
        advertise_and_send_window();
        go_to_sleep();
    }

    release_fiber();
}
```

Pour tester la configuration bluetooth il est impératif de supprimer de temps de veille profonde du micro:bit et de lui demander d'émettre toutes les 5 secondes. Cela permet de vérifier que le micro:bit est bien détecté par le Raspberry Pi et que les données sont bien reçues. Modifiez simplement la valeur de `SLEEP_INTERVAL_MS` dans le code ci-dessous pour tester la configuration bluetooth.

de:
```c++
#define SLEEP_INTERVAL_MS 300000 // 300_000 = 5 min.
```
à:
```c++
#define SLEEP_INTERVAL_MS 5000 // 5 secondes
```


## Le serveur Raspberry Pi

Donc pour communiquer avec le micro:bit, on utilise le protocole Bluetooth Low Energy (BLE). Le micro:bit expose un service BLE qui permet de lire les caractéristiques de température et de luminosité. Le Raspberry Pi utilise la bibliothèque `bleak` pour scanner et se connecter au micro:bit, puis lire les valeurs.

Dans un premier temps il faut déterminer l'adresse MAC du micro:bit. Pour cela on va utiliser un script de scan BLE qui va lister tous les périphériques à proximité. Une fois l'adresse MAC connue, on peut se connecter au micro:bit et lire les caractéristiques.

[**src/prob/scan_microbit_verbose.py**](https://github.com/stadiello/prob_server/blob/main/src/prob/scan_microbit_verbose.py)

```python
import asyncio
from bleak import BleakScanner

MICROBIT_UART_SERVICE = "e95d5400-251d-470a-a062-fa1922dfa9a8"

async def main():
    print("Scan BLE pendant 15 secondes...")
    devices = await BleakScanner.discover(timeout=15, return_adv=True)

    found = []

    for address, (device, adv) in devices.items():
        name = device.name or adv.local_name or "UNKNOWN"
        rssi = adv.rssi
        service_uuids = [s.lower() for s in adv.service_uuids]

        is_microbit_uart = MICROBIT_UART_SERVICE in service_uuids

        found.append((rssi, name, address, service_uuids, is_microbit_uart))

    found.sort(reverse=True, key=lambda x: x[0])

    for rssi, name, address, services, is_microbit_uart in found:
        marker = " <-- MICROBIT UART ?" if is_microbit_uart else ""
        print(f"{rssi:4} dBm | {name:30} | {address}{marker}")

        if services:
            for s in services:
                print(f"          service: {s}")

asyncio.run(main())
```

Le `MICROBIT_UART_SERVICE` et le UUID qui identifie le service Bluetooth “UART” du micro:bit.

Une fois que l'on a l'adresse MAC du micro:bit, on peut se connecter et lire les caractéristiques de température et de luminosité que le micro:bit envoie sous forme de chaînes de caractères.

[read_microbit_uart.py](https://github.com/stadiello/prob_server/blob/main/src/prob/read_microbit_uart.py)

```python
import asyncio
import csv
from datetime import datetime
from pathlib import Path

from bleak import BleakClient, BleakScanner

TARGET_NAME = "BBC micro:bit"
TARGET_ADDRESS = "F8:B4:AB:35:89:E3"

# UUIDs réellement exposés par ton micro:bit
UART_TX_UUID = "6e400002-b5a3-f393-e0a9-e50e24dcca9e"  # micro:bit -> Raspberry Pi
UART_RX_UUID = "6e400003-b5a3-f393-e0a9-e50e24dcca9e"  # Raspberry Pi -> micro:bit

OUTPUT_DIR = Path("/home/pi/microbit_data")
OUTPUT_DIR.mkdir(parents=True, exist_ok=True)

SCAN_TIMEOUT_SECONDS = 20
READ_DURATION_SECONDS = 20
PAUSE_BETWEEN_CYCLES_SECONDS = 30
PAUSE_AFTER_ERROR_SECONDS = 10

buffer = ""


def log(message: str) -> None:
    print(f"{datetime.now().isoformat(timespec='seconds')} | {message}", flush=True)


def output_path() -> Path:
    return OUTPUT_DIR / f"microbit_{datetime.now().strftime('%Y-%m-%d')}.csv"


def ensure_header(path: Path) -> None:
    if not path.exists():
        with path.open("w", newline="") as f:
            writer = csv.writer(f)
            writer.writerow([
                "raspberry_timestamp",
                "device_id",
                "seq",
                "microbit_uptime_ms",
                "temperature_c",
                "light_level",
                "sound_level",
            ])


async def find_microbit():
    return await BleakScanner.find_device_by_filter(
        lambda d, ad: (
            d.address == TARGET_ADDRESS
            or (d.name is not None and TARGET_NAME in d.name)
            or (ad.local_name is not None and TARGET_NAME in ad.local_name)
        ),
        timeout=SCAN_TIMEOUT_SECONDS,
    )


async def collect_once() -> bool:
    global buffer
    buffer = ""

    log("Recherche du micro:bit...")
    device = await find_microbit()

    if device is None:
        log("micro:bit non trouvé")
        return False

    log(f"Trouvé: {device.name} {device.address}")
    log("Connexion...")

    received_lines = []

    def on_data(sender, data):
        global buffer

        text = data.decode("utf-8", errors="ignore")
        buffer += text

        while "\n" in buffer:
            line, buffer = buffer.split("\n", 1)
            line = line.strip()

            if line:
                received_at = datetime.now().isoformat(timespec="seconds")
                log(f"Reçu: {line}")
                received_lines.append((received_at, line))

    async with BleakClient(device, timeout=30.0) as client:
        log(f"Connecté: {client.is_connected}")

        await client.start_notify(UART_TX_UUID, on_data)

        log(f"Lecture pendant {READ_DURATION_SECONDS} secondes...")
        await asyncio.sleep(READ_DURATION_SECONDS)

        await client.stop_notify(UART_TX_UUID)

    if not received_lines:
        log("Connexion OK, mais aucune donnée reçue.")
        return True

    path = output_path()
    ensure_header(path)

    with path.open("a", newline="") as f:
        writer = csv.writer(f)

        for received_at, line in received_lines:
            parts = line.split(",")

            if len(parts) != 6:
                log(f"Ignoré, format invalide: {line}")
                continue

            device_id, seq, uptime_ms, temp_c, light, sound = parts

            writer.writerow([
                received_at,
                device_id,
                seq,
                uptime_ms,
                temp_c,
                light,
                sound,
            ])

    log(f"CSV écrit dans: {path}")
    return True


async def main():
    log("Démarrage du logger BLE micro:bit")

    while True:
        try:
            await collect_once()
            log(f"Pause {PAUSE_BETWEEN_CYCLES_SECONDS} secondes avant prochain cycle")
            await asyncio.sleep(PAUSE_BETWEEN_CYCLES_SECONDS)

        except Exception as e:
            log(f"Erreur: {type(e).__name__}: {e}")
            log(f"Pause {PAUSE_AFTER_ERROR_SECONDS} secondes après erreur")
            await asyncio.sleep(PAUSE_AFTER_ERROR_SECONDS)

if __name__ == "__main__":
    asyncio.run(main())
```

Maintenant que le Raspberry Pi est capable de scanner et de se connecter au micro:bit et qu'il peut récupérer les données et les stocker dans un fichier CSV il est temps de passer à l’automatisation de la collecte des données. Pour cela on va utiliser `systemd` pour créer un service qui va lancer le script Python.

Pour ce faire vous allez créer un fichier de service systemd, par exemple `/etc/systemd/system/microbit-ble-logger.service` :

```ini
[Unit]
Description=Microbit BLE UART Logger
After=bluetooth.service network-online.target
Requires=bluetooth.service

[Service]
Type=simple
User=pi
WorkingDirectory=/home/pi/prob
ExecStart=/home/pi/.cache/pypoetry/virtualenvs/prob-P0AEL-Po-py3.9/bin/python /home/pi/prob/src/prob/read_microbit_uart.py
Restart=always
RestartSec=10
StandardOutput=journal
StandardError=journal

[Install]
WantedBy=multi-user.target
```

Puis on recharge les services systemd et on active le service :

```sh
sudo systemctl daemon-reload
sudo systemctl enable microbit-ble-logger.service
sudo systemctl restart microbit-ble-logger.service
```

### Problème rencontré : 

#### version de bleak sur Raspberry Pi

Ce problème est minime, je le mentionne ici juste pour que personne ne perde de temps avec cela. Si vous utilisez comme moi un Raspberry Pi 3 B+ avec une version récente de Raspbian, vous aurez probalement la version python 3.9 au maximun donc il vous faudra installer la version "bleak (==0.21.1)".

#### Concurrence Wi-Fi / Bluetooth

Un des premiers problèmes intéressants rencontrés concerne la concurrence entre le Wi-Fi et le Bluetooth.

Sur Raspberry Pi, le Wi-Fi et le Bluetooth partagent souvent des ressources radio proches.
En pratique, cela peut créer des comportements instables quand on veut faire les deux en même temps :

* scanner régulièrement en Bluetooth ;
* garder une connexion SSH stable en Wi-Fi ;
* servir un dashboard local ;
* transférer des fichiers ou faire du debug à distance.

Dans mon cas, le problème typique est le suivant :

[
Bluetooth\ scan\ intensif + Wi-Fi\ actif = instabilité
]

Durant la connexion SSH, le Raspberry Pi ne détectait pas le Bluetooth du micro:bit, ou bien le scan Bluetooth échouait. Le diagnostic du problème a été effectué en désactivant le Wi-Fi et en testant le scan Bluetooth seul avec la commande suivante `sudo ip link set wlan0 down`, ce qui a permis de confirmer que le problème venait bien de la concurrence entre les deux protocoles.

#### Choix techniques pour stabiliser la solution

Il se trouve que je possédais un Adaptateur USB Nano WiFi [Archer T2U Nano](https://www.tp-link.com/fr/home-networking/adapter/archer-t2u-nano/), je me suis donc demandé si je pouvais utiliser ce dongle Wi-Fi pour séparer le Wi-Fi et le Bluetooth sur deux interfaces différentes. Après quelques tests, j'ai pu confirmer que cela stabilisait la solution. Je conservais mon accès SSH sur le dongle Wi-Fi, et je désactivais le Wi-Fi interne du Raspberry Pi pour que le Bluetooth puisse fonctionner correctement. Et là miracle je vois enfin mon micro:bit dans la liste des périphériques Bluetooth scannés par le Raspberry Pi. 

``` t
pi@raspberrypi:~/prob $ poetry run python src/prob/scan_microbit_verbose.py
Scan BLE pendant 15 secondes...
...
-68 dBm | BBC micro:bit [totug] | F8:B4:AB:35:89:E3
...
```
Puis une réussite de l'acquisition des données du micro:bit :

``` t
Trouvé: BBC micro:bit [totug] F8:B4:AB:35:89:E3
Connecté: True
Lecture pendant 20 secondes...
Reçu: T01,1,5021,23,128,0
Reçu: T01,2,10030,23,129,0
Reçu: T01,3,15041,23,130,0
CSV écrit dans: /home/pi/microbit_data/microbit_2026-07-08.csv
```

---

## Conclusion

En conclusion, la solution reste encore expérimentale, il ya plein de limitations, on peut citer par exemple :

* autonomie réelle du capteur incertaine;
* robustesse du Bluetooth à tester dans différents environnements ;
* micro:bit v2 pas forcément le plus adapté pour un capteur autonome ;
* gestion de plusieurs capteurs à améliorer ;
* stockage CSV à remplacer éventuellement par une base locale ;
* absence de supervision automatique du service.

Mais quand on regarde des capteurs de température connectés qui sont vendus entre 10 et 50€, si vous voulez faire votre propre capteur, comprendre les technologies et apprendre en faisant, finalement avec un micro:bit v2 à 20 euros, vous pouvez vous amuser.

# Le code
[Le server](https://github.com/stadiello/prob_server/tree/main)

[Le capteur](https://github.com/stadiello/microbit_prob)