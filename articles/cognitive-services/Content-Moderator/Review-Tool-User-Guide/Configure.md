---
title: 'Configuración de la herramienta de revisión: Content Moderator'
titleSuffix: Azure Cognitive Services
description: Use la herramienta de revisión para configurar o recuperar su equipo, etiquetas, conectores, flujos de trabajo y credenciales de Content Moderator.
services: cognitive-services
author: PatrickFarley
manager: mikemcca
ms.service: cognitive-services
ms.subservice: content-moderator
ms.topic: conceptual
ms.date: 03/15/2019
ms.author: pafarley
ms.openlocfilehash: 4e3ed696bba0a2d739e87f465561c156d6752f23
ms.sourcegitcommit: c05e595b9f2dbe78e657fed2eb75c8fe511610e7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112033948"
---
# <a name="configure-the-review-tool"></a>Configuración de la herramienta de revisión

[!INCLUDE [deprecation notice](../includes/tool-deprecation.md)]


La [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com) presenta varias características importantes a las que puede acceder mediante el menú **Settings** (Configuración) del panel.

![Menú de configuración de la herramienta de revisión de Content Moderator](images/settings-1.png)

## <a name="manage-team-and-subteams"></a>Administración de equipo y subequipos

La pestaña **Team** (Equipo) le permite administrar el equipo y los subequipos&mdash;grupos de usuarios que pueden recibir una notificación cuando se inicien determinadas [revisiones humanas](../review-api.md#reviews). Solo puede tener un equipo (que se crea al registrarse con la herramienta de revisión), pero puede crear varios subequipos. El administrador del equipo puede invitar a miembros, establecer sus permisos y asignarlos a distintos subequipos.

![Configuración de equipos de la herramienta de revisión](images/settings-2-team.png)

Los subequipos son útiles para crear equipos de escalamiento o equipos dedicados a revisar categorías específicas de contenido. Por ejemplo, podría enviar contenido para adultos a otro equipo para que lo revisen más a fondo.

En esta sección se explica cómo crear subequipos y asignar rápidamente revisiones sobre la marcha. Sin embargo, puede usar [flujos de trabajo](workflows.md) para asignar revisiones según criterios específicos.

### <a name="create-a-subteam"></a>Creación de un subequipo

Vaya a la sessión **Subteams** (Subequipos) y haga clic en **Add Subteam** (Agregar subequipo) Escriba el nombre del subequipo en el cuadro de diálogo y haga clic en **Save** (Guardar).

![Nombre del subequipo](images/1-Teams-2.PNG)

#### <a name="invite-teammates"></a>Invitación a compañeros de equipo

No puede asignar a alguien a un subequipo si no es ya miembro del equipo predeterminado, así que primero debe agregar revisores al equipo predeterminado. Haga clic en **Invite** (Invitar) en la pestaña **Team** (Equipo).

![Invitar a usuarios](images/invite-users.png)

#### <a name="assign-teammates-to-subteam"></a>Asignación de los miembros del equipo al subequipo

Haga clic en el botón **Add Member** (Agregar miembro) para asignar miembros del equipo predeterminado a uno o varios subequipos. Solo puede agregar usuarios existentes a un subequipo. Para agregar nuevos usuarios que no están en la herramienta de revisión, invíteles mediante el botón "Invite" (Invitar) en la página de configuración del equipo.

![Asignar miembros del subequipo](images/1-Teams-3.PNG)

### <a name="assign-reviews-to-subteams"></a>Asignación de revisiones a los subequipos

Después de haber creado sus subequipos y haber asignado los miembros, puede comenzar a asignar [revisiones](../review-api.md#reviews) de contenido a esos subequipos. Para ello se usa la pestaña **Review** (Revisar) del sitio.
Para asignar contenido a un subequipo, haga clic en el botón de puntos suspensivos en la esquina superior derecha, seleccione **Move to** (Mover a) y seleccione un subequipo.

![Asignar una revisión de imagen a un subequipo](images/3-review-image-subteam-1.png)

### <a name="switch-between-subteams"></a>Cambiar entre subequipos

Si es miembro de más de un subequipo, puede cambiar entre esos subequipos y modificar las revisiones de contenido que se muestran al usuario. En la pestaña **Review** (Revisar), seleccione el menú desplegable llamado **Default** (Predeterminado) y seleccione **Choose Subteam** (Elegir subequipo). Puede ver las revisiones de contenido de subequipos diferentes, pero solo si es miembro de ellos.

![Cambiar entre subequipos](images/3-review-image-subteam-2.png)

## <a name="tags"></a>Etiquetas

La pestaña **Tags** (Etiquetas) le permite definir etiquetas de moderación personalizadas, que se suman a las dos etiquetas de moderación predeterminadas&mdash;**isadult** (**a**) y **isracy** (**r**). Cuando se crea una etiqueta personalizada, se vuelve disponible en las revisiones junto con las etiquetas predeterminadas. Puede cambiar qué etiquetas mostrar en las revisiones cambiando su configuración de visibilidad.

![Vista de etiquetas, que incluye las casillas "Is visible" (Está visible)](images/tags-4-disable.png)

### <a name="create-custom-tags"></a>Creación de etiquetas personalizadas

Para crear una etiqueta, debe escribir un código corto, el nombre y la descripción en los campos correspondientes.

- **Short code** (Código corto): escriba un código de dos letras para la etiqueta. Por ejemplo, **cb**
- **Name**: Escriba un nombre de etiqueta corto y descriptivo en minúsculas, sin espacios. Por ejemplo, **isbullying**.
- **Description** (Descripción): (opcional) escriba una descripción del tipo de contenido que identifica la etiqueta. Ejemplo: **Representaciones o ejemplos de ciberacoso**.

Haga clic en **Add** (Agregar) para agregar una etiqueta y haga clic en **Save** (Guardar) cuando haya terminado de crear etiquetas.

![Cuadro de diálogo de creación de etiquetas de la herramienta de revisión](images/settings-3-tags.png)

### <a name="delete-tags"></a>Eliminación de etiquetas

Puede eliminar etiquetas personalizadas son solo seleccionar el icono de Papelera junto a sus entradas en la lista de etiquetas, pero no puede eliminar las etiquetas predeterminadas.

## <a name="connectors"></a>Conectores

La pestaña **Connectors** (Conectores) le permite administrar los conectores, que son complementos específicos del servicio que pueden procesar contenido de distintas maneras como parte de los [flujos de trabajo](../review-api.md#workflows) de contenido.

El conector predeterminado al crear un flujo de trabajo es el conector de Content Moderator, que puede marcar contenido como **adulto** o **subido de tono**, buscar palabras soeces, etc. Sin embargo, puede usar otros conectores, aquí enumerados, siempre que tenga credenciales para sus respectivos servicios (para usar el conector de Face, por ejemplo, deberá obtener una clave de suscripción de [Face](../../face/overview.md)).

La [herramienta de revisión](./human-in-the-loop.md) incluye los conectores siguientes:

- Emotion
- Caras
- PhotoDNA Cloud Service
- Text Analytics

### <a name="add-a-connector"></a>Agregar un conector

Para agregar un conector (y que esté disponible para su uso en los [flujos de trabajo](../review-api.md#workflows)) de contenido, seleccione el botón **Connect** (Conectar) adecuado. En el cuadro de diálogo siguiente, escriba la clave de suscripción para ese servicio. Cuando haya terminado, el nuevo conector debe aparecer en la parte superior de la página.

![Configuración de conectores de Content Moderator](images/settings-4-connectors.png)

## <a name="workflows"></a>Workflows

La pestaña **Workflows** (Flujos de trabajo) le permite administrar sus [flujos de trabajo](../review-api.md#workflows). Los flujos de trabajo son filtros basados en la nube del contenido, y funcionan con conectores para ordenarlo de diversas maneras y realizar las acciones adecuadas. Aquí, puede definir, editar y probar los flujos de trabajo. Consulte [Definición y uso de los flujos de trabajo](Workflows.md) para guiarle en este procedimiento.

![Configuración de flujos de trabajo de Content Moderator](images/settings-5-workflows.png)

## <a name="credentials"></a>Credenciales

La pestaña **Credentials** (Credenciales) proporciona acceso rápido a su clave de suscripción de Content Moderator, que necesitará para acceder a cualquiera de los servicios de moderación desde una llamada de REST o el SDK de cliente.

![Credenciales de Content Moderator](images/settings-6-credentials.png)

### <a name="use-external-credentials-for-workflows"></a>Uso de credenciales externas para flujos de trabajo

La [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com) genera una clave de evaluación gratuita para los servicios de Azure Content Moderator cuando se registra, pero también puede configurarlo para usar una clave existente de su cuenta de Azure. Esta opción se recomienda para escenarios a gran escala, ya que las claves de evaluación gratuita poseen límites de uso estrictos ([Precios y límites](https://azure.microsoft.com/pricing/details/cognitive-services/content-moderator/)).

Si ha creado un [recurso de Content Moderator](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesContentModerator) en Azure, diríjase a él en Azure Portal y seleccione la hoja **Claves**. Copie una de las claves.

![Claves de Content Moderator en Azure Portal](images/credentials-azure-portal-keys.PNG)

En la pestaña **Credentials** (Credenciales) de la [herramienta de revisión](https://contentmoderator.cognitive.microsoft.com), vaya al panel **Workflow Settings** (Configuración del flujo de trabajo), seleccione **Edit** (Editar) y pegue la clave en el campo **Ocp-Apim-Subscription-Key**. Ahora, los flujos de trabajo que llaman a las API de moderación usarán su credencial de Azure.

> [!NOTE]
> Los otros dos campos del panel **Workflow Settings** (Configuración del flujo de trabajo) son para listas de términos e imágenes personalizadas. Consulte las guías de [términos personalizado](../try-terms-list-api.md) o [imágenes personalizadas](../try-image-list-api.md) para más información al respecto.

### <a name="use-your-azure-account-with-the-review-apis"></a>Uso de la cuenta de Azure con las API de revisión

Para usar la clave de Azure con las API de revisión, deberá recuperar su identificador de recurso. Vaya al recurso de Content Moderator en Azure Portal y seleccione la hoja **Propiedades**. Copie el valor del identificador de recurso y péguelo en el campo **Whitelisted Resource Id(s)** (Identificador de recursos en lista de permitidos) de la pestaña **Credentials** (Credenciales).

![Id. de recurso de Content Moderator en Azure Portal](images/credentials-azure-portal-resourceid.PNG)

Si ha especificado la clave de suscripción en ambos lugares, no se usará la clave de prueba que se incluye con su cuenta de la herramienta de revisión, pero seguirá estando disponible.

## <a name="next-steps"></a>Pasos siguientes

Siga el [inicio rápido de la herramienta de revisión](../quick-start.md) para empezar a usar la herramienta de revisión en escenarios de moderación de contenido.