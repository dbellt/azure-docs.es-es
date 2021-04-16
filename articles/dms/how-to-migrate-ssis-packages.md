---
title: Nueva migración de paquetes de SSIS a una base de datos única de SQL
titleSuffix: Azure Database Migration Service
description: Obtenga información sobre cómo migrar o volver a implementar paquetes y proyectos de SQL Server Integration Services a una base de datos única de Azure SQL Database mediante Azure Database Migration Service o Data Migration Assistant.
services: database-migration
author: pochiraju
ms.author: rajpo
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: how-to
ms.date: 02/20/2020
ms.openlocfilehash: e42b5ab2bbdebb2584b38531bd6458a98563ef42
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105643742"
---
# <a name="redeploy-ssis-packages-to-azure-sql-database-with-azure-database-migration-service"></a>Nueva implementación de paquetes de SSIS a Azure SQL Database con Azure Database Migration Service

Si usa SQL Server Integration Services (SSIS) y quiere migrar sus proyectos o paquetes de SSIS desde la SSISDB de origen que hospeda SQL Server a la SSISDB de destino que hospeda Azure SQL Database, puede volver a implementarlos usando el Asistente para la implementación de Integration Services. Puede iniciar al asistente desde dentro de SQL Server Management Studio (SSMS).

Si la versión de SSIS que usa es anterior a 2012, antes de volver a implementar los proyectos o paquetes de SSIS en el modelo de implementación del proyecto, deberá convertirlos mediante el Asistente de conversión de Integration Services Project, que también se puede iniciar desde SSMS. Para obtener más información, vea el artículo [Para convertir un proyecto al modelo de implementación de proyectos](/sql/integration-services/packages/deploy-integration-services-ssis-projects-and-packages#convert).

> [!NOTE]
> Azure Database Migration Service (DMS) actualmente no admite la migración de una SSISDB de origen a Azure SQL Database, pero puede volver a implementar los proyectos o paquetes de SSIS mediante el siguiente proceso.

En este artículo aprenderá a:
> [!div class="checklist"]
>
> * Evaluar los proyectos o paquetes SSIS de origen.
> * Migrar de paquetes o proyectos SSIS a Azure.

## <a name="prerequisites"></a>Requisitos previos

Para completar estos pasos, necesitará lo siguiente:

* SSMS versión 17.2 o posterior.
* Una instancia del servidor de base de datos de destino para hospedar SSISDB. Si todavía no tiene una, cree un [servidor de SQL lógico](../azure-sql/database/logical-servers.md) (sin una base de datos) en Azure Portal; para ello, vaya al [formulario](https://ms.portal.azure.com/#create/Microsoft.SQLServer) de SQL Server (solo servidor lógico).
* Se debe aprovisionar SSIS en la instancia de Azure Data Factory (ADF) que contiene Azure-SSIS Integration Runtime (IR) con la SSISDB de destino hospedada por SQL Database (como se describe en el artículo [Aprovisionamiento de la instancia de Azure-SSIS Integration Runtime en Azure Data Factory](../data-factory/tutorial-deploy-ssis-packages-azure.md)).

## <a name="assess-source-ssis-projectspackages"></a>Evaluar los proyectos o paquetes SSIS de origen

Si bien la evaluación de la SSISDB de origen aún no está integrada en Database Migration Assistant (DMA) ni Azure Database Migration Service (DMS), los proyectos o paquetes de SSIS se evaluarán o validarán a medida que se vuelvan a implementar en la SSISDB de destino hospedada por Azure SQL Database.

## <a name="migrate-ssis-projectspackages"></a>Migración de paquetes o proyectos SSIS

Para migrar proyectos o paquetes de SSIS a Azure SQL Database, siga los pasos a continuación.

1. Abra SSMS y después haga clic en **Opciones** para mostrar el cuadro de diálogo **Conectarse al servidor**.

2. En la pestaña **Iniciar sesión**, especifique la información necesaria para conectarse al servidor que hospedará la SSISDB de destino.

    ![Pestaña Iniciar sesión de SSIS](media/how-to-migrate-ssis-packages/dms-ssis-login-tab.png)

3. En la pestaña **Propiedades de la conexión**, en el cuadro de texto **Conectar a base de datos**, seleccione o escriba **SSISDB** y después haga clic en **Conectar**.

    ![Pestaña Propiedades de la conexión de SSIS](media/how-to-migrate-ssis-packages/dms-ssis-conncetion-properties-tab.png)

4. En el Explorador de objetos de SSMS, expanda el nodo **Catálogos de Integration Services**, expanda **SSISDB** y si no hay ninguna carpeta existente, haga clic en **SSISDB** y cree una carpeta.

5. En **SSISDB**, expanda cualquier carpeta, haga clic en **Proyectos** y después seleccione **Implementar proyecto**.

    ![Nodo SSISDB de SSIS expandido](media/how-to-migrate-ssis-packages/dms-ssis-ssisdb-node-expanded.png)

6. En el Asistente para la implementación de Integration Services, en la página **Introducción**, revise la información y después haga clic en **Siguiente**.

    ![Página de Introducción del Asistente para la implementación](media/how-to-migrate-ssis-packages/dms-deployment-wizard-introduction-page.png)

7. En la página **Seleccionar origen**, especifique el proyecto SSIS existente que desea implementar.

    Si SSMS también está conectado al SQL Server que hospeda la SSISDB de origen, seleccione **Catálogo de Integration Services** y después escriba la ruta de acceso del proyecto y el nombre del servidor en el catálogo para implementar el proyecto directamente.

    Como alternativa, seleccione **Archivo de implementación de proyecto** y después especifique la ruta de acceso a un archivo de implementación de proyecto (.ispac) existente para implementar el proyecto.

    ![Página Seleccionar origen del Asistente para la implementación](media/how-to-migrate-ssis-packages/dms-deployment-wizard-select-source-page.png)
 
8. Seleccione **Next** (Siguiente).
9. Especifique el destino del proyecto en la página **Seleccionar destino**.

    a. En el cuadro de texto Nombre del servidor, escriba el nombre completo del servidor (<nombre_del_servidor>.database.windows.net).

    b. Proporcione la información de autenticación y después haga clic en **Conectar**.

    ![Página Seleccionar destino del Asistente para la implementación](media/how-to-migrate-ssis-packages/dms-deployment-wizard-select-destination-page.png)

    c. Haga clic en **Examinar** para especificar la carpeta de destino en SSISDB y después seleccione **Siguiente**.

    > [!NOTE]
    > El botón **Siguiente** se habilita solo después de hacer clic en **Conectar**.

10. En la página **Validar**, vea los errores o advertencias y después, si es necesario, modifique los paquetes según corresponda.

    ![Página Validación del Asistente para la implementación](media/how-to-migrate-ssis-packages/dms-deployment-wizard-validate-page.png)

11. Seleccione **Next** (Siguiente).

12. Revise la configuración de implementación en la página **Revisar**.

    > [!NOTE]
    > Puede cambiar la configuración si selecciona **Anterior** o cualquiera de los vínculos de paso del panel de la izquierda.

13. Haga clic en **Implementar** para iniciar el proceso de implementación.

14. Una vez completado el proceso de implementación, puede ver la página Resultados, en la que se muestra el éxito o fracaso de cada acción de implementación.
    a. Si se han producido errores en alguna acción, en la columna **Resultado**, seleccione **Erróneo** para que aparezca una explicación del error.
    b. Si lo desea, haga clic en **Guardar informe** para guardar los resultados en un archivo XML.

15. Haga clic en **Cerrar** para salir del Asistente para la implementación de Integration Services.

Si se realiza correctamente la implementación del proyecto sin errores, puede seleccionar todos los paquetes que contiene para ejecutarlos en la instancia de Azure-SSIS IR.

## <a name="next-steps"></a>Pasos siguientes

* Revise las directrices de migración en la [Guía de migración de bases de datos](https://datamigration.microsoft.com/).