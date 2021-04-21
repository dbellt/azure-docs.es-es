---
title: Habilitación de las características del área de trabajo de Synapse
description: En este documento se describe cómo un usuario puede habilitar las características del área de trabajo de Synapse en un grupo de SQL dedicado (anteriormente, SQL DW).
services: synapse-analytics
author: antvgski
manager: igorstan
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 11/25/2020
ms.author: anvang
ms.reviewer: jrasnick
ms.openlocfilehash: 605f495fbc5913663499f5411abcaa531cea97ee
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107312219"
---
# <a name="enabling-synapse-workspace-features-for-a-dedicated-sql-pool-formerly-sql-dw"></a>Habilitación de las características del área de trabajo de Synapse en un grupo de SQL dedicado (anteriormente, SQL DW)

Ahora, todos los usuarios de SQL Data Warehouse pueden acceder y usar una instancia de un grupo de SQL dedicado (anteriormente, SQL DW) a través de Synapse Studio y el área de trabajo. Los usuarios pueden usar Synapse Studio y el área de trabajo sin que ello afecte a la automatización, las conexiones ni las herramientas. En este artículo se explica cómo un usuario de Azure Synapse Analytics puede habilitar las características del área de trabajo de Synapse en un grupo de SQL dedicado (anteriormente, SQL DW). El usuario puede expandir su solución de análisis aprovechando las nuevas funcionalidades repletas de características que ahora están disponibles a través del área de trabajo de Synapse y Studio.   

## <a name="prerequisites"></a>Requisitos previos
Antes de habilitar las características del área de trabajo de Synapse en el almacenamiento de datos, debe asegurarse de que tiene lo siguiente:
- Derechos para crear y administrar los recursos de SQL que están hospedados en el servidor lógico de SQL.
- Permisos de escritura en el host SQL Server 
- Derechos para crear recursos de Azure Synapse.
- Un administrador de Azure Active Directory identificado en el servidor lógico

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/).

## <a name="enabling-synapse-workspace-features-for-an-existing-dedicated-sql-pool-formerly-sql-dw"></a>Habilitación de las características del área de trabajo de Synapse en un grupo de SQL dedicado (anteriormente, SQL DW)

Las características del área de trabajo de Synapse se pueden habilitar en cualquier grupo de SQL dedicado (anteriormente, SQL DW), en una región admitida. Esta funcionalidad solo está disponible a través de Azure Portal.

Siga estos pasos para crear un área de trabajo de Synapse para el almacenamiento de datos que ya existe.
1. Seleccione un grupo de SQL dedicado (anteriormente, SQL DW) y abra la página de información general.
2. Seleccione **Nueva área de trabajo de Synapse** en la barra de menús superior o en el mensaje de justo debajo.
3. Después de revisar la lista de almacenamiento de datos que están disponibles a través de la nueva área de trabajo de Synapse, en la página **Crear área de trabajo de Azure Synapse**. Haga clic en **Continuar** para seguir.
4. En la página Aspectos básicos, el grupo de SQL dedicado (la sección **Detalles del proyecto**) se debe rellenar previamente con la misma **suscripción** y el mismo **grupo de recursos** que se implementaron en el servidor lógico. En **Detalles del área de trabajo**, el nombre del **área de trabajo** se rellena previamente con el mismo nombre y la misma región que el servidor lógico de SQL para asegurarse de que la conexión entre la nueva área de trabajo de Synapse y el servidor lógico se pueda crear durante el proceso de aprovisionamiento. El aprovisionamiento posterior de esta conexión se debe conservar para garantizar el acceso continuado a las instancias del grupo de SQL dedicado (anteriormente, SQL DW) a través del área de trabajo y Studio.
5. Vaya a Select Data Lake Storage Gen 2 (Seleccionar Data Lake Storage Gen 2).
6. Seleccione **Crear nuevo** o seleccione un **Data Lake Storage Gen2** antes de seleccionar **Siguiente: Redes**.
7. Elija una **contraseña de administrador de SQL** para la **instancia sin servidor**. Al cambiar esta contraseña, no se actualizan ni cambian las credenciales de SQL del servidor lógico. Si prefiere una contraseña definida por el sistema, deje estos campos vacíos. Esta contraseña puede cambiarse en cualquier momento dentro del área de trabajo nueva. El nombre de administrador debe ser el mismo que se usa en SQL Server.
8. Seleccione la **configuración de red preferida** y **Revisar y crear** para iniciar el aprovisionamiento del área de trabajo.
9. Seleccione **Ir al recurso** para abrir el área de trabajo nueva.

    > [!NOTE]
    > Todas las instancias del grupo de SQL dedicado (anteriormente, SQL DW) hospedadas en el servidor lógico están disponibles a través de la nueva área de trabajo.

## <a name="post-provisioning-steps"></a>Pasos posteriores al aprovisionamiento
Se deben completar los pasos siguientes para asegurarse de que se puede acceder a las instancias del grupo de SQL dedicado (anteriormente, SQL DW) a través de Synapse Studio.
1. En la página de información general del área de trabajo de Synapse, seleccione **Servidor conectado**. El **servidor conectado** le llevará al servidor lógico de SQL conectado que hospeda los almacenamientos de datos. En el menú Esencial, seleccione **Servidor conectado**.
2. Abra **Firewalls y redes virtuales** y asegúrese de que la dirección IP de cliente o un intervalo IP predeterminado tienen acceso al servidor lógico.
3. Abra **Administrador de Active Directory** y asegúrese de que se ha establecido un administrador de AAD en el servidor lógico.
4. Seleccione una de las instancias del grupo de SQL dedicado (anteriormente, SQL DW) hospedadas en el servidor lógico. En la página de información general, seleccione **Iniciar Synapse Studio** o vaya a [Iniciar sesión en Synapse Studio](https://web.azuresynapse.net) e inicie sesión en el área de trabajo.

5. Abra el **centro de datos** y expanda el grupo de SQL dedicado en el explorador de objetos para asegurarse de que tiene acceso y puede consultar el almacenamiento de datos.

    > [!NOTE] 
    > Un área de trabajo conectada se puede eliminar en cualquier momento. Al eliminar el área de trabajo, no se eliminará el grupo de SQL dedicado conectado (anteriormente SQL DW). La característica Área de trabajo se puede volver a habilitar en el grupo de SQL dedicado (anteriormente SQL DW) cuando se ha completado la operación de eliminación.

## <a name="next-steps"></a>Pasos siguientes
Introducción al [área de trabajo de Synapse y Studio](../get-started.md).
