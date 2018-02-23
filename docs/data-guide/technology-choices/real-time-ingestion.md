---
title: "A valós idejű üzenet adatfeldolgozást technológia kiválasztása"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 2e6578b779950b5ef11bda7b8ba1fb2e45e09f4e
ms.sourcegitcommit: 3d9ee03e2dda23753661a80c7106d1789f5223bb
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/23/2018
---
# <a name="choosing-a-real-time-message-ingestion-technology-in-azure"></a>Az Azure-ban a valós idejű üzenet adatfeldolgozást technológia kiválasztása

Valós idejű feldolgozására, amelyek rögzítve lesznek a valós idejű és a feldolgozott minimális késéssel adatstreamek foglalkozik. Sok valós idejű feldolgozással megoldás adatfeldolgozást üzenettároló puffer üzenetek, illetve kibővített feldolgozás, a megbízható és egyéb üzenet Üzenetsor-kezelés szemantikáját támogatásához szükséges. 

## <a name="what-are-your-options-for-real-time-message-ingestion"></a>Mik a valós idejű üzenet adatfeldolgozást lehetőségei?

- [Azure Event Hubs](/azure/event-hubs/)
- [Azure IoT Hub](/azure/iot-hub/)
- [Kafka on HDInsight](/azure/hdinsight/kafka/apache-kafka-get-started)

## <a name="azure-event-hubs"></a>Azure Event Hubs

[Az Azure Event Hubs](/azure/event-hubs/) egy kiválóan méretezhető adatfolyam platform és esemény adatfeldolgozást szolgáltatás, képes fogadása és feldolgozása több millió esemény / másodperc. Az Event Hubs képes az elosztott szoftverek és eszközök által generált események, adatok vagy telemetria feldolgozására és tárolására. Az eseményközpontokba elküldött adatok bármilyen valós idejű elemzési szolgáltató vagy kötegelési/tárolóadapter segítségével átalakíthatók és tárolhatók. Az Event Hubs a közzétételi-feliratkozási képességek, nagy léptékű, így a big data forgatókönyvéhez megfelelő kisebb késést biztosít.

## <a name="azure-iot-hub"></a>Azure IoT Hub

[Az Azure IoT Hub](/azure/iot-hub/) egy felügyelt szolgáltatás, amely lehetővé teszi az IoT-eszközök millióira és a felhő alapú háttérből közötti megbízható és biztonságos kétirányú kommunikációt.

Az IoT-központ szolgáltatás a következők:

* Több beállítások eszközről a felhőbe és felhő eszközre kommunikációhoz. A lehetőségek közé tartozik az egyirányú üzenetküldés, a fájlátvitel és a kérés-válasz módszerek.
* Egyéb Azure-szolgáltatásokhoz való üzenettovábbítás.
* Queryable eszköz metaadatait tárolja, és szinkronizálja az állapotadatokat.
* Scure kommunikációs és hozzáférés-vezérlés eszközönkénti biztonsági kulcsok vagy X.509-tanúsítványokat használ.
* Eszköz kapcsolatot, valamint az eszköz identity management események megfigyelése.

Üzenet adatfeldolgozást tekintetében az IoT-központ hasonlít az Event Hubs. Azonban kifejezetten készült IoT-eszköz kapcsolat, nem csak üzenet adatfeldolgozást kezeléséhez. További információkért lásd: [összehasonlítása az Azure IoT-központ és az Azure Event Hubs](/azure/iot-hub/iot-hub-compare-event-hubs). 

## <a name="kafka-on-hdinsight"></a>Kafka on HDInsight

[Apache Kafka](https://kafka.apache.org/) egy nyílt forráskódú elosztott adatfolyam platform, amely valós idejű adatok folyamatok és adatfolyam-továbbítási alkalmazások készítéséhez használható. A Kafka az üzenetsorokhoz hasonló üzenetközvetítő funkciót is biztosít, amellyel adatstreameket tehet közzé, illetve feliratkozhat rájuk. Vízszintes méretezhető, hibatűrő, és nagyon gyorsan. [A HDInsight Kafka](/azure/hdinsight/kafka/apache-kafka-get-started) egy Kafka biztosít a felügyelt magas szinten méretezhető és magas rendelkezésre állású szolgáltatásként az Azure-ban. 

Néhány gyakori alkalmazási esetei Kafka a következők:

* **Üzenetküldési**. Mivel az támogatja-e a közzététel-előfizetés üzenet mintát, egy üzenet broker gyakran használt Kafka.
* **Követés tevékenység**. Mivel a Kafka sorrendben naplózási bejegyzések, használat nyomon követését, és hozza létre a tevékenységek, például egy webhelyen a felhasználói műveleteket.
* **Összesítési**. Használja a streamfeldolgozási, kombinálhatja és központosíthatja az adatokat az operatív adatok be különböző adatfolyamokba adatait is összevonhatja.
* **Átalakítás**. Adatfolyam-feldolgozást használó egyesítheti, és funkciógazdagabbá teheti az adatokat több bemeneti témakör egy vagy több kimeneti témaköröket.

## <a name="key-selection-criteria"></a>Kulcs kiválasztási feltételek

A választási lehetőségek szűkítéséhez indítása ezen kérdések megválaszolásával:

- Az IoT-eszközök és az Azure közötti kétirányú kommunikációs kell? Ha igen, válassza ki az IoT-központot.

- Van szüksége az egyes eszközök kezelésére, és tudni visszavonhatja a hozzáférést egy adott eszközhöz? Ha igen, válassza ki az IoT-központot.

## <a name="capability-matrix"></a>Képesség mátrix

A következő táblázat összefoglalja a főbb változásai képességeit. 

| | IoT Hub | Event Hubs | Kafka on HDInsight |
| --- | --- | --- | --- |
| Felhő eszközre kommunikáció | Igen | Nem | Nem |
| Eszköz által kezdeményezett fájl feltöltése | Igen | Nem | Nem |
| Eszköz állapota adatai | [Eszköz twins](/azure/iot-hub/iot-hub-devguide-device-twins) | Nem | Nem |
| Protokolltámogatás | MQTT, AMQP, HTTPS <sup>1</sup> | AMQP, HTTPS | [Kafka protokoll](https://cwiki.apache.org/confluence/display/KAFKA/A+Guide+To+The+Kafka+Protocol) |
| Biztonság | Eszköz identitás; visszavonható hozzáférés-vezérlés. | Megosztott elérési házirendeket; közzétevői házirendek korlátozott visszavonására. | Hitelesítés SASL; moduláris engedélyezési; támogatja külső hitelesítési szolgáltatások integrációja. |

[1] használhatja [Azure IoT protokoll-átjáró](/azure/iot-hub/iot-hub-protocol-gateway) ahhoz, hogy az IoT hub protokoll kiigazítása egyéni átjáróként.

További információkért lásd: [összehasonlítása az Azure IoT-központ és az Azure Event Hubs](/azure/iot-hub/iot-hub-compare-event-hubs).
