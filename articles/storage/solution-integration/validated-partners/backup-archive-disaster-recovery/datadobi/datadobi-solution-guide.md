---
title: Copia de seguridad de los datos en Azure con DobiProtect
titleSuffix: Azure Storage
description: Proporciona información general de los factores que se deben tener en cuenta y los pasos que deben seguirse para usar Azure como destino de almacenamiento y ubicación de recuperación para DobiProtect.
author: karauten
ms.author: karauten
ms.date: 04/12/2021
ms.topic: conceptual
ms.service: storage
ms.subservice: partner
ms.openlocfilehash: ef74136627f7f7e90e94b91f212438d8e9daf0d6
ms.sourcegitcommit: 1fbd591a67e6422edb6de8fc901ac7063172f49e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/07/2021
ms.locfileid: "109489979"
---
# <a name="backup-to-azure-with-dobiprotect"></a>Copia de seguridad en Azure con DobiProtect

Este artículo le ayuda a integrar una infraestructura de DobiProtect con Azure Blob Storage. Incluye requisitos previos, consideraciones, implementación y una guía de operaciones. En este artículo se aborda el uso de Azure como destino de copia de seguridad fuera del sitio y como sitio de recuperación en caso de desastre, lo que impide el funcionamiento normal en el sitio principal.
DobiProtect incluye DobiSync y DobiReplicate. DobiSync le permite proteger los datos mediante la creación periódica de una copia de ellos y su almacenamiento en un servidor de almacenamiento de objetos.  DobiReplicate le permite crear una copia de objetos o datos de un objeto o almacén principal conectado a la red, en un almacén secundario y mantener sincronizada la información y los metadatos relacionados. Por lo que respecta a este documento, solo se incluirán instrucciones para DobiSync.

## <a name="reference-architecture"></a>Arquitectura de referencia

En el siguiente diagrama se proporciona una arquitectura de referencia para las implementaciones locales a Azure y dentro de Azure.

![Arquitectura de referencia de DobiProtect en Azure](../media/dobiprotect-diagram.jpg)

La implementación de DobiProtect existente se puede integrar fácilmente en Azure si se agregan una o varias cuentas de Azure Storage como destino de almacenamiento en la nube. DobiProtect también le permite recuperar copias de seguridad desde el entorno local en Azure, lo que le proporciona un sitio de recuperación a petición en Azure.

## <a name="before-you-begin"></a>Antes de empezar

Un poco de planificación inicial le permitirá usar Azure como destino de copia de seguridad fuera del sitio y como sitio de recuperación.

### <a name="get-started-with-azure"></a>Introducción a Azure

Microsoft ofrece un marco para empezar a trabajar con Azure. [Cloud Adoption Framework](/azure/architecture/cloud-adoption/) (CAF) es un enfoque detallado para la transformación digital empresarial y una guía completa para la planificación de una adopción de la nube de calidad de producción. CAF incluye una [guía paso a paso de configuración de Azure](/azure/cloud-adoption-framework/ready/azure-setup-guide/) para ayudarle a ponerse en marcha de forma rápida y segura. Puede encontrar una versión interactiva en [Azure Portal](https://portal.azure.com/?feature.quickstart=true#blade/Microsoft_Azure_Resources/QuickstartCenterBlade). Encontrará arquitecturas de ejemplo, procedimientos recomendados específicos para la implementación de aplicaciones y recursos de aprendizaje gratuitos que le ayudarán a convertirse en un experto en Azure.

### <a name="consider-the-network-between-your-location-and-azure"></a>Consideraciones sobre la red entre su ubicación y Azure

Tanto si se usan los recursos en la nube para ejecutar los procesos de producción, pruebas y desarrollo, como si se trata de un destino de copia de seguridad y un sitio de recuperación, es importante comprender las necesidades de ancho de banda para la propagación inicial de copias de seguridad y para las transferencias diarias en curso.

Azure Data Box permite transferir la línea de base de la copia de seguridad inicial a Azure sin requerir más ancho de banda. Esto resulta útil si se estima que la transferencia de línea de base tarda más de lo admisible. Puede usar el estimador de transferencia de datos al crear una cuenta de almacenamiento para calcular el tiempo necesario para transferir la copia de seguridad inicial.

![Muestra estimador de transferencia de datos de Azure Storage en el portal.](../media/az-storage-transfer.png)

Recuerde que necesitará suficiente capacidad de red para admitir las transferencias de datos diarias en la ventana de transferencia necesaria (ventana de copia de seguridad) sin afectar a las aplicaciones de producción. En esta sección se describen las herramientas y técnicas disponibles para evaluar las necesidades de la red.

#### <a name="determine-how-much-bandwidth-youll-need"></a>Cálculo del ancho de banda necesario

Para determinar la cantidad de ancho de banda que necesitará, use los siguientes recursos:

- Herramientas de evaluación e informes independientes del software de copia de seguridad, como:
  - [MiTrend](https://mitrend.com/)
  - [Aptare](https://www.veritas.com/insights/aptare-it-analytics)
  - [Datavoss](https://www.datavoss.com/)

#### <a name="determine-unutilized-internet-bandwidth"></a>Cálculo del ancho de banda de Internet no utilizado

Es importante conocer cuánto ancho de banda tiene disponible a diario que no se suela utilizar (o su *capacidad de aumento*). Esto le ayudará a evaluar si puede cumplir los siguientes objetivos:

- hora inicial de carga cuando no se usa Azure Data Box para la propagación sin conexión.
- completar copias de seguridad diarias en función de la tasa de cambio identificada anteriormente y la ventana de copia de seguridad.

Use los siguientes métodos para identificar la capacidad de aumento de ancho de banda que sus copias de seguridad de Azure pueden consumir.

- Si ya es cliente de Azure ExpressRoute, consulte su [uso del circuito](../../../../../expressroute/expressroute-monitoring-metrics-alerts.md#circuits-metrics) en Azure Portal.
- Póngase en contacto con su ISP. Debería poder enviarle informes que muestren su uso diario y mensual existente.
- Hay varias herramientas que pueden medir el uso mediante la supervisión del tráfico de red en el nivel de enrutador o conmutador. Entre ellas se incluyen las siguientes:
  - [Bandwidth Analyzer Pack de Solarwinds](https://www.solarwinds.com/network-bandwidth-analyzer-pack?CMP=ORG-BLG-DNS)
  - [PRTG de Paessler](https://www.paessler.com/bandwidth_monitoring)
  - [Network Assistant de Cisco](https://www.cisco.com/c/en/us/products/cloud-systems-management/network-assistant/index.html)
  - [WhatsUp Gold](https://www.whatsupgold.com/network-traffic-monitoring)

### <a name="choose-the-right-storage-options"></a>Elección de las opciones de almacenamiento adecuadas

Al usar Azure como destino de copia de seguridad, hará uso de [Azure Blob Storage](../../../../blobs/storage-blobs-introduction.md). Blob Storage es la solución de almacenamiento de objetos de Microsoft. Blob Storage está optimizado para almacenar grandes cantidades de datos no estructurados, que no cumplen con ninguna definición o modelo de datos. Además, Azure Storage es duradero, seguro y escalable y ofrece una alta disponibilidad. Puede seleccionar el almacenamiento adecuado para la carga de trabajo con el fin de proporcionar el [nivel de resistencia](../../../../common/storage-redundancy.md) para cumplir los contratos de nivel de servicio internos. Blob Storage es un servicio de pago por uso. [Cada mes se le cobra](../../../../blobs/storage-blob-storage-tiers.md#pricing-and-billing) la cantidad de datos almacenados, el acceso a esos datos y, en el caso de los niveles de acceso esporádico y de archivo, un período de retención mínimo necesario. En las tablas siguientes se resumen las opciones de resistencia y de niveles aplicables a los datos de copia de seguridad.

**Opciones de resistencia de Blob Storage:**

|  |Con redundancia local  |Con redundancia de zona  |Con redundancia geográfica  |Con redundancia de zona geográfica  |
|---------|---------|---------|---------|---------|
|**N.º efectivo de copias**     | 3         | 3         | 6         | 6 |
|**N.º de zonas de disponibilidad**     | 1         | 3         | 2         | 4 |
|**N.º de regiones**     | 1         | 1         | 2         | 2 |
|**Conmutación por error manual en la región secundaria**     | N/D         | N/D         | Sí         | Sí |

**Niveles de Blob Storage:**

|  | nivel de acceso frecuente   |nivel de acceso esporádico   | nivel de archivo |
| ----------- | ----------- | -----------  | -----------  |
| **Disponibilidad** | 99,9 %         | 99%         | Sin conexión      |
| **Cargos de uso** | Mayores costos de almacenamiento, menores costos de acceso y transacciones | Menores costos de almacenamiento, mayores costos de acceso y transacciones | Menores costos de almacenamiento, mayores costos de acceso y transacciones |
| **Retención de datos mínima requerida**| N/D | 30 días | 180 días |
| **Latencia (tiempo hasta el primer byte)** | Milisegundos | Milisegundos | Horas |

#### <a name="sample-backup-to-azure-cost-model"></a>Ejemplo de modelo de costes de copia de seguridad en Azure

El pago por uso puede ser intimidatorio para los clientes que no están familiarizados con la nube. Aunque solo paga por la capacidad usada, el precio también incluye las transacciones (lectura o escritura) y la [salida de los datos](https://azure.microsoft.com/pricing/details/bandwidth/) que se vuelven a leer en el entorno local cuando se usa el [plan de datos ilimitados o local directo de Azure ExpressRoute](https://azure.microsoft.com/pricing/details/expressroute/), donde se incluye la salida de datos de Azure. Puede usar la [calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/) para realizar análisis de tipo "What if". Puede basar el análisis en la lista de precios o en los [precios de capacidad reservada de Azure Storage](../../../../../cost-management-billing/reservations/save-compute-costs-reservations.md), que pueden suponer un ahorro de hasta el 38%. Este es un ejercicio de precios de ejemplo para modelar el coste mensual de la copia de seguridad en Azure. Se trata solo de un ejemplo. *Los precios pueden variar debido a actividades que no se capturan aquí.*

|Factor de coste  |Coste mensual  |
|---------|---------|
|100 TB de datos de copia de seguridad en el almacenamiento esporádico     |1556,48 USD         |
|2 TB de datos nuevos escritos por día x 30 días     |39 USD en transacciones          |
|Total mensual estimado     |1595,48 USD         |
|---------|---------|
|Restauración única de 5 TB en un entorno local a través de la red pública de Internet   | 491,26 USD         |

> [!NOTE]
> Esta estimación se generó en la calculadora de precios de Azure con los precios de pago por uso de Este de EE. UU., y se basa en un tamaño de subfragmento de 32 MB que genera 65 536 solicitudes PUT (transacciones de escritura) por día. Este ejemplo puede no reflejar los precios actuales de Azure o no ser aplicable a sus requisitos.

## <a name="implementation-guidance"></a>Guía de implementación

En esta sección se proporciona una breve guía sobre cómo agregar Azure Storage a una implementación local de DobiSync. 

1. Abra Azure Portal y busque **cuentas de Storage**. También puede hacer clic en el icono predeterminado de **cuentas de Storage**.

    ![Muestra la adición de cuentas de Storage en Azure Portal.](../media/azure-portal.png)
  
    ![Muestra dónde ha escrito "Storage" en el cuadro de búsqueda de Azure Portal.](../media/locate-storage-account.png)

2. Seleccione **Crear** para agregar una cuenta. Seleccione o cree un grupo de recursos, proporcione un nombre único, elija la región, seleccione el rendimiento **Estándar**, deje siempre el tipo de cuenta como **Storage V2** y elija el nivel de replicación que se ajuste a los SLA y el nivel predeterminado que aplicará el software de copia de seguridad. Una cuenta de Azure Storage hace que los niveles de acceso frecuente, esporádico y de archivo estén disponibles en una sola cuenta, y las directivas de DobiSync permiten usar varios niveles para administrar de forma eficaz el ciclo de vida de los datos.

    ![Muestra la configuración de la cuenta de Storage en el portal.](../media/account-create-1.png)

3. No cambie las opciones de redes predeterminadas para ahora y pase a **Protección de datos**. En este caso, puede habilitar la opción "Eliminación temporal", que le permite recuperar un archivo de copia de seguridad eliminado accidentalmente dentro del período de retención definido y, además, ofrece protección contra la eliminación accidental o malintencionada.

    ![Muestra la configuración de la protección de datos en el portal.](../media/account-create-2.png)

4. A continuación, le recomendamos usar la configuración predeterminada de la pantalla **Opciones avanzadas** para los casos de uso de copias de seguridad en Azure.

    ![Muestra la pestaña Opciones avanzadas en el portal.](../media/account-create-3.png)

5. Agregue etiquetas para la organización si usa el etiquetado y cree su cuenta.

6. Ahora solo debe completar dos pasos rápidos para poder agregar la cuenta al entorno de DobiSync. En Azure Portal, vaya a la cuenta que creó y seleccione **Contenedores** en el menú **Blob service**. Agregue un contenedor y elija un nombre descriptivo. A continuación, navegue hasta el elemento **Claves de acceso** en **Configuración** y copie el **nombre de la cuenta de Storage** y una de las dos claves de acceso. Necesitará el nombre del contenedor, el nombre de la cuenta y la clave de acceso en los pasos siguientes.

    ![Muestra la creación del contenedor en el portal.](../media/container-c.png)

    ![Muestra la configuración de la clave de acceso en el portal.](../media/access-key.png)

7. (*Opcional*) Puede agregar capas de seguridad adicionales a la implementación.

    1. Configure el acceso basado en roles para limitar quién puede hacer cambios en la cuenta de Storage. Para obtener más información, consulte [Roles integrados en las operaciones de administración](../../../../common/authorization-resource-provider.md#built-in-roles-for-management-operations).
    1. Restrinja el acceso a la cuenta a segmentos de red específicos con la [configuración del firewall de Storage](../../../../common/storage-network-security.md) para evitar intentos de acceso desde fuera de la red corporativa.

        ![Muestra la configuración del firewall de Storage en el portal.](../media/storage-firewall.png)

    1. [Bloquee la eliminación](../../../../../azure-resource-manager/management/lock-resources.md) de la cuenta para evitar eliminar accidentalmente la cuenta de Storage.

        ![Muestra el bloqueo de eliminación de la cuenta en el portal.](../media/resource-lock.png)

    1. Configure los [procedimientos recomendados de seguridad](../../../../../storage/blobs/security-recommendations.md) adicionales.

1. En DobiSync, vaya a **Configuration** -> **Object Storage** (Configuración > Almacenamiento de objetos) y agregue un almacenamiento de objetos:

    ![Aparece la interfaz de usuario para agregar un nuevo almacenamiento de objetos en DobiSync.](../media/dobisync-new-storage.png)

9. Especifique el **nombre de cuenta** de la **cuenta de almacenamiento** y la **clave de cuenta**: 

     ![Captura de pantalla para la adición de credenciales de blob de Azure en DobiSync](../media/dobisync-add-storage.jpg)

10. Especifique un nombre para la conexión de **Azure Blob Storage**: ![Captura de pantalla de la interfaz de usuario de detalles del destino de sincronización de DobiSync](../media/dobisync-target-details.jpg)
    

11. En el paso siguiente verá todos los **contenedores de blobs** configurados en esta **cuenta de almacenamiento** concreta. Seleccione el protocolo y las credenciales de los contenedores que serán los destinos de sincronización, por ejemplo: 

    ![Captura de pantalla de la interfaz de usuario de contenedores de DobiSync](../media/dobisync-container-list.jpg)
 
12. Asigne los servidores proxy a esta conexión de blobs de Azure y haga clic en **Probar conexión** para asegurarse de que los servidores proxy puedan comunicarse con los contenedores:

    ![Captura de pantalla de la interfaz de usuario de contenedores de DobiSync](../media/dobisync-container-checked.jpg)

13. Se muestran los **resultados de la prueba** de conexión: 

    ![Captura de pantalla de la interfaz de usuario de prueba de conexión de DobiSync](../media/dobisync-connection.jpg)

13. Haga clic en "Finish" (Finalizar) para finalizar la configuración de blobs de Azure. A continuación, puede iniciar una nueva sesión de sincronización.

## <a name="operational-guidance"></a>Guía de operaciones

#### <a name="azure-portal"></a>Azure Portal

Azure proporciona una solución de supervisión sólida mediante [Azure Monitor](../../../../../azure-monitor/essentials/monitor-azure-resource.md). Puede [configurar Azure monitor](../../../../blobs/monitor-blob-storage.md) para realizar un seguimiento de la capacidad, las transacciones, la disponibilidad, la autenticación, etc. de Azure Storage. Puede encontrar la referencia completa de las métricas que se recopilan [aquí](../../../../blobs/monitor-blob-storage-reference.md). Algunas métricas útiles para hacer un seguimiento son BlobCapacity (para asegurarse de que permanece por debajo del [límite máximo de capacidad de la cuenta de almacenamiento](../../../../common/scalability-targets-standard-account.md)), Ingress y Egress (para hacer un seguimiento de la cantidad de datos que se escriben y se leen de la cuenta de Azure Storage) y SuccessE2ELatency (para hacer un seguimiento del tiempo de ida y vuelta de las solicitudes entre Azure Storage y MediaAgent).

También puede [crear alertas de registro](../../../../../service-health/alerts-activity-log-service-notifications-portal.md) para hacer un seguimiento del estado del servicio Azure Storage y ver el [Panel de estado de Azure](https://status.azure.com/status) en cualquier momento.

#### <a name="dobisync-documentation"></a>Documentación de DobiSync

Para más información sobre la configuración de DobiSync, visite el [manual del usuario](https://downloads.datadobi.com/NAS/olh/latest/dobisync.html) de DobiSync.

### <a name="how-to-open-support-cases"></a>Procedimiento para abrir casos de soporte técnico

Cuando necesite ayuda con la copia de seguridad en una solución de Azure, debe abrir un caso con Datadobi y con Azure. Esto permitirá que las organizaciones de soporte técnico colaboren si fuera necesario.

#### <a name="to-open-a-case-with-datadobi"></a>Procedimiento para abrir un caso con Datadobi

En el [sitio de soporte técnico de Datadobi](https://support.datadobi.com/s/), inicie sesión y abra un caso.

#### <a name="to-open-a-case-with-azure"></a>Procedimiento para abrir un caso con Azure

En [Azure Portal](https://portal.azure.com), busque **soporte técnico** en la barra de búsqueda de la parte superior. Seleccione **Ayuda y soporte técnico** -> **Nueva solicitud de soporte técnico**.

> [!NOTE]
> Cuando abra un caso, especifique que necesita ayuda con "Azure Storage" o "Redes de Azure". No especifique "Azure Backup". Azure Backup es el nombre de un servicio de Azure, y el caso no se asignaría correctamente.

### <a name="links-to-relevant-datadobi-documentation"></a>Vínculos a la documentación pertinente de Datadobi

Consulte la siguiente documentación de Datadobi para obtener más detalles:

- [Guía de requisitos previos](https://downloads.datadobi.com/NAS/guides/latest/prerequisites.html)
- [Guía de instalación de DobiProtect](https://downloads.datadobi.com/NAS/guides/latest/installguide.html)

## <a name="next-steps"></a>Pasos siguientes

Datadobi ha facilitado la implementación de su solución en Azure para proteger Azure Virtual Machines y muchos otros servicios de Azure. Para más información, consulte la siguiente referencia:

- [Protección de datos de archivos en Azure con DobiProtect](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/datadobi1602192408529.datadobi-dobiprotect?tab=overview)