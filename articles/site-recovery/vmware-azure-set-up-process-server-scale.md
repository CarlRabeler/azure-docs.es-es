---
title: Configuración de un servidor de procesos en Azure para conmutación por recuperación de máquinas virtuales y servidores físicos de VMware con Azure Site Recovery | Microsoft Docs
description: En este artículo se describe cómo configurar un servidor de procesos en Azure para la conmutación por recuperación de máquinas virtuales de Azure a VMware.
services: site-recovery
author: AnoopVasudavan
manager: gauravd
ms.service: site-recovery
ms.topic: article
ms.date: 03/05/2018
ms.author: anoopkv
ms.openlocfilehash: 9d9270d8c6d2ffc5e42dfc6f94818fdace89bfb5
ms.sourcegitcommit: 168426c3545eae6287febecc8804b1035171c048
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/08/2018
---
# <a name="set-up-a-process-server-in-azure-for-failback"></a>Configuración de un servidor de procesos en Azure para la conmutación por recuperación

Después de conmutar por error máquinas virtuales o servidores físicos de VMware a Azure mediante [Site Recovery](site-recovery-overview.md), puede conmutarlos por recuperación al sitio local cuando vuelva a estar en funcionamiento. Para la conmutación por recuperación, necesita configurar un servidor de procesos temporal en Azure para controlar la replicación desde Azure en un entorno local. Esta máquina virtual se puede eliminar cuando finalice la conmutación por recuperación.

## <a name="before-you-start"></a>Antes de comenzar

Obtenga más información sobre los procesos de [reprotección](vmware-azure-reprotect.md) y [conmutación por recuperación](vmware-azure-failback.md).

[!INCLUDE [site-recovery-vmware-process-server-prerequ](../../includes/site-recovery-vmware-azure-process-server-prereq.md)]

## <a name="deploy-a-process-server-in-azure"></a>Implementación de un servidor de procesos en Azure

1. En el almacén > **Infraestructura de Site Recovery**> **Administrar** > **Servidores de configuración**, seleccione el servidor de configuración.
2. En la página del servidor, haga clic en **+ Servidor de procesos**.
3. En la página **Agregar servidores de procesos**, seleccione implementar el servidor de procesos en Azure.
4. Especifique la configuración de Azure, incluida la suscripción utilizada para la conmutación por error, un grupo de recursos, la región de Azure usada para la conmutación por error y la red virtual en la que se encuentran las máquinas virtuales de Azure. Si ha usado varias redes de Azure, necesitará un servidor de procesos en cada una de ellas.
5. En **Nombre del servidor**, **Nombre de usuario** y **Contraseña**, especifique un nombre para el servidor de procesos y las credenciales a las que se asignarán permisos de administrador en el servidor.
6. Especifique una cuenta de almacenamiento que se usará para los discos de máquina virtual del servidor, la subred en la que se ubicará la máquina virtual del servidor de procesos y la dirección IP del servidor que se asignará cuando se inicie la máquina virtual.
7. Haga clic en el botón **Aceptar** para iniciar la implementación de la máquina virtual del servidor de procesos.

>

## <a name="registering-the-process-server-running-in-azure-to-a-configuration-server-running-on-premises"></a>Registro del servidor de procesos (que se ejecuta en Azure) en un servidor de configuración (que se ejecuta de forma local)

Cuando la máquina virtual del servidor de procesos esté en funcionamiento, deberá registrarla con el servidor de configuración local de la siguiente manera:

[!INCLUDE [site-recovery-vmware-register-process-server](../../includes/site-recovery-vmware-register-process-server.md)]


