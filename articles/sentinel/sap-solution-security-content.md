---
title: 'Solución Azure Sentinel para SAP: referencia de contenido de seguridad | Microsoft Docs'
description: Obtenga información sobre el contenido de seguridad integrado que proporciona la solución Azure Sentinel para SAP.
author: batamig
ms.author: bagold
ms.service: azure-sentinel
ms.topic: reference
ms.custom: mvc
ms.date: 05/12/2021
ms.subservice: azure-sentinel
ms.openlocfilehash: dc5aa5da39e25a8db899b84a0410f07c480ee3e1
ms.sourcegitcommit: 0ce834cd348bb8b28a5f7f612c2807084cde8e8f
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109815281"
---
# <a name="azure-sentinel-sap-solution-security-content-reference-public-preview"></a>Solución Azure Sentinel para SAP: referencia de contenido de seguridad (versión preliminar pública)

En este artículo se detalla el contenido de seguridad disponible para [la solución Azure Sentinel para SAP](sap-deploy-solution.md#deploy-sap-security-content).

El contenido de seguridad disponible incluye un libro integrado y reglas de análisis integradas. También puede agregar [listas de seguimiento](watchlists.md) relacionadas con SAP para usarlas en búsquedas, reglas de detección, búsqueda de amenazas y cuadernos de estrategias de respuesta.

> [!IMPORTANT]
> La solución Azure Sentinel para SAP está actualmente en VERSIÓN PRELIMINAR. En la página [Términos de uso complementarios para las Versiones preliminares de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) se incluyen términos legales adicionales que se aplican a las características de Azure que se encuentran en versión beta, versión preliminar o que todavía no se han publicado para su disponibilidad general.
>


## <a name="sap---system-applications-and-products-workbook"></a>Libro SAP: aplicaciones y productos del sistema

El libro [SAP: aplicaciones y productos del sistema](sap-deploy-solution.md#deploy-sap-security-content) se usa para visualizar y supervisar los datos ingeridos a través del conector de datos de SAP.

Por ejemplo:

:::image type="content" source="media/sap/sap-workbook.png" alt-text="Libro SAP: aplicaciones y productos del sistema.":::

Para más información, vea [Tutorial: Visualización y supervisión de los datos](tutorial-monitor-your-data.md).

## <a name="built-in-analytics-rules"></a>Reglas de análisis integradas

En las tablas siguientes se muestran las [reglas de análisis](sap-deploy-solution.md#deploy-sap-security-content) integradas que se incluyen en la solución Azure Sentinel para SAP, implementada desde el marketplace de soluciones de Azure Sentinel.

### <a name="high-level-built-in-sap-solution-analytics-rules"></a>Reglas de análisis de soluciones para SAP integradas de alto nivel

|Nombre de la regla  |Descripción  |Acción de origen  |Tácticas  |
|---------|---------|---------|---------|
|**SAP- Alta - Cambio en usuario con privilegios confidenciales**     |  Identifica los cambios de los usuarios con privilegios confidenciales.    <br> <br>Mantener a los usuarios con privilegios en la lista de reproducción [SAP: usuarios con privilegios](#users). |   Cambiar los detalles o las autorizaciones de los usuarios mediante `SU01`. <br><br>**Orígenes de datos:** SAPcon: registro de auditoría     |  Elevación de privilegios, acceso a credenciales       |
|**SAP- Alta - Cambio de configuración de cliente**     |     Identifica los cambios en la configuración del cliente, como el rol de cliente o el modo de grabación de cambios.    |  Realizar cambios de configuración del cliente mediante el código de transacción `SCC4`. <br><br>**Orígenes de datos:** SAPcon: registro de auditoría  |    Evasión de defensa, filtración, persistencia     |
|**SAP- Alta - Los datos han cambiado durante la actividad de depuración**     | Identifica los cambios que se han producido en los datos del runtime durante una actividad de depuración.         | 1. Activar la depuración ("/h"). <br>2. Seleccionar el campo que se va a cambiar y actualizar su valor.<br><br>**Orígenes de datos:** SAPcon: registro de auditoría        |  Ejecución, desplazamiento lateral       |
|**SAP- Alta - Desactivación del registro de auditoría de seguridad**     | Identifica la desactivación del registro de auditoría de seguridad.        |  Deshabilitar el registro de auditoría de seguridad mediante `SM19/RSAU_CONFIG`. <br><br>**Orígenes de datos:** SAPcon: registro de auditoría |   Filtración, evasión de defensa, persistencia      |
|**SAP- Alta - Ejecución de un programa de ABAP confidencial**     |Identifica la ejecución directa de un programa de ABAP confidencial. <br><br>Mantener los programas de ABAP en la lista de reproducción [SAP: programas de ABAP confidenciales](#programs).        | Ejecutar un programa directamente mediante `SE38`/`SA38`/`SE80`. <br> <br>**Orígenes de datos:** SAPcon: registro de auditoría      |     Filtración, desplazamiento lateral, ejecución    |
|**SAP- Alta - Ejecución de un código de transacción confidencial**     | Identifica la ejecución de un código de transacción confidencial. <br><br>Mantener los códigos de transacción en la lista de reproducción [SAP : códigos de transacción confidenciales](#transactions).       |  Ejecutar un código de transacción confidencial. <br><br>**Orígenes de datos:** SAPcon: registro de auditoría      |    Detección, ejecución     |
|**SAP- Alta - Módulo de funciones probado**     |  Identifica las pruebas de un módulo de función.       |   Probar un módulo de funciones mediante `SE37` / `SE80`.  <br><br>**Orígenes de datos**: SAPcon: registro de auditoría    |   Colección, evasión de defensa, desplazamiento lateral      |
|**SAP - Alta - HANA DB - Asignación de autorizaciones de administrador**     |   Identifica la asignación de roles o privilegios administrativos.      |  Asignar a un usuario cualquier rol o privilegio administrativos.  <br><br>**Orígenes de datos:** Agente de Linux: Syslog     | Elevación de privilegios        |
|**SAP - Alta - HANA DB - Cambios en la directiva de registro de auditoría**     |  Identifica los cambios en las directivas de registro de auditoría de HANA DB.       |     Crear o actualizar la directiva de auditoría existente en las definiciones de seguridad. <br> <br>**Orígenes de datos:** Agente de Linux: Syslog    |  Desplazamiento lateral, evasión de la defensa, persistencia       |
|**SAP - Alta - HANA DB - Desactivación del registro de auditoría**     |    Identifica la desactivación del registro de auditoría de HANA DB.     |    Desactive el registro de auditoría en la definición de seguridad de HANA DB. <br><br>**Orígenes de datos**: agente de Linux: Syslog     |  Persistencia, desplazamiento lateral, evasión de la defensa       |
|  **SAP - Alta - HANA DB - Acciones de administración de usuarios**   | Identifica las acciones de administración de usuarios.        |  Crear, actualizar o eliminar un usuario de base de datos. <br><br>**Orígenes de datos**: agente de Linux: Syslog*       |Elevación de privilegios         |
|**SAP- Alta - Inicio de sesión desde una red inesperada**     |   Identifica un inicio de sesión desde una red inesperada. <br><br>Mantener las redes en la lista de control [SAP: redes](#networks).    |    Iniciar sesión en el sistema de back-end desde una dirección IP que no esté asignada a una de las redes. <br><br>**Orígenes de datos**: SAPcon: registro de auditoría    |   Acceso inicial      |
|**SAP- Alta - Ejecución RFC de un módulo de funciones confidenciales**     | Modelos de funciones confidenciales que se usarán en las detecciones pertinentes.    <br><br>Mantener los módulos de funciones de la lista de reproducción [SAP: módulos de funciones confidenciales](#module).   |      Ejecutar un módulo de funciones mediante RFC.  <br><br>**Orígenes de datos**: SAPcon: registro de auditoría |  Ejecución, desplazamiento lateral, detección       |
|**SAP- Alta - Usuario con privilegios confidenciales que inició sesión**     |    Identifica el cuadro de diálogo de inicio de sesión de un usuario con privilegios confidenciales. <br><br>Mantener a los usuarios con privilegios en la lista de reproducción [SAP: usuarios con privilegios](#users).    |  Iniciar sesión en el sistema back-end mediante `SAP*` u otro usuario con privilegios.  <br><br>**Orígenes de datos**: SAPcon: registro de auditoría     |   acceso inicial, acceso a credenciales      |
|  **SAP- Alta - Un usuario con privilegios confidenciales realiza un cambio en otro usuario**   |   Identifica los cambios de usuarios confidenciales con privilegios en otros usuarios.       | Cambiar los detalles o las autorizaciones de los usuarios mediante SU01.  <br><br>**Orígenes de datos**: SAPcon: registro de auditoría     |   Elevación de privilegios, acceso a credenciales       |
|**SAP- Alta - Cambio de configuración del sistema**     | Identifica los cambios en la configuración del sistema.        |   Adapte las opciones de cambio del sistema o la modificación de componentes de software mediante el código de transacción `SE06`.<br><br>**Orígenes de datos**: SAPcon: registro de auditoría |Filtración, evasión de defensa, persistencia   |
|     |         |         |         |

### <a name="medium-level-built-in-sap-solution-analytics-rules"></a>Reglas de análisis de soluciones para SAP integradas de nivel medio

|Nombre de la regla  |Descripción  |Acción de origen  |Tácticas  |
|---------|---------|---------|---------|
|**SAP- Media - Asignación de un perfil confidencial**     |  Identifica nuevas asignaciones de un perfil confidencial a un usuario. <br><br>Mantener los perfiles confidenciales de la lista de reproducción [SAP: perfiles confidenciales](#profiles).      |    Asigne un perfil a un usuario mediante `SU01`. <br><br>**Orígenes de datos**: SAPcon: cambiar registro de documentos    |  Elevación de privilegios       |
|**SAP- Media - Asignación de un rol confidencial**     |    Identifica nuevas asignaciones para un rol confidencial para un usuario.     <br><br>Mantener los roles confidenciales de la lista de reproducción [SAP: roles confidenciales](#roles).|  Asignar un rol a un usuario mediante `SU01` / `PFCG`. <br><br>**Orígenes de datos**: SAPcon: cambiar registro de documentos y registro de auditoría     |   Elevación de privilegios      |
|**SAP- Media - Ataques por fuerza bruta**     |     Identifica los ataques por fuerza bruta en el sistema SAP, según los intentos de inicio de sesión que no se han podido realizar en el sistema back-end.    |   Intentar iniciar sesión desde la misma dirección IP en varios sistemas o clientes dentro del intervalo de tiempo programado. <br><br>**Orígenes de datos**: SAPcon: registro de auditoría      | Acceso con credenciales        |
|**SAP - Media - Asignación de autorizaciones críticas: nuevo valor de autorización**     | Identifica la asignación de un valor de un objeto de autorización crítica a un nuevo usuario.  <br><br>Mantener objetos de autorizaciones críticas de la lista de reproducción [SAP: objetos de autorizaciones críticas](#objects).      |  Asignar un objeto de autorización nuevo o actualizar uno existente en un rol mediante `PFCG`. <br><br>**Orígenes de datos**: SAPcon: cambiar registro de documentos      |     Elevación de privilegios    |
|**SAP - Media - Asignación de autorizaciones críticas: nueva asignación de usuario**     |  Identifica la asignación de un valor de un objeto de autorización crítica a un nuevo usuario. <br><br>Mantener objetos de autorizaciones críticas de la lista de reproducción [SAP: objetos de autorizaciones críticas](#objects).    |    Asigne un nuevo usuario a un rol que contenga valores de autorizaciones críticas mediante `SU01`/`PFCG`. <br><br>**Orígenes de datos**: SAPcon: cambiar registro de documentos    |  Elevación de privilegios       |
|**SAP - Media - Actividades de depuración**     |  Identifica todas las actividades relacionadas con la depuración.       |Activar Debug ("/h") en el sistema, depurar un proceso activo, agregar un punto de interrupción al código fuente, etc. <br><br>**Orígenes de datos**: SAPcon: registro de auditoría |   Detección      |
|**SAP - Media - Varios inicios de sesión por IP**     |  Identifica el inicio de sesión de varios usuarios desde la misma dirección IP en un intervalo de tiempo programado.       |    Iniciar sesión con varios usuarios desde la misma dirección IP. <br><br>**Orígenes de datos**: SAPcon: registro de auditoría | Acceso inicial        |
|**SAP - Media - Varios inicios de sesión por usuario**     | Identifica los inicios de sesión del mismo usuario desde varios terminales en un intervalo de tiempo programado.  <br><br>Solo está disponible a través del método Audit SAL, para las versiones 7.5 y posteriores de SAP.      |   Iniciar sesión con el mismo usuario con diferentes direcciones IP.   <br><br>**Orígenes de datos**: SAPcon: registro de auditoría   |  PreAttack, acceso con credenciales, acceso inicial, colección       |
|**SAP - Media - Cambio de configuración del registro de auditoría de seguridad**     |  Identifica los cambios en la configuración del registro de auditoría de seguridad.       |   Cambiar cualquier configuración de registro de auditoría de seguridad mediante `SM19`/`RSAU_CONFIG`, como los filtros, el estado, el modo de grabación, etc. <br><br>**Orígenes de datos**: SAPcon: registro de auditoría      |    Persistencia, filtración, evasión de defensa     |
|**SAP- Media - La transacción está desbloqueada**     |Identifica el desbloqueo de una transacción.         |     Desbloquear un código de transacción mediante `SM01`/`SM01_DEV`/`SM01_CUS`. <br><br>**Orígenes de datos**: SAPcon: registro de auditoría   |  Persistencia, ejecución       |
|     |         |         |         |

### <a name="low-level-built-in-sap-solution-analytics-rules"></a>Reglas de análisis de soluciones para SAP integradas de bajo nivel

|Nombre de la regla  |Descripción  |Acción de origen  |Tácticas  |
|---------|---------|---------|---------|
|**SAP - Baja - Varios cambios de contraseña por usuario**     |   Identifica varios cambios de contraseña por usuario.      |   Cambiar la contraseña de usuario <br><br>**Orígenes de datos**: SAPcon: registro de auditoría      |    Acceso con credenciales     |
|**SAP - Baja - Acceso directo a tablas confidenciales mediante inicio de sesión a través de cuadro de diálogo**     |   Identifica el acceso a tablas genéricas a través del inicio de sesión mediante cuadro de diálogo.      |  Abrir el contenido de las tablas mediante `SE11`/`SE16`/`SE16N`. <br><br>**Orígenes de datos**: SAPcon: registro de auditoría      |    Detección     |
|     |         |         |         |

## <a name="available-watchlists"></a>Listas de reproducción disponibles

En la tabla siguiente se enumeran las [listas de reproducción](sap-deploy-solution.md#deploy-sap-security-content) disponibles para la solución Azure Sentinel para SAP y los campos de cada lista.

Estas listas de reproducción proporcionan la configuración de la solución de supervisión continua de amenazas de Azure Sentinel para SAP y se puede acceder a ellas en el repositorio de GitHub de Azure Sentinel en https://github.com/Azure/Azure-Sentinel/tree/master/Solutions/SAP/Analytics/Watchlists.


|Nombre de lista de reproducción  |Descripción y campos  |
|---------|---------|
|<a name="objects"></a>**SAP: objetos Autorizaciones críticas**     |  Objeto Autorizaciones críticas, en el que se deben regular las asignaciones.     <br><br>- **AuthorizationObject**: un objeto de autorización de SAP, como `S_DEVELOP`, `S_TCODE` o `Table TOBJ`. <br>- **AuthorizationField**: un campo de autorización de SAP, como `OBJTYP` o `TCD`.    <br>- **AuthorizationValue**: un valor de un campo de autorización de SAP, como `DEBUG`.       <br>- **ActivityField**: campo de actividad de SAP. En la mayoría de los casos, este valor será `ACTVT`. Para objetos de autorización sin campo **Activity** (Actividad) o solo con un campo **Activity**, rellenado con `NOT_IN_USE`.        <br>- **Actividad**: actividad de SAP, según el objeto de autorización, como: `01`: Crear; `02`: Cambiar; `03`: Mostrar, etc.      <br>-  **Descripción**: una descripción significativa del objeto de autorización crítico. |
|**SAP: redes excluidas** | Para el mantenimiento interno de redes excluidas, como omitir instancias de Web Dispatcher, servidores de terminal, etc. <br><br>-**Red**: una dirección IP de red o un intervalo, como `111.68.128.0/17`. <br>-**Descripción**: una descripción de red significativa.|
|**Usuarios excluidos de SAP** |Los usuarios del sistema que han iniciado sesión en el sistema y deben ignorarse. Por ejemplo, alertas de varios inicios de sesión por parte del mismo usuario. <br><br>- **Usuario**: usuario de SAP <br>-**Descripción**: una descripción del usuario significativa. |
|<a name="networks"></a>**SAP: redes**     |  Redes internas y de mantenimiento para la identificación de inicios de sesión no autorizados.      <br><br>- **Red**: dirección IP o rango de direcciones IP de red, como `111.68.128.0/17`.     <br>- **Descripción**: una descripción de red significativa.|
|<a name="users"></a>**SAP: usuarios con privilegios**.     |   Usuarios con privilegios que tienen restricciones adicionales.  <br><br>- **Usuario**: el usuario de ABAP, como `DDIC` o `SAP`. <br>- **Descripción**: una descripción del usuario significativa. |
|<a name= "programs"></a>**SAP: programas de ABAP confidenciales**     |      Programas de ABAP confidenciales (informes), donde se debe regular la ejecución.   <br><br>- **ABAPProgram**: programa o informe de ABAP, como `RSPFLDOC`     <br>- **Descripción:** una descripción significativa del programa.|
|<a name="module"></a>**SAP: módulo de funciones confidenciales**     |   Redes internas y de mantenimiento para la identificación de inicios de sesión no autorizados.      <br><br>- **FunctionModule**: un módulo de funciones de ABAP, como `RSAU_CLEAR_AUDIT_LOG`       <br>- **Descripción**: una descripción significativa del módulo.     |
|<a name="profiles"></a>**SAP: perfiles confidenciales**     |  Perfiles confidenciales, donde se deben regular las asignaciones.     <br><br>- **Perfil**: perfil de autorización de SAP, como `SAP_ALL` o `SAP_NEW`      <br>- **Descripción:** una descripción significativa del perfil.|
|<a name="tables"></a>**SAP: tablas confidenciales**     |  Tablas confidenciales, donde se debe regular el acceso.  <br><br>- **Tabla**: tabla de diccionarios de ABAP, como `USR02` o `PA008` <br>- **Descripción**: una descripción significativa de la tabla. |
|<a name="roles"></a>**SAP: roles confidenciales**     |  Roles confidenciales, donde se debe regular la asignación.    <br><br>- **Rol**: rol de autorización de SAP, como `SAP_BC_BASIS_ADMIN`  <br>- **Descripción**: una descripción significativa del rol. |
|<a name="transactions"></a>**SAP: transacciones confidenciales**     |     Transacciones confidenciales en las que se debe regular la ejecución.  <br><br>- **TransactionCode**: código de transacción de SAP, como `RZ11` <br>- **Descripción**: una descripción significativa del código. |
|<a name="systems"></a>**SAP: sistemas**     |    Describe el panorama de los sistemas SAP en función del rol y del uso.<br><br>- **SystemID**: el identificador del sistema de SAP (SYSID) <br>- **SystemRole**: el rol de sistema SAP, uno de los siguientes valores: `Sandbox`, `Development`, `Quality Assurance`, `Training`, `Production` <br>- **SystemUsage**: el uso del sistema SAP, uno de los siguientes valores: `ERP`, `BW`, `Solman`, `Gateway`, `Enterprise Portal`        |
| | |


## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte:

- [Tutorial: Implementación de la solución Azure Sentinel para SAP](sap-deploy-solution.md)
- [Referencia sobre los registros de la solución Azure Sentinel para SAP](sap-solution-log-reference.md)
- [Implementación del conector de datos de SAP de Azure Sentinel en el entorno local](sap-solution-deploy-alternate.md)
- [Requisitos detallados de SAP para la solución Azure Sentinel para SAP](sap-solution-detailed-requirements.md)