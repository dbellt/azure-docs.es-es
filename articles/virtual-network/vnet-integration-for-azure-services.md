---
title: Integración de red virtual de servicios de Azure para el aislamiento de red
titlesuffix: Azure Virtual Network
description: En este artículo se describen los distintos métodos para integrar un servicio de Azure en una red virtual que le permite acceder de forma segura al servicio de Azure.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: NA
ms.topic: conceptual
ms.workload: infrastructure-services
ms.date: 12/01/2020
ms.author: kumud
ms.openlocfilehash: 007424969672167d7ca81b2130cda8e0a5da8000
ms.sourcegitcommit: 9ad20581c9fe2c35339acc34d74d0d9cb38eb9aa
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110539428"
---
# <a name="integrate-azure-services-with-virtual-networks-for-network-isolation"></a>Integración de servicios de Azure con redes virtuales para el aislamiento de red

La integración de Virtual Network (VNet) para un servicio de Azure le permite bloquear el acceso al servicio únicamente a la infraestructura de red virtual. La infraestructura de VNet también incluye redes virtuales del mismo nivel y redes locales.

La integración de VNet proporciona a los servicios de Azure las ventajas del aislamiento de red y puede realizarse mediante uno o varios de los métodos siguientes:
- [Implementación de instancias dedicadas del servicio en una red virtual](virtual-network-for-azure-services.md). Luego se puede acceder de forma privada a los servicios dentro de la red virtual y desde redes locales.
- Usando un [punto de conexión privado](../private-link/private-endpoint-overview.md) que le conecta de forma privada y segura a un servicio con la tecnología de [Azure Private Link](../private-link/private-link-overview.md). Un punto de conexión privado usa una dirección IP privada de su VNet y coloca el servicio de manera eficaz en la red virtual.
- Accediendo al servicio mediante puntos de conexión públicos mediante la ampliación de una red virtual al servicio a través de los [puntos de conexión de servicio](virtual-network-service-endpoints-overview.md). Los puntos de conexión de servicio permiten que los recursos de servicio se protejan en la red virtual.
- Usando [etiquetas de servicio](service-tags-overview.md) para permitir o denegar el tráfico a los recursos de Azure hacia y desde puntos de conexión de dirección IP pública.

## <a name="deploy-dedicated-azure-services-into-virtual-networks"></a>Implementación de servicios de Azure dedicados en las redes virtuales

Al implementar servicios de Azure dedicados en una red virtual, puede comunicarse con los recursos de los servicios de forma privada mediante direcciones IP privadas.

![Implementación de servicios de Azure dedicados en las redes virtuales](./media/virtual-network-for-azure-services/deploy-service-into-vnet.png)

La implementación de un servicio de Azure dedicado en una red virtual ofrece las siguientes funcionalidades:
- Los recursos dentro de la red virtual pueden comunicarse entre sí de forma privada a través de direcciones IP privadas. Ejemplo: transferencia directa de datos entre HDInsight y SQL Server que se ejecutan en una máquina virtual, en la red virtual.
- Los recursos locales pueden acceder a recursos de una red virtual mediante direcciones IP privadas a través de VPN de sitio a sitio (VPN Gateway) o ExpressRoute.
- Las redes virtuales pueden estar emparejadas para habilitar que los recursos de las redes virtuales se comuniquen entre sí mediante direcciones IP privadas.
- Normalmente, las instancias de servicio en una red virtual están totalmente administradas por el servicio de Azure. Esta administración incluye la supervisión del estado de los recursos y el escalado con carga.
- Las instancias de los servicios se implementan en una subred de una red virtual. El acceso de red entrante y saliente para la subred se debe abrir a través de grupos de seguridad de red, siguiendo las instrucciones proporcionadas por el servicio.
- Determinados servicios imponen restricciones en la subred en la que están implementados. Estas restricciones limitan la aplicación de directivas, rutas o la combinación de máquinas virtuales y recursos de servicio dentro de la misma subred. Compruebe las restricciones específicas de cada servicio porque podrían cambiar con el tiempo. Algunos ejemplos de estos servicios son Azure NetApp Files, Dedicated HSM, Azure Container Instances y App Service.
- Opcionalmente, los servicios pueden requerir una subred delegada como un identificador explícito de que una subred puede hospedar un servicio determinado. Mediante la delegación, los servicios obtienen permisos explícitos para crear recursos específicos del servicio en la subred delegada.
- Vea un ejemplo de una respuesta de API REST en una red virtual con una subred delegada. Puede ver una lista completa de los servicios que usan el modelo de subred delegada a través de la API de Available Delegations.

Para obtener una lista de los servicios que se pueden implementar en una red virtual, consulte [Implementación de servicios de Azure dedicados en las redes virtuales](virtual-network-for-azure-services.md).

## <a name="private-link-and-private-endpoints"></a>Private Link con puntos de conexión privados

Los puntos de conexión privados permiten la entrada de tráfico desde la red virtual a un recurso de Azure de forma segura. Este vínculo privado se establece sin que sean necesarias las direcciones IP públicas. Un punto de conexión privado es una interfaz de red especial para un servicio de Azure en la red virtual. Cuando se crea un punto de conexión privado para el recurso, dicho punto de conexión privado proporciona conectividad segura entre los clientes de la red virtual y el recurso de Azure. Al punto de conexión privado se le asigna una dirección IP del intervalo de direcciones IP de su red virtual. La conexión entre el punto de conexión privado y el servicio de Azure es un vínculo privado.

En el diagrama, el lado derecho muestra una instancia de Azure SQL Database como servicio PaaS de destino. El destino puede ser [cualquier servicio que admita puntos de conexión privados](../private-link/availability.md). Hay varias instancias de SQL Server lógico para varios clientes, a las que se puede acceder a través de direcciones IP públicas.

En este caso, está expuesta una instancia de SQL Server lógico con un punto de conexión privado. El punto de conexión hace que SQL Server sea accesible a través de una dirección IP privada en la red virtual del cliente. Debido al cambio en la configuración de DNS, la aplicación cliente ahora envía su tráfico directamente a ese punto de conexión privado. El servicio de destino verá el tráfico que se origine desde una dirección IP privada de la red virtual. 

El vínculo privado está representado por la flecha verde. Todavía puede _existir_ una dirección IP pública para el recurso de destino junto con el punto de conexión privado. La aplicación cliente ya no usa la dirección IP pública. El firewall ahora puede no permitir el acceso a esa dirección IP pública, de modo que _solo_ sea accesible a través de puntos de conexión privados. Las conexiones a SQL Server sin un punto de conexión privado desde la red virtual se originarán desde una dirección IP pública. Este flujo está representado por la flecha azul.

![Puntos de conexión privados](./media/network-isolation/architecture-private-endpoints.png)

Normalmente, la aplicación cliente usa un nombre de host DNS para comunicarse con el servicio de destino. No se necesita ningún cambio en la aplicación. La [resolución DNS de la red virtual debe configurarse](../private-link/private-endpoint-dns.md) para resolver ese mismo nombre de host en la dirección IP privada del recurso de destino, en lugar de en la dirección IP pública original. Con una ruta de acceso privada entre el cliente y el servicio de destino, el cliente no depende de la dirección IP pública. El servicio de destino puede desactivar el acceso público.

La exposición de instancias individuales permite [evitar el robo de datos](../private-link/private-endpoint-overview.md#network-security-of-private-endpoints). Un actor malintencionado no podrá recopilar información de la base de datos y cargarla en otra base de datos pública o cuenta de almacenamiento. Puede impedir el acceso a las direcciones IP públicas de _todos_ los servicios PaaS. Todavía puede permitir el acceso a instancias de PaaS a través de sus puntos de conexión privados.

Para obtener más información sobre el vínculo privado y una lista de los servicios de Azure admitidos, consulte [¿Qué es Azure Private Link?](../private-link/private-link-overview.md)

## <a name="service-endpoints"></a>Puntos de conexión del servicio

Los puntos de conexión de servicio proporcionan conectividad segura y directa a los servicios de Azure a través de la red troncal de Azure. Los puntos de conexión permiten proteger los recursos de Azure únicamente para las redes virtuales. Los puntos de conexión de servicio permiten que las direcciones IP privadas de la red virtual se comuniquen con un servicio de Azure sin necesidad de una dirección IP pública saliente.

Sin los puntos de conexión de servicio, restringir el acceso solo a la red virtual puede ser complicado. La dirección IP de origen podría cambiar o podría compartirse con otros clientes. Por ejemplo, los servicios PaaS con direcciones IP salientes compartidas. Con los puntos de conexión de servicio, la dirección IP de origen que ve el servicio de destino se convierte en una dirección IP privada de la red virtual. Este cambio de tráfico de entrada permite identificar fácilmente el origen y usarlo para configurar las reglas de firewall adecuadas. Por ejemplo, para permitir solo el tráfico desde una subred específica dentro de esa red virtual.

Con los puntos de conexión de servicio, las entradas DNS para los servicios de Azure permanecen tal cual y siguen resolviendo las direcciones IP públicas asignadas al servicio de Azure.

En el diagrama siguiente, el lado derecho es el mismo servicio PaaS de destino. A la izquierda, está la red virtual de un cliente con dos subredes: la subred A, que tiene un punto de conexión de servicio hacia `Microsoft.Sql`, y la subred B, que no tiene definido ningún punto de conexión de servicio. 

Cuando un recurso de la subred B intente comunicarse con cualquier instancia de SQL Server, usará una dirección IP pública para la comunicación saliente. Este tráfico está representado por la flecha azul. El firewall de SQL Server debe usar esa dirección IP pública para permitir o bloquear el tráfico de red. 

Cuando un recurso de la subred A intente llegar a un servidor de bases de datos, se verá como una dirección IP privada desde dentro de la red virtual. Este tráfico está representado por las flechas verdes. El firewall de SQL Server ahora puede permitir o bloquear específicamente la subred A. No es necesario conocer la dirección IP pública del servicio de origen.

![Puntos de conexión de servicio](./media/network-isolation/architecture-service-endpoints.png)

Los puntos de conexión de servicio se aplican a **todas** las instancias del servicio de destino. Por ejemplo, **todas** las instancias de SQL Server de los clientes de Azure, no solo la instancia del cliente.

Para obtener más información, vea [Puntos de conexión de servicio de red virtual](virtual-network-service-endpoints-overview.md).

## <a name="service-tags"></a>Etiquetas de servicio

Una etiqueta de servicio representa un grupo de prefijos de direcciones IP de un servicio de Azure determinado. Con las etiquetas de servicio, puede definir controles de acceso a la red en [grupos de seguridad de red](./network-security-groups-overview.md#security-rules) o [Azure Firewall](../firewall/service-tags.md). Puede permitir o denegar el tráfico del servicio. Para permitir o denegar el tráfico, especifique la etiqueta de servicio en el campo de origen o destino de una regla. 

![Permitir o denegar el tráfico mediante etiquetas de servicio](./media/network-isolation/service-tags.png)

Logre el aislamiento de red y proteja los recursos de Azure de la red de Internet mientras accede a los servicios de Azure que tienen puntos de conexión públicos. Cree reglas de grupo de seguridad de red entrantes y salientes para denegar el tráfico hacia y desde **Internet** y permitir el tráfico hacia y desde **AzureCloud**. Para más etiquetas de servicio, consulte las [etiquetas de servicio disponibles](service-tags-overview.md#available-service-tags) de los servicios específicos de Azure.

Para más información sobre las etiquetas de servicio y los servicios de Azure compatibles con ellas, consulte [Etiquetas de servicio de red virtual](service-tags-overview.md).

## <a name="compare-private-endpoints-and-service-endpoints"></a>Comparación de puntos de conexión privados y puntos de conexión de servicio

En lugar de mirar solo sus diferencias, vale la pena señalar que tanto los puntos de conexión de servicio como los puntos de conexión privados tienen características en común.

Ambas características se usan para lograr control más detallado del firewall en el servicio de destino. Por ejemplo, restringir el acceso a las bases de datos o cuentas de almacenamiento de SQL Server. Sin embargo, cada uno se opera de manera diferente, como se describe con más detalle en las secciones anteriores.

Ambos enfoques superan el problema del [agotamiento de puertos de traducción de direcciones de red de origen (SNAT)](../load-balancer/load-balancer-outbound-connections.md#scenarios). Puede que el agotamiento se presente al realizar la tunelización del tráfico a través de una aplicación virtual de red (NVA) o un servicio con limitaciones de puerto SNAT. Cuando se usan puntos de conexión de servicio o puntos de conexión privados, el tráfico toma una ruta de acceso optimizada directamente al servicio de destino. Ambos enfoques pueden beneficiar a las aplicaciones que consumen mucho ancho de banda, ya que se reducen la latencia y el costo.

En ambos casos, todavía puede garantizar que el tráfico que entra en el servicio de destino pasará a través de un firewall de red o NVA. Este procedimiento es diferente para ambos enfoques. Cuando use puntos de conexión de servicio, debe configurar el punto de conexión de servicio en la subred de **firewall**, en lugar de la subred en la que está implementado el servicio de origen. Cuando se usan puntos de conexión privados, se coloca una ruta definida por el usuario (UDR) para la dirección IP del punto de conexión privado en la subred de **origen**. No en la subred del punto de conexión privado.

| Consideración                                                                                                                                    | Puntos de conexión de servicio                                                                                                           | Puntos de conexión privados                                                                                                                                                 |
| ------------------------------------------------------------------------------------------------------------------------------------------------ | --------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Ámbito en el que se aplica la configuración                                                                                                   | Servicio completo (por ejemplo, _todas_ las instancias de SQL Server o las cuentas de almacenamiento de _todos_ los clientes)                                      | Instancia individual (por ejemplo, una instancia de SQL Server o una cuenta de almacenamiento de _su_ propiedad)                                                                    |
| El tráfico de Azure a Azure permanece en la red troncal de Azure                                                                                       | Sí                                                                                                                         | Sí                                                                                                                                                               |
| El servicio puede deshabilitar su dirección IP pública                                                                                                        | No                                                                                                                          | Sí                                                                                                                                                               |
| Se puede acceder al servicio sin usar ninguna dirección IP pública                                                                                       | No                                                                                                                          | Sí                                                                                                                                                               |
| Puede restringir fácilmente el tráfico procedente de una instancia de Azure Virtual Network                                                                             | Sí (permita el acceso desde subredes específicas o use NSG)                                                                   | No*                                                                                                                                                               |
| Puede restringir fácilmente el tráfico procedente de un servidor local (VPN/ExpressRoute)                                                                           | N/D**                                                                                                                       | No*                                                                                                                                                               |
| Requiere cambios de DNS                                                                                                                             | No                                                                                                                          | Sí (consulte la [configuración de DNS](../private-link/private-endpoint-dns.md))                                                                 |
| Afecta el costo de la solución                                                                                                                | No                                                                                                                          | Sí (consulte los [precios del vínculo privado](https://azure.microsoft.com/pricing/details/private-link/))                                                                       |
| Afecta al [Acuerdo de Nivel de Servicio compuesto](/azure/architecture/framework/resiliency/business-metrics#composite-slas) de la solución | No                                                                                                                          | Sí (el propio servicio del vínculo privado tiene un [Acuerdo de Nivel de Servicio del 99,99 %](https://azure.microsoft.com/support/legal/sla/private-link/))                                                 |

*Todo lo que tenga una línea de visión de red en el punto de conexión privado tendrá acceso de nivel de red. Este acceso no se puede controlar mediante un NSG en el propio punto de conexión privado.

**No se puede acceder desde redes locales a los recursos de los servicios de Azure protegidos en las redes virtuales. Si quiere permitir el tráfico desde el entorno local, permita las direcciones IP públicas (normalmente NAT) desde el entorno local o ExpressRoute. Estas direcciones IP se pueden agregar a través de la configuración del firewall de IP para los recursos de los servicios de Azure. Para más información, consulte las [preguntas más frecuentes de la red virtual](virtual-networks-faq.md#can-an-on-premises-devices-ip-address-that-is-connected-through-azure-virtual-network-gateway-vpn-or-expressroute-gateway-access-azure-paas-service-over-vnet-service-endpoints).

## <a name="next-steps"></a>Pasos siguientes

- Obtenga información sobre cómo [integrar la aplicación con una red de Azure](../app-service/web-sites-integrate-with-vnet.md).
- Obtenga información sobre cómo [restringir el acceso a los recursos mediante etiquetas de servicio](tutorial-restrict-network-access-to-resources.md).
- Obtenga información sobre cómo [conectarse de forma privada a una cuenta de Azure Cosmos mediante Azure Private Link](../private-link/tutorial-private-endpoint-cosmosdb-portal.md).