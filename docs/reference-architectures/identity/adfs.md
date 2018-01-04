---
title: "Az Azure Active Directory összevonási szolgáltatások (AD FS) megvalósítása"
description: "Hogyan megvalósításához egy biztonságos hibrid hálózati architektúra az Active Directory összevonási szolgáltatás engedélyezése az Azure-ban.\nútmutatást, vpn-átjáró, expressroute, terheléselosztó, virtuális hálózat, active Directoryval"
author: telmosampaio
ms.date: 11/28/2016
pnp.series.title: Identity management
pnp.series.prev: adds-forest
cardTitle: Extend AD FS to Azure
ms.openlocfilehash: b8c9ae0621c087c68d449dd13e60046104c01513
ms.sourcegitcommit: 8ab30776e0c4cdc16ca0dcc881960e3108ad3e94
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 12/08/2017
---
# <a name="extend-active-directory-federation-services-ad-fs-to-azure"></a>Az Azure Active Directory összevonási szolgáltatások (AD FS) kiterjesztése

A referencia-architektúrában valósítja meg, amely kiterjeszti a helyszíni hálózat Azure és a biztonságos hibrid hálózat [Active Directory összevonási szolgáltatások (AD FS)] [ active-directory-federation-services] végrehajtásához összevont hitelesítési és engedélyezési Azure-beli összetevőnél. [**A megoldás üzembe helyezése**.](#deploy-the-solution)

[![0]][0]

*Töltse le az architektúra [Visio-fájlját][visio-download].*

Az AD FS lehet üzemeltethető a helyszínen, de ha az alkalmazás egy hibrid Azure végrehajtott bizonyos részei, hatékonyabb, ha az AD FS-ben a felhő replikálni lehet. 

Az ábrán látható a következő esetekben:

* A fiókpartner-szervezet alkalmazáskód fér hozzá egy webalkalmazást az Azure-beli virtuális hálózatán belül található.
* Belül az Active Directory tartományi szolgáltatások (DS) tárolt hitelesítő adatokkal rendelkező egy külső, regisztrált felhasználó hozzáfér egy webalkalmazást az Azure-beli virtuális hálózatán belül található.
* A virtuális hálózat egy jogosult eszközzel csatlakozó felhasználó végrehajtja a egy webalkalmazást az Azure-beli virtuális hálózatán belül található.

Ez az architektúra a jellemző használati többek között:

* Hibrid alkalmazások ahol munkaterhelések Futtatás részben helyszínen és részben az Azure.
* Összevont engedélyezési teszi közzé a webes alkalmazásokat az erőforráspartner-szervezetek használó megoldások.
* A rendszer támogatja a hozzáférést a szervezeti tűzfalon kívül futó webböngészőknek.
* Azok a rendszerek, amelyek lehetővé teszik a felhasználók férhetnek hozzá a webes alkalmazások által hitelesített külső eszközöket, például a távoli számítógépeken, jegyzetfüzeteket és egyéb mobileszközökön való kapcsolódás. 

A referencia-architektúrában összpontosít *passzív összevonási*, a következő helyen, amely az összevonási kiszolgálók mellett dönt, hogy hogyan és mikor a felhasználó hitelesítéséhez. A felhasználó megadja a bejelentkezési adatok, az alkalmazás indításakor. A mechanizmus böngészők által leggyakrabban használt, és magában foglalja a protokoll, amely a böngésző átirányítja egy helyet, ahol a felhasználó hitelesíti magát. Az AD FS is támogatja *aktív összevonási*, ha egy alkalmazás időt vesz igénybe, hogy megadja a hitelesítő adatokat, további felhasználói beavatkozás nélkül felelős az, de a forgatókönyv ebbe az architektúrába hatókörén kívül esik.

További szempontokat lásd: [megoldás választása a integrálása a helyszíni Active Directoryról szinkronizálva az Azure][considerations]. 

## <a name="architecture"></a>Architektúra

Ez az architektúra kiterjeszti a leírt végrehajtása [Extending Active Directory tartományi szolgáltatások, Azure][extending-ad-to-azure]. Az alábbi összetevőket tartalmaz.

* **Az Active Directory tartományi szolgáltatások alhálózati**. Az Active Directory tartományi szolgáltatások-kiszolgálók a saját alhálózati tűzfal működött hálózati biztonsági csoport (NSG) szabályait tartalmazza.

* **AD DS-kiszolgálók**. Az Azure-ban virtuális gépeket futtató tartományvezérlők. Ezek a kiszolgálók a tartományon belüli helyi identitások hitelesítést nyújt.

* **AD FS alhálózati**. Az AD FS-kiszolgálók találhatók a saját alhálózaton belül az NSG-szabályok tűzfal működött.

* **AD FS-kiszolgálók**. Az AD FS-kiszolgáló hitelesítési és engedélyezési összevont adja meg. Ebben az architektúrában akkor a következő feladatokat:
  
  * A fogadó biztonsági jogkivonatot tartalmazó jogcímeket egy összevonási partnerkiszolgáló tett egy partner felhasználó nevében. Az AD FS ellenőrzi, hogy a jogkivonatok érvényes előtt a webalkalmazás Azure-ban futó kérések engedélyezésére a jogcímeket. 
  
    Azure-ban futó webes alkalmazás a *függő entitás*. A partner összevonási kiszolgáló kell jogcímeket kiadni, amelyek a webalkalmazás értendők. A partner összevonási kiszolgálók nevezzük *fiókpartnerek*, mert elküldenék hozzáférési kérelmek hitelesített fiókok a fiókpartner-szervezet nevében. Az AD FS-kiszolgáló nevezzük *erőforrás partnerek* mert (webalkalmazás) erőforrásokhoz való hozzáférést biztosítanak.

  * Hitelesítése és engedélyezése a bejövő kéréseket a külső felhasználók egy webböngésző vagy az Active Directory tartományi szolgáltatások használatával webes alkalmazásokhoz, a hozzáférést igénylő eszközre és a [Active Directory Eszközregisztrációs szolgáltatás] [ ADDRS].
    
  Az AD FS-kiszolgáló van konfigurálva az Azure terheléselosztó keresztül érhetők el a farmhoz. Ez a megvalósítás javítja rendelkezésre állását és méretezhetőségét. Az AD FS-kiszolgálók nem érhetők el közvetlenül. Minden internetes forgalomhoz a az AD FS webalkalmazás-proxy kiszolgálók és a Szegélyhálózaton (más néven a szegélyhálózaton) keresztül szűrve van.

  Active Directory összevonási szolgáltatások működésével kapcsolatos további információkért lásd: [Active Directory összevonási szolgáltatások – áttekintés][active-directory-federation-services-overview]. Emellett a cikk [AD FS üzembe helyezése az Azure-ban] [ adfs-intro] tartalmaz egy megvalósítási részletes részletes bemutatása.

* **AD FS proxy alhálózati**. Az AD FS-proxy kiszolgáló védő NSG-szabályok a saját alhálózaton belül is tartalmazza. A kiszolgálók az alhálózaton az interneten keresztül az Azure virtuális hálózat és az Internet között tűzfal biztosító hálózati virtuális készülékek érhetők el.

* **Az AD FS webalkalmazás-proxy (WAP) kiszolgálók**. A bejövő kéréseket a fiókpartner-szervezetek és a külső eszközökről AD FS-kiszolgáló működésének virtuális gépeken. A WAP-kiszolgálókkal egy szűrő, az AD FS-kiszolgáló közvetlen hozzáférés az internetről védelmi összekötőként. Csakúgy, mint az AD FS-kiszolgáló központi telepítése a WAP terheléselosztás kiszolgálófarm kiszolgálók lehetővé teszi nagyobb rendelkezésre állását és méretezhetőségét, mint központi telepítése különálló kiszolgálók gyűjteménye.
  
  > [!NOTE]
  > WAP-kiszolgálókkal telepítésével kapcsolatos részletes információkért lásd: [telepítése és a webalkalmazás-Proxy kiszolgáló konfigurálása][install_and_configure_the_web_application_proxy_server]
  > 
  > 

* **Fiókpartner szervezetében dolgozó**. A fiókpartner-szervezet hozzáférést kérő webes alkalmazást futtat egy Azure-beli webalkalmazáshoz. A a fiókpartner-szervezet összevonási kiszolgáló hitelesíti a helyi kérelmekre, és elküldi a tartalmazó Azure-beli AD FS jogcímalapú biztonsági jogkivonatokat. Az Azure AD FS érvényesíti a biztonsági jogkivonatokat, és ha érvényes továbbíthatja a webalkalmazás Azure-beli engedélyezésére azokat a jogcímeket.
  
  > [!NOTE]
  > Egy VPN-alagúton közvetlen hozzáférést biztosít az AD FS megbízható partnerek Azure átjáró használatával is konfigurálhatja. A partnerek érkező kérelmeket nem haladnak át a WAP-kiszolgálókkal.
  > 
  > 

A kijelzők a architektúra, amely nem kapcsolódik az AD FS kapcsolatos további információkért lásd a következő:
- [A biztonságos hibrid hálózati architektúra végrehajtása az Azure-ban][implementing-a-secure-hybrid-network-architecture]
- [A biztonságos hibrid hálózati architektúra Internet-hozzáféréssel rendelkező végrehajtása az Azure-ban][implementing-a-secure-hybrid-network-architecture-with-internet-access]
- [Az Azure-ban egy biztonságos hibrid hálózati architektúra az Active Directory identitások megvalósítása][extending-ad-to-azure].


## <a name="recommendations"></a>Ajánlatok

Az alábbi javaslatok a legtöbb forgatókönyvre vonatkoznak. Kövesse ezeket a javaslatokat, ha nincsenek ezeket felülíró követelményei. 

### <a name="vm-recommendations"></a>Virtuális gépekre vonatkozó javaslatok

Hozzon létre virtuális gépek a várt forgalom mennyiségét kezeléséhez elegendő erőforrással. Használja a meglévő AD FS a helyszíni kiindulási pontként futtató gépek méretét. Az erőforrás-használat figyelése. Méretezze át a virtuális gépeket, és csökkentheti, ha azok túl nagy.

Kövesse a felsorolt [a Windows virtuális gépek Azure-on futó][vm-recommendations].

### <a name="networking-recommendations"></a>Hálózatokra vonatkozó javaslatok

Állítsa be a hálózati illesztő statikus magánhálózati IP-címekkel rendelkező AD FS- és WAP kiszolgálókat üzemeltető virtuális gépek mindegyikéhez.

Ne adja meg az AD FS virtuális gépek nyilvános IP-címeket. További információkért lásd: a biztonsági szempontok című szakaszban ismertetjük.

Állítsa be az IP-cím, az elsődleges és másodlagos tartomány névkiszolgálók szolgáltatás (DNS) az egyes AD FS és a WAP VM hivatkozhasson rá az Active Directory Tartományi virtuális gépek hálózati adaptereihez. Az Active Directory Tartományi virtuális gépek DNS rendszerűnek kell lennie. Ez a lépés nem szükséges ahhoz, hogy egyes virtuális gépek a tartományhoz való csatlakozáshoz.

### <a name="ad-fs-availability"></a>AD FS rendelkezésre állása 

Az AD FS-farm létrehozása a szolgáltatás rendelkezésre állásának növelése érdekében legalább két kiszolgálóval. A farm AD FS virtuális gépek különböző tárfiókok használata. Ez a megközelítés segít biztosítani, hogy egyetlen tárfiók hibája miatt nem teszi a teljes farm nem érhető el.

> [!IMPORTANT]
> Azt javasoljuk, hogy használatát [által kezelt lemezeken](/azure/storage/storage-managed-disks-overview). Kezelt lemezeken nincs szükség a storage-fiók. Egyszerűen adja meg, méretének és típusú lemez, és azt magas rendelkezésre állású úgy van telepítve. A [architektúrák hivatkozhat](/azure/architecture/reference-architectures/) jelenleg nem telepítheti a felügyelt lemezek, de a [sablon építőelemeket](https://github.com/mspnp/template-building-blocks/wiki) központi telepítése a kezelt lemezeken 2-es verzióját frissíti.

Hozzon létre külön az Azure rendelkezésre állási készletek az AD FS és WAP virtuális gépeket. Győződjön meg arról, hogy nincsenek-e legalább két virtuális gépek minden. Egyes rendelkezésre állási csoportot rendelkeznie kell legalább két frissítési tartományok és két tartalék tartományok.

A terheléselosztó konfigurálására az AD FS virtuális gépek és WAP virtuális gépek az alábbiak szerint:

* Egy Azure terheléselosztó számára biztosít hozzáférést a WAP virtuális gépeket, és a osszák szét a farm AD FS-kiszolgáló belső terheléselosztót használja.
* Port 443-as (HTTPS) az AD FS/WAP-kiszolgálókkal való szereplő forgalom csak továbbítja.
* Adjon meg egy statikus IP-címet a terheléselosztóhoz.
* Hozzon létre egy HTTPS helyett a TCP protokoll állapotmintáihoz. A ping paranccsal ellenőrizheti, hogy egy AD FS-kiszolgáló működik-e a 443-as portot.
  
  > [!NOTE]
  > AD FS-kiszolgálók úgy próbál mintavételi használ a load balancer nem a HTTPS-végpontnak a kiszolgálónév jelzése (SNI) protokoll használatára.
  > 
  > 
* Adja hozzá a DNS *A* rekord, a tartományhoz, az AD FS terheléselosztóhoz. Adja meg a terheléselosztó IP-címét, és adjon neki egy nevet a tartomány (például adfs.contoso.com). Ez a név az ügyfelek és az AD FS kiszolgálófarm eléréséhez használja a WAP-kiszolgálókkal.

### <a name="ad-fs-security"></a>AD FS biztonsági 

Közvetlen veszélyeknek való kitettség megelőzése, az AD FS-kiszolgáló az internethez. AD FS-kiszolgálók azok a számítógépek a tartományhoz, amely a biztonsági jogkivonat biztosításához teljes körű engedéllyel rendelkeznek. Ha egy kiszolgáló sérül, a rosszindulatú felhasználók kiadhatnak teljes körű hozzáférési jogkivonatot kibocsátani minden webes alkalmazáshoz és AD FS által védett összes összevonási kiszolgálóknak. Ha a rendszer megbízható partner helyekről nem csatlakoztatható külső felhasználók által érkező kérések kell kezelni, a WAP-kiszolgálókkal segítségével ezeket a kérelmeket kezeli. További információkért lásd: [összevonási kiszolgálóproxy elhelyezése][where-to-place-an-fs-proxy].

Helyezze el az AD FS-kiszolgáló és a WAP-kiszolgálókkal rendelkező saját tűzfalak külön alhálózatokon. NSG-szabályok segítségével határozza meg a tűzfal-szabályokat. Ha átfogóbb védelmi van szüksége egy további biztonsági szegélyhálózati körül kiszolgálók egy párt alhálózatok segítségével megvalósítható, és virtuális készülékekre (NVAs), a dokumentumban leírt [biztonságos hibrid hálózat az Azure-ban Internet-hozzáféréssel rendelkező architektúra][implementing-a-secure-hybrid-network-architecture-with-internet-access]. Összes tűzfalnak engedélyezni kell az adatforgalmat a 443-as (HTTPS) porton.

Korlátozhatja a hozzáférést a közvetlen bejelentkezés az AD FS és a WAP-kiszolgálókat. Csak a DevOps személyzet kell kapcsolódnia.

A WAP-kiszolgálókkal nem csatlakozik a tartományhoz.

### <a name="ad-fs-installation"></a>Az AD FS-telepítés 

A cikk [egy összevonási kiszolgálók farmja központi telepítésének] [ Deploying_a_federation_server_farm] részletes útmutatást nyújt a telepítése és konfigurálása az AD FS. A következő feladatokat a farm első AD FS-kiszolgáló konfigurálása előtt:

1. A kiszolgáló hitelesítése nyilvánosan megbízható tanúsítványok beszerzését. A *tulajdonosnévvel* tartalmaznia kell a név használni kívánt az összevonási szolgáltatás eléréséhez. Ez lehet a DNS-név, a terheléselosztóhoz, például regisztrált *adfs.contoso.com* (ne használja, mint a helyettesítő nevek **. contoso.com*, biztonsági okokból). Egy tanúsítvány használható az összes AD FS kiszolgáló virtuális gépeken. Vásárolhat egy tanúsítványt egy megbízható hitelesítésszolgáltatótól, de ha a szervezete használja az Active Directory tanúsítványszolgáltatás létrehozhat saját. 
   
    A *tulajdonos alternatív neve* használják az eszközregisztrációs szolgáltatást (DRS) való hozzáférés engedélyezése a külső eszközökről. Ezt a következő formában kell *: enterpriseregistration.contoso.com*.
   
    További információkért lásd: [beszerzése és a Secure Sockets Layer (SSL) tanúsítvány konfigurálása az AD FS][adfs_certificates].

2. A tartományvezérlőn hozzon létre egy új legfelső szintű kulcsot a kulcsszolgáltató szolgáltatás számára. A hatékony idő az aktuális idő 10 óra (Ez a konfiguráció csökkenti a késés terjesztése és a tartomány közötti kulcsok szinkronizálása előforduló) mínusz beállítása. Ez a lépés nem a csoport az AD FS szolgáltatás futtatásához használt szolgáltatásfiók létrehozása támogatásához szükséges. A következő PowerShell-parancs ennek példáját mutatja be:
   
    ```powershell
    Add-KdsRootKey -EffectiveTime (Get-Date).AddHours(-10)
    ```

3. Minden AD FS-kiszolgáló virtuális gép felvételét a tartományba.

> [!NOTE]
> AD FS-ben az elsődleges tartományvezérlő (PDC) emulátorának rugalmas egy fő művelettel futtató tartományvezérlőt telepíti (FSMO) szerepkör, a tartomány fut és elérhető az AD FS virtuális gépek kell lennie. << RBC: van-e lehetőség legyen ez kevesebb ismétlődő? >>
> 
> 

### <a name="ad-fs-trust"></a>AD FS-megbízhatóság 

Az Active Directory összevonási szolgáltatások telepítése, és az összevonási kiszolgálók a fiókpartner-szervezetek közötti összevonási megbízhatósági kapcsolatot létesítsen. Konfigurálja a jogcímek szűrése és a leképezés szükséges. 

* Minden egyes partnerszervezethez DevOps személyzet meg kell adnia egy függőentitás-megbízhatóságot az AD FS-kiszolgálókon keresztül érhető el webes alkalmazásokhoz.
* A szervezet DevOps munkatársaival kell konfigurálnia a jogcím-szolgáltatói megbízhatóság ahhoz, hogy bízzon meg a jogcímeket, amely a fiókpartner-szervezetek, adja meg az AD FS-kiszolgáló.
* A szervezet DevOps munkatársaival is be kell állítania az AD FS felelt meg a jogcímek továbbítása a szervezet webes alkalmazásokhoz.
  
További információkért lásd: [összevonási megbízhatósági létrehozó][establishing-federation-trust].

A vállalat webes alkalmazások közzététele, és elérhetővé teszi azokat a külső partnerekkel való használatával előhitelesítési segítségével a WAP-kiszolgálókkal. További információkért lásd: [AD FS előhitelesítést használó alkalmazások közzétételének][publish_applications_using_AD_FS_preauthentication]

Az AD FS támogatja a token átalakítása és növelését. Az Azure Active Directory nem biztosítja ezt a szolgáltatást. Az AD FS a megbízhatósági kapcsolatok beállításakor a következőket teheti:

* A jogcímek átalakításához engedélyezési szabályok konfigurálása. Például egy nem Microsoft-partner szervezet olyanra, amely az adott Active Directory Tartományi adhatják meg a szervezet által használt alakból csoport biztonsági is leképezheti.
* Alakítsa át a jogcímeket egy adott formátumból egy másikra. Például leképezheti a SAML 2.0 SAML 1.1, ha az alkalmazás csak SAML 1.1 jogcímek támogatja.

### <a name="ad-fs-monitoring"></a>Az AD FS figyelése 

A [Microsoft System Center felügyeleti csomag az Active Directory összevonási szolgáltatások 2012 R2] [ oms-adfs-pack] proaktív és reaktív felügyelete, az összevonási kiszolgáló az AD FS üzembe helyezése. Ez a felügyeleti csomag figyeli:

* Az események, hogy az AD FS szolgáltatás az eseménynaplókban rögzíti.
* Az AD FS teljesítményszámlálók gyűjtése a teljesítményadatokat. 
* Általános állapotát, az AD FS rendszer és a webes alkalmazások (függő entitások), és lehetővé teszi a riasztások kritikus hibák és figyelmeztetések. 

## <a name="scalability-considerations"></a>Méretezési szempontok

A következőket kell figyelembe venni, a cikk összesített [az AD FS üzembe helyezés megtervezésében][plan-your-adfs-deployment], egyfajta kiindulópontot biztosítva az AD FS-farmok méretezési:

* Ha kevesebb mint 1000 felhasználó, dedikált kiszolgálókat nem hoz létre, de ehelyett mindegyik az Active Directory Tartományi kiszolgálón a felhőben az AD FS telepítéséhez. Győződjön meg arról, hogy rendelkezik-e legalább két Active Directory Tartományi kiszolgálók rendelkezésre álljon. Egyetlen WAP-kiszolgáló létrehozása.
* Ha 1000 és 15000 felhasználók között, hozzon létre két dedikált AD FS-kiszolgáló és két dedikált WAP-kiszolgálókkal.
* Ha 15000 és 60000 felhasználók között, hozzon létre három és öt dedikált AD FS-kiszolgáló és legalább két dedikált WAP-kiszolgáló között.

Ezeket a szempontokat azt feltételezik, kettős négymagos virtuális gép (szabványos D4_v2 vagy nagyobb frekvenciával) használt méretek az Azure-ban.

Használatakor a belső Windows-adatbázis AD FS konfigurációs adatainak tárolásához, azonban legfeljebb nyolc, a farm AD FS-kiszolgáló számára. Ha várhatóan, hogy szüksége lesz a jövőben további, használja az SQL Server. További információkért lásd: [az AD FS konfigurációs adatbázis szerepe][adfs-configuration-database].

## <a name="availability-considerations"></a>Rendelkezésre állási szempontok

SQL Server vagy a belső Windows-adatbázis segítségével az AD FS konfigurációs adatainak tárolására. A belső Windows-adatbázis alapvető redundanciát biztosít. Módosításokat írja közvetlenül csak egy AD FS adatbázis az AD FS fürtben, amíg a többi kiszolgáló lekéréses replikáció segítségével hozzájuk tartozó adatbázisok naprakészen tartása. Adja meg az SQL Server használatával is teljes adatbázis-redundancia és a Feladatátvételi fürtszolgáltatás vagy tükrözés használata magas rendelkezésre állású.

## <a name="manageability-considerations"></a>Felügyeleti szempontok

DevOps személyzet kell készíteni a következő feladatok végezhetők el:

* Az összevonási kiszolgálók, például az AD FS-farm kezelése, az összevonási kiszolgálókon a megbízhatósági házirend kezelése és az összevonási szolgáltatás által használt tanúsítványok kezelése kezelése.
* A WAP-kiszolgálókkal, beleértve a WAP-farmot, és a tanúsítványok kezelése kezelése.
* Webes alkalmazások, beleértve a függő entitások számára, a hitelesítési módszerek és a jogcímek leképezések konfigurálása kezelése.
* AD FS-összetevők biztonsági mentéséről.

## <a name="security-considerations"></a>Biztonsági szempontok

Az AD FS a HTTPS protokollt használja, ezért győződjön meg arról, hogy az NSG-szabályok az alhálózat tartalmazó webes réteg virtuális gépek engedély HTTPS-kéréseket. Ezeket a kérelmeket az alhálózatok, a webes réteg, üzleti szint, adatrétegbeli, személyes DMZ, nyilvános DMZ és az AD FS-kiszolgáló tartalmazó alhálózat tartalmazó is származnak a helyszíni hálózat.

Érdemes lehet virtuális hálózati berendezések csoportja, amely a virtuális hálózat széle a naplózási célokra áthaladó forgalom részletes információkat naplózza.

## <a name="deploy-the-solution"></a>A megoldás üzembe helyezése

A megoldás érhető el a [GitHub] [ github] központi telepítése a referencia-architektúrában. Szüksége lesz a legújabb verzióját a [Azure CLI] [ azure-cli] futtatni a Powershell-parancsfájlt, amely a megoldás telepít. A referencia-architektúrában telepítéséhez kövesse az alábbi lépéseket:

1. Töltse le, vagy klónozza a megoldás mappát [GitHub] [ github] a helyi számítógépre.

2. Nyissa meg az Azure parancssori felület, és keresse meg a helyi mappát.

3. Futtassa az alábbi parancsot:
   
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> <mode>
    ```
   
    Cserélje le `<subscription id>` az Azure-előfizetés-azonosítóval.
   
    A `<location>` paraméter esetében adjon meg egy Azure-régiót (pl. `eastus` vagy `westus`).
   
    A `<mode>` paraméter szabályozza a lépésköz legyen a központi telepítést, és a következő értékek egyike lehet:
   
   * `Onpremise`: A helyszíni szimulált környezetben telepíti. A központi telepítés tesztelése és kísérletet, ha nem rendelkezik meglévő helyszíni hálózat, vagy ha azt szeretné, hogy a referencia-architektúrában módosítása nélkül tesztelheti a meglévő konfigurációját a helyi hálózati is használható.
   * `Infrastructure`: a virtuális hálózat infrastruktúra és a jump be telepíti.
   * `CreateVpn`: egy Azure virtuális hálózati átjáró telepíti, és azt a szimulált a helyszíni hálózathoz csatlakozik.
   * `AzureADDS`: központilag telepíti az Active Directory Tartományi kiszolgálóként működnek virtuális gépek, virtuális gépeken telepíti az Active Directory, és létrehozza a tartományt az Azure-ban.
   * `AdfsVm`: az AD FS virtuális gépeket telepít, és csatlakoztatja őket a tartományhoz az Azure-ban.
   * `PublicDMZ`: az Azure nyilvános DMZ telepíti.
   * `ProxyVm`: az AD FS proxy virtuális gépeket telepít, és csatlakoztatja őket a tartományhoz az Azure-ban.
   * `Prepare`: az előző központitelepítéseinek listáját, telepíti. **Ez a lehetőség ajánlott, ha egy teljesen új központi telepítést hoz létre, és nem rendelkezik meglévő helyszíni infrastruktúra.** 
   * `Workload`: nem kötelezően telepíti a webes, üzleti és adat réteg virtuális gépek és a támogató hálózati. Nem tartalmazza a `Prepare` telepítési módban.
   * `PrivateDMZ`: nem kötelezően telepíti a titkos DMZ elé Azure-ban a `Workload` fent telepített virtuális gépek. Nem tartalmazza a `Prepare` telepítési módban.

4. Várjon, amíg az üzembe helyezés befejeződik. Ha követte a `Prepare` beállítást, a központi telepítést fogad több órát, és az üzenettel befejeződött`Preparation is completed. Please install certificate to all AD FS and proxy VMs.`

5. Az Ugrás panel újraindítása (*ra-AD FS-mgmt-vm1* a a *ra-AD FS-security-rg* csoport) engedélyezi a DNS-beállítások érvénybe léptetéséhez.

6. [Az AD FS SSL-tanúsítvány beszerzése] [ adfs_certificates] és a tanúsítvány telepítése az AD FS virtuális gépeken. Figyelje meg, hogy a kapcsolódás őket a jump mezőben keresztül. Az IP-címek *10.0.5.4* és *10.0.5.5*. Az alapértelmezett felhasználónév az *contoso\testuser* jelszóval  *AweSome@PW* .
   
   > [!NOTE]
   > A megjegyzéseket, a telepítés-ReferenceArchitecture.ps1 parancsfájl ezen a ponton részletes útmutatást egy önaláírt teszttanúsítványt és a szolgáltató használatával hozhat létre a `makecert` parancsot. Azonban elvégzi ezeket a lépéseket, mint egy **tesztelése** csak a makecert éles környezetben által létrehozott tanúsítványok használata nélkül.
   > 
   > 

7. Futtassa a következő PowerShell-parancsot az AD FS kiszolgálófarm telepítése:
   
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Adfs
    ``` 

8. Ugrás a keret, tallózással `https://adfs.contoso.com/adfs/ls/idpinitiatedsignon.htm` tesztelése az AD FS telepítése (Előfordulhat, hogy kapott egy tanúsítványt, hogy figyelmen kívül hagyhatja a teszteléshez figyelmeztetés). Győződjön meg arról, hogy megjelenik-e a Contoso Corporation bejelentkezési oldal. Jelentkezzen be a *contoso\testuser* jelszóval  *AweS0me@PW* .

9. Az SSL-tanúsítvány telepítése az AD FS proxy virtuális gépek. Az IP-címek *10.0.6.4* és *10.0.6.5*.

10. Futtassa a következő PowerShell-parancsot az első AD FS-proxy kiszolgálót:
   
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Proxy1
    ```

11. Kövesse az utasításokat a parancsfájl által megjelenített első proxykiszolgáló a telepítés tesztelésére.

12. Futtassa a következő PowerShell-parancsot a második proxy kiszolgálót:
    
    ```powershell
    .\Deploy-ReferenceArchitecture.ps1 <subscription id> <location> Proxy2
    ```

13. Kövesse az utasításokat, a parancsfájl által megjelenített, a teljes proxykonfigurációt teszteléséhez.

## <a name="next-steps"></a>További lépések

* További tudnivalók [az Azure Active Directory][aad].
* További tudnivalók [az Azure Active Directory B2C][aadb2c].

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
[active-directory-federation-services]: https://technet.microsoft.com/windowsserver/dd448613.aspx
[security-considerations]: #security-considerations
[recommendations]: #recommendations
[active-directory-federation-services-overview]: https://technet.microsoft.com/library/hh831502(v=ws.11).aspx
[establishing-federation-trust]: https://blogs.msdn.microsoft.com/alextch/2011/06/27/establishing-federation-trust/
[Deploying_a_federation_server_farm]:  https://azure.microsoft.com/documentation/articles/active-directory-aadconnect-azure-adfs/
[install_and_configure_the_web_application_proxy_server]: https://technet.microsoft.com/library/dn383662.aspx
[publish_applications_using_AD_FS_preauthentication]: https://technet.microsoft.com/library/dn383640.aspx
[managing-adfs-components]: https://technet.microsoft.com/library/cc759026.aspx
[oms-adfs-pack]: https://www.microsoft.com/download/details.aspx?id=41184
[azure-powershell-download]: https://azure.microsoft.com/documentation/articles/powershell-install-configure/
[aad]: https://azure.microsoft.com/documentation/services/active-directory/
[aadb2c]: https://azure.microsoft.com/documentation/services/active-directory-b2c/
[adfs-intro]: /azure/active-directory/active-directory-aadconnect-azure-adfs
[github]: https://github.com/mspnp/reference-architectures/tree/master/identity/adfs
[adfs_certificates]: https://technet.microsoft.com/library/dn781428(v=ws.11).aspx
[considerations]: ./considerations.md
[visio-download]: https://archcenter.azureedge.net/cdn/identity-architectures.vsdx
[0]: ./images/adfs.png "biztonságos és az Active Directory hibrid hálózati architektúra"
