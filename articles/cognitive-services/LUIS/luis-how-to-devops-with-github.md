---
title: 'Procedimiento para aplicar DevOps con GitHub: LUIS'
titleSuffix: Azure Cognitive Services
description: Aplique DevOps con Language Understanding (LUIS) y GitHub.
services: cognitive-services
author: andycw
manager: cmayo
ms.custom: seodec18
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: conceptual
ms.date: 06/5/2020
ms.author: anwigley
ms.openlocfilehash: 448b3d93ed58e4cfc73da576f0c5871600400ac6
ms.sourcegitcommit: 5c136a01bddfccb2cc9f7e7e7741e2cf2651ddbe
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/03/2021
ms.locfileid: "111352512"
---
# <a name="apply-devops-to-luis-app-development-using-github-actions"></a>Aplicación de DevOps al desarrollo de aplicaciones de LUIS mediante Acciones de GitHub

Vaya al [repositorio de plantillas de DevOps para LUIS](https://github.com/Azure-Samples/LUIS-DevOps-Template), donde obtendrá una solución completa que implementará los procedimientos recomendados de ingeniería de software y DevOps para LUIS. Puede usar este repositorio de plantillas para crear su propio repositorio con compatibilidad integrada para los flujos de trabajo de CI/CD y procedimientos que habilitan el [control de código fuente](luis-concept-devops-sourcecontrol.md), [las compilaciones automatizadas](luis-concept-devops-automation.md), las [pruebas](luis-concept-devops-testing.md) y la [administración de versiones](luis-concept-devops-automation.md#release-management) con LUIS para su propio proyecto.

## <a name="the-luis-devops-template-repo"></a>Repositorio de plantillas de DevOps para LUIS

En el [repositorio de plantillas de DevOps para LUIS](https://github.com/Azure-Samples/LUIS-DevOps-Template) se explican los procedimientos para realizar lo siguiente:

* **Clonar el repositorio de plantillas**: copie la plantilla en su propio repositorio de GitHub.
* **Configurar los recursos de LUIS**: cree los [recursos de creación y predicción de LUIS en Azure](./luis-how-to-azure-subscription.md) que usarán los flujos de trabajo de integración continua.
* **Configurar los flujos de trabajo de CI/CD**: configure los parámetros de los flujos de trabajo de CI/CD y almacénelos en [Secretos de GitHub](https://help.github.com/actions/configuring-and-managing-workflows/creating-and-storing-encrypted-secrets).
* **Comprender el [bucle interno de desarrollo](/dotnet/architecture/containerized-lifecycle/design-develop-containerized-apps/docker-apps-inner-loop-workflow)** : el desarrollador realiza actualizaciones en una aplicación de LUIS de ejemplo mientras trabaja en una rama de desarrollo, prueba las actualizaciones y, a continuación, genera una solicitud de incorporación de cambios para proponer cambios y para buscar la aprobación de la revisión.
* **Ejecutar flujos de trabajo de CI/CD**: ejecute [flujos de trabajo de integración continua para compilar y probar una aplicación de LUIS](luis-concept-devops-automation.md) mediante Acciones de GitHub.
* **Realizar pruebas automatizadas**: realice [pruebas por lotes automatizadas para una aplicación de LUIS](luis-concept-devops-testing.md) a fin de evaluar la calidad de esta.
* **Implementar la aplicación de LUIS**: ejecute un [trabajo de entrega continua (CD)](luis-concept-devops-automation.md#continuous-delivery-cd) para publicar la aplicación de LUIS.
* **Usar el repositorio con su propio proyecto**: use el repositorio con su propia aplicación de LUIS.

## <a name="next-steps"></a>Pasos siguientes

* Use el [repositorio de plantillas de DevOps para LUIS](https://github.com/Azure-Samples/LUIS-DevOps-Template) con el fin de aplicar DevOps con su propio proyecto.
* [Control de código fuente y estrategias de ramificación para LUIS](luis-concept-devops-sourcecontrol.md)
* [Pruebas de DevOps para LUIS](luis-concept-devops-testing.md)
* [Flujos de trabajo de automatización de DevOps para LUIS](luis-concept-devops-automation.md)
