---
title: AD DS-erőforráserdő létrehozása az Azure-ban
titleSuffix: Azure Reference Architectures
description: >-
  Megbízható Active Directory-tartomány létrehozása az Azure-ban.

  guidance,vpn-gateway,expressroute,load-balancer,virtual-network,active-directory
author: telmosampaio
ms.date: 05/02/2018
ms.topic: reference-architecture
ms.service: architecture-center
ms.subservice: reference-architecture
ms.custom: seodec18, identity
ms.openlocfilehash: bf62544028741d500e9c4172d4ac0206c95cb919
ms.sourcegitcommit: 40f3561cc94f721eca50d33f2d75dc974cb6f92b
ms.translationtype: MT
ms.contentlocale: hu-HU
ms.lasthandoff: 01/29/2019
ms.locfileid: "55147314"
---
# <a name="create-an-active-directory-domain-services-ad-ds-resource-forest-in-azure"></a><span data-ttu-id="4d226-104">Active Directory Domain Services- (AD DS-) erőforráserdő létrehozása az Azure-ban</span><span class="sxs-lookup"><span data-stu-id="4d226-104">Create an Active Directory Domain Services (AD DS) resource forest in Azure</span></span>

<span data-ttu-id="4d226-105">A referenciaarchitektúra bemutatja, hogyan hozható létre egy külön Active Directory-tartomány az Azure-ban, amelyet a helyszíni AD-erdőben található tartományok megbízhatónak tekintenek.</span><span class="sxs-lookup"><span data-stu-id="4d226-105">This reference architecture shows how to create a separate Active Directory domain in Azure that is trusted by domains in your on-premises AD forest.</span></span> <span data-ttu-id="4d226-106">[**A megoldás üzembe helyezése.**](#deploy-the-solution)</span><span class="sxs-lookup"><span data-stu-id="4d226-106">[**Deploy this solution**](#deploy-the-solution).</span></span>

![Biztonságos hibrid hálózati architektúra külön Active Directory-tartományok](./images/adds-forest.png)

<span data-ttu-id="4d226-108">*Töltse le az architektúra [Visio-fájlját][visio-download].*</span><span class="sxs-lookup"><span data-stu-id="4d226-108">*Download a [Visio file][visio-download] of this architecture.*</span></span>

<span data-ttu-id="4d226-109">Az Active Directory Domain Services (AD DS) hierarchikus struktúrában tárolja az identitásadatokat.</span><span class="sxs-lookup"><span data-stu-id="4d226-109">Active Directory Domain Services (AD DS) stores identity information in a hierarchical structure.</span></span> <span data-ttu-id="4d226-110">A hierarchikus struktúra legfelső szintű csomópontját erdőnek hívják.</span><span class="sxs-lookup"><span data-stu-id="4d226-110">The top node in the hierarchical structure is known as a forest.</span></span> <span data-ttu-id="4d226-111">Az erdő tartományokat, a tartományok pedig egyéb típusú objektumokat tartalmaznak.</span><span class="sxs-lookup"><span data-stu-id="4d226-111">A forest contains domains, and domains contain other types of objects.</span></span> <span data-ttu-id="4d226-112">Ez a referenciaarchitektúra egy AD DS-erdőt hoz létre az Azure-ban, amely egyirányú kimenő bizalmi kapcsolattal rendelkezik egy helyszíni tartománnyal.</span><span class="sxs-lookup"><span data-stu-id="4d226-112">This reference architecture creates an AD DS forest in Azure with a one-way outgoing trust relationship with an on-premises domain.</span></span> <span data-ttu-id="4d226-113">Az Azure-ban lévő erdő tartalmaz egy olyan tartományt, amely a helyszínen nem létezik.</span><span class="sxs-lookup"><span data-stu-id="4d226-113">The forest in Azure contains a domain that does not exist on-premises.</span></span> <span data-ttu-id="4d226-114">A bizalmi kapcsolat miatt a helyszíni tartományokra történő bejelentkezések megbízhatók a külön Azure-tartományban lévő erőforrások eléréséhez.</span><span class="sxs-lookup"><span data-stu-id="4d226-114">Because of the trust relationship, logons made against on-premises domains can be trusted for access to resources in the separate Azure domain.</span></span>

<span data-ttu-id="4d226-115">Az architektúra gyakori használati módjai például a felhőben tárolt objektumok és identitások biztonsági elkülönítésének fenntartása, illetve az egyes tartományok migrálása a helyszínről a felhőbe.</span><span class="sxs-lookup"><span data-stu-id="4d226-115">Typical uses for this architecture include maintaining security separation for objects and identities held in the cloud, and migrating individual domains from on-premises to the cloud.</span></span>

<span data-ttu-id="4d226-116">További szempontok: [Megoldás választása a helyszíni Active Directory Azure-ral való integrálásához][considerations].</span><span class="sxs-lookup"><span data-stu-id="4d226-116">For additional considerations, see [Choose a solution for integrating on-premises Active Directory with Azure][considerations].</span></span>

## <a name="architecture"></a><span data-ttu-id="4d226-117">Architektúra</span><span class="sxs-lookup"><span data-stu-id="4d226-117">Architecture</span></span>

<span data-ttu-id="4d226-118">Az architektúra a következő összetevőket tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="4d226-118">The architecture has the following components.</span></span>

- <span data-ttu-id="4d226-119">**Helyszíni hálózat**.</span><span class="sxs-lookup"><span data-stu-id="4d226-119">**On-premises network**.</span></span> <span data-ttu-id="4d226-120">A helyszíni hálózat a saját Active Directory-erdőjét és tartományait tartalmazza.</span><span class="sxs-lookup"><span data-stu-id="4d226-120">The on-premises network contains its own Active Directory forest and domains.</span></span>
- <span data-ttu-id="4d226-121">**Active Directory-kiszolgálók**.</span><span class="sxs-lookup"><span data-stu-id="4d226-121">**Active Directory servers**.</span></span> <span data-ttu-id="4d226-122">Ezek a felhőben virtuális gépként futó tartományvezérlők, amelyek tartományi szolgáltatásokat biztosítanak.</span><span class="sxs-lookup"><span data-stu-id="4d226-122">These are domain controllers implementing domain services running as VMs in the cloud.</span></span> <span data-ttu-id="4d226-123">Ezek a kiszolgálók egy egy vagy több tartományt tartalmazó erdőt üzemeltetnek a helyszíniektől elkülönítve.</span><span class="sxs-lookup"><span data-stu-id="4d226-123">These servers host a forest containing one or more domains, separate from those located on-premises.</span></span>
- <span data-ttu-id="4d226-124">**Egyirányú bizalmi kapcsolat**.</span><span class="sxs-lookup"><span data-stu-id="4d226-124">**One-way trust relationship**.</span></span> <span data-ttu-id="4d226-125">Az ábrán az Azure-ban lévő tartományból a helyszíni tartományba tartó egyirányú bizalmi kapcsolatra látható egy példa.</span><span class="sxs-lookup"><span data-stu-id="4d226-125">The example in the diagram shows a one-way trust from the domain in Azure to the on-premises domain.</span></span> <span data-ttu-id="4d226-126">Ez a kapcsolat lehetővé teszi a helyszíni felhasználók számára, hogy elérjék az Azure-ban lévő tartomány erőforrásait, de fordított irányban ez nem lehetséges.</span><span class="sxs-lookup"><span data-stu-id="4d226-126">This relationship enables on-premises users to access resources in the domain in Azure, but not the other way around.</span></span> <span data-ttu-id="4d226-127">Kétirányú megbízhatósági kapcsolat is létrehozható, ha a felhőalapú felhasználóknak is hozzáférést kell biztosítani a helyszíni erőforrásokhoz.</span><span class="sxs-lookup"><span data-stu-id="4d226-127">It is possible to create a two-way trust if cloud users also require access to on-premises resources.</span></span>
- <span data-ttu-id="4d226-128">**Active Directory-alhálózat**.</span><span class="sxs-lookup"><span data-stu-id="4d226-128">**Active Directory subnet**.</span></span> <span data-ttu-id="4d226-129">Az AD DS-kiszolgálók külön alhálózaton üzemelnek.</span><span class="sxs-lookup"><span data-stu-id="4d226-129">The AD DS servers are hosted in a separate subnet.</span></span> <span data-ttu-id="4d226-130">A hálózati biztonsági csoport (NSG) szabályai megvédik az AD DS-kiszolgálókat, és tűzfalat biztosítanak a nem várt forrásból érkező adatforgalom ellen.</span><span class="sxs-lookup"><span data-stu-id="4d226-130">Network security group (NSG) rules protect the AD DS servers and provide a firewall against traffic from unexpected sources.</span></span>
- <span data-ttu-id="4d226-131">**Azure-átjáró**.</span><span class="sxs-lookup"><span data-stu-id="4d226-131">**Azure gateway**.</span></span> <span data-ttu-id="4d226-132">Az Azure-átjáró kapcsolatot biztosít a helyszíni hálózat és az Azure-beli virtuális hálózat között.</span><span class="sxs-lookup"><span data-stu-id="4d226-132">The Azure gateway provides a connection between the on-premises network and the Azure VNet.</span></span> <span data-ttu-id="4d226-133">Ez lehet [VPN-kapcsolat] [ azure-vpn-gateway] vagy [Azure ExpressRoute][azure-expressroute].</span><span class="sxs-lookup"><span data-stu-id="4d226-133">This can be a [VPN connection][azure-vpn-gateway] or [Azure ExpressRoute][azure-expressroute].</span></span> <span data-ttu-id="4d226-134">További információ: [Biztonságos hibrid hálózati architektúra megvalósítása az Azure-ban][implementing-a-secure-hybrid-network-architecture].</span><span class="sxs-lookup"><span data-stu-id="4d226-134">For more information, see [Implementing a secure hybrid network architecture in Azure][implementing-a-secure-hybrid-network-architecture].</span></span>

## <a name="recommendations"></a><span data-ttu-id="4d226-135">Javaslatok</span><span class="sxs-lookup"><span data-stu-id="4d226-135">Recommendations</span></span>

<span data-ttu-id="4d226-136">Az Active Directory Azure-ban való megvalósítására vonatkozó konkrét ajánlásokért tekintse meg az alábbi cikkeket:</span><span class="sxs-lookup"><span data-stu-id="4d226-136">For specific recommendations on implementing Active Directory in Azure, see the following articles:</span></span>

- <span data-ttu-id="4d226-137">[Az Azure Active Directory Domain Services (AD DS) kiterjesztése az Azure-ra][adds-extend-domain].</span><span class="sxs-lookup"><span data-stu-id="4d226-137">[Extending Active Directory Domain Services (AD DS) to Azure][adds-extend-domain].</span></span>
- <span data-ttu-id="4d226-138">[Útmutató a Windows Server Active Directory szolgáltatás Azure-beli virtuális gépekre való telepítéséhez][ad-azure-guidelines].</span><span class="sxs-lookup"><span data-stu-id="4d226-138">[Guidelines for Deploying Windows Server Active Directory on Azure Virtual Machines][ad-azure-guidelines].</span></span>

### <a name="trust"></a><span data-ttu-id="4d226-139">Bizalmi kapcsolat</span><span class="sxs-lookup"><span data-stu-id="4d226-139">Trust</span></span>

<span data-ttu-id="4d226-140">A helyszíni tartományokat nem ugyanaz az erdő tartalmazza, mint a felhőben lévőket.</span><span class="sxs-lookup"><span data-stu-id="4d226-140">The on-premises domains are contained within a different forest from the domains in the cloud.</span></span> <span data-ttu-id="4d226-141">A helyszíni felhasználók felhőbeli hitelesítésének engedélyezéséhez az Azure-ban lévő tartományoknak meg kell bízniuk a helyszíni erdőben található bejelentkezési tartományban.</span><span class="sxs-lookup"><span data-stu-id="4d226-141">To enable authentication of on-premises users in the cloud, the domains in Azure must trust the logon domain in the on-premises forest.</span></span> <span data-ttu-id="4d226-142">Hasonlóképpen, ha a felhő bejelentkezési tartományt biztosít a külső felhasználók számára, szükség lehet arra, hogy a helyszíni erdő megbízzon a felhőtartományban.</span><span class="sxs-lookup"><span data-stu-id="4d226-142">Similarly, if the cloud provides a logon domain for external users, it may be necessary for the on-premises forest to trust the cloud domain.</span></span>

<span data-ttu-id="4d226-143">Bizalmi kapcsolatokat létesíthet erdőszinten [erdőszintű bizalmi kapcsolatok létrehozásával][creating-forest-trusts] vagy tartományszinten [külső bizalmi kapcsolatok létrehozásával][creating-external-trusts].</span><span class="sxs-lookup"><span data-stu-id="4d226-143">You can establish trusts at the forest level by [creating forest trusts][creating-forest-trusts], or at the domain level by [creating external trusts][creating-external-trusts].</span></span> <span data-ttu-id="4d226-144">Az erdőszintű bizalmi kapcsolat két erdő összes tartománya között kapcsolatot hoz létre.</span><span class="sxs-lookup"><span data-stu-id="4d226-144">A forest level trust creates a relationship between all domains in two forests.</span></span> <span data-ttu-id="4d226-145">A külső tartományszintű bizalmi kapcsolat csak két adott tartomány között hoz létre kapcsolatot.</span><span class="sxs-lookup"><span data-stu-id="4d226-145">An external domain level trust only creates a relationship between two specified domains.</span></span> <span data-ttu-id="4d226-146">Különböző erdőkben lévő tartományok között csak külső tartományszintű bizalmi kapcsolatot hozzon létre.</span><span class="sxs-lookup"><span data-stu-id="4d226-146">You should only create external domain level trusts between domains in different forests.</span></span>

<span data-ttu-id="4d226-147">A bizalmi kapcsolatok lehetnek egyirányúak vagy kétirányúak:</span><span class="sxs-lookup"><span data-stu-id="4d226-147">Trusts can be unidirectional (one-way) or bidirectional (two-way):</span></span>

- <span data-ttu-id="4d226-148">Az egyirányú bizalmi kapcsolat lehetővé teszi egy tartományban vagy erdőben (*bejövő* tartomány vagy erdő) lévő felhasználóknak, hogy hozzáférjenek a másikban (*kimenő* tartomány vagy erdő) tárolt erőforrásokhoz.</span><span class="sxs-lookup"><span data-stu-id="4d226-148">A one-way trust enables users in one domain or forest (known as the *incoming* domain or forest) to access the resources held in another (the *outgoing* domain or forest).</span></span>
- <span data-ttu-id="4d226-149">A kétirányú bizalmi kapcsolat lehetővé teszi a tartományban vagy az erdőben lévő felhasználók számára, hogy hozzáférjenek a másikban tárolt erőforrásokhoz.</span><span class="sxs-lookup"><span data-stu-id="4d226-149">A two-way trust enables users in either domain or forest to access resources held in the other.</span></span>

<span data-ttu-id="4d226-150">A következő táblázat összefoglalja a bizalmi kapcsolati konfigurációkat néhány egyszerű forgatókönyvhöz:</span><span class="sxs-lookup"><span data-stu-id="4d226-150">The following table summarizes trust configurations for some simple scenarios:</span></span>

| <span data-ttu-id="4d226-151">Forgatókönyv</span><span class="sxs-lookup"><span data-stu-id="4d226-151">Scenario</span></span> | <span data-ttu-id="4d226-152">Helyszíni bizalmi kapcsolat</span><span class="sxs-lookup"><span data-stu-id="4d226-152">On-premises trust</span></span> | <span data-ttu-id="4d226-153">Felhőbeli bizalmi kapcsolat</span><span class="sxs-lookup"><span data-stu-id="4d226-153">Cloud trust</span></span> |
| --- | --- | --- |
| <span data-ttu-id="4d226-154">A helyszíni felhasználóknak hozzá kell férniük a felhőben lévő erőforrásokhoz, de fordított irányban ez nem szükséges</span><span class="sxs-lookup"><span data-stu-id="4d226-154">On-premises users require access to resources in the cloud, but not vice versa</span></span> |<span data-ttu-id="4d226-155">Egyirányú, bejövő</span><span class="sxs-lookup"><span data-stu-id="4d226-155">One-way, incoming</span></span> |<span data-ttu-id="4d226-156">Egyirányú, kimenő</span><span class="sxs-lookup"><span data-stu-id="4d226-156">One-way, outgoing</span></span> |
| <span data-ttu-id="4d226-157">A felhőbeli felhasználóknak hozzá kell férniük a helyszíni erőforrásokhoz, de fordított irányban ez nem szükséges</span><span class="sxs-lookup"><span data-stu-id="4d226-157">Users in the cloud require access to resources located on-premises, but not vice versa</span></span> |<span data-ttu-id="4d226-158">Egyirányú, kimenő</span><span class="sxs-lookup"><span data-stu-id="4d226-158">One-way, outgoing</span></span> |<span data-ttu-id="4d226-159">Egyirányú, bejövő</span><span class="sxs-lookup"><span data-stu-id="4d226-159">One-way, incoming</span></span> |
| <span data-ttu-id="4d226-160">A felhőben lévő és a helyszíni felhasználóknak is hozzá kell férniük a felhőben és a helyszínen tárolt erőforrásokhoz is</span><span class="sxs-lookup"><span data-stu-id="4d226-160">Users in the cloud and on-premises both requires access to resources held in the cloud and on-premises</span></span> |<span data-ttu-id="4d226-161">Kétirányú, bejövő és kimenő</span><span class="sxs-lookup"><span data-stu-id="4d226-161">Two-way, incoming and outgoing</span></span> |<span data-ttu-id="4d226-162">Kétirányú, bejövő és kimenő</span><span class="sxs-lookup"><span data-stu-id="4d226-162">Two-way, incoming and outgoing</span></span> |

## <a name="scalability-considerations"></a><span data-ttu-id="4d226-163">Méretezési szempontok</span><span class="sxs-lookup"><span data-stu-id="4d226-163">Scalability considerations</span></span>

<span data-ttu-id="4d226-164">Az Active Directory automatikusan skálázható az ugyanazon tartományba tartozó tartományvezérlők esetén.</span><span class="sxs-lookup"><span data-stu-id="4d226-164">Active Directory is automatically scalable for domain controllers that are part of the same domain.</span></span> <span data-ttu-id="4d226-165">A kérelmek a tartományok összes vezérlője között oszlanak meg.</span><span class="sxs-lookup"><span data-stu-id="4d226-165">Requests are distributed across all controllers within a domain.</span></span> <span data-ttu-id="4d226-166">Ha hozzáad egy másik tartományvezérlőt, ez automatikusan szinkronizálódik a tartománnyal.</span><span class="sxs-lookup"><span data-stu-id="4d226-166">You can add another domain controller, and it synchronizes automatically with the domain.</span></span> <span data-ttu-id="4d226-167">Ne konfiguráljon külön terheléselosztót arra, hogy a tartományban lévő vezérlőkre irányítsa a forgalmat.</span><span class="sxs-lookup"><span data-stu-id="4d226-167">Do not configure a separate load balancer to direct traffic to controllers within the domain.</span></span> <span data-ttu-id="4d226-168">Győződjön meg arról, hogy minden tartományvezérlő elegendő memória-és tároló-erőforrással rendelkezik a tartomány-adatbázis kezeléséhez.</span><span class="sxs-lookup"><span data-stu-id="4d226-168">Ensure that all domain controllers have sufficient memory and storage resources to handle the domain database.</span></span> <span data-ttu-id="4d226-169">Minden tartományvezérlő virtuális gépet ugyanakkora méretűre állítson be.</span><span class="sxs-lookup"><span data-stu-id="4d226-169">Make all domain controller VMs the same size.</span></span>

## <a name="availability-considerations"></a><span data-ttu-id="4d226-170">Rendelkezésre állási szempontok</span><span class="sxs-lookup"><span data-stu-id="4d226-170">Availability considerations</span></span>

<span data-ttu-id="4d226-171">Építsen ki legalább két tartományvezérlőt minden tartományhoz.</span><span class="sxs-lookup"><span data-stu-id="4d226-171">Provision at least two domain controllers for each domain.</span></span> <span data-ttu-id="4d226-172">Ez lehetővé teszi a kiszolgálók közötti automatikus replikációt.</span><span class="sxs-lookup"><span data-stu-id="4d226-172">This enables automatic replication between servers.</span></span> <span data-ttu-id="4d226-173">Hozzon létre egy rendelkezésre állási csoportot az egyes tartományokat kezelő, Active Directory-kiszolgálóként működő virtuális gépekhez.</span><span class="sxs-lookup"><span data-stu-id="4d226-173">Create an availability set for the VMs acting as Active Directory servers handling each domain.</span></span> <span data-ttu-id="4d226-174">Helyezzen el legalább két kiszolgálót ebben a rendelkezésre állási csoportban.</span><span class="sxs-lookup"><span data-stu-id="4d226-174">Put at least two servers in this availability set.</span></span>

<span data-ttu-id="4d226-175">Vegye fontolóra, hogy minden tartományban kijelöljön egy vagy több kiszolgálót [készenléti műveleti főkiszolgálónak][standby-operations-masters], arra az esetre, ha nem sikerülne csatlakozni a mozgó egyedüli főkiszolgálói (FSMO) szerepkört betöltő kiszolgálóhoz.</span><span class="sxs-lookup"><span data-stu-id="4d226-175">Also, consider designating one or more servers in each domain as [standby operations masters][standby-operations-masters] in case connectivity to a server acting as a flexible single master operation (FSMO) role fails.</span></span>

## <a name="manageability-considerations"></a><span data-ttu-id="4d226-176">Felügyeleti szempontok</span><span class="sxs-lookup"><span data-stu-id="4d226-176">Manageability considerations</span></span>

<span data-ttu-id="4d226-177">További információ a kezelési és monitorozási szempontokról: [Az Active Directory kiterjesztése az Azure-ra][adds-extend-domain].</span><span class="sxs-lookup"><span data-stu-id="4d226-177">For information about management and monitoring considerations, see [Extending Active Directory to Azure][adds-extend-domain].</span></span>

<span data-ttu-id="4d226-178">További információ: [Az Active Directory monitorozása][monitoring_ad].</span><span class="sxs-lookup"><span data-stu-id="4d226-178">For additional information, see [Monitoring Active Directory][monitoring_ad].</span></span> <span data-ttu-id="4d226-179">A feladatok megkönnyítéséhez olyan eszközöket is telepíthet a kezelési alhálózatban lévő monitorozási kiszolgálókra, mint a [Microsoft Systems Center][microsoft_systems_center].</span><span class="sxs-lookup"><span data-stu-id="4d226-179">You can install tools such as [Microsoft Systems Center][microsoft_systems_center] on a monitoring server in the management subnet to help perform these tasks.</span></span>

## <a name="security-considerations"></a><span data-ttu-id="4d226-180">Biztonsági szempontok</span><span class="sxs-lookup"><span data-stu-id="4d226-180">Security considerations</span></span>

<span data-ttu-id="4d226-181">Az erdőszintű bizalmi kapcsolatok tranzitívak.</span><span class="sxs-lookup"><span data-stu-id="4d226-181">Forest level trusts are transitive.</span></span> <span data-ttu-id="4d226-182">Ha erdőszintű bizalmi kapcsolatot hoz létre egy helyszíni és egy felhőben lévő erdő között, a bizalmi kapcsolat az ezekben az erdőkben létrehozott új tartományokra is kiterjed.</span><span class="sxs-lookup"><span data-stu-id="4d226-182">If you establish a forest level trust between an on-premises forest and a forest in the cloud, this trust is extended to other new domains created in either forest.</span></span> <span data-ttu-id="4d226-183">Ha biztonsági okokból tartományokat használ az elkülönítéshez, érdemes lehet csak tartományszintű bizalmi kapcsolatokat létrehoznia.</span><span class="sxs-lookup"><span data-stu-id="4d226-183">If you use domains to provide separation for security purposes, consider creating trusts at the domain level only.</span></span> <span data-ttu-id="4d226-184">A tartományszintű bizalmi kapcsolatok nem tranzitívak.</span><span class="sxs-lookup"><span data-stu-id="4d226-184">Domain level trusts are non-transitive.</span></span>

<span data-ttu-id="4d226-185">Active Directory-specifikus biztonsági szempontokért tekintse meg [Az Active Directory Azure-ra való kiterjesztését][adds-extend-domain] ismertető cikk biztonsági szempontokról szóló szakaszát.</span><span class="sxs-lookup"><span data-stu-id="4d226-185">For Active Directory-specific security considerations, see the security considerations section in [Extending Active Directory to Azure][adds-extend-domain].</span></span>

## <a name="deploy-the-solution"></a><span data-ttu-id="4d226-186">A megoldás üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="4d226-186">Deploy the solution</span></span>

<span data-ttu-id="4d226-187">Ennek az architektúrának egy üzemelő példánya elérhető a [GitHubon][github].</span><span class="sxs-lookup"><span data-stu-id="4d226-187">A deployment for this architecture is available on [GitHub][github].</span></span> <span data-ttu-id="4d226-188">Vegye figyelembe, hogy a teljes üzembe helyezés eltarthat akár két órát, amely tartalmazza a VPN gateway létrehozása, és futtatja a parancsfájlokat, amelyek az AD DS konfigurálása.</span><span class="sxs-lookup"><span data-stu-id="4d226-188">Note that the entire deployment can take up to two hours, which includes creating the VPN gateway and running the scripts that configure AD DS.</span></span>

### <a name="prerequisites"></a><span data-ttu-id="4d226-189">Előfeltételek</span><span class="sxs-lookup"><span data-stu-id="4d226-189">Prerequisites</span></span>

1. <span data-ttu-id="4d226-190">Klónozza, ágaztassa vagy töltse le a zip-fájlját a [GitHub-adattár](https://github.com/mspnp/identity-reference-architectures).</span><span class="sxs-lookup"><span data-stu-id="4d226-190">Clone, fork, or download the zip file for the [GitHub repository](https://github.com/mspnp/identity-reference-architectures).</span></span>

2. <span data-ttu-id="4d226-191">Telepítés [az Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span><span class="sxs-lookup"><span data-stu-id="4d226-191">Install [Azure CLI 2.0](/cli/azure/install-azure-cli?view=azure-cli-latest).</span></span>

3. <span data-ttu-id="4d226-192">Telepítse a [Azure építőelemeiről](https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks) npm-csomag.</span><span class="sxs-lookup"><span data-stu-id="4d226-192">Install the [Azure building blocks](https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks) npm package.</span></span>

   ```bash
   npm install -g @mspnp/azure-building-blocks
   ```

4. <span data-ttu-id="4d226-193">Parancsot a parancssorba bash-parancssorból vagy PowerShell-parancssorból, jelentkezzen be Azure-fiókjába a következő:</span><span class="sxs-lookup"><span data-stu-id="4d226-193">From a command prompt, bash prompt, or PowerShell prompt, sign into your Azure account as follows:</span></span>

   ```bash
   az login
   ```

### <a name="deploy-the-simulated-on-premises-datacenter"></a><span data-ttu-id="4d226-194">A szimulált helyszíni adatközpont üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="4d226-194">Deploy the simulated on-premises datacenter</span></span>

1. <span data-ttu-id="4d226-195">Keresse meg a `identity/adds-forest` mappájában, a GitHub-adattárban.</span><span class="sxs-lookup"><span data-stu-id="4d226-195">Navigate to the `identity/adds-forest` folder of the GitHub repository.</span></span>

2. <span data-ttu-id="4d226-196">Nyissa meg az `onprem.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="4d226-196">Open the `onprem.json` file.</span></span> <span data-ttu-id="4d226-197">Keresse meg példányait `adminPassword` és `Password` és adja meg a jelszavak adatait.</span><span class="sxs-lookup"><span data-stu-id="4d226-197">Search for instances of `adminPassword` and `Password` and add values for the passwords.</span></span>

3. <span data-ttu-id="4d226-198">Futtassa a következő parancsot, és várjon, amíg a telepítés befejezéséhez:</span><span class="sxs-lookup"><span data-stu-id="4d226-198">Run the following command and wait for the deployment to finish:</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p onprem.json --deploy
    ```

### <a name="deploy-the-azure-vnet"></a><span data-ttu-id="4d226-199">Az Azure virtuális hálózat üzembe helyezése</span><span class="sxs-lookup"><span data-stu-id="4d226-199">Deploy the Azure VNet</span></span>

1. <span data-ttu-id="4d226-200">Nyissa meg az `azure.json` fájlt.</span><span class="sxs-lookup"><span data-stu-id="4d226-200">Open the `azure.json` file.</span></span> <span data-ttu-id="4d226-201">Keresse meg példányait `adminPassword` és `Password` és adja meg a jelszavak adatait.</span><span class="sxs-lookup"><span data-stu-id="4d226-201">Search for instances of `adminPassword` and `Password` and add values for the passwords.</span></span>

2. <span data-ttu-id="4d226-202">Ugyanebben a fájlban keresse meg a példányok `sharedKey` , és adja meg a VPN-kapcsolat megosztott kulcsok.</span><span class="sxs-lookup"><span data-stu-id="4d226-202">In the same file, search for instances of `sharedKey` and enter shared keys for the VPN connection.</span></span>

    ```json
    "sharedKey": "",
    ```

3. <span data-ttu-id="4d226-203">Futtassa a következő parancsot, és várjon, amíg az üzembe helyezés befejeződik.</span><span class="sxs-lookup"><span data-stu-id="4d226-203">Run the following command and wait for the deployment to finish.</span></span>

    ```bash
    azbb -s <subscription_id> -g <resource group> -l <location> -p azure.json --deploy
    ```

   <span data-ttu-id="4d226-204">Telepítse a helyszíni virtuális hálózatnak ugyanabban az erőforráscsoportban.</span><span class="sxs-lookup"><span data-stu-id="4d226-204">Deploy to the same resource group as the on-premises VNet.</span></span>

### <a name="test-the-ad-trust-relation"></a><span data-ttu-id="4d226-205">Az AD megbízhatósági kapcsolat tesztelése</span><span class="sxs-lookup"><span data-stu-id="4d226-205">Test the AD trust relation</span></span>

1. <span data-ttu-id="4d226-206">Az Azure Portallal, keresse meg a létrehozott erőforráscsoportot.</span><span class="sxs-lookup"><span data-stu-id="4d226-206">Use the Azure portal, navigate to the resource group that you created.</span></span>

2. <span data-ttu-id="4d226-207">Az Azure portal használatával keresse meg a virtuális gép nevű `ra-adt-mgmt-vm1`.</span><span class="sxs-lookup"><span data-stu-id="4d226-207">Use the Azure portal to find the VM named `ra-adt-mgmt-vm1`.</span></span>

3. <span data-ttu-id="4d226-208">Kattintson a `Connect` parancsra egy, a virtuális gépre irányuló távoli asztali munkamenet megnyitásához.</span><span class="sxs-lookup"><span data-stu-id="4d226-208">Click `Connect` to open a remote desktop session to the VM.</span></span> <span data-ttu-id="4d226-209">A felhasználónév `contoso\testuser`, és a jelszó pedig a megadott a `onprem.json` alkalmazásparaméter-fájlt.</span><span class="sxs-lookup"><span data-stu-id="4d226-209">The username is `contoso\testuser`, and the password is the one that you specified in the `onprem.json` parameter file.</span></span>

4. <span data-ttu-id="4d226-210">A belül a távoli asztali munkamenetet, nyissa meg egy másik távoli asztali munkamenetet 192.168.0.4, amely az IP-címet a virtuális gép nevű `ra-adtrust-onpremise-ad-vm1`.</span><span class="sxs-lookup"><span data-stu-id="4d226-210">From inside your remote desktop session, open another remote desktop session to 192.168.0.4, which is the IP address of the VM named `ra-adtrust-onpremise-ad-vm1`.</span></span> <span data-ttu-id="4d226-211">A felhasználónév `contoso\testuser`, és a jelszó pedig a megadott a `azure.json` alkalmazásparaméter-fájlt.</span><span class="sxs-lookup"><span data-stu-id="4d226-211">The username is `contoso\testuser`, and the password is the one that you specified in the `azure.json` parameter file.</span></span>

5. <span data-ttu-id="4d226-212">A belül a távoli asztali munkamenetet `ra-adtrust-onpremise-ad-vm1`, lépjen a **Kiszolgálókezelő** kattintson **eszközök** > **Active Directory-tartományok és megbízhatósági kapcsolatok**.</span><span class="sxs-lookup"><span data-stu-id="4d226-212">From inside the remote desktop session for `ra-adtrust-onpremise-ad-vm1`, go to **Server Manager** and click **Tools** > **Active Directory Domains and Trusts**.</span></span>

6. <span data-ttu-id="4d226-213">A bal oldali ablaktáblán kattintson a jobb gombbal a contoso.com, és válassza **tulajdonságok**.</span><span class="sxs-lookup"><span data-stu-id="4d226-213">In the left pane, right-click on the contoso.com and select **Properties**.</span></span>

7. <span data-ttu-id="4d226-214">Kattintson a **megbízik** fülre. Egy bejövő bizalmi kapcsolat állapottal treyresearch.net kell megjelennie.</span><span class="sxs-lookup"><span data-stu-id="4d226-214">Click the **Trusts** tab. You should see treyresearch.net listed as an incoming trust.</span></span>

![Képernyőkép az Active Directory erdőszintű bizalmi kapcsolat párbeszédpanel](./images/ad-forest-trust.png)

## <a name="next-steps"></a><span data-ttu-id="4d226-216">További lépések</span><span class="sxs-lookup"><span data-stu-id="4d226-216">Next steps</span></span>

- <span data-ttu-id="4d226-217">A [helyszíni AD DS-tartomány Azure-ra való kiterjesztésére][adds-extend-domain] vonatkozó ajánlott eljárások megismerése</span><span class="sxs-lookup"><span data-stu-id="4d226-217">Learn the best practices for [extending your on-premises AD DS domain to Azure][adds-extend-domain]</span></span>
- <span data-ttu-id="4d226-218">Az Azure-alapú [AD FS-infrastruktúra létrehozására][adfs] vonatkozó ajánlott eljárások megismerése.</span><span class="sxs-lookup"><span data-stu-id="4d226-218">Learn the best practices for [creating an AD FS infrastructure][adfs] in Azure.</span></span>

<!-- links -->
[adds-extend-domain]: adds-extend-domain.md
[adfs]: adfs.md
[azure-cli-2]: /azure/install-azure-cli
[azbb]: https://github.com/mspnp/template-building-blocks/wiki/Install-Azure-Building-Blocks

[implementing-a-secure-hybrid-network-architecture]: ../dmz/secure-vnet-hybrid.md
[implementing-a-secure-hybrid-network-architecture-with-internet-access]: ../dmz/secure-vnet-dmz.md

[running-VMs-for-an-N-tier-architecture-on-Azure]: ../virtual-machines-windows/n-tier.md

[ad-azure-guidelines]: https://msdn.microsoft.com/library/azure/jj156090.aspx
[azure-expressroute]: /azure/expressroute/expressroute-introduction
[azure-vpn-gateway]: /azure/vpn-gateway/vpn-gateway-about-vpngateways
[considerations]: ./considerations.md
[creating-external-trusts]: https://technet.microsoft.com/library/cc816837(v=ws.10).aspx
[creating-forest-trusts]: https://technet.microsoft.com/library/cc816810(v=ws.10).aspx
[github]: https://github.com/mspnp/identity-reference-architectures/tree/master/adds-forest
[incoming-trust]: https://raw.githubusercontent.com/mspnp/identity-reference-architectures/master/adds-forest/extensions/incoming-trust.ps1
[microsoft_systems_center]: https://microsoft.com/cloud-platform/system-center
[monitoring_ad]: https://msdn.microsoft.com/library/bb727046.aspx
[resource-manager-overview]: /azure/azure-resource-manager/resource-group-overview
[solution-script]: https://raw.githubusercontent.com/mspnp/identity-reference-architectures/master/adds-forest/Deploy-ReferenceArchitecture.ps1
[standby-operations-masters]: https://technet.microsoft.com/library/cc794737(v=ws.10).aspx
[outgoing-trust]: https://raw.githubusercontent.com/mspnp/identity-reference-architectures/master/adds-forest/extensions/outgoing-trust.ps1
[verify-a-trust]: https://technet.microsoft.com/library/cc753821.aspx
[visio-download]: https://archcenter.blob.core.windows.net/cdn/identity-architectures.vsdx
