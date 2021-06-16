---
title: 'Inicio rápido: Conexión a Azure SQL Database con Java con Acciones de GitHub'
description: Uso de Azure SQL desde un flujo de trabajo de Acciones de GitHub
author: juliakm
services: sql-database
ms.service: sql-database
ms.subservice: connect
ms.topic: quickstart
ms.author: jukullam
ms.date: 05/05/2021
ms.custom: github-actions-azure
ms.reviewer: mathoma
ms.openlocfilehash: 5a46b5c5f8d8d4d39f93f6a91210e70fa41ea20d
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110694642"
---
# <a name="use-github-actions-to-connect-to-azure-sql-database"></a>Uso de Acciones de GitHub para conectarse a Azure SQL Database

Comience a trabajar con [Acciones de GitHub](https://docs.github.com/en/actions) mediante un flujo de trabajo para implementar actualizaciones de base de datos en [Azure SQL Database](../azure-sql-iaas-vs-paas-what-is-overview.md). 

## <a name="prerequisites"></a>Requisitos previos

Necesitará: 
- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Un repositorio de GitHub con un paquete dacpac (`Database.dacpac`). Si no tiene una cuenta de GitHub, [regístrese para obtener una gratis](https://github.com/join).  
- Una base de datos de Azure SQL.
    - [Inicio rápido: Creación de una base de datos única de Azure SQL Database](single-database-create-quickstart.md)
    - [Cómo crear un paquete dacpac a partir de una base de datos de SQL Server existente](/sql/relational-databases/data-tier-applications/export-a-data-tier-application)

## <a name="workflow-file-overview"></a>Información general sobre el archivo de flujo de trabajo

Un flujo de trabajo de Acciones de GitHub se define mediante un archivo YAML (.yml) en la ruta de acceso `/.github/workflows/` de su repositorio. En esta definición se incluyen los diversos pasos y parámetros que componen el flujo de trabajo.

El archivo tiene dos secciones:

|Sección  |Tareas  |
|---------|---------|
|**Autenticación** | 1. Defina una entidad de servicio. <br /> 2. Cree un secreto de GitHub. |
|**Implementar** | 1. Implemente la base de datos. |

## <a name="generate-deployment-credentials"></a>Genere las credenciales de implementación.

Puede crear una [entidad de servicio](../../active-directory/develop/app-objects-and-service-principals.md) mediante el comando [az ad sp create-for-rbac](/cli/azure/ad/sp#az_ad_sp_create_for_rbac) de la [CLI de Azure](/cli/azure/). Puede ejecutar este comando mediante [Azure Cloud Shell](https://shell.azure.com/) en Azure Portal o haciendo clic en el botón **Probar**.

Reemplace los marcadores de posición `server-name` por el nombre del servidor SQL hospedado en Azure. Reemplace `subscription-id` y `resource-group` por el identificador de suscripción y el grupo de recursos conectado a su servidor SQL.  

```azurecli-interactive
   az ad sp create-for-rbac --name {server-name} --role contributor \
                            --scopes /subscriptions/{subscription-id}/resourceGroups/{resource-group} \
                            --sdk-auth
```

La salida es un objeto JSON con las credenciales de asignación de roles que proporcionan acceso a la base de datos, de forma parecida a como se muestra en este ejemplo. Copie el objeto JSON de salida para más adelante.

```output 
  {
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    (...)
  }
```

> [!IMPORTANT]
> Siempre es recomendable conceder acceso mínimo. El ámbito del ejemplo anterior se limita al servidor específico y no a todo el grupo de recursos.

## <a name="copy-the-sql-connection-string"></a>Copie la cadena de conexión SQL. 

En Azure Portal, vaya a Azure SQL Database y abra **Configuración** > **Cadenas de conexión**. Copie la cadena de conexión de **ADO.NET**. Reemplace los valores de marcador de posición por `your_database` y `your_password`. La cadena de conexión tendrá un aspecto similar a esta salida. 

```output
    Server=tcp:my-sql-server.database.windows.net,1433;Initial Catalog={your-database};Persist Security Info=False;User ID={admin-name};Password={your-password};MultipleActiveResultSets=False;Encrypt=True;TrustServerCertificate=False;Connection Timeout=30;
```

Usará la cadena de conexión como secreto de GitHub. 

## <a name="configure-the-github-secrets"></a>Configuración de los secretos de GitHub

1. En [GitHub](https://github.com/), examine el repositorio.

1. Seleccione **Settings > Secrets > New secret** (Configuración > Secretos > Secreto nuevo).

1. Pegue la salida JSON completa del comando de la CLI de Azure en el campo de valor del secreto. Asigne al secreto el nombre `AZURE_CREDENTIALS`.

    Cuando configure el archivo de flujo de trabajo más adelante, usará el secreto para la entrada `creds` de la acción de inicio de sesión de Azure. Por ejemplo:

    ```yaml
    - uses: azure/login@v1
    with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
   ```

1. Vuelva a seleccionar **Nuevo secreto**. 

1. Pegue el valor de la cadena de conexión en el campo de valor del secreto. Asigne al secreto el nombre `AZURE_SQL_CONNECTION_STRING`.


## <a name="add-your-workflow"></a>Agregar el flujo de trabajo

1. Vaya a **Acciones** del repositorio de GitHub. 

2. Seleccione **Set up your workflow yourself** (Configure el flujo de trabajo usted mismo). 

2. Elimine todo lo que aparezca después de la sección `on:` del archivo de flujo de trabajo. Por ejemplo, el flujo de trabajo restante puede tener el siguiente aspecto. 

    ```yaml
    name: CI

    on:
    push:
        branches: [ master ]
    pull_request:
        branches: [ master ]
    ```

1. Cambie el nombre del flujo de trabajo `SQL for GitHub Actions` y agregue las acciones de restauración e inicio de sesión. Estas acciones comprobarán el código del sitio y se autenticarán con Azure mediante el secreto de GitHub `AZURE_CREDENTIALS` que creó anteriormente. 

    ```yaml
    name: SQL for GitHub Actions

    on:
    push:
        branches: [ master ]
    pull_request:
        branches: [ master ]

    jobs:
    build:
        runs-on: windows-latest
        steps:
        - uses: actions/checkout@v1
        - uses: azure/login@v1
        with:
            creds: ${{ secrets.AZURE_CREDENTIALS }}
    ```

1. Use la acción de implementación de Azure SQL para conectarse a la instancia de SQL. Reemplace `SQL_SERVER_NAME` por el nombre del servidor. Debe tener un paquete dacpac (`Database.dacpac`) en el nivel raíz del repositorio. 

    ```yaml
    - uses: azure/sql-action@v1
      with:
        server-name: SQL_SERVER_NAME
        connection-string: ${{ secrets.AZURE_SQL_CONNECTION_STRING }}
        dacpac-package: './Database.dacpac'
    ``` 

1. Complete el flujo de trabajo agregando una acción al cierre de sesión de Azure. Este es el flujo de trabajo completado. El archivo aparecerá en la carpeta `.github/workflows` del repositorio.

    ```yaml
   name: SQL for GitHub Actions

    on:
    push:
        branches: [ master ]
    pull_request:
        branches: [ master ]


    jobs:
    build:
        runs-on: windows-latest
        steps:
        - uses: actions/checkout@v1
        - uses: azure/login@v1
        with:
            creds: ${{ secrets.AZURE_CREDENTIALS }}

    - uses: azure/sql-action@v1
      with:
        server-name: SQL_SERVER_NAME
        connection-string: ${{ secrets.AZURE_SQL_CONNECTION_STRING }}
        dacpac-package: './Database.dacpac'

        # Azure logout 
    - name: logout
      run: |
         az logout
    ```

## <a name="review-your-deployment"></a>Revisar la implementación

1. Vaya a la opción de **Acciones** del repositorio de GitHub. 

1. Abra el primer resultado para ver los registros detallados de la ejecución del flujo de trabajo. 
 
   :::image type="content" source="media/quickstart-sql-github-actions/github-actions-run-sql.png" alt-text="Registro de ejecución de las acciones de GitHub":::

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando la base de datos y el repositorio de Azure SQL ya no sean necesarios, limpie los recursos que implementó eliminando el grupo de recursos y el repositorio de GitHub. 

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Más información sobre la integración de Azure y GitHub](/azure/developer/github/)