---
title: Implementación de Start/Stop VMs v2 (versión preliminar)
description: En este artículo se indica cómo implementar la característica Start/Stop VMs v2 (versión preliminar) para las máquinas virtuales de Azure de su suscripción de Azure.
services: azure-functions
ms.subservice: ''
ms.date: 03/29/2021
ms.topic: conceptual
ms.openlocfilehash: 9ca808fffbd26c8837ad9a43447f60e99f89d922
ms.sourcegitcommit: 5fd1f72a96f4f343543072eadd7cdec52e86511e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/01/2021
ms.locfileid: "106111470"
---
# <a name="deploy-startstop-vms-v2-preview"></a>Implementación de Start/Stop VMs v2 (versión preliminar)

Siga la secuencia de pasos que aparecen en este tema para instalar la característica Start/Stop VMs v2 (versión preliminar). Después de completar el proceso de instalación, configure las programaciones para personalizar esta característica según sus necesidades.

## <a name="deploy-feature"></a>Implementación de la característica

La implementación se inicia desde la organización de GitHub Start/Stop VMs v2 [aquí](https://github.com/microsoft/startstopv2-deployments/blob/main/README.md). Si bien esta característica está pensada para administrar todas las máquinas virtuales de su suscripción en todos los grupos de recursos desde una implementación única dentro de la suscripción, puede instalar otra instancia de ella en función del modelo de operaciones o de los requisitos de su organización. También se puede configurar para administrar de forma centralizada las máquinas virtuales de varias suscripciones.

Para administrar su administración y eliminación, se recomienda implementar la característica Start/Stop VMs v2 (versión preliminar) en un grupo de recursos dedicado.

> [!NOTE]
> Actualmente, esta versión preliminar no le permite especificar una cuenta de almacenamiento ni un recurso de Application Insights.

1. Abra el explorador y vaya a la [organización de GitHub](https://github.com/microsoft/startstopv2-deployments/blob/main/README.md) Start/Stop VMs v2.
1. Seleccione la opción de implementación correspondiente al entorno de nube de Azure en el que se crean las máquinas virtuales de Azure. Se abrirá la página de implementación de Azure Resource Manager personalizada en Azure Portal.
1. Si se le solicita hacerlo, inicie sesión en [Azure Portal](https://portal.azure.com).
1. Escriba los siguientes valores:

    |Nombre |Value |
    |-----|------|
    |Region |Seleccione una región cerca de usted para los recursos nuevos.|
    |Nombre del grupo de recursos |Especifique el nombre del grupo de recursos que contendrá los recursos individuales para Start/Stop VMs. |
    |Región del grupo de recursos |Especifique la región del grupo de recursos. Por ejemplo, **Centro de EE. UU**.|
    |Nombre de la aplicación de funciones de Azure |Escriba un nombre que sea válido en una ruta de acceso de la dirección URL. El nombre que escriba se valida para asegurarse de que es único en Azure Functions. |
    |Nombre de Application Insights |Especifique el nombre de la instancia de Application Insights que contendrá el análisis para Start/Stop VMs. |
    |Región de Application Insights |Especifique la región de la instancia de Application Insights.|
    |Nombre de la cuenta de almacenamiento |Especifique el nombre de la cuenta de Azure Storage para almacenar la telemetría de ejecución de Start/Stop VMs. |
    |Dirección de correo electrónico |Especifique una o varias direcciones de correo electrónico para recibir notificaciones de estado, separadas por una coma (,).|

    :::image type="content" source="media/deploy/deployment-template-details.png" alt-text="Configuración de implementación de plantillas de Start/Stop VMs":::

1. Seleccione **Revisar y crear** en la parte inferior de la página.
1. Seleccione **Crear** para iniciar la implementación.
1. Seleccione el icono de campana (notificaciones) en la parte superior de la pantalla para ver el estado de implementación. Verá **Implementación en curso**. Espere hasta que se complete la implementación.
1. Seleccione **Ir al grupo de recursos** desde el panel de notificación. Debería ver una pantalla parecida a:

    :::image type="content" source="media/deploy/deployment-results-resource-list.png" alt-text="Lista de recursos de implementación de plantillas de Start/Stop VMs":::

## <a name="enable-multiple-subscriptions"></a>Habilitación de varias suscripciones

Una vez finalizada la implementación de inicio o detención, realice los pasos siguientes para habilitar iniciar/detener las máquinas virtuales V2 (versión preliminar) para realizar acciones en varias suscripciones.

1. Copie el valor del nombre de la aplicación de funciones de Azure que especificó durante la implementación.

1. En el portal, vaya a la suscripción secundaria. Seleccione la suscripción y, luego, **Access Control (IAM)** .

1. Seleccione **Agregar** y, luego, **Agregar asignación de roles**.

1. Seleccione el rol **Colaborador** en la lista desplegable **Rol**.

1. Escriba el nombre de la aplicación de funciones de Azure en el campo **Seleccionar**. Seleccione el nombre de la función en los resultados.

1. Seleccione **Guardar** para confirmar los cambios.

## <a name="configure-schedules-overview"></a>Información general sobre la configuración de programaciones

Para administrar el método de automatización a fin de controlar el inicio y la detención de las máquinas virtuales, puede configurar una o varias de las aplicaciones lógicas incluidas según sus requisitos.

- Programadas: las acciones de inicio y detención se basan en una programación que se especifica en las máquinas virtuales clásicas y de Azure Resource Manager.**ststv2_vms_Scheduled_start** y **ststv2_vms_Scheduled_stop** configuran el inicio y la detención programados.

- Secuenciado: las acciones de inicio y detención se basan en una programación dirigida a las máquinas virtuales con etiquetas de secuencia predefinidas. Solo se admiten dos etiquetas con nombre: **sequencestart** y **sequencestop**. **ststv2_vms_Sequenced_start** y **ststv2_vms_Sequenced_stop** configuran el inicio y la detención secuenciados.

    > [!NOTE]
    > En este escenario solo se admiten las máquinas virtuales de Azure Resource Manager.

- AutoStop: esta funcionalidad solo se usa para realizar una acción de detención en máquinas virtuales de Azure Resource Manager y clásicas en función de su uso de CPU. También puede ser una *acción* programada, que crea alertas en las máquinas virtuales y, en función de la condición, la alerta se desencadena para realizar la acción de detención.**ststv2_vms_AutoStop** configura la funcionalidad de detención automática.

Si necesita programaciones adicionales, puede duplicar una de las instancias de Logic Apps proporcionadas mediante la opción **Clonar** de Azure Portal.

:::image type="content" source="media/deploy/logic-apps-clone-option.png" alt-text="Selección de la opción Clonar para duplicar una aplicación lógica":::

## <a name="scheduled-start-and-stop-scenario"></a>Escenario de inicio y detención programados

Siga los pasos que se indican a continuación a fin de configurar la acción de inicio y detención programada para máquinas virtuales de Azure Resource Manager y clásicas. Por ejemplo, puede configurar la programación **ststv2_vms_Scheduled_start** para que se inicien en la mañana, cuando esté en la oficina, y para detener todas las máquinas virtuales de una suscripción cuando salga de trabajo por la tarde, según la programación **ststv2_vms_Scheduled_stop**.

Es posible configurar la aplicación lógica para que solo inicie las máquinas virtuales.

En cada escenario, puede dirigir la acción a una o varias suscripciones, a uno o varios grupos de recursos y especificar una o varias máquinas virtuales en una lista de inclusión o de exclusión. No puede especificarlas todas en la misma aplicación lógica.

1. Inicie sesión en [Azure Portal](https://portal.azure.com) y vaya a **Aplicaciones lógicas**.

1. En la lista de aplicaciones lógicas, seleccione **ststv2_vms_Scheduled_start** para configurar el inicio programado. Si desea configurar la detención programada, seleccione **ststv2_vms_Scheduled_stop**.

1. En el panel de la izquierda, seleccione **Diseñador de aplicaciones lógicas**.

1. Cuando aparezca el Diseñador de aplicaciones lógicas, en el panel del diseñador, seleccione **Periodicidad** para configurar la programación de las aplicaciones lógicas. Para información sobre las opciones de periodicidad específicas, consulte [Programación de tareas periódicas](../../connectors/connectors-native-recurrence.md#add-the-recurrence-trigger).

    :::image type="content" source="media/deploy/schedule-recurrence-property.png" alt-text="Configuración de la frecuencia de periodicidad de una aplicación lógica":::

1. En el panel del diseñador, seleccione **Try de función** para configurar los valores de destino. En el cuerpo de la solicitud, si quiere administrar las máquinas virtuales en todos los grupos de recursos de la suscripción, modifíquelo tal como se muestra en el ejemplo siguiente.

    ```json
    {
      "Action": "start",
      "EnableClassic": false,
      "RequestScopes": {
        "ExcludedVMLists": [],
        "Subscriptions": [
          "/subscriptions/12345678-1234-5678-1234-123456781234/"
        ]
     }
    }
    ```

    Especifique varias suscripciones en la matriz `subscriptions` con cada valor separado con una coma, tal como se indica en el ejemplo siguiente.

    ```json
    "Subscriptions": [
          "/subscriptions/12345678-1234-5678-1234-123456781234/",
          "/subscriptions/11111111-0000-1111-2222-444444444444/"
        ]
    ```

    En el cuerpo de la solicitud, si quiere administrar las máquinas virtuales de grupos de recursos específicos, modifíquelo tal como se muestra en el ejemplo siguiente. Cada ruta de acceso a recurso especificada se debe separar con una coma. Si es necesario, puede especificar uno o varios grupos de recursos.

    En este ejemplo también se muestra cómo excluir una máquina virtual. Si desea excluir la máquina virtual, especifique la ruta de acceso al recurso de máquinas virtuales o use un carácter comodín.

    ```json
    {
      "Action": "start",
      "EnableClassic": false,
      "RequestScopes": {
        "ResourceGroups": [
          "/subscriptions/12345678-1234-5678-1234-123456781234/resourceGroups/rg1/",
          "/subscriptions/11111111-0000-1111-2222-444444444444/resourceGroups/rg2/"
        ],
        "ExcludedVMLists": [
         "/subscriptions/12345678-1111-2222-3333-1234567891234/resourceGroups/vmrg1/providers/Microsoft.Compute/virtualMachines/vm1"
        ]
      }
    }
    ```

    Aquí, la acción se hará en todas las máquinas virtuales, excepto en aquellas cuyo nombre empieza con Az y Bz en ambas suscripciones.

    ```json
    {
      "Action": "start",
      "EnableClassic": false,
      "RequestScopes": {
        "ExcludedVMLists": [“Az*”,“Bz*”],
       "Subscriptions": [
          "/subscriptions/12345678-1234-5678-1234-123456781234/",
          "/subscriptions/11111111-0000-1111-2222-444444444444/"
    
        ]
      }
    }
    ```

    En el cuerpo de la solicitud, si quiere administrar un conjunto específico de máquinas virtuales dentro de la suscripción, modifique el cuerpo de la solicitud tal como se muestra en el ejemplo siguiente. Cada ruta de acceso a recurso especificada se debe separar con una coma. Si es necesario, puede especificar una máquina virtual.

    ```json
    {
      "Action": "start",
      "EnableClassic": true,
      "RequestScopes": {
        "ExcludedVMLists": [],
        "VMLists": [
          "/subscriptions/12345678-1234-5678-1234-123456781234/resourceGroups/rg1/providers/Microsoft.Compute/virtualMachines/vm1",
          "/subscriptions/12345678-1234-5678-1234-123456781234/resourceGroups/rg3/providers/Microsoft.Compute/virtualMachines/vm2",
          "/subscriptions/11111111-0000-1111-2222-444444444444/resourceGroups/rg2/providers/Microsoft.ClassicCompute/virtualMachines/vm30"
          
        ]
    }
    ```

## <a name="sequenced-start-and-stop-scenario"></a>Escenario de inicio y detención secuenciados

En un entorno que incluye dos o más componentes en varias máquinas virtuales de Azure Resource Manager de una arquitectura de aplicación distribuida, es importante que se pueda decidir la secuencia en que los componentes se inician o detienen en orden.

1. En la lista de aplicaciones lógicas, seleccione **ststv2_vms_Sequenced_start** para configurar el inicio secuenciado. Si desea configurar la detención secuenciada, seleccione **ststv2_vms_Sequenced_stop**.

1. En el panel de la izquierda, seleccione **Diseñador de aplicaciones lógicas**.

1. Cuando aparezca el Diseñador de aplicaciones lógicas, en el panel del diseñador, seleccione **Periodicidad** para configurar la programación de las aplicaciones lógicas. Para información sobre las opciones de periodicidad específicas, consulte [Programación de tareas periódicas](../../connectors/connectors-native-recurrence.md#add-the-recurrence-trigger).

    :::image type="content" source="media/deploy/schedule-recurrence-property.png" alt-text="Configuración de la frecuencia de periodicidad de una aplicación lógica":::

1. En el panel del diseñador, seleccione **Try de función** para configurar los valores de destino. En el cuerpo de la solicitud, si quiere administrar las máquinas virtuales en todos los grupos de recursos de la suscripción, modifíquelo tal como se muestra en el ejemplo siguiente.

    ```json
    {
      "Action": "start",
      "EnableClassic": false,
      "RequestScopes": {
        "ExcludedVMLists": [],
        "Subscriptions": [
          "/subscriptions/12345678-1234-5678-1234-123456781234/"
        ]
     },
       "Sequenced": true
    }
    ```

    Especifique varias suscripciones en la matriz `subscriptions` con cada valor separado con una coma, tal como se indica en el ejemplo siguiente.

    ```json
    "Subscriptions": [
          "/subscriptions/12345678-1234-5678-1234-123456781234/",
          "/subscriptions/11111111-0000-1111-2222-444444444444/"
        ]
    ```

    En el cuerpo de la solicitud, si quiere administrar las máquinas virtuales de grupos de recursos específicos, modifíquelo tal como se muestra en el ejemplo siguiente. Cada ruta de acceso a recurso especificada se debe separar con una coma. Si es necesario, puede especificar un grupo de recursos.

    En este ejemplo también se muestra cómo excluir una máquina virtual mediante la ruta de acceso al recurso en comparación con el ejemplo del inicio o detención programados, que usaba caracteres comodín.

    ```json
    {
      "Action": "start",
      "EnableClassic": false,
      "RequestScopes": {
        "ResourceGroups": [
          "/subscriptions/12345678-1234-5678-1234-123456781234/resourceGroups/rg1/",
          "/subscriptions/11111111-0000-1111-2222-444444444444/resourceGroups/rg2/"
        ],
        "ExcludedVMLists": [
         "/subscriptions/12345678-1111-2222-3333-1234567891234/resourceGroups/vmrg1/providers/Microsoft.Compute/virtualMachines/vm1"
        ]
      },
       "Sequenced": true
    }
    ```

    En el cuerpo de la solicitud, si quiere administrar un conjunto específico de máquinas virtuales dentro de una suscripción, modifique el cuerpo de la solicitud tal como se muestra en el ejemplo siguiente. Cada ruta de acceso a recurso especificada se debe separar con una coma. Si es necesario, puede especificar una máquina virtual.

    ```json
    {
      "Action": "start",
      "EnableClassic": true,
      "RequestScopes": {
        "ExcludedVMLists": [],
        "VMLists": [
          "/subscriptions/12345678-1234-5678-1234-123456781234/resourceGroups/rg1/providers/Microsoft.Compute/virtualMachines/vm1",
          "/subscriptions/12345678-1234-5678-1234-123456781234/resourceGroups/rg2/providers/Microsoft.ClassicCompute/virtualMachines/vm2",
          "/subscriptions/11111111-0000-1111-2222-444444444444/resourceGroups/rg2/providers/Microsoft.ClassicCompute/virtualMachines/vm30"
        ]
      },
       "Sequenced": true
    }
    ```

## <a name="auto-stop-scenario"></a>Escenario de detención automática

La característica Start/Stop VMs v2 (versión preliminar) puede ayudarlo a administrar el costo de la ejecución de máquinas virtuales de Azure Resource Manager y clásicas en su suscripción mediante la evaluación de las máquinas virtuales de Azure que no se utilizan durante los períodos de poca actividad, por ejemplo, las horas no laborables, y apagarlas automáticamente si el uso del procesador es menor de un porcentaje especificado.

Las propiedades de alerta de métricas siguientes en el cuerpo de la solicitud admiten la personalización:

- AutoStop_MetricName
- AutoStop_Condition
- AutoStop_Threshold
- AutoStop_Description
- AutoStop_Frequency
- AutoStop_Severity
- AutoStop_Threshold
- AutoStop_TimeAggregationOperator
- AutoStop_TimeWindow

Para más información sobre cómo funcionan las alertas de métricas de Azure Monitor y cómo configurarlas, consulte [Alertas de métricas en Azure Monitor](../../azure-monitor/alerts/alerts-metric-overview.md).

1. En la lista de aplicaciones lógicas, seleccione **ststv2_vms_AutoStop** para configurar la detención automática.

1. En el panel de la izquierda, seleccione **Diseñador de aplicaciones lógicas**.

1. Cuando aparezca el Diseñador de aplicaciones lógicas, en el panel del diseñador, seleccione **Periodicidad** para configurar la programación de las aplicaciones lógicas. Para información sobre las opciones de periodicidad específicas, consulte [Programación de tareas periódicas](../../connectors/connectors-native-recurrence.md#add-the-recurrence-trigger).

    :::image type="content" source="media/deploy/schedule-recurrence-property.png" alt-text="Configuración de la frecuencia de periodicidad de una aplicación lógica":::

1. En el panel del diseñador, seleccione **Try de función** para configurar los valores de destino. En el cuerpo de la solicitud, si quiere administrar las máquinas virtuales en todos los grupos de recursos de la suscripción, modifíquelo tal como se muestra en el ejemplo siguiente.

    ```json
    {
      "Action": "stop",
      "EnableClassic": false,    
      "AutoStop_MetricName": "Percentage CPU",
      "AutoStop_Condition": "LessThan",
      "AutoStop_Description": "Alert to stop the VM if the CPU % exceed the threshold",
      "AutoStop_Frequency": "00:05:00",
      "AutoStop_Severity": "2",
      "AutoStop_Threshold": "5",
      "AutoStop_TimeAggregationOperator": "Average",
      "AutoStop_TimeWindow": "06:00:00",
      "RequestScopes":{        
        "Subscriptions":[
            "/subscriptions/12345678-1111-2222-3333-1234567891234/",
            "/subscriptions/12345678-2222-4444-5555-1234567891234/"
        ],
        "ExcludedVMLists":[]
      }        
    }
    ```

    En el cuerpo de la solicitud, si quiere administrar las máquinas virtuales de grupos de recursos específicos, modifíquelo tal como se muestra en el ejemplo siguiente. Cada ruta de acceso a recurso especificada se debe separar con una coma. Si es necesario, puede especificar un grupo de recursos.

    ```json
    {
      "Action": "stop",
      "AutoStop_Condition": "LessThan",
      "AutoStop_Description": "Alert to stop the VM if the CPU % exceed the threshold",
      "AutoStop_Frequency": "00:05:00",
      "AutoStop_MetricName": "Percentage CPU",
      "AutoStop_Severity": "2",
      "AutoStop_Threshold": "5",
      "AutoStop_TimeAggregationOperator": "Average",
      "AutoStop_TimeWindow": "06:00:00",
      "EnableClassic": true,
      "RequestScopes": {
        "ExcludedVMLists": [],
        "ResourceGroups": [
          "/subscriptions/12345678-1111-2222-3333-1234567891234/resourceGroups/vmrg1/",
          "/subscriptions/12345678-1111-2222-3333-1234567891234/resourceGroupsvmrg2/",
          "/subscriptions/12345678-2222-4444-5555-1234567891234/resourceGroups/VMHostingRG/"
          ]
      }
    }
    ```

    En el cuerpo de la solicitud, si quiere administrar un conjunto específico de máquinas virtuales dentro de la suscripción, modifique el cuerpo de la solicitud tal como se muestra en el ejemplo siguiente. Cada ruta de acceso a recurso especificada se debe separar con una coma. Si es necesario, puede especificar una máquina virtual.

    ```json
    {
      "Action": "stop",
      "AutoStop_Condition": "LessThan",
      "AutoStop_Description": "Alert to stop the VM if the CPU % exceed the threshold",
      "AutoStop_Frequency": "00:05:00",
      "AutoStop_MetricName": "Percentage CPU",
      "AutoStop_Severity": "2",
      "AutoStop_Threshold": "5",
      "AutoStop_TimeAggregationOperator": "Average",
      "AutoStop_TimeWindow": "06:00:00",
      "EnableClassic": true,
      "RequestScopes": {
        "ExcludedVMLists": [],
        "VMLists": [
          "/subscriptions/12345678-1111-2222-3333-1234567891234/resourceGroups/rg3/providers/Microsoft.ClassicCompute/virtualMachines/Clasyvm11",
          "/subscriptions/12345678-1111-2222-3333-1234567891234/resourceGroups/vmrg1/providers/Microsoft.Compute/virtualMachines/vm1"
        ]
      }
    }
    ```

## <a name="next-steps"></a>Pasos siguientes

Si desea aprender a supervisar el estado de las máquinas virtuales de Azure administradas por la característica Start/Stop VMs v2 (versión preliminar) y realizar otras tareas de administración, consulte el artículo [Administración de Start/Stop VMs](manage.md).