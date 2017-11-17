---
title: "Válaszfal minta"
description: "Elemek készletekbe vonni az alkalmazások elkülönítése, hogy ha nem sikerül, a többi tovább működik"
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: a2c499d77fafc4bee6b74ee0e0d84e6c23b47851
ms.sourcegitcommit: b0482d49aab0526be386837702e7724c61232c60
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 11/14/2017
---
# <a name="bulkhead-pattern"></a>Válaszfal minta

Különítse el a készletekbe kérelem elemei, hogy ha nem sikerül, a többi tovább működik.

Ebben a mintában nevű *válaszfal* , mert azt egy szállítási rendelkező üzenetpanelt partíciója hasonlít. Ha az egy szállítási rendelkező biztonsága sérül, csak a sérült szakasz beírja vízjel, amely megakadályozza a szállítási elhelyezés. 

## <a name="context-and-problem"></a>A környezetben, és probléma

A felhő alapú alkalmazás tartalmazhat több szolgáltatás minden egyes szolgáltatással, hogy egy vagy több fogyasztó számára. Túlterhelés vagy hiba történt a szolgáltatás hatással van-e a szolgáltatás összes ügyfelet.

Ezenkívül egy végfelhasználói előfordulhat, hogy kérelmet küldeni több szolgáltatás egyidejűleg, erőforrások használatával az egyes kérelmek. Az ügyfél elküldi a kérelmet egy szolgáltatás, amely helytelenül van konfigurálva, vagy nem válaszol, amikor az ügyfél kérelem által használt erőforrások előfordulhat, hogy nem szabadul fel időben. A szolgáltatás tovább kérelmeket, mert ezeket az erőforrásokat is kimerül. Például az ügyfél kapcsolatot is lehet kimerült. Ezen a ponton kérelmek más szolgáltatások a fogyasztó által érintett. Végül a fogyasztó nem kéréseket küldhetnek más szolgáltatásokkal, nem csak az eredeti nem válaszoló szolgáltatás.

Az erőforrás-fogyási ugyanazon probléma azokat a szolgáltatásokat, több felhasználóból. Egyik ügyféltől származó kérelmek nagy számú is lefoglalhat a szolgáltatásban elérhető erőforrások. Más felhasználói nem tudnak a szolgáltatás felhasználásához effektus a kaszkádolt hibája okozza.

## <a name="solution"></a>Megoldás

A szolgáltatáspéldány partíció különböző csoportok fogyasztói terhelés és a rendelkezésre állási követelmények alapján. Ez a kialakítás segít elkülöníteni a hibákat, és lehetővé teszi szolgáltatási szolgáltatások fenntartása az egyes fogyasztók hiba esetén.

A fogyasztó is képes partícióazonosító erőforrásokat, győződjön meg arról, hogy egy szolgáltatás meghívásához használt erőforrásokat egy másik szolgáltatás hívásához használt erőforrások nem befolyásolják. Például, amely behívja több szolgáltatás ügyféllel rendelt minden egyes szolgáltatás kapcsolatkészlet. Ha egy szolgáltatás megkezdi az sikertelen lesz, csak az adott szolgáltatáshoz, lehetővé téve az ügyfélnek továbbra is használja a többi szolgáltatáshoz hozzárendelt kapcsolatkészlet hatással van.

Ez a minta a következő előnyöket nyújtja:

- Elkülöníti a fogyasztók és szolgáltatások az egymásra épülő hibák. Lehet, hogy egy felhasználó vagy a szolgáltatást érintő probléma elkülönített belül a saját válaszfal, akadályozza meg, hogy a teljes megoldás is megfelelően működjenek.
- Lehetővé teszi bizonyos funkciók szolgáltatás meghibásodása megőrzése érdekében. Szolgáltatások és funkciók az alkalmazás továbbra is működnek majd.
- Szolgáltatások telepítését, amelyek a felhasználó alkalmazásokban különböző szolgáltatásminőség lehetővé teszi lehetővé. Egy magas prioritású fogyasztói készlet beállítható úgy, hogy fontos szolgáltatások használatára. 

Az alábbi ábrán látható kapcsolatkészletek hívó egyes szolgáltatások köré strukturált válaszfalak. Ha a szolgáltatás A nem sikerül, vagy valamilyen egyéb miatt nem probléma, a kapcsolatot a kapcsolatkészletből, elkülönített, ezért a szolgáltatáshoz rendelt szálkészlet csak munkaterhelés egy befolyásolja. Szolgáltatás B és C használó munkaterhelések nem érinti, és folytathatja megszakítás nélkül.

![](./_images/bulkhead-1.png) 

A következő ábrán egy egyetlen szolgáltatást több ügyfél. Minden egyes ügyfélnek külön szolgáltatáspéldány van hozzárendelve. Ügyfél 1 túl sok kérelmet és a példány túlterhelik. Minden szolgáltatáspéldány el különítve a többi, mert az ügyfelek továbbra is hívások.

![](./_images/bulkhead-2.png)
     
## <a name="issues-and-considerations"></a>Problémákat és szempontok

- Definiáljon partíciókat a üzleti és műszaki követelményeit, az alkalmazás körül.
- Particionálás és fogyasztók válaszfalak be, vegye figyelembe a technológiai, valamint a terhelés költségének, teljesítményének és kezelhetőség szempontjából által kínált elkülönítési szintjét.
- Vegye figyelembe a válaszfalak kombinálásával ismételt próbálkozás után, az áramköri megszakító, és a szabályozás minták kifinomultabb tartalék kezelési biztosításához.
- Válaszfalak a fogyasztók particionálásakor érdemes folyamatok, a szál készletek és a szemaforok. Például a projektek [Netflix Hystrix] [ hystrix] és [Polly] [ polly] keretrendszere, amely fogyasztói válaszfalak létrehozása kínálnak.
- Szolgáltatások a válaszfalak particionálásakor fontolja meg olyan önálló virtuális gépek, a tárolók és a folyamatok üzembe őket. Tárolók rövid egyensúlyt erőforrás-elkülönítést, viszonylag alacsony terhelés mellett.
- Aszinkron üzenetek használatával kommunikáló szolgáltatások el lehet különíteni a várólisták más-más részhalmazához. Minden várólista rendelkezhet egy dedikált példánya a várólista, vagy egy különálló csoportot olyan algoritmussal created és feldolgozási mennyi példányok üzenetek feldolgozása.
- Meghatározza a válaszfalak részletességgel szintjét. Például ha azt szeretné, partíciók bérlők szét, akkor sikerült helyezze mindegyik bérlő külön partíción, put több bérlő több partícióra.
- Mindegyik partíció teljesítmény- és SLA figyelése.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes használni ezt a mintát

A minta használata:

- Különítse el a használt háttér szolgáltatások felhasználásához, különösen akkor, ha az alkalmazás biztosíthat bizonyos fokú funkciót, akkor is, ha egy szolgáltatás nem válaszol.
- Különítse el a fogyasztói kritikus fogyasztók.
- Az alkalmazás a kaszkádolt meghibásodásával szembeni védelemhez.

Ez a minta nem alkalmasak lehetnek esetén:

- Erőforrások kevésbé hatékony használatát nem lehet a projekt elfogadható.
- Nincs szükség a hozzáadott összetettsége

## <a name="example"></a>Példa

A következő Kubernetes konfigurációs fájl létrehoz egy elszigetelt tárolót, hogy egyetlen szolgáltatás, futtassa a saját CPU és memória-erőforrásokat és a határértékeket.

```yml
apiVersion: v1
kind: Pod
metadata:
  name: drone-management
spec:
  containers:
  - name: drone-management-container
    image: drone-service
    resources:
      requests:
        memory: "64Mi"
        cpu: "250m"
      limits:
        memory: "128Mi"
        cpu: "1"
```

## <a name="related-guidance"></a>Kapcsolódó útmutató

- [Áramköri megszakító minta](./circuit-breaker.md)
- [Rugalmas alkalmazások tervezése az Azure számára](../resiliency/index.md)
- [Ismételje meg a minta](./retry.md)
- [Sávszélesség-szabályozási minta](./throttling.md)


<!-- links -->

[hystrix]: https://github.com/Netflix/Hystrix
[polly]: https://github.com/App-vNext/Polly