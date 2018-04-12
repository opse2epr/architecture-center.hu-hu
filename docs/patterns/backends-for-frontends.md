---
title: Háttérrendszerek és előtérrendszerek minta
description: Elkülönített, adott előtérbeli alkalmazások vagy felületek által használt háttérszolgáltatásokat hozhat létre.
author: dragon119
ms.date: 06/23/2017
ms.openlocfilehash: 2991d7a3e05b3ce6cd5148a552bae6d4ba8f7c4c
ms.sourcegitcommit: c441fd165e6bebbbbbc19854ec6f3676be9c3b25
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/30/2018
---
# <a name="backends-for-frontends-pattern"></a>Háttérrendszerek és előtérrendszerek minta

Elkülönített, adott előtérbeli alkalmazások vagy felületek által használt háttérszolgáltatásokat hozhat létre. Ez a minta akkor hasznos, ha el szeretné kerülni, hogy egyetlen háttérkiszolgálót több felülethez is testre kelljen szabnia. A mintát először Sam Newman írta le.

## <a name="context-and-problem"></a>Kontextus és probléma

Egy alkalmazás kezdetben egy asztali webes felhasználói felületet célozhat. Általában ezzel párhuzamosan fejlesztik a háttérszolgáltatást, amely az adott felhasználói felület szolgáltatásait biztosítja. Az alkalmazás felhasználói bázisának növekedésével idővel fejlesztenek egy mobilalkalmazást is, amelynek ugyanazzal a háttérrendszerrel kell együttműködnie. A háttérszolgáltatás általános célú háttérrendszerré válik, amely az asztali és a mobilfelületek igényeit egyaránt kiszolgálja.

De a képességei a mobileszközök jelentősen eltér a asztali tekintetében képernyőméret, teljesítmény, a böngésző, és megjeleníti a korlátozások. Ennek eredményeképpen a mobilalkalmazások háttérrendszereinek követelményei eltérnek az asztali webes felhasználói felületekéitől. 

Ezeknek a különbségeknek köszönhetően a háttérkiszolgálóra egymással versengő követelmények vonatkoznak. A háttérrendszert rendszeresen és jelentős mértékben módosítani szükséges, hogy az asztali webes felhasználói felületet és a mobilalkalmazást egyaránt ki tudja szolgálni. Gyakran külön felületfejlesztő csapatok dolgoznak ez egyes előtérrendszereken, így a háttérrendszer szűk keresztmetszetté válik a fejlesztési folyamatban. Az egymással ütköző frissítési követelmények és a szolgáltatás mindkét előtérrendszeren való működőképessége iránti elvárások miatt esetleg rengeteg erőfeszítést kell fektetni egyetlen üzembe helyezhető erőforrásra.

![](./_images/backend-for-frontend.png) 

Ahogy a fejlesztés a háttérszolgáltatásra összpontosít, esetleg egy külön csapat alakul a háttérrendszer kezelésére és karbantartására. Ez végül azt eredményezi, hogy megszakad a kapcsolat a felületet és a háttérrendszert fejlesztő csapatok közt, és nő a teher háttérfejlesztő csapaton, hogy megfeleljenek a különböző felületfejlesztő csapatok egymással versengő elvárásainak. Ha valamelyik felületfejlesztő csapat a háttérrendszer módosítását igényli, a változásokat érvényesíttetni kell a többi felületfejlesztő csapattal is, mielőtt integrálhatóak lennének a háttérrendszerbe. 

## <a name="solution"></a>Megoldás

Felhasználói felületenként egy háttérrendszert hozzon létre. Az egyes háttérrendszerek működését és teljesítményét az adott előtérrendszernek megfelelően finomhangolja, és ne törődjön azzal, hogy hogyan hat a többi előtérfelületre.

![](./_images/backend-for-frontend-example.png) 

Mivel mindegyik háttérrendszer egy adott felülethez tartozik, optimalizálható is hozzá. Ennek eredményeképpen kisebb, kevésbé összetett és valószínűleg gyorsabb lesz, mint egy olyan általános háttérrendszer lenne, amely az összes felület követelményeinek megpróbál megfelelni. Minden felületfejlesztő csapat autonóm módon meghatározhatja a saját háttérrendszerét, és nem függ a központi háttérrendszer-fejlesztő csapattól. Így a felületfejlesztő csapat rugalmasan határozhatja meg a nyelvválasztást, a kiadási ütemezést, a számítási feladatok prioritását és a háttérrendszerbe integrált szolgáltatásokat.

További információkért lásd: [Minta: Háttérrendszerek és előtérrendszerek](http://samnewman.io/patterns/architectural/bff/).

## <a name="issues-and-considerations"></a>Problémák és megfontolandó szempontok

- Vegye számba, hogy hány háttérrendszert kell telepíteni.
- Ha a különböző felületek (például a mobilügyfelek) azonos kéréseket adnak majd le, fontolja meg, hogy valóban érdemes-e külön háttérrendszert kialakítani minden egyes felülethez, vagy esetleg elegendő egyetlen háttérrendszer.
- Ennek a mintának az implementálásakor nagyon valószínű, hogy a kód duplikálva lesz az egyes szolgáltatásokban.
- Az egyes előterekre irányuló háttérrendszereknek csak ügyfélspecifikus logikát és működést szabad tartalmaznia. Az általános üzleti logikát és az egyéb globális szolgáltatásokat az alkalmazás valamely más részében kell kezelni.
- Gondolja végig, hogy ennek a mintának az alkalmazása miként módosítja a felelősségi köröket a fejlesztői csapatban.
- Vegye számításba, hogy mennyi ideig tart megvalósítani a mintát. Okoznak majd az új háttérrendszerek kialakításába fektetett erőfeszítések műszaki adósságot, miközben továbbra is támogatja a meglévő általános háttérrendszert?

## <a name="when-to-use-this-pattern"></a>Mikor érdemes ezt a mintát használni?

Használja ezt a mintát, ha:

- Egy megosztott vagy általános célú háttérszolgáltatást kell fenntartani jelentős fejlesztési többletterhelés mellett.
- A háttérrendszert a specifikus ügyfélfelületek követelményeihez szeretné optimalizálni.
- Az általános célú háttérrendszer testreszabásával több felületet szeretne kiszolgálni.
- Valamely másik felhasználói felület háttérrendszeréhez egy alternatív programnyelv alkalmasabb lenne.

Nem érdemes ezt a mintát használni, ha:

- Ha a felületek ugyanazokat vagy hasonló kéréseket intéznek a háttérrendszerhez.
- Ha csak egyetlen felület kommunikál a háttérrendszerrel.

## <a name="related-guidance"></a>Kapcsolódó útmutatók

- [Átjáróösszesítés minta](./gateway-aggregation.md)
- [Átjárókiürítés minta](./gateway-offloading.md)
- [Átjáró-útválasztási minta](./gateway-routing.md)


