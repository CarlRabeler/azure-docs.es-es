---
title: 'Diseño de la primera base de datos de Azure SQL Database: C# | Microsoft Docs'
description: Aprenda a diseñar la primera base de datos de SQL Azure Dabatase y a conectarse a ella con un programa de C# mediante ADO.NET.
services: sql-database
author: MightyPen
manager: craigg-msft
ms.reviewer: CarlRabeler
ms.service: sql-database
ms.custom: develop databases, mvc, devcenter
ms.topic: tutorial
ms.date: 03/15/2018
ms.openlocfilehash: 3b6f260983e3c826bf558f0fe6d1a0fa6ae6b3af
ms.sourcegitcommit: a36a1ae91968de3fd68ff2f0c1697effbb210ba8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/17/2018
---
# <a name="design-an-azure-sql-database-and-connect-with-cx23-and-adonet"></a>Diseño de una base de datos de SQL Azure Database y conexión con C&#x23; y ADO.NET

Azure SQL Database es una base de datos como servicio (DBaaS) relacional en Microsoft Cloud (Azure). En este tutorial se aprenderá a usar Azure Portal y ADO.NET con Visual Studio para: 

> [!div class="checklist"]
> * Crear una base de datos en Azure Portal
> * Establecer una regla de firewall de nivel de servidor en Azure Portal
> * Conectarse a la base de datos con ADO.NET y Visual Studio
> * Crear tablas con ADO.NET
> * Insertar, actualizar y eliminar datos con ADO.NET 
> * Consultar datos con ADO.NET

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/) antes de empezar.

## <a name="prerequisites"></a>requisitos previos

Una instalación de [Visual Studio Community 2017, Visual Studio Professional 2017 o Visual Studio Enterprise 2017](https://www.visualstudio.com/downloads/).

<!-- The following included .md, sql-database-tutorial-portal-create-firewall-connection-1.md, is long.
And it starts with a ## H2.
-->

[!INCLUDE [sql-database-tutorial-portal-create-firewall-connection-1](../../includes/sql-database-tutorial-portal-create-firewall-connection-1.md)]


<!-- The following included .md, sql-database-csharp-adonet-create-query-2.md, is long.
And it starts with a ## H2.
-->

[!INCLUDE [sql-database-csharp-adonet-create-query-2](../../includes/sql-database-csharp-adonet-create-query-2.md)]


## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha aprendido las tareas básicas de una base de datos, como crear una base de datos y tablas, cargar y consultar datos, y restaurar la base de datos a un momento anterior en el tiempo. Ha aprendido a:
> [!div class="checklist"]
> * Creación de una base de datos
> * Configurar una regla de firewall
> * Conectarse a la base de datos con [Visual Studio y C#](sql-database-connect-query-dotnet-visual-studio.md)
> * Cree las tablas.
> * Insertar, actualizar y eliminar datos
> * Datos de consulta

Vaya al siguiente tutorial para más información sobre la migración de los datos.

> [!div class="nextstepaction"]
>[Migración de una base de datos SQL Server a Azure SQL Database](sql-database-migrate-your-sql-server-database.md)

