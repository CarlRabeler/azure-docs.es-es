---
title: 'Alta disponibilidad: servicio Azure SQL Database | Microsoft Docs'
description: Obtenga información acerca de las funcionalidades y características de alta disponibilidad de Azure SQL Database
services: sql-database
author: anosov1960
manager: craigg
ms.service: sql-database
ms.topic: article
ms.date: 03/19/2018
ms.author: sashan
ms.reviewer: carlrab
ms.openlocfilehash: d26fe28d301cf563dc6bdb3d9e17903dea3e73fc
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/23/2018
---
# <a name="high-availability-and-azure-sql-database"></a>Alta disponibilidad y Azure SQL Database
Desde el comienzo de la oferta de PaaS de Azure SQL Database, Microsoft se comprometió a integrar la alta disponibilidad (HA) en el servicio para que los clientes no tuvieran que administrar, agregar lógicas especiales o tomar decisiones acerca de la alta disponibilidad. Microsoft ofrece a sus clientes un Acuerdo de Nivel de Servicio para mantener un control total sobre la configuración y el funcionamiento del sistema de alta disponibilidad. El Acuerdo de Nivel de Servicio de alta disponibilidad se aplica a una base de datos SQL en una región y no proporciona protección en los casos de que se produzca un error total en la región debido a factores externos que estén fuera del control de Microsoft (por ejemplo, desastres naturales, guerras, terrorismo, disturbios, acción gubernamental o un error de red o dispositivos externo a los centros de datos de Microsoft, incluido el sitio del cliente o entre este sitio y el centro de datos de Microsoft).

Para simplificar el problema de espacio de la alta disponibilidad, Microsoft usa los siguientes supuestos:
1.  Los errores de hardware y software son inevitables.
2.  El personal de operaciones comete errores que dar lugar a problemas.
3.  Las operaciones de mantenimiento planeadas pueden provocar interrupciones. 

Aunque dichos eventos individuales son poco frecuentes, si incluimos la nube, estos se pueden producir semanal o diariamente. 

## <a name="fault-tolerant-sql-databases"></a>Bases de datos SQL con tolerancia a errores
Los clientes están especialmente interesados en la resistencia de sus propias bases de datos y les interesa menos la resistencia del servicio SQL Database en general. Un tiempo de actividad del 99,99 % en un servicio no tiene sentido si "mi base de datos" forma parte del 0,01 % de las bases de datos que están inactivas. Cada base de datos debe poder tolerar errores y la mitigación de estos nunca debe derivarse en la pérdida de una transacción confirmada. 

En cuanto a los datos, SQL Database utiliza el almacenamiento local (LS) basado en discos y discos duros virtuales adjuntados directamente y en almacenamiento remoto (RS) basado en los blobs en páginas de Azure Premium Storage. 
- El almacenamiento local se utiliza en las bases de datos y grupos Premium que están diseñados para aplicaciones de OLTP críticas con grandes requisitos de IOPS. 
- El almacenamiento remoto se utiliza en los niveles de servicio Básico y Estándar, diseñados para cargas de trabajo con un presupuesto limitado que necesiten un almacenamiento y una capacidad de proceso que escalen de forma independiente. Usan un blob en páginas individual para archivos de registro y bases de datos y tienen mecanismos de conmutación por error y replicación de almacenamiento integrados.

En ambos casos, la replicación, la detección de errores y los mecanismos de conmutación por error de SQL Database están totalmente automatizados y operan sin intervención humana. Esta arquitectura está diseñada para garantizar que nunca se pierdan datos confirmados y que la durabilidad de esos datos tenga prioridad sobre todo lo demás.

Ventajas principales:
- Los clientes obtienen todas las ventajas de las bases de datos replicadas sin tener que configurar o mantener hardware, software, sistemas operativos o entornos de virtualización complicados.
- El sistema mantiene las propiedades ACID completas de las bases de datos relacionales.
- Las conmutaciones por error están totalmente automatizados para que no pierdan datos confirmados.
- El servicio se encarga de administrar de forma dinámica el enrutamiento de conexiones a la réplica principal sin necesidad de lógicas de aplicación.
- El alto nivel de redundancia automatizada se proporciona sin cargo adicional.

> [!NOTE]
> Es posible que la arquitectura de alta disponibilidad descrita cambie sin previo aviso. 

## <a name="data-redundancy"></a>Redundancia de datos

La solución de alta disponibilidad en SQL Database se basa en la tecnología [Grupos de disponibilidad AlwaysOn](/sql/database-engine/availability-groups/windows/overview-of-always-on-availability-groups-sql-server) de SQL Server para que puedan funcionar las bases de datos LS y RS con diferencias mínimas. En la configuración de LS, la tecnología del grupo de disponibilidad AlwaysOn se usa para la persistencia, mientras que en RS se usa para la disponibilidad (RTO bajo). 

## <a name="local-storage-configuration"></a>Configuración de almacenamiento local

En esta configuración, el servicio de administración (MS) es el encargado de poner en línea cada base de datos dentro del anillo de control. Una réplica principal y al menos dos réplicas secundarias (conjunto de cuórum) se ubican en el anillo del inquilino que abarca tres subsistemas físicos independientes dentro del mismo centro de datos. Todas las lecturas y escrituras se envían por la puerta de enlace (GW) a la réplica principal y las escrituras se replican asincrónicamente en las réplicas secundarias. SQL Database emplea un esquema de confirmación basado en cuórum donde los datos se escriben en la réplica principal y en una de las secundarias antes de que la transacción se considere confirmada.

El sistema de conmutación por error de [Service Fabric](../service-fabric/service-fabric-overview.md) recompila de forma automática réplicas a medida que se van produciendo errores en los nodos y mantiene la pertenencia al conjunto del cuórum a medida que los nodos entran y salen del sistema. El mantenimiento planeado se coordina con sumo cuidado para evitar que el conjunto de cuórum caiga por debajo de un recuento de réplica mínimo (normalmente 2). Este modelo funciona bien en bases de datos Premium, pero requiere redundancia de ambos componentes de proceso y almacenamiento y tiene un costo mayor.

## <a name="remote-storage-configuration"></a>Configuración de almacenamiento remoto

En las configuraciones de almacenamiento remoto (niveles Básico y Estándar), se guarda exactamente una copia en el almacenamiento de blobs remoto y utiliza las funcionalidades de los sistemas de almacenamiento para poder detectar la degradación de bits, la redundancia y la durabilidad. 

La arquitectura de alta disponibilidad se muestra en el diagrama siguiente:
 
![arquitectura de alta disponibilidad](./media/sql-database-high-availability/high-availability-architecture.png)

## <a name="failure-detection-and-recovery"></a>Detección de errores y recuperación 
Un sistema distribuido a gran escala necesita tener un sistema de detección de errores que sea altamente fiable y que pueda detectar errores de forma precisa, rápidamente y lo más cerca posible del cliente. En SQL Database, este sistema se basa en Azure Service Fabric. 

Con la réplica principal, se hace evidente inmediatamente si esta ha fallado y cuándo lo ha hecho; debido a ello, el trabajo no puede continuar porque todas las lecturas y escrituras tienen lugar primero en la réplica principal. Este proceso para convertir una réplica secundaria en principal tiene un objetivo de tiempo de recuperación (RTO) de 30 segundos y un objetivo de punto de recuperación (RPO) de 0. Para mitigar el impacto del RTO de 30 segundos, lo mejor es intentar reconectarse varias veces con un tiempo de espera menor error para los intentos fallidos de conexión.

Cuando se produce un error en una réplica secundaria, la base de datos baja hasta un conjunto de cuórum mínimo y sin reservas. Service Fabric inicia el proceso de reconfiguración de forma similar al proceso que sigue al error de la réplica principal, por lo que después de una breve espera para determinar si el error es permanente, se crea otra réplica secundaria. En los casos de estado de fuera de servicio temporal como, por ejemplo, un error del sistema operativo o una actualización, no se genera inmediatamente la nueva réplica para permitir que el nodo con error se reinicie. 

En cuanto a las configuraciones de almacenamiento remoto,SQL Database usa la funcionalidad AlwaysON en bases de datos de conmutación por error durante las actualizaciones. Para ello, una nueva instancia de SQL se deriva de antemano como parte de los eventos planeados de la actualización y se adjunta para recuperar el archivo de base de datos del almacenamiento remoto. En caso de que se bloquee el proceso o sucedan otros eventos no planeados, Windows Fabric se encarga de administrar la disponibilidad de las instancias y, como último paso de recuperación, adjunta el archivo de base de datos remota.

## <a name="zone-redundant-configuration-preview"></a>Configuración de redundancia de zona (versión preliminar)

De forma predeterminada, se crean las réplicas del conjunto de cuórum para las configuraciones de almacenamiento local en el mismo centro de datos. Con la incorporación de las [zonas de disponibilidad de Azure](../availability-zones/az-overview.md), podrá colocar las diferentes réplicas de los conjuntos de cuórum en zonas de disponibilidad diferentes de la misma región. Para eliminar un punto de error único, también se duplica el anillo de control en varias zonas como tres anillos de puerta de enlace. El enrutamiento a un anillo de puerta de enlace específico se controla mediante [Azure Traffic Manager](../traffic-manager/traffic-manager-overview.md) (ATM). Debido a que la configuración de redundancia de zona no crea una redundancia de base de datos adicional, el uso de las zonas de disponibilidad en el nivel de servicio Premium está disponible sin costo adicional. Al seleccionar una base de datos con redundancia de zona, puede hacer que las bases de datos Premium sean resistentes a un conjunto mucho más amplio de errores, incluidas las interrupciones catastróficas de los centros de datos, sin realizar ningún cambio en la lógica de la aplicación. También puede convertir cualquier grupo o base de datos Premium a la configuración de redundancia de zona.

Como el conjunto de cuórum con redundancia de zona tiene réplicas en distintos centros de datos situados a cierta distancia entre ellos, la mayor latencia de red puede aumentar el tiempo de confirmación y, por lo tanto, afectar al rendimiento de algunas cargas de trabajo OLTP. Siempre puede volver a la configuración de zona única; para ello, deshabilite la configuración de redundancia de zona. Este proceso es una operación de tamaño de datos y es similar a la actualización del objetivo de nivel de servicio (SLO) normal. Al final del proceso, la base de datos o el grupo se migra desde un anillo con redundancia de zona a un anillo de zona única, o viceversa.

> [!IMPORTANT]
> Las bases de datos con redundancia de zona y los grupos elásticos solo se admiten en el nivel de servicio Premium. Durante la versión preliminar pública, las copias de seguridad y los registros de auditoría se almacenan en un almacenamiento RA-GRS y, por lo tanto, podrían no estar disponibles automáticamente en el caso de una interrupción que afecte a toda la zona. 

En el diagrama siguiente se ilustra la versión con redundancia de zona de la arquitectura de alta disponibilidad:
 
![arquitectura de alta disponibilidad con redundancia de zona](./media/sql-database-high-availability/high-availability-architecture-zone-redundant.png)

## <a name="conclusion"></a>Conclusión
Azure SQL Database está totalmente integrado en la plataforma Azure y depende en gran medida de Service Fabric para la detección de errores y recuperación, de Azure Storage Blobs para la protección de datos, y de la zonas de disponibilidad para una mayor tolerancia a errores. Al mismo tiempo, Azure SQL Database usa la tecnología Always On de SQL Server para la replicación y la conmutación por error. La combinación de estas tecnologías permite que las aplicaciones obtengan todos los beneficios de un modelo de almacenamiento mixto y puedan admitir los SLA más exigentes. 

## <a name="next-steps"></a>Pasos siguientes

- Obtener más información acerca de las [zonas de disponibilidad de Azure](../availability-zones/az-overview.md).
- Obtener más información sobre [Service Fabric](../service-fabric/service-fabric-overview.md).
- Obtener más información acerca de [Azure Traffic Manager](../traffic-manager/traffic-manager-overview.md). 
