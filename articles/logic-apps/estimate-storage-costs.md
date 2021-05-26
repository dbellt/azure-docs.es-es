---
title: Cálculo de los costos de almacenamiento de Azure Logic Apps de inquilino único
description: Calcule los costos de almacenamiento de los flujos de trabajo mediante la calculadora de almacenamiento de Logic Apps.
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: how-to
ms.date: 05/13/2021
ms.openlocfilehash: 093dd29a4e9eda9157a49c6cc44144b5e8a4bbec
ms.sourcegitcommit: 58e5d3f4a6cb44607e946f6b931345b6fe237e0e
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/25/2021
ms.locfileid: "110369155"
---
# <a name="estimate-storage-costs-for-workflows-in-single-tenant-azure-logic-apps"></a>Cálculo de los costos de almacenamiento de flujos de trabajo en Azure Logic Apps de inquilino único

Azure Logic Apps usa [Azure Storage](/azure/storage/) en las operaciones de almacenamiento. Con el tradicional Azure Logic Apps *multiinquilino*, el uso y los costos de almacenamiento se asocian a la aplicación lógica. Ahora, con Azure Logic Apps *de inquilino único*, puede usar su propia cuenta de almacenamiento. Estos costos de almacenamiento se enumeran por separado en la factura de Azure. Esta funcionalidad le proporciona más flexibilidad y control sobre los datos de la aplicación lógica.

> [!NOTE]
> Este artículo se aplica a los flujos de trabajo en el entorno de Azure Logic Apps de inquilino único. Estos flujos de trabajo existen en la misma aplicación lógica y en un inquilino único que comparten el mismo almacenamiento. Para obtener más información, consulte [Entorno de servicio de integración de inquilino único o multiinquilino](single-tenant-overview-compare.md).

Los costos de almacenamiento cambian en función del contenido del flujo de trabajo. Los distintos desencadenadores, acciones y cargas tienen como resultado diferentes operaciones y necesidades de almacenamiento. En este artículo se describe cómo calcular los costos de almacenamiento cuando se usa su propia cuenta de Azure Storage con aplicaciones lógicas basadas en un inquilino único. En primer lugar, puede [calcular el número de operaciones de almacenamiento que realizará](#estimate-storage-needs) mediante la calculadora de almacenamiento de Logic Apps. A continuación, puede [calcular los posibles costos de almacenamiento](#estimate-storage-costs) con estos números en la calculadora de precios de Azure.

## <a name="prerequisites"></a>Requisitos previos

* Una cuenta y una suscripción de Azure. Si aún no tiene una, [regístrese para obtener una cuenta de Azure gratuita](https://azure.microsoft.com/free/).

* Flujo de trabajo de Logic Apps basado en un inquilino único. Puede crear un flujo de trabajo [mediante Azure Portal](create-single-tenant-workflows-azure-portal.md) o [Visual Studio Code](create-single-tenant-workflows-visual-studio-code.md). Si aún no tiene un flujo de trabajo, puede usar el ejemplo de los flujos de trabajo pequeño, mediano y grande en la calculadora de almacenamiento.

* Una cuenta de almacenamiento de Azure que quiera usar con el flujo de trabajo. Si no dispone de una cuenta de almacenamiento, [cree una](../storage/common/storage-account-create.md).

## <a name="get-your-workflows-json-code"></a>Obtención del código JSON del flujo de trabajo

Si tiene un flujo de trabajo para calcular, obtenga el código JSON para el flujo de trabajo:

1. Inicie sesión en [Azure Portal](https://portal.azure.com/).
1. Vaya al servicio **Logic Apps** y seleccione el flujo de trabajo.
1. En el menú de la aplicación lógica, en **Herramientas de desarrollo**, seleccione **Vista de código de aplicación lógica**.
1. Copie el código JSON del flujo de trabajo.

## <a name="estimate-storage-needs"></a>Cálculo de las necesidades de almacenamiento

1. Vaya a la [calculadora de almacenamiento de Logic Apps](https://logicapps.azure.com/calculator).

   :::image type="content" source="./media/estimate-storage-costs/storage-calculator.png" alt-text="Captura de pantalla de la calculadora de almacenamiento de Logic Apps, que muestra la interfaz de entrada." lightbox="./media/estimate-storage-costs/storage-calculator.png":::

1. Escriba, cargue o seleccione el código JSON para un flujo de trabajo de aplicación lógica basado en un inquilino único.

   * Si copió código en la sección anterior, péguelo en el cuadro para **pegar o cargar workflow.json**. 
   * Si tiene el archivo **workflow.json** guardado localmente, elija **Examinar archivos** en **Seleccionar flujo de trabajo**. Elija el archivo y, a continuación, seleccione **Abrir**.
   * Si aún no tiene un flujo de trabajo, elija uno de los flujos de trabajo de ejemplo en **Seleccionar flujo de trabajo**.

1. Revise las opciones en **Opciones avanzadas**. Esta configuración depende del tipo de flujo de trabajo y puede incluir:

   * Una opción para especificar el número de veces que se ejecutan los bucles.
   * Una opción para seleccionar todas las acciones con cargas de más de 32 KB.

1. En **Monthly runs** (Ejecuciones mensuales), escriba el número de veces que ejecuta el flujo de trabajo cada mes.
1. Seleccione **Calcular** y espere a que se ejecute el cálculo.
1. En **Storage Operation Breakdown and Calculation Steps** (Desglose de la operación de almacenamiento y pasos de cálculo), revise los cálculos de **Operation Counts** (Recuentos de operaciones).

    Puede ver los recuentos de operaciones calculados por ejecución y por mes en las dos tablas. Se muestran las operaciones siguientes:

    * **Blob (lectura)** , para operaciones de lectura de Azure Blob Storage.
    * **Blob (escritura)** , para operaciones de escritura de Azure Blob Storage.
    * **Cola**, para las operaciones de colas de clase 2 de Colas de Azure.
    * **Tablas**, para operaciones de Table Storage de Azure.

    Cada operación tiene un número de recuento mínimo, máximo y de "mejor suposición". Elija el número más pertinente que se usará para [calcular los costos de la operación de almacenamiento](#estimate-storage-costs) en función de su escenario individual. Normalmente, se recomienda usar el recuento de "mejor suposición" para una mayor precisión. Sin embargo, también puede usar el recuento máximo para asegurarse de que el cálculo de costos tiene un búfer.

    :::image type="content" source="./media/estimate-storage-costs/storage-calculator-results.png" alt-text="Captura de pantalla de la calculadora de almacenamiento de Logic Apps, que muestra la salida con operaciones calculadas." lightbox="./media/estimate-storage-costs/storage-calculator-results.png":::

## <a name="estimate-storage-costs"></a>Cálculo de los costes de almacenamiento

Después de [calcular las necesidades de almacenamiento del flujo de trabajo de la aplicación lógica](#estimate-storage-needs), puede calcular los posibles costos de almacenamiento mensuales. Puede calcular los precios de los siguientes tipos de operación de almacenamiento:

* [Operaciones de lectura y escritura de almacenamiento en blobs](#estimate-blob-storage-operations-costs)
* [Operaciones de almacenamiento en cola](#estimate-queue-operations-costs)
* [Operaciones de almacenamiento en tabla](#estimate-table-operations-costs)

### <a name="estimate-blob-storage-operations-costs"></a>Cálculo de los costos de las operaciones de almacenamiento en blobs

> [!NOTE]
> Esta característica no está disponible actualmente. Por ahora, todavía puede usar la calculadora para calcular el [almacenamiento en cola](#estimate-queue-operations-costs) y el [almacenamiento en tabla](#estimate-table-operations-costs).

Para calcular los costos mensuales de las operaciones de almacenamiento en blobs de la aplicación lógica:

1. Vaya a la [calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/).
1. En la pestaña **Productos**, seleccione **Almacenamiento** &gt; **Cuentas de almacenamiento**. O bien, en el cuadro de búsqueda de la **barra de búsqueda**, escriba **Cuentas de almacenamiento** y seleccione el icono.
    :::image type="content" source="./media/estimate-storage-costs/pricing-calculator-storage-tile.png" alt-text="Captura de pantalla de la calculadora de precios de Azure, en la que se muestra el icono para agregar la vista Cuentas de almacenamiento." lightbox="./media/estimate-storage-costs/pricing-calculator-storage-tile.png":::
1. En la notificación de **cuentas de almacenamiento agregadas**, seleccione **Ver** para ver la sección **Cuentas de almacenamiento** de la calculadora. O bien, vaya manualmente la sección **Cuentas de almacenamiento** manualmente.
1. En **Región,** seleccione la región de la aplicación lógica.
1. En **Tipo**, seleccione **Almacenamiento de blobs en bloque**.
1. En **Nivel de rendimiento**, seleccione el nivel de rendimiento.
1. En **Redundancia**, seleccione el nivel de redundancia.
1. No haga más ajustes.
1. En **Operaciones de escritura**, escriba el número de operaciones de **blob (escritura)** de la calculadora de almacenamiento de Logic Apps *tal cual*.
1. En **Operaciones de lectura**, escriba el número de operaciones de **blob (lectura)** de la calculadora de almacenamiento de Logic Apps *tal cual*.
1. Revise los costos estimados de las operaciones de almacenamiento de blobs.

### <a name="estimate-queue-operations-costs"></a>Cálculo de los costos de las operaciones de cola

Para calcular los costos mensuales de las operaciones de cola de la aplicación lógica:

1. Vaya a la [calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/).
1. En la pestaña **Productos**, seleccione **Almacenamiento** &gt; **Cuentas de almacenamiento**. O bien, en el cuadro de búsqueda de la **barra de búsqueda**, escriba **Cuentas de almacenamiento** y seleccione el icono.
    :::image type="content" source="./media/estimate-storage-costs/pricing-calculator-storage-tile.png" alt-text="Captura de pantalla de la calculadora de precios de Azure, en la que se muestra el icono para agregar la vista Cuentas de almacenamiento." lightbox="./media/estimate-storage-costs/pricing-calculator-storage-tile.png":::
1. En la notificación de **cuentas de almacenamiento agregadas**, seleccione **Ver** para ver la sección **Cuentas de almacenamiento** de la calculadora. O bien, vaya manualmente la sección **Cuentas de almacenamiento** manualmente.
1. En **Región,** seleccione la región de la aplicación lógica.
1. En **Tipo**, seleccione **Queue Storage**.
1. En **Tipo de cuenta de almacenamiento**, seleccione el tipo de cuenta de almacenamiento.
1. En **Redundancia**, seleccione el nivel de redundancia.
1. En **Operaciones de clase 2 de cola**, escriba el número de operaciones de **cola** de la calculadora de almacenamiento de Logic Apps *dividido por 10 000*. Este paso es necesario porque la calculadora funciona en unidades transaccionales para las operaciones de cola.
1. Revise los costos estimados de las operaciones de cola.

### <a name="estimate-table-operations-costs"></a>Cálculo de los costos de las operaciones de tabla

Para calcular los costos mensuales de las operaciones de almacenamiento en tablas de la aplicación lógica:

1. Vaya a la [calculadora de precios de Azure](https://azure.microsoft.com/pricing/calculator/).
1. En la pestaña **Productos**, seleccione **Almacenamiento** &gt; **Cuentas de almacenamiento**. O bien, en el cuadro de búsqueda de la **barra de búsqueda**, escriba **Cuentas de almacenamiento** y seleccione el icono.
    :::image type="content" source="./media/estimate-storage-costs/pricing-calculator-storage-tile.png" alt-text="Captura de pantalla de la calculadora de precios de Azure, en la que se muestra el icono para agregar la vista Cuentas de almacenamiento." lightbox="./media/estimate-storage-costs/pricing-calculator-storage-tile.png":::
1. En la notificación de **cuentas de almacenamiento agregadas**, seleccione **Ver** para ver la sección **Cuentas de almacenamiento** de la calculadora. O bien, vaya manualmente la sección **Cuentas de almacenamiento** manualmente.
1. En **Región,** seleccione la región de la aplicación lógica.
1. En **Tipo**, seleccione **Table Storage**.
1. En **Nivel**, seleccione el nivel de rendimiento.
1. En **Redundancia**, seleccione el nivel de redundancia.
1. En **Transacciones de almacenamiento**, escriba el número de operaciones de **tabla** de la calculadora de almacenamiento de Logic Apps *dividido por 10 000*. Este paso es necesario porque la calculadora funciona en unidades transaccionales para las operaciones de cola.
1. Revise los costos estimados de las operaciones de almacenamiento en tablas.

## <a name="next-step"></a>Paso siguiente

> [!div class="nextstepaction"]
> [Planeamiento y administración de los costos de Logic Apps](plan-manage-costs.md)
