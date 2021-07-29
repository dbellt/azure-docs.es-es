---
author: linda33wj
ms.service: data-factory
ms.topic: include
ms.date: 07/13/2020
ms.author: jingwang
ms.openlocfilehash: 66b7159d19838cf08158d794f9b8e8819efc3f41
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112040129"
---
<!--
    Separate the generic requirement on Self-hosted Integration Runtime setup from connector articles.
-->
Si el almacén de datos se encuentra en una red local, una red virtual de Azure o una nube privada virtual de Amazon, debe configurar un [entorno de ejecución de integración autohospedado](../create-self-hosted-integration-runtime.md) para conectarse a él.

Si el almacén de datos es un servicio de datos en la nube administrado, puede usar Azure Integration Runtime. Si el acceso está restringido a las direcciones IP que están aprobadas en las reglas de firewall, puede agregar [direcciones IP de Azure Integration Runtime](../azure-integration-runtime-ip-addresses.md) a la lista de permitidos. 

También puede usar la característica del [entorno de ejecución de integración de red virtual administrada](../tutorial-managed-virtual-network-on-premise-sql-server.md) de Azure Data Factory para acceder a la red local sin instalar ni configurar un entorno de ejecución de integración autohospedado.

Consulte [Estrategias de acceso a datos](../data-access-strategies.md) para más información sobre los mecanismos de seguridad de red y las opciones que admite Data Factory.
