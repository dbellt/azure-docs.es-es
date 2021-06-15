---
title: 'Ejemplo de aplicación multiinquilino: SaaS de Wingtip'
description: Proporciona pasos e instrucciones para instalar y ejecutar la aplicación SaaS multiinquilino de ejemplo, Wingtip Tickets, que utiliza Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: scenario
ms.custom: seo-lt-2019, sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: MashaMSFT
ms.author: mathoma
ms.reviewer: ''
ms.date: 12/18/2018
ms.openlocfilehash: 6360666b0b5d5b6aaaba5173e766e50bc73f30ad
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110693378"
---
# <a name="general-guidance-for-working-with-wingtip-tickets-sample-saas-apps"></a>Instrucciones generales para trabajar con aplicaciones SaaS de ejemplo de vales de Wingtip
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Este artículo contiene las instrucciones generales para ejecutar la aplicación SaaS de ejemplo Wingtip Tickets, que usa Azure SQL Database.

## <a name="download-and-unblock-the-wingtip-tickets-saas-scripts"></a>Descargar y desbloquear los scripts SaaS de vales de Wingtip

Es posible que Windows bloquee el contenido ejecutable (scripts, archivos DLL) cuando se descarguen y extraigan archivos ZIP desde un origen externo. Al extraer los scripts de un archivo ZIP, **siga los pasos siguientes para desbloquear el archivo .zip antes de extraerlo**. Esto garantiza que se puedan ejecutar los scripts.

1. Navegue al repositorio de GitHub de SaaS de vales de Wingtip y busque el patrón de inquilino de base de datos que quiere explorar:
    - [WingtipTicketsSaaS-StandaloneApp](https://github.com/Microsoft/WingtipTicketsSaaS-StandaloneApp)
    - [WingtipTicketsSaaS-DbPerTenant](https://github.com/Microsoft/WingtipTicketsSaaS-DbPerTenant)
    - [WingtipTicketsSaaS-MultiTenantDb](https://github.com/Microsoft/WingtipTicketsSaaS-MultiTenantDb)
2. Haga clic en **Clone or download** (Clonar o descargar).
3. Haga clic en **Download ZIP** (Descargar archivo ZIP) y guarde el archivo.
4. Haga clic con el botón derecho en el archivo ZIP y seleccione **Propiedades**. El nombre del archivo ZIP se corresponderá con el nombre de repositorio. (Ejemplo: _WingtipTicketsSaaS-DbPerTenant-master.zip_)
5. En la pestaña **General**, seleccione **Desbloquear**.
6. Haga clic en **OK**.
7. Extraiga los archivos.

Los scripts se encuentran en la carpeta *..\\Learning Modules* (módulos de aprendizaje).


## <a name="working-with-the-wingtip-tickets-powershell-scripts"></a>Trabajar con los scripts de PowerShell de vales de Wingtip

Para sacar el máximo partido del ejemplo debe profundizar en los scripts proporcionados. Use puntos de interrupción y recorra los scripts a medida que se ejecutan, y examine los detalles de cómo se implementan los distintos patrones de SaaS. Para recorrer con facilidad los scripts proporcionados y los módulos para entenderlos mejor, se recomienda usar [PowerShell ISE](/powershell/scripting/components/ise/introducing-the-windows-powershell-ise).

### <a name="update-the-configuration-file-for-your-deployment"></a>Actualizar el archivo de configuración para la implementación

Edite el archivo **UserConfig.psm1** con el valor de usuario y grupo de recursos que se estableció durante la implementación:

1. Abra *PowerShell ISE* y cargue ...\\Learning Modules\\*UserConfig.psm1*
2. Actualice *ResourceGroupName* y *Name* con los valores específicos para la implementación (solo en las líneas 10 y 11).
3. Guarde los cambios.

Al establecer estos valores aquí simplemente evita tener que actualizar estos valores específicos de la implementación en todos los scripts.

### <a name="execute-the-scripts-by-pressing-f5"></a>Ejecutar los scripts al presionar F5

Varios scripts usan *$PSScriptRoot* para la navegación por las carpetas y *$PSScriptRoot* solo se evalúa cuando se ejecutan los scripts presionando **F5**.    Resaltar y ejecutar una selección (**F8**) puede dar lugar a errores, por tanto, presione **F5** al ejecutar los scripts.

### <a name="step-through-the-scripts-to-examine-the-implementation"></a>Siga paso a paso los scripts para examinar la implementación

La mejor manera de comprender los scripts es recorrerlos para ver lo que hacen. Consulte los scripts **Demo-** incluidos que presentan un flujo de trabajo general fácil de seguir. Los scripts **Demo-** muestran los pasos necesarios para realizar cada tarea, por tanto, establezca puntos de interrupción y profundice en las llamadas individuales para ver los detalles de implementación para los distintos patrones de SaaS.

Sugerencias para explorar y recorrer los scripts de PowerShell:

- Abra los scripts **Demo-** en PowerShell ISE.
- Ejecute o continúe con **F5** (no se recomienda el uso de **F8** porque *$PSScriptRoot* no se evalúa cuando se ejecutan las selecciones de un script).
- Para colocar puntos de interrupción, haga clic o seleccione una línea y presione **F9**.
- Salte una llamada de función o script con **F10**.
- Vaya a una llamada de función o script con **F11**.
- Salga de la llamada de función o script actual con **MAYÚS + F11**.


## <a name="explore-database-schema-and-execute-sql-queries-using-ssms"></a>Exploración del esquema de base de datos y ejecución de consultas SQL con SSMS

Use [SQL Server Management Studio (SSMS)](/sql/ssms/download-sql-server-management-studio-ssms) para conectarse y examinar las bases de datos y los servidores de la aplicación.

Inicialmente, la implementación tiene inquilinos y servidores del catálogo a los que conectarse. La denominación de los servidores depende del patrón de inquilino de base de datos (vea a continuación para obtener información específica).

   - **Aplicación independiente:** servidores para cada inquilino (p. ej. *servidor contosoconcerthall-&lt;Usuario&gt;* ) y *catalog-sa-&lt;Usuario&gt;*
   - **Base de datos por inquilino: servidores** *tenants1-dpt-&lt;Usuario&gt;* y *catalog-dpt-&lt;Usuario&gt;*
   - **Base de datos de varios inquilinos: servidores** *tenants1-mt-&lt;Usuario&gt;* y *catalog-mt-&lt;Usuario&gt;*

Para garantizar una conexión de demostración correcta, todos los servidores tienen una [regla de firewall](firewall-configure.md) que permite todas las direcciones IP.


1. Abra *SSMS* y conéctese a los inquilinos. El nombre del servidor depende del patrón de inquilino de base de datos que seleccionó (vea a continuación para obtener información específica).
    - **Aplicación independiente:** servidores de inquilinos individuales (p. ej. *contosoconcerthall-&lt;Usuario&gt;.database.windows.net*)
    - **Base de datos por inquilino:** *tenants1-dpt-&lt;Usuario&gt;.database.windows.net*
    - **Base de datos multiinquilino:** *tenants1-MT-&lt;Usuario&gt;.database.windows.net*
2. Haga clic en **Conectar** > **Motor de base de datos...** :

   ![Servidor de catálogo](./media/saas-tenancy-wingtip-app-guidance-tips/connect.png)

3. Las credenciales de demostración son: Inicio de sesión = *developer* y contraseña = *P\@ssword1*

    En la imagen siguiente se muestra el inicio de sesión para el patrón *Base de datos por inquilino*.
    ![connection](./media/saas-tenancy-wingtip-app-guidance-tips/tenants1-connect.png)



4. Repita los pasos 2 y 3 y conéctese al servidor del catálogo (consulte a continuación si hay nombres de servidores específicos basados en el patrón de inquilino de base de datos seleccionado).
    - **Aplicación independiente:** *catalog-sa-&lt;usuario&gt;.database.windows.net*
    - **Base de datos por inquilino:** *catalog-dpt-&lt;Usuario&gt;.database.windows.net*
    - **Base de datos multiinquilino:** *catalog-MT-&lt;Usuario&gt;.database.windows.net*


Después de conectarse correctamente, debería ver todos los servidores. La lista de bases de datos podría ser diferente, dependiendo de los inquilinos que haya aprovisionado.

En la imagen siguiente se muestra el inicio de sesión para el patrón *Base de datos por inquilino*.

![Explorador de objetos](./media/saas-tenancy-wingtip-app-guidance-tips/object-explorer.png)



## <a name="next-steps"></a>Pasos siguientes
- [Implementación de la aplicación independiente SaaS de vales de Wingtip](./saas-standaloneapp-get-started-deploy.md)
- [Implementación de la aplicación de base de datos por inquilino de SaaS de vales de Wingtip](./saas-dbpertenant-get-started-deploy.md)
- [Implementación de la aplicación de base de datos de varios inquilinos de SaaS de vales de Wingtip](./saas-multitenantdb-get-started-deploy.md)