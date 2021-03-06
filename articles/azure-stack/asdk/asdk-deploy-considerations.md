---
title: Requisitos previos para la implementación del Kit de desarrollo de Azure Stack (ASDK) | Microsoft Docs
description: Consulte los requisitos de hardware y del entorno para instalar el Kit de desarrollo de Azure Stack (ASDK).
services: azure-stack
documentationcenter: ''
author: jeffgilb
manager: femila
editor: ''
ms.assetid: ''
ms.service: azure-stack
ms.workload: na
pms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/21/2018
ms.author: jeffgilb
ms.reviewer: misainat
ms.openlocfilehash: f4b55bb3287f67792b3257c3f62256437f5625ca
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/23/2018
---
# <a name="azure-stack-deployment-planning-considerations"></a>Consideraciones de planeación de la implementación de Azure Stack
Antes de implementar el Kit de desarrollo de Azure Stack (ASDK), asegúrese de que el equipo host del kit de desarrollo cumple los requisitos que se describen en este artículo.


## <a name="hardware"></a>Hardware
| Componente | Mínima | Recomendado |
| --- | --- | --- |
| Unidades de disco: sistema operativo |Un disco del sistema operativo con un mínimo de 200 GB disponibles para la partición del sistema (SSD o HDD) |Un disco del sistema operativo con un mínimo de 200 GB disponibles para la partición del sistema (SSD o HDD) |
| Unidades de disco: datos generales del kit de desarrollo<sup>*</sup>  |Cuatro discos. Cada disco proporciona un mínimo de 140 GB de capacidad (SSD o HDD). Se utilizan todos los discos disponibles. |Cuatro discos. Cada disco proporciona un mínimo de 250 GB de capacidad (SSD o HDD). Se utilizan todos los discos disponibles. |
| Proceso: CPU |Socket dual: 12 núcleos físicos (total) |Socket dual: 16 núcleos físicos (total) |
| Proceso: memoria |96 GB de RAM |128 GB de RAM (Este es el mínimo para admitir proveedores de recursos de PaaS.)|
| Proceso: BIOS |Hyper-V habilitado (con compatibilidad para SLAT) |Hyper-V habilitado (con compatibilidad para SLAT) |
| Red: NIC |Certificación de Windows Server 2012 R2 necesaria para NIC; no se necesitan características especializadas |Certificación de Windows Server 2012 R2 necesaria para NIC; no se necesitan características especializadas |
| Certificación del logotipo de hardware |[Certificado para Windows Server 2012 R2](http://windowsservercatalog.com/results.aspx?&chtext=&cstext=&csttext=&chbtext=&bCatID=1333&cpID=0&avc=79&ava=0&avq=0&OR=1&PGS=25&ready=0) |[Certificado para Windows Server 2016](http://windowsservercatalog.com/results.aspx?&chtext=&cstext=&csttext=&chbtext=&bCatID=1333&cpID=0&avc=79&ava=0&avq=0&OR=1&PGS=25&ready=0) |

<sup>*</sup>Si tiene previsto agregar muchos de los [elementos de Marketplace](asdk-marketplace-item.md) desde Azure, necesitará más capacidad de la que se recomienda.

**Configuración de unidad de disco de datos:** todas las unidades de datos deben ser del mismo tipo (todas SAS, SATA o NVMe) y capacidad. Si se usan unidades de disco SAS, las unidades de disco debe estar conectadas a través de una ruta de acceso única (no se proporciona compatibilidad con MPIO ni con rutas de acceso múltiples).

**Opciones de configuración HBA**

* (Opción preferida) HBA simple
* HBA RAID: el adaptador debe configurarse en modo "pass through"
* HBA de RAID: los discos deben configurarse como único disco, RAID-0

**Combinaciones de tipos de medios y bus admitidas**

* SATA HDD
* SAS HDD
* RAID HDD
* RAID SSD (si el tipo de soporte físico es no especificado o desconocido<sup>*</sup>)
* SATA SSD + SATA HDD
* SAS SSD + SAS HDD
* NVMe

<sup>*</sup> Las controladoras de RAID sin la funcionalidad pass-through no reconocen el tipo de soporte físico. Estos controladores marca tanto HDD como SSD como No especificado. En ese caso, se usa una unidad SSD como almacenamiento persistente, en lugar de dispositivos de almacenamiento en caché. Por lo tanto, puede implementar el kit de desarrollo en esas SSD.

**HBA de ejemplo**: LSI 9207 8i, LSI-9300-8i o LSI-9265-8i en modo Paso a través

Están disponibles configuraciones de OEM de ejemplo.

## <a name="operating-system"></a>Sistema operativo
|  | **Requisitos** |
| --- | --- |
| **Versión del SO.** |Windows Server 2012 R2 o posterior. La versión del sistema operativo no es crítica antes de iniciar la implementación, ya que podrá arrancar el equipo host en el disco duro virtual que se incluye en la instalación de Azure Stack. El sistema operativo y todas las revisiones necesarias ya están integradas en la imagen. No use ninguna clave para activar las instancias de Windows Server utilizadas en el kit de desarrollo. |

> [!TIP]
> Después de instalar el sistema operativo, puede usar el [Comprobador de implementación para Azure Stack](https://gallery.technet.microsoft.com/Deployment-Checker-for-50e0f51b) para confirmar que el hardware cumple todos los requisitos.

## <a name="account-requirements"></a>Requisitos de cuenta
Normalmente, se implementa el kit de desarrollo con conectividad a Internet, donde puede conectarse a Microsoft Azure. En este caso, debe configurar una cuenta de Azure Active Directory (Azure AD) para implementar el kit de desarrollo.

Si el entorno no está conectado a Internet o no desea usar Azure AD, puede implementar Azure Stack mediante el uso de los Servicios de federación de Active Directory (AD FS). El kit de desarrollo incluye sus propias instancias de AD FS y Active Directory Domain Services. Si implementa mediante el uso de esta opción, no tiene que configurar cuentas de antemano.

>[!NOTE]
Si implementa mediante la opción de AD FS, debe volver a implementar Azure Stack para cambiar a Azure AD.

### <a name="azure-active-directory-accounts"></a>Cuentas de Azure Active Directory
Para implementar Azure Stack mediante una cuenta de Azure AD, debe preparar una cuenta de Azure AD antes de ejecutar el script de PowerShell de implementación. Esta cuenta se convierte en el administrador global para el inquilino de Azure AD. Se utiliza para aprovisionar y delegar aplicaciones y entidades de servicio para todos los servicios de Azure Stack que interactúan con Azure Active Directory y Graph API. También se utiliza como el propietario de la suscripción de proveedor predeterminada (que puede cambiar más adelante). Puede iniciar sesión en el portal del administrador del sistema de Azure Stack mediante el uso de esta cuenta.

1. Cree una cuenta de Azure AD que sea el administrador de directorios de al menos una instancia de Azure AD. Si ya tiene una, puede usarla. En caso contrario, puede crearla de forma gratuita en [https://azure.microsoft.com/free/](http://azure.microsoft.com/pricing/free/) (en China, visite <http://go.microsoft.com/fwlink/?LinkID=717821>). Si tiene previsto más adelante [registrar Azure Stack en Azure](asdk-register.md), también debe tener una suscripción en la cuenta recién creada.
   
    Guarde dichas credenciales para usarlas como administrador del servicio. Esta cuenta puede configurar y administrar recursos en la nube, cuentas de usuario, planes de inquilinos, cuotas y precios. En el portal, pueden crear nubes de sitios web, nubes privadas de máquinas virtuales, crear planes y administrar suscripciones de usuario.
1. Cree al menos una cuenta de usuario de prueba en Azure AD para que pueda iniciar sesión en el kit de desarrollo como un inquilino.
   
   | **Cuenta de Azure Active Directory** | **¿Admitida?** |
   | --- | --- |
   | Cuenta profesional o educativa con una suscripción de Azure pública válida |Sí |
   | Cuenta de Microsoft con una suscripción de Azure pública válida |Sí |
   | Cuenta profesional o educativa con una suscripción de Azure China válida |Sí |
   | Cuenta profesional o educativa con una suscripción de Azure Gobierno de Estados Unidos válida |Sí |

## <a name="network"></a>Red
### <a name="switch"></a>Switch
Un puerto disponible en un conmutador para la máquina del kit de desarrollo.  

La máquina del kit de desarrollo admite la conexión a un puerto de acceso de conmutador o puerto de enlace. No se requieren características especializadas en el conmutador. Si está usando un puerto de enlace o necesita configurar un identificador de VLAN, tendrá que proporcionar el identificador de VLAN como un parámetro de implementación.

### <a name="subnet"></a>Subred
No conecte el equipo del kit de desarrollo a las subredes siguientes:

* 192.168.200.0/24
* 192.168.100.0/27
* 192.168.101.0/26
* 192.168.102.0/24
* 192.168.103.0/25
* 192.168.104.0/25

Estas subredes están reservadas para las redes internas dentro del entorno del kit de desarrollo.

### <a name="ipv4ipv6"></a>IPv4/IPv6
Se admite solo IPv4. No se pueden crear redes IPv6.

### <a name="dhcp"></a>DHCP
Asegúrese de que hay un servidor DHCP disponible en la red al que se conecta la tarjeta NIC. Si DHCP no está disponible, debe preparar una red IPv4 estática adicional además de la usada por el host. Debe proporcionar esa dirección IP y la puerta de enlace como parámetro de implementación.

### <a name="internet-access"></a>Acceso a Internet
Azure Stack necesita acceso a Internet, ya sea directamente o a través de un proxy transparente. Azure Stack no admite la configuración de un proxy web para habilitar el acceso a Internet. Tanto la dirección IP del host como la nueva dirección IP asignada a MAS-BGPNAT01 (por DHCP o dirección IP estática) deben poder acceder a Internet. Se usan los puertos 80 y 443 en los dominios graph.windows.net y login.microsoftonline.com.


## <a name="next-steps"></a>Pasos siguientes
[Descarga del paquete de implementación de ASDK](asdk-download.md)
