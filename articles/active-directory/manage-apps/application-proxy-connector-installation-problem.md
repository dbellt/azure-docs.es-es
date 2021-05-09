---
title: Problema al instalar el conector de agente del proxy de aplicación
description: Cómo solucionar los problemas que se produzcan al instalar el conector del agente de Application Proxy para Azure Active Directory.
services: active-directory
author: kenwith
manager: daveba
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: troubleshooting
ms.date: 01/28/2021
ms.author: kenwith
ms.reviewer: japere
ms.openlocfilehash: 0f76f03883746b6f4b87bb817f8adde850ed28b3
ms.sourcegitcommit: 4a54c268400b4158b78bb1d37235b79409cb5816
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108169469"
---
# <a name="problem-installing-the-application-proxy-agent-connector"></a>Problema al instalar el conector de agente del proxy de aplicación

El conector de Application Proxy de Microsoft Azure Active Directory es un componente de dominio interno que utiliza conexiones salientes para establecer la conectividad desde el punto de conexión disponible en la nube hacia el dominio interno.

## <a name="general-problem-areas-with-connector-installation"></a>Áreas problemáticas generales con la instalación del conector

Cuando se produce un error en la instalación de un conector, la causa principal suele pertenecer a una de las áreas siguientes. **Como precursor de cualquier solución de problemas, asegúrese de reiniciar el conector.**

1.  **Conectividad**: para completar una instalación correcta, el nuevo conector necesita registrarse y establecer propiedades de confianza futuras. Para ello, se conecta al servicio en la nube de Application Proxy de Azure Active Directory.

2.  **Establecimiento de confianza**: el nuevo conector crea un certificado autofirmado y se registra en el servicio en la nube.

3.  **Autenticación del administrador**: durante la instalación, el usuario debe proporcionar credenciales de administrador para completar la instalación del conector.

> [!NOTE]
> Los registros de instalación del conector pueden encontrarse en la carpeta% TEMP%, y pueden ayudar a proporcionar información adicional sobre lo que provoca un error de instalación.

## <a name="verify-connectivity-to-the-cloud-application-proxy-service-and-microsoft-login-page"></a>Comprobación de la conectividad con el servicio de proxy de aplicación en la nube y la página de inicio de sesión de Microsoft

**Objetivo:** compruebe que la máquina del conector se pueda conectar al punto de conexión de registro de Application Proxy, así como a la página de inicio de sesión de Microsoft.

1.  En el servidor del conector, ejecute una prueba de puertos con [telnet](/windows-server/administration/windows-commands/telnet) u otra herramienta para este fin y compruebe si los puertos 443 y 80 están abiertos.

2.  Si alguno de estos puertos no es correcto, compruebe que el firewall o el proxy de back-end tengan acceso a los dominios y puertos necesarios; consulte [Preparación del entorno local](application-proxy-add-on-premises-application.md#prepare-your-on-premises-environment).

3.  Abra un explorador (en otra pestaña), vaya a la siguiente página web: `https://login.microsoftonline.com` y asegúrese de que pueda iniciar sesión en esa página.

## <a name="verify-machine-and-backend-components-support-for-application-proxy-trust-certificate"></a>Comprobación de que los componentes de máquina y back-end admiten el certificado de confianza del proxy de aplicación

**Objetivo:** compruebe que la máquina del conector, el proxy de back-end y el firewall admitan el certificado creado por el conector para la confianza futura y que el certificado sea válido.

>[!NOTE]
>El conector intenta crear un certificado SHA512 que sea compatible con TLS 1.2. Si la máquina o el firewall y el proxy de back-end no admiten TLS1.2, la instalación producirá un error.
>
>

**Revise los requisitos previos necesarios:**

1.  Compruebe que la máquina sea compatible con TLS 1.2: todas las versiones de Windows posteriores a 2012 R2 deberían admitir TLS 1.2. Si la máquina del conector es de la versión 2012 R2 o una anterior, asegúrese de que los siguientes KB estén instalados en la máquina: <https://support.microsoft.com/help/2973337/sha512-is-disabled-in-windows-when-you-use-tls-1.2>

2.  Póngase en contacto con el administrador de la red y pídale que compruebe que el proxy y el firewall de back-end no bloqueen SHA512 para el tráfico saliente.

**Para comprobar el certificado de cliente:**

compruebe la huella digital del certificado de cliente actual. El almacén de certificados se puede encontrar en `%ProgramData%\microsoft\Microsoft AAD Application Proxy Connector\Config\TrustSettings.xml`.

```
<?xml version="1.0" encoding="utf-8"?>
<ConnectorTrustSettingsFile xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <CloudProxyTrust>
    <Thumbprint>4905CC64B2D81BBED60962ECC5DCF63F643CCD55</Thumbprint>
    <IsInUserStore>false</IsInUserStore>
  </CloudProxyTrust>
</ConnectorTrustSettingsFile>
```

Los valores de **IsInUserStore** posibles son **true** y **false**. Un valor de **true** significar que el certificado renovado automáticamente se almacena en el contenedor personal del almacén de certificados del usuario del servicio de red. Un valor de **false** significa que el certificado de cliente se creó durante la instalación o el registro iniciado por el comando Register-AppProxyConnector y se almacenó en el contenedor personal del almacén de certificados del equipo local.

Si el valor es **true**, siga estos pasos para comprobar el certificado:
1. Descargue [PsTools.zip](/sysinternals/downloads/pstools)
2. Extraiga [PsExec](/sysinternals/downloads/psexec) del paquete y ejecute **psexec -i -u "nt authority\network service" cmd.exe** desde un símbolo del sistema con privilegios elevados.
3. Ejecute **certmgr.msc** en el símbolo del sistema recién aparecido.
4. En la consola de administración, expanda el contenedor personal y haga clic en Certificados.
5. Busque el certificado emitido por **connectorregistrationca.msappproxy.net**.

Si el valor es **false**, siga estos pasos para comprobar el certificado:
1. Ejecute **certlm.msc**
2. En la consola de administración, expanda el contenedor personal y haga clic en Certificados.
3. Busque el certificado emitido por **connectorregistrationca.msappproxy.net**.

**Para renovar el certificado de cliente:**

Si un conector no se conecta al servicio durante varios meses, puede que tenga los certificados caducados. El error de la renovación del certificado conduce a un certificado expirado. Esto hace que el servicio del conector deje de funcionar. El evento 1000 se registra en el registro de administración del conector:

"Error en el nuevo registro del conector: el certificado de confianza del conector expiró. Ejecute el cmdlet de PowerShell Register-AppProxyConnector en el equipo donde se ejecuta el conector para volver a registrar el conector".

En este caso, desinstale y vuelva a instalar el conector para desencadenar el registro, o bien puede ejecutar estos comandos de PowerShell:

```
Import-module AppProxyPSModule
Register-AppProxyConnector
```

Para más información sobre el comando Register-AppProxyConnector, consulte [Creación de un script de instalación desatendida para el conector de Azure AD Application Proxy](./application-proxy-register-connector-powershell.md).

## <a name="verify-admin-is-used-to-install-the-connector"></a>Comprobación de que se use el administrador para instalar el conector

**Objetivo:** compruebe que el usuario que intenta instalar el conector sea un administrador con las credenciales correctas. Actualmente, el usuario debe ser como mínimo administrador de aplicaciones para que la instalación se complete correctamente.

**Para comprobar que las credenciales sean correctas:**

Conéctese a `https://login.microsoftonline.com` y use las mismas credenciales. Asegúrese de que el inicio de sesión se haya realizado correctamente. Para comprobar el rol de usuario, vaya a **Azure Active Directory**  -&gt; **Usuarios y grupos** -&gt; **Todos los usuarios**. 

Seleccione su cuenta de usuario y "Rol del directorio" en el menú resultante. Compruebe que el rol seleccionado sea "Administrador de aplicaciones". Si no puede acceder a alguna de las páginas en estos pasos, significa que no tiene el rol necesario.

## <a name="next-steps"></a>Pasos siguientes
[Descripción de los conectores del Proxy de aplicación de Azure AD](application-proxy-connectors.md)
