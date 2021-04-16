---
title: Docker pull para el contenedor de salud
titleSuffix: Azure Cognitive Services
description: Comando docker pull para el contenedor de Text Analytics for Health
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: text-analytics
ms.topic: include
ms.date: 03/25/2021
ms.author: aahi
ms.openlocfilehash: 8595af7f46b63f9991d6a02279ccad76afb38311
ms.sourcegitcommit: 99fc6ced979d780f773d73ec01bf651d18e89b93
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/31/2021
ms.locfileid: "106089722"
---
Rellene y envíe el [formulario de solicitud de Cognitive Services](https://aka.ms/csgate) para solicitar acceso a la versión preliminar pública de Text Analytics for Health.  Esta aplicación se aplica tanto al contenedor como a la versión preliminar pública de la API web hospedada.
El formulario solicita información acerca del usuario y de su empresa, así como del escenario de usuario para el que se va a usar el contenedor. Después de haber enviado el formulario, el equipo de Azure Cognitive Services lo revisa para asegurarse de que cumple los criterios de acceso al registro de contenedor privado.

> [!IMPORTANT]
> * En el formulario, debe usar una dirección de correo electrónico asociada a un identificador de suscripción de Azure.
> * El recurso Text Analytics (punto de conexión de facturación y apiKey) que se usa para ejecutar el contenedor se debe haber creado con el identificador de suscripción de Azure aprobado. 
> * Compruebe el correo electrónico (bandeja de entrada y carpetas de correo no deseado) para obtener las actualizaciones del estado de la aplicación por parte de Microsoft.
> * Esta dirección URL del contenedor ha cambiado, vea los ejemplos siguientes. El contenedor no estará disponible para su descarga a partir del `containerpreview.azurecr.io` 26 de abril de 2021.


Tras la aprobación, use el comando [`docker pull`](https://docs.docker.com/engine/reference/commandline/pull/) para descargar esta imagen del contenedor del registro de contenedor público de Microsoft.

```
docker pull mcr.microsoft.com/azure-cognitive-services/textanalytics/healthcare:latest
```
