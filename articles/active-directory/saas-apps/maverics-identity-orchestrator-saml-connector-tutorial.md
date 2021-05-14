---
title: 'Tutorial: Integración del inicio de sesión único (SSO) de Azure Active Directory con Maverics Identity Orchestrator SAML Connector | Microsoft Docs'
description: Aprenda a configurar el inicio de sesión único entre Azure Active Directory y Maverics Identity Orchestrator SAML Connector.
services: active-directory
author: jeevansd
manager: CelesteDG
ms.reviewer: celested
ms.service: active-directory
ms.subservice: saas-app-tutorial
ms.workload: identity
ms.topic: tutorial
ms.date: 03/17/2021
ms.author: jeedes
ms.openlocfilehash: 402f6cd6961108cdf1e9c94fb4f93309fbf15ead
ms.sourcegitcommit: 950e98d5b3e9984b884673e59e0d2c9aaeabb5bb
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/18/2021
ms.locfileid: "107599033"
---
# <a name="integrate-azure-ad-single-sign-on-with-maverics-identity-orchestrator-saml-connector"></a>Integración del inicio de sesión único de Azure AD con Maverics Identity Orchestrator SAML Connector

Maverics Identity Orchestrator de Strata proporciona una manera sencilla de integrar aplicaciones locales con Azure Active Directory (Azure AD) para la autenticación y el control de acceso. Maverics Orchestrator es capaz de modernizar la autenticación y la autorización para las aplicaciones que se basan actualmente en encabezados, cookies y otros métodos de autenticación de propiedad. Las instancias de Maverics Orchestrator se pueden implementar de forma local o en la nube. 

En este tutorial de acceso híbrido se muestra cómo migrar una aplicación web local que está protegida por un producto de administración de acceso web heredado con el fin de usar Azure AD para la autenticación y el control de acceso. Estos son los pasos básicos:

1. Configuración de Maverics Orchestrator
1. Redirección de una aplicación mediante proxy
1. Registro de una aplicación en Azure AD
1. Autenticación mediante Azure y autorización del acceso a la aplicación
1. Adición de encabezados para el acceso a aplicaciones sin problemas
1. Trabajo con varias aplicaciones

## <a name="prerequisites"></a>Requisitos previos

* Una suscripción de Azure AD. Si no tiene una suscripción, puede crear una [cuenta gratuita](https://azure.microsoft.com/free/).
* Una suscripción con el inicio de sesión único (SSO) de Maverics Identity Orchestrator SAML Connector habilitado. Para conseguir el software de Maverics, póngase en contacto con el [equipo de ventas de Strata](mailto:sales@strata.io).
* Al menos una aplicación que usa la autenticación basada en encabezados. Los ejemplos funcionan en una aplicación denominada Connectulum hospedada en `https://app.connectulum.com`.
* Una máquina Linux para hospedar Maverics Orchestrator
  * Sistema operativo: RHEL 7.7 o posteriores, CentOS 7 o posteriores
  * Disco: >= 10 GB
  * Memoria: >= 4 GB
  * Puertos: 22 (SSH/SCP), 443, 7474
  * Acceso raíz para tareas administrativas o de instalación
  * Salida de la red desde el servidor que hospeda Maverics Identity Orchestrator a la aplicación protegida

## <a name="step-1-set-up-the-maverics-orchestrator"></a>Paso 1: Configuración de Maverics Orchestrator

### <a name="install-maverics"></a>Instale Maverics.

1. Obtenga el paquete RPM de Maverics más reciente. Copie el paquete en el sistema en el que quiera instalar el software de Maverics.

1. Instale el paquete de Maverics; para ello, sustituya el nombre de archivo `maverics.rpm` por el suyo.

   `sudo rpm -Uvf maverics.rpm`

   Después de instalar Maverics, se ejecutará como servicio en `systemd`. Use el siguiente comando para comprobar que el servicio se está ejecutando:

   `sudo systemctl status maverics`

1. Para reiniciar Orchestrator y seguir los registros, puede ejecutar el siguiente comando:

   `sudo service maverics restart; sudo journalctl --identifier=maverics -f`

Después de instalar Maverics, se crea el archivo predeterminado `maverics.yaml` en el directorio `/etc/maverics`. Antes de editar la configuración para incluir `appgateways` y `connectors`, el archivo de configuración tendrá el siguiente aspecto:

```yaml
# © Strata Identity Inc. 2020. All Rights Reserved. Patents Pending.

version: 0.1
listenAddress: ":7474"
```

### <a name="configure-dns"></a>Configurar el DNS

DNS será útil para que no tenga que recordar la dirección IP del servidor de Orchestrator.

Edite el archivo de hosts de la máquina del explorador (su portátil) mediante una IP de Orchestrator hipotética de 12.34.56.78. En los sistemas operativos basados en Linux, este archivo se encuentra en `/etc/hosts`. En Windows, se encuentra en `C:\windows\system32\drivers\etc`.

```
12.34.56.78 sonar.maverics.com
12.34.56.78 connectulum.maverics.com
```

Para confirmar que DNS está configurado según lo previsto, puede hacer una solicitud al punto de conexión de estado de Orchestrator. En el explorador, solicite http://sonar.maverics.com:7474/status.

### <a name="configure-tls"></a>Configuración de TLS

La comunicación a través de canales seguros para comunicarse con su instancia de Orchestrator es fundamental para mantener la seguridad. Puede agregar un par de certificado y clave en la sección `tls` para lograr esto.

Para generar un certificado autofirmado y una clave para el servidor de Orchestrator, ejecute el siguiente comando desde el directorio `/etc/maverics`:

`openssl req -new -newkey rsa:4096 -x509 -sha256 -days 365 -nodes -out maverics.crt -keyout maverics.key`

> [!NOTE]
> En los entornos de producción, es probable que desee usar un certificado firmado por una entidad de certificación conocida para evitar advertencias en el explorador. [Let's Encrypt](https://letsencrypt.org/) es una estupenda opción y gratuita si está buscando una entidad de certificación de confianza.

Ahora, use el certificado y la clave recién generados para Orchestrator. El archivo de configuración debe contener ahora este código:

```yaml
version: 0.1
listenAddress: ":443"

tls:
  maverics:
    certFile: /etc/maverics/maverics.crt
    keyFile: /etc/maverics/maverics.key
```

Para confirmar que TLS está configurado según lo esperado, reinicie el servicio Maverics y realice una solicitud al punto de conexión de estado.

## <a name="step-2-proxy-an-application"></a>Paso 2: Redirección de una aplicación mediante proxy

A continuación, configure la redirección mediante proxy básica en la instancia de Orchestrator mediante `appgateways`. Este paso le ayuda a validar que Orchestrator tenga la conectividad necesaria a la aplicación protegida.

El archivo de configuración debe contener ahora este código:

```yaml
version: 0.1
listenAddress: ":443"

tls:
  maverics:
    certFile: /etc/maverics/maverics.crt
    keyFile: /etc/maverics/maverics.key

appgateways:
  - name: sonar
    location: /
    # Replace https://app.sonarsystems.com with the address of your protected application
    upstream: https://app.sonarsystems.com
```

Para confirmar que la redirección mediante proxy funciona según lo esperado, reinicie el servicio Maverics y realice una solicitud a la aplicación mediante el proxy de Maverics. Opcionalmente, puede realizar una solicitud a recursos de aplicación específicos

## <a name="step-3-register-an-enterprise-application-in-azure-ad"></a>Paso 3: Registro de una aplicación en Azure AD

Ahora, cree una nueva aplicación empresarial en Azure AD que se usará para autenticar a los usuarios finales.

> [!NOTE]
> Cuando se usan características de Azure AD como el acceso condicional, es importante crear una aplicación empresarial por aplicación local. Esto permite el acceso condicional por aplicación, la evaluación de riesgos por aplicación, los permisos asignados por aplicación, etc. Por lo general, una aplicación empresarial en Azure AD se asigna a un conector de Azure en Maverics.

Para registrar una aplicación empresarial en Azure AD:

1. En el inquilino de Azure AD, vaya a **Aplicaciones empresariales** y seleccione **Nueva aplicación**. En la galería de Azure AD, busque **Maverics Identity Orchestrator SAML Connector** y selecciónelo.

1. En el panel **Propiedades** de Maverics Identity Orchestrator SAML Connector, establezca **¿Asignación de usuarios?** en **No** para permitir que la aplicación funcione para todos los usuarios del directorio.

1. En el panel **Información general** de Maverics Identity Orchestrator SAML Connector, seleccione **Configurar inicio de sesión único** y, a continuación, seleccione **SAML**.

1. En el **Inicio de sesión basado en SAML** de Maverics Identity Orchestrator SAML Connector, edite **Configuración básica de SAML**; para ello, seleccione el botón **Editar** (icono de lápiz).

   ![Captura de pantalla del botón Editar de la "Configuración básica de SAML".](common/edit-urls.png)

1. Escriba un **identificador de entidad** de `https://sonar.maverics.com`. El identificador de entidad debe ser único en todas las aplicaciones del inquilino y puede ser un valor arbitrario. En la sección siguiente, usará este valor al definir el campo `samlEntityID` para el conector de Azure.

1. Escriba una **URL de respuesta** de `https://sonar.maverics.com/acs`. En la sección siguiente, usará este valor al definir el campo `samlConsumerServiceURL` para el conector de Azure.

1. Escriba una **dirección URL de inicio de sesión** de `https://sonar.maverics.com/`. Este campo no lo utilizará Maverics, pero es necesario en Azure AD para permitir que los usuarios accedan a la aplicación mediante el portal Mis aplicaciones de Azure AD.

1. Seleccione **Guardar**.

1. En la sección **Certificado de firma de SAML**, seleccione el botón **Copiar** para copiar el valor de **Dirección URL de metadatos de federación de aplicación** y guárdelo en su equipo.

   ![Captura de pantalla del botón Copiar de "Certificado de firma de SAML".](common/copy-metadataurl.png)

## <a name="step-4-authenticate-via-azure-and-authorize-access-to-the-application"></a>Paso 4: Autenticación mediante Azure y autorización del acceso a la aplicación

A continuación, ponga en uso la aplicación empresarial que acaba de crear mediante la configuración del conector de Azure en Maverics. Esta configuración de `connectors` emparejada con el bloque `idps` permite a Orchestrator autenticar a los usuarios.

El archivo de configuración debe contener ahora el siguiente código: Asegúrese de reemplazar `METADATA_URL` por el valor de dirección URL de metadatos de federación de aplicación del paso anterior.

```yaml
version: 0.1
listenAddress: ":443"

tls:
  maverics:
    certFile: /etc/maverics/maverics.crt
    keyFile: /etc/maverics/maverics.key

idps:
  - name: azureSonarApp

appgateways:
  - name: sonar
    location: /
    # Replace https://app.sonarsystems.com with the address of your protected application
    upstream: https://app.sonarsystems.com

    policies:
      - resource: /
        allowIf:
          - equal: ["{{azureSonarApp.authenticated}}", "true"]

connectors:
  - name: azureSonarApp
    type: azure
    authType: saml
    # Replace METADATA_URL with the App Federation Metadata URL
    samlMetadataURL: METADATA_URL
    samlConsumerServiceURL: https://sonar.maverics.com/acs
    samlEntityID: https://sonar.maverics.com
```

Para confirmar que la autenticación funciona según lo esperado, reinicie el servicio Maverics y realice una solicitud a un recurso de aplicación mediante el proxy de Maverics. Debe redirigirse a Azure para la autenticación antes de obtener acceso al recurso.

## <a name="step-5-add-headers-for-seamless-application-access"></a>Paso 5: Adición de encabezados para el acceso a aplicaciones sin problemas

Todavía no envía encabezados a la aplicación de nivel superior. Vamos a agregar `headers` a la solicitud a medida que pasa por el proxy de Maverics para permitir que la aplicación de nivel superior identifique al usuario.

El archivo de configuración debe contener ahora este código:

```yaml
version: 0.1
listenAddress: ":443"

tls:
  maverics:
    certFile: /etc/maverics/maverics.crt
    keyFile: /etc/maverics/maverics.key

idps:
  - name: azureSonarApp

appgateways:
  - name: sonar
    location: /
    # Replace https://app.sonarsystems.com with the address of your protected application
    upstream: https://app.sonarsystems.com

    policies:
      - resource: /
        allowIf:
          - equal: ["{{azureSonarApp.authenticated}}", "true"]

    headers:
      email: azureSonarApp.name
      firstname: azureSonarApp.givenname
      lastname: azureSonarApp.surname

connectors:
  - name: azureSonarApp
    type: azure
    authType: saml
    # Replace METADATA_URL with the App Federation Metadata URL
    samlMetadataURL: METADATA_URL
    samlConsumerServiceURL: https://sonar.maverics.com/acs
    samlEntityID: https://sonar.maverics.com
```

Para confirmar que la autenticación funciona según lo esperado, realice una solicitud a un recurso de aplicación mediante el proxy de Maverics. Ahora la aplicación protegida debe recibir encabezados en la solicitud. 

No dude en editar las claves de encabezado si la aplicación espera encabezados diferentes. Todas las notificaciones que se devuelven de Azure AD como parte del flujo SAML están disponibles para su uso en encabezados. Por ejemplo, puede incluir otro encabezado de `secondary_email: azureSonarApp.email`, donde `azureSonarApp` es el nombre del conector y `email` es una notificación devuelta desde Azure AD. 

## <a name="step-6-work-with-multiple-applications"></a>Paso 6: Trabajo con varias aplicaciones

Ahora, echemos un vistazo a lo que se necesita para redirigir mediante proxy en varias aplicaciones que se encuentran en hosts diferentes. Para conseguirlo, configure otra puerta de enlace de aplicaciones, otra aplicación empresarial en Azure AD y otro conector.

El archivo de configuración debe contener ahora este código:

```yaml
version: 0.1
listenAddress: ":443"

tls:
  maverics:
    certFile: /etc/maverics/maverics.crt
    keyFile: /etc/maverics/maverics.key

idps:
  - name: azureSonarApp
  - name: azureConnectulumApp

appgateways:
  - name: sonar
    host: sonar.maverics.com
    location: /
    # Replace https://app.sonarsystems.com with the address of your protected application
    upstream: https://app.sonarsystems.com

    policies:
      - resource: /
        allowIf:
          - equal: ["{{azureSonarApp.authenticated}}", "true"]

    headers:
      email: azureSonarApp.name
      firstname: azureSonarApp.givenname
      lastname: azureSonarApp.surname

  - name: connectulum
    host: connectulum.maverics.com
    location: /
    # Replace https://app.connectulum.com with the address of your protected application
    upstream: https://app.connectulum.com

    policies:
      - resource: /
        allowIf:
          - equal: ["{{azureConnectulumApp.authenticated}}", "true"]

    headers:
      email: azureConnectulumApp.name
      firstname: azureConnectulumApp.givenname
      lastname: azureConnectulumApp.surname

connectors:
  - name: azureSonarApp
    type: azure
    authType: saml
    # Replace METADATA_URL with the App Federation Metadata URL
    samlMetadataURL: METADATA_URL
    samlConsumerServiceURL: https://sonar.maverics.com/acs
    samlEntityID: https://sonar.maverics.com

  - name: azureConnectulumApp
    type: azure
    authType: saml
    # Replace METADATA_URL with the App Federation Metadata URL
    samlMetadataURL: METADATA_URL
    samlConsumerServiceURL: https://connectulum.maverics.com/acs
    samlEntityID: https://connectulum.maverics.com
```

Es posible que haya observado que el código agrega un campo `host` a sus definiciones de puerta de enlace de aplicaciones. El campo `host` permite a Maverics Orchestrator distinguir el host de nivel superior al que se va a redirigir el tráfico mediante proxy.

Para confirmar que la puerta de enlace de aplicaciones recién agregada funciona según lo previsto, realice una solicitud a `https://connectulum.maverics.com`.

## <a name="advanced-scenarios"></a>Escenarios avanzados

### <a name="identity-migration"></a>Migración de identidad

¿No soporta que su herramienta de administración de acceso web que ha llegado al final de su vida útil, pero no tiene una forma de migrar a sus usuarios sin que se produzcan restablecimientos masivos de contraseñas? Maverics Orchestrator admite la migración de identidades mediante `migrationgateways`.

### <a name="web-server-gateways"></a>Puertas de enlace de servidor web

¿No desea reutilizar el tráfico de la red y del proxy mediante Maverics Orchestrator? No pasa nada. Maverics Orchestrator se puede emparejar con puertas de enlace de servidor web (módulos) para ofrecer las mismas soluciones sin proxy.

## <a name="wrap-up"></a>Encapsulado

En este punto, ha instalado Maverics Orchestrator, ha creado y configurado una aplicación empresarial en Azure AD y ha configurado Orchestrator como proxy para una aplicación protegida, al tiempo que requiere autenticación y aplica la directiva. Para más información sobre cómo se puede usar Maverics Orchestrator para casos de uso de la administración de identidades distribuida, [póngase en contacto con Strata](mailto:sales@strata.io).

## <a name="next-steps"></a>Pasos siguientes

- [¿Qué es el acceso a aplicaciones y el inicio de sesión único con Azure Active Directory?](../manage-apps/what-is-single-sign-on.md)
- [¿Qué es el acceso condicional en Azure Active Directory?](../conditional-access/overview.md)
