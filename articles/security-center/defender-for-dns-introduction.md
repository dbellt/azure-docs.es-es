---
title: 'Azure Defender para DNS: ventajas y características'
description: Obtenga información sobre las ventajas y características de Azure Defender para DNS.
author: memildin
ms.author: memildin
ms.date: 05/12/2021
ms.topic: overview
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 976ffc86fd82bb85fefb16b4f442111a07d1f652
ms.sourcegitcommit: a038863c0a99dfda16133bcb08b172b6b4c86db8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/29/2021
ms.locfileid: "113003609"
---
# <a name="introduction-to-azure-defender-for-dns"></a>Introducción a Azure Defender para DNS

[Azure DNS](../dns/dns-overview.md) es un servicio de hospedaje para dominios de DNS que ofrece resolución de nombres mediante la infraestructura de Microsoft Azure. Al hospedar dominios en Azure, puede administrar los registros de DNS con las mismas credenciales, API, herramientas y facturación que con los demás servicios de Azure.

Azure Defender para DNS proporciona una capa adicional de protección a los recursos, ya que están conectados a Azure DNS mediante lo siguiente:

- supervisión continua de todas las consultas de DNS de los recursos de Azure
- ejecución de análisis de seguridad avanzado y generación de una alerta si detecta alguna actividades sospechosa

## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Disponibilidad general (GA)|
|Precios:|**Azure Defender para DNS** se factura como se indica en [Precios de Security Center](https://azure.microsoft.com/pricing/details/security-center/).|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![No](./media/icons/no-icon.png) Nacionales o soberanas (US Gov, Azure China)|
|||

## <a name="what-are-the-benefits-of-azure-defender-for-dns"></a>Ventajas de Azure Defender para DNS

Azure Defender para DNS protege los recursos que están conectados a Azure DNS frente a problemas, entre los que se incluyen:

- Filtrado de datos de los recursos de Azure mediante la tunelización de DNS.
- Malware que se comunica con el servidor de comando y control.
- Comunicación con dominios malintencionados como suplantación de identidad (phishing) y minería de datos de cifrado
- Ataques de DNS: comunicación con solucionadores de DNS malintencionados 

En la [página de referencia de alertas](alerts-reference.md#alerts-dns) se encuentra una lista completa de las alertas que proporciona Azure Defender para DNS.

## <a name="dependencies"></a>Dependencias

Azure Defender para DNS no usa agentes. 

Para proteger la capa de DNS, habilite Azure Defender para DNS en todas las suscripciones, tal como se describe en el apartado [Habilitación de Azure Defender](enable-azure-defender.md).


## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha obtenido información sobre Azure Defender para DNS. Para obtener material relacionado, vea el siguiente artículo: 

- Las alertas de seguridad las puede haber generado Security Center, o bien las puede haber recibido Security Center de distintos productos de seguridad. Para exportar todas estas alertas a Azure Sentinel, a cualquier SIEM de terceros o a cualquier otra herramienta externa, siga las instrucciones de [Exportación de alertas a un SIEM](continuous-export.md).

- > [!div class="nextstepaction"]
    > [Habilitación de Azure Defender](enable-azure-defender.md)