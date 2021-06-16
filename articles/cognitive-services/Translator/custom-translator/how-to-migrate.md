---
title: Migración del área de trabajo y los proyectos de Microsoft Translator Hub - Custom Translator
titleSuffix: Azure Cognitive Services
description: En este artículo se explica cómo migrar los proyectos y el área de trabajo del centro a Custom Translator de Azure Cognitive Services.
author: laujan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 05/26/2020
ms.author: lajanuar
ms.topic: conceptual
ms.openlocfilehash: 89658ce88b4f1ac9d5bacac7bd45511b4aa0a1be
ms.sourcegitcommit: 5c136a01bddfccb2cc9f7e7e7741e2cf2651ddbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111354275"
---
# <a name="migrate-hub-workspace-and-projects-to-custom-translator"></a>Migración del área de trabajo y los proyectos de Microsoft Translator Hub a Custom Translator

Puede migrar fácilmente el área de trabajo y los proyectos de [Microsoft Translator Hub](https://hub.microsofttranslator.com/) a Traductor personalizado. La migración se inicia desde Microsoft Hub al seleccionar un área de trabajo o proyecto; a continuación, se selecciona un área de trabajo en Traductor personalizado y, a continuación, se seleccionan los entrenamientos que desea transferir. Después de iniciar la migración, se transferirán la opciones de configuración de entrenamiento seleccionadas con todos los documentos pertinentes. Se entrenan los modelos implementados y se pueden implementar automáticamente tras la finalización.

Estas acciones se realizan durante la migración:
* Se transfieren los nombres de todos los documentos y las definiciones del proyecto con el prefijo "hub_" antes del nombre. La prueba generada automáticamente y los datos de optimización se llamarán hub_systemtune_\<modelid> o hub_systemtest_\<modelid>.
* Los entrenamientos que estaban en estado implementado en el momento de la migración se entrenarán automáticamente con los documentos de entrenamiento del centro. Este entrenamiento no se cargará en la suscripción. Si se ha seleccionado implementación automática para la migración, el modelo entrenado se implementará tras la finalización. Se aplicarán cargos de hospedaje normales.
* Los entrenamientos migrados que no estaban en estado implementado se colocan en estado de borrador migrado. En este estado, tendrá la opción de entrenar un modelo con la definición migrada, pero se aplicarán cargos de entrenamiento normales.
* En cualquier momento, la puntuación BLEU migrada desde el entrenamiento del centro se puede encontrar en la página TrainingDetails del modelo en el encabezado "Puntuación BLEU del centro de MT".

> [!Note] 
> Para que un entrenamiento se realice correctamente, el Traductor personalizado necesita un mínimo de 10 000 frases extraídas únicas. El Traductor personalizado no puede realizar un entrenamiento con menos del [mínimo sugerido](./sentence-alignment.md#suggested-minimum-number-of-sentences).

## <a name="find-custom-translator-workspace-id"></a>Búsqueda del identificador de área de trabajo de Traductor personalizado

Para migrar un área de trabajo de [Microsoft Translator Hub](https://hub.microsofttranslator.com/) necesita el identificador del área de trabajo de destino en Traductor personalizado. El área de trabajo de destino en Custom Translator es el lugar al que se migrarán todas las áreas de trabajo y proyectos de Microsoft Translator Hub.

Puede encontrar el identificador del área de trabajo de destino en la página de configuración de Traductor personalizado:

1. Vaya a la página "Configuración" en el portal de Custom Translator.

2. Encontrará el identificador del área de trabajo en la sección de información básica.

    ![Búsqueda del identificador del área de trabajo de destino](media/how-to/how-to-find-destination-ws-id.png)

3. Conserve el identificador del área de trabajo de destino para consultarlo durante el proceso de migración.

## <a name="migrate-a-project"></a>Migración de un proyecto

Si desea migrar los proyectos de forma selectiva, Microsoft Translator Hub le ofrece esa posibilidad.

Para migrar un proyecto:

1. Inicie sesión en Microsoft Translator Hub.

2. Vaya a la página "Proyectos".

3. Haga clic en el vínculo "Migrar" del proyecto adecuado.

    ![Captura de pantalla que resalta el botón Migrar para el proyecto seleccionado.](media/how-to/how-to-migrate-from-hub.png)

4. Al presionar el vínculo de migración, aparecerá un formulario que le permite:
   * Especificar el área de trabajo que desea transferir a Traductor personalizado
   * Indicar si desea transferir todos los entrenamientos con entrenamientos correctos o solo los entrenamientos implementados. De forma predeterminada, se transferirán todos los entrenamientos correctos.
   * Indicar si desea que los entrenamientos se implementen automáticamente cuando se complete el entrenamiento. De forma predeterminada, el entrenamiento no se implementará automáticamente cuando se complete.

5. Haga clic en "Enviar solicitud".

## <a name="migrate-a-workspace"></a>Migración de un área de trabajo

Además de migrar un único proyecto, también puede migrar todos los proyectos con entrenamientos correctos de un área de trabajo. Esto hará que cada proyecto del área de trabajo se evalúe como si se hubiera presionado el vínculo de migración. Esta característica es adecuada para los usuarios con muchos proyectos que desean migrar todos a Traductor personalizado con la misma configuración. La migración de un área de trabajo se puede iniciar en la página de configuración de Translator Hub.

Para migrar un área de trabajo:

1. Inicie sesión en Microsoft Translator Hub.

2. Vaya a la página Configuración.

3. En la página "Configuración", haga clic en "Migrate Workspace data to Custom Translator" (Migración de datos de área de trabajo a Custom Translator).

    ![Captura de pantalla que resalta la opción para migrar datos del área de trabajo al Traductor personalizado.](media/how-to/how-to-migrate-workspace-from-hub.png)

4. En la siguiente página, seleccione cualquiera de estas dos opciones:

    a. Solo los entrenamientos implementados: si selecciona esta opción se migrarán solo los sistemas implementados y los documentos relacionados.

    b. Todos los entrenamientos correctos: si selecciona esta opción se migrarán todos los entrenamientos y los documentos relacionados.

    c. Escriba el identificador del área de trabajo de destino en Custom Translator.

    ![Migración desde Microsoft Translator Hub](media/how-to/how-to-migrate-from-hub-screen.png)

5. Haga clic en Enviar solicitud.

## <a name="migration-history"></a>Historial de migraciones

Si solicita la migración del área de trabajo o de los proyectos de Hub, puede ver el historial de migraciones en la página Configuración de Traductor personalizado.

Para ver el historial de migraciones realice estos pasos:

1. Vaya a la página "Configuración" en el portal de Custom Translator.

2. En la sección del historial de migraciones de la página Configuración, haga clic en Historial de migraciones.

    ![Historial de migraciones](media/how-to/how-to-migration-history.png)

La página Historial de migraciones muestra la siguiente información como resumen para cualquier migración que solicite.

1. Migrado por: nombre y correo electrónico del usuario que envió esta solicitud de migración

2. Migrado el: marca de fecha y hora de la migración

3. Proyectos: número de proyectos para los que se solicitó la migración en comparación con el número de proyectos que se migraron correctamente.

4. Entrenamientos: número de entrenamientos para los que se solicitó la migración en comparación con el número de entrenamientos que se migraron correctamente.

5. Documentos: número de documentos para los que se solicitó la migración en comparación con el número de documentos que se migraron correctamente.

    ![Detalles del historial de migraciones](media/how-to/how-to-migration-history-details.png)

Si desea obtener un informe de migración más detallado sobre los proyectos, aprendizajes y documentos, puede exportar los detalles en un archivo .csv.

## <a name="implementation-notes"></a>Notas de implementación
* Los sistemas con pares de idiomas que todavía NO están disponibles en el Traductor personalizado solo podrán acceder a los datos o anular la implementación a través del Traductor personalizado. Estos proyectos se marcarán como "No disponible" en la página Proyectos. A medida que se habiliten nuevos pares de idiomas con el Traductor personalizado, los proyectos quedarán activos para su entrenamiento e implementación. 
* La migración de un proyecto del centro a Traductor personalizado no tendrá ningún impacto en los entrenamientos o proyectos del centro. No se eliminan documentos ni proyectos del centro durante una migración y no se anula la implementación de modelos.
* Solo se permite migrar una vez por proyecto. Si necesita repetir una migración de un proyecto, póngase en contacto con nosotros.
* El Traductor personalizado admite los pares de idiomas de NMT desde y hacia el inglés. [Visualización de la lista completa de los idiomas admitidos](../language-support.md#customization). El centro no requiere modelos de referencia y, por tanto, admite varios miles de lenguajes. Puede migrar un par de idiomas no admitidos, pero solo se realizará la migración de los documentos y las definiciones del proyecto. No se podrá entrenar el modelo nuevo. Además, estos documentos y proyectos se mostrarán como inactivos para indicar que no se pueden utilizar en este momento. Si se agrega compatibilidad para estos proyectos y/o documentos, estarán activos y se podrán entrenar.
* Traductor personalizado no admite actualmente datos de entrenamiento monolingüe. Como con los pares de idiomas no admitidos, puede migrar documentos monolingües, pero se mostrarán como inactivos hasta que se admitan datos monolingües.
* Traductor personalizado requiere oraciones 10 000 frases en paralelo para el entrenamiento. Microsoft Hub podría entrenar sobre un conjunto más pequeño de datos. Si se migra un entrenamiento que no cumple este requisito, no será entrenado.

## <a name="custom-translator-versus-hub"></a>Custom Translator frente a Hub

En la siguiente tabla se comparan las características de Microsoft Translator Hub y Custom Translator.

| Característica | Hub | Custom Translator |
| ------- | :-: | :---------------: |
| Estado de la característica de personalización    | Disponibilidad general    | Disponibilidad general |
| Versión de Text API    | V2     | V3  |
| Personalización de SMT    | Sí    | No |
| Personalización de NMT    | No    | Sí |
| Nueva personalización unificada de servicios de voz    | No    | Sí |
| Sin seguimiento | Sí | Sí |

## <a name="new-languages"></a>Nuevos idiomas

Si es una comunidad u organización que trabaja para crear un sistema de idioma para Translator, vaya a [custommt@microsoft.com](mailto:custommt@microsoft.com) para obtener más información.

## <a name="next-steps"></a>Pasos siguientes

- [Entrenamiento de un modelo](how-to-train-model.md).
- Comience a utilizar el modelo de traducción personalizado implementado mediante [Traductor V3](../reference/v3-0-translate.md?tabs=curl).