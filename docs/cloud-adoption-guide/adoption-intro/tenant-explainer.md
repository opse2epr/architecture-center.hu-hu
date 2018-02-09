---
title: "Explainer: Mi az Azure Active Directory-bérlő?"
description: "Az Azure Active Directory identitást biztosíthat az Azure-ban (IDaaS) szolgáltatás belső működését ismerteti"
author: petertay
ms.openlocfilehash: ce5a33b92047e1f360eee8fcbc7a726bcf8cd19f
ms.sourcegitcommit: 2e8b06e9c07875d65b91d5431bfd4bc465a7a242
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 02/09/2018
---
# <a name="explainer-what-is-an-azure-active-directory-tenant"></a>Explainer: Mi az Azure Active Directory-bérlő?

Az a [hogyan Azure működik?](azure-explainer.md) explainer a cikkben megtanulta, hogy Azure gyűjteményei, kiszolgálók és hálózati eszközt futtató virtualizált hardver- és a felhasználók nevében. Is megtanulta, hogy az a kiszolgáló futtatása egy elosztott vezénylési alkalmazás létrehozása, olvasása, frissítése és törlése az Azure-erőforrások kezeléséhez.

Azonban módon teheti meg - Azure nem engedélyezi a bárki végrehajtani a műveleteket erőforráson. Azure ezeket a műveleteket egy megbízható digitális identitásokat nevű szolgáltatás használatával korlátozza a hozzáférést **Azure Active Directory** (az Azure AD). Az Azure AD a felhasználónév, a jelszavakat, a profil adatai és az egyéb adatokat tárolja. Az Azure Active Directory-felhasználók be vannak szegmentált **bérlők**. A bérlő egy logikai szerkezet, amely általában egy szervezet társított Azure AD egy biztonságos, dedikált példányát jelöli.

Ahhoz, hogy a bérlő létrehozása, az Azure-nak a **rendszer-jogosultságú fiók**. Ez a jogosultsági szintű fiók az Azure-fiók vagy a nagyvállalati szerződéssel társítva. Mindkét esetben van számlázási szerkezetek, és nem tárolja az Azure AD &mdash; ezeknek a fiókoknak a rendkívül biztonságos számlázási adatbázisban tárolódnak. 

A bérlő létrehozása után egy **bérlői azonosító** a bérlő jön létre, és menti a rendkívül biztonságos belső Azure AD-adatbázis. A magas jogosultsági szintű fiók tulajdonosa ezután jelentkezzen be egy Azure-portálon és felhasználók hozzáadása az újonnan létrehozott Azure AD-bérlő. 

A legtöbb vállalat már rendelkezik legalább egy identitás-kezelő szolgáltatás, általában az Active Directory tartományi szolgáltatások (AD DS). Az Azure AD képes a szinkronizálás vagy az AD DS-ből, a felhasználói identitás összevonását, így a vállalatok nem kell külön-külön a két környezetben identitás kezelése. Ez ismerteti részletesen a köztes és speciális Bevezetési fázis cikkekben digitális identitás.

## <a name="next-steps"></a>További lépések

* Most, hogy az Azure AD-bérlő megismerte a legalapvetőbb Bevezetési fázis első lépéseként-e további [Azure Active Directory-bérlő beszerzése][how-to-get-aad-tenant]. Tekintse át a [kialakítási útmutató a Azure AD-bérlő](tenant.md).

<!-- Links -->
[how-to-get-aad-tenant]: /azure/active-directory/develop/active-directory-howto-tenant?toc=/azure/architecture/cloud-adoption-guide/toc.json