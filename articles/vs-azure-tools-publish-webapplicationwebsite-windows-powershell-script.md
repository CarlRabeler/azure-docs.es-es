---
title: Publish-WebApplicationWebSite (script de Windows PowerShell) | Microsoft Docs
description: Aprenda a publicar un proyecto web en un sitio web de Azure. Este script crea los recursos necesarios en su suscripción de Azure si no existen.
services: visual-studio-online
documentationcenter: na
author: ghogen
manager: douge
editor: ''
ms.assetid: 63cfaa2d-f04d-40dc-8677-345385c278d5
ms.service: multiple
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: multiple
ms.date: 11/11/2016
ms.author: ghogen
ms.openlocfilehash: aaa1f679b0368b0ca93305fe867a63f3971a788c
ms.sourcegitcommit: 34e0b4a7427f9d2a74164a18c3063c8be967b194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2018
---
# <a name="publish-webapplicationwebsite-windows-powershell-script"></a>Publicación de WebApplicationWebSite (script de Windows PowerShell)
## <a name="syntax"></a>Sintaxis
Publica un proyecto web en un sitio web de Azure. El script crea los recursos necesarios en su suscripción de Azure si no existen.

    Publish-WebApplicationWebSite
    –Configuration <configuration>
    -SubscriptionName <subscriptionName>
    -WebDeployPackage <packageName>
    -DatabaseServerPassword @{Name = "name"; Password = "password"}
    -SendHostMessagesToOutput
    -Verbose


## <a name="configuration"></a>Configuración
La ruta de acceso al archivo de configuración JSON que describe los detalles de la implementación.

| . | Valor predeterminado |
| --- | --- |
| Alias |None |
| ¿Necesario? |true |
| Posición |con nombre |
| Valor predeterminado |None |
| ¿Aceptar la entrada de la canalización? |false |
| ¿Aceptar caracteres comodín? |false |

## <a name="subscriptionname"></a>SubscriptionName
Nombre de la suscripción de Azure en la que desea crear el sitio web.

| . | Valor predeterminado |
| --- | --- |
| Alias |None |
| ¿Necesario? |false |
| Posición |con nombre |
| Valor predeterminado |None |
| ¿Aceptar la entrada de la canalización? |false |
| ¿Aceptar caracteres comodín? |false |

## <a name="webdeploypackage"></a>WebDeployPackage
La ruta de acceso al paquete de implementación web para publicar en el sitio web. Puede crear este paquete mediante el Asistente de publicación web en Visual Studio. Para obtener más información, consulte [Introducción a Azure Cloud Services y ASP.NET](http://go.microsoft.com/fwlink/p/?LinkID=623089).

| . | Valor predeterminado |
| --- | --- |
| Alias |None |
| ¿Necesario? |false |
| Posición |con nombre |
| Valor predeterminado |None |
| ¿Aceptar la entrada de la canalización? |false |
| ¿Aceptar caracteres comodín? |false |

## <a name="databaseserverpassword"></a>DatabaseServerPassword
El nombre de usuario y la contraseña de la base de datos SQL en Azure.

| . | Valor predeterminado |
| --- | --- |
| Alias |None |
| ¿Necesario? |false |
| Posición |con nombre |
| Valor predeterminado |None |
| ¿Aceptar la entrada de la canalización? |false |
| ¿Aceptar caracteres comodín? |false |

## <a name="sendhostmessagestooutput"></a>SendHostMessagesToOutput
Si es true, imprimir mensajes del script a la secuencia de salida.

| . | Valor predeterminado |
| --- | --- |
| Alias |None |
| ¿Necesario? |false |
| Posición |con nombre |
| Valor predeterminado |false |
| ¿Aceptar la entrada de la canalización? |false |
| ¿Aceptar caracteres comodín? |false |

## <a name="remarks"></a>Comentarios
Para obtener una explicación completa de cómo usar el script para crear entornos de desarrollo y pruebas, consulte [Usar scripts de Windows PowerShell para la publicación en entornos de desarrollo y pruebas](vs-azure-tools-publishing-using-powershell-scripts.md).

El archivo de configuración JSON especifica los detalles de lo que va a implementarse. Incluye la información que especificó cuando creó el proyecto, como el nombre y el nombre de usuario para el sitio web. También incluye la base de datos que se va a aprovisionar, si la hubiera. El código siguiente muestra un archivo de configuración de JSON de ejemplo:

    {
        "environmentSettings": {
            "webSite": {
                "name": "WebApplication10554",
                "location": "West US"
            },
            "databases": [
                {
                    "connectionStringName": "DefaultConnection",
                    "databaseName": "WebApplication10554_db",
                    "serverName": "iss00brc88",
                    "user": "sqluser2",
                    "password": "",
                    "edition": "",
                    "size": "",
                    "collation": "",
                    "location": "West US"
                }
            ]
        }
    }

Puede editar el archivo de configuración de JSON para cambiar lo que se implementa. Una sección webSite es obligatoria pero la sección database es opcional.

## <a name="next-steps"></a>Pasos siguientes
Para obtener más información, consulte [Publish-WebApplicationVM (script de Windows PowerShell)](vs-azure-tools-publish-webapplicationvm.md)

