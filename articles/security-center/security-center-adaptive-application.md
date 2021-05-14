---
title: Controles de aplicación adaptables en Azure Security Center
description: Este documento le ayuda a usar los controles de aplicaciones adaptables de Azure Security Center para incluir en la lista de permitidos las aplicaciones que se ejecutan en máquinas de Azure.
author: memildin
manager: rkarlin
ms.service: security-center
ms.topic: how-to
ms.date: 04/21/2021
ms.author: memildin
ms.openlocfilehash: 9e2dcace673a1c7215634434f9e89ddc6b953a63
ms.sourcegitcommit: 3c460886f53a84ae104d8a09d94acb3444a23cdc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/21/2021
ms.locfileid: "107834631"
---
# <a name="use-adaptive-application-controls-to-reduce-your-machines-attack-surfaces"></a>Uso de controles de aplicaciones adaptables para reducir las superficies de ataque de las máquinas

Conozca las ventajas de los controles de aplicaciones adaptables de Azure Security Center y cómo puede mejorar la seguridad con esta característica inteligente controlada por datos.


## <a name="what-are-security-centers-adaptive-application-controls"></a>¿Qué son los controles de aplicaciones adaptables de Security Center?

Los controles de aplicaciones adaptables son una solución inteligente y automatizada que permite definir listas de aplicaciones permitidas seguras conocidas para las máquinas. 

A menudo, las organizaciones tienen colecciones de máquinas que ejecutan de forma rutinaria los mismos procesos. Security Center usa el aprendizaje automático para analizar las aplicaciones que se ejecutan en las máquinas y crear una lista del software seguro conocido. Las listas de permitidos se basan en las cargas de trabajo específicas de Azure, y se pueden personalizar aún más las recomendaciones mediante las instrucciones siguientes.

Cuando haya habilitado y configurado controles de aplicaciones adaptables, recibirá alertas de seguridad si alguna aplicación ejecuta otros distintos a los definidos como seguros.


## <a name="what-are-the-benefits-of-adaptive-application-controls"></a>¿Cuáles son las ventajas de los controles de aplicaciones adaptables?

Al definir listas de aplicaciones seguras conocidas y generar alertas cuando se ejecuta algo diferente, puede conseguir varios objetivos de protección:

- Identificar posible malware, incluso el que podría faltar en las soluciones antimalware
- Mejorar el cumplimiento de las directivas de seguridad locales que dictan el uso exclusivo de software con licencia
- Evitar la ejecución de aplicaciones antiguas o no admitidas
- Impedir software específico prohibido por la organización
- Aumentar la supervisión de las aplicaciones que acceden a datos confidenciales

Actualmente no hay opciones de cumplimiento disponibles. El fin de los controles de aplicaciones adaptables es generar alertas de seguridad si alguna aplicación ejecuta elementos que no son los que se han definido como seguros.

## <a name="availability"></a>Disponibilidad

|Aspecto|Detalles|
|----|:----|
|Estado de la versión:|Disponibilidad general (GA)|
|Precios:|Requiere [Azure Defender para servidores](defender-for-servers-introduction.md).|
|Máquinas admitidas:|![Sí](./media/icons/yes-icon.png) Máquinas de Azure y que no son de Azure que ejecutan Windows y Linux<br>![Sí](./media/icons/yes-icon.png) Máquinas de [Azure Arc](../azure-arc/index.yml)|
|Roles y permisos necesarios:|Los roles **Lector de seguridad** y **Lector** pueden ver grupos y las listas de aplicaciones seguras conocidas.<br>Los roles **Colaborador** y **Administrador de seguridad** pueden editar grupos y las listas de aplicaciones seguras conocidas.|
|Nubes:|![Sí](./media/icons/yes-icon.png) Nubes comerciales<br>![Sí](./media/icons/yes-icon.png) Nacionales o soberanas (US Gov, China Gov, otros gobiernos)|
|||



## <a name="enable-application-controls-on-a-group-of-machines"></a>Habilitación de controles de aplicaciones en un grupo de máquinas

Si Security Center ha identificado grupos de máquinas en las suscripciones que ejecutan sistemáticamente un conjunto similar de aplicaciones, se le avisará con la siguiente recomendación: **Adaptive application controls for defining safe applications should be enabled on your machines** (Los controles de aplicaciones adaptables para definir aplicaciones seguras deben estar habilitados en las máquinas).

Seleccione la recomendación o abra la página de controles de aplicaciones adaptables para ver la lista de grupos de máquinas y aplicaciones seguras conocidas.

1. Abra el panel de Azure Defender y, en el área de protección avanzada, seleccione **Controles de aplicaciones adaptables**.

    :::image type="content" source="./media/security-center-adaptive-application/opening-adaptive-application-control.png" alt-text="Apertura de controles de aplicaciones adaptables desde el panel de Azure" lightbox="./media/security-center-adaptive-application/opening-adaptive-application-control.png":::

    Se abre la página **Controles de aplicaciones adaptables** con las máquinas virtuales agrupadas en las siguientes pestañas:

    - **Configuradas**: grupos de máquinas que ya tienen una lista de aplicaciones permitidas. Para cada grupo, en la pestaña Configuradas se muestra:
        - el número de máquinas en el grupo
        - las alertas recientes

    - **Recomendadas**: grupos de máquinas que ejecutan de forma sistemática las mismas aplicaciones y no tienen configurada una lista de permitidos. Se recomienda habilitar los controles de aplicaciones adaptables para estos grupos.
    
      > [!TIP]
      > Si ve un nombre de grupo con el prefijo "REVIEWGROUP", contiene máquinas con una lista de aplicaciones parcialmente coherente. Security Center no puede encontrar un patrón, pero recomienda revisar este grupo para ver si _puede_ definir manualmente algunas reglas de controles de aplicaciones adaptables, como se describe en [Edición de una regla de controles de aplicaciones adaptables de un grupo](#edit-a-groups-adaptive-application-controls-rule).
      >
      > También puede mover máquinas de este grupo a otro, como se describe en [Movimiento de una máquina virtual de un grupo a otro](#move-a-machine-from-one-group-to-another).

    - **Ninguna recomendación**: máquinas sin una lista de aplicaciones permitidas definida y que no admiten la característica. Es posible que su máquina virtual esté en esta pestaña por las siguientes razones:
      - Falta un agente de Log Analytics
      - El agente de Log Analytics no envía eventos
      - Es una máquina Windows con una directiva de [AppLocker](/windows/security/threat-protection/windows-defender-application-control/applocker/applocker-overview) ya existente habilitada por un GPO o una directiva de seguridad local.

      > [!TIP]
      > Security Center necesita al menos dos semanas de datos para definir las recomendaciones únicas por grupo de máquinas. Las máquinas que se han creado recientemente, o que pertenecen a suscripciones que solo se habilitaron recientemente con Azure Defender, aparecerán en la pestaña **Ninguna recomendación**.


1. Seleccione la pestaña **Recomendadas**. Aparecen los grupos de máquinas con listas de permitidos recomendados.

   ![Pestaña Recomendadas](./media/security-center-adaptive-application/adaptive-application-recommended-tab.png)

1. Seleccione un grupo. 

1. Para configurar la nueva regla, revise las distintas secciones de la página **Configure application control rules** (Configurar reglas de control de aplicaciones) y el contenido, que será único para este grupo de máquinas específico:

   ![Configuración de una nueva regla](./media/security-center-adaptive-application/adaptive-application-create-rule.png)

   1. **Seleccionar máquinas**: de forma predeterminada, se seleccionan todas las máquinas del grupo identificado. Para excluir alguna de ellas de esta regla, anule su selección.
   
   1. **Aplicaciones recomendadas**: revise esta lista de aplicaciones que son comunes a las máquinas de este grupo y que se recomienda que tengan permiso para ejecutarse.
   
   1. **Más aplicaciones**: revise esta lista de aplicaciones que se ven con menos frecuencia en las máquinas de este grupo o que tienen vulnerabilidades conocidas. Un icono de advertencia indica que un atacante podría usar una determinada aplicación para pasar por alto una lista de aplicaciones permitidas. Se recomienda que revise atentamente estas aplicaciones.

      > [!TIP]
      > Ambas listas de aplicaciones incluyen la opción de restringir una aplicación específica a determinados usuarios. Adopte el principio de privilegios mínimos siempre que sea posible.
      > 
      > Los editores definen las aplicaciones; si una aplicación no tiene información del editor (está sin firmar), se crea una regla de ruta de acceso para la ruta de acceso completa de la aplicación específica.

   1. Para aplicar la regla, seleccione **Auditar**. 




## <a name="edit-a-groups-adaptive-application-controls-rule"></a>Edición de una regla de controles de aplicaciones adaptables de un grupo

Puede que decida editar la lista de permitidos de un grupo de máquinas debido a cambios conocidos en la organización. 

Para editar las reglas de un grupo de máquinas:

1. Abra el panel de Azure Defender y, en el área de protección avanzada, seleccione **Controles de aplicaciones adaptables**.

1. En la pestaña **Configuradas**, seleccione el grupo con la regla que quiere editar.

1. Revise las distintas secciones de la página **Configure application control rules** (Configurar reglas de control de aplicaciones), como se describe en [Habilitación de controles de aplicaciones adaptables en un grupo de máquinas](#enable-application-controls-on-a-group-of-machines).

1. Opcionalmente, agregue una o varias reglas personalizadas:

   1. Seleccione **Agregar regla**.

      ![Adición de una regla personalizada](./media/security-center-adaptive-application/adaptive-application-add-custom-rule.png)

   1. Si va a definir una ruta de acceso segura conocida, cambie **Tipo de regla** a "Ruta de acceso" y escriba una ruta única. Puede incluir caracteres comodín en la ruta de acceso.
   
      > [!TIP]
      > Algunos escenarios en los que pueden resultar útiles los caracteres comodín en una ruta de acceso son:
      > 
      > * El uso de un carácter comodín al final de una ruta de acceso para permitir todos los ejecutables dentro de esta carpeta y sus subcarpetas.
      > * El uso de un carácter comodín en medio de una ruta de acceso para permitir un nombre conocido de ejecutable con un nombre de carpeta variable (por ejemplo, carpetas de usuario personales que contienen un archivo ejecutable conocido, nombres de carpeta generados automáticamente, etc.).
  
   1. Defina los usuarios permitidos y los tipos de archivo protegidos.

   1. Cuando haya terminado de definir la regla, seleccione **Agregar**.

1. Seleccione **Guardar** para aplicar los cambios.


## <a name="review-and-edit-a-groups-settings"></a>Revisión y edición de la configuración de un grupo

1. Para ver los detalles y la configuración de su grupo, seleccione **Configuración de grupo**.

    En este panel se muestra el nombre del grupo (que se puede modificar), el tipo de sistema operativo, la ubicación y otros detalles pertinentes.

    :::image type="content" source="./media/security-center-adaptive-application/adaptive-application-group-settings.png" alt-text="Página Configuración de grupo para controles de aplicaciones adaptables" lightbox="./media/security-center-adaptive-application/adaptive-application-group-settings.png":::

1. Opcionalmente, puede modificar el nombre del grupo o los modos de protección de tipos de archivo.

1. Seleccione **Aplicar** y **Guardar**.



## <a name="respond-to-the-allowlist-rules-in-your-adaptive-application-control-policy-should-be-updated-recommendation"></a>Respuesta a la recomendación "Allowlist rules in your adaptive application control policy should be updated" (Se deben actualizar las reglas de listas de permitidos de la directiva de control de aplicaciones adaptables)

Esta recomendación aparece cuando la funcionalidad de aprendizaje automático de Security Center identifica un comportamiento potencialmente legítimo que no se ha permitido anteriormente. La recomendación sugiere nuevas reglas para las definiciones existentes con el fin de reducir el número de alertas de falsos positivos.

Para corregir los problemas:

1. En la página de recomendaciones, seleccione la recomendación **Allowlist rules in your adaptive application control policy should be updated** (Se deben actualizar las reglas de listas de permitidos de la directiva de control de aplicaciones adaptables) para ver los grupos con un comportamiento potencialmente legítimo identificado recientemente.

1. Seleccione el grupo con la regla que quiere editar.

1. Revise las distintas secciones de la página **Configure application control rules** (Configurar reglas de control de aplicaciones), como se describe en [Habilitación de controles de aplicaciones adaptables en un grupo de máquinas](#enable-application-controls-on-a-group-of-machines).

1. Para aplicar los cambios, seleccione **Auditar**.




## <a name="audit-alerts-and-violations"></a>Auditoría de alertas e infracciones

1. Abra el panel de Azure Defender y, en el área de protección avanzada, seleccione **Controles de aplicaciones adaptables**.

1. Para ver los grupos con máquinas que tienen alertas recientes, revise los grupos que aparecen en la pestaña **Configurados**.

1. Para investigar más, seleccione un grupo.

   ![Alertas recientes](./media/security-center-adaptive-application/recent-alerts.png)

1. Para tener más detalles, junto con la lista de máquinas afectadas, seleccione una alerta.

    En la página de alertas se muestran los detalles adicionales de las alertas y se proporciona un vínculo **Realizar acción** con recomendaciones sobre cómo mitigar la amenaza.

    :::image type="content" source="media/security-center-adaptive-application/adaptive-application-alerts-start-time.png" alt-text="La hora de inicio de las alertas de controles de aplicaciones adaptables":::

    > [!NOTE]
    > Los controles de aplicaciones adaptables calculan los eventos una vez cada doce horas. La "hora de inicio de la actividad" que se muestra en la página Alertas es la hora a la que los controles de aplicaciones adaptables crearon la alerta, **no** el tiempo que estuvo activo el proceso sospechoso.


## <a name="move-a-machine-from-one-group-to-another"></a>Movimiento de una máquina virtual de un grupo a otro

Cuando se mueve una máquina de un grupo a otro, la directiva de control de aplicaciones que se le ha aplicado cambia a la configuración del grupo al que se ha movido. También se puede mover una máquina de un grupo configurado a uno no configurado, y se quitarán las reglas de control de aplicaciones que se aplicaron a la máquina.

1. Abra el panel de Azure Defender y, en el área de protección avanzada, seleccione **Controles de aplicaciones adaptables**.

1. En la página **Controles de aplicaciones adaptables**, en la pestaña **Configurados**, seleccione el grupo que contiene la máquina que se va a mover.

1. Abra la lista de **Máquinas configuradas**.

1. Abra el menú de la máquina en los tres puntos que aparecen al final de la fila y seleccione **Mover**. Se abre el panel **Move machine to a different group** (Mover la máquina a un grupo diferente).

1. Seleccione el grupo de destino y elija **Move machine** (Mover máquina).

1. Para guardar los cambios, seleccione **Guardar**.





## <a name="manage-application-controls-via-the-rest-api"></a>Administración de controles de aplicaciones a través de la API REST 

Para administrar los controles de aplicaciones adaptables mediante programación, use nuestra API REST. 

La documentación de API pertinente está disponible en [la sección Controles de aplicación adaptables de la documentación de la API de Security Center](/rest/api/securitycenter/adaptiveapplicationcontrols).

Algunas de las funciones que están disponibles en la API REST:

* **List** recupera todas las recomendaciones de grupo y proporciona un archivo JSON con un objeto para cada grupo.

* **Get** recupera el archivo JSON con los datos completos de la recomendación (es decir, la lista de máquinas, el editor, las reglas de ruta de acceso, etc.).

* **Put** configura la regla (use el archivo JSON que recuperó con **Get** como cuerpo de esta solicitud).
 
   > [!IMPORTANT]
   > La función **Put** espera menos parámetros de los que contiene el archivo JSON devuelto por el comando Get.
   >
   > Antes de usar el archivo JSON en la solicitud Put, quite las siguientes propiedades: recommendationStatus, configurationStatus, issues, location y sourceSystem.


## <a name="faq---adaptive-application-controls"></a>Preguntas frecuentes: controles de aplicaciones adaptables

- [¿Hay alguna opción para aplicar los controles de aplicaciones?](#are-there-any-options-to-enforce-the-application-controls)
- [¿Por qué veo una aplicación Qualys en mis aplicaciones recomendadas?](#why-do-i-see-a-qualys-app-in-my-recommendeded-applications)

### <a name="are-there-any-options-to-enforce-the-application-controls"></a>¿Hay alguna opción para aplicar los controles de aplicaciones?
Actualmente no hay ninguna opción disponible. El fin de los controles de aplicaciones adaptables es generar **alertas de seguridad** si alguna aplicación ejecuta elementos que no son los que se han definido como seguros. Tienen una serie de ventajas ([¿Cuáles son las ventajas de los controles de aplicación adaptables?](#what-are-the-benefits-of-adaptive-application-controls)) y son extremadamente personalizables, como se muestra en esta página.

### <a name="why-do-i-see-a-qualys-app-in-my-recommendeded-applications"></a>¿Por qué veo una aplicación Qualys en mis aplicaciones recomendadas?
[Azure Defender para servidores](defender-for-servers-introduction.md) incluye el examen de vulnerabilidades de las máquinas sin costo adicional. No se necesita ninguna licencia ni cuenta de Qualys, ya que todo se administra sin problemas en Security Center. Para obtener detalles de este escáner e instrucciones sobre cómo implementarlo, consulte [Solución de evaluación de vulnerabilidades integrada de Defender](deploy-vulnerability-assessment-vm.md).

Para asegurarse de que no se generan alertas al implementar Security Center el escáner, la lista de permitidos recomendados de controles de aplicaciones adaptables incluye el escáner para todas las máquinas. 


## <a name="next-steps"></a>Pasos siguientes
En este documento ha aprendido a usar los controles de aplicaciones adaptables de Azure Security Center para definir listas de aplicaciones permitidas que se ejecutan en máquinas de Azure y que no son de Azure. Para más información sobre algunas de las otras características de protección de la carga de trabajo en la nube de Security Center, consulte:

* [Descripción del acceso a la máquina virtual Just-in-Time (JIT)](just-in-time-explained.md)
* [Protección de los clústeres de Azure Kubernetes](defender-for-kubernetes-introduction.md)