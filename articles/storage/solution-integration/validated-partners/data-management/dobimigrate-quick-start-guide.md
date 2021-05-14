---
title: Migración de los datos de archivo a Azure con Datadobi DobiMigrate
titleSuffix: Azure Storage
description: Se proporciona una guía de introducción para implementar Datadobi DobiMigrate y migrar los datos a una solución NAS de Azure Files, Azure NetApp Files o ISV.
author: dukicn
ms.author: nikoduki
ms.date: 04/27/2021
ms.topic: conceptual
ms.service: storage
ms.subservice: partner
ms.openlocfilehash: 43a519cfd08ff96e2e53a07223f6cda31e98df50
ms.sourcegitcommit: 43be2ce9bf6d1186795609c99b6b8f6bb4676f47
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108280940"
---
# <a name="migrate-data-to-azure-with-datadobi-dobimigrate"></a>Migración de datos a Azure con Datadobi DobiMigrate

Este artículo le ayuda a integrar la infraestructura de Datadobi DobiMigrate con Azure Storage. Incluye requisitos previos, consideraciones, implementación y una guía de operaciones.

DobiMigrate permite migraciones de archivos y objetos entre plataformas de almacenamiento. Los datos se migran desde el entorno local a Azure de forma rápida, sencilla y rentable.

## <a name="reference-architecture"></a>Arquitectura de referencia

En el siguiente diagrama se proporciona una arquitectura de referencia para las implementaciones locales a Azure y dentro de Azure.

:::image type="content" source="./media/dobimigrate-quick-start-guide/dobimigrate-reference-architecture.png" alt-text="Arquitectura de referencia que describe la configuración básica de DobiMigrate":::

La implementación existente de DobiMigrate se puede integrar fácilmente con Azure agregando y configurando una conexión de Azure.

## <a name="before-you-begin"></a>Antes de empezar

Un poco de planificación inicial le permitirá usar Azure como destino de copia de seguridad fuera del sitio y como sitio de recuperación.

### <a name="get-started-with-azure"></a>Introducción a Azure

Microsoft ofrece un marco para empezar a trabajar con Azure. [Cloud Adoption Framework](/azure/architecture/cloud-adoption/) (CAF) es un enfoque detallado para la transformación digital empresarial y una guía completa para la planificación de una adopción de la nube de calidad de producción. CAF incluye una [guía paso a paso de configuración de Azure](/azure/cloud-adoption-framework/ready/azure-setup-guide/) para ayudarle a ponerse en marcha de forma rápida y segura. Puede encontrar una versión interactiva en [Azure Portal](https://portal.azure.com/?feature.quickstart=true#blade/Microsoft_Azure_Resources/QuickstartCenterBlade). Encontrará arquitecturas de ejemplo, procedimientos recomendados específicos para la implementación de aplicaciones y recursos de aprendizaje gratuitos que le ayudarán a convertirse en un experto en Azure.

### <a name="considerations-for-migrations"></a>Consideraciones sobre las migraciones

Al considerar las migraciones de datos de archivo a Azure, varios aspectos son importantes. Antes de continuar, consulte estos recursos para más información:

- [Introducción a la migración del almacenamiento](/azure/storage/common/storage-migration-overview)
- Características admitidas más recientes en DobiMigrate en la [matriz de comparación de herramientas de migración](/azure/storage/solution-integration/validated-partners/data-management/migration-tools-comparison).

Recuerde que necesitará suficiente capacidad de red para respaldar las migraciones sin que las aplicaciones de producción se vean afectadas. En esta sección se describen las herramientas y técnicas disponibles para evaluar las necesidades de la red.

#### <a name="determine-unutilized-internet-bandwidth"></a>Cálculo del ancho de banda de Internet no utilizado

Es importante conocer cuánto ancho de banda tiene disponible a diario que no se suela utilizar (o su *capacidad de aumento*). Puede resultarle útil evaluar si puede cumplir los siguientes objetivos:

- Tiempo inicial de las migraciones cuando no se usa Azure Data Box como el método sin conexión.
- Tiempo necesario para realizar la resincronización incremental antes de la conmutación final al servicio de archivos de destino.

Use los siguientes métodos para identificar la capacidad de aumento de ancho de banda que Azure puede consumir.

- Si ya es cliente de Azure ExpressRoute, consulte su [uso del circuito](../../../../expressroute/expressroute-monitoring-metrics-alerts.md#circuits-metrics) en Azure Portal.
- Póngase en contacto con el ISP y solicite informes para mostrar el uso diario y mensual existente.
- Hay varias herramientas que pueden medir el uso mediante la supervisión del tráfico de red en el nivel de enrutador o conmutador:
  - [SolarWinds Bandwidth Analyzer Pack](https://www.solarwinds.com/network-bandwidth-analyzer-pack?CMP=ORG-BLG-DNS)
  - [PRTG de Paessler](https://www.paessler.com/bandwidth_monitoring)
  - [Network Assistant de Cisco](https://www.cisco.com/c/en/us/products/cloud-systems-management/network-assistant/index.html)
  - [WhatsUp Gold](https://www.whatsupgold.com/network-traffic-monitoring)

## <a name="implementation-guidance"></a>Guía de implementación

En esta sección se proporciona una breve guía sobre cómo agregar recursos compartidos de Azure Files a un entorno local mediante la implementación de DobiMigrate en Azure. 

1. Abra Azure Portal y busque **Cuentas de almacenamiento**. 

    :::image type="content" source="./media/dobimigrate-quick-start-guide/azure-locate-storage-account.png" alt-text="Muestra dónde ha escrito &quot;Storage&quot; en el cuadro de búsqueda de Azure Portal.":::

    También puede hacer clic en el icono predeterminado **Cuentas de almacenamiento**.

    :::image type="content" source="./media/dobimigrate-quick-start-guide/azure-portal.png" alt-text="Se muestra cómo agregar una cuenta de almacenamiento en Azure Portal.":::

2. Seleccione **Crear** para agregar una cuenta.
   1. Seleccione un grupo de recursos existente o haga clic en **Crear nuevo** para crear uno.
   2. Especifique un nombre único para la cuenta de almacenamiento.
   3. Elija la región.
   4. Seleccione rendimiento **Estándar** o **Prémium**, en función de sus necesidades. Si selecciona **Prémium**, elija **Recursos compartidos de archivos** en **Premium account type** (Tipo de cuenta prémium).
   5. Elija el valor de **[Redundancia](/azure/storage/common/storage-redundancy)** que satisfaga los sus requisitos de protección de datos.
   
   :::image type="content" source="./media/dobimigrate-quick-start-guide/azure-account-create-1.png" alt-text="Se muestra la configuración de la cuenta de almacenamiento en el portal.":::

3. A continuación, se recomienda usar la configuración predeterminada de la pantalla **Opciones avanzadas**. Si va a migrar a Azure Files, se recomienda habilitar **Large file shares** (Recursos compartidos de archivos grandes) si está disponible.

   :::image type="content" source="./media/dobimigrate-quick-start-guide/azure-account-create-2.png" alt-text="Muestra la pestaña Opciones avanzadas en el portal.":::

4. Por ahora, no cambie las opciones de redes predeterminadas y continúe con **Protección de datos**. Puede elegir habilitar la eliminación temporal, que le permite recuperar datos eliminados por accidente dentro del período de retención definido. La eliminación temporal ofrece protección contra la eliminación accidental o malintencionada.

   :::image type="content" source="./media/dobimigrate-quick-start-guide/azure-account-create-3.png" alt-text="Muestra la configuración de la protección de datos en el portal.":::

5. Agregue etiquetas para la organización si usa el etiquetado y haga clic en **Crear** para crear su cuenta.
 
6. Ahora solo debe completar dos pasos rápidos para poder agregar la cuenta al entorno de DobiMigrate. Vaya a la cuenta que creó en Azure Portal y seleccione Recursos compartidos de archivos en el menú de servicio de archivo. Agregue un recurso compartido de archivos y elija un nombre descriptivo. A continuación, navegue hasta el elemento Claves de acceso en Configuración y copie el nombre de la cuenta de Storage y una de las dos claves de acceso.

   :::image type="content" source="./media/dobimigrate-quick-start-guide/azure-access-key.png" alt-text="Muestra la configuración de la clave de acceso en el portal.":::

7. Vaya a las propiedades del recurso compartido de archivos de Azure y tome la dirección URL; la necesitará para agregar la conexión de Azure a DobiMigrate:

   :::image type="content" source="./media/dobimigrate-quick-start-guide/azure-files-endpoint.png" alt-text="Búsqueda del punto de conexión de Azure Files.":::

8. (_Opcional_) Puede agregar capas de seguridad adicionales a la implementación.
 
   1. Configure el acceso basado en roles para limitar quién puede hacer cambios en la cuenta de Storage. Para obtener más información, consulte [Roles integrados en las operaciones de administración](/azure/storage/common/authorization-resource-provider#built-in-roles-for-management-operations).
 
   2.  Restrinja el acceso a la cuenta a segmentos de red específicos con la [configuración del firewall de almacenamiento](/azure/storage/common/storage-network-security). Configure las opciones del firewall para evitar el acceso desde fuera de la red corporativa.

       :::image type="content" source="./media/dobimigrate-quick-start-guide/azure-storage-firewall.png" alt-text="Muestra la configuración del firewall de Storage en el portal.":::

   3.  [Bloquee la eliminación](/azure/azure-resource-manager/management/lock-resources) de la cuenta para evitar eliminar accidentalmente la cuenta de Storage.

       :::image type="content" source="./media/dobimigrate-quick-start-guide/azure-resource-lock.png" alt-text="Muestra el bloqueo de eliminación de la cuenta en el portal.":::

   4.  Configure [procedimientos recomendados de seguridad](/azure/storage/blobs/security-recommendations) adicionales.

9.  En DobiMigrate, vaya a Configuration -> File Servers (Configuración > Servidores de archivos). Haga clic en **Add** (Agregar) para agregar Microsoft Azure Files como tipo de servidor de archivos:

    :::image type="content" source="./media/dobimigrate-quick-start-guide/dobimigrate-server-type.png" alt-text="Adición de Microsoft Azure files como tipo de servidor.":::

10. Especifique el nombre, los detalles de la conexión de Azure Files y las credenciales de la cuenta de almacenamiento:
 
    :::image type="content" source="./media/dobimigrate-quick-start-guide/dobimigrate-connection-details.png" alt-text="Configuración de los detalles de conexión de Azure Files.":::

11. Asigne los servidores proxy a la conexión de Azure Files y haga clic en **Probar conexión** para confirmar que los servidores proxy pueden comunicarse con Azure Files:
 
    :::image type="content" source="./media/dobimigrate-quick-start-guide/dobimigrate-test-connection.png" alt-text="Detalles de la prueba de la conexión.":::

    Se muestran los resultados de la prueba de conexión:

    :::image type="content" source="./media/dobimigrate-quick-start-guide/dobimigrate-test-results.png" alt-text="Se muestran los resultados de las conexiones de prueba.":::

12. En **SMB Migration Shares** (Recursos compartidos de migración SMB), verá todos los recursos compartidos de archivos de Azure que se aprovisionan en esta cuenta de almacenamiento. Establezca **Asignación** en **Manual** para los recursos compartidos que se encuentran en el ámbito de migración, por ejemplo:
 
    :::image type="content" source="./media/dobimigrate-quick-start-guide/dobiprotect-azure-files-shares.png" alt-text="Se muestran los recursos compartidos disponibles.":::

13. Haga clic en **Finalizar** para completar la configuración de Azure Files. A continuación, puede iniciar una nueva tarea de migración.

## <a name="support"></a>Soporte técnico 

Cuando necesite ayuda con la migración a una solución de Azure, debe abrir un caso tanto con Datadobi como con Azure.

### <a name="to-open-a-case-with-datadobi"></a>Procedimiento para abrir un caso con Datadobi

En el [sitio de soporte técnico de Datadobi](https://support.datadobi.com/s/), inicie sesión y abra un caso.

### <a name="to-open-a-case-with-azure"></a>Procedimiento para abrir un caso con Azure

En [Azure Portal](https://portal.azure.com/), busque **soporte técnico** en la barra de búsqueda de la parte superior. Seleccione **Ayuda y soporte técnico** -> **Nueva solicitud de soporte técnico**.

## <a name="marketplace"></a>Marketplace

Datadobi ha facilitado la implementación de su solución en Azure para proteger Azure Virtual Machines y muchos otros servicios de Azure. Para más información, consulte las siguientes referencias:

- [Migración de datos de archivos a Azure con DobiMigrate](https://azuremarketplace.microsoft.com/marketplace/apps/datadobi1602192408529.datadobi-dobimigrate?tab=overview)


## <a name="additional-resources"></a>Recursos adicionales

- [Información general sobre la migración del almacenamiento](/azure/storage/common/storage-migration-overview)
- [Manual del usuario de DobiMigrate](https://downloads.datadobi.com/NAS/olh/latest/dobimigrate.html)
- [Guía de requisitos previos de DobiMigrate](https://downloads.datadobi.com/NAS/guides/latest/prerequisites.html)
- [Guía de instalación de DobiMigrate](https://downloads.datadobi.com/NAS/guides/latest/installguide.html)
