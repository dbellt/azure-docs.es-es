---
title: Grupo de confianza del servidor
titleSuffix: Azure SQL Managed Instance
description: Obtenga información sobre el grupo de confianza del servidor y cómo administrar la confianza entre instancias de Azure SQL Managed Instance.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: conceptual
author: sasapopo
ms.author: sasapopo
ms.reviewer: sstein, bonova
ms.date: 10/08/2020
ms.openlocfilehash: e1f732c29d977ff421496809f149ee722c766072
ms.sourcegitcommit: 38d81c4afd3fec0c56cc9c032ae5169e500f345d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109517431"
---
# <a name="use-server-trust-groups-to-set-up-and-manage-trust-between-sql-managed-instances"></a>Uso de grupos de confianza de servidor para configurar y administrar la confianza entre instancias de SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Un grupo de confianza de servidor es un concepto que se usa para administrar la confianza entre instancias de Azure SQL Managed Instance. Al crear un grupo, se establece una confianza basada en certificados entre sus miembros. Esta confianza se puede usar para diferentes escenarios entre instancias. Al quitar servidores del grupo o eliminar el grupo, se quita la confianza entre los servidores. Para crear o eliminar el grupo de confianza del servidor, el usuario debe tener permisos de escritura en la instancia administrada.
Un [grupo de confianza del servidor](/azure/templates/microsoft.sql/allversions) es un objeto de Azure Resource Manager que se ha etiquetado como **grupo de confianza de SQL** en Azure Portal.

> [!NOTE]
> El grupo de confianza del servidor se incluye en la versión preliminar pública de las transacciones distribuidas entre instancias de Azure SQL Managed Instance y actualmente tiene algunas limitaciones que se describirán más adelante en este artículo.

## <a name="server-trust-group-setup"></a>Configuración de un grupo de confianza del servidor

El grupo de confianza del servidor se puede configurar mediante [Azure PowerShell](https://docs.microsoft.com/powershell/module/az.sql/new-azsqlservertrustgroup) o la [CLI de Azure](https://docs.microsoft.com/cli/azure/sql/stg). En la siguiente sección se describe la configuración del grupo de confianza del servidor mediante Azure Portal.

1. Vaya a [Azure Portal](https://portal.azure.com/).

2. Vaya a la instancia de Azure SQL Managed Instance que tiene previsto agregar a un nuevo grupo de confianza del servidor.

3. En la configuración de **Seguridad**, seleccione la pestaña **SQL trust groups** (Grupos de confianza SQL).

   :::image type="content" source="./media/server-trust-group-overview/security-sql-trust-groups.png" alt-text="Grupos de confianza del servidor":::

4. En la página de configuración del grupo de confianza del servidor, seleccione el icono **Nuevo grupo**.

   :::image type="content" source="./media/server-trust-group-overview/server-trust-group-create-new-group.png" alt-text="Nuevo grupo":::

5. En la hoja de creación del **grupo de confianza de SQL**, defina el **Nombre del grupo**. Debe ser único en todas las regiones en que residen los miembros del grupo. El **ámbito de confianza** define el tipo de escenario entre instancias que se habilita con el grupo de confianza del servidor. En la vista previa, el único ámbito de confianza aplicable es **Transacciones distribuidas**, por lo que está preseleccionado y no se puede cambiar. Todos los **miembros del grupo** deben pertenecer a la misma **suscripción**, pero pueden estar en grupos de recursos distintos. Seleccione el **Grupo de recursos** y **SQL Server/instancia** para elegir la instancia de Azure SQL Managed Instance que será miembro del grupo.

   :::image type="content" source="./media/server-trust-group-overview/server-trust-group-create-blade.png" alt-text="Hoja de creación de grupo de confianza del servidor":::

6. Una vez que se hayan completado todos los campos obligatorios, haga clic en **Guardar**.

## <a name="server-trust-group-maintenance-and-deletion"></a>Mantenimiento y eliminación de grupos de confianza del servidor

No se puede editar el grupo de confianza del servidor. Para quitar una instancia administrada de un grupo, debe eliminar el grupo y crear uno nuevo.

En la siguiente sección se describe el proceso de eliminación de grupos de confianza del servidor. 
1. Vaya a Azure Portal.
2. Navegue a una instancia administrada que pertenezca al grupo de confianza.
3. En la configuración de **Seguridad**, seleccione la pestaña **SQL trust groups** (Grupos de confianza SQL).
4. Seleccione el grupo de recursos que quiere eliminar.
   :::image type="content" source="./media/server-trust-group-overview/server-trust-group-manage-select.png" alt-text="Seleccione el grupo de confianza del servidor":::
5. Haga clic en **Eliminar grupo**.
   :::image type="content" source="./media/server-trust-group-overview/server-trust-group-manage-delete.png" alt-text="Eliminación del grupo de confianza del servidor":::
6. Escriba el nombre del grupo de confianza del servidor para confirmar la eliminación y haga clic en **Eliminar**.
   :::image type="content" source="./media/server-trust-group-overview/server-trust-group-manage-delete-confirm.png" alt-text="Confirmación de la eliminación del grupo de confianza del servidor":::

> [!NOTE]
> Es posible que la eliminación del grupo de confianza del servidor no quite inmediatamente la confianza entre las dos instancias administradas. Para aplicar la eliminación de la confianza, puede invocar la [conmutación por error](/powershell/module/az.sql/Invoke-AzSqlInstanceFailover) de instancias administradas. Consulte los [Problemas conocidos](../database/doc-changes-updates-release-notes.md?tabs=managed-instance#known-issues) para obtener información actualizada sobre este tema.

## <a name="limitations"></a>Limitaciones

Durante la versión preliminar pública, se aplican las siguientes limitaciones a los grupos de confianza del servidor:
 * El nombre del grupo de confianza de servidor debe ser único en todas las regiones en que se encuentren sus miembros.
 * El grupo solo puede contener instancias de Azure SQL Managed Instance y deben estar en la misma suscripción de Azure.
 * En la versión preliminar, el grupo puede tener exactamente dos instancias administradas. Para ejecutar transacciones distribuidas en más de dos instancias administradas, deberá crear un grupo de confianza del servidor para cada par de instancias administradas.
 * Las transacciones distribuidas son el único ámbito aplicable para los grupos de confianza del servidor.
 * El grupo de confianza del servidor solo se puede administrar desde Azure Portal. Próximamente, habrá compatibilidad con PowerShell y la CLI.
 * El grupo de confianza del servidor no se puede editar en Azure Portal. Solo se puede crear o quitar.
 * Puede haber limitaciones adicionales de transacciones distribuidas relacionadas con el escenario. La más importante es que debe existir una conectividad entre las instancias administradas mediante puntos de conexión privados, VNET o emparejamiento de VNET. Asegúrese de conocer las [limitaciones de las transacciones distribuidas para Managed Instance](../database/elastic-transactions-overview.md#limitations).

## <a name="next-steps"></a>Pasos siguientes

* Para obtener más información acerca de las transacciones distribuidas en Azure SQL Managed Instance, consulte [Transacciones distribuidas](../database/elastic-transactions-overview.md).
* Para obtener las actualizaciones de versión y el estado de los problemas conocidos, vea [Notas de la versión de Managed Instance](../database/doc-changes-updates-release-notes.md).
* Si tiene solicitudes de características, agréguelas al [Foro de Managed Instance](https://feedback.azure.com/forums/915676-sql-managed-instance).
