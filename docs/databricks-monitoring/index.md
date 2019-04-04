---
title: Az Azure Databricks monitorozása az Azure Monitor segítségével
description: Egy Scala-kódtár, amely lehetővé teszi az Azure Databricks monitorozását az Azure Log Analyticsben
author: petertaylor9999
ms.date: 03/26/2019
ms.openlocfilehash: 4544d3abc3264ec459a80ac1a61a912e6d30d6b2
ms.sourcegitcommit: 1a3cc91530d56731029ea091db1f15d41ac056af
ms.translationtype: HT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/03/2019
ms.locfileid: "58887692"
---
# <a name="monitoring-azure-databricks"></a>Az Azure Databricks monitorozása

[Az Azure Databricks](/azure/azure-databricks/) egy gyors, hatékony [Apache Spark](https://spark.apache.org/)-alapú elemzési szolgáltatás, amely megkönnyíti a big data-elemzések és a mesterségesintelligencia-megoldások gyors fejlesztését és üzembe helyezését. Rengeteg felhasználó használja ki a notebookok nyújtotta egyszerűség előnyeit az Azure Databricks-megoldásokban. A robusztusabb számítási lehetőségeket igénylő felhasználók esetében az Azure Databricks támogatja az egyéni alkalmazáskód elosztott végrehajtását.

A monitorozás kritikus részét képezi az éles megoldásoknak, így az Azure Databricks hatékony funkciókat kínál az egyéni alkalmazásmetrikák, a streamelési lekérdezési események és az alkalmazásnapló-üzenetek monitorozásához. Az Azure Databricks továbbítani tudja ezeket a monitorozási adatokat a különböző naplózási szolgáltatásoknak.

Az alábbi cikkek a monitorozási adatok az Azure Databricksből az [Azure Monitorba](/azure/azure-monitor/overview), az Azure monitorozási adatok számára fenntartott platformjára való továbbításának módját mutatják be. A cikkeket sorrendben olvassa el.

1. [Az Azure Databricks konfigurálása a metrikák az Azure Monitornak történő továbbítására](./configure-cluster.md)
1. [Azure Databricks-alkalmazásnaplók küldése az Azure Monitornak](./application-logs.md)
1. [Irányítópultok használata az Azure Databricks-metrikák megjelenítésére](./dashboards.md)

A cikkekhez kapcsolódó kódtár kibővíti az Azure Databricks alapvető monitorozási funkcióját, hogy a Spark-metrikákat, eseményeket és naplózási információkat is továbbítsa az Azure Monitornak.

A cikkeket és a kiegészítő kódtárat az Azure Spark és az Azure Databricks megoldásfejlesztőinek szánták. A kódot Java Archive (JAR-) fájlokba kell beépíteni, majd üzembe kell helyezni egy Azure Databricks-fürtön. A kód [Scala](https://www.scala-lang.org/) és Java nyelven íródott, és [Maven](https://maven.apache.org) projektobjektum-modell (POM-) fájlokat tartalmaz a kimeneti JAR-fájlok létrehozásához. A Java, a Scala és a Maven ismerete a javasolt előfeltételek közé tartozik.

## <a name="next-steps"></a>További lépések

Első lépésként hozza létre a kódtárat, és helyezze üzembe az Azure Databricks-fürtön.

> [!div class="nextstepaction"]
> [Az Azure Databricks konfigurálása a metrikák az Azure Monitornak történő továbbítására](./configure-cluster.md)