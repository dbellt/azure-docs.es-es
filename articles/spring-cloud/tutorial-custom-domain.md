---
title: 'Tutorial: Asignación de un dominio personalizado existente a Azure Spring Cloud'
description: Asignación a Azure Spring Cloud de un nombre existente personalizado del servicio de nombres distribuido (DNS)
author: MikeDodaro
ms.service: spring-cloud
ms.topic: tutorial
ms.date: 03/19/2020
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: 7d01f3d5efeee2db5f468a0fe6217d1ff3c313a7
ms.sourcegitcommit: 2654d8d7490720a05e5304bc9a7c2b41eb4ae007
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107375022"
---
# <a name="tutorial-map-an-existing-custom-domain-to-azure-spring-cloud"></a>Tutorial: Asignación de un dominio personalizado existente a Azure Spring Cloud

**Este artículo se aplica a:** ✔️ Java ✔️ C#

El servicio de nombres de dominio (DNS) es una técnica para almacenar nombres de nodos de red en toda una red. En este tutorial se asigna un dominio, como www.contoso.com, mediante un registro CNAME. El dominio personalizado se protege con un certificado y se muestra cómo aplicar Seguridad de la capa de transporte (TLS), también conocida como Capa de sockets seguros (SSL). 

El tráfico web se cifra mediante certificados. Estos certificados TLS/SSL se pueden almacenar en Azure Key Vault. 

## <a name="prerequisites"></a>Prerrequisitos
* Una aplicación implementada en Azure Spring Cloud (consulte [Inicio rápido: Inicio de una aplicación de Azure Spring Cloud existente desde Azure Portal](spring-cloud-quickstart.md)) o puede usar una aplicación existente.
* Un nombre de dominio con acceso al registro DNS para el proveedor de dominios, como GoDaddy.
* Un certificado privado (es decir, el certificado autofirmado) de un proveedor de terceros. El certificado debe coincidir con el dominio.
* Una instancia implementada de [Azure Key Vault](../key-vault/general/overview.md).

## <a name="keyvault-private-link-considerations"></a>Consideraciones de vínculo privado de Key Vault

Las IP de administración de Azure Spring Cloud no forman parte de los servicios de Microsoft de confianza de Azure. Por lo tanto, para permitir que Azure Spring Cloud cargue certificados desde una instancia de Key Vault protegida con conexiones de puntos de conexión privado, debe agregar las siguientes direcciones IP al firewall de Azure Key Vault:

```
20.53.123.160 52.143.241.210 40.65.234.114 52.142.20.14 20.54.40.121 40.80.210.49 52.253.84.152 20.49.137.168 40.74.8.134 51.143.48.243
```

## <a name="import-certificate"></a>Importación de certificado
### <a name="prepare-your-certificate-file-in-pfx-optional"></a>Preparación del archivo de certificado en PFX (opcional)
Azure Key Vault admite la importación de certificados privados en formato PEM y PFX. Si el archivo PEM que ha obtenido del proveedor de certificados no funciona en la sección siguiente: [Guardado del certificado en Key Vault](#save-certificate-in-key-vault), siga los pasos que se indican a continuación para generar un archivo PFX para Azure Key Vault.

#### <a name="merge-intermediate-certificates"></a>Combinación de certificados intermedios

Si la entidad emisora de certificados ofrece varios certificados en la cadena de certificados, debe combinar los certificados en orden.

Para ello, abra cada certificado que ha recibido en un editor de texto.

Cree un archivo para el certificado combinado, denominado _mergedcertificate.crt_. En un editor de texto, copie el contenido de cada certificado en este archivo. Los certificados deben seguir el orden de la cadena de certificados, comenzando por el certificado y terminando por el certificado raíz. Debe ser similar al ejemplo siguiente:

```
-----BEGIN CERTIFICATE-----
<your entire Base64 encoded SSL certificate>
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
<The entire Base64 encoded intermediate certificate 1>
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
<The entire Base64 encoded intermediate certificate 2>
-----END CERTIFICATE-----

-----BEGIN CERTIFICATE-----
<The entire Base64 encoded root certificate>
-----END CERTIFICATE-----
```

#### <a name="export-certificate-to-pfx"></a>Exportar el certificado a PFX

Exporte el certificado TLS/SSL personalizado con la clave privada que utilizó para generar la solicitud de certificado.

Si la solicitud de certificado se genera con OpenSSL, se crea un archivo de clave privada. Para exportar el certificado a PFX, ejecute el comando siguiente: Reemplace los marcadores de posición _&lt;private-key-file>_ y _&lt;merged-certificate-file>_ por la ruta a la clave privada y al archivo de certificado combinado.

```bash
openssl pkcs12 -export -out myserver.pfx -inkey <private-key-file> -in <merged-certificate-file>
```

Cuando se le pida, defina una contraseña de exportación. Esta contraseña deberá usarla cuando posteriormente cargue el certificado TLS/SSL en Azure Key Vault.

Si usó IIS o _Certreq.exe_ para generar la solicitud de certificado, instale el certificado en la máquina local y luego [exporte el certificado a PFX](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/cc754329(v=ws.11)).

### <a name="save-certificate-in-key-vault"></a>Guardado del certificado en Key Vault
Para importar un certificado, es necesario que el archivo codificado PEM o PFX esté en el disco y se debe tener la clave privada. 
#### <a name="portal"></a>[Portal](#tab/Azure-portal)
Para cargar el certificado en el almacén de claves:
1. Vaya a la instancia de Key Vault.
1. En el panel de navegación izquierdo, haga clic en **Certificados**.
1. En el menú superior, haga clic en **Generar o importar**.
1. En el cuadro de diálogo **Crear un certificado** en **Método de creación de certificados**, seleccione `Import`.
1. En **Cargar archivo de certificado**, vaya a la ubicación del certificado y selecciónela.
1. En **Contraseña**, escriba la clave privada del certificado.
1. Haga clic en **Crear**.

    ![Importación del certificado 1](./media/custom-dns-tutorial/import-certificate-a.png)

#### <a name="cli"></a>[CLI](#tab/Azure-CLI)

```azurecli
az keyvault certificate import --file <path to .pfx file> --name <certificate name> --vault-name <key vault name> --password <export password>
```
---

### <a name="grant-azure-spring-cloud-access-to-your-key-vault"></a>Concesión de acceso al almacén de claves a Azure Spring Cloud

Para conceder acceso a su almacén de claves a Azure Spring Cloud antes de importar el certificado:
#### <a name="portal"></a>[Portal](#tab/Azure-portal)
1. Vaya a la instancia de Key Vault.
1. En el panel de navegación izquierdo, haga clic en **Directiva de acceso**.
1. En el menú superior, haga clic en **Agregar directiva de acceso**.
1. Rellene la información, haga clic en el botón **Agregar** y, a continuación, **guarde** la directiva de acceso.

| Permiso de secreto | Permiso de certificado | Selección de la entidad de seguridad |
|--|--|--|
| Get, List | Get, List | Administración de dominios en Azure Spring Cloud |

![Importación del certificado 2](./media/custom-dns-tutorial/import-certificate-b.png)

#### <a name="cli"></a>[CLI](#tab/Azure-CLI)

Conceda acceso de lectura al almacén de claves a Azure Spring Cloud y reemplace `<key vault resource group>` y `<key vault name>` en el siguiente comando.
```
az keyvault set-policy -g <key vault resource group> -n <key vault name>  --object-id 938df8e2-2b9d-40b1-940c-c75c33494239 --certificate-permissions get list --secret-permissions get list
``` 
---

### <a name="import-certificate-to-azure-spring-cloud"></a>Importación del certificado en Azure Spring Cloud
#### <a name="portal"></a>[Portal](#tab/Azure-portal)
1. Vaya a la instancia del servicio. 
1. En el panel de navegación izquierdo de la aplicación, seleccione **Configuración de TLS/SSL**.
1. Luego, haga clic en **Import Key Vault Certificate** (Importar certificado de Key Vault).

    ![Importación de certificado](./media/custom-dns-tutorial/import-certificate.png)

1. Cuando haya importado correctamente el certificado, lo verá en la lista de **Certificados de clave privada**.

    ![Certificado de clave privada](./media/custom-dns-tutorial/key-certificates.png)

#### <a name="cli"></a>[CLI](#tab/Azure-CLI)

```
az spring-cloud certificate add --name <cert name> --vault-uri <key vault uri> --vault-certificate-name <key vault cert name>
```

Para mostrar una lista de los certificados importados:

```
az spring-cloud certificate list --resource-group <resource group name> --service <service name>
```
---

> [!IMPORTANT] 
> Para proteger un dominio personalizado con este certificado, todavía debe enlazar el certificado a un dominio determinado. Siga los pasos descritos en esta sección: [Agregar enlace SSL](#add-ssl-binding).

## <a name="add-custom-domain"></a>Adición de un dominio personalizado
Puede usar un registro CNAME para asignar un nombre DNS personalizado a Azure Spring Cloud. 

> [!NOTE] 
> No se admite el registro D. 

### <a name="create-the-cname-record"></a>Crear un registro CNAME
Vaya a su proveedor de DNS y agregue un registro CNAME para asignar el dominio a < service_name>.azuremicroservices.io. Aquí <service_name> es el nombre de la instancia de Azure Spring Cloud. Se admite el dominio y el subdominio con comodín. Después de agregar el registro CNAME, la página de registros DNS se parecerá al ejemplo siguiente: 

![Página de registros DNS](./media/custom-dns-tutorial/dns-records.png)

## <a name="map-your-custom-domain-to-azure-spring-cloud-app"></a>Asignación del dominio personalizado a la aplicación de Azure Spring Cloud
Si no tiene una aplicación en Azure Spring Cloud, siga las instrucciones que se indican en [Inicio rápido: Inicio de una aplicación de Azure Spring Cloud existente desde Azure Portal](./spring-cloud-quickstart.md).

#### <a name="portal"></a>[Portal](#tab/Azure-portal)
Vaya a la página de la aplicación.

1. Seleccione **Dominio personalizado**.
2. Luego, elija **Add Custom Domain** (Agregar dominio personalizado). 

    ![Dominio personalizado](./media/custom-dns-tutorial/custom-domain.png)

3. Escriba el nombre de dominio completo para el que ha agregado un registro CNAME, como www.contoso.com. Asegúrese de que el tipo de registro de nombre de host esté establecido en CNAME (<service_name>.azuremicroservices.io)
4. Haga clic en **Validar** para habilitar el botón **Agregar**.
5. Haga clic en **Agregar**.

    ![Adición de un dominio personalizado](./media/custom-dns-tutorial/add-custom-domain.png)

Una aplicación puede tener varios dominios, pero un dominio solo puede asignarse a una aplicación. Cuando haya asignado correctamente el dominio personalizado a la aplicación, lo verá en la tabla de dominios personalizados.

![Tabla de dominios personalizados](./media/custom-dns-tutorial/custom-domain-table.png)

#### <a name="cli"></a>[CLI](#tab/Azure-CLI)
```
az spring-cloud app custom-domain bind --domain-name <domain name> --app <app name> --resource-group <resource group name> --service <service name>
```

Para mostrar la lista de dominios personalizados:
```
az spring-cloud app custom-domain list --app <app name> --resource-group <resource group name> --service <service name>
```
---

> [!NOTE]
> Una etiqueta **Not Secure** (No seguro) para el dominio personalizado significa que aún no está enlazado a un certificado SSL. Todas las solicitudes HTTPS a este dominio personalizado desde un explorador recibirán un error o una advertencia.

## <a name="add-ssl-binding"></a>Agregar enlace SSL

#### <a name="portal"></a>[Portal](#tab/Azure-portal)
En la tabla de dominios personalizados, seleccione **Agregar enlace SSL** como se muestra en la ilustración anterior.  
1. Seleccione el **certificado** o impórtelo.
1. Haga clic en **Save**(Guardar).

    ![Agregar enlace SSL 1](./media/custom-dns-tutorial/add-ssl-binding.png)

#### <a name="cli"></a>[CLI](#tab/Azure-CLI)
```
az spring-cloud app custom-domain update --domain-name <domain name> --certificate <cert name> --app <app name> --resource-group <resource group name> --service <service name>
```
---

Después de agregar correctamente el enlace SSL, el estado del dominio será seguro: **Correcto**. 

![Agregar enlace SSL 2](./media/custom-dns-tutorial/secured-domain-state.png)

## <a name="enforce-https"></a>Aplicación de HTTPS
De forma predeterminada, los usuarios pueden seguir accediendo a la aplicación mediante HTTP, pero puede redirigir todas las solicitudes HTTP al puerto HTTPS.
#### <a name="portal"></a>[Portal](#tab/Azure-portal)
En la página de la aplicación, en el panel de navegación izquierdo, seleccione **Dominio personalizado**. Luego, establezca **Solo HTTPS** en *True*.

![Agregar enlace SSL 3](./media/custom-dns-tutorial/enforce-http.png)

#### <a name="cli"></a>[CLI](#tab/Azure-CLI)
```
az spring-cloud app update -n <app name> --resource-group <resource group name> --service <service name> --https-only
```
---
Una vez finalizada la operación, vaya a cualquiera de las direcciones URL HTTP que apuntan a la aplicación. Observe que las direcciones URL HTTP no funcionan.

## <a name="see-also"></a>Consulte también
* [¿Qué es Azure Key Vault?](../key-vault/general/overview.md)
* [Importación de un certificado](../key-vault/certificates/certificate-scenarios.md#import-a-certificate)
* [Inicio de la aplicación Spring Cloud desde la CLI de Azure](./spring-cloud-quickstart.md)
