---
title: Ejemplo de plano técnico de CIS Microsoft Azure Foundations Benchmark v1.3.0
description: Introducción a un ejemplo de plano técnico de CIS Microsoft Azure Foundations Benchmark v1.3.0. Este ejemplo de plano técnico ayuda a los clientes a evaluar determinados controles concretos.
ms.date: 03/11/2021
ms.topic: sample
ms.openlocfilehash: 402d01d05a36bbcdcc36e60e16a05e51c9a8e30c
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108757680"
---
# <a name="cis-microsoft-azure-foundations-benchmark-v130-blueprint-sample"></a>Ejemplo de plano técnico de CIS Microsoft Azure Foundations Benchmark v1.3.0

El ejemplo de plano técnico de CIS Microsoft Azure Foundations Benchmark v1.3.0 proporciona directrices de protección de gobernanza mediante [Azure Policy](../../policy/overview.md) que le ayudan a evaluar recomendaciones específicas de CIS Microsoft Azure Foundations Benchmark v1.3.0. Este plano técnico ayuda a los clientes a implementar un conjunto básico de directivas para cualquier arquitectura implementada de Azure que deba implementar las recomendaciones de CIS Microsoft Azure Foundations Benchmark v1.3.0.

## <a name="recommendation-mapping"></a>Asignación de recomendaciones

La [asignación de recomendaciones de Azure Policy](../../policy/samples/cis-azure-1-3-0.md) proporciona detalles sobre las definiciones de directiva incluidas en este plano técnico y cómo se asignan estas definiciones de directiva a las **recomendaciones** en CIS Microsoft Azure Foundations Benchmark v1.3.0. Cuando se asignan a una arquitectura, Azure Policy evalúa los recursos para detectar posibles incumplimientos de las definiciones de directiva asignadas. Para más información, consulte [Azure Policy](../../policy/overview.md).

## <a name="deploy"></a>Implementación

Para implementar el ejemplo de plano técnico de CIS Microsoft Azure Foundations Benchmark v1.3.0 de Azure Blueprints, debe realizar los pasos siguientes:

> [!div class="checklist"]
> - Crear un plano técnico a partir del ejemplo
> - Marcar la copia del ejemplo como **publicada**
> - Asignar la copia del plano técnico a una suscripción existente

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free) antes de empezar.

### <a name="create-blueprint-from-sample"></a>Creación de un plano técnico a partir del ejemplo

En primer lugar, implemente el ejemplo de plano técnico mediante la creación de un plano técnico en su entorno tomando el ejemplo como punto de partida.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la página **Introducción** de la izquierda, seleccione el botón **Crear** en _Crear un plano técnico_.

1. Busque el ejemplo de plano técnico **CIS Microsoft Azure Foundations Benchmark v1.3.0** en _Otros ejemplos_ y seleccione **Usar este ejemplo**.

1. Escriba los _Aspectos básicos_ del ejemplo de plano técnico:

   - **Nombre del plano técnico**: Proporcione un nombre para la copia del ejemplo de plano técnico de CIS Microsoft Azure Foundations Benchmark.
   - **Ubicación de definición**: use los puntos suspensivos y seleccione el grupo de administración donde guardar la copia del ejemplo.

1. Seleccione la pestaña _Artefactos_ en la parte superior de la página **Siguiente: Artefactos** en la parte inferior de la página.

1. Revise la lista de artefactos incluidos en el ejemplo de plano técnico. Muchos de los artefactos tienen parámetros que se definirán más tarde. Seleccione **Guardar borrador** cuando haya terminado de revisar el ejemplo de plano técnico.

### <a name="publish-the-sample-copy"></a>Publicación de la copia del ejemplo

La copia del ejemplo de plano técnico ahora se ha creado en el entorno. Se crea en el modo **Borrador** y debe **publicarse** antes de que se pueda asignar e implementar. La copia del ejemplo de plano técnico se puede personalizar para adecuarla a su entorno y necesidades, pero esa modificación puede apartarla de la alineación con las recomendaciones de CIS Microsoft Azure Foundations Benchmark v1.3.0.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar su copia del ejemplo de plano técnico y, a continuación, selecciónela.

1. Seleccione **Publicar plano técnico** en la parte superior de la página. En la nueva página de la derecha, especifique una **versión** para la copia del ejemplo de plano técnico. Esta propiedad es útil si realiza una modificación posteriormente. En **Notas de cambios**, escriba "Primera versión publicada del ejemplo de plano técnico de CIS Microsoft Azure Foundations Benchmark". A continuación, seleccione **Publicar** en la parte inferior de la página.

### <a name="assign-the-sample-copy"></a>Asignación de la copia de ejemplo

Una vez que la copia del ejemplo de plano técnico se haya **publicado** correctamente, se podrá asignar a una suscripción dentro del grupo de administración donde se guardó. En este paso se proporcionan los parámetros para hacer que cada implementación de la copia del ejemplo de plano técnico sea única.

1. Seleccione **Todos los servicios** en el panel izquierdo. Busque y seleccione **Planos técnicos**.

1. En la parte izquierda, seleccione la página **Definiciones del plano técnico**. Use los filtros para buscar su copia del ejemplo de plano técnico y, a continuación, selecciónela.

1. Seleccione **Asignar plano técnico** en la parte superior de la página de definición del plano técnico.

1. Proporcione los valores de parámetro para la asignación de plano técnico:

   - Aspectos básicos

     - **Suscripciones**: seleccione una o varias de las suscripciones que están en el grupo de administración donde guardó la copia del ejemplo de plano técnico. Si selecciona más de una suscripción, se creará una asignación para cada una mediante los parámetros especificados.
     - **Nombre de asignación**: el nombre se rellena de antemano de forma automática en función del nombre del plano técnico.
       Cámbielo si fuera necesario o déjelo tal cual.
     - **Ubicación**: seleccione una región para la identidad administrada en la que se va a crear. Azure Blueprint usa esta identidad administrada para implementar todos los artefactos del plano técnico asignado. Para más información, consulte [Identidades administradas para recursos de Azure](../../../active-directory/managed-identities-azure-resources/overview.md).
     - **Versión de definición de Blueprint**: Elija una versión **publicada** de la copia del ejemplo de plano técnico.

   - Asignación de bloqueo

     Seleccione el valor de bloqueo de plano técnico para el entorno. Para más información, consulte [Bloqueo de recursos en planos técnicos](../concepts/resource-locking.md).

   - Identidad administrada

     Deje la opción predeterminada de identidad administrada _asignada por el sistema_.

   - Parámetros de artefacto

     Los parámetros definidos en esta sección se aplican al artefacto en el que se define. Estos parámetros son [parámetros dinámicos](../concepts/parameters.md#dynamic-parameters), ya que se definen durante la asignación del plano técnico. Para obtener una lista completa de los parámetros de los artefactos y sus descripciones, consulte la [tabla de parámetros de los artefactos](#artifact-parameters-table).

1. Una vez que se hayan especificado todos los parámetros, seleccione **Asignar** en la parte inferior de la página. Se crea la asignación del plano técnico y comienza la implementación del artefacto. La implementación tarda aproximadamente una hora. Para comprobar el estado de implementación, abra la asignación del plano técnico.

> [!WARNING]
> El servicio Azure Blueprints y los ejemplos de plano técnico incorporados son **gratuitos**. El precio de los recursos de Azure se [calcula por producto](https://azure.microsoft.com/pricing/). Use la [calculadora de precios](https://azure.microsoft.com/pricing/calculator/) para estimar el costo de la ejecución de los recursos implementados en este ejemplo de plano técnico.

### <a name="artifact-parameters-table"></a>Tabla de parámetros de los artefactos

En la tabla siguiente se proporciona una lista de los parámetros del artefacto de plano técnico:

|Nombre del artefacto|Tipo de artefacto|Nombre de parámetro|Descripción|
|-|-|-|-|
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Lista de extensiones de máquina virtual aprobadas para usarse|Lista de extensiones de máquina virtual separadas por punto y coma. Para ver una lista completa de extensiones, use el comando Get-AzVMExtensionImage de Azure PowerShell.|
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Las instancias administradas de SQL deben usar claves administradas por el cliente para cifrar los datos en reposo.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: la configuración de Evaluación de vulnerabilidad para SQL Server debe contener una dirección de correo electrónico para recibir los informes de los exámenes.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Los registros de diagnóstico de Azure Data Lake Store deben estar habilitados.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: El cifrado de discos debe aplicarse en máquinas virtuales|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Key Vault debe tener habilitada la protección contra operaciones de purga.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Asegúrese de que la aplicación de API tenga la opción de "Certificados de cliente (certificados de cliente entrantes)" activada.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Los servidores SQL Server deben usar claves administradas por el cliente para cifrar los datos en reposo.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Se debe usar una identidad administrada en la aplicación de funciones.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Se debe habilitar Azure Defender para Key Vault.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: No deben existir roles de propietario de suscripción personalizados.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto de la directiva: las claves deben tener establecidas fechas de expiración.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Se debe permitir el cifrado de datos transparente en bases de datos SQL|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: la evaluación de vulnerabilidades debe estar habilitada en SQL Managed Instance.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de PHP es la más reciente si se usa como parte de la aplicación de API.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: El administrador de Azure Active Directory debe aprovisionarse para servidores SQL Server|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Se debe habilitar Azure Defender para Key Vault.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Las cuentas de almacenamiento deben restringir el acceso mediante el uso de las reglas de red virtual.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Se debe usar una identidad administrada en la aplicación web.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: El acceso SSH desde Internet debe estar bloqueado.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Se deben cifrar los discos no conectados.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Se debe habilitar Azure Defender para Storage.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: las cuentas de almacenamiento deben restringir el acceso a la red.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Los registros de diagnóstico de Logic Apps deben estar habilitados.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Los registro de diagnóstico de IoT Hub deben estar habilitados.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: FTPS solo debe exigirse en su aplicación de funciones.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Debe existir una alerta de registro de actividad para las operaciones de seguridad específicas (Microsoft.Security/securitySolutions/delete).|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Debe existir una alerta de registro de actividad para las operaciones de seguridad específicas (Microsoft.Security/securitySolutions/write).|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Se debe habilitar la transferencia segura a las cuentas de almacenamiento|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Los registros de diagnóstico de las cuentas de Batch deben estar habilitados.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: El aprovisionamiento automático del agente de Log Analytics debe habilitarse en la suscripción.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de Java es la más reciente si se usa como parte de la aplicación web.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: FTPS debe exigirse en su aplicación web.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Se debe habilitar Azure Defender para los servidores.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Las suscripciones deben tener una dirección de correo electrónico de contacto para los problemas de seguridad.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: No se debe permitir el acceso público a la cuenta de almacenamiento.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Se debe habilitar Azure Defender para Kubernetes.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: La limitación de conexiones debe estar habilitada para los servidores de bases de datos PostgreSQL.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Asegúrese de que la aplicación web tenga la opción de "Certificados de cliente (certificados de cliente entrantes)" activada.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Las cuentas externas con permisos de escritura deben quitarse de la suscripción|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Las cuentas externas con permisos de lectura deben quitarse de la suscripción|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Se debe habilitar Azure Defender para servidores SQL Server en las máquinas.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: La notificación por correo electrónico de las alertas de gravedad alta debe estar habilitada.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: La cuenta de almacenamiento debe utilizar una clave administrada por el cliente para el cifrado.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de Python es la más reciente si se usa como parte de la aplicación web.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de Python" es la más reciente si se usa como parte de la aplicación de funciones.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de PHP es la más reciente si se usa como parte de la aplicación web.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de Python es la más reciente si se usa como parte de la aplicación de API.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Los registros de diagnóstico de Virtual Machine Scale Sets deben estar habilitados.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Se debe habilitar Azure Defender para servidores de Azure SQL Database.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Los registros de diagnóstico del centro de eventos deben estar habilitados.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Se deben instalar actualizaciones del sistema en las máquinas|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de Java es la más reciente si se usa como parte de la aplicación de API.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Los servidores SQL Server se deben configurar con una retención de auditoría de 90 días, o más.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de HTTP es la más reciente si se usa para ejecutar la aplicación web.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: La versión más reciente de TLS debe usarse en la aplicación de API.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: MFA debe estar habilitada en las cuentas con permisos de escritura en la suscripción|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: La autenticación debe estar habilitada en la aplicación web.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Los secretos deben tener las fechas de expiración establecidas.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de HTTP es la más reciente si se usa para ejecutar la aplicación de API.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: FTPS solo debe ser necesario en la aplicación de API.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de Java es la más reciente si se usa como parte de la aplicación de funciones.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Acceso a la aplicación web solo a través de HTTPS|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: La auditoría de SQL Server debe estar habilitada|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: MFA debe estar habilitada en las cuentas con permisos de propietario en la suscripción|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: La seguridad avanzada de datos debe estar habilitada en los servidores SQL Server|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: se debe habilitar Advanced Data Security en la instancia administrada de SQL.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: El control de acceso basado en roles (RBAC) debe usarse en los servicios de Kubernetes.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Supervisión de la falta de Endpoint Protection en Azure Security Center|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Los registros de diagnóstico de los servicios Search deben estar habilitados.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Los registros de diagnóstico de App Services deben estar habilitados.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Debe existir una alerta de registro de actividad para operaciones administrativas específicas (Microsoft.Network/networkSecurityGroups/delete). |Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Debe existir una alerta de registro de actividad para operaciones administrativas específicas (Microsoft.Network/networkSecurityGroups/securityRules/delete). |Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Debe existir una alerta de registro de actividad para operaciones administrativas específicas (Microsoft.Network/networkSecurityGroups/securityRules/write). |Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Debe existir una alerta de registro de actividad para operaciones administrativas específicas (Microsoft.Network/networkSecurityGroups/write). |Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Debe existir una alerta de registro de actividad para operaciones administrativas específicas (Microsoft.Sql/servers/firewallRules/delete). |Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Debe existir una alerta de registro de actividad para operaciones administrativas específicas (Microsoft.Sql/servers/firewallRules/write). |Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Solo se deben instalar las extensiones de VM aprobadas.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Es necesario habilitar Azure Defender para los registros de contenedor.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: La identidad administrada debe usarse en la aplicación de API.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: La autenticación debe estar habilitada en la aplicación de API.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Debe existir una alerta de registro de actividad para operaciones de directiva específicas (Microsoft.Authorization/policyAssignments/delete). |Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Debe existir una alerta de registro de actividad para operaciones de directiva específicas (Microsoft.Authorization/policyAssignments/write). |Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: La autenticación debe estar habilitada en la aplicación de funciones.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Los registros de diagnóstico de Data Lake Analytics deben estar habilitados.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Las cuentas de almacenamiento deben permitir el acceso desde los servicios de Microsoft de confianza.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Los registros de diagnóstico de Key Vault deben habilitarse.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: La opción "Aplicar conexión SSL" debe estar habilitada para los servidores de base de datos PostgreSQL.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Asegúrese de que la versión de HTTP es la más reciente si se usa para ejecutar la aplicación de funciones.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: MFA debe estar habilitada en las cuentas con permisos de lectura en la suscripción|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: El acceso de RDP desde Internet debe estar bloqueado.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: La opción "Aplicar conexión SSL" debe estar habilitada para los servidores de base de datos MySQL.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Asegúrese de que la aplicación de funciones tenga la opción "Certificados de cliente (certificados de cliente entrantes)" activada.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Los puntos de control de registro deben estar habilitados en los servidores de base de datos PostgreSQL.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Las conexiones de registro deben estar habilitadas en los servidores de base de datos PostgreSQL.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Las desconexiones deben estar registradas en los servidores de base de datos PostgreSQL.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: La evaluación de vulnerabilidades debe estar activada en sus servidores de SQL Server.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: La versión más reciente de TLS debe usarse en la aplicación web.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Las cuentas externas con permisos de propietario deben quitarse de la suscripción|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Los registros de diagnóstico de Service Bus deben estar habilitados.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: Los registros de diagnóstico de Azure Stream Analytics debe estar habilitados.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: La versión más reciente de TLS debe usarse en la aplicación de funciones.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Efecto para la directiva: La cuenta de almacenamiento que contiene el contenedor con los registros de actividad debe estar cifrada con BYOK.|Para obtener más información sobre los efectos, visite [https://aka.ms/policyeffects](../../policy/concepts/effects.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Si los registros de diagnóstico del conjunto de escalado de máquinas virtuales están habilitados, incluir los clústeres de AKS al realizar la auditoría||
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Versión de Java más reciente para App Services|Versión de Java más reciente admitida para App Services|
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Versión de Python más reciente para Linux para App Services|Versión de Python más reciente admitida para App Services|
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Lista de regiones en las que se debe habilitar Network Watcher|Para ver una lista completa de regiones, ejecute el comando Get-AzLocation de PowerShell.|
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Versión de PHP más reciente para App Services|Versión de PHP más reciente admitida para App Services|
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Período de retención requerido (días) para los registros de recursos|Para más información sobre los registros de recursos, visite [https://aka.ms/resourcelogs](../../../azure-monitor/essentials/resource-logs.md). |
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Nombre del grupo de recursos de Network Watcher|Nombre del grupo de recursos donde se encuentran las instancias de Network Watcher.|
|CIS Microsoft Azure Foundations Benchmark v1.3.0|Asignación de directiva|Configuración de auditoría necesaria de los servidores SQL Server||

## <a name="next-steps"></a>Pasos siguientes

Artículos adicionales sobre planos técnicos y cómo utilizarlos:

- Información acerca del [ciclo de vida del plano técnico](../concepts/lifecycle.md).
- Descubra cómo utilizar [parámetros estáticos y dinámicos](../concepts/parameters.md).
- Aprenda a personalizar el [orden de secuenciación de planos técnicos](../concepts/sequencing-order.md).
- Averigüe cómo usar el [bloqueo de recursos de planos técnicos](../concepts/resource-locking.md).
- Aprenda a [actualizar las asignaciones existentes](../how-to/update-existing-assignments.md).
