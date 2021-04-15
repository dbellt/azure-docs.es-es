---
title: Regiones y Availability Zones en Azure
description: Obtenga información sobre las regiones y Availability Zones en Azure para satisfacer sus requisitos técnicos y normativos.
author: prsandhu
ms.service: azure
ms.topic: conceptual
ms.date: 02/23/2021
ms.author: prsandhu
ms.reviewer: cynthn
ms.custom: fasttrack-edit, mvc
ms.openlocfilehash: 44ff938d0990a1a7794140a82c0e9222bb69a5d7
ms.sourcegitcommit: 77d7639e83c6d8eb6c2ce805b6130ff9c73e5d29
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/05/2021
ms.locfileid: "106384262"
---
# <a name="regions-and-availability-zones-in-azure"></a>Regiones y Availability Zones en Azure

Los servicios de Microsoft Azure están disponibles globalmente para impulsar las operaciones en la nube a un nivel óptimo. Puede elegir la mejor región para sus necesidades en función de las consideraciones técnicas y normativas: capacidades de servicio, residencia de datos, requisitos de cumplimiento y latencia.

## <a name="terminology"></a>Terminología

Para conocer mejor las regiones y Availability Zones en Azure, le resultará más fácil comprender los términos o conceptos clave.

| Término o concepto | Descripción |
| --- | --- |
| region | Un conjunto de centros de datos implementados dentro de un perímetro que define la latencia y conectados a través de una red regional dedicada de baja latencia. |
| geography | Un área del mundo que contiene al menos una región de Azure. Las zonas geográficas definen un mercado discreto que conserva los límites de residencia de datos y cumplimiento. Las zonas geográficas permiten a los clientes con necesidades específicas de residencia de datos y cumplimiento normativo mantener sus datos y aplicaciones cerca. Las zonas geográficas son tolerantes a errores hasta el punto de resistir una interrupción total del funcionamiento de una región gracias a su conexión con nuestra infraestructura de red dedicada de alta capacidad. |
| Zona de disponibilidad | Ubicaciones físicas exclusivas dentro de una región. Cada zona de disponibilidad consta de uno o varios centros de datos equipados con alimentación, refrigeración y redes independientes. |
| Región recomendada | Una región que proporciona la gama más amplia de capacidades de servicio que está diseñada para admitir Availability Zones ahora o en el futuro. Se designan en Azure Portal como **Recomendado**. |
| Región alternativa (otra) | Una región que extiende la superficie de Azure dentro de un límite de residencia de datos donde también existe una región recomendada. Las regiones alternativas ayudan a optimizar la latencia y proporcionan una segunda región para las necesidades de recuperación ante desastres. No están diseñadas para admitir Availability Zones (aunque Azure realiza una evaluación periódica de estas regiones para determinar si deben convertirse en regiones recomendadas). Se designan en Azure Portal como **Otras**. |
| Servicio fundamental | Un servicio básico de Azure que está disponible en todas las regiones cuando la región está disponible con carácter general. |
| Servicio estándar | Un servicio de Azure que está disponible en todas las regiones recomendadas en un plazo de 90 días a partir de la disponibilidad general de la región o de la disponibilidad basada en la demanda en regiones alternativas. |
| Servicio especializado | Un servicio de Azure que ofrece una disponibilidad basada en la demanda entre regiones respaldadas por hardware personalizado o especializado. |
| Servicio regional | Un servicio de Azure que se implementa de forma regional y permite al cliente especificar la región en la que se implementará el servicio. Para obtener una lista completa, vea [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=all). |
| Servicio no regional | Un servicio de Azure para el que no hay ninguna dependencia en una región específica de Azure. Los servicios no regionales se implementan en dos o más regiones y, si hay un error regional, la instancia del servicio en otra región sigue atendiendo a los clientes. Para obtener una lista completa, vea [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=all). |

## <a name="regions"></a>Regions

Una región es un conjunto de centros de datos implementados dentro de un perímetro definido por la latencia y conectados a través de una red regional dedicada de baja latencia. Azure ofrece la flexibilidad de implementar aplicaciones donde sea necesario, incluso en varias regiones para ofrecer una resistencia entre regiones. Para más información, vea [Información general sobre el pilar de resistencia](/azure/architecture/framework/resiliency/overview).

## <a name="availability-zones"></a>Zonas de disponibilidad

Una zona de disponibilidad constituye una oferta de alta disponibilidad que protege las aplicaciones y los datos de los errores en el centro de datos. Las zonas de disponibilidad son ubicaciones físicas exclusivas dentro de una región de Azure. Cada zona de disponibilidad consta de uno o varios centros de datos equipados con alimentación, refrigeración y redes independientes. Para garantizar la resistencia, hay un mínimo de tres zonas independientes en todas las regiones habilitadas. La separación física de las zonas de disponibilidad dentro de una región protege las aplicaciones y los datos frente a los errores del centro de datos. Los servicios con redundancia de zona replican las aplicaciones y los datos entre zonas de disponibilidad para protegerlos frente a puntos de error únicos. Con las zonas de disponibilidad, Azure ofrece el mejor Acuerdo de Nivel de Servicio del sector de tiempo de actividad de máquina virtual, con un 99,99 %. En el [SLA de Azure](https://azure.microsoft.com/support/legal/sla/virtual-machines/) completo se explica la disponibilidad garantizada de Azure como un conjunto.

Una zona de disponibilidad de una región de Azure es una combinación de un dominio de error y un dominio de actualización. Por ejemplo, si crea tres o más máquinas virtuales en tres zonas de una región de Azure, las máquinas virtuales se distribuyen eficazmente en tres dominios de error y tres dominios de actualización. La plataforma Azure reconoce esta distribución entre dominios de actualización para asegurarse de que las máquinas virtuales de distintas zonas se programan para actualizarse a la vez.

Coloque sus recursos de proceso, almacenamiento, red y datos dentro de una zona y replíquelos en otras para conseguir una alta disponibilidad en la arquitectura de sus aplicaciones. Los servicios de Azure que admiten zonas de disponibilidad se dividen en dos categorías:

- **Servicios de zona**: donde un recurso se ancla a una zona específica (por ejemplo, máquinas virtuales, discos administrados o direcciones IP estándar) o
- **Servicios con redundancia de zona**: cuando la plataforma de Azure se replica automáticamente entre zonas (por ejemplo, almacenamiento con redundancia de zona y SQL Database).

Para lograr una continuidad del negocio integral en Azure, cree la arquitectura de aplicación mediante la combinación de zonas de disponibilidad y pares de regiones de Azure. Puede replicar de forma sincrónica las aplicaciones y los datos mediante zonas de disponibilidad dentro de una región de Azure para conseguir alta disponibilidad, y replicar de forma asincrónica entre regiones de Azure para la protección de recuperación ante desastres.
 
![vista conceptual de una zona fuera de servicio en una región](./media/az-overview/az-graphic-two.png)

> [!IMPORTANT]
> Los identificadores de la zona de disponibilidad (los números 1, 2 y 3 de la imagen anterior) se asignan lógicamente a las zonas físicas reales para cada suscripción de manera independiente. Esto significa que la zona de disponibilidad 1 en una suscripción determinada podría hacer referencia a una zona física diferente de la zona de disponibilidad 1 en una suscripción diferente. En consecuencia, se recomienda no confiar en identificadores de zona de disponibilidad de diferentes suscripciones para la selección de máquina virtual.

## <a name="region-and-service-categories"></a>Categorías de región y servicio

El enfoque de Azure sobre la disponibilidad de los servicios de Azure entre regiones se describe mejor al expresar los servicios disponibles en las regiones recomendadas y alternativas.

- **Región recomendada**: una región que proporciona la gama más amplia de capacidades de servicio que está diseñada para admitir Availability Zones ahora o en el futuro. Se designan en Azure Portal como **Recomendado**.
- **Región alternativa (otra)** : una región que extiende la superficie de Azure dentro de un límite de residencia de datos donde también existe una región recomendada. Las regiones alternativas ayudan a optimizar la latencia y proporcionan una segunda región para las necesidades de recuperación ante desastres. No están diseñadas para admitir Availability Zones (aunque Azure realiza una evaluación periódica de estas regiones para determinar si deben convertirse en regiones recomendadas). Se designan en Azure Portal como **Otras**.

### <a name="comparing-region-types"></a>Comparación de tipos de regiones

Los servicios de Azure se agrupan en tres categorías: servicios fundamentales, estándar y especializados. La directiva general de Azure sobre la implementación de servicios en una región determinada se basa principalmente en el tipo de región, las categorías de servicio y la demanda del cliente:

- **Fundamentales**: están disponibles en todas las regiones recomendadas y alternativas cuando la región está disponible con carácter general, o en un plazo de 90 días después de que un nuevo servicio fundamental esté disponible con carácter general.
- **Estándar**: disponibles en todas las regiones recomendadas en un plazo de 90 días a partir de la disponibilidad general de la región; se basan en la demanda en las regiones alternativas (muchos ya están implementados en un gran subconjunto de regiones alternativas).
- **Especializados**: ofertas de servicio dirigidas, a menudo centradas en el sector o respaldadas por hardware personalizado o especializado. Disponibilidad basada en la demanda entre regiones (muchos ya están implementados en un gran subconjunto de regiones recomendadas).

Para ver los servicios que se implementan en una región determinada, así como la futura hoja de ruta de la versión preliminar o la disponibilidad general de los servicios de una región, vea [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/?products=all).

Si una oferta de servicio no está disponible en una región específica, puede compartir su interés poniéndose en contacto con su representante de ventas de Microsoft.

| Tipo de región | No regional | Fundamental | Estándar | Especializada | Zonas de disponibilidad | Residencia de datos |
| --- | --- | --- | --- | --- | --- | --- |
| Recomendado | :heavy_check_mark: | :heavy_check_mark: | :heavy_check_mark: | Basado en la demanda | :heavy_check_mark: | :heavy_check_mark: |
| Alternativa | :heavy_check_mark: | :heavy_check_mark: | Basado en la demanda | Basado en la demanda | N/D | :heavy_check_mark: |

### <a name="services-by-category"></a>Servicios por categoría

Como se mencionó anteriormente, Azure clasifica los servicios en tres categorías: fundamental, estándar y especializado. Las categorías de servicio se asignan en la disponibilidad general. A menudo, los servicios inician su ciclo de vida como un servicio especializado y, a medida que aumenta la demanda y el uso, se pueden promocionar a estándar o fundamental. En la tabla siguiente se muestra la categoría de los servicios como fundamental o estándar. Se debe tener en cuenta lo siguiente sobre la tabla:

- Algunos servicios no son regionales. Para obtener información y una lista de servicios no regionales, vea [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/).
- Las generaciones anteriores de servicios o máquinas virtuales no aparecen en la lista. Para más información, vea la documentación en [Generaciones anteriores de tamaños de máquina virtual](../virtual-machines/sizes-previous-gen.md).
- A los servicios no se les asigna una categoría hasta la disponibilidad general (GA). Para obtener información y una lista de servicios en versión preliminar, vea [Productos disponibles por región](https://azure.microsoft.com/global-infrastructure/services/). 

> [!div class="mx-tableFixed"]
> | Fundamental                           | Estándar                                        | 
> |----------------------------------------|---------------------------------------------------|
> | Cuentas de almacenamiento                       | API Management                                    | 
> | Application Gateway                    | Configuración de la aplicación                                 | 
> | Azure Backup                           | App Service                                       | 
> | Azure Cosmos DB                        | Automation                                        | 
> | Azure Data Lake Storage Gen2           | Azure Active Directory Domain Services            | 
> | Azure ExpressRoute                     | Azure Bastion                                     | 
> | IP pública de Azure                        | Azure Cache for Redis                             | 
> | Azure SQL Database                     | Azure Cognitive Search                            | 
> | Azure SQL Managed Instance             | Azure Cognitive Services                          | 
> | Disk Storage                           | Azure Cognitive Services: Computer Vision         | 
> | Event Hubs                             | Azure Cognitive Services: Content Moderator       | 
> | Key Vault                              | Azure Cognitive Services: Caras                    | 
> | Equilibrador de carga                          | Azure Cognitive Services: Lector inmersivo        | 
> | Azure Service Bus                            | Azure Cognitive Services: Language Understanding  | 
> | Service Fabric                         | Azure Cognitive Services: Speech Services         | 
> | Almacenamiento: Niveles de Blob Storage de acceso frecuente y de acceso esporádico   | Azure Cognitive Services: Text Analytics          | 
> | Almacenamiento: Managed Disks                 | Azure Cognitive Services: Traductor              | 
> | Virtual Machine Scale Sets             | Explorador de datos de Azure                               | 
> | Virtual Machines                       | Azure Data Share                                  | 
> | Virtual Machines: Azure Dedicated Host | Azure Database for MySQL                          | 
> | Virtual Machines: Serie Av2           | Azure Database for PostgreSQL                     | 
> | Virtual Machines: Serie Bs            | Azure DDoS Protection                             | 
> | Virtual Machines: Serie DSv2          | Azure Firewall                                    | 
> | Virtual Machines: Serie DSv3          | Azure Firewall Manager                            | 
> | Virtual Machines: Serie Dv2           | Azure Functions                                   | 
> | Virtual Machines: Serie Dv3           | Azure IoT Hub                                     |     
> | Virtual Machines: Serie ESv3          | Azure Kubernetes Service (AKS)                    | 
> | Virtual Machines: Serie Ev3           | Azure Machine Learning                            | 
> | Virtual Network                        | Azure Monitor: Application Insights               | 
> | VPN Gateway                            | Azure Monitor: Log Analytics                      | 
> |                                        | Azure Private Link                                | 
> |                                        | Red Hat OpenShift en Azure                           | 
> |                                        | Azure Site Recovery                               | 
> |                                        | Azure Stream Analytics                            | 
> |                                        | Azure Synapse Analytics                           | 
> |                                        | Batch                                             | 
> |                                        | Cloud Services: Serie M                          | 
> |                                        | Container Instances                               | 
> |                                        | Container Registry                                | 
> |                                        | Data Factory                                      | 
> |                                        | Event Grid                                        | 
> |                                        | HDInsight                                         |  
> |                                        | Logic Apps                                        | 
> |                                        | Media Services                                    | 
> |                                        | Network Watcher                                   | 
> |                                        | Notification Hubs                                 | 
> |                                        | Premium Blob Storage                              | 
> |                                        | Premium Files Storage                             | 
> |                                        | Virtual Machines: Serie Ddsv4                    | 
> |                                        | Virtual Machines: Serie Ddv4                     | 
> |                                        | Virtual Machines: Serie Dsv4                     | 
> |                                        | Virtual Machines: Serie Dv4                      | 
> |                                        | Virtual Machines: Serie Edsv4                    | 
> |                                        | Virtual Machines: Serie Edv4                     | 
> |                                        | Virtual Machines: Serie Esv4                     | 
> |                                        | Virtual Machines: Serie Ev4                      | 
> |                                        | Virtual Machines: Serie Fsv2                     | 
> |                                        | Virtual Machines: Serie M                        | 
> |                                        | Red WAN virtual                                       | 



### <a name="specialized-services"></a>Servicios especializados
Como se mencionó anteriormente, Azure clasifica los servicios en tres categorías: fundamental, estándar y especializado. Las categorías de servicio se asignan en la disponibilidad general. A menudo, los servicios inician su ciclo de vida como un servicio especializado y, a medida que aumenta la demanda y el uso, se pueden promocionar a estándar o fundamental. En esta tabla se muestran los servicios especializados. 

> [!div class="mx-tableFixed"]
> | Especializada                                          |
> |------------------------------------------------------|
> | API de Azure para FHIR                                   |
> | Azure Analysis Services                              |
> | Azure Cognitive Services: Anomaly Detector           |
> | Azure Cognitive Services: Custom Vision              |
> | Azure Cognitive Services: Form Recognizer            |
> | Azure Cognitive Services: Personalizer               |
> | Azure Cognitive Services: QnA Maker                  |
> | Azure Database for MariaDB                           |
> | Azure Database Migration Service                     |
> | Azure Dedicated HSM                                  |
> | Azure Digital Twins                                  |
> | Azure Health Bot                                     |
> | Azure HPC Cache                                      |
> | Azure Lab Services                                   |
> | Azure NetApp Files                                   |
> | Servicio Azure SignalR                                |
> | Servicio Azure Spring Cloud                           |
> | Azure Time Series Insights                           |
> | Azure VMware Solution                                |
> | Azure VMware Solution by CloudSimple                 |
> | Data Lake Analytics                                  |
> | Azure Machine Learning Studio (clásico)              |
> | Spatial Anchors                                      |
> | Almacenamiento: Archive Storage                             |
> | Almacenamiento en disco Ultra                                   |
> | Video Indexer                                        |
> | Virtual Machines: Serie DASv4                       |
> | Virtual Machines: Serie DAv4                        |
> | Virtual Machines: Serie DCsv2                       |
> | Virtual Machines: Serie EASv4                       |
> | Virtual Machines: Serie EAv4                        |
> | Virtual Machines: Serie HBv1                        |
> | Virtual Machines: Serie HBv2                        |
> | Virtual Machines: Serie HCv1                        |
> | Virtual Machines: Serie H                           |
> | Virtual Machines: Serie LSv2                        |
> | Virtual Machines: Serie Mv2                         |
> | Virtual Machines: Serie NCv3                        |
> | Virtual Machines: Serie NDv2                        |
> | Virtual Machines: Serie NVv3                        |
> | Virtual Machines: Serie NVv4                        | 
> | Virtual Machines: SAP HANA en Azure (instancias grandes)  |




## <a name="next-steps"></a>Pasos siguientes

- [Regiones que admiten Availability Zones en Azure](az-region.md)
- [Plantillas de inicio rápido](https://aka.ms/azqs)
