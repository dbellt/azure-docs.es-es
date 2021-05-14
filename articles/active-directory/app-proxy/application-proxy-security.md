---
title: Consideraciones sobre la seguridad de Azure Active Directory Application Proxy
description: Se tratan las consideraciones de seguridad al utilizar el Proxy de aplicación de Azure AD.
services: active-directory
author: kenwith
manager: mtillman
ms.service: active-directory
ms.subservice: app-proxy
ms.workload: identity
ms.topic: conceptual
ms.date: 04/21/2021
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: a6951b6d52fad3b0d2bbed41422600d5f28892f0
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108186734"
---
# <a name="security-considerations-for-accessing-apps-remotely-with-azure-active-directory-application-proxy"></a>Consideraciones de seguridad para acceder a aplicaciones de forma remota con Azure Active Directory Application Proxy

En este artículo se explican los componentes que funcionan para mantener a los usuarios y las aplicaciones seguros cuando se usa el Proxy de aplicación de Azure Active Directory.

El diagrama siguiente muestra cómo Azure AD permite el acceso remoto seguro a sus aplicaciones locales.

 ![Diagrama de acceso remoto seguro a través del Proxy de aplicación de Azure AD](./media/application-proxy-security/secure-remote-access.png)

## <a name="security-benefits"></a>Ventajas de seguridad

El Proxy de aplicación de Azure AD ofrece las siguientes ventajas en materia de seguridad:

### <a name="authenticated-access"></a>Acceso autenticado 

Si decide usar Azure la autenticación previa de Azure Active Directory, solo las conexiones autenticadas pueden acceder a la red.

El Proxy de aplicación de Azure AD se basa en el servicio de token de seguridad (STS) de Azure AD para toda la autenticación.  La autenticación previa, por su propia naturaleza, bloquea un número significativo de ataques anónimos, ya que las identidades autenticadas son las únicas que pueden tener acceso a la aplicación de back-end.

Si elige el acceso directo como método de autenticación previa, no disfruta de esta ventaja. 

### <a name="conditional-access"></a>Acceso condicional

Aplique controles de directiva más completos antes de que se establezcan las conexiones a la red.

Con el [acceso condicional](../conditional-access/concept-conditional-access-cloud-apps.md), es posible definir las restricciones sobre cómo los usuarios pueden acceder a sus aplicaciones. Puede crear directivas que restrinjan los inicios de sesión en función de la ubicación, el nivel de autenticación y el perfil de riesgo del usuario.

También puede usar el acceso condicional para configurar directivas de Multi-Factor Authentication, agregando otro nivel de seguridad para la autenticación de los usuarios. Además, las aplicaciones también pueden enrutarse a Microsoft Cloud App Security mediante el acceso condicional de Azure AD para proporcionar supervisión y controles en tiempo real, mediante las directivas de [acceso](/cloud-app-security/access-policy-aad) y [sesión](/cloud-app-security/session-policy-aad).

### <a name="traffic-termination"></a>Terminación del tráfico

Todo el tráfico se termina en la nube.

Como el Proxy de aplicación de Azure AD es un servidor proxy inverso, todo el tráfico que va a las aplicaciones de back-end se termina en el servicio. Se puede restablecer la sesión solo con el servidor back-end, lo que significa que los servidores back-end no se exponen al tráfico HTTP directo. Esta configuración implica que está mejor protegido frente a ataques dirigidos.

### <a name="all-access-is-outbound"></a>Todo el acceso es de salida. 

No es necesario abrir conexiones de entrada a la red corporativa.

Los conectores de Proxy de aplicación solo usan conexiones salientes al servicio Azure AD Application Proxy, lo que significa que no es necesario abrir los puertos de firewall para las conexiones entrantes. Los servidores proxy tradicionales requerían una red perimetral (también conocida como *DMZ* *o* *subred filtrada*) y permitían el acceso a las conexiones no autenticadas en el perímetro de la red. Este escenario hacía necesaria inversiones adicionales en productos de firewall de aplicaciones web para analizar el tráfico y proteger el entorno. Con Proxy de aplicación no se necesita una red perimetral, ya que todas las conexiones son de salida y tienen lugar a través de un canal seguro.

Para más información sobre los conectores, consulte [Descripción de los conectores del Proxy de aplicación de Azure AD](application-proxy-connectors.md).

### <a name="cloud-scale-analytics-and-machine-learning"></a>Análisis de escala en la nube y aprendizaje automático 

Obtenga una protección de seguridad avanzada.

Dado que forma parte de Azure Active Directory, el Proxy de aplicación puede sacar provecho de [Azure AD Identity Protection](../identity-protection/overview-identity-protection.md), con datos de Microsoft Security Response Center y Digital Crimes Unit. Juntos identificamos de manera proactiva las cuentas en peligro y ofrecemos protección frente a inicios de sesión de alto riesgo. Para determinar qué intentos de inicio de sesión son de alto riesgo se tienen en cuenta diversos factores. Estos factores incluyen el marcado de dispositivos infectados, el acceso desde redes anónimas, y ubicaciones atípicas o poco probables.

Muchos de estos informes y eventos ya están disponibles a través de una API para la integración con los sistemas de información de seguridad y administración de eventos (SIEM).

### <a name="remote-access-as-a-service"></a>Acceso remoto en forma de servicio

No tiene que preocuparse sobre cómo mantener y aplicar revisiones a los servidores locales.

El software al que no se aplican revisiones sigue siendo objeto de un gran número de ataques. El proxy de aplicación de Azure AD es un servicio de escala de Internet que es propiedad de Microsoft, por lo que siempre obtendrá las actualizaciones y revisiones de seguridad más recientes.

Para mejorar la seguridad de las aplicaciones publicadas por el Proxy de aplicación de Azure AD, se impide que los robots de agentes de búsqueda indexen y archiven sus aplicaciones. Cada vez que un robot de agente de búsqueda intenta recuperar la configuración del robot de una aplicación publicada, el Proxy de aplicación responde con un archivo robots.txt que incluye `User-agent: * Disallow: /`.

#### <a name="azure-ddos-protection-service"></a>Servicio Azure DDoS Protection

Las aplicaciones publicadas a través del servicio de proxy de aplicación están protegidas frente a ataques de denegación de servicio distribuido (DDoS). Esta protección la administra Microsoft y se habilita automáticamente en todos nuestros centros de recursos. El servicio Azure DDoS Protection proporciona supervisión continua del tráfico y mitigación en tiempo real de los ataques de nivel de red habituales. 

## <a name="under-the-hood"></a>En segundo plano

El Proxy de aplicación de Azure AD consta de dos partes:

* El servicio en la nube: Este servicio se ejecuta en Azure y es donde se realizan las conexiones de usuario o cliente externos.
* [El conector local](application-proxy-connectors.md): en un componente local, el conector atiende las solicitudes del servicio Azure AD Application Proxy y controla las conexiones con las aplicaciones internas. 

Se establece un flujo entre el conector y el servicio de Proxy de aplicación cuando:

* El conector se configura en primer lugar.
* El conector extrae información de configuración desde el servicio Application Proxy.
* Un usuario tiene acceso a una aplicación publicada.

>[!NOTE]
>Todas las comunicaciones se producen a través de TLS, y siempre se originan en el conector del servicio Proxy de aplicación. Este servicio solo es de salida.

El conector usa un certificado de cliente para autenticarse en el servicio de Proxy de aplicación para casi todas las llamadas. La única excepción para este proceso es el momento de la instalación inicial, donde se establece el certificado de cliente.

### <a name="installing-the-connector"></a>Instalación del conector

Los siguientes eventos de flujo ocurren cuando el conector se configura en primer lugar:

1. El registro del conector en el servicio se produce en el marco de la instalación del conector. Se solicita a los usuarios que escriban sus credenciales de administrador de Azure AD.  A continuación, el token obtenido con esta autenticación se presenta al servicio Azure AD Application Proxy.
2. El servicio Application Proxy evalúa el token. Comprueba si el usuario es Administrador global en el inquilino.  Si el usuario no es un administrador, el proceso se termina.
3. El conector genera una solicitud de certificado de cliente y la pasa, junto con el token, al servicio Application Proxy. A su vez, el servicio comprueba el token y firma la solicitud de certificado de cliente.
4. El conector usa el certificado de cliente para la comunicación posterior con el servicio de Proxy de aplicación.
5. El conector realiza una extracción inicial de los datos de configuración del sistema desde el servicio usando su certificado de cliente y ya está listo para aceptar solicitudes.

### <a name="updating-the-configuration-settings"></a>Actualización de los valores de configuración

Cada vez que el servicio de Proxy de aplicación actualiza los valores de configuración, tienen lugar los siguientes eventos de flujo:

1. El conector se conecta al punto de conexión de configuración en el servicio de Proxy de aplicación mediante su certificado de cliente.
2. Una vez que se valida el certificado de cliente, el servicio de Proxy de aplicación devuelve datos de configuración al conector; por ejemplo, el grupo de conectores al que debe pertenecer el conector.
3. El conector genera una nueva solicitud de certificado si el certificado actual tiene más de 180 días de antigüedad, lo cual permite actualizar de manera efectiva el certificado de cliente cada 180 días.

### <a name="accessing-published-applications"></a>Acceso a aplicaciones publicadas

Cuando los usuarios tienen acceso a una aplicación publicada, los siguientes eventos tienen lugar entre el servicio Application Proxy y el conector del proxy de aplicación:

1. El servicio autentica al usuario de la aplicación
2. El servicio realiza una solicitud en la cola del conector
3. Un conector procesa la solicitud de la cola
4. El conector espera una respuesta
5. El servicio transmite en secuencias los datos al usuario

Para más información sobre lo que ocurre en cada uno de estos pasos, continúe leyendo.


#### <a name="1-the-service-authenticates-the-user-for-the-app"></a>1. El servicio autentica al usuario de la aplicación

Si ha configurado la aplicación para que use el acceso directo como método de autenticación previa, se omiten los pasos descritos en esta sección.

Si la aplicación está configurada para usar la autenticación previa con Azure AD, se redirige a los usuarios a STS de Azure AD para que se autentiquen y se siguen los pasos a continuación:

1. El proxy de aplicación comprueba los requisitos de directiva de acceso condicional para la aplicación específica. Este paso garantiza que el usuario se ha asignado a la aplicación. Si se requiere la verificación en dos pasos, la secuencia de autenticación solicita al usuario un segundo método de autenticación.

2. Una vez que hayan pasado todas las comprobaciones, el STS de Azure AD emite un token firmado para la aplicación y redirige al usuario al servicio de Proxy de aplicación.

3. El proxy de aplicación comprueba que el token se haya emitido para corregir la aplicación. También realiza otras comprobaciones como verificar que el token se firmó con Azure AD y que sigue dentro de la ventana válida.

4. El Proxy de aplicación establece una cookie de autenticación cifrada para indicar que se ha producido la autenticación para la aplicación. Esta cookie incluye una marca de tiempo de expiración basada en el token de Azure AD y otros datos, como el nombre de usuario en que se basa la autenticación. Esta cookie se cifra con una clave privada conocida solo por el servicio de Proxy de aplicación.

5. El Proxy de aplicación redirige al usuario a la dirección URL solicitada originalmente.

Si se produce un error en cualquier parte de los pasos de la autenticación previa, se deniega la solicitud del usuario y este ve un mensaje que indica el origen del problema.


#### <a name="2-the-service-places-a-request-in-the-connector-queue"></a>2. El servicio realiza una solicitud en la cola del conector

Los conectores mantienen abierta una conexión de salida hacia el servicio Application Proxy. Cuando llega una solicitud, el servicio la pone en cola en una de las conexiones abiertas para que el conector la recoja.

La solicitud incluye elementos de la aplicación, como los encabezados de la solicitud y los datos de la cookie cifrada, el usuario que realiza la solicitud y el identificador de solicitud. Aunque los datos de la cookie cifrada se envían con la solicitud, no es la propia cookie de autenticación.

#### <a name="3-the-connector-processes-the-request-from-the-queue"></a>3. El conector procesa la solicitud de la cola. 

En función de la solicitud, el Proxy de aplicación realizará una de las siguientes acciones:

* Si la solicitud es una operación sencilla, como cuando no hay ningún dato dentro del cuerpo (por ejemplo, una solicitud *GET* de RESTful), el conector establece una conexión con el recurso interno de destino y espera una respuesta.

* Si la solicitud tiene datos asociados en el cuerpo, como en una operación de *POST* de RESTful, el conector establece una conexión saliente mediante el certificado de cliente con la instancia del Proxy de aplicación. Esta conexión se establece para solicitar los datos y abrir una conexión a los recursos internos. Tras la recepción de la solicitud del conector, el servicio de Proxy de aplicación comienza a aceptar el contenido del usuario y reenvía los datos al conector. El conector, a su vez, reenvía los datos a los recursos internos.

#### <a name="4-the-connector-waits-for-a-response"></a>4. El conector espera una respuesta.

Una vez completada la operación de solicitud y transmisión de todo el contenido al back-end, el conector espera una respuesta.

Después de recibir una respuesta, el conector establece una conexión saliente con el servicio de Proxy de aplicación para devolver los detalles del encabezado y empezar a transmitir los datos de devolución.

#### <a name="5-the-service-streams-data-to-the-user"></a>5. El servicio transmite por secuencias los datos al usuario. 

Aquí puede tener lugar cierto procesamiento de la aplicación. Si configura el proxy de aplicación para traducir los encabezados o las direcciones URL en la aplicación, ese procesamiento se produce según sea necesario durante este paso.


## <a name="next-steps"></a>Pasos siguientes

[Consideraciones sobre la topología de red al usar el Proxy de aplicación de Azure Active Directory](application-proxy-network-topology.md)

[Descripción de los conectores del Proxy de aplicación de Azure AD](application-proxy-connectors.md)