---
title: Administración de roles en servicios en la nube de Azure con Visual Studio | Microsoft Docs
description: Obtenga información sobre cómo agregar y quitar roles en los servicios en la nube de Azure con Visual Studio.
services: visual-studio-online
documentationcenter: na
author: ghogen
manager: douge
editor: ''
ms.assetid: 5ec9ae2e-8579-4e5d-999e-8ae05b629bd1
ms.service: multiple
ms.devlang: dotnet
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: multiple
ms.date: 03/21/2017
ms.author: ghogen
ms.openlocfilehash: fb49eaa69f2265f6b312ecbdab1435e7c50236b8
ms.sourcegitcommit: 34e0b4a7427f9d2a74164a18c3063c8be967b194
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2018
---
# <a name="managing-roles-in-azure-cloud-services-with-visual-studio"></a>Administración de roles en servicios en la nube de Azure con Visual Studio
Una vez creado el servicio en la nube de Azure, puede agregarle nuevos roles o quitarle roles existentes. También puede importar un proyecto existente y convertirlo en un rol. Por ejemplo, puede importar una aplicación web ASP.NET y designarla como rol web.

## <a name="adding-a-role-to-an-azure-cloud-service"></a>Incorporación de un rol a un servicio en la nube de Azure
Los pasos siguientes le explican cómo agregar un rol web o un rol de trabajo a un proyecto de servicio en la nube de Azure en Visual Studio.

1. Cree o abra un proyecto de servicio en la nube de Azure en Visual Studio.

1. En el **Explorador de soluciones**, expanda el nodo del proyecto

1. Haga clic con el botón derecho en el nodo **Roles** para ver el menú contextual. En el menú contextual, seleccione **Agregar** y, luego, seleccione un rol web o un rol de trabajo existente en la solución actual o cree un proyecto de rol web o de trabajo. También puede seleccionar un proyecto adecuado, por ejemplo, un proyecto de aplicación web ASP.NET, y asociarlo a un proyecto de rol.

    ![Opciones de menú para agregar un rol a un proyecto de servicio en la nube de Azure](media/vs-azure-tools-cloud-service-project-managing-roles/add-role.png)

## <a name="removing-a-role-from-an-azure-cloud-service"></a>Eliminación de un rol de un servicio en la nube de Azure
Los pasos siguientes le explican cómo quitar un rol web o un rol de trabajo de un proyecto de servicio en la nube de Azure en Visual Studio.

1. Cree o abra un proyecto de servicio en la nube de Azure en Visual Studio.

1. En el **Explorador de soluciones**, expanda el nodo del proyecto

1. Expanda el nodo **Roles**.

1. Haga clic con el botón derecho en el nodo que desea quitar y, en el menú contextual, seleccione **Quitar**. 

    ![Opciones de menú para agregar un rol a un servicio en la nube de Azure](media/vs-azure-tools-cloud-service-project-managing-roles/remove-role.png)

## <a name="readding-a-role-to-an-azure-cloud-service-project"></a>Nueva incorporación de un rol a un proyecto de servicio en la nube de Azure
Si quita un rol del proyecto de servicio en la nube pero posteriormente decide volver a agregarlo al proyecto, solo se agregarán la declaración del rol y los atributos básicos como, por ejemplo, los extremos y la información de diagnóstico. No se agrega ningún recurso o referencia adicional al archivo `ServiceDefinition.csdef` ni al archivo `ServiceConfiguration.cscfg`. Si quiere agregar esta información, tiene que volver a agregarla manualmente en estos archivos.

Por ejemplo, es posible que quite un rol de servicio web y más adelante decida agregarlo de nuevo a la solución. Si lo hace, se produce un error. Para evitar este error, tiene que agregar de nuevo el elemento `<LocalResources>` que se muestra en el siguiente código XML al archivo `ServiceDefinition.csdef`. Use el nombre del rol de servicio web que volvió a agregar al proyecto como parte del nombre del atributo para el elemento **<LocalStorage>** . En este ejemplo el nombre del rol de servicio web es **WCFServiceWebRole1**.

    <WebRole name="WCFServiceWebRole1">
        <Sites>
          <Site name="Web">
            <Bindings>
              <Binding name="Endpoint1" endpointName="Endpoint1" />
            </Bindings>
          </Site>
        </Sites>
        <Endpoints>
          <InputEndpoint name="Endpoint1" protocol="http" port="80" />
        </Endpoints>
        <Imports>
          <Import moduleName="Diagnostics" />
        </Imports>
       <LocalResources>
          <LocalStorage name="WCFServiceWebRole1.svclog" sizeInMB="1000" cleanOnRoleRecycle="false" />
       </LocalResources>
    </WebRole>

## <a name="next-steps"></a>Pasos siguientes
- [Configuración de los roles para un servicio en la nube de Azure con Visual Studio](vs-azure-tools-configure-roles-for-cloud-service.md)
