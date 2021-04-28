---
title: Compatibilidad con la operación de traslado por tipo de recurso
description: Enumera los tipos de recursos de Azure que se pueden trasladar a un nuevo grupo de recursos, suscripción o región.
ms.topic: conceptual
ms.date: 04/16/2021
ms.openlocfilehash: a56a9e6f04aa800e16bbab0190ce7b41d87da590
ms.sourcegitcommit: 425420fe14cf5265d3e7ff31d596be62542837fb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/20/2021
ms.locfileid: "107740095"
---
# <a name="move-operation-support-for-resources"></a>Compatibilidad con la operación de traslado para recursos

En este artículo se indica si un tipo de recurso de Azure es compatible con la operación de traslado. También proporciona información sobre las condiciones especiales que se deben tener en cuenta a la hora de mover un recurso.

> [!IMPORTANT]
> En la mayoría de los casos, un recurso secundario no se puede mover con independencia de su recurso principal. Los recursos secundarios tienen un tipo de recurso con el formato `<resource-provider-namespace>/<parent-resource>/<child-resource>`. Por ejemplo, `Microsoft.ServiceBus/namespaces/queues` es un recurso secundario de `Microsoft.ServiceBus/namespaces`. Al mover el recurso principal, el recurso secundario se mueve con él automáticamente. Si no ve un recurso secundario en este artículo, puede dar por sentado que se mueve con el recurso principal. Si el recurso principal no admite el movimiento, el recurso secundario no se puede mover.

Vaya a un espacio de nombres del proveedor de recursos:
> [!div class="op_single_selector"]
> - [Microsoft.AAD](#microsoftaad)
> - [microsoft.aadiam](#microsoftaadiam)
> - [Microsoft.Addons](#microsoftaddons)
> - [Microsoft.ADHybridHealthService](#microsoftadhybridhealthservice)
> - [Microsoft.Advisor](#microsoftadvisor)
> - [Microsoft.AlertsManagement](#microsoftalertsmanagement)
> - [Microsoft.AnalysisServices](#microsoftanalysisservices)
> - [Microsoft.ApiManagement](#microsoftapimanagement)
> - [Microsoft.AppConfiguration](#microsoftappconfiguration)
> - [Microsoft.AppPlatform](#microsoftappplatform)
> - [Microsoft.AppService](#microsoftappservice)
> - [Microsoft.Attestation](#microsoftattestation)
> - [Microsoft.Authorization](#microsoftauthorization)
> - [Microsoft.Automation](#microsoftautomation)
> - [Microsoft.AVS](#microsoftavs)
> - [Microsoft.AzureActiveDirectory](#microsoftazureactivedirectory)
> - [Microsoft.AzureData](#microsoftazuredata)
> - [Microsoft.AzureStack](#microsoftazurestack)
> - [Microsoft.AzureStackHCI](#microsoftazurestackhci)
> - [Microsoft.Batch](#microsoftbatch)
> - [Microsoft.Billing](#microsoftbilling)
> - [Microsoft.BingMaps](#microsoftbingmaps)
> - [Microsoft.BizTalkServices](#microsoftbiztalkservices)
> - [Microsoft.Blockchain](#microsoftblockchain)
> - [Microsoft.BlockchainTokens](#microsoftblockchaintokens)
> - [Microsoft.Blueprint](#microsoftblueprint)
> - [Microsoft.BotService](#microsoftbotservice)
> - [Microsoft.Cache](#microsoftcache)
> - [Microsoft.Capacity](#microsoftcapacity)
> - [Microsoft.Cdn](#microsoftcdn)
> - [Microsoft.CertificateRegistration](#microsoftcertificateregistration)
> - [Microsoft.ClassicCompute](#microsoftclassiccompute)
> - [Microsoft.ClassicInfrastructureMigrate](#microsoftclassicinfrastructuremigrate)
> - [Microsoft.ClassicNetwork](#microsoftclassicnetwork)
> - [Microsoft.ClassicStorage](#microsoftclassicstorage)
> - [Microsoft.ClassicSubscription](#microsoftclassicsubscription)
> - [Microsoft.CognitiveServices](#microsoftcognitiveservices)
> - [Microsoft.Commerce](#microsoftcommerce)
> - [Microsoft.Compute](#microsoftcompute)
> - [Microsoft.Consumption](#microsoftconsumption)
> - [Microsoft.ContainerInstance](#microsoftcontainerinstance)
> - [Microsoft.ContainerRegistry](#microsoftcontainerregistry)
> - [Microsoft.ContainerService](#microsoftcontainerservice)
> - [Microsoft.ContentModerator](#microsoftcontentmoderator)
> - [Microsoft.CortanaAnalytics](#microsoftcortanaanalytics)
> - [Microsoft.CostManagement](#microsoftcostmanagement)
> - [Microsoft.CustomerInsights](#microsoftcustomerinsights)
> - [Microsoft.CustomerLockbox](#microsoftcustomerlockbox)
> - [Microsoft.CustomProviders](#microsoftcustomproviders)
> - [Microsoft.DataBox](#microsoftdatabox)
> - [Microsoft.DataBoxEdge](#microsoftdataboxedge)
> - [Microsoft.Databricks](#microsoftdatabricks)
> - [Microsoft.DataCatalog](#microsoftdatacatalog)
> - [Microsoft.DataConnect](#microsoftdataconnect)
> - [Microsoft.DataExchange](#microsoftdataexchange)
> - [Microsoft.DataFactory](#microsoftdatafactory)
> - [Microsoft.DataLake](#microsoftdatalake)
> - [Microsoft.DataLakeAnalytics](#microsoftdatalakeanalytics)
> - [Microsoft.DataLakeStore](#microsoftdatalakestore)
> - [Microsoft.DataMigration](#microsoftdatamigration)
> - [Microsoft.DataProtection](#microsoftdataprotection)
> - [Microsoft.DataShare](#microsoftdatashare)
> - [Microsoft.DBforMariaDB](#microsoftdbformariadb)
> - [Microsoft.DBforMySQL](#microsoftdbformysql)
> - [Microsoft.DBforPostgreSQL](#microsoftdbforpostgresql)
> - [Microsoft.DeploymentManager](#microsoftdeploymentmanager)
> - [Microsoft.DesktopVirtualization](#microsoftdesktopvirtualization)
> - [Microsoft.Devices](#microsoftdevices)
> - [Microsoft.DevOps](#microsoftdevops)
> - [Microsoft.DevSpaces](#microsoftdevspaces)
> - [Microsoft.DevTestLab](#microsoftdevtestlab)
> - [Microsoft.DigitalTwins](#microsoftdigitaltwins)
> - [Microsoft.DocumentDB](#microsoftdocumentdb)
> - [Microsoft.DomainRegistration](#microsoftdomainregistration)
> - [Microsoft.EnterpriseKnowledgeGraph](#microsoftenterpriseknowledgegraph)
> - [Microsoft.EventGrid](#microsofteventgrid)
> - [Microsoft.EventHub](#microsofteventhub)
> - [Microsoft.Experimentation](#microsoftexperimentation)
> - [Microsoft.Falcon](#microsoftfalcon)
> - [Microsoft.Features](#microsoftfeatures)
> - [Microsoft.Genomics](#microsoftgenomics)
> - [Microsoft.GuestConfiguration](#microsoftguestconfiguration)
> - [Microsoft.HanaOnAzure](#microsofthanaonazure)
> - [Microsoft.HardwareSecurityModules](#microsofthardwaresecuritymodules)
> - [Microsoft.HDInsight](#microsofthdinsight)
> - [Microsoft.HealthcareApis](#microsofthealthcareapis)
> - [Microsoft.HybridCompute](#microsofthybridcompute)
> - [Microsoft.HybridData](#microsofthybriddata)
> - [Microsoft.HybridNetwork](#microsofthybridnetwork)
> - [Microsoft.Hydra](#microsofthydra)
> - [Microsoft.ImportExport](#microsoftimportexport)
> - [microsoft.insights](#microsoftinsights)
> - [Microsoft.IoTCentral](#microsoftiotcentral)
> - [Microsoft.IoTSpaces](#microsoftiotspaces)
> - [Microsoft.KeyVault](#microsoftkeyvault)
> - [Microsoft.Kubernetes](#microsoftkubernetes)
> - [Microsoft.KubernetesConfiguration](#microsoftkubernetesconfiguration)
> - [Microsoft.Kusto](#microsoftkusto)
> - [Microsoft.LabServices](#microsoftlabservices)
> - [Microsoft.LocationBasedServices](#microsoftlocationbasedservices)
> - [Microsoft.LocationServices](#microsoftlocationservices)
> - [Microsoft.Logic](#microsoftlogic)
> - [Microsoft.MachineLearning](#microsoftmachinelearning)
> - [Microsoft.MachineLearningCompute](#microsoftmachinelearningcompute)
> - [Microsoft.MachineLearningExperimentation](#microsoftmachinelearningexperimentation)
> - [Microsoft.MachineLearningModelManagement](#microsoftmachinelearningmodelmanagement)
> - [Microsoft.MachineLearningServices](#microsoftmachinelearningservices)
> - [Microsoft.Maintenance](#microsoftmaintenance)
> - [Microsoft.ManagedIdentity](#microsoftmanagedidentity)
> - [Microsoft.ManagedNetwork](#microsoftmanagednetwork)
> - [Microsoft.ManagedServices](#microsoftmanagedservices)
> - [Microsoft.Management](#microsoftmanagement)
> - [Microsoft.Maps](#microsoftmaps)
> - [Microsoft.Marketplace](#microsoftmarketplace)
> - [Microsoft.MarketplaceApps](#microsoftmarketplaceapps)
> - [Microsoft.MarketplaceOrdering](#microsoftmarketplaceordering)
> - [Microsoft.Media](#microsoftmedia)
> - [Microsoft.Microservices4Spring](#microsoftmicroservices4spring)
> - [Microsoft.Migrate](#microsoftmigrate)
> - [Microsoft.MixedReality](#microsoftmixedreality)
> - [Microsoft.NetApp](#microsoftnetapp)
> - [Microsoft.Network](#microsoftnetwork)
> - [Microsoft.NotificationHubs](#microsoftnotificationhubs)
> - [Microsoft.ObjectStore](#microsoftobjectstore)
> - [Microsoft.OffAzure](#microsoftoffazure)
> - [Microsoft.OperationalInsights](#microsoftoperationalinsights)
> - [Microsoft.OperationsManagement](#microsoftoperationsmanagement)
> - [Microsoft.Peering](#microsoftpeering)
> - [Microsoft.PolicyInsights](#microsoftpolicyinsights)
> - [Microsoft.Portal](#microsoftportal)
> - [Microsoft.PowerBI](#microsoftpowerbi)
> - [Microsoft.PowerBIDedicated](#microsoftpowerbidedicated)
> - [Microsoft.ProjectBabylon](#microsoftprojectbabylon)
> - [Microsoft.ProviderHub](#microsoftproviderhub)
> - [Microsoft.Quantum](#microsoftquantum)
> - [Microsoft.RecoveryServices](#microsoftrecoveryservices)
> - [Microsoft.RedHatOpenShift](#microsoftredhatopenshift)
> - [Microsoft.Relay](#microsoftrelay)
> - [Microsoft.ResourceGraph](#microsoftresourcegraph)
> - [Microsoft.ResourceHealth](#microsoftresourcehealth)
> - [Microsoft.Resources](#microsoftresources)
> - [Microsoft.SaaS](#microsoftsaas)
> - [Microsoft.Search](#microsoftsearch)
> - [Microsoft.Security](#microsoftsecurity)
> - [Microsoft.SecurityInsights](#microsoftsecurityinsights)
> - [Microsoft.SerialConsole](#microsoftserialconsole)
> - [Microsoft.ServerManagement](#microsoftservermanagement)
> - [Microsoft.ServiceBus](#microsoftservicebus)
> - [Microsoft.ServiceFabric](#microsoftservicefabric)
> - [Microsoft.ServiceFabricMesh](#microsoftservicefabricmesh)
> - [Microsoft.Services](#microsoftservices)
> - [Microsoft.SignalRService](#microsoftsignalrservice)
> - [Microsoft.SoftwarePlan](#microsoftsoftwareplan)
> - [Microsoft.Solutions](#microsoftsolutions)
> - [Microsoft.Sql](#microsoftsql)
> - [Microsoft.SqlVirtualMachine](#microsoftsqlvirtualmachine)
> - [Microsoft.Storage](#microsoftstorage)
> - [Microsoft.StorageCache](#microsoftstoragecache)
> - [Microsoft.StorageSync](#microsoftstoragesync)
> - [Microsoft.StorageSyncDev](#microsoftstoragesyncdev)
> - [Microsoft.StorageSyncInt](#microsoftstoragesyncint)
> - [Microsoft.StorSimple](#microsoftstorsimple)
> - [Microsoft.StreamAnalytics](#microsoftstreamanalytics)
> - [Microsoft.StreamAnalyticsExplorer](#microsoftstreamanalyticsexplorer)
> - [Microsoft.Subscription](#microsoftsubscription)
> - [microsoft.support](#microsoftsupport)
> - [Microsoft.Synapse](#microsoftsynapse)
> - [Microsoft.TimeSeriesInsights](#microsofttimeseriesinsights)
> - [Microsoft.Token](#microsofttoken)
> - [Microsoft.VirtualMachineImages](#microsoftvirtualmachineimages)
> - [microsoft.visualstudio](#microsoftvisualstudio)
> - [Microsoft.VMware](#microsoftvmware)
> - [Microsoft.VMwareCloudSimple](#microsoftvmwarecloudsimple)
> - [Microsoft.VnfManager](#microsoftvnfmanager)
> - [Microsoft.VSOnline](#microsoftvsonline)
> - [Microsoft.Web](#microsoftweb)
> - [Microsoft.WindowsESU](#microsoftwindowsesu)
> - [Microsoft.WindowsIoT](#microsoftwindowsiot)
> - [Microsoft.WorkloadBuilder](#microsoftworkloadbuilder)
> - [Microsoft.WorkloadMonitor](#microsoftworkloadmonitor)

## <a name="microsoftaad"></a>Microsoft.AAD

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | domainservices | No | No |  No |

## <a name="microsoftaadiam"></a>microsoft.aadiam

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | diagnosticsettings | No | No | No |
> | diagnosticsettingscategories | No | No | No |
> | privatelinkforazuread | Sí | Sí | No |
> | tenants | Sí | Sí | No |

## <a name="microsoftaddons"></a>Microsoft.Addons

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | supportproviders | No | No | No |

## <a name="microsoftadhybridhealthservice"></a>Microsoft.ADHybridHealthService

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | aadsupportcases | No | No | No |
> | addsservices | No | No | No |
> | agents | No | No | No |
> | anonymousapiusers | No | No | No |
> | configuración | No | No | No |
> | logs | No | No | No |
> | reports | No | No | No |
> | servicehealthmetrics | No | No | No |
> | services | No | No | No |

## <a name="microsoftadvisor"></a>Microsoft.Advisor

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | configuraciones | No | No | No |
> | generaterecommendations | No | No | No |
> | metadata | No | No | No |
> | de películas | No | No | No |
> | suppressions | No | No | No |

## <a name="microsoftalertsmanagement"></a>Microsoft.AlertsManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | actionRules | Sí | Sí | No |
> | alerts | No | No | No |
> | alertslist | No | No | No |
> | alertsmetadata | No | No | No |
> | alertssummary | No | No | No |
> | alertssummarylist | No | No | No |
> | smartdetectoralertrules | Sí | Sí | No |
> | smartgroups | No | No | No |

## <a name="microsoftanalysisservices"></a>Microsoft.AnalysisServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | servers | Sí | Sí | No |

## <a name="microsoftapimanagement"></a>Microsoft.ApiManagement

> [!IMPORTANT]
> Un servicio API Management establecido en la SKU de consumo no se puede mover.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | reportfeedback | No | No | No |
> | service | Sí | Sí | Sí (con una plantilla) <br/><br/> [Migración de API Management entre regiones](../../api-management/api-management-howto-migrate.md). |

## <a name="microsoftappconfiguration"></a>Microsoft.AppConfiguration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | configurationstores | Sí | Sí | No |
> | configurationstores/eventgridfilters | No | No | No |

## <a name="microsoftappplatform"></a>Microsoft.AppPlatform

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | spring | Sí | Sí | No |

## <a name="microsoftappservice"></a>Microsoft.AppService

> [!IMPORTANT]
> Vea [Guía de movimiento de App Service](./move-limitations/app-service-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | apiapps | No | No | Sí (con una plantilla)<br/><br/> [Traslado de una aplicación de App Service a otra región](../../app-service/manage-move-across-regions.md) |
> | appidentities | No | No | No |
> | gateways | No | No | No |

## <a name="microsoftattestation"></a>Microsoft.Attestation

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | attestationproviders | Sí | Sí | No |

## <a name="microsoftauthorization"></a>Microsoft.Authorization

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | classicadministrators | No | No | No |
> | dataaliases | No | No | No |
> | denyassignments | No | No | No |
> | elevateaccess | No | No | No |
> | findorphanroleassignments | No | No | No |
> | locks | No | No | No |
> | permisos | No | No | No |
> | policyassignments | No | No | No |
> | policydefinitions | No | No | No |
> | policysetdefinitions | No | No | No |
> | privatelinkassociations | No | No | No |
> | resourcemanagementprivatelinks | No | No | No |
> | roleassignments | No | No | No |
> | roleassignmentsusagemetrics | No | No | No |
> | roledefinitions | No | No | No |

## <a name="microsoftautomation"></a>Microsoft.Automation

> [!IMPORTANT]
> Los runbooks deben encontrarse en el mismo grupo de recursos que la cuenta de Automation.
>
> Para obtener más información, consulte [Traslado de la cuenta de Azure Automation a otra suscripción](../../automation/how-to/move-account.md?toc=/azure/azure-resource-manager/toc.json).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | automationaccounts | Sí | Sí | Sí (con una plantilla) <br/><br/> [Uso de la replicación geográfica](../../automation/automation-managing-data.md#geo-replication-in-azure-automation) |
> | automationaccounts/configurations | Sí | Sí | No |
> | automationaccounts/runbooks | Sí | Sí | No |

## <a name="microsoftavs"></a>Microsoft.AVS

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | privateclouds | Sí | Sí | No |

## <a name="microsoftazureactivedirectory"></a>Microsoft.AzureActiveDirectory

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | b2cdirectories | Sí | Sí | No |
> | b2ctenants | No | No | No |

## <a name="microsoftazuredata"></a>Microsoft.AzureData

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | datacontrollers | No | No | No |
> | hybriddatamanagers | No | No | No |
> | postgresinstances | No | No | No |
> | sqlinstances | No | No | No |
> | sqlmanagedinstances | No | No | No |
> | sqlserverinstances | No | No | No |
> | sqlserverregistrations | Sí | Sí | No |

## <a name="microsoftazurestack"></a>Microsoft.AzureStack

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | cloudmanifestfiles | No | No | No |
> | registrations | Sí | Sí | No |

## <a name="microsoftazurestackhci"></a>Microsoft.AzureStackHCI

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | clusters | No | No | No |

## <a name="microsoftbatch"></a>Microsoft.Batch

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | batchaccounts | Sí | Sí | Las cuentas de Batch no se pueden migrar directamente de una región a otra, pero puede usar una plantilla para exportar una plantilla, modificarla e implementarla en la nueva región. <br/><br/> Más información sobre [cómo mover una cuenta de Batch entre regiones](../../batch/best-practices.md#moving-batch-accounts-across-regions). |

## <a name="microsoftbilling"></a>Microsoft.Billing

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | billingaccounts | No | No | No |
> | billingperiods | No | No | No |
> | billingpermissions | No | No | No |
> | billingproperty | No | No | No |
> | billingroleassignments | No | No | No |
> | billingroledefinitions | No | No | No |
> | departments | No | No | No |
> | enrollmentaccounts | No | No | No |
> | invoices | No | No | No |
> | transfers | No | No | No |

## <a name="microsoftbingmaps"></a>Microsoft.BingMaps

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | mapapis | No | No | No |

## <a name="microsoftbiztalkservices"></a>Microsoft.BizTalkServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | biztalk | No | No | No |

## <a name="microsoftblockchain"></a>Microsoft.Blockchain

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | blockchainmembers | No | No | No <br/><br/> La red de cadena de bloques no puede tener nodos en regiones diferentes. |
> | cordamembers | No | No | No |
> | watchers | No | No | No |

## <a name="microsoftblockchaintokens"></a>Microsoft.BlockchainTokens

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | tokenservices | No | No | No |

## <a name="microsoftblueprint"></a>Microsoft.Blueprint

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | blueprintassignments | No | No | No |
> | blueprints | No | No | No |

## <a name="microsoftbotservice"></a>Microsoft.BotService

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | botservices | Sí | Sí | No |

## <a name="microsoftcache"></a>Microsoft.Cache

> [!IMPORTANT]
> Si la instancia de Azure Redis Cache está configurada con una red virtual, la instancia no se puede mover a otra suscripción. Consulte las [Limitaciones de movimiento de redes](./move-limitations/networking-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | redis | Sí | Sí | No |
> | redisenterprise | No | No | No |

## <a name="microsoftcapacity"></a>Microsoft.Capacity

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | appliedreservations | No | No | No |
> | calculateexchange | No | No | No |
> | calculateprice | No | No | No |
> | calculatepurchaseprice | No | No | No |
> | catalogs | No | No | No |
> | commercialreservationorders | No | No | No |
> | cambio | No | No | No |
> | reservationorders | No | No | No |
> | reservations | No | No | No |
> | resources | No | No | No |
> | validatereservationorder | No | No | No |

## <a name="microsoftcdn"></a>Microsoft.Cdn

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | cdnwebapplicationfirewallmanagedrulesets | No | No | No |
> | cdnwebapplicationfirewallpolicies | Sí | Sí | No |
> | edgenodes | No | No | No |
> | profiles | Sí | Sí | No |
> | profiles/endpoints | Sí | Sí | No |

## <a name="microsoftcertificateregistration"></a>Microsoft.CertificateRegistration

> [!IMPORTANT]
> Vea [Guía de movimiento de App Service](./move-limitations/app-service-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | certificateorders | Sí | Sí | No |

## <a name="microsoftclassiccompute"></a>Microsoft.ClassicCompute

> [!IMPORTANT]
> Vea [Guía de movimiento de implementación clásica](./move-limitations/classic-model-move-limitations.md). Los recursos de implementación clásica se pueden mover entre suscripciones con una operación específica de ese escenario.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | capabilities | No | No | No |
> | domainnames | Sí | No | No |
> | quotas | No | No | No |
> | resourcetypes | No | No | No |
> | validatesubscriptionmoveavailability | No | No | No |
> | virtualmachines | Sí | Sí | No |

## <a name="microsoftclassicinfrastructuremigrate"></a>Microsoft.ClassicInfrastructureMigrate

> [!IMPORTANT]
> Vea [Guía de movimiento de implementación clásica](./move-limitations/classic-model-move-limitations.md). Los recursos de implementación clásica se pueden mover entre suscripciones con una operación específica de ese escenario.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | classicinfrastructureresources | No | No | No |

## <a name="microsoftclassicnetwork"></a>Microsoft.ClassicNetwork

> [!IMPORTANT]
> Vea [Guía de movimiento de implementación clásica](./move-limitations/classic-model-move-limitations.md). Los recursos de implementación clásica se pueden mover entre suscripciones con una operación específica de ese escenario.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | capabilities | No | No | No |
> | expressroutecrossconnections | No | No | No |
> | expressroutecrossconnections/peerings | No | No | No |
> | gatewaysupporteddevices | No | No | No |
> | networksecuritygroups | No | No | No |
> | quotas | No | No | No |
> | reservedips | No | No | No |
> | virtualnetworks | No | No | No |

## <a name="microsoftclassicstorage"></a>Microsoft.ClassicStorage

> [!IMPORTANT]
> Vea [Guía de movimiento de implementación clásica](./move-limitations/classic-model-move-limitations.md). Los recursos de implementación clásica se pueden mover entre suscripciones con una operación específica de ese escenario.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | disks | No | No | No |
> | images | No | No | No |
> | osimages | No | No | No |
> | osplatformimages | No | No | No |
> | publicimages | No | No | No |
> | quotas | No | No | No |
> | storageaccounts | Sí | No | Sí |
> | vmimages | No | No | No |

## <a name="microsoftclassicsubscription"></a>Microsoft.ClassicSubscription

> [!IMPORTANT]
> Vea [Guía de movimiento de implementación clásica](./move-limitations/classic-model-move-limitations.md). Los recursos de implementación clásica se pueden mover entre suscripciones con una operación específica de ese escenario.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | operaciones | No | No | No |

## <a name="microsoftcognitiveservices"></a>Microsoft.CognitiveServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | accounts | Sí | Sí | No |
> | Cognitive Search | **pendiente** | **pendiente** | Se admite con pasos manuales.<br/><br/> Más información sobre cómo [mover el servicio Azure Cognitive Search a otra región](../../search/search-howto-move-across-regions.md). |

## <a name="microsoftcommerce"></a>Microsoft.Commerce

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | ratecard | No | No | No |
> | usageaggregates | No | No | No |

## <a name="microsoftcompute"></a>Microsoft.Compute

> [!IMPORTANT]
> Vea [Guía de movimiento de Virtual Machines](./move-limitations/virtual-machines-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | availabilitysets | Sí | Sí |  Sí <br/><br/> Use [Azure Resource Mover](../../resource-mover/tutorial-move-region-virtual-machines.md) para mover conjuntos de disponibilidad. |
> | diskaccesses | No | No | No |
> | diskencryptionsets | No | No | No |
> | disks | Sí | Sí | Sí <br/><br/> Use [Azure Resource Mover](../../resource-mover/tutorial-move-region-virtual-machines.md) para trasladar máquinas virtuales de Azure y discos relacionados. |
> | galleries | No | No | No |
> | galleries/images | No | No | No |
> | galleries/images/versions | No | No | No |
> | hostgroups | No | No | No |
> | hostgroups/hosts | No | No | No |
> | images | Sí | Sí | No |
> | proximityplacementgroups | Sí | Sí | No |
> | restorepointcollections | No | No | No |
> | restorepointcollections/restorepoints | No | No | No |
> | sharedvmextensions | No | No | No |
> | sharedvmimages | No | No | No |
> | sharedvmimages/versions | No | No | No |
> | snapshots | Sí | Sí | No |
> | sshpublickeys | No | No | No |
> | virtualmachines | Sí | Sí | Sí <br/><br/> Use [Azure Resource Mover](../../resource-mover/tutorial-move-region-virtual-machines.md) para trasladar máquinas virtuales de Azure. |
> | virtualmachines/extensions | Sí | Sí | No |
> | virtualmachinescalesets | Sí | Sí | No |

## <a name="microsoftconsumption"></a>Microsoft.Consumption

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | aggregatedcost | No | No | No |
> | balances | No | No | No |
> | budgets | No | No | No |
> | charges | No | No | No |
> | costtags | No | No | No |
> | credits | No | No | No |
> | events | No | No | No |
> | forecasts | No | No | No |
> | lots | No | No | No |
> | marketplaces | No | No | No |
> | pricesheets | No | No | No |
> | products | No | No | No |
> | reservationdetails | No | No | No |
> | reservationrecommendationdetails | No | No | No |
> | reservationrecommendations | No | No | No |
> | reservationsummaries | No | No | No |
> | reservationtransactions | No | No | No |
> | etiquetas | No | No | No |
> | tenants | No | No | No |
> | terms | No | No | No |
> | usagedetails | No | No | No |

## <a name="microsoftcontainerinstance"></a>Microsoft.ContainerInstance

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | containergroups | No | No | No |
> | serviceassociationlinks | No | No | No |

## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | registries | Sí | Sí | No |
> | registries / agentpools | Sí | Sí | No |
> | registries/buildtasks | Sí | Sí | No |
> | registries/replications | Sí | Sí | No |
> | registries/tasks | Sí | Sí | No |
> | registries/webhooks | Sí | Sí | No |

## <a name="microsoftcontainerservice"></a>Microsoft.ContainerService

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | containerservices | No | No | No |
> | managedclusters | No | No | No |
> | openshiftmanagedclusters | No | No | No |

## <a name="microsoftcontentmoderator"></a>Microsoft.ContentModerator

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | applications | No | No | No |

## <a name="microsoftcortanaanalytics"></a>Microsoft.CortanaAnalytics

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | accounts | No | No | No |

## <a name="microsoftcostmanagement"></a>Microsoft.CostManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | alerts | No | No | No |
> | billingaccounts | No | No | No |
> | budgets | No | No | No |
> | cloudconnectors | No | No | No |
> | conectores | Sí | Sí | No |
> | departments | No | No | No |
> | dimensions | No | No | No |
> | enrollmentaccounts | No | No | No |
> | exports | No | No | No |
> | externalbillingaccounts | No | No | No |
> | forecast | No | No | No |
> | Query | No | No | No |
> | registro | No | No | No |
> | reportconfigs | No | No | No |
> | reports | No | No | No |
> | configuración | No | No | No |
> | showbackrules | No | No | No |
> | views | No | No | No |

## <a name="microsoftcustomerinsights"></a>Microsoft.CustomerInsights

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | hubs | No | No | No |

## <a name="microsoftcustomerlockbox"></a>Microsoft.CustomerLockbox

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | Solicitudes | No | No | No |

## <a name="microsoftcustomproviders"></a>Microsoft.CustomProviders

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | associations | No | No | No |
> | resourceproviders | Sí | Sí | No |

## <a name="microsoftdatabox"></a>Microsoft.DataBox

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | jobs | No | No | No |

## <a name="microsoftdataboxedge"></a>Microsoft.DataBoxEdge

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | availableskus | No | No | No |
> | databoxedgedevices | No | No | No |

## <a name="microsoftdatabricks"></a>Microsoft.Databricks

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | workspaces | No | No | No |

## <a name="microsoftdatacatalog"></a>Microsoft.DataCatalog

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | catalogs | Sí | Sí | No |
> | datacatalogs | No | No | No |

## <a name="microsoftdataconnect"></a>Microsoft.DataConnect

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | connectionmanagers | No | No | No |

## <a name="microsoftdataexchange"></a>Microsoft.DataExchange

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | packages | No | No | No |
> | plans | No | No | No |

## <a name="microsoftdatafactory"></a>Microsoft.DataFactory

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | datafactories | Sí | Sí | No |
> | factories | Sí | Sí | No |

## <a name="microsoftdatalake"></a>Microsoft.DataLake

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | datalakeaccounts | No | No | No |

## <a name="microsoftdatalakeanalytics"></a>Microsoft.DataLakeAnalytics

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | accounts | Sí | Sí | No |

## <a name="microsoftdatalakestore"></a>Microsoft.DataLakeStore

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | accounts | Sí | Sí | No |

## <a name="microsoftdatamigration"></a>Microsoft.DataMigration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | services | No | No | No |
> | services/projects | No | No | No |
> | slots | No | No | No |

## <a name="microsoftdataprotection"></a>Microsoft.DataProtection

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ---------- |
> | backupvaults | No | No | No |

## <a name="microsoftdatashare"></a>Microsoft.DataShare

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | accounts | Sí | Sí | No |

## <a name="microsoftdbformariadb"></a>Microsoft.DBforMariaDB

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | servers | Sí | Sí | Puede usar una réplica de lectura entre regiones para trasladar un servidor existente. [Más información](../../postgresql/howto-move-regions-portal.md).<br/><br/> Si el servicio se aprovisiona con almacenamiento de copia de seguridad con redundancia geográfica, puede usar la restauración geográfica para restaurar en otras regiones. [Más información](../../mariadb/concepts-business-continuity.md#recover-from-an-azure-regional-data-center-outage).

## <a name="microsoftdbformysql"></a>Microsoft.DBforMySQL

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | flexibleServers | No | No | No |
> | servers | Sí | Sí | Puede usar una réplica de lectura entre regiones para trasladar un servidor existente. [Más información](../../mysql/howto-move-regions-portal.md).

## <a name="microsoftdbforpostgresql"></a>Microsoft.DBforPostgreSQL

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | flexibleServers | No | No | No |
> | servergroups | No | No | No |
> | servers | Sí | Sí | Puede usar una réplica de lectura entre regiones para trasladar un servidor existente. [Más información](../../postgresql/howto-move-regions-portal.md).
> | serversv2 | Sí | Sí | No |
> | singleservers | Sí | Sí | No |

## <a name="microsoftdeploymentmanager"></a>Microsoft.DeploymentManager

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | artifactsources | Sí | Sí | No |
> | rollouts | Sí | Sí | No |
> | servicetopologies | Sí | Sí | No |
> | servicetopologies/services | Sí | Sí | No |
> | servicetopologies/services/serviceunits | Sí | Sí | No |
> | steps | Sí | Sí | No |

## <a name="microsoftdesktopvirtualization"></a>Microsoft.DesktopVirtualization

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | applicationgroups | Sí | Sí | No |
> | hostpools | Sí | Sí | No |
> | workspaces | Sí | Sí | No |

## <a name="microsoftdevices"></a>Microsoft.Devices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | elasticpools | No | No | No. El recurso no está expuesto. |
> | elasticpools/iothubtenants | No | No | No. El recurso no está expuesto. |
> | iothubs | Sí | Sí | Sí. [Más información](../../iot-hub/iot-hub-how-to-clone.md) |
> | provisioningservices | Sí | Sí | No |

## <a name="microsoftdevops"></a>Microsoft.DevOps

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | pipelines | Sí | Sí | No |
> | controllers | **pendiente** | **pendiente** | No |

## <a name="microsoftdevspaces"></a>Microsoft.DevSpaces

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | controllers | Sí | Sí | No |
> | Clúster de AKS | **pendiente** | **pendiente** | No<br/><br/> [Más información](../../dev-spaces/faq.md#can-i-migrate-my-aks-cluster-with-azure-dev-spaces-to-another-region) sobre el movimiento a otra región.

## <a name="microsoftdevtestlab"></a>Microsoft.DevTestLab

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | labcenters | No | No | No |
> | labs | Sí | No | No |
> | labs/environments | Sí | Sí | No |
> | labs/servicerunners | Sí | Sí | No |
> | labs/virtualmachines | Sí | No | No |
> | schedules | Sí | Sí | No |

## <a name="microsoftdigitaltwins"></a>Microsoft.DigitalTwins

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | digitaltwinsinstances | No | No | Sí, se vuelven a crear los recursos en una nueva región. [Más información](../../digital-twins/how-to-move-regions.md) |

## <a name="microsoftdocumentdb"></a>Microsoft.DocumentDB

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | databaseaccountnames | No | No | No |
> | databaseaccounts | Sí | Sí | No |

## <a name="microsoftdomainregistration"></a>Microsoft.DomainRegistration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | domains | Sí | Sí | No |
> | generatessorequest | No | No | No |
> | topleveldomains | No | No | No |
> | validatedomainregistrationinformation | No | No | No |

## <a name="microsoftenterpriseknowledgegraph"></a>Microsoft.EnterpriseKnowledgeGraph

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | services | Sí | Sí | No |

## <a name="microsofteventgrid"></a>Microsoft.EventGrid

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | domains | Sí | Sí | No |
> | eventsubscriptions | No: no se puede mover de forma independiente, sino que se hace automáticamente con el recurso suscrito. | No: no se puede mover de forma independiente, sino que se hace automáticamente con el recurso suscrito. | No |
> | extensiontopics | No | No | No |
> | partnernamespaces | Sí | Sí | No |
> | partnerregistrations | No | No | No |
> | partnertopics | Sí | Sí | No |
> | systemtopics | Sí | Sí | No |
> | topics | Sí | Sí | No |
> | topictypes | No | No | No |

## <a name="microsofteventhub"></a>Microsoft.EventHub

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | clusters | Sí | Sí | No |
> | espacios de nombres | Sí | Sí | Sí (con una plantilla)<br/><br/> [Movimiento de un espacio de nombres de Event Hubs a otra región](../../event-hubs/move-across-regions.md) |
> | sku | No | No | No |

## <a name="microsoftexperimentation"></a>Microsoft.Experimentation

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | experimentworkspaces | No | No | No |

## <a name="microsoftfalcon"></a>Microsoft.Falcon

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | espacios de nombres | Sí | Sí | No |

## <a name="microsoftfeatures"></a>Microsoft.Features

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | featureproviders | No | No | No |
> | features | No | No | No |
> | providers | No | No | No |
> | subscriptionfeatureregistrations | No | No | No |

## <a name="microsoftgenomics"></a>Microsoft.Genomics

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | accounts | No | No | No |

## <a name="microsoftguestconfiguration"></a>Microsoft.GuestConfiguration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | automanagedaccounts | No | No | No |
> | automanagedvmconfigurationprofiles | No | No | No |
> | guestconfigurationassignments | No | No | No |
> | software | No | No | No |
> | softwareupdateprofile | No | No | No |
> | softwareupdates | No | No | No |

## <a name="microsofthanaonazure"></a>Microsoft.HanaOnAzure

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | hanainstances | No | No | No |
> | sapmonitors | No | No | No |

## <a name="microsofthardwaresecuritymodules"></a>Microsoft.HardwareSecurityModules

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | dedicatedhsms | No | No | No |

## <a name="microsofthdinsight"></a>Microsoft.HDInsight

> [!IMPORTANT]
> Puede mover clústeres de HDInsight a una nueva suscripción o un nuevo grupo de recursos. Sin embargo, no puede mover entre suscripciones los recursos de red vinculados al clúster de HDInsight (por ejemplo, la red virtual, una NIC o un equilibrador de carga). Además, tampoco se puede mover a un nuevo grupo de recursos una NIC que está conectada a una máquina virtual del clúster.
>
> Al mover un clúster de HDInsight a una nueva suscripción, mueva primero otros recursos (por ejemplo, la cuenta de almacenamiento). Después, mover el clúster de HDInsight.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | clusters | Sí | Sí | No |

## <a name="microsofthealthcareapis"></a>Microsoft.HealthcareApis

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | services | Sí | Sí | No |

## <a name="microsofthybridcompute"></a>Microsoft.HybridCompute

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | machines | Sí | Sí | No |
> | machines/extensions | Sí | Sí | No |

## <a name="microsofthybriddata"></a>Microsoft.HybridData

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | datamanagers | Sí | Sí | No |

## <a name="microsofthybridnetwork"></a>Microsoft.HybridNetwork

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | devices | No | No | No |
> | vnfs | No | No | No |

## <a name="microsofthydra"></a>Microsoft.Hydra

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | components | No | No | No |
> | networkscopes | No | No | No |

## <a name="microsoftimportexport"></a>Microsoft.ImportExport

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | jobs | Sí | Sí | No |

## <a name="microsoftinsights"></a>Microsoft.Insights

> [!IMPORTANT]
> Asegúrese de que el movimiento a una nueva suscripción no exceda las [cuotas de suscripción](azure-subscription-service-limits.md#azure-monitor-limits).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | accounts | **pendiente** | **pendiente** | No. [Más información](../../azure-monitor/faq.md#how-do-i-move-an-application-insights-resource-to-a-new-region). |
> | actiongroups | Sí | Sí | No |
> | activitylogalerts | No | No | No |
> | alertrules | Sí | Sí | No |
> | autoscalesettings | Sí | Sí | No |
> | baseline | No | No | No |
> | components | Sí | Sí | No |
> | datacollectionrules | No | No | No |
> | diagnosticsettings | No | No | No |
> | diagnosticsettingscategories | No | No | No |
> | eventcategories | No | No | No |
> | eventtypes | No | No | No |
> | extendeddiagnosticsettings | No | No | No |
> | guestdiagnosticsettings | No | No | No |
> | listmigrationdate | No | No | No |
> | logdefinitions | No | No | No |
> | logprofiles | No | No | No |
> | logs | No | No | No |
> | metricalerts | No | No | No |
> | metricbaselines | No | No | No |
> | metricbatch | No | No | No |
> | metricdefinitions | No | No | No |
> | metricnamespaces | No | No | No |
> | Métricas | No | No | No |
> | migratealertrules | No | No | No |
> | migratetonewpricingmodel | No | No | No |
> | myworkbooks | No | No | No |
> | notificationgroups | No | No | No |
> | privatelinkscopes | No | No | No |
> | rollbacktolegacypricingmodel | No | No | No |
> | scheduledqueryrules | Sí | Sí | No |
> | Topología | No | No | No |
> | transacciones | No | No | No |
> | vminsightsonboardingstatuses | No | No | No |
> | webtests | Sí | Sí | No |
> | webtests/gettestresultfile | No | No | No |
> | workbooks | Sí | Sí | No |
> | workbooktemplates | Sí | Sí | No |

## <a name="microsoftiotcentral"></a>Microsoft.IoTCentral

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | apptemplates | No | No | No |
> | iotapps | Sí | Sí | No |

## <a name="microsoftiothub"></a>Microsoft.IoTHub

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | iothub | **pendiente** | **pendiente** | Sí (clonación del centro) <br/><br/> [Clonación de una instancia de IoT Hub en otra región](../../iot-hub/iot-hub-how-to-clone.md) |

## <a name="microsoftiotspaces"></a>Microsoft.IoTSpaces

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | graph | Sí | Sí | No |

## <a name="microsoftkeyvault"></a>Microsoft.KeyVault

> [!IMPORTANT]
> Los almacenes de claves usados para el cifrado de discos no se pueden mover a grupos de recursos de la misma suscripción ni entre suscripciones.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | deletedvaults | No | No | No |
> | hsmpools | No | No | No |
> | managedhsms | No | No | No |
> | vaults | Sí | Sí | No |

## <a name="microsoftkubernetes"></a>Microsoft.Kubernetes

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | connectedclusters | Sí | Sí | No |
> | registeredsubscriptions | No | No | No |

## <a name="microsoftkubernetesconfiguration"></a>Microsoft.KubernetesConfiguration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | sourcecontrolconfigurations | No | No | No |

## <a name="microsoftkusto"></a>Microsoft.Kusto

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | clusters | Sí | Sí | No |

## <a name="microsoftlabservices"></a>Microsoft.LabServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | labaccounts | No | No | No |
> | users | No | No | No |

## <a name="microsoftlocationbasedservices"></a>Microsoft.LocationBasedServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | accounts | No | No | No |

## <a name="microsoftlocationservices"></a>Microsoft.LocationServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | accounts | No | No | No, es un servicio global. |

## <a name="microsoftlogic"></a>Microsoft.Logic

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | hostingenvironments | No | No | No |
> | integrationaccounts | Sí | Sí | No |
> | integrationserviceenvironments | Sí | No | No |
> | integrationserviceenvironments / managedapis | Sí | No | No |
> | isolatedenvironments | No | No | No |
> | workflows | Sí | Sí | No |

## <a name="microsoftmachinelearning"></a>Microsoft.MachineLearning

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | commitmentplans | No | No | No |
> | webservices | Sí | No | No |
> | workspaces | Sí | Sí | No |

## <a name="microsoftmachinelearningcompute"></a>Microsoft.MachineLearningCompute

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | operationalizationclusters | No | No | No |

## <a name="microsoftmachinelearningexperimentation"></a>Microsoft.MachineLearningExperimentation

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | accounts | No | No | No |
> | teamaccounts | No | No | No |

## <a name="microsoftmachinelearningmodelmanagement"></a>Microsoft.MachineLearningModelManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | accounts | No | No | No |

## <a name="microsoftmachinelearningservices"></a>Microsoft.MachineLearningServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | workspaces | No | No | No |

## <a name="microsoftmaintenance"></a>Microsoft.Maintenance

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | configurationassignments | No | No | Sí. [Más información](../../virtual-machines/move-region-maintenance-configuration.md) |
> | maintenanceconfigurations | Sí | Sí | Sí. [Más información](../../virtual-machines/move-region-maintenance-configuration-resources.md) |
> | updates | No | No | No |

## <a name="microsoftmanagedidentity"></a>Microsoft.ManagedIdentity

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | Identidades | No | No | No |
> | userassignedidentities | No | No | No |

## <a name="microsoftmanagednetwork"></a>Microsoft.ManagedNetwork

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | managednetworks | No | No | No |
> | managednetworks / managednetworkgroups | No | No | No |
> | managednetworks / managednetworkpeeringpolicies | No | No | No |
> | notificación | No | No | No |

## <a name="microsoftmanagedservices"></a>Microsoft.ManagedServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | marketplaceregistrationdefinitions | No | No | No |
> | registrationassignments | No | No | No |
> | registrationdefinitions | No | No | No |

## <a name="microsoftmanagement"></a>Microsoft.Management

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | getentities | No | No | No |
> | managementgroups | No | No | No |
> | managementgroups/settings | No | No | No |
> | resources | No | No | No |
> | starttenantbackfill | No | No | No |
> | tenantbackfillstatus | No | No | No |

## <a name="microsoftmaps"></a>Microsoft.Maps

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | accounts | Sí | Sí | No, Azure Maps es un servicio geoespacial. |
> | accounts / privateatlases | Sí | Sí | No |

## <a name="microsoftmarketplace"></a>Microsoft.Marketplace

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | offers | No | No | No |
> | offertypes | No | No | No |
> | privategalleryitems | No | No | No |
> | privatestoreclient | No | No | No |
> | privatestores | No | No | No |
> | products | No | No | No |
> | publishers | No | No | No |
> | registro | No | No | No |

## <a name="microsoftmarketplaceapps"></a>Microsoft.MarketplaceApps

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | classicdevservices | No | No | No |

## <a name="microsoftmarketplaceordering"></a>Microsoft.MarketplaceOrdering

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | agreements | No | No | No |
> | offertypes | No | No | No |

## <a name="microsoftmedia"></a>Microsoft.Media

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | mediaservices | Sí | Sí | No |
> | mediaservices/liveevents | Sí | Sí | No |
> | mediaservices/streamingendpoints | Sí | Sí | No |

## <a name="microsoftmicroservices4spring"></a>Microsoft.Microservices4Spring

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | appclusters | No | No | No |

## <a name="microsoftmigrate"></a>Microsoft.Migrate

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | assessmentprojects | No | No | No |
> | migrateprojects | No | No | No |
> | movecollections | No | No | No |
> | projects | No | No | No |

## <a name="microsoftmixedreality"></a>Microsoft.MixedReality

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ---------- |
> | holographicsbroadcastaccounts | No | No | No |
> | objectunderstandingaccounts | No | No | No |
> | remoterenderingaccounts | Sí | Sí | No |
> | spatialanchorsaccounts | Sí | Sí | No |

## <a name="microsoftnetapp"></a>Microsoft.NetApp

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | netappaccounts | No | No | No |
> | netappaccounts/capacitypools | No | No | No |
> | netappaccounts/capacitypools/volumes | No | No | No |
> | netappaccounts/capacitypools/volumes/mounttargets | No | No | No |
> | netappaccounts/capacitypools/volumes/snapshots | No | No | No |

## <a name="microsoftnetwork"></a>Microsoft.Network

> [!IMPORTANT]
> Consulte [Guía de movimiento de redes](./move-limitations/networking-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | applicationgateways | No | No | No |
> | applicationgatewaywebapplicationfirewallpolicies | No | No | No |
> | applicationsecuritygroups | Sí | Sí | No |
> | azurefirewalls | No | No | No |
> | bastionhosts | No | No | No |
> | bgpservicecommunities | No | No | No |
> | connections | Sí | Sí | No |
> | ddoscustompolicies | Sí | Sí | No |
> | ddosprotectionplans | No | No | No |
> | dnszones | Sí | Sí | No |
> | expressroutecircuits | No | No | No |
> | expressroutegateways | No | No | No |
> | expressrouteserviceproviders | No | No | No |
> | firewallpolicies | Sí | Sí | No |
> | frontdoors | No | No | No |
> | ipallocations | Sí | Sí | No |
> | ipgroups | Sí | Sí | No |
> | loadbalancers | Sí: SKU básico<br> Sí: SKU estándar | Sí: SKU básico<br>No: SKU estándar | Sí <br/><br/> Use [Azure Resource Mover](../../resource-mover/tutorial-move-region-virtual-machines.md) para trasladar equilibradores de carga externos e internos. |
> | localnetworkgateways | Sí | Sí | No |
> | natgateways | No | No | No |
> | networkexperimentprofiles | No | No | No |
> | networkintentpolicies | Sí | Sí | No |
> | networkinterfaces | Sí | Sí | Sí <br/><br/> Use [Azure Resource Mover](../../resource-mover/tutorial-move-region-virtual-machines.md) para trasladar adaptadores de red. |
> | networkprofiles | No | No | No |
> | networksecuritygroups | Sí | Sí | Sí <br/><br/> Use [Azure Resource Mover](../../resource-mover/tutorial-move-region-virtual-machines.md) para trasladar los grupos de seguridad de red (NGS). |
> | networkwatchers | No | No | No |
> | networkwatchers/connectionmonitors | Sí | No | No |
> | networkwatchers / flowlogs | Sí | No | No |
> | networkwatchers/pingmeshes | Sí | No | No |
> | p2svpngateways | No | No | No |
> | privatednszones | Sí | Sí | No |
> | privatednszones/virtualnetworklinks | Sí | Sí | No |
> | privatednszonesinternal | No | No | No |
> | privateendpointredirectmaps | No | No | No |
> | privateendpoints | No | No | No |
> | privatelinkservices | No | No | No |
> | publicipaddresses | Sí: SKU básico<br>Sí: SKU estándar | Sí: SKU básico<br>No: SKU estándar | Sí<br/><br/> Use [Azure Resource Mover](../../resource-mover/tutorial-move-region-virtual-machines.md) para trasladar direcciones IP públicas. |
> | publicipprefixes | Sí | Sí | No |
> | routefilters | No | No | No |
> | routetables | Sí | Sí | No |
> | securitypartnerproviders | Sí | Sí | No |
> | serviceendpointpolicies | Sí | Sí | No |
> | trafficmanagergeographichierarchies | No | No | No |
> | trafficmanagerprofiles | Sí | Sí | No |
> | trafficmanagerprofiles/heatmaps | No | No | No |
> | trafficmanagerusermetricskeys | No | No | No |
> | virtualhubs | No | No | No |
> | virtualnetworkgateways | Sí | Sí | No |
> | virtualnetworks | Sí | Sí | No |
> | virtualnetworktaps | No | No | No |
> | virtualrouters | Sí | Sí | No |
> | virtualwans | No | No |
> | vpngateways (Virtual WAN) | No | No | No |
> | vpnserverconfigurations | No | No | No |
> | vpnsites (Virtual WAN) | No | No | No |

## <a name="microsoftnotificationhubs"></a>Microsoft.NotificationHubs

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | espacios de nombres | Sí | Sí | No |
> | namespaces/notificationhubs | Sí | Sí | No |

## <a name="microsoftobjectstore"></a>Microsoft.ObjectStore

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | osnamespaces | Sí | Sí | No |

## <a name="microsoftoffazure"></a>Microsoft.OffAzure

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | hypervsites | No | No | No |
> | importsites | No | No | No |
> | serversites | No | No | No |
> | vmwaresites | No | No | No |

## <a name="microsoftoperationalinsights"></a>Microsoft.OperationalInsights

> [!IMPORTANT]
> Asegúrese de que el movimiento a una nueva suscripción no exceda las [cuotas de suscripción](azure-subscription-service-limits.md#azure-monitor-limits).
>
> Las áreas de trabajo que tienen una cuenta de automatización vinculada no se pueden mover. Antes de comenzar una operación de movimiento, asegúrese de desvincular todas las cuentas de automatización.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | clusters | No | No | No |
> | deletedworkspaces | No | No | No |
> | linktargets | No | No | No |
> | storageinsightconfigs | No | No | No |
> | workspaces | Sí | Sí | No |

## <a name="microsoftoperationsmanagement"></a>Microsoft.OperationsManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | managementassociations | No | No | No |
> | managementconfigurations | Sí | Sí | No |
> | solutions | Sí | Sí | No |
> | views | Sí | Sí | No |

## <a name="microsoftpeering"></a>Microsoft.Peering

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | legacypeerings | No | No | No |
> | peerasns | No | No | No |
> | peeringlocations | No | No | No |
> | peerings | No | No | No |
> | peeringservicecountries | No | No | No |
> | peeringservicelocations | No | No | No |
> | peeringserviceproviders | No | No | No |
> | peeringservices | No | No | No |

## <a name="microsoftpolicyinsights"></a>Microsoft.PolicyInsights

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | policyevents | No | No | No |
> | policystates | No | No | No |
> | policytrackedresources | No | No | No |
> | remediations | No | No | No |

## <a name="microsoftportal"></a>Microsoft.Portal

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | consoles | No | No | No |
> | dashboards | Sí | Sí | No |
> | usersettings | No | No | No |

## <a name="microsoftpowerbi"></a>Microsoft.PowerBI

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | workspacecollections | Sí | Sí | No |

## <a name="microsoftpowerbidedicated"></a>Microsoft.PowerBIDedicated

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | capacities | Sí | Sí | No |

## <a name="microsoftprojectbabylon"></a>Microsoft.ProjectBabylon

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ---------- |
> | accounts | No | No | No |

## <a name="microsoftpurview"></a>Microsoft.Purview

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ---------- |
> | accounts | **pendiente** | **pendiente** | No |

## <a name="microsoftproviderhub"></a>Microsoft.ProviderHub

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | availableaccounts | No | No | No |
> | providerregistrations | No | No | No |
> | rollouts | No | No | No |

## <a name="microsoftquantum"></a>Microsoft.Quantum

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | workspaces | No | No | No |

## <a name="microsoftrecoveryservices"></a>Microsoft.RecoveryServices

> [!IMPORTANT]
> Vea [Guía de movimiento de Recovery Services](../../backup/backup-azure-move-recovery-services-vault.md?toc=/azure/azure-resource-manager/toc.json).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | replicationeligibilityresults | No | No | No |
> | vaults | Sí | Sí | No.<br/><br/> No se admite la migración de almacenes de Recovery Services para Azure Backup entre regiones de Azure.<br/><br/> En almacenes de Recovery Services para Azure Site Recovery, puede [deshabilitar y volver a crear el almacén](../../site-recovery/move-vaults-across-regions.md) en la región de destino. |

## <a name="microsoftredhatopenshift"></a>Microsoft.RedHatOpenShift

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | openshiftclusters | No | No | No |

## <a name="microsoftrelay"></a>Microsoft.Relay

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | espacios de nombres | Sí | Sí | No |

## <a name="microsoftresourcegraph"></a>Microsoft.ResourceGraph

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | Consultas | Sí | Sí | No |
> | resourcechangedetails | No | No | No |
> | resourcechanges | No | No | No |
> | resources | No | No | No |
> | resourceshistory | No | No | No |
> | subscriptionsstatus | No | No | No |

## <a name="microsoftresourcehealth"></a>Microsoft.ResourceHealth

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | childresources | No | No | No |
> | emergingissues | No | No | No |
> | events | No | No | No |
> | metadata | No | No | No |
> | Notificaciones | No | No | No |

## <a name="microsoftresources"></a>Microsoft.Resources

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | deployments | No | No | No |
> | deploymentscripts | No | No | Sí<br/><br/>[Mover recursos de Microsoft.Resources a nueva región](microsoft-resources-move-regions.md) |
> | deploymentscripts/logs | No | No | No |
> | vínculos | No | No | No |
> | providers | No | No | No |
> | resourcegroups | No | No | No |
> | resources | No | No | No |
> | subscriptions | No | No | No |
> | etiquetas | No | No | No |
> | templatespecs | No | No | Sí<br/><br/>[Mover recursos de Microsoft.Resources a nueva región](microsoft-resources-move-regions.md) |
> | templatespecs/versions | No | No | No |
> | tenants | No | No | No |

## <a name="microsoftsaas"></a>Microsoft.SaaS

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | applications | Sí | No | No |
> | saasresources | No | No | No |

## <a name="microsoftsearch"></a>Microsoft.Search

> [!IMPORTANT]
> No puede mover varios recursos de Search de regiones diferentes en una operación. En su lugar, muévalos en operaciones independientes.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | resourcehealthmetadata | No | No | No |
> | searchservices | Sí | Sí | No |

## <a name="microsoftsecurity"></a>Microsoft.Security

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | adaptivenetworkhardenings | No | No | No |
> | advancedthreatprotectionsettings | No | No | No |
> | alerts | No | No | No |
> | allowedconnections | No | No | No |
> | applicationwhitelistings | No | No | No |
> | assessmentmetadata | No | No | No |
> | assessments | No | No | No |
> | autodismissalertsrules | No | No | No |
> | automations | Sí | Sí | No |
> | autoprovisioningsettings | No | No | No |
> | complianceresults | No | No | No |
> | compliances | No | No | No |
> | datacollectionagents | No | No | No |
> | devicesecuritygroups | No | No | No |
> | discoveredsecuritysolutions | No | No | No |
> | externalsecuritysolutions | No | No | No |
> | informationprotectionpolicies | No | No | No |
> | iotsecuritysolutions | Sí | Sí | No |
> | iotsecuritysolutions/analyticsmodels | No | No | No |
> | iotsecuritysolutions/analyticsmodels/aggregatedalerts | No | No | No |
> | iotsecuritysolutions/analyticsmodels/aggregatedrecommendations | No | No | No |
> | jitnetworkaccesspolicies | No | No | No |
> | directivas | No | No | No |
> | pricings | No | No | No |
> | regulatorycompliancestandards | No | No | No |
> | regulatorycompliancestandards/regulatorycompliancecontrols | No | No | No |
> | regulatorycompliancestandards/regulatorycompliancecontrols/regulatorycomplianceassessments | No | No | No |
> | securitycontacts | No | No | No |
> | securitysolutions | No | No | No |
> | securitysolutionsreferencedata | No | No | No |
> | securitystatuses | No | No | No |
> | securitystatusessummaries | No | No | No |
> | servervulnerabilityassessments | No | No | No |
> | configuración | No | No | No |
> | subassessments | No | No | No |
> | tareas | No | No | No |
> | topologies | No | No | No |
> | workspacesettings | No | No | No |

## <a name="microsoftsecurityinsights"></a>Microsoft.SecurityInsights

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | aggregations | No | No | No |
> | alertrules | No | No | No |
> | alertruletemplates | No | No | No |
> | automationrules | No | No | No |
> | bookmarks | No | No | No |
> | cases | No | No | No |
> | dataconnectors | No | No | No |
> | entities | No | No | No |
> | entityqueries | No | No | No |
> | incidents | No | No | No |
> | officeconsents | No | No | No |
> | configuración | No | No | No |
> | threatintelligence | No | No | No |

## <a name="microsoftserialconsole"></a>Microsoft.SerialConsole

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | consoleservices | No | No | No |

## <a name="microsoftservermanagement"></a>Microsoft.ServerManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | gateways | No | No | No |
> | nodes | No | No | No |

## <a name="microsoftservicebus"></a>Microsoft.ServiceBus

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | espacios de nombres | Sí | Sí | No |
> | premiummessagingregions | No | No | No |
> | sku | No | No | No |

## <a name="microsoftservicefabric"></a>Microsoft.ServiceFabric

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | applications | No | No | No |
> | clusters | Sí | Sí | No |
> | containergroups | No | No | No |
> | containergroupsets | No | No | No |
> | edgeclusters | No | No | No |
> | managedclusters | No | No | No |
> | networks | No | No | No |
> | secretstores | No | No | No |
> | volumes | No | No | No |

## <a name="microsoftservicefabricmesh"></a>Microsoft.ServiceFabricMesh

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | applications | Sí | Sí | No |
> | containergroups | No | No | No |
> | gateways | Sí | Sí | No |
> | networks | Sí | Sí | No |
> | secrets | Sí | Sí | No |
> | volumes | Sí | Sí | No |

## <a name="microsoftservices"></a>Microsoft.Services

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | rollouts | No | No | No |

## <a name="microsoftsignalrservice"></a>Microsoft.SignalRService

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | signalr | Sí | Sí | No |

## <a name="microsoftsoftwareplan"></a>Microsoft.SoftwarePlan

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | hybridusebenefits | No | No | No |

## <a name="microsoftsolutions"></a>Microsoft.Solutions

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | applicationdefinitions | No | No | No |
> | applications | No | No | No |
> | jitrequests | No | No | No |

## <a name="microsoftsql"></a>Microsoft.Sql

> [!IMPORTANT]
> La base de datos y el servidor deben residir en el mismo grupo de recursos. Cuando se mueve un servidor SQL Server, se mueven también todas sus bases de datos. Este comportamiento se aplica a las bases de datos de Azure SQL Database y Azure Synapse Analytics.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | instancepools | No | No | No |
> | locations | Sí | Sí | No |
> | managedinstances | No | No | Sí <br/><br/> [Más información](../../azure-sql/database/move-resources-across-regions.md) sobre cómo trasladar instancias administradas entre regiones. |
> | managedinstances/databases | No | No | Sí |
> | servers | Sí | Sí |Sí |
> | servers/databases | Sí | Sí | Sí <br/><br/> [Más información](../../azure-sql/database/move-resources-across-regions.md) sobre cómo trasladar bases de datos entre regiones.<br/><br/> [Más información](../../resource-mover/tutorial-move-region-sql.md) sobre el uso de Azure Resource Mover para trasladar las bases de datos de Azure SQL.  |
> | servers/databases/backuplongtermretentionpolicies | Sí | Sí | No |
> | servers/elasticpools | Sí | Sí | Sí <br/><br/> [Más información](../../azure-sql/database/move-resources-across-regions.md) sobre cómo trasladar grupos elásticos entre regiones.<br/><br/> [Más información](../../resource-mover/tutorial-move-region-sql.md) sobre el uso de Azure Resource Mover para trasladar los grupos elásticos de Azure SQL.  |
> | servers / jobaccounts | Sí | Sí | No |
> | servers / jobagents | Sí | Sí | No |
> | virtualclusters | Sí | Sí | Sí |

## <a name="microsoftsqlvirtualmachine"></a>Microsoft.SqlVirtualMachine

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | sqlvirtualmachinegroups | Sí | Sí | No |
> | sqlvirtualmachines | Sí | Sí | No |

## <a name="microsoftstorage"></a>Microsoft.Storage

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | storageaccounts | Sí | Sí | Sí<br/><br/> [Traslado de una cuenta de Azure Storage a otra región](../../storage/common/storage-account-move.md) |

## <a name="microsoftstoragecache"></a>Microsoft.StorageCache

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | caches | No | No | No |

## <a name="microsoftstoragesync"></a>Microsoft.StorageSync

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | storagesyncservices | Sí | Sí | No |

## <a name="microsoftstoragesyncdev"></a>Microsoft.StorageSyncDev

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | storagesyncservices | No | No | No |

## <a name="microsoftstoragesyncint"></a>Microsoft.StorageSyncInt

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | storagesyncservices | No | No | No |

## <a name="microsoftstorsimple"></a>Microsoft.StorSimple

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | managers | No | No | No |

## <a name="microsoftstreamanalytics"></a>Microsoft.StreamAnalytics

> [!IMPORTANT]
> Los trabajos de Stream Analytics no se pueden mover si se encuentran en estado de ejecución.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | clusters | No | No | No |
> | streamingjobs | Sí | Sí | No |

## <a name="microsoftstreamanalyticsexplorer"></a>Microsoft.StreamAnalyticsExplorer

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | environments | No | No | No |
> | instances | No | No | No |

## <a name="microsoftsubscription"></a>Microsoft.Subscription

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | subscriptions | No | No | No |

## <a name="microsoftsupport"></a>Microsoft.Support

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | services | No | No | No |
> | supporttickets | No | No | No |

## <a name="microsoftsynapse"></a>Microsoft.Synapse

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | workspaces | No | No | No |
> | workspaces / bigdatapools | No | No | No |
> | workspaces / sqlpools | No | No | No |

## <a name="microsofttimeseriesinsights"></a>Microsoft.TimeSeriesInsights

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | environments | Sí | Sí | No |
> | environments/eventsources | Sí | Sí | No |
> | environments/referencedatasets | Sí | Sí | No |

## <a name="microsofttoken"></a>Microsoft.Token

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | almacenes | Sí | Sí | No |

## <a name="microsoftvirtualmachineimages"></a>Microsoft.VirtualMachineImages

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | imagetemplates | No | No | No |

## <a name="microsoftvisualstudio"></a>Microsoft.VisualStudio

> [!IMPORTANT]
> Para cambiar la suscripción de Azure DevOps, vea [Cambiar la suscripción de Azure usada para la facturación](/azure/devops/organizations/billing/change-azure-subscription?toc=/azure/azure-resource-manager/toc.json).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | account | No | No | No |
> | account/extension | No | No | No |
> | account/project | No | No | No |

## <a name="microsoftvmware"></a>Microsoft.VMware

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | arczones | No | No | No |
> | resourcepools | No | No | No |
> | vcenters | No | No | No |
> | virtualmachines | No | No | No |
> | virtualmachinetemplates | No | No | No |
> | virtualnetworks | No | No | No |

## <a name="microsoftvmwarecloudsimple"></a>Microsoft.VMwareCloudSimple

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | dedicatedcloudnodes | No | No | No |
> | dedicatedcloudservices | No | No | No |
> | virtualmachines | No | No | No |

## <a name="microsoftvnfmanager"></a>Microsoft.VnfManager

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | devices | No | No | No |
> | vnfs | No | No | No |

## <a name="microsoftvsonline"></a>Microsoft.VSOnline

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | accounts | No | No | No |
> | plans | No | No | No |
> | registeredsubscriptions | No | No | No |

## <a name="microsoftweb"></a>Microsoft.Web

> [!IMPORTANT]
> Vea [Guía de movimiento de App Service](./move-limitations/app-service-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | availablestacks | No | No | No |
> | billingmeters | No | No | No |
> | certificates | No | Sí | No |
> | connectiongateways | Sí | Sí | No |
> | connections | Sí | Sí | No |
> | customapis | Sí | Sí | No |
> | deletedsites | No | No | No |
> | deploymentlocations | No | No | No |
> | georegions | No | No | No |
> | hostingenvironments | No | No | No |
> | kubeenvironments | Sí | Sí | No |
> | publishingusers | No | No | No |
> | de películas | No | No | No |
> | resourcehealthmetadata | No | No | No |
> | runtimes | No | No | No |
> | serverfarms | Sí | Sí | No |
> | serverfarms/eventgridfilters | No | No | No |
> | sites | Sí | Sí | No |
> | sites/premieraddons | Sí | Sí | No |
> | sites/slots | Sí | Sí | No |
> | sourcecontrols | No | No | No |
> | staticsites | No | No | No |

## <a name="microsoftwindowsesu"></a>Microsoft.WindowsESU

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | multipleactivationkeys | No | No | No |

## <a name="microsoftwindowsiot"></a>Microsoft.WindowsIoT

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | deviceservices | No | No | No |

## <a name="microsoftworkloadbuilder"></a>Microsoft.WorkloadBuilder

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | workloads | No | No | No |

## <a name="microsoftworkloadmonitor"></a>Microsoft.WorkloadMonitor

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción | Movimiento de región |
> | ------------- | ----------- | ---------- | ----------- |
> | components | No | No | No |
> | componentssummary | No | No | No |
> | monitorinstances | No | No | No |
> | monitorinstancessummary | No | No | No |
> | monitors | No | No | No |

## <a name="third-party-services"></a>Servicios de terceros

Actualmente los servicios de terceros no son compatibles con la operación de traslado.

## <a name="next-steps"></a>Pasos siguientes

- Para ver los comandos para trasladar recursos, vea [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](move-resource-group-and-subscription.md).
- [Más información](../../resource-mover/overview.md) sobre el servicio Resource Mover.
- Para obtener los mismos datos como un archivo de valores separados por comas, descargue [move-support-resources.csv](https://github.com/tfitzmac/resource-capabilities/blob/master/move-support-resources.csv).
