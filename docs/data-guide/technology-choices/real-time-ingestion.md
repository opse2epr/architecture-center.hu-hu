---
title: "A valós idejű üzenet adatfeldolgozást technológia kiválasztása"
description: 
author: zoinerTejada
ms:date: 02/12/2018
ms.openlocfilehash: 4f76e63a50c1d689ea3a37219a44aa94477a2e2e
ms.sourcegitcommit: 90cf2de795e50571d597cfcb9b302e48933e7f18
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/14/2018
---
# <a name="choosing-a-real-time-message-ingestion-technology-in-azure"></a><span data-ttu-id="50d06-102">Az Azure-ban a valós idejű üzenet adatfeldolgozást technológia kiválasztása</span><span class="sxs-lookup"><span data-stu-id="50d06-102">Choosing a real-time message ingestion technology in Azure</span></span>

<span data-ttu-id="50d06-103">Valós idejű feldolgozására, amelyek rögzítve lesznek a valós idejű és a feldolgozott minimális késéssel adatstreamek foglalkozik.</span><span class="sxs-lookup"><span data-stu-id="50d06-103">Real time processing deals with streams of data that are captured in real-time and processed with minimal latency.</span></span> <span data-ttu-id="50d06-104">Sok valós idejű feldolgozással megoldás adatfeldolgozást üzenettároló puffer üzenetek, illetve kibővített feldolgozás, a megbízható és egyéb üzenet Üzenetsor-kezelés szemantikáját támogatásához szükséges.</span><span class="sxs-lookup"><span data-stu-id="50d06-104">Many real-time processing solutions need a message ingestion store to act as a buffer for messages, and to support scale-out processing, reliable delivery, and other message queuing semantics.</span></span> 

## <a name="what-are-your-options-for-real-time-message-ingestion"></a><span data-ttu-id="50d06-105">Mik a valós idejű üzenet adatfeldolgozást lehetőségei?</span><span class="sxs-lookup"><span data-stu-id="50d06-105">What are your options for real-time message ingestion?</span></span>

- [<span data-ttu-id="50d06-106">Azure Event Hubs</span><span class="sxs-lookup"><span data-stu-id="50d06-106">Azure Event Hubs</span></span>](/azure/event-hubs/)
- [<span data-ttu-id="50d06-107">Azure IoT Hub</span><span class="sxs-lookup"><span data-stu-id="50d06-107">Azure IoT Hub</span></span>](/azure/iot-hub/)
- [<span data-ttu-id="50d06-108">Kafka on HDInsight</span><span class="sxs-lookup"><span data-stu-id="50d06-108">Kafka on HDInsight</span></span>](/azure/hdinsight/kafka/apache-kafka-get-started)

## <a name="azure-event-hubs"></a><span data-ttu-id="50d06-109">Azure Event Hubs</span><span class="sxs-lookup"><span data-stu-id="50d06-109">Azure Event Hubs</span></span>

<span data-ttu-id="50d06-110">[Az Azure Event Hubs](/azure/event-hubs/) egy kiválóan méretezhető adatfolyam platform és esemény adatfeldolgozást szolgáltatás, képes fogadása és feldolgozása több millió esemény / másodperc.</span><span class="sxs-lookup"><span data-stu-id="50d06-110">[Azure Event Hubs](/azure/event-hubs/) is a highly scalable data streaming platform and event ingestion service, capable of receiving and processing millions of events per second.</span></span> <span data-ttu-id="50d06-111">Az Event Hubs képes az elosztott szoftverek és eszközök által generált események, adatok vagy telemetria feldolgozására és tárolására.</span><span class="sxs-lookup"><span data-stu-id="50d06-111">Event Hubs can process and store events, data, or telemetry produced by distributed software and devices.</span></span> <span data-ttu-id="50d06-112">Az eseményközpontokba elküldött adatok bármilyen valós idejű elemzési szolgáltató vagy kötegelési/tárolóadapter segítségével átalakíthatók és tárolhatók.</span><span class="sxs-lookup"><span data-stu-id="50d06-112">Data sent to an event hub can be transformed and stored using any real-time analytics provider or batching/storage adapters.</span></span> <span data-ttu-id="50d06-113">Az Event Hubs a közzétételi-feliratkozási képességek, nagy léptékű, így a big data forgatókönyvéhez megfelelő kisebb késést biztosít.</span><span class="sxs-lookup"><span data-stu-id="50d06-113">Event Hubs provides publish-subscribe capabilities with low latency at massive scale, which makes it appropriate for big data scenarios.</span></span>

## <a name="azure-iot-hub"></a><span data-ttu-id="50d06-114">Azure IoT Hub</span><span class="sxs-lookup"><span data-stu-id="50d06-114">Azure IoT Hub</span></span>

<span data-ttu-id="50d06-115">[Az Azure IoT Hub](/azure/iot-hub/) egy felügyelt szolgáltatás, amely lehetővé teszi az IoT-eszközök millióira és a felhő alapú háttérből közötti megbízható és biztonságos kétirányú kommunikációt.</span><span class="sxs-lookup"><span data-stu-id="50d06-115">[Azure IoT Hub](/azure/iot-hub/) is a managed service that enables reliable and secure bidirectional communications between millions of IoT devices and a cloud-based back end.</span></span>

<span data-ttu-id="50d06-116">Az IoT-központ szolgáltatás a következők:</span><span class="sxs-lookup"><span data-stu-id="50d06-116">Feature of IoT Hub include:</span></span>

* <span data-ttu-id="50d06-117">Több beállítások eszközről a felhőbe és felhő eszközre kommunikációhoz.</span><span class="sxs-lookup"><span data-stu-id="50d06-117">Multiple options for device-to-cloud and cloud-to-device communication.</span></span> <span data-ttu-id="50d06-118">A lehetőségek közé tartozik az egyirányú üzenetküldés, a fájlátvitel és a kérés-válasz módszerek.</span><span class="sxs-lookup"><span data-stu-id="50d06-118">These options include one-way messaging, file transfer, and request-reply methods.</span></span>
* <span data-ttu-id="50d06-119">Egyéb Azure-szolgáltatásokhoz való üzenettovábbítás.</span><span class="sxs-lookup"><span data-stu-id="50d06-119">Message routing to other Azure services.</span></span>
* <span data-ttu-id="50d06-120">Queryable eszköz metaadatait tárolja, és szinkronizálja az állapotadatokat.</span><span class="sxs-lookup"><span data-stu-id="50d06-120">Queryable store for device metadata and synchronized state information.</span></span>
* <span data-ttu-id="50d06-121">Scure kommunikációs és hozzáférés-vezérlés eszközönkénti biztonsági kulcsok vagy X.509-tanúsítványokat használ.</span><span class="sxs-lookup"><span data-stu-id="50d06-121">Scure communications and access control using per-device security keys or X.509 certificates.</span></span>
* <span data-ttu-id="50d06-122">Eszköz kapcsolatot, valamint az eszköz identity management események megfigyelése.</span><span class="sxs-lookup"><span data-stu-id="50d06-122">Monitoring of device connectivity and device identity management events.</span></span>

<span data-ttu-id="50d06-123">Üzenet adatfeldolgozást tekintetében az IoT-központ hasonlít az Event Hubs.</span><span class="sxs-lookup"><span data-stu-id="50d06-123">In terms of message ingestion, IoT Hub is similar to Event Hubs.</span></span> <span data-ttu-id="50d06-124">Azonban kifejezetten készült IoT-eszköz kapcsolat, nem csak üzenet adatfeldolgozást kezeléséhez.</span><span class="sxs-lookup"><span data-stu-id="50d06-124">However, it was specifically designed for managing IoT device connectivity, not just message ingestion.</span></span> <span data-ttu-id="50d06-125">További információkért lásd: [összehasonlítása az Azure IoT-központ és az Azure Event Hubs](/azure/iot-hub/iot-hub-compare-event-hubs).</span><span class="sxs-lookup"><span data-stu-id="50d06-125">For more information, see [Comparison of Azure IoT Hub and Azure Event Hubs](/azure/iot-hub/iot-hub-compare-event-hubs).</span></span> 

## <a name="kafka-on-hdinsight"></a><span data-ttu-id="50d06-126">Kafka on HDInsight</span><span class="sxs-lookup"><span data-stu-id="50d06-126">Kafka on HDInsight</span></span>

<span data-ttu-id="50d06-127">[Apache Kafka](https://kafka.apache.org/) egy nyílt forráskódú elosztott adatfolyam platform, amely valós idejű adatok folyamatok és adatfolyam-továbbítási alkalmazások készítéséhez használható.</span><span class="sxs-lookup"><span data-stu-id="50d06-127">[Apache Kafka](https://kafka.apache.org/) is an open-source distributed streaming platform that can be used to build real-time data pipelines and streaming applications.</span></span> <span data-ttu-id="50d06-128">A Kafka az üzenetsorokhoz hasonló üzenetközvetítő funkciót is biztosít, amellyel adatstreameket tehet közzé, illetve feliratkozhat rájuk.</span><span class="sxs-lookup"><span data-stu-id="50d06-128">Kafka also provides message broker functionality similar to a message queue, where you can publish and subscribe to named data streams.</span></span> <span data-ttu-id="50d06-129">Vízszintes méretezhető, hibatűrő, és nagyon gyorsan.</span><span class="sxs-lookup"><span data-stu-id="50d06-129">It is horizontally scalable, fault-tolerant, and extremely fast.</span></span> <span data-ttu-id="50d06-130">[A HDInsight Kafka](/azure/hdinsight/kafka/apache-kafka-get-started) egy Kafka biztosít a felügyelt magas szinten méretezhető és magas rendelkezésre állású szolgáltatásként az Azure-ban.</span><span class="sxs-lookup"><span data-stu-id="50d06-130">[Kafka on HDInsight](/azure/hdinsight/kafka/apache-kafka-get-started) provides a Kafka as a managed, highly scalable, and highly available service in Azure.</span></span> 

<span data-ttu-id="50d06-131">Néhány gyakori alkalmazási esetei Kafka a következők:</span><span class="sxs-lookup"><span data-stu-id="50d06-131">Some common use cases for Kafka are:</span></span>

* <span data-ttu-id="50d06-132">**Üzenetküldési**.</span><span class="sxs-lookup"><span data-stu-id="50d06-132">**Messaging**.</span></span> <span data-ttu-id="50d06-133">Mivel az támogatja-e a közzététel-előfizetés üzenet mintát, egy üzenet broker gyakran használt Kafka.</span><span class="sxs-lookup"><span data-stu-id="50d06-133">Because it supports the publish-subscribe message pattern, Kafka is often used as a message broker.</span></span>
* <span data-ttu-id="50d06-134">**Követés tevékenység**.</span><span class="sxs-lookup"><span data-stu-id="50d06-134">**Activity tracking**.</span></span> <span data-ttu-id="50d06-135">Mivel a Kafka sorrendben naplózási bejegyzések, használat nyomon követését, és hozza létre a tevékenységek, például egy webhelyen a felhasználói műveleteket.</span><span class="sxs-lookup"><span data-stu-id="50d06-135">Because Kafka provides in-order logging of records, it can be used to track and re-create activities, such as user actions on a web site.</span></span>
* <span data-ttu-id="50d06-136">**Összesítési**.</span><span class="sxs-lookup"><span data-stu-id="50d06-136">**Aggregation**.</span></span> <span data-ttu-id="50d06-137">Használja a streamfeldolgozási, kombinálhatja és központosíthatja az adatokat az operatív adatok be különböző adatfolyamokba adatait is összevonhatja.</span><span class="sxs-lookup"><span data-stu-id="50d06-137">Using stream processing, you can aggregate information from different streams to combine and centralize the information into operational data.</span></span>
* <span data-ttu-id="50d06-138">**Átalakítás**.</span><span class="sxs-lookup"><span data-stu-id="50d06-138">**Transformation**.</span></span> <span data-ttu-id="50d06-139">Adatfolyam-feldolgozást használó egyesítheti, és funkciógazdagabbá teheti az adatokat több bemeneti témakör egy vagy több kimeneti témaköröket.</span><span class="sxs-lookup"><span data-stu-id="50d06-139">Using stream processing, you can combine and enrich data from multiple input topics into one or more output topics.</span></span>

## <a name="key-selection-criteria"></a><span data-ttu-id="50d06-140">Kulcs kiválasztási feltételek</span><span class="sxs-lookup"><span data-stu-id="50d06-140">Key selection criteria</span></span>

<span data-ttu-id="50d06-141">A választási lehetőségek szűkítéséhez indítása ezen kérdések megválaszolásával:</span><span class="sxs-lookup"><span data-stu-id="50d06-141">To narrow the choices, start by answering these questions:</span></span>

- <span data-ttu-id="50d06-142">Az IoT-eszközök és az Azure közötti kétirányú kommunikációs kell?</span><span class="sxs-lookup"><span data-stu-id="50d06-142">Do you need two-way communication between your IoT devices and Azure?</span></span> <span data-ttu-id="50d06-143">Ha igen, válassza ki az IoT-központot.</span><span class="sxs-lookup"><span data-stu-id="50d06-143">If so, choose IoT Hub.</span></span>

- <span data-ttu-id="50d06-144">Van szüksége az egyes eszközök kezelésére, és tudni visszavonhatja a hozzáférést egy adott eszközhöz?</span><span class="sxs-lookup"><span data-stu-id="50d06-144">Do you need to manage access for individual devices and be able to revoke access to a specific device?</span></span> <span data-ttu-id="50d06-145">Ha igen, válassza ki az IoT-központot.</span><span class="sxs-lookup"><span data-stu-id="50d06-145">If yes, choose IoT Hub.</span></span>

## <a name="capability-matrix"></a><span data-ttu-id="50d06-146">Képesség mátrix</span><span class="sxs-lookup"><span data-stu-id="50d06-146">Capability matrix</span></span>

<span data-ttu-id="50d06-147">A következő táblázat összefoglalja a főbb változásai képességeit.</span><span class="sxs-lookup"><span data-stu-id="50d06-147">The following tables summarize the key differences in capabilities.</span></span> 

| | <span data-ttu-id="50d06-148">IoT Hub</span><span class="sxs-lookup"><span data-stu-id="50d06-148">IoT Hub</span></span> | <span data-ttu-id="50d06-149">Event Hubs</span><span class="sxs-lookup"><span data-stu-id="50d06-149">Event Hubs</span></span> | <span data-ttu-id="50d06-150">Kafka on HDInsight</span><span class="sxs-lookup"><span data-stu-id="50d06-150">Kafka on HDInsight</span></span> |
| --- | --- | --- | --- |
| <span data-ttu-id="50d06-151">Felhő eszközre kommunikáció</span><span class="sxs-lookup"><span data-stu-id="50d06-151">Cloud-to-device communications</span></span> | <span data-ttu-id="50d06-152">Igen</span><span class="sxs-lookup"><span data-stu-id="50d06-152">Yes</span></span> | <span data-ttu-id="50d06-153">Nem</span><span class="sxs-lookup"><span data-stu-id="50d06-153">No</span></span> | <span data-ttu-id="50d06-154">Nem</span><span class="sxs-lookup"><span data-stu-id="50d06-154">No</span></span> |
| <span data-ttu-id="50d06-155">Eszköz által kezdeményezett fájl feltöltése</span><span class="sxs-lookup"><span data-stu-id="50d06-155">Device-initiated file upload</span></span> | <span data-ttu-id="50d06-156">Igen</span><span class="sxs-lookup"><span data-stu-id="50d06-156">Yes</span></span> | <span data-ttu-id="50d06-157">Nem</span><span class="sxs-lookup"><span data-stu-id="50d06-157">No</span></span> | <span data-ttu-id="50d06-158">Nem</span><span class="sxs-lookup"><span data-stu-id="50d06-158">No</span></span> |
| <span data-ttu-id="50d06-159">Eszköz állapota adatai</span><span class="sxs-lookup"><span data-stu-id="50d06-159">Device state information</span></span> | [<span data-ttu-id="50d06-160">Eszköz twins</span><span class="sxs-lookup"><span data-stu-id="50d06-160">Device twins</span></span>](/azure/iot-hub/iot-hub-devguide-device-twins) | <span data-ttu-id="50d06-161">Nem</span><span class="sxs-lookup"><span data-stu-id="50d06-161">No</span></span> | <span data-ttu-id="50d06-162">Nem</span><span class="sxs-lookup"><span data-stu-id="50d06-162">No</span></span> |
| <span data-ttu-id="50d06-163">Protokolltámogatás</span><span class="sxs-lookup"><span data-stu-id="50d06-163">Protocol support</span></span> | <span data-ttu-id="50d06-164">MQTT, AMQP, HTTPS <sup>1</sup></span><span class="sxs-lookup"><span data-stu-id="50d06-164">MQTT, AMQP, HTTPS <sup>1</sup></span></span> | <span data-ttu-id="50d06-165">AMQP, HTTPS</span><span class="sxs-lookup"><span data-stu-id="50d06-165">AMQP, HTTPS</span></span> | [<span data-ttu-id="50d06-166">Kafka protokoll</span><span class="sxs-lookup"><span data-stu-id="50d06-166">Kafka Protocol</span></span>](https://cwiki.apache.org/confluence/display/KAFKA/A+Guide+To+The+Kafka+Protocol) |
| <span data-ttu-id="50d06-167">Biztonság</span><span class="sxs-lookup"><span data-stu-id="50d06-167">Security</span></span> | <span data-ttu-id="50d06-168">Eszköz identitás; visszavonható hozzáférés-vezérlés.</span><span class="sxs-lookup"><span data-stu-id="50d06-168">Per-device identity; revocable access control.</span></span> | <span data-ttu-id="50d06-169">Megosztott elérési házirendeket; közzétevői házirendek korlátozott visszavonására.</span><span class="sxs-lookup"><span data-stu-id="50d06-169">Shared access policies; limited revocation through publisher policies.</span></span> | <span data-ttu-id="50d06-170">Hitelesítés SASL; moduláris engedélyezési; támogatja külső hitelesítési szolgáltatások integrációja.</span><span class="sxs-lookup"><span data-stu-id="50d06-170">Authentication using SASL; pluggable authorization; integration with external authentication services supported.</span></span> |

<span data-ttu-id="50d06-171">[1] használhatja [Azure IoT protokoll-átjáró](/azure/iot-hub/iot-hub-protocol-gateway) ahhoz, hogy az IoT hub protokoll kiigazítása egyéni átjáróként.</span><span class="sxs-lookup"><span data-stu-id="50d06-171">[1] You can also use [Azure IoT protocol gateway](/azure/iot-hub/iot-hub-protocol-gateway) as a custom gateway to enable protocol adaptation for IoT Hub.</span></span>

<span data-ttu-id="50d06-172">További információkért lásd: [összehasonlítása az Azure IoT-központ és az Azure Event Hubs](/azure/iot-hub/iot-hub-compare-event-hubss).</span><span class="sxs-lookup"><span data-stu-id="50d06-172">For more information, see [Comparison of Azure IoT Hub and Azure Event Hubs](/azure/iot-hub/iot-hub-compare-event-hubss).</span></span>