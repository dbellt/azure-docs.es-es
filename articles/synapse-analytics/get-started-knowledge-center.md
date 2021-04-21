---
title: 'Tutorial: Introducción a la exploración del Centro de conocimientos de Synapse'
description: En este tutorial, aprenderá a usar el Centro de conocimientos de Synapse.
services: synapse-analytics
author: saveenr
ms.author: saveenr
manager: julieMSFT
ms.reviewer: jrasnick
ms.service: synapse-analytics
ms.subservice: workspace
ms.topic: tutorial
ms.date: 04/04/2021
ms.openlocfilehash: 92d1534b5dd0233ce88b81d605c6502b67a121df
ms.sourcegitcommit: b4fbb7a6a0aa93656e8dd29979786069eca567dc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/13/2021
ms.locfileid: "107307901"
---
# <a name="explore-the-synapse-knowledge-center"></a>Exploración del Centro de conocimientos de Synapse

En este tutorial, aprenderá a usar el Centro de conocimientos de Synapse Studio.

## <a name="introduction-to-the-knowledge-center"></a>Introducción al Centro de conocimientos

Hay dos formas de buscar el Centro de conocimientos en Synapse Studio:

  1. En el centro de conectividad de inicio, cerca de la parte superior derecha de la página, haga clic en **Obtener información**.
  2. En la barra de menú de la parte superior, haga clic en **?** y, a continuación, en **Centro de conocimientos**.

Elija cualquier método y abra el **Centro de conocimientos**.

Una vez que esté visible, se percatará de que el **Centro de conocimientos** le permite realizar estas tres acciones:
* **Usar ejemplos inmediatamente**. Si quiere un ejemplo rápido de cómo funciona Synapse, elija esta opción.
* **Examinar la galería**. Esta opción le permite vincular conjuntos de datos de ejemplo y agregar código de ejemplo en forma de scripts de SQL, cuadernos y canalizaciones.
* **Visita por Synapse Studio**. Esta opción le permite realizar una breve visita por las partes básicas de Synapse Studio. Resulta útil si nunca ha usado Synapse Studio.

## <a name="exploring-use-samples-immediately"></a>Exploración: use los ejemplos de inmediato

En esta sección, hay tres elementos:
* Exploración de datos de ejemplo con Spark
* Consulta de datos con SQL
* Creación de una tabla externa con SQL

1. En el **Centro de conocimientos**, haga clic en **Use samples immediately** (Usar ejemplos inmediatamente).
1. Seleccione **Consultar datos con SQL**.
1. Haga clic en **Usar ejemplo**.
1. Se abrirá un nuevo script SQL de ejemplo.
1. Desplácese hasta la primera consulta (líneas 28 a 32) y seleccione el texto de la consulta.
1. Haga clic en Ejecutar. Solo se ejecutará el código que haya seleccionado.

## <a name="loading-more-nyc-taxi-data"></a>Carga de más datos de taxis de Nueva York

1. Vaya al **Centro de conocimiento** y haga clic en **Examinar la galería**.
1. Seleccione la pestaña **Scripts de SQL** en la parte superior.
1. Seleccione el ejemplo de ingesta de datos **Carga de conjuntos de datos de taxis de Nueva York** y haga clic en **Continuar**.
1. En **Grupo de SQL**, elija **Seleccionar un grupo existente**, seleccione **SQLPOOL1** y, a continuación, seleccione la base de datos **SQLPOOL1** que creó anteriormente.
1. Haga clic en **Abrir script**.
1. Se abrirá un nuevo script SQL de ejemplo.
1. Haga clic en **Ejecutar**
1. Se crearán varias tablas para todos los datos de taxi de Nueva York y se cargarán con el comando T-SQL COPY. Si ha creado estas tablas en los pasos de inicio rápido anteriores, seleccione y ejecute solo el código para crear y copiar las tablas que no existen.

    > [!NOTE] 
    > Al utilizar la galería de ejemplo para el script de SQL con un grupo de SQL dedicado (anteriormente SQL DW), solo podrá usar un grupo de SQL dedicado existente (anteriormente SQL DW).

## <a name="next-steps"></a>Pasos siguientes

> [!div class="nextstepaction"]
> [Incorporación de un administrador](get-started-add-admin.md)

