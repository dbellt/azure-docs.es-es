---
title: Introducción a Azure Defender y los planes disponibles
description: Conozca los planes, las protecciones y las alertas de Azure Defender. A continuación, habilite Azure Defender en sus suscripciones para la seguridad avanzada.
author: memildin
ms.author: memildin
ms.date: 9/30/2020
ms.topic: overview
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 2c8147399f37f5f9fe2b3c01bc2c273ffc59be31
ms.sourcegitcommit: 80d311abffb2d9a457333bcca898dfae830ea1b4
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/26/2021
ms.locfileid: "110469614"
---
# <a name="introduction-to-azure-defender"></a>Introducción a Azure Defender

Las características de Azure Security Center abarcan los dos extensos pilares de seguridad en la nube:

- **Administración de la posición de seguridad en la nube (CSPM)** : Security Center está disponible de forma **gratuita** para todos los usuarios de Azure. La experiencia gratuita incluye características de CSPM, como la puntuación segura, la detección de configuraciones incorrectas de seguridad en las máquinas de Azure, el inventario de recursos, etc. Use estas características de CSPM para fortalecer la posición en la nube híbrida y realizar un seguimiento del cumplimiento con las directivas integradas.

- **Protección de la carga de trabajo en la nube (CWP)** : la plataforma integrada de protección de cargas de trabajo en la nube (CWP) de Security Center, **Azure Defender**, ofrece la protección avanzada e inteligente de los recursos y las cargas de trabajo híbridas de Azure. La habilitación de Azure Defender aporta diversas características de seguridad adicionales, como se describe en esta página. Además de las directivas integradas, cuando haya habilitado un plan de Azure Defender, puede agregar directivas e iniciativas personalizadas. Puede agregar estándares normativos, como NIST y Azure CIS, así como [Azure Security Benchmark](/security/benchmark/azure/introduction) para obtener una vista realmente personalizada de su cumplimiento.

El panel de Azure Defender en Security Center permite la visibilidad y el control de las características de CWP en un entorno:

:::image type="content" source="./media/azure-defender/sample-defender-dashboard.png" alt-text="Ejemplo del panel de Azure Defender" lightbox="./media/azure-defender/sample-defender-dashboard.png":::

## <a name="what-resource-types-can-azure-defender-secure"></a>¿Qué tipos de recursos puede proteger Azure Defender?

Azure Defender proporciona alertas de seguridad y protección contra amenazas avanzada para máquinas virtuales, bases de datos SQL, contenedores, aplicaciones web, la red y mucho más.

Al habilitar Azure Defender en el área **Precios y configuración** de Azure Security Center, se habilitan a la vez los siguientes planes de Defender, que proporcionan defensas completas para las capas de proceso, datos y servicio de su entorno:

- [Azure Defender para servidores](defender-for-servers-introduction.md)
- [Azure Defender para App Service](defender-for-app-service-introduction.md)
- [Azure Defender para Storage](defender-for-storage-introduction.md)
- [Azure Defender para SQL](defender-for-sql-introduction.md)
- [Azure Defender para Kubernetes](defender-for-kubernetes-introduction.md)
- [Azure Defender para registros de contenedor](defender-for-container-registries-introduction.md)
- [Azure Defender para Key Vault](defender-for-key-vault-introduction.md)
- [Azure Defender para Resource Manager](defender-for-resource-manager-introduction.md)
- [Azure Defender para DNS](defender-for-dns-introduction.md)
- [Azure Defender para bases de datos relacionales de código abierto](defender-for-databases-introduction.md)

Cada uno de estos planes se explica por separado en la documentación de Security Center.

> [!TIP]
> Azure Defender para IoT (versión preliminar) es un producto independiente. Encontrará todos los detalles en [Presentación de la versión preliminar de Azure Defender para IoT](../defender-for-iot/overview.md). 

## <a name="hybrid-cloud-protection"></a>Protección de la nube híbrida

Además de defender su entorno de Azure, puede agregar funcionalidades de Azure Defender al entorno de nube híbrida:

- Protección de los servidores que no son de Azure
- Proteja sus máquinas virtuales de otras nubes (como AWS y GCP)

Obtenga inteligencia personalizada sobre las amenazas y alertas prioritarias según su entorno específico para que pueda centrarse en lo que más le importa.

Para ampliar la protección a las máquinas virtuales y a las base de datos SQL que estén en otras nubes o en el entorno local, implemente [Azure Arc](https://azure.microsoft.com/services/azure-arc/) y habilite Azure Defender. Azure Arc para servidores es un servicio gratuito, pero los servicios que se usan en servidores habilitados para Arc, por ejemplo, Azure Defender, se cobran según los precios de ese servicio. Obtenga más información en [Incorporación de máquinas que no sean de Azure con Azure Arc](quickstart-onboard-machines.md#add-non-azure-machines-with-azure-arc).

> [!TIP]
> El conector nativo para AWS controla de forma transparente la implementación de Azure Arc. Obtenga más información en [Conexión de las cuentas de AWS a Azure Security Center](quickstart-onboard-aws.md).



## <a name="azure-defender-security-alerts"></a>Alertas de seguridad de Azure Defender 

Cuando Azure Defender detecta una amenaza en cualquiera de las áreas del entorno, genera una alerta de seguridad. Estas alertas describen los detalles de los recursos afectados, los pasos de corrección sugeridos y, en algunos casos, una opción para desencadenar una aplicación lógica como respuesta.

Tanto si Security Center genera una alerta, como si la recibe de un producto de seguridad integrado, puede exportarse. Para exportar las alertas a Azure Sentinel, la SIEM de terceros, o cualquier otra herramienta externa, siguen las instrucciones de [Transmisión de alertas a una solución de administración de servicios de TI, SIEM o SOAR](export-to-siem.md).

> [!NOTE]
> Las alertas de orígenes diferentes pueden tardar un tiempo distinto en aparecer. Por ejemplo, las alertas que requieren un análisis del tráfico de red pueden tardar más en aparecer que las alertas relacionadas con procesos sospechosos que se ejecutan en máquinas virtuales.


## <a name="azure-defender-advanced-protection-capabilities"></a>Funcionalidades de protección avanzada de Azure Defender

Azure Defender usa análisis avanzado para las recomendaciones adaptadas relacionadas con los recursos. 

Entre las protecciones se incluyen la protección de los puertos de administración de las máquinas virtuales con acceso Just-in-Time y controles de aplicaciones adaptables para crear listas de permitidos con las aplicaciones que deben o no ejecutarse en las máquinas. 

Use los iconos de protección avanzada del panel de Azure Defender para supervisar y configurar cada una de estas protecciones. 

## <a name="vulnerability-assessment-and-management"></a>Evaluación y administración de vulnerabilidades

Azure Defender incluye el examen de vulnerabilidades de las máquinas virtuales y de los registros de contenedor sin costo adicional. Los exámenes se basan en la tecnología de Qualys, pero no se necesita ninguna licencia ni cuenta de Qualys, ya que todo se administra íntegramente en Security Center. 

Revise los resultados de estos exámenes de vulnerabilidades y responda a todos ellos desde Security Center. De esta forma, Security Center se aproxima a convertirse en el panel por excelencia para todos los trabajos de seguridad en la nube.

Más información en las siguientes páginas:

- [Solución de evaluación de vulnerabilidades integrada en Security Center para Azure Virtual Machines](deploy-vulnerability-assessment-vm.md)
- [Identificación de vulnerabilidades en las imágenes de los registros de contenedor de Azure](defender-for-container-registries-usage.md#identify-vulnerabilities-in-images-in-other-container-registries)



## <a name="next-steps"></a>Pasos siguientes

En este artículo, aprendió las ventajas de Azure Defender. 

> [!div class="nextstepaction"]
> [Habilitación de Azure Defender](enable-azure-defender.md)