---
title: Recuperación ante desastres para cuentas de integración
description: Configuración de las cuentas de integración y los artefactos B2B con recuperación ante desastres entre regiones en Azure Logic Apps
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: jonfan, estfan, logicappspm
ms.topic: article
ms.date: 04/10/2017
ms.openlocfilehash: 2705364fb60e0de02ae9c119228a93be8313f5eb
ms.sourcegitcommit: e39ad7e8db27c97c8fb0d6afa322d4d135fd2066
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111984849"
---
# <a name="set-up-cross-region-disaster-recovery-for-integration-accounts-in-azure-logic-apps"></a>Configuración de recuperación ante desastres entre regiones para cuentas de integración: Azure Logic Apps

Las cargas de trabajo de B2B implican transacciones monetarias como pedidos y facturas. Durante un evento de desastre, es fundamental que una empresa se recupere rápidamente a fin de cumplir los Acuerdos de Nivel de Servicio a nivel de negocio acordados con sus asociados. En este artículo se demuestra cómo crear un plan de continuidad empresarial para cargas de trabajo de B2B. 

* Preparación para la recuperación ante desastres 
* Conmutar por error a la región secundaria durante un desastre 
* Recurrir a la región primaria después de un evento de desastre

## <a name="disaster-recovery-readiness"></a>Preparación para la recuperación ante desastres  

1. Identifique una región secundaria y cree una [cuenta de integración](../logic-apps/logic-apps-enterprise-integration-create-integration-account.md) en ella.

2. Agregue socios, esquemas y acuerdos para los flujos de mensajes necesarios donde el estado de ejecución se tenga que replicar a la cuenta de integración de la región secundaria.

   > [!TIP]
   > Asegúrese de que haya coherencia en la convención de nomenclatura de los artefactos de la cuenta de integración entre regiones. 

3. Para extraer el estado de ejecución de la región primaria, cree una aplicación lógica en la región secundaria. 

   Esta aplicación lógica debe tener un *desencadenador* y una *acción*. 
   El desencadenador debe conectarse a la cuenta de integración de la región primaria y la acción debe conectarse a la cuenta de integración de la región secundaria. 
   Según el intervalo de tiempo, el desencadenador sondea la tabla de estado de ejecución de la región primaria y extrae los nuevos registros, si los hubiera. La acción los actualiza en la cuenta de integración de una región secundaria. 
   Esto ayuda a obtener el estado incremental en tiempo de ejecución de la región primaria a la secundaria.

4. La continuidad empresarial de la cuenta de integración de Logic Apps está diseñada para admitir protocolos basados en B2B: X12, AS2 y EDIFACT. Para ver los pasos detallados, seleccione los vínculos correspondientes.

5. Se recomienda implementar todos los recursos de la región primaria también en la región secundaria. 

   Los recursos de la región primaria incluyen Azure SQL Database o Azure Cosmos DB, Azure Service Bus/Azure Event Hubs para mensajería, Azure API Management y la característica Azure Logic Apps de Azure App Service.   

6. Establezca una conexión desde una región primaria a una región secundaria. Para extraer el estado de ejecución de una región primaria, cree una aplicación lógica en una región secundaria. 

   Esta aplicación lógica debe tener un desencadenador y una acción. 
   El desencadenador se debe conectar a la cuenta de integración de una región primaria. 
   La acción se debe conectar a la cuenta de integración de una región secundaria. 
   Según el intervalo de tiempo, el desencadenador sondea la tabla de estado de ejecución de la región primaria y extrae los nuevos registros, si los hubiera. 
   La acción los actualiza en la cuenta de integración de una región secundaria. 
   Este proceso ayuda a obtener el estado incremental en el entorno de tiempo de ejecución de la región primaria a la secundaria.

La continuidad empresarial en una cuenta de integración de Logic Apps proporciona compatibilidad con los protocolos B2B X12, AS2 y EDIFACT. Para ver los pasos detallados sobre cómo usar X12 y AS2, consulte [X12](../logic-apps/logic-apps-enterprise-integration-b2b-business-continuity.md#x12) y [AS2](../logic-apps/logic-apps-enterprise-integration-b2b-business-continuity.md#as2) en este artículo.

## <a name="fail-over-to-a-secondary-region-during-a-disaster-event"></a>Conmutación por error a una región secundaria durante un evento de desastre

Durante un desastre, si la región primaria no está disponible para la continuidad empresarial, el tráfico se dirige a la región secundaria. Una región secundaria ayuda a que una empresa recupere rápidamente sus funciones para cumplir con el RPO y RTO acordado con sus asociados. También minimiza los esfuerzos de conmutación por error de una región a otra. 

Hay una latencia prevista mientras se copian números de control de una región primaria a una secundaria. Para evitar el envío de números de control generados duplicados a los asociados durante un desastre, se recomienda aumentar los números de control en los contratos de región secundaria con [cmdlets de PowerShell](/powershell/module/azurerm.logicapp/set-azurermintegrationaccountgeneratedicn).

## <a name="fall-back-to-a-primary-region-post-disaster-event"></a>Reversión al evento posterior al desastre de la región primaria

Para revertir a una región primaria si está disponible, siga estos pasos:

1. Deje de aceptar mensajes de socios en la región secundaria.  

2. Aumente los números de control generados para todos los contratos de región primaria con [cmdlets de PowerShell](/powershell/module/azurerm.logicapp/set-azurermintegrationaccountgeneratedicn).  

3. Dirigir el tráfico desde la región secundaria a la región primaria.

4. Compruebe que la aplicación lógica creada en la región secundaria para extraer el estado de ejecución de la región primaria está habilitada.

## <a name="x12"></a>X12 

La continuidad empresarial de los documentos EDI X12 se basa en los números de control:

> [!TIP]
> También puede usar la [plantilla de inicio rápido de X12](https://azure.microsoft.com/resources/templates/logic-app-b2b-disaster-recovery-replication/) para crear aplicaciones lógicas. La creación de cuentas de integración primaria y secundaria es un requisito previo para usar la plantilla. La plantilla ayuda a crear dos aplicaciones lógicas, una para números de control recibidos y otra para números de control generados. Las acciones y los desencadenadores respectivos se crean en las aplicaciones lógicas, conectando el desencadenador a la cuenta de integración primaria y la acción a la cuenta de integración secundaria.

**Requisitos previos**

Seleccione comprobar la configuración duplicada en la configuración de recepción de contrato X12 a fin de habilitar la recuperación ante desastres para los mensajes entrantes.

![Seleccionar comprobar la configuración duplicada](./media/logic-apps-enterprise-integration-b2b-business-continuity/dupcheck.png)  

1. Cree una [aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md) en una región secundaria.    

2. Busque en **X12** y seleccione **X12: Cuando se modifica un número de control**.   

   ![Búsqueda de X12](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn1.png)

   El desencadenador le pide que establezca una conexión con una cuenta de integración. 
   El desencadenador se debe conectar a la cuenta de integración de una región primaria.

3. Escriba un nombre de conexión, seleccione la *cuenta de integración de la región primaria* en la lista y seleccione **Crear**.   

   ![Captura de pantalla que muestra dónde escribir un nombre de conexión y seleccionar la cuenta de integración de la región primaria. ](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn2.png)

4. La configuración **DateTime para iniciar la sincronización de números de control** es opcional. La **frecuencia** se puede establecer en **Día**, **Hora**, **Minuto** o **Segundo** con un intervalo.   

   ![Captura de pantalla que muestra la opción Fecha y hora en la que iniciar la sincronización del número de control.](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn3.png)

5. Seleccione **Nuevo paso** > **Agregar una acción**.

   ![Captura de pantalla que muestra el botón Nuevo paso y la opción Agregar una acción.](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn4.png)

6. Busque en **X12** y seleccione **X12: Permite agregar o actualizar números de control**.   

   ![Agregar o actualizar los números de control](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn5.png)

7. Para conectar una acción a la cuenta de integración de una región secundaria, seleccione **Cambiar conexión** > **Agregar nueva conexión** para ver una lista de las cuentas de integración disponibles. Escriba un nombre de conexión, seleccione la *cuenta de integración de la región secundaria* en la lista y haga clic en **Crear**. 

   ![Captura de pantalla que muestra dónde agregar el nombre de la cuenta de integración de una región secundaria.](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn6.png)

8. Cambie a entradas sin procesar haciendo clic en el icono situado en la esquina superior derecha.

   ![Captura de pantalla que muestra el icono que se debe seleccionar para cambiar a sin formato.](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12rawinputs.png)

9. Seleccione Cuerpo en el selector de contenido dinámico y guarde la aplicación lógica.

   ![Campos de contenido dinámico](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn7.png)

   Según el intervalo de tiempo, el desencadenador sondea la tabla de número de control recibido de la región primaria y extrae los nuevos registros. 
   La acción los actualiza en la cuenta de integración de una región secundaria. 
   Si no hay actualizaciones, el estado del desencadenador aparece como **Omitido**.   

   ![Tabla de número de control](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12recevicedcn8.png)

Según el intervalo de tiempo, el estado en tiempo de ejecución incremental se replica de una región primaria a una secundaria. Durante un desastre, si la región primaria no está disponible, el tráfico se dirige a la región secundaria para la continuidad empresarial. 

## <a name="edifact"></a>EDIFACT 

La continuidad empresarial de los documentos EDI EDIFACT se basa en los números de control.

**Requisitos previos**

Seleccione comprobar la configuración duplicada en la configuración de recepción de contrato EDIFACT a fin de habilitar la recuperación ante desastres para los mensajes entrantes.

![Captura de pantalla que muestra la configuración de comprobación duplicada en la configuración de recepción del contrato EDIFACT.](./media/logic-apps-enterprise-integration-b2b-business-continuity/edifactdupcheck.png)  

1. Cree una [aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md) en una región secundaria.    

2. Busque en **EDIFACT** y seleccione **EDIFACT: Cuando se modifica un número de control**.

   ![Búsqueda de EDIFACT](./media/logic-apps-enterprise-integration-b2b-business-continuity/edifactcn1.png)

   El desencadenador le pide que establezca una conexión con una cuenta de integración. 
   El desencadenador se debe conectar a la cuenta de integración de una región primaria. 

3. Escriba un nombre de conexión, seleccione la *cuenta de integración de la región primaria* en la lista y seleccione **Crear**.    

   ![Nombre de la cuenta de integración de la región primaria](./media/logic-apps-enterprise-integration-b2b-business-continuity/X12CN2.png)

4. La configuración **DateTime para iniciar la sincronización de números de control** es opcional. La **frecuencia** se puede establecer en **Día**, **Hora**, **Minuto** o **Segundo** con un intervalo.    

   ![Captura de pantalla que muestra la configuración de fecha y hora y frecuencia.](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn3.png)

6. Seleccione **Nuevo paso** > **Agregar una acción**.    

   ![Captura de pantalla que muestra dónde seleccionar Agregar una acción.](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn4.png)

7. Busque en **EDIFACT** y seleccione **EDIFACT: Permite agregar o actualizar números de control**.   

   ![Captura de pantalla que muestra dónde agregar o actualizar números de control.](./media/logic-apps-enterprise-integration-b2b-business-continuity/EdifactChooseAction.png)

8. Para conectar una acción a la cuenta de integración de una región secundaria, seleccione **Cambiar conexión** > **Agregar nueva conexión** para ver una lista de las cuentas de integración disponibles. Escriba un nombre de conexión, seleccione la *cuenta de integración de la región secundaria* en la lista y haga clic en **Crear**.

   ![Cree el nombre de la cuenta de integración de una región secundaria.](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12cn6.png)

9. Cambie a entradas sin procesar haciendo clic en el icono situado en la esquina superior derecha.

   ![Captura de pantalla que resalta el icono que se debe seleccionar si quiere cambiar a entradas sin procesar.](./media/logic-apps-enterprise-integration-b2b-business-continuity/Edifactrawinputs.png)

10. Seleccione Cuerpo en el selector de contenido dinámico y guarde la aplicación lógica.   

   ![Captura de pantalla que muestra el selector de contenido dinámico en el que puede seleccionar Cuerpo.](./media/logic-apps-enterprise-integration-b2b-business-continuity/X12CN7.png)

   Según el intervalo de tiempo, el desencadenador sondea la tabla de número de control recibido de la región primaria y extrae los nuevos registros.
   La acción los actualiza en la cuenta de integración de una región secundaria. 
   Si no hay actualizaciones, el estado del desencadenador aparece como **Omitido**.

   ![Tabla de número de control](./media/logic-apps-enterprise-integration-b2b-business-continuity/x12recevicedcn8.png)

Según el intervalo de tiempo, el estado en tiempo de ejecución incremental se replica de una región primaria a una secundaria. Durante un desastre, si la región primaria no está disponible, el tráfico se dirige a la región secundaria para la continuidad empresarial. 

## <a name="as2"></a>AS2 

La continuidad empresarial de los documentos que usan el protocolo AS2 se basa en el identificador de mensaje y el valor MIC.

> [!TIP]
> También puede usar la [plantilla de inicio rápido de AS2](https://github.com/Azure/azure-quickstart-templates/pull/3302) para crear aplicaciones lógicas. La creación de cuentas de integración primaria y secundaria es un requisito previo para usar la plantilla. La plantilla ayuda a crear una aplicación lógica que tiene un desencadenador y una acción. La aplicación lógica crea una conexión desde un desencadenador a una cuenta de integración primaria y una acción a una cuenta de integración secundaria.

1. Cree una [aplicación lógica](../logic-apps/quickstart-create-first-logic-app-workflow.md) en la región secundaria.  

2. Busque en **AS2** y seleccione **AS2: Cuando se crea un valor MIC**.   

   ![Búsqueda de AS2](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid1.png)

   Un desencadenador le pide que establezca una conexión con una cuenta de integración. 
   El desencadenador se debe conectar a la cuenta de integración de una región primaria. 
   
3. Escriba un nombre de conexión, seleccione la *cuenta de integración de la región primaria* en la lista y seleccione **Crear**.

   ![Captura de pantalla que muestra dónde escribir un nombre de conexión cuando se crea un valor de MIC.](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid2.png)

4. La configuración **DateTime para iniciar la sincronización del valor MIC** es opcional. La **frecuencia** se puede establecer en **Día**, **Hora**, **Minuto** o **Segundo** con un intervalo.   

   ![DateTime y frecuencia](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid3.png)

5. Seleccione **Nuevo paso** > **Agregar una acción**.  

   ![Nuevo paso y Agregar una acción](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid4.png)

6. Busque en **AS2** y seleccione **AS2: Agregar o actualizar contenidos MIC**.  

   ![Incorporación o actualización de MIC](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid5.png)

7. Para conectar una acción a la cuenta de integración secundaria, seleccione **Cambiar conexión** > **Agregar nueva conexión** para ver una lista de las cuentas de integración disponibles. Escriba un nombre de conexión, seleccione la *cuenta de integración de la región secundaria* en la lista y haga clic en **Crear**.

   ![Nombre de la cuenta de integración de la región secundaria](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid6.png)

8. Cambie a entradas sin procesar haciendo clic en el icono situado en la esquina superior derecha.

   ![Cambiar a entradas sin procesar](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2rawinputs.png)

9. Seleccione Cuerpo en el selector de contenido dinámico y guarde la aplicación lógica.   

   ![Contenido dinámico](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid7.png)

   Según el intervalo de tiempo, el desencadenador sondea la tabla de la región primaria y extrae los nuevos registros. La acción los actualiza en la cuenta de integración de una región secundaria. 
   Si no hay actualizaciones, el estado del desencadenador aparece como **Omitido**.  

   ![Tabla de la región primaria](./media/logic-apps-enterprise-integration-b2b-business-continuity/as2messageid8.png)

Según el intervalo de tiempo, el estado en tiempo de ejecución incremental se replica de la región primaria a la secundaria. Durante un desastre, si la región primaria no está disponible, el tráfico se dirige a la región secundaria para la continuidad empresarial. 

## <a name="next-steps"></a>Pasos siguientes

[Supervisión de mensajes B2B con los registros de Azure Monitor](../logic-apps/monitor-b2b-messages-log-analytics.md)
