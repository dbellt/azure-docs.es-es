---
author: linda33wj
ms.service: data-factory
ms.topic: include
ms.date: 07/13/2020
ms.author: jingwang
ms.openlocfilehash: fbde8bc28f8fc34b7a6a6443950b8733c6dcff45
ms.sourcegitcommit: 32e0fedb80b5a5ed0d2336cea18c3ec3b5015ca1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "96023214"
---
<!--
    Separate the generic requirement on Self-hosted Integration Runtime set-up from connector articles.
-->
Si el almacén de datos se encuentra en una red local, una red virtual de Azure o una nube privada virtual de Amazon, debe configurar un [entorno de ejecución de integración autohospedado](../articles/data-factory/create-self-hosted-integration-runtime.md) para conectarse a él.

O bien, si el almacén de datos es un servicio de datos en la nube administrado, puede usar Azure Integration Runtime. Si el acceso está restringido a las direcciones IP que están aprobadas en las reglas de firewall, puede agregar [direcciones IP de Azure Integration Runtime](../articles/data-factory/azure-integration-runtime-ip-addresses.md) a la lista de permitidos. 

Consulte [Estrategias de acceso a datos](../articles/data-factory/data-access-strategies.md) para más información sobre los mecanismos de seguridad de red y las opciones que admite Data Factory.
