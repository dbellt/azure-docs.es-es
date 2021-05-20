---
title: 'Archivos de SharePoint: QnA Maker'
description: Agregue orígenes de datos protegidos de SharePoint a la base de conocimiento para enriquecerla con preguntas y respuestas que se pueden proteger con Active Directory.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 02/20/2020
ms.openlocfilehash: c15bac8417fdba5f87551dc13311b5272c0926ee
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108743856"
---
# <a name="add-a-secured-sharepoint-data-source-to-your-knowledge-base"></a>Adición de un origen de datos protegido de SharePoint a la base de conocimiento

Agregue orígenes de datos basados en la nube de SharePoint protegidos a la base de conocimiento para enriquecerla con preguntas y respuestas que se pueden proteger con Azure Active Directory.

Cuando se agrega un documento protegido de SharePoint a la base de conocimiento, como el Administrador de QnA Maker, debe solicitar permiso de Active Directory para QnA Maker. Una vez que se concede este permiso desde el administrador de Active Directory a QnA Maker para acceder a SharePoint, no se tiene que conceder otra vez. Las adiciones posteriores de documentos a la base de conocimiento no necesitarán autorización si se encuentran en el mismo recurso de SharePoint.

Si el administrador de base de conocimiento de QnA Maker no es el administrador de Active Directory, deberá comunicar con el administrador de Active Directory para finalizar este proceso.

## <a name="prerequisites"></a>Prerrequisitos

* SharePoint basado en la nube: QnA Maker usa Microsoft Graph para los permisos. Si SharePoint es local, no podrá realizar extracciones de SharePoint, porque Microsoft Graph no podrá determinar los permisos.
* Formato de dirección URL: QnA Maker solo admite direcciones URL de SharePoint que se generan para compartir y tienen el formato `https://\*.sharepoint.com`

## <a name="add-supported-file-types-to-knowledge-base"></a>Adición de tipos de archivo compatibles a la base de conocimiento

Puede agregar todos los [tipos de archivos](../index.yml) compatibles con QnA Maker desde un sitio de SharePoint a la base de conocimiento. Es posible que tenga que conceder [permisos](#permissions) si el recurso de archivo está protegido.

1. Desde la biblioteca con el sitio de SharePoint, seleccione el menú del botón de puntos suspensivos del archivo, `...`.
1. Copie la dirección URL del archivo.

   ![Obtenga la dirección URL del archivo SharePoint seleccionando el menú del botón de puntos suspensivos del archivo y luego copiando la dirección URL.](../media/add-sharepoint-datasources/get-sharepoint-file-url.png)

1. En el portal de QnA Maker, en la página **Configuración**, agregue la dirección URL a la base de conocimiento.

### <a name="images-with-sharepoint-files"></a>Imágenes con archivos de SharePoint

Si los archivos incluyen imágenes, estas no se extraen. Puede agregar la imagen, desde el portal de QnA Maker, después de que se haya extraído el archivo en pares de QnA.

Agregue la imagen con la siguiente sintaxis de Markdown:

```markdown
![Explanation or description of image](URL of public image)
```

El texto que hay entre corchetes, `[]`, explica la imagen. La dirección URL que hay entre paréntesis, `()`, es el vínculo directo a la imagen.

Cuando se prueba el par de QnA en el panel de prueba interactiva, en el portal de QnA Maker se muestra la imagen, en lugar del texto de Markdown. Esto valida que la imagen se puede recuperar públicamente desde la aplicación cliente.

## <a name="permissions"></a>Permisos

La concesión de permisos se produce cuando se agrega un archivo protegido desde un servidor que ejecuta SharePoint a una base de conocimiento. Según la configuración de SharePoint y los permisos de la persona que agrega el archivo, esto podría requerir lo siguiente:

* Ningún paso adicional: la persona que agrega el archivo cuenta con todos los permisos necesarios.
* Los pasos de [administrador de la base de conocimiento](#knowledge-base-manager-add-sharepoint-data-source-in-qna-maker-portal) y [administrador de Active Directory](#active-directory-manager-grant-file-read-access-to-qna-maker).

Consulte los pasos que aparecen debajo.

### <a name="knowledge-base-manager-add-sharepoint-data-source-in-qna-maker-portal"></a>Administrador de base de conocimiento: adición de origen de datos de SharePoint en el portal de QnA Maker

Cuando el **administrador de QnA Maker** agrega un documento protegido de SharePoint a una base de conocimiento, el administrador de la base de conocimiento inicia una solicitud de permiso que debe completar el administrador de Active Directory.

La solicitud empieza con un elemento emergente para autenticarse en una cuenta de Active Directory.

![Autenticación de una cuenta de usuario](../media/add-sharepoint-datasources/authenticate-user-account.png)

Una vez que el administrador de QnA Maker selecciona la cuenta, el administrador de Azure Active Directory recibirá un aviso por el que necesitan permitir el acceso de la aplicación de QnA Maker (no del administrador de QnA Maker) al recurso de SharePoint. El administrador de Azure Active Directory debe hacer esto para todos los recursos de SharePoint, pero no para todos los documentos de ese recurso.

### <a name="active-directory-manager-grant-file-read-access-to-qna-maker"></a>Administrador de Active Directory: concesión de acceso de lectura de archivo a QnA Maker

El administrador de Active Directory (no el de QnA Maker) necesita conceder acceso a QnA Maker para poder acceder al recurso de SharePoint seleccionando [este vínculo](https://login.microsoftonline.com/common/oauth2/v2.0/authorize?response_type=id_token&scope=Files.Read%20Files.Read.All%20Sites.Read.All%20User.Read%20User.ReadBasic.All%20profile%20openid%20email&client_id=c2c11949-e9bb-4035-bda8-59542eb907a6&redirect_uri=https%3A%2F%2Fwww.qnamaker.ai%3A%2FCreate&state=68) con el fin de autorizar a la aplicación empresarial Portal SharePoint de QnA Maker los permisos de lectura de archivo.

![El administrador de Azure Active Directory concede permisos de forma interactiva.](../media/add-sharepoint-datasources/aad-manager-grants-permission-interactively.png)

<!--
The Active Directory manager must grant QnA Maker access either by application name, `QnAMakerPortalSharePoint`, or by application ID, `c2c11949-e9bb-4035-bda8-59542eb907a6`.
-->
<!--
### Grant access from the interactive pop-up window

The Active Directory manager will get a pop-up window requesting permissions to the `QnAMakerPortalSharePoint` app. The pop-up window includes the QnA Maker Manager email address that initiated the request, an `App Info` link to learn more about **QnAMakerPortalSharePoint**, and a list of permissions requested. Select **Accept** to provide those permissions.

![Azure Active Directory manager grants permission interactively](../media/add-sharepoint-datasources/aad-manager-grants-permission-interactively.png)
-->
<!--

### Grant access from the App Registrations list

1. The Active Directory manager signs in to the Azure portal and opens **[App registrations list](https://ms.portal.azure.com/#blade/Microsoft_AAD_IAM/ApplicationsListBlade)**.

1. Search for and select the **QnAMakerPortalSharePoint** app. Change the second filter box from **My apps** to **All apps**. The app information will open on the right side.

    ![Select QnA Maker app in App registrations list](../media/add-sharepoint-datasources/select-qna-maker-app-in-app-registrations.png)

1. Select **Settings**.

    [![Select Settings in the right-side blade](../media/add-sharepoint-datasources/select-settings-for-qna-maker-app-registration.png)](../media/add-sharepoint-datasources/select-settings-for-qna-maker-app-registration.png#lightbox)

1. Under **API access**, select **Required permissions**.

    ![Select 'Settings', then under 'API access', select 'Required permission'](../media/add-sharepoint-datasources/select-required-permissions-in-settings-blade.png)

1. Do not change any settings in the **Enable Access** window. Select **Grant Permission**.

    [![Under 'Grant Permission', select 'Yes'](../media/add-sharepoint-datasources/grant-app-required-permissions.png)](../media/add-sharepoint-datasources/grant-app-required-permissions.png#lightbox)

1. Select **YES** in the pop-up confirmation windows.

    ![Grant required permissions](../media/add-sharepoint-datasources/grant-required-permissions.png)
-->
### <a name="grant-access-from-the-azure-active-directory-admin-center"></a>Concesión de acceso desde el centro de administración de Azure Active Directory

1. El administrador de Active Directory inicia sesión en Azure Portal y abre **[Aplicaciones empresariales](https://aad.portal.azure.com/#blade/Microsoft_AAD_IAM/StartboardApplicationsMenuBlade/AllApps)** .

1. Busque `QnAMakerPortalSharePoint` para seleccionar la aplicación de QnA Maker.

    [![Búsqueda de QnAMakerPortalSharePoint en la lista de Aplicaciones empresariales](../media/add-sharepoint-datasources/search-enterprise-apps-for-qna-maker.png)](../media/add-sharepoint-datasources/search-enterprise-apps-for-qna-maker.png#lightbox)

1. En **Seguridad**, vaya a **Permisos**. Seleccione **Conceder consentimiento del administrador para la organización**.

    [![Selección del usuario autenticado para el administrador de Active Directory](../media/add-sharepoint-datasources/grant-aad-permissions-to-enterprise-app.png)](../media/add-sharepoint-datasources/grant-aad-permissions-to-enterprise-app.png#lightbox)

1. Seleccione una cuenta de inicio de sesión con permisos para conceder permisos de Active Directory.




## <a name="add-sharepoint-data-source-with-apis"></a>Adición de un origen de datos de SharePoint con API

Existe una solución alternativa para agregar el contenido de SharePoint más reciente mediante la API con Azure Blob Storage, a continuación se indican los pasos: 
1.  Descargue los archivos de SharePoint localmente. El usuario que llama a la API debe tener acceso a SharePoint. 
1.  Cárguelos en Azure Blob Storage. Esto creará un acceso compartido seguro [mediante el token de SAS](../../../storage/common/storage-sas-overview.md#how-a-shared-access-signature-works). 
1. Pase la dirección URL del blob generada con el token de SAS a la API de QnA Maker. Para permitir la extracción de respuestas a preguntas de los archivos, debe agregar el tipo de archivo de sufijo como "&ext=pdf" o "&ext=doc" al final de la dirección URL antes de pasarla a la API de QnA Maker.


<!--
## Get SharePoint File URI

Use the following steps to transform the SharePoint URL into a sharing token.

1. Encode the URL using [base64](https://en.wikipedia.org/wiki/Base64).

1. Convert the base64-encoded result to an unpadded base64url format with the following character changes.

    * Remove the equal character, `=` from the end of the value.
    * Replace `/` with `_`.
    * Replace `+` with `-`.
    * Append `u!` to be beginning of the string.

1. Sign in to Graph explorer and run the following query, where `sharedURL` is ...:

    ```
    https://graph.microsoft.com/v1.0/shares/<sharedURL>/driveitem
    ```

    Get the **@microsoft.graph.downloadUrl** and use this as `fileuri` in the QnA Maker APIs.

### Add or update a SharePoint File URI to your knowledge base

Use the **@microsoft.graph.downloadUrl** from the previous section as the `fileuri` in the QnA Maker API for [adding a knowledge base](/rest/api/cognitiveservices/qnamaker4.0/knowledgebase) or [updating a knowledge base](/rest/api/cognitiveservices/qnamaker/knowledgebase/update). The following fields are mandatory: name, fileuri, filename, source.

```
{
    "name": "Knowledge base name",
    "files": [
        {
            "fileUri": "<@microsoft.graph.downloadURL>",
            "fileName": "filename.xlsx",
            "source": "<SharePoint link>"
        }
    ],
    "urls": [],
    "users": [],
    "hostUrl": "",
    "qnaList": []
}
```



## Remove QnA Maker app from SharePoint authorization

1. Use the steps in the previous section to find the Qna Maker app in the Active Directory admin center.
1. When you select the **QnAMakerPortalSharePoint**, select **Overview**.
1. Select **Delete** to remove permissions.

-->

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Colaboración en su base de conocimiento](../index.yml)
