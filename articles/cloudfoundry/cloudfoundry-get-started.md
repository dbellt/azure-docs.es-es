---
title: Introducción a Cloud Foundry en Microsoft Azure
description: Ejecución de OSS o Pivotal Cloud Foundry en Microsoft Azure
author: seanmck
ms.service: virtual-machines
ms.subservice: workloads
ms.topic: article
ms.date: 01/19/2017
ms.author: seanmck
ms.openlocfilehash: 2f80f07cad0e82ee35fe71223e1282ea24f791bb
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107530933"
---
# <a name="cloud-foundry-on-azure"></a>Cloud Foundry en Azure

Cloud Foundry es una plataforma como servicio (PaaS) de código abierto para compilar, implementar y operar aplicaciones de factor 12 desarrolladas en diversos lenguajes y marcos. En este documento, se describen las opciones disponibles para ejecutar Cloud Foundry en Azure y cómo puede comenzar a trabajar.

## <a name="cloud-foundry-offerings"></a>Ofertas de Cloud Foundry

Existen dos formas de Cloud Foundry que se ejecutan en Azure: Cloud Foundry de código abierto (OSS CF) y Pivotal Cloud Foundry (PCF). OSS CF es una versión totalmente de [código abierto](https://github.com/cloudfoundry) de Cloud Foundry administrada por Cloud Foundry Foundation. Pivotal Cloud Foundry es una distribución empresarial de Cloud Foundry de Pivotal Software Inc. Analicemos algunas de las diferencias entre ambas ofertas.

### <a name="open-source-cloud-foundry"></a>Cloud Foundry de código abierto

Puede implementar OSS Cloud Foundry en Azure si primero implementa un director BOSH y después Cloud Foundry, mediante las [instrucciones proporcionadas en GitHub](https://github.com/cloudfoundry-incubator/bosh-azure-cpi-release/blob/master/docs/guidance.md). Para aprender más sobre el uso de OSS CF, consulte la [documentación](https://docs.cloudfoundry.org/) proporcionada por Cloud Foundry Foundation.

Microsoft se esfuerza al máximo en ofrecer asistencia para OSS CF a través de los siguientes canales comunitarios:

- #<a name="bosh-azure-cpi-channel-on-cloud-foundry-slack"></a>Canal bosh-azure-cpi en [Cloud Foundry Slack](https://slack.cloudfoundry.org/)
- [Lista de correo cf-bosh](https://lists.cloudfoundry.org/pipermail/cf-bosh)
- Problemas de GitHub para [CPI](https://github.com/cloudfoundry-incubator/bosh-azure-cpi-release/issues) y el [agente de servicio](https://github.com/Azure/meta-azure-service-broker/issues)

>[!NOTE]
> El nivel de asistencia para los recursos de Azure, como las máquinas virtuales donde se ejecuta Cloud Foundry, depende de su contrato de soporte de Azure. La asistencia comunitaria de este tipo solo se aplica a los componentes específicos de Cloud Foundry.

### <a name="pivotal-cloud-foundry"></a>Pivotal Cloud Foundry

Pivotal Cloud Foundry incluye la misma plataforma central que la distribución OSS, además de un conjunto de herramientas de administración propias y soporte técnico empresarial. Para ejecutar PCF en Azure, debe comprar una licencia a Pivotal. La oferta PCF de Azure Marketplace incluye una licencia de prueba de 90 días.

Entre las herramientas, se incluyen [Pivotal Operations Manager](https://docs.pivotal.io/ops-manager/2-10/install/), una aplicación web que simplifica la implementación y la administración de una fundación de Cloud Foundry, y [Pivotal Apps Manager](https://docs.pivotal.io/pivotalcf/console/), una aplicación web para administrar usuarios y aplicaciones.

Además de los canales indicados antes para OSS CF, una licencia PCF le da derecho a recibir soporte técnico de Pivotal. Microsoft y Pivotal también han habilitado flujos de trabajo de soporte que permiten ponerse en contacto con cualquiera de ellos para obtener ayuda y redirigir su consulta en función de dónde se encuentre el problema.

## <a name="azure-service-broker"></a>Agente de servicio de Azure

Cloud Foundry fomenta la metodología de ["aplicaciones de doce factores"](https://12factor.net/), que promueve una separación clara entre los procesos de aplicación sin estado y los servicios de respaldo con estado. Los [agentes de servicio](https://docs.cloudfoundry.org/services/api.html) proporcionan una manera uniforme de aprovisionar y enlazar los servicios de respaldo a las aplicaciones. El [agente de servicio de Azure](https://github.com/Azure/meta-azure-service-broker) proporciona algunos de los servicios clave de Azure a través de este canal, incluidos Azure Storage y Azure SQL.

Si usa Pivotal Cloud Foundry, el agente de servicio también está [disponible como icono](https://docs.pivotal.io/azure-sb/installing.html) en Pivotal Network.

## <a name="related-resources"></a>Recursos relacionados

### <a name="azure-devops-services-plugin"></a>Complemento Azure DevOps Services

Cloud Foundry se adapta perfectamente al desarrollo ágil de software, incluido el uso de la integración continua (CI) y la entrega continua (CD). Si usa Azure DevOps Services para administrar los proyectos y le gustaría configurar una canalización de CI/CD hacia Cloud Foundry, puede usar la [extensión de compilación Cloud Foundry para Azure DevOps Services](https://marketplace.visualstudio.com/items?itemName=ms-vsts.cloud-foundry-build-extension). El complemento simplifica el proceso de configurar y automatizar implementaciones en Cloud Foundry, tanto si se ejecuta en Azure como en otro entorno.

## <a name="next-steps"></a>Pasos siguientes

- [Implementar Pivotal Cloud Foundry desde Azure Marketplace](https://azuremarketplace.microsoft.com/marketplace/apps/pivotal.pivotal-cloud-foundry)
- [Implementar una aplicación en Cloud Foundry en Azure](./cloudfoundry-deploy-your-first-app.md)
