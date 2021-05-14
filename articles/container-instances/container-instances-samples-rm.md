---
title: Ejemplos de plantillas de Azure Resource Manager
description: Busque ejemplos de plantillas de Azure Resource Manager para implementar Azure Container Instances en distintas configuraciones.
ms.topic: article
ms.date: 03/07/2019
ms.openlocfilehash: 12ecfc5c1066aad39ef2e3c10832d461426b5158
ms.sourcegitcommit: eda26a142f1d3b5a9253176e16b5cbaefe3e31b3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/11/2021
ms.locfileid: "109733701"
---
# <a name="azure-resource-manager-templates-for-azure-container-instances"></a>Plantillas de Azure Resource Manager para Azure Container Instances

Las siguientes plantillas de ejemplo implementan instancias de contenedor en diversas configuraciones.

Para opciones de implementación, consulte la sección [Implementación](#deployment). Si desea crear sus propias plantillas, la [Referencia de plantillas de Resource Manager][ref]de Azure Container Instances detalla el formato de la plantilla y las propiedades disponibles.

## <a name="sample-templates"></a>Plantillas de ejemplo

| Plantilla | Descripción |
|-|-|
| **Aplicaciones** ||
| [WordPress][app-wp] | Crea un sitio web de WordPress y su base de datos MySQL en un grupo de contenedores. El contenido del sitio de WordPress y la base de datos MySQL se conservan en un recurso compartido de Azure Files. También crea una instancia de puerta de enlace para exponer el acceso a la red pública de WordPress. |
| [MS NAV con IIS y SQL Server][app-nav] | Implementa un único contenedor de Windows con un entorno de Dynamics NAV y Dynamics 365 Business Central con las características completas. |
| **Volúmenes** ||
| [emptyDir][vol-emptydir] | Implementa dos contenedores de Linux que comparten un volumen de emptyDir. |
| [gitRepo][vol-gitrepo] | Implementa un contenedor de Linux que clona un repositorio de GitHub y lo monta como un volumen. |
| [secret][vol-secret] | Implementa un contenedor de Linux con un certificado PFX montado como un volumen secreto. |
| **Redes** ||
| [Contenedor con UDP expuesto][net-udp] | Implementa un contenedor de Windows o Linux que expone un puerto UDP. |
| [Contenedor de Linux con dirección IP pública][net-publicip] | Implementa un único contenedor de Linux accesible a través de una dirección IP pública. |
| [Implementación de un grupo de contenedores con una red virtual][net-vnet] | Implementa una red virtual, subred, perfil de red y grupo de contenedores nuevos. |
| **Recursos de Azure** ||
| [Creación de cuenta de Azure Storage y recurso compartido de archivos][az-files] | Usa la CLI de Azure en una instancia del contenedor para crear una cuenta de almacenamiento y un recurso compartido de Azure Files.

## <a name="deployment"></a>Implementación

Tiene varias opciones para implementar recursos con plantillas de Resource Manager:

[CLI de Azure][deploy-cli]

[Azure PowerShell][deploy-powershell]

[Azure Portal][deploy-portal]

[REST API][deploy-rest]

<!-- LINKS - External -->
[app-nav]: https://github.com/Azure/azure-quickstart-templates/tree/master/101-aci-dynamicsnav
[app-wp]: https://github.com/Azure/azure-quickstart-templates/tree/master/201-aci-wordpress
[az-files]: https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.containerinstance/aci-storage-file-share
[net-publicip]: https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.containerinstance/aci-linuxcontainer-public-ip
[net-udp]: https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.containerinstance/aci-udp
[net-vnet]: https://github.com/Azure/azure-quickstart-templates/tree/master/101-aci-vnet
[repo]: https://github.com/Azure/azure-quickstart-templates
[vol-emptydir]: https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.containerinstance/aci-linuxcontainer-volume-emptydir
[vol-gitrepo]: https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.containerinstance/aci-linuxcontainer-volume-gitrepo
[vol-secret]: https://github.com/Azure/azure-quickstart-templates/tree/master/quickstarts/microsoft.containerinstance/aci-linuxcontainer-volume-secret

<!-- LINKS - Internal -->
[deploy-cli]: ../azure-resource-manager/templates/deploy-cli.md
[deploy-portal]: ../azure-resource-manager/templates/deploy-portal.md
[deploy-powershell]: ../azure-resource-manager/templates/deploy-powershell.md
[deploy-rest]: ../azure-resource-manager/templates/deploy-rest.md
[ref]: /azure/templates/microsoft.containerinstance/containergroups
