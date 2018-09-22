---
title: Hatékonyan skálázható és biztonságos WordPress-webhely létrehozása
description: A forgatókönyv egy hatékonyan méretezhető és biztonságos WordPress-webhely media eseményeket létrehozásához bevált
author: david-stanford
ms.date: 09/18/2018
ms.openlocfilehash: 595220a79e872fc102d85d239f292c5ad987123f
ms.sourcegitcommit: b7e521ba317f4fcd3253c80ac0c0a355eaaa56c5
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 09/21/2018
ms.locfileid: "46534569"
---
# <a name="highly-scalable-and-secure-wordpress-website"></a>Hatékonyan skálázható és biztonságos WordPress-webhely létrehozása

Ez a mintaforgatókönyv olyan vállalatok, amelyek rendelkeznie kell a WordPress egy hatékonyan méretezhető és biztonságos telepített alkalmazható. Ebben a forgatókönyvben egy nagy egyezmény lett megadva, és tudta sikeresen méretezhető a megnövekedett forgalom munkamenetek hajtott. a hely központi alapul.

## <a name="related-use-cases"></a>Kapcsolódó alkalmazási helyzetek

Ebben a forgatókönyvben a következő használati esetek, vegye figyelembe:

* Media a forgalom megnövekedésénél kiváltó események.
* A tartalomkezelő rendszer WordPress használó blogok.
* Az üzleti és e-kereskedelmi webhely, amely a WordPress használata.
* Webhelyek és más tartalomkezelő rendszerek.

## <a name="architecture"></a>Architektúra

[![Az Azure-összetevőket részt vesz egy méretezhető és biztonságos WordPress üzembe helyezési architektúra áttekintése](media/secure-scalable-wordpress.png)](media/secure-scalable-wordpress.png#lightbox)

Ebben a forgatókönyvben egy Ubuntu-webkiszolgálók használó WordPress és MariaDB méretezhető és biztonságos telepítését ismertetjük. Nincsenek az ebben a forgatókönyvben az első két különböző adatfolyam-gyűjteményre a felhasználók férhetnek hozzá a webhelyen:

1. Felhasználók CDN keresztül férhetnek hozzá az előtér-webhely.
2. A CDN-t használja, mint az Azure load balancerben, mint a forrás, és lekéri onnan nem gyorsítótárazott adatokat.
3. Az Azure load balancer elosztja a kérelmeket a virtuálisgép-méretezési csoportok webkiszolgálók.
4. A WordPress alkalmazás lekéri az összes dinamikus adatokat kívül a Maria DB fürtök, az Azure Files-ban üzemeltetett összes statikus tartalmat.
5. SSL-kulcsok tárolási Azure Key Vaultban.

A második munkafolyamat hogyan szerzők új tartalmat közreműködőként a következő:

1. Szerzők VPN be a nyilvános VPN-átjárót.
2. VPN-hitelesítési adatait az Azure Active Directoryban tárolódik.
3. A felügyeleti helyettesítő mezőkbe, majd létrejön a kapcsolat.
4. A rendszergazda jump boxon a szerző ezután tud csatlakozni az Azure load balancer a szerzői műveletekhez részben fürt.
5. Az Azure load balancer elosztja a forgalmat a virtuális gép méretezési legyen írási hozzáférése a Maria DB fürt webalkalmazás-kiszolgálók között.
6. Új statikus tartalmat töltenek fel az Azure files és a dinamikus tartalom íródik a Maria DB fürtbe.
7. Ezeket a módosításokat replikálja a másodlagos régió rsync keresztül vagy elsődleges és tartalék kiszolgálók közötti replikáció.

### <a name="components"></a>Összetevők

* [Az Azure CDN] [ cdn-docs] olyan elosztott hálózata, a kiszolgálók, amely hatékonyan kézbesíti a webes tartalmakat a felhasználók számára. Gyorsítótárazott tartalmat tárolnak a késés minimalizálása érdekében a végfelhasználók közelében lévő jelenléti pontok helyeken peremhálózati kiszolgálókon.
* [Az Azure Virtual Network] [ vnet-docs] lehetővé teszi az erőforrások, például virtuális gépek biztonságosan kommunikálnak egymással, az internethez, és a helyszíni hálózatokkal. Virtuális hálózatok adja meg az elkülönítés és Szegmentálás, szűrő és útvonal-forgalom és helyek közötti kapcsolat engedélyezését. A két hálózat virtuális hálózatok közötti társviszony együtt kapcsolódik.
* [Azure-beli hálózati biztonsági csoportok] [ nsg-docs] tartalmaznak, amelyek engedélyezik vagy megtagadják a bejövő vagy kimenő hálózati forgalmat a forrás vagy cél IP-cím, port és protokoll alapján biztonsági szabályokból álló listát. Ebben a forgatókönyvben a virtuális hálózatok a hálózati biztonsági csoport szabályait, amelyek korlátozzák a forgalmat az alkalmazás-összetevők közötti biztonságosak.
* [Az Azure load balancer] [ loadbalancer-docs] osztja el a szabályok és az állapotmintákat megfelelően bejövő forgalmat. Egy terheléselosztót biztosít alacsony késéssel és nagy átviteli sebességet, és akár több milliónyi összes TCP és UDP-alkalmazás méretezhető. Load balancer segítségével ebben a forgatókönyvben az előtér-webkiszolgálók, a content Delivery network forgalom elosztása.
* [Azure-beli virtuálisgép-méretezési csoportok] [ scaleset-docs] hozhat létre és manager azonos, load csoportja elosztott terhelésű virtuális gépek. A virtuálisgép-példányok száma automatikusan növelhető vagy csökkenthető a pillanatnyi igényeknek megfelelően vagy egy meghatározott ütemezés szerint. Ez a forgatókönyv - egy szolgáltató tartalom, előtér-webkiszolgálón a két külön virtuálisgép-méretezési csoportokat használja, és egyet az előtér-webservers használja, hozzon létre új tartalmat.
* [Az Azure Files] [ azure-files-docs] egy teljes körűen felügyelt fájlmegosztást a felhőben, ebben a forgatókönyvben a WordPress tartalom mindegyikét üzemeltető biztosítja, hogy az összes virtuális géphez férhessenek hozzá az adatokhoz.
* [Az Azure Key Vault] [ azure-key-vault-docs] tárolja, és szorosan jelszavak, tanúsítványok és kulcsok való hozzáférésének szolgál.
* [Az Azure Active Directory] [ aad-docs] szolgáltatás több-bérlős felhőalapú címtár és Identitáskezelés felügyeleti szolgáltatás.  Ebben a forgatókönyvben szolgál hitelesítéséhez be a webhely és a VPN-alagutat.

### <a name="alternatives"></a>Alternatív megoldások

* [Az SQL Server for Linux] [ sql-linux] lecserélheti a MariaDB-tárolót.

* [Azure database for MySQL-hez] [ mysql-docs] is cserélje le a MariaDB-tárolót, ha inkább egy teljes körűen felügyelt megoldás.

## <a name="considerations"></a>Megfontolandó szempontok

### <a name="availability"></a>Rendelkezésre állás

A Virtuálisgép-példányok ebben a forgatókönyvben a két keresztül RSYNC a WordPress-tartalom és a fő tartalék kiszolgálók közötti replikálás a MariaDB-fürtökhöz között replikálja az adatokat több régióban üzembe.

Rendelkezésre állási témaköröket talál a [rendelkezésre állási ellenőrzőlista] [ availability] a az Azure Architecture Centert.

### <a name="scalability"></a>Méretezhetőség

Ebben a forgatókönyvben a két előtérbeli webes fürt minden egyes régióban a virtual machine scale sets használ. A méretezési csoportok a frontend alkalmazásrétegek futtató Virtuálisgép-példányok száma automatikusan méretezheti az ügyfelek igényei szerint válaszul, vagy egy meghatározott ütemezés alapján. További információkért lásd: [az automatikus méretezés a virtual machine scale sets áttekintése][vmssautoscale-docs].

A háttérrendszer a MariaDB-fürt egy rendelkezésre állási csoportban. További információkért lásd: a [oktatóanyag MariaDB-fürtökhöz][mariadb-tutorial].

Méretezhetőség témaköröket talál a [méretezési ellenőrzőlista] [ scalability] a az Azure Architecture Centert.

### <a name="security"></a>Biztonság

A virtuális hálózat hálózati biztonsági csoportok által védett, és a forgalom az előtér-alkalmazás szint be. A szabályok korlátozzák a forgalmat, hogy csak az előtérbeli alkalmazás szintű Virtuálisgép-példányok férhessenek hozzá a háttér adatbázis szint. Nincs kimenő internetes forgalom engedélyezve van az adatbázisszint. A támadás által elfoglalt tárterület csökkentéséhez nincs közvetlen Távoli szolgáltatásfelügyelet portjai nyitva. További információkért lásd: [Azure-beli hálózati biztonsági csoportok][nsg-docs].

Biztonságos forgatókönyvek tervezésével kapcsolatos általános útmutatásért tekintse meg a [Azure Security dokumentációja][security].

### <a name="resiliency"></a>Rugalmasság

Több régióban, az adatreplikáció és a virtual machine scale sets együtt ebben a forgatókönyvben használt Azure-terheléselosztók. A hálózati összetevők csatlakoztatott Virtuálisgép-példányok forgalom elosztását, és biztosíthatja, hogy a forgalom csak megtörténik a kifogástalan állapotú virtuális gépeket az állapot-mintavételei bele. Hálózati összetevők mindegyikét vannak fronted, ami lehetővé teszi a hálózati erőforrásokhoz, és az alkalmazás rugalmas problémákat, amelyek egyébként zavarja a forgalom és végfelhasználói hozzáférése a CDN-n keresztül.

Rugalmas forgatókönyvek tervezésével kapcsolatos általános útmutatásért lásd: [rugalmas alkalmazások tervezése az Azure][resiliency].

## <a name="pricing"></a>Díjszabás

Ebben a forgatókönyvben költségének megismeréséhez, a szolgáltatások mindegyike a költségkalkulátor az előre konfigurált.  Tekintse meg, hogyan díjszabását szeretné módosítani az adott használati esetekhez, módosítsa a megfelelő változókat egyezik a várt forgalomhoz.

Biztosítunk egy előre konfigurált [profil költség] [ pricing] fenti Architektúradiagram alapján. Konfigurálja a díjkalkulátor az használati esetekhez, van néhány fő szempontjait:

* Érkező forgalom mennyiségének, sebességhez GB/hó szempontjából? Forgalom mennyisége a legnagyobb hatással lesz a a költség, mert hatással lesz, amelyek szükségesek a surface az adatokat a virtuálisgép-méretezési csoportban lévő virtuális gépek számát.  Ezenkívül azt fogja közvetlenül összekapcsolását, amely akkor jelenik meg a CDN-en keresztül adatok mennyisége az.

* Új adatok mennyiségét lesz a webhelye írásakor kell? A webhely írt új adathoz utal. a régiók közötti tükrözött adatok mennyiségét.

* A tartalom mekkora dinamikus? Mekkora az statikus?  A CDN a dinamikus és statikus tartalmak hatású mennyi adatot rendelkezik kérdezhetők le és hogyan lehet az adatbázisszint sokkal eltéréseinek kerül végrehajtásra.

<!-- links -->
[architecture]: ./media/secure-scalable-wordpress.png
[cdn-docs]: /azure/cdn/cdn-overview
[vnet-docs]: /azure/virtual-network/virtual-networks-overview
[loadbalancer-docs]: /azure/load-balancer/load-balancer-overview
[nsg-docs]: /azure/virtual-network/security-overview
[azure-files-docs]: /azure/storage/files/storage-files-introduction
[azure-key-vault-docs]: /azure/key-vault/key-vault-overview
[aad-docs]: /azure/active-directory/fundamentals/active-directory-whatis
[mysql-docs]: /azure/mysql/overview
[sql-linux]: /azure/virtual-machines/linux/sql/sql-server-linux-virtual-machines-overview
[mariadb-tutorial]: /azure/virtual-machines/linux/classic/mariadb-mysql-cluster
[scaleset-docs]: /azure/virtual-machine-scale-sets/overview
[security]: /azure/security/
[availability]: /architecture/checklist/availability
[resiliency]: /azure/architecture/resiliency/
[scalability]: /azure/architecture/checklist/scalability
[vmssautoscale-docs]: /azure/virtual-machine-scale-sets/virtual-machine-scale-sets-autoscale-overview
[pricing]: https://azure.com/e/a8c4809dab444c1ca4870c489fbb196b