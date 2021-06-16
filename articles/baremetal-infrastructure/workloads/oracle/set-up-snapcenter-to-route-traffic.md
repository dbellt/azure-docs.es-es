---
title: Configuración de SnapCenter para enrutar el tráfico de Azure a servidores de Oracle BareMetal
description: Aprenda a configurar SnapCenter de NetApp para enrutar el tráfico de Azure a servidores de Oracle BareMetal Infrastructure.
ms.topic: how-to
ms.subservice: baremetal-oracle
ms.date: 05/05/2021
ms.openlocfilehash: 9d8f83712822dcf556efe1098eadd1cfdb4867e6
ms.sourcegitcommit: e1d5abd7b8ded7ff649a7e9a2c1a7b70fdc72440
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/27/2021
ms.locfileid: "110576236"
---
# <a name="set-up-netapp-snapcenter-to-route-traffic-from-azure-to-oracle-baremetal-servers"></a>Configuración de SnapCenter de NetApp para enrutar el tráfico de Azure a servidores de Oracle BareMetal

En este artículo, veremos cómo configurar SnapCenter de NetApp para enrutar el tráfico de Azure a servidores de Oracle BareMetal Infrastructure. 

## <a name="prerequisites"></a>Requisitos previos

> [!div class="checklist"]
> - **Requisitos del sistema SnapCenter Server:** Azure Windows 2019 o posterior con 4 vCPU, 16 GB de RAM y un mínimo de 500 GB de almacenamiento SSD prémium administrado.
> - **Requisitos de red de ExpressRoute:** Un usuario de SnapCenter para Oracle BareMetal debe trabajar con Microsoft Operations para crear las cuentas de red para habilitar la comunicación entre la máquina virtual (VM) de almacenamiento personal y la VM de SnapCenter en Azure.
> - **Java 1.8 en instancias de BareMetal:** Los complementos de SnapCenter requieren que Java 1.8 esté instalado en las instancias de BareMetal.

## <a name="steps-to-integrate-snapcenter"></a>Pasos para integrar SnapCenter

Estos son los pasos que tiene que seguir para configurar SnapCenter de NetApp para enrutar el tráfico de Azure a servidores de Oracle BareMetal Infrastructure: 

1. Genere una solicitud de incidencia de soporte técnico para comunicar la clave pública generada por el usuario al equipo de Microsoft Ops. Soporte técnico tiene que configurar el usuario de SnapCenter para que tenga acceso al sistema de almacenamiento. 

2. Cree una VM en su instancia de Azure Virtual Network (red virtual) que tenga acceso a las instancias de BareMetal; esta VM se usa para SnapCenter. 

3. Descargue e instale SnapCenter. 

4. Lleve a cabo operaciones de copia de seguridad y recuperación. 

>[!NOTE]
> En estos pasos se supone que ya ha creado una conexión de ExpressRoute entre la suscripción en Azure y el inquilino en Oracle HaaS.

## <a name="create-a-support-ticket-for-user-role-storage-setup"></a>Creación de una incidencia de soporte técnico para la configuración del almacenamiento del rol de usuario

1. Abra Azure Portal y vaya a la página Suscripciones. Seleccione la suscripción a BareMetal.
2. En la página de la suscripción de BareMetal, seleccione **Grupos de recursos**.
3. Seleccione un grupo de recursos adecuado en una región.
    
    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/select-resource-group.png" alt-text="Captura de pantalla que muestra los grupos de recursos en la página de la suscripción.":::

4. Seleccione una entrada de SKU correspondiente a SAP HANA en Azure Storage. 

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/select-sku.png" alt-text="Captura de pantalla de un grupo de recursos que muestra una SKU resaltada de tipo SAP HANA en Azure.":::

5. Seleccione **Nueva solicitud de soporte técnico**.

6. En la pestaña **Básico**, especifique la siguiente información:
    - **Tipo de problema**: Técnico
    -   **Suscripción**: Su suscripción
    -   **Servicio**: BareMetal Infrastructure
    -   **Recurso**: Su grupo de recursos
    -   **Resumen**:Configuración de acceso a SnapCenter
    -   **Tipo de problema**: Instalación y configuración
    -   **Subtipo de problema**: Configuración de SnapCenter

7. En la **Descripción** del vale de soporte técnico, en la pestaña  **Detalles** , pegue el contenido de un archivo *.pem en el cuadro de texto para obtener más detalles. También puede comprimir un archivo *.pem y cargarlo. snapcenter.pem será la clave pública para el usuario de SnapCenter. Use el ejemplo siguiente para crear un archivo *.pem a partir de una de las instancias de BareMetal. 

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/pem.png" alt-text="Captura de pantalla que muestra el contenido de ejemplo de un archivo .pem.":::

    >[!NOTE]
    >El nombre del archivo "snapcenter" es el nombre de usuario necesario para hacer llamadas API REST. Por lo tanto, se recomienda que el nombre de archivo sea descriptivo.

8.  Seleccione **Revisar y crear** para revisar la incidencia de soporte técnico.

9.  Una vez que se envía el certificado de clave pública, Microsoft configura el nombre de usuario de SnapCenter para el inquilino junto con la dirección IP de la máquina virtual de almacenamiento (SVM). Microsoft le dará la dirección IP de la SVM.

10. Después de recibir la dirección IP de la SVM, establezca una contraseña para acceder a la SVM, que usted controla.

    El siguiente es un ejemplo de la llamada REST desde una instancia grande de HANA o una VM de la red virtual, que tiene acceso a un entorno de instancia grande de HANA y que se usará para establecer la contraseña.
    
    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/sample-rest-call.png" alt-text="Captura de pantalla que muestra una llamada REST de ejemplo.":::

11. Asegúrese de que no hay una variable de proxy activa en la instancia de BareMetal que se usa para crear el archivo *.pem.

     :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/ensure-no-proxy.png" alt-text="Captura de pantalla que muestra un proxy HTTP sin configurar para asegurar que no hay ninguna variable de proxy activa en la instancia de BareMetal al crear el archivo *.pem.":::

12. Desde la máquina cliente, ahora es posible ejecutar comandos sin un nombre de usuario y contraseña para los comandos de REST habilitados. Pruebe la conexión: 

    Sin proxy:

    :::image type="content" source="media/netapp-snapcenter-integration-oracle-baremetal/connection-test.png" alt-text="Captura de pantalla que muestra la prueba de conexión y los comandos REST habilitados sin nombre de usuario ni contraseña.":::


       >[!NOTE]
       > Se puede agregar "--verbose" a cualquiera de los comandos curl para proporcionar más detalles sobre por qué un comando no se ha completado correctamente.

## <a name="install-snapcenter"></a>Instalación de SnapCenter

1. En la VM Windows aprovisionada, vaya al [sitio web de NetApp](https://mysupport.netapp.com/site/products/all/details/snapcenter/downloads-tab).

2. Inicie sesión y descargue SnapCenter. Downloads > SnapCenter > seleccione la versión **4.4**.

3. Instale SnapCenter 4.4. Seleccione **Next** (Siguiente).

    El instalador comprobará los requisitos previos de la VM. Preste atención al tamaño de la VM, especialmente en entornos de mayor tamaño. Es correcto seguir instalando aunque haya pendiente un reinicio.

4. Configure las credenciales de usuario para SnapCenter. De manera predeterminada, se rellena con el usuario de Windows que inició la aplicación para la instalación. A menos que haya un conflicto de puertos, se recomienda usar los puertos predeterminados.

    El asistente para la instalación tardará algún tiempo en completarse y mostrar el progreso.
 
5. Una vez que se haya completado la instalación, seleccione **Finish** (Finalizar).  Tome nota de la dirección URL del portal web de SnapCenter.  También se puede acceder haciendo doble clic en el icono de SnapCenter que aparecerá en el escritorio una vez que se complete la instalación.
 
## <a name="disable-enhanced-messaging-service-ems-messages-to-netapp-auto-support"></a>Deshabilitación de los mensajes del servicio de mensajería mejorado (EMS) para la compatibilidad automática de NetApp

La recopilación de datos de EMS está habilitada de manera predeterminada y se ejecuta cada siete días después de la fecha de instalación. Puede deshabilitar la recopilación de datos en cualquier momento.

1. Desde una línea de comandos de PowerShell, establezca una sesión con SnapCenter:

   ```powershell-interactive
   Open-SmConnection
   ```

2. Inicie sesión con sus credenciales.

3. Deshabilite la recopilación de datos de EMS: 

   ```powershell-interactive
   Disable-SmDataCollectionEms
   ```
   
## <a name="next-steps"></a>Pasos siguientes

Obtenga más información sobre cómo configurar SnapCenter:

> [!div class="nextstepaction"]
> [Configurar SnapCenter](configure-snapcenter-oracle-baremetal.md)
