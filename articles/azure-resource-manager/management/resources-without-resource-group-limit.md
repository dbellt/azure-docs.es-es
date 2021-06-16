---
title: Recursos sin límite de recuento de 800
description: Aquí se enumeran los tipos de recursos de Azure que pueden tener más de 800 instancias en un grupo de recursos.
ms.topic: conceptual
ms.date: 04/12/2021
ms.openlocfilehash: f77bd964003638ca33e1f30e352b1f35a866ac6a
ms.sourcegitcommit: 23040f695dd0785409ab964613fabca1645cef90
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112060253"
---
# <a name="resources-not-limited-to-800-instances-per-resource-group"></a>Recursos no limitados a 800 instancias por grupo de recursos

De forma predeterminada, puede implementar hasta 800 instancias de un tipo de recurso en cada grupo de recursos. Sin embargo, algunos tipos de recursos están exentos del límite de 800 instancias. En este artículo se enumeran los tipos de recursos de Azure que pueden tener más de 800 instancias en un grupo de recursos. Todos los demás tipos de recursos se limitan a 800 instancias.

Para usar algunos tipos de recursos, debe ponerse en contacto con el servicio de soporte técnico para que se elimine el límite de 800 instancias. Esos tipos de recursos se indican en este artículo.


## <a name="microsoftalertsmanagement"></a>Microsoft.AlertsManagement

* resourceHealthAlertRules
* smartDetectorAlertRules

## <a name="microsoftautomation"></a>Microsoft.Automation

* automationAccounts

## <a name="microsoftazurestack"></a>Microsoft.AzureStack

* edgeSubscriptions
* linkedSubscriptions
* registrations
* registrations/customerSubscriptions
* registrations/products

## <a name="microsoftbotservice"></a>Microsoft.BotService

* botServices: de forma predeterminada, con un límite de 800 instancias. Este límite se puede incrementar poniéndose en contacto con soporte técnico.

## <a name="microsoftcompute"></a>Microsoft.Compute

* disks
* galleries
* galleries/images
* galleries/images/versions
* images
* snapshots
* virtualMachineScaleSets: de forma predeterminada, está limitado a 800 instancias. Este límite se puede incrementar poniéndose en contacto con soporte técnico.
* virtualMachines

## <a name="microsoftcontainerinstance"></a>Microsoft.ContainerInstance

* containerGroups

## <a name="microsoftcontainerregistry"></a>Microsoft.ContainerRegistry

* registries/buildTasks
* registries/buildTasks/listSourceRepositoryProperties
* registries/buildTasks/steps
* registries/buildTasks/steps/listBuildArguments
* registries/eventGridFilters
* registries/replications
* registries/tasks
* registries/webhooks

## <a name="microsoftd365customerinsights"></a>Microsoft.D365CustomerInsights

* instances

## <a name="microsoftdbformariadb"></a>Microsoft.DBforMariaDB

* servers

## <a name="microsoftdbformysql"></a>Microsoft.DBforMySQL

* flexibleServers
* servers

## <a name="microsoftdbforpostgresql"></a>Microsoft.DBforPostgreSQL

* flexibleServers
* serverGroups
* serverGroupsv2
* servers
* serversv2

## <a name="microsoftdevtestlab"></a>Microsoft.DevTestLab

* schedules

## <a name="microsoftenterpriseknowledgegraph"></a>Microsoft.EnterpriseKnowledgeGraph

* services

## <a name="microsofteventhub"></a>Microsoft.EventHub

* clusters
* espacios de nombres

## <a name="microsoftexperimentation"></a>Microsoft.Experimentation

* experimentWorkspaces

## <a name="microsoftguestconfiguration"></a>Microsoft.GuestConfiguration

* autoManagedVmConfigurationProfiles
* configurationProfileAssignments
* guestConfigurationAssignments
* software
* softwareUpdateProfile
* softwareUpdates

## <a name="microsofthybridcompute"></a>Microsoft.HybridCompute

* machines: admite hasta 5000 instancias.
* machines/extensions: admite un número ilimitado de instancias de extensión de VM.

## <a name="microsoftinsights"></a>microsoft.insights

* metricalerts
* scheduledQueryRules

## <a name="microsoftlogic"></a>Microsoft.Logic

* integrationAccounts
* workflows

## <a name="microsoftmedia"></a>Microsoft.Media

* mediaservices/liveEvents

## <a name="microsoftnetapp"></a>Microsoft.NetApp

* netAppAccounts
* netAppAccounts/capacityPools
* netAppAccounts/capacityPools/volumes
* netAppAccounts/capacityPools/volumes/mountTargets
* netAppAccounts/capacityPools/volumes/snapshots
* netAppAccounts/volumeGroups

## <a name="microsoftnetwork"></a>Microsoft.Network

* applicationGatewayWebApplicationFirewallPolicies
* applicationSecurityGroups
* bastionHosts
* ddosProtectionPlans
* dnszones
* dnszones/A
* dnszones/AAAA
* dnszones/CAA
* dnszones/CNAME
* dnszones/MX
* dnszones/NS
* dnszones/PTR
* dnszones/SOA
* dnszones/SRV
* dnszones/TXT
* dnszones/all
* dnszones/recordsets
* networkIntentPolicies
* networkInterfaces
* privateDnsZones
* privateDnsZones/A
* privateDnsZones/AAAA
* privateDnsZones/CNAME
* privateDnsZones/MX
* privateDnsZones/PTR
* privateDnsZones/SOA
* privateDnsZones/SRV
* privateDnsZones/TXT
* privateDnsZones/all
* privateDnsZones/virtualNetworkLinks
* privateEndpoints
* privateLinkServices
* publicIPAddresses: de forma predeterminada, con un límite de 800 instancias. Este límite se puede incrementar poniéndose en contacto con soporte técnico.
* serviceEndpointPolicies
* trafficmanagerprofiles
* virtualNetworkTaps

## <a name="microsoftportalsdk"></a>Microsoft.PortalSdk

* rootResources

## <a name="microsoftpowerbi"></a>Microsoft.PowerBI

* workspaceCollections: de forma predeterminada, con un límite de 800 instancias. Este límite se puede incrementar poniéndose en contacto con soporte técnico.

## <a name="microsoftpowerbidedicated"></a>Microsoft.PowerBIDedicated

* autoScaleVCores: de forma predeterminada, está limitado a 800 instancias. Este límite se puede incrementar poniéndose en contacto con soporte técnico.
* capacities: de forma predeterminada, está limitado a 800 instancias. Este límite se puede incrementar poniéndose en contacto con soporte técnico.

## <a name="microsoftrelay"></a>Microsoft.Relay

* espacios de nombres

## <a name="microsoftscheduler"></a>Microsoft.Scheduler

* jobcollections

## <a name="microsoftservicebus"></a>Microsoft.ServiceBus

* espacios de nombres

## <a name="microsoftsingularity"></a>Microsoft.Singularity

* accounts
* accounts/accountQuotaPolicies
* accounts/groupPolicies
* accounts/jobs
* accounts/storageContainers

## <a name="microsoftstorage"></a>Microsoft.Storage

* storageAccounts

## <a name="microsoftsql"></a>Microsoft.Sql

* servers/databases

## <a name="microsoftweb"></a>Microsoft.Web

* apiManagementAccounts/apis
* sites

## <a name="next-steps"></a>Pasos siguientes

Para obtener una lista completa de las cuotas y los límites, consulte [Suscripción de Azure y límites de servicio, cuotas y restricciones](azure-subscription-service-limits.md).
