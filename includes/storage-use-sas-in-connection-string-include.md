---
author: tamram
ms.service: storage
ms.topic: include
ms.date: 10/26/2018
ms.author: tamram
ms.openlocfilehash: 2f27c50b1d016265c20102521a137bcbb0646115
ms.sourcegitcommit: f28ebb95ae9aaaff3f87d8388a09b41e0b3445b5
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/29/2021
ms.locfileid: "67186533"
---
Si posee una dirección URL de firma de acceso compartido (SAS) que le concede acceso a los recursos de una cuenta de almacenamiento, puede utilizar la SAS en una cadena de conexión. Dado que SAS incluye la información necesaria para autenticar la solicitud, una cadena de conexión con un SAS proporciona el protocolo, el punto de conexión de servicio y las credenciales necesarias para acceder al recurso.

Para crear una cadena de conexión que incluya una firma de acceso compartido, especifique la cadena con el siguiente formato:

```
BlobEndpoint=myBlobEndpoint;
QueueEndpoint=myQueueEndpoint;
TableEndpoint=myTableEndpoint;
FileEndpoint=myFileEndpoint;
SharedAccessSignature=sasToken
```

Cada punto de conexión de servicio es opcional, aunque la cadena de conexión debe contener al menos uno.

> [!NOTE]
> Se recomienda usar HTTPS con SAS.
>
> Si está especificando SAS en una cadena de conexión en un archivo de configuración, debe codificar caracteres especiales en la dirección URL.
>
>

### <a name="service-sas-example"></a>Ejemplo de SAS de servicio
Este es un ejemplo de una cadena de conexión que incluye un SAS de servicio para el almacenamiento de blobs:

```
BlobEndpoint=https://storagesample.blob.core.windows.net;
SharedAccessSignature=sv=2015-04-05&sr=b&si=tutorial-policy-635959936145100803&sig=9aCzs76n0E7y5BpEi2GvsSv433BZa22leDOZXX%2BXXIU%3D
```

Aquí se muestra un ejemplo de la misma cadena de conexión con codificación de caracteres especiales:

```
BlobEndpoint=https://storagesample.blob.core.windows.net;
SharedAccessSignature=sv=2015-04-05&amp;sr=b&amp;si=tutorial-policy-635959936145100803&amp;sig=9aCzs76n0E7y5BpEi2GvsSv433BZa22leDOZXX%2BXXIU%3D
```

### <a name="account-sas-example"></a>Ejemplo de SAS de cuenta
Este es un ejemplo de una cadena de conexión que incluye un SAS de cuenta para el almacenamiento de blobs y archivos: Tenga en cuenta que se especifican los puntos de conexión para ambos servicios:

```
BlobEndpoint=https://storagesample.blob.core.windows.net;
FileEndpoint=https://storagesample.file.core.windows.net;
SharedAccessSignature=sv=2015-07-08&sig=iCvQmdZngZNW%2F4vw43j6%2BVz6fndHF5LI639QJba4r8o%3D&spr=https&st=2016-04-12T03%3A24%3A31Z&se=2016-04-13T03%3A29%3A31Z&srt=s&ss=bf&sp=rwl
```

Aquí se muestra un ejemplo de la misma cadena de conexión con codificación de dirección URL:

```
BlobEndpoint=https://storagesample.blob.core.windows.net;
FileEndpoint=https://storagesample.file.core.windows.net;
SharedAccessSignature=sv=2015-07-08&amp;sig=iCvQmdZngZNW%2F4vw43j6%2BVz6fndHF5LI639QJba4r8o%3D&amp;spr=https&amp;st=2016-04-12T03%3A24%3A31Z&amp;se=2016-04-13T03%3A29%3A31Z&amp;srt=s&amp;ss=bf&amp;sp=rwl
```

