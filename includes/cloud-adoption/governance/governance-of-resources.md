<!-- TEMPLATE FILE - DO NOT ADD METADATA -->
<!-- markdownlint-disable MD002 MD041 -->

### <a name="governance-of-resources"></a>Az erőforrások cégirányítási

Cégirányítási kényszerítése az előfizetések közötti Azure tervezetek és a kapcsolódó eszközök belül a tervezet származnak.

1. Hozzon létre egy nevű tervezet `governance-baseline`.
    1. Standard szintű Azure-szerepkörök használatának kényszerítése.
    2. Kényszerítéséhez, hogy a felhasználók csak hitelesítésre meglévő RBAC meghatározásában.
    3. Ez a megoldás összes előfizetést a felügyeleti csoporton belül érvényesek.
2. Hozzon létre egy Azure szabályzat alkalmazásához vagy érvényesíteni a következő:
    1. Erőforrás-címkézési kell értékekre van szükség, üzleti funkciót, az Adatbesorolás, kritikusság, SLA-t, környezet és alkalmazás.
    2. Az alkalmazás címke értékének egyeznie kell az erőforráscsoport nevét.
    3. Ellenőrizze a szerepkör-hozzárendelések minden erőforráscsoport és erőforrások.
3. Közzététel és a alkalmazni a `governance-baseline` tervezet egyes felügyeleti csoporthoz.

Ezek a minták lehetővé teszik az erőforrások felderítése és nyomon követi, és alapszintű szerepkörkezelés kényszerítése.

### <a name="demilitarized-zone-dmz"></a>Demilitarizált zónában (DMZ)

Szokás bizonyos szintű hozzáférést a helyszíni erőforrásokhoz az adott előfizetés esetében. Ez lehet a áttelepítési forgatókönyvek vagy fejlesztési forgatókönyvek, kis, ha az egyes függő erőforrások továbbra is a helyszíni adatközpont vannak. Ebben az esetben a cégirányítási MVP ad hozzá az alábbi gyakorlati tanácsokat:

1. Szegélyhálózat (DMZ) felhő létesíteni.
    1. A [felhőalapú DMZ referenciaarchitektúráit](/azure/architecture/reference-architectures/dmz/secure-vnet-hybrid) hoz létre egy minta és az üzembe helyezési modellt VPN-átjáró létrehozása az Azure-ban.
    2. Ellenőrizze, hogy DMZ-t megfelelő kapcsolati és biztonsági követelmények vannak érvényben a egy helyi peremhálózati eszköz, a helyi adatközpontban.
    3. Ellenőrizze, hogy a helyi peremhálózati eszköz kompatibilis az Azure VPN-átjáróra vonatkozó követelmények.
    <!-- 4. Once connection to the on-premisess VPN has been verified, capture the Resource Manager template created by that reference architecture. -->
2. Hozzon létre egy második tervezet nevű `dmz`.
    1. A Resource Manager-sablon hozzáadása a VPN Gateway a tervezethez.
3. A szegélyhálózat (DMZ) tervezet alkalmazása bármilyen helyszíni internetkapcsolatának előfizetésekhez. Ez a megoldás a cégirányítási MVP tervezet mellett kell alkalmazni.

Az egyik legnagyobb aggodalmát IT-biztonsági és cégirányítási hagyományos csapatok által kiváltott, a korai szakaszában felhőre való áttérés meglévő eszközeivel veszélyeztetése kockázatát. A fenti megközelítés lehetővé teszi, hogy a felhő bevezetésének csapatok hozhat létre, és hibrid megoldásokat, kisebb kockázat a helyszíni tartalmakhoz való áttelepítéséhez. Az újabb fejlődést szem előtt tartva a ideiglenes megoldást akkor távolítható el.

> [!NOTE]
> A fenti hozhat létre gyorsan egy alapkonfiguráció cégirányítási MVP kiindulópontként szolgál. Ez a cégirányítási utazás csak a kezdet. További fejlődést szem előtt tartva szükség lesz a vállalati továbbra is a felhőre, és további kockázati a következő területeken történik:
>
> - Alapvető fontosságú számítási feladatokhoz
> - Védett adatok
> - Költségkezelés
> - Több felhő forgatókönyvek
>
>Ezen felül az MVP részleteit, egy fiktív vállalat, az az alábbi cikkekben ismertetett példa utazás alapulnak. Erősen ajánlott Ismerkedés a sorozat további cikkek az ajánlott eljárás végrehajtása előtt.
