---
title: Solicitud de aumento de cuota y obtención de soporte técnico
description: Creación de una solicitud de soporte técnico en Azure Portal para Azure Synapse Analytics Solicite aumentos de cuota o soporte técnico para solucionar un problema.
services: synapse-analytics
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 03/10/2020
author: julieMSFT
ms.author: jrasnick
ms.reviewer: igorstan
ms.custom: seo-lt-2019, azure-synapse
ms.openlocfilehash: 5256c6d25a9c7acfc45cdffee05c95fb3407c24a
ms.sourcegitcommit: 590f14d35e831a2dbb803fc12ebbd3ed2046abff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107568325"
---
# <a name="request-quota-increases-and-get-support-for-azure-synapse-analytics"></a>Solicitud de aumentos de cuota y obtención de soporte técnico para Azure Synapse Analytics

En este artículo se describe cómo enviar una incidencia de soporte técnico en Azure Portal para Azure Synapse Analytics. Este proceso le permite solicitar un aumento de la cuota o enviar una solicitud de soporte técnico al equipo de soporte técnico de ingeniería.

## <a name="create-a-support-ticket"></a>Creación de una incidencia de soporte técnico

Use los pasos siguientes para crear una nueva solicitud de soporte técnico desde Azure Portal para Azure Synapse Analytics.

1. En el menú de [Azure Portal](https://portal.azure.com), seleccione **Ayuda y soporte técnico**.

   ![El vínculo de Ayuda y soporte técnico](./media/sql-data-warehouse-get-started-create-support-ticket/help-plus-support.png)


1. Seleccione **Ayuda y soporte técnico** y **Nueva solicitud de soporte técnico**.

    ![Creación de una solicitud de soporte técnico](./media/sql-data-warehouse-get-started-create-support-ticket/new-support-request.png)

1. Revise el [Plan de soporte técnico de Azure](https://azure.microsoft.com/support/plans/?WT.mc_id=Support_Plan_510979/).

   * El soporte técnico para **facturación, cuota y administración de suscripciones** está disponible en todos los niveles.
   * El soporte técnico para problemas del tipo **break-fix** se proporciona en los niveles [Desarrollador](https://azure.microsoft.com/support/plans/developer/), [Estándar](https://azure.microsoft.com/support/plans/standard/), [Professional Direct](https://azure.microsoft.com/support/plans/prodirect/) o [Premier](https://azure.microsoft.com/support/plans/premier/). Los problemas "break-fix" son aquellos que experimentan los clientes mientras usan Azure en los que hay una posibilidad razonable de que hayan sido causados por Microsoft.
   * El **aprendizaje para desarrolladores** y los **servicios de asesoramiento** están disponibles en los niveles de soporte técnico [Professional Direct](https://azure.microsoft.com/support/plans/prodirect/) y [Premier](https://azure.microsoft.com/support/plans/premier/).

   Si tiene un plan de soporte técnico Premier, también puede informar sobre problemas relacionados con Azure Synapse Analytics en el [portal Microsoft Premier Online](https://premier.microsoft.com/). Consulte [Planes de soporte técnico de Azure](https://azure.microsoft.com/support/plans/?WT.mc_id=Support_Plan_510979/) para obtener más información sobre los diversos planes de soporte técnico, incluido su ámbito, los tiempos de respuesta, el precio, etc.  Si quiere ver las preguntas más frecuentes sobre el soporte técnico de Azure, consulte [Preguntas más frecuentes de soporte técnico de Azure](https://azure.microsoft.com/support/faq/).

1. En **Tipo de problema**, seleccione el tipo de problema apropiado. En caso de problemas de break-fix, seleccione **Técnico**. Para las solicitudes de aumento de cuota, seleccione **Límites de servicio y suscripción (cuotas)** .

   ![Selección de un tipo de problema](./media/sql-data-warehouse-get-started-create-support-ticket/select-quota-issue-type.png)  

   > [!NOTE]
   > El resto de este artículo se centra en las solicitudes de aumento de cuota. No obstante, también puede seleccionar **Técnico** aquí para solicitudes de soporte técnico para la resolución de problemas. Si selecciona **Técnico**, se le pedirá que proporcione un resumen y, a continuación, identifique un tipo de problema en **Seleccionar el tipo de problema**. Es posible que vea soluciones que le ayuden a resolver el problema. Si las soluciones presentadas no resuelven el problema, seleccione **Siguiente: Detalles** y rellene el formulario para enviar la incidencia de soporte técnico.

1. Para las solicitudes de aumento de cuota, seleccione **Azure Synapse Analytics** para el **Tipo de cuota**. Después, seleccione **Next: Soluciones >>** .

   ![Selección de un tipo de cuota](./media/sql-data-warehouse-get-started-create-support-ticket/select-quota-type.png)

1. En la ventana **Detalles**, seleccione **Escribir detalles** para especificar información adicional.

   ![El vínculo "Proporcionar detalles"](./media/sql-data-warehouse-get-started-create-support-ticket/provide-details-link.png)

## <a name="quota-request-types"></a>Tipos de solicitud de cuota

Cuando selecciona **Proporcionar detalles** aparece la ventana **Detalles de la cuota**, que le permite agregar información adicional. En las secciones siguientes se describen las diferentes solicitudes de cuota disponibles para Azure Synapse Analytics.

### <a name="synapse-sql-pool-data-warehouse-units-dwus-per-server"></a>Unidades de almacenamiento de datos (DWU) del grupo de Synapse SQL por servidor

Siga estos pasos para solicitar un aumento de las DWU por servidor.

1. Seleccione el tipo de cuota **Synapse SQL pool DWUs per server** (DWU del grupo de Synapse SQL por servidor).

1. Seleccione el **Recurso** al que quiere aplicar el aumento de cuota mediante la lista desplegable.

1. Escriba la nueva cuota en la sección **Solicitar cuota**.

1. Seleccione **Guardar y continuar**.

   ![Detalles de la cuota de DWU](./media/sql-data-warehouse-get-started-create-support-ticket/quota-details-dwus.png)


### <a name="servers-per-subscription"></a>Servidores por suscripción

Para solicitar un aumento en el número de servidores por suscripción, tendrá que completar los siguientes pasos:

1. Seleccione el tipo de cuota **SQL Servers per subscription** (Servidores SQL Server por suscripción).

1. En la lista **Ubicación**, seleccione la región de Azure que se va a usar. La cuota es por suscripción en cada región.

1. En el campo **Solicitar cuota**, escriba la solicitud del número máximo de servidores de esa región.

   ![Detalles de la cuota de servidores](./media/sql-data-warehouse-get-started-create-support-ticket/quota-details-servers.png)



1. Seleccione **Guardar y continuar**.

Algunos tipos de oferta no están disponibles en todas las regiones. Puede ver el siguiente error:

![Error de acceso a la región](./media/sql-data-warehouse-get-started-create-support-ticket/region-access-error.png)

### <a name="enable-subscription-access-to-a-region"></a>Habilitar el acceso de suscripciones a una región

Para habilitar el acceso a una región para una suscripción, debe completar los pasos siguientes:  

1. Seleccione el tipo de cuota **Acceso a la región del grupo de Synapse SQL (almacenamiento de datos)** .

1. Seleccione la región; para ello, elija una **Ubicación** de la lista desplegable.

1. Indique el requisito de rendimiento de DWU en la sección **DWU necesarios**.

1. Escriba la **Descripción de los requisitos empresariales**. 

1. Seleccione **Guardar y continuar**.

![Acceso a la región](./media/sql-data-warehouse-get-started-create-support-ticket/quota-details-region.png)


### <a name="for-other-quota-requests"></a>Para otras solicitudes de cuota

Seleccione **Other quota request** (Otra solicitud de cuota) en el menú desplegable Tipo de cuota para otros tipos de solicitud de cuota:

![Otros detalles de cuota](./media/sql-data-warehouse-get-started-create-support-ticket/quota-details.png)

## <a name="submit-your-request"></a>Enviar la solicitud

El paso final consiste en rellenar los detalles restantes de la solicitud de soporte técnico de SQL Database. Después, seleccione **Next: Revisar y crear>>** .

![Detalles de Revisar y crear](./media/sql-data-warehouse-get-started-create-support-ticket/review-create-details.png)

Después de revisar los detalles de la solicitud, Seleccione **Crear** para enviar la solicitud.

![Crear un vale](./media/sql-data-warehouse-get-started-create-support-ticket/create-ticket.png)

## <a name="monitor-a-support-ticket"></a>Supervisión de una incidencia de soporte técnico

Una vez que ha enviado la solicitud de asistencia, el equipo de soporte técnico de Azure se pondrá en contacto con usted. Para comprobar tanto el estado de la solicitud como los detalles de la misma, seleccione **Todas las solicitudes de soporte técnico** en el panel.

![Comprobar estado](./media/sql-data-warehouse-get-started-create-support-ticket/monitor-ticket.png)

## <a name="other-resources"></a>Otros recursos

También puede conectar con la comunidad de Azure Synapse Analytics en [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-synapse+or+azure-sql-data-warehouse) o mediante la [página de preguntas y respuestas de Microsoft para Azure Synapse Analytics](/answers/topics/azure-synapse-analytics.html).