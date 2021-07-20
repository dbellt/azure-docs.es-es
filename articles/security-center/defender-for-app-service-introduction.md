---
title: 'Azure Defender para App Service: ventajas y características'
description: Obtenga información sobre las funcionalidades de Azure Defender para App Service y cómo habilitarlo en una suscripción.
author: memildin
ms.author: memildin
ms.date: 01/25/2021
ms.topic: overview
ms.service: security-center
manager: rkarlin
ms.openlocfilehash: 75d9c649b245878e1f10c1aca9b9dd9956741fca
ms.sourcegitcommit: a038863c0a99dfda16133bcb08b172b6b4c86db8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/29/2021
ms.locfileid: "113002640"
---
# <a name="protect-your-web-apps-and-apis"></a>Protección de las API y las aplicaciones web

## <a name="prerequisites"></a>Prerrequisitos

Security Center está integrado de forma nativa con App Service, lo que elimina la necesidad de implementación e incorporación; la integración es transparente.

Para proteger el plan de Azure App Service con Azure Defender para App Service, necesitará lo siguiente:

- Un plan de App Service admitido asociado a máquinas dedicadas. Los planes admitidos se enumeran en [Disponibilidad](#availability).

- Azure Defender habilitado en la suscripción, tal como se describe en [Inicio rápido: Habilitación de Azure Defender](enable-azure-defender.md).

    > [!TIP]
    > Opcionalmente, puede habilitar planes individuales en Azure Defender (como Azure Defender para App Service).

## <a name="availability"></a>Disponibilidad

| Aspecto                       | Detalles                                                                                                                                                                                        |
|------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Estado de la versión:               | Disponibilidad general (GA)                                                                                                                                                                      |
| Precios:                     | [Azure Defender para App Service](azure-defender.md) se factura como se indica en [Precios de Security Center](https://azure.microsoft.com/pricing/details/security-center/).<br>La facturación se realiza en función de las instancias de proceso totales en todos los planes.       |
| Planes de App Service admitidos: | Se admiten [todos los planes de App Service](https://azure.microsoft.com/pricing/details/app-service/plans/) excepto [Azure Functions en el plan de consumo](../azure-functions/functions-scale.md). |
| Nubes:                      | ![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![No](./media/icons/no-icon.png) Nacionales o soberanas (US Gov, Azure China)                                                     |
|                              |                                                                                                                                                                                                |

## <a name="what-are-the-benefits-of-azure-defender-for-app-service"></a>¿Cuáles son las ventajas de Azure Defender para App Service?

Azure App Service es una plataforma totalmente administrada para crear y hospedar aplicaciones web y API. Al ser totalmente administrada, no hay que preocuparse por la infraestructura. Proporciona administración, supervisión y conclusiones operativas para satisfacer requisitos de seguridad, cumplimiento normativo y rendimiento de nivel empresarial. Para más información, consulte [Azure App Service](https://azure.microsoft.com/services/app-service/).

**Azure Defender para App Service** usa la escala de la nube para identificar ataques dirigidos a aplicaciones que se ejecutan mediante App Service. Los atacantes sondean las aplicaciones web para buscar y aprovechar los puntos débiles. Antes de enrutarse a entornos específicos, las solicitudes para las aplicaciones que se ejecutan en Azure atraviesan varias puertas de enlace donde se las inspecciona y registra. A continuación, estos datos se usan para identificar vulnerabilidades y atacantes, así como para aprender nuevos patrones que se usarán más adelante.

Al habilitar Azure Defender para App Service, se beneficia inmediatamente de los siguientes servicios que ofrece este plan de Azure Defender:

- **Protección**: Security Center evalúa los recursos incluidos en el plan de App Service y genera recomendaciones de seguridad en función de lo que halle. Siga las instrucciones detalladas de estas recomendaciones para proteger los recursos de App Service.

- **Detección**: Azure Defender detecta un elevado número de amenazas para los recursos de App Service, ya que supervisa:
    - la instancia de la máquina virtual en la que se ejecuta App Service y su interfaz de administración.
    - las solicitudes y respuestas tanto que se envían a las aplicaciones de App Service, como que se envían desde ellas
    - los espacios aislados y las máquinas virtuales subyacentes
    - los registros internos de App Service (disponibles gracias a la visibilidad que tiene Azure como proveedor de nube)

Como solución nativa en la nube, Azure Defender puede identificar metodologías de ataque que se aplican a varios destinos. Por ejemplo, desde un único host sería difícil identificar un ataque distribuido desde un pequeño subconjunto de direcciones IP que rastrean puntos de conexión similares en varios hosts.

Conjuntamente, los datos del registro y la infraestructura puede indicar lo que ocurre: desde un nuevo ataque que prolifera en la red hasta riesgos concretos en las máquinas de los clientes. Por lo tanto, incluso si se implementa Security Center una vez que se han aprovechado las vulnerabilidades de una aplicación web, es posible que pueda detectar los ataques en curso.


## <a name="what-threats-can-azure-defender-for-app-service-detect"></a>¿Qué amenazas puede detectar Azure Defender para App Service?

### <a name="threats-by-mitre-attck-tactics"></a>Amenazas por tácticas de MITRE ATT&CK

Azure Defender supervisa muchas amenazas para los recursos de App Service. Las alertas abarcan casi la lista completa de tácticas de MITRE ATT&CK, desde los ataques previos al comando y el control. Azure Defender puede detectar:

- **Amenazas previas a ataques**: Defender puede detectar la ejecución de varios tipos de detectores de vulnerabilidades que los atacantes usan con frecuencia para sondear las aplicaciones en busca de puntos débiles.

- **Amenazas en el acceso inicial** - [Microsoft Threat Intelligence](https://go.microsoft.com/fwlink/?linkid=2128684) alimenta estas alertas que incluyen el desencadenamiento de una alerta cuando una dirección IP malintencionada conocida se conecta a la interfaz de FTP de Azure App Service.

- **Amenazas a la ejecución**: Defender puede detectar intentos de ejecución de comandos con privilegios elevados, comandos de Linux en Windows App Service, comportamiento de ataque sin archivos, herramientas de minería de moneda digital y muchas otras actividades de ejecución de código sospechosas y malintencionadas.

### <a name="dangling-dns-detection"></a>Detección de DNS pendiente

Azure Defender para App Service también identifica las entradas de DNS que queden en el registrador de DNS cuando se retira un sitio web de App Service (se conocen como entradas DNS pendientes). Cuando se quita un sitio web, pero no se quita su dominio personalizado del registrador de DNS, la entrada DNS apunta a un recurso que no existe y el subdominio es vulnerable a una adquisición. Azure Defender no examina el registrador de DNS en busca de entradas de DNS pendientes *existentes*, sino que le avisa cuando se retira un sitio web de App Service y su dominio personalizado (entrada DNS) no se elimina.

Las adquisiciones de subdominios son una amenaza muy grave que se producen de forma común en las organizaciones. Cuando un actor de amenazas detecta una entrada DNS pendiente, crea su propio sitio en la dirección de destino. A continuación, el tráfico dirigido al dominio de la organización se dirige al sitio del actor de amenazas, que este puede usar ese tráfico para una amplia gama de actividades malintencionadas.

La protección contra DNS pendiente está disponibles tanto si los dominios se administran con Azure DNS como con un registrador de dominios externo y se aplica a App Service en Windows y en Linux.

:::image type="content" source="media/defender-for-app-service-introduction/dangling-dns-alert.png" alt-text="Un ejemplo de una alerta de Azure Defender por la detección de una entrada de DNS pendiente. Habilite Azure Defender para App Service para recibir esta y otras alertas en su entorno." lightbox="media/defender-for-app-service-introduction/dangling-dns-alert.png":::

Para más información sobre DNS pendientes y la amenaza de adquisición de subdominios, consulte [Evitar las entradas DNS pendientes y evitar la adquisición de subdominios](../security/fundamentals/subdomain-takeover.md).

Para obtener una lista completa de las alertas de Azure App Service, consulte la [Tabla de referencia de alertas](alerts-reference.md#alerts-azureappserv).

> [!NOTE]
> Es posible que Defender no desencadene alertas de DNS pendientes si el dominio personalizado no apunta directamente a un recurso de App Service o si Defender no ha supervisado el tráfico a su sitio web desde que se habilitó la protección contra DNS pendiente (porque no habrá registros que ayuden a identificar el dominio personalizado).

## <a name="next-steps"></a>Pasos siguientes

En este artículo, ha obtenido información sobre Azure Defender para App Service. 

Para obtener material relacionado, consulte los siguientes artículos: 

- Para exportar las alertas a Azure Sentinel, la SIEM de terceros, o cualquier otra herramienta externa, siguen las instrucciones de [Transmisión de alertas a una solución de administración de servicios de TI, SIEM o SOAR](export-to-siem.md).
- Para obtener una lista de las alertas de Azure Defender para App Service, consulte la [Tabla de referencia de alertas](alerts-reference.md#alerts-azureappserv).
- Para más información sobre los planes de App Service, consulte [Planes de App Service](https://azure.microsoft.com/pricing/details/app-service/plans/).
> [!div class="nextstepaction"]
> [Habilitación de Azure Defender](enable-azure-defender.md)