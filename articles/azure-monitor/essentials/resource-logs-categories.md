---
title: Servicios y categorías admitidos de los registros de recursos de Azure Monitor
description: Referencia de Azure Monitor para conocer el esquema de los eventos y servicios admitidos para los registros de recursos de Azure.
ms.topic: reference
ms.date: 03/30/2021
ms.openlocfilehash: b9bac212a8be040349a8d78c64d7beab291e1f9e
ms.sourcegitcommit: 2f322df43fb3854d07a69bcdf56c6b1f7e6f3333
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2021
ms.locfileid: "108016029"
---
# <a name="supported-categories-for-azure-resource-logs"></a>Categorías admitidas en los registros de recursos de Azure

> [!NOTE]
> Esta lista se genera automáticamente. Cualquier modificación realizada en esta lista a través de GitHub puede sobrescribirse sin previo aviso. Póngase en contacto con el autor de este artículo para obtener más información sobre cómo hacer actualizaciones permanentes.

Los [registros de recursos de Azure Monitor](../essentials/platform-logs-overview.md) son los registros emitidos por los servicios de Azure que describen el funcionamiento de dichos servicios o recursos. Todos los registros de recursos disponibles a través de Azure Monitor comparten un esquema común de nivel superior, con flexibilidad para que cada servicio emita propiedades únicas para sus propios eventos.

Los registros de recurso se conocían anteriormente como registros de diagnóstico. El nombre se cambió en octubre de 2019, dado que los tipos de registros recopilados por Azure Monitor evolucionaron para incluir algo más que tan solo el recurso de Azure.

Una combinación del tipo de recurso (disponible en la propiedad `resourceId`) y la `category` identifica un esquema de forma única. Hay un esquema común para todos los registros de recursos con campos específicos de servicio que se han agregado para distintas categorías de registros. Para obtener más información, consulte [Esquema específico de servicio y común para los registros de recursos de Azure]().

## <a name="costs"></a>Costos

Hay costos asociados al envío y al almacenamiento de datos en Log Analytics, Azure Storage o el centro de eventos. Puede que deba pagar el costo de obtener los datos en estas ubicaciones y de conservarlos ahí.  Los registros de recursos son un tipo de datos que se pueden enviar a estas ubicaciones. 

Hay un costo adicional para exportar algunas categorías de registros de recursos a estas ubicaciones. Dichos registros con costos de exportación se muestran en la tabla siguiente. Para más información sobre precios, consulte la sección Registros de la plataforma de la [página Precios de Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

## <a name="supported-log-categories-per-resource-type"></a>Categorías de registro admitidas por tipo de recurso

A continuación, se muestra una lista de los tipos de registros disponibles para cada tipo de recurso. 

Es posible que algunas categorías solo se admitan para tipos específicos de recursos. Consulte la documentación específica de recurso si cree que falta un recurso. Por ejemplo, las categorías Microsoft.Sql/servers/databases no están disponibles para todos los tipos de bases de datos. Para más información, consulte la [información sobre el registro de diagnóstico de SQL Database](../../azure-sql/database/metrics-diagnostic-telemetry-logging-streaming-export-configure.md). 

Si le parece que falta algo, puede escribir un comentario de GitHub en la parte inferior de este artículo.

## <a name="microsoftaaddomainservices"></a>Microsoft.AAD/DomainServices

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|AccountLogon|AccountLogon|No|
|AccountManagement|AccountManagement|No|
|DetailTracking|DetailTracking|No|
|DirectoryServiceAccess|DirectoryServiceAccess|No|
|LogonLogoff|LogonLogoff|No|
|ObjectAccess|ObjectAccess|No|
|PolicyChange|PolicyChange|No|
|PrivilegeUse|PrivilegeUse|No|
|SystemSecurity|SystemSecurity|No|


## <a name="microsoftaadiamtenants"></a>microsoft.aadiam/tenants

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|Inicio de sesión|Inicio de sesión|Sí|


## <a name="microsoftanalysisservicesservers"></a>Microsoft.AnalysisServices/servers

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|Motor|Motor|No|
|Servicio|Servicio|No|


## <a name="microsoftapimanagementservice"></a>Microsoft.ApiManagement/service

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|GatewayLogs|Registros relacionados con la puerta de enlace de ApiManagement|No|


## <a name="microsoftappconfigurationconfigurationstores"></a>Microsoft.AppConfiguration/configurationStores

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|HttpRequest|Solicitudes HTTP|Sí|


## <a name="microsoftappplatformspring"></a>Microsoft.AppPlatform/Spring

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|ApplicationConsole|Consola de aplicación|No|
|SystemLogs|Registros del sistema|No|


## <a name="microsoftattestationattestationproviders"></a>Microsoft.Attestation/attestationProviders

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|AuditEvent|Categoría de registro de mensajes AuditEvent.|No|
|ERR|Categoría de registro de mensajes de error.|No|
|INF|Categoría de registro de mensajes informativos.|No|
|WRN|Categoría de registro de mensajes de advertencia.|No|


## <a name="microsoftautomationautomationaccounts"></a>Microsoft.Automation/automationAccounts

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|DscNodeStatus|Estado del nodo de DSC|No|
|JobLogs|Registros de trabajo|No|
|JobStreams|Flujos de trabajo|No|


## <a name="microsoftautonomousdevelopmentplatformaccounts"></a>Microsoft.AutonomousDevelopmentPlatform/accounts

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|Auditoría|Auditoría|Sí|
|Operativos|Operativos|Sí|


## <a name="microsoftbatchbatchaccounts"></a>Microsoft.Batch/batchAccounts

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|ServiceLog|Registros de servicios|No|


## <a name="microsoftbatchaiworkspaces"></a>Microsoft.BatchAI/workspaces

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|BaiClusterEvent|BaiClusterEvent|No|
|BaiClusterNodeEvent|BaiClusterNodeEvent|No|
|BaiJobEvent|BaiJobEvent|No|


## <a name="microsoftblockchainblockchainmembers"></a>Microsoft.Blockchain/blockchainMembers

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|BlockchainApplication|Aplicación de cadena de bloques|No|
|FabricOrderer|Solicitante de Fabric|No|
|FabricPeer|Nodo del mismo nivel de Fabric|No|
|Proxy|Proxy|No|


## <a name="microsoftblockchaincordamembers"></a>Microsoft.Blockchain/cordaMembers

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|BlockchainApplication|Aplicación de cadena de bloques|No|


## <a name="microsoftbotservicebotservices"></a>microsoft.botservice/botservices

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|BotRequest|Solicitudes de los canales al bot|No|


## <a name="microsoftcdncdnwebapplicationfirewallpolicies"></a>Microsoft.Cdn/cdnwebapplicationfirewallpolicies

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|WebApplicationFirewallLogs|Registros de firewall de la aplicación web|No|


## <a name="microsoftcdnprofiles"></a>Microsoft.Cdn/profiles

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|AzureCdnAccessLog|Registro de acceso a Azure CDN|No|
|FrontDoorAccessLog|Registro de acceso de FrontDoor|Sí|
|FrontDoorHealthProbeLog|Registro de sondeo de estado de FrontDoor|Sí|
|FrontDoorWebApplicationFirewallLog|Registro de firewall de aplicaciones web de FrontDoor|Sí|


## <a name="microsoftcdnprofilesendpoints"></a>Microsoft.Cdn/profiles/endpoints

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|CoreAnalytics|Obtiene las métricas del punto de conexión; por ejemplo, ancho de banda, salida, etc.|No|


## <a name="microsoftclassicnetworknetworksecuritygroups"></a>Microsoft.ClassicNetwork/networkSecurityGroups

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|Evento de flujo de reglas de grupo de seguridad de red|Evento de flujo de reglas de grupo de seguridad de red|No|


## <a name="microsoftcognitiveservicesaccounts"></a>Microsoft.CognitiveServices/accounts

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|Auditoría|Registros de auditoría|No|
|RequestResponse|Registros de solicitud y respuesta|No|
|Seguimiento|Registros de seguimiento|No|


## <a name="microsoftcommunicationcommunicationservices"></a>Microsoft.Communication/CommunicationServices

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|AuthOperational|Registros de autenticación operativos|Sí|
|ChatOperational|Registros de chat operativos|No|
|SMSOperational|Registros de SMS operativos|No|
|Uso|Registros de uso|No|


## <a name="microsoftcontainerregistryregistries"></a>Microsoft.ContainerRegistry/registries

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|ContainerRegistryLoginEvents|Eventos de inicio de sesión|No|
|ContainerRegistryRepositoryEvents|Registros de RepositoryEvent|No|


## <a name="microsoftcontainerservicemanagedclusters"></a>Microsoft.ContainerService/managedClusters

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|cluster-autoscaler|Cluster Autoscaler de Kubernetes|No|
|guard|guard|No|
|kube-apiserver|Servidor de la API de Kubernetes|No|
|kube-audit|Auditoría de Kubernetes|No|
|kube-audit-admin|Registros de administrador de auditoría de Kubernetes|No|
|kube-controller-manager|Administrador del controlador de Kubernetes|No|
|kube-scheduler|Programador de Kubernetes|No|


## <a name="microsoftcustomprovidersresourceproviders"></a>Microsoft.CustomProviders/resourceproviders

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|AuditLogs|Registros de auditoría para llamadas de MiniRP|No|


## <a name="microsoftd365customerinsightsinstances"></a>Microsoft.D365CustomerInsights/instances

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|Auditoría|Eventos de auditoría|No|
|Operativos|Eventos operativos|No|


## <a name="microsoftdatabricksworkspaces"></a>Microsoft.Databricks/workspaces

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|accounts|Cuentas de Databricks|No|
|clusters|Clústeres de Databricks|No|
|dbfs|Sistema de archivos de Databricks|No|
|instancePools|Grupos de instancias|No|
|jobs|Trabajos de Databricks|No|
|notebook|Cuaderno de Databricks|No|
|secrets|Secretos de Databricks|No|
|sqlPermissions|Permisos SQL de Databricks|No|
|ssh|SSH de Databricks|No|
|área de trabajo|Área de trabajo de Databricks|No|


## <a name="microsoftdatacollaborationworkspaces"></a>Microsoft.DataCollaboration/workspaces

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|CollaborationAudit|Auditoría de colaboración|Sí|
|DataAssets|Recursos de datos|No|
|Canalizaciones|Canalizaciones|No|
|Proposals|Propuestas|No|
|Scripts|Scripts|No|


## <a name="microsoftdatafactoryfactories"></a>Microsoft.DataFactory/factories

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|ActivityRuns|Registro de ejecuciones de actividad de canalización|No|
|PipelineRuns|Registro de ejecuciones de canalización|No|
|SandboxActivityRuns|Registro de ejecuciones de actividad de espacio aislado|Sí|
|SandboxPipelineRuns|Registro de ejecuciones de canalización de espacio aislado|Sí|
|SSISIntegrationRuntimeLogs|Registros de SQL Server Integration Services|No|
|SSISPackageEventMessageContext|Contexto de mensaje de evento del paquete SSIS|No|
|SSISPackageEventMessages|Mensajes de evento del paquete SSIS|No|
|SSISPackageExecutableStatistics|Estadísticas de ejecutable del paquete SSIS|No|
|SSISPackageExecutionComponentPhases|Fases del componente de ejecución del paquete SSIS|No|
|SSISPackageExecutionDataStatistics|Estadísticas de datos de ejecución del paquete SSIS|No|
|TriggerRuns|Registro de ejecuciones de desencadenador|No|


## <a name="microsoftdatalakeanalyticsaccounts"></a>Microsoft.DataLakeAnalytics/accounts

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|Auditoría|Registros de auditoría|No|
|Requests|Registros de solicitud|No|


## <a name="microsoftdatalakestoreaccounts"></a>Microsoft.DataLakeStore/accounts

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|Auditoría|Registros de auditoría|No|
|Requests|Registros de solicitud|No|


## <a name="microsoftdatashareaccounts"></a>Microsoft.DataShare/accounts

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|ReceivedShareSnapshots|Instantáneas de recursos compartidos recibidas|No|
|SentShareSnapshots|Instantáneas de recursos compartidos enviadas|No|
|Recursos compartidos|Recursos compartidos|No|
|ShareSubscriptions|Suscripciones a recursos compartidos|No|


## <a name="microsoftdbformariadbservers"></a>Microsoft.DBforMariaDB/servers

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|MySqlAuditLogs|Registros de auditoría de MariaDB|No|
|MySqlSlowLogs|Registros de servidor de MariaDB|No|


## <a name="microsoftdbformysqlflexibleservers"></a>Microsoft.DBforMySQL/flexibleServers

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|MySqlAuditLogs|Registros de auditoría de MySQL|No|
|MySqlSlowLogs|Registros lentos de MySQL|No|


## <a name="microsoftdbformysqlservers"></a>Microsoft.DBforMySQL/servers

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|MySqlAuditLogs|Registros de auditoría de MySQL|No|
|MySqlSlowLogs|Registros de MySQL Server|No|


## <a name="microsoftdbforpostgresqlflexibleservers"></a>Microsoft.DBforPostgreSQL/flexibleServers

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|PostgreSQLLogs|Registros del servidor PostgreSQL|No|


## <a name="microsoftdbforpostgresqlservergroupsv2"></a>Microsoft.DBForPostgreSQL/serverGroupsv2

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|PostgreSQLLogs|Registros del servidor PostgreSQL|Sí|


## <a name="microsoftdbforpostgresqlservers"></a>Microsoft.DBforPostgreSQL/servers

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|PostgreSQLLogs|Registros del servidor PostgreSQL|No|
|QueryStoreRuntimeStatistics|Estadísticas de tiempo de ejecución del almacén de consultas de PostgreSQL|No|
|QueryStoreWaitStatistics|Estadísticas de espera del almacén de consultas de PostgreSQL|No|


## <a name="microsoftdbforpostgresqlserversv2"></a>Microsoft.DBforPostgreSQL/serversv2

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|PostgreSQLLogs|Registros del servidor PostgreSQL|No|


## <a name="microsoftdesktopvirtualizationapplicationgroups"></a>Microsoft.DesktopVirtualization/applicationgroups

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|Punto de control|Punto de control|No|
|Error|Error|No|
|Administración|Administración|No|


## <a name="microsoftdesktopvirtualizationhostpools"></a>Microsoft.DesktopVirtualization/hostpools

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|AgentHealthStatus|AgentHealthStatus|No|
|Punto de control|Punto de control|No|
|Conexión|Conexión|No|
|Error|Error|No|
|HostRegistration|HostRegistration|No|
|Administración|Administración|No|


## <a name="microsoftdesktopvirtualizationworkspaces"></a>Microsoft.DesktopVirtualization/workspaces

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|Punto de control|Punto de control|No|
|Error|Error|No|
|Fuente|Fuente|No|
|Administración|Administración|No|


## <a name="microsoftdeviceselasticpoolsiothubtenants"></a>Microsoft.Devices/ElasticPools/IotHubTenants

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|C2DCommands|Comandos C2D|No|
|C2DTwinOperations|Operaciones gemelas C2D|No|
|Configurations|Configurations|No|
|Conexiones|Conexiones|No|
|D2CTwinOperations|D2CTwinOperations|No|
|DeviceIdentityOperations|Operaciones de identidad de dispositivos|No|
|DeviceStreams|Flujos de dispositivos (versión preliminar)|No|
|DeviceTelemetry|Telemetría de dispositivo|No|
|DirectMethods|Métodos directos|No|
|DistributedTracing|Seguimiento distribuido (versión preliminar)|No|
|FileUploadOperations|Operaciones de carga de archivos|No|
|JobsOperations|Operaciones de trabajos|No|
|Rutas|Rutas|No|
|TwinQueries|Consultas gemelas|No|


## <a name="microsoftdevicesiothubs"></a>Microsoft.Devices/IotHubs

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|C2DCommands|Comandos C2D|No|
|C2DTwinOperations|Operaciones gemelas C2D|No|
|Configurations|Configurations|No|
|Conexiones|Conexiones|No|
|D2CTwinOperations|D2CTwinOperations|No|
|DeviceIdentityOperations|Operaciones de identidad de dispositivos|No|
|DeviceStreams|Flujos de dispositivos (versión preliminar)|No|
|DeviceTelemetry|Telemetría de dispositivo|No|
|DirectMethods|Métodos directos|No|
|DistributedTracing|Seguimiento distribuido (versión preliminar)|No|
|FileUploadOperations|Operaciones de carga de archivos|No|
|JobsOperations|Operaciones de trabajos|No|
|Rutas|Rutas|No|
|TwinQueries|Consultas gemelas|No|


## <a name="microsoftdevicesprovisioningservices"></a>Microsoft.Devices/provisioningServices

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|DeviceOperations|Operaciones de dispositivo|No|
|ServiceOperations|Operaciones del servicio|No|


## <a name="microsoftdigitaltwinsdigitaltwinsinstances"></a>Microsoft.DigitalTwins/digitalTwinsInstances

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|DigitalTwinsOperation|DigitalTwinsOperation|No|
|EventRoutesOperation|EventRoutesOperation|No|
|ModelsOperation|ModelsOperation|No|
|QueryOperation|QueryOperation|No|


## <a name="microsoftdocumentdbdatabaseaccounts"></a>Microsoft.DocumentDB/databaseAccounts

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|CassandraRequests|CassandraRequests|No|
|ControlPlaneRequests|ControlPlaneRequests|No|
|DataPlaneRequests|DataPlaneRequests|No|
|GremlinRequests|GremlinRequests|No|
|MongoRequests|MongoRequests|No|
|PartitionKeyRUConsumption|PartitionKeyRUConsumption|No|
|PartitionKeyStatistics|PartitionKeyStatistics|No|
|QueryRuntimeStatistics|QueryRuntimeStatistics|No|


## <a name="microsofteventgriddomains"></a>Microsoft.EventGrid/domains

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|DeliveryFailures|Registros de errores de entrega|No|
|PublishFailures|Registros de errores de publicación|No|


## <a name="microsofteventgridpartnernamespaces"></a>Microsoft.EventGrid/partnerNamespaces

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|DeliveryFailures|Registros de errores de entrega|No|
|PublishFailures|Registros de errores de publicación|No|


## <a name="microsofteventgridpartnertopics"></a>Microsoft.EventGrid/partnerTopics

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|DeliveryFailures|Registros de errores de entrega|No|


## <a name="microsofteventgridsystemtopics"></a>Microsoft.EventGrid/systemTopics

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|DeliveryFailures|Registros de errores de entrega|No|


## <a name="microsofteventgridtopics"></a>Microsoft.EventGrid/topics

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|DeliveryFailures|Registros de errores de entrega|No|
|PublishFailures|Registros de errores de publicación|No|


## <a name="microsofteventhubnamespaces"></a>Microsoft.EventHub/namespaces

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|ArchiveLogs|Registros de archivo|No|
|AutoScaleLogs|Registros de escalado automático|No|
|CustomerManagedKeyUserLogs|Registros de clave administrada de cliente|No|
|EventHubVNetConnectionEvent|Registros de conexión de filtrado de red virtual/IP|No|
|KafkaCoordinatorLogs|Registros de coordinador de Kafka|No|
|KafkaUserErrorLogs|Registros de error de usuario de Kafka|No|
|OperationalLogs|Registros operativos|No|


## <a name="microsoftexperimentationexperimentworkspaces"></a>microsoft.experimentation/experimentWorkspaces

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|Solicitud|Solicitud|No|


## <a name="microsofthealthcareapisservices"></a>Microsoft.HealthcareApis/services

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|AuditLogs|Registros de auditoría|No|


## <a name="microsoftinsightsautoscalesettings"></a>microsoft.insights/autoscalesettings

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|AutoscaleEvaluations|Evaluaciones de escalabilidad automática|No|
|AutoscaleScaleActions|Acciones de escalabilidad automática|No|


## <a name="microsoftinsightscomponents"></a>Microsoft.Insights/Components

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|AppAvailabilityResults|Resultados de disponibilidad|No|
|AppBrowserTimings|Intervalos de explorador|No|
|AppDependencies|Dependencias|No|
|AppEvents|Eventos|No|
|AppExceptions|Excepciones|No|
|AppMetrics|Métricas|No|
|AppPageViews|Vistas de página|No|
|AppPerformanceCounters|Contadores de rendimiento|No|
|AppRequests|Requests|No|
|AppSystemEvents|Eventos del sistema|No|
|AppTraces|Traces|No|


## <a name="microsoftkeyvaultmanagedhsms"></a>microsoft.keyvault/managedhsms

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|AuditEvent|Evento de auditoría|No|


## <a name="microsoftkeyvaultvaults"></a>Microsoft.KeyVault/vaults

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|AuditEvent|Registros de auditoría|No|


## <a name="microsoftkustoclusters"></a>Microsoft.Kusto/Clusters

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|Comando|Comando|No|
|FailedIngestion|Operaciones de ingesta con error|No|
|IngestionBatching|Procesamiento por lotes de ingesta|No|
|Consultar|Consultar|No|
|SucceededIngestion|Operaciones de ingesta correctas|No|
|TableDetails|Detalles de tabla|No|
|TableUsageStatistics|Estadísticas de uso de tablas|No|


## <a name="microsoftlogicintegrationaccounts"></a>Microsoft.Logic/integrationAccounts

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|IntegrationAccountTrackingEvents|Eventos de seguimiento de la cuenta de integración|No|


## <a name="microsoftlogicworkflows"></a>Microsoft.Logic/workflows

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|WorkflowRuntime|Eventos de diagnóstico en tiempo de ejecución de flujo de trabajo|No|


## <a name="microsoftmachinelearningservicesworkspaces"></a>Microsoft.MachineLearningServices/workspaces

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|AmlComputeClusterEvent|AmlComputeClusterEvent|No|
|AmlComputeClusterNodeEvent|AmlComputeClusterNodeEvent|No|
|AmlComputeCpuGpuUtilization|AmlComputeCpuGpuUtilization|No|
|AmlComputeJobEvent|AmlComputeJobEvent|No|
|AmlRunStatusChangedEvent|AmlRunStatusChangedEvent|No|


## <a name="microsoftmediamediaservices"></a>Microsoft.Media/mediaservices

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|KeyDeliveryRequests|Solicitudes de entrega de clave|No|


## <a name="microsoftnetworkapplicationgateways"></a>Microsoft.Network/applicationGateways

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|ApplicationGatewayAccessLog|Registro de acceso de Application Gateway|No|
|ApplicationGatewayFirewallLog|Registro de Firewall de Application Gateway|No|
|ApplicationGatewayPerformanceLog|Registro de rendimiento de Application Gateway|No|


## <a name="microsoftnetworkazurefirewalls"></a>Microsoft.Network/azurefirewalls

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|AzureFirewallApplicationRule|Regla de aplicación de Azure Firewall|No|
|AzureFirewallDnsProxy|Proxy de DNS de Azure Firewall|No|
|AzureFirewallNetworkRule|Regla de red de Azure Firewall|No|


## <a name="microsoftnetworkbastionhosts"></a>Microsoft.Network/bastionHosts

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|BastionAuditLogs|Registros de auditoría de Bastion|No|


## <a name="microsoftnetworkexpressroutecircuits"></a>Microsoft.Network/expressRouteCircuits

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|PeeringRouteLog|Registros de tablas de rutas de emparejamiento|No|


## <a name="microsoftnetworkfrontdoors"></a>Microsoft.Network/frontdoors

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|FrontdoorAccessLog|Registro de acceso de Front Door|No|
|FrontdoorWebApplicationFirewallLog|Registro de firewall de aplicaciones web de Front Door|No|


## <a name="microsoftnetworknetworksecuritygroups"></a>Microsoft.Network/networksecuritygroups

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|NetworkSecurityGroupEvent|Evento de grupo de seguridad de red|No|
|NetworkSecurityGroupFlowEvent|Evento de flujo de reglas de grupo de seguridad de red|No|
|NetworkSecurityGroupRuleCounter|Contador de reglas de grupo de seguridad de red|No|


## <a name="microsoftnetworkp2svpngateways"></a>Microsoft.Network/p2sVpnGateways

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|GatewayDiagnosticLog|Registros de diagnóstico de puerta de enlace|No|
|IKEDiagnosticLog|Registros de IKE Diagnostic|No|
|P2SDiagnosticLog|Registros de P2S Diagnostic|No|


## <a name="microsoftnetworkpublicipaddresses"></a>Microsoft.Network/publicIPAddresses

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|DDoSMitigationFlowLogs|Registros de flujos de las decisiones de mitigación de DDoS|No|
|DDoSMitigationReports|Informes de las mitigaciones de DDoS|No|
|DDoSProtectionNotifications|Notificaciones de protección contra DDoS|No|


## <a name="microsoftnetworktrafficmanagerprofiles"></a>Microsoft.Network/trafficManagerProfiles

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|ProbeHealthStatusEvents|Evento de resultados de estado de sondeo de Traffic Manager|No|


## <a name="microsoftnetworkvirtualnetworkgateways"></a>Microsoft.Network/virtualNetworkGateways

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|GatewayDiagnosticLog|Registros de diagnóstico de puerta de enlace|No|
|IKEDiagnosticLog|Registros de IKE Diagnostic|No|
|P2SDiagnosticLog|Registros de P2S Diagnostic|No|
|RouteDiagnosticLog|Registros de diagnóstico de ruta|No|
|TunnelDiagnosticLog|Registros de diagnóstico de túnel|No|


## <a name="microsoftnetworkvirtualnetworks"></a>Microsoft.Network/virtualNetworks

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|VMProtectionAlerts|Alertas de protección de máquina virtual|No|


## <a name="microsoftnetworkvpngateways"></a>Microsoft.Network/vpnGateways

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|GatewayDiagnosticLog|Registros de diagnóstico de puerta de enlace|No|
|IKEDiagnosticLog|Registros de IKE Diagnostic|No|
|RouteDiagnosticLog|Registros de diagnóstico de ruta|No|
|TunnelDiagnosticLog|Registros de diagnóstico de túnel|No|


## <a name="microsoftnotificationhubsnamespaces"></a>Microsoft.NotificationHubs/namespaces

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|OperationalLogs|Registros operativos|No|


## <a name="microsoftoperationalinsightsworkspaces"></a>Microsoft.OperationalInsights/workspaces

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|Auditoría|Registros de auditoría|No|


## <a name="microsoftpowerbitenants"></a>Microsoft.PowerBI/tenants

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|Motor|Motor|No|


## <a name="microsoftpowerbitenantsworkspaces"></a>Microsoft.PowerBI/tenants/workspaces

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|Motor|Motor|No|


## <a name="microsoftpowerbidedicatedcapacities"></a>Microsoft.PowerBIDedicated/capacities

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|Motor|Motor|No|


## <a name="microsoftpurviewaccounts"></a>microsoft.purview/accounts

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|ScanStatusLogEvent|ScanStatus|No|


## <a name="microsoftrecoveryservicesvaults"></a>Microsoft.RecoveryServices/Vaults

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|AddonAzureBackupAlerts|Datos de alerta de Azure Backup del complemento|No|
|AddonAzureBackupJobs|Datos de trabajo de Azure Backup del complemento|No|
|AddonAzureBackupPolicy|Datos de directiva de Azure Backup del complemento|No|
|AddonAzureBackupProtectedInstance|Datos de instancia protegida de Azure Backup del complemento|No|
|AddonAzureBackupStorage|Datos de almacenamiento de Azure Backup del complemento|No|
|AzureBackupReport|Datos de informes de Azure Backup|No|
|AzureSiteRecoveryEvents|Eventos de Azure Site Recovery|No|
|AzureSiteRecoveryJobs|Trabajos de Azure Site Recovery|No|
|AzureSiteRecoveryProtectedDiskDataChurn|Actividad de los datos del disco protegido de Azure Site Recovery|No|
|AzureSiteRecoveryRecoveryPoints|Puntos de recuperación de Azure Site Recovery|No|
|AzureSiteRecoveryReplicatedItems|Elementos replicados de Azure Site Recovery|No|
|AzureSiteRecoveryReplicationDataUploadRate|Velocidad de carga de datos de replicación de Azure Site Recovery|No|
|AzureSiteRecoveryReplicationStats|Estadísticas de replicación de Azure Site Recovery|No|
|CoreAzureBackup|Datos principales de Azure Backup|No|


## <a name="microsoftrelaynamespaces"></a>Microsoft.Relay/namespaces

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|HybridConnectionsEvent|Eventos de HybridConnections|No|
|HybridConnectionsLogs|HybridConnectionsLogs|No|


## <a name="microsoftsearchsearchservices"></a>Microsoft.Search/searchServices

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|OperationLogs|Registros de operaciones|No|


## <a name="microsoftservicebusnamespaces"></a>Microsoft.ServiceBus/namespaces

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|OperationalLogs|Registros operativos|No|


## <a name="microsoftsignalrservicesignalr"></a>Microsoft.SignalRService/SignalR

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|AllLogs|Registros de Azure SignalR Service|No|


## <a name="microsoftsignalrservicewebpubsub"></a>Microsoft.SignalRService/WebPubSub

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|AllLogs|Registros del servicio Azure Web PubSub.|Sí|


## <a name="microsoftsqlmanagedinstances"></a>Microsoft.Sql/managedInstances

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|DevOpsOperationsAudit|Registros de auditoría de operaciones Devops|No|
|ResourceUsageStats|Estadísticas de uso de recursos|No|
|SQLSecurityAuditEvents|Evento de auditoría de seguridad de SQL|No|


## <a name="microsoftsqlmanagedinstancesdatabases"></a>Microsoft.Sql/managedInstances/databases

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|Errors|Errors|No|
|QueryStoreRuntimeStatistics|Estadísticas de tiempo de ejecución del Almacén de consultas|No|
|QueryStoreWaitStatistics|Estadísticas de espera del Almacén de consultas|No|
|SQLInsights|SQL Insights|No|


## <a name="microsoftsqlserversdatabases"></a>Microsoft.Sql/servers/databases

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|AutomaticTuning|Ajuste automático|No|
|Blocks|Blocks|No|
|DatabaseWaitStatistics|Estadísticas de espera de base de datos|No|
|Interbloqueos|Interbloqueos|No|
|DevOpsOperationsAudit|Registros de auditoría de operaciones Devops|No|
|DmsWorkers|Trabajo de DMS|No|
|Errors|Errors|No|
|ExecRequests|Solicitudes de ejecución|No|
|QueryStoreRuntimeStatistics|Estadísticas de tiempo de ejecución del Almacén de consultas|No|
|QueryStoreWaitStatistics|Estadísticas de espera del Almacén de consultas|No|
|RequestSteps|Pasos de la solicitud|No|
|SQLInsights|SQL Insights|No|
|SqlRequests|Solicitudes de SQL|No|
|SQLSecurityAuditEvents|Evento de auditoría de seguridad de SQL|No|
|Tiempos de expiración|Tiempos de expiración|No|
|Esperas|Esperas|No|


## <a name="microsoftstoragestorageaccountsblobservices"></a>Microsoft.Storage/storageAccounts/blobServices

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|StorageDelete|StorageDelete|Sí|
|StorageRead|StorageRead|Sí|
|StorageWrite|StorageWrite|Sí|


## <a name="microsoftstoragestorageaccountsfileservices"></a>Microsoft.Storage/storageAccounts/fileServices

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|StorageDelete|StorageDelete|Sí|
|StorageRead|StorageRead|Sí|
|StorageWrite|StorageWrite|Sí|


## <a name="microsoftstoragestorageaccountsqueueservices"></a>Microsoft.Storage/storageAccounts/queueServices

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|StorageDelete|StorageDelete|Sí|
|StorageRead|StorageRead|Sí|
|StorageWrite|StorageWrite|Sí|


## <a name="microsoftstoragestorageaccountstableservices"></a>Microsoft.Storage/storageAccounts/tableServices

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|StorageDelete|StorageDelete|Sí|
|StorageRead|StorageRead|Sí|
|StorageWrite|StorageWrite|Sí|


## <a name="microsoftstreamanalyticsstreamingjobs"></a>Microsoft.StreamAnalytics/streamingjobs

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|Creación|Creación|No|
|Ejecución|Ejecución|No|


## <a name="microsoftsynapseworkspaces"></a>Microsoft.Synapse/workspaces

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|BuiltinSqlReqsEnded|Solicitudes finalizadas de grupo de SQL integrado|No|
|GatewayApiRequests|Solicitudes de API de puerta de enlace de Azure Synapse|No|
|IntegrationActivityRuns|Ejecuciones de actividad de integración|Sí|
|IntegrationPipelineRuns|Ejecuciones de canalización de integración|Sí|
|IntegrationTriggerRuns|Ejecuciones de desencadenador de integración|Sí|
|SQLSecurityAuditEvents|Evento de auditoría de seguridad de SQL|No|
|SynapseRbacOperations|Operaciones de RBAC de Synapse|No|


## <a name="microsoftsynapseworkspacesbigdatapools"></a>Microsoft.Synapse/workspaces/bigDataPools

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|BigDataPoolAppsEnded|Aplicaciones finalizadas del grupo de macrodatos|No|


## <a name="microsoftsynapseworkspacessqlpools"></a>Microsoft.Synapse/workspaces/sqlPools

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|DmsWorkers|Trabajo de DMS|No|
|ExecRequests|Solicitudes de ejecución|No|
|RequestSteps|Pasos de la solicitud|No|
|SqlRequests|Solicitudes de SQL|No|
|SQLSecurityAuditEvents|Evento de auditoría de seguridad de SQL|No|
|Esperas|Esperas|No|


## <a name="microsofttimeseriesinsightsenvironments"></a>Microsoft.TimeSeriesInsights/environments

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|Entrada|Entrada|No|
|Administración|Administración|No|


## <a name="microsofttimeseriesinsightsenvironmentseventsources"></a>Microsoft.TimeSeriesInsights/environments/eventsources

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|Entrada|Entrada|No|
|Administración|Administración|No|


## <a name="microsoftwebhostingenvironments"></a>microsoft.web/hostingenvironments

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|AppServiceEnvironmentPlatformLogs|Registros de plataforma de App Service Environment|No|


## <a name="microsoftwebsites"></a>microsoft.web/sites

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|AppServiceAntivirusScanAuditLogs|Informe de registros de auditoría de antivirus|No|
|AppServiceAppLogs|Registros de aplicaciones de App Service|No|
|AppServiceAuditLogs|Acceso a registros de auditoría|No|
|AppServiceConsoleLogs|Registros de consola de App Service|No|
|AppServiceFileAuditLogs|Registros de auditoría de cambios de contenido de sitio|No|
|AppServiceHTTPLogs|Registros de HTTP|No|
|AppServiceIPSecAuditLogs|Registros de auditoría de IPSecurity|No|
|AppServicePlatformLogs|Registros de la plataforma App Service|No|
|FunctionAppLogs|Registros de aplicaciones de función|No|


## <a name="microsoftwebsitesslots"></a>microsoft.web/sites/slots

|Category|Nombre para mostrar de categoría|Costos de exportación|
|---|---|---|
|AppServiceAntivirusScanAuditLogs|Informe de registros de auditoría de antivirus|No|
|AppServiceAppLogs|Registros de aplicaciones de App Service|No|
|AppServiceAuditLogs|Acceso a registros de auditoría|No|
|AppServiceConsoleLogs|Registros de consola de App Service|No|
|AppServiceFileAuditLogs|Registros de auditoría de cambios de contenido de sitio|No|
|AppServiceHTTPLogs|Registros de HTTP|No|
|AppServiceIPSecAuditLogs|Registros de auditoría de IPSecurity|No|
|AppServicePlatformLogs|Registros de la plataforma App Service|No|
|FunctionAppLogs|Registros de aplicaciones de función|No|


## <a name="next-steps"></a>Pasos siguientes

* [Más información sobre los registros de recurso](../essentials/platform-logs-overview.md)
* [Transmisión de registros de recursos a **Event Hubs**](./resource-logs.md#send-to-azure-event-hubs)
* [Cambio de la configuración de diagnóstico del registro de recursos con la API REST de Azure Monitor](/rest/api/monitor/diagnosticsettings)
* [Análisis de registros desde Azure Storage con Log Analytics](./resource-logs.md#send-to-log-analytics-workspace)

