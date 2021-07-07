---
title: 'Tutorial: Configuración de HTTPS en un dominio personalizado de Azure CDN'
description: En este tutorial, aprenderá a habilitar y deshabilitar HTTPS en su dominio personalizado de punto de conexión de Azure CDN.
services: cdn
author: asudbring
ms.service: azure-cdn
ms.topic: tutorial
ms.date: 03/26/2021
ms.author: allensu
ms.custom: mvc
ms.openlocfilehash: d9520f3a6c6ffadf7186b0b3c5c83fe872711316
ms.sourcegitcommit: 190658142b592db528c631a672fdde4692872fd8
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/11/2021
ms.locfileid: "112007794"
---
# <a name="tutorial-configure-https-on-an-azure-cdn-custom-domain"></a>Tutorial: Configuración de HTTPS en un dominio personalizado de Azure CDN

En este tutorial se muestra cómo habilitar el protocolo HTTPS en un dominio personalizado asociado a un punto de conexión de Azure CDN. 

El protocolo HTTPS en el dominio personalizado (por ejemplo, https:\//www.contoso.com), garantiza que los datos confidenciales se entreguen de manera segura mediante TLS/SSL. Cuando el explorador web se conecta a través de HTTPS, el explorador valida el certificado del sitio web. El explorador comprueba que está emitido por una entidad de certificación legítima. Este proceso aporta seguridad y protege las aplicaciones web de posibles ataques.

Azure CDN admite HTTPS en un punto de conexión de CDN de manera predeterminada. Por ejemplo, si crea un punto de conexión de CDN (como https:\//contoso.azureedge.net), HTTPS se habilita de manera automática.  

Algunos de los atributos clave de la característica de HTTPS personalizado son:

- Sin costo adicional: la adquisición o renovación de certificados no tiene costos y el tráfico HTTPS no supone un costo adicional. Solo es preciso pagar por los GB de salida de la red CDN.

- Habilitación simple: el aprovisionamiento en un solo clic está disponible desde [Azure Portal](https://portal.azure.com). También puede utilizar la API de REST u otras herramientas de desarrollo para habilitar la característica.

- La administración completa de certificados está disponible: 
    * toda la adquisición y administración de certificados se controla de manera automática. 
    * Los certificados se aprovisionan y renuevan automáticamente antes de su expiración.

En este tutorial, aprenderá a:
> [!div class="checklist"]
> - Habilitar el protocolo HTTPS en su dominio personalizado
> - Usar un certificado administrado por CDN 
> - Usar su propio certificado
> - Validar el dominio
> - Deshabilitar el protocolo HTTPS en su dominio personalizado

## <a name="prerequisites"></a>Prerrequisitos

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)] 

Para poder completar los pasos de este tutorial, cree un perfil de CDN y al menos un punto de conexión de CDN. Para más información, consulte [Inicio rápido: Creación de un perfil y un punto de conexión de Azure CDN](cdn-create-new-endpoint.md).

Asocie un dominio personalizado de Azure CDN en el punto de conexión de CDN. Para más información, consulte el [Tutorial: Incorporación de un dominio personalizado a un punto de conexión de Azure CDN](cdn-map-content-to-custom-domain.md).

> [!IMPORTANT]
> Los certificados administrados por CDN no están disponibles para dominios raíz o de Apex. Si el dominio personalizado de Azure CDN es un dominio raíz o de Apex, debe usar la característica Traiga su propio certificado. 
>

---

## <a name="tlsssl-certificates"></a>Certificados TLS/SSL

Para habilitar HTTPS en un dominio personalizado de Azure CDN, use un certificado TLS/SSL. Puede usar un certificado que Azure CDN administre o usar su propio certificado.

# <a name="option-1-default-enable-https-with-a-cdn-managed-certificate"></a>[Opción 1 (valor predeterminado): habilitación de HTTPS con un certificado administrado por CDN](#tab/option-1-default-enable-https-with-a-cdn-managed-certificate)

Azure CDN controla las tareas de administración de certificados, como la adquisición y renovación. Después de habilitar la característica, el proceso se inicia inmediatamente. 

Si el dominio personalizado ya está asignado al punto de conexión de CDN, no es necesario realizar ninguna otra acción. Azure CDN procesa los pasos y completa su solicitud automáticamente.

Si su dominio personalizado se asigna en otra parte, use el correo electrónico para validar la propiedad del dominio.

Para habilitar HTTPS en un dominio personalizado, siga estos pasos:

1. Vaya a [Azure Portal](https://portal.azure.com) para buscar un certificado administrado por Azure CDN. Busque y seleccione **Perfiles de CDN**. 

2. Elija el perfil:
    * **Azure CDN Estándar de Microsoft**
    * **Azure CDN Estándar de Akamai**
    * **Azure CDN estándar de Verizon**
    * **Azure CDN premium de Verizon**

3. En la lista de puntos de conexión de CDN, haga clic en el que contiene el dominio personalizado.

    ![Lista de puntos de conexión](./media/cdn-custom-ssl/cdn-select-custom-domain-endpoint.png)

    Aparece la página **Punto de conexión**.

4. En la lista de dominios personalizados, seleccione el dominio personalizado para el que desea habilitar HTTPS.

    ![La captura de pantalla muestra la página de dominio personalizado, con la opción para usar un certificado propio.](./media/cdn-custom-ssl/cdn-custom-domain.png)

    Aparece la página **Dominio personalizado**.

5. En el tipo de administración de certificados, seleccione **Certificate management type** (Administrado por CDN).

6. Seleccione **On** (Activado) para habilitar HTTPS.

    ![Estado de HTTPS de dominio personalizado](./media/cdn-custom-ssl/cdn-select-cdn-managed-certificate.png)

7. Continúe y [valide el dominio](#validate-the-domain).


# <a name="option-2-enable-https-with-your-own-certificate"></a>[Opción 2: habilitación de HTTPS con su propio certificado](#tab/option-2-enable-https-with-your-own-certificate)

> [!IMPORTANT]
> Esta opción solo está disponible con los perfiles **Azure CDN de Microsoft** y **Azure CDN de Verizon**. 
>
 
Puede usar su propio certificado para habilitar la característica HTTPS. Este proceso se realiza a través de una integración con Azure Key Vault, lo cual permite almacenar de forma segura los certificados. Azure Front Door usa este mecanismo de seguridad para obtener el certificado y requiere algunos pasos adicionales. Al crear el certificado TLS/SSL, tiene que crear una cadena de certificados completa con una entidad de certificación (CA) permitida que forme parte de la [lista de CA de confianza de Microsoft](https://ccadb-public.secure.force.com/microsoft/IncludedCACertificateReportForMSFT). Si usa una CA no permitida, la solicitud se rechazará.  Si se presenta un certificado sin una cadena completa, no se garantiza que las solicitudes que involucran ese certificado funcionen según lo previsto. En el caso de Azure CDN de Verizon, se aceptará cualquier entidad de certificación válida.

### <a name="prepare-your-azure-key-vault-account-and-certificate"></a>Preparación de la cuenta y el certificado de Azure Key Vault
 
1. Azure Key Vault: tiene que tener una cuenta de Azure Key Vault ejecutándose en la misma suscripción que el perfil de Azure CDN y los puntos de conexión de CDN para los que quiera habilitar HTTPS personalizado. Cree una cuenta de Azure Key Vault si no tiene una.
 
2. Certificados de Azure Key Vault: Si tiene un certificado, cárguelo directamente en su cuenta de Azure Key Vault. Si no tiene un certificado, cree uno nuevo directamente mediante Azure Key Vault.

> [!NOTE]
> El certificado debe tener una cadena de certificados completa con certificados de hoja e intermedios, y la CA raíz debe formar parte de la [lista de CA de confianza de Microsoft](https://ccadb-public.secure.force.com/microsoft/IncludedCACertificateReportForMSFT).

### <a name="register-azure-cdn"></a>Registro de Azure CDN

Registre Azure CDN como una aplicación en Azure Active Directory mediante PowerShell.

1. Si es necesario, instale [Azure PowerShell](/powershell/azure/install-az-ps) en la máquina local.

2. En PowerShell, ejecute el siguiente comando:

     `New-AzADServicePrincipal -ApplicationId "205478c0-bd83-4e1b-a9d6-db63a3e1e1c8"`
    > [!NOTE]
    > **205478c0-bd83-4e1b-a9d6-db63a3e1e1c8** es la entidad de servicio de **Microsoft.AzureFrontDoor-Cdn**.

    ```bash  
    New-AzADServicePrincipal -ApplicationId "205478c0-bd83-4e1b-a9d6-db63a3e1e1c8"

    Secret                : 
    ServicePrincipalNames : {205478c0-bd83-4e1b-a9d6-db63a3e1e1c8, 
                                https://microsoft.onmicrosoft.com/033ce1c9-f832-4658-b024-ef1cbea108b8}
    ApplicationId         : 205478c0-bd83-4e1b-a9d6-db63a3e1e1c8
    ObjectType            : ServicePrincipal
    DisplayName           : Microsoft.AzureFrontDoor-Cdn
    Id                    : c87be08f-686a-4d9f-8ef8-64707dbd413e
    Type                  :
    ```
### <a name="grant-azure-cdn-access-to-your-key-vault"></a>Concesión de acceso de Azure CDN al almacén de claves
 
Conceda permisos de Azure CDN para acceder a los certificados (secretos) de su cuenta de Azure Key Vault.

1. En el almacén de claves, en la sección **Configuración**, seleccione **Directivas de acceso**. En el panel derecho, seleccione **+ Agregar directiva de acceso**:

    :::image type="content" source="./media/cdn-custom-ssl/cdn-new-access-policy.png" alt-text="Creación de directiva de acceso de Key Vault para CDN" border="true":::

2. En la página **Agregar directiva de acceso**, seleccione **Ninguna seleccionada** junto a **Seleccionar la entidad de seguridad**. En la página **Entidad de seguridad**, escriba **205478c0-bd83-4e1b-a9d6-db63a3e1e1c8**. Seleccione **Microsoft.AzureFrontdoor-Cdn**.  Elija **Seleccionar**:

3. En **Seleccionar la entidad de seguridad**, busque **205478c0-bd83-4e1b-a9d6-db63a3e1e1c8** y elija **Microsoft.AzureFrontDoor-Cdn**. Elija **Seleccionar**.

    :::image type="content" source="./media/cdn-custom-ssl/cdn-access-policy-settings.png" alt-text="Selección de la entidad de servicio de Azure CDN" border="true":::
    
4. Seleccione **Permisos de certificado**. Active las casillas de **Get** y **List** para permitir que CDN pueda obtener y enumerar los certificados.

5. Seleccione **Permisos de secretos**. Active las casillas de **Get** y **List** para permitir que CDN pueda obtener y enumerar los secretos:

    :::image type="content" source="./media/cdn-custom-ssl/cdn-vault-permissions.png" alt-text="Selección de permisos de CDN en el almacén de claves" border="true":::

6. Seleccione **Agregar**. 

> [!NOTE]
> Azure CDN puede acceder ahora a este almacén de claves y a los certificados (secretos) almacenados en él. Todas las instancias de la red CDN creadas en esta suscripción tendrán acceso a los certificados de este almacén de claves.
 
### <a name="select-the-certificate-for-azure-cdn-to-deploy"></a>Selección del certificado de Azure CDN que se va a implementar
 
1. Vuelva al portal de Azure CDN y seleccione el perfil y el punto de conexión de CDN para los que quiere habilitar HTTPS personalizado. 

2. En la lista de dominios personalizados, seleccione el dominio personalizado para el que desea habilitar HTTPS.

    Aparece la página **Dominio personalizado**.

3. En el tipo de administración de certificados, seleccione **Use my own certificate** (Usar mi propio certificado). 

    :::image type="content" source="./media/cdn-custom-ssl/cdn-configure-your-certificate.png" alt-text="Captura de pantalla de cómo configurar el certificado para el punto de conexión de CDN":::

4. Seleccione un almacén de claves, un certificado o secreto y la versión del certificado o secreto.

    Azure CDN muestra la siguiente información: 
    - Las cuentas del almacén de claves de su identificador de suscripción. 
    - Los certificados o secretos en el almacén de claves seleccionado. 
    - Las versiones del certificado o secreto disponibles.
 
    > [!NOTE]
    > Para que el certificado se rote automáticamente a la versión más reciente cuando haya disponible una última versión del certificado en la instancia de Key Vault, establezca la versión del certificado o del secreto en "latest". Si se selecciona una versión específica, tendrá que volver a seleccionar la nueva versión manualmente para la rotación de certificados. La nueva versión del certificado o el secreto tarda hasta 24 horas en implementarse. 
   
5. Seleccione **On** (Activado) para habilitar HTTPS.
  
6. Si usa su certificado, no se requiere la validación del dominio. Continúe con [Esperar a la propagación](#wait-for-propagation).

---

## <a name="validate-the-domain"></a>Validar el dominio

Si ya tiene un dominio personalizado en uso asignado a su punto de conexión personalizado con un registro CNAME o usa su propio certificado, continúe con [El dominio personalizado está asignado al punto de conexión de CDN](#custom-domain-is-mapped-to-your-cdn-endpoint-by-a-cname-record). 

En caso contrario, si la entrada del registro CNAME del punto de conexión ya no existe o no contiene el subdominio cdnverify, continúe con [El dominio personalizado no está asignado al punto de conexión de CDN](#custom-domain-isnt-mapped-to-your-cdn-endpoint).

### <a name="custom-domain-is-mapped-to-your-cdn-endpoint-by-a-cname-record"></a>El dominio personalizado está asignado al punto de conexión de CDN mediante un registro CNAME

Cuando agrega un dominio personalizado a su punto de conexión, crea un registro CNAME en el registrador de dominios de DNS asignado a su nombre de host del punto de conexión de CDN. 

Si el registro CNAME aún existe, pero no contiene el subdominio cdnverify, la CA DigiCert lo usa para validar automáticamente la titularidad de su dominio personalizado. 

Si usa su propio certificado, no se requiere la validación del dominio.

El registro CNAME debería tener el siguiente formato:

* *Nombre* es el nombre del dominio personalizado.
* *Valor* es el nombre de host del punto de conexión de CDN.

| Nombre            | Tipo  | Value                 |
|-----------------|-------|-----------------------|
| <www.contoso.com> | CNAME | contoso.azureedge.net |

Para obtener más información sobre los registros CNAME, consulte [Creación de los registros DNS de CNAME](./cdn-map-content-to-custom-domain.md).

Si el registro CNAME tiene el formato correcto, DigiCert automáticamente comprueba su nombre de dominio personalizado y crea un certificado para el dominio. DigitCert no enviará un correo electrónico de comprobación, y usted no tendrá que aprobar la solicitud. El certificado tiene una validez de un año y se renovará automáticamente antes de expirar. Continúe con [Esperar a la propagación](#wait-for-propagation). 

La validación automática suele tardar unas horas. Si no ve su dominio validado en 24 horas, abra una incidencia de soporte técnico.

>[!NOTE]
>Si tiene un registro de autorización de entidad de certificación (CAA) con el proveedor de DNS, debe incluir DigiCert como entidad de certificación válida. Un registro CAA permite que los propietarios de dominios especifiquen con sus proveedores de DNS qué entidades de certificación están autorizadas para emitir certificados para su dominio. Si una entidad de certificación recibe un pedido de un certificado para un dominio que posee un registro CAA y dicha entidad no figura como emisor autorizado, no se le permite emitir el certificado para ese dominio o subdominio. Para obtener información acerca de cómo administrar registros de CAA, consulte [Manage CAA records](https://support.dnsimple.com/articles/manage-caa-record/) (Administrar registros de CAA). Para obtener una herramienta de registros de CAA, consulte [CAA Record Helper](https://sslmate.com/caa/) (Asistente de registros de CAA).

### <a name="custom-domain-isnt-mapped-to-your-cdn-endpoint"></a>El dominio personalizado no está asignado al punto de conexión de CDN

>[!NOTE]
>Si usa **Azure CDN de Akamai**, debe configurar la entrada CNAME siguiente para habilitar la validación automatizada de dominios. "_acme-challenge.&lt;host de dominio personalizado&gt; -> CNAME -> &lt;host de dominio personalizado&gt;.ak-acme-challenge.azureedge.net"

Si la entrada del registro CNAME contiene el subdominio cdnverify, siga el resto de las instrucciones que aparecen en este paso.

DigiCert envía un correo electrónico de comprobación a las siguientes direcciones de correo electrónico. Compruebe que puede realizar las aprobaciones directamente desde una de las siguientes direcciones:

* **admin@your-domain-name.com** 
* **administrator@your-domain-name.com** 
* **webmaster@your-domain-name.com** 
* **hostmaster@your-domain-name.com**  
* **postmaster@your-domain-name.com**  

Debería recibir un correo electrónico en unos minutos para aprobar la solicitud. Si utiliza un filtro de correo no deseado, agregue verification@digicert.com a la lista de permitidos. Si no recibe un correo electrónico en 24 horas, póngase en contacto con el equipo de soporte técnico de Microsoft.
    
![Correo electrónico de validación del dominio](./media/cdn-custom-ssl/domain-validation-email.png)

Al seleccionar el vínculo de aprobación, se le conducirá al siguiente formulario de aprobación en línea: 
    
![Formulario de validación del dominio](./media/cdn-custom-ssl/domain-validation-form.png)

Siga las instrucciones que aparecen en el formulario; tiene dos opciones de comprobación:

- Puede aprobar todos los pedidos futuros realizados a través de la misma cuenta del mismo dominio raíz, por ejemplo, contoso.com. Se recomienda este método si piensa agregar más dominios personalizados al mismo dominio raíz.

- Puede aprobar solo el nombre de host específico usado en esta solicitud. Se requiere otra aprobación para las solicitudes posteriores.

Tras la aprobación, DigiCert completa la creación del certificados para el nombre de dominio personalizado. El certificado tiene una validez de un año y se renovará automáticamente antes de expirar.

## <a name="wait-for-propagation"></a>Esperar a la propagación

Una vez que el nombre de dominio se valide, la característica HTTPS del dominio personalizado tarda entre 6 y 8 horas en estar activa. Una vez completado el proceso, el estado en el cuadro de diálogo "Personalizar HTTPS" de Azure Portal cambia a **Habilitado**. Los cuatro pasos de la operación del cuadro de diálogo Dominio personalizado se marcan como completados. El dominio personalizado ahora está listo para usar HTTPS.

![Cuadro de diálogo Habilitar HTTPS](./media/cdn-custom-ssl/cdn-enable-custom-ssl-complete.png)

### <a name="operation-progress"></a>Progreso de la operación

En la tabla siguiente se muestra el progreso de la operación que se produce cuando se habilita HTTPS. Después de habilitar HTTPS, los cuatro pasos de la operación aparecen en el cuadro de diálogo del dominio personalizado. A medida que se activa cada paso, aparecen detalles de subpasos adicionales bajo el paso. No se producirán todos estos subpasos. Después de que un paso se completa correctamente, aparece una marca de verificación verde junto a él. 

| Paso de la operación | Detalles del subpaso de la operación | 
| --- | --- |
| 1 Envío de la solicitud | Envío de la solicitud |
| | Se está enviando la solicitud HTTPS. |
| | Se ha enviado correctamente la solicitud HTTPS. |
| 2 Validación del dominio | El dominio se valida automáticamente si está asignado por CNAME al punto de conexión de CDN. En caso contrario, se enviará una solicitud de verificación al correo electrónico que aparece en el registro de su dominio (registrante WHOIS).|
| | La propiedad del dominio se ha validado correctamente. |
| | La solicitud de validación de la propiedad del dominio ha expirado (el cliente probablemente no respondió dentro del plazo de 6 días). HTTPS no se habilitará en el dominio. * |
| | El cliente rechazó la solicitud de validación de la propiedad del dominio. HTTPS no se habilitará en el dominio. * |
| 3 Aprovisionamiento del certificado | La entidad de certificación está emitiendo actualmente el certificado necesario para habilitar HTTPS en el dominio. |
| | El certificado se ha emitido y actualmente se está implementando en la red CDN. Esto puede tardar hasta 6 horas. |
| | El certificado se ha implementado correctamente en la red CDN. |
| 4 Finalizado | Se habilitó correctamente HTTPS en el dominio. |

\* Este mensaje no aparecerá a menos que se haya producido un error. 

Si se produce un error antes de que se envíe la solicitud, verá el mensaje de error siguiente:

<code>
We encountered an unexpected error while processing your HTTPS request. Please try again and contact support if the issue persists.
</code>



## <a name="clean-up-resources---disable-https"></a>Clean up resources - disable HTTPS (Limpiar recursos: deshabilitar HTTPS)

En esta sección, aprenderá a deshabilitar HTTPS para el dominio personalizado.

### <a name="disable-the-https-feature"></a>Deshabilitación de la característica HTTPS 

1. En [Azure Portal](https://portal.azure.com), busque y seleccione **Perfiles de CDN**. 

2. Elija el perfil **Azure CDN estándar de MIcrosoft**, **Azure CDN estándar de Verizon** o **Azure CDN prémium de Verizon**.

3. En la lista de puntos de conexión, escoja el que contenga el dominio personalizado.

4. Elija el dominio personalizado en el que desea deshabilitar HTTPS.

    ![Lista de dominios personalizados](./media/cdn-custom-ssl/cdn-custom-domain-HTTPS-enabled.png)

5. Elija **Desactivar** para deshabilitar HTTPS y haga clic en **Aplicar**.

    ![Cuadro de diálogo Personalizar HTTPS](./media/cdn-custom-ssl/cdn-disable-custom-ssl.png)

### <a name="wait-for-propagation"></a>Esperar a la propagación

Cuando se deshabilita la característica HTTPS del dominio personalizado, puede tardar hasta 6 a 8 horas para que surta efecto. Una vez completado el proceso, el estado en el cuadro de diálogo "Personalizar HTTPS" de Azure Portal cambia a **Deshabilitado**. Los tres pasos de la operación del cuadro de diálogo Dominio personalizado se marcan como completados. El dominio personalizado ya no usa HTTPS.

![Cuadro de diálogo Deshabilitar HTTPS](./media/cdn-custom-ssl/cdn-disable-custom-ssl-complete.png)

#### <a name="operation-progress"></a>Progreso de la operación

En la tabla siguiente se muestra el progreso de la operación que se produce cuando se deshabilita HTTPS. Después de deshabilitar HTTPS, los tres pasos de la operación aparecen en el cuadro de diálogo Dominio personalizado. A medida que se activa cada paso, aparecen más detalles bajo el paso. Después de que un paso se completa correctamente, aparece una marca de verificación verde junto a él. 

| Progreso de la operación | Detalles de la operación | 
| --- | --- |
| 1 Envío de la solicitud | Enviando la solicitud |
| 2 Desaprovisionamiento del certificado | Eliminando certificado |
| 3 Finalizado | Certificado eliminado |

## <a name="frequently-asked-questions"></a>Preguntas más frecuentes

1. *¿Quién es el proveedor de certificados y qué tipo de certificado se utiliza?*

    Se usa un certificado dedicado, que proporciona Digicert, para el dominio personalizado de:
    
    * **Azure CDN de Verizon**
    * **Azure CDN de Microsoft**

2. *¿Usa TLS/SSL basado en IP o SNI?*

    Tanto **Azure CDN de Verizon** como **Azure CDN estándar de Microsoft** usan TLS/SSL de SNI.

3. *¿Qué debo hacer si no recibo el correo electrónico de comprobación de dominio de DigiCert?*

    Si no usa el subdominio *cdnverify* y la entrada CNAME es para el nombre de host del punto de conexión, no recibirá un correo electrónico de comprobación de dominio.
     
    La validación se produce automáticamente. En caso contrario, si no tiene una entrada CNAME y no ha recibido un correo electrónico en 24 horas, póngase en contacto con el servicio de soporte técnico de Microsoft.

4. *¿Son menos seguros los certificados SAN que los certificados dedicados?*
    
    Los certificados SAN siguen los mismos estándares de cifrado y seguridad que los certificados dedicados. Todos los certificados TLS/SSL emitidos usan SHA-256 para mejorar la seguridad del servidor.

5. *¿Necesito un registro de autorización de entidad de certificación con mi proveedor de DNS?*

    Actualmente no se requiere un registro de autorización de entidad de certificación. Pero si tiene uno, debe incluir DigiCert como entidad de certificación válida.

6. *El 20 de junio de 2018, Azure CDN de Verizon comenzó a usar un certificado dedicado con TLS/SSL de SNI de forma predeterminada. ¿Qué sucede con los dominios personalizados existentes que usan certificados de nombre alternativo del firmante (SAN) TLS/SSL basado en IP?*

    En los próximos meses, los dominios existentes se migrarán gradualmente a un único certificado si Microsoft analiza que solo se realizan solicitudes de cliente SNI a la aplicación. 
    
    Si se detectan clientes que no son SNI, los dominios permanecen en el certificado SAN con TLS/SSL basado en IP. Las solicitudes al servicio o a los clientes que no sean SNI no se verán afectadas.

7. *Funcionamiento de la renovación de certificados con Bring Your Own Certificate*

    Para asegurarse de que se implementa un certificado más reciente en la infraestructura de PoP, cargue el nuevo certificado en Azure Key Vault. En la configuración de TLS en Azure CDN, elija la versión más reciente del certificado y seleccione Guardar. Azure CDN propagará el nuevo certificado actualizado. 

8. *¿Es necesario volver a habilitar HTTPS después de reiniciar el punto de conexión?*

    Sí. Si usa **Azure CDN de Akamai**, si el punto de conexión se detiene y se reinicia, debe volver a habilitar la configuración de HTTPS si el valor estaba activo antes.


## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha aprendido a:

> [!div class="checklist"]
> - Habilitar el protocolo HTTPS en su dominio personalizado
> - Usar un certificado administrado por CDN 
> - Usar su propio certificado
> - Validar el dominio
> - Deshabilitar el protocolo HTTPS en su dominio personalizado

Avance al siguiente tutorial para aprender a configurar el almacenamiento en caché en el punto de conexión de CDN.

> [!div class="nextstepaction"]
> [Tutorial: Establecimiento de las reglas de almacenamiento en caché de Azure CDN](cdn-caching-rules-tutorial.md)
