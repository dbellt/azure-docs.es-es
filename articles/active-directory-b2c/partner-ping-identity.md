---
title: Tutorial para configurar Azure Active Directory B2C con Ping Identity
titleSuffix: Azure AD B2C
description: Aprenda a integrar la autenticación de Azure AD B2C con Ping Identity
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 01/20/2021
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 066fd6f91b19da211a73ac12fb6dca94085399ac
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107256640"
---
# <a name="tutorial-configure-ping-identity-with-azure-active-directory-b2c-for-secure-hybrid-access"></a>Tutorial: Configuración de Ping Identity con Azure Active Directory B2C para el acceso híbrido seguro

En este tutorial de ejemplo va a aprender a extender Azure Active Directory (AD) B2C con [PingAccess](https://www.pingidentity.com/en/software/pingaccess.html#:~:text=%20Modern%20Access%20Managementfor%20the%20Digital%20Enterprise%20,consistent%20enforcement%20of%20security%20policies%20by...%20More) y [PingFederate](https://www.pingidentity.com/en/software/pingfederate.html) a fin de habilitar el acceso híbrido seguro.

Muchas propiedades web existentes, como los sitios y las aplicaciones web de comercio electrónico, expuestas a Internet se implementan detrás de un sistema proxy que a veces se conoce como sistema proxy inverso. Estos sistemas proxy proporcionan varias funciones que incluyen la autenticación previa, la aplicación de directivas y el enrutamiento del tráfico. Los casos de uso de ejemplo incluyen la protección de la aplicación web frente al tráfico web entrante y la aplicación de administración de sesiones uniforme en implementaciones de servidor distribuidas.

En la mayoría de los casos, esta configuración incluye una capa de traducción de autenticación que externaliza la autenticación de la aplicación web. Los proxies inversos proporcionan el contexto de los usuarios autenticados a las aplicaciones web de una forma más sencilla, como un valor de encabezado en formato claro o de resumen. En una configuración de este tipo, las aplicaciones no usan ningún token estándar del sector, como Lenguaje de marcado de aserción de seguridad (SAML), OAuth u Open ID Connect (OIDC), sino que dependen del proxy para proporcionar el contexto de autenticación y mantener la sesión con el agente del usuario final, como el explorador o la aplicación nativa. Como servicio que se ejecuta en un "intermediario", los proxies pueden proporcionar el control de sesión definitivo. Esto también significa que el servicio proxy debe ser muy eficaz y escalable, no convertirse en un cuello de botella ni en un único punto de error para las aplicaciones detrás del servicio proxy. El diagrama es una representación de una implementación de proxy inverso típica y del flujo de las comunicaciones.

![Imagen que muestra la implementación de proxy inverso](./media/partner-ping/reverse-proxy.png)

Si se encuentra en una situación en la que quiere modernizar la plataforma de identidad de tales configuraciones, se planteará las siguientes cuestiones.

- ¿Cómo se puede desvincular el esfuerzo de modernización de la aplicación de la modernización de la plataforma de identidad?

- ¿Cómo se puede establecer un entorno de coexistencia con autenticación moderna y heredada que consuma del proveedor de servicios de identidad modernizado?

  a. ¿Cómo fomentar la coherencia de la experiencia del usuario final?

  b. ¿Cómo proporcionar una experiencia de inicio de sesión única en las aplicaciones que coexisten?

Vamos a hablar de un enfoque para resolver estas cuestiones mediante la integración de Azure AD B2C con las tecnologías [PingAccess](https://www.pingidentity.com/en/software/pingaccess.html#:~:text=%20Modern%20Access%20Managementfor%20the%20Digital%20Enterprise%20,consistent%20enforcement%20of%20security%20policies%20by...%20More) y [PingFederate](https://www.pingidentity.com/en/software/pingfederate.html).

## <a name="coexistence-environment"></a>Entorno de coexistencia

Una solución sencilla técnicamente viable que también resulta rentable es configurar el sistema proxy inverso para que use el sistema de identidad modernizado y delegue la autenticación.  
En este caso, los proxies admiten los protocolos de autenticación modernos y usan la autenticación basada en el redireccionamiento (pasiva) que envía al usuario al nuevo proveedor de identidades (IdP).

### <a name="azure-ad-b2c-as-an-identity-provider"></a>Azure AD B2C como proveedor de identidades

Azure AD B2C tiene la capacidad de definir **directivas** que activan diferentes experiencias y comportamientos de usuario que también se denominan **recorridos del usuario** según la orquestación del extremo de servidor. Cada directiva de este tipo expone un punto de conexión de protocolo que puede realizar la autenticación como si fuera un IdP. No se necesita ningún control especial de las directivas concretas en el lado de la aplicación. La aplicación simplemente realiza una solicitud de autenticación estándar del sector al punto de conexión de autenticación específico del protocolo expuesto por la directiva de interés.  
Azure AD B2C se puede configurar de modo que comparta el mismo emisor en varias directivas o un único emisor para cada directiva. Cada aplicación puede apuntar a una o muchas de estas directivas si realiza una solicitud de autenticación nativa del protocolo y activa comportamientos de usuario deseados, como el inicio de sesión, el registro y las ediciones de perfil. El diagrama muestra los flujos de trabajo de aplicación OIDC y SAML.

![Imagen que muestra la implementación de OIDC y SAML](./media/partner-ping/azure-ad-identity-provider.png)

Aunque el escenario mencionado funciona bien en el caso de las aplicaciones modernizadas, puede ser todo un reto que las aplicaciones heredadas redirijan correctamente al usuario, ya que la solicitud de acceso a las aplicaciones puede no incluir el contexto de la experiencia de usuario. En la mayoría de los casos, la capa de proxy o un agente integrado en la aplicación web intercepta la solicitud de acceso.

### <a name="pingaccess-as-a-reverse-proxy"></a>PingAccess como proxy inverso

Muchos clientes han implementado PingAccess como proxy inverso para desempeñar uno o varios roles, como se ha indicado anteriormente en este artículo. PingAccess puede interceptar una solicitud directa al actuar como intermediario o como un redireccionamiento que proviene de un agente que se ejecuta en el servidor de aplicaciones web.

PingAccess se puede configurar con OIDC, OAuth2 o SAML para realizar la autenticación en un proveedor de autenticación ascendente. Se puede configurar un único IdP ascendente para este fin en el servidor de PingAccess. El diagrama siguiente muestra esta configuración.

![Imagen que muestra PingAccess con la implementación de OIDC](./media/partner-ping/authorization-flow.png)

En una implementación de Azure AD B2C típica en la que varias directivas exponen varios **IdP**, supone un reto, dado que PingAccess solo se puede configurar con un único IdP ascendente.  

### <a name="pingfederate-as-a-federation-proxy"></a>PingFederate como proxy de federación

PingFederate es un puente de identidad empresarial que, en caso necesario, puede configurarse por completo como proveedor de autenticación o proxy para otros IdP ascendentes. En el siguiente diagrama se muestra esta capacidad.

![Imagen que muestra la implementación de PingFederate](./media/partner-ping/pingfederate.png)

Esta capacidad se puede usar para cambiar de forma contextual, dinámica o declarativa una solicitud entrante a una directiva de Azure AD B2C específica. A continuación se representa el flujo de secuencia de protocolo de esta configuración.

![Imagen que muestra el flujo de trabajo de PingAccess y PingFederate](./media/partner-ping/pingaccess-pingfederate-workflow.png)

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesitará lo siguiente:

- Suscripción a Azure. Si no tiene una, obtenga una [cuenta gratuita](https://azure.microsoft.com/free/).

- Un [inquilino de Azure AD B2C](./tutorial-create-tenant.md) vinculado a la suscripción de Azure.

- PingAccess y PingFederate implementados en contenedores de Docker o directamente en máquinas virtuales de Azure.

## <a name="connectivity"></a>Conectividad

Compruebe que los siguientes elementos están conectados.

- **Servidor de PingAccess**: puede comunicarse con el servidor de PingFederate, el explorador cliente, OIDC, OAuth conocido y la detección de claves publicada por el servicio Azure AD B2C y el servidor de PingFederate.

- **Servidor de PingFederate**: puede comunicarse con el servidor de PingAccess, el explorador cliente, OIDC, OAuth conocido y la detección de claves publicada por el servicio Azure AD B2C.

- **Aplicación de AuthN heredada o basada en encabezados**: puede comunicarse con el servidor de PingAccess y desde este.

- **Aplicación de usuario de confianza de SAML**: puede acceder al tráfico del explorador desde el cliente. Puede acceder a los metadatos de federación de SAML publicados por el servicio Azure AD B2C.

- **Aplicación moderna**: puede acceder al tráfico del explorador desde el cliente. Puede acceder a OIDC, OAuth conocido y la detección de claves publicada por el servicio Azure AD B2C.

- **API REST**: puede acceder al tráfico desde un cliente nativo o web. Puede acceder a OIDC, OAuth conocido y la detección de claves publicada por el servicio Azure AD B2C.

## <a name="configure-azure-ad-b2c"></a>Configuración de Azure AD B2C

Para este fin, puede usar los flujos de usuario básicos o las directivas avanzadas de Identity Enterprise Framework (IEF). PingAccess genera el punto de conexión de metadatos en función del valor **Emisor** con la convención de detección basada en [WebFinger](https://tools.ietf.org/html/rfc7033).
Para seguir esta convención, actualice la actualización del emisor de Azure AD B2C mediante las propiedades de las directivas de los flujos de usuario.

![Imagen que muestra la configuración del token](./media/partner-ping/token-setting.png)

En las directivas avanzadas, se puede configurar mediante el elemento de metadatos **IssuanceClaimPattern** en el valor **AuthorityWithTfp** del [perfil técnico del emisor de tokens JWT](./jwt-issuer-technical-profile.md).

## <a name="configure-pingaccesspingfederate"></a>Configuración de PingAccess o PingFederate

En la siguiente sección se habla de la configuración requerida.
El diagrama muestra el flujo de usuario general de la integración.

![Imagen que muestra la integración de PingAccess y PingFederate](./media/partner-ping/pingaccess.png)

### <a name="configure-pingfederate-as-the-token-provider"></a>Configuración de PingFederate como proveedor de tokens

Para configurar PingFederate como proveedor de tokens de PingAccess, asegúrese de que se establezca la conectividad de PingFederate a PingAccess seguida de la conectividad de PingAccess a PingFederate.  
Vea [este artículo](https://docs.pingidentity.com/bundle/pingaccess-61/page/zgh1581446287067.html) para obtener los pasos de configuración.

### <a name="configure-a-pingaccess-application-for-header-based-authentication"></a>Configuración de la autenticación basada en encabezados en una aplicación de PingAccess

Se debe crear una aplicación de PingAccess para la aplicación web de destino con autenticación basada en encabezados. Siga estos pasos.

#### <a name="step-1--create-a-virtual-host"></a>Paso 1: Creación de un host virtual

>[!IMPORTANT]
>Para configurar, es necesario crear un host virtual para cada aplicación. Para obtener más información sobre las consideraciones de configuración y sus impactos, vea [Consideraciones clave](https://docs.pingidentity.com/bundle/pingaccess-43/page/reference/pa_c_KeyConsiderations.html).

Siga estos pasos para crear un host virtual:

1. Vaya a **Configuración** > **Acceso** > **Virtual Hosts** (Hosts virtuales).

2. Seleccione **Add Virtual Host** (Agregar host virtual).

3. En el campo Host, escriba la parte del FQDN de la dirección URL de la aplicación.

4. En el campo Puerto, escriba **443**.

5. Seleccione **Guardar**.

#### <a name="step-2--create-a-web-session"></a>Paso 2: Creación de una sesión web

Siga estos pasos para crear una sesión web:

1. Vaya a **Configuración** > **Acceso** > **Web Sessions** (Sesiones web).

2. Seleccione **Add Web Session** (Agregar sesión web).

3. Proporcione un **Nombre** para la sesión web.

4. Seleccione el **Cookie Type** (Tipo de cookie), ya sea **Signed JWT** (JWT firmado) o **Encrypted JWT** (JWT cifrado).

5. Proporcione un valor único para la **Audiencia**.

6. En el campo **Id. de cliente**, escriba el **Id. de aplicación de Azure AD**.

7. En el campo **Secreto de cliente**, escriba la **Clave** que ha generado para la aplicación en Azure AD.

8. Opcional: puede crear y usar notificaciones personalizadas con Microsoft Graph API. Si decide hacerlo, seleccione **Opciones avanzadas** y desactive las opciones **Request Profile** (Solicitar perfil) y **Refresh User Attributes** (Actualizar atributos de usuario). Para obtener más información sobre el uso de notificaciones personalizadas, vea [Uso de una notificación personalizada](../active-directory/manage-apps/application-proxy-configure-single-sign-on-with-headers.md).

9. Seleccione **Guardar**.

#### <a name="step-3--create-identity-mapping"></a>Paso 3: Creación de una asignación de identidad

>[!NOTE]
>La asignación de identidad se puede usar con varias aplicaciones si más de una espera los mismos datos en el encabezado.

Siga estos pasos para crear una asignación de identidad:

1. Vaya a **Configuración** > **Acceso** > **Identity Mappings** (Asignaciones de identidad).

2. Seleccione **Add Identity Mapping** (Agregar asignación de identidad).

3. Especifique un **Nombre**.

4. Seleccione el tipo de asignación **Type of Header Identity Mapping** (Tipo de asignación de identidad de encabezado).

5. En la tabla **Attribute-Mapping** (Asignación y atributos), especifique las asignaciones necesarias. Por ejemplo,

   Nombre del atributo | Nombre de encabezado |
   |-------|--------|
   |upn | x-userprinciplename |
   |email   |    x-email  |
   |oid   | x-oid  |
   |scp   |     x-scope |
   |amr    |    x-amr    |

6. Seleccione **Guardar**.

#### <a name="step-4--create-a-site"></a>Paso 4: Creación de un sitio

>[!NOTE]
>En algunas configuraciones, es posible que un sitio contenga más de una aplicación. Un sitio se puede usar con más de una aplicación, si resulta adecuado.

Siga estos pasos para crear un sitio:

1. Vaya a **Principal** > **Sitios**.

2. Seleccione **Agregar sitio**.

3. Especifique un **Nombre** para el sitio.

4. Especifique el **Destino** del sitio. El destino es el par de nombre de host y puerto del servidor que hospeda la aplicación. No escriba la ruta de acceso de la aplicación en este campo. Por ejemplo, una aplicación en https://mysite:9999/AppName tiene un valor de destino de mysite: 9.999

5. Indique si el destino espera conexiones seguras.

6. Si el destino espera conexiones seguras, establezca el grupo Certificado de confianza en **Trust Any** (Confiar en cualquiera).

7. Seleccione **Guardar**.

#### <a name="step-5--create-an-application"></a>Paso 5: Creación de una aplicación

Siga estos pasos para crear una aplicación de PingAccess en cada aplicación de Azure que quiera proteger.

1. Vaya a **Principal** > **Aplicaciones**.

2. Seleccione **Agregar aplicación**.

3. Escriba un **Nombre** para la aplicación.

4. También puede escribir una **Descripción** para la aplicación.

5. Especifique la **Context Root** (Raíz contextual) de la aplicación. Por ejemplo, una aplicación en https://mysite:9999/AppName tiene una raíz contextual de /AppName. La raíz contextual debe comenzar con una barra diagonal (/), no debe terminar con otra (/) y puede tener más de una capa de profundidad, por ejemplo, /Apps/MyApp.

6. Seleccione el **Host virtual** que ha creado.

   >[!NOTE]
   >La combinación de host virtual y raíz contextual debe ser única en PingAccess.

7. Seleccione la **Sesión web** que ha creado.

8. Seleccione el **Sitio** que ha creado y que contiene la aplicación.

9. Seleccione la **Asignación de identidad** que ha creado.

10. Seleccione **Habilitado** para habilitar el sitio al guardar.

11. Seleccione **Guardar**.

### <a name="configure-the-pingfederate-authentication-policy"></a>Configuración de la directiva de autenticación de PingFederate

Configure la directiva de autenticación de PingFederate para federar en los distintos IdP proporcionados por los inquilinos de Azure AD B2C.

1. Cree un contrato para enlazar los atributos entre los IdP y el SP. Para obtener más información, vea [Centro de federación y contratos de directivas de autenticación](https://docs.pingidentity.com/bundle/pingfederate-101/page/ope1564002971971.html). Probablemente solo necesite un contrato, a menos que el SP requiera otro conjunto de atributos de cada IdP.

2. En cada IdP, cree una conexión de IdP entre el IdP y PingFederate, el centro de federación como SP.

3. En la ventana **Target Session Mapping** (Asignación de sesión de destino), agregue los contratos de directivas de autenticación aplicables a la conexión de IdP.

4. En la ventana **Selectores**, configure un selector de autenticación. Por ejemplo, vea una instancia del **Identifier First Adapter** (Primer adaptador del identificador) para asignar cada IdP a la conexión de IdP correspondiente en una directiva de autenticación.

5. Cree una conexión de SP entre PingFederate, el centro de federación como IdP y el SP.

6. Agregue el contrato de directiva de autenticación correspondiente a la conexión de SP en la ventana **Authentication Source Mapping** (Asignación de origen de autenticación).

7. Trabaje con cada IdP para conectarse a PingFederate, el centro de federación como SP.

8. Trabaje con el SP para conectarse a PingFederate, el centro de federación como IdP.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los artículos siguientes:

- [Directivas personalizadas de Azure AD B2C](./custom-policy-overview.md)

- [Introducción a las directivas personalizadas en Azure AD B2C](tutorial-create-user-flows.md?pivots=b2c-custom-policy)
