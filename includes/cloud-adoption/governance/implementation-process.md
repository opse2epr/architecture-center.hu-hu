<!-- TEMPLATE FILE - DO NOT ADD METADATA -->

## <a name="dependent-decisions"></a>Függő döntéseket hozhat

A következő döntéseket hozhat a teams, a felhő Cégirányítási csapat kívül származnak. Minden egyes végrehajtását azonos csoportok származnak. Azonban a felhő Cégirányítási csapat felelős ellenőrizze, hogy ezek megvalósításokhoz következetesen alkalmazzák megoldás megvalósításáról.

### <a name="identity-baseline"></a>Identitásra építve

Identitásra építve az alapvető kiindulási pontként valamennyi cégirányítási. Mielőtt megpróbálja alkalmazni a cégirányítási, identitás kell létrehozni. A létrehozott identitás stratégia alkalmazza a cégirányítási megoldásokkal.
A cégirányítási tartson a folyamatban, az Identitáskezelés csapata is megköveteli a **címtár-szinkronizálás** minta:

- RBAC biztosítja az Azure Active Directory (Azure AD), a címtár-szinkronizálás vagy a "azonos bejelentkezés", amely az Office 365 vállalati áttelepítés során lett megvalósítva. Megvalósítás útmutatásért lásd: [Referenciaarchitektúra az Azure AD-integrációs](/azure/architecture/reference-architectures/identity/azure-ad).
- Az Azure AD-bérlő hitelesítés és hozzáférés az Azure-ban üzembe helyezett eszközök is szabályozzák.

Az MVP cégirányítási a cégirányítási csapat kényszeríti a replikált bérlő előfizetés cégirányítási azokat az eszközöket, tárgyalt keresztül alkalmazás [a cikk későbbi részében](#subscription-model). A jövőbeli fejlesztések, a cégirányítási csapat sikerült kényszerítését gazdag azokat az eszközöket az Azure ad-ben a képesség kiterjesztését.

### <a name="security-baseline-networking"></a>Biztonsági alapkonfiguráció: Hálózat

Szoftveralapú hálózati kezdeti fontos szempont a biztonsági alapkonfiguráció a rendszer. A cégirányítási létrehozó MVP attól függ, hogyan hálózatok biztonságos konfigurálhatók meghatározásához a biztonsági csapat korai döntéseket hozhat.

Adja meg a követelmények hiánya, IT-biztonsági lejátszása, biztonságos és a szükséges egy **felhőalapú DMZ** mintát. Ez azt jelenti, hogy az Azure-környezetek maguk irányítása nagyon csekély lesz.

- Az Azure-előfizetések egy meglévő data center VPN-en keresztül lehet csatlakozni, de a kell minden meglévő kövesse a helyszíni informatikai cégirányítási szabályzatok kapcsolat védett erőforrások demilitarized zóna tekintetében. VPN-kapcsolat vonatkozó megvalósítási útmutatóért lásd: [VPN Referenciaarchitektúra](/azure/architecture/reference-architectures/hybrid-networking/vpn).
- Alhálózat, a tűzfal és az Útválasztás kapcsolatos döntéseket minden alkalmazás vagy munkaterhelés érdeklődőnek a funkciófrissítéseket jelenleg folyamatban van.
- További elemzés szükséges bármely védett adatok vagy az alapvető fontosságú számítási feladatokhoz kibocsátása előtt.

Ebben a mintában felhőalapú hálózatokhoz csak csatlakozhat a helyszíni erőforrásokhoz, amely az Azure-ral kompatibilis előre lefoglalt VPN-kapcsolattal. A kapcsolaton keresztül a forgalom például egy demilitarizált zóna érkező forgalmat kezeli a rendszer. További szempontok biztonságosan kezelni az adatforgalom az Azure-ból a helyszíni peremhálózati eszközön is szükséges.

A felhő Cégirányítási csapat tagjai a hálózat és az IT-biztonsági csoportoknak a rendszeres értekezleteket, proaktív módon meghívta annak érdekében, hogy lépéselőnyben legyenek hálózati igényeket és kockázatokról.

### <a name="security-baseline-encryption"></a>Biztonsági alapkonfiguráció: Titkosítás

A titkosítás egy másik alapvető döntést az alapvető biztonsági területen belül. Mivel a vállalat jelenleg nem még tárolja védett adatokat a felhőben, a biztonsági csapat úgy döntött, a kevésbé agresszív mintát a titkosításhoz.
Ezen a ponton egy **Felhőbeli natív** titkosítási mintát javasolt, de nem szükséges a fejlesztői csapatnak sem.

- Cégirányítási követelmények nem állította titkosítás használatával kapcsolatban, mert a jelenlegi vállalati szabályzat nem teszi lehetővé a felhőbeli üzleti szempontból kritikus vagy a védett adatok.
- További elemzést védett adatokat vagy alapvető fontosságú számítási feladatokhoz kibocsátása előtt meg kell adni

## <a name="policy-enforcement"></a>Szabályzat érvénybe léptetése

Az első döntés, üzembe helyezési gyorsítás kapcsolatban a minta kényszerítésre kijelölt. Ez narratíva a cégirányítási csapat úgy döntött, hogy végrehajtja a **automatikus végrehajtás** mintát.

- Az Azure Security Center biztonsági kockázatokat figyelése a biztonság és identitás csapatok számára elérhető lesz. Mindkét csapatok is valószínűleg a Security Center használatával új kockázatok azonosítása és hogyan fejlesztheti tovább a vállalati házirend.
- Az összes előfizetés szükséges RBAC a hitelesítési kényszerítési szabályozzák.
- Az Azure Policy fog közzé minden egyes felügyeleti csoporthoz, és alkalmazza az összes előfizetés. Azonban a házirendek érvényben szintjét nagyon korlátozott lesz a kezdeti Cégirányítási MVP a.
- Bár az Azure felügyeleti csoportok vannak használatban, viszonylag egyszerű hierarchia várható.
- Az Azure tervezetek telepíthet és frissíthet előfizetések RBAC követelmények, a Resource Manager-sablonokkal és az Azure Policy alkalmazza a felügyeleti csoportok használható.

## <a name="applying-the-dependent-patterns"></a>A függő minták alkalmazása

A következő döntéseket hozhat a minták a fenti házirend végrehajtási stratégia útján lehet kikényszeríteni mutatják be:

**Identitásra építve**. Tervezetek Azure RBAC-követelmények győződjön meg arról, hogy az összes előfizetés konzisztens identitás konfigurálása az előfizetés szintjén állítja be.

**Biztonsági alapkonfiguráció: Hálózatkezelés**. A felhő Cégirányítási csapat fenntart egy Resource Manager-sablon létrehozásához szükséges VPN-átjáró, Azure és a helyszíni VPN-eszköz között. Az alkalmazás csapat egy VPN-kapcsolatot igényel, a felhő Cégirányítási csapat az átjáró Resource Manager-sablon Azure tervezetek keresztül fog vonatkoznak.

**Biztonsági alapkonfiguráció: Titkosítási**. Ezen a ponton az út nem a házirend betartatása szükséges a ezen a területen. Ez a program újabb fejlesztések során javított kell változat.
