---
title: 'Azure VMware Solution by CloudSimple: configuración de tablas de firewall y reglas'
description: Se describe cómo configurar tablas de firewall y reglas de nube privada para restringir el tráfico en las subredes y las redes VLAN.
author: shortpatti
ms.author: v-patsho
ms.date: 08/15/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: 4ecc1010afb547d97f0a5c21ff1caa0886edbb48
ms.sourcegitcommit: 516eb79d62b8dbb2c324dff2048d01ea50715aa1
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/28/2021
ms.locfileid: "108176871"
---
# <a name="set-up-firewall-tables-and-rules-for-private-clouds"></a>Configuración de tablas de firewall y reglas para las nubes privadas

Las tablas de firewall y las reglas asociadas permiten aplicar restricciones sobre el tráfico en determinadas subredes y redes VLAN.

* Una subred puede estar asociada a una tabla de firewall.
* Una tabla de firewall puede estar asociada a varias subredes.

## <a name="add-a-new-firewall-table"></a>Adición de una nueva tabla de firewall

1. [Acceda al portal de CloudSimple](access-cloudsimple-portal.md) y seleccione **Network** (Red) en el menú lateral.
2. Seleccione **Firewall Tables** (Tablas de firewall).
3. Seleccione **Create firewall table** (Crear tabla de firewall).

    ![Página VLAN/subnet (VLAN o subred)](media/firewall-tables-page.png)

4. Escriba un nombre para la tabla.
5. Se muestra una regla predeterminada para la tabla. Haga clic en **Create New Rule** (Crear nueva regla) para crear otra regla. Vea el procedimiento siguiente para más información.
6. Haga clic en **Done** (Listo) para guardar la tabla de firewall.

> [!IMPORTANT]
> Puede crear hasta dos tablas de Firewall por cada nube privada.

## <a name="firewall-rules"></a>Reglas de firewall

Las reglas de firewall determinan cómo trata el firewall determinados tipos de tráfico. En la pestaña **Rules** (Reglas) de una tabla de firewall seleccionada se enumeran todas las reglas asociadas.

![Tabla de reglas de firewall](media/firewall-rules-tab.png)

## <a name="create-a-firewall-rule"></a>Creación de una regla de firewall

1. Muestre la configuración para crear una regla de firewall de cualquiera de estas maneras:
    * Haga clic en **Add Rule** (Agregar regla) al crear una tabla de firewall.
    * Seleccione una tabla de firewall determinada en la página **Network > Firewall Tables** (Red > Tablas de firewall) y haga clic en **Create new firewall rule** (Crear nueva regla de firewall).
2. Configure la regla de la siguiente manera:
    * **Nombre**. Asigne un nombre a la regla.
    * **Prioridad**. Asigne una prioridad a la regla. Las reglas con números más bajos se ejecutan primero.
    * **Tipo de tráfico**. Seleccione si la regla es para el tráfico de la nube privada, Internet o VPN (sin estado) o para una dirección IP pública (con estado).
    * **Protocolo**. Seleccione el protocolo que se incluye en la regla (TCP, UDP o cualquier otro).
    * **Dirección**. Seleccione si la regla es para el tráfico entrante o saliente. Debe definir reglas independientes para el tráfico entrante y saliente.
    * **Acción**. Seleccione la acción que se realizará si la regla concuerda (permitir o denegar).
    * **Origen**. Especifique los orígenes que se incluyen en la regla (bloque CIDR, interno o cualquier origen).
    * **Intervalo de puertos de origen**. Especifique el intervalo de puertos sujetos a la regla.
    * **Dirección**. Seleccione entrada o salida.
    * **Destino**. Especifique los destinos que se incluyen en la regla (bloque CIDR, interno o cualquier origen).
    * **Intervalo de puertos de origen**. Especifique el intervalo de puertos sujetos a la regla.

    ![Regla de adición de tablas de firewall](media/firewall-rule-create.png)

3. Haga clic en **Done** (Listo) para guardar la regla y agregarla a la lista de reglas de la tabla de firewall.

> [!IMPORTANT]
> Cada tabla de Firewall puede tener hasta diez reglas de entrada y veinte reglas de salida. Estos límites se pueden incrementar [poniéndose en contacto con soporte técnico](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

## <a name="attach-vlanssubnets"></a><a name="attach-vlans-subnet"></a>Asociación de redes VLAN o subredes

Después de definir una tabla de firewall, puede especificar las subredes que están sujetas a las reglas de la tabla.

1. En la página **Network** > **Firewall Tables** (Red > Tablas de firewall), seleccione una tabla de firewall.
2. Abra la pestaña **Attached VLANs/Subnet** (VLAN o subred asociada).
3. Haga clic en **Attach to a VLAN/Subnet** (Asociar a VLAN o subred).
4. Seleccione la nube privada y la red VLAN. Se muestran el nombre de la subred asociada y el bloque CIDR.
5. Haga clic en **Enviar**.
