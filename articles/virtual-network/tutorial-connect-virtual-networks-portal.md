---
title: 'Conexión de redes virtuales con emparejamiento de redes virtuales: Azure Portal | Microsoft Docs'
description: Aprenda a conectar redes virtuales con el emparejamiento de redes virtuales.
services: virtual-network
documentationcenter: virtual-network
author: jimdial
manager: jeconnoc
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: ''
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/13/2018
ms.author: jdial
ms.custom: ''
ms.openlocfilehash: 0962a917186277a34abbda17b8fea87bcf4ad1e9
ms.sourcegitcommit: 8aab1aab0135fad24987a311b42a1c25a839e9f3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/16/2018
---
# <a name="connect-virtual-networks-with-virtual-network-peering-using-the-azure-portal"></a>Conexión de redes virtuales con emparejamiento de redes virtuales usando Azure Portal

Puede conectar redes virtuales entre sí con el emparejamiento de redes virtuales. Una vez que las redes virtuales están emparejadas, los recursos de ambas se pueden comunicar entre sí con el mismo ancho de banda y la misma latencia que si estuvieran en la misma red virtual. En este artículo, aprenderá a:

> [!div class="checklist"]
> * Crear dos redes virtuales
> * Conectar dos redes virtuales con el emparejamiento de redes virtuales
> * Implementar una máquina virtual (VM) en cada red virtual
> * Comunicarse entre máquinas virtuales

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="log-in-to-azure"></a>Inicio de sesión en Azure 

Inicie sesión en Azure Portal en https://portal.azure.com.

## <a name="create-virtual-networks"></a>Creación de redes virtuales

1. Seleccione **+ Crear un recurso** en la esquina superior izquierda de Azure Portal.
2. Seleccione **Redes** y **Red virtual**.
3. Escriba o seleccione la siguiente información, acepte los valores predeterminados para el resto de la configuración y luego seleccione **Crear**:

    |Configuración|Valor|
    |---|---|
    |NOMBRE|myVirtualNetwork1|
    |Espacio de direcciones|10.0.0.0/16|
    |La suscripción| Seleccione su suscripción.|
    |Grupos de recursos| Haga clic en **Crear nuevo** y escriba *myResourceGroup*.|
    |Ubicación| Seleccione **Este de EE. UU**.|
    |Nombre de subred|Subnet1|
    |Intervalo de direcciones de subred|10.0.0.0/24|

      ![Crear una red virtual](./media/tutorial-connect-virtual-networks-portal/create-virtual-network.png)

4. Complete de nuevo los pasos del 1 al 3, con los cambios siguientes:

    |Configuración|Valor|
    |---|---|
    |NOMBRE|myVirtualNetwork2|
    |Espacio de direcciones|10.1.0.0/16|
    |Grupos de recursos| Seleccione **Usar existente** y después seleccione **myResourceGroup**.|
    |Intervalo de direcciones de subred|10.1.0.0/24|

## <a name="peer-virtual-networks"></a>Emparejamiento de redes virtuales

1. En el cuadro Búsqueda que se encuentra en la parte superior de Azure Portal, escriba *MyVirtualNetwork1*. Cuando la opción **myVirtualNetwork1** aparezca en los resultados de la búsqueda, selecciónela.
2. En **CONFIGURACIÓN**, seleccione **Emparejamientos** y, luego, seleccione **+Agregar**, como se muestra en la imagen siguiente:

    ![Creación de emparejamiento](./media/tutorial-connect-virtual-networks-portal/create-peering.png)

3. Escriba o seleccione la siguiente información, acepte los valores predeterminados para el resto de la configuración y luego seleccione **Aceptar**.

    |Configuración|Valor|
    |---|---|
    |NOMBRE|myVirtualNetwork1-myVirtualNetwork2|
    |La suscripción| Seleccione su suscripción.|
    |Red virtual|myVirtualNetwork2: para seleccionar la red virtual *myVirtualNetwork2*, seleccione **Red virtual** y, después, **myVirtualNetwork2**.|

    ![Configuración de emparejamiento](./media/tutorial-connect-virtual-networks-portal/peering-settings.png)

    El **ESTADO DE EMPAREJAMIENTO** es *Iniciado*, tal y como se muestra en la siguiente imagen:

    ![Estado de emparejamiento](./media/tutorial-connect-virtual-networks-portal/peering-status.png)

    Si no ve el estado, actualice el explorador.

4. En el cuadro **Búsqueda** que se encuentra en la parte superior de Azure Portal, escriba *MyVirtualNetwork2*. Cuando la opción **myVirtualNetwork2** aparezca en los resultados de la búsqueda, selecciónela.
5. Complete de nuevo los pasos del 2 al 3, con los cambios siguientes y luego seleccione **Aceptar**:

    |Configuración|Valor|
    |---|---|
    |NOMBRE|myVirtualNetwork2-myVirtualNetwork1|
    |Red virtual|myVirtualNetwork1|

    El **ESTADO DE EMPAREJAMIENTO** es *Conectado*. Azure también cambia el estado de emparejamiento del emparejamiento *myVirtualNetwork2 myVirtualNetwork1* de *Iniciado* a *Conectado.* El emparejamiento de red virtual no se habrá establecido correctamente hasta que el estado de emparejamiento de ambas redes virtuales no sea *Conectado.* 

## <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Cree una máquina virtual en cada red virtual para que puedan comunicarse entre ellas en un paso posterior.

### <a name="create-the-first-vm"></a>Creación de la primera máquina virtual

1. Seleccione **+ Crear un recurso** en la esquina superior izquierda de Azure Portal.
2. Seleccione **Compute** y, después, seleccione **Windows Server 2016 Datacenter**. Puede seleccionar otro sistema operativo, pero en los pasos restantes se supone que seleccionó **Windows Server 2016 Datacenter**. 
3. Escriba o seleccione la siguiente información para **Aspectos básicos**, acepte los valores predeterminados para el resto de la configuración y luego seleccione **Crear**:

    |Configuración|Valor|
    |---|---|
    |NOMBRE|myVm1|
    |Nombre de usuario| Escriba un nombre de usuario de su elección.|
    |Password| Escriba una contraseña de su elección. La contraseña debe tener al menos 12 caracteres de largo y cumplir con los [requisitos de complejidad definidos](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm).|
    |Grupos de recursos| Seleccione **Usar existente** y después seleccione **myResourceGroup**.|
    |Ubicación| Seleccione **Este de EE. UU**.|
4. Seleccione un tamaño de máquina virtual en **Elegir un tamaño**.
5. Seleccione los valores siguientes en **Configuración** y, después, seleccione **Aceptar**:
    |Configuración|Valor|
    |---|---|
    |Red virtual| myVirtualNetwork1: si no está ya seleccionado, seleccione **Red virtual** y, luego, **myVirtualNetwork1** en **Elegir red virtual**.|
    |Subred| Subred1: si aún no está seleccionado, seleccione **Subred** y después seleccione **Subred1** en **Elegir subred**.|
    
    ![Configuración de máquina virtual](./media/tutorial-connect-virtual-networks-portal/virtual-machine-settings.png)
 
6. En **Crear** de la página **Resumen**, seleccione **Crear** para iniciar la implementación de la máquina virtual.

### <a name="create-the-second-vm"></a>Creación de la segunda máquina virtual

Complete de nuevo los pasos del 1 al 6, con los cambios siguientes:

|Configuración|Valor|
|---|---|
|NOMBRE | myVm2|
|Red virtual | myVirtualNetwork2|

Las máquinas virtuales tardan unos minutos en crearse. No siga con los pasos restantes hasta que se creen ambas máquinas virtuales.

## <a name="communicate-between-vms"></a>Comunicarse entre máquinas virtuales

1. En el cuadro *Búsqueda* que se encuentra en la parte superior del portal, escriba *myVm1*. Seleccione **MyVm1** cuando aparezca en los resultados de búsqueda.
2. Cree una conexión a Escritorio remoto a la máquina virtual *myVm1* seleccionando **Conectar**, tal como se indica en la imagen siguiente:

    ![Conexión a la máquina virtual](./media/tutorial-connect-virtual-networks-portal/connect-to-virtual-machine.png)  

3. Para conectarse a la máquina virtual, abra el archivo RDP descargado. Cuando se le pida, seleccione **Conectar**.
4. Escriba el nombre de usuario y la contraseña que especificó al crear la máquina virtual (puede que deba seleccionar **Más opciones** y luego **Usar una cuenta diferente**, para especificar las credenciales que escribió cuando creó la máquina virtual). A continuación, seleccione **Aceptar**.
5. Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Seleccione **Sí** para continuar con la conexión.
6. En un paso posterior, se usa ping para comunicarse con la máquina virtual *myVm2* desde la máquina virtual *myVm1*. Ping usa el Protocolo de mensajes de control de Internet (ICMP) que, de forma predeterminada, se deniega a través del Firewall de Windows. En la máquina virtual *myVm1*, habilite el Protocolo de mensajes de control de Internet (ICMP) a través del Firewall de Windows para que pueda hacer ping en esta máquina virtual desde *myVm2* en un paso posterior, mediante PowerShell:

    ```powershell
    New-NetFirewallRule –DisplayName “Allow ICMPv4-In” –Protocol ICMPv4
    ```
    
    Aunque en este artículo se usa ping para comunicarse entre máquinas virtuales, no se recomienda permitir ICMP mediante el Firewall de Windows para las implementaciones de producción.

7. Para conectar con la máquina virtual *myVm2*, escriba el siguiente comando desde el símbolo del sistema en la máquina virtual *myVm1*:

    ```
    mstsc /v:10.1.0.4
    ```
    
8. Puesto que ha habilitado ping en *myVm1*, ahora puede hacer ping a él por dirección IP:

    ```
    ping 10.0.0.4
    ```
    
9. Desconecte las sesiones RDP para ambos *myVm1* y *myVm2*.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no sea necesario, elimine el grupo de recursos y todos los recursos que contiene: 

1. Escriba *myResourceGroup* en el cuadro **Buscar** que se encuentra en la parte superior del portal. Seleccione **myResourceGroup** cuando lo vea en los resultados de la búsqueda.
2. Seleccione **Eliminar grupo de recursos**.
3. Escriba *myResourceGroup* para **ESCRIBA EL NOMBRE DEL GRUPO DE RECURSOS:** y seleccione **Eliminar**.

**<a name="register"></a>Registro para la versión preliminar del emparejamiento de red virtual global**

El emparejamiento de redes virtuales en las mismas regiones tiene disponibilidad general. El emparejamiento de redes virtuales de diferentes regiones actualmente se encuentra en versión preliminar. Consulte las [actualizaciones de redes virtuales](https://azure.microsoft.com/updates/?product=virtual-network) para ver las regiones disponibles. Para emparejar redes virtuales en diferentes regiones, primero tiene que registrarse para la versión preliminar. No se puede registrar con el portal, pero puede registrar usando [PowerShell](tutorial-connect-virtual-networks-powershell.md#register) o [CLI de Azure](tutorial-connect-virtual-networks-cli.md#register). Si intenta emparejar redes virtuales en diferentes regiones antes de registrarse para la funcionalidad, el emparejamiento produce un error.

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha aprendido a conectar dos redes, de la misma ubicación de Azure, con el emparejamiento de redes virtuales. También puede emparejar redes virtuales de [diferentes regiones](#register), en [diferentes suscripciones de Azure](create-peering-different-subscriptions.md#portal) y puede crear [diseños de red de concentrador y radio](/azure/architecture/reference-architectures/hybrid-networking/hub-spoke?toc=%2fazure%2fvirtual-network%2ftoc.json#vnet-peering) con emparejamiento. Antes de emparejar redes virtuales de producción, se recomienda familiarizarse en detalle con la [introducción al emparejamiento](virtual-network-peering-overview.md), la [administración del emparejamiento](virtual-network-manage-peering.md) y los [límites de red virtual ](../azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits). 

Continúe para conectar su equipo a una red virtual mediante VPN e interactuar con recursos en una red virtual, o en redes virtuales emparejadas.

> [!div class="nextstepaction"]
> [Conexión de un equipo a una red virtual](../vpn-gateway/vpn-gateway-howto-point-to-site-resource-manager-portal.md?toc=%2fazure%2fvirtual-network%2ftoc.json)
