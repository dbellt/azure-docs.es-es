---
title: Creación de un runbook de Python 3 (versión preliminar) en Azure Automation
description: En este artículo se muestra cómo crear, probar y publicar un sencillo runbook de Python 3 (versión preliminar) en la cuenta de Azure Automation.
services: automation
ms.subservice: process-automation
ms.date: 04/28/2021
ms.topic: tutorial
ms.custom: has-adal-ref, devx-track-python
ms.openlocfilehash: 4badf99bcae4fa6d32492960f3e5322ee3508f25
ms.sourcegitcommit: a5dd9799fa93c175b4644c9fe1509e9f97506cc6
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108205060"
---
# <a name="tutorial-create-a-python-3-runbook-preview"></a>Tutorial: Creación de un runbook de Python 3 (versión preliminar)

Este tutorial le guiará por la creación de un [runbook de Python 3](../automation-runbook-types.md#python-runbooks) (versión preliminar) en Azure Automation. Los runbooks utilizan Python 2 y Python 3. Puede modificar directamente el código del runbook con el editor de texto en Azure Portal.

> [!div class="checklist"]
> * Creación de un runbook de Python sencillo
> * Prueba y publicación del runbook
> * Ejecución y seguimiento del estado del trabajo de runbook
> * Actualización del runbook para iniciar una máquina virtual de Azure con parámetros de runbook

## <a name="prerequisites"></a>Prerrequisitos

Para completar este tutorial, necesitará lo siguiente:

- Suscripción de Azure. Si aún no tiene ninguna, puede [activar las ventajas de la suscripción a MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) o suscribirse para obtener una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

- [Cuenta de Automation](../automation-security-overview.md) para contener el Runbook y autenticarse en recursos de Azure. Esta cuenta debe tener permiso para iniciar y detener la máquina virtual. En este tutorial se necesita la [cuenta de ejecución](../automation-security-overview.md#run-as-accounts). 

- Una máquina virtual de Azure. Durante este tutorial iniciará y detendrá esta máquina, por lo que no debe ser una máquina virtual de producción.

## <a name="create-a-new-runbook"></a>Crear un runbook

Empieza creando un runbook simple cuya salida sea el texto *Hola mundo*.

1. En Azure Portal, abra su cuenta de Automation.

    La página de la cuenta de Automation proporciona una vista rápida de los recursos que hay en esa cuenta. Ya debería tener algunos recursos. Muchas de esos recursos son los módulos que se incluyen automáticamente en una cuenta nueva de Automation. También debe tener el recurso de credencial Cuenta de ejecución que se menciona en los [requisitos previos](#prerequisites).

2. En **Automatización de procesos**, haga clic en **Runbooks** para abrir la lista de runbooks.

3. Seleccione **Agregar un Runbook** para crear un runbook.

4. Asigne al runbook el nombre **MyFirstRunbook-Python**.

5. Seleccione **Python 3** para **Tipo de Runbook**.

6. Seleccione **Crear** para crear el runbook y abra el editor de texto.

## <a name="add-code-to-the-runbook"></a>Agregar código al runbook

Ahora, agregue un comando simple para imprimir el texto `Hello World`.

```python
print("Hello World!")
```

Seleccione **Guardar** para guardar el runbook.

## <a name="test-the-runbook"></a>Probar el runbook

Antes de publicar el runbook para que esté disponible en producción, puede que quiera probarlo para asegurarse de que funciona correctamente. Al probar un runbook, se ejecuta el borrador y se visualiza la salida de forma interactiva.

1. Seleccione **Panel de prueba** para abrir el panel **Prueba**.

2. Seleccione **Iniciar** para iniciar la prueba. Esta debe ser la única opción habilitada.

3. Se crea un [trabajo de runbook](../automation-runbook-execution.md) y se muestra su estado.
   El estado del trabajo se inicia como **En cola**, lo que indica que está esperando a que haya disponible un trabajo de runbook en la nube. Su estado cambia a **Iniciando** cuando un trabajo de runbook solicita el trabajo. Cuando el runbook comienza a ejecutarse realmente, el estado es **En ejecución**.

4. Cuando se complete el trabajo del runbook, se mostrará su resultado. En este caso, debería ver `Hello World`.

5. Cierre el panel **Prueba** para volver al lienzo.

## <a name="publish-and-start-the-runbook"></a>Publicar e iniciar el runbook

El runbook que acaba de crear aún está en modo de borrador. Tiene que publicarlo antes de que pueda ejecutarlo en producción. Al publicar un runbook, se sobrescribe la versión publicada existente con la versión de borrador. En este caso, no tiene una versión publicada aún porque acaba de crear el runbook.

1. Seleccione **Publicar** para publicar el runbook y **Sí** cuando se le solicite.

2. Si se desplaza a la izquierda para ver el runbook en la página **Runbooks**, verá **Publicado** en **Estado de creación**.

3. Desplácese de nuevo a la derecha para ver el panel de **MyFirstRunbook-Python3**.

   Las opciones en la parte superior le permiten iniciar el runbook, verlo o programarlo para que se inicie en algún momento en el futuro.

4. Seleccione **Iniciar** y **Aceptar** cuando se abra el panel **Iniciar Runbook**.

5. Se abre un panel **Trabajo** para el trabajo de runbook que acaba de crear. Puede cerrar este panel, pero déjelo abierto por ahora para que pueda ver el progreso del trabajo.

6. El estado del trabajo se muestra en **Resumen del trabajo** y coincide con los estados que vio cuando probó el runbook.

7. Cuando el estado del runbook aparezca como **Completado**, seleccione **Salida**. Se abre el panel **Salida**, donde verá `Hello World`.

8. Cierre el panel **Salida**.

9. Seleccione **Todos los registros** para abrir el panel **Transmisiones** para el trabajo de runbook. Solo debería ver `Hello World` en el flujo Salida. No obstante, en este panel se pueden mostrar otras transmisiones de un trabajo de runbook, como **Error** y **Modo detallado**, si el runbook escribe en ellas.

10. Cierre el panel **Transmisiones** y **Trabajo** para volver al panel de **MyFirstRunbook-Python3**.

11. Seleccione **Trabajos** para abrir la página **Trabajos** de este runbook. Esta página enumera todos los trabajos creados por este runbook. Solo debería ver un trabajo en la lista ya que solo ejecutó el trabajo una vez.

12. Puede seleccionar este trabajo para abrir el mismo panel **Trabajo** que vio al iniciar el runbook. Esta panel permite volver atrás en el tiempo y ver los detalles de cualquier trabajo que se creó para un runbook determinado.

## <a name="add-authentication-to-manage-azure-resources"></a>Agregar autenticación para administrar recursos de Azure

Ha probado y publicado su runbook, pero hasta ahora no hace nada útil. Quiere que administre recursos de Azure.
Para ello, el script debe autenticarse mediante las credenciales de la cuenta de ejecución de la cuenta de Automation.

> [!NOTE]
> La cuenta de Automation se debe haber creado con la característica de ejecución para que haya un certificado de ejecución.
> Si la cuenta de Automation no se ha creado con la cuenta de ejecución, se puede autenticar como se describe en [Autenticación con las bibliotecas de administración de Azure para Python](/azure/python/python-sdk-azure-authenticate) o [Creación de una cuenta de ejecución](../create-run-as-account.md).

1. Seleccione **Editar** en el panel **MyFirstRunbook-Python3** para abrir el editor de texto.

2. Agregue el siguiente código para autenticarse en Azure:

    ```python
    import os
    from azure.mgmt.compute import ComputeManagementClient
    import azure.mgmt.resource
    import automationassets
    
    def get_automation_runas_credential(runas_connection):
        from OpenSSL import crypto
        import binascii
        from msrestazure import azure_active_directory
        import adal
    
        # Get the Azure Automation RunAs service principal certificate
        cert = automationassets.get_automation_certificate("AzureRunAsCertificate")
        pks12_cert = crypto.load_pkcs12(cert)
        pem_pkey = crypto.dump_privatekey(crypto.FILETYPE_PEM,pks12_cert.get_privatekey())
    
        # Get run as connection information for the Azure Automation service principal
        application_id = runas_connection["ApplicationId"]
        thumbprint = runas_connection["CertificateThumbprint"]
        tenant_id = runas_connection["TenantId"]
    
        # Authenticate with service principal certificate
        resource ="https://management.core.windows.net/"
        authority_url = ("https://login.microsoftonline.com/"+tenant_id)
        context = adal.AuthenticationContext(authority_url)
        return azure_active_directory.AdalAuthentication(
        lambda: context.acquire_token_with_client_certificate(
                resource,
                application_id,
                pem_pkey,
                thumbprint)
        )
    
    # Authenticate to Azure using the Azure Automation RunAs service principal
    runas_connection = automationassets.get_automation_connection("AzureRunAsConnection")
    azure_credential = get_automation_runas_credential(runas_connection)
    ```

## <a name="add-code-to-create-python-compute-client-and-start-the-vm"></a>Agregar código para crear un cliente de Compute de Python e iniciar la máquina virtual

Para trabajar con máquinas virtuales de Azure, cree una instancia del [cliente de Azure Compute para Python](/python/api/azure-mgmt-compute/azure.mgmt.compute.computemanagementclient).

Use el cliente de Compute para iniciar la máquina virtual. Agregue el siguiente código al runbook:

```python
# Initialize the compute management client with the Run As credential and specify the subscription to work against.
compute_client = ComputeManagementClient(
    azure_credential,
    str(runas_connection["SubscriptionId"])
)


print('\nStart VM')
async_vm_start = compute_client.virtual_machines.start(
    "MyResourceGroup", "TestVM")
async_vm_start.wait()
```

Donde `MyResourceGroup` es el nombre del grupo de recursos que contiene la máquina virtual y `TestVM` es el nombre de la máquina virtual que quiere iniciar.

Pruebe y ejecute el runbook de nuevo para constatar que inicia la máquina virtual.

## <a name="use-input-parameters"></a>Usar parámetros de entrada

Actualmente, el runbook usa valores codificados de forma rígida en los nombres del grupo de recursos y de la máquina virtual. Ahora vamos a agregar código para obtener estos valores de parámetros de entrada.

Use la variable `sys.argv` para obtener los valores de parámetro. Agregue el siguiente código al runbook, inmediatamente después de las demás instrucciones `import`:

```python
import sys

resource_group_name = str(sys.argv[1])
vm_name = str(sys.argv[2])
```

Con esta operación se importa el módulo `sys` y se crean dos variables para contener los nombres del grupo de recursos y de la máquina virtual. Observe que el elemento de la lista de argumentos, `sys.argv[0]`, es el nombre del script y no lo especifica el usuario.

Ahora, puede modificar las dos últimas líneas del runbook para usar los valores de parámetro de entrada en lugar de los valores codificados de forma rígida:

```python
async_vm_start = compute_client.virtual_machines.start(
    resource_group_name, vm_name)
async_vm_start.wait()
```

Al iniciar un runbook de Python (ya sea en el panel **Prueba** o como runbook publicado), se pueden especificar los valores de los parámetros en la página **Iniciar Runbook**, en **Parámetros**.

Cuando empiece a escribir un valor en el primer cuadro, aparecerá un segundo cuadro (y así sucesivamente) para que pueda especificar varios valores de parámetro si es necesario.

Los valores estarán disponibles para el script en la matriz `sys.argv` como en el código que se acaba de agregar.

Escriba el nombre del grupo de recursos como valor del primer parámetro y el nombre de la máquina virtual que se va a iniciar como valor del segundo parámetro.

![Escribir valores de parámetro](../media/automation-tutorial-runbook-textual-python/runbook-python-params.png)

Seleccione **Aceptar** para iniciar el runbook. El runbook se ejecuta e inicia la máquina virtual que se ha especificado.

## <a name="error-handling-in-python&quot;></a>Control de errores en Python

También puede usar las siguientes convenciones para recuperar varios flujos de los runbooks de Python, como los flujos WARNING, ERROR y DEBUG.

```python
print(&quot;Hello World output")
print("ERROR: - Hello world error")
print("WARNING: - Hello world warning")
print("DEBUG: - Hello world debug")
print("VERBOSE: - Hello world verbose")
```

En el ejemplo siguiente se muestra esta convención utilizada en un bloque `try...except`.

```python
try:
    raise Exception('one', 'two')
except Exception as detail:
    print ('ERROR: Handling run-time error:', detail)
```

## <a name="next-steps"></a>Pasos siguientes

- Para más información sobre los tipos de Runbook, sus ventajas y sus limitaciones, consulte [Tipos de runbooks de Azure Automation](../automation-runbook-types.md).

- Para más información sobre cómo desarrollar para Azure con Python, vea [Azure para desarrolladores de Python](/azure/python/).

- Para ver runbooks de ejemplo de Python 3, consulte el repositorio [GitHub de Azure Automation](https://github.com/azureautomation/runbooks/tree/master/Utility/Python).
