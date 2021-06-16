---
title: 'Restricción del acceso a los recursos de PaaS (tutorial): Azure Portal'
description: En este tutorial aprenderá a limitar y restringir el acceso de la red a los recursos de Azure, como Azure Storage y Azure SQL Database, con puntos de conexión de servicio de red virtual mediante Azure Portal.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
manager: mtillman
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: virtual-network
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 12/11/2020
ms.author: kumud
ms.openlocfilehash: 789816e37c3aaf9678fd9cb87ff6362442709498
ms.sourcegitcommit: 3bb9f8cee51e3b9c711679b460ab7b7363a62e6b
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/14/2021
ms.locfileid: "112082215"
---
# <a name="tutorial-restrict-network-access-to-paas-resources-with-virtual-network-service-endpoints-using-the-azure-portal"></a>Tutorial: Restricción del acceso de la red a los recursos de PaaS mediante puntos de conexión de servicio de red virtual mediante Azure Portal.

Los puntos de conexión de servicio de red virtual permiten que el acceso de la red a algunos recursos de servicio de Azure esté restringido a una subred de la red virtual. También se puede quitar el acceso de Internet a los recursos. Los puntos de conexión de servicio proporcionan a la red virtual conexión directa con los servicios de Azure compatibles, de modo que se puede usar el espacio de direcciones privadas de la red virtual para acceder a los servicios de Azure. El tráfico destinado a los recursos de Azure a través de los puntos de conexión de servicio siempre se mantiene en la red troncal de Microsoft Azure. En este tutorial, aprenderá a:

> [!div class="checklist"]
> * Crear una red virtual con una subred
> * Agregar una subred y habilitar un punto de conexión de servicio
> * Crear un recurso de Azure y permitir que la red solo pueda acceder a él desde una subred
> * Implementar una máquina virtual en cada subred
> * Confirmar el acceso a un recurso desde una subred
> * Confirmar que se ha denegado el acceso a un recurso desde una subred e Internet

Si lo prefiere, puede completar este tutorial con la [CLI de Azure](tutorial-restrict-network-access-to-resources-cli.md) o [Azure PowerShell](tutorial-restrict-network-access-to-resources-powershell.md).

Si no tiene una suscripción a Azure, cree una [cuenta gratuita](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) antes de empezar.

## <a name="log-in-to-azure"></a>Inicio de sesión en Azure

Inicie sesión en Azure Portal en https://portal.azure.com.

## <a name="create-a-virtual-network"></a>Creación de una red virtual

1. Seleccione **+ Crear un recurso** en la esquina superior izquierda de Azure Portal.
2. Seleccione **Redes** y, después, **Redes virtuales**.
3. Haga clic en **+ Agregar** y escriba la siguiente información: 

   |Configuración|Valor|
   |----|----|
   |Subscription| Seleccione su suscripción.|
   |Resource group | Haga clic en **Crear nuevo** y escriba *myResourceGroup*.|
   |Nombre| Escriba *myVirtualNetwork* |
   |Region| Seleccione **(EE.UU.) Este de EE. UU.** |

   ![Especificar información básica acerca de la red virtual](./media/tutorial-restrict-network-access-to-resources/create-virtual-network.png)

4. Haga clic en **Siguiente: direcciones IP >**
   
   |Configuración|Value|
   |----|----|
   |Espacio de IPv4Address| Deje el valor predeterminado |
   |Nombre de subred| Haga clic en **predeterminada** y cambie el nombre de "default" (predeterminada) a "Public" (Pública).|
   |Intervalo de direcciones de subred| Deje el valor predeterminado|

5. Haga clic en **Siguiente: seguridad >** 
   
   |Configuración|Value|
   |----|----|   
   |BastionHost| Deshabilitar|
   |Protección contra DDOS| Deshabilitar|
   |Firewall| Deshabilitar|

6. Al finalizar, haga clic en **Revisar y crear**.
7. Si se superan las comprobaciones de validación, haga clic en **Crear**.
8. Espere a que finalice la implementación y haga clic en **Ir al recurso** o pase a la sección siguiente. 

## <a name="enable-a-service-endpoint"></a>Habilitación de un punto de conexión de servicio

Los punto de conexión de servicio están habilitados por servicio, por subred. Para crear una subred y habilitar un punto de conexión de servicio para ella:

1. Si aún no está en la página de recursos de la red virtual, puede buscar la red recién creada en el cuadro **Buscar recursos, servicios y documentos** de la parte superior del portal, escribir *myVirtualNetwork* y seleccionarlo en la lista.
2. En el menú **Configuración** (izquierda), seleccione **Subredes** y, después, seleccione **+ Subred**, tal como se muestra:

    ![Subred agregada](./media/tutorial-restrict-network-access-to-resources/add-subnet.png) 

3. En **Agregar subred**, escriba o seleccione los siguientes datos y haga clic en **Aceptar**:

    |Configuración|Valor|
    |----|----|
    |Nombre| Privada |
    |Intervalo de direcciones| Deje el valor predeterminado|
    |Puntos de conexión del servicio| Seleccione **Microsoft.Storage**|
    |Directivas de puntos de conexión de servicio | Deje el valor predeterminado como 0 |

> [!CAUTION]
> Antes de habilitar un punto de conexión de servicio para una subred existente que tenga recursos, consulte [Modificación de la configuración de subred](virtual-network-manage-subnet.md#change-subnet-settings).

4. Haga clic en **Guardar** y, después, cierre la ventana Subred a la derecha. La subred recién creada debería aparecer en la lista.

## <a name="restrict-network-access-for-a-subnet"></a>Restricción del acceso de la red para una subred

De forma predeterminada, todas las instancias de máquina virtual pueden comunicarse con todos los recursos. La comunicación con todos los recursos de una subred se puede limitar mediante la creación de un grupo de seguridad de red y su posterior asociación a la subred:

1. En la esquina superior izquierda de Azure Portal, seleccione **Todos los servicios**.
2. Seleccione **Redes** y, después, seleccione (o busque) **Grupos de seguridad de red**.
3. En la página **Grupos de seguridad de red**, haga clic en **+ Agregar**.
4. Escriba la siguiente información 

    |Configuración|Valor|
    |----|----|
    |Subscription| Seleccione su suscripción.|
    |Resource group | Seleccione *myResourceGroup* en la lista.|
    |Nombre| Escriba **myNsgPrivate**. |
    |Location| Seleccione **Este de EE. UU**. |

5. Haga clic en **Revisar y crear** y, cuando se supere la comprobación de validación, haga clic en **Crear**.
6. Una vez creado el grupo de seguridad de red, haga clic en **Ir al recurso** o busque *myNsgPrivate*.
7. En el menú **Configuración** de la izquierda, seleccione **Reglas de seguridad de salida**.
8. Seleccione **+Agregar**.
9. Cree una regla que permita la comunicación saliente con el servicio Azure Storage. Especifique o seleccione los siguientes datos y haga clic en **Agregar**:

    |Configuración|Valor|
    |----|----|
    |Source| Seleccione **VirtualNetwork** |
    |Source port ranges| * |
    |Destination | Seleccione **Service Tag** (Etiqueta de servicio)|
    |Etiqueta de servicio de destino | Seleccione **Storage** (Almacenamiento)|
    |Intervalos de puertos de destino| Deje el valor predeterminado *8080* |
    |Protocolo|Any|
    |Acción|Allow|
    |Priority|100|
    |Nombre|Cambie el nombre a **Allow-Storage-All**|

10. Cree otra regla de seguridad de salida que deniegue la comunicación a Internet. Esta regla invalida una regla predeterminada en todos los grupos de seguridad de red que permite la comunicación saliente de Internet. Repita los pasos 6 a 9 anteriores, pero use los siguientes valores:

    |Configuración|Valor|
    |----|----|
    |Source| Seleccione **VirtualNetwork** |
    |Source port ranges| * |
    |Destination | Seleccione **Service Tag** (Etiqueta de servicio)|
    |Etiqueta de servicio de destino| Seleccione **Internet**|
    |Intervalos de puertos de destino| * |
    |Protocolo|Any|
    |Acción|**Cambie el valor predeterminado a *Denegar*** |
    |Priority|110|
    |Nombre|Cambie a *Deny-Internet-All*|

11. Cree una *regla de seguridad de entrada* que permita que llegue a la red tráfico RDP desde cualquier lugar. La regla invalidará cualquier regla de seguridad predeterminada que deniegue todo el tráfico de entrada procedente de Internet. Las conexiones entre Escritorio remoto y la subred están permitidas, por lo que dicha conectividad podrá probarse en un paso posterior. 
12. En **Configuración**, seleccione **Reglas de seguridad de entrada**.
13. Seleccione **+ Agregar** y use los siguientes valores:

    |Configuración|Valor|
    |----|----|
    |Source| Any |
    |Source port ranges| * |
    |Destination | Seleccione **VirtualNetwork**|
    |Intervalos de puertos de destino| Cámbielo a *3389* |
    |Protocolo|Any|
    |Acción|Allow|
    |Priority|120|
    |Nombre|Cambie a *Allow-RDP-All*|

   >[!WARNING] 
   > El puerto 3389 de RDP se expone a Internet. Este puerto solo se recomienda para realizar pruebas. En *Entornos de producción*, se recomienda usar una red privada virtual o una conexión privada.

1.  En **Configuración**, seleccione **Subredes**.
2.  Haga clic en **+ Asociar**.
3.  En **Red virtual**, seleccione **myVirtualNetwork**.
4.  En **Subred**, seleccione **Privada** y después **Aceptar**.

## <a name="restrict-network-access-to-a-resource"></a>Restricción del acceso de la red a un recurso

Los pasos que deben seguirse para restringir el acceso de la red a los recursos creados a través de los servicios de Azure, que se habilitan para los puntos de conexión del servicio, varían de un servicio a otro. Consulte en la documentación de cada servicio concreto los pasos necesarios para dicho servicio. Como ejemplo, de aquí en adelante se explican los pasos necesarios para restringir el acceso de red en una cuenta de Azure Storage.

### <a name="create-a-storage-account"></a>Crear una cuenta de almacenamiento

1. Seleccione **+ Crear un recurso** en la esquina superior izquierda de Azure Portal.
2. Escriba "Cuenta de almacenamiento" en la barra de búsqueda y selecciónelo en el menú desplegable.
3. Haga clic en **+ Agregar**.
4. Escriba la siguiente información:

    |Configuración|Valor|
    |----|----|
    |Subscription| Seleccione su suscripción.|
    |Resource group| Seleccione *myResourceGroup*.|
    |Nombre de la cuenta de almacenamiento| Especifique un nombre que sea único en todas las ubicaciones de Azure, que tenga entre 3 y 24 caracteres de longitud y que esté compuesto exclusivamente de números y letras en minúscula.|
    |Location| Seleccione **(EE.UU.) Este de EE. UU.** |
    |Rendimiento|Estándar|
    |Tipo de cuenta| StorageV2 (uso general v2)|
    |Replicación| Almacenamiento con redundancia local (LRS)|

5. Select **Crear y revisar**, y cuando se superen las comprobaciones de validación, haga clic en **Crear**. 

>[!NOTE] 
> La implementación puede tardar un par de minutos en completarse.

6. Después de que se cree la cuenta de almacenamiento, haga clic en **Ir al recurso**.

### <a name="create-a-file-share-in-the-storage-account"></a>Creación de un recurso compartido de archivos en la cuenta de almacenamiento

1. Vaya a la página de información general de la cuenta de almacenamiento.
2. Seleccione el icono de la aplicación **Recursos compartidos de archivos** y haga clic en **+ Recurso compartido de archivos**.

    |Configuración|Valor|
    |----|----|
    |Nombre| my-file-share|
    |Quota| "Establecer en el máximo" |

   ![Cuenta de almacenamiento](./media/tutorial-restrict-network-access-to-resources/storage-account.png) 

3. Haga clic en **Crear**.
4. El recurso compartido de archivos se debe mostrar en la ventana de Azure si no hace clic en **Actualizar**.

### <a name="restrict-network-access-to-a-subnet"></a>Restricción del acceso de la red a una subred

De forma predeterminada, las cuentas de almacenamiento aceptan conexiones de red procedentes de clientes de cualquier red, incluido Internet. Puede restringir tanto el acceso a la red desde Internet como a las restantes subredes de todas las redes virtuales (excepto a la subred *Private* de la red virtual *myVirtualNetwork*). Para restringir acceso de la red a una subred:

1. En la opción **Configuración** de la cuenta de almacenamiento (nombre único), seleccione **Redes**.
2. Seleccione **Redes seleccionadas**.
3. Seleccione **+ Agregar red virtual existente**.
4. En **Agregar redes**, seleccione los siguientes valores y haga clic en **Agregar**:

    |Configuración|Valor|
    |----|----|
    |Subscription| Seleccione su suscripción.|
    |Redes virtuales| **myVirtualNetwork**|
    |Subredes| **Privado**|

    ![Captura de pantalla que muestra el panel Agregar redes donde se pueden indicar los valores especificados.](./media/tutorial-restrict-network-access-to-resources/virtual-networks.png)

5. Haga clic en **Agregar** e, inmediatamente después, haga clic en el icono **Guardar** para guardar los cambios.
6. En la opción **Configuración** de la cuenta de almacenamiento, seleccione **Claves de acceso**, como se muestra en la siguiente imagen:

      ![Captura de pantalla que muestra Teclas de acceso seleccionado en Configuración, donde se puede obtener una clave.](./media/tutorial-restrict-network-access-to-resources/storage-access-key.png)

7. Haga clic en **Mostrar claves** y anota los valores de **Clave**, ya que tendrá que escribir manualmente la clave1 en un paso posterior, al asignar el recurso compartido de archivos a una letra de unidad de una máquina virtual.

## <a name="create-virtual-machines"></a>Creación de máquinas virtuales

Para probar el acceso de la red a una cuenta de almacenamiento, implemente una máquina virtual en cada subred.

### <a name="create-the-first-virtual-machine"></a>Creación de la primera máquina virtual

1. En la barra "Buscar recursos . . ." busque **Máquinas virtuales**.
2. Seleccione **+ Agregar > Máquina virtual**. 
3. Escriba la siguiente información:

   |Configuración|Valor|
   |----|----|
   |Subscription| Seleccione su suscripción.|
   |Resource group| Seleccione **myResourceGroup, que se creó anteriormente.|
   |Nombre de la máquina virtual| Escriba *myVmPublic*.|
   |Region | (EE. UU.) Este de EE. UU.
   |Opciones de disponibilidad| Zona de disponibilidad|
   |Zona de disponibilidad | 1 |
   |Imagen | Windows Server 2019 Datacenter - Gen 1 |
   |Size | Seleccione el tamaño de la instancia de máquina virtual que desea usar |
   |Nombre de usuario|Escriba un nombre de usuario de su elección.|
   |Contraseña| Escriba una contraseña de su elección. La contraseña debe tener al menos 12 caracteres de largo y cumplir con los [requisitos de complejidad definidos](../virtual-machines/windows/faq.yml?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm-).|
   |Puertos de entrada públicos | Permitir los puertos seleccionados |
   |Selección de puertos de entrada | Deje el valor predeterminado establecido en *RDP (3389)* |

   ![Seleccionar una red virtual](./media/tutorial-restrict-network-access-to-resources/virtual-machine-settings.png)
  
4. Seleccione la pestaña **Redes** y, después, seleccione **myVirtualNetwork**. 
5. Seleccione la subred *Public*.
6. En **Grupo de seguridad de red de NIC**, seleccione **Avanzado**. El portal crea automáticamente un grupo de seguridad de red que permite el puerto 3389, que debe estar abierto para conectarse a la máquina virtual en un paso posterior. 

   ![Escribir información esencial acerca de una máquina virtual](./media/tutorial-restrict-network-access-to-resources/virtual-machine-basics.png)

7. Seleccione **Revisar y crear** y, después, **Crear**, y espere hasta que finalice la implementación.
8. Haga clic en **Ir al recurso** o abra la página **Inicio > Máquinas virtuales** y seleccione la máquina virtual que acaba de crear, *myVmPublic*, que debería haberse iniciado.

### <a name="create-the-second-virtual-machine"></a>Creación de la segunda máquina virtual

1. Repita los pasos 1 a 8, pero, en el paso 3, asigne a la máquina virtual el nombre *myVmPrivate* y, en **Public inbound port**  (Puerto de entrada público), seleccione "None" (Ninguno). 
2. En el paso 4-5, seleccione la subred **Private**.

>[!NOTE]
> La configuración de **Grupo de seguridad de red de NIC** y **Public inbound ports** (Puertos de entrada públicos) debe reflejar la imagen que se muestra a continuación, incluida la ventana de confirmación azul que indica: "all public internet traffic will be blocked by default" (de manera predeterminada se bloqueará todo el tráfico de la red Internet pública).

   ![Creación de una máquina virtual privada](./media/tutorial-restrict-network-access-to-resources/create-private-virtual-machine.png)

3. Seleccione **Revisar y crear** y, después, **Crear**, y espere hasta que finalice la implementación. 

>[!WARNING]
> No continúe con el paso siguiente hasta que la implementación haya finalizado.

4. Espere hasta que aparezca la ventana de confirmación que se muestra a continuación y haga clic en **Ir al recurso**.

   ![Crear una ventana de confirmación de máquina virtual privada](./media/tutorial-restrict-network-access-to-resources/create-vm-confirmation-window.png)

## <a name="confirm-access-to-storage-account"></a>Confirmación del acceso a la cuenta de almacenamiento

1. Una vez creada máquina virtual *myVmPrivate*, haga clic en **Ir al recurso**. 
2. Conéctese a la máquina virtual, para lo que debe seleccionar **Conectar > RDP**.
3. Cuando seleccione el botón **Conectar**, se creará un archivo de Protocolo de Escritorio remoto (.rdp). Haga clic en **Descargar archivo RDP** para realizar la descarga en el equipo.  
4. Abra el archivo .rdp descargado. Cuando se le pida, seleccione **Conectar**. Escriba el nombre de usuario y la contraseña que especificó al crear la máquina virtual. Puede que deba seleccionar **More choices** (Más opciones) y, luego, **Use a different account** (Usar una cuenta diferente) para especificar las credenciales que escribió al crear la máquina virtual. En el campo del correo electrónico, escriba las credenciales de "Cuenta de administrador: nombre de usuario" que especificó anteriormente. 
5. Seleccione **Aceptar**.
6. Puede recibir una advertencia de certificado durante el proceso de inicio de sesión. Si recibe la advertencia, seleccione **Sí** o **Continuar** para continuar con la conexión. Debería ver que la máquina virtual se inicia como se muestra:

   ![Mostrar máquina virtual privada en ejecución](./media/tutorial-restrict-network-access-to-resources/virtual-machine-running.png)

7. En la ventana de la máquina virtual, abra una instancia de la CLI de PowerShell.
8. Utilice el script siguiente para asignar el recurso compartido de archivos de Azure a la unidad Z mediante PowerShell. Antes de ejecutar los comandos siguientes, reemplace `<storage-account-key>` y los dos campos `<storage-account-name>` por los valores que especificó entre comillas anteriormente en [Creación de una cuenta de almacenamiento](#create-a-storage-account).

   ```powershell
   $acctKey = ConvertTo-SecureString -String "<storage-account-key>" -AsPlainText -Force
   $credential = New-Object System.Management.Automation.PSCredential -ArgumentList "Azure\<storage-account-name>", $acctKey
   New-PSDrive -Name Z -PSProvider FileSystem -Root "\\<storage-account-name>.file.core.windows.net\my-file-share" -Credential $credential
   ```

   PowerShell devuelve una salida similar a la del ejemplo siguiente:

   ```powershell
   Name           Used (GB)     Free (GB) Provider      Root
   ----           ---------     --------- --------      ----
   Z                                      FileSystem    \\vnt.file.core.windows.net\my-f...
   ```

   El recurso compartido de archivos de Azure se ha asignado correctamente a la unidad Z.

9.   Cierre la sesión de Escritorio remoto a la máquina virtual *myVmPrivate*.

## <a name="confirm-access-is-denied-to-storage-account"></a>Confirmación de la denegación del acceso a la cuenta de almacenamiento

1. En el cuadro *Search resources, services, and docs* (Buscar recursos, servicios y documentos) que encontrará en la parte superior del portal, escriba **myVmPublic**.
2. Cuando aparezca **myVmPublic** en los resultados de búsqueda, selecciónelo.
3. Siga los pasos 1 a 8 que se indican en [Confirmación del acceso a la cuenta de almacenamiento](#confirm-access-to-storage-account) con la máquina virtual *myVmPublic*.

   Tras una breve espera, recibirá un error `New-PSDrive : Access is denied`. El acceso se deniega porque la máquina virtual *myVmPublic* está implementada en la subred *Public*. La subred *Public* no tiene un punto de conexión de servicio habilitado para Azure Storage. La cuenta de almacenamiento solo permite el acceso a la red desde la subred *Private*, no desde la subred *Public*.

4. Cierre la sesión de Escritorio remoto a la máquina virtual *myVmPublic*.
5. Cuando se encuentre de nuevo en Azure Portal, vaya a la cuenta de almacenamiento con un nombre único que creó anteriormente.
6. En File service, seleccione **Recursos compartidos de archivos** y, después el recurso *my-file-share* que creó anteriormente.
7. Debería recibir el siguiente mensaje de error:

   ![Error de acceso denegado](./media/tutorial-restrict-network-access-to-resources/access-denied-error.png)
   
>[!NOTE] 
> El acceso se deniega porque el equipo no se encuentra en la subred *Privada* de la red virtual *MyVirtualNetwork*.

## <a name="clean-up-resources"></a>Limpieza de recursos

Cuando ya no sea necesario, elimine el grupo de recursos y todos los recursos que contiene:

1. Escriba *myResourceGroup* en el cuadro **Buscar** que se encuentra en la parte superior del portal. Seleccione **myResourceGroup** cuando lo vea en los resultados de la búsqueda.
2. Seleccione **Eliminar grupo de recursos**.
3. Escriba *myResourceGroup* para **ESCRIBA EL NOMBRE DEL GRUPO DE RECURSOS:** y seleccione **Eliminar**.

## <a name="next-steps"></a>Pasos siguientes

En este tutorial, ha habilitado un punto de conexión de servicio en una subred de la red virtual. Ha aprendido que puede habilitar puntos de conexión de servicio para los recursos implementados desde varios servicios de Azure. Ha creado una cuenta de Azure Storage y ha restringido el acceso de la red a la cuenta de almacenamiento que se encuentra en una subred de la red virtual. Para más información acerca de los puntos de conexión de servicio, consulte [Introducción a los puntos de conexión de un servicio](virtual-network-service-endpoints-overview.md) y [Administración de subredes](virtual-network-manage-subnet.md).

Si tiene varias redes virtuales en la cuenta, es posible que desee conectar dos de ellas para que los recursos que están dentro de cada red virtual puedan comunicarse entre sí. Para aprender a conectar redes virtuales, pase al siguiente tutorial.

> [!div class="nextstepaction"]
> [Conexión de redes virtuales](./tutorial-connect-virtual-networks-portal.md)
