---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 0f7d69d1bf2bd0c1f3b6b8a3a67a788155f30d73
ms.sourcegitcommit: 02d443532c4d2e9e449025908a05fb9c84eba039
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2021
ms.locfileid: "108746907"
---
> [!NOTE]
> Al realizar cambios en la configuración de RBS del adaptador de StorSimple para SharePoint, es necesario haber iniciado sesión con una cuenta de usuario que pertenezca al grupo Admins. del dominio. Además, debe tener acceso a la página de configuración desde un explorador que se ejecuta en el mismo host que la Administración central.
> 
> 

#### <a name="to-configure-rbs"></a>Para configurar RBS
1. Abra la página Administración central de SharePoint y vaya a **Configuración del sistema**. 
2. En la sección **Azure StorSimple**, haga clic en **Configurar el adaptador de StorSimple**.
   
    ![Configuración del adaptador de StorSimple](./media/storsimple-sharepoint-adapter-configure-rbs/HCS_SSASP_ConfigRBS1-include.png) 
3. En la página **Configurar el adaptador de StorSimple**:
   
   1. Asegúrese de que la casilla **Habilitar la edición de la ruta de acceso** está seleccionada.
   2. En el cuadro de texto, escriba la ruta de acceso de convención de nomenclatura universal (UNC) del almacén de datos de blob.
      
      > [!NOTE]
      > El volumen de almacenamiento de blobs debe alojarse en un volumen iSCSI configurado en el dispositivo de StorSimple.

   3. Haga clic en el botón **Habilitar** debajo de cada base de datos de contenido que desea configurar para el almacenamiento remoto.
      
      > [!NOTE]
      > El almacén de blobs debe ser compartido y accesible para todos los servidores web front-end (WFE) y la cuenta de usuario que está configurada para la granja de servidores de SharePoint debe tener acceso al recurso compartido.
      
      ![Habilitación del proveedor de RBS](./media/storsimple-sharepoint-adapter-configure-rbs/HCS_SSASP_ConfigRBS2-include.png)
      
      Al habilitar o deshabilitar RBS, también verá el mensaje siguiente.
      
      ![Configuración del adaptador de StorSimple: habilitar y deshabilitar](./media/storsimple-sharepoint-adapter-configure-rbs/HCS_ConfigureStorSimpleAdapterEnableDisableMessage-include.png)

   4. Haga clic en el botón **Actualizar** para aplicar la configuración. Al hacer clic en el botón **Actualizar**, el estado de la configuración de RBS se actualizará en todos los servidores WEF, y toda la granja tendrá RBS habilitado. Aparece el mensaje siguiente.
      
      ![Mensaje de configuración del adaptador](./media/storsimple-sharepoint-adapter-configure-rbs/HCS_SSASP_ConfigRBS3-include.png)
      
      > [!NOTE]
      > Si va a configurar RBS para una granja de SharePoint con un gran número de bases de datos (más de 200), podría agotarse el tiempo de espera de la página web Administración central de SharePoint. Si esto sucede, actualice la página. Esto no afecta al proceso de configuración.

4. Verifique la configuración:
   
   1. Inicie sesión en el sitio web Administración central de SharePoint y vaya a la página **Configuración del adaptador de StorSimple**.
   2. Compruebe los detalles de la configuración para asegurarse de que coinciden con la configuración que especificó. 
5. Compruebe que RBS funciona correctamente:
   
   1. Cargue un documento en SharePoint. 
   2. Vaya a la ruta de acceso UNC que configuró. Asegúrese de que se ha creado la estructura de directorios de RBS y que contiene el objeto cargado.
6. (Opcional) Puede usar el cmdlet de PowerShell `Migrate()` de Microsoft RBS incluido con SharePoint para migrar el contenido de BLOB existente para el dispositivo StorSimple. Para más información, consulte [Migrar contenido a o desde EDR en SharePoint 2013][6] o [Migración de contenido dentro o fuera de RBS (SharePoint Foundation 2010)][7].
7. (Opcional) En instalaciones de prueba, puede comprobar que los blobs se han extraído de la base de datos de contenido como sigue: 
   
   1. Inicie SQL Management Studio
   2. Ejecute la consulta ListBlobsInDB_2010.sql o ListBlobsInDB_2013.sql, como se indica a continuación.
      
      ```
      **ListBlobsInDB_2013.sql**
      
        USE WSS_Content
        GO
      
        SELECT DocStreams.DocId,
      
               LeafName AS Name,
               Content,
               AllDocs.Size AS OrigSizeOfContent,
               LEN(CAST(Content AS VARBINARY(MAX))) AS SizeOfContentInDB,
               DocStreams.RbsId,
               TimeLastModified
      
        FROM DocStreams
      
             INNER JOIN AllDocs ON DocStreams.DocId = AllDocs.Id
        ORDER BY TimeLastModified DESC
        GO
      
      **ListBlobsInDB_2010.sql**
      
        USE WSS_Content
        GO
      
        SELECT AllDocStreams.Id,
      
               LeafName AS Name,
               Content,
               AllDocs.Size AS OrigSizeOfContent,
               LEN(CAST(Content AS VARBINARY(MAX))) AS SizeOfContentInDB,
               RbsId,
               TimeLastModified
        FROM AllDocStreams
      
             INNER JOIN AllDocs ON AllDocStreams.Id = AllDocs.Id
        ORDER BY TimeLastModified DESC
        GO
      ```
      
      Si RBS se ha configurado correctamente, debe aparecer un valor NULL en la columna SizeOfContentInDB para cualquier objeto que se ha cargado y externalizado correctamente con RBS.
8. (Opcional) Después de configurar RBS y mover todo el contenido de BLOB al dispositivo de StorSimple, puede mover la base de datos de contenido al dispositivo. Si elige mover la base de datos de contenido, recomendamos configurar el almacenamiento de la base de datos de contenido en el dispositivo como el volumen principal. Use los procedimientos recomendados de migración de SQL Server para mover la base de datos de contenido al dispositivo de StorSimple. 
   
   > [!NOTE]
   > El traslado de la base de datos de contenido al dispositivo solo se admite para el dispositivo de la serie StorSimple 8000 (no admitido para las series 5000 o 7000).
   
   Si almacena los blobs y la base de datos de contenido en volúmenes independientes en el dispositivo de StorSimple, se recomienda configurarlos en el mismo contenedor de volumen. Esto garantiza que se hará la copia de seguridad de ellos juntos.
   
   > [!WARNING]
   > Si no tiene RBS habilitado, no es recomendable mover la base de datos de contenido al dispositivo StorSimple. Se trata de una configuración no probada.
   
9. Vaya al paso siguiente: [Configuración de la recolección de elementos no utilizados](#configure-garbage-collection).

[6]: /SharePoint/administration/migrate-content-into-or-out-of-rbs
[7]: /previous-versions/office/sharepoint-foundation-2010/ff628255(v=office.14)