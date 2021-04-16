---
title: Uso de una identidad administrada para conectar Azure SQL a una aplicación de Azure Spring Cloud
description: Configuración de una identidad administrada para conectar Azure SQL a una aplicación de Azure Spring Cloud.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 03/25/2021
ms.custom: devx-track-java
ms.openlocfilehash: ed729dde51316b9a67f396e3f7de3d7d9f6d4568
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107378795"
---
# <a name="use-a-managed-identity-to-connect-azure-sql-database-to-an-azure-spring-cloud-app"></a>Uso de una identidad administrada para conectar Azure SQL Database a una aplicación de Azure Spring Cloud

**Este artículo se aplica a:** ✔️ Java

En este artículo se describe cómo crear una identidad administrada para una aplicación de Azure Spring Cloud y cómo usarla para acceder a Azure SQL Database.

[Azure SQL Database](https://azure.microsoft.com/services/sql-database/) es servicio de base de datos relacional inteligente y totalmente administrado creado para la nube. Siempre está actualizado, con características basadas en inteligencia artificial y automatizadas que optimizan el rendimiento y la durabilidad. Las opciones de almacenamiento de proceso e hiperescala sin servidor escalan automáticamente los recursos a petición, por lo que puede centrarse en la creación de nuevas aplicaciones sin preocuparse por el tamaño de almacenamiento o la administración de recursos.

## <a name="prerequisites"></a>Requisitos previos
En este ejemplo se utilizan los siguientes recursos.
* Siga el [tutorial de JPA de datos de Spring](https://docs.microsoft.com/azure/developer/java/spring-framework/configure-spring-data-jpa-with-azure-sql-server) para aprovisionar un Azure SQL Database y conseguir que funcione con una aplicación de Java de forma local
* Siga el [tutorial de identidad administrada asignada por el sistema de Azure Spring Cloud](https://docs.microsoft.com/azure/spring-cloud/spring-cloud-howto-enable-system-assigned-managed-identity) para aprovisionar una aplicación de Azure Spring Cloud con una instancia administrada habilitada

## <a name="grant-permission-to-the-managed-identity"></a>Conceder permiso a la identidad administrada
Conéctese al servidor SQL y ejecute la siguiente consulta SQL:

```sql
CREATE USER [<MIName>] FROM EXTERNAL PROVIDER;
ALTER ROLE db_datareader ADD MEMBER [<MIName>];
ALTER ROLE db_datawriter ADD MEMBER [<MIName>];
ALTER ROLE db_ddladmin ADD MEMBER [<MIName>];
GO
```

Este <MIName> sigue la regla: `<service instance name>/apps/<app name>`, por ejemplo, myspringcloud/apps/sqldemo. También puede consultar el MIName con CLI de Azure:

```azurecli
az ad sp show --id <identity object ID> --query displayName
```

## <a name="configure-your-java-app-to-use-managed-identity"></a>Configuración de la aplicación de Java para usar la identidad administrada
Abra el archivo `src/main/resources/application.properties` y agregue `Authentication=ActiveDirectoryMSI;` en la parte final de la siguiente línea. Asegúrese de usar el valor correcto para la variable $AZ_DATABASE_NAME.

```properties
spring.datasource.url=jdbc:sqlserver://$AZ_DATABASE_NAME.database.windows.net:1433;database=demo;encrypt=true;trustServerCertificate=false;hostNameInCertificate=*.database.windows.net;loginTimeout=30;Authentication=ActiveDirectoryMSI;
```

## <a name="build-and-deploy-the-app-to-azure-spring-cloud"></a>Compile e implemente la aplicación en Azure Spring Cloud
Vuelva a compilar la aplicación e implementarla en la aplicación Azure Spring Cloud aprovisionada en el segundo punto de viñeta de requisitos previos. Ahora tiene una aplicación de Spring Boot, autenticada por una identidad administrada que usa JPA para almacenar y recuperar datos de un Azure SQL Database en la nube de Azure Spring Cloud.

## <a name="next-steps"></a>Pasos siguientes

* [Procedimiento para obtener acceso al blob de almacenamiento con identidad administrada en Azure Spring Cloud](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/tree/master/managed-identity-storage-blob)
* [Habilitación de la identidad administrada asignada por el sistema para una aplicación de Azure Spring Cloud](./spring-cloud-howto-enable-system-assigned-managed-identity.md)
* [Más información sobre las identidades administradas para recursos de Azure](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/active-directory/managed-identities-azure-resources/overview.md)
* [Autenticación de Azure Spring Cloud con Key Vault en Acciones de GitHub](./spring-cloud-github-actions-key-vault.md)