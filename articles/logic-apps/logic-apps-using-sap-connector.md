---
title: Conexión a sistemas SAP
description: Cómo acceder a recursos de SAP y administrarlos mediante la automatización de flujos de trabajo con Azure Logic Apps
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: estfan, daviburg, logicappspm
ms.topic: article
ms.date: 03/30/2021
tags: connectors
ms.openlocfilehash: ec5046e40b6fade0e4d56023c404cc736a46f105
ms.sourcegitcommit: f5448fe5b24c67e24aea769e1ab438a465dfe037
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 03/30/2021
ms.locfileid: "105969111"
---
# <a name="connect-to-sap-systems-from-azure-logic-apps"></a>Conexión a sistemas SAP desde Azure Logic Apps

En este artículo, se explica cómo acceder a los recursos de SAP desde Logic Apps mediante el [conector de SAP](/connectors/sap/).

## <a name="prerequisites"></a>Requisitos previos

* Suscripción a Azure. Si aún no tiene ninguna suscripción de Azure, [regístrese para obtener una cuenta gratuita de Azure](https://azure.microsoft.com/free/).

* Una aplicación lógica desde la que quiere acceder a los recursos de SAP. Si no está familiarizado con Logic Apps, consulte la [información general del servicio de Logic Apps](../logic-apps/logic-apps-overview.md) y la [guía de inicio rápido para crear su primera aplicación lógica en Azure Portal](../logic-apps/quickstart-create-first-logic-app-workflow.md).

    * Si ha usado una versión anterior del conector de SAP que ha quedado en desuso, debe [migrar al conector actual](#migrate-to-current-connector) para poder conectarse al servidor de SAP.

    * Si está ejecutando la aplicación lógica en instancias multiinquilino de Azure, consulte los [requisitos previos de instancias multiinquilino](#multi-tenant-azure-prerequisites).

    * Si está ejecutando la aplicación lógica en un [entorno del servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md) de nivel prémium, consulte los [requisitos previos de ISE](#ise-prerequisites).

* Un [servidor de aplicaciones de SAP](https://wiki.scn.sap.com/wiki/display/ABAP/ABAP+Application+Server) o [servidor de mensajes de SAP](https://help.sap.com/saphelp_nw70/helpdata/en/40/c235c15ab7468bb31599cc759179ef/frameset.htm) para el que quiere obtener acceso desde Logic Apps. Para más información sobre los servidores de SAP y las acciones de SAP que puede usar con el conector, consulte [Compatibilidad de SAP](#sap-compatibility).

    * Debe configurar el servidor SAP para permitir el uso de RFC. Para obtener más información, consulte la siguiente nota de SAP: [460089: perfiles de autorización mínimos para programas RFC externos](https://launchpad.support.sap.com/#/notes/460089). 

* Contenido del mensaje que puede enviar al servidor de SAP, como un archivo de IDoc de ejemplo. Este contenido debe estar en formato XML e incluir el espacio de nombres para la acción de SAP que se va a utilizar. Puede [enviar documentos IDoc con un esquema de archivo plano ajustándolos en un sobre XML](#send-flat-file-idocs).

* Para usar el desencadenador **Cuando se recibe un mensaje de SAP**, también debe realizar estos pasos:

    * Configure los permisos de seguridad de la puerta de enlace de SAP con esta opción: `"TP=Microsoft.PowerBI.EnterpriseGateway HOST=<gateway-server-IP-address> ACCESS=*"`.

    * Configure el registro de seguridad de la puerta de enlace de SAP para buscar la lista de control de acceso (ACL). Para más información, vea el [tema de ayuda de SAP para configurar el registro de la puerta de enlace](https://help.sap.com/erp_hcm_ias2_2015_02/helpdata/en/48/b2a710ca1c3079e10000000a42189b/frameset.htm). Si no, recibirá este error: `"Registration of tp Microsoft.PowerBI.EnterpriseGateway from host <host-name> not allowed"`.

    > [!NOTE]
    > Este desencadenador utiliza la misma ubicación de URI para renovar y cancelar una suscripción de webhook. La operación de renovación usa el método `PATCH` de HTTP, mientras que la operación de cancelación usa el método `DELETE` de HTTP. Este comportamiento puede provocar que una operación de renovación aparezca como una operación de cancelación en el historial del desencadenador, pero la operación sigue siendo una renovación porque el desencadenador usa `PATCH` como método HTTP, no `DELETE`.

### <a name="sap-compatibility"></a>Compatibilidad de SAP

El conector de SAP es compatible con los siguientes tipos de sistemas SAP:

* Sistemas SAP basados en HANA locales y basados en la nube, como S/4 HANA.

* Sistemas SAP clásicos locales, como R/3 y ECC.

El conector de SAP admite los siguientes tipos de integración de datos y mensajes de los sistemas basados en SAP NetWeaver:

* Documento intermedio (IDoc)

* Interfaz de programación de aplicaciones empresariales (BAPI)

* Llamada a función remota (RFC) y RFC transaccional (tRFC)

El conector de SAP emplea la [biblioteca de SAP .NET Connector (NCo)](https://support.sap.com/en/product/connectors/msnet.html). Puede usar las siguientes acciones y desencadenadores de SAP con el conector:

* Acción **Enviar un mensaje a SAP** para [enviar iDoc a través de tRFC](#send-idoc-action), que puede usar para:

    * [Llamar a las funciones de BAPI mediante RFC](#call-bapi-action)

    * Llamar a RFC/tRFC en sistemas SAP

    * Crear o cerrar sesiones con estado

    * Confirmar o revertir transacciones de BAPI

    * Confirmar un identificador de transacción

    * Enviar IDoc, obtener el estado de un IDoc a partir de su número y obtener una lista de iDoc para una transacción

    * Leer una tabla de SAP

* **Cuando se recibe un mensaje del desencadenador de SAP**, que puede usar para:

    * Recibir IDoc a través de tRFC

    * Llamar a las funciones de BAPI mediante RFC

    * Llamar a RFC/tRFC en sistemas SAP

* Acción **Generar esquemas**, que puede usar para generar esquemas de artefactos SAP para IDoc, BAPI o RFC.

Para usar estas acciones de SAP, primero debe autenticar la conexión con un nombre de usuario y una contraseña. El conector de SAP también admite [comunicaciones de red seguras (SNC)](https://help.sap.com/doc/saphelp_nw70/7.0.31/e6/56f466e99a11d1a5b00000e835363f/content.htm?no_cache=true). SNC puede usarse con el inicio de sesión único (SSO) de SAP NetWeaver o con funcionalidades de seguridad adicionales de productos externos. Si usa SNC, consulte los [requisitos previos de comunicaciones de red seguras](#snc-prerequisites).

### <a name="migrate-to-current-connector"></a>Migración al conector actual

Los conectores anteriores del servidor de aplicaciones de SAP y el servidor de mensajes de SAP están en desuso desde el 29 de febrero de 2020. Para migrar al conector de SAP actual, siga estos pasos:

1. Actualice la [puerta de enlace de datos local](https://www.microsoft.com/download/details.aspx?id=53127) a la versión actual. Para más información, consulte [Instalación de una puerta de enlace de datos local para Azure Logic Apps](../logic-apps/logic-apps-gateway-install.md).

1. En la aplicación lógica que usa el conector de SAP anterior, elimine la acción **Enviar a SAP**.

1. Agregue la acción **Enviar mensaje a SAP** desde el conector de SAP más reciente.

1. Vuelva a conectarse al sistema SAP en la nueva acción.

1. Guarde la aplicación lógica.

### <a name="multi-tenant-azure-prerequisites"></a>Requisitos previos de instancias multiinquilino de Azure

Estos requisitos previos se aplican si la aplicación lógica se ejecuta en instancias multiinquilino de Azure. El conector de SAP administrado no se ejecuta de forma nativa en un [ISE](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md).

> [!TIP]
> Si usa un ISE de nivel prémium, puede usar el conector del ISE de SAP en lugar del conector de SAP administrado. Para más información, consulte los [Requisitos previos del Entorno del servicio de integración (ISE)](#ise-prerequisites).

El conector de SAP administrado se integra con los sistemas SAP locales a través de la [puerta de enlace de datos local](../logic-apps/logic-apps-gateway-connection.md). Por ejemplo, en escenarios de envío de mensajes, al enviar un mensaje desde una aplicación lógica hasta un sistema SAP, la puerta de enlace de datos actúa como cliente RFC y reenvía las solicitudes recibidas de la aplicación lógica a SAP. Del mismo modo, en escenarios de recepción de mensajes, la puerta de enlace de datos actúa como servidor RFC que recibe solicitudes de SAP y las reenvía a la aplicación lógica.

* [Descargue e instale la puerta de enlace de datos local](../logic-apps/logic-apps-gateway-install.md) en un equipo host o máquina virtual que exista en la misma red virtual que el sistema SAP al que se va a conectar.

* [Cree un recurso de puerta de enlace de Azure](../logic-apps/logic-apps-gateway-connection.md#create-azure-gateway-resource) para la puerta de enlace en Azure Portal. Esta puerta de enlace le ayuda a acceder de forma segura a los datos y recursos que se encuentran en el entorno local. Asegúrese de usar una versión compatible de la puerta de enlace.

    * Si tiene problemas con la puerta de enlace, intente [actualizar a la versión más reciente](https://aka.ms/on-premises-data-gateway-installer), que puede incluir actualizaciones para resolver el problema.

* [Descargue e instale la biblioteca cliente de SAP más actualizada](#sap-client-library-prerequisites) en el mismo equipo local que la puerta de enlace de datos local.

### <a name="ise-prerequisites"></a>Requisitos previos del Entorno del servicio de integración (ISE)

Estos requisitos previos se aplican si está ejecutando la aplicación lógica en un ISE de nivel prémium. Sin embargo, no se aplican a las aplicaciones lógicas que se ejecutan en un ISE de nivel de desarrollador. Un ISE proporciona acceso a los recursos que están protegidos por una red virtual de Azure y ofrece otros conectores nativos del ISE que permiten a las aplicaciones lógicas acceder directamente a los recursos locales sin usar la puerta de enlace de datos local.

1. Si aún no tiene una cuenta de Azure Storage y un contenedor de blobs, cree ese contenedor mediante [Azure Portal](../storage/blobs/storage-quickstart-blobs-portal.md) o el [Explorador de Azure Storage](../storage/blobs/storage-quickstart-blobs-storage-explorer.md).

1. [Descargue e instale la biblioteca más actualizada de clientes de SAP](#sap-client-library-prerequisites) en el equipo local. Debe tener los siguientes archivos de ensamblado:

   * libicudecnumber.dll

   * rscp4n.dll

   * sapnco.dll

   * sapnco_utils.dll

1. Cree un archivo .zip que incluya estos archivos de ensamblado. Cargue el paquete en el contenedor de blobs en Azure Storage.

1. En Azure Portal o el Explorador de Azure Storage, vaya a la ubicación del contenedor donde cargó el archivo. zip.

1. Copie la dirección URL de la ubicación del contenedor. Asegúrese de incluir el token de firma de acceso compartido (SAS) para autorizarlo. De lo contrario, se producirá un error en la implementación del conector del ISE de SAP.

1. Instale e implemente el conector de SAP en el ISE. Para más información, vea [Adición de conectores del ISE](../logic-apps/add-artifacts-integration-service-environment-ise.md#add-ise-connectors-environment).

   1. En [Azure Portal](https://portal.azure.com), busque y abra el ISE.

   1. En el menú del ISE, seleccione **Conectores administrados** &gt; **Agregar**. En la lista de conectores, busque y seleccione **SAP**.

   1. En el panel **Add a new managed connector** (Agregar un nuevo conector administrado), en el cuadro **SAP package** (Paquete de SAP), pegue la dirección URL del archivo .zip que contiene los ensamblados de SAP. Vuelva a asegurarse de incluir el token de SAS.
 
  1. Haga clic en **Crear** para terminar de crear el conector del ISE.

1. Si la instancia de SAP y el ISE se encuentran en distintas redes virtuales, también debe [poner en el mismo nivel esas redes](../virtual-network/tutorial-connect-virtual-networks-portal.md) para que estén conectadas.

### <a name="sap-client-library-prerequisites"></a>Requisitos previos de la biblioteca de cliente de SAP

Estos son los requisitos previos para la biblioteca cliente de SAP que se usa con el conector.

* Instale la versión más reciente: [SAP Connector (NCo 3.0) para Microsoft .NET 3.0.22.0 compilado con .NET Framework 4.0  - Windows de 64 bits (x64)](https://support.sap.com/en/product/connectors/msnet.html). Las versiones anteriores de SAP NCo pueden experimentar problemas cuando se envía más de un mensaje de IDoc al mismo tiempo. Esta condición bloquea todos los mensajes posteriores que se envían al destino de SAP, lo que hace que se agote el tiempo de espera de los mensajes.

* Debe instalar la versión de 64 bits para la biblioteca cliente de SAP, ya que la puerta de enlace de datos solo se ejecuta en sistemas de 64 bits. Al instalar la versión de 32 bits no admitida, se produce un error de "imagen incorrecta".

* Copie los archivos de ensamblado de la carpeta de instalación predeterminada a otra ubicación, según el escenario que se indica a continuación.

    * En el caso de las aplicaciones lógicas que se ejecutan en un ISE, siga los [Requisitos previos del Entorno del servicio de integración (ISE)](#ise-prerequisites) en su lugar.

    * En el caso de las aplicaciones lógicas que se ejecutan en instancias multiinquilino de Azure y usan la puerta de enlace de datos local, copie los archivos de ensamblado de la carpeta de instalación de la puerta de enlace de datos. 

        
        * Si se produce un error en la conexión de SAP con un mensaje que le pide que **compruebe la información de la cuenta o los permisos y pruebe otra vez**, asegúrese de que se han copiado los archivos de ensamblado en la carpeta de instalación de la puerta de enlace de datos.
        
        * Para solucionar otros problemas, use el [visor de registro de enlaces de ensamblados de .NET](/dotnet/framework/tools/fuslogvw-exe-assembly-binding-log-viewer). Esta herramienta le permite comprobar que los archivos de ensamblado se encuentran en la ubicación correcta. 
        
        * Si lo prefiere, seleccione la opción **Global Assembly Cache registration** (Registro de la caché global de ensamblados) al instalar la biblioteca cliente de SAP.

Tenga en cuenta las siguientes relaciones entre la biblioteca cliente de SAP, .NET Framework, el entorno de ejecución de .NET y la puerta de enlace:

* Tanto Microsoft SAP Adapter como el servicio de host de la puerta de enlace usan .NET Framework 4.7.2.

* SAP NCo para .NET Framework 4.0 funciona con procesos que usan las versiones 4.0 a 4.8 del entorno de ejecución de .NET.

* NCo de SAP para .NET Framework 2.0 funciona con procesos que usan las versiones 2.0 a 3.5 del entorno de ejecución de .NET y ya no funciona con la puerta de enlace más reciente.

### <a name="snc-prerequisites"></a>Requisitos previos de SNC

Si usa una puerta de enlace de datos local con SNC opcional, que solo se admite en instancias multiinquilino de Azure, debe configurar estas opciones adicionales.

Si usa SNC con el inicio de sesión único (SSO), asegúrese de que el servicio de la puerta de enlace de datos se ejecuta con un usuario asignado con el usuario de SAP. Para cambiar la cuenta predeterminada, seleccione **Cambiar cuenta** y escriba las credenciales de usuario.

![Captura de pantalla de la configuración de puerta de enlace de datos local en Azure Portal que muestra la página de configuración del servicio con el botón para cambiar la cuenta de servicio de puerta de enlace seleccionada.](./media/logic-apps-using-sap-connector/gateway-account.png)

Si habilita SNC a través de un producto de seguridad externo, copie la biblioteca o los archivos de SNC en el mismo equipo donde está instalada la puerta de enlace de datos. Algunos ejemplos de productos SNC incluyen [sapseculib](https://help.sap.com/saphelp_nw74/helpdata/en/7a/0755dc6ef84f76890a77ad6eb13b13/frameset.htm), Kerberos y NTLM. Para más información sobre la habilitación de SNC para la puerta de enlace de datos, vea [Habilitación de las comunicaciones de red seguras](#enable-secure-network-communications).

## <a name="send-idoc-messages-to-sap-server"></a>Envío de mensajes de IDoc al servidor de SAP

Siga estos ejemplos para crear una aplicación lógica que envíe un mensaje de IDoc a un servidor de SAP y devuelva una respuesta:

1. [Cree una aplicación lógica desencadenada por una solicitud HTTP.](#create-http-request-trigger)

1. [Cree una acción en el flujo de trabajo para enviar un mensaje a SAP.](#create-sap-action-to-send-message)

1. [Cree una acción de respuesta HTTP en el flujo de trabajo.](#create-http-response-action)

1. [Cree un patrón de solicitud-respuesta de llamada de función remota (RFC) si usa un RFC para recibir respuestas de SAP ABAP.](#create-rfc-request-response)

1. [Pruebe la aplicación lógica.](#test-logic-app)

### <a name="create-http-request-trigger"></a>Creación del desencadenador de solicitud HTTP

> [!NOTE]
> Cuando una aplicación lógica recibe IDoc de SAP, el [desencadenador de solicitud](../connectors/connectors-native-reqres.md) admite el formato XML sin formato de SAP. Para recibir IDoc como XML sin formato, use el desencadenador **cuando se reciba un mensaje de SAP**. Establezca el parámetro **Formato de IDoc** en **SapPlainXml**.

En primer lugar, cree una aplicación lógica con un punto de conexión en Azure para enviar *solicitudes HTTP POST* a la aplicación lógica. Cuando la aplicación lógica reciba estas solicitudes HTTP, el [desencadenador](../logic-apps/logic-apps-overview.md#logic-app-concepts) se activará y ejecutará el paso siguiente en el flujo de trabajo.

1. En [Azure Portal](https://portal.azure.com), cree una aplicación lógica en blanco que abra el **Diseñador de Logic Apps**.

1. En el cuadro de búsqueda, escriba `http request` como filtro. En la lista **Desencadenadores**, seleccione **Cuando se recibe una solicitud HTTP**.

   ![Captura de pantalla del diseñador de Logic Apps, que muestra un nuevo desencadenador de solicitud HTTP que se va a agregar a la aplicación lógica.](./media/logic-apps-using-sap-connector/add-http-trigger-logic-app.png)

1. Guarde la aplicación lógica para poder generar una dirección URL de punto de conexión para ella. En la barra de herramientas del diseñador, seleccione **Save** (Guardar). La dirección URL del punto de conexión aparece ahora en el desencadenador. 

   ![Captura de pantalla del diseñador de Logic Apps que muestra el desencadenador de solicitud HTTP con la dirección URL POST generada copiándose.](./media/logic-apps-using-sap-connector/generate-http-endpoint-url.png)

### <a name="create-sap-action-to-send-message"></a>Creación de una acción de SAP para enviar el mensaje

A continuación, cree una acción para enviar el mensaje de IDoc a SAP cuando se active el [desencadenador de solicitud HTTP](#create-http-request-trigger).

1. En el diseñador de Logic Apps, seleccione **Nuevo paso**.

   ![Captura de pantalla del diseñador de Logic Apps, que muestra que se está editando la aplicación lógica para agregar un nuevo paso.](./media/logic-apps-using-sap-connector/add-sap-action-logic-app.png)

1. En el cuadro de búsqueda, escriba `sap` como filtro. En la lista **Actions** (Acciones), seleccione **Send message to SAP** (Enviar mensajes a SAP).
  
   ![Captura de pantalla del diseñador de Logic Apps que muestra la selección de la acción "Enviar mensaje a SAP".](media/logic-apps-using-sap-connector/select-sap-send-action.png)

   O bien, puede elegir la pestaña **Enterprise** y seleccionar la acción de SAP.

   ![Captura de pantalla del diseñador de Logic Apps que muestra la selección de la acción "Enviar mensaje a SAP" en la pestaña Enterprise.](media/logic-apps-using-sap-connector/select-sap-send-action-ent-tab.png)

1. Si la conexión ya existe, continúe con el siguiente paso. Si se le pide que cree una nueva conexión, proporcione la siguiente información para conectarse al servidor de SAP local.

   1. Proporcione un nombre para la conexión.

   1. Si usa la puerta de enlace de datos, siga estos pasos:
   
      1. En la sección **Puerta de enlace de datos**, en **Suscripción**, primero seleccione la suscripción de Azure para el recurso de puerta de enlace de datos que creó en el Azure Portal para la instalación de su puerta de enlace de datos.
   
      1. En **Puerta de enlace de conexión**, seleccione su recurso de puerta de enlace de datos en Azure.

   1. Siga proporcionando la información de conexión. Para la propiedad **Tipo de inicio de sesión**, siga el paso en función de si la propiedad está establecida en **Servidor de aplicaciones** o **Grupo**:
   
      * Para **Servidos de aplicaciones**, estas propiedades, que generalmente parecen opcionales, son necesarias:

        ![Creación de conexiones del servidor de aplicaciones de SAP](media/logic-apps-using-sap-connector/create-SAP-application-server-connection.png)

      * Para **Grupo**, estas propiedades, que generalmente parecen opcionales, son necesarias:

        ![Creación de conexiones del servidor de mensajes de SAP](media/logic-apps-using-sap-connector/create-SAP-message-server-connection.png)  

      De forma predeterminada, se usa el establecimiento inflexible de tipos para comprobar si hay valores no válidos, para lo cual se realiza la validación de XML con respecto al esquema. Este comportamiento puede ayudarlo a detectar problemas con antelación. La opción **Escritura segura** está disponible para la compatibilidad con versiones anteriores y solo comprueba la longitud de cadena. Más información sobre la opción [Escritura segura](#safe-typing).

   1. Cuando haya finalizado, seleccione **Crear**.

      Logic Apps configura y comprueba la conexión para asegurarse de que funciona correctamente.

    > [!NOTE]

    > Si recibe el siguiente error, hay un problema con la instalación de la biblioteca cliente de SAP NCo: 
    >
    > **Error en la prueba de conexión. Error: "Error al procesar la solicitud". Detalles del error: "No se puede cargar el archivo o ensamblado "sapnco, Version=3.0.0.42, Culture=neutral, PublicKeyToken 50436dca5c7f7d23" o una de sus dependencias. El sistema no encuentra el archivo especificado".**
    >
    > Asegúrese de [instalar la versión necesaria de la biblioteca cliente de SAP NCo y cumplir todos los demás requisitos previos](#sap-client-library-prerequisites).

1. Ahora, busque y seleccione una acción en el servidor SAP.

   1. En el cuadro **SAP Action** (Acción de SAP), elija el icono de carpeta. Desde la lista de archivos, busque y seleccione el mensaje de SAP que quiere utilizar. Para navegar por la lista, utilice las flechas.

      En este ejemplo se selecciona un IDOC con el tipo **Orders** (Pedidos).

      ![Búsqueda y selección de la acción IDoc](./media/logic-apps-using-sap-connector/SAP-app-server-find-action.png)

      Si no encuentra la acción que quiere, puede escribir manualmente una ruta de acceso, por ejemplo:

      ![Proporcionar manualmente la ruta de acceso a la acción IDoc](./media/logic-apps-using-sap-connector/SAP-app-server-manually-enter-action.png)

      > [!TIP]
      > Proporcione el valor de la **acción de SAP** mediante el editor de expresiones. De este modo, puede usar la misma acción para diferentes tipos de mensajes.

      Para más información sobre las operaciones de IDoc, vea [Esquemas de mensaje para operaciones de IDoc](/biztalk/adapters-and-accelerators/adapter-sap/message-schemas-for-idoc-operations).

   1. Haga clic en el cuadro **Mensaje de entrada** para que aparezca la lista de contenido dinámico. En dicha lista, en **When a HTTP request is received** (Cuando se recibe una solicitud HTTP), seleccione el campo **Cuerpo**.

      En este paso, se incluye el contenido del cuerpo del desencadenador de la solicitud HTTP y se envía la salida al servidor SAP.

      ![Seleccione la propiedad "Cuerpo" en el desencadenador](./media/logic-apps-using-sap-connector/SAP-app-server-action-select-body.png)

      Cuando haya terminado, la acción SAP debe parecerse a este ejemplo:

      ![Completar la acción SAP](./media/logic-apps-using-sap-connector/SAP-app-server-complete-action.png)

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

#### <a name="send-flat-file-idocs"></a>Envío de IDoc de archivos planos

Puede usar documentos IDoc con un esquema de archivo plano ajustándolos en un sobre XML. Para enviar un IDoc de archivo plano, use las instrucciones genéricas para [crear una acción de SAP para enviar el mensaje de IDoc](#create-sap-action-to-send-message) con los siguientes cambios.

1. Para la acción **Enviar mensaje a SAP**, use el URI de acción de SAP `http://microsoft.lobservices.sap/2007/03/Idoc/SendIdoc`.

1. Dé formato a un mensaje de entrada con un sobre XML. Por ejemplo, vea la siguiente carga XML de ejemplo:

```xml
<ReceiveIdoc xmlns="http://Microsoft.LobServices.Sap/2007/03/Idoc/"><idocData>EDI_DC 3000000001017945375750 30INVOIC011BTSVLINV30KUABCABCFPPC LDCA X004010810 4 SAPMSX LSEDI ABCABCFPPC 000d3ae4-723e-1edb-9ca4-cc017365c9fd 20210217054521INVOICINVOIC01ZINVOIC2RE 20210217054520
E2EDK010013000000001017945375000001E2EDK01001000000010 ABCABC1.00000 0060 INVO9988298128 298.000 298.000 LB Z4LR EN 0005065828 L
E2EDKA1 3000000001017945375000002E2EDKA1 000000020 RS ABCABCFPPC 0005065828 ABCABCABC ABCABC Inc. Limited Risk Distributor ABCABC 1950 ABCABCABCA Blvd ABCABAABCAB L5N8L9 CA ABCABC E ON V-ABCABC LDCA
E2EDKA1 3000000001017945375000003E2EDKA1 000000020 AG 0005065828 ABCABCFPPC ABCABC ABCABC ABCABC - FPP ONLY 88 ABCABC Crescent ABCABAABCAB L5R 4A2 CA ABCABC 111 111 1111 E ON ABCABCFPPC EN
E2EDKA1 3000000001017945375000004E2EDKA1 000000020 RE 0005065828 ABCABCFPPC ABCABC ABCABC ABCABC - FPP ONLY 88 ABCABC Crescent ABCABAABCAB L5R 4A2 CA ABCABC 111 111 1111 E ON ABCABCFPPC EN
E2EDKA1 3000000001017945375000005E2EDKA1 000000020 RG 0005065828 ABCABCFPPC ABCABC ABCABC ABCABC - FPP ONLY 88 ABCABC Crescent ABCABAABCAB L5R 4A2 CA ABCABC 111 111 1111 E ON ABCABCFPPC EN
E2EDKA1 3000000001017945375000006E2EDKA1 000000020 WE 0005001847 41 ABCABC ABCABC INC (ABCABC) DC A. ABCABCAB 88 ABCABC CRESCENT ABCABAABCAB L5R 4A2 CA ABCABC 111-111-1111 E ON ABCABCFPPC EN
E2EDKA1 3000000001017945375000007E2EDKA1 000000020 Z3 0005533050 ABCABCABC ABCABC Inc. ABCA Bank Swift Code -ABCABCABCAB Sort Code - 1950 ABCABCABCA Blvd. Acc No -1111111111 ABCABAABCAB L5N8L9 CA ABCABC E ON ABCABCFPPC EN
E2EDKA1 3000000001017945375000008E2EDKA1 000000020 BK 1075 ABCABCABC ABCABC Inc 1950 ABCABCABCA Blvd ABCABAABCAB ON L5N 8L9 CA ABCABC (111) 111-1111 (111) 111-1111 ON
E2EDKA1 3000000001017945375000009E2EDKA1 000000020 CR 1075 CONTACT ABCABCABC 1950 ABCABCABCA Blvd ABCABAABCAB ON L5N 8L9 CA ABCABC (111) 111-1111 (111) 111-1111 ON
E2EDK02 3000000001017945375000010E2EDK02 000000020 0099988298128 20210217
E2EDK02 3000000001017945375000011E2EDK02 000000020 00140-N6260-S 20210205
E2EDK02 3000000001017945375000012E2EDK02 000000020 0026336270425 20210217
E2EDK02 3000000001017945375000013E2EDK02 000000020 0128026580537 20210224
E2EDK02 3000000001017945375000014E2EDK02 000000020 01740-N6260-S
E2EDK02 3000000001017945375000015E2EDK02 000000020 900IAC
E2EDK02 3000000001017945375000016E2EDK02 000000020 901ZSH
E2EDK02 3000000001017945375000017E2EDK02 000000020 9078026580537 20210217
E2EDK03 3000000001017945375000018E2EDK03 000000020 02620210217
E2EDK03 3000000001017945375000019E2EDK03 000000020 00120210224
E2EDK03 3000000001017945375000020E2EDK03 000000020 02220210205
E2EDK03 3000000001017945375000021E2EDK03 000000020 01220210217
E2EDK03 3000000001017945375000022E2EDK03 000000020 01120210217
E2EDK03 3000000001017945375000023E2EDK03 000000020 02420210217
E2EDK03 3000000001017945375000024E2EDK03 000000020 02820210418
E2EDK03 3000000001017945375000025E2EDK03 000000020 04820210217
E2EDK17 3000000001017945375000026E2EDK17 000000020 001DDPDelivered Duty Paid
E2EDK17 3000000001017945375000027E2EDK17 000000020 002DDPdestination
E2EDK18 3000000001017945375000028E2EDK18 000000020 00160 0 Up to 04/18/2021 without deduction
E2EDK28 3000000001017945375000029E2EDK28 000000020 CA BOFACATT Bank of ABCABAB ABCABC ABCABAB 50127217 ABCABCABC ABCABC Inc.
E2EDK28 3000000001017945375000030E2EDK28 000000020 CA 026000082 ABCAbank ABCABC ABCABAB 201456700OLD ABCABCABC ABCABC Inc.
E2EDK28 3000000001017945375000031E2EDK28 000000020 GB ABCAGB2L ABCAbank N.A ABCABA E14, 5LB GB63ABCA18500803115593 ABCABCABC ABCABC Inc. GB63ABCA18500803115593
E2EDK28 3000000001017945375000032E2EDK28 000000020 CA 020012328 ABCABANK ABCABC ABCABAB ON M5J 2M3 2014567007 ABCABCABC ABCABC Inc.
E2EDK28 3000000001017945375000033E2EDK28 000000020 CA 03722010 ABCABABC ABCABABC Bank of Commerce ABCABAABCAB 64-04812 ABCABCABC ABCABC Inc.
E2EDK28 3000000001017945375000034E2EDK28 000000020 IE IHCC In-House Cash Center IHCC1075 ABCABCABC ABCABC Inc.
E2EDK28 3000000001017945375000035E2EDK28 000000020 CA 000300002 ABCAB Bank of ABCABC ABCABAB 0021520584OLD ABCABCABC ABCABC Inc.
E2EDK28 3000000001017945375000036E2EDK28 000000020 US USCC US Cash Center (IHC) city USCC1075 ABCABCABC ABCABC Inc.
E2EDK29 3000000001017945375000037E2EDK29 000000020 0064848944US A CAD CA ABCABC CA United States US CA A Air Air
E2EDKT1 3000000001017945375000038E2EDKT1 000000020 ZJ32E EN
E2EDKT2 3000000001017945375000039E2EDKT2 000038030 GST/HST877845941RT0001 *
E2EDKT2 3000000001017945375000040E2EDKT2 000038030 QST1021036966TQ0001 *
E2EDKT1 3000000001017945375000041E2EDKT1 000000020 Z4VL
E2EDKT2 3000000001017945375000042E2EDKT2 000041030 0.000 *
E2EDKT1 3000000001017945375000043E2EDKT1 000000020 Z4VH
E2EDKT2 3000000001017945375000044E2EDKT2 000043030 *
E2EDK14 3000000001017945375000045E2EDK14 000000020 008LDCA
E2EDK14 3000000001017945375000046E2EDK14 000000020 00710
E2EDK14 3000000001017945375000047E2EDK14 000000020 00610
E2EDK14 3000000001017945375000048E2EDK14 000000020 015Z4F2
E2EDK14 3000000001017945375000049E2EDK14 000000020 0031075
E2EDK14 3000000001017945375000050E2EDK14 000000020 021M
E2EDK14 3000000001017945375000051E2EDK14 000000020 0161075
E2EDK14 3000000001017945375000052E2EDK14 000000020 962M
E2EDP010013000000001017945375000053E2EDP01001000000020 000011 2980.000 EA 298.000 LB MOUSE 298.000 Z4TN 4260
E2EDP02 3000000001017945375000054E2EDP02 000053030 00140-N6260-S 00000120210205 DFUE
E2EDP02 3000000001017945375000055E2EDP02 000053030 0026336270425 00001120210217
E2EDP02 3000000001017945375000056E2EDP02 000053030 0168026580537 00001020210224
E2EDP02 3000000001017945375000057E2EDP02 000053030 9100000 00000120210205 DFUE
E2EDP02 3000000001017945375000058E2EDP02 000053030 911A 00000120210205 DFUE
E2EDP02 3000000001017945375000059E2EDP02 000053030 912PP 00000120210205 DFUE
E2EDP02 3000000001017945375000060E2EDP02 000053030 91300 00000120210205 DFUE
E2EDP02 3000000001017945375000061E2EDP02 000053030 914CONTACT ABCABCABC 00000120210205 DFUE
E2EDP02 3000000001017945375000062E2EDP02 000053030 963 00000120210205 DFUE
E2EDP02 3000000001017945375000063E2EDP02 000053030 965 00000120210205 DFUE
E2EDP02 3000000001017945375000064E2EDP02 000053030 9666336270425 00000120210205 DFUE
E2EDP02 3000000001017945375000065E2EDP02 000053030 9078026580537 00001020210205 DFUE
E2EDP03 3000000001017945375000066E2EDP03 000053030 02920210217
E2EDP03 3000000001017945375000067E2EDP03 000053030 00120210224
E2EDP03 3000000001017945375000068E2EDP03 000053030 01120210217
E2EDP03 3000000001017945375000069E2EDP03 000053030 02520210217
E2EDP03 3000000001017945375000070E2EDP03 000053030 02720210217
E2EDP03 3000000001017945375000071E2EDP03 000053030 02320210217
E2EDP03 3000000001017945375000072E2EDP03 000053030 02220210205
E2EDP19 3000000001017945375000073E2EDP19 000053030 001418VVZ
E2EDP19 3000000001017945375000074E2EDP19 000053030 002RJR-00001 AB ABCABCABC Mouse FORBUS BLUETOOTH
E2EDP19 3000000001017945375000075E2EDP19 000053030 0078471609000
E2EDP19 3000000001017945375000076E2EDP19 000053030 003889842532685
E2EDP19 3000000001017945375000077E2EDP19 000053030 011CN
E2EDP26 3000000001017945375000078E2EDP26 000053030 00459064.20
E2EDP26 3000000001017945375000079E2EDP26 000053030 00352269.20
E2EDP26 3000000001017945375000080E2EDP26 000053030 01052269.20
E2EDP26 3000000001017945375000081E2EDP26 000053030 01152269.20
E2EDP26 3000000001017945375000082E2EDP26 000053030 0126795.00
E2EDP26 3000000001017945375000083E2EDP26 000053030 01552269.20
E2EDP26 3000000001017945375000084E2EDP26 000053030 00117.54
E2EDP26 3000000001017945375000085E2EDP26 000053030 00252269.20
E2EDP26 3000000001017945375000086E2EDP26 000053030 940 2980.000
E2EDP26 3000000001017945375000087E2EDP26 000053030 939 2980.000
E2EDP05 3000000001017945375000088E2EDP05 000053030 + Z400MS List Price 52269.20 17.54 1 EA CAD 2980
E2EDP05 3000000001017945375000089E2EDP05 000053030 + XR1 Tax Jur Code Level 6795.00 13.000 52269.20
E2EDP05 3000000001017945375000090E2EDP05 000053030 + Tax Subtotal1 6795.00 2.28 1 EA CAD 2980
E2EDP05 3000000001017945375000091E2EDP05 000053030 + Taxable Amount + TaxSubtotal1 59064.20 19.82 1 EA CAD 2980
E2EDP04 3000000001017945375000092E2EDP04 000053030 CX 13.000 6795.00 7000000000
E2EDP04 3000000001017945375000093E2EDP04 000053030 CX 0 0 7001500000
E2EDP04 3000000001017945375000094E2EDP04 000053030 CX 0 0 7001505690
E2EDP28 3000000001017945375000095E2EDP28 000053030 00648489440000108471609000 CN CN ABCAB ZZ 298.000 298.000 LB US 400 United Stat KY
E2EDPT1 3000000001017945375000096E2EDPT1 000053030 0001E EN
E2EDPT2 3000000001017945375000097E2EDPT2 000096040 AB ABCABCABC Mouse forBus Bluetooth EN/XC/XD/XX Hdwr Black For Bsnss *
E2EDS01 3000000001017945375000098E2EDS01 000000020 0011
E2EDS01 3000000001017945375000099E2EDS01 000000020 01259064.20 CAD
E2EDS01 3000000001017945375000100E2EDS01 000000020 0056795.00 CAD
E2EDS01 3000000001017945375000101E2EDS01 000000020 01159064.20 CAD
E2EDS01 3000000001017945375000102E2EDS01 000000020 01052269.20 CAD
E2EDS01 3000000001017945375000103E2EDS01 000000020 94200000 CAD
E2EDS01 3000000001017945375000104E2EDS01 000000020 9440.00 CAD
E2EDS01 3000000001017945375000105E2EDS01 000000020 9450.00 CAD
E2EDS01 3000000001017945375000106E2EDS01 000000020 94659064.20 CAD
E2EDS01 3000000001017945375000107E2EDS01 000000020 94752269.20 CAD
E2EDS01 3000000001017945375000108E2EDS01 000000020 EXT
Z2XSK010003000000001017945375000109Z2XSK01000000108030 Z400 52269.20
Z2XSK010003000000001017945375000110Z2XSK01000000108030 XR1 13.000 6795.00 CX
</idocData></ReceiveIdoc>
```

### <a name="create-http-response-action"></a>Creación de una acción de respuesta HTTP

Ahora, agregue una acción de respuesta al flujo de trabajo de la aplicación lógica e incluya el resultado de la acción SAP. De este modo, la aplicación lógica devuelve los resultados del servidor SAP al solicitante original.

1. En el diseñador de Logic Apps, en la acción de SAP, seleccione **Nuevo paso**.

1. En el cuadro de búsqueda, escriba `response` como filtro. En la lista **Actions** (Acciones), seleccione **Response** (Respuesta).

1. Haga clic en el cuadro **Cuerpo** para que aparezca la lista de contenido dinámico. En dicha lista, en **Send message to SAP** (Enviar mensaje a SAP), seleccione el campo **Body** (Cuerpo).

   ![Completar una acción SAP](./media/logic-apps-using-sap-connector/select-sap-body-for-response-action.png)

1. Guarde la aplicación lógica.

#### <a name="create-rfc-request-response"></a>Creación de una solicitud-respuesta de RFC

> [!NOTE]
> El desencadenador de SAP recibe IDoc a través de tRFC, que no tiene un parámetro de respuesta por diseño. 

Debe crear un patrón de solicitud y respuesta si necesita recibir respuestas mediante una llamada de función remota (RFC) a Logic Apps de ABAP para SAP. Para recibir IDoc en la aplicación lógica, debe hacer que la primera acción sea una [solicitud HTTP](../connectors/connectors-native-reqres.md#add-a-response-action) con un código de estado `200 OK` y ningún contenido. Este paso recomendado completa la transferencia asincrónica de LUW para SAP a través de tRFC inmediatamente, lo que deja la conversación de CPIC para SAP disponible de nuevo. Después, puede agregar otras acciones en la aplicación lógica para procesar el IDoc recibido sin bloquear más transferencias adicionales.

Para implementar un patrón de solicitud y respuesta, primero debe detectar el esquema RFC mediante el [comando `generate schema`](#generate-schemas-for-artifacts-in-sap). El esquema generado tiene dos posibles nodos raíz: 

1. El nodo de solicitud, que es la llamada que recibe de SAP.

1. El nodo de respuesta, que es la respuesta a SAP.

En el ejemplo siguiente, se genera un patrón de solicitud y respuesta desde el módulo RFC `STFC_CONNECTION`. Se analiza el código XML de la solicitud para extraer un valor de nodo en el que SAP solicita `<ECHOTEXT>`. La respuesta inserta la marca de tiempo actual como un valor dinámico. Se recibirá una respuesta similar cuando envíe una llamada de función remota `STFC_CONNECTION` desde una aplicación lógica a SAP.

```http

<STFC_CONNECTIONResponse xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
  <ECHOTEXT>@{first(xpath(xml(triggerBody()?['Content']), '/*[local-name()="STFC_CONNECTION"]/*[local-name()="REQUTEXT"]/text()'))}</ECHOTEXT>
  <RESPTEXT>Azure Logic Apps @{utcNow()}</RESPTEXT>


```

### <a name="test-logic-app"></a>Prueba de la aplicación lógica

1. Si la aplicación lógica no está aún habilitada, en el menú de la aplicación lógica, elija **Información general**. En la barra de herramientas, seleccione **Enable** (Habilitar).

1. En la barra de herramientas del diseñador, seleccione **Run** (Ejecutar). Con este paso, se inicia manualmente la aplicación lógica.

1. Desencadene la aplicación lógica mediante el envío de una solicitud HTTP POST a la dirección URL del desencadenador de la solicitud HTTP.
Incluya el contenido del mensaje con la solicitud. Para enviar la solicitud, puede usar una herramienta como [Postman](https://www.getpostman.com/apps).

   En el caso de este artículo, la solicitud envía un archivo de IDoc, que debe estar en formato XML e incluir el espacio de nombres de la acción de SAP que está utilizando, por ejemplo:

   ```xml
   <?xml version="1.0" encoding="UTF-8" ?>
   <Send xmlns="http://Microsoft.LobServices.Sap/2007/03/Idoc/2/ORDERS05//720/Send">
      <idocData>
         <...>
      </idocData>
   </Send>
   ```

1. Después de enviar la solicitud HTTP, espere la respuesta de la aplicación lógica.

   > [!NOTE]
   > La aplicación lógica puede agotar el tiempo de espera si todos los pasos necesarios para la respuesta no finalizan dentro del [límite de tiempo de espera de la solicitud](./logic-apps-limits-and-config.md). Si se produce esta situación, es posible que se bloqueen las solicitudes. A fin de poder diagnosticar problemas, obtenga información sobre cómo puede [comprobar y supervisar las aplicaciones lógicas](../logic-apps/monitor-logic-apps.md).

Ahora ha creado una aplicación lógica que puede comunicarse con el servidor SAP. Ahora que ha configurado una conexión de SAP para la aplicación lógica, puede explorar otras acciones SAP disponibles, como RFC y BAPI.

## <a name="receive-message-from-sap"></a>Recibir un mensaje de SAP

En este ejemplo se usa una aplicación lógica que se desencadena cuando la aplicación recibe un mensaje de un sistema SAP.

### <a name="add-an-sap-trigger"></a>Adición de un desencadenador de SAP

1. En Azure Portal, cree una aplicación lógica en blanco que abra el diseñador de Logic Apps.

1. En el cuadro de búsqueda, escriba `sap` como filtro. En la lista **Desencadenadores**, seleccione **When a message is received from SAP** (Cuando se recibe un mensaje de SAP).

   ![Agregar un desencadenador de SAP](./media/logic-apps-using-sap-connector/add-sap-trigger-logic-app.png)

   O bien, puede elegir la pestaña **Enterprise** y después seleccionar la acción de desencadenar:

   ![Adición de un desencadenador de SAP desde la pestaña Enterprise (Empresa)](./media/logic-apps-using-sap-connector/add-sap-trigger-ent-tab.png)

1. Si la conexión ya existe, continúe con el paso siguiente para configurar la acción de SAP. Sin embargo, si se le solicitan detalles de conexión, proporcione la información para que pueda crear una conexión a su servidor SAP local ahora.

   1. Proporcione un nombre para la conexión.

   1. Si usa la puerta de enlace de datos, siga estos pasos:

      1. En la sección **Puerta de enlace de datos**, en **Suscripción**, primero seleccione la suscripción de Azure para el recurso de puerta de enlace de datos que creó en el Azure Portal para la instalación de su puerta de enlace de datos.

      1. En **Puerta de enlace de conexión**, seleccione su recurso de puerta de enlace de datos en Azure.

   1. Siga proporcionando información acerca de la conexión. Para la propiedad **Tipo de inicio de sesión**, siga el paso en función de si la propiedad está establecida en **Servidor de aplicaciones** o **Grupo**:

      * Para **Servidos de aplicaciones**, estas propiedades, que generalmente parecen opcionales, son necesarias:

        ![Creación de conexiones del servidor de aplicaciones de SAP](media/logic-apps-using-sap-connector/create-SAP-application-server-connection.png)

      * Para **Grupo**, estas propiedades, que generalmente parecen opcionales, son necesarias:

        ![Creación de conexiones del servidor de mensajes de SAP](media/logic-apps-using-sap-connector/create-SAP-message-server-connection.png)

      De forma predeterminada, se usa el establecimiento inflexible de tipos para comprobar si hay valores no válidos, para lo cual se realiza la validación de XML con respecto al esquema. Este comportamiento puede ayudarlo a detectar problemas con antelación. La opción **Escritura segura** está disponible para la compatibilidad con versiones anteriores y solo comprueba la longitud de cadena. Más información sobre la opción [Escritura segura](#safe-typing).

   1. Cuando haya finalizado, seleccione **Crear**.

      Logic Apps configura y comprueba la conexión para asegurarse de que funciona correctamente.

1. Proporcione los [parámetros necesarios](#parameters) en función de la configuración del sistema SAP. 

   Puede [filtrar los mensajes que recibe desde el servidor SAP especificando una lista de acciones de SAP](#filter-with-sap-actions).

   Puede seleccionar una acción de SAP desde el selector de archivos:

   ![Agregar acción de SAP a la aplicación lógica](media/logic-apps-using-sap-connector/select-SAP-action-trigger.png)  

   Si lo prefiere, puede especificar una acción manualmente:

   ![Escribir manualmente la acción de SAP que desea usar](media/logic-apps-using-sap-connector/manual-enter-SAP-action-trigger.png)

   Este es un ejemplo que muestra la apariencia de la acción cuando se configura el desencadenador para recibir varios mensajes.

   ![Ejemplo de desencadenador que recibe múltiples mensajes](media/logic-apps-using-sap-connector/example-trigger.png)

   Para más información sobre la acción de SAP, vea [Esquemas de mensaje para operaciones de IDoc](/biztalk/adapters-and-accelerators/adapter-sap/message-schemas-for-idoc-operations).

1. Ahora guarde la aplicación lógica para poder empezar a recibir mensajes del sistema SAP. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

La aplicación lógica está preparada para recibir mensajes del sistema SAP.

> [!NOTE]
> El desencadenador de SAP no es un desencadenador de sondeo, sino un desencadenador basado en webhook. Si usa la puerta de enlace de datos, el desencadenador se llama desde la puerta de enlace de datos solo cuando existe un mensaje, por lo que no es necesario realizar ningún sondeo.

Si recibe un error **500 Puerta de enlace no válida** con un mensaje similar al **servicio "sapgw00" desconocido**, reemplace el nombre del servicio de puerta de enlace en la configuración de la conexión de API y el desencadenador por su número de puerto. En el siguiente error de ejemplo, `sapgw00` debe reemplazarse por un número de puerto real, por ejemplo, `3300`. 

```json
{
    "body": {
        "error": {
            "code": 500,
            "source": "EXAMPLE-FLOW-NAME.eastus.environments.microsoftazurelogicapps.net",
            "clientRequestId": "00000000-0000-0000-0000-000000000000",
            "message": "BadGateway",
            "innerError": {
                "error": {
                    "code": "UnhandledException",
                    "message": "\nERROR service 'sapgw00' unknown\nTIME Wed Nov 11 19:37:50 2020\nRELEASE 721\nCOMPONENT NI (network interface)\nVERSION 40\nRC -3\nMODULE ninti.c\nLINE 933\nDETAIL NiPGetServByName: 'sapgw00' not found\nSYSTEM CALL getaddrinfo\nCOUNTER 1\n\nRETURN CODE: 20"
                }
            }
        }
    }
}
```

#### <a name="parameters"></a>Parámetros

Junto con entradas de cadena y número simples, el conector de SAP acepta estos parámetros de tabla (entradas `Type=ITAB`):

* Parámetros de dirección de tabla, tanto de entrada como de salida, para versiones anteriores de SAP.

* Parámetros de cambio, que reemplazan los parámetros de dirección de la tabla para las versiones más recientes de SAP.

* Parámetros de tabla jerárquica

#### <a name="filter-with-sap-actions"></a>Filtrado con acciones de SAP

Opcionalmente, puede filtrar los mensajes que la aplicación lógica recibe del servidor SAP proporcionando una lista, o una matriz, con una o varias acciones de SAP. De forma predeterminada, esta matriz está vacía, lo que significa que la aplicación lógica recibe todos los mensajes del servidor SAP sin filtrar. 

Al configurar el filtro de matriz, el desencadenador solo recibe mensajes de los tipos de acción de SAP especificados y rechaza todos los demás mensajes del servidor SAP. Sin embargo, este filtro no afecta al establecimiento de tipos de la carga recibida, que puede ser flexible o inflexible.

Cualquier filtrado de acciones de SAP se produce en el nivel del adaptador de SAP para la puerta de enlace de datos local. Para obtener más información, vea [Envío de IDoc de prueba a Logic Apps desde SAP](#test-sending-idocs-from-sap).

Si no puede enviar paquetes de IDoc desde SAP al desencadenador de la aplicación lógica, consulte el mensaje de rechazo de llamada de RFC transaccional (tRFC) en el cuadro de diálogo tRFC de SAP (T-CODE SM58). En la interfaz de SAP, puede recibir los siguientes mensajes de error, que se recortan debido a los límites de subcadena del campo **Status Text** (Texto de estado).

##### <a name="the-requestcontext-on-the-ireplychannel-was-closed-without-a-reply-being-sent"></a>Se cerró el elemento RequestContext en la interfaz IReplyChannel sin que se enviara una respuesta

Este mensaje de error indica errores inesperados cuando el controlador comodín del canal finaliza el canal debido a un error y vuelve a generar el canal para procesar otros mensajes.

Para confirmar que la aplicación lógica recibió el IDoc, [agregue una acción de respuesta](../connectors/connectors-native-reqres.md#add-a-response-action) que devuelva un código de estado `200 OK`. Deje el cuerpo vacío y no realice modificaciones ni realice adiciones a los encabezados. El IDoc se transporta a través de tRFC, que no permite una carga de respuesta.

Para rechazar el IDoc en su lugar, responda con cualquier código de estado HTTP distinto de `200 OK`. A continuación, el adaptador de SAP devuelve una excepción a SAP en su nombre. Solo debe rechazar el IDoc para señalar los errores de transporte a SAP, como un IDoc mal enrutado que la aplicación no puede procesar. No debe rechazar un IDoc para los errores de nivel de aplicación, como los problemas con los datos contenidos en el IDoc. Si retrasa la aceptación del transporte para la validación en el nivel de la aplicación, es posible que experimente un rendimiento negativo debido al bloqueo de la conexión a raíz del transporte de otros IDOC.

Si recibe este mensaje de error y experimenta errores sistémicos al llamar a Logic Apps, compruebe que ha configurado la configuración de red para el servicio de puerta de enlace de datos local de su entorno específico. Por ejemplo, si su entorno de red requiere el uso de un proxy para llamar a los puntos de conexión de Azure, debe configurar el servicio de puerta de enlace de datos local para usar el proxy. Para más información, vea [Configuración de proxy](/dotnet/framework/network-programming/proxy-configuration).

Si recibe este mensaje de error y experimenta errores intermitentes al llamar a Logic Apps, puede que tenga que aumentar el número de reintentos o el intervalo de reintento. 

1. Compruebe la configuración de SAP en el archivo de configuración del servicio de puerta de enlace de datos local, `Microsoft.PowerBI.EnterpriseGateway.exe.config`. La configuración del número de reintentos es similar a `WebhookRetryMaximumCount="2"`. La configuración del intervalo de reintento es similar a `WebhookRetryDefaultDelay="00:00:00.10"` y el formato del intervalo de tiempo es `HH:mm:ss.ff`. 
    
1. Guarde los cambios y reinicie la puerta de enlace de datos local.

##### <a name="the-segment-or-group-definition-e2edk36001-was-not-found-in-the-idoc-meta"></a>No se encontró el segmento o la definición de grupo E2EDK36001 en los metadatos del IDoc

Este mensaje de error indica que se produjeron errores esperados con otros errores. Por ejemplo, el error de generación de una carga XML de IDoc porque SAP no lanza sus segmentos. Como resultado, faltan los metadatos de tipo de segmento necesarios para la conversión.

Para que SAP publique estos segmentos, póngase en contacto con el ingeniero de ABAP para su sistema SAP.
### <a name="asynchronous-request-reply-for-triggers"></a>Solicitud-respuesta asincrónica para los desencadenadores

El conector de SAP es compatible con el [patrón de solicitud-respuesta asincrónico](/azure/architecture/patterns/async-request-reply) de Azure para los desencadenadores de Logic Apps. Puede usar este patrón para crear solicitudes correctas que podrían no realizarse con el patrón de solicitud-respuesta sincrónico predeterminado. 

> [!TIP]
> En aplicaciones lógicas con varias acciones de respuesta, todas las acciones de respuesta deben usar el mismo patrón de solicitud-respuesta. Por ejemplo, si la aplicación lógica usa un control de cambio con varias acciones de respuesta posibles, debe configurar todas las acciones de respuesta para usar el mismo patrón de solicitud-respuesta, ya sea sincrónico o asincrónico. 

La habilitación de la respuesta asincrónica para la acción de respuesta permite que la aplicación lógica responda con `202 Accepted` cuando se haya aceptado una solicitud para su procesamiento. La respuesta contiene un encabezado de ubicación que puede usar para recuperar el estado final de la solicitud.

Para configurar un patrón de solicitud-respuesta asincrónico para la aplicación lógica mediante el conector de SAP:

1. Abra la aplicación lógica en el **diseñador de Logic Apps**.

1. Confirme que el conector de SAP es el desencadenador de la aplicación lógica.

1. Abra la acción de **respuesta** de la aplicación lógica. En la barra de título de la acción, seleccione el menú ( **...** ) &gt; **Configuración**.

1. En la **configuración** de la acción de respuesta, active la alternancia en **Respuesta asincrónica**. Seleccione Listo.

1. Guarde los cambios en la aplicación lógica.

## <a name="find-extended-error-logs"></a>Búsqueda de registros de errores ampliados

Para ver los mensajes de error completos, recurra a los registros ampliados del adaptador de SAP. También puede [habilitar un archivo de registro ampliado para el conector de SAP](#extended-sap-logging-in-on-premises-data-gateway).

En el caso de las versiones de puerta de enlace de datos local de junio de 2020 y posteriores, puede [habilitar los registros de puerta de enlace en la configuración de la aplicación](/data-integration/gateway/service-gateway-tshoot#collect-logs-from-the-on-premises-data-gateway-app). 

* El nivel de registro predeterminado es **Advertencia**.

* Si habilita **Registro adicional** en la configuración de **Diagnóstico** de la aplicación de puerta de enlace de datos local, el nivel de registro aumenta a **Informativo**.

* Para aumentar el nivel de registro a **Detallado**, actualice el siguiente valor en el archivo de configuración. Normalmente el archivo de configuración se encuentra en `C:\Program Files\On-premises data gateway\Microsoft.PowerBI.DataMovement.Pipeline.GatewayCore.dll.config`.

```json
<setting name="SapTraceLevel" serializeAs="String">
   <value>Verbose</value>
</setting>
```

En el caso de las versiones de puerta de enlace de datos local de abril de 2020 y anteriores, los registros están deshabilitados de forma predeterminada.

### <a name="extended-sap-logging-in-on-premises-data-gateway"></a>Registro de SAP ampliado en la puerta de enlace de datos local

Si usa una [puerta de enlace de datos local para Logic Apps](../logic-apps/logic-apps-gateway-install.md), puede configurar un archivo de registro ampliado para el conector de SAP. Puede usar la puerta de enlace de datos local para redirigir los eventos de Seguimiento de eventos para Windows (ETW) en los archivos de registro giratorios que se incluyen en los archivos .zip de registro de la puerta de enlace. 

Puede [exportar todos los registros de configuración y servicio de la puerta de enlace](/data-integration/gateway/service-gateway-tshoot#collect-logs-from-the-on-premises-data-gateway-app) a un archivo .zip en desde la configuración de la aplicación de puerta de enlace.

> [!NOTE]
> El registro ampliado puede afectar al rendimiento de las aplicaciones lógicas cuando está siempre habilitado. Se recomienda desactivar los archivos de registro ampliados una vez que haya terminado de analizar y solucionar un problema.

#### <a name="capture-etw-events"></a>Captura de eventos ETW

Opcionalmente, los usuarios avanzados pueden capturar eventos ETW directamente. Puede [consumir los datos de Azure Diagnostics en Event Hubs](../azure-monitor/agents/diagnostics-extension-stream-event-hubs.md) o [recopilar los datos para registros de Azure Monitor](../azure-monitor/agents/diagnostics-extension-logs.md). Para obtener más información, consulte [Procedimientos recomendados para recopilar y almacenar datos](/azure/architecture/best-practices/monitoring#collecting-and-storing-data). Puede usar [PerfView](https://github.com/Microsoft/perfview/blob/master/README.md) para trabajar con los archivos ETL resultantes o puede escribir su propio programa. Este tutorial usa PerfView:

1. En el menú de PerfView, seleccione **Recopilar** &gt; **Recopilar** para capturar los eventos.

1. En el campo **Proveedor adicional**, escriba `*Microsoft-LobAdapter` para especificar el proveedor de SAP para capturar eventos del adaptador de SAP. Si no especifica esta información, el seguimiento solo incluye eventos ETW generales.

1. Deje el resto de los valores predeterminados. Si quiere, puede cambiar el nombre de archivo o la ubicación en el campo **Archivo de datos**.

1. Seleccione **Iniciar colección** para iniciar el seguimiento.

1. Una vez que haya reproducido el problema o recopilado suficientes datos de análisis, seleccione **Detener recopilación**.

Para compartir los datos con otra entidad, como los ingenieros de Soporte técnico de Azure, comprima el archivo ETL.

Para ver el contenido del seguimiento:

1. En PerfView, seleccione **Archivo** &gt; **Abrir** y, a continuación, seleccione el archivo ETL que acaba de generar.

1. En la barra lateral de PerfView, en la sección **Eventos** del archivo ETL.

1. En **Filtro**, filtre por `Microsoft-LobAdapter` para ver solo los eventos y procesos de puerta de enlace pertinentes.

### <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

1. Para desencadenar la aplicación lógica, envíe un mensaje desde el sistema SAP.

1. En el menú de la aplicación lógica, seleccione **Información general**. Revise el **historial de ejecuciones** para ver las nuevas ejecuciones de la aplicación lógica.

1. Abra la ejecución más reciente, que muestra el mensaje enviado desde su sistema SAP en la sección de salidas del desencadenador.

### <a name="test-sending-idocs-from-sap"></a>Prueba de envío de IDoc desde SAP

Para enviar IDoc desde SAP a la aplicación lógica, necesita la siguiente configuración mínima:

> [!IMPORTANT]
> Siga estos pasos solo al probar la configuración de SAP con la aplicación lógica. Los entornos de producción requieren configuración adicional.

1. [Configuración de un destino RFC en SAP](#create-rfc-destination)

1. [Creación de una conexión ABAP con el destino RFC](#create-abap-connection)

1. [Creación de un puerto receptor](#create-receiver-port)

1. [Creación de un puerto remitente](#create-sender-port)

1. [Creación de un asociado del sistema lógico](#create-logical-system-partner)

1. [Creación de perfil de socio](#create-partner-profiles)

1. [Prueba de envío de mensajes](#test-sending-messages)

#### <a name="create-rfc-destination"></a>Creación de destino RFC

1. Para abrir la opción **Configuration of RFC Connections** (Configuración de las conexiones RFC), en la interfaz de SAP, use el código de transacción (código T) **sm59** con el prefijo **/n**.

1. Seleccione **TCP/IP Connections** > **Create** (Conexiones TCP/IP > Crear).

1. Cree un nuevo destino RFC con la siguiente configuración:
    
    * En **RFC Destination** (Destino RFC), escriba un nombre.
    
    * En la pestaña **Technical Settings** (Configuración técnica), en **Activation Type** (Tipo de activación), seleccione **Registered Server Program** (Programa de servidor registrado). En **Program ID** (Id. de programa), escriba un valor. En SAP, el desencadenador de la aplicación lógica se registrará con este identificador.

    > [!IMPORTANT]
    > El **identificador de programa** de SAP distingue mayúsculas de minúsculas. Asegúrese de que usa de forma coherente el mismo formato de mayúsculas y minúsculas para el **identificador de programa** al configurar la aplicación lógica y el servidor SAP. De lo contrario, podría recibir los siguientes errores en el monitor tRFC (T-Code SM58) al intentar enviar un IDoc a SAP:
    >
    > * **No se encontró la función IDOC_INBOUND_ASYNCHRONOUS.**
    > * **No se admite el cliente RFC que no sea ABAP (tipo de socio).**
    >
    > Para más información de SAP, consulte las notas siguientes (inicio de sesión necesario) <https://launchpad.support.sap.com/#/notes/2399329> y <https://launchpad.support.sap.com/#/notes/353597>.
    
    * En la pestaña **Unicode**, para **Communication Type with Target System** (Tipo de comunicación con sistema de destino), seleccione **Unicode**.

1. Guarde los cambios.

1. Registre el nuevo **Program ID** (Id. de programa) con Azure Logic Apps.

1. Para probar la conexión, en la interfaz de SAP, en el nuevo **RFC Destination** (Destino RFC), seleccione **Connection Test** (Prueba de conexión).

#### <a name="create-abap-connection"></a>Creación de conexión ABAP

1. Para abrir la opción **Configuration of RFC Connections** (Configuración de las conexiones RFC), en la interfaz de SAP, use el código de transacción (código T) **sm59** _ con el prefijo _ */n**.

1. Seleccione **ABAP Connections** > **Create** (Conexiones ABAP > Crear).

1. **RFC Destination** (Destino RFC), escriba el identificador de [su sistema SAP de prueba](#create-rfc-destination).

1. Guarde los cambios.

1. Para probar la conexión, seleccione **Connection Test** (Prueba de conexión).

#### <a name="create-receiver-port"></a>Creación de un puerto receptor

1. Para abrir la configuración **Ports In IDOC processing** (Procesamiento de puertos en IDoc), en la interfaz de SAP, use el código de transacción (código T) **we21** con el prefijo **/n**.

1. Seleccione **Ports** > **Transactional RFC** > **Create** (Puertos > RFC transaccional > Crear).

1. En el cuadro de configuración que se abre, seleccione **own port name** (nombre propio del puerto). En el puerto de prueba, escriba un **nombre**. Guarde los cambios.

1. En la configuración del nuevo puerto receptor, para **RFC destination** (Destino RFC), escriba el identificador del [destino RFC de prueba](#create-rfc-destination).

1. Guarde los cambios.

#### <a name="create-sender-port"></a>Creación de un puerto remitente

1.  Para abrir la configuración **Ports In IDOC processing** (Procesamiento de puertos en IDoc), en la interfaz de SAP, use el código de transacción (código T) **we21** con el prefijo **/n**.

1. Seleccione **Ports** > **Transactional RFC** > **Create** (Puertos > RFC transaccional > Crear).

1. En el cuadro de configuración que se abre, seleccione **own port name** (nombre propio del puerto). En el puerto de prueba, escriba un **nombre** que empiece por **SAP**. Todos los nombres de puerto remitente deben comenzar con las letras **SAP**, por ejemplo, **SAPTEST**. Guarde los cambios.

1. En la configuración del nuevo puerto remitente, para **RFC destination** (Destino RFC), escriba el identificador de [la conexión de ABAP](#create-abap-connection).

1. Guarde los cambios.

#### <a name="create-logical-system-partner"></a>Creación de un asociado del sistema lógico

1. Para abrir la configuración para **cambiar la vista de sistemas lógicos referente al apartado de información general**, en la interfaz de SAP, use el código de transacción (código T) **bd54**.

1. Acepte el mensaje de advertencia que aparece: **Caution: The table is cross-client** (Precaución, la tabla afecta a varios clientes).

1. Encima de la lista que muestra los sistemas lógicos existentes, seleccione **New Entries** (Nuevas entradas).

1. En el nuevo sistema lógico, escriba un identificador de **Log.System** y una descripción breve del **nombre**. Guarde los cambios.

1. Cuando aparezca **Prompt for Workbench** (Solicitud de banco de trabajo), cree una nueva solicitud. Para ello, proporcione una descripción (si ya ha creado una solicitud, omita este paso).

1. Después de crear la solicitud de banco de trabajo, vincule esa solicitud a la solicitud de actualización de la tabla. Para confirmar que la tabla se ha actualizado, guarde los cambios.

#### <a name="create-partner-profiles"></a>Creación de perfiles de socios

En entornos de producción, debe crear dos perfiles de socio. El primer perfil es para el remitente, que es su organización y el sistema SAP. El segundo perfil es para el receptor, que es la aplicación lógica.

1. Para abrir la configuración **Partner profiles** (Perfiles de asociado), en la interfaz de SAP, use el código de transacción (código T) **we20** con el prefijo **/n**.

1. En **Partner Profiles** (Perfiles de socio), seleccione **Partner Type LS** > **Create** (Tipo de socio LS > Crear).

1. Cree un nuevo perfil de socio con la siguiente configuración:

    * En **Partner No.** (Núm. de socio), escriba el [identificador de socio del sistema lógico](#create-logical-system-partner).

    * En **Partn. Type** (Tipo de socio), escriba **LS**.

    * En **Agent** (Agente), escriba el identificador de la cuenta de usuario de SAP que utilizará al registrar identificadores de programa para Azure Logic Apps u otros sistemas que no sean de SAP.

1. Guarde los cambios. Si no ha [creado el socio del sistema lógico](#create-logical-system-partner), aparecerá el error **Enter a valid partner number** (Escriba un número de socio válido).

1. En la configuración del perfil de socio, en **Outbound parmtrs.** (Parámetros de salida), seleccione **Create outbound parameter** (Crear parámetro de salida).

1. Cree un nuevo parámetro de salida con la siguiente configuración:

    * Escriba su **tipo de mensaje**, por ejemplo, **CREMAS**.

    * Escriba el [identificador del puerto receptor](#create-receiver-port).

    * Escriba un tamaño de IDoc para **Pack. Tamaño**. O bien, para [enviar IDoc de uno en uno desde SAP](#receive-idoc-packets-from-sap), seleccione **Pass IDoc Immediately** (Pasar IDoc de inmediato).

1. Guarde los cambios.

#### <a name="test-sending-messages"></a>Prueba de envío de mensajes

1. Para abrir la configuración **Test Tool for IDoc Processing** (Procesamiento de la herramienta de prueba para IDoc), en la interfaz de SAP, use el código de transacción (código T) **we19** con el prefijo **/n**.

1. En **Template for test** (Plantilla para prueba), seleccione **Via message type** (Mediante tipo de mensaje) y escriba su tipo de mensaje; por ejemplo, **CREMAS**. Seleccione **Crear**.

1. Confirme el mensaje **Which IDoc type?** (¿Qué tipo de IDoc?) seleccionando **Continue** (Continuar).

1. Seleccione el nodo **EDIDC**. Escriba los valores adecuados para los puertos receptor y remitente. Seleccione **Continuar**.

1. Seleccione **Standard Outbound Processing** (Procesamiento de salida estándar).

1. Para iniciar el procesamiento de IDoc de salida, seleccione **Continue** (Continuar). Una vez finalizado el procesamiento, aparece el mensaje **IDoc sent to SAP system or external program** (IDoc enviado a sistema SAP o programa externo).

1.  Para comprobar si hay errores de procesamiento, use el código de transacción **sm58** con el prefijo **/n**.

## <a name="receive-idoc-packets-from-sap"></a>Recepción de paquetes IDoc de SAP

Puede configurar SAP para que [envíe IDoc en paquetes](https://help.sap.com/viewer/8f3819b0c24149b5959ab31070b64058/7.4.16/4ab38886549a6d8ce10000000a42189c.html), que son lotes o grupos de IDoc. Para recibir paquetes IDoc, el conector de SAP y, en concreto, el desencadenador, no necesitan una configuración adicional. Pero para procesar cada elemento en un paquete IDoc después de que el desencadenador reciba el paquete, se necesitan algunos pasos adicionales para dividir el paquete en IDoc individuales.

Este es un ejemplo que muestra cómo extraer IDoc individuales de un paquete mediante la [función `xpath()`](./workflow-definition-language-functions-reference.md#xpath):

1. Antes de empezar, necesita una aplicación lógica con un desencadenador de SAP. Si aún no tiene esta aplicación lógica, siga los pasos anteriores de este tema para [configurar una aplicación lógica con un desencadenador de SAP](#receive-message-from-sap).

    > [!IMPORTANT]
    > El **identificador de programa** de SAP distingue mayúsculas de minúsculas. Asegúrese de que usa de forma coherente el mismo formato de mayúsculas y minúsculas para el **identificador de programa** al configurar la aplicación lógica y el servidor SAP. De lo contrario, podría recibir los siguientes errores en el monitor tRFC (T-Code SM58) al intentar enviar un IDoc a SAP:
    >
    > * **No se encontró la función IDOC_INBOUND_ASYNCHRONOUS.**
    > * **No se admite el cliente RFC que no sea ABAP (tipo de socio).**
    >
    > Para más información de SAP, consulte las notas siguientes (inicio de sesión necesario) <https://launchpad.support.sap.com/#/notes/2399329> y <https://launchpad.support.sap.com/#/notes/353597>.

   Por ejemplo:

   ![Agregar disparador SAP a la aplicación lógica](./media/logic-apps-using-sap-connector/first-step-trigger.png)

1. [Agregue una acción de respuesta a la aplicación lógica](../connectors/connectors-native-reqres.md#add-a-response-action) para responder inmediatamente con el estado de la solicitud de SAP. Se recomienda agregar esta acción inmediatamente después del desencadenador para liberar el canal de comunicación con el servidor SAP. Elija uno de los siguientes códigos de estado (`statusCode`) para usarlo en la acción de respuesta:

    * **202 Aceptado**, lo que significa que la solicitud se ha aceptado para el procesamiento, pero el procesamiento aún no se ha completado.

    * **204 Sin contenido**, lo que significa que el servidor ha realizado correctamente la solicitud y no hay contenido adicional para enviar en el cuerpo de la carga de respuesta. 

    * **200 - CORRECTO**. Este código de estado siempre contiene una carga útil, aunque el servidor genere un cuerpo de carga de longitud cero. 

1. Obtenga el espacio de nombres raíz de IDoc XML que la aplicación lógica recibe de SAP. Para extraer este espacio de nombres del documento XML, agregue un paso que cree una variable de cadena local y almacene dicho espacio de nombres mediante una expresión `xpath()`:

   `xpath(xml(triggerBody()?['Content']), 'namespace-uri(/*)')`

   ![Obtención del espacio de nombres raíz del IDoc](./media/logic-apps-using-sap-connector/get-namespace.png)

1. Para extraer un IDoc individual, agregue un paso que cree una variable de matriz y almacene la colección de IDoc mediante otra expresión `xpath()`:

   `xpath(xml(triggerBody()?['Content']), '/*[local-name()="Receive"]/*[local-name()="idocData"]')`

   ![Obtención de una matriz de elementos](./media/logic-apps-using-sap-connector/get-array.png)

   La variable de matriz hace que cada IDoc esté disponible para que la aplicación lógica lo procese individualmente mediante la enumeración de la colección. En este ejemplo, la aplicación lógica transfiere cada IDoc a un servidor SFTP mediante un bucle:

   ![Envío del IDoc al servidor SFTP](./media/logic-apps-using-sap-connector/loop-batch.png)

   Cada IDoc debe incluir el espacio de nombres raíz, que es el motivo por el que el contenido del archivo se ajusta dentro de un elemento `<Receive></Receive` junto con el espacio de nombres raíz antes de enviar el IDoc a la aplicación de nivel inferior o servidor SFTP en este caso.

Puede usar la plantilla de inicio rápido para este patrón si selecciona esta plantilla en el diseñador de Logic Apps al crear una nueva aplicación lógica.

![Seleccionar plantilla de aplicación de lógica de lote](./media/logic-apps-using-sap-connector/select-batch-logic-app-template.png)

## <a name="generate-schemas-for-artifacts-in-sap"></a>Generar esquemas para artefactos en SAP

En este ejemplo, se utiliza una aplicación lógica que se puede desencadenar con una solicitud HTTP. Para generar los esquemas para los IDoc y BAPI especificados, la acción de SAP **Generate schema** (Generar esquema) envía una solicitud a un sistema SAP.

Esta acción de SAP devuelve un [esquema XML](#sample-xml-schemas), no el contenido o los datos del propio documento XML. Los esquemas que se devuelven en la respuesta se cargan en una cuenta de integración mediante el conector de Azure Resource Manager. Los esquemas contienen las siguientes partes:

* La estructura del mensaje de solicitud. Use esta información para formar la lista `get` de BAPI.

* La estructura del mensaje de respuesta. Utilice esta información para analizar la respuesta. 

Para enviar el mensaje de solicitud, use la acción genérica de SAP **Send message to SAP** (Enviar mensaje a SAP) o las acciones **Call BAPI** (Llamar a BAPI) de destino.

### <a name="sample-xml-schemas"></a>Esquemas XML de ejemplo

Si está aprendiendo a generar un esquema XML para usarlo en la creación de un documento de ejemplo, consulte los ejemplos siguientes. En estos ejemplos se muestra cómo puede trabajar con muchos tipos de cargas, entre las que se incluyen:

* [Solicitudes RFC](#xml-samples-for-rfc-requests)

* [Solicitudes BAPI](#xml-samples-for-bapi-requests)

* [Solicitudes IDoc](#xml-samples-for-idoc-requests)

* Tipos de datos de esquemas XML simples o complejos

* Parámetros de tabla

* Comportamientos XML opcionales

Puede comenzar el esquema XML con un prólogo XML opcional. El conector de SAP funciona con o sin el prólogo XML.

```xml

<?xml version="1.0" encoding="utf-8">

```

#### <a name="xml-samples-for-rfc-requests"></a>Ejemplos XML para solicitudes de RFC

El ejemplo siguiente es una llamada de RFC simple. El nombre de RFC es `STFC_CONNECTION`. Esta solicitud usa el espacio de nombres predeterminado `xmlns=`, pero puede asignar y utilizar alias de espacio de nombres como `xmmlns:exampleAlias=`. El valor del espacio de nombres es el espacio de nombres para todas las RFC en SAP para servicios de Microsoft. Hay un parámetro de entrada simple en la solicitud: `<REQUTEXT>`.

```xml

<STFC_CONNECTION xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
  <REQUTEXT>exampleInput</REQUTEXT>
</STFC_CONNECTION>

```

El ejemplo siguiente es una llamada de RFC con un parámetro de tabla. Esta llamada de ejemplo y su grupo de RFC de prueba están disponibles como parte de todos los sistemas SAP. El nombre del parámetro de tabla es `TCPICDAT`. El tipo de línea de tabla es `ABAPTEXT` y este elemento se repite para cada fila de la tabla. Este ejemplo contiene una sola línea, denominada `LINE`. Las solicitudes con un parámetro de tabla pueden contener cualquier número de campos, donde el número es un entero positivo (*n*). 

```xml

<STFC_WRITE_TO_TCPIC xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
  <RESTART_QNAME>exampleQName</RESTART_QNAME>
    <TCPICDAT>
      <ABAPTEXT xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
        <LINE>exampleFieldInput1</LINE>
      <ABAPTEXT xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
        <LINE>exampleFieldInput2</LINE>
      <ABAPTEXT xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
        <LINE>exampleFieldInput3</LINE>
      </ABAPTEXT>
    </TCPICDAT>
</STFC_WRITE_TO_TCPIC>

```

El ejemplo siguiente es una llamada de RFC con un parámetro de tabla que tiene un campo anónimo. Un campo anónimo es cuando el campo no tiene asignado ningún nombre. Los tipos complejos se declaran en un espacio de nombres independiente, en el que la declaración establece un nuevo valor predeterminado para el nodo actual y todos sus elementos secundarios. En el ejemplo se usa el código hexadecimal `x002F` como carácter de escape para el símbolo */* , porque este símbolo está reservado en el nombre de campo de SAP.

```xml

<RFC_XML_TEST_1 xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
  <IM_XML_TABLE>
    <RFC_XMLCNT xmlns="http://Microsoft.LobServices.Sap/2007/03/Rfc/">
      <_x002F_AnonymousField>exampleFieldInput</_x002F_AnonymousField>
    </RFC_XMLCNT>
  </IM_XML_TABLE>
</RFC_XML_TEST_1>

```

En el ejemplo siguiente se incluyen los prefijos de los espacios de nombres. Puede declarar todos los prefijos a la vez o puede declarar un número de ellos como atributos de un nodo. El alias del espacio de nombres de RFC `ns0` se utiliza como la raíz y los parámetros para el tipo básico.

> [!NOTE]
> Los tipos complejos se declaran en otro espacio de nombres para tipos de RFC con el alias `ns3` en lugar del espacio de nombres de RFC normal con el alias `ns0`.

```xml

<ns0:BBP_RFC_READ_TABLE xmlns:ns0="http://Microsoft.LobServices.Sap/2007/03/Rfc/" xmlns:ns3="http://Microsoft.LobServices.Sap/2007/03/Types/Rfc/">
  <ns0:DELIMITER>0</ns0:DELIMITER>
  <ns0:QUERY_TABLE>KNA1</ns0:QUERY_TABLE>
  <ns0:ROWCOUNT>250</ns0:ROWCOUNT>
  <ns0:ROWSKIPS>0</ns0:ROWSKIPS>
  <ns0:FIELDS>
    <ns3:RFC_DB_FLD>
      <ns3:FIELDNAME>KUNNR</ns3:FIELDNAME>
    </ns3:RFC_DB_FLD>
  </ns0:FIELDS>
</ns0:BBP_RFC_READ_TABLE>

```

#### <a name="xml-samples-for-bapi-requests"></a>Ejemplos XML para solicitudes de BAPI

Los siguientes ejemplos XML son solicitudes de ejemplo para [llamar al método BAPI](#call-bapi-action).

> [!NOTE]
> SAP pone los objetos comerciales a disposición de los sistemas externos mediante su descripción en respuesta a RFC `RPY_BOR_TREE_INIT`, que Logic Apps emite sin ningún filtro de entrada. Logic Apps inspecciona la tabla de salida `BOR_TREE`. El campo `SHORT_TEXT` se utiliza para los nombres de objetos comerciales. Logic Apps no puede acceder a los objetos comerciales no devueltos por SAP en la tabla de salida.
> 
> Si usa objetos comerciales personalizados, debe asegurarse de publicar y liberar estos objetos comerciales en SAP. De lo contrario, SAP no muestra los objetos comerciales personalizados en la tabla de salida `BOR_TREE`. No se puede acceder a los objetos comerciales personalizados en Logic Apps hasta que se exponen los objetos comerciales desde SAP. 

En el ejemplo siguiente se obtiene una lista de bancos mediante el método de BAPI `GETLIST`. Este ejemplo contiene el objeto comercial de un banco: `BUS1011`. 

```xml

<GETLIST xmlns="http://Microsoft.LobServices.Sap/2007/03/Bapi/BUS1011">
  <BANK_CTRY>US</BANK_CTRY>
  <MAX_ROWS>10</MAX_ROWS>
</GETLIST>

```

En el ejemplo siguiente se crea un objeto de banco mediante el método `CREATE`. En este ejemplo se usa el mismo objeto comercial que en el ejemplo anterior: `BUS1011`. Cuando use el método `CREATE` para crear un banco, asegúrese de confirmar los cambios porque este método no se confirma de forma predeterminada.

> [!TIP]
> Asegúrese de que el documento XML sigue las reglas de validación configuradas en el sistema SAP. Por ejemplo, en este documento de referencia, la clave del banco (`<BANK_KEY>`) debe ser un número de enrutamiento bancario, que en Estados Unidos también se conoce como ABA.

```xml

<CREATE xmlns="http://Microsoft.LobServices.Sap/2007/03/Bapi/BUS1011">
  <BANK_ADDRESS>
    <BANK_NAME xmlns="http://Microsoft.LobServices.Sap/2007/03/Types/Rfc">ExampleBankName</BANK_NAME>
    <REGION xmlns="http://Microsoft.LobServices.Sap/2007/03/Types/Rfc">ExampleRegionName</REGION>
    <STREET xmlns="http://Microsoft.LobServices.Sap/2007/03/Types/Rfc">ExampleStreetAddress</STREET>
    <CITY xmlns="http://Microsoft.LobServices.Sap/2007/03/Types/Rfc">Redmond</CITY>
  </BANK_ADDRESS>
  <BANK_COUNTRY>US</BANK_COUNTRY>
  <BANK_KEY>123456789</BANK_KEY>
</CREATE>

```

En el ejemplo siguiente se obtienen detalles de un banco mediante el número de enrutamiento bancario, el valor de `<BANK_KEY>`. 

```xml

<GETDETAIL xmlns="http://Microsoft.LobServices.Sap/2007/03/Bapi/BUS1011">
  <BANK_COUNTRY>US</BANK_COUNTRY>
  <BANK_KEY>123456789</BANK_KEY>
</GETDETAIL>

```

#### <a name="xml-samples-for-idoc-requests"></a>Ejemplos de XML para solicitudes de IDoc

Para generar un esquema XML de IDoc para SAP sin formato, use la aplicación **Inicio de sesión de SAP** y el código de transacción `WE-60`. Acceda a la documentación de SAP a través de la GUI y genere esquemas XML en formato XSD para sus tipos y extensiones de IDoc. Para obtener una explicación de los formatos y las cargas de SAP genéricos y sus cuadros de diálogo integrados, consulte la [documentación de SAP](https://help.sap.com/viewer/index).

En este ejemplo se declaran los espacios de nombres y el nodo raíz. El URI del código de ejemplo (`http://Microsoft.LobServices.Sap/2007/03/Idoc/3/ORDERS05//700/Send`) declara la siguiente configuración:

* `/IDoc` es la nota raíz para todos los IDoc
* `/3` es la versión de tipos de registro para definiciones de segmento comunes
* `/ORDERS05` es el tipo de IDoc
* `//` es un segmento vacío, porque no hay ninguna extensión IDoc
* `/700` es la versión de SAP
* `/Send` es la acción para enviar la información a SAP

```xml

<ns0:Send xmlns:ns0="http://Microsoft.LobServices.Sap/2007/03/Idoc/3/ORDERS05//700/Send" xmlns:ns3="http://schemas.microsoft.com/2003/10/Serialization" xmlns:ns1="http://Microsoft.LobServices.Sap/2007/03/Types/Idoc/Common/" xmlns:ns2="http://Microsoft.LobServices.Sap/2007/03/Idoc/3/ORDERS05//700">
  <ns0:idocData>

```

Puede repetir el nodo `idocData` para enviar un lote de IDoc en una sola llamada. En el ejemplo siguiente, hay un registro de control (`EDI_DC40`) y varios registros de datos.

```xml

<...>
  <ns0:idocData>
    <ns2:EDI_DC40>
      <ns1:TABNAM>EDI_DC40</ns1:TABNAM>
<...>
      <ns1:ARCKEY>Cor1908207-5</ns1:ARCKEY>
    </ns2:EDI_DC40>
    <ns2:E2EDK01005>
      <ns2:DATAHEADERCOLUMN_SEGNAM>E23DK01005</ns2:DATAHEADERCOLUMN_SEGNAM>
      <ns2:CURCY>USD</ns2:CURCY>
    </ns2:E2EDK01005>
    <ns2:E2EDK03>
<...>
  </ns0:idocData>

```

El ejemplo siguiente es un registro de control IDoc de ejemplo, que usa el prefijo `EDI_DC`. Debe actualizar los valores para que coincidan con la instalación de SAP y el tipo de IDoc. Por ejemplo, el código de cliente de IDoc puede no ser `800`. Póngase en contacto con su equipo de SAP para asegurarse de que está usando los valores correctos para la instalación de SAP.

```xml

<ns2:EDI_DC40>
  <ns:TABNAM>EDI_DC40</ns1:TABNAM>
  <ns:MANDT>800</ns1:MANDT>
  <ns:DIRECT>2</ns1:DIRECT>
  <ns:IDOCTYP>ORDERS05</ns1:IDOCTYP>
  <ns:CIMTYP></ns1:CIMTYP>
  <ns:MESTYP>ORDERS</ns1:MESTYP>
  <ns:STD>X</ns1:STD>
  <ns:STDVRS>004010</ns1:STDVRS>
  <ns:STDMES></ns1:STDMES>
  <ns:SNDPOR>SAPENI</ns1:SNDPOR>
  <ns:SNDPRT>LS</ns1:SNDPRT>
  <ns:SNDPFC>AG</ns1:SNDPFC>
  <ns:SNDPRN>ABAP1PXP1</ns1:SNDPRN>
  <ns:SNDLAD></ns1:SNDLAD>
  <ns:RCVPOR>BTSFILE</ns1:RCVPOR>
  <ns:RCVPRT>LI</ns1:RCVPRT>

```

El ejemplo siguiente es un registro de datos de ejemplo con segmentos sin formato. En este ejemplo se usa el formato de fecha de SAP. Los documentos fuertemente tipados pueden utilizar formatos de fecha XML nativos, como `2020-12-31 23:59:59`.

```xml

<ns2:E2EDK01005>
  <ns2:DATAHEADERCOLUMN_SEGNAM>E2EDK01005</ns2:DATAHEADERCOLUMN_SEGNAM>
    <ns2:CURCY>USD</ns2:CURCY>
    <ns2:BSART>OR</ns2:BSART>
    <ns2:BELNR>1908207-5</ns2:BELNR>
    <ns2:ABLAD>CC</ns2:ABLAD>
  </ns2>
  <ns2:E2EDK03>
    <ns2:DATAHEADERCOLUMN_SEGNAM>E2EDK03</ns2:DATAHEADERCOLUMN_SEGNAM>
      <ns2:IDDAT>002</ns2:IDDAT>
      <ns2:DATUM>20160611</ns2:DATUM>
  </ns2:E2EDK03>

```

El ejemplo siguiente es un registro de datos con segmentos agrupados. El registro incluye un nodo primario de grupo (`E2EDKT1002GRP`) y varios nodos secundarios, como `E2EDKT1002` y `E2EDKT2001`. 

```xml

<ns2:E2EDKT1002GRP>
  <ns2:E2EDKT1002>
    <ns2:DATAHEADERCOLUMN_SEGNAM>E2EDKT1002</ns2:DATAHEADERCOLUMN_SEGNAM>
      <ns2:TDID>ZONE</ns2:TDID>
  </ns2:E2EDKT1002>
  <ns2:E2EDKT2001>
    <ns2:DATAHEADERCOLUMN_SEGNAM>E2EDKT2001</ns2:DATAHEADERCOLUMN_SEGNAM>
      <ns2:TDLINE>CRSD</ns2:TDLINE>
  </ns2:E2EDKT2001>
</ns2:E2EDKT1002GRP>

```


El método recomendado es crear un identificador de IDoc para su uso con tRFC. Puede establecer este identificador de transacción (`tid`) mediante la [operación de envío de IDoc](/connectors/sap/#send-idoc) en la API del conector de SAP.

El ejemplo siguiente es un método alternativo para establecer el identificador de la transacción, o `tid`. En este ejemplo, el último nodo de segmento de registro de datos y el nodo de datos de IDoc están cerrados. A continuación, el GUID (`guid`) se usa como el identificador de tRFC para detectar duplicados. 

```xml

    </E2STZUM002GRP>
  </idocData>
  <guid>8820ea40-5825-4b2f-ac3c-b83adc34321c</guid>
</Send>

```

### <a name="add-an-http-request-trigger"></a>Adición de un desencadenador de solicitud HTTP

1. En Azure Portal, cree una aplicación lógica en blanco que abra el diseñador de Logic Apps.

1. En el cuadro de búsqueda, escriba `http request` como filtro. En la lista **Desencadenadores**, seleccione **Cuando se recibe una solicitud HTTP**.

   ![Adición de un desencadenador de solicitud HTTP](./media/logic-apps-using-sap-connector/add-http-trigger-logic-app.png)

1. Ahora puede guardar la aplicación lógica para poder generar una dirección URL del punto de conexión para la aplicación lógica.
En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

   La dirección URL del punto de conexión aparece ahora en el desencadenador, por ejemplo:

   ![Generación de una dirección URL para el punto de conexión](./media/logic-apps-using-sap-connector/generate-http-endpoint-url.png)

### <a name="add-an-sap-action-to-generate-schemas"></a>Adición de una acción de SAP para generar esquemas

1. En el diseñador de Logic Apps, seleccione **Nuevo paso**.

   ![Agregar un nuevo paso a la aplicación lógica](./media/logic-apps-using-sap-connector/add-sap-action-logic-app.png)

1. En el cuadro de búsqueda, escriba `sap` como filtro. En la lista **Actions** (Acciones), seleccione **Generate schemas** (Generar esquemas).
  
   ![Agregar la acción "generar esquemas" a la aplicación lógica](media/logic-apps-using-sap-connector/select-sap-schema-generator-action.png)

   O bien, puede elegir la pestaña **Enterprise** y seleccionar la acción de SAP.

   ![Selección de la acción de envío de SAP desde la pestaña Enterprise](media/logic-apps-using-sap-connector/select-sap-schema-generator-ent-tab.png)

1. Si la conexión ya existe, continúe con el paso siguiente para configurar la acción de SAP. Sin embargo, si se le solicitan detalles de conexión, proporcione la información para que pueda crear una conexión a su servidor SAP local ahora.

   1. Proporcione un nombre para la conexión.

   1. En la sección **Puerta de enlace de datos**, en **Suscripción**, primero seleccione la suscripción de Azure para el recurso de puerta de enlace de datos que creó en el Azure Portal para la instalación de su puerta de enlace de datos. 
   
   1. En **Puerta de enlace de conexión**, seleccione su recurso de puerta de enlace de datos en Azure.

   1. Siga proporcionando información acerca de la conexión. Para la propiedad **Tipo de inicio de sesión**, siga el paso en función de si la propiedad está establecida en **Servidor de aplicaciones** o **Grupo**:
   
      * Para **Servidos de aplicaciones**, estas propiedades, que generalmente parecen opcionales, son necesarias:

        ![Creación de conexiones del servidor de aplicaciones de SAP](media/logic-apps-using-sap-connector/create-SAP-application-server-connection.png)

      * Para **Grupo**, estas propiedades, que generalmente parecen opcionales, son necesarias:

        ![Creación de conexiones del servidor de mensajes de SAP](media/logic-apps-using-sap-connector/create-SAP-message-server-connection.png)  

      De forma predeterminada, se usa el establecimiento inflexible de tipos para comprobar si hay valores no válidos, para lo cual se realiza la validación de XML con respecto al esquema. Este comportamiento puede ayudarlo a detectar problemas con antelación. La opción **Escritura segura** está disponible para la compatibilidad con versiones anteriores y solo comprueba la longitud de cadena. Más información sobre la opción [Escritura segura](#safe-typing).

   1. Cuando haya finalizado, seleccione **Crear**.

      Logic Apps configura y comprueba la conexión para asegurarse de que funciona correctamente.

1. Proporcione la ruta de acceso del artefacto para el que quiere generar el esquema.

   Puede seleccionar la acción de SAP desde el selector de archivos:

   ![Selección de una acción de SAP](media/logic-apps-using-sap-connector/select-SAP-action-schema-generator.png)  

   O bien, puede escribir manualmente la acción:

   ![Introducción manual de una acción de SAP](media/logic-apps-using-sap-connector/manual-enter-SAP-action-schema-generator.png)

   Para generar esquemas de más de un artefacto, proporcione los detalles de la acción de SAP de cada artefacto, por ejemplo:

   ![Selección de la opción para agregar un elemento](media/logic-apps-using-sap-connector/schema-generator-array-pick.png)

   ![Visualización de dos elementos](media/logic-apps-using-sap-connector/schema-generator-example.png)

   Para más información sobre la acción de SAP, vea [Esquemas de mensaje para operaciones de IDoc](/biztalk/adapters-and-accelerators/adapter-sap/message-schemas-for-idoc-operations).

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

### <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

1. En la barra de herramientas del diseñador, elija **Run** (Ejecutar) para desencadenar una ejecución de la aplicación lógica.

1. Abra la ejecución y compruebe las salidas de la acción **Generate schemas** (Generar esquemas).

   Las salidas muestran los esquemas generados para la lista de mensajes especificada.

### <a name="upload-schemas-to-an-integration-account"></a>Carga de esquemas en una cuenta de integración

Si lo desea, puede descargar o almacenar los esquemas generados en repositorios, como un blob, el almacenamiento o la cuenta de integración. Las cuentas de integración proporcionan una excelente experiencia con otras acciones de XML, por lo que en este ejemplo se muestra cómo cargar los esquemas en una cuenta de integración de la misma aplicación lógica mediante el conector de Azure Resource Manager.

1. En el diseñador de Logic Apps, seleccione **Nuevo paso**.

1. En el cuadro de búsqueda, escriba `Resource Manager` como filtro. Seleccione **Create or update a resource** (Crear o actualizar un recurso).

   ![Selección de la acción de Azure Resource Manager](media/logic-apps-using-sap-connector/select-azure-resource-manager-action.png)

1. Escriba los detalles, incluida su suscripción a Azure, el grupo de recursos de Azure y la cuenta de integración. Para agregar tokens de SAP a los campos, haga clic dentro del cuadro de esos campos y seleccione de la lista de contenido dinámico que aparece.

   1. Abra la lista **Agregar nuevo parámetro** y seleccione los campos **Ubicación** y **Propiedades**.

   1. Proporcione detalles de estos nuevos campos como se muestra en este ejemplo.

      ![Introducción de detalles de la acción de Azure Resource Manager](media/logic-apps-using-sap-connector/azure-resource-manager-action.png)

   La acción **Generate schemas** (Generar esquemas) de SAP genera esquemas como una colección, de modo que el diseñador agrega automáticamente un bucle **For each** a la acción. El siguiente es un ejemplo que muestra cómo aparece esta acción:

   ![Acción de Azure Resource Manager con el bucle "for each"](media/logic-apps-using-sap-connector/azure-resource-manager-action-foreach.png)

   > [!NOTE]
   > Los esquemas utilizan el formato codificado en Base64. Para cargar los esquemas en una cuenta de integración, deben descodificarse mediante la función `base64ToString()`. El siguiente es un ejemplo en el que se muestra el código del elemento `"properties"`:
   >
   > ```json
   > "properties": {
   >    "Content": "@base64ToString(items('For_each')?['Content'])",
   >    "ContentType": "application/xml",
   >    "SchemaType": "Xml"
   > }
   > ```

1. Guarde la aplicación lógica. En la barra de herramientas del diseñador, seleccione **Save** (Guardar).

### <a name="test-your-logic-app"></a>Comprobación de la aplicación lógica

1. En la barra de herramientas del diseñador, elija **Run** (Ejecutar) para desencadenar manualmente la aplicación lógica.

1. Tras ejecutarse correctamente, vaya a la cuenta de integración y compruebe que existen los esquemas generados.

## <a name="enable-secure-network-communications"></a>Habilitación de las comunicaciones de red seguras

Antes de empezar, asegúrese de que cumple los [requisitos previos](#prerequisites) enumerados anteriormente, que solo se aplican cuando se usa la puerta de enlace de datos y las aplicaciones lógicas se ejecutan en instancias multiinquilino de Azure:

* Asegúrese de que la puerta de enlace de datos local está instalada en un equipo que se encuentre en la misma red que el sistema SAP.

* Para el inicio de sesión único, la puerta de enlace de datos se ejecuta con un usuario que se asigna a un usuario de SAP.

* La biblioteca de SNC que proporciona las funciones de seguridad adicionales está instalada en la misma máquina que la puerta de enlace de datos. Algunos ejemplos incluyen [sapseculib](https://help.sap.com/saphelp_nw74/helpdata/en/7a/0755dc6ef84f76890a77ad6eb13b13/frameset.htm), Kerberos y NTLM.

   Para habilitar SNC en las solicitudes a o desde el sistema SAP, active la casilla **Usar SNC** en la conexión de SAP y proporcione estas propiedades:

   ![Configuración de SAP SNC en la conexión](media/logic-apps-using-sap-connector/configure-sapsnc.png)

   | Propiedad | Descripción |
   |----------| ------------|
   | **SNC Library Path** (Ruta de acceso a la biblioteca de SNC) | El nombre de la biblioteca de SNC o la ruta de acceso relativa a la ubicación de instalación de NCo o la ruta de acceso absoluta. Algunos ejemplos son `sapsnc.dll`, `.\security\sapsnc.dll` o `c:\security\sapsnc.dll`. |
   | **SNC SSO** (SSO de SNC) | Cuando se conecta mediante SNC, la identidad de SNC normalmente se usa para autenticar al autor de llamada. Otra opción consiste en invalidarla para que se pueda usar la información de usuario y contraseña para autenticar al autor de llamada, pero la línea siga cifrada. |
   | **SNC My Name** (Mi nombre de SNC) | En la mayoría de los casos, esta propiedad se puede omitir. La solución de SNC instalada conoce normalmente su propio nombre SNC. En el caso de soluciones que admitan varias identidades, puede que deba especificar la identidad que se usará para este destino o servidor en concreto. |
   | **SNC Partner Name** (Nombre del asociado de SNC) | El nombre del SNC back-end. |
   | **SNC Quality of Protection** (Calidad de protección de SNC) | La calidad de servicio que se usará para la comunicación SNC de este destino o servidor en particular. El sistema back-end define el valor predeterminado. El valor máximo lo define el producto de seguridad usado para SNC. |
   |||

   > [!NOTE]
   > No establezca las variables de entorno SNC_LIB y SNC_LIB_64 en la máquina donde tenga la puerta de enlace de datos y la biblioteca de SNC. Si lo hace, tienen prioridad sobre el valor de la biblioteca de SNC pasado a través del conector.

## <a name="safe-typing"></a>Escritura segura

De forma predeterminada, al crear la conexión de SAP se usa el establecimiento inflexible de tipos para comprobar si hay valores no válidos; para ello, se realiza la validación de XML con respecto al esquema. Este comportamiento puede ayudarlo a detectar problemas con antelación. La opción **Escritura segura** está disponible para la compatibilidad con versiones anteriores y solo comprueba la longitud de cadena. Si elige **Escritura segura**, los tipos DATS y TIMS en SAP se tratan como cadenas en lugar de como sus equivalentes XML, `xs:date` y `xs:time`, donde `xmlns:xs="http://www.w3.org/2001/XMLSchema"`. La escritura segura afecta al comportamiento de generación de todos los esquemas, al mensaje de envío de "se ha enviado" la carga y de "se ha recibido" la respuesta y al desencadenador. 

Cuando se usa el establecimiento inflexible de tipos (la opción **Escritura segura** no está habilitada), el esquema asigna los tipos DATS y TIMS a tipos XML más sencillos:

```xml
<xs:element minOccurs="0" maxOccurs="1" name="UPDDAT" nillable="true" type="xs:date"/>
<xs:element minOccurs="0" maxOccurs="1" name="UPDTIM" nillable="true" type="xs:time"/>
```

Cuando se envían mensajes con establecimiento inflexible de tipos, la respuesta DATS y TIMS se adapta al formato del tipo XML coincidente:

```xml
<DATE>9999-12-31</DATE>
<TIME>23:59:59</TIME>
```

Cuando la opción **Escritura segura** está habilitada, el esquema solo asigna los tipos DATS y TIMS a campos de cadena XML con restricciones de longitud, por ejemplo:

```xml
<xs:element minOccurs="0" maxOccurs="1" name="UPDDAT" nillable="true">
  <xs:simpleType>
    <xs:restriction base="xs:string">
      <xs:maxLength value="8" />
    </xs:restriction>
  </xs:simpleType>
</xs:element>
<xs:element minOccurs="0" maxOccurs="1" name="UPDTIM" nillable="true">
  <xs:simpleType>
    <xs:restriction base="xs:string">
      <xs:maxLength value="6" />
    </xs:restriction>
  </xs:simpleType>
</xs:element>
```

Cuando se envían mensajes con **Escritura segura** habilitada, la respuesta DATS y TIMS se parece a la de este ejemplo:

```xml
<DATE>99991231</DATE>
<TIME>235959</TIME>
```

## <a name="advanced-scenarios"></a>Escenarios avanzados

### <a name="change-language-headers"></a>Cambio de los encabezados de idioma

Cuando se conecta a SAP desde Logic Apps, el idioma predeterminado de la conexión es el inglés. Puede establecer el idioma de la conexión mediante el [encabezado HTTP estándar `Accept-Language`](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html#sec14.4) con las solicitudes entrantes.

> [!TIP]
> La mayoría de los exploradores web agregan un encabezado `Accept-Language` basado en la configuración del usuario. El explorador web aplica este encabezado cuando se crea una nueva conexión de SAP en el diseñador de Logic Apps. Si no desea crear conexiones de SAP en el idioma preferido del explorador web, actualice la configuración del explorador web para que use el idioma que prefiera o cree la conexión de SAP mediante Azure Resource Manager en lugar del diseñador de Logic Apps. 

Por ejemplo, puede enviar una solicitud con el encabezado `Accept-Language` a la aplicación lógica mediante el desencadenador de la **solicitud HTTP**. Todas las acciones de la aplicación lógica reciben el encabezado. A continuación, SAP usa los idiomas especificados en sus mensajes del sistema, como los mensajes de error de BAPI.

Los parámetros de conexión de SAP para una aplicación lógica no tienen una propiedad de idioma. Por lo tanto, si usa el encabezado `Accept-Language`, es posible que reciba el siguiente error: **Please check your account info and/or permissions and try again** (Compruebe la información de la cuenta o los permisos e inténtelo de nuevo). En este caso, compruebe los registros de errores del componente de SAP en su lugar. En realidad, el error se produce en el componente de SAP que usa el encabezado, por lo que podría obtener uno de estos mensajes de error:

* `"SAP.Middleware.Connector.RfcLogonException: Select one of the installed languages"`
* `"SAP.Middleware.Connector.RfcAbapMessageException: Select one of the installed languages"`

### <a name="confirm-transaction-explicitly"></a>Confirmación de la transacción explícitamente

Cuando se envían transacciones a SAP desde Logic Apps, este intercambio se produce en dos pasos, tal como se describe en el documento de SAP [Transactional RFC Server Programs](https://help.sap.com/doc/saphelp_nwpi71/7.1/22/042ad7488911d189490000e829fbbd/content.htm?no_cache=true) (Programas del servidor RFC transaccional). De manera predeterminada, la acción **Enviar a SAP** controla ambos pasos para la transferencia de la función y la confirmación de la transacción en una sola llamada. El conector de SAP ofrece la opción de desacoplar estos pasos. Puede enviar un IDoc y, en lugar de confirmar automáticamente la transacción, puede usar la acción explícita **Confirmar id. de transacción**.

Esta capacidad de desacoplar la confirmación del id. de transacción resulta útil cuando no quiere duplicar las transacciones en SAP, por ejemplo, en escenarios en los que se pueden producir errores debido a causas como problemas de red. Al confirmar el id. de la transacción por separado, la transacción solo se completa una vez en el sistema SAP.

Este es un ejemplo que muestra este patrón:

1. Cree una aplicación lógica en blanco y agregue un desencadenador HTTP.

1. En el conector de SAP, agregue la acción **Enviar IDOC**. Proporcione los detalles del IDoc que envía al sistema SAP.

1. Para confirmar explícitamente el ID de la transacción en un paso separado, en el campo **Confirmar TID**, seleccione **No**. Para el campo opcional **GUID del id. de transacción**, puede especificar manualmente el valor o hacer que el conector genere y devuelva automáticamente este GUID en la respuesta de la acción Enviar IDOC.

   ![Propiedades de la acción Enviar IDOC](./media/logic-apps-using-sap-connector/send-idoc-action-details.png)

1. Para confirmar explícitamente el id. de transacción, agregue la acción **Confirmar id. de transacción**, asegurándose de [evitar el envío de IDoc duplicados a SAP](#avoid-sending-duplicate-idocs). Haga clic en el cuadro **Id. de transacción** para que aparezca la lista de contenido dinámico. En esa lista, seleccione el valor de **Id. de transacción** que devuelve la acción **Enviar IDOC**.

   ![Acción Confirmar id. de transacción](./media/logic-apps-using-sap-connector/explicit-transaction-id.png)

   Después de ejecutar este paso, la transacción actual se marca como completa en ambos extremos, en el lado del conector de SAP y en el lado del sistema SAP.

#### <a name="avoid-sending-duplicate-idocs"></a>Evitar el envío de IDoc duplicados

Si experimenta un problema con IDoc duplicados que se envían a SAP desde la aplicación lógica, siga estos pasos para crear una variable de cadena que sirva como el identificador de la transacción de IDoc. La creación de este identificador de transacción ayuda a evitar transmisiones de red duplicadas cuando hay problemas, como interrupciones temporales, problemas de red o confirmaciones perdidas.

> [!NOTE]
> Los sistemas SAP olvidan un identificador de transacción después de un tiempo especificado, o 24 horas de forma predeterminada. Por consiguiente, SAP nunca no puede confirmar un identificador de transacción si se desconoce el id. o el GUID.
> Si se produce un error en la confirmación de un identificador de transacción, este error indica que hubo un problema en la comunicación con el sistema SAP antes de que SAP pudiera reconocer la confirmación.

1. En el diseñador de Logic Apps, agregue la acción **Inicializar la variable** a la aplicación lógica. 

1. En el editor de la acción **Inicializar la variable**, defina las siguientes configuraciones. A continuación, guarde los cambios.

    1. Para **Nombre**, escriba un nombre para la variable. Por ejemplo, `IDOCtransferID`.

    1. Para **Tipo**, seleccione **Cadena** como el tipo de variable.

    1. Para **Valor**, seleccione el cuadro de texto **Escriba el valor inicial** para abrir el menú de contenido dinámico. Seleccione la pestaña **Expresiones**. En la lista de funciones, escriba la función `guid()`. Después, seleccione **Aceptar** para guardar los cambios. El campo **Valor** se establece ahora en la función `guid()`, que genera un GUID.

1. Después de la acción **Inicializar la variable**, agregue la acción **Enviar de IDoc**.

1. En el editor de la acción **Enviar IDOC**, defina las siguientes configuraciones. A continuación, guarde los cambios.

    1. Para **Tipo de IDOC** seleccione el tipo de mensaje y, en **Mensaje de entrada de IDOC**, especifique el mensaje.

    1. Para **Versión de SAP**, seleccione los valores de la configuración de SAP.

    1. Para **Versión de tipos de registro**, seleccione los valores de la configuración de SAP.

    1. Para **Confirmar TID**, seleccione **No**.

    1. Seleccione **Add new parameter list** (Agregar nueva lista de parámetros) > **GUID del id. de transacción**. Seleccione el cuadro de texto para abrir el menú de contenido dinámico. En la pestaña **Variables**, seleccione el nombre de la variable que creó. Por ejemplo, `IDOCtransferID`.

1. En la barra de título de la acción **Enviar IDOC**, seleccione **...**  > **Configuración**. Para **Directiva de reintentos**, se recomienda seleccionar **Valor predeterminado** &gt; **Listo**. Sin embargo, en su lugar, puede configurar una directiva personalizada para sus necesidades específicas. En el caso de las directivas personalizadas, se recomienda configurar al menos un reintento para superar las interrupciones temporales de la red.

1. Después de la acción **Enviar IDOC**, agregue la acción **Confirmar id. de transacción**.

1. En el editor de la acción **Confirmar id. de transacción**, defina las siguientes configuraciones. A continuación, guarde los cambios.

    1. Para **Id. de transacción**, escriba de nuevo el nombre de la variable. Por ejemplo, `IDOCtransferID`.

1. Opcionalmente, valide la desduplicación en el entorno de prueba. Repita la acción **Enviar IDoc** con el mismo GUID del **identificador de transacción** que usó en el paso anterior. Cuando se envía el mismo IDoc dos veces, puede validar que SAP pueda identificar la duplicación de la llamada tRFC y resolver las dos llamadas a un solo mensaje entrante de IDoc.

## <a name="known-issues-and-limitations"></a>Limitaciones y problemas conocidos

Estos son los problemas y limitaciones actualmente conocidos para el conector de SAP (que no sea ISE) administrado: 

* En general, el desencadenador de SAP no admite los clústeres de puerta de enlace de datos. En algunos casos de conmutación por error, el nodo de puerta de enlace de datos que se comunica con el sistema SAP puede diferir del nodo activo, lo que produce un comportamiento inesperado.

  * En los escenarios de envío, se admiten clústeres de puerta de enlace de datos en modo de conmutación por error. 

  * Los clústeres de puerta de enlace de datos en modo de equilibrio de carga no se admiten con las acciones de SAP con estado. Estas acciones incluyen la **creación de una sesión con estado**, la **confirmación de una transacción BAPI**, la **reversión de transacción BAPI**, el **cierre de la sesión con estado** y todas las acciones que especifican un valor de **identificador de sesión**. Las comunicaciones con estado deben permanecer en el mismo nodo de clúster de puerta de enlace de datos. 

  * En el caso de las acciones de SAP con estado, use la puerta de enlace de datos en modo no clúster o en un clúster configurado solo para la conmutación por error.

* El conector SAP no admite actualmente las cadenas de enrutador SAP. La puerta de enlace de datos local debe existir en la misma LAN que el sistema SAP al que quiere conectarse.

## <a name="connector-reference"></a>Referencia de conectores

Si necesita más detalles técnicos sobre este conector, como los desencadenadores, las acciones y los límites que se describen en el archivo de Swagger del conector, vea la [página de referencia del conector](/connectors/sap/). Se proporciona documentación adicional para Logic Apps para las siguientes acciones:

* [Llamar a BAPI](#call-bapi-action)

* [Envío de IDOC](#send-idoc-action)

> [!NOTE]
> En el caso de las aplicaciones lógicas de un [entorno de servicio de integración (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), la versión con la etiqueta ISE de este conector usa en su lugar los [límites de mensajes de ISE](../logic-apps/logic-apps-limits-and-config.md#message-size-limits).

### <a name="call-bapi-action"></a>Acción Llamar a BAPI

La acción [Llamar a BAPI (`CallBapi`)](
https://docs.microsoft.com/connectors/sap/#call-bapi-(preview)) llama al método BAPI en el servidor de SAP. 

Debe usar los parámetros siguientes con la llamada: 

* **Objeto de negocios** (`businessObject`), que es un menú desplegable que permite búsquedas.

* **Método** (`method`), que rellena los métodos disponibles después de seleccionar **Business Object**. Los métodos disponibles varían en función del **objeto de negocios** seleccionado.

* **Parámetros de BAPI de entrada** (`body`), en los que se llama al documento XML que contiene los valores de parámetro de entrada del método BAPI para la llamada, o al URI de Storage Blob que contiene los parámetros de BAPI.

Para obtener ejemplos detallados de cómo usar la acción Llamar a BAPI, consulte la [Ejemplos XML para solicitudes de BAPI](#xml-samples-for-bapi-requests).

> [!TIP]
> Si usa el diseñador de Logic Apps para editar la solicitud de BAPI, puede usar las siguientes funciones de búsqueda: 
> 
> * Seleccione un objeto en el diseñador para ver un menú desplegable de los métodos disponibles.
> * Filtre los tipos de objetos comerciales por palabra clave mediante la lista de búsqueda que la llamada API de BAPI proporciona.

### <a name="send-idoc-action"></a>Acción Enviar IDoc

La acción [Enviar IDoc (`SendIDoc`)](/connectors/sap/) envía el mensaje de IDoc al servidor de SAP.

Debe usar los parámetros siguientes con la llamada: 

* **Tipo de IDoc con extensión opcional** (`idocType`), que es un menú desplegable que permite búsquedas.

    * El parámetro opcional de **versión de lanzamiento de SAP** (`releaseVersion`) rellena los valores después de seleccionar el tipo de IDoc y depende del tipo de IDoc seleccionado.

* **Mensaje de IDoc de entrada** (`body`), en el que se llama al documento XML que contiene la carga de IDoc, o al URI de Storage Blob que contiene el documento XML de IDoc. Este documento debe cumplir el esquema XML de IDoc de SAP de acuerdo con la documentación de IDoc WE60 o el esquema generado para el URI de acción de IDoc de SAP coincidente.

Para obtener ejemplos detallados de cómo usar la acción Enviar IDoc, consulte el [tutorial para enviar mensajes de IDoc al servidor de SAP](#send-idoc-messages-to-sap-server).

Para saber cómo usar el parámetro opcional **Confirmar TID** (`confirmTid`), consulte el [tutorial para confirmar la transacción de forma explícita](#confirm-transaction-explicitly).

## <a name="next-steps"></a>Pasos siguientes

* [Conéctese a sistemas locales](../logic-apps/logic-apps-gateway-connection.md) desde Azure Logic Apps.

* Aprenda a validar, transformar y usar otras operaciones de mensaje con [Enterprise Integration Pack](../logic-apps/logic-apps-enterprise-integration-overview.md).

* Conozca otros [conectores de Logic Apps](../connectors/apis-list.md).