---
title: Creación de implementaciones de actualizaciones para Update Management en Azure Automation
description: En este artículo se describe cómo programar implementaciones de actualizaciones y revisar su estado.
services: automation
ms.subservice: update-management
ms.date: 04/19/2021
ms.topic: conceptual
ms.openlocfilehash: e410e5de529bde122fe42d21b593a6fc483dcbc0
ms.sourcegitcommit: 6f1aa680588f5db41ed7fc78c934452d468ddb84
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107726690"
---
# <a name="how-to-deploy-updates-and-review-results"></a>Implementación de actualizaciones y revisión de los resultados

En este artículo se describe cómo programar una implementación de actualizaciones y revisar el proceso una vez completada la implementación. Puede configurar una implementación de actualizaciones desde una máquina virtual de Azure seleccionada, desde el servidor habilitado para Arc seleccionado o desde la cuenta de Automation en todas las máquinas y servidores configurados.

En cada escenario, la implementación que se crea tiene como destino esa máquina o servidor seleccionado o, en caso de la creación de una implementación desde la cuenta de Automation, puede tener como destino una o varias máquinas. Al programar una implementación de actualizaciones desde una máquina virtual de Azure o un servidor habilitado para Arc, los pasos son los mismos que para la implementación desde la cuenta de Automation, con estas excepciones:

* El sistema operativo se preselecciona de manera automática en función del SO de la máquina.
* La máquina de destino que se va a actualizar se establece automáticamente.
* Al configurar la programación, puede especificar **Actualizar ahora**, si sucede una vez o si usa una programación recurrente.

> [!IMPORTANT]
> Al crear una implementación de actualizaciones, acepta los términos de los términos de licencia de software (CLUF) proporcionados por la empresa que ofrece actualizaciones para su sistema operativo.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en el [Portal de Azure](https://portal.azure.com)

## <a name="schedule-an-update-deployment"></a>Programación de una implementación de actualizaciones

Cuando se programa una implementación de actualizaciones, se crea un recurso de [programación](../shared-resources/schedules.md) vinculado al runbook **Patch-MicrosoftOMSComputers** que controla la implementación de actualizaciones en la máquina o las máquinas de destino. Tiene que programar una implementación que se ajuste a la programación y a la ventana de servicio de su versión para instalar las actualizaciones. Puede elegir los tipos de actualizaciones que desea incluir en la implementación. Por ejemplo, puede incluir actualizaciones de seguridad o críticas y excluir paquetes acumulativos de actualizaciones.

>[!NOTE]
>Si elimina el recurso de programación desde Azure Portal o mediante PowerShell después de crear la implementación, esta operación de eliminación interrumpe la implementación de actualizaciones programada y, cuando se intenta volver a configurar el recurso de programación desde el portal, aparece un error. Solo se puede eliminar el recurso de programación si se elimina la programación de implementaciones correspondiente.  

Para programar una implementación de actualizaciones nueva, siga estos pasos. En función del recurso seleccionado (es decir, cuenta de Automation, servidor habilitado para Arc, máquina virtual de Azure), los pasos siguientes se aplican a todos ellos con pequeñas diferencias al configurar la programación de la implementación.

1. En el portal, para programar una implementación de:

   * Una o varias máquinas, vaya a **Cuentas de Automation** y seleccione su cuenta de Automation con Update Management habilitado en la lista.
   * En el caso de una máquina virtual de Azure, vaya a **Máquinas virtuales** y seleccione su VM en la lista.
   * En el caso de un servidor habilitado para Arc, vaya a **Servidores: Azure Arc** y seleccione su servidor en la lista.

2. En función del recurso seleccionado, para ir a Update Management:

   * Si seleccionó su cuenta de Automation, vaya a **Update Management** en **Update Management** y, luego, seleccione **Programar implementación de actualizaciones**.
   * Si seleccionó una VM de Azure, vaya a **Actualizaciones de invitado y host** y, luego, seleccione **Ir a Update Management**.
   * Si seleccionó un servidor habilitado para Arc, vaya a **Update Management** y, luego, seleccione **Programar implementación de actualizaciones**.

3. En **Nueva implementación de la actualización**, utilice el campo **Nombre** para escribir un nombre único para la implementación.

4. Seleccione el sistema operativo para la implementación de actualizaciones.

    > [!NOTE]
    > Esta opción no está disponible si seleccionó una VM de Azure o un servidor habilitado para Arc. El sistema operativo se identifica automáticamente.

5. En la región **Grupos para actualizar**, defina una consulta que combine una suscripción, grupos de recursos, ubicaciones y etiquetas con el fin de crear un grupo dinámico de VM de Azure e incluirlo en su implementación. Para más información, consulte [Uso de grupos dinámicos con Update Management](configure-groups.md).

    > [!NOTE]
    > Esta opción no está disponible si seleccionó una VM de Azure o un servidor habilitado para Arc. La máquina se establece automáticamente como el destino de la implementación programada.

6. En el área **Máquinas para actualizar**, seleccione una búsqueda guardada, un grupo importado, o bien la opción **Máquinas** del menú desplegable, y elija máquinas específicas. Mediante esta opción, puede ver el grado de preparación del agente de Log Analytics para cada máquina. Para más información sobre los distintos métodos de creación de grupos de equipos en los registros de Azure Monitor, consulte el artículo sobre los [grupos de equipos en los registros de Azure Monitor](../../azure-monitor/logs/computer-groups.md). Es posible incluir hasta un máximo de 1000 máquinas en una implementación de actualizaciones programada.

    > [!NOTE]
    > Esta opción no está disponible si seleccionó una VM de Azure o un servidor habilitado para Arc. La máquina se establece automáticamente como el destino de la implementación programada.

7. Use el área **Clasificaciones de actualizaciones** para especificar [clasificaciones de actualizaciones](view-update-assessments.md#work-with-update-classifications) para productos. En cada producto, anule la selección de todas las clasificaciones de actualizaciones compatibles, salvo las que se incluirán en la implementación de actualizaciones.

   :::image type="content" source="./media/deploy-updates/update-classifications-example.png" alt-text="Ejemplo que muestra la selección de clasificaciones de actualizaciones específicas.":::

    Si la implementación está pensada para aplicar solo un conjunto de actualizaciones específico, es necesario anular la selección de todas las clasificaciones de actualización preseleccionadas al configurar la opción **Incluir o excluir las actualizaciones** como se describe en el paso siguiente. Esto garantiza que solo se instalen en las máquinas de destino las actualizaciones que haya especificado para *incluirse* en esta implementación.

   >[!NOTE]
   > La implementación de actualizaciones mediante la clasificación de actualizaciones no funciona en las versiones RTM de CentOS. Para implementar correctamente actualizaciones para CentOS, seleccione todas las clasificaciones para asegurarse de que se aplican las actualizaciones. Actualmente no hay ningún método compatible para habilitar la disponibilidad de datos de clasificación nativos en CentOS. Consulte lo siguiente para obtener más información sobre [las clasificaciones de actualización](overview.md#update-classifications).

8. Use la región **Incluir o excluir las actualizaciones** para agregar o excluir actualizaciones seleccionadas de la implementación. En la página **Incluir/excluir**, escriba los números de identificador de artículo de KB que se van a incluir o excluir para actualizaciones de Windows. Para distribuciones de Linux compatibles, especifique el nombre del paquete.

   :::image type="content" source="./media/deploy-updates/include-specific-updates-example.png" alt-text="Ejemplo que muestra cómo incluir actualizaciones específicas.":::

   > [!IMPORTANT]
   > Es importante recordar que las exclusiones tienen prioridad sobre las inclusiones. Por ejemplo, si define una regla de exclusión con `*`, Update Management excluirá todas las revisiones o los paquetes de la instalación. Las revisiones excluidas aparecen todavía como que faltan en la máquina. En el caso de las máquinas Linux, si incluye un paquete que tiene un paquete dependiente excluido, Update Management no instalará el paquete principal.

   > [!NOTE]
   > No se pueden especificar actualizaciones reemplazadas para incluirlas en la implementación de actualizaciones.

   Estos son algunos escenarios de ejemplo que le ayudarán a comprender cómo usar la inclusión, la exclusión y la clasificación de actualizaciones simultáneamente en las implementaciones de actualizaciones:

   * Si solo quiere instalar una lista específica de actualizaciones, no debe seleccionar ninguna **clasificación de actualizaciones** y proporcionar una lista de las actualizaciones que se aplicarán mediante la opción **Incluir**.

   * Si quiere instalar solo actualizaciones críticas y de seguridad, junto con una o varias actualizaciones de controladores opcionales, debe seleccionar **Seguridad** y **Crítica** en **Clasificaciones de actualizaciones**. A continuación, en la opción **Incluir**, especifique las actualizaciones del controlador.

   * Si quiere instalar solo actualizaciones críticas y de seguridad, pero si quiere omitir una o varias actualizaciones para Python y así evitar que se rompa la aplicación heredada, debe seleccionar **Seguridad** y **Crítica** en **Clasificaciones de actualizaciones**. A continuación, para la opción **Excluir**, agregue los paquetes de Python que se omitirán.

9. Seleccione **Configuración de programación**. La hora de inicio predeterminada es 30 minutos después de la hora actual. Puede establecer la hora de inicio en cualquier momento a partir de 10 minutos en el futuro.

    > [!NOTE]
    > Esta opción es distinta si seleccionó un servidor habilitado para Arc. Puede seleccionar **Actualizar ahora** o una hora de inicio 20 minutos en el futuro.

10. Use **Periodicidad** para especificar si la implementación se producirá una vez o de manera periódica y, luego, seleccione **Aceptar**.

11. En la región **Scripts previos + scripts posteriores**, seleccione los scripts que se van a ejecutar antes y después de la implementación. Para más información, consulte [Administración de scripts previos y posteriores](pre-post-scripts.md).

12. Use el campo **Ventana de mantenimiento (minutos)** con el fin de indicar la cantidad de tiempo permitido para la instalación de actualizaciones. Tenga en cuenta los siguientes detalles al especificar una ventana de mantenimiento:

    * Las ventanas de mantenimiento controlan el número de actualizaciones que se instalan.
    * Update Management no detiene la instalación de nuevas actualizaciones si se está aproximando el final de una ventana de mantenimiento.
    * Update Management no finaliza las actualizaciones en curso si se supera la ventana de mantenimiento. No se intentarán las actualizaciones que quedan por instalar. Si esto ocurre con frecuencia, debe volver a evaluar la duración de la ventana de mantenimiento.
    * Normalmente, si se supera la ventana de mantenimiento de Windows, se debe a que una actualización de Service Pack tarda mucho tiempo en instalarse.

    > [!NOTE]
    > Para evitar que las actualizaciones se apliquen en Ubuntu fuera de la ventana de mantenimiento, vuelva a configurar el paquete `Unattended-Upgrade` para deshabilitar las actualizaciones automáticas. Para más información sobre cómo configurar el paquete, consulte el tema sobre [actualizaciones automáticas en la guía de Ubuntu Server](https://help.ubuntu.com/lts/serverguide/automatic-updates.html).

13. En el campo **Opciones de reinicio**, especifique cómo abordar los reinicios durante la implementación. Están disponibles las siguientes opciones: 
    * Reboot if necessary (default) [Reiniciar si es necesario (valor predeterminado)]
    * Always reboot (Reiniciar siempre)
    * Never reboot (No reiniciar nunca)
    * Solo reiniciar (no se instalarán actualizaciones)

    > [!NOTE]
    > Las claves del Registro que se muestran en la sección [Claves del Registro usadas para administrar reinicios](/windows/deployment/update/waas-restart#registry-keys-used-to-manage-restart) pueden provocar un evento de reinicio si el valor de **Opciones de reinicio** es **No reiniciar nunca**.

14. Cuando haya terminado de configurar la implementación, seleccione **Crear**.

    ![Panel de configuración de la programación de actualizaciones](./media/deploy-updates/manageupdates-schedule-win.png)

    > [!NOTE]
    > Cuando haya terminado de configurar la programación de la implementación de un servidor habilitado para Arc seleccionado, seleccione **Revisar y crear**.

15. Volverá al panel de estado. Seleccione **Programaciones de implementación** para ver la programación de implementación que creó. Se muestra un máximo de 500 programaciones. Si tiene más de 500 programas y desea revisar la lista completa, consulte el método API REST [Configuraciones de actualización de software: lista](/rest/api/automation/softwareupdateconfigurations/list). Especifique la versión de API 2019-06-01 o superior.

## <a name="schedule-an-update-deployment-programmatically"></a>Programación de una implementación de actualizaciones mediante programación

Para aprender a crear una implementación de actualizaciones con la API REST, consulte [Configuraciones de actualizaciones de software: crear](/rest/api/automation/softwareupdateconfigurations/create).

También puede usar un runbook de ejemplo para crear una implementación semanal de actualizaciones. Para más información acerca de este runbook, consulte [Create a weekly update deployment for one or more VMs in a resource group](https://github.com/azureautomation/create-a-weekly-update-deployment-for-one-or-more-vms-in-a-resource-group) (Crear una implementación de actualización semanal para una o más VM en un grupo de recursos).

## <a name="check-deployment-status"></a>Comprobar el estado de implementación

Una vez iniciada la implementación programa, puede ver el estado en la pestaña **Historial** en **Update Management**. Si la implementación se está ejecutando, su estado es **En curso**. Cuando la implementación finaliza correctamente, el estado cambia a **Correcto**. Si se producen errores con una o varias actualizaciones en la implementación, el estado es **Con errores**.

## <a name="view-results-of-a-completed-update-deployment"></a>Visualización de los resultados de una implementación de actualizaciones que se ha completado

Una vez finalizada la implementación, puede seleccionarla para consultar los resultados.

[ ![Panel de estado de implementación de actualizaciones para una implementación específica](./media/deploy-updates/manageupdates-view-results.png)](./media/deploy-updates/manageupdates-view-results-expanded.png#lightbox)

**Resultados de actualización** ofrece un resumen del número total de actualizaciones y de los resultados de la implementación en las máquinas virtuales de destino. En la tabla de la derecha, se muestra un análisis detallado de las actualizaciones y de los resultados individuales de la instalación.

Los valores disponibles son:

* **No intentado**: la actualización no se instaló porque no se disponía de tiempo suficiente, de acuerdo con la duración especificada para la ventana de mantenimiento.
* **No seleccionado**: la actualización no se seleccionó para su implementación.
* **Correcta**: la actualización se realizó correctamente.
* **Error**: la actualización generó un error.

Seleccione **Todos los registros** para ver todas las entradas de registro creadas por la implementación.

Seleccione **Salida** para ver el flujo de trabajo del runbook que administra la implementación de actualizaciones en las máquinas virtuales de destino.

Seleccione **Errores** para ver información detallada acerca de los errores de la implementación.

## <a name="next-steps"></a>Pasos siguientes

* Para obtener información sobre cómo crear alertas que le notifiquen los resultados de la implementación de actualizaciones, consulte [Creación de alertas para Update Management](configure-alerts.md).
* Para solucionar los errores de Update Management generales, consulte [Solución de problemas de Update Management](../troubleshoot/update-management.md).
