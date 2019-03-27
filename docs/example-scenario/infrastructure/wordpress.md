---
title: Nagymértékben skálázható és biztonságos WordPress-webhelyek
titleSuffix: Azure Example Scenarios
description: Nagymértékben skálázható és biztonságos WordPress-webhelyeket hozhat létre médiaeseményekhez.
author: david-stanford
ms.date: 09/18/2018
ms.topic: example-scenario
ms.service: architecture-center
ms.subservice: example-scenario
social_image_url: /azure/architecture/example-scenario/infrastructure/media/secure-scalable-wordpress.png
ms.openlocfilehash: 6032247dce0d090885bc560d963f1e714d91f69c
ms.sourcegitcommit: c053e6edb429299a0ad9b327888d596c48859d4a
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 03/20/2019
ms.locfileid: "58299046"
---
# <a name="highly-scalable-and-secure-wordpress-website"></a>Hatékonyan skálázható és biztonságos WordPress-webhely létrehozása

Ebben a példaforgatókönyvben olyan vállalatok, amelyek rendelkeznie kell a WordPress egy hatékonyan méretezhető és biztonságos telepített alkalmazható. Ebben a forgatókönyvben egy nagy egyezmény lett megadva, és tudta sikeresen méretezhető a megnövekedett forgalom munkamenetek hajtott. a hely központi alapul.

## <a name="relevant-use-cases"></a>Alkalmazási helyzetek

Egyéb alkalmazási helyzetek a következők:

- Media a forgalom megnövekedésénél kiváltó események.
- A tartalomkezelő rendszer WordPress használó blogok.
- Az üzleti és e-kereskedelmi webhely, amely a WordPress használata.
- A webhelyek más tartalomkezelő rendszerek használatával jönnek létre.

## <a name="architecture"></a>Architektúra

[![Az Azure-összetevőket részt vesz egy méretezhető és biztonságos WordPress üzembe helyezési architektúra áttekintése](media/secure-scalable-wordpress.png)](media/secure-scalable-wordpress.png#lightbox)

Ebben a forgatókönyvben egy Ubuntu-webkiszolgálók használó WordPress és MariaDB méretezhető és biztonságos telepítését ismertetjük. Nincsenek az ebben a forgatókönyvben az első két különböző adatfolyam-gyűjteményre a felhasználók férhetnek hozzá a webhelyen:

1. Felhasználók CDN keresztül férhetnek hozzá az előtér-webhely.
2. A CDN-t egy Azure load balancert használja, mint a forrás, és lekéri onnan nem gyorsítótárazott adatokat.
3. Az Azure load balancer elosztja a kérelmeket a virtuálisgép-méretezési csoportok webkiszolgálók.
4. A WordPress alkalmazás lekéri az összes dinamikus adatokat kívül a Maria DB fürtök, az Azure Files-ban üzemeltetett összes statikus tartalmat.
5. SSL-kulcsok tárolási Azure Key Vaultban.

A második munkafolyamat hogyan szerzők új tartalmat közreműködőként a következő:

1. Szerzők biztonságosan csatlakozhat a nyilvános VPN-átjárót.
2. VPN-hitelesítési adatait az Azure Active Directoryban tárolódik.
3. A felügyeleti helyettesítő mezőkbe, majd létrejön a kapcsolat.
4. A rendszergazda jump boxon a szerző ezután tud csatlakozni az Azure load balancer a szerzői műveletekhez részben fürt.
5. Az Azure load balancer elosztja a forgalmat a virtuális gép méretezési legyen írási hozzáférése a Maria DB fürt webalkalmazás-kiszolgálók között.
6. Új statikus tartalmat töltenek fel az Azure files és a dinamikus tartalom íródik a Maria DB fürtbe.
7. Ezeket a módosításokat replikálja a másodlagos régió rsync keresztül vagy elsődleges és tartalék kiszolgálók közötti replikáció.

### <a name="components"></a>Összetevők

- [Az Azure Content Delivery Network (CDN)](/azure/cdn/cdn-overview) kiszolgálók olyan elosztott hálózata, amely hatékonyan kézbesíti a webes tartalmat a felhasználók számára. CDN tárolja a gyorsítótárazott tartalom peremhálózati kiszolgálókon a jelenléti pont helyeken figyeléséért, a végfelhasználók a késés minimalizálása.
- [Virtuális hálózatok](/azure/virtual-network/virtual-networks-overview) lehetővé teszik az erőforrások, például a virtuális gépeket, hogy biztonságosan kommunikálhassanak egymással, az internetes és a helyszíni hálózatok. Virtuális hálózatok adja meg az elkülönítés és Szegmentálás, szűrő és útvonal-forgalom és helyek közötti kapcsolat engedélyezését. A két hálózat virtuális hálózatok közötti társviszony-n keresztül csatlakoznak.
- [Hálózati biztonsági csoportok](/azure/virtual-network/security-overview) tartalmaznak, amelyek engedélyezik vagy megtagadják a bejövő vagy kimenő hálózati forgalmat a forrás vagy cél IP-cím, port és protokoll alapján biztonsági szabályokból álló listát. Ebben a forgatókönyvben a virtuális hálózatok a hálózati biztonsági csoport szabályait, amelyek korlátozzák a forgalmat az alkalmazás-összetevők közötti biztonságosak.
- [Terheléselosztók](/azure/load-balancer/load-balancer-overview) megfelelően szabályok és az állapotmintákat a bejövő forgalom elosztását. Egy terheléselosztót biztosít alacsony késéssel és nagy átviteli sebességet, és akár több milliónyi összes TCP és UDP-alkalmazás méretezhető. Load balancer segítségével ebben a forgatókönyvben az előtér-webkiszolgáló a content Delivery network a forgalom elosztása.
- [A Virtual machine scale sets] [ docs-vmss] segítségével hozhatja létre, és azonos elosztott terhelésű virtuális gépek egy csoportjának kezelését. A virtuálisgép-példányok száma automatikusan növelhető vagy csökkenthető a pillanatnyi igényeknek megfelelően vagy egy meghatározott ütemezés szerint. Ez a forgatókönyv - egyet az előtér-webkiszolgáló-kiszolgálók pedig tartalékként funkcionálnak tartalom, két külön virtuálisgép-méretezési csoportok használja, és egyet az előtér-webservers használja, hozzon létre új tartalmat.
- [Az Azure Files](/azure/storage/files/storage-files-introduction) egy teljes körűen felügyelt fájlmegosztást a felhőben, ebben a forgatókönyvben a WordPress tartalom mindegyikét üzemeltető biztosítja, hogy az összes virtuális géphez férhessenek hozzá az adatokhoz.
- [Az Azure Key Vault](/azure/key-vault/key-vault-overview) tárolja, és szorosan jelszavak, tanúsítványok és kulcsok való hozzáférésének szolgál.
- [Az Azure Active Directory (Azure AD)](/azure/active-directory/fundamentals/active-directory-whatis) szolgáltatás több-bérlős felhőalapú címtár és Identitáskezelés felügyeleti szolgáltatás. Ebben a forgatókönyvben az Azure AD hitelesítési szolgáltatásokat nyújt a webhely és a VPN-alagutat.

### <a name="alternatives"></a>Alternatív megoldások

- [Az SQL Server for Linux](/azure/virtual-machines/linux/sql/sql-server-linux-virtual-machines-overview) lecserélheti a MariaDB-tárolót.
- [Azure database for MySQL-hez](/azure/mysql/overview) is cserélje le a MariaDB-tárolót, ha inkább egy teljes körűen felügyelt megoldás.

## <a name="considerations"></a>Megfontolandó szempontok

### <a name="availability"></a>Rendelkezésre állás

A Virtuálisgép-példányok ebben a forgatókönyvben a két keresztül RSYNC a WordPress-tartalom és a fő tartalék kiszolgálók közötti replikálás a MariaDB-fürtökhöz között replikálja az adatokat több régióban üzembe.

Rendelkezésre állási témaköröket talál a [rendelkezésre állási ellenőrzőlista] [ availability] a az Azure Architecture Centert.

### <a name="scalability"></a>Méretezhetőség

Ebben a forgatókönyvben a két webes előtér-fürt minden egyes régióban a virtual machine scale sets használ. A méretezési csoportok az előtér-alkalmazás szinten futtató Virtuálisgép-példányok száma automatikusan méretezheti az ügyfelek igényei szerint válaszul, vagy egy meghatározott ütemezés alapján. További információkért lásd: [az automatikus méretezés a virtual machine scale sets áttekintése][docs-vmss-autoscale].

A háttér pedig egy MariaDB-fürt rendelkezésre állási csoportban. További információkért lásd: a [oktatóanyag MariaDB-fürtökhöz][mariadb-tutorial].

Méretezhetőség témaköröket talál a [méretezési ellenőrzőlista] [ scalability] a az Azure Architecture Centert.

### <a name="security"></a>Biztonság

A virtuális hálózat hálózati biztonsági csoportok által védett, és a forgalom az előtér-alkalmazás szinten be. A szabályok korlátozzák a forgalmat, hogy csak az előtér-alkalmazás szintű Virtuálisgép-példányok férhessenek hozzá a háttér adatbázis szint. Nincs kimenő internetes forgalom engedélyezve van az adatbázisszint. A támadás által elfoglalt tárterület csökkentéséhez nincs közvetlen Távoli szolgáltatásfelügyelet portjai nyitva. További információkért lásd: [Azure-beli hálózati biztonsági csoportok][docs-nsg].

Biztonságos forgatókönyvek tervezésével kapcsolatos általános útmutatásért tekintse meg a [Azure Security dokumentációja][security].

### <a name="resiliency"></a>Rugalmasság

Több régióban, az adatreplikáció és a virtual machine scale sets együtt ebben a forgatókönyvben használt Azure-terheléselosztók. A hálózati összetevők csatlakoztatott Virtuálisgép-példányok forgalom elosztását, és biztosíthatja, hogy a forgalom csak megtörténik a kifogástalan állapotú virtuális gépeket az állapot-mintavételei bele. Hálózati összetevők mindegyikét vannak fronted egy CDN-n keresztül. Ez lehetővé teszi a hálózati erőforrásokhoz, és az alkalmazás rugalmas problémákra utal, amelyek egyébként zavarja a forgalom és végfelhasználói hozzáférése.

Rugalmas forgatókönyvek tervezésével kapcsolatos általános útmutatásért lásd: [rugalmas alkalmazások tervezése az Azure][resiliency].

## <a name="pricing"></a>Díjszabás

Ebben a forgatókönyvben költségének megismeréséhez, a szolgáltatások mindegyike a költségkalkulátor az előre konfigurált. Tekintse meg, hogyan díjszabását szeretné módosítani az adott használati esetekhez, módosítsa a megfelelő változókat egyezik a várt forgalomhoz.

Biztosítunk egy előre konfigurált [profil költség] [ pricing] fenti Architektúradiagram alapján. Konfigurálja a díjkalkulátor az használati esetekhez, van néhány fő szempontjait:

- Érkező forgalom mennyiségének, sebességhez GB/hó szempontjából? Forgalom mennyisége a legnagyobb hatással lesz a a költség, mert hatással lesz, amelyek szükségesek a surface az adatokat a virtuálisgép-méretezési csoportban lévő virtuális gépek számát. Ezenkívül azt fogja közvetlenül összekapcsolását, amely akkor jelenik meg a CDN-en keresztül adatok mennyisége az.
- Új adatok mennyiségét lesz a webhelye írásakor kell? A webhely írt új adathoz utal. a régiók közötti tükrözött adatok mennyiségét.
- A tartalom mekkora dinamikus? Mekkora az statikus? A CDN a dinamikus és statikus tartalmak hatású mennyi adatot rendelkezik kérdezhetők le és hogyan lehet az adatbázisszint sokkal eltéréseinek kerül végrehajtásra.

<!-- links -->
[architecture]: ./media/architecture-secure-scalable-wordpress.png
[mariadb-tutorial]: /azure/virtual-machines/linux/classic/mariadb-mysql-cluster
[docs-vmss]: /azure/virtual-machine-scale-sets/overview
[docs-vmss-autoscale]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview
[docs-nsg]: /azure/virtual-network/security-overview
[security]: /azure/security/
[availability]: ../../checklist/availability.md
[resiliency]: /azure/architecture/resiliency/
[scalability]: /azure/architecture/checklist/scalability
[pricing]: https://azure.com/e/a8c4809dab444c1ca4870c489fbb196b
