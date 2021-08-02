---
title: 'Azure Portal: Enmascaramiento de datos dinámicos'
description: Cómo empezar a usar el enmascaramiento dinámico de datos de Azure SQL Database en Azure Portal
services: sql-database
ms.service: sql-database
ms.subservice: security
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: DavidTrigano
ms.author: datrigan
ms.reviewer: vanto
ms.date: 04/28/2020
ms.openlocfilehash: 3aea8f26acfbb0feb9390f5b35c451cda0ac7726
ms.sourcegitcommit: 942a1c6df387438acbeb6d8ca50a831847ecc6dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112017653"
---
# <a name="get-started-with-sql-database-dynamic-data-masking-with-the-azure-portal"></a>Introducción al enmascaramiento dinámico de datos de SQL Database con Azure Portal
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

En este artículo se muestra cómo implementar el [enmascaramiento dinámico de datos](dynamic-data-masking-overview.md) con Azure Portal. También puede implementar el enmascaramiento dinámico de datos mediante [Cmdlets de Azure SQL Database](/powershell/module/az.sql/) o la [API REST](/rest/api/sql/).

> [!NOTE]
> Esta característica no se puede establecer para la instancia administrada de SQL mediante el portal (use PowerShell o la API REST). Para obtener más información, vea [Dynamic Data Masking](/sql/relational-databases/security/dynamic-data-masking).

## <a name="enable-dynamic-data-masking"></a>Habilitación del enmascaramiento dinámico de datos

1. Inicie Azure Portal en [https://portal.azure.com](https://portal.azure.com).
2. Vaya al recurso de base de datos en Azure Portal. 
3. Seleccione la hoja **Enmascaramiento dinámico de datos** en la sección **Seguridad**. 

   ![Captura de pantalla que muestra la sección Seguridad con Enmascaramiento dinámico de datos resaltado.](./media/dynamic-data-masking-configure-portal/dynamic-data-masking-in-portal.png)

4. En la página de configuración de **Enmascaramiento dinámico de datos** puede ver algunas columnas de base de datos que el motor de recomendaciones ha marcado para enmascaramiento. Para aceptar las recomendaciones, solo tiene que hacer clic en **Agregar máscara** para una o varias columnas y se creará una máscara en función del tipo predeterminado para esta columna. Para cambiar la función de enmascaramiento, haga clic en la regla de enmascaramiento y edite formato de campo de enmascaramiento en el formato diferente que elija. Asegúrese de hacer clic en **Guardar** para guardar la configuración.

    ![Captura de pantalla que muestra la página de configuración de Enmascaramiento dinámico de datos.](./media/dynamic-data-masking-configure-portal/5_ddm_recommendations.png)

5. Para agregar una máscara para cualquier columna de la base de datos, en la parte superior de la hoja de configuración **Enmascaramiento dinámico de datos**, haga clic en **Agregar máscara** para abrir la página de configuración **Agregar regla de enmascaramiento**.

    ![Captura de pantalla que muestra la página de configuración Agregar regla de enmascaramiento.](./media/dynamic-data-masking-configure-portal/6_ddm_add_mask.png)

6. Seleccione los valores para **Esquema**, **Tabla** y **Columna** para definir el campo designado para enmascararse.
7. **Seleccione el método de enmascaramiento** en la lista de categorías de enmascaramiento de datos confidenciales.

    ![Captura de pantalla que muestra las categorías de enmascaramiento de datos confidenciales en la sección Seleccione el método de enmascaramiento.](./media/dynamic-data-masking-configure-portal/7_ddm_mask_field_format.png)

8. Haga clic en **Agregar** en la página de regla de enmascaramiento de datos para actualizar el conjunto de reglas de enmascaramiento en la directiva de enmascaramiento dinámico de datos.
9. Escriba los usuarios SQL o las identidades de Azure Active Directory (Azure AD) que deben excluirse del enmascaramiento y acceda a los datos confidenciales sin máscara. Esto debe ser una lista separada por puntos y coma de usuarios. Los usuarios con privilegios de administrador siempre tienen acceso a los datos originales sin máscara.

    ![Panel de navegación](./media/dynamic-data-masking-configure-portal/8_ddm_excluded_users.png)

    > [!TIP]
    > Para hacer que el nivel de aplicación pueda mostrar datos confidenciales para los usuarios con privilegios de la aplicación, agregue el usuario SQL o identidad de Azure AD que la aplicación usa para consultar la base de datos. Se recomienda que esta lista incluya un número mínimo de usuarios con privilegio para minimizar la exposición de los datos confidenciales.

10. Haga clic en **Guardar** en la página de configuración de enmascaramiento de datos para guardar la directiva de enmascaramiento nueva o actualizada.

## <a name="next-steps"></a>Pasos siguientes

- Para obtener información general sobre el enmascaramiento dinámico de datos, consulte [este artículo](dynamic-data-masking-overview.md).
- También puede implementar el enmascaramiento dinámico de datos mediante [Cmdlets de Azure SQL Database](/powershell/module/az.sql/) o la [API REST](/rest/api/sql/).
