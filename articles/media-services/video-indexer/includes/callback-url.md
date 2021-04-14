---
author: Juliako
ms.service: media-services
ms.topic: include
ms.date: 11/13/2020
ms.author: juliako
ms.openlocfilehash: fbc77b960cac0db2d077345c74d64485bd7ad8bb
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "95994507"
---
Una dirección URL que se usa para notificar al cliente (mediante una solicitud POST) los siguientes eventos:

- Indexación de los cambios de estado: 
    - Propiedades:    
    
        |Nombre|Descripción|
        |---|---|
        |id|Identificador del vídeo|
        |state|El estado del vídeo|  
    - Ejemplo: https:\//test.com/notifyme?projectName=MyProject&id=1234abcd&state=Processed
- Persona identificada en el vídeo:
  - Propiedades
    
      |Nombre|Descripción|
      |---|---|
      |id| Identificador del vídeo|
      |faceId|El identificador de caras que aparece en el índice de vídeo|
      |knownPersonId|El identificador de persona que es único dentro de un modelo de cara|
      |personName|El nombre de la persona|
        
    - Ejemplo: https:\//test.com/notifyme?projectName=MyProject&id=1234abcd&faceid=12&knownPersonId=CCA84350-89B7-4262-861C-3CAC796542A5&personName=Inigo_Montoya 
