---
author: linda33wj
ms.service: data-factory
ms.topic: include
ms.date: 07/13/2020
ms.author: jingwang
ms.openlocfilehash: f2d93449744ea7993ec22f200c51fea8280d1576
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108747813"
---
<!--
    Separate the generic requirement on Self-hosted Integration Runtime set-up from connector articles.
-->
Si el almacén de datos se encuentra en una red local, una red virtual de Azure o una nube privada virtual de Amazon, debe configurar un [entorno de ejecución de integración autohospedado](../create-self-hosted-integration-runtime.md) para conectarse a él.

O bien, si el almacén de datos es un servicio de datos en la nube administrado, puede usar Azure Integration Runtime. Si el acceso está restringido a las direcciones IP que están aprobadas en las reglas de firewall, puede agregar [direcciones IP de Azure Integration Runtime](../azure-integration-runtime-ip-addresses.md) a la lista de permitidos. 

Consulte [Estrategias de acceso a datos](../data-access-strategies.md) para más información sobre los mecanismos de seguridad de red y las opciones que admite Data Factory.
