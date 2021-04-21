---
title: Asignación de un dominio personalizado a un punto de conexión de Azure Blob Storage
titleSuffix: Azure Storage
description: Asignación de un dominio personalizado a un punto de conexión web o de Blob Storage en una cuenta de Azure Storage.
author: normesta
ms.service: storage
ms.topic: how-to
ms.date: 02/12/2021
ms.author: normesta
ms.reviewer: dineshm
ms.subservice: blobs
ms.openlocfilehash: 45ae3d80202bfb29074461f899798d278eb0895b
ms.sourcegitcommit: 49b2069d9bcee4ee7dd77b9f1791588fe2a23937
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/16/2021
ms.locfileid: "107538359"
---
# <a name="map-a-custom-domain-to-an-azure-blob-storage-endpoint"></a>Asignación de un dominio personalizado a un punto de conexión de Azure Blob Storage

Puede asignar un dominio personalizado a un punto de conexión del servicio de blobs o a un punto de conexión de un [sitio web estático](storage-blob-static-website.md). 

> [!NOTE] 
> Esta asignación solo funciona para subdominios (por ejemplo: `www.contoso.com`). Si desea que su punto de conexión web esté disponible en el dominio raíz (por ejemplo: `contoso.com`), tendrá que usar Azure CDN. Para obtener instrucciones, consulte la sección [Asignación de un dominio personalizado con HTTPS habilitado](#enable-https) de este artículo. Dado que va a ir a esa sección de este artículo para activar el dominio raíz de su dominio personalizado, el paso de dicha sección para habilitar HTTPS es opcional. 

<a id="enable-http"></a>

## <a name="map-a-custom-domain-with-only-http-enabled"></a>Asignación de un dominio personalizado en el que solo HTTP está habilitado

Este enfoque es más sencillo, pero solo permite el acceso HTTP. Si la cuenta de almacenamiento está configurada para [requerir la transferencia segura](../common/storage-require-secure-transfer.md) a través de HTTPS, debe habilitar el acceso HTTPS a su dominio personalizado. 

Para habilitar el acceso HTTPS, consulte la sección [Asignación de un dominio personalizado con HTTPS habilitado](#enable-https) de este artículo. 

<a id="map-a-domain"></a>

### <a name="map-a-custom-domain"></a>Asignación de un dominio personalizado

> [!IMPORTANT]
> Mientras completa la configuración, el dominio personalizado dejará de estar disponible durante un breve período para los usuarios. Si el dominio admite actualmente una aplicación con un acuerdo de nivel de servicio (SLA) que requiere que no haya tiempo de inactividad, siga los pasos de la sección [Asignación de un dominio personalizado sin tiempo de inactividad](#zero-down-time) de este artículo para asegurarse de que los usuarios puedan acceder al dominio mientras se realiza la asignación de DNS.

Si no le preocupa que el dominio no disponible durante un breve período para sus usuarios, siga estos pasos.

:heavy_check_mark: Paso 1: Obtención del nombre de host del punto de conexión del almacenamiento.

:heavy_check_mark: Paso 2: Creación de un registro de nombres canónicos (CNAME) con su proveedor de dominios.

:heavy_check_mark: Paso 3: Registro del dominio personalizado en Azure. 

:heavy_check_mark: Paso 4: Prueba de un dominio personalizado.

<a id="endpoint"></a>

#### <a name="step-1-get-the-host-name-of-your-storage-endpoint"></a>Paso 1: Obtención del nombre de host del punto de conexión del almacenamiento 

El nombre de host es la dirección URL del punto de conexión del almacenamiento sin el identificador del protocolo ni la barra diagonal final. 

1. En [Azure Portal](https://portal.azure.com), vaya a la cuenta de almacenamiento.

2. En el panel de menús, en **Configuración**, seleccione **Propiedades**.  

3. Copie el valor de **Punto de conexión principal de Blob service** o de **Punto de conexión del sitio web estático principal** en un archivo de texto. 
  
   > [!NOTE]
   > No se admite el punto de conexión de almacenamiento de Data Lake (por ejemplo, `https://mystorageaccount.dfs.core.windows.net/`).

4. Quite el identificador de protocolo (por ejemplo, `HTTPS`) y la barra diagonal final de esa cadena. La siguiente tabla contiene ejemplos.

   | Tipo de punto de conexión |  endpoint | nombre de host |
   |------------|-----------------|-------------------|
   |Blob service  | `https://mystorageaccount.blob.core.windows.net/` | `mystorageaccount.blob.core.windows.net` |
   |sitio web estático  | `https://mystorageaccount.z5.web.core.windows.net/` | `mystorageaccount.z5.web.core.windows.net` |
  
   Deje este valor para más adelante.

<a id="create-cname-record"></a>

#### <a name="step-2-create-a-canonical-name-cname-record-with-your-domain-provider"></a>Paso 2: Creación de un registro de nombres canónicos (CNAME) con su proveedor de dominios

Cree un registro CNAME que apunte al nombre de host. Un registro CNAME es un tipo de registro de sistema de nombres de dominio (DNS) que asigna un nombre de dominio de origen a un nombre de dominio de destino.

1. Inicie sesión en el sitio web del registrador de su dominio y vaya a la página de administración del valor de DNS.

   Podría encontrar la página en una sección como **Nombre de dominio**, **DNS** o **Administración del servidor de nombres**.

2. Busque la sección para la administración de registros CNAME. 

   Tiene que dirigirse a la página de configuración avanzada y buscar **CNAME**, **Alias** o **Subdomains**.

3. Cree un registro CNAME. Como parte de ese registro, especifique los siguientes elementos: 

   - El alias de subdominio, como `www` o `photos`. El subdominio es obligatorio, pero los dominios raíz no se admiten. 
      
   - El nombre de host que obtuvo en la sección [Obtención del nombre de host de un punto de conexión del almacenamiento](#endpoint) de este mismo artículo. 

<a id="register"></a>

#### <a name="step-3-register-your-custom-domain-with-azure"></a>Paso 3: Registro de su dominio personalizado en Azure

##### <a name="portal"></a>[Portal](#tab/azure-portal)

1. En [Azure Portal](https://portal.azure.com), vaya a la cuenta de almacenamiento.

2. En el panel de menú, en **Blob service**, seleccione **Dominio personalizado**.

   > [!NOTE]
   > Esta opción no aparece en las cuentas que tienen habilitada la característica de espacio de nombres jerárquico. Para esas cuentas, use PowerShell o la CLI de Azure para completar este paso.

   ![Opción de dominio personalizado](./media/storage-custom-domain-name/custom-domain-button.png "dominio personalizado")

   Se abre el panel **Dominio personalizado**.

3. En el cuadro de texto **Nombre de dominio**, escriba el nombre de su dominio personalizado, incluido el subdominio  
   
   Por ejemplo, si su dominio es *contoso.com* y su alias de subdominio es *www*, escriba `www.contoso.com`. Si su subdominio es *photos*, escriba `photos.contoso.com`.

4. Para registrar el dominio personalizado, elija el botón **Guardar**.

   Una vez que el registro CNAME se ha propagado por los servidores de nombres de dominio (DNS), y si los usuarios tienen los permisos apropiados, pueden ver los datos de blob mediante el dominio personalizado.

##### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Ejecute el siguiente comando de PowerShell:

```powershell
Set-AzStorageAccount -ResourceGroupName <resource-group-name> -Name <storage-account-name> -CustomDomainName <custom-domain-name> -UseSubDomain $false
```

- Reemplace el marcador de posición `<resource-group-name>` por el nombre del grupo de recursos.

- Reemplace el marcador de posición `<storage-account-name>` por el nombre de la cuenta de almacenamiento.

- Reemplace el marcador de posición `<custom-domain-name>` por el nombre del dominio personalizado, incluido el subdominio.

  Por ejemplo, si su dominio es *contoso.com* y su alias de subdominio es *www*, escriba `www.contoso.com`. Si su subdominio es *photos*, escriba `photos.contoso.com`.

Una vez que el registro CNAME se ha propagado por los servidores de nombres de dominio (DNS), y si los usuarios tienen los permisos apropiados, pueden ver los datos de blob mediante el dominio personalizado.

##### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Ejecute el siguiente comando de PowerShell:

```azurecli
az storage account update \
   --resource-group <resource-group-name> \ 
   --name <storage-account-name> \
   --custom-domain <custom-domain-name> \
   --use-subdomain false
  ```

- Reemplace el marcador de posición `<resource-group-name>` por el nombre del grupo de recursos.

- Reemplace el marcador de posición `<storage-account-name>` por el nombre de la cuenta de almacenamiento.

- Reemplace el marcador de posición `<custom-domain-name>` por el nombre del dominio personalizado, incluido el subdominio.

  Por ejemplo, si su dominio es *contoso.com* y su alias de subdominio es *www*, escriba `www.contoso.com`. Si su subdominio es *photos*, escriba `photos.contoso.com`.

Una vez que el registro CNAME se ha propagado por los servidores de nombres de dominio (DNS), y si los usuarios tienen los permisos apropiados, pueden ver los datos de blob mediante el dominio personalizado.

---

#### <a name="step-4-test-your-custom-domain"></a>Paso 4: Prueba de un dominio personalizado

Para confirmar que el dominio personalizado se haya asignado al punto de conexión de Blob service, cree un blob en un contenedor público en la cuenta de almacenamiento. A continuación, en un explorador web, acceda al blob mediante el uso de un identificador URI con el siguiente formato: `http://<subdomain.customdomain>/<mycontainer>/<myblob>`.

Por ejemplo, para acceder a un formulario web del contenedor `myforms` en el subdominio personalizado *photos.contoso.com*, debe usar el siguiente identificador URI: `http://photos.contoso.com/myforms/applicationform.htm`.

<a id="zero-down-time"></a>

### <a name="map-a-custom-domain-with-zero-downtime"></a>Asignación de un dominio personalizado sin tiempo de inactividad

> [!NOTE]
> Si no le preocupa que el dominio no esté disponible durante un breve período para los usuarios, considere la posibilidad de aplicar los pasos que se describen en la sección [Asignación de un dominio personalizado](#map-a-domain) de este artículo. Es un enfoque más sencillo con menos pasos.  

Si el dominio admite actualmente una aplicación con un acuerdo de nivel de servicio (SLA) que requiere que no haya tiempo de inactividad, siga estos pasos para asegurarse de que los usuarios puedan acceder al dominio mientras se realiza la asignación de DNS. 

:heavy_check_mark: Paso 1: Obtención del nombre de host del punto de conexión del almacenamiento.

:heavy_check_mark: Paso 2: Creación de un registro de nombres canónicos (CNAME) intermediario con su proveedor de dominios

:heavy_check_mark: Paso 3: Registro previo del dominio personalizado en Azure.

:heavy_check_mark: Paso 4: Creación de un registro CNAME con su proveedor de dominios.

:heavy_check_mark: Paso 5: Prueba de un dominio personalizado.

<a id="endpoint-2"></a>

#### <a name="step-1-get-the-host-name-of-your-storage-endpoint"></a>Paso 1: Obtención del nombre de host del punto de conexión del almacenamiento 

El nombre de host es la dirección URL del punto de conexión del almacenamiento sin el identificador del protocolo ni la barra diagonal final. 

1. En [Azure Portal](https://portal.azure.com), vaya a la cuenta de almacenamiento.

2. En el panel de menús, en **Configuración**, seleccione **Propiedades**.  

3. Copie el valor de **Punto de conexión principal de Blob service** o de **Punto de conexión del sitio web estático principal** en un archivo de texto. 

   > [!NOTE]
   > No se admite el punto de conexión de almacenamiento de Data Lake (por ejemplo, `https://mystorageaccount.dfs.core.windows.net/`).

4. Quite el identificador de protocolo (por ejemplo, `HTTPS`) y la barra diagonal final de esa cadena. La siguiente tabla contiene ejemplos.

   | Tipo de punto de conexión |  endpoint | nombre de host |
   |------------|-----------------|-------------------|
   |Blob service  | `https://mystorageaccount.blob.core.windows.net/` | `mystorageaccount.blob.core.windows.net` |
   |sitio web estático  | `https://mystorageaccount.z5.web.core.windows.net/` | `mystorageaccount.z5.web.core.windows.net` |
  
   Deje este valor para más adelante.

#### <a name="step-2-create-an-intermediary-canonical-name-cname-record-with-your-domain-provider"></a>Paso 2: Creación de un registro de nombres canónicos (CNAME) intermediario con su proveedor de dominios

Cree un registro CNAME temporal que apunte al nombre de host. Un registro CNAME es un tipo de registro de DNS que asigna un nombre de dominio de origen a un nombre de dominio de destino.

1. Inicie sesión en el sitio web del registrador de su dominio y vaya a la página de administración del valor de DNS.

   Podría encontrar la página en una sección como **Nombre de dominio**, **DNS** o **Administración del servidor de nombres**.

2. Busque la sección para la administración de registros CNAME. 

   Tiene que dirigirse a la página de configuración avanzada y buscar **CNAME**, **Alias** o **Subdomains**.

3. Cree un registro CNAME. Como parte de ese registro, especifique los siguientes elementos: 

   - El alias de subdominio, como `www` o `photos`. El subdominio es obligatorio, pero los dominios raíz no se admiten.

     Agregue el subdominio `asverify` al alias. Por ejemplo, `asverify.www` o `asverify.photos`.
       
   - El nombre de host que obtuvo en la sección [Obtención del nombre de host de un punto de conexión del almacenamiento](#endpoint) de este mismo artículo. 

     Agregue el subdominio `asverify` al nombre de host. Por ejemplo: `asverify.mystorageaccount.blob.core.windows.net`.

#### <a name="step-3-pre-register-your-custom-domain-with-azure"></a>Paso 3: Registro previo del dominio personalizado en Azure

Al realizar un registro previo de su dominio personalizado en Azure, permite que Azure reconozca el dominio personalizado sin tener que modificar el registro DNS para el dominio. De esa forma cuando modifique el registro DNS del dominio, se asignará al punto de conexión del blob sin que exista tiempo de inactividad.

##### <a name="portal"></a>[Portal](#tab/azure-portal)

1. En [Azure Portal](https://portal.azure.com), vaya a la cuenta de almacenamiento.

2. En el panel de menú, en **Blob service**, seleccione **Dominio personalizado**.

   > [!NOTE]
   > Esta opción no aparece en las cuentas que tienen habilitada la característica de espacio de nombres jerárquico. Para esas cuentas, use PowerShell o la CLI de Azure para completar este paso.

   ![Opción de dominio personalizado](./media/storage-custom-domain-name/custom-domain-button.png "dominio personalizado")

   Se abre el panel **Dominio personalizado**.

3. En el cuadro de texto **Nombre de dominio**, escriba el nombre de su dominio personalizado, incluido el subdominio  
   
   Por ejemplo, si su dominio es *contoso.com* y su alias de subdominio es *www*, escriba `www.contoso.com`. Si su subdominio es *photos*, escriba `photos.contoso.com`.

4. Seleccione la casilla **Usar validación CNAME indirecta**.

5. Para registrar el dominio personalizado, elija el botón **Guardar**.
  
   Si el registro se realiza correctamente, en el portal se notificará que la cuenta de almacenamiento se actualizó correctamente. Azure ha comprobado el dominio personalizado, pero no se ha realizado el enrutamiento del tráfico al dominio en la cuenta de almacenamiento hasta que cree un registro CNAME con el proveedor de dominios. Lo hará en la sección siguiente.

##### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Ejecute el siguiente comando de PowerShell:

```powershell
Set-AzStorageAccount -ResourceGroupName <resource-group-name> -Name <storage-account-name> -CustomDomainName <custom-domain-name> -UseSubDomain $true
```

- Reemplace el marcador de posición `<resource-group-name>` por el nombre del grupo de recursos.

- Reemplace el marcador de posición `<storage-account-name>` por el nombre de la cuenta de almacenamiento.

- Reemplace el marcador de posición `<custom-domain-name>` por el nombre del dominio personalizado, incluido el subdominio.

  Por ejemplo, si su dominio es *contoso.com* y su alias de subdominio es *www*, escriba `www.contoso.com`. Si su subdominio es *photos*, escriba `photos.contoso.com`.

Aún no se ha realizado el enrutamiento del tráfico al dominio en la cuenta de almacenamiento hasta que cree un registro CNAME con el proveedor de dominios. Lo hará en la sección siguiente.

##### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Ejecute el siguiente comando de PowerShell:

```azurecli
az storage account update \
   --resource-group <resource-group-name> \ 
   --name <storage-account-name> \
   --custom-domain <custom-domain-name> \
   --use-subdomain true
  ```

- Reemplace el marcador de posición `<resource-group-name>` por el nombre del grupo de recursos.

- Reemplace el marcador de posición `<storage-account-name>` por el nombre de la cuenta de almacenamiento.

- Reemplace el marcador de posición `<custom-domain-name>` por el nombre del dominio personalizado, incluido el subdominio.

  Por ejemplo, si su dominio es *contoso.com* y su alias de subdominio es *www*, escriba `www.contoso.com`. Si su subdominio es *photos*, escriba `photos.contoso.com`.

Aún no se ha realizado el enrutamiento del tráfico al dominio en la cuenta de almacenamiento hasta que cree un registro CNAME con el proveedor de dominios. Lo hará en la sección siguiente.

---

#### <a name="step-4-create-a-cname-record-with-your-domain-provider"></a>Paso 4: Creación de un registro CNAME con su proveedor de dominios

Cree un registro CNAME temporal que apunte al nombre de host.

1. Inicie sesión en el sitio web del registrador de su dominio y vaya a la página de administración del valor de DNS.

   Podría encontrar la página en una sección como **Nombre de dominio**, **DNS** o **Administración del servidor de nombres**.

2. Busque la sección para la administración de registros CNAME. 

   Tiene que dirigirse a la página de configuración avanzada y buscar **CNAME**, **Alias** o **Subdomains**.

3. Cree un registro CNAME. Como parte de ese registro, especifique los siguientes elementos: 

   - El alias de subdominio, como `www` o `photos`. El subdominio es obligatorio, pero los dominios raíz no se admiten.
      
   - El nombre de host que obtuvo en la sección [Obtención del nombre de host de un punto de conexión del almacenamiento](#endpoint-2) de este mismo artículo. 

#### <a name="step-5-test-your-custom-domain"></a>Paso 5: Prueba de un dominio personalizado

Para confirmar que el dominio personalizado se haya asignado al punto de conexión de Blob service, cree un blob en un contenedor público en la cuenta de almacenamiento. A continuación, en un explorador web, acceda al blob mediante el uso de un identificador URI con el siguiente formato: `http://<subdomain.customdomain>/<mycontainer>/<myblob>`.

Por ejemplo, para acceder a un formulario web del contenedor `myforms` en el subdominio personalizado *photos.contoso.com*, debe usar el siguiente identificador URI: `http://photos.contoso.com/myforms/applicationform.htm`.

### <a name="remove-a-custom-domain-mapping"></a>Eliminación de una asignación de dominio personalizado

Para quitar una asignación de dominio personalizado, anule el registro del dominio personalizado. Realice uno de los siguientes procedimientos.

#### <a name="portal"></a>[Portal](#tab/azure-portal)

1. En [Azure Portal](https://portal.azure.com), vaya a la cuenta de almacenamiento.

2. En el panel de menú, en **Blob service**, seleccione **Dominio personalizado**.  
   Se abre el panel **Dominio personalizado**.

3. Borre el contenido del cuadro de texto que incluye el nombre de dominio personalizado.

4. Seleccione el botón **Guardar**.

Una vez se haya quitado correctamente el dominio personalizado, verá una notificación del portal que indica que la cuenta de almacenamiento se actualizó correctamente.

#### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Para quitar un registro de dominio personalizado, use el cmdlet de PowerShell [Set-AzStorageAccount](/powershell/module/az.storage/set-azstorageaccount) y especifique una cadena vacía (`""`) para el valor del argumento `-CustomDomainName`.

* Formato de comando:

  ```powershell
  Set-AzStorageAccount `
      -ResourceGroupName "<resource-group-name>" `
      -AccountName "<storage-account-name>" `
      -CustomDomainName ""
  ```

* Ejemplo de comando:

  ```powershell
  Set-AzStorageAccount `
      -ResourceGroupName "myresourcegroup" `
      -AccountName "mystorageaccount" `
      -CustomDomainName ""
  ```

#### <a name="azure-cli"></a>[CLI de Azure](#tab/azure-cli)

Para quitar un registro de dominio personalizado, use el comando de la CLI [az storage account update](/cli/azure/storage/account) y especifique una cadena vacía (`""`) para el valor del argumento `--custom-domain`.

* Formato de comando:

  ```azurecli
  az storage account update \
      --name <storage-account-name> \
      --resource-group <resource-group-name> \
      --custom-domain ""
  ```

* Ejemplo de comando:

  ```azurecli
  az storage account update \
      --name mystorageaccount \
      --resource-group myresourcegroup \
      --custom-domain ""
  ```
---

<a id="enable-https"></a>

## <a name="map-a-custom-domain-with-https-enabled"></a>Asignación de un dominio personalizado con HTTPS habilitado

Este enfoque implica más pasos, pero habilita el acceso HTTPS. 

Si no necesita que los usuarios accedan al contenido de su blob o web mediante HTTPS, consulte la sección [Asignación de un dominio personalizado en el que solo HTTP está habilitado](#enable-http) de este mismo artículo. 

1. Habilite [Azure CDN](../../cdn/cdn-overview.md) punto de conexión de su blob o web. 

   Para ver un punto de conexión de Blob Storage, consulte [Integración de una cuenta de Azure Storage con Azure CDN](../../cdn/cdn-create-a-storage-account-with-cdn.md). 

   Para obtener un punto de conexión de un sitio web estático, consulte [Integración de un sitio web estático con Azure CDN](static-website-content-delivery-network.md).

2. [Asignación del contenido de Azure CDN a un dominio personalizado](../../cdn/cdn-map-content-to-custom-domain.md).

3. [Habilitación de HTTPS en un dominio personalizado de Azure CDN](../../cdn/cdn-custom-ssl.md).

   > [!NOTE] 
   > Al actualizar el sitio web estático, asegúrese de borrar el contenido almacenado en caché de los servidores perimetrales de CDN purgando el punto de conexión de CDN. Para más información, consulte [Purgar un punto de conexión de Azure CDN](../../cdn/cdn-purge-endpoint.md).

4. (Opcional) Consulte las siguientes instrucciones:

   * [Tokens de firma de acceso compartido (SAS) con Azure CDN](../../cdn/cdn-storage-custom-domain-https.md#shared-access-signatures).

   * [Redireccionamiento de HTTP a HTTPS con Azure CDN](../../cdn/cdn-storage-custom-domain-https.md#http-to-https-redirection).

   * [Precios y facturación cuando se usa Blob Storage con Azure CDN](../../cdn/cdn-storage-custom-domain-https.md#pricing-and-billing).

## <a name="next-steps"></a>Pasos siguientes

* [Hospedaje de sitios web estáticos en Azure Blob Storage](storage-blob-static-website.md)