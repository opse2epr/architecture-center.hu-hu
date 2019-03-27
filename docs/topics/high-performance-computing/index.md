---
title: Nagy teljesítményű feldolgozás (HPC) az Azure-ban
description: Útmutató Azure-on futó HPC számítási feladatok készítéséhez
author: adamboeglin
ms.date: 2/4/2019
---
<!-- markdownlint-disable MD033 -->
<!-- markdownlint-disable MD026 -->

# <a name="high-performance-computing-hpc-on-azure"></a>Nagy teljesítményű feldolgozás (HPC) az Azure-ban

## <a name="introduction-to-hpc"></a>A HPC bemutatása

<!-- markdownlint-disable MD034 -->

> [!VIDEO https://www.youtube.com/embed/rKURT32faJk]

<!-- markdownlint-enable MD034 -->

A nagy teljesítményű feldolgozás (HPC), más néven „Big Compute” számos CPU- vagy GPU-alapú számítógépet használ összetett matematikai feladatok megoldására.

Számos iparág használ HPC-t a legnehezebb problémák megoldásához.  Ilyenek többek között a következő számítási feladatok:

- Genomics
- Olaj- és gázszimulációk
- Pénzügy
- Félvezetők tervezése
- Mérnöki tevékenységek
- Időjárás-modellezés

### <a name="how-is-hpc-different-on-the-cloud"></a>Miben más a felhőbeli HPC?

A helyszíni és a felhőbeli HPC-rendszerek közötti egyik legfontosabb különbség, hogy az erőforrások igény szerint, dinamikusan adhatók hozzá vagy távolíthatók el.  A dinamikus skálázás megszünteti a számítási kapacitás okozta szűk keresztmetszetet, és így lehetővé teszi, hogy az ügyfelek a feladataik követelményeinek megfelelően igazítsák az infrastruktúrájuk méretét.

A következő cikk további részleteket tartalmaz a dinamikus méretezési képességről.

- [A Big Compute architektúrastílus](/azure/architecture/guide/architecture-styles/big-compute?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Automatikus skálázással kapcsolatos ajánlott eljárások](/azure/architecture/best-practices/auto-scaling?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="implementation-checklist"></a>Megvalósítási ellenőrzőlista

Amikor saját HPC-megoldást tervez megvalósítani az Azure-on, mindenképpen tekintse át a következő témaköröket:

<!-- markdownlint-disable MD032 -->

> [!div class="checklist"]
> - A követelményei alapján válassza ki a megfelelő [architektúrát](#infrastructure)
> - Legyen tisztában azzal, hogy melyik [számítási](#compute) lehetőségek felelnek meg számítási feladataihoz
> - Azonosítsa az igényeinek megfelelő [tárolási](#storage) megoldást
> - Döntse el, hogyan fogja [kezelni](#management) az összes erőforrást
> - Optimalizálja [alkalmazását](#hpc-applications) a felhőhöz
> - Gondoskodjon az infrastruktúra [védelméről](#security)

<!-- markdownlint-enable MD032 -->

## <a name="infrastructure"></a>Infrastruktúra

Számos infrastruktúra-összetevőre van szükség egy HPC-rendszer felépítéséhez.  A HPC számítási feladatok választott kezelési módjától függetlenül a Compute, a Storage és a Networking adja meg az alapvető összetevőket.

### <a name="example-hpc-architectures"></a>Példák HPC-architektúrákra

Számos különböző módon tervezheti és valósíthatja meg HPC-architektúráját az Azure-on.  A HPC-alkalmazások több ezer számítási magra is felskálázhatók, kiterjeszthetik a helyszíni fürtöket, de 100%-ban natív felhőalapú megoldásként is futtathatók.

A következő forgatókönyvek a HPC-megoldások felépítésének néhány gyakori módját írják le.

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/apps/hpc-saas?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/apps/media/architecture-hpc-saas.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">CAE-szolgáltatások az Azure-ban</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>SaaS-platformot biztosíthat CAE-projektekhez az Azure-ban.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/infrastructure/hpc-cfd?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/infrastructure/media/architecture-hpc-cfd.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Számítási folyadékdinamikai (CFD) szimulációk az Azure-ban</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Számítási folyadékdinamikai (CFD) szimulációkat hajthat végre az Azure-ban.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/infrastructure/video-rendering?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/infrastructure/media/architecture-video-rendering.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">3D-s videórenderelés az Azure-ban</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Natív HPC számítási feladatok futtatása az Azure-ban az Azure Batch szolgáltatás használatával</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

### <a name="compute"></a>Compute

Az Azure számos, nagy CPU- és GPU-igényű számítási feladatokhoz optimalizált méretet nyújt.

#### <a name="cpu-based-virtual-machines"></a>CPU-alapú virtuális gépek

- [Linux rendszerű virtuális gépek](/azure/virtual-machines/linux/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Windows rendszerű virtuális gépek](/azure/virtual-machines/windows/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) virtuális gépei
  
#### <a name="gpu-enabled-virtual-machines"></a>GPU-kompatibilis virtuális gépek

Az N-sorozatú virtuális gépeken NVIDIA GPU-k találhatók, amelyek nagy számítási igényű vagy nagy grafikai igényű alkalmazásokhoz vannak tervezve, például mesterséges intelligencia (AI) tanításához és vizualizációhoz.

- [Linux rendszerű virtuális gépek](/azure/virtual-machines/linux/sizes-gpu?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Windows rendszerű virtuális gépek](/azure/virtual-machines/windows/sizes-gpu?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

### <a name="storage"></a>Storage

A nagy méretű Batch és HPC számítási feladatok olyan adattárolási és hozzáférési igényekkel rendelkeznek, amelyek meghaladják a hagyományos felhőalapú fájlrendszerek képességeit.  Számos megoldás létezik az Azure-ban futó HPC-alkalmazások sebesség- és kapacitásigényeinek kezelésére

- [Avere vFXT](https://azure.microsoft.com/services/storage/avere-vfxt/) – gyorsabb, hozzáférhetőbb adattárolás a peremhálózaton végzett nagy teljesítményű számításokhoz
- [BeeGFS](https://azure.microsoft.com/resources/implement-glusterfs-on-azure/)
- [Tárolásra optimalizált virtuális gépek](/azure/virtual-machines/windows/sizes-storage?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Blob, tábla és Queue Storage](/azure/storage/storage-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Azure SMB fájltároló](/azure/storage/files/storage-files-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Intel Cloud Edition Lustre](https://azuremarketplace.microsoft.com/marketplace/apps/intel.intel-cloud-edition-gs)

Az Azure-on használt Lustre, GlusterFS és BeeGFS összehasonlításáért tekintse meg [a párhuzamos fájlrendszerekkel foglalkozó Azure e-könyvet](https://blogs.msdn.microsoft.com/azurecat/2018/06/11/azurecat-ebook-parallel-virtual-file-systems-on-microsoft-azure/).

### <a name="networking"></a>Hálózat

A H16r, H16mr, A8 és A9 virtuális gépek magas teljesítményű, háttérben futó RDMA-hálózathoz csatlakozhatnak. Ez a hálózat javíthatja a Microsoft MPI vagy az Intel MPI alatt futó szorosan összekapcsolt párhuzamos alkalmazások teljesítményét.

- [RDMA-kompatibilis példányok](/azure/virtual-machines/windows/sizes-hpc?context=/azure/architecture/topics/high-performance-computing/context/hpc-context#rdma-capable-instances)
- [Virtual Network](/azure/virtual-network/virtual-networks-overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [ExpressRoute](/azure/expressroute/expressroute-introduction?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="management"></a>Kezelés

### <a name="do-it-yourself"></a>Önkiszolgáló

A HPC-rendszerek az alapoktól való felépítése az Azure-ban jelentős mértékű rugalmasságot nyújt, de gyakran nagyon sok karbantartást igényel.  

1. Állítsa be saját fürtkörnyezetét az Azure-beli virtuális gépeken vagy a [virtuálisgép-méretezési csoportokban](/azure/virtual-machine-scale-sets/overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).
2. Helyezzen üzembe vezető [számításifeladat-kezelőket](#workload-managers), infrastruktúrát és [alkalmazásokat](#hpc-applications) Azure Resource Manager-sablonok segítségével.
3. Válasszon [HPC és GPU virtuálisgép-méreteket](#hpc-and-gpu-sizes), amelyek speciális hardvert és hálózati kapcsolatokat tartalmaznak az MPI és GPU számítási feladatokhoz.
4. Adjon hozzá [nagy teljesítményű tárolót](#hpc-storage) az I/O-igényes számítási feladatok számára.

### <a name="hybrid-and-cloud-bursting"></a>Hibrid és felhőalapú teljesítménynövelés

Ha meglévő helyszíni HPC rendszere van, amelyet az Azure-hoz szeretne csatlakoztatni, számos erőforrás segíthet ennek megkezdésében.

Először tekintse át a dokumentáció [A helyszíni hálózat Azure-hoz való csatlakoztatásának lehetőségei](/azure/architecture/reference-architectures/hybrid-networking/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) című cikkét.  Ezután a következő csatlakoztatási lehetőségekről szerezhet információt:

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/vpn?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/vpn.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Helyszíni hálózat csatlakoztatása az Azure-hoz VPN-átjáró használatával</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Ez a referenciaarchitektúra bemutatja, hogyan lehet kibővíteni a helyszíni hálózatot az Azure-ra helyek közötti virtuális magánhálózat (VPN) használatával.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/expressroute?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/expressroute.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Helyszíni hálózat csatlakoztatása az Azure-hoz ExpressRoute használatával</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Az ExpressRoute-kapcsolatok dedikált, privát kapcsolatot használnak egy külső kapcsolatszolgáltatón keresztül. A privát kapcsolat kiterjeszti a helyszíni hálózatot az Azure-ba.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/expressroute-vpn-failover?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/expressroute-vpn-failover.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Helyszíni hálózat csatlakoztatása az Azure-hoz VPN-feladatátvételt biztosító ExpressRoute használatával</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Egy helyek közötti, magas rendelkezésre állású és biztonságos hálózati architektúrát építhet ki, amely a VPN-átjáróval feladatátvételt biztosító ExpressRoute használatával összekapcsolt Azure-beli virtuális hálózatból és helyszíni hálózatból áll.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

A hálózati kapcsolat biztonságos létrejötte után megkezdheti a felhőalapú számítási erőforrások igény szerinti használatát a meglévő [számításifeladat-kezelő](#workload-manager) teljesítménynövelési képességeivel.

### <a name="marketplace-solutions"></a>A Marketplace-ről származó megoldások

Az [Azure Marketplace-en](https://azuremarketplace.microsoft.com/marketplace/) számos számításifeladat-kezelő érhető el.

- [RogueWave CentOS-alapú HPC](https://azuremarketplace.microsoft.com/marketplace/apps/RogueWave.CentOSbased73HPC?tab=Overview)
- [SUSE Linux Enterprise Server HPC-hez](https://azure.microsoft.com/marketplace/partners/suse/suselinuxenterpriseserver12optimizedforhighperformancecompute/)
- [TIBCO Grid Server motor](https://azuremarketplace.microsoft.com/marketplace/apps/tibco-software.gridserverlinuxengine?tab=Overview)
- [Windows és Linux rendszerhez való Azure Data Science VM](/azure/machine-learning/data-science-virtual-machine/overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [D3View](https://azuremarketplace.microsoft.com/marketplace/apps/xfinityinc.d3view-v5?tab=Overview)
- [UberCloud](https://azure.microsoft.com/search/marketplace/?q=ubercloud)

### <a name="azure-batch"></a>Azure Batch

Az [Azure Batch](/azure/batch/batch-technical-overview?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) platformszolgáltatás lehetővé teszi, hogy hatékonyan futtasson nagyméretű párhuzamos és nagy teljesítményű feldolgozási (HPC) alkalmazásokat a felhőben. Az Azure Batch számításigényes munkák futtatását ütemezi virtuális gépek felügyelt készletében, és automatikusan képes méretezni a számítási erőforrásokat a feladatok igényeinek megfelelően.

A SaaS-szolgáltatók vagy -fejlesztők a Batch SDK-kal és -eszközökkel HPC-alkalmazásokat vagy tárolókhoz kapcsolódó számítási feladatokat integrálhatnak az Azure-ral, adatokat bocsáthatnak rendelkezésre az Azure-ba, valamint feladat-végrehajtási folyamatokat hozhatnak létre.

### <a name="azure-cyclecloud"></a>Azure CycleCloud

Az [Azure CycleCloud](/azure/cyclecloud/?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) nyújtja a legegyszerűbb módszert a HPC számítási feladatok bármilyen ütemezővel (például Slurm, Grid Engine, HPC Pack, HTCondor, LSF, PBS Pro vagy Symphony) történő kezelésére az Azure-ban.

A CycleCloud a következőket teszi lehetővé:

- Teljes fürtök és más erőforrások (ütemező, számítási virtuális gépek, tárterület, hálózat és gyorsítótár) üzembe helyezése
- A feladatok, az adatok és a felhő munkafolyamatainak összehangolása
- Teljes körű szabályozás biztosítása a rendszergazdáknak afölött, hogy melyik felhasználók futtathatnak feladatokat, illetve hol és milyen költségek mellett tehetik meg ezt.
- Fürtök testreszabása és optimalizálása speciális szabályzatok és irányítási funkciók, például költségvezérlés, Active Directory-integráció, monitorozás és jelentéskészítés segítségével
- A jelenlegi feladatütemező és alkalmazások használata módosítás nélkül
- Beépített, automatikus skálázási funkció és élesben kipróbált referenciaarchitektúrák számos HPC számítási feladathoz és iparághoz

### <a name="workload-managers"></a>Számításifeladat-kezelők

Az alábbiakban néhány példa látható az Azure-infrastruktúrában futtatható fürt- és számításifeladat-kezelőkre. Önálló fürtöket hozhat létre az Azure-beli virtuális gépeken, illetve adatlöketet vihet át rájuk egy helyszíni fürtről.

- [Alces Flight Compute](https://azuremarketplace.microsoft.com/marketplace/apps/alces-flight-limited.alces-flight-compute-solo?tab=Overview)
- [TIBCO DataSynapse GridServer](https://azure.microsoft.com/blog/tibco-datasynapse-comes-to-the-azure-marketplace/)
- [Bright Cluster Manager](http://www.brightcomputing.com/technology-partners/microsoft)
- [IBM Spectrum Symphony és Symphony LSF](https://azure.microsoft.com/blog/ibm-and-microsoft-azure-support-spectrum-symphony-and-spectrum-lsf/)
- [PBS Pro](http://pbspro.org)
- [Altair](http://www.altair.com/)
- [Rescale](https://www.rescale.com/azure/)
- [Microsoft HPC Pack](https://technet.microsoft.com/library/mt744885.aspx)
  - [Windows rendszerhez készült HPC Pack](/azure/virtual-machines/windows/hpcpack-cluster-options?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
  - [Linux rendszerhez készült HPC Pack](/azure/virtual-machines/linux/hpcpack-cluster-options?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

#### <a name="containers"></a>Containers

Néhány HPC számítási feladat kezeléséhez tárolók is használhatók.  Az Azure Kubernetes Service-hez (AKS) hasonló szolgáltatásokkal egyszerűen helyezhetők üzembe a felügyelt Kubernetes-fürtök az Azure-ban.

- [Azure Kubernetes Service (AKS)](/azure/aks/intro-kubernetes?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Container Registry](/azure/container-registry/container-registry-intro?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="cost-management"></a>Cost Management

A HPC költségeinek kezelése az Azure-on különböző módokon végezhető el.  Mindenképpen tekintse át az [Azure vásárlási lehetőségeit](https://azure.microsoft.com/pricing/purchase-options/) a cégének legmegfelelőbb módszer kiválasztásához.

Az [alacsony prioritású virtuális gépekkel](/azure/virtual-machine-scale-sets/virtual-machine-scale-sets-use-low-priority?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) jelentős költségmegtakarítás mellett használhatja ki a nem használt kapacitásunkat.

## <a name="security"></a>Biztonság

Az Azure ajánlott biztonsági eljárásait az [Azure Security dokumentációjában](/azure/security/azure-security?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) találja.  

A [Felhőalapú teljesítménynövelés](#hybrid-and-cloud-bursting) szakaszban elérhető hálózati konfigurációk mellett érdemes lehet megvalósítani egy küllős konfigurációt a hálózati erőforrások elkülönítése érdekében:

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/hub-spoke.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Küllős hálózati topológia implementálása az Azure-ban</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Az agy egy virtuális hálózat (VNet) az Azure-ban, amely központi kapcsolódási pontként szolgál a helyszíni hálózathoz. A küllők az agyhoz kapcsolódó virtuális hálózatok, és a számítási feladatok elkülönítésére használhatók.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/reference-architectures/hybrid-networking/shared-services?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="/azure/architecture/reference-architectures/hybrid-networking/images/shared-services.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Küllős hálózati topológia implementálása megosztott szolgáltatásokkal az Azure-ban</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>Ez a referenciaarchitektúra a küllős referenciaarchitektúrára épít, hogy az összes küllő által használható megosztott szolgáltatásokat lehessen foglalni a központba.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="hpc-applications"></a>HPC-alkalmazások

Egyéni vagy kereskedelmi HPC-alkalmazások futtatása az Azure-ban. Az ebben a szakaszban szereplő számos példa tesztelve lett, hogy hatékonyan lehessen méretezni további virtuális gépekkel vagy számítási magokkal. Az [Azure Marketplace-en](https://azuremarketplace.microsoft.com/marketplace) üzembe helyezésre kész megoldásokat talál.

> [!NOTE]
> Minden kereskedelmi alkalmazás szállítójánál érdeklődjön a felhőbeli futtatásra vonatkozó licencelési vagy egyéb korlátozásokról. Nem minden szállító kínál használatalapú licencet. Lehet, hogy a megoldásához licenckiszolgálót kell használnia a felhőben, vagy helyszíni licenckiszolgálóhoz kell csatlakoznia.

### <a name="engineering-applications"></a>Mérnöki alkalmazások

- [Altair RADIOSS](https://azure.microsoft.com/blog/availability-of-altair-radioss-rdma-on-microsoft-azure/)
- [ANSYS CFD](https://azure.microsoft.com/blog/ansys-cfd-and-microsoft-azure-perform-the-best-hpc-scalability-in-the-cloud/)
- [MATLAB Distributed Computing Server](/azure/virtual-machines/windows/matlab-mdcs-cluster?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [StarCCM+](https://blogs.msdn.microsoft.com/azurecat/2017/07/07/run-star-ccm-in-an-azure-hpc-cluster/)
- [OpenFOAM](https://simulation.azure.com/casestudies/Team-182-ABB-UC-Final.pdf)

### <a name="graphics-and-rendering"></a>Grafika és renderelés

- [Autodesk Maya, 3ds Max és Arnold](/azure/batch/batch-rendering-service?context=/azure/architecture/topics/high-performance-computing/context/hpc-context) az Azure Batch-en

### <a name="ai-and-deep-learning"></a>AI és mély tanulás

- [Microsoft Cognitive Toolkit](/cognitive-toolkit/cntk-on-azure)
- [Deep Learning VM](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-ads.dsvm-deep-learning)
- [Batch Shipyard mélytanulási módszerek](https://github.com/Azure/batch-shipyard/tree/master/recipes#deeplearning)

### <a name="mpi-providers"></a>MPI-szolgáltatók

- [Microsoft MPI](/message-passing-interface/microsoft-mpi)

## <a name="remote-visualization"></a>Távoli vizualizáció

<ul class="columns is-multiline has-margin-left-none has-margin-bottom-none has-padding-top-medium">
    <li class="column is-one-third has-padding-top-small-mobile has-padding-bottom-small">
        <a class="is-undecorated is-full-height is-block"
            href="/azure/architecture/example-scenario/infrastructure/linux-vdi-citrix?context=/azure/architecture/topics/high-performance-computing/context/hpc-context">
            <article class="card has-outline-hover is-relative is-fullheight">
                    <figure class="image has-margin-right-none has-margin-left-none has-margin-top-none has-margin-bottom-none">
                        <img role="presentation" alt="" src="../../example-scenario/infrastructure/media/azure-citrix-sample-diagram.png">
                    </figure>
                <div class="card-content has-text-overflow-ellipsis">
                    <div class="has-padding-bottom-none">
                        <h3 class="is-size-4 has-margin-top-none has-margin-bottom-none has-text-primary">Linux rendszerű virtuális asztali környezetek a Citrix-szel</h3>
                    </div>
                    <div class="is-size-7 has-margin-top-small has-line-height-reset">
                        <p>VDI-környezetet hozhat létre Linux rendszerű asztali környezetekhez a Citrix használatával az Azure-ban.</p>
                    </div>
                </div>
            </article>
        </a>
    </li>
</ul>

## <a name="performance-benchmarks"></a>Teljesítménymérések

- [Számításiteljesítmény-mérések](/azure/virtual-machines/windows/compute-benchmark-scores?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)

## <a name="customer-stories"></a>Ügyfelek történetei

Sok ügyfél ért el hatalmas sikereket az Azure HPC számítási feladatokhoz való használata során.  Az alábbiakban néhány ilyen ügyfél esettanulmányát láthatja:

- [ANEO](https://customers.microsoft.com/story/it-provider-finds-highly-scalable-cloud-based-hpc-redu)
- [AXA Global P&C](https://customers.microsoft.com/story/axa-global-p-and-c)
- [Axioma](https://customers.microsoft.com/story/axioma-delivers-fintechs-first-born-in-the-cloud-multi-asset-class-enterprise-risk-solution)
- [d3View](https://customers.microsoft.com/story/big-data-solution-provider-adopts-new-cloud-gains-thou)
- [EFS](https://customers.microsoft.com/story/efs-professionalservices-azure)
- [Hymans Robertson](https://customers.microsoft.com/story/hymans-robertson)
- [MetLife](https://enterprise.microsoft.com/customer-story/industries/insurance/metlife/)
- [Microsoft Research](https://customers.microsoft.com/doclink/fast-lmm-and-windows-azure-put-genetics-research-on-fa)
- [Milliman](https://customers.microsoft.com/story/actuarial-firm-works-to-transform-insurance-industry-w)
- [Mitsubishi UFJ Securities International](https://customers.microsoft.com/story/powering-risk-compute-grids-in-the-cloud)
- [NeuroInitiative](https://customers.microsoft.com/story/neuroinitiative-health-provider-azure)
- [Schlumberger](https://azure.microsoft.com/blog/big-compute-for-large-engineering-simulations)
- [Towers Watson](https://customers.microsoft.com/story/insurance-tech-provider-delivers-disruptive-solutions)

## <a name="other-important-information"></a>További fontos információk

- A nagy méretű számítási feladatok futtatásának megkísérlése előtt győződjön meg arról, hogy megnövelte a [vCPU-kvótát](/azure/virtual-machines/linux/quotas?context=/azure/architecture/topics/high-performance-computing/context/hpc-context).

## <a name="next-steps"></a>További lépések

A legújabb bejelentéseket itt találja:

- [A Microsoft HPC és Batch csapatának blogja](http://blogs.technet.com/b/windowshpc/)
- Látogasson el az [Azure blogra](https://azure.microsoft.com/blog/tag/hpc/).

### <a name="microsoft-batch-examples"></a>Microsoft Batch – példák

Ezek az oktatóanyagok az alkalmazások Microsoft Batch-en való futtatásának részleteit tartalmazzák

- [A fejlesztés megkezdése a Batch használatával](/azure/batch/quick-run-dotnet?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Az Azure Batch kódmintáinak használata](https://github.com/Azure/azure-batch-samples)
- [Alacsony prioritású virtuális gépek használata a Batch szolgáltatással](/azure/batch/batch-low-pri-vms?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)
- [Tárolóalapú HPC számítási feladatok futtatása a Batch Shipyard használatával](https://github.com/Azure/batch-shipyard)
- [Párhuzamos R számítási feladatok futtatása a Batch szolgáltatásban](https://github.com/Azure/doAzureParallel)
- [Igény szerinti Spark-feladatok futtatása a Batch szolgáltatásban](https://github.com/Azure/aztk)
- [Nagy számítási igényű virtuális gépek használata Batch-készletekben](/azure/batch/batch-pool-compute-intensive-sizes?context=/azure/architecture/topics/high-performance-computing/context/hpc-context)