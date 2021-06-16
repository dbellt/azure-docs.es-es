---
title: archivo de inclusión
description: archivo de inclusión
services: azure-communication-services
author: lakshmans
manager: ankita
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 03/11/2021
ms.topic: include
ms.custom: include file
ms.author: lakshmans
ms.openlocfilehash: 86f26e7fbe96284dee5eb606ede22ae63701f309
ms.sourcegitcommit: c385af80989f6555ef3dadc17117a78764f83963
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/04/2021
ms.locfileid: "111430511"
---
Introducción a Azure Communication Services con SMS SDK de Communication Services para Python para enviar mensajes de texto.

Este inicio rápido supone un pequeño costo en su cuenta de Azure.

> [!NOTE]
> Busque el código finalizado de este inicio rápido en [GitHub](https://github.com/Azure-Samples/communication-services-python-quickstarts/tree/main/send-sms-quickstart)

## <a name="prerequisites"></a>Requisitos previos

- Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Python](https://www.python.org/downloads/) 2.7 o 3.6+.
- Un recurso activo de Communication Services y una cadena de conexión. [Cree un recurso de Communication Services](../../create-communication-resource.md).
- Un número de teléfono habilitado para SMS. [Obtención de un número de teléfono](../get-phone-number.md).

### <a name="prerequisite-check"></a>Comprobación de requisitos previos

- En una ventana de terminal o de comandos, ejecute `python --version` para comprobar que Python está instalado.
- Para ver los números de teléfono asociados a su recurso de Communication Services, inicie sesión en [Azure Portal](https://portal.azure.com/), busque el recurso de Communication Services y abra la pestaña **números de teléfono** en el panel de navegación izquierdo.

## <a name="setting-up"></a>Instalación

### <a name="create-a-new-python-application"></a>Creación de una nueva aplicación de Python

Abra la ventana de comandos o el terminal, cree un nuevo directorio para la aplicación y navegue hasta él.

```console
mkdir sms-quickstart && cd sms-quickstart
```

Use un editor de texto para crear un archivo denominado **send-sms.py** en el directorio raíz del proyecto y agregue la estructura del programa, incluido el control de excepciones básico. En las secciones siguientes agregará todo el código fuente de esta guía de inicio rápido a este archivo.

```python
import os
from azure.communication.sms import SmsClient

try:
    # Quickstart code goes here
except Exception as ex:
    print('Exception:')
    print(ex)
```

### <a name="install-the-package"></a>Instalar el paquete

Aún en el directorio de aplicaciones, instale el paquete de SMS SDK de Azure Communication Services para Python con el comando `pip install`.

```console
pip install azure-communication-sms
```

## <a name="object-model"></a>Modelo de objetos

Las clases e interfaces siguientes controlan algunas de las características principales de SMS SDK de Azure Communication Services para Python.

| Nombre                                  | Descripción                                                  |
| ------------------------------------- | ------------------------------------------------------------ |
| SmsClient | Esta clase es necesaria para la funcionalidad de los SMS. Cree una instancia de esta clase con la información de suscripción y úsela para enviar mensajes de texto.                                                                                                                 |
| SmsSendResult               | Esta clase contiene el resultado del servicio SMS.                                          |

## <a name="authenticate-the-client"></a>Autenticar el cliente

Cree una instancia de la clase **SmsClient** con la cadena de conexión. Aprenda a [administrar la cadena de conexión del recurso](../../create-communication-resource.md#store-your-connection-string).

```python
# Create the SmsClient object which will be used to send SMS messages
sms_client = SmsClient.from_connection_string(<connection_string>)
```
Para simplificar, en este inicio rápido usamos cadenas de conexión, pero en entornos de producción se recomiendan las [identidades administradas](../../../quickstarts/managed-identity.md), ya que son más seguras y se administran a escala.


## <a name="send-a-11-sms-message"></a>Envío de un SMS de un remitente a un solo destinatario

Para enviar un SMS a un solo destinatario, llame al método ```send``` desde el **SmsClient** con el número de teléfono de un solo destinatario. También puede incluir parámetros opcionales para especificar si el informe de entregas debe estar habilitado y establecer etiquetas personalizadas. Agregue este código al final del bloque `try` en **send-sms.py**:

```python

# calling send() with sms values
sms_responses = sms_client.send(
    from_="<from-phone-number>",
    to="<to-phone-number>",
    message="Hello World via SMS",
    enable_delivery_report=True, # optional property
    tag="custom-tag") # optional property

```

Debe reemplazar `<from-phone-number>` por un número de teléfono habilitado para SMS asociado al recurso de servicio de comunicación y `<to-phone-number>` por el número de teléfono al que desea enviar un mensaje.

> [!WARNING]
> Tenga en cuenta que los números de teléfono se deben proporcionar en formato estándar internacional E.164. (por ejemplo, +14255550123).

## <a name="send-a-1n-sms-message"></a>Envío de un SMS de un remitente a varios destinatarios

Para enviar un SMS a una lista de destinatarios, llame al método ```send``` desde **SmsClient** con una lista de números de teléfono de destinatarios. También puede incluir parámetros opcionales para especificar si el informe de entregas debe estar habilitado y establecer etiquetas personalizadas. Agregue este código al final del bloque `try` en **send-sms.py**:

```python

# calling send() with sms values
sms_responses = sms_client.send(
    from_="<from-phone-number>",
    to=["<to-phone-number-1>", "<to-phone-number-2>"],
    message="Hello World via SMS",
    enable_delivery_report=True, # optional property
    tag="custom-tag") # optional property

```

Debe reemplazar `<from-phone-number>` por un número de teléfono habilitado para SMS asociado al servicio de comunicación y `<to-phone-number-1>` `<to-phone-number-2>` con los números de teléfono a los que desea enviar mensajes.

> [!WARNING]
> Tenga en cuenta que los números de teléfono se deben proporcionar en formato estándar internacional E.164. (por ejemplo, +14255550123).

## <a name="optional-parameters"></a>Parámetros opcionales

El parámetro `enable_delivery_report` es un parámetro opcional que puede usar para configurar los informes de entrega. Resulta útil para los escenarios en los que quiere emitir eventos cuando se entregan mensajes SMS. Consulte el inicio rápido [Control de eventos SMS](../handle-sms-events.md) a fin de configurar los informes de entrega para los mensajes SMS.

El parámetro `tag` es un parámetro opcional que puede usar para aplicar una etiqueta al informe de entregas.

## <a name="run-the-code"></a>Ejecución del código
Ejecute la aplicación desde el directorio de la aplicación con el comando `python`.

```console
python send-sms.py
```

El script de Python completo debería tener un aspecto similar al siguiente:

```python

import os
from azure.communication.sms import SmsClient

try:
    # Create the SmsClient object which will be used to send SMS messages
    sms_client = SmsClient.from_connection_string("<connection string>")
    # calling send() with sms values
    sms_responses = sms_client.send(
       from_="<from-phone-number>",
       to="<to-phone-number>",
       message="Hello World via SMS",
       enable_delivery_report=True, # optional property
       tag="custom-tag") # optional property

except Exception as ex:
    print('Exception:')
    print(ex)
```
