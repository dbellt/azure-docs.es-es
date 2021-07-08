---
title: 'Inicio rápido: Creación de una cuenta de Purview mediante Python'
description: Creación de una cuenta de Azure Purview mediante Python
author: nayenama
ms.author: nayenama
ms.service: purview
ms.subservice: purview-data-catalog
ms.devlang: python
ms.topic: quickstart
ms.date: 04/02/2021
ms.openlocfilehash: ec1f8890d9626ca1ed96f538bd464da7c30caf17
ms.sourcegitcommit: 7c44970b9caf9d26ab8174c75480f5b09ae7c3d7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/27/2021
ms.locfileid: "112983795"
---
# <a name="quickstart-create-a-purview-account-using-python"></a>Inicio rápido: Creación de una cuenta de Purview mediante Python

En este inicio rápido, creará una cuenta de Purview con Python. 

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta de Azure con una suscripción activa. [Cree una cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Su propio [inquilino de Azure Active Directory](../active-directory/fundamentals/active-directory-access-create-new-tenant.md).

* La cuenta debe tener permiso para crear recursos en la suscripción

* Si **Azure Policy** impide a todas las aplicaciones crear una **cuenta de Storage** y un **espacio de nombres de EventHub**, debe crear una excepción de directiva mediante etiqueta, que se especificará durante el proceso de creación de una cuenta de Purview. La razón principal es que con cada cuenta de Purview creada se debe crear un grupo de recursos administrado y dentro de este, una cuenta de Storage y un espacio de nombres de EventHub. Para más información, consulte [Creación del portal del catálogo](create-catalog-portal.md).


## <a name="install-the-python-package"></a>Instalar el paquete de Python

1. Abra un terminal o símbolo del sistema con privilegios de administrador. 
2. En primer lugar, instale el paquete de Python para los recursos de administración de Azure:

    ```python
    pip install azure-mgmt-resource
    ```
3. Para instalar el paquete de Python para Purview, ejecute el siguiente comando:

    ```python
    pip install azure-mgmt-purview
    ```

    El [SDK de Python para Purview](https://github.com/Azure/azure-sdk-for-python) admite Python 2.7, 3.3, 3.4, 3.5, 3.6 y 3.7.

4. Para instalar el paquete de Python para la autenticación de Azure Identity, ejecute el siguiente comando:

    ```python
    pip install azure-identity
    ```
    > [!NOTE] 
    > El paquete "azure-identity" puede tener conflictos con "azure-cli" en algunas dependencias comunes. Si encuentra algún problema de autenticación, elimine el paquete "azure-cli" y sus dependencias, o bien use un equipo limpio sin una instalación del paquete "azure-cli" para que funcione.
    
## <a name="create-a-purview-client"></a>Creación de un cliente de Purview

1. Cree un archivo llamado **purview.py**. Agregue las siguientes instrucciones para agregar referencias a espacios de nombres.

    ```python
    from azure.identity import ClientSecretCredential 
    from azure.mgmt.resource import ResourceManagementClient
    from azure.mgmt.purview import PurviewManagementClient
    from azure.mgmt.purview.models import *
    from datetime import datetime, timedelta
    import time
    ```

2. Agregue el código siguiente al método **Main**, que crea una instancia de la clase PurviewManagementClient. Este objeto se usa para crear una cuenta de Purview, eliminarla, comprobar la disponibilidad de los nombres y otras operaciones del proveedor de recursos.
 
    ```python
    def main():
    
    # Azure subscription ID
    subscription_id = '<subscription ID>'
    
    # This program creates this resource group. If it's an existing resource group, comment out the code that creates the resource group
    rg_name = '<resource group>'

    # The purview name. It must be globally unique.
    purview_name = '<purview account name>'

    # Location name, where Purview account must be created.
    location = '<location name>'    

    # Specify your Active Directory client ID, client secret, and tenant ID
    credentials = ClientSecretCredential(client_id='<service principal ID>', client_secret='<service principal key>', tenant_id='<tenant ID>') 
    # resource_client = ResourceManagementClient(credentials, subscription_id)
    purview_client = PurviewManagementClient(credentials, subscription_id)
    ```

## <a name="create-a-purview-account"></a>Creación de una cuenta de Purview

1. Agregue el código siguiente al método **Main**, que crea una **cuenta de Purview**. Si ya existe el grupo de recursos, convierta en comentario la primera instrucción `create_or_update`.

   ```python
    # create the resource group
    # comment out if the resource group already exits
    resource_client.resource_groups.create_or_update(rg_name, rg_params)

    #Create a purview
    identity = Identity(type= "SystemAssigned")
    sku = AccountSku(name= 'Standard', capacity= 4)
    purview_resource = Account(identity=identity,sku=sku,location =location )
       
    try:
        pa = (purview_client.accounts.begin_create_or_update(rg_name, purview_name, purview_resource)).result()
        print("location:", pa.location, " Purview Account Name: ", pa.name, " Id: " , pa.id ," tags: " , pa.tags)  
    except:
        print("Error")
        print_item(pa)
 
    while (getattr(pa,'provisioning_state')) != "Succeeded" :
        pa = (purview_client.accounts.get(rg_name, purview_name))  
        print(getattr(pa,'provisioning_state'))
        if getattr(pa,'provisioning_state') != "Failed" :
            print("Error in creating Purview account")
            break
        time.sleep(30)      
      ```

2. Ahora, agregue la siguiente instrucción para invocar el método **Main** cuando se ejecute el programa:

   ```python
   # Start the main method
   main()
   ```

## <a name="full-script"></a>Script completo

Este es el código de Python completo:

```python
    
    from azure.identity import ClientSecretCredential 
    from azure.mgmt.resource import ResourceManagementClient
    from azure.mgmt.purview import PurviewManagementClient
    from azure.mgmt.purview.models import *
    from datetime import datetime, timedelta
    import time
    
     # Azure subscription ID
    subscription_id = '<subscription ID>'
    
    # This program creates this resource group. If it's an existing resource group, comment out the code that creates the resource group
    rg_name = '<resource group>'

    # The purview name. It must be globally unique.
    purview_name = '<purview account name>'

    # Specify your Active Directory client ID, client secret, and tenant ID
    credentials = ClientSecretCredential(client_id='<service principal ID>', client_secret='<service principal key>', tenant_id='<tenant ID>') 
    # resource_client = ResourceManagementClient(credentials, subscription_id)
    purview_client = PurviewManagementClient(credentials, subscription_id)
    
    # create the resource group
    # comment out if the resource group already exits
    resource_client.resource_groups.create_or_update(rg_name, rg_params)

    #Create a purview
    identity = Identity(type= "SystemAssigned")
    sku = AccountSku(name= 'Standard', capacity= 4)
    purview_resource = Account(identity=identity,sku=sku,location ="southcentralus" )
       
    try:
        pa = (purview_client.accounts.begin_create_or_update(rg_name, purview_name, purview_resource)).result()
        print("location:", pa.location, " Purview Account Name: ", purview_name, " Id: " , pa.id ," tags: " , pa.tags) 
    except:
        print("Error in submitting job to create account")
        print_item(pa)
 
    while (getattr(pa,'provisioning_state')) != "Succeeded" :
        pa = (purview_client.accounts.get(rg_name, purview_name))  
        print(getattr(pa,'provisioning_state'))
        if getattr(pa,'provisioning_state') != "Failed" :
            print("Error in creating Purview account")
            break
        time.sleep(30)    

# Start the main method
main()
```

## <a name="run-the-code"></a>Ejecución del código

Compile e inicie la aplicación y, a continuación, compruebe la ejecución de la canalización.

La consola imprime el progreso de la creación de la factoría de datos, el servicio vinculado, los conjuntos de datos, la canalización y la ejecución de canalización. Espere hasta que vea los detalles de ejecución de actividad de copia con el tamaño de los datos leídos/escritos. A continuación, use herramientas como [Explorador de Azure Storage](https://azure.microsoft.com/features/storage-explorer/) para comprobar que los blobs se copian a "outputBlobPath" desde "inputBlobPath", como se especificó en las variables.

Este es la salida de ejemplo:

```console
location: southcentralus  Purview Account Name:  purviewpython7  Id:  /subscriptions/8c2c7b23-848d-40fe-b817-690d79ad9dfd/resourceGroups/Demo_Catalog/providers/Microsoft.Purview/accounts/purviewpython7  tags:  None
Creating
Creating
Succeeded
```

## <a name="verify-the-output"></a>Comprobación del resultado

Vaya a la página **Purview accounts** (Cuentas de Purview) en Azure Portal y compruebe la cuenta creada con el código anterior. 

## <a name="delete-purview-account"></a>Eliminación de la cuenta de Purview

Para eliminar la cuenta de Purview, agregue el siguiente código al programa:

```python
pa = purview_client.accounts.begin_delete(rg_name, purview_name).result()
```

## <a name="next-steps"></a>Pasos siguientes

En el código de este tutorial se crea y se elimina una cuenta de Purview. Ahora puede descargar el SDK de Python y conocer otras acciones del proveedor de recursos que puede realizar para una cuenta de Purview.

Vaya al siguiente artículo, donde aprenderá a permitir que los usuarios accedan a su cuenta de Azure Purview. 

> [!div class="nextstepaction"]
> [Adición de usuarios a una cuenta de Azure Purview](catalog-permissions.md)
