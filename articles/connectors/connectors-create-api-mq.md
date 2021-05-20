---
title: Conexión con el servidor IBM MQ
description: Conéctese a un servidor MQ local o en Azure desde un flujo de trabajo mediante Azure Logic Apps.
services: logic-apps
ms.suite: integration
author: ChristopherHouser
ms.author: chrishou
ms.reviewer: valthom, estfan, logicappspm
ms.topic: article
ms.date: 04/26/2021
tags: connectors
ms.openlocfilehash: 80ff8508caefd355f00a0407df0d9a65c76c999a
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108742110"
---
# <a name="connect-to-an-ibm-mq-server-from-a-workflow-in-azure-logic-apps"></a>Conexión a un servidor IBM MQ desde un flujo de trabajo en Azure Logic Apps

El conector de MQ le ayuda a conectar los flujos de trabajo de la aplicación lógica a un servidor IBM MQ en el entorno local o en Azure. A continuación, puede hacer que los flujos de trabajo reciban y envíen mensajes almacenados en el servidor MQ. En este artículo se proporciona una guía de introducción al uso del conector de MQ en la que se muestra cómo conectarse al servidor MQ y agregar una acción de MQ al flujo de trabajo. Por ejemplo, puede empezar por examinar un único mensaje en una cola y luego intentar otras acciones.

Este conector incluye un cliente de MQ de Microsoft que se comunica con un servidor IBM MQ remoto mediante una red TCP/IP. Puede conectarse a las siguientes versiones de IBM WebSphere MQ:

* MQ 7.5
* MQ 8.0
* MQ 9.0, 9.1 y 9.2

<a name="available-operations"></a>

## <a name="available-operations"></a>Operaciones disponibles

El conector de IBM MQ proporciona acciones, pero no desencadenadores.

* Azure Logic Apps multiinquilino: al crear un flujo de trabajo de una aplicación lógica basada en el consumo, puede conectarse a un servidor MQ mediante el conector de MQ *administrado*.

* Azure Logic Apps para un solo inquilino (versión preliminar): al crear un flujo de trabajo de aplicación lógica de la versión preliminar, puede conectarse a un servidor MQ mediante el conector de MQ administrado o las operaciones *integradas* de MQ (versión preliminar).

Para más información sobre la diferencia entre un conector administrado y las operaciones integradas, consulte [Términos clave de Logic Apps](../logic-apps/logic-apps-overview.md#logic-app-concepts).

#### <a name="managed"></a>[Administrado](#tab/managed)

En la lista siguiente solo se describen algunas de las operaciones administradas disponibles para MQ:

* Examinar un único mensaje o una matriz de mensajes sin eliminarlos del servidor MQ. Para varios mensajes, puede especificar el número máximo de mensajes que se devolverán por cada lote. De lo contrario, se devuelven todos los mensajes.
* Eliminar un mensaje individual o una matriz de mensajes del servidor MQ.
* Recibir un único mensaje o una matriz de mensajes y, a continuación, eliminarlos del servidor MQ.
* Enviar un único mensaje al servidor MQ.

Para obtener información sobre todas las operaciones del conector administrado y otra información técnica, como propiedades, límites, etc., consulte la [página de referencia del conector de MQ](/connectors/mq/).

#### <a name="built-in-preview"></a>[Integrado (versión preliminar)](#tab/built-in)

En la lista siguiente solo se describen algunas de las operaciones integradas disponibles para MQ:

* Recibir un mensaje individual o una matriz de mensajes del servidor MQ. Para varios mensajes, puede especificar el número máximo de mensajes que se devolverán por cada lote y el tamaño máximo del lote en KB.
* Enviar un mensaje individual o una matriz de mensajes al servidor MQ.

Estas operaciones integradas de MQ también tienen las siguientes funcionalidades, además de las ventajas de todas las demás funcionalidades de las aplicaciones lógicas en el [servicio Logic Apps para inquilino único](../logic-apps/logic-apps-overview-preview.md):

* Cifrado de los datos en tránsito mediante Seguridad de la capa de transporte (TLS)
* Codificación de mensajes para las operaciones de envío y recepción
* Compatibilidad con la integración de redes virtuales de Azure cuando la aplicación lógica usa el plan Premium de Azure Functions

---

## <a name="limitations"></a>Limitaciones

El conector de MQ no usa el campo de **formato** del mensaje ni realiza ninguna conversión de juegos de caracteres. El conector solo coloca los datos que aparecen en el campo de mensaje en un mensaje JSON y envía el mensaje.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta y una suscripción de Azure. Si no tiene una suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Si usa un servidor MQ local, [instale la puerta de enlace de datos local](../logic-apps/logic-apps-gateway-install.md) en un servidor de la red. Para que el conector de MQ funcione, el servidor donde se instala la puerta de enlace de datos local también debe tener instalado .NET Framework 4.6.

  Después de instalar la puerta de enlace, debe crear también un recurso de puerta de enlace de datos en Azure. El conector de MQ usa este recurso para acceder al servidor MQ. Para más información, consulte [Conexión a orígenes de datos locales desde Azure Logic Apps](../logic-apps/logic-apps-gateway-connection.md). 

  > [!NOTE]
  > No se necesita la puerta de enlace en los escenarios siguientes:
  > 
  > * Va a usar las operaciones integradas, no el conector administrado.
  > * El servidor MQ está disponible públicamente o está disponible en Azure.

* El flujo de trabajo de la aplicación lógica desde donde quiere acceder al servidor MQ. El recurso de aplicación lógica debe tener la misma ubicación que el recurso de puerta de enlace en Azure.

  El conector de MQ no tiene desencadenadores, por lo que el flujo de trabajo ya debe comenzar con un desencadenador o primero debe agregar un desencadenador al flujo de trabajo. Por ejemplo, puede usar el [desencadenador de periodicidad](../connectors/connectors-native-recurrence.md).

  Si no está familiarizado con Azure Logic Apps, pruebe este [inicio rápido para crear un flujo de trabajo de aplicación lógica de ejemplo](../logic-apps/quickstart-create-first-logic-app-workflow.md) que se ejecuta en el servicio Logic Apps multiinquilino.

<a name="create-connection"></a>

## <a name="create-an-mq-connection"></a>Creación de una conexión de MQ 

Cuando agregue una acción de MQ por primera vez, se le pedirá que cree una conexión con el servidor MQ.

> [!NOTE]
> Actualmente, el conector de MQ solo admite la autenticación de servidor, no la de cliente. Para obtener más información, consulte [Connection and authentication problems](#connection-problems) (Problemas de conexión y autenticación).

#### <a name="managed"></a>[Administrado](#tab/managed)

1. Si se conecta a un servidor MQ local, seleccione **Conectarse mediante una puerta de enlace de datos local**.

1. Proporcione la información de conexión para el servidor MQ.

   | Propiedad | Local o Azure | Descripción |
   |----------|----------------------|-------------|
   | **Puertas de enlace** | Solo en entornos locales | Seleccione **Connect via on-premises data gateway** (Conectarse a través de la puerta de enlace de datos local). |
   | **Nombre de la conexión** | Ambos | El nombre que usar para su conexión |
   | **Server** | Ambos | Uno de los valores siguientes: <p><p>- Nombre de host del servidor MQ <br>- Dirección IP seguida de dos puntos y el número de puerto |
   | **Nombre del administrador de colas** | Ambos | Administrador de colas que quiere utilizar |
   | **Nombre del canal** | Ambos | Canal para conectarse al administrador de colas |
   | **Nombre predeterminado de la cola** | Ambos | Nombre predeterminado de la cola |
   | **Conectar como** | Ambos | Nombre de usuario para conectarse al servidor MQ |
   | **Nombre de usuario** | Ambos | Credenciales de nombre de usuario |
   | **Contraseña** | Ambos | Credenciales de contraseña |
   | **¿Quiere habilitar SSL?** | Solo en entornos locales | Usar Seguridad de la capa de transporte (TLS) o Capa de sockets seguros (SSL) |
   | **Puerta de enlace: suscripción** | Solo en entornos locales | Suscripción de Azure asociada al recurso de puerta de enlace en Azure |
   | **Puerta de enlace: puerta de enlace de conexión** | Solo en entornos locales | Recurso de puerta de enlace que se va a usar |
   ||||

   Por ejemplo:

   ![Captura de pantalla que muestra los detalles de conexión de MQ administrado.](media/connectors-create-api-mq/managed-connection-properties.png)

1. Seleccione **Crear** cuando haya terminado.

#### <a name="built-in-preview"></a>[Integrado (versión preliminar)](#tab/built-in)

1. Proporcione la información de conexión para el servidor MQ.

   | Propiedad | Local o Azure | Descripción |
   |----------|----------------------|-------------|
   | **Nombre de la conexión** | Ambos | El nombre que usar para su conexión |
   | **Nombre del servidor** | Ambos | Nombre o dirección IP del servidor MQ |
   | **Número de puerto** | Ambos | Número de puerto TCP para conectarse al administrador de colas en el host |
   | **Canal** | Ambos | Canal para conectarse al administrador de colas |
   | **Nombre del administrador de colas** | Ambos | Administrador de colas que quiere utilizar |
   | **Nombre predeterminado de la cola** | Ambos | Nombre predeterminado de la cola |
   | **Conectar como** | Ambos | Nombre de usuario para conectarse al servidor MQ |
   | **Nombre de usuario** | Ambos | Credenciales de nombre de usuario |
   | **Contraseña** | Ambos | Credenciales de contraseña |
   | **Usar TLS** | Ambos | Usar Seguridad de la capa de transporte (TLS) |
   ||||

   Por ejemplo:

   ![Captura de pantalla que muestra los detalles de conexión de MQ integrado.](media/connectors-create-api-mq/built-in-connection-properties.png)

1. Seleccione **Crear** cuando haya terminado.

---

<a name="add-action"></a>

## <a name="add-an-mq-action"></a>Adición de una acción de MQ

En Azure Logic Apps, una acción sigue al desencadenador o a otra acción y realiza alguna operación en el flujo de trabajo. En los pasos siguientes se describe la manera general de agregar una acción, por ejemplo, **Examinar un único mensaje**.

1. En el Diseñador de aplicaciones lógicas, abra el flujo de trabajo, si aún no está abierto.

1. En el desencadenador u otra acción, agregue un nuevo paso.

   Para agregar un paso entre los ya existentes, mueva el mouse sobre la flecha. Seleccione el signo más (+) que aparece y, a continuación, seleccione **Agregar una acción**.

1. En el cuadro de búsqueda de operaciones, escriba `mq`. En la lista de acciones, seleccione la acción **Examinar mensaje**.

1. Si se le pide que cree una conexión al servidor MQ, [proporcione la información de conexión solicitada](#create-connection).

1. En la acción, proporcione los valores de las propiedades que necesita la acción.

   Para especificar propiedades adicionales, abra la lista **Agregar nuevo parámetro** y seleccione las propiedades que quiera agregar.

1. Cuando esté listo, seleccione **Guardar** en la barra de herramientas del diseñador.

1. Para probar el flujo de trabajo, seleccione **Ejecutar** en la barra de herramientas del diseñador.

   Una vez que finaliza la ejecución, el diseñador muestra el historial de ejecución del flujo de trabajo junto con el estado del paso.

1. Para revisar las entradas y salidas de cada paso que se ejecutó (no omitido), expanda o seleccione el paso.

   * Para revisar más detalles de entrada, seleccione **Mostrar entradas sin procesar**.
   * Para revisar más detalles de salida, seleccione **Mostrar salidas sin procesar**. Si establece **IncludeInfo** en **true**, se incluirá una salida adicional.

## <a name="troubleshoot-problems"></a>Solucionar problemas

### <a name="failures-with-browse-or-receive-actions"></a>Errores con las acciones de exploración o recepción

Si ejecuta una acción de exploración o recepción en una cola vacía, se produce un error en la acción con las siguientes salidas de encabezado:

![Error de MQ que indica que no hay mensajes](media/connectors-create-api-mq/mq-no-message-error.png)

<a name="connection-problems"></a>

### <a name="connection-and-authentication-problems"></a>Problemas de conexión y autenticación

Cuando el flujo de trabajo intente conectarse al servidor MQ local, puede que aparezca este error:

`"MQ: Could not Connect the Queue Manager '<queue-manager-name>': The Server was expecting an SSL connection."`

* Si usa el conector de MQ directamente en Azure, el servidor MQ debe usar un certificado emitido por una [entidad de certificación](https://www.ssl.com/faqs/what-is-a-certificate-authority/) de confianza.

* El servidor MQ requiere que defina la especificación de cifrado que se va a usar para las conexiones TLS. Sin embargo, por motivos de seguridad y para incluir los mejores conjuntos de seguridad, el sistema operativo Windows envía un conjunto de especificaciones de cifrado admitidas.

  El sistema operativo donde se ejecuta el servidor MQ elige los conjuntos que se usarán. Para que la configuración coincida, debe cambiar la configuración del servidor MQ para que la especificación de cifrado coincida con la opción elegida en la negociación de TLS.

  Al intentar la conexión, el servidor MQ registra un mensaje de evento que indica que se produjo un error en el intento de conexión porque el servidor MQ ha elegido una especificación de cifrado incorrecta. El mensaje del evento contiene la especificación de cifrado que el servidor MQ eligió de la lista. Actualice la especificación de cifrado en la configuración del canal para que coincida con la especificación de cifrado del mensaje del evento.

## <a name="connector-reference"></a>Referencia de conectores

Para obtener información sobre todas las operaciones del conector administrado y otra información técnica, como propiedades, límites, etc., consulte la [página de referencia del conector de MQ](/connectors/mq/).

## <a name="next-steps"></a>Pasos siguientes

* Obtenga más información sobre otros [conectores de Logic Apps](../connectors/apis-list.md)
