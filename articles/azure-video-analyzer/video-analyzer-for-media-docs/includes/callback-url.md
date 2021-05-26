---
author: Juliako
ms.topic: include
ms.date: 11/13/2020
ms.author: juliako
ms.openlocfilehash: db2f6fe6175b8ff4d6371cb7074fb8c83af51c14
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110386908"
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
