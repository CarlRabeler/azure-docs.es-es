---
title: 'Ejemplo de script de CLI de Azure: crear una red para aplicaciones de niveles múltiples| Microsoft Docs'
description: 'Ejemplo de script de CLI de Azure: crear una red virtual para aplicaciones de niveles múltiples.'
services: virtual-network
documentationcenter: virtual-network
author: jimdial
manager: jeconnoc
editor: ''
tags: ''
ms.assetid: ''
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: article
ms.tgt_pltfrm: ''
ms.workload: infrastructure
ms.date: 03/20/2018
ms.author: jdial
ms.openlocfilehash: fdccf145b9cd9d64076cd6c181420adbc992c6a9
ms.sourcegitcommit: 48ab1b6526ce290316b9da4d18de00c77526a541
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/23/2018
---
# <a name="create-a-network-for-multi-tier-applications"></a>Creación de una red para aplicaciones de niveles múltiples

Este ejemplo de script crea una red virtual con subredes de front-end y back-end. El tráfico a la subred de front-end está limitado a HTTP y SSH, mientras que el tráfico a la subred de back-end está limitado a MySQL, al puerto 3306. Después de ejecutar el script, tendrá dos máquinas virtuales, una en cada subred, en las que puede implementar el servidor web y el software MySQL.

Puede ejecutar el script desde Azure [Cloud Shell](https://shell.azure.com/bash) o desde una instalación de CLI de Azure local. Si usa la CLI localmente, este script requiere que ejecute la versión 2.0.28 o posterior. Ejecute `az --version` para ver cuál es la versión instalada. Si necesita instalarla o actualizarla, consulte [Instalación de la CLI de Azure 2.0](/cli/azure/install-azure-cli). Si ejecuta la CLI localmente, también debe ejecutar `az login` para crear una conexión con Azure.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]


## <a name="sample-script"></a>Script de ejemplo


[!code-azurecli-interactive[main](../../../cli_scripts/virtual-network/virtual-network-multi-tier-application/virtual-network-multi-tier-application.sh  "Virtual network for multi-tier application")]

## <a name="clean-up-deployment"></a>Limpieza de la implementación 

Ejecute el siguiente comando para quitar el grupo de recursos, la VM y todos los recursos relacionados:

```azurecli
az group delete --name MyResourceGroup --yes
```

## <a name="script-explanation"></a>Explicación del script

Este script usa los siguientes comandos para crear un grupo de recursos, una red virtual y grupos de seguridad de red. Cada comando de la tabla siguiente crea un vínculo a documentación específica del comando:

| Get-Help | Notas |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Crea un grupo de recursos en el que se almacenan todos los recursos. |
| [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create) | Crea una subred de front-end y una red virtual de Azure. |
| [az network subnet create](/cli/azure/network/vnet/subnet#az_network_vnet_subnet_create) | Crea una subred de back-end. |
| [az network public-ip create](/cli/azure/network/public-ip#az_network_public_ip_create) | Crea una dirección IP pública para acceder a la VM desde Internet. |
| [az network nic create](/cli/azure/network/nic#az_network_nic_create) | Crea interfaces de red virtual y las asocia a subredes de front-end y back-end de la red virtual. |
| [az network nsg create](/cli/azure/network/nsg#az_network_nsg_create) | Crea grupos de seguridad de red (NSG) que están asociados a las subredes de front-end y back-end. |
| [az network nsg rule create](/cli/azure/network/nsg/rule#az_network_nsg_rule_create) |Crea reglas NSG que permiten o bloquean puertos específicos para subredes concretas. |
| [az vm create](/cli/azure/vm#az_vm_create) | Crea máquinas virtuales se crea y asocia una NIC a cada VM. Este comando también especifica la imagen de máquina virtual que se usará y las credenciales administrativas. |
| [az group delete](/cli/azure/group#az_group_delete) | Elimina un grupo de recursos y todos los recursos que contiene. |

## <a name="next-steps"></a>Pasos siguientes

Para más información sobre la CLI de Azure, consulte la [documentación de la CLI de Azure](/cli/azure).

Puede encontrar más ejemplos de script de CLI de red virtual en [ejemplos de CLI de red virtual](../cli-samples.md).
