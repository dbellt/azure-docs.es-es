---
title: 'CI/CD para Habla personalizada: servicio de Voz'
titleSuffix: Azure Cognitive Services
description: Aplique DevOps con los flujos de trabajo de Habla personalizada y CI/CD. Implemente una solución de DevOps existente para su propio proyecto.
services: cognitive-services
author: KatieProchilo
manager: cmayomsft
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 06/09/2020
ms.author: kaprochi
ms.openlocfilehash: 06a5bd2366f2a24e263e8ffa1c905a5cbd6afc9b
ms.sourcegitcommit: 070122ad3aba7c602bf004fbcf1c70419b48f29e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111440347"
---
# <a name="cicd-for-custom-speech"></a>CI/CD para Habla personalizada

Implemente procesos automáticos de entrenamiento, pruebas y administración de versiones para permitir una mejora continua de los modelos de Habla personalizada a medida que se aplican actualizaciones a los datos de entrenamiento y pruebas. A través de una implementación eficaz de los flujos de trabajo de CI/CD, puede asegurarse de que el punto de conexión siempre esté disponible para el modelo de Habla personalizada de mejor rendimiento.

La [integración continua](/devops/develop/what-is-continuous-integration) (CI) es la práctica de ingeniería que consiste en confirmar con frecuenta las actualizaciones en un repositorio compartido y realizar una compilación automatizada con él. Los flujos de trabajo de CI para Habla personalizada entrenan un nuevo modelo a partir de sus orígenes de datos y realizan pruebas automatizadas en ese nuevo modelo para asegurarse de que funciona mejor que el modelo anterior.

La [entrega continua](/devops/deliver/what-is-continuous-delivery) (CD) usa los modelos del proceso de CI y crea un punto de conexión para cada modelo de Habla personalizada mejorado. La CD facilita la integración de los puntos de conexión en las soluciones.

Es posible obtener soluciones de CI/CD personalizadas; no obstante, para una solución sólida y precompilada, use el [repositorio de plantillas de DevOps para Voz](https://github.com/Azure-Samples/Speech-Service-DevOps-Template), que ejecuta flujos de trabajo de CI/CD mediante Acciones de GitHub.

## <a name="cicd-workflows-for-custom-speech"></a>Flujos de trabajo de CI/CD para Habla personalizada

El objetivo de estos flujos de trabajo es garantizar que cada modelo de Habla personalizada tenga una mejor precisión de reconocimiento que la versión anterior. Si las actualizaciones de los datos de entrenamiento o pruebas mejoran la precisión, estos flujos de trabajo crean un nuevo punto de conexión de Habla personalizada.

Los servidores Git, como GitHub y Azure DevOps, pueden ejecutar flujos de trabajo automatizados cuando se producen eventos Git específicos, como fusiones mediante combinación o solicitudes de incorporación de cambios. Por ejemplo, se puede desencadenar un flujo de trabajo de CI cuando las actualizaciones de los datos de prueba se envían a la rama *principal*. Los distintos servidores Git tendrán herramientas diferentes, pero permitirán la creación de scripts con comandos de la interfaz de línea de comandos (CLI) para que puedan ejecutarse en un servidor de compilación.

Durante el proceso, los flujos de trabajo deben asignar un nombre a los datos, pruebas, archivos de prueba, modelos y puntos de conexión, así como almacenarlos, de modo que se puedan rastrear de vuelta a su confirmación o versión de origen. También resulta útil asignar un nombre a estos recursos para facilitar la identificación de cuáles se crearon después de actualizar los datos de prueba frente a los datos de entrenamiento.

### <a name="ci-workflow-for-testing-data-updates"></a>Flujo de trabajo de CI para actualizaciones de datos de prueba

El principal objetivo de los flujos de trabajo de CI/CD es crear un nuevo modelo con los datos de entrenamiento y probar dicho modelo con los datos de prueba para determinar si la métrica [Word Error Rate](how-to-custom-speech-evaluate-data.md#evaluate-custom-speech-accuracy) (WER) ha mejorado en comparación con el modelo de mejor rendimiento anterior (el "modelo de referencia"). Si el nuevo modelo funciona mejor, se convierte en el nuevo modelo de pruebas con el que se comparan los modelos futuros.

El flujo de trabajo de CI para las actualizaciones de datos de pruebas debe volver a probar el modelo de referencia actual con los datos de prueba actualizados para calcular el WER revisado. Esto garantiza que, cuando el WER de un modelo nuevo se compare con el WER del banco de pruebas, ambos modelos se habrán probado con los mismos datos de prueba y se estarán comparando elementos semejantes.

Este flujo de trabajo se debe desencadenar con las actualizaciones de los datos de prueba y también debe:

- Probar el modelo de banco de pruebas con los datos de pruebas actualizados.
- Almacenar la salida de la prueba, que contiene el WER del modelo de banco de pruebas, con los datos actualizados.
- El valor WER de estas pruebas se convertirá en el nuevo WER de banco de pruebas que los modelos futuros deberán vencer.
- El flujo de trabajo de CD no se ejecuta para las actualizaciones de los datos de prueba.

### <a name="ci-workflow-for-training-data-updates"></a>Flujo de trabajo de CI para actualizaciones de datos de entrenamiento

Las actualizaciones de los datos de entrenamiento implican actualizaciones del modelo personalizado.

Este flujo de trabajo se debe desencadenar con las actualizaciones de los datos de entrenamiento y también debe:

- Entrenar un nuevo modelo con los datos de entrenamiento actualizados.
- Probar el nuevo modelo con los datos de prueba.
- Almacenar la salida de la prueba, que contiene el valor de WER.
- Comparar el valor de WER del modelo nuevo con la WER del modelo de banco de pruebas.
- Si el valor de WER no mejora, detenga el flujo de trabajo.
- Si el valor de WER mejora, ejecute el flujo de trabajo de CD para crear un punto de conexión de Habla personalizada.

### <a name="cd-workflow"></a>Flujo de trabajo de CD

Después de que una actualización de los datos de entrenamiento mejora el reconocimiento de un modelo, el flujo de trabajo de CD debe ejecutarse automáticamente para crear un nuevo punto de conexión para ese modelo y hacer que dicho punto esté disponible para que se pueda usar en una solución.

#### <a name="release-management"></a>Administración de versiones

La mayoría de los equipos requieren un proceso de revisión y aprobación manual para implementarse un entorno de producción. En el caso de una implementación de producción, quizá quiera asegurarse de que se produce cuando las personas clave del equipo de desarrollo están disponibles para dar soporte técnico, o durante períodos de tráfico bajo.

### <a name="tools-for-custom-speech-workflows"></a>Herramientas para los flujos de trabajo de Habla personalizada

Use las siguientes herramientas de flujos de trabajo de automatización de CI/CD para Habla personalizada:

- [CLI de Azure](/cli/azure/) para crear una autenticación de la entidad de servicio de Azure, consultar las suscripciones a Azure y almacenar los resultados de las pruebas en el Blob de Azure.
- [CLI de Azure Speech](spx-overview.md) para interactuar con el servicio de voz desde la línea de comandos o desde un flujo de trabajo automatizado.

## <a name="devops-solution-for-custom-speech-using-github-actions"></a>Solución de DevOps para Habla personalizada mediante Acciones de GitHub

En el caso de una solución DevOps ya implementada para Habla personalizada, visite el [repositorio de plantillas de DevOps para Voz](https://github.com/Azure-Samples/Speech-Service-DevOps-Template). Cree una copia de la plantilla y comience a desarrollar modelos personalizados con un sistema DevOps sólido que incluye pruebas, entrenamiento y control de versiones mediante Acciones de GitHub. El repositorio proporciona datos de prueba y entrenamiento de ejemplo para ayudarle a configurar y explicar el flujo de trabajo. Después de la instalación inicial, reemplace los datos de ejemplo con los datos del proyecto.

El [repositorio de plantillas de DevOps para Voz](https://github.com/Azure-Samples/Speech-Service-DevOps-Template) proporciona la infraestructura y las instrucciones detalladas para:

- Copiar el repositorio de plantillas en su cuenta de GitHub y, a continuación, crear los recursos de Azure y una [entidad de servicio](../../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) para los flujos de trabajo de CI/CD de Acciones de GitHub.
- Realizar el "[bucle interno de desarrollo](/dotnet/architecture/containerized-lifecycle/design-develop-containerized-apps/docker-apps-inner-loop-workflow)". Actualizar los datos de entrenamiento y prueba desde una rama de características, probar los cambios con un modelo de desarrollo temporal y generar una solicitud de incorporación de cambios para proponer y revisar los cambios.
- Entrenar modelos con el flujo de trabajo de CI mediante Acciones de GitHub cuando se actualizan los datos de entrenamiento en una solicitud de incorporación de cambios para la rama *master*.
- Realizar pruebas de precisión automatizadas para establecer la métrica [Word Error Rate](how-to-custom-speech-evaluate-data.md#evaluate-custom-speech-accuracy) (WER) de un modelo. Almacenar los resultados de pruebas en el Blob de Azure.
- Ejecutar el flujo de trabajo de CD para crear un punto de conexión cuando mejore el valor de WER.

## <a name="next-steps"></a>Pasos siguientes

Más información sobre DevOps con Voz:

- Use el [repositorio de plantillas de DevOps para Voz](https://github.com/Azure-Samples/Speech-Service-DevOps-Template) para implementar DevOps para Habla personalizada con Acciones de GitHub.
