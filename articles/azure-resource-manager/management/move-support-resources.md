---
title: Compatibilidad con la operación de traslado por tipo de recurso
description: Enumera los tipos de recursos de Azure que se pueden trasladar a un nuevo grupo de recursos o suscripción.
ms.topic: conceptual
ms.date: 04/08/2021
ms.openlocfilehash: f3832a32f15debd8737aa16b0a9347b435a238c0
ms.sourcegitcommit: b28e9f4d34abcb6f5ccbf112206926d5434bd0da
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107227456"
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
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | domainservices | No | No |

## <a name="microsoftaadiam"></a>microsoft.aadiam

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | diagnosticsettings | No | No |
> | diagnosticsettingscategories | No | No |
> | privatelinkforazuread | Sí | Sí |
> | tenants | Sí | Sí |

## <a name="microsoftaddons"></a>Microsoft.Addons

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | supportproviders | No | No |

## <a name="microsoftadhybridhealthservice"></a>Microsoft.ADHybridHealthService

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | aadsupportcases | No | No |
> | addsservices | No | No |
> | agents | No | No |
> | anonymousapiusers | No | No |
> | configuración | No | No |
> | logs | No | No |
> | reports | No | No |
> | servicehealthmetrics | No | No |
> | services | No | No |

## <a name="microsoftadvisor"></a>Microsoft.Advisor

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | configuraciones | No | No |
> | generaterecommendations | No | No |
> | metadata | No | No |
> | de películas | No | No |
> | suppressions | No | No |

## <a name="microsoftalertsmanagement"></a>Microsoft.AlertsManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | actionRules | Sí | Sí |
> | alerts | No | No |
> | alertslist | No | No |
> | alertsmetadata | No | No |
> | alertssummary | No | No |
> | alertssummarylist | No | No |
> | smartdetectoralertrules | Sí | Sí |
> | smartgroups | No | No |

## <a name="microsoftanalysisservices"></a>Microsoft.AnalysisServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | servers | Sí | Sí |

## <a name="microsoftapimanagement"></a>Microsoft.ApiManagement

> [!IMPORTANT]
> Un servicio API Management establecido en la SKU de consumo no se puede mover.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | reportfeedback | No | No |
> | service | Sí | Sí |

## <a name="microsoftappconfiguration"></a>Microsoft.AppConfiguration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | configurationstores | Sí | Sí |
> | configurationstores/eventgridfilters | No | No |

## <a name="microsoftappplatform"></a>Microsoft.AppPlatform

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | spring | Sí | Sí |

## <a name="microsoftappservice"></a>Microsoft.AppService

> [!IMPORTANT]
> Vea [Guía de movimiento de App Service](./move-limitations/app-service-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | apiapps | No | No |
> | appidentities | No | No |
> | gateways | No | No |

## <a name="microsoftattestation"></a>Microsoft.Attestation

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | attestationproviders | Sí | Sí |

## <a name="microsoftauthorization"></a>Microsoft.Authorization

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | classicadministrators | No | No |
> | dataaliases | No | No |
> | denyassignments | No | No |
> | elevateaccess | No | No |
> | findorphanroleassignments | No | No |
> | locks | No | No |
> | permisos | No | No |
> | policyassignments | No | No |
> | policydefinitions | No | No |
> | policysetdefinitions | No | No |
> | privatelinkassociations | No | No |
> | resourcemanagementprivatelinks | No | No |
> | roleassignments | No | No |
> | roleassignmentsusagemetrics | No | No |
> | roledefinitions | No | No |

## <a name="microsoftautomation"></a>Microsoft.Automation

> [!IMPORTANT]
> Los runbooks deben encontrarse en el mismo grupo de recursos que la cuenta de Automation.
>
> Para obtener más información, consulte [Traslado de la cuenta de Azure Automation a otra suscripción](../../automation/how-to/move-account.md?toc=/azure/azure-resource-manager/toc.json).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | automationaccounts | Sí | Sí |
> | automationaccounts/configurations | Sí | Sí |
> | automationaccounts/runbooks | Sí | Sí |

## <a name="microsoftavs"></a>Microsoft.AVS

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | privateclouds | Sí | Sí |

## <a name="microsoftazureactivedirectory"></a>Microsoft.AzureActiveDirectory

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | b2cdirectories | Sí | Sí |
> | b2ctenants | No | No |

## <a name="microsoftazuredata"></a>Microsoft.AzureData

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | datacontrollers | No | No |
> | hybriddatamanagers | No | No |
> | postgresinstances | No | No |
> | sqlinstances | No | No |
> | sqlmanagedinstances | No | No |
> | sqlserverinstances | No | No |
> | sqlserverregistrations | Sí | Sí |

## <a name="microsoftazurestack"></a>Microsoft.AzureStack

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | cloudmanifestfiles | No | No |
> | registrations | Sí | Sí |

## <a name="microsoftazurestackhci"></a>Microsoft.AzureStackHCI

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | clusters | No | No |

## <a name="microsoftbatch"></a>Microsoft.Batch

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | batchaccounts | Sí | Sí |

## <a name="microsoftbilling"></a>Microsoft.Billing

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | billingaccounts | No | No |
> | billingperiods | No | No |
> | billingpermissions | No | No |
> | billingproperty | No | No |
> | billingroleassignments | No | No |
> | billingroledefinitions | No | No |
> | departments | No | No |
> | enrollmentaccounts | No | No |
> | invoices | No | No |
> | transfers | No | No |

## <a name="microsoftbingmaps"></a>Microsoft.BingMaps

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | mapapis | No | No |

## <a name="microsoftbiztalkservices"></a>Microsoft.BizTalkServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | biztalk | No | No |

## <a name="microsoftblockchain"></a>Microsoft.Blockchain

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | blockchainmembers | No | No |
> | cordamembers | No | No |
> | watchers | No | No |

## <a name="microsoftblockchaintokens"></a>Microsoft.BlockchainTokens

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | tokenservices | No | No |

## <a name="microsoftblueprint"></a>Microsoft.Blueprint

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | blueprintassignments | No | No |
> | blueprints | No | No |

## <a name="microsoftbotservice"></a>Microsoft.BotService

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | botservices | Sí | Sí |

## <a name="microsoftcache"></a>Microsoft.Cache

> [!IMPORTANT]
> Si la instancia de Azure Redis Cache está configurada con una red virtual, la instancia no se puede mover a otra suscripción. Consulte las [Limitaciones de movimiento de redes](./move-limitations/networking-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | redis | Sí | Sí |
> | redisenterprise | No | No |

## <a name="microsoftcapacity"></a>Microsoft.Capacity

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | appliedreservations | No | No |
> | calculateexchange | No | No |
> | calculateprice | No | No |
> | calculatepurchaseprice | No | No |
> | catalogs | No | No |
> | commercialreservationorders | No | No |
> | cambio | No | No |
> | reservationorders | No | No |
> | reservations | No | No |
> | resources | No | No |
> | validatereservationorder | No | No |

## <a name="microsoftcdn"></a>Microsoft.Cdn

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | cdnwebapplicationfirewallmanagedrulesets | No | No |
> | cdnwebapplicationfirewallpolicies | Sí | Sí |
> | edgenodes | No | No |
> | profiles | Sí | Sí |
> | profiles/endpoints | Sí | Sí |

## <a name="microsoftcertificateregistration"></a>Microsoft.CertificateRegistration

> [!IMPORTANT]
> Vea [Guía de movimiento de App Service](./move-limitations/app-service-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | certificateorders | Sí | Sí |

## <a name="microsoftclassiccompute"></a>Microsoft.ClassicCompute

> [!IMPORTANT]
> Vea [Guía de movimiento de implementación clásica](./move-limitations/classic-model-move-limitations.md). Los recursos de implementación clásica se pueden mover entre suscripciones con una operación específica de ese escenario.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | capabilities | No | No |
> | domainnames | Sí | No |
> | quotas | No | No |
> | resourcetypes | No | No |
> | validatesubscriptionmoveavailability | No | No |
> | virtualmachines | Sí | Sí |

## <a name="microsoftclassicinfrastructuremigrate"></a>Microsoft.ClassicInfrastructureMigrate

> [!IMPORTANT]
> Vea [Guía de movimiento de implementación clásica](./move-limitations/classic-model-move-limitations.md). Los recursos de implementación clásica se pueden mover entre suscripciones con una operación específica de ese escenario.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | classicinfrastructureresources | No | No |

## <a name="microsoftclassicnetwork"></a>Microsoft.ClassicNetwork

> [!IMPORTANT]
> Vea [Guía de movimiento de implementación clásica](./move-limitations/classic-model-move-limitations.md). Los recursos de implementación clásica se pueden mover entre suscripciones con una operación específica de ese escenario.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | capabilities | No | No |
> | expressroutecrossconnections | No | No |
> | expressroutecrossconnections/peerings | No | No |
> | gatewaysupporteddevices | No | No |
> | networksecuritygroups | No | No |
> | quotas | No | No |
> | reservedips | No | No |
> | virtualnetworks | No | No |

## <a name="microsoftclassicstorage"></a>Microsoft.ClassicStorage

> [!IMPORTANT]
> Vea [Guía de movimiento de implementación clásica](./move-limitations/classic-model-move-limitations.md). Los recursos de implementación clásica se pueden mover entre suscripciones con una operación específica de ese escenario.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | disks | No | No |
> | images | No | No |
> | osimages | No | No |
> | osplatformimages | No | No |
> | publicimages | No | No |
> | quotas | No | No |
> | storageaccounts | Sí | No |
> | vmimages | No | No |

## <a name="microsoftclassicsubscription"></a>Microsoft.ClassicSubscription

> [!IMPORTANT]
> Vea [Guía de movimiento de implementación clásica](./move-limitations/classic-model-move-limitations.md). Los recursos de implementación clásica se pueden mover entre suscripciones con una operación específica de ese escenario.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | operaciones | No | No |

## <a name="microsoftcognitiveservices"></a>Microsoft.CognitiveServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | Sí | Sí |

## <a name="microsoftcommerce"></a>Microsoft.Commerce

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | ratecard | No | No |
> | usageaggregates | No | No |

## <a name="microsoftcompute"></a>Microsoft.Compute

> [!IMPORTANT]
> Vea [Guía de movimiento de Virtual Machines](./move-limitations/virtual-machines-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | availabilitysets | Sí | Sí |
> | diskaccesses | No | No |
> | diskencryptionsets | No | No |
> | disks | Sí | Sí |
> | galleries | No | No |
> | galleries/images | No | No |
> | galleries/images/versions | No | No |
> | hostgroups | No | No |
> | hostgroups/hosts | No | No |
> | images | Sí | Sí |
> | proximityplacementgroups | Sí | Sí |
> | restorepointcollections | No | No |
> | restorepointcollections/restorepoints | No | No |
> | sharedvmextensions | No | No |
> | sharedvmimages | No | No |
> | sharedvmimages/versions | No | No |
> | snapshots | Sí | Sí |
> | sshpublickeys | No | No |
> | virtualmachines | Sí | Sí |
> | virtualmachines/extensions | Sí | Sí |
> | virtualmachinescalesets | Sí | Sí |

## <a name="microsoftconsumption"></a>Microsoft.Consumption

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | aggregatedcost | No | No |
> | balances | No | No |
> | budgets | No | No |
> | charges | No | No |
> | costtags | No | No |
> | credits | No | No |
> | events | No | No |
> | forecasts | No | No |
> | lots | No | No |
> | marketplaces | No | No |
> | pricesheets | No | No |
> | products | No | No |
> | reservationdetails | No | No |
> | reservationrecommendationdetails | No | No |
> | reservationrecommendations | No | No |
> | reservationsummaries | No | No |
> | reservationtransactions | No | No |
> | etiquetas | No | No |
> | tenants | No | No |
> | terms | No | No |
> | usagedetails | No | No |

## <a name="microsoftcontainerinstance"></a>Microsoft.ContainerInstance

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | containergroups | No | No |
> | serviceassociationlinks | No | No |

## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | registries | Sí | Sí |
> | registries / agentpools | Sí | Sí |
> | registries/buildtasks | Sí | Sí |
> | registries/replications | Sí | Sí |
> | registries/tasks | Sí | Sí |
> | registries/webhooks | Sí | Sí |

## <a name="microsoftcontainerservice"></a>Microsoft.ContainerService

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | containerservices | No | No |
> | managedclusters | No | No |
> | openshiftmanagedclusters | No | No |

## <a name="microsoftcontentmoderator"></a>Microsoft.ContentModerator

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | applications | No | No |

## <a name="microsoftcortanaanalytics"></a>Microsoft.CortanaAnalytics

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | No | No |

## <a name="microsoftcostmanagement"></a>Microsoft.CostManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | alerts | No | No |
> | billingaccounts | No | No |
> | budgets | No | No |
> | cloudconnectors | No | No |
> | conectores | Sí | Sí |
> | departments | No | No |
> | dimensions | No | No |
> | enrollmentaccounts | No | No |
> | exports | No | No |
> | externalbillingaccounts | No | No |
> | forecast | No | No |
> | Query | No | No |
> | registro | No | No |
> | reportconfigs | No | No |
> | reports | No | No |
> | configuración | No | No |
> | showbackrules | No | No |
> | views | No | No |

## <a name="microsoftcustomerinsights"></a>Microsoft.CustomerInsights

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | hubs | No | No |

## <a name="microsoftcustomerlockbox"></a>Microsoft.CustomerLockbox

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | Solicitudes | No | No |

## <a name="microsoftcustomproviders"></a>Microsoft.CustomProviders

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | associations | No | No |
> | resourceproviders | Sí | Sí |

## <a name="microsoftdatabox"></a>Microsoft.DataBox

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | jobs | No | No |

## <a name="microsoftdataboxedge"></a>Microsoft.DataBoxEdge

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | availableskus | No | No |
> | databoxedgedevices | No | No |

## <a name="microsoftdatabricks"></a>Microsoft.Databricks

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | workspaces | No | No |

## <a name="microsoftdatacatalog"></a>Microsoft.DataCatalog

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | catalogs | Sí | Sí |
> | datacatalogs | No | No |

## <a name="microsoftdataconnect"></a>Microsoft.DataConnect

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | connectionmanagers | No | No |

## <a name="microsoftdataexchange"></a>Microsoft.DataExchange

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | packages | No | No |
> | plans | No | No |

## <a name="microsoftdatafactory"></a>Microsoft.DataFactory

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | datafactories | Sí | Sí |
> | factories | Sí | Sí |

## <a name="microsoftdatalake"></a>Microsoft.DataLake

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | datalakeaccounts | No | No |

## <a name="microsoftdatalakeanalytics"></a>Microsoft.DataLakeAnalytics

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | Sí | Sí |

## <a name="microsoftdatalakestore"></a>Microsoft.DataLakeStore

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | Sí | Sí |

## <a name="microsoftdatamigration"></a>Microsoft.DataMigration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | services | No | No |
> | services/projects | No | No |
> | slots | No | No |

## <a name="microsoftdataprotection"></a>Microsoft.DataProtection

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | backupvaults | No | No |

## <a name="microsoftdatashare"></a>Microsoft.DataShare

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | Sí | Sí |

## <a name="microsoftdbformariadb"></a>Microsoft.DBforMariaDB

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | servers | Sí | Sí |

## <a name="microsoftdbformysql"></a>Microsoft.DBforMySQL

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | flexibleServers | No | No |
> | servers | Sí | Sí |

## <a name="microsoftdbforpostgresql"></a>Microsoft.DBforPostgreSQL

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | flexibleServers | No | No |
> | servergroups | No | No |
> | servers | Sí | Sí |
> | serversv2 | Sí | Sí |
> | singleservers | Sí | Sí |

## <a name="microsoftdeploymentmanager"></a>Microsoft.DeploymentManager

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | artifactsources | Sí | Sí |
> | rollouts | Sí | Sí |
> | servicetopologies | Sí | Sí |
> | servicetopologies/services | Sí | Sí |
> | servicetopologies/services/serviceunits | Sí | Sí |
> | steps | Sí | Sí |

## <a name="microsoftdesktopvirtualization"></a>Microsoft.DesktopVirtualization

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | applicationgroups | Sí | Sí |
> | hostpools | Sí | Sí |
> | workspaces | Sí | Sí |

## <a name="microsoftdevices"></a>Microsoft.Devices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | elasticpools | No | No |
> | elasticpools/iothubtenants | No | No |
> | iothubs | Sí | Sí |
> | provisioningservices | Sí | Sí |

## <a name="microsoftdevops"></a>Microsoft.DevOps

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | pipelines | Sí | Sí |

## <a name="microsoftdevspaces"></a>Microsoft.DevSpaces

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | controllers | Sí | Sí |

## <a name="microsoftdevtestlab"></a>Microsoft.DevTestLab

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | labcenters | No | No |
> | labs | Sí | No |
> | labs/environments | Sí | Sí |
> | labs/servicerunners | Sí | Sí |
> | labs/virtualmachines | Sí | No |
> | schedules | Sí | Sí |

## <a name="microsoftdigitaltwins"></a>Microsoft.DigitalTwins

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | digitaltwinsinstances | No | No |

## <a name="microsoftdocumentdb"></a>Microsoft.DocumentDB

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | databaseaccountnames | No | No |
> | databaseaccounts | Sí | Sí |

## <a name="microsoftdomainregistration"></a>Microsoft.DomainRegistration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | domains | Sí | Sí |
> | generatessorequest | No | No |
> | topleveldomains | No | No |
> | validatedomainregistrationinformation | No | No |

## <a name="microsoftenterpriseknowledgegraph"></a>Microsoft.EnterpriseKnowledgeGraph

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | services | Sí | Sí |

## <a name="microsofteventgrid"></a>Microsoft.EventGrid

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | domains | Sí | Sí |
> | eventsubscriptions | No: no se puede mover de forma independiente, sino que se hace automáticamente con el recurso suscrito. | No: no se puede mover de forma independiente, sino que se hace automáticamente con el recurso suscrito. |
> | extensiontopics | No | No |
> | partnernamespaces | Sí | Sí |
> | partnerregistrations | No | No |
> | partnertopics | Sí | Sí |
> | systemtopics | Sí | Sí |
> | topics | Sí | Sí |
> | topictypes | No | No |

## <a name="microsofteventhub"></a>Microsoft.EventHub

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | clusters | Sí | Sí |
> | espacios de nombres | Sí | Sí |
> | sku | No | No |

## <a name="microsoftexperimentation"></a>Microsoft.Experimentation

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | experimentworkspaces | No | No |

## <a name="microsoftfalcon"></a>Microsoft.Falcon

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | espacios de nombres | Sí | Sí |

## <a name="microsoftfeatures"></a>Microsoft.Features

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | featureproviders | No | No |
> | features | No | No |
> | providers | No | No |
> | subscriptionfeatureregistrations | No | No |

## <a name="microsoftgenomics"></a>Microsoft.Genomics

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | No | No |

## <a name="microsoftguestconfiguration"></a>Microsoft.GuestConfiguration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | automanagedaccounts | No | No |
> | automanagedvmconfigurationprofiles | No | No |
> | guestconfigurationassignments | No | No |
> | software | No | No |
> | softwareupdateprofile | No | No |
> | softwareupdates | No | No |

## <a name="microsofthanaonazure"></a>Microsoft.HanaOnAzure

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | hanainstances | No | No |
> | sapmonitors | No | No |

## <a name="microsofthardwaresecuritymodules"></a>Microsoft.HardwareSecurityModules

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | dedicatedhsms | No | No |

## <a name="microsofthdinsight"></a>Microsoft.HDInsight

> [!IMPORTANT]
> Puede mover clústeres de HDInsight a una nueva suscripción o un nuevo grupo de recursos. Sin embargo, no puede mover entre suscripciones los recursos de red vinculados al clúster de HDInsight (por ejemplo, la red virtual, una NIC o un equilibrador de carga). Además, tampoco se puede mover a un nuevo grupo de recursos una NIC que está conectada a una máquina virtual del clúster.
>
> Al mover un clúster de HDInsight a una nueva suscripción, mueva primero otros recursos (por ejemplo, la cuenta de almacenamiento). Después, mover el clúster de HDInsight.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | clusters | Sí | Sí |

## <a name="microsofthealthcareapis"></a>Microsoft.HealthcareApis

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | services | Sí | Sí |

## <a name="microsofthybridcompute"></a>Microsoft.HybridCompute

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | machines | Sí | Sí |
> | machines/extensions | Sí | Sí |

## <a name="microsofthybriddata"></a>Microsoft.HybridData

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | datamanagers | Sí | Sí |

## <a name="microsofthybridnetwork"></a>Microsoft.HybridNetwork

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | devices | No | No |
> | vnfs | No | No |

## <a name="microsofthydra"></a>Microsoft.Hydra

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | components | No | No |
> | networkscopes | No | No |

## <a name="microsoftimportexport"></a>Microsoft.ImportExport

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | jobs | Sí | Sí |

## <a name="microsoftinsights"></a>microsoft.insights

> [!IMPORTANT]
> Asegúrese de que el movimiento a una nueva suscripción no exceda las [cuotas de suscripción](azure-subscription-service-limits.md#azure-monitor-limits).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | actiongroups | Sí | Sí |
> | activitylogalerts | No | No |
> | alertrules | Sí | Sí |
> | autoscalesettings | Sí | Sí |
> | baseline | No | No |
> | components | Sí | Sí |
> | datacollectionrules | No | No |
> | diagnosticsettings | No | No |
> | diagnosticsettingscategories | No | No |
> | eventcategories | No | No |
> | eventtypes | No | No |
> | extendeddiagnosticsettings | No | No |
> | guestdiagnosticsettings | No | No |
> | listmigrationdate | No | No |
> | logdefinitions | No | No |
> | logprofiles | No | No |
> | logs | No | No |
> | metricalerts | No | No |
> | metricbaselines | No | No |
> | metricbatch | No | No |
> | metricdefinitions | No | No |
> | metricnamespaces | No | No |
> | Métricas | No | No |
> | migratealertrules | No | No |
> | migratetonewpricingmodel | No | No |
> | myworkbooks | No | No |
> | notificationgroups | No | No |
> | privatelinkscopes | No | No |
> | rollbacktolegacypricingmodel | No | No |
> | scheduledqueryrules | Sí | Sí |
> | Topología | No | No |
> | transacciones | No | No |
> | vminsightsonboardingstatuses | No | No |
> | webtests | Sí | Sí |
> | webtests/gettestresultfile | No | No |
> | workbooks | Sí | Sí |
> | workbooktemplates | Sí | Sí |

## <a name="microsoftiotcentral"></a>Microsoft.IoTCentral

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | apptemplates | No | No |
> | iotapps | Sí | Sí |

## <a name="microsoftiotspaces"></a>Microsoft.IoTSpaces

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | graph | Sí | Sí |

## <a name="microsoftkeyvault"></a>Microsoft.KeyVault

> [!IMPORTANT]
> Los almacenes de claves usados para el cifrado de discos no se pueden mover a grupos de recursos de la misma suscripción ni entre suscripciones.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | deletedvaults | No | No |
> | hsmpools | No | No |
> | managedhsms | No | No |
> | vaults | Sí | Sí |

## <a name="microsoftkubernetes"></a>Microsoft.Kubernetes

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | connectedclusters | Sí | Sí |
> | registeredsubscriptions | No | No |

## <a name="microsoftkubernetesconfiguration"></a>Microsoft.KubernetesConfiguration

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | sourcecontrolconfigurations | No | No |

## <a name="microsoftkusto"></a>Microsoft.Kusto

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | clusters | Sí | Sí |

## <a name="microsoftlabservices"></a>Microsoft.LabServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | labaccounts | No | No |
> | users | No | No |

## <a name="microsoftlocationbasedservices"></a>Microsoft.LocationBasedServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | No | No |

## <a name="microsoftlocationservices"></a>Microsoft.LocationServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | No | No |

## <a name="microsoftlogic"></a>Microsoft.Logic

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | hostingenvironments | No | No |
> | integrationaccounts | Sí | Sí |
> | integrationserviceenvironments | Sí | No |
> | integrationserviceenvironments / managedapis | Sí | No |
> | isolatedenvironments | No | No |
> | workflows | Sí | Sí |

## <a name="microsoftmachinelearning"></a>Microsoft.MachineLearning

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | commitmentplans | No | No |
> | webservices | Sí | No |
> | workspaces | Sí | Sí |

## <a name="microsoftmachinelearningcompute"></a>Microsoft.MachineLearningCompute

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | operationalizationclusters | No | No |

## <a name="microsoftmachinelearningexperimentation"></a>Microsoft.MachineLearningExperimentation

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | No | No |
> | teamaccounts | No | No |

## <a name="microsoftmachinelearningmodelmanagement"></a>Microsoft.MachineLearningModelManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | No | No |

## <a name="microsoftmachinelearningservices"></a>Microsoft.MachineLearningServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | workspaces | No | No |

## <a name="microsoftmaintenance"></a>Microsoft.Maintenance

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | configurationassignments | No | No |
> | maintenanceconfigurations | Sí | Sí |
> | updates | No | No |

## <a name="microsoftmanagedidentity"></a>Microsoft.ManagedIdentity

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | Identidades | No | No |
> | userassignedidentities | No | No |

## <a name="microsoftmanagednetwork"></a>Microsoft.ManagedNetwork

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | managednetworks | No | No |
> | managednetworks / managednetworkgroups | No | No |
> | managednetworks / managednetworkpeeringpolicies | No | No |
> | notificación | No | No |

## <a name="microsoftmanagedservices"></a>Microsoft.ManagedServices

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | marketplaceregistrationdefinitions | No | No |
> | registrationassignments | No | No |
> | registrationdefinitions | No | No |

## <a name="microsoftmanagement"></a>Microsoft.Management

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | getentities | No | No |
> | managementgroups | No | No |
> | managementgroups/settings | No | No |
> | resources | No | No |
> | starttenantbackfill | No | No |
> | tenantbackfillstatus | No | No |

## <a name="microsoftmaps"></a>Microsoft.Maps

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | Sí | Sí |
> | accounts / privateatlases | Sí | Sí |

## <a name="microsoftmarketplace"></a>Microsoft.Marketplace

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | offers | No | No |
> | offertypes | No | No |
> | privategalleryitems | No | No |
> | privatestoreclient | No | No |
> | privatestores | No | No |
> | products | No | No |
> | publishers | No | No |
> | registro | No | No |

## <a name="microsoftmarketplaceapps"></a>Microsoft.MarketplaceApps

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | classicdevservices | No | No |

## <a name="microsoftmarketplaceordering"></a>Microsoft.MarketplaceOrdering

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | agreements | No | No |
> | offertypes | No | No |

## <a name="microsoftmedia"></a>Microsoft.Media

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | mediaservices | Sí | Sí |
> | mediaservices/liveevents | Sí | Sí |
> | mediaservices/streamingendpoints | Sí | Sí |

## <a name="microsoftmicroservices4spring"></a>Microsoft.Microservices4Spring

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | appclusters | No | No |

## <a name="microsoftmigrate"></a>Microsoft.Migrate

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | assessmentprojects | No | No |
> | migrateprojects | No | No |
> | movecollections | No | No |
> | projects | No | No |

## <a name="microsoftmixedreality"></a>Microsoft.MixedReality

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | holographicsbroadcastaccounts | No | No |
> | objectunderstandingaccounts | No | No |
> | remoterenderingaccounts | Sí | Sí |
> | spatialanchorsaccounts | Sí | Sí |

## <a name="microsoftnetapp"></a>Microsoft.NetApp

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | netappaccounts | No | No |
> | netappaccounts/capacitypools | No | No |
> | netappaccounts/capacitypools/volumes | No | No |

## <a name="microsoftnetwork"></a>Microsoft.Network

> [!IMPORTANT]
> Consulte [Guía de movimiento de redes](./move-limitations/networking-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | applicationgateways | No | No |
> | applicationgatewaywebapplicationfirewallpolicies | No | No |
> | applicationsecuritygroups | Sí | Sí |
> | azurefirewalls | No | No |
> | bastionhosts | No | No |
> | bgpservicecommunities | No | No |
> | connections | Sí | Sí |
> | ddoscustompolicies | Sí | Sí |
> | ddosprotectionplans | No | No |
> | dnszones | Sí | Sí |
> | expressroutecircuits | No | No |
> | expressroutegateways | No | No |
> | expressrouteserviceproviders | No | No |
> | firewallpolicies | Sí | Sí |
> | frontdoors | No | No |
> | ipallocations | Sí | Sí |
> | ipgroups | Sí | Sí |
> | loadbalancers | Sí: SKU básico<br> Sí: SKU estándar | Sí: SKU básico<br>No: SKU estándar |
> | localnetworkgateways | Sí | Sí |
> | natgateways | No | No |
> | networkexperimentprofiles | No | No |
> | networkintentpolicies | Sí | Sí |
> | networkinterfaces | Sí | Sí |
> | networkprofiles | No | No |
> | networksecuritygroups | Sí | Sí |
> | networkwatchers | No | No |
> | networkwatchers/connectionmonitors | Sí | No |
> | networkwatchers / flowlogs | Sí | No |
> | networkwatchers/pingmeshes | Sí | No |
> | p2svpngateways | No | No |
> | privatednszones | Sí | Sí |
> | privatednszones/virtualnetworklinks | Sí | Sí |
> | privatednszonesinternal | No | No |
> | privateendpointredirectmaps | No | No |
> | privateendpoints | No | No |
> | privatelinkservices | No | No |
> | publicipaddresses | Sí: SKU básico<br>Sí: SKU estándar | Sí: SKU básico<br>No: SKU estándar |
> | publicipprefixes | Sí | Sí |
> | routefilters | No | No |
> | routetables | Sí | Sí |
> | securitypartnerproviders | Sí | Sí |
> | serviceendpointpolicies | Sí | Sí |
> | trafficmanagergeographichierarchies | No | No |
> | trafficmanagerprofiles | Sí | Sí |
> | trafficmanagerprofiles/heatmaps | No | No |
> | trafficmanagerusermetricskeys | No | No |
> | virtualhubs | No | No |
> | virtualnetworkgateways | Sí | Sí |
> | virtualnetworks | Sí | Sí |
> | virtualnetworktaps | No | No |
> | virtualrouters | Sí | Sí |
> | virtualwans | No | No |
> | vpngateways (Virtual WAN) | No | No |
> | vpnserverconfigurations | No | No |
> | vpnsites (Virtual WAN) | No | No |

## <a name="microsoftnotificationhubs"></a>Microsoft.NotificationHubs

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | espacios de nombres | Sí | Sí |
> | namespaces/notificationhubs | Sí | Sí |

## <a name="microsoftobjectstore"></a>Microsoft.ObjectStore

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | osnamespaces | Sí | Sí |

## <a name="microsoftoffazure"></a>Microsoft.OffAzure

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | hypervsites | No | No |
> | importsites | No | No |
> | serversites | No | No |
> | vmwaresites | No | No |

## <a name="microsoftoperationalinsights"></a>Microsoft.OperationalInsights

> [!IMPORTANT]
> Asegúrese de que el movimiento a una nueva suscripción no exceda las [cuotas de suscripción](azure-subscription-service-limits.md#azure-monitor-limits).
>
> Las áreas de trabajo que tienen una cuenta de automatización vinculada no se pueden mover. Antes de comenzar una operación de movimiento, asegúrese de desvincular todas las cuentas de automatización.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | clusters | No | No |
> | deletedworkspaces | No | No |
> | linktargets | No | No |
> | storageinsightconfigs | No | No |
> | workspaces | Sí | Sí |

## <a name="microsoftoperationsmanagement"></a>Microsoft.OperationsManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | managementassociations | No | No |
> | managementconfigurations | Sí | Sí |
> | solutions | Sí | Sí |
> | views | Sí | Sí |

## <a name="microsoftpeering"></a>Microsoft.Peering

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | legacypeerings | No | No |
> | peerasns | No | No |
> | peeringlocations | No | No |
> | peerings | No | No |
> | peeringservicecountries | No | No |
> | peeringservicelocations | No | No |
> | peeringserviceproviders | No | No |
> | peeringservices | No | No |

## <a name="microsoftpolicyinsights"></a>Microsoft.PolicyInsights

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | policyevents | No | No |
> | policystates | No | No |
> | policytrackedresources | No | No |
> | remediations | No | No |

## <a name="microsoftportal"></a>Microsoft.Portal

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | consoles | No | No |
> | dashboards | Sí | Sí |
> | usersettings | No | No |

## <a name="microsoftpowerbi"></a>Microsoft.PowerBI

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | workspacecollections | Sí | Sí |

## <a name="microsoftpowerbidedicated"></a>Microsoft.PowerBIDedicated

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | capacities | Sí | Sí |

## <a name="microsoftprojectbabylon"></a>Microsoft.ProjectBabylon

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | No | No |

## <a name="microsoftproviderhub"></a>Microsoft.ProviderHub

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | availableaccounts | No | No |
> | providerregistrations | No | No |
> | rollouts | No | No |

## <a name="microsoftquantum"></a>Microsoft.Quantum

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | workspaces | No | No |

## <a name="microsoftrecoveryservices"></a>Microsoft.RecoveryServices

> [!IMPORTANT]
> Vea [Guía de movimiento de Recovery Services](../../backup/backup-azure-move-recovery-services-vault.md?toc=/azure/azure-resource-manager/toc.json).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | replicationeligibilityresults | No | No |
> | vaults | Sí | Sí |

## <a name="microsoftredhatopenshift"></a>Microsoft.RedHatOpenShift

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | openshiftclusters | No | No |

## <a name="microsoftrelay"></a>Microsoft.Relay

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | espacios de nombres | Sí | Sí |

## <a name="microsoftresourcegraph"></a>Microsoft.ResourceGraph

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | Consultas | Sí | Sí |
> | resourcechangedetails | No | No |
> | resourcechanges | No | No |
> | resources | No | No |
> | resourceshistory | No | No |
> | subscriptionsstatus | No | No |

## <a name="microsoftresourcehealth"></a>Microsoft.ResourceHealth

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | childresources | No | No |
> | emergingissues | No | No |
> | events | No | No |
> | metadata | No | No |
> | Notificaciones | No | No |

## <a name="microsoftresources"></a>Microsoft.Resources

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | deployments | No | No |
> | deploymentscripts | No | No |
> | deploymentscripts/logs | No | No |
> | vínculos | No | No |
> | providers | No | No |
> | resourcegroups | No | No |
> | resources | No | No |
> | subscriptions | No | No |
> | etiquetas | No | No |
> | templatespecs | No | No |
> | templatespecs/versions | No | No |
> | tenants | No | No |

## <a name="microsoftsaas"></a>Microsoft.SaaS

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | applications | Sí | No |
> | saasresources | No | No |

## <a name="microsoftsearch"></a>Microsoft.Search

> [!IMPORTANT]
> No puede mover varios recursos de Search de regiones diferentes en una operación. En su lugar, muévalos en operaciones independientes.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | resourcehealthmetadata | No | No |
> | searchservices | Sí | Sí |

## <a name="microsoftsecurity"></a>Microsoft.Security

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | adaptivenetworkhardenings | No | No |
> | advancedthreatprotectionsettings | No | No |
> | alerts | No | No |
> | allowedconnections | No | No |
> | applicationwhitelistings | No | No |
> | assessmentmetadata | No | No |
> | assessments | No | No |
> | autodismissalertsrules | No | No |
> | automations | Sí | Sí |
> | autoprovisioningsettings | No | No |
> | complianceresults | No | No |
> | compliances | No | No |
> | datacollectionagents | No | No |
> | devicesecuritygroups | No | No |
> | discoveredsecuritysolutions | No | No |
> | externalsecuritysolutions | No | No |
> | informationprotectionpolicies | No | No |
> | iotsecuritysolutions | Sí | Sí |
> | iotsecuritysolutions/analyticsmodels | No | No |
> | iotsecuritysolutions/analyticsmodels/aggregatedalerts | No | No |
> | iotsecuritysolutions/analyticsmodels/aggregatedrecommendations | No | No |
> | jitnetworkaccesspolicies | No | No |
> | directivas | No | No |
> | pricings | No | No |
> | regulatorycompliancestandards | No | No |
> | regulatorycompliancestandards/regulatorycompliancecontrols | No | No |
> | regulatorycompliancestandards/regulatorycompliancecontrols/regulatorycomplianceassessments | No | No |
> | securitycontacts | No | No |
> | securitysolutions | No | No |
> | securitysolutionsreferencedata | No | No |
> | securitystatuses | No | No |
> | securitystatusessummaries | No | No |
> | servervulnerabilityassessments | No | No |
> | configuración | No | No |
> | subassessments | No | No |
> | tareas | No | No |
> | topologies | No | No |
> | workspacesettings | No | No |

## <a name="microsoftsecurityinsights"></a>Microsoft.SecurityInsights

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | aggregations | No | No |
> | alertrules | No | No |
> | alertruletemplates | No | No |
> | automationrules | No | No |
> | bookmarks | No | No |
> | cases | No | No |
> | dataconnectors | No | No |
> | entities | No | No |
> | entityqueries | No | No |
> | incidents | No | No |
> | officeconsents | No | No |
> | configuración | No | No |
> | threatintelligence | No | No |

## <a name="microsoftserialconsole"></a>Microsoft.SerialConsole

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | consoleservices | No | No |

## <a name="microsoftservermanagement"></a>Microsoft.ServerManagement

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | gateways | No | No |
> | nodes | No | No |

## <a name="microsoftservicebus"></a>Microsoft.ServiceBus

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | espacios de nombres | Sí | Sí |
> | premiummessagingregions | No | No |
> | sku | No | No |

## <a name="microsoftservicefabric"></a>Microsoft.ServiceFabric

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | applications | No | No |
> | clusters | Sí | Sí |
> | containergroups | No | No |
> | containergroupsets | No | No |
> | edgeclusters | No | No |
> | managedclusters | No | No |
> | networks | No | No |
> | secretstores | No | No |
> | volumes | No | No |

## <a name="microsoftservicefabricmesh"></a>Microsoft.ServiceFabricMesh

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | applications | Sí | Sí |
> | containergroups | No | No |
> | gateways | Sí | Sí |
> | networks | Sí | Sí |
> | secrets | Sí | Sí |
> | volumes | Sí | Sí |

## <a name="microsoftservices"></a>Microsoft.Services

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | rollouts | No | No |

## <a name="microsoftsignalrservice"></a>Microsoft.SignalRService

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | signalr | Sí | Sí |

## <a name="microsoftsoftwareplan"></a>Microsoft.SoftwarePlan

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | hybridusebenefits | No | No |

## <a name="microsoftsolutions"></a>Microsoft.Solutions

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | applicationdefinitions | No | No |
> | applications | No | No |
> | jitrequests | No | No |

## <a name="microsoftsql"></a>Microsoft.Sql

> [!IMPORTANT]
> La base de datos y el servidor deben residir en el mismo grupo de recursos. Cuando se mueve un servidor SQL Server, se mueven también todas sus bases de datos. Este comportamiento se aplica a las bases de datos de Azure SQL Database y Azure Synapse Analytics.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | instancepools | No | No |
> | locations | Sí | Sí |
> | managedinstances | No | No |
> | servers | Sí | Sí |
> | servers/databases | Sí | Sí |
> | servers/databases/backuplongtermretentionpolicies | Sí | Sí |
> | servers/elasticpools | Sí | Sí |
> | servers / jobaccounts | Sí | Sí |
> | servers / jobagents | Sí | Sí |
> | virtualclusters | Sí | Sí |

## <a name="microsoftsqlvirtualmachine"></a>Microsoft.SqlVirtualMachine

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | sqlvirtualmachinegroups | Sí | Sí |
> | sqlvirtualmachines | Sí | Sí |

## <a name="microsoftstorage"></a>Microsoft.Storage

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | storageaccounts | Sí | Sí |

## <a name="microsoftstoragecache"></a>Microsoft.StorageCache

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | caches | No | No |

## <a name="microsoftstoragesync"></a>Microsoft.StorageSync

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | storagesyncservices | Sí | Sí |

## <a name="microsoftstoragesyncdev"></a>Microsoft.StorageSyncDev

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | storagesyncservices | No | No |

## <a name="microsoftstoragesyncint"></a>Microsoft.StorageSyncInt

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | storagesyncservices | No | No |

## <a name="microsoftstorsimple"></a>Microsoft.StorSimple

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | managers | No | No |

## <a name="microsoftstreamanalytics"></a>Microsoft.StreamAnalytics

> [!IMPORTANT]
> Los trabajos de Stream Analytics no se pueden mover si se encuentran en estado de ejecución.

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | clusters | No | No |
> | streamingjobs | Sí | Sí |

## <a name="microsoftstreamanalyticsexplorer"></a>Microsoft.StreamAnalyticsExplorer

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | environments | No | No |
> | instances | No | No |

## <a name="microsoftsubscription"></a>Microsoft.Subscription

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | subscriptions | No | No |

## <a name="microsoftsupport"></a>microsoft.support

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | services | No | No |
> | supporttickets | No | No |

## <a name="microsoftsynapse"></a>Microsoft.Synapse

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | workspaces | No | No |
> | workspaces / bigdatapools | No | No |
> | workspaces / sqlpools | No | No |

## <a name="microsofttimeseriesinsights"></a>Microsoft.TimeSeriesInsights

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | environments | Sí | Sí |
> | environments/eventsources | Sí | Sí |
> | environments/referencedatasets | Sí | Sí |

## <a name="microsofttoken"></a>Microsoft.Token

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | almacenes | Sí | Sí |

## <a name="microsoftvirtualmachineimages"></a>Microsoft.VirtualMachineImages

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | imagetemplates | No | No |

## <a name="microsoftvisualstudio"></a>microsoft.visualstudio

> [!IMPORTANT]
> Para cambiar la suscripción de Azure DevOps, vea [Cambiar la suscripción de Azure usada para la facturación](/azure/devops/organizations/billing/change-azure-subscription?toc=/azure/azure-resource-manager/toc.json).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | account | No | No |
> | account/extension | No | No |
> | account/project | No | No |

## <a name="microsoftvmware"></a>Microsoft.VMware

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | arczones | No | No |
> | resourcepools | No | No |
> | vcenters | No | No |
> | virtualmachines | No | No |
> | virtualmachinetemplates | No | No |
> | virtualnetworks | No | No |

## <a name="microsoftvmwarecloudsimple"></a>Microsoft.VMwareCloudSimple

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | dedicatedcloudnodes | No | No |
> | dedicatedcloudservices | No | No |
> | virtualmachines | No | No |

## <a name="microsoftvnfmanager"></a>Microsoft.VnfManager

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | devices | No | No |
> | vnfs | No | No |

## <a name="microsoftvsonline"></a>Microsoft.VSOnline

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | accounts | No | No |
> | plans | No | No |
> | registeredsubscriptions | No | No |

## <a name="microsoftweb"></a>Microsoft.Web

> [!IMPORTANT]
> Vea [Guía de movimiento de App Service](./move-limitations/app-service-move-limitations.md).

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | availablestacks | No | No |
> | billingmeters | No | No |
> | certificates | No | Sí |
> | connectiongateways | Sí | Sí |
> | connections | Sí | Sí |
> | customapis | Sí | Sí |
> | deletedsites | No | No |
> | deploymentlocations | No | No |
> | georegions | No | No |
> | hostingenvironments | No | No |
> | kubeenvironments | Sí | Sí |
> | publishingusers | No | No |
> | de películas | No | No |
> | resourcehealthmetadata | No | No |
> | runtimes | No | No |
> | serverfarms | Sí | Sí |
> | serverfarms/eventgridfilters | No | No |
> | sites | Sí | Sí |
> | sites/premieraddons | Sí | Sí |
> | sites/slots | Sí | Sí |
> | sourcecontrols | No | No |
> | staticsites | No | No |

## <a name="microsoftwindowsesu"></a>Microsoft.WindowsESU

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Subscription |
> | ------------- | ----------- | ---------- |
> | multipleactivationkeys | No | No |

## <a name="microsoftwindowsiot"></a>Microsoft.WindowsIoT

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | deviceservices | No | No |

## <a name="microsoftworkloadbuilder"></a>Microsoft.WorkloadBuilder

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | workloads | No | No |

## <a name="microsoftworkloadmonitor"></a>Microsoft.WorkloadMonitor

> [!div class="mx-tableFixed"]
> | Tipo de recurso | Resource group | Suscripción |
> | ------------- | ----------- | ---------- |
> | components | No | No |
> | componentssummary | No | No |
> | monitorinstances | No | No |
> | monitorinstancessummary | No | No |
> | monitors | No | No |

## <a name="third-party-services"></a>Servicios de terceros

Actualmente los servicios de terceros no son compatibles con la operación de traslado.

## <a name="next-steps"></a>Pasos siguientes

Para ver los comandos para trasladar recursos, vea [Traslado de los recursos a un nuevo grupo de recursos o a una nueva suscripción](move-resource-group-and-subscription.md).

Para obtener los mismos datos como un archivo de valores separados por comas, descargue [move-support-resources.csv](https://github.com/tfitzmac/resource-capabilities/blob/master/move-support-resources.csv).
