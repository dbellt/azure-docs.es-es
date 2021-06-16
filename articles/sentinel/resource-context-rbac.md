---
title: Administración del acceso a los datos de Azure Sentinel por recurso | Microsoft Docs
description: En este artículo se explica cómo administrar el acceso a los datos de Azure Sentinel por los recursos a los que puede acceder un usuario. La administración del acceso por recurso permite proporcionar acceso solo a datos específicos, sin la experiencia completa de Azure Sentinel. Este método también se conoce como RBAC de contexto de recursos.
services: sentinel
cloud: na
documentationcenter: na
author: batamig
manager: rkarlin
ms.assetid: ''
ms.service: azure-sentinel
ms.subservice: azure-sentinel
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/03/2021
ms.author: bagol
ms.openlocfilehash: 1c4a32e79f54bd2626f2eb4f3a3cc97266d3bab0
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111958500"
---
# <a name="manage-access-to-azure-sentinel-data-by-resource"></a>Administración del acceso a los datos de Azure Sentinel por recurso

Normalmente, los usuarios que tienen acceso a un área de trabajo de Azure Sentinel también tienen acceso a todos los datos del área de trabajo, incluido el contenido de seguridad. Los administradores pueden usar los [roles de Azure](roles.md) para configurar el acceso a características específicas de Azure Sentinel, en función de los requisitos de acceso de su equipo.

Sin embargo, es posible que algunos usuarios necesiten acceder solo a datos específicos del área de trabajo de Azure Sentinel, pero no deban tener acceso a todo el entorno de Azure Sentinel. Por ejemplo, puede que quiera proporcionar a un equipo de operaciones que no son de seguridad (no SOC) el acceso a los datos de eventos de Windows de los servidores que son de su propiedad.

En tales casos, se recomienda configurar el control de acceso basado en rol (RBAC) en función de los recursos que se permiten a los usuarios, en lugar de proporcionarles acceso al área de trabajo de Azure Sentinel o a características específicas de Azure Sentinel. Este método también se conoce como configuración de **RBAC de contexto de recursos**.

Cuando los usuarios tienen acceso a los datos de Azure Sentinel a través de los recursos a los que pueden acceder en lugar del área de trabajo de Azure Sentinel, pueden ver los registros y los libros mediante los siguientes métodos:

- **A través del propio recurso**, como una máquina virtual de Azure. Utilice este método para ver los registros y libros solo para un recurso específico.

- **A través de Azure Monitor**. Utilice este método cuando desee crear consultas que abarquen varios recursos o grupos de recursos. Al navegar a los registros y libros en Azure Monitor, defina el ámbito en uno o varios grupos de recursos o recursos específicos.

Habilite RBAC de contexto de recursos en Azure Monitor. Para más información, consulte [Administración del acceso a los datos de registro y las áreas de trabajo en Azure Monitor](../azure-monitor/logs/manage-access.md).

> [!NOTE]
> Si los datos no son un recurso de Azure, como los datos de Syslog, CEF o AAD, o los datos recopilados por un recopilador personalizado, deberá configurar manualmente el id. de recurso que se usa para identificar los datos y habilitar el acceso. Para más información, consulte [Configuración explícita de RBAC de contexto de recursos](#explicitly-configure-resource-context-rbac).
>
> Además, las [funciones y](../azure-monitor/logs/functions.md) las búsquedas guardadas no se admiten en contextos centrados en recursos. Por lo tanto, las características de Azure Sentinel como el análisis y la [normalización](normalization.md) no se admiten para RBAC del contexto de recursos en Azure Sentinel.
> 

## <a name="scenarios-for-resource-context-rbac"></a>Escenarios de RBAC de contexto de recursos

En la tabla siguiente se resaltan los escenarios en los que el RBAC de contexto de recursos es más útil. Tenga en cuenta las diferencias en los requisitos de acceso entre los equipos de SOC y los equipos que no son de SOC.

| Tipo de requisito |Equipo de SOC  |Equipo que no es de SOC  |
|---------|---------|---------|
|**Permisos**     | Toda la área de trabajo        |   Solo recursos específicos      |
|**Acceso a datos**     |  Todos los datos del área de trabajo       | Solo los datos de los recursos a los que el equipo está autorizado a acceder        |
|**Experiencia**     |  La experiencia completa de Azure Sentinel, posiblemente limitada por los [permisos funcionales](roles.md) asignados al usuario.       |  Solo consultas de registro y libros       |
|     |         |         |

Si el equipo tiene requisitos de acceso similares al equipo que no es de SOC descrito en la tabla anterior, el RBAC de contexto de recursos puede ser una buena solución para su organización.

## <a name="alternative-methods-for-implementing-resource-context-rbac"></a>Métodos alternativos para implementar RBAC de contexto de recursos

En función de los permisos necesarios en su organización, es posible que el uso de RBAC de contexto de recursos no proporcione una solución completa.

En la lista siguiente se describen los escenarios en los que otras soluciones para el acceso a datos pueden ajustarse mejor a sus requisitos:

|Escenario  |Solución  |
|---------|---------|
|**Una subsidiaria tiene un equipo de SOC que requiere una experiencia completa de Azure Sentinel**.     |  En este caso, use una arquitectura de varias áreas de trabajo para separar los permisos de los datos. <br><br>Para más información, consulte: <br>- [Extensión de Azure Sentinel entre áreas de trabajo e inquilinos](extend-sentinel-across-workspaces-tenants.md)<br>    - [Procesamiento de incidentes en varias áreas de trabajo a la vez](multiple-workspace-view.md)          |
|**Quiere proporcionar acceso a un tipo de evento específico**.     |  Por ejemplo, proporcione a un administrador de Windows el acceso a los eventos de seguridad de Windows en todos los sistemas. <br><br>En tales casos, use [RBAC de nivel de tabla](https://techcommunity.microsoft.com/t5/azure-sentinel/table-level-rbac-in-azure-sentinel/ba-p/965043) para definir los permisos de cada tabla.       |
| **Limitación del acceso a un nivel más granular, ya sea no basado en el recurso o a solo un subconjunto de los campos de un evento.**   |   Por ejemplo, puede que quiera limitar el acceso a los registros de Office 365 en función de la subsidiaria de un usuario. <br><br>En este caso, proporcione acceso a los datos mediante la integración incorporada con [Paneles e informes de Power BI](../azure-monitor/visualize/powerbi.md).      |
| | |

## <a name="explicitly-configure-resource-context-rbac"></a>Configuración explícita de RBAC de contexto de recursos

Siga estos pasos si desea configurar RBAC de contexto de recursos, pero los datos no son un recurso de Azure.

Por ejemplo, los datos de su área de trabajo de Azure Sentinel que no son recursos de Azure son los datos de Syslog, CEF o AAD, o los datos recopilados por un recopilador personalizado.

**Para configurar explícitamente RBAC de contexto de recursos**:

1. Asegúrese de que ha [habilitado RBAC de contexto de recursos](../azure-monitor/logs/manage-access.md) en Azure Monitor. 

1. [Cree un grupo de recursos](../azure-resource-manager/management/manage-resource-groups-portal.md) para cada equipo de usuarios que necesite acceder a los recursos sin todo el entorno de Azure Sentinel.

    Asigne [permisos de lector de registro](../azure-monitor/logs/manage-access.md#resource-permissions) para cada uno de los miembros del equipo.

1. Asigne recursos a los grupos de equipos de recursos creados y etiquete los eventos con los id. de recursos correspondientes.

    Cuando los recursos de Azure envían datos a Azure Sentinel, las entradas de registro se etiquetan automáticamente con el id. de recurso del origen de datos.

    > [!TIP]
    > Se recomienda agrupar los recursos a los que va a conceder acceso en un grupo de recursos específico creado para dicho fin.
    >
    > Si no es posible, asegúrese de que el equipo tenga permisos de lector de registro directamente para los recursos a los que quiere que tengan acceso.
    >

    Para más información sobre los id. de recurso, vea:

    - [Id. de recursos con reenvío de registros](#resource-ids-with-log-forwarding)
    - [Id. de recursos con recopilación Logstash](#resource-ids-with-logstash-collection)
    - [Id. de recursos con recopilación de API de Log Analytics](#resource-ids-with-the-log-analytics-api-collection)

### <a name="resource-ids-with-log-forwarding"></a>Id. de recursos con reenvío de registros

Cuando los eventos se recopilan con el [formato de evento común (CEF)](connect-common-event-format.md) o [Syslog](connect-syslog.md), el reenvío de registros se usa para recopilar eventos de varios sistemas de origen.

Por ejemplo, cuando una máquina virtual de reenvío de CEF o Syslog escucha los orígenes que envían eventos de Syslog y los reenvía a Azure Sentinel, el id. de recurso de la máquina virtual de reenvío de registros se asigna a todos los eventos que reenvían.

Si tiene varios equipos, asegúrese de que hay máquinas virtuales de reenvío de registros independientes que procesan los eventos para cada equipo.

Por ejemplo, la separación de las máquinas virtuales garantiza que los eventos de Syslog que pertenecen al equipo A se recopilan con la máquina virtual A del recopilador.

> [!TIP]
> - Cuando use una máquina virtual local u otra máquina virtual en la nube, como AWS, como reenviador de registros, asegúrese de que tiene un id. de recurso mediante la implementación de [Azure Arc](../azure-arc/servers/overview.md).
> - Para escalar el entorno de la máquina virtual de reenvío de registros, considere la posibilidad de crear un [conjunto de escalado de máquina virtual](https://techcommunity.microsoft.com/t5/azure-sentinel/scaling-up-syslog-cef-collection/ba-p/1185854) para recopilar los registros de CEF y Sylog.


### <a name="resource-ids-with-logstash-collection"></a>Id. de recursos con recopilación Logstash

Si va a recopilar los datos mediante el [complemento de salida Logstash](connect-logstash.md) de Azure Sentinel, use el campo **azure_resource_id** para configurar el recopilador personalizado a fin de incluir el id. de recurso en la salida.

Si usa RBAC de contexto de recursos y quiere que los eventos recopilados por la API estén disponibles para usuarios específicos, use el id. de recurso del grupo de recursos que [creó para los usuarios](#explicitly-configure-resource-context-rbac).

Por ejemplo, el código siguiente muestra un archivo de configuración Logstash de muestra:

``` ruby
 input {
     beats {
         port => "5044"
     }
 }
 filter {
 }
 output {
     microsoft-logstash-output-azure-loganalytics {
       workspace_id => "4g5tad2b-a4u4-147v-a4r7-23148a5f2c21" # <your workspace id>
       workspace_key => "u/saRtY0JGHJ4Ce93g5WQ3Lk50ZnZ8ugfd74nk78RPLPP/KgfnjU5478Ndh64sNfdrsMni975HJP6lp==" # <your workspace key>
       custom_log_table_name => "tableName"
       azure_resource_id => "/subscriptions/wvvu95a2-99u4-uanb-hlbg-2vatvgqtyk7b/resourceGroups/contosotest" # <your resource ID>   
     }
 }
```

> [!TIP]
> Puede que quiera agregar varias secciones `output` para diferenciar las etiquetas aplicadas a distintos eventos.
>
### <a name="resource-ids-with-the-log-analytics-api-collection"></a>Id. de recursos con recopilación de API de Log Analytics

Al realizar la recopilación mediante la [API del recopilador de datos de Log Analytics](../azure-monitor/logs/data-collector-api.md), puede asignar a los eventos un id. de recurso mediante el encabezado de solicitud HTTP [*x-ms-AzureResourceId*](../azure-monitor/logs/data-collector-api.md#request-headers).

Si usa RBAC de contexto de recursos y quiere que los eventos recopilados por la API estén disponibles para usuarios específicos, use el id. de recurso del grupo de recursos que [creó para los usuarios](#explicitly-configure-resource-context-rbac).


## <a name="next-steps"></a>Pasos siguientes

Para obtener más información, vea [Permisos de Azure Sentinel](roles.md).