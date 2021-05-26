---
title: Recopilación de datos e informes | Servicios de datos habilitados para Azure Arc
description: Explica el tipo de datos que transmiten los servicios de datos habilitados para Arc a Microsoft.
author: MikeRayMSFT
ms.author: mikeray
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
ms.topic: conceptual
ms.date: 04/27/2021
ms.custom: template-concept
ms.openlocfilehash: dcaf900164e3d9d4b9b3b54e27a79fdae937abc5
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110472775"
---
# <a name="azure-arc-data-services-data-collection-and-reporting"></a>Servicios de datos para Azure Arc | Informes y recopilación de datos

En este artículo se describen los datos que los servicios de datos habilitados para Azure Arc transmiten a Microsoft. 

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="related-products"></a>Productos relacionados

Los servicios de datos habilitados para Azure Arc pueden usar algunos de los productos siguientes o todos ellos:

- MI de SQL: Azure Arc 
- Hiperescala de PostgreSQL: Azure Arc
- Azure Data Studio
- CLI de Azure (az)
- CLI de datos de Azure (`azdata`) 

## <a name="directly-connected"></a>Conexión directa

Cuando un clúster está configurado para conectarse directamente a Azure, algunos datos se transmiten automáticamente a Microsoft. 

En la tabla siguiente se describe el tipo de datos, cómo se envían y los requisitos.  

|Categoría de datos|¿Qué datos se envían?|¿Cómo se envían?|¿Es necesario?
|:----|:----|:----|:----|
|Datos operativos|Métricas y registros|Automático, cuando se configura para hacerlo|No
Datos de facturación e inventario|Inventario como el número de instancias y uso, como el número de núcleos virtuales consumidos|Automático |Sí
Diagnóstico|Información de diagnóstico para solucionar problemas|Exportada y proporcionada manualmente al Soporte técnico de Microsoft|Solo para el ámbito de solución de problemas y sigue las [directivas de privacidad](https://privacy.microsoft.com/privacystatement) estándar
Programa para la mejora de la experiencia del usuario (CEIP)|[Resumen del CEIP](/sql/sql-server/usage-and-diagnostic-data-configuration-for-sql-server)|Automático, si está permitido|No

## <a name="indirectly-connected"></a>Conexión indirecta

Cuando un clúster no se configura para estar conectado directamente a Azure, no transmite automáticamente datos operativos, de facturación o de inventario a Microsoft. Para transmitir datos a Microsoft, debe configurar la exportación. 

En la tabla siguiente se describe el tipo de datos, cómo se envían y los requisitos.  

|Categoría de datos|¿Qué datos se envían?|¿Cómo se envían?|¿Es necesario?
|:----|:----|:----|:----|
|Datos operativos|Métricas y registros|Manual|No
Datos de facturación e inventario|Inventario como el número de instancias y uso, como el número de núcleos virtuales consumidos|Manual |Sí
Diagnóstico|Información de diagnóstico para solucionar problemas|Exportada y proporcionada manualmente al Soporte técnico de Microsoft|Solo para el ámbito de solución de problemas y sigue las [directivas de privacidad](https://privacy.microsoft.com/privacystatement) estándar
Programa para la mejora de la experiencia del usuario (CEIP)|[Resumen del CEIP](/sql/sql-server/usage-and-diagnostic-data-configuration-for-sql-server)|Automático, si está permitido|No

## <a name="detailed-description-of-data"></a>Descripción detallada de los datos

Esta sección proporciona más detalles sobre la información que se incluye con las transmisiones de servicios de datos habilitados de Azure Arc a Microsoft.

### <a name="operational-data"></a>Datos operativos

Los datos operativos se recopilan para todas las instancias de base de datos y para la propia plataforma de servicios de datos habilitados para Arc. Existen dos tipos de datos operativos: 

- Métricas: métricas relacionadas con el rendimiento y la capacidad, que se recopilan en una base de datos de Influx proporcionada como parte de los servicios de datos habilitados para Arc. Puede consultar estas métricas en el panel de Grafana. 

- Registros: los registros emitidos por todos los componentes, incluidos errores, advertencias y eventos informativos, se recopilan en una base de datos de Elasticsearch proporcionada como parte de los servicios de datos habilitados para Arc. Puede consultar los registros en el panel de Kibana. 

Los datos operativos que se almacenan de forma local requieren privilegios administrativos integrados para consultarlos en Grafana o Kibana. 

Los datos operativos no abandonan su entorno a menos que elija exportar o cargar (modo de conexión indirecta) o enviar automáticamente (modo de conexión directa) los datos a Azure Monitor o Log Analytics. Los datos van a un área de trabajo de Log Analytics, que el usuario controla. 

Si los datos se envían a Azure Monitor o Log Analytics, puede elegir en qué región o centro de datos de Azure reside el área de trabajo de Log Analytics. Después, el usuario puede controlar el acceso para verlo o copiarlo desde otras ubicaciones. 

### <a name="billing-and-inventory-data"></a>Datos de facturación e inventario 

Los datos de facturación se usan para hacer un seguimiento del uso facturable. Estos datos son esenciales para la ejecución del servicio y deben transmitirse de forma manual o automática en todos los modos. 

Cada instancia de base de datos y el propio controlador de datos aparecerán en Azure como un recurso de Azure en Azure Resource Manager. 

Existen tres tipos de recursos: 

- SQL Managed Instance habilitado para Arc 
- El grupo de servidores de Hiperescala de PostgreSQL habilitado para Arc 
- SQL Server habilitado para Arc 
- Controlador de datos 

En las secciones siguientes se muestran las propiedades, los tipos y las descripciones que se recopilan y almacenan de cada tipo de recurso: 

### <a name="arc-enabled-sql-server"></a>SQL Server habilitado para Arc 
- Edición de SQL Server. 
   - `string: Edition` 
- Identificador de recurso del recurso contenedor (Azure Arc para servidores). 
   - `string: ContainerResourceId` 
- Hora en la que se creó el recurso. 
   - `string: CreateTime` 
- Número de procesadores lógicos usados por la instancia de SQL Server.
   - `string: VCore` 
- Estado de conectividad de la nube. 
   - `string: Status` 
- Nivel de actualización de SQL Server. 
   - `string: PatchLevel` 
- Intercalación de SQL Server. 
   - `string: Collation`
- Versión actual de SQL Server.
   - `string: CurrentVersion`
- Nombre de instancia de SQL Server. 
   - `string: InstanceName`
- Puertos TCP dinámicos usados por SQL Server. 
   - `string: TcpDynamicPorts`
- Puertos TCP estáticos usados por SQL Server.
   - `string: TcpStaticPorts` 
- Identificador de producto de SQL Server.
   - `string: ProductId`
- Estado de aprovisionamiento de SQL Server.
   - `string: ProvisioningState`

### <a name="data-controller"></a>Controlador de datos 

- Información de la ubicación.
   - `public OnPremiseProperty OnPremiseProperty` 
- Información de Kubernetes sin procesar (`kubectl get datacontroller`) 
   - `object: K8sRaw` 
- Última fecha cargada desde el clúster local.
   - `System.DateTime: LastUploadedDate` 
- Estado del controlador de datos
   - `string: ProvisioningState` 

### <a name="postgresql-hyperscale-server-group"></a>Grupo de servidores Hiperescala de PostgreSQL 

- Identificador del controlador de datos
   - `string: DataControllerId`
- Nombre del administrador de la instancia
   - `string: Admin`
- Nombre de usuario y contraseña para autenticación básica
   - `public: BasicLoginInformation BasicLoginInformation` - Información de Kubernetes sin procesar (`kubectl get postgres12`) 
   - `object: K8sRaw` - Última fecha cargada desde el clúster local. 
   - `System.DateTime: LastUploadedDate` 
- Estado de aprovisionamiento del grupo
   - `string: ProvisioningState` 

### <a name="sql-managed-instance"></a>Instancia administrada de SQL 

- Identificador de la instancia administrada
   - `public string: DataControllerId` 
- Nombre de usuario del administrador de la instancia 
   - `string: Admin` 
- Hora de inicio de la instancia 
   - `string: StartTime`
- Hora de finalización de la instancia 
   - `string: EndTime` 
- Información de Kubernetes sin procesar (`kubectl get sqlmi`) 
   - `object: K8sRaw` 
- Nombre de usuario y contraseña para autenticación básica. 
   - `public: BasicLoginInformation BasicLoginInformation`
- Última fecha cargada desde el clúster local. 
   - `public: System.DateTime LastUploadedDate` 
- Estado de aprovisionamiento de la instancia administrada de SQL
   - `public string: ProvisioningState` 

### <a name="examples"></a>Ejemplos

Ejemplo de documento JSON de datos de inventario de recurso que se envía a Azure para crear recursos de Azure en su suscripción. 

```json
{ 

        "customObjectName": "<resource type>-2020-29-5-23-13-17-164711", 

        "uid": "4bc3dc6b-9148-4c7a-b7dc-01afc1ef5373", 

        "instanceName": "sqlInstance001", 

        "instanceNamespace": "arc", 

        "instanceType": "<resource>", 

        "location": "eastus", 

        "resourceGroupName": "production-resources", 

        "subscriptionId": "<subscription_id>", 

        "isDeleted": false, 

        "externalEndpoint": "32.191.39.83:1433", 

        "vCores": "2", 

        "createTimestamp": "05/29/2020 23:13:17", 

        "updateTimestamp": "05/29/2020 23:13:17" 

    } 
```

 

Los datos de facturación capturan la hora de inicio ("creada") y la hora de finalización ("eliminada") de una instancia determinada, así como cualquier inicio y momento en el que se produce un cambio en el número de núcleos disponibles para una instancia determinada ("límite de núcleo"). 

```json
{ 

          "requestType": "usageUpload", 

          "clusterId": "4b0917dd-e003-480e-ae74-1a8bb5e36b5d", 

          "name": "DataControllerTestName", 

          "subscriptionId": "<subscription_id>", 

          "resourceGroup": "production-resources", 

          "location": "eastus", 

          "uploadRequest": { 

            "exportType": "usages", 

            "dataTimestamp": "2020-06-17T22:32:24Z", 

            "data": "[{\"name\":\"sqlInstance001\", 

                       \"namespace\":\"arc\", 

                       \"type\":\"<resource type>\", 

                       \"eventSequence\":1,  

                       \"eventId\":\"50DF90E8-FC2C-4BBF-B245-CB20DC97FF24\", 

                       \"startTime\":\"2020-06-17T19:11:47.7533333\", 

                       \"endTime\":\"2020-06-17T19:59:00\", 

                       \"quantity\":1, 

                       \"id\":\"<subscription_id>\"}]", 

           "signature":"MIIE7gYJKoZIhvcNAQ...2xXqkK" 

```

### <a name="diagnostic-data"></a>Datos de diagnóstico

Si le atiende el soporte técnico, es posible que se le pida que proporcione registros de instancia de base de datos, registros de Kubernetes y otros registros de diagnóstico. El equipo de soporte técnico le proporcionará una ubicación segura donde podrá cargar la información. Las vistas de administración dinámica (DMV) también pueden proporcionar datos de diagnóstico. Las DMV o consultas que se usan podrían contener detalles de metadatos de esquema de base de datos, pero no suelen contener datos de cliente. Los datos de diagnóstico no contienen contraseñas, IP de clúster ni datos de identificación individual. Estos se limpian y los registros se anonimizan para que se puedan almacenar cuando sea posible. No se transmiten automáticamente y el administrador debe cargarlos manualmente. 

|Nombre del campo  |Notas  |
|---------|---------|
|Registros de error |Los archivos de registro que capturan errores pueden contener datos personales o del cliente (consulte a continuación). Están restringidos y los comparte el usuario |
|DMV      |Las vistas de administración dinámica pueden contener consultas y planes de consulta, pero están restringidas y las comparte el usuario     |
|Vistas    |Las vistas pueden contener datos de clientes pero están restringidas y solo las comparte el usuario     |
|Volcado de memoria - Datos de clientes | Retención máxima de 30 días de volcados de memoria: puede contener datos de control de acceso <br/><br/> En los volcados de memoria del cliente puede haber objetos de estadísticas, valores de datos de filas y textos de consulta    |
|Volcados de memoria - Datos personales | Los siguientes elementos requieren el consentimiento del usuario para poder incluirlos: máquina, inicios de sesión, nombres de usuario, correos electrónicos, información de ubicación, identificación del cliente  |

### <a name="customer-experience-improvement-program-ceip-telemetry"></a>Programa para la mejora de la experiencia del usuario (CEIP) (Telemetría) 

La telemetría se usa para realizar un seguimiento de las métricas de uso del producto y la información del entorno. Consulte [Complemento de privacidad de SQL Server](/sql/sql-server/sql-server-privacy/). 

## <a name="next-steps"></a>Pasos siguientes
[Carga de datos a uso en Azure Monitor](upload-usage-data.md)
