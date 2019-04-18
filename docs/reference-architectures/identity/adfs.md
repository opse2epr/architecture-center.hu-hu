---
title: Kiterjesztheti a helyszíni AD FS az Azure-bA
titleSuffix: Azure Reference Architectures
description: Az Active Directory összevonási szolgáltatás engedélyezésével biztonságos hibrid hálózati architektúra megvalósítása az Azure-ban.
author: telmosampaio
ms.date: 12/18.2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18, identity
ms.openlocfilehash: 873b6a86da14e00d0a537f910d10922444cc1ded
ms.sourcegitcommit: 579c39ff4b776704ead17a006bf24cd4cdc65edd
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 04/17/2019
ms.locfileid: "59640736"
---
# <a name="extend-active-directory-federation-services-ad-fs-to-azure"></a>Az Active Directory összevonási szolgáltatások (AD FS) kiterjesztése az Azure-ra

Ez a referenciaarchitektúra egy olyan biztonságos hibrid hálózatot valósít meg, amely kiterjeszti a helyszíni hálózatot az Azure-ra, és az [Active Directory összevonási szolgáltatások (AD FS)][active-directory-federation-services] használatával összevont hitelesítést és engedélyezést végez az Azure-ban futó összetevők számára. [**A megoldás üzembe helyezése.**](#deploy-the-solution)

![Biztonságos hibrid hálózati architektúra az Active Directoryval](./images/adfs.png)

*Töltse le az architektúra [Visio-fájlját][visio-download].*

Az AD FS üzemeltethető a helyszínen, de ha az alkalmazás egy hibrid, amelynek egyes részei az Azure-ban futnak, az AD FS replikálása a felhőben hatékonyabb megoldás lehet.

A diagram a következő eseteket mutatja be:

- Egy partnerszervezettől származó alkalmazáskód hozzáfér egy, az Ön Azure-beli virtuális hálózatán futó webalkalmazáshoz.
- Egy külső, az Active Directory Domain Servicesben (DS) tárolt hitelesítő adatokkal rendelkező regisztrált felhasználó hozzáfér az Ön Azure-beli virtuális hálózatán futó webalkalmazáshoz.
- Egy felhasználó, aki hitelesített eszközről csatlakozik az Ön virtuális hálózatához, elindít egy webalkalmazást, amely az Ön Azure virtuális hálózatán belül fut.

Az architektúra gyakori használati módjai többek között a következők:

- Hibrid alkalmazások, amelyekben a számítási feladatok részben a helyszínen, részben pedig az Azure-ban futnak.
- Megoldások, amelyek összevont engedélyezéssel tesznek elérhetővé webalkalmazásokat partnerszervezetek számára.
- Rendszerek, amelyek támogatják a hozzáférést a vállalati tűzfalon kívül futó webböngészőknek.
- Rendszerek, amelyek lehetővé teszik a felhasználók számára a webes alkalmazásokhoz való hozzáférést hitelesített külső eszközökkel, például távoli számítógépekkel, noteszgépekkel és egyéb mobileszközökkel.

Ez a referenciaarchitektúra a *passzív összevonásra* összpontosít, amelyben az összevonási kiszolgálók döntenek arról, hogy hogyan és mikor hitelesítsenek egy adott felhasználót. A felhasználó az alkalmazás indításakor megadja a bejelentkezési adatokat. Ezt a mechanizmust legtöbbször webböngészők használják. Egy olyan protokollt alkalmaz, amely átirányítja a felhasználót arra az oldalra, ahol elvégezheti a hitelesítést. Az AD FS az *aktív összevonást* is támogatja. Ennek során egy alkalmazás felelőssége a hitelesítő adatok biztosítása, további felhasználói interakció igénylése nélkül – erre az esetre azonban ez az architektúra nem terjed ki.

További szempontok: [Megoldás választása a helyszíni Active Directory Azure-ral való integrálásához][considerations].

## <a name="architecture"></a>Architektúra

Ez az architektúra kiterjeszti [Az AD DS kiterjesztése az Azure-ra][extending-ad-to-azure] című részben leírt megvalósítást. Az alábbi összetevőket tartalmazza.

- **AD DS-alhálózat**. Az AD DS-kiszolgálók a saját alhálózatukban találhatók. Tűzfalként a hálózati biztonsági csoport (NSG) szabályai szolgálnak.

- **AD DS-kiszolgálók**. Az Azure-ban virtuális gépekként futó tartományvezérlők. Ezek a kiszolgálók végzik a helyi identitások hitelesítését a tartományon belül.

- **AD FS-alhálózat**. Az AD FS-kiszolgálók a saját alhálózatukban találhatók. Tűzfalként az NSG-szabályok szolgálnak.

- **AD FS-kiszolgálók**. Az AD FS-kiszolgálók összevont engedélyezést és hitelesítést biztosítanak. Ebben az architektúrában a következő feladatokat látják el:

  - A partner összevonási kiszolgáló által partnerfelhasználó nevében küldött jogcímeket tartalmazó biztonsági jogkivonatok fogadása. Az AD FS ellenőrzi, hogy a jogkivonatok érvényesek-e, mielőtt átadja azokat az Azure-ban futó webalkalmazásnak, amely a kérelmek engedélyezését végzi.

    Az Azure-ban futó alkalmazások a *függő*. A partner összevonási kiszolgálónak olyan jogcímeket kell kiadni, amelyeket a webalkalmazás értelmezni tud. A partner összevonási kiszolgálókat *fiókpartnereknek* nevezik, mert ezek küldenek hozzáférési kérelmeket a partnerszervezet hitelesített fiókjainak nevében. Az AD FS-kiszolgálókat *erőforráspartnereknek* nevezik, mert ezek biztosítanak hozzáférést az erőforrásokhoz (a webalkalmazáshoz).

  - Webböngészőt vagy webalkalmazás-hozzáférést igénylő eszközt futtató külső felhasználóktól érkező kérelmek hitelesítése és engedélyezése az AD DS-sel és az [Active Directory eszközregisztrációs szolgáltatásával][ADDRS].

  Az AD FS-kiszolgálók farmként vannak konfigurálva, amely egy Azure-terheléselosztón keresztül érhető el. Ez a megvalósítás javítja a rendelkezésre állást és a méretezhetőséget. Az AD FS-kiszolgálók nem érhetők el közvetlenül az internetről. Minden internetes forgalom szűrve van AD FS webalkalmazás-proxy kiszolgálókon és egy szegélyhálózaton (DMZ) keresztül.

  Információk az AD FS működéséről: [Active Directory összevonási szolgáltatások – áttekintés][active-directory-federation-services-overview]. Emellett [Az AD FS üzembe helyezése az Azure-ban][adfs-intro] című cikk lépésről lépésre bemutatja a megvalósítást.

- **AD FS proxyalhálózat**. Az AD FS-proxykiszolgálók lehetnek a saját alhálózatukon belül, ahol védelmüket az NSG-szabályok biztosítják. Az alhálózaton belüli kiszolgálók hálózati virtuális készülékeken keresztül kapcsolódnak az internethez. Ezek tűzfalat biztosítanak az Azure virtuális hálózat és az internet között.

- **AD FS webalkalmazás-proxy (WAP) kiszolgálók**. Ezek a virtuális gépek AD FS-kiszolgálókként működnek a partnerszervezetektől és a külső eszközöktől érkező kérelmek számára. A WAP-kiszolgálók szűrőként működnek, védelmet képezve az AD FS-kiszolgálók számára az internetről való közvetlen hozzáféréssel szemben. Ahogy az AD FS-kiszolgálók esetében is, a WAP-kiszolgálók farmban, terheléselosztással való üzemeltetése jobb rendelkezésre állást és méretezhetőséget biztosít, mint ha különálló kiszolgálók gyűjteményeként működtetné azokat.

  > [!NOTE]
  > Részletes információk a WAP-kiszolgálók telepítéséről: [A webalkalmazás-proxy kiszolgáló telepítése és konfigurálása][install_and_configure_the_web_application_proxy_server]
  >

- **Partnerszervezet**. Egy Azure-beli webalkalmazáshoz való hozzáférést kérő webes alkalmazást futtató partnerszervezet. A partnerszervezet összevonási kiszolgálója helyileg hitelesíti a kérelmeket, és elküldi a jogcímeket tartalmazó biztonsági jogkivonatokat az Azure-ban futó AD FS-nek. Az AD FS az Azure-ban érvényesíti a biztonsági jogkivonatokat, és az érvényeseket továbbítja az Azure-beli webalkalmazásnak engedélyezés céljából.

  > [!NOTE]
  > Emellett konfigurálhat egy VPN-alagutat is az Azure-átjáróval, amelyen keresztül közvetlen hozzáférést biztosíthat a megbízható partnerek számára. Az ezen partnerektől érkező kérelmek nem haladnak át a WAP-kiszolgálókon.
  >

## <a name="recommendations"></a>Javaslatok

Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei.

### <a name="networking-recommendations"></a>Hálózatokra vonatkozó javaslatok

A hálózati adaptert minden, AD FS- és WAP-kiszolgálót futtató virtuális géphez statikus magánhálózati IP-címmel konfigurálja.

Ne adjon nyilvános IP-címeket az AD FS-t futtató virtuális gépeknek. További információkért lásd: a [biztonsági szempontok](#security-considerations) szakaszban.

Állítsa be az elsődleges és a másodlagos tartománynév-kiszolgálók (DNS) IP-címeit az egyes AD FS és WAP virtuális gépek hálózati adaptereihez úgy, hogy az Active Directory DS virtuális gépeire hivatkozzanak. Az Active Directory DS virtuális gépeknek DNS kell futtatnia. Ez a lépés szükséges ahhoz, hogy az egyes virtuális gépek csatlakozhassanak a tartományhoz.

### <a name="ad-fs-installation"></a>Az AD FS telepítése

Az [Összevonási kiszolgálók farmjának központi telepítése][Deploying_a_federation_server_farm] című cikkben részletes útmutatás található az AD FS telepítéséhez és konfigurálásához. A farm első AD FS-kiszolgálójának konfigurálása előtt végezze el a következő műveleteket:

1. Szerezzen be egy nyilvánosan megbízható tanúsítványt a kiszolgálók hitelesítéséhez. A *tulajdonosnévnek* azt a nevet kell tartalmaznia, amelyet az ügyfelek az összevonási szolgáltatás eléréséhez használnak. Ez lehet a terheléselosztóhoz regisztrált DNS-név, például *adfs.contoso.com* (biztonsági okokból ne használjon helyettesítő karaktereket tartalmazó neveket, mint például a **.contoso.com*). Használja ugyanazt a tanúsítványt az összes AD FS-kiszolgáló virtuális gépein. Tanúsítványt megbízható hitelesítésszolgáltatótól vásárolhat, de ha a vállalata az Active Directory tanúsítványszolgáltatást használja, létrehozhatja a sajátját is.

    A *tulajdonos alternatív nevét* az eszközregisztrációs szolgáltatás (DRS) használja a külső eszközökről való hozzáférés engedélyezéséhez. Ennek a következő formátumban kell lennie: *enterpriseregistration.contoso.com*.

    További információkért lásd: [Secure Sockets Layer (SSL-) tanúsítvány beszerzése és konfigurálása][adfs_certificates].

2. A tartományvezérlőn hozzon létre egy új legfelső szintű kulcsot a kulcsszolgáltató szolgáltatás számára. Az érvényesség kezdetének adja meg az aktuális időpontnál 10 órával korábbi időpontot (ez a konfiguráció csökkenti a késést, amely a kulcsok a tartományban való kiosztásakor és szinkronizálásakor léphet fel). Ez a lépés az AD FS-szolgáltatás futtatásához használt csoportos szolgáltatásfiók létrehozásának támogatásához szükséges. A következő PowerShell-parancs erre mutat példát:

    ```powershell
    Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
    ```

3. Minden AD FS-kiszolgáló virtuális gépét vegye fel a tartományba.

> [!NOTE]
> Az AD FS telepítéséhez a tartomány elsődleges tartományvezérlő (PDC) emulátorának mozgó egyedüli főkiszolgálói (FSMO) szerepkörét futtató tartományvezérlőnek futnia kell, és elérhetőnek kell lennie az AD FS virtuális gépekről. <<RBC: Van mód, hogy ez kevesebb ismétlődő? >>
>

### <a name="ad-fs-trust"></a>AD FS-megbízhatóság

Alakítson ki összevonási megbízhatóságot az üzemelő AD FS és a partnerszervezetek összevonási kiszolgálói között. Konfigurálja az összes szükséges jogcímszűrést és -leképezést.

- Minden egyes partnerszervezet fejlesztési és üzemeltetési csapatának meg kell adnia egy függőentitás-megbízhatóságot az AD FS-kiszolgálókon keresztül elérhető webes alkalmazásokhoz.
- Vállalata fejlesztési és üzemeltetési csapatának be kell állítania a jogcímszolgáltatói megbízhatóságot ahhoz, hogy lehetővé tegye az AD FS-kiszolgálók számára a partnerszervezetek által biztosított jogcímek megbízhatóként való kezelését.
- A vállalat fejlesztési és üzemeltetési csapatának emellett azt is be kell állítania, hogy az AD FS átadja a jogcímeket a vállalat szervezetének webalkalmazásainak.

További információkért lásd: [Összevonási megbízhatóság létrehozása][establishing-federation-trust].

Tegye közzé vállalata webalkalmazásait, és tegye elérhetővé azokat a külső partnerek számára a WAP-kiszolgálókon keresztüli előhitelesítéssel. További információkért lásd: [Alkalmazások közzététele AD FS előhitelesítéssel][publish_applications_using_AD_FS_preauthentication]

Az AD FS támogatja a jogkivonatok átalakítását és kiegészítését. Az Azure Active Directory nem biztosítja ezt a funkciót. Az AD FS-sel a megbízhatósági kapcsolatok beállításakor a következőket teheti:

- Konfigurálhatja a jogcímek átalakítását az engedélyezési szabályokhoz. Például leképezheti a csoportos biztonságot egy nem Microsoft-partner szervezet úgy, hogy az Active Directory tartományi szolgáltatások hitelesítheti a szervezet által használt ábrázolásból olyan.
- Átalakíthatja a jogcímeket másik formátumba. Például leképezést végezhet a SAML 2.0-ról a SAML 1.1-re, ha alkalmazása csak a SAML 1.1 formátumú jogcímeket támogatja.

### <a name="ad-fs-monitoring"></a>AD FS-figyelés

Az [Active Directory 2012 R2 összevonási szolgáltatásokhoz készült Microsoft System Center felügyeleti csomag][oms-adfs-pack] proaktív és reaktív felügyeletet is biztosít az összevonási kiszolgálóhoz tartozó AD FS üzemelő példányához. Ez a felügyeleti csomag a következőket figyeli:

- AD FS szolgáltatás által az eseménynaplókban rögzített eseményeket.
- Az AD FS teljesítményszámlálói által gyűjtött teljesítményadatokat.
- A AD FS rendszer és a webes alkalmazások (függő entitások) általános állapotát. Emellett riasztásokat biztosít a kritikus fontosságú problémákról és figyelmeztetésekről.

## <a name="scalability-considerations"></a>Méretezési szempontok

AD FS-farmok méretezésének megkezdéséhez lásd [Az AD FS üzembe helyezésének megtervezése][plan-your-adfs-deployment] című cikkben összefoglalt szempontokat:

- Ha 1000-nél kevesebb felhasználóval rendelkezik, ne hozzon létre dedikált kiszolgálókat. Ehelyett telepítse az AD FS-t a felhőben található minden egyes Active Directory DS-kiszolgálóra. A rendelkezésre állás fenntartása érdekében győződjön meg arról, hogy legalább két Active Directory DS-kiszolgálóval rendelkezik. Egyetlen WAP-kiszolgálót hozzon létre.
- Ha felhasználóinak száma 1000 és 15 000 között van, hozzon létre két dedikált AD FS-kiszolgálót és két dedikált WAP-kiszolgálót.
- Ha felhasználóinak száma 15 000 és 60 000 között van, hozzon létre 3–5 dedikált AD FS-kiszolgálót és legalább két dedikált WAP-kiszolgálót.

Ezeket a szempontok azt feltételezik, hogy kettős négymagos virtuálisgép-méretet (szabványos D4_v2 vagy jobb) használ az Azure-ban.

Ha a belső Windows-adatbázist használja az AD FS konfigurációs adatainak tárolásához, legfeljebb nyolc AD FS-kiszolgálót használhat a farmban. Ha várhatóan többre lesz szüksége a jövőben, az SQL Servert használja. További információkért lásd: [Az AD FS konfigurációs adatbázisának szerepe][adfs-configuration-database].

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

Hozzon létre egy AD FS-farmot legalább két kiszolgálóval a szolgáltatás rendelkezésre állásának növeléséhez. Használjon különböző tárfiókokat a farm AD FS virtuális gépeihez. Ez a megközelítés segít biztosítani, hogy egyetlen tárfiók hibája miatt ne váljon elérhetetlenné a teljes farm.

Hozzon létre külön Azure rendelkezésre állási csoportokat az AD FS és a WAP virtuális gépekhez. Ügyeljen arra, hogy legalább két virtuális gép legyen minden egyes csoportban. Az egyes rendelkezésre állási csoportoknak rendelkezniük kell legalább két frissítési tartománnyal és két tartalék tartománnyal.

Az AD FS és a WAP virtuális gépek terheléselosztóit a következőképpen konfigurálja:

- Használjon Azure-terheléselosztót a WAP virtuális gépekhez való külső hozzáférés biztosítására, és egy belső terheléselosztót a farm AD FS-kiszolgálóira irányuló terhelés elosztására.
- Az AD FS- és WAP-kiszolgálók felé kizárólag a 443-as porton (HTTPS) megjelenő forgalmat továbbítsa.
- A terheléselosztónak statikus IP-címet adjon.
- Hozzon létre egy állapotmintát elleni HTTP használatával `/adfs/probe`. További információkért lásd: [hardver Load Balancer állapot-ellenőrzések és a webalkalmazás-Proxy / AD FS 2012 R2](https://blogs.technet.microsoft.com/applicationproxyblog/2014/10/17/hardware-load-balancer-health-checks-and-web-application-proxy-ad-fs-2012-r2/).

  > [!NOTE]
  > Az AD FS-kiszolgálók a Kiszolgálónév jelzése (SNI) protokollt használják, így ha egy HTTPS-végpont használatával próbál mintát venni, a terheléselosztó hibába ütközik.
  >

- Adja hozzá egy DNS *A*-rekordot az AD FS-terheléselosztó tartományához. Adja meg a terheléselosztó IP-címét, és adjon neki nevet a tartományban (például adfs.contoso.com). Ezt a nevet használják az ügyfelek és a WAP-kiszolgálók az AD FS-kiszolgálófarm eléréséhez.

Az AD FS konfigurációs adatainak tárolására használhatja az SQL Servert vagy a belső Windows-adatbázist. A belső Windows-adatbázis alapvető redundanciát biztosít. A rendszer kizárólag az AD FS-fürt egyik AD FS-adatbázisába írja a módosításokat. A többi kiszolgáló leküldéses replikációval tartja naprakészen az adatbázisokat. Az SQL Server használata teljes adatbázis-redundanciát és magas rendelkezésre állást biztosíthat feladatátvételi fürtszolgáltatással vagy tükrözéssel.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

A fejlesztési és üzemeltetési csapatnak a következő feladatok elvégzésére kell felkészülnie:

- Az összevonási kiszolgálók kezelése, beleértve az AD FS-farm, az összevonási kiszolgálókhoz tartozó megbízhatósági házirendek és az összevonási szolgáltatások által használt tanúsítványok kezelését.
- A WAP-kiszolgálók kezelése, beleértve a WAP-farmok és a tanúsítványok kezelését.
- A webalkalmazások kezelése, beleértve a függő entitások, a hitelesítési módszerek és a jogcímleképezések konfigurálását.
- Az AD FS-összetevők biztonsági mentése.

## <a name="security-considerations"></a>Biztonsági szempontok

Az AD FS HTTPS-t használ, ezért ügyeljen arra, hogy az NSG-szabályok az alhálózat, amely tartalmazza a webes szint virtuális gépek engedély HTTPS-kéréseket. Ezek a kérelmek származhatnak a helyszíni hálózatról, a webes, az üzleti vagy az adatréteget tartalmazó alhálózatokról, a privát vagy a nyilvános DMZ-ről vagy az AD FS-kiszolgálókat tartalmazó alhálózatról.

Akadályozza meg az AD FS-kiszolgálók közvetlen elérését az internetről. Az AD FS-kiszolgálók tartományhoz kapcsolt számítógépek, amelyek teljes körű engedéllyel rendelkeznek a biztonsági jogkivonatok kiállításához. Ha egy kiszolgáló sérül, a rosszindulatú felhasználók teljes körű hozzáférési jogkivonatokat bocsáthatnak ki az AD FS által védett összes webalkalmazáshoz és összevont kiszolgálóhoz. Ha a rendszernek olyan kérelmeket is kezelnie kell, amelyek külső, nem megbízható partneroldalakról kapcsolódó felhasználóktól származnak, ezekhez a kérelmekhez használjon WAP-kiszolgálókat. További információkért lásd: [Összevonási kiszolgálóproxy elhelyezése][where-to-place-an-fs-proxy].

Az AD FS- és a WAP-kiszolgálókat különálló alhálózatokban helyezze el, saját tűzfallal. A tűzfalszabályok meghatározásához használhat NSG-szabályokat. Az összes tűzfalnak engedélyezni kell az adatforgalmat a 443-as (HTTPS) porton.

Korlátozza a közvetlen bejelentkezést az AD FS- és a WAP-kiszolgálókra. Csak a fejlesztési és üzemeltetési csapat számára legyen lehetséges a kapcsolódás. A WAP-kiszolgálókat ne csatlakoztassa a tartományhoz.

Érdemes lehet virtuális hálózati készülékeket használni, amelyek naplózzák a részletes információkat a virtuális hálózat peremén átmenő forgalomról.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

Ennek az architektúrának egy üzemelő példánya elérhető a [GitHubon][github]. Vegye figyelembe, hogy a teljes üzembe helyezés eltarthat akár két órát, amely tartalmazza a VPN gateway létrehozása, és futtatja a parancsfájlokat, amelyek az Active Directory és az AD FS konfigurálása.

### <a name="prerequisites"></a>Előfeltételek

1. Klónozza, ágaztassa vagy töltse le a zip-fájlját a [GitHub-adattár](https://github.com/mspnp/identity-reference-architectures).

1. Telepítse az [Azure CLI 2.0-t.](/cli/azure/install-azure-cli?view=azure-cli-latest)

1. Telepítse [az Azure építőelemei](https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks) npm-csomagot.

   ```bash
   npm install -g @mspnp/azure-building-blocks
   ```

1. Jelentkezzen be Azure-fiókjába egy parancssorból, Bash-parancssorból vagy PowerShell-parancssorból a következő módon:

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a>A szimulált helyszíni adatközpont üzembe helyezése

1. Keresse meg a `adfs` mappájában, a GitHub-adattárban.

1. Nyissa meg az `onprem.json` fájlt. Keresse meg a példányok `adminPassword`, `Password`, és `SafeModeAdminPassword` frissíteniük a jelszavakat, és.

1. Futtassa a következő parancsot, és várjon, amíg a telepítés befejezéséhez:

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onprem.json --deploy
    ```

### <a name="deploy-the-azure-infrastructure"></a>Az Azure-infrastruktúra üzembe helyezése

1. Nyissa meg az `azure.json` fájlt.  Keresse meg példányait `adminPassword` és `Password` és adja meg a jelszavak adatait.

1. Futtassa a következő parancsot, és várjon, amíg a telepítés befejezéséhez:

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p azure.json --deploy
    ```

### <a name="set-up-the-ad-fs-farm"></a>Állítsa be az AD FS-farm

1. Nyissa meg az `adfs-farm-first.json` fájlt.  Keresse meg `AdminPassword` , és cserélje le az alapértelmezett jelszót.

1. Futtassa az alábbi parancsot:

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p adfs-farm-first.json --deploy
    ```

1. Nyissa meg az `adfs-farm-rest.json` fájlt.  Keresse meg `AdminPassword` , és cserélje le az alapértelmezett jelszót.

1. Futtassa a következő parancsot, és várjon, amíg a telepítés befejezéséhez:

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p adfs-farm-rest.json --deploy
    ```

### <a name="configure-ad-fs-part-1"></a>Konfigurálhatja az AD FS (1. rész)

1. Nevű virtuális Gépet egy távoli asztal munkamenetet nyit meg `ra-adfs-jb-vm1`, azaz a jumpboxot virtuális Gépet. Felhasználónév: `testuser`.

1. A jumpbox, nyissa meg a nevű virtuális Gépet egy távoli asztali munkamenetet `ra-adfs-proxy-vm1`. A magánhálózati IP-cím 10.0.6.4.

1. A távoli asztali munkamenet, futtassa a [PowerShell ISE-ben](/powershell/scripting/components/ise/windows-powershell-integrated-scripting-environment--ise-).

1. A PowerShell-lel keresse meg a következő könyvtárban:

    ```powershell
    C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0
    ```

1. A parancsfájl panelen illessze be a következő kódot, és futtassa:

    ```powershell
    . .\adfs-webproxy.ps1
    $cd = @{
        AllNodes = @(
            @{
                NodeName = 'localhost'
                PSDscAllowPlainTextPassword = $true
                PSDscAllowDomainUser = $true
            }
        )
    }

    $c1 = Get-Credential -UserName testuser -Message "Enter password"
    InstallWebProxyApp -DomainName contoso.com -FederationName adfs.contoso.com -WebApplicationProxyName "Contoso App" -AdminCreds $c1 -ConfigurationData $cd
    Start-DscConfiguration .\InstallWebProxyApp
    ```

    Jelenleg a `Get-Credential` kéri, adja meg a jelszót, a központi telepítési alkalmazásparaméter-fájlt adott meg.

1. Futtassa a következő parancsot a állapotának figyelése a [DSC](/powershell/dsc/overview/overview) konfiguráció:

    ```powershell
    Get-DscConfigurationStatus
    ```

    Konzisztencia elérni több percet is igénybe vehet. Ebben az időszakban előfordulhat, hogy hibába ütközik, a parancsban. Ha a konfiguráció sikeres, a kimenet az alábbihoz hasonlóan kell kinéznie:

    ```powershell
    PS C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0> Get-DscConfigurationStatus

    Status     StartDate                 Type            Mode  RebootRequested      NumberOfResources
    ------     ---------                 ----            ----  ---------------      -----------------
    Success    12/17/2018 8:21:09 PM     Consistency     PUSH  True                 4
    ```

### <a name="configure-ad-fs-part-2"></a>Konfigurálhatja az AD FS (2. rész)

1. A jumpbox, nyissa meg a nevű virtuális Gépet egy távoli asztali munkamenetet `ra-adfs-proxy-vm2`. A magánhálózati IP-cím 10.0.6.5.

1. A távoli asztali munkamenet, futtassa a [PowerShell ISE-ben](/powershell/scripting/components/ise/windows-powershell-integrated-scripting-environment--ise-).

1. Keresse meg a következő könyvtárban:

    ```powershell
    C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0
    ```

1. A parancsfájl panelen és futtassa a parancsfájlt a következő elmúlt:

    ```powershell
    . .\adfs-webproxy-rest.ps1
    $cd = @{
        AllNodes = @(
            @{
                NodeName = 'localhost'
                PSDscAllowPlainTextPassword = $true
                PSDscAllowDomainUser = $true
            }
        )
    }

    $c1 = Get-Credential -UserName testuser -Message "Enter password"
    InstallWebProxy -DomainName contoso.com -FederationName adfs.contoso.com -WebApplicationProxyName "Contoso App" -AdminCreds $c1 -ConfigurationData $cd
    Start-DscConfiguration .\InstallWebProxy
    ```

    Jelenleg a `Get-Credential` kéri, adja meg a jelszót, a központi telepítési alkalmazásparaméter-fájlt adott meg.

1. A következő parancsot a DSC-konfiguráció állapotának figyelése:

    ```powershell
    Get-DscConfigurationStatus
    ```

    Konzisztencia elérni több percet is igénybe vehet. Ebben az időszakban előfordulhat, hogy hibába ütközik, a parancsban. Ha a konfiguráció sikeres, a kimenet az alábbihoz hasonlóan kell kinéznie:

    ```powershell
    PS C:\Packages\Plugins\Microsoft.Powershell.DSC\2.77.0.0\DSCWork\adfs-v2.0> Get-DscConfigurationStatus

    Status     StartDate                 Type            Mode  RebootRequested      NumberOfResources
    ------     ---------                 ----            ----  ---------------      -----------------
    Success    12/17/2018 8:21:09 PM     Consistency     PUSH  True                 4
    ```

    Egyes esetekben ez DSC sikertelen lesz. Ha az állapot ellenőrzése látható `Status=Failure` és `Type=Consistency`, próbálja meg újra futtatni a 4. lépés.

### <a name="sign-into-ad-fs"></a>Jelentkezzen be az Active Directory összevonási szolgáltatások

1. A jumpbox, nyissa meg a nevű virtuális Gépet egy távoli asztali munkamenetet `ra-adfs-adfs-vm1`. A magánhálózati IP-cím 10.0.5.4.

1. Kövesse a [az oldal Idp-Intiated bejelentkezés engedélyezése](/windows-server/identity/ad-fs/troubleshooting/ad-fs-tshoot-initiatedsignon#enable-the-idp-intiated-sign-on-page) ahhoz, hogy a bejelentkezési lapot.

1. A jump boxon keresse meg a `https://adfs.contoso.com/adfs/ls/idpinitiatedsignon.htm`. Egy tanúsítványt, hogy figyelmen kívül hagyhatja ezt a vizsgálatot a figyelmeztetés jelenhet meg.

1. Győződjön meg arról, hogy a Contoso Corporation bejelentkezési oldala jelenik meg. Jelentkezzen be, **contoso\testuser**.

<!-- links -->
[extending-ad-to-azure]: adds-extend-domain.md

[vm-recommendations]: ../virtual-machines-windows/single-vm.md
[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md
[hybrid-azure-on-prem-vpn]: ../hybrid-networking/vpn.md

[azure-cli]: /azure/azure-resource-manager/xplat-cli-azure-resource-manager
[DRS]: https://technet.microsoft.com/library/dn280945.aspx
[where-to-place-an-fs-proxy]: https://technet.microsoft.com/library/dd807048.aspx
[ADDRS]: https://technet.microsoft.com/library/dn486831.aspx
[plan-your-adfs-deployment]: https://msdn.microsoft.com/library/azure/dn151324.aspx
[ad_network_recommendations]: #network_configuration_recommendations_for_AD_DS_VMs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[create_service_account_for_adfs_farm]: https://technet.microsoft.com/library/dd807078.aspx
[adfs-configuration-database]: https://technet.microsoft.com/library/ee913581(v=ws.11).aspx
[active-directory-federation-services]: /windows-server/identity/active-directory-federation-services
[security-considerations]: #security-considerations
[recommendations]: #recommendations
[active-directory-federation-services-overview]: https://technet.microsoft.com/library/hh831502(v=ws.11).aspx
[establishing-federation-trust]: https://blogs.msdn.microsoft.com/alextch/2011/06/27/establishing-federation-trust/
[Deploying_a_federation_server_farm]:  /windows-server/identity/ad-fs/deployment/deploying-a-federation-server-farm
[install_and_configure_the_web_application_proxy_server]: https://technet.microsoft.com/library/dn383662.aspx
[publish_applications_using_AD_FS_preauthentication]: https://technet.microsoft.com/library/dn383640.aspx
[managing-adfs-components]: https://technet.microsoft.com/library/cc759026.aspx
[oms-adfs-pack]: https://www.microsoft.com/download/details.aspx?id=41184
[azure-powershell-download]: /powershell/azure/overview
[aad]: /azure/active-directory/
[aadb2c]: /azure/active-directory-b2c/
[adfs-intro]: /azure/active-directory/hybrid/whatis-hybrid-identity
[github]: https://github.com/mspnp/identity-reference-architectures/tree/master/adfs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[considerations]: ./considerations.md
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
