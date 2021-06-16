---
title: Cambio en contexto de la versión de SQL Server
description: Aprenda a cambiar la versión de la máquina virtual de SQL Server en Azure.
services: virtual-machines-windows
documentationcenter: na
author: ramakoni1
manager: ramakoni1
tags: azure-resource-manager
ms.service: virtual-machines-sql
ms.subservice: management
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 06/08/2020
ms.author: RamaKoni
ms.reviewer: sqlblt, daleche
ms.custom: seo-lt-2019
ms.openlocfilehash: eb8d0150bafe795f2b23fa18581adc60d091aaf2
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112078840"
---
# <a name="in-place-change-of-sql-server-version-on-azure-vm"></a>Cambio en contexto de la versión de SQL Server en la máquina virtual de Azure

[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

En este artículo se describe cómo cambiar la versión de Microsoft SQL Server en una máquina virtual (VM) Windows en Microsoft Azure.

## <a name="prerequisites"></a>Requisitos previos

Para realizar una actualización local de SQL Server, se aplican estas condiciones:

- Se requiere el soporte de instalación de la versión deseada de SQL Server. Los clientes que tengan [Software Assurance](https://www.microsoft.com/licensing/licensing-programs/software-assurance-default) pueden obtener el soporte de instalación en el [centro de licencias por volumen](https://www.microsoft.com/Licensing/servicecenter/default.aspx). Los clientes que no tienen Software Assurance pueden usar el soporte de instalación de una imagen de VM con SQL Server de Azure Marketplace con una versión posterior de SQL Server (generalmente ubicada en C:\SQLServerFull).
- Las actualizaciones de edición deben seguir las [rutas de actualización compatibles](/sql/database-engine/install-windows/supported-version-and-edition-upgrades-version-15).

## <a name="planning-for-version-change"></a>Planeamiento del cambio de versión

Se recomienda revisar estos elementos antes de llevar a cabo el cambio de versión:

1. Revise las novedades de la versión a la que planea actualizarse:

   - Novedades de [SQL 2019](/sql/sql-server/what-s-new-in-sql-server-ver15)
   - Novedades de [SQL 2017](/sql/sql-server/what-s-new-in-sql-server-2017)
   - Novedades de [SQL 2016](/sql/sql-server/what-s-new-in-sql-server-2016)


1. Se recomienda comprobar la [certificación de compatibilidad](/sql/database-engine/install-windows/compatibility-certification) de la versión a la que se va a cambiar para que pueda usar los modos de compatibilidad de la base de datos a fin de minimizar el efecto de la actualización.
1. Puede revisar estos artículos para asegurarse de que el resultado sea correcto:

   - [Vídeo: Modernización de SQL Server | Pam Lahoud y Pedro Lopes | 20 años de PASS](https://www.youtube.com/watch?v=5RPkuQHcxxs&feature=youtu.be)
   - [Asistente para experimentación con bases de datos para pruebas de AB](/sql/dea/database-experimentation-assistant-overview)
   - [Actualización de bases de datos mediante el Asistente para la optimización de consultas](/sql/relational-databases/performance/upgrade-dbcompat-using-qta)
   - [Cambio del nivel de compatibilidad de la base de datos y uso del Almacén de consultas](/sql/database-engine/install-windows/change-the-database-compatibility-mode-and-use-the-query-store)

## <a name="upgrade-sql-version"></a>Actualización de la versión de SQL

> [!WARNING]
> Al actualizar la versión de SQL Server se reiniciará el servicio para SQL Server junto con los demás servicios asociados, como Analysis Services y R Services.

Para actualizar la versión de SQL Server, obtenga los soportes de instalación de SQL Server correspondientes a la versión más reciente que pueda [admitir la ruta de actualización](/sql/database-engine/install-windows/supported-version-and-edition-upgrades-version-15) de SQL Server y siga estos pasos:

1. Haga una copia de seguridad de las bases de datos, incluidas las bases de datos de sistema (excepto tempdb) y de usuario, antes de iniciar el proceso. También puede crear una copia de seguridad de nivel de máquina virtual coherente con la aplicación mediante los servicios Azure Backup.
1. Inicie Setup.exe desde el soporte de instalación de SQL Server.
1. El Asistente para instalación inicia el Centro de instalación de SQL Server. Para actualizar una instancia existente de SQL Server, seleccione **Instalación** en el panel de navegación y seleccione **Upgrade from an earlier version of SQL Server** (Actualización desde una versión anterior de SQL Server).

   :::image type="content" source="./media/change-sql-server-version/upgrade.png" alt-text="Selección de la opción Actualización de versión de SQL Server":::

1. En la página **Clave del producto**, seleccione una opción para indicar si va a actualizar a una edición gratuita de SQL Server o si tiene una clave de PID para una versión de producción del producto. Para más información, consulte [Ediciones y características admitidas de SQL Server 2019 (15.x)](/sql/sql-server/editions-and-components-of-sql-server-version-15) y [Actualizaciones de ediciones y versiones admitidas (SQL Server 2016)](/sql/database-engine/install-windows/supported-version-and-edition-upgrades).
1. Seleccione **Siguiente** hasta llegar a la página **Listo para la actualización** y seleccione **Actualizar**. La ventana de configuración pude dejar de responder durante varios minutos mientras el cambio surte efecto. Una página **Completada** confirmará que la actualización se completó. Para un procedimiento de actualización paso a paso, consulte [el procedimiento completo](/sql/database-engine/install-windows/upgrade-sql-server-using-the-installation-wizard-setup#procedure).

   :::image type="content" source="./media/change-sql-server-version/complete-page.png" alt-text="Página Completada":::

Si cambió la edición de SQL Server además de la versión, actualice también la edición y consulte la sección **Comprobación de la versión y edición en el portal** para cambiar la instancia de VM con SQL.

   :::image type="content" source="./media/change-sql-server-version/change-portal.png" alt-text="Cambio de metadatos de la versión":::

## <a name="downgrade-the-version-of-sql-server"></a>Cambio a una versión anterior de SQL Server

Para cambiar a una versión anterior de SQL Server, debe desinstalar completamente SQL Server y volver a instalarlo con la versión deseada. Esto es similar a hacer una instalación nueva de SQL Server, porque no podrá restaurar la base de datos anterior de una versión posterior en la versión anterior recién instalada. Las bases de datos tendrán que volver a crearse desde el principio. Si también cambió la edición de SQL Server durante la actualización, cambie la propiedad **Edición** de la VM con SQL Server en Azure Portal al valor de la edición nueva. De esta forma se actualizan los metadatos y la facturación asociados a esta máquina virtual.

> [!WARNING]
> No se admite el cambio a una versión anterior de SQL Server en contexto.

Para cambiar a una versión anterior de SQL Server, siga estos pasos:

1. Asegúrese de no estar usando ninguna característica que esté [disponible solo en la versión más reciente](https://social.technet.microsoft.com/wiki/contents/articles/24222.find-enterprise-only-features-in-your-database.aspx).
1. Haga copias de seguridad de todas las bases de datos, incluidas las bases de datos de sistema (excepto tempdb) y de usuario.
1. Exporte todos los objetos de nivel de servidor necesarios (como desencadenadores de servidor, roles, inicios de sesión, servidores vinculados, trabajos, credenciales y certificados).
1. Si no tiene scripts para volver a crear las bases de usuario en la versión anterior, debe generar un script de todos los objetos y exportar todos los datos con BCP.exe, SSIS o DACPAC.

   Asegúrese de seleccionar las opciones correctas al incluir en el script tales elementos como la versión de destino, los objetos dependientes y las opciones avanzadas.

   :::image type="content" source="./media/change-sql-server-version/scripting-options.png" alt-text="Opciones de scripting":::

1. Desinstale completamente todos los servicios asociados y SQL Server.
1. Reinicie la VM.
1. Instale SQL Server con el soporte correspondiente a la versión deseada del programa.
1. Instale los últimos Service Pack y las actualizaciones acumulativas.
1. Importe todos los objetos de nivel de servidor necesarios (que se exportaron en el paso 3).
1. Vuelva a crear todas las bases de datos de usuario necesarias desde cero (mediante el uso de los archivos o scripts creados en el paso 4).

## <a name="verify-the-version-and-edition-in-the-portal"></a>Comprobación de la versión y edición en el portal

Una vez que cambie la versión de SQL Server, registre nuevamente la VM con SQL Server con la [extensión Agente de IaaS de SQL](sql-agent-extension-manually-register-single-vm.md) para que pueda usar Azure Portal para ver la versión de SQL Server. El número de versión que aparece ahora debería reflejar la versión y edición recién actualizadas de la instalación de SQL Server.

:::image type="content" source="./media/change-sql-server-version/verify-portal.png" alt-text="Comprobación de versión":::

> [!NOTE]
> Si ya se registró con la extensión Agente de IaaS de SQL, [anule el registro](sql-agent-extension-manually-register-single-vm.md#unregister-from-extension) y vuelva a [registrar el recurso de VM con SQL](sql-agent-extension-manually-register-single-vm.md#register-with-extension) para que detecte la versión y edición correctas de SQL Server instaladas en la VM. De esta forma se actualizan los metadatos y la información de facturación asociados a esta máquina virtual.

## <a name="remarks"></a>Observaciones

- Se recomienda iniciar copias de seguridad, actualizar las estadísticas, recompilar los índices y comprobar la coherencia una vez que se completa la actualización. También puede comprobar los niveles de compatibilidad de la base de datos individuales para asegurarse de que reflejen el nivel deseado.
- Una vez que actualice SQL Server en la VM, asegúrese de que la propiedad **Edición** de SQL Server en Azure Portal coincida con el número de la edición instalada para la facturación.
- La posibilidad de [cambiar la edición](change-sql-server-edition.md#change-edition-in-portal) es una característica de la extensión Agente de IaaS de SQL. Al implementar una imagen de Azure Marketplace desde Azure Portal, se registra automáticamente una VM con SQL Server con la extensión. Sin embargo, los clientes que instalen automáticamente SQL Server deberán [registrar su VM con SQL Server](sql-agent-extension-manually-register-single-vm.md) de forma manual.
- Si quita el recurso de máquina virtual con SQL Server, se restaurará la configuración de edición codificada de forma rígida de la imagen.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los siguientes artículos.

- [Introducción a SQL Server en máquinas virtuales Windows](sql-server-on-azure-vm-iaas-what-is-overview.md)
- [Preguntas más frecuentes de SQL Server en máquinas virtuales Windows](frequently-asked-questions-faq.yml)
- [Orientación de precios de SQL Server para máquinas virtuales de Azure](pricing-guidance.md)
- [Notas de la versión de SQL Server en Azure Virtual Machines](doc-changes-updates-release-notes.md)