---
title: Proveedores de recursos por servicios de Azure
description: Se enumeran todos los espacios de nombres de proveedor de recursos para Azure Resource Manager y se muestra el servicio de Azure para ese espacio de nombres.
ms.topic: conceptual
ms.date: 03/16/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 40c166a85da1ad5370b6e90096061790f1c77d49
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108745746"
---
# <a name="resource-providers-for-azure-services"></a>Proveedores de recursos para servicios de Azure

En este artículo se muestra cómo se asignan los espacios de nombres de proveedor de recursos a servicios de Azure. Si no conoce el proveedor de recursos, consulte [Búsqueda del proveedor de recursos](#find-resource-provider).

## <a name="match-resource-provider-to-service"></a>Coincidencia del proveedor de recursos con el servicio

Los proveedores de recursos con la marca **registrado** se registran de forma predeterminada para la suscripción. Para más información, consulte [Registro](#registration).

| Espacio de nombres del proveedor de recursos | Servicio de Azure |
| --------------------------- | ------------- |
| Microsoft.AAD | [Azure Active Directory Domain Services](../../active-directory-domain-services/index.yml) |
| Microsoft.Addons | core |
| Microsoft.ADHybridHealthService: [registrado](#registration) | [Azure Active Directory](../../active-directory/index.yml) |
| Microsoft.Advisor | [Azure Advisor](../../advisor/index.yml) |
| Microsoft.AlertsManagement | [Azure Monitor](../../azure-monitor/index.yml) |
| Microsoft.AnalysisServices | [Azure Analysis Services](../../analysis-services/index.yml) |
| Microsoft.ApiManagement | [API Management](../../api-management/index.yml) |
| Microsoft.AppConfiguration | [Azure App Configuration](../../azure-app-configuration/index.yml) |
| Microsoft.AppPlatform | [Azure Spring Cloud](../../spring-cloud/overview.md) |
| Microsoft.Attestation | Servicio de atestación de Azure |
| Microsoft.Authorization: [registrado](#registration) | [Azure Resource Manager](../index.yml) |
| Microsoft.Automation | [Automation](../../automation/index.yml) |
| Microsoft.AutonomousSystems | [Sistema autónomo](https://www.microsoft.com/ai/autonomous-systems) |
| Microsoft.AVS | [Azure VMware Solution](../../azure-vmware/index.yml) |
| Microsoft.AzureActiveDirectory | [Azure Active Directory B2C](../../active-directory-b2c/index.yml) |
| Microsoft.AzureArcData | Servicios de datos habilitados para Azure Arc |
| Microsoft.AzureData | Registro de SQL Server |
| Microsoft.AzureStack | core |
| Microsoft.AzureStackHCI | [Azure Stack HCI](/azure-stack/hci/overview) |
| Microsoft.Batch | [Batch](../../batch/index.yml) |
| Microsoft.Billing: [registrado](#registration) | [Administración de costos y facturación](/azure/billing/) |
| Microsoft.BingMaps | [Mapas de Bing](/BingMaps/#pivot=main&panel=BingMapsAPI) |
| Microsoft.Blockchain | [Azure Blockchain Service](../../blockchain/workbench/index.yml) |
| Microsoft.BlockchainTokens | [Azure Blockchain Tokens](https://azure.microsoft.com/services/blockchain-tokens/) |
| Microsoft.Blueprint | [Azure Blueprints](../../governance/blueprints/index.yml) |
| Microsoft.BotService | [Azure Bot Service](/azure/bot-service/) |
| Microsoft.Cache | [Azure Cache for Redis](../../azure-cache-for-redis/index.yml) |
| Microsoft.Capacity | core |
| Microsoft.Cdn | [Content Delivery Network](../../cdn/index.yml) |
| Microsoft.CertificateRegistration | [Certificados de App Service](../../app-service/configure-ssl-certificate.md#import-an-app-service-certificate) |
| Microsoft.ChangeAnalysis | [Azure Monitor](../../azure-monitor/index.yml) |
| Microsoft.ClassicCompute | Máquina virtual con el modelo de implementación clásica |
| Microsoft.ClassicInfrastructureMigrate | Migración del modelo de implementación clásica |
| Microsoft.ClassicNetwork | Red virtual con el modelo de implementación clásica |
| Microsoft.ClassicStorage | Almacenamiento del modelo de implementación clásica |
| Microsoft.ClassicSubscription: [registrado](#registration) | Modelo de implementación clásica |
| Microsoft.CognitiveServices | [Cognitive Services](../../cognitive-services/index.yml) |
| Microsoft.Commerce: [registrado](#registration) | core |
| Microsoft.Compute | [Máquinas virtuales](../../virtual-machines/index.yml)<br />[Conjuntos de escalado de máquina virtual](../../virtual-machine-scale-sets/index.yml) |
| Microsoft.Consumption: [registrado](#registration) | [Cost Management](/azure/cost-management/) |
| Microsoft.ContainerInstance | [Azure Container Instances](../../container-instances/index.yml) |
| Microsoft.ContainerRegistry | [Container Registry](../../container-registry/index.yml) |
| Microsoft.ContainerService | [Azure Kubernetes Service (AKS)](../../aks/index.yml) |
| Microsoft.CostManagement: [registrado](#registration) | [Cost Management](/azure/cost-management/) |
| Microsoft.CostManagementExports | [Cost Management](/azure/cost-management/) |
| Microsoft.CustomerLockbox | [Caja de seguridad del cliente de Microsoft Azure](../../security/fundamentals/customer-lockbox-overview.md) |
| Microsoft.CustomProviders | [Proveedores personalizados de Azure](../custom-providers/overview.md) |
| Microsoft.DataBox | [Azure Data Box](../../databox/index.yml) |
| Microsoft.DataBoxEdge | [Azure Stack Edge](../../databox-online/azure-stack-edge-overview.md) |
| Microsoft.Databricks | [Azure Databricks](/azure/azure-databricks/) |
| Microsoft.DataCatalog | [Data Catalog](../../data-catalog/index.yml) |
| Microsoft.DataFactory | [Data Factory](../../data-factory/index.yml) |
| Microsoft.DataLakeAnalytics | [Análisis de Data Lake](../../data-lake-analytics/index.yml) |
| Microsoft.DataLakeStore | [Azure Data Lake Storage Gen2](../../storage/blobs/data-lake-storage-introduction.md) |
| Microsoft.DataMigration | [Azure Database Migration Service](../../dms/index.yml) |
| Microsoft.DataProtection | Protección de datos |
| Microsoft.DataShare | [Azure Data Share](../../data-share/index.yml) |
| Microsoft.DBforMariaDB | [Azure Database for MariaDB](../../mariadb/index.yml) |
| Microsoft.DBforMySQL | [Azure Database for MySQL](../../mysql/index.yml) |
| Microsoft.DBforPostgreSQL | [Azure Database para PostgreSQL](../../postgresql/index.yml) |
| Microsoft.DesktopVirtualization | [Windows Virtual Desktop](../../virtual-desktop/index.yml) |
| Microsoft.Devices | [Azure IoT Hub](../../iot-hub/index.yml)<br />[Servicio Azure IoT Hub Device Provisioning](../../iot-dps/index.yml) |
| Microsoft.DevOps | [Azure DevOps](/azure/devops/) |
| Microsoft.DevSpaces | [Azure Dev Spaces](../../dev-spaces/index.yml) |
| Microsoft.DevTestLab | [Azure Lab Services](../../lab-services/index.yml) |
| Microsoft.DigitalTwins | [Azure Digital Twins](../../digital-twins/overview.md) |
| Microsoft.DocumentDB | [Azure Cosmos DB](../../cosmos-db/index.yml) |
| Microsoft.DomainRegistration | [App Service](../../app-service/index.yml) |
| Microsoft.DynamicsLcs | [Lifecycle Services](https://lcs.dynamics.com/Logon/Index ) |
| Microsoft.EnterpriseKnowledgeGraph | Gráfico de conocimiento empresarial |
| Microsoft.EventGrid | [Event Grid](../../event-grid/index.yml) |
| Microsoft.EventHub | [Event Hubs](../../event-hubs/index.yml) |
| Microsoft.Features: [registrado](#registration) | [Azure Resource Manager](../index.yml) |
| Microsoft.GuestConfiguration | [Azure Policy](../../governance/policy/index.yml) |
| Microsoft.HanaOnAzure | [SAP HANA en Azure (instancias grandes)](../../virtual-machines/workloads/sap/hana-overview-architecture.md) |
| Microsoft.HardwareSecurityModules | [Azure Dedicated HSM](../../dedicated-hsm/index.yml) |
| Microsoft.HDInsight | [HDInsight](../../hdinsight/index.yml) |
| Microsoft.HealthcareApis | [Azure API for FHIR](../../healthcare-apis/fhir/index.yml) |
| Microsoft.HybridCompute | [Azure Arc](../../azure-arc/index.yml) |
| Microsoft.HybridData | [StorSimple](../../storsimple/index.yml) |
| Microsoft.HybridNetwork  | [Private Edge Zones](../../networking/edge-zones-overview.md) |
| Microsoft.ImportExport | [Azure Import/Export](../../import-export/storage-import-export-service.md) |
| Microsoft.Insights | [Azure Monitor](../../azure-monitor/index.yml) |
| Microsoft.IoTCentral | [Azure IoT Central](../../iot-central/index.yml) |
| Microsoft.IoTSpaces | [Azure Digital Twins](../../digital-twins/index.yml) |
| Microsoft.Intune | [Azure Monitor](../../azure-monitor/index.yml) |
| Microsoft.KeyVault | [Key Vault](../../key-vault/index.yml) |
| Microsoft.Kubernetes | [Azure Kubernetes Service (AKS)](../../aks/index.yml) |
| Microsoft.KubernetesConfiguration | [Azure Kubernetes Service (AKS)](../../aks/index.yml) |
| Microsoft.Kusto | [Azure Data Explorer](/azure/data-explorer/) |
| Microsoft.LabServices | [Azure Lab Services](../../lab-services/index.yml) |
| Microsoft.Logic | [Logic Apps](../../logic-apps/index.yml) |
| Microsoft.MachineLearning | [Machine Learning Studio](../../machine-learning/classic/index.yml) |
| Microsoft.MachineLearningServices | [Azure Machine Learning](../../machine-learning/index.yml) |
| Microsoft.Maintenance | [Mantenimiento de Azure](../../virtual-machines/maintenance-control-cli.md) |
| Microsoft.ManagedIdentity | [Identidades administradas para recursos de Azure](../../active-directory/managed-identities-azure-resources/index.yml) |
| Microsoft.ManagedNetwork | Redes virtuales administradas por servicios PaaS |
| Microsoft.ManagedServices | [Azure Lighthouse](../../lighthouse/index.yml) |
| Microsoft.Management | [Grupos de administración](../../governance/management-groups/index.yml) |
| Microsoft.Maps | [Azure Maps](../../azure-maps/index.yml) |
| Microsoft.Marketplace | core |
| Microsoft.MarketplaceApps | core |
| Microsoft.MarketplaceOrdering: [registrado](#registration) | core |
| Microsoft.Media | [Media Services](../../media-services/index.yml) |
| Microsoft.Microservices4Spring | [Azure Spring Cloud](../../spring-cloud/overview.md) |
| Microsoft.Migrate | [Azure Migrate](../../migrate/migrate-services-overview.md) |
| Microsoft.MixedReality | [Azure Spatial Anchors](../../spatial-anchors/index.yml) |
| Microsoft.NetApp | [Azure NetApp Files](../../azure-netapp-files/index.yml) |
| Microsoft.Network | [Application Gateway](../../application-gateway/index.yml)<br />[Azure Bastion](../../bastion/index.yml)<br />[Azure DDoS Protection](../../ddos-protection/ddos-protection-overview.md)<br />[DNS de Azure](../../dns/index.yml)<br />[Información técnica de ExpressRoute](../../expressroute/index.yml)<br />[Azure Firewall](../../firewall/index.yml)<br />[Azure Front Door Service](../../frontdoor/index.yml)<br />[Azure Private Link](../../private-link/index.yml)<br />[Equilibrador de carga](../../load-balancer/index.yml)<br />[Network Watcher](../../network-watcher/index.yml)<br />[Traffic Manager](../../traffic-manager/index.yml)<br />[Virtual Network](../../virtual-network/index.yml)<br />[Virtual WAN](../../virtual-wan/index.yml)<br />[VPN Gateway](../../vpn-gateway/index.yml)<br /> |
| Microsoft.Notebooks | [Azure Notebooks](https://notebooks.azure.com/help/introduction) |
| Microsoft.NotificationHubs | [Centros de notificaciones](../../notification-hubs/index.yml) |
| Microsoft.ObjectStore | Object Store |
| Microsoft.OffAzure | [Azure Migrate](../../migrate/migrate-services-overview.md) |
| Microsoft.OperationalInsights | [Azure Monitor](../../azure-monitor/index.yml) |
| Microsoft.OperationsManagement | [Azure Monitor](../../azure-monitor/index.yml) |
| Microsoft.Peering | [Azure Peering Service](../../peering-service/index.yml) |
| Microsoft.PolicyInsights | [Azure Policy](../../governance/policy/index.yml) |
| Microsoft.Portal: [registrado](#registration) | [Azure Portal](../../azure-portal/index.yml) |
| Microsoft.PowerBI | [Power BI](/power-bi/power-bi-overview) |
| Microsoft.PowerBIDedicated | [Power BI Embedded](/azure/power-bi-embedded/) |
| Microsoft.PowerPlatform | [Power Platform](/power-platform/) |
| Microsoft.ProjectBabylon | [Azure Data Catalog](../../data-catalog/overview.md) |
| Microsoft.Quantum | [Azure Quantum](https://azure.microsoft.com/services/quantum/) |
| Microsoft.RecoveryServices | [Azure Site Recovery](../../site-recovery/index.yml) |
| Microsoft.RedHatOpenShift | [Red Hat OpenShift en Azure](../../virtual-machines/linux/openshift-get-started.md) |
| Microsoft.Relay | [Azure Relay](../../azure-relay/relay-what-is-it.md) |
| Microsoft.ResourceGraph: [registrado](#registration) | [Azure Resource Graph](../../governance/resource-graph/index.yml) |
| Microsoft.ResourceHealth | [Azure Service Health](../../service-health/index.yml) |
| Microsoft.Resources: [registrado](#registration) | [Azure Resource Manager](../index.yml) |
| Microsoft.SaaS | core |
| Microsoft.Scheduler | [Scheduler](../../scheduler/index.yml) |
| Microsoft.Search | [Azure Cognitive Search](../../search/index.yml) |
| Microsoft.Security | [Centro de seguridad](../../security-center/index.yml) |
| Microsoft.SecurityInsights | [Azure Sentinel](../../sentinel/index.yml) |
| Microsoft.SerialConsole: [registrado](#registration) | [Consola serie de Azure para Windows](/troubleshoot/azure/virtual-machines/serial-console-windows) |
| Microsoft.ServiceBus | [Service Bus](/azure/service-bus/) |
| Microsoft.ServiceFabric | [Service Fabric](../../service-fabric/index.yml) |
| Microsoft.Services | core |
| Microsoft.SignalRService | [Servicio Azure SignalR](../../azure-signalr/index.yml) |
| Microsoft.SoftwarePlan | Licencia |
| Microsoft.Solutions | [Azure Managed Applications](../managed-applications/index.yml) |
| Microsoft.Sql | [Azure SQL Database](../../azure-sql/database/index.yml)<br /> [Instancia administrada de Azure SQL](../../azure-sql/managed-instance/index.yml) <br />[Azure Synapse Analytics](/azure/sql-data-warehouse/) |
| Microsoft.SqlVirtualMachine | [SQL Server en Azure Virtual Machines](../../azure-sql/virtual-machines/windows/sql-server-on-azure-vm-iaas-what-is-overview.md) |
| Microsoft.Storage | [Storage](../../storage/index.yml) |
| Microsoft.StorageCache | [Azure HPC Cache](../../hpc-cache/index.yml) |
| Microsoft.StorageSync | [Storage](../../storage/index.yml) |
| Microsoft.StorSimple | [StorSimple](../../storsimple/index.yml) |
| Microsoft.StreamAnalytics | [Azure Stream Analytics](../../stream-analytics/index.yml) |
| Microsoft.Subscription | core |
| microsoft.support: [registrado](#registration) | core |
| Microsoft.Synapse | [Azure Synapse Analytics](/azure/sql-data-warehouse/) |
| Microsoft.TimeSeriesInsights | [Azure Time Series Insights](../../time-series-insights/index.yml) |
| Microsoft.Token | Token |
| Microsoft.VirtualMachineImages | [Azure Image Builder](../../virtual-machines/image-builder-overview.md) |
| microsoft.visualstudio | [Azure DevOps](/azure/devops/) |
| Microsoft.VMware | [Azure VMware Solution](../../azure-vmware/index.yml) |
| Microsoft.VMwareCloudSimple | [Azure VMware Solution by CloudSimple](../../vmware-cloudsimple/index.md) |
| Microsoft.VSOnline | [Azure DevOps](/azure/devops/) |
| Microsoft.Web | [App Service](../../app-service/index.yml)<br />[Funciones de Azure](../../azure-functions/index.yml) |
| Microsoft.WindowsDefenderATP | [Protección contra amenazas avanzada de Microsoft Defender](../../security-center/security-center-wdatp.md) |
| Microsoft.WindowsESU | Actualizaciones de seguridad ampliada |
| Microsoft.WindowsIoT | [Windows 10 IoT Core Services](/windows-hardware/manufacture/iot/iotcoreservicesoverview) |
| Microsoft.WorkloadMonitor | [Azure Monitor](../../azure-monitor/index.yml) |

## <a name="registration"></a>Registro

Los proveedores de recursos anteriores que llevan la marca **registrado** están registrados de forma predeterminada para la suscripción. Para usar los otros proveedores de recursos, debe [registrarlos](resource-providers-and-types.md). Sin embargo, otros proveedores de recursos se registran automáticamente cuando realiza determinadas acciones. Por ejemplo, si crea un recurso a través del portal, el portal registra automáticamente los proveedores de recursos no registrados necesarios. Cuando se implementan recursos a través de una [plantilla de Azure Resource Manager](../templates/overview.md), también se registran los proveedores de recursos necesarios.

> [!IMPORTANT]
> Solo debe registrar un proveedor de recursos cuando esté listo para usarlo. El paso de registro permite mantener los privilegios mínimos dentro de la suscripción. Un usuario malintencionado no puede usar proveedores de recursos que no están registrados.

## <a name="find-resource-provider"></a>Búsqueda del proveedor de recursos

Si tiene una infraestructura existente en Azure, pero no está seguro de qué proveedor de recursos se usa, puede utilizar la CLI de Azure o PowerShell para buscar el proveedor de recursos. Especifique el nombre del grupo de recursos que contiene el recurso que desea encontrar.

En el ejemplo siguiente se usa la CLI de Azure:

```azurecli-interactive
az resource list -g examplegroup
```

Los resultados incluyen el tipo de recurso. El espacio de nombres del proveedor de recursos es la primera parte del tipo de recurso. En el ejemplo siguiente se muestra el proveedor de recursos **Microsoft.KeyVault**.

```json
[
  {
    ...
    "type": "Microsoft.KeyVault/vaults"
  }
]
```

En el ejemplo siguiente se usa PowerShell:

```azurepowershell-interactive
Get-AzResource -ResourceGroupName examplegroup
```

Los resultados incluyen el tipo de recurso. El espacio de nombres del proveedor de recursos es la primera parte del tipo de recurso. En el ejemplo siguiente se muestra el proveedor de recursos **Microsoft.KeyVault**.

```azurepowershell
Name              : examplekey
ResourceGroupName : examplegroup
ResourceType      : Microsoft.KeyVault/vaults
...
```

## <a name="next-steps"></a>Pasos siguientes

Para obtener más información sobre los proveedores de recursos, incluido cómo registrar un proveedor de recursos, consulte [Tipos y proveedores de recursos de Azure](resource-providers-and-types.md).
