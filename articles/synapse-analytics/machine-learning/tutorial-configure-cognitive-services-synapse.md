---
title: 'Tutorial: Requisitos previos para el uso de Cognitive Services en Azure Synapse'
description: Aprenda a configurar los requisitos previos para usar Cognitive Services en Azure Synapse.
services: synapse-analytics
ms.service: synapse-analytics
ms.subservice: machine-learning
ms.topic: tutorial
ms.reviewer: jrasnick, garye
ms.date: 11/20/2020
author: nelgson
ms.author: negust
ms.openlocfilehash: da1760def8f2c036854839787b1051e8f52358d0
ms.sourcegitcommit: ce9178647b9668bd7e7a6b8d3aeffa827f854151
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/12/2021
ms.locfileid: "109809659"
---
# <a name="tutorial-prerequisites-for-using-cognitive-services-in-azure-synapse-analytics"></a>Tutorial: Requisitos previos para el uso de Cognitive Services en Azure Synapse

En este tutorial, aprenderá a configurar los requisitos previos para usar Azure Cognitive Services en Azure Synapse Analytics de forma segura.

Esta tutorial abarca lo siguiente:
> [!div class="checklist"]
> - La creación de un recurso de Cognitive Services, como Text Analytics o Anomaly Detector.
> - El almacenamiento de una clave de autenticación para los recursos de Cognitive Services como secretos en Azure Key Vault, y la configuración del acceso para un área de trabajo de Azure Synapse Analytics.
> - La creación de un servicio vinculado de Azure Key Vault en el área de trabajo de Azure Synapse Analytics.

Si no tiene una suscripción a Azure, [cree una cuenta gratuita antes de empezar](https://azure.microsoft.com/free/).

## <a name="prerequisites"></a>Requisitos previos

- Un [área de trabajo de Azure Synapse Analytics](../get-started-create-workspace.md) con una cuenta de Azure Data Lake Storage Gen2 configurada como ubicación de almacenamiento predeterminado. Asegúrese de ser el *colaborador de datos de blobs de almacenamiento* en el sistema de archivos de Azure Data Lake Storage Gen2 en el que está trabajando.

## <a name="sign-in-to-the-azure-portal"></a>Inicio de sesión en Azure Portal

Inicie sesión en [Azure Portal](https://portal.azure.com/).

## <a name="create-a-cognitive-services-resource"></a>Creación de un recurso de Cognitive Services

[Azure Cognitive Services](../../cognitive-services/index.yml) incluye muchos tipos de servicios. Text Analytics y Anomaly Detector son dos ejemplos en los tutoriales de Azure Synapse.

Puede crear un recurso de [Text Analytics](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics) en Azure Portal:

![Captura de pantalla que muestra Text Analytics en el portal, con el botón Crear.](media/tutorial-configure-cognitive-services/tutorial-configure-cognitive-services-00b.png)

Puede crear un recurso de [Anomaly Detector](https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesTextAnalytics) en Azure Portal:

![Captura de pantalla que muestra Anomaly Detector en el portal, con el botón Crear.](media/tutorial-configure-cognitive-services/tutorial-configure-cognitive-services-00a.png)

## <a name="create-a-key-vault-and-configure-secrets-and-access"></a>Creación de un almacén de claves y configuración de los secretos y el acceso

1. Cree un [almacén de claves](https://ms.portal.azure.com/#create/Microsoft.KeyVault) en Azure Portal.
2. Vaya a **Key Vault** > **Directivas de acceso** y conceda permisos de [identidad administrada del área de trabajo de Azure Synapse](../security/synapse-workspace-managed-identity.md) para leer secretos de Azure Key Vault.

   > [!NOTE]
   > Asegúrese de que se guardan los cambios en la directiva. Este paso es fácil de pasar por alto.

   ![Captura de pantalla que muestra las selecciones para agregar una directiva de acceso.](media/tutorial-configure-cognitive-services/tutorial-configure-cognitive-services-00c.png)

3. Vaya al recurso de Cognitive Services. Por ejemplo, vaya a **Anomaly Detector** > **Claves y punto de conexión**. A continuación, copie una de las dos claves en el Portapapeles.

4. Vaya a **Key Vault** > **Secreto** para crear un secreto nuevo. Especifique el nombre del secreto y pegue la clave del paso anterior en el campo **Valor**. Por último, seleccione **Crear**.

   ![Captura de pantalla que muestra las selecciones para crear un secreto.](media/tutorial-configure-cognitive-services/tutorial-configure-cognitive-services-00d.png)

   > [!IMPORTANT]
   > Asegúrese de memorizar o anotar este nombre de secreto. Lo usará más adelante para conectarse a Cognitive Services desde Synapse Studio.

## <a name="create-an-azure-key-vault-linked-service-in-azure-synapse"></a>Creación de un servicio vinculado de Azure Key Vault en Azure Synapse

1. Abra el área de trabajo en Synapse Studio. 
2. Vaya a **Administrar** > **Servicios vinculados**. Cree un servicio vinculado de **Azure Key Vault**. Para ello, apunte al almacén de claves que acaba de crear. 
3. Seleccione el botón **Probar conexión** para comprobar la conexión. Si el color de la conexión es verde, seleccione **Crear** y, a continuación, **Publicar todo** para guardar el cambio.

![Captura de pantalla que muestra Azure Key Vault como un nuevo servicio vinculado.](media/tutorial-configure-cognitive-services/tutorial-configure-cognitive-services-00e.png)

Ya puede continuar con uno de los tutoriales para usar la experiencia de Azure Cognitive Services en Synapse Studio.

## <a name="next-steps"></a>Pasos siguientes

- [Tutorial: Análisis de sentimiento con Azure Cognitive Services](tutorial-cognitive-services-sentiment.md)
- [Tutorial: Detección de anomalías con Azure Cognitive Services](tutorial-cognitive-services-sentiment.md)
- [Tutorial: Asistente para puntuación del modelo de Machine Learning para grupos de SQL dedicados](tutorial-sql-pool-model-scoring-wizard.md).
- [Funcionalidades de aprendizaje automático en Azure Synapse Analytics](what-is-machine-learning.md)
