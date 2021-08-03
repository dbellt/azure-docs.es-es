---
title: Examen de Azure Cosmos Database (SQL API)
description: En esta guía se describen los detalles de cómo examinar Azure Cosmos Database (SQL API).
author: djpmsft
ms.author: daperlov
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: how-to
ms.date: 05/08/2021
ms.openlocfilehash: 23825bd3ab41891f775f26e2ee2b052e3a041401
ms.sourcegitcommit: 832e92d3b81435c0aeb3d4edbe8f2c1f0aa8a46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/07/2021
ms.locfileid: "111560149"
---
# <a name="register-and-scan-azure-cosmos-database-sql-api"></a>Registro y examen de Azure Cosmos Database (SQL API)

En este artículo se describe cómo registrar una cuenta de Azure Cosmos Database (SQL API) en Azure Purview y cómo configurar un examen.

## <a name="supported-capabilities"></a>Funcionalidades admitidas

Azure Cosmos Database (SQL API) admite exámenes completos e incrementales para capturar los metadatos y el esquema. Los exámenes también clasifican los datos automáticamente en función de las reglas de clasificación personalizadas y del sistema.

## <a name="prerequisites"></a>Requisitos previos

- Antes de registrar los orígenes de datos, cree una cuenta de Azure Purview. Para más información sobre cómo crear una cuenta de Purview, consulte [Inicio rápido: creación de una cuenta de Azure Purview](create-catalog-portal.md).
- Tenga en cuenta que debe ser administrador de los orígenes de datos de Azure Purview.

## <a name="setting-up-authentication-for-a-scan"></a>Configuración de la autenticación para un examen

Solo hay una manera de configurar la autenticación de Azure Cosmos Database (SQL API):

- Clave de cuenta
 
### <a name="account-key"></a>Clave de cuenta

Cuando el método de autenticación seleccionado es **Clave de cuenta**, debe obtener la clave de acceso y almacenarla en el almacén de claves:

1. Vaya a la cuenta de Cosmos DB en Azure Portal. 
1. Seleccione **Configuración** > **Claves**. 
1. Copie una clave PRINCIPAL o SECUNDARIA de las *claves de lectura y escritura* o *claves de solo lectura*, y guárdela en algún lugar para los pasos siguientes.
1. Vaya a almacén de claves.
1. Seleccione **Configuración > Secretos**.
1. Seleccione **+ Generar/Importar** y escriba el **nombre** y el **valor** como la *clave* de la cuenta de Azure Cosmos DB.
1. Seleccione **Crear** para completar la acción.
1. Si el almacén de claves no está conectado todavía a Purview, necesitará [crear una conexión del almacén de claves](manage-credentials.md#create-azure-key-vaults-connections-in-your-azure-purview-account).
1. Por último, [cree una nueva credencial](manage-credentials.md#create-a-new-credential) mediante la clave para configurar el examen.

## <a name="register-an-azure-cosmos-database-sql-api-account"></a>Registro de una cuenta de Azure Cosmos Database (SQL API)

Para registrar una nueva cuenta de Azure Cosmos Database (SQL API) en el catálogo de datos, haga lo siguiente:

1. Vaya a la cuenta de Purview.
1. Seleccione **Sources** (Orígenes) en el panel de navegación izquierdo.
1. Seleccione **Registrar**.
1. En **Register sources** (Registrar orígenes), seleccione **Azure Cosmos DB (SQL API)** .
1. Seleccione **Continuar**

:::image type="content" source="media/register-scan-azure-cosmos-database/register-new-data-source.png" alt-text="Registro de un origen de datos nuevo" border="true":::

En la pantalla **Register sources (Azure Cosmos DB (SQL API))** (Registrar orígenes [Azure Cosmos DB (SQL API)]), haga lo siguiente:

1. Escriba un **nombre** con el que se mostrará el origen de datos en el catálogo.
2. Elija la suscripción de Azure para filtrar las bases de datos de Azure Cosmos.
3. Seleccione un nombre de cuenta de Cosmos DB adecuado.
4. Seleccione una colección o cree una nueva (opcional).
5. Seleccione **Registrar** para registrar el origen de datos.

:::image type="content" source="media/register-scan-azure-cosmos-database/register-sources.png" alt-text="Opciones de registro de orígenes" border="true":::


## <a name="creating-and-running-a-scan"></a>Creación y ejecución de un examen

Para crear y ejecutar un nuevo examen, siga estos pasos:

1. Seleccione la pestaña **Mapa de datos** en el panel izquierdo de Purview Studio.

1. Seleccione el origen de datos de Azure Cosmos DB que ha registrado.

1. Seleccione **Nuevo análisis**.

1. Seleccione la credencial para conectarse al origen de datos. 

   :::image type="content" source="media/register-scan-azure-cosmos-database/set-up-scan-cosmos.png" alt-text="Configurar examen":::

1. Elija los elementos adecuados de la lista para limitar el ámbito del examen a bases de datos específicas.

   :::image type="content" source="media/register-scan-azure-cosmos-database/cosmos-database-scope-your-scan.png" alt-text="Ámbito del examen":::

1. A continuación, seleccione un conjunto de reglas de examen. Puede elegir entre los valores predeterminados del sistema, los conjuntos de reglas personalizadas existentes o la creación de un conjunto de reglas en línea.

   :::image type="content" source="media/register-scan-azure-cosmos-database/select-scan-rule-set.png" alt-text="Conjunto de reglas de examen":::

1. Elija el desencadenador del examen. Puede configurar una programación o ejecutar el examen una vez.

   :::image type="content" source="media/register-scan-azure-cosmos-database/trigger-scan.png" alt-text="trigger":::

1. Revise el examen y seleccione **Save and run** (Guardar y ejecutar).

[!INCLUDE [view and manage scans](includes/view-and-manage-scans.md)]

## <a name="next-steps"></a>Pasos siguientes

- [Examen del catálogo de datos de Azure Purview](how-to-browse-catalog.md)
- [Búsqueda en el catálogo de datos de Azure Purview](how-to-search-catalog.md)
