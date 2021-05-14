---
ms.date: 02/16/2021
ms.topic: reference-architecture
author: kriation
title: Arquitectura de referencia de Azure Spring Cloud
ms.author: akaleshian
ms.service: spring-cloud
description: Esta arquitectura de referencia es una base que usa un diseño empresarial típico de concentrador y radio (hub-and-spoke) para el uso de Azure Spring Cloud.
ms.openlocfilehash: d45d70227be2d88ca69e8a88bbc116a7f8045e74
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108132690"
---
# <a name="azure-spring-cloud-reference-architecture"></a>Arquitectura de referencia de Azure Spring Cloud

Esta arquitectura de referencia es una base que usa un diseño empresarial típico de concentrador y radio (hub-and-spoke) para el uso de Azure Spring Cloud. En el diseño, Azure Spring Cloud se implementa en un único radio que depende de los servicios compartidos hospedados en el concentrador. La arquitectura se crea con componentes para lograr los principios del [Marco de buena arquitectura de Microsoft Azure][16].

Para obtener una implementación de esta arquitectura, consulte el repositorio de [Arquitectura de referencia de Azure Spring Cloud][10] en GitHub.

Entre las opciones de implementación para esta arquitectura se incluyen Azure Resource Manager (ARM), Terraform y la CLI de Azure. Los artefactos de este repositorio ofrecen una base que se puede personalizar para su entorno. Puede agrupar recursos, como Azure Firewall o Application Gateway, en diferentes grupos de recursos o suscripciones. Esta agrupación ayuda a mantener independientes diferentes funciones, como la infraestructura de TI, la seguridad, los equipos de aplicaciones empresariales, etc.

## <a name="planning-the-address-space"></a>Planeación del espacio de direcciones

Azure Spring Cloud requiere dos subredes dedicadas:

* Entorno de ejecución del servicio
* Aplicaciones de Spring Boot

Cada una de estas subredes requiere un clúster dedicado. Varios clústeres no pueden compartir las mismas subredes. El tamaño mínimo de cada subred es /28. El número de instancias de la aplicación que Azure Spring Cloud puede admitir varía en función del tamaño de la subred. Puede encontrar los requisitos detallados de la red virtual (Vnet) en la sección [Requisitos de red virtual][11] de [Implementación de Azure Spring Cloud en una red virtual][17].

> [!WARNING]
> El tamaño de subred seleccionado no puede superponerse con el espacio de direcciones de la red virtual existente, y no debe superponerse con ningún intervalo de direcciones de subred emparejada ni local.

## <a name="use-cases"></a>Casos de uso

Los usos habituales de esta arquitectura incluyen:

* Aplicaciones internas implementadas en entornos de nube híbrida
* Aplicaciones orientadas al exterior

Estos casos de uso son similares, excepto en lo que refiere a la seguridad y las reglas de tráfico de red. Esta arquitectura está diseñada para admitir los matices de cada uno.

## <a name="private-applications"></a>Aplicaciones privadas

En la lista siguiente se describen los requisitos de infraestructura para las aplicaciones privadas. Estos requisitos son habituales en entornos altamente regulados.

* No hay una salida directa a la red pública de Internet, excepto por el tráfico del plano de control.
* El tráfico de salida debe viajar a través de una aplicación virtual de red (NVA) central (por ejemplo, Azure Firewall).
* Los datos en reposo deben estar cifrados.
* Los datos en tránsito deben estar cifrados.
* Se pueden usar canalizaciones de implementación de DevOps (por ejemplo, Azure DevOps) y requerir conectividad de red a Azure Spring Cloud.
* El enfoque de seguridad de Confianza cero de Microsoft requiere que los secretos, los certificados y las credenciales se conserven en un almacén seguro. El servicio recomendado es Azure Key Vault.
* Los registros del servicio de nombres de dominio (DNS) del host de aplicación se deben almacenar en el DNS privado de Azure.
* La resolución de nombres de hosts en el entorno local y en la nube debe ser bidireccional.
* Debe cumplirse al menos una prueba comparativa de seguridad.
* Las dependencias de los servicios de Azure se deben comunicar a través de puntos de conexión de servicio o Private Link.
* Los grupos de recursos administrados por la implementación de Azure Spring Cloud no se deben modificar.
* Las subredes administradas por la implementación de Azure Spring Cloud no se deben modificar.
* Una subred solo debe tener una instancia de Azure Spring Cloud.
* Si se usa [Config Server de Azure Spring Cloud][8] para cargar las propiedades de configuración desde un repositorio, el repositorio debe ser privado.

En la lista siguiente se muestran los componentes que componen el diseño:

* Red local
  * Servicio de nombres de dominio (DNS)
  * Puerta de enlace
* Suscripción al centro
  * Subred de Azure Firewall
  * Subred de Application Gateway
  * Subred de servicios compartidos
* Suscripción conectada
  * Redes virtuales del mismo nivel
  * Subred de Azure Bastion

En la lista siguiente se describen los servicios de Azure en esta arquitectura de referencia:

* [Azure Spring Cloud][1]: Servicio administrado diseñado y optimizado específicamente para aplicaciones de Spring Boot basadas en Java y aplicaciones de [Steeltoe][9] basadas en .NET.

* [Azure Key Vault][2]: Servicio de administración de credenciales con respaldo de hardware, que tiene una estrecha integración con los servicios de identidad de Microsoft y los recursos de proceso.

* [Azure Monitor][3]: Conjunto de servicios de supervisión global para aplicaciones que se implementan en Azure y en el entorno local.

* [Azure Security Center][4]: Sistema unificado de protección contra amenazas y administración de seguridad para cargas de trabajo en entornos locales, varias nubes y Azure.

* [Azure Pipelines][5]: Servicio completo de integración continua y desarrollo continuo (CI/CD) que puede implementar automáticamente aplicaciones de Spring Boot actualizadas en Azure Spring Cloud.

El siguiente diagrama representa un diseño de concentrador y radio bien diseñado que aborda los requisitos anteriores:

![Diagrama de arquitectura de referencia para aplicaciones privadas](./media/spring-cloud-reference-architecture/architecture-private.png)

## <a name="public-applications"></a>Aplicaciones públicas

En la lista siguiente se describen los requisitos de infraestructura para las aplicaciones públicas. Estos requisitos son habituales en entornos altamente regulados.

* El tráfico de entrada debe estar administrado por al menos Application Gateway o Azure Front Door.
* Debe estar habilitada la versión Estándar de Azure DDoS Protection.
* No hay una salida directa a la red pública de Internet, excepto por el tráfico del plano de control.
* El tráfico de salida debe atravesar una aplicación virtual de red (NVA) central (por ejemplo, Azure Firewall).
* Los datos en reposo deben estar cifrados.
* Los datos en tránsito deben estar cifrados.
* Se pueden usar canalizaciones de implementación de DevOps (por ejemplo, Azure DevOps) y requerir conectividad de red a Azure Spring Cloud.
* El enfoque de seguridad de Confianza cero de Microsoft requiere que los secretos, los certificados y las credenciales se conserven en un almacén seguro. El servicio recomendado es Azure Key Vault.
* Los registros del DNS del host de aplicación se deben almacenar en el DNS privado de Azure.
* Las direcciones enrutables a Internet deben almacenarse en el DNS público de Azure.
* La resolución de nombres de hosts en el entorno local y en la nube debe ser bidireccional.
* Debe cumplirse al menos una prueba comparativa de seguridad.
* Las dependencias de los servicios de Azure se deben comunicar a través de puntos de conexión de servicio o Private Link.
* Los grupos de recursos administrados por la implementación de Azure Spring Cloud no se deben modificar.
* Las subredes administradas por la implementación de Azure Spring Cloud no se deben modificar.
* Una subred solo debe tener una instancia de Azure Spring Cloud.

En la lista siguiente se muestran los componentes que componen el diseño:

* Red local
  * Servicio de nombres de dominio (DNS)
  * Puerta de enlace
* Suscripción al centro
  * Subred de Azure Firewall
  * Subred de Application Gateway
  * Subred de servicios compartidos
* Suscripción conectada
  * Redes virtuales del mismo nivel
  * Subred de Azure Bastion

En la lista siguiente se describen los servicios de Azure en esta arquitectura de referencia:

* [Azure Spring Cloud][1]: Servicio administrado diseñado y optimizado específicamente para aplicaciones de Spring Boot basadas en Java y aplicaciones de [Steeltoe][9] basadas en .NET.

* [Azure Key Vault][2]: Servicio de administración de credenciales con respaldo de hardware, que tiene una estrecha integración con los servicios de identidad de Microsoft y los recursos de proceso.

* [Azure Monitor][3]: Conjunto de servicios de supervisión global para aplicaciones que se implementan en Azure y en el entorno local.

* [Azure Security Center][4]: Sistema unificado de protección contra amenazas y administración de seguridad para cargas de trabajo en entornos locales, varias nubes y Azure.

* [Azure Pipelines][5]: Servicio completo de integración continua y desarrollo continuo (CI/CD) que puede implementar automáticamente aplicaciones de Spring Boot actualizadas en Azure Spring Cloud.

* [Azure Application Gateway][6]: Equilibrador de carga responsable del tráfico de aplicaciones con la descarga de Seguridad de la capa de transporte (TLS) en el nivel 7.

* [Azure Application Firewall][7]: Característica de Azure Application Gateway que proporciona a las aplicaciones una protección centralizada contra vulnerabilidades de seguridad comunes.

El siguiente diagrama representa un diseño de concentrador y radio bien diseñado que aborda los requisitos anteriores:

![Diagrama de arquitectura de referencia para aplicaciones públicas](./media/spring-cloud-reference-architecture/architecture-public.png)

## <a name="azure-spring-cloud-on-premises-connectivity"></a>Conectividad local de Azure Spring Cloud

Las aplicaciones que se ejecutan en Azure Spring Cloud pueden comunicarse con varios recursos locales, externos y de Azure. Mediante el diseño de concentrador y radio, las aplicaciones pueden enrutar el tráfico externamente o a la red local mediante ExpressRoute o una red privada virtual (VPN) de sitio a sitio.

## <a name="azure-well-architected-framework-considerations"></a>Consideraciones sobre el Marco de buena arquitectura de Azure

El [Marco de buena arquitectura de Azure][16] es un conjunto de principios rectores que se deben seguir para establecer una base sólida para la infraestructura. El marco consta de las siguientes categorías: optimización de costos, excelencia operativa, eficiencia de rendimiento, confiabilidad y seguridad.

### <a name="cost-optimization"></a>Optimización de costos

Debido a la naturaleza del diseño como sistema distribuido, la expansión de la infraestructura es una realidad. Esta realidad genera costos inesperados e incontrolables. Azure Spring Cloud se ha creado con componentes que se escalan para poder satisfacer la demanda y optimizar el costo. El centro de esta arquitectura es Azure Kubernetes Service (AKS). El servicio está diseñado para reducir la complejidad y la sobrecarga operativa de la administración de Kubernetes, lo que incluye las eficiencias en el costo operativo del clúster.

Puede implementar diferentes aplicaciones y tipos de aplicación en una sola instancia de Azure Spring Cloud. El servicio admite el escalado automático de aplicaciones desencadenado por métricas o programaciones que pueden mejorar el uso y la rentabilidad.

También puede usar Application Insights y Azure Monitor para reducir los costos operativos. Con la visibilidad proporcionada por la solución de registro completa, puede implementar la automatización para escalar los componentes del sistema en tiempo real. También puede analizar los datos de registro para revelar ineficiencias en el código de la aplicación, que se pueden abordar para mejorar el costo y el rendimiento generales del sistema.

### <a name="operational-excellence"></a>Excelencia operativa

Azure Spring Cloud aborda varios aspectos de la excelencia operativa. Puede combinar estos aspectos para asegurarse de que el servicio se ejecute de manera eficaz en entornos de producción, como se describe en la lista siguiente:

* Puede usar Azure Pipelines para asegurarse de que las implementaciones sean confiables y coherentes, al tiempo que le ayuda a evitar errores humanos.
* Puede usar Azure Monitor y Application Insights para almacenar datos de registro y de telemetría.
  Puede evaluar los datos recopilados de registros y de métricas para garantizar el mantenimiento y el rendimiento de las aplicaciones. La Supervisión de rendimiento de aplicaciones (APM) se encuentra totalmente integrada en el servicio a través de un agente de Java. Este agente proporciona visibilidad en todas las aplicaciones y dependencias implementadas sin necesidad de código adicional. Para obtener más información, consulte la entrada de blog sobre [Supervisión sencilla de aplicaciones y dependencias en Azure Spring Cloud][15].
* Puede usar Azure Security Center para asegurarse de que las aplicaciones mantengan la seguridad; para ello puede brindar una plataforma para analizar y evaluar los datos proporcionados.
* El servicio admite varios patrones de implementación. Para más información, consulte [Configuración de un entorno de ensayo en Azure Spring Cloud][14].

### <a name="reliability"></a>Confiabilidad

Azure Spring Cloud está diseñado con AKS como componente fundamental. Aunque AKS proporciona un nivel de resistencia a través de la agrupación en clústeres, esta arquitectura de referencia incorpora servicios y consideraciones arquitectónicas para aumentar la disponibilidad de la aplicación si se produce un error en el componente.

Al basarse en un diseño de concentrador y radio bien definido, la base de esta arquitectura garantiza que se puede implementar en varias regiones. En el caso de uso de las aplicaciones privadas, la arquitectura utiliza DNS privado de Azure para garantizar la disponibilidad continuada durante un error geográfico. Para el caso de uso de las aplicaciones públicas, Azure Front Door y Azure Application Gateway garantizan la disponibilidad.

### <a name="security"></a>Seguridad

La seguridad de esta arquitectura se aborda por su cumplimiento de los controles y pruebas comparativas definidos por el sector. Los controles de esta arquitectura provienen de [Cloud Controls Matrix][19] (CCM) de [Cloud Security Alliance][18] (CSA) y [Microsoft Azure Foundations Benchmark][20] (MAFB) de [Center for Internet Security][21] (CIS). En los controles aplicados, el enfoque se centra en los principios de diseño de seguridad principales relativos a la gobernanza, las conexiones de red y seguridad de las aplicaciones. Es su responsabilidad administrar los principios de diseño de identidad, administración de acceso y almacenamiento, en la medida en que se relacionen con su infraestructura de destino.

#### <a name="governance"></a>Gobernanza

El aspecto principal de la gobernanza que aborda esta arquitectura es la segregación a través del aislamiento de los recursos de red. En CCM, DCS-08 recomienda el control de entrada y salida para el centro de datos. Para satisfacer el control, la arquitectura usa un diseño de concentrador y radio con grupos de seguridad de red (NSG) para filtrar el tráfico de Este a Oeste entre los recursos. La arquitectura también filtra el tráfico entre los servicios centrales en el concentrador y los recursos en el radio. La arquitectura usa una instancia de Azure Firewall para administrar el tráfico entre Internet y los recursos de la arquitectura.

En la siguiente lista se muestra el control que aborda la seguridad del centro de datos en esta referencia:

| Id. de control de CCM de CSA | Dominio de control de CCM de CSA |
| :----------------- | :----------------------|
| DCS-08 | Seguridad del centro de datos: entrada de personas no autorizadas |

#### <a name="network"></a>Red

El diseño de red que admite esta arquitectura se obtiene del modelo tradicional de concentrador y radio. Esta decisión garantiza que el aislamiento de red sea una construcción fundamental. El control IV-06 de CCM recomienda que el tráfico entre las redes y las máquinas virtuales esté restringido y supervisado entre los entornos de confianza y los que no son de confianza. Esta arquitectura adopta el control mediante la implementación de los NSG para el tráfico de Este a Oeste, y Azure Firewall para el tráfico de Norte a Sur. El control IPY-04 de CCM recomienda que la infraestructura use protocolos de red seguros para el intercambio de datos entre servicios. Todos los servicios de Azure que admiten esta arquitectura usan protocolos seguros estándar, como TLS para HTTP y SQL.

En la siguiente lista se muestran los controles de CCM que abordan la seguridad de red en esta referencia:

| Id. de control de CCM de CSA | Dominio de control de CCM de CSA |
| :----------------- | :----------------------|
| IPY-04             | Protocolos de red      |
| IV-06             | Seguridad de redes       |

Además, la implementación de red se protege mediante la definición de controles de MAFB. Los controles garantizan que el tráfico en el entorno esté restringido desde la red pública de Internet.

En la siguiente lista se muestran los controles de CIS que abordan la seguridad de red en esta referencia:

| Id. de control de CIS | Descripción del control de CIS |
| :------------- | :---------------------- |
| 6.2 | Asegúrese de que el acceso SSH está restringido desde Internet. |
| 6.3 | Asegúrese de que ninguna instancia de SQL Database permita la entrada 0.0.0.0/0 (cualquier IP). |
| 6.5 | Asegúrese de que Network Watcher esté "Habilitado". |
| 6.6 | Asegúrese de que la entrada mediante UDP esté restringida desde Internet. |

Azure Spring Cloud requiere tráfico de administración para la salida de Azure cuando se implementa en un entorno seguro. Para ello, debe permitir las reglas de red y de aplicación enumeradas en [Responsabilidades del cliente para ejecutar Azure Spring Cloud en una red virtual](./vnet-customer-responsibilities.md).

#### <a name="application-security"></a>Seguridad de la aplicación

Este principio de diseño cubre los componentes fundamentales de identidad, protección de datos, administración de claves y configuración de aplicaciones. Por diseño, una aplicación implementada en Azure Spring Cloud se ejecuta con los privilegios mínimos necesarios para funcionar. El conjunto de controles de autorización está directamente relacionado con la protección de datos cuando se usa el servicio. La administración de claves fortalece este enfoque de seguridad de aplicaciones por capas.

En la siguiente lista se muestran los controles de CCM que abordan la administración de claves en esta referencia:

| Id. de control de CCM de CSA | Dominio de control de CCM de CSA |
| :----------------- | :--------------------- |
| EKM-01 | Cifrado y administración de claves: derecho |
| EKM-02 | Cifrado y administración de claves: generación de claves |
| EKM-03 | Cifrado y administración de claves: protección de datos confidenciales |
| EKM-04 | Cifrado y administración de claves: almacenamiento y acceso |

A partir de CCM, EKM-02 y EKM-03 recomiendan las directivas y los procedimientos para administrar las claves y para usar los protocolos de cifrado para proteger los datos confidenciales. EKM-01 recomienda que todas las claves criptográficas tengan propietarios identificables para que puedan administrarse. EKM-04 recomienda el uso de algoritmos estándar.

En la siguiente lista se muestran los controles de CIS que abordan la administración de claves en esta referencia:

| Id. de control de CIS | Descripción del control de CIS |
| :------------- | :---------------------- |
| 8.1 | Asegúrese de que la fecha de expiración se haya establecido en todas las claves. |
| 8,2 | Asegúrese de que la fecha de expiración se haya establecido en todos los secretos. |
| 8,4 | Asegúrese de que el almacén de claves se pueda recuperar. |

Los controles 8.1 y 8.2 de CIS recomiendan que se establezcan fechas de expiración para las credenciales con el fin de garantizar que se aplique la rotación. El control 8.4 de CIS garantiza que el contenido del almacén de claves se pueda restaurar para mantener la continuidad empresarial.

Los aspectos de seguridad de las aplicaciones establecen una base para el uso de esta arquitectura de referencia a los efectos de admitir una carga de trabajo de Spring en Azure.

## <a name="next-steps"></a>Pasos siguientes

Explore esta arquitectura de referencia a través de las implementaciones ARM, Terraform y la CLI de Azure disponibles en el repositorio [Arquitectura de referencia de Azure Spring Cloud][10].

<!-- Reference links in article -->
[1]: ./index.yml
[2]: ../key-vault/index.yml
[3]: ../azure-monitor/index.yml
[4]: ../security-center/index.yml
[5]: /azure/devops/pipelines/
[6]: ../application-gateway/index.yml
[7]: ../web-application-firewall/index.yml
[8]: ./how-to-config-server.md
[9]: https://steeltoe.io/
[10]: https://github.com/Azure/azure-spring-cloud-reference-architecture
[11]: ./how-to-deploy-in-azure-virtual-network.md#virtual-network-requirements
[12]: ./vnet-customer-responsibilities.md#azure-spring-cloud-network-requirements
[13]: ./vnet-customer-responsibilities.md#azure-spring-cloud-fqdn-requirements--application-rules
[14]: ./how-to-staging-environment.md
[15]: https://devblogs.microsoft.com/java/monitor-applications-and-dependencies-in-azure-spring-cloud/
[16]: /azure/architecture/framework/
[17]: ./how-to-deploy-in-azure-virtual-network.md#virtual-network-requirements
[18]: https://cloudsecurityalliance.org/
[19]: https://cloudsecurityalliance.org/research/working-groups/cloud-controls-matrix
[20]: /azure/security/benchmarks/v2-cis-benchmark
[21]: https://www.cisecurity.org/