---
title: Egy valós idejű üzenetek feldolgozási technológia kiválasztása
description: ''
author: zoinerTejada
ms.date: 02/12/2018
ms.openlocfilehash: 64d6fca0a8ffac45f605e90a11cd2b3e53db287f
ms.sourcegitcommit: e7e0e0282fa93f0063da3b57128ade395a9c1ef9
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/05/2018
ms.locfileid: "52901617"
---
# <a name="choosing-a-real-time-message-ingestion-technology-in-azure"></a>Egy valós idejű üzenetek feldolgozási technológia kiválasztása az Azure-ban

Valós idejű feldolgozás mutatnak, amelyek rögzítve lesznek a valós idejű és a feldolgozott minimális késéssel foglalkozik. Valós idejű feldolgozás számos megoldás, amely az üzenetek pufferként működik, és a kibővített feldolgozást, a megbízható kézbesítést és a többi üzenetsor-kezelési szemantikákat üzenet Adatbetöltési tárolóra van szükségük. 

## <a name="what-are-your-options-for-real-time-message-ingestion"></a>Mik azok a valós idejű üzenetbetöltés lehetőségei?

- [Azure Event Hubs](/azure/event-hubs/)
- [Azure IoT Hub](/azure/iot-hub/)
- [A HDInsight alatt futó Kafka](/azure/hdinsight/kafka/apache-kafka-get-started)

## <a name="azure-event-hubs"></a>Azure Event Hubs

[Az Azure Event Hubs](/azure/event-hubs/) egy kiválóan méretezhető adatstreamelési platform és Eseményfeldolgozási szolgáltatás, amely fogadására és feldolgozására másodpercenként több millió. Az Event Hubs képes az elosztott szoftverek és eszközök által generált események, adatok vagy telemetria feldolgozására és tárolására. Az eseményközpontokba elküldött adatok bármilyen valós idejű elemzési szolgáltató vagy kötegelési/tárolóadapter segítségével átalakíthatók és tárolhatók. Az Event Hubs biztosít a közzétételi-feliratkozási képességek az alacsony késésű, nagy mennyiségben, ami lehetővé teszi a big data jellegű alkalmazási megfelelő.

## <a name="azure-iot-hub"></a>Azure IoT Hub

[Az Azure IoT Hub](/azure/iot-hub/) egy felügyelt szolgáltatás, amely megbízható és biztonságos kétirányú kommunikációt tesz lehetővé több millió IoT-eszközök és a egy felhőalapú háttérrendszer között.

IoT-központ szolgáltatás a következők:

* Eszközről a felhőbe és a felhőből az eszközre irányuló kommunikáció számos lehetőséget. A lehetőségek közé tartozik az egyirányú üzenetküldés, a fájlátvitel és a kérés-válasz módszerek.
* Üzenetirányítás más Azure-szolgáltatásokhoz.
* Az eszközök metaadataihoz és Szinkronizáltsági állapotával információk lekérdezhető tárolót.
* A biztonságos kommunikációt és hozzáférés-vezérlés az eszközönkénti biztonsági hitelesítő adatok vagy X.509-tanúsítványok használatával.
* Eszköz csatlakoztatása és az eszközidentitás-kezelési események figyelését.

Tekintetében üzenetbetöltés az IoT Hub hasonlít az Event hubs szolgáltatásba. Azonban kifejezetten készült IoT-eszköz kapcsolatot, nem csak üzenetbetöltés kezeléséhez. További információkért lásd: [összehasonlítása az Azure IoT Hub és az Azure Event Hubs](/azure/iot-hub/iot-hub-compare-event-hubs). 

## <a name="kafka-on-hdinsight"></a>Kafka on HDInsight

[Az Apache Kafka](https://kafka.apache.org/) van egy nyílt forráskódú elosztott adatstreamelési platform, amely segítségével valós idejű adatfolyamatok és streamelési alkalmazásokat hozhat létre. A Kafka az üzenetsorokhoz hasonló üzenetközvetítő funkciót is biztosít, amellyel adatstreameket tehet közzé, illetve feliratkozhat rájuk. Fontos, hogy horizontálisan méretezhető, hibatűrő és rendkívül gyors. [A HDInsight alatt futó Kafka](/azure/hdinsight/kafka/apache-kafka-get-started) Kafka biztosítja az Azure-ban felügyelt, rugalmasan méretezhető és magas rendelkezésre állású szolgáltatást. 

A Kafka néhány gyakori alkalmazási helyzetek a következők:

* **Üzenetkezelés**. Mivel támogatja a közzétételi-feliratkozási, a Kafkát gyakran használják üzenetközvetítőként.
* **Tevékenységkövetés**. Mivel a Kafka érkezési sorrendben naplózás a rekordok, használat nyomon követése, és hozza létre a tevékenységek, például a webhely felhasználói műveletek.
* **Összesítés**. Adatfolyam-feldolgozó, összesítheti a egyesítse és központosítsa az operatív adatokat nyerhet ki adatokat a különböző Streamek információit.
* **Átalakítás**. Streamfeldolgozás használatával egyesítheti és bővítheti az adatokat több bemeneti témakörből egy vagy több kimeneti témakörbe.

## <a name="key-selection-criteria"></a>Fontosabb kritériumok

Így szűkítheti, első lépésként a kérdések megválaszolása:

- Szükség van az IoT-eszközök és az Azure közötti kétirányú kommunikáció? Ha igen, válassza ki az IoT Hub.

- Szüksége kezelheti az egyes eszközök hozzáférését, és visszavonhatja a hozzáférést egy adott eszközhöz tudni? Ha igen, válassza ki az IoT Hub.

## <a name="capability-matrix"></a>Képességmátrix

A következő táblázat összefoglalja a fő különbségeket, a képességek. 

| | IoT Hub | Event Hubs | Kafka on HDInsight |
| --- | --- | --- | --- |
| Felhőből az eszközre irányuló kommunikáció | Igen | Nem | Nem |
| Eszköz által kezdeményezett fájl feltöltése | Igen | Nem | Nem |
| Eszközállapot-adatokat | [Ikereszközök](/azure/iot-hub/iot-hub-devguide-device-twins) | Nem | Nem |
| Protokolltámogatás | MQTT, AMQP, HTTPS <sup>1</sup> | AMQP ÉS HTTPS | [A Kafka-protokoll](https://cwiki.apache.org/confluence/display/KAFKA/A+Guide+To+The+Kafka+Protocol) |
| Biztonság | Eszközönkénti identitás; visszavonható hozzáférés-vezérlés. | Megosztott elérési házirendeket; korlátozott visszavonása a közzétevői házirendek használatával. | Hitelesítés az SASL; moduláris engedélyezési; külső hitelesítés támogatott szolgáltatások integrációja. |

[1] is használhatja [Azure IoT-protokollátjáró](/azure/iot-hub/iot-hub-protocol-gateway) egyéni átjáróként az IoT Hub protokoll-betanítás engedélyezéséhez.

További információkért lásd: [összehasonlítása az Azure IoT Hub és az Azure Event Hubs](/azure/iot-hub/iot-hub-compare-event-hubs).
