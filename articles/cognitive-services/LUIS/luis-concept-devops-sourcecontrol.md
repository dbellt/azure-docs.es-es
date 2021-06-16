---
title: 'Ramas de control de código fuente y desarrollo: LUIS'
description: Cómo mantener la aplicación Language Understanding (LUIS) bajo control de código fuente. Cómo aplicar las actualizaciones a una aplicación de LUIS al trabajar en una rama de desarrollo.
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 11/18/2020
ms.openlocfilehash: b860510610330d6c5d60f41dccc9e82bc77685f5
ms.sourcegitcommit: c072eefdba1fc1f582005cdd549218863d1e149e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/10/2021
ms.locfileid: "111969435"
---
# <a name="devops-practices-for-luis"></a>Prácticas de DevOps para LUIS

Los ingenieros de software que desarrollan una aplicación de Language Understanding (LUIS) pueden aplicar prácticas de DevOps en torno al [control de código fuente](luis-concept-devops-sourcecontrol.md), las [compilaciones automatizadas](luis-concept-devops-automation.md), las [pruebas](luis-concept-devops-testing.md) y la [administración de versiones](luis-concept-devops-automation.md#release-management) según estas instrucciones.

## <a name="source-control-and-branch-strategies-for-luis"></a>Control de código fuente y estrategias de ramificación para LUIS

Uno de los factores clave de los que depende el éxito de DevOps es el [control de código fuente](/azure/devops/user-guide/source-control). Un sistema de control de código fuente permite a los desarrolladores colaborar en el código y realizar el seguimiento de los cambios. El uso de ramas permite a los desarrolladores cambiar entre diferentes versiones de la base de código y trabajar de forma independiente de otros miembros del equipo. Cuando los desarrolladores generan una [solicitud de incorporación de cambios](https://help.github.com/github/collaborating-with-issues-and-pull-requests/about-pull-requests) (PR) para proponer actualizaciones de una rama a otra, o cuando se combinan los cambios, pueden ser el desencadenador de [compilaciones automatizadas](luis-concept-devops-automation.md) para compilar y probar continuamente el código.

Mediante los conceptos y las instrucciones que se describen en este documento, puede desarrollar una aplicación de LUIS mientras realiza el seguimiento de los cambios en un sistema de control de código fuente y seguir estos procedimientos recomendados de ingeniería de software:

- **Control de código fuente**
  - El código fuente de la aplicación de LUIS está en un formato legible.
  - El modelo se puede compilar a partir del código fuente de forma repetible.
  - El código fuente se puede administrar mediante un repositorio de código fuente.
  - Las credenciales y los secretos, como las claves de creación y suscripción, nunca se almacenan en el código fuente.

- **Rama y combinación**
  - Los desarrolladores pueden trabajar desde ramas independientes.
  - Los desarrolladores pueden trabajar en varias ramas simultáneamente.
  - Es posible integrar los cambios en una aplicación de LUIS de una rama en otra a través de la fusión mediante cambio de base o la combinación.
  - Los desarrolladores pueden combinar una solicitud de incorporación de cambios con la rama primaria.

- **Control de versiones**
  - Cada componente de una aplicación de gran tamaño debe tener versiones independientes, lo que permite a los desarrolladores detectar cambios o actualizaciones importantes simplemente con la consulta del número de versión.

- **Revisiones de código**
  - Los cambios en la solicitud de incorporación de cambios se presentan como código fuente en lenguaje natural que puede revisarse antes de aceptar dicha solicitud.

## <a name="source-control"></a>Control de código fuente

Para mantener la [definición del esquema de la aplicación](./app-schema-definition.md) de una aplicación de LUIS en un sistema de administración de código fuente, use la representación del [formato LUDown (`.lu`)](/azure/bot-service/file-format/bot-builder-lu-file-format) de la aplicación. El formato `.lu` se prefiere al formato `.json` porque es legible, lo que facilita la creación y revisión de cambios en las solicitudes de incorporación de cambios.

### <a name="save-a-luis-app-using-the-ludown-format"></a>Almacenamiento de una aplicación de LUIS con el formato LUDown

Para guardar una aplicación de LUIS en formato `.lu` y colocarla bajo control de código fuente:

- OPCIONES: [Exporte la versión de la aplicación](./luis-how-to-manage-versions.md#other-actions) como `.lu` desde el [portal de LUIS](https://www.luis.ai/) y agréguela al repositorio de control de código fuente.

- O: Utilice un editor de texto para crear un archivo `.lu` para una aplicación de LUIS y agréguelo al repositorio de control de código fuente.

> [!TIP]
> Si trabaja con la exportación de JSON de una aplicación de LUIS, puede [convertirla al formato LUDown](https://github.com/microsoft/botframework-cli/tree/master/packages/luis#bf-luisconvert).  Use la opción `--sort` para asegurarse de que las intenciones y las expresiones estén ordenadas alfabéticamente.  
> Tenga en cuenta que la funcionalidad de exportación de **.LU** integrada en el portal de LUIS ya ordena la salida.

### <a name="build-the-luis-app-from-source"></a>Compilación de la aplicación de LUIS a partir de código fuente

En el caso de una aplicación de LUIS, *compilar a partir del código fuente* significa [crear una nueva versión de la aplicación de LUIS mediante la importación del código fuente de `.lu`](./luis-how-to-manage-versions.md#import-version), [entrenar la versión](./luis-how-to-train.md) y [publicarla](./luis-how-to-publish-app.md). Puede hacerlo en el portal de LUIS o en la línea de comandos:

- Use el portal de LUIS para [importar la versión `.lu`](./luis-how-to-manage-versions.md#import-version) de la aplicación del control de código fuente, y para [entrenar](./luis-how-to-train.md) y [publicar](./luis-how-to-publish-app.md) la aplicación.

- Use la [interfaz de la línea de comandos de Bot Framework para LUIS](https://github.com/microsoft/botbuilder-tools/tree/master/packages/LUIS) en la línea de comandos o en un flujo de trabajo de CI/CD para [importar](https://github.com/microsoft/botframework-cli/blob/master/packages/luis/README.md#bf-luisversionimport) la versión `.lu` de la aplicación del control de código fuente en una aplicación de LUIS, y para [entrenar](https://github.com/microsoft/botframework-cli/blob/master/packages/luis/README.md#bf-luistrainrun) y [publicar](https://github.com/microsoft/botframework-cli/blob/master/packages/luis/README.md#bf-luisapplicationpublish) la aplicación.

### <a name="files-to-maintain-under-source-control"></a>Archivos que se van a mantener bajo control de código fuente

Los siguientes tipos de archivos para la aplicación de LUIS se deben mantener bajo control de código fuente:

- Archivo `.lu` para la aplicación de LUIS

- [Archivos de definición de pruebas unitarias](luis-concept-devops-testing.md#writing-tests) (expresiones y resultados esperados)

- [Archivos de prueba por lotes](./luis-how-to-batch-test.md#batch-test-file) (expresiones y resultados esperados) usados para las pruebas de rendimiento

### <a name="credentials-and-keys-are-not-checked-in"></a>Las credenciales y las claves no están insertadas en el repositorio

No incluya claves de suscripción ni valores confidenciales similares en los archivos que inserte en el repositorio, donde personal no autorizado pueda verlos. Entre las claves y otros valores que debe evitar insertar en el repositorio se incluyen:

- Claves de creación y predicción de LUIS
- Puntos de conexión de creación y predicción de LUIS
- Claves de suscripción a Azure
- Tokens de acceso, como el token de una [entidad de servicio](/cli/azure/ad/sp) de Azure, que se usan para la autenticación de la automatización.

#### <a name="strategies-for-securely-managing-secrets"></a>Estrategias para administrar secretos de forma segura

Las estrategias para administrar secretos de forma segura incluyen:

- Si utiliza el control de versiones de Git, puede almacenar secretos en tiempo de ejecución en un archivo local y agregar un patrón que coincida con el nombre de archivo en un archivo [.gitignore](https://git-scm.com/docs/gitignore) para evitar la inserción del archivo en el repositorio.
- En un flujo de trabajo de automatización, puede almacenar secretos de forma segura en la configuración de parámetros que ofrece esa tecnología de automatización. Por ejemplo, si usa [Acciones de GitHub](https://github.com/features/actions), puede almacenar secretos de forma segura en [secretos de GitHub](https://help.github.com/en/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets).

## <a name="branching-and-merging"></a>Rama y combinación

Los sistemas de control de versiones distribuidos como Git proporcionan flexibilidad a loso miembros del equipo para publicar, compartir, revisar e iterar los cambios de código a través de las ramas de desarrollo compartidas con otros usuarios. Adopte una [estrategia de rama de Git](/azure/devops/repos/git/git-branching-guidance) que sea adecuada para su equipo.

Sea cual sea la estrategia de rama que adopte, un principio clave para todas es que los miembros del equipo pueden trabajar en la solución dentro de una *rama de características* independientemente del trabajo que se esté llevando a cabo en otras ramas.

Para permitir el trabajo independiente en ramas con un proyecto de LUIS:

- **La rama principal tiene su propia aplicación LUIS.** Esta aplicación representa el estado actual de la solución para el proyecto, y su versión activa actual siempre debe estar asignada al código fuente de `.lu` que se encuentra en la rama principal. Todas las actualizaciones del código fuente de `.lu` para esta aplicación deben revisarse y probarse para que esta aplicación se pueda implementar en entornos de compilación como el de producción en cualquier momento. Si las actualizaciones de `.lu` se combinan en la rama principal de una rama de características, debe crear una nueva versión en la aplicación de LUIS e [incrementar el número de versión](#versioning).

- **Cada rama de características debe usar su propia instancia de una aplicación de LUIS**. Los desarrolladores trabajan con esta aplicación en una rama de características sin riesgo de que ello afecte a los desarrolladores que trabajan en otras ramas. Esta aplicación de "rama de desarrollo" es una copia de trabajo que se debe eliminar al eliminar la rama de características.

![Rama de características de Git](./media/luis-concept-devops-sourcecontrol/feature-branch.png)

### <a name="developers-can-work-from-independent-branches"></a>Los desarrolladores pueden trabajar desde ramas independientes

Los desarrolladores pueden trabajar en actualizaciones en una aplicación de LUIS independientemente de otras ramas mediante las siguientes acciones:

1. Crear una rama de características desde la rama principal (en función de la estrategia de la rama, normalmente maestra o de desarrollo).

1. [Crear una nueva aplicación de LUIS en el portal de LUIS](./luis-how-to-start-new-app.md) ("*aplicación de rama de desarrollo*") exclusivamente para admitir el trabajo en la rama de características.

   * Si el código fuente de `.lu` de la solución ya existe en la rama, porque se guardó después del trabajo realizado en otra rama anteriormente en el proyecto, importe el archivo `.lu` para crear la aplicación de rama de desarrollo de LUIS.

   * Si va a empezar a trabajar en un proyecto nuevo, todavía no tendrá el código fuente de `.lu` de la aplicación de LUIS principal en el repositorio. Para crear el archivo `.lu`, exporte la aplicación de rama de desarrollo desde el portal cuando haya completado el trabajo de la rama de características y envíelo como parte de la solicitud de incorporación de cambios.

1. Trabajar en la versión activa de la aplicación de rama de desarrollo para implementar los cambios necesarios. Se recomienda trabajar solo en una versión única de la aplicación de rama de desarrollo para todo el trabajo de la rama de características. Si crea más de una versión en la aplicación de rama desarrollo, procure realizar un seguimiento de la versión que contiene los cambios que quiere insertar en el repositorio al presentar la solicitud de incorporación de cambios.

1. Probar las actualizaciones. Consulte [Pruebas de DevOps para LUIS](luis-concept-devops-testing.md) para más información sobre cómo probar la aplicación de rama de desarrollo.

1. Exportar la versión activa de la aplicación de rama de desarrollo como `.lu` desde la [lista de versiones](./luis-how-to-manage-versions.md).

1. Insertar las actualizaciones en el repositorio e invitar a la revisión por homólogos de las actualizaciones. Si usa GitHub, generará una [solicitud de incorporación de cambios](https://help.github.com/en/github/collaborating-with-issues-and-pull-requests/about-pull-requests).

1. Cuando se aprueben los cambios, combine las actualizaciones en la rama principal. En este momento, creará una nueva [versión](./luis-how-to-manage-versions.md) de la aplicación de LUIS *principal* mediante el archivo `.lu` actualizado en la rama principal. Consulte [Control de versiones](#versioning) para conocer las consideraciones sobre cómo establecer el nombre de la versión.

1. Cuando se elimina la rama de características, se recomienda eliminar la aplicación de rama de desarrollo de LUIS creado para el trabajo de la rama de características.

### <a name="developers-can-work-in-multiple-branches-concurrently"></a>Los desarrolladores pueden trabajar en varias ramas simultáneamente

Si sigue el patrón descrito anteriormente en [Los desarrolladores pueden trabajar desde ramas independientes](#developers-can-work-from-independent-branches), usará una aplicación de LUIS única en cada rama de características. Un solo desarrollador puede trabajar en varias ramas simultáneamente, siempre y cuando cambie a la aplicación de rama de desarrollo de LUIS correcta para la rama en la que está trabajando actualmente.

Se recomienda usar el mismo nombre tanto para la rama de características como para la aplicación de rama de desarrollo de LUIS que cree para el trabajo de la rama de características, para reducir la probabilidad que trabaje accidentalmente en la aplicación equivocada.

Como hemos visto anteriormente y para que resulte más fácil, se recomienda trabajar en una sola versión en cada aplicación de rama de desarrollo. Si usa varias versiones, procure activar la versión correcta al cambiar entre aplicaciones de rama de desarrollo.

### <a name="multiple-developers-can-work-on-the-same-branch-concurrently"></a>Varios desarrolladores pueden trabajar en la misma rama simultáneamente

Puede permitir que varios desarrolladores trabajen en la misma rama de características a la vez:

- Los desarrolladores extraen la misma rama de características del repositorio, y envían e incorporan los cambios que ellos mismos y otros desarrolladores han enviado en el transcurso del trabajo, del modo habitual.

- Si sigue el patrón descrito anteriormente en [Los desarrolladores pueden trabajar desde ramas independientes](#developers-can-work-from-independent-branches), esta rama usará una aplicación de LUIS única para admitir el desarrollo. El primer miembro del equipo de desarrollo que comience a trabajar en la rama de características creará esa aplicación de "rama de desarrollo" de LUIS.

- [Agregue miembros del equipo como colaboradores](./luis-how-to-collaborate.md) a la aplicación de rama de desarrollo de LUIS.

- Cuando se complete el trabajo de la rama de características, exporte la versión activa de la aplicación de rama de desarrollo de LUIS como `.lu` de la [lista de versiones](./luis-how-to-manage-versions.md), guarde el archivo `.lu` actualizado en el repositorio y, a continuación, inserte los cambios en el repositorio y realice la solicitud de incorporación de cambios.

### <a name="incorporating-changes-from-one-branch-to-another-with-rebase-or-merge"></a>Incorporación de los cambios de una rama en otra con fusión mediante cambio de base o combinación

Es posible que otros desarrolladores del equipo que trabajen en otra rama hayan realizado actualizaciones en el código fuente de `.lu` y las hayan combinado en la rama principal después de que usted haya creado la rama de características. Es posible que desee incorporar sus cambios en la versión de trabajo antes de continuar realizando cambios en la rama de características. Para hacerlo, [realice la fusión mediante cambio de base o combinación con la rama principal](https://git-scm.com/book/en/v2/Git-Branching-Rebasing) de la misma manera que con cualquier otro recurso de código. Dado que la aplicación de LUIS en formato LUDown es legible, admite la combinación mediante herramientas de combinación estándar.

Siga estas sugerencias si realiza la fusión mediante cambio de base de la aplicación de LUIS en una rama de características:

- Antes de la fusión mediante cambio de base o combinación, asegúrese de que la copia local del código fuente de `.lu` de la aplicación tenga todos los cambios más recientes que ha aplicado con el portal de LUIS. Para ello, primero vuelva a exportar la aplicación desde el portal. De este modo, se asegurará de no perder los cambios que ha realizado en el portal y que todavía no se han exportado.

- Durante la fusión mediante combinación, use las herramientas estándar para resolver cualquier conflicto de combinación.

- Después de la fusión mediante cambio de base o combinación, no olvide volver a importar la aplicación en el portal, a fin de trabajar con la aplicación actualizada mientras sigue aplicando sus propios cambios.

### <a name="merge-prs"></a>Combinación de solicitudes de incorporación de cambios

Una vez aprobada la solicitud de incorporación de cambios, puede realizar la fusión mediante la combinación de los cambios en la rama principal. No se aplica ninguna consideración especial al código fuente LUDown de una aplicación de LUIS: es legible y admite la combinación mediante herramientas de combinación estándar. Los conflictos de fusión mediante combinación se pueden resolver de la misma forma que con otros archivos de código fuente.

Una vez que se ha combinado la solicitud de incorporación de cambios, se recomienda realizar una limpieza:

- Elimine la rama del repositorio.

- Elimine la aplicación de "rama de desarrollo" de LUIS que creó para el trabajo de la rama de características.

Del mismo modo que con los recursos de código de aplicación, debe escribir pruebas unitarias para acompañar a las actualizaciones de la aplicación de LUIS. Debe emplear flujos de trabajo de integración continua para probar lo siguiente:

- Las actualizaciones en una solicitud de incorporación de cambios antes de la combinación de dicha solicitud.
- La aplicación de la rama principal de LUIS después de aprobar una solicitud de incorporación de cambios y de que los cambios se hayan combinado en la rama principal.

Para obtener más información sobre las pruebas de DevOps para LUIS, consulte [Pruebas de DevOps para LUIS](luis-concept-devops-testing.md). Para obtener más información sobre la implementación de flujos de trabajo, consulte [Flujos de trabajo de Automation de DevOps para LUIS](luis-concept-devops-automation.md).

## <a name="code-reviews"></a>Revisiones de código

Una aplicación de LUIS en formato LUDown es legible para el usuario y admite la comunicación de los cambios en una solicitud de incorporación de cambios adecuada para su revisión. Los archivos de prueba unitaria también se escriben en formato LUDown y también se pueden revisar fácilmente en una solicitud de incorporación de cambios.

## <a name="versioning"></a>Control de versiones

Una aplicación consta de varios componentes que pueden incluir elementos como un bot que se ejecute en [Azure Bot Service](/azure/bot-service/bot-service-overview-introduction), [QnA Maker](https://www.qnamaker.ai/), el [servicio Voz de Azure](../speech-service/overview.md), etc. Para lograr el objetivo de las aplicaciones de acoplamiento flexible, use el [control de versiones](/azure/devops/learn/git/what-is-version-control) para que cada componente de una aplicación tenga versiones independientes, lo que permite a los desarrolladores detectar cambios importantes o actualizaciones simplemente mediante la consulta del número de versión. Es más fácil crear una versión de la aplicación de LUIS independientemente de otros componentes si la mantiene en su propio repositorio.

La aplicación de LUIS de la rama principal debe tener aplicado un esquema de control de versiones. Al combinar las actualizaciones con el archivo `.lu` de una aplicación de LUIS en la rama principal, importará ese código fuente actualizado a una nueva versión de la rama principal en la aplicación de LUIS.

Se recomienda usar un esquema de control de versiones numérico para la versión de la aplicación principal de LUIS, por ejemplo:

`major.minor[.build[.revision]]`

Con cada actualización, se incrementa el último dígito del número de versión.

La versión principal o secundaria se puede usar para indicar el ámbito de los cambios en la funcionalidad de la aplicación de LUIS:

* Versión principal: un cambio significativo, como la compatibilidad con una nueva [intención](./luis-concept-intent.md) o [entidad](./luis-concept-entity-types.md).
* Versión secundaria: un cambio menor de compatibilidad con versiones anteriores, como después de un nuevo entrenamiento significativo.
* Compilación: no cambia ninguna funcionalidad, solo es una compilación diferente.

Una vez que haya determinado el número de versión de la última revisión de la aplicación principal de LUIS, debe compilar y probar la nueva versión de la aplicación, y publicarla en un punto de conexión donde se pueda usar en entornos de compilación diferentes, como los de control de calidad o producción. Se recomienda encarecidamente automatizar todos estos pasos en un flujo de trabajo de integración continua (CI).

Consulte:
- [Flujos de trabajo de automatización](luis-concept-devops-automation.md) para obtener más información sobre cómo implementar un flujo de trabajo de CI para probar y publicar una aplicación de LUIS.
- [Administración de versiones](luis-concept-devops-automation.md#release-management) para obtener información sobre cómo implementar la aplicación de LUIS.

### <a name="versioning-the-feature-branch-luis-app"></a>Control de versiones de la aplicación de "rama de características" de LUIS

Si trabaja con una aplicación de "rama de desarrollo" de LUIS que ha creado para admitir el trabajo en una rama de características, exportará la aplicación cuando se complete el trabajo e incluirá el archivo `'lu` actualizado en la solicitud de incorporación de cambios. La rama del repositorio y la aplicación de "rama de desarrollo" de LUIS deben eliminarse después de combinar la solicitud de incorporación de cambios con la rama principal. Dado que esta aplicación existe únicamente para admitir el trabajo en la rama de características, no hay ningún esquema de control de versiones concreto que deba aplicar en esta aplicación.

Cuando los cambios de la solicitud de incorporación de cambios se hayan combinado con la rama principal, debe aplicar el control de versiones para que la versión de todas las actualizaciones se cree de forma independiente en la rama principal.

## <a name="next-steps"></a>Pasos siguientes

* Obtener información sobre las [pruebas de LUIS DevOps](luis-concept-devops-testing.md)
* Obtener información sobre cómo [implementar DevOps para LUIS con GitHub](./luis-concept-devops-automation.md)