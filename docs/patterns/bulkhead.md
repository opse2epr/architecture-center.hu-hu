---
title: Válaszfal minta
titleSuffix: Cloud Design Patterns
description: Készletekbe választja szét egy alkalmazás elemeit, hogy ha az egyik meghibásodna, a többi tovább üzemeljen.
keywords: tervezési minta
author: dragon119
ms.date: 06/23/2017
ms.topic: design-pattern
ms.service: architecture-center
ms.subservice: cloud-fundamentals
ms.custom: seodec18
ms.openlocfilehash: 1ad646f5c8f4b329b0d0e9a29c83e86848b13ab0
ms.sourcegitcommit: 1b50810208354577b00e89e5c031b774b02736e2
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/23/2019
ms.locfileid: "54488341"
---
# <a name="bulkhead-pattern"></a>Válaszfal minta

Készletekbe választja szét egy alkalmazás elemeit, hogy ha az egyik meghibásodna, a többi tovább üzemeljen.

Ezt a mintát *Válaszfal* mintának nevezik, mert egy hajótest felosztott részeire emlékeztet. Ha a hajótest megsérül, csak a sérült szakasz telik meg vízzel, így a hajó nem süllyed el.

## <a name="context-and-problem"></a>Kontextus és probléma

Egy felhőalapú alkalmazás több szolgáltatást is tartalmazhat, és minden szolgáltatás egy vagy több fogyasztóval is rendelkezhet. Ha egy szolgáltatás túl van terhelve vagy meghibásodik, az minden fogyasztót érint.

Továbbá egy fogyasztó egyszerre több szolgáltatáshoz is küldhet kéréseket, és minden kéréshez erőforrásokat használ fel. Ha egy fogyasztó kérést küld egy rosszul konfigurált vagy nem válaszoló szolgáltatásnak, előfordulhat, hogy az ügyfél kérése által felhasznált erőforrások hogy nem szabadulnak fel megfelelő időn belül. Ahogy újabb kérések érkeznek a szolgáltatáshoz, ezek az erőforrások kimerülhetnek. Kimerülhet például az ügyfél kapcsolatkészlete. Ez már a fogyasztó más szolgáltatásokhoz küldött kéréseire is kihat. Végül a fogyasztó már más szolgáltatásoknak sem fog tudni kéréseket küldeni, nem csak az eredeti nem válaszoló szolgáltatásnak.

Ugyanez az erőforrásfogyás a több fogyasztóval rendelkező szolgáltatásokat is érinti. Az egy ügyféltől nagy számban érkező kérések kimeríthetik a szolgáltatás elérhető erőforrásait. Így a többi fogyasztó tudja igénybe venni a szolgáltatást, ez pedig lépcsőzetesen terjedő hibákhoz vezet.

## <a name="solution"></a>Megoldás

Particionálja a szolgáltatáspéldányokat különböző csoportokba a fogyasztók mennyisége és a rendelkezésre állási követelmények alapján. Ez a tervezés segít a hibák elkülönítésében, és meghibásodás esetén is biztosítja, hogy a szolgáltatás bizonyos fogyasztók számára elérhető legyen.

A fogyasztók szintén particionálhatják az erőforrásokat, így az egyik szolgáltatás hívásához használt erőforrások nem befolyásolják a másik szolgáltatás hívásához használtakat. Például egy több szolgáltatást is hívó fogyasztóhoz szolgáltatásonként külön kapcsolatkészlet rendelhető hozzá. Ha egy szolgáltatás kezd meghibásodni, ez csak az adott szolgáltatáshoz hozzárendelt kapcsolatkészletet érinti, így a fogyasztó tovább használhatja a többi szolgáltatást.

A minta használata többek között a következő előnyökkel jár:

- Elkülöníti a fogyasztókat és szolgáltatásokat a lépcsőzetesen terjedő hibáktól. A fogyasztókat vagy szolgáltatásokat érintő hibák elkülöníthetők a saját válaszfalukon belül, így megakadályozható a teljes megoldás meghibásodása.
- Lehetővé teszi a működésképesség bizonyos szintű fenntartását a szolgáltatás meghibásodása esetén is. Az alkalmazás többi szolgáltatása és funkciója továbbra is működhet.
- Lehetővé teszi olyan szolgáltatások üzembe helyezését, amelyek eltérő minőségű szolgáltatást biztosítanak a felhasználó alkalmazások számára. Beállítható egy magas prioritású fogyasztókészlet a magas prioritású szolgáltatások használatához.

Az alábbi ábra a különálló szolgáltatásokat hívó kapcsolatkészletek köré strukturált válaszfalakat ábrázol. Ha az A szolgáltatás meghibásodik vagy egyéb hibát okoz, a kapcsolatkészlet elkülönítése miatt a hiba csak azokat a számítási feladatokat érinti, amelyek az A szolgáltatáshoz hozzárendelt szálkészletet használják. A B és C szolgáltatásokat használó számítási feladatokat ez nem befolyásolja, és megszakítás nélkül működhetnek tovább.

![A válaszfal minta első ábrája](./_images/bulkhead-1.png)

A következő diagram több ügyfelet ábrázol, amelyek ugyanazt a szolgáltatást hívják. Minden ügyfélhez különálló szolgáltatáspéldány van hozzárendelve. Az 1. ügyfél túl sok kérést küldött, és túlterhelte a példányát. Mivel minden szolgáltatáspéldány külön van választva a többitől, a többi ügyfél továbbra is küldhet hívásokat.

![A válaszfal minta első ábrája](./_images/bulkhead-2.png)

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

- Határozza meg a partíciókat az alkalmazás üzleti és műszaki követelményei alapján.
- Amikor válaszfalakkal particionálja a szolgáltatásokat vagy fogyasztókat, vegye figyelembe a technológia által nyújtott elkülönítési szinteket és a terhelést (költségek, teljesítmény és kezelhetőség) is.
- Vegye fontolóra a válaszfalak újrapróbálkozási, áramkör-megszakító és szabályozási mintákkal történő kombinálását a kifinomultabb hibakezelés biztosítása érdekében.
- A fogyasztók válaszfalakkal történő particionálása esetén érdemes megfontolni a folyamatok, szálkészletek és szemaforok használatát. A [Netflix Hystrix][hystrix] és [Polly][polly], illetve hasonló projektek biztosítják a keretrendszert a fogyasztói válaszfalak létrehozásához.
- A szolgáltatások válaszfalakkal történő particionálása esetén érdemes megfontolni a különálló virtuális gépeken, tárolókban vagy folyamatokban történő üzembe helyezést. A tárolók viszonylag csekély többletterhelés mellett biztosítják az erőforrások elkülönítését.
- Az aszinkron üzenetekkel kommunikáló szolgáltatások különböző üzenetsor-készletekkel különíthetők el. Minden üzenetsor rendelkezhet egy dedikált példánykészlettel, amely feldolgozza az üzenetsorban található üzeneteket, vagy egyetlen példánycsoporttal, amely egy algoritmussal eltávolítja az üzeneteket a sorból, és kiosztja őket feldolgozásra.
- Határozza meg a választófalak részletességi szintjét. Például több partícióra kiterjedő bérlők terjeszteni szeretné, ha, sikerült helyezze el az egyes bérlők egy külön partícióban, vagy akár többet elhelyezi egy partíciót.
- Monitorozza az egyes partíciók teljesítményét és SLA-ját.

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Ez a minta a következőkre használható:

- háttérszolgáltatások készletét fogyasztó erőforrások elkülönítése, különösen, ha az alkalmazás akkor is működőképes marad valamilyen szinten, ha az egyik szolgáltatás nem válaszol;
- kritikus fontosságú fogyasztók különválasztása a standard fogyasztóktól;
- az alkalmazás védelme a lépcsőzetesen terjedő hibákkal szemben.

Nem érdemes ezt a mintát használni, ha:

- az erőforrások kevésbé hatékony felhasználása nem elfogadható a projektben;
- a hozzáadott összetettség nem szükséges.

## <a name="example"></a>Példa

Az alábbi Kubernetes konfigurációs fájl egy egyetlen szolgáltatást futtató elkülönített tárolót hoz létre, amely saját processzor- és memória-erőforrásokkal, valamint korlátokkal rendelkezik.

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

## <a name="related-guidance"></a>Kapcsolódó útmutatók

- [Rugalmas alkalmazások tervezése az Azure számára](../resiliency/index.md)
- [Áramkör-megszakító minta](./circuit-breaker.md)
- [Újrapróbálkozási minta](./retry.md)
- [Szabályozási minta](./throttling.md)

<!-- links -->

[hystrix]: https://github.com/Netflix/Hystrix
[polly]: https://github.com/App-vNext/Polly
