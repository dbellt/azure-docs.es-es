---
title: Tutorial para configurar Azure Active Directory B2C con Strata
titleSuffix: Azure AD B2C
description: Aprenda a integrar la autenticación de Azure AD B2C con WhoIAM para la verificación del usuario.
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 10/25/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 49def79c467aa8f2b3fda9688dc77fcb003a34a5
ms.sourcegitcommit: 20f8bf22d621a34df5374ddf0cd324d3a762d46d
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/09/2021
ms.locfileid: "107256572"
---
# <a name="tutorial-for-extending-azure-ad-b2c-to-protect-on-premises-applications-using-strata"></a>Tutorial para ampliar Azure AD B2C para proteger Strata

En este tutorial de ejemplo, obtendrá información para integrar Azure Active Directory (AD) B2C con [Maverics Identity Orchestrator](https://www.strata.io/maverics-identity-orchestrator/) de Strata.
Maverics Identity Orchestrator amplía Azure AD B2C para proteger las aplicaciones locales. Se conecta a cualquier sistema de identidades, migra de manera transparente usuarios y credenciales, sincroniza directivas y configuraciones y abstrae la autenticación y la administración de sesiones. Con Strata Enterprises puede pasar rápidamente de un sistema heredado a Azure AD B2C sin necesidad de volver a escribir las aplicaciones. La solución tiene las siguientes ventajas:

- **Inicio de sesión único (SSO) del cliente en las aplicaciones híbridas locales**: Azure AD B2C admite el inicio de sesión único de cliente con Maverics Identity Orchestrator. Los usuarios inician sesión con sus cuentas hospedadas en Azure AD B2C o en el proveedor de identidades de redes sociales (IdP). Maverics extiende el inicio de sesión único a las aplicaciones que históricamente se han protegido mediante sistemas de identidad heredados, como Symantec SiteMinder.

- **Ampliar el inicio de sesión único basado en estándares a las aplicaciones sin rescribirlas**: use Azure AD B2C para administrar el acceso de usuario y habilitar el inicio de sesión único con los conectores SAML o OIDC de Maverics Identity Orchestrator.

- **Configuración sencillas**: Azure AD B2C proporciona una sencilla interfaz de usuario paso a paso para conectar los conectores SAML o OIDC de Maverics Identity Orchestrator a Azure AD B2C.

## <a name="prerequisites"></a>Requisitos previos

Para empezar, necesitará lo siguiente:

- Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).

- Un [inquilino de Azure AD B2C](./tutorial-create-tenant.md) vinculado a la suscripción de Azure.

- Una instancia de [Azure Key Vault](https://azure.microsoft.com/services/key-vault/) para almacenar secretos que utiliza Maverics Identity Orchestrator. Se usa para conectarse a Azure AD B2C u otros proveedores de atributos, como un directorio o una base de datos de protocolo ligero de acceso a directorios (LDAP).

- Una instancia de [Maverics Identity Orchestrator](https://www.strata.io/maverics-identity-orchestrator/) que está instalada y en ejecución en una máquina virtual de Azure o en el servidor local de su elección. Para más información sobre cómo obtener el software y el acceso a la documentación de instalación y configuración, póngase en contacto con [Strata](https://www.strata.io/contact/)

- Una aplicación local que pasará de un sistema de identidades heredado a Azure AD B2C.

## <a name="scenario-description"></a>Descripción del escenario

La integración Maverics de Strata incluye los siguientes componentes:

- **Azure AD B2C**: el servidor de autorización responsable de verificar las credenciales del usuario. Los usuarios autenticados pueden acceder a aplicaciones locales mediante una cuenta local almacenada en el directorio de Azure AD B2C.

- **Un proveedor de identidades empresarial o de redes sociales externo**: podría ser cualquier proveedor de OpenID Connect, Facebook, Google o GitHub. Consulte la información sobre el uso de [proveedores de identidades externos](./technical-overview.md#external-identity-providers) con Azure AD B2C.  

- **Maverics Identity Orchestrator de Strata**: el servicio que organiza el inicio de sesión del usuario y pasa de manera transparente la identidad a las aplicaciones a través de encabezados HTTP.

En el siguiente diagrama de arquitectura se muestra la implementación.

![Imagen que muestra la arquitectura de una integración de Azure AD B2C con Strata Maverics para permitir el acceso a aplicaciones híbridas](./media/partner-strata/strata-architecture-diagram.png)

| Pasos | Descripción |
|:-------|:---------------|
| 1. | El usuario realiza una solicitud para acceder a la aplicación hospedada en el entorno local. Maverics Identity Orchestrator redirige mediante proxy la solicitud realizada por el usuario a la aplicación.|
| 2. | Orchestrator comprueba el estado de autenticación del usuario. Si no recibe un token de sesión, o el token de sesión proporcionado no es válido, envía al usuario a Azure AD B2C para autenticarse.|
| 3. | Azure AD B2C envía la solicitud de autenticación al proveedor de identidades de redes sociales configuradas.|
| 4. | El proveedor de identidades desafía al usuario para pedir sus credenciales. Según el proveedor de identidades, el usuario puede requerir la autenticación multifactor (MFA).|
| 5. | El proveedor de identidades devuelve la respuesta de autenticación a Azure AD B2C. Opcionalmente, el usuario puede crear una cuenta local en el directorio de Azure AD B2C durante este paso.|
| 6. | Azure AD B2C envía la solicitud del usuario al punto de conexión especificado durante el registro de la aplicación Orchestrator en el inquilino de Azure AD B2C.|
| 7. | Orchestrator evalúa las directivas de acceso y calcula los valores de atributo que se van a incluir en los encabezados HTTP reenviados a la aplicación. Durante este paso, Orchestrator puede llamar a proveedores de atributos adicionales para recuperar la información necesaria para establecer los valores de encabezado correctamente. Orchestrator establece los valores de encabezado y envía la solicitud a la aplicación.|
| 8. | El usuario ya está autenticado y tiene acceso a la aplicación.|

## <a name="get-maverics-identity-orchestrator"></a>Obtención de Maverics Identity Orchestrator
Para obtener el software que usará para integrar su aplicación local heredada con Azure AD B2C, póngase en contacto con [Strata](https://www.strata.io/contact/). Después de obtener el software, siga los pasos que se indican a continuación para determinar los requisitos previos específicos de Orchestrator y realizar los pasos de instalación y configuración necesarios.

## <a name="configure-your-azure-ad-b2c-tenant"></a>Configuración del inquilino de Azure AD B2C

1. **Registrar su aplicación**

   a. [Registre Orchestrator como una aplicación](./tutorial-register-applications.md?tabs=app-reg-ga) en el inquilino de Azure AD B2C.
   >[!Note]
   >Necesitará el nombre e id. del inquilino, el id. de cliente, el secreto de cliente, las notificaciones configuradas y el URI de redirección más adelante, cuando configure la instancia de Orchestrator.

   b. Conceda a Microsoft MS Graph API los permisos para sus aplicaciones. La aplicación necesitará los siguientes permisos: `offline_access` y `openid`.

   c. Agregue el URI de redirección para la aplicación. Este URI coincidirá con el parámetro `oauthRedirectURL` de la configuración del conector de Azure AD B2C de Orchestrator; por ejemplo, `https://example.com/oidc-endpoint`.

2. **Crear un flujo de usuario**: Cree un [flujo de usuario de registro e inicio de sesión](./tutorial-create-user-flows.md).

3. **Agregar un proveedor de identidades**: elija el inicio de sesión de su usuario con una cuenta local o un [proveedor de identidades](./add-identity-provider.md) empresarial o de redes sociales.

4. **Definir atributos de usuario**: defina los atributos que se recopilarán durante el registro.

5. **Especificar las notificaciones de aplicación**: especifique los atributos que se devolverán a la aplicación a través de la instancia de Orchestrator. Orchestrator consume atributos de las notificaciones devueltas por Azure AD B2C y puede recuperar atributos adicionales de otros sistemas de identidades conectados, como los directorios y las bases de datos LDAP. Esos atributos se establecen en encabezados HTTP y se envían a la aplicación local de nivel superior.

## <a name="configure-maverics-identity-orchestrator"></a>Configuración de Maverics Identity Orchestrator

En las secciones siguientes, le guiaremos a través de los pasos necesarios para configurar la instancia de Orchestrator. Para obtener soporte técnico y documentación adicionales, póngase en contacto con [Strata](https://www.strata.io/contact/).

### <a name="maverics-identity-orchestrator-server-requirements"></a>Requisitos del servidor de Maverics Identity Orchestrator

Puede ejecutar la instancia de Orchestrator en cualquier servidor, ya sea de manera local o en una infraestructura de nube pública de un proveedor como Azure, AWS o GCP.

- Sistema operativo: REHL 7.7 o posterior, CentOS 7 o posterior

- Disco: 10 GB (pequeño)

- Memoria: 16 GB

- Puertos: 22 (SSH/SCP), 443, 80

- Acceso raíz para tareas administrativas o de instalación

- Maverics Identity Orchestrator se ejecuta como el usuario `maverics` en `systemd`

- Salida de red del servidor que hospeda a Maverics Identity Orchestrator con la capacidad de comunicarse con el inquilino de Azure AD.

### <a name="install-maverics-identity-orchestrator"></a>Instalación de Maverics Identity Orchestrator

1. Obtenga el paquete RPM de Maverics más reciente. Coloque el paquete en el sistema en el que quiera instalar Maverics. Si va a copiar el archivo en un host remoto, [SCP](https://www.ssh.com/ssh/scp/) es una herramienta útil.

2. Para instalar el paquete de Maverics, ejecute el siguiente comando y reemplace `maverics.rpm` por el nombre de su archivo.

   `sudo rpm -Uvf maverics.rpm`

   De forma predeterminada, Maverics se instala en el directorio `/usr/local/bin`.

3. Después de instalar Maverics, se ejecutará como servicio en `systemd`.  Utilice el siguiente comando para comprobar que el servicio de Maverics se está ejecutando:

   `sudo service maverics status`

  Si la instalación de Orchestrator se realizó correctamente, debería mostrarse un mensaje similar al siguiente:

```
Redirecting to /bin/systemctl status maverics.service
  maverics.service - Maverics
  Loaded: loaded (/etc/systemd/system/maverics.service; enabled; vendor preset: disabled)
  Active: active (running) since Thu 2020-08-13 16:48:01 UTC; 24h ago
  Main PID: 330772 (maverics)
  Tasks: 5 (limit: 11389)
  Memory: 14.0M
  CGroup: /system.slice/maverics.service
          └─330772 /usr/local/bin/maverics --config /etc/maverics/maverics.yaml
  ```

4. Si el servicio de Maverics no se inicia, ejecute el siguiente comando para investigar el problema:

   `journalctl --unit=maverics.service --reverse`

   La entrada de registro más reciente aparecerá al principio de la salida.

Después de instalar Maverics, se crea el archivo `maverics.yaml` predeterminado en el directorio `/etc/maverics`.

Configure Orchestrator para proteger la aplicación. Intégrese con Azure AD B2C, almacene y recupere secretos de [Azure Key Vault](https://azure.microsoft.com/services/key-vault/?OCID=AID2100131_SEM_bf7bdd52c7b91367064882c1ce4d83a9:G:s&ef_id=bf7bdd52c7b91367064882c1ce4d83a9:G:s&msclkid=bf7bdd52c7b91367064882c1ce4d83a9). Defina la ubicación desde la que Orchestrator debe leer su configuración.

### <a name="supply-configuration-using-environment-variables"></a>Especificación de configuraciones mediante variables de entorno

Proporcione la configuración a las instancias de Orchestrator mediante variables de entorno.

`MAVERICS_CONFIG`

Esta variable de entorno indica a la instancia de Orchestrator qué archivos de configuración YAML debe usar y dónde encontrarlos durante el inicio o el reinicio. Establezca la variable de entorno en `/etc/maverics/maverics.env`.

### <a name="create-the-orchestrators-tls-configuration"></a>Creación de la configuración de TLS de Orchestrator

En el campo `tls` del archivo `maverics.yaml` se declaran las configuraciones de seguridad de la capa de transporte que utilizará la instancia de Orchestrator. Los conectores pueden usar los objetos TLS y el servidor de Orchestrator.

La clave `maverics` está reservada para el servidor de Orchestrator. Todas las demás claves están disponibles y se pueden usar para insertar un objeto TLS en un conector determinado.

```yaml
tls:
  maverics:
    certFile: /etc/maverics/maverics.cert
    keyFile: /etc/maverics/maverics.key
```

### <a name="configure-the-azure-ad-b2c-connector"></a>Configuración del conector de Azure AD B2C

Las instancias de Orchestrator utilizan conectores para integrarse con proveedores de atributos y autenticación. En este caso, esta instancia de App Gateway para Orchestrator usa el conector de Azure AD B2C como proveedor de atributos y autenticación. Azure AD B2C usa el proveedor de identidades de redes sociales para la autenticación y, a continuación, actúa como proveedor de atributos para la instancia de Orchestrator, pasando atributos en las notificaciones configuradas en encabezados HTTP.  

La configuración de este conector corresponde a la aplicación registrada en el inquilino de Azure AD B2C.

1. Copie el id. de cliente, el secreto y el URI de redirección de la configuración de la aplicación en el inquilino.

2. Asigne un nombre al conector, que se muestra aquí como `azureADB2C`, y establezca el `type` del conector en `azure`. Tome nota del nombre del conector, ya que este valor se utiliza en otros parámetros de configuración más adelante.

3. En esta integración, el valor de `authType` debe establecerse en `oidc`.

4. Establezca el id. de cliente que copió en el paso 1 como el valor del parámetro `oauthClientID`.

5. Establezca el secreto de cliente que copió en el paso 1 como el valor del parámetro `oauthClientSecret`.

6. Establezca el URI de redirección que copió en el paso 1 como el valor del parámetro `oauthRedirectURL`.

7. El conector de OIDC de Azure AD B2C usa el punto de conexión de OIDC conocido para detectar metadatos, incluidas las direcciones URL y las claves de firma. Establezca el valor de `oidcWellKnownURL` en el punto de conexión del inquilino.

```yaml
connectors:
  name: azureADB2C
  type: azure
  oidcWellKnownURL: https://<tenant name>.b2clogin.com/<tenant name>.onmicrosoft.com/B2C_1_login/v2.0/.well-known/openid-configuration
  oauthRedirectURL: https://example.com/oidc-endpoint
  oauthClientID: <azureADB2CClientID>
  oauthClientSecret: <azureADB2CClientSecret>
  authType: oidc
```

### <a name="define-azure-ad-b2c-as-your-authentication-provider"></a>Definición de Azure AD B2C como proveedor de autenticación

Un proveedor de autenticación determina cómo realizar la autenticación para un usuario que no ha presentado ninguna sesión válida como parte de la solicitud de recursos de la aplicación. La configuración de su inquilino de Azure AD B2C determina cómo desafiar a un usuario para obtener las credenciales y aplicar las directivas de autenticación adicionales. Por ejemplo, para que sea obligatoria una segunda fase para completar el proceso de autenticación y decidir qué notificaciones se deben devolver a la instancia de App Gateway de Orchestrator después de que la autenticación se realice correctamente.

El valor de `authProvider` debe coincidir con el valor de `name` del conector.

```yaml
authProvider: azureADB2C
```

### <a name="protect-your-on-premises-app-with-an-orchestrator-app-gateway"></a>Protección de la aplicación local con una instancia de App Gateway de Orchestrator

La configuración de App Gateway de Orchestrator declara cómo Azure AD B2C debe proteger la aplicación y cómo los usuarios deben acceder a la aplicación.

1. Cree un nombre para la instancia de App Gateway. Puede usar un nombre descriptivo o un nombre de host completo como identificador de la aplicación.

2. Establezca `location`. En el ejemplo siguiente se usa la raíz `/` de la aplicación; sin embargo, puede ser cualquier ruta de acceso URL de la aplicación.

3. Defina la aplicación protegida en `upstream` mediante la convención host:puerto: `https://example.com:8080`.

4. Establezca los valores para las páginas de error y no autorizadas.

5. Defina los nombres de los encabezados HTTP y los valores de atributo que se deben proporcionar a la aplicación para establecer la autenticación y controlar el acceso a la aplicación. Los nombres de los encabezados son arbitrarios y normalmente se corresponden con la configuración de la aplicación. El conector que proporciona los valores de atributo los convierte en espacios de nombres. En el ejemplo siguiente, los valores devueltos desde Azure AD B2C tienen como prefijo el nombre de conector `azureADB2C`; en él, el sufijo es el nombre del atributo que contiene el valor necesario, por ejemplo `given_name`.

6. Establezca las directivas que se van a evaluar y aplicar. Hay tres acciones definidas: `allowUnauthenticated`, `allowAnyAuthenticated` y `allowIfAny`. Cada acción está asociada a un `resource` y la directiva se evalúa para ese `resource`.

>[!NOTE]
>Tanto los `headers` como las `policies` utilizan las extensiones de servicio de JavaScript o GoLang para implementar una lógica arbitraria que mejora considerablemente las capacidades predeterminadas.

```yaml
appgateways:
  - name: Sonar
    location: /
    upstream: https://example.com:8080
    errorPage: https://example.com:8080/sonar/error
    unauthorizedPage: https://example.com:8080/sonar/accessdenied

    headers:
      SM_USER: azureADB2C.sub
      firstname: azureADB2C.given_name
      lastname: azureADB2C.family_name

    policies:
      - resource: ~ \.(jpg|png|ico|svg)
        allowUnauthenticated: true
      - resource: /
        allowAnyAuthenticated: true
      - resource: /sonar/daily_deals
        allowIfAny:
          azureADB2C.customAttribute: Rewards Member
```

### <a name="use-azure-key-vault-as-your-secrets-provider"></a>Uso de Azure Key Vault como proveedor de secretos

Es importante proteger los secretos que utiliza Orchestrator para conectarse a Azure AD B2C y a cualquier otro sistema de identidades. De manera predeterminada, Maverics cargará secretos en texto sin formato a partir del archivo `maverics.yaml`; sin embargo, en este tutorial, usará Azure Key Vault como proveedor de secretos.

Siga las instrucciones para [crear una nueva instancia de Key Vault](../key-vault/secrets/quick-create-portal.md) que la instancia de Orchestrator usará como proveedor de secretos. Agregue sus secretos al almacén y tome nota del `SECRET NAME` asignado a cada secreto. Por ejemplo, `AzureADB2CClientSecret`.

Para declarar un valor como secreto en un archivo de configuración `maverics.yaml`, encapsule el secreto entre corchetes angulares:

```yaml
connectors:
  - name: AzureADB2C
    type: azure
    oauthClientID: <AzureADB2CClientID>
    oauthClientSecret: <AzureADB2CClientSecret>
```

El valor especificado dentro de los corchetes angulares debe corresponder al valor de `SECRET NAME` asignado al secreto en la instancia de Azure Key Vault.

Para cargar secretos de Azure Key Vault, establezca la variable de entorno `MAVERICS_SECRET_PROVIDER` en el archivo `/etc/maverics/maverics.env`, con las credenciales que se encuentran en el archivo azure-credentials.json con el siguiente patrón:

`MAVERICS_SECRET_PROVIDER='azurekeyvault://<KEYVAULT NAME>.vault.azure.net?clientID=<APPID>&clientSecret=<PASSWORD>&tenantID=<TENANT>'`

### <a name="put-everything-together"></a>Todo junto

Este es el aspecto que tendrá la configuración de Orchestrator al completar las configuraciones descritas anteriormente.

```yaml
version: 0.4.2
listenAddress: ":443"
tls:
  maverics:
    certFile: certs/maverics.crt
    keyFile: certs/maverics.key

authProvider: azureADB2C

connectors:
  - name: azureADB2C
    type: azure
    oidcWellKnownURL: https://<tenant name>.b2clogin.com/<tenant name>.onmicrosoft.com/B2C_1_login/v2.0/.well-known/openid-configuration
    oauthRedirectURL: https://example.com/oidc-endpoint
    oauthClientID: <azureADB2CClientID>
    oauthClientSecret: <azureADB2CClientSecret>
    authType: oidc

appgateways:
  - name: Sonar
    location: /
    upstream: http://example.com:8080
    errorPage: http://example.com:8080/sonar/accessdenied
    unauthorizedPage: http://example.com:8080/sonar/accessdenied

    headers:
      SM_USER: azureADB2C.sub
      firstname: azureADB2C.given_name
      lastname: azureADB2C.family_name

    policies:
      - resource: ~ \.(jpg|png|ico|svg)
        allowUnauthenticated: true
      - resource: /
        allowAnyAuthenticated: true
      - resource: /sonar/daily_deals
        allowIfAny:
          azureADB2C.customAttribute: Rewards Member
```

## <a name="test-the-flow"></a>Prueba del flujo

1. Vaya a la URL de la aplicación local, `https://example.com/sonar/dashboard`.

2. Orchestrator debe redirigirse a la página que configuró en el flujo de usuario.

3. Seleccione el proveedor de identidades en la lista de la página.

4. Una vez que se le redirija al proveedor de identidades, especifique sus credenciales según se solicite, incluido un token de MFA si el proveedor de identidades lo requiere.

5. Después de autenticarse correctamente, debe redirigirse a Azure AD B2C, que reenvía la solicitud de aplicación al URI de redireccionamiento de Orchestrator.

6. Orchestrator evalúa las directivas, calcula los encabezados y envía el usuario a la aplicación de nivel superior.  

7. Debería ver la aplicación solicitada.

## <a name="next-steps"></a>Pasos siguientes

Para más información, consulte los artículos siguientes:

- [Directivas personalizadas de Azure AD B2C](./custom-policy-overview.md)

- [Introducción a las directivas personalizadas en Azure AD B2C](tutorial-create-user-flows.md?pivots=b2c-custom-policy)
