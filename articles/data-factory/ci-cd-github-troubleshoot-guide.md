---
title: Solución de problemas de CI/CD, Azure DevOps y GitHub en ADF
description: Use distintos métodos para solucionar problemas de CI/CD en ADF.
author: ssabat
ms.author: susabat
ms.reviewer: susabat
ms.service: data-factory
ms.topic: troubleshooting
ms.date: 04/27/2021
ms.openlocfilehash: e5745f195fe7620aeb7ffe009c13c52cd5f02e62
ms.sourcegitcommit: 49bd8e68bd1aff789766c24b91f957f6b4bf5a9b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/29/2021
ms.locfileid: "108228647"
---
# <a name="troubleshoot-ci-cd-azure-devops-and-github-issues-in-adf"></a>Solución de problemas de CI/CD, Azure DevOps y GitHub en ADF 

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

En este artículo vamos a explorar métodos habituales de solución de problemas de integración continua/implementación continua (CI/CD), Azure DevOps y GitHub en Azure Data Factory.

Si tiene preguntas o problemas en relación con el uso de las técnicas de DevOps o el control de código fuente, aquí encontrará algunos artículos que pueden resultarle útiles:

- Consulte [Control de código fuente en ADF](source-control.md) para obtener información sobre cómo se lleva a cabo el control de código fuente en ADF. 
- Consulte [CI/CD en ADF](continuous-integration-deployment.md) para obtener más información sobre cómo se lleva a cabo la integración continua/implementación continua de DevOps en ADF.
 
## <a name="common-errors-and-messages"></a>Errores habituales y mensajes

### <a name="connect-to-git-repository-failed-due-to-different-tenant"></a>Error de conexión al repositorio de Git debido a un inquilino distinto

#### <a name="issue"></a>Incidencia
    
A veces se producen problemas de autenticación, como el estado HTTP 401. Esto ocurre especialmente cuando tiene varios inquilinos con una cuenta de invitado, lo que puede complicar las cosas.

#### <a name="cause"></a>Causa

Según se observa, el token se ha obtenido del inquilino original, pero ADF está en el inquilino invitado e intenta usar el token para visitar DevOps en el inquilino invitado. Este no es el comportamiento esperado.

#### <a name="recommendation"></a>Recomendación

Debe usar el token emitido desde el inquilino invitado. Por ejemplo, tiene que asignar la misma instancia de Azure Active Directory para que sea su inquilino invitado y DevOps, de modo que pueda establecer correctamente el comportamiento del token y usar el inquilino correcto.

### <a name="template-parameters-in-the-parameters-file-are-not-valid"></a>Los parámetros de plantilla del archivo de parámetros no son válidos

#### <a name="issue"></a>Incidencia

Si eliminamos un desencadenador en la rama de desarrollo que ya está disponible en la rama de prueba o de producción con la **misma** configuración (por ejemplo, la frecuencia y el intervalo), la implementación de la canalización de versión se realiza correctamente y el desencadenador correspondiente se eliminará en los entornos respectivos. Sin embargo, si tiene **diferentes** configuraciones (como la frecuencia y el intervalo) para el desencadenador en los entornos de prueba o producción y elimina el mismo desencadenador en el entorno de desarrollo, se produce un error en la implementación.

#### <a name="cause"></a>Causa

Error de la canalización de CI/CD:

`
2020-07-20T11:19:02.1276769Z ##[error]Deployment template validation failed: 'The template parameters 'Trigger_Salesforce_properties_typeProperties_recurrence_frequency, Trigger_Salesforce_properties_typeProperties_recurrence_interval, Trigger_Salesforce_properties_typeProperties_recurrence_startTime, Trigger_Salesforce_properties_typeProperties_recurrence_timeZone' in the parameters file are not valid; they are not present in the original template and can therefore not be provided at deployment time. The only supported parameters for this template are 'factoryName, PlanonDWH_connectionString, PlanonKeyVault_properties_typeProperties_baseUrl
`

#### <a name="recommendation"></a>Recomendación

Este error se produce porque a menudo se elimina un desencadenador que está parametrizado, por lo que los parámetros no estarán disponibles en la plantilla de Azure Resource Manager (ARM) (dado que el desencadenador ya no existe). Dado que el parámetro ya no está en la plantilla de ARM, es necesario actualizar los parámetros invalidados en la canalización de DevOps. De lo contrario, cada vez que cambian los parámetros de la plantilla de ARM, deben actualizar los parámetros invalidados en la canalización de DevOps (en la tarea de implementación).

### <a name="updating-property-type-is-not-supported"></a>No se admite la actualización del tipo de propiedad

#### <a name="issue"></a>Incidencia

Error de la canalización de versión de CI/CD:

`
2020-07-06T09:50:50.8716614Z There were errors in your deployment. Error code: DeploymentFailed.
2020-07-06T09:50:50.8760242Z ##[error]At least one resource deployment operation failed. Please list deployment operations for details. Please see https://aka.ms/DeployOperations for usage details.
2020-07-06T09:50:50.8771655Z ##[error]Details:
2020-07-06T09:50:50.8772837Z ##[error]DataFactoryPropertyUpdateNotSupported: Updating property type is not supported.
2020-07-06T09:50:50.8774148Z ##[error]DataFactoryPropertyUpdateNotSupported: Updating property type is not supported.
2020-07-06T09:50:50.8775530Z ##[error]Check out the troubleshooting guide to see if your issue is addressed: https://docs.microsoft.com/azure/devops/pipelines/tasks/deploy/azure-resource-group-deployment#troubleshooting
2020-07-06T09:50:50.8776801Z ##[error]Task failed while creating or updating the template deployment.
`

#### <a name="cause"></a>Causa

Este error se debe a que en la fábrica de destino hay un entorno de ejecución de integración con el mismo nombre, pero de un tipo distinto. El entorno de ejecución de integración debe ser del mismo tipo durante la implementación.

#### <a name="recommendation"></a>Recomendación

- Consulte los siguientes procedimientos recomendados para CI/CD:

    https://docs.microsoft.com/azure/data-factory/continuous-integration-deployment#best-practices-for-cicd 
- Los entornos de ejecución de integración no cambian a menudo y son similares en todas las fases de CI/CD, por lo que Data Factory espera el mismo nombre y el mismo tipo de entorno de ejecución de integración en todas las fases de CI/CD. Si el nombre y los tipos y propiedades son diferentes, asegúrese de que las configuraciones del entorno de ejecución de integración de origen y de destino sean iguales y, a continuación, implemente la canalización de versión.
- Si quiere compartir entornos de ejecución de integración en todas las fases, considere la posibilidad de usar una factoría ternaria solo para contener los entornos de ejecución de integración compartidos. Puede usar esta factoría compartida en todos los entornos como un tipo de entorno de ejecución de integración vinculado.

### <a name="document-creation-or-update-failed-because-of-invalid-reference"></a>Error al crear o actualizar documentos debido a una referencia no válida

#### <a name="issue"></a>Incidencia

Al intentar publicar cambios en una instancia de Data Factory, aparece el mensaje de error siguiente:

`
"error": {
        "code": "BadRequest",
        "message": "The document creation or update failed because of invalid reference '<entity>'.",
        "target": "/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/<rgname>/providers/Microsoft.DataFactory/factories/<datafactory>/pipelines/<pipeline>",
        "details": null
    }
`
### <a name="cause"></a>Causa

Ha desasociado la configuración de Git y la ha configurado de nuevo con la marca de importar recursos seleccionada, que establece la instancia de Data Factory como sincronizada. Esto significa que no hay ningún cambio durante la publicación.

#### <a name="resolution"></a>Resolución

Desasocie la configuración de Git y vuelva a configurarla, asegurándose de NO activar la casilla de importar recursos existentes.

### <a name="data-factory-move-failing-from-one-resource-group-to-another"></a>No se puede mover una instancia de Data Factory de un grupo de recursos a otro

#### <a name="issue"></a>Incidencia

No se puede trasladar una instancia de Data Factory de un grupo de recursos a otro y se recibe el siguiente error:

`
{
    "code": "ResourceMoveProviderValidationFailed",
    "message": "Resource move validation failed. Please see details. Diagnostic information: timestamp 'xxxxxxxxxxxxZ', subscription id 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx', tracking id 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx', request correlation id 'xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx'.",
    "details": [
        {
            "code": "BadRequest",
            "target": "Microsoft.DataFactory/factories",
            "message": "One of the resources contain integration runtimes that are either SSIS-IRs in starting/started/stopping state, or Self-Hosted IRs which are shared with other resources. Resource move is not supported for those resources."
        }
    ]
}
`

#### <a name="resolution"></a>Resolución

Puede eliminar el entorno de ejecución de integración de SSIS y el entorno de ejecución de integración compartido para permitir la operación de traslado. Si no desea eliminar los entornos de ejecución de integración, lo mejor es seguir el documento de copia y clonación para realizar una copia y, tras ello, eliminar la antigua instancia de Data Factory.

###  <a name="unable-to-export-and-import-arm-template"></a>No se puede exportar ni importar la plantilla de ARM

#### <a name="issue"></a>Incidencia

No es posible exportar ni importar la plantilla de ARM. No se ha producido ningún error en el portal; sin embargo, en el seguimiento del explorador, puede ver el siguiente error:

`Failed to load resource: the server responded with a status code of 401 (Unauthorized)`

#### <a name="cause"></a>Causa

Ha creado un rol de cliente como usuario y no tenía el permiso necesario. Cuando la fábrica se carga en la interfaz de usuario, se comprueba una serie de valores de control de exposición para la fábrica. En este caso, el rol de acceso del usuario no tiene permiso para acceder a la API *queryFeaturesValue*. Para acceder a esta API, la característica de parámetros globales se desactiva. La ruta de acceso al código de exportación de ARM se basa en parte en la característica de parámetros globales.

#### <a name="resolution"></a>Resolución

Para resolver el problema, debe agregar el siguiente permiso al rol: *Microsoft.DataFactory/factories/queryFeaturesValue/action*. Este permiso se debe incluir de forma predeterminada en el rol "Colaborador de Data Factory".

###  <a name="cannot-automate-publishing-for-cicd"></a>No se puede automatizar la publicación de CI/CD 

#### <a name="cause"></a>Causa

Hasta hace poco, la única forma de publicar la canalización de ADF para implementaciones era usar el clic del botón del portal de ADF. Ahora, puede hacer que el proceso sea automático. 

#### <a name="resolution"></a>Solución

Se ha mejorado el proceso de CI/CD. La característica **Publicación automatizada** toma, valida y exporta todas las características de plantilla de ARM de la experiencia de usuario de ADF. Hace que la lógica se consuma a través de un paquete NPM disponible públicamente [@microsoft/azure-data-factory-utilities](https://www.npmjs.com/package/@microsoft/azure-data-factory-utilities). Este método le permite desencadenar mediante programación estas acciones en lugar de tener que ir a la interfaz de usuario de ADF y hacer clic en un botón. Este método proporciona a las canalizaciones de CI/CD una **verdadera** experiencia de integración continua. Siga las [mejoras de publicación de CI/CD de ADF](./continuous-integration-deployment-improvements.md) para obtener más información. 

###  <a name="cannot-publish-because-of-4-mb-arm-template-limit"></a>No se puede publicar debido a un límite de la plantilla de ARM de 4 MB.  

#### <a name="issue"></a>Problema

No se puede implementar porque se alcanzó el límite de tamaño total de 4 MB de la plantilla de Azure Resource Manager. Necesita una solución para implementar después de rebasar el límite. 

#### <a name="cause"></a>Causa

Azure Resource Manager restringe el tamaño de la plantilla a 4 MB. Limite el tamaño de la plantilla a 4 MB y cada archivo de parámetros a 64 KB. El límite de 4 MB se aplica al estado final de la plantilla una vez se ha ampliado con definiciones de recursos iterativas y los valores de variables y parámetros. Sin embargo, ha superado el límite. 

#### <a name="resolution"></a>Solución

En el caso de soluciones pequeñas o medianas, es más fácil entender y mantener una única plantilla. Puede ver todos los recursos y valores en un único archivo. Para los escenarios avanzados, las plantillas vinculadas le permiten desglosar la solución en componentes dirigidos. Siga los procedimientos recomendados en [Uso de plantillas vinculadas y anidadas](../azure-resource-manager/templates/linked-templates.md?tabs=azure-powershell).

### <a name="cannot-connect-to-git-enterprise"></a>No se puede establecer la conexión a GIT Enterprise  

##### <a name="issue"></a>Problema

No se puede conectar a GIT Enterprise debido a problemas de permisos. Puede ver un error como **422 - Entidad no procesable.**

#### <a name="cause"></a>Causa

* No ha configurado OAuth para ADF. 
* La dirección URL está mal configurada.

##### <a name="resolution"></a>Solución

Conceda acceso de OAuth a ADF al principio. Después, tiene que usar la dirección URL correcta para conectarse a GIT Enterprise. La configuración debe establecerse en las organizaciones del cliente. Por ejemplo, ADF probará *https://hostname/api/v3/search/repositories?q=user%3<customer credential>....* al principio y se producirán errores. A continuación, probará *https://hostname/api/v3/orgs/<org>/<repo>...* y la operación se realizará correctamente. 
 
### <a name="cannot-recover-from-a-deleted-data-factory"></a>No es posible la recuperación desde una factoría de datos eliminada.

#### <a name="issue"></a>Problema
El cliente eliminó la instancia de Data Factory o el grupo de recursos que contiene dicha instancia. Al cliente le gustaría saber cómo restaurar una factoría de datos eliminada.

#### <a name="cause"></a>Causa

Es posible recuperar la instancia de Data Factory solo si el cliente tiene configurado el control de código fuente (DevOps o Git). Esta acción proporcionará todos los recursos publicados más recientes y **no** restaurará la canalización, el conjunto de datos y el servicio vinculado sin publicar. Si no hay ningún control de código fuente, no es posible recuperar una instancia de Data Factory eliminada del back-end porque, una vez que el servicio recibe el comando deleted, se elimina la instancia y no se almacena ninguna copia de seguridad.

#### <a name="resolution"></a>Solución

Para recuperar la instancia de Data Factory eliminada que tiene el control de código fuente, consulte los pasos siguientes:

 * Cree una nueva instancia de Azure Data Factory.

 * Vuelva a configurar Git con los mismos valores, pero asegúrese de importar los recursos existentes de Data Factory en el repositorio seleccionado y elija Nueva rama.

 * Cree una solicitud de incorporación de cambios para combinar los cambios con la rama de colaboración y publíquela.

 * Si el cliente tenía un entorno de ejecución de integración autohospedado en el ADF eliminado, tendrá que crear una nueva instancia en el nuevo ADF, desinstalar y volver a instalar la instancia en su máquina local o VM con la nueva clave obtenida. Una vez completada la configuración del IR, el cliente deberá cambiar el servicio vinculado para que apunte al nuevo IR y probar la conexión, o se producirá el error **Referencia no válida**.

### <a name="cannot-deploy-to-different-stage-using-automatic-publish-method"></a>No se puede implementar en una fase diferente mediante el método de publicación automática

#### <a name="issue"></a>Problema
El cliente ha seguido todos los pasos necesarios, como instalar el paquete NPM y configurar una fase superior mediante Azure DevOps y ADF, pero la implementación no tiene lugar.

#### <a name="cause"></a>Causa

Aunque los paquetes de npm se pueden consumir de varias maneras, una de las ventajas principales es su consumo a través de una Canalización de Azure. En cada fusión mediante combinación en la rama de colaboración, se puede desencadenar una canalización que primero valide todo el código y, a continuación, exporte la plantilla de ARM a un artefacto de compilación, que una canalización de versión puede consumir. En la canalización inicial, el archivo YAML debe ser válido y estar completo.


#### <a name="resolution"></a>Solución

La siguiente sección no es válida porque la carpeta package.json no es válida.

```
- task: Npm@1
  inputs:
    command: 'custom'
    workingDir: '$(Build.Repository.LocalPath)/<folder-of-the-package.json-file>' #replace with the package.json folder
    customCommand: 'run build validate $(Build.Repository.LocalPath) /subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/testResourceGroup/providers/Microsoft.DataFactory/factories/yourFactoryName'
  displayName: 'Validate'
```
Debe incluir DataFactory en customCommand, como *"run build validate $(Build.Repository.LocalPath)/DataFactory/subscriptions/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/resourceGroups/testResourceGroup/providers/Microsoft.DataFactory/factories/nombreDeSuFábrica".* Asegúrese de que el archivo YAML generado para la fase superior tenga los artefactos JSON necesarios.


## <a name="next-steps"></a>Pasos siguientes

Para obtener más ayuda para solucionar problemas, pruebe los siguientes recursos:

*  [Blog de Data Factory](https://azure.microsoft.com/blog/tag/azure-data-factory/)
*  [Solicitud de características de Data Factory](https://feedback.azure.com/forums/270578-data-factory)
*  [Vídeos de Azure](https://azure.microsoft.com/resources/videos/index/?sort=newest&services=data-factory)
*  [Foro de Stack Overflow para Data Factory](https://stackoverflow.com/questions/tagged/azure-data-factory)
*  [Información de Twitter sobre Data Factory](https://twitter.com/hashtag/DataFactory)
