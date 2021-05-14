---
title: Creación de una biblioteca de contenido para implementar máquinas virtuales en Azure VMware Solution
description: Cree una biblioteca de contenido para implementar una máquina virtual en una nube privada de Azure VMware Solution.
ms.topic: how-to
ms.date: 02/03/2021
ms.openlocfilehash: b27d2682d8799bec6b09a08e5063359113b20a88
ms.sourcegitcommit: 2aeb2c41fd22a02552ff871479124b567fa4463c
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/22/2021
ms.locfileid: "107873890"
---
# <a name="create-a-content-library-to-deploy-vms-in-azure-vmware-solution"></a>Creación de una biblioteca de contenido para implementar máquinas virtuales en Azure VMware Solution

Una biblioteca de contenido almacena y administra el contenido en forma de elementos de biblioteca. Un solo elemento de biblioteca consta de uno o varios archivos que se usan para implementar máquinas virtuales (VM). 

En este artículo, le guiaremos por el procedimiento de creación de una biblioteca de contenido.  A continuación, veremos cómo implementar una máquina virtual mediante una imagen ISO de la biblioteca de contenido.

## <a name="prerequisites"></a>Requisitos previos

Para realizar este tutorial, se necesita un segmento NSX-T (conmutador lógico) y un servicio DHCP administrado.  Para más información, consulte el artículo [Administración de DHCP en Azure VMware Solution](manage-dhcp.md).

## <a name="create-a-content-library"></a>Crear una biblioteca de contenido

1. En el cliente de vSphere local, seleccione **Menu > Content Libraries** (Menú > Bibliotecas de contenido).

   ![Selección de Menu -> Content Libraries (Menú > Bibliotecas de contenido)](./media/content-library/vsphere-menu-content-libraries.png)

1. Seleccione el botón **Add** (Agregar) para crear una biblioteca de contenido.

   ![Selección del botón Add (Agregar) para crear una biblioteca de contenido.](./media/content-library/create-new-content-library.png)

1. Especifique un nombre y confirme la dirección IP del servidor vCenter y seleccione **Next** (Siguiente).

   ![Especifique un nombre, anote su elección y, luego, seleccione Next (Siguiente).](./media/content-library/new-content-library-step1.png)

1. Seleccione **Local content library** (Biblioteca de contenido local) y elija **Next** (Siguiente).

   ![En este ejemplo, se creará una biblioteca de contenido local. Seleccione Next (Siguiente).](./media/content-library/new-content-library-step2.png)

1. Seleccione el almacén de datos que almacenará la biblioteca de contenido y, luego, elija **Next** (Siguiente).

   ![Seleccione el almacén de datos donde quiere hospedar la biblioteca de contenido y seleccione Next (Siguiente).](./media/content-library/new-content-library-step3.png)

1. Revise y compruebe la configuración de la biblioteca de contenido y, luego, seleccione **Finish** (Finalizar).

   ![Compruebe la configuración y seleccione Finish (Finalizar).](./media/content-library/new-content-library-step4.png)

## <a name="upload-an-iso-image-to-the-content-library"></a>Cargar una imagen ISO en la biblioteca de contenido

Ahora que se ha creado la biblioteca de contenido, puede agregar una imagen ISO para implementar una máquina virtual en un clúster de nube privada. 

1. En el cliente de vSphere, seleccione **Menu > Content Libraries** (Menú > Bibliotecas de contenido).

1. Haga clic con el botón derecho en la biblioteca de contenido que quiera usar para la nueva imagen ISO y seleccione **Import Item** (Importar elemento).

1. Importe un elemento de la biblioteca para el origen mediante una de las acciones siguientes y, luego, seleccione **Import** (Importar):
   1. Seleccione la dirección URL y proporcione una dirección URL para descargar una imagen ISO.

   1. Seleccione **Local File** (Archivo local) para cargarla desde el sistema local.

   > [!TIP]
   > También puede definir un nombre de elemento personalizado y notas para el destino.

1. Abra la biblioteca y seleccione la pestaña **Other Types** (Otros tipos) para comprobar que la imagen ISO se cargó correctamente.


## <a name="deploy-a-vm-to-a-private-cloud-cluster"></a>Implementación de una máquina virtual en un clúster de nube privada

1. En el cliente de vSphere, seleccione **Menu > Hosts and Clusters** (Menú > Hosts y clústeres).

1. En el panel izquierdo, expanda el árbol y seleccione un clúster.

1. Seleccione **Actions > New Virtual Machine** (Acciones > Nueva máquina virtual).

1. Siga los pasos del asistente y modifique los valores que desee.

1. Seleccione **New CD/DVD Drive > Client Device > Content Library ISO File** (Nueva unidad de CD o DVD > Archivo ISO de la biblioteca de contenido).

1. Seleccione la imagen ISO cargada en la sección anterior y, luego, elija **OK** (Aceptar).

1. Active la casilla **Connect** (Conectar) para que la imagen ISO se monte en el momento del encendido.

1. Seleccione **New Network > Select dropdown > Browse** (Nueva red > Seleccionar lista desplegable > Examinar).

1. Seleccione el **conmutador lógico (segmento)** y elija **OK** (Aceptar).

1. Modifique cualquier otra configuración de hardware y seleccione **Next** (Siguiente).

1. Compruebe la configuración y seleccione **Finish** (Finalizar).


## <a name="next-steps"></a>Pasos siguientes

Ahora que se ha explicado la creación de una biblioteca de contenido para implementar máquinas virtuales en Azure VMware Solution, puede que desee obtener información sobre:

- [Procedimiento de migración de las cargas de trabajo de VM a la nube privada](tutorial-deploy-vmware-hcx.md)
- [Administración del ciclo de vida de las máquinas virtuales de Azure VMware Solution](lifecycle-management-of-azure-vmware-solution-vms.md)

<!-- LINKS - external-->

<!-- LINKS - internal -->
