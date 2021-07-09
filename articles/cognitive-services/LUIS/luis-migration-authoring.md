---
title: Migración a una clave de creación de recursos de Azure
titleSuffix: Azure Cognitive Services
description: En este artículo se describe cómo migrar la autenticación de creación de Language Understanding (LUIS) desde una cuenta de correo electrónico a un recurso de Azure.
services: cognitive-services
author: aahill
ms.author: aahi
manager: nitinme
ms.custom: seodec18, contperf-fy21q2
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 05/28/2021
ms.openlocfilehash: cf40cea7e92cdb36f489943d5af2ef140bd92a9e
ms.sourcegitcommit: 20acb9ad4700559ca0d98c7c622770a0499dd7ba
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/29/2021
ms.locfileid: "110706912"
---
# <a name="migrate-to-an-azure-resource-authoring-key"></a>Migración a una clave de creación de recursos de Azure

> [!IMPORTANT]
>  A partir del 3 de diciembre de 2020, los usuarios existentes de LUIS deben haber completado el proceso de migración para continuar con la creación de aplicaciones de LUIS.

La autenticación de creación de Language Understanding (LUIS) ha cambiado de una cuenta de correo electrónico a un recurso de Azure. Use este artículo para obtener información sobre cómo migrar su cuenta, si aún no ha migrado.  


## <a name="what-is-migration"></a>¿Qué es la migración?

La migración es el proceso de cambiar la autenticación de creación de una cuenta de correo electrónico a un recurso de Azure. La cuenta se vinculará a una suscripción de Azure y un recurso de creación de Azure después de la migración.

La migración debe realizarse desde el [portal de LUIS](https://www.luis.ai). Por ejemplo, si crea las claves de creación con la CLI de LUIS, de todos modos tendrá que completar el proceso de migración en el portal de LUIS. Puede seguir teniendo coautores en las aplicaciones después de la migración, pero se agregarán en el nivel de recursos de Azure en lugar de en el nivel de aplicación. No se puede revertir la migración de la cuenta.

> [!Note]
> * Si necesita crear un recurso de predicción en tiempo de ejecución, hay [un proceso independiente](luis-how-to-azure-subscription.md#create-luis-resources) para hacerlo.
> * Consulte la sección [notas de migración](#migration-notes) a continuación para más información sobre cómo se verán afectadas las aplicaciones y los colaboradores. 
> * La creación de la aplicación de LUIS es gratuita, lo que está indicado por el nivel F0. [Más información sobre planes de tarifa](luis-limits.md#key-limits).

## <a name="migration-prerequisites"></a>Requisitos previos de la migración

* Una suscripción válida a Azure. Pídale al administrador de inquilinos que lo agregue a la suscripción, o bien [regístrese aquí para obtener una gratis](https://azure.microsoft.com/free/cognitive-services).
* Un recurso de creación de LUIS de Azure desde el portal de LUIS o desde [Azure Portal](https://portal.azure.com/#create/Microsoft.CognitiveServicesLUISAllInOne). 
    * La creación de un recurso de creación desde el portal de LUIS forma parte del proceso de migración que se describe en la sección siguiente.
* Si tiene rol de colaborador en las aplicaciones, estas no se migrarán automáticamente. Se le pedirá que exporte estas aplicaciones mientras completa el flujo de migración. También puede usar la [API de exportación](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5890b47c39e2bb052c5b9c40). Puede volver a importar la aplicación en LUIS después de la migración. El proceso de importación crea una aplicación nueva con un nuevo identificador de aplicación, para el que usted es el propietario.        
* Si es el propietario de la aplicación, no tendrá que exportar las aplicaciones, ya que se migrarán automáticamente. Se proporciona una plantilla de correo electrónico con una lista de todos los colaboradores de cada aplicación, por lo que se les puede notificar sobre el proceso de migración.

## <a name="migration-steps"></a>Pasos de migración

1. Al iniciar sesión en el [portal de LUIS](https://www.luis.ai), se abrirá una ventana de migración de Azure con los pasos para realizar la migración. Si la descarta, no podrá continuar con la creación de las aplicaciones de LUIS y la única acción que se mostrará será continuar con la migración.

    > [!div class="mx-imgBorder"]
    > ![Presentación de la ventana de migración](./media/migrate-authoring-key/notify-azure-migration.png)

2. Si tiene colaboradores en algunas de sus aplicaciones, verá una lista de los nombres de las aplicaciones que le pertenecen, junto con la región de creación y los correos electrónicos de los colaboradores de cada aplicación. Se recomienda enviar a los colaboradores un correo electrónico para notificarles sobre la migración. Para ello, haga clic en el botón del símbolo de **enviar** situado a la izquierda del nombre de la aplicación.
Si un colaborador tiene un recurso de predicción asignado a la aplicación, aparecerá un símbolo de `*` junto al nombre de la aplicación. Después de la migración, estas aplicaciones seguirán teniendo estos recursos de predicción asignados, aunque los colaboradores no tendrán acceso para crear las aplicaciones. Sin embargo, esta asignación se interrumpirá si el propietario del recurso de predicción [regeneró las claves](./luis-how-to-azure-subscription.md#regenerate-an-azure-key) a partir de Azure Portal.  

   > [!div class="mx-imgBorder"]
   > ![Notificación para los colaboradores](./media/migrate-authoring-key/notify-azure-migration-collabs.png)


   Para cada colaborador y aplicación, la aplicación de correo electrónico predeterminada se abre con un correo electrónico ligeramente formateado. Puede editar el correo electrónico antes de enviarlo. La plantilla de correo electrónico incluye el id. de la aplicación y el nombre de la aplicación exactos.

   ```html
   Dear Sir/Madam,

   I will be migrating my LUIS account to Azure. Consequently, you will no longer have access to the following app:

   App Id: <app-ID-omitted>
   App name: Human Resources

   Thank you
   ```
   > [!Note]
   > Una vez que haya migrado la cuenta a Azure, las aplicaciones dejarán de estar disponibles para los colaboradores.

3. Si es colaborador en alguna aplicación, se muestra una lista de los nombres de aplicación compartidas con usted, junto con los correos electrónicos del propietario y de la región de creación en cada aplicación. Se recomienda exportar una copia de las aplicaciones haciendo clic en el botón Exportar situado a la izquierda del nombre de la aplicación. Puede volver a importar estas aplicaciones después de migrar, porque no se migrarán automáticamente con usted.
Si tiene un recurso de predicción asignado a una aplicación, aparecerá un símbolo de `*` junto al nombre de la aplicación. Después de la migración, el recurso de predicción seguirá asignándose a estas aplicaciones, aunque ya no tendrá acceso para crear estas aplicaciones. Si quiere interrumpir la asignación entre el recurso de predicción y la aplicación, deberá ir a Azure Portal y [regenerar las claves](./luis-how-to-azure-subscription.md#regenerate-an-azure-key).

   > [!div class="mx-imgBorder"]
   > ![Exportación de la aplicaciones.](./media/migrate-authoring-key/migration-export-apps.png)


4. En la ventana para migrar regiones, se le pedirá que migre las aplicaciones a un recurso de Azure en la misma región en la que se crearon. LUIS tiene tres regiones [y portales](./luis-reference-regions.md#luis-authoring-regions) de creación. La ventana mostrará las regiones en las que se crearon las aplicaciones de su propiedad. Las regiones de migración mostradas pueden variar en función del portal regional que use y de las aplicaciones que haya creado. 

   > [!div class="mx-imgBorder"]
   > ![Migración en varias regiones.](./media/migrate-authoring-key/migration-regional-flow.png)

5. Para cada región, elija crear un nuevo recurso de creación de LUIS o migrar a uno existente mediante los botones.

   > [!div class="mx-imgBorder"]
   > ![Elección para crear un recurso de creación o usar uno existente](./media/migrate-authoring-key/migration-multiregional-resource.png)

   Proporcione la siguiente información:

   * **Nombre del inquilino**: el inquilino con el que está asociada la suscripción de Azure. De manera predeterminada, esta opción está establecida en el inquilino que se usa actualmente. Para cambiar de inquilinos, cierre esta ventana y seleccione el avatar en la parte superior derecha de la pantalla, que contiene sus iniciales. Haga clic en **Migrar a Azure** para volver a abrir la ventana.
   * **Nombre de la suscripción de Azure**: la suscripción que se asociará con el recurso. Si tiene más de una suscripción que pertenece a su inquilino, seleccione la que desee en la lista desplegable.
   * **Nombre del recurso de creación**: el nombre personalizado que elija. Se usa como parte de la dirección URL para las consultas de punto de conexión de creación y predicción. Si va a crear un nuevo recurso de creación, asegúrese de que su nombre solo incluya caracteres alfanuméricos o el carácter `-`, y que no comienza ni termina por `-`. Si se incluye algún otro símbolo en el nombre, se producirá un error al crear el recurso.
   * **Nombre del grupo de recursos de Azure**: nombre del grupo de recursos personalizado que elija de la lista desplegable. Los grupos de recursos le permiten agrupar los recursos de Azure para el acceso y la administración. Si actualmente no tiene ningún grupo de recursos en la suscripción, no se le permitirá crear uno en el portal de LUIS. Vaya a [Azure Portal](https://ms.portal.azure.com/#create/Microsoft.ResourceGroup) para crear uno y vuelva a LUIS para continuar con el proceso de inicio de sesión.

6. Una vez que haya migrado correctamente en todas las regiones, haga clic en Finalizar. Ahora dispondrá de acceso a las aplicaciones. Puede continuar con la creación y el mantenimiento de todas las aplicaciones en todas las regiones del portal.

## <a name="migration-notes"></a>Notas de migración

* Antes de la migración, los coautores se conocen como _colaboradores_ en el nivel de aplicación de LUIS. Después de la migración, el rol de _colaborador_ de Azure se usa para la misma funcionalidad en el nivel de recurso de Azure.
* Si ha iniciado sesión en más de un [portal regional de LUIS](./luis-reference-regions.md#luis-authoring-regions), se le pedirá que migre en varias regiones a la vez.
* Las aplicaciones se migrarán automáticamente si usted es el propietario de la aplicación. Las aplicaciones no se migrarán con usted si es colaborador en la aplicación. Sin embargo, se solicitará a los colaboradores que exporten las aplicaciones que necesiten.
* Los propietarios de aplicaciones no pueden elegir un subconjunto de las aplicaciones para su migración y no hay ninguna manera de que un propietario sepa si los colaboradores han realizado alguna migración.
* La migración no mueve ni agrega automáticamente colaboradores al recurso de creación de Azure. El propietario de la aplicación es el que necesita completar este paso después de la migración. Este paso requiere [permisos para el recurso de creación de Azure](./luis-how-to-collaborate.md).
* Una vez que se hayan asignado colaboradores al recurso de Azure, deberán migrarse para poder tener acceso a las aplicaciones. De lo contrario, no tendrán acceso para crear las aplicaciones.


## <a name="using-apps-after-migration"></a>Uso de aplicaciones después de la migración

Después del proceso de migración, todas las aplicaciones de LUIS que sean de su propiedad estarán asignadas ahora a un único recurso de creación de LUIS.
La lista **Aplicaciones** muestra las aplicaciones que se migraron al recurso de creación nuevo. Antes de obtener acceso a las aplicaciones, seleccione **Elegir otro recurso de creación** para seleccionar el recurso de creación y la suscripción para ver las aplicaciones que se pueden crear.

> [!div class="mx-imgBorder"]
> ![Selección de la suscripción y el recurso de creación](./media/migrate-authoring-key/select-sub-and-resource.png)


Si tiene previsto editar las aplicaciones mediante programación, necesitará los valores de clave de creación. Estos valores se muestran al hacer clic en **Administrar** en la parte superior de la pantalla del portal de LUIS y, a continuación, seleccionar **Recursos de Azure**. También están disponibles en la página **Clave y puntos de conexión** del recurso en Azure Portal. También puede crear más recursos de creación y asignarlos desde la misma página.

## <a name="adding-contributors-to-authoring-resources"></a>Incorporación de colaboradores a recursos de creación

[!INCLUDE [Manage contributors for the Azure authoring resource for language understanding](./includes/manage-contributors-authoring-resource.md)]

Aprenda a [agregar colaboradores](luis-how-to-collaborate.md) en el recurso de creación. Los colaboradores tendrán acceso a todas las aplicaciones de ese recurso.

Puede agregar colaboradores al recurso de creación desde Azure Portal, en la página **Access Control (IAM)** correspondiente a ese recurso. Para más información, consulte la sección sobre cómo [agregar colaboradores a una aplicación](luis-how-to-collaborate.md).

> [!Note]
> Si el propietario de la aplicación de LUIS ha realizado la migración y ha agregado el usuario como colaborador en el recurso de Azure, el colaborador seguirá sin tener acceso a la aplicación, salvo que también migre.

## <a name="troubleshooting-the-migration-process"></a>Solución de problemas del proceso de migración

Si no encuentra la suscripción de Azure en la lista desplegable:
* Asegúrese de tener una suscripción válida de Azure que tenga autorización para crear recursos de Cognitive Services. Vaya a [Azure Portal](https://ms.portal.azure.com) y compruebe el estado de la suscripción. En caso de no tener ninguna, [cree una cuenta gratuita de Azure](https://azure.microsoft.com/free/cognitive-services/).
* Asegúrese de que se encuentra en el inquilino adecuado asociado a su suscripción válida. Para cambiar de inquilinos, seleccione el avatar en la parte superior derecha de la pantalla, que contiene sus iniciales.

  > [!div class="mx-imgBorder"]
  > ![Página para cambiar de directorio](./media/migrate-authoring-key/switch-directories.png)

Si tiene un recurso de creación existente, pero no puede encontrarlo al seleccionar la opción **Use Existing Authoring Resource** (Usar el recurso de creación existente):
* Probablemente, el recurso se creó en una región distinta a la que está intentando migrar.
* En su lugar, cree un recurso nuevo desde el portal de LUIS.

Si selecciona la opción **Create a new authoring resource** (Crear un nuevo recurso de creación) y se produce un error en la migración con el mensaje "Failed retrieving user's Azure information, retry again later" (Error al recuperar la información de Azure del usuario, vuelva a intentarlo más tarde):
* Es posible que la suscripción tenga 10 o más recursos de creación por región y suscripción. En ese caso, no podrá crear un nuevo recurso de creación.
* Migre seleccionando la opción **Use Existing Authoring Resource** (Usar el recurso de creación existente) y seleccione uno de los recursos existentes en su suscripción.

## <a name="create-new-support-request"></a>Creación de una solicitud de soporte técnico

Si tiene algún problema con la migración que no se aborda en la sección de solución de problemas, [cree un tema de soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest) y proporcione la información siguiente con los campos siguientes:

   * **Tipo de problema**: Requisitos previos técnicos
   * **Suscripción**: Elija una suscripción en la lista desplegable.
   * **Servicio**: Busque y seleccione "Cognitive Services".
   * **Recursos**: Elija un recurso de LUIS si ya existe uno. Si no es así, seleccione Pregunta general.

## <a name="next-steps"></a>Pasos siguientes

* Revise [Conceptos sobre las claves de creación y en tiempo de ejecución](luis-how-to-azure-subscription.md).
* Revise cómo [asignar claves](luis-how-to-azure-subscription.md) y [agregar colaboradores](luis-how-to-collaborate.md).
