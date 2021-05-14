---
title: Administración de usuarios y roles en una aplicación de Azure IoT Central | Microsoft Docs
description: Como administrador, aquí se indica la forma de administrar usuarios y roles en su aplicación de Azure IoT Central
author: lmasieri
ms.author: lmasieri
ms.date: 04/16/2021
ms.topic: how-to
ms.service: iot-central
services: iot-central
manager: corywink
ms.openlocfilehash: cff8830d180b0c234e54f7578ed9fafafeb598f0
ms.sourcegitcommit: 79c9c95e8a267abc677c8f3272cb9d7f9673a3d7
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/19/2021
ms.locfileid: "107719176"
---
# <a name="manage-users-and-roles-in-your-iot-central-application"></a>Administrar usuarios y roles en la aplicación de IoT Central

En este artículo se describe cómo los administradores pueden agregar, editar y eliminar usuarios en la aplicación de Azure IoT Central. También se describe cómo administrar roles en la aplicación.

Para tener acceso a la sección **Administración** y poder usarla, debe disponer del rol **Administrador** para la aplicación Azure IoT Central. Si crea una aplicación de Azure IoT Central, se le asigna automáticamente el rol **Administrator** para esa aplicación.

## <a name="add-users"></a>Agregar usuarios

Cada usuario debe tener una cuenta de usuario antes de poder iniciar sesión y acceder a la aplicación. IoT Central admite actualmente cuentas de Microsoft y cuentas de Azure Active Directory, pero no grupos de Azure Active Directory.

Para más información, consulte la [ayuda de la cuenta de Microsoft](https://support.microsoft.com/products/microsoft-account?category=manage-account) y la [Guía de inicio rápido: Adición de nuevos usuarios a Azure Active Directory](../../active-directory/fundamentals/add-users-azure-active-directory.md).

1. Para agregar un usuario a una aplicación de IoT Central, vaya a la página **Usuarios** de la sección **Administración**.

  :::image type="content" source="media/howto-manage-users-roles/manage-users-pnp.png" alt-text="Administración de usuarios":::

1. Para agregar un usuario, en la página **Usuarios**, seleccione **+ Agregar usuario**.

1. Seleccione un rol para el usuario en el menú desplegable **Rol**. Más información sobre los roles en la sección [Administración de roles](#manage-roles) de este artículo.

  :::image type="content" source="media/howto-manage-users-roles/add-user-pnp.png" alt-text="Adición de un usuario y selección de un rol":::

  > [!NOTE]
  > Un usuario que tenga un rol personalizado que le conceda permisos para agregar otros usuarios solo puede agregar usuarios a un rol que tenga los mismos permisos que el suyo o unos inferiores.

  > [!NOTE]
  > Si se elimina un usuario de Azure Active Directory y, después, se vuelve a agregar, el usuario no podrá iniciar sesión en la aplicación IoT Central. Para volver a habilitar el acceso, el administrador de la aplicación debe eliminar al usuario y en la aplicación y volver a agregarlo también.

### <a name="edit-the-roles-that-are-assigned-to-users"></a>Modificar los roles asignados a usuarios

Los roles no se pueden cambiar una vez asignados. Para cambiar el rol asignado a un usuario, elimine el usuario y agréguelo de nuevo con un rol diferente.

> [!NOTE]
> Los roles asignados son específicos de la aplicación de IoT Central y no se pueden administrar desde Azure Portal.

## <a name="delete-users"></a>Eliminación de usuarios

Para eliminar usuarios, seleccione una o más casillas en la página **Usuarios**. A continuación, seleccione **Eliminar**.

## <a name="manage-roles"></a>Administrar roles

Los roles le permiten controlar qué usuarios de la organización tienen permiso para realizar varias tareas en IoT Central. Hay tres roles integrados que se pueden asignar a los usuarios de la aplicación. También puede [crear roles personalizados](#create-a-custom-role) si necesita un control más preciso.

> [!div class="mx-imgBorder"]
> ![Administración de la selección de roles](media/howto-manage-users-roles/manage-roles-pnp.png)

### <a name="administrator"></a>Administrador

Los usuarios del rol **Administrador** pueden administrar y controlar todas las partes de la aplicación, incluida la facturación.

El usuario que crea una aplicación se asigna automáticamente al rol **Administrador**. Siempre debe haber al menos un usuario en el rol **Administrador**.

### <a name="builder"></a>Generador

Los usuarios del rol **Generador** pueden administrar todas las partes de la aplicación, pero no pueden realizar cambios en las pestañas Administración o Exportación de datos continua.

### <a name="operator"></a>Operator

Los usuarios de rol **Operador** pueden supervisar el estado y el mantenimiento del dispositivo. No se les permite realizar cambios en las plantillas de dispositivo ni administrar la aplicación. Esto significa que los operadores pueden agregar y eliminar dispositivos, administrar conjuntos de dispositivos y ejecutar análisis y trabajos.

## <a name="create-a-custom-role"></a>Crear un rol personalizado

Si la solución requiere controles de acceso más precisos, puede crear roles con conjuntos de permisos personalizados. Para crear un rol personalizado, vaya a la página **Roles** de la sección **Administration** (Administración) de la aplicación: A continuación, seleccione **+ New role** (+ Nuevo rol) y agregue un nombre y una descripción para el rol. Seleccione los permisos que necesita el rol y, luego, seleccione **Save** (Guardar).

Puede agregar usuarios al rol personalizado de la misma manera que agrega usuarios a un rol integrado.

> [!div class="mx-imgBorder"]
> ![Creación de un rol personalizado](media/howto-manage-users-roles/create-custom-role-pnp.png)

### <a name="custom-role-options"></a>Opciones de roles personalizados

Al definir un rol personalizado, elige el conjunto de permisos que se concede a un usuario si es miembro del rol. Algunos permisos dependen de otros. Por ejemplo, si agrega a un rol el permiso **Update personal dashboards** (Actualizar paneles personales), se agrega automáticamente el permiso **View personal dashboards** (Ver paneles personales). En las tablas siguientes se resumen los permisos disponibles (y sus dependencias), que puede usar al crear roles personalizados.

#### <a name="managing-devices"></a>Administración de dispositivos

**Permisos de la plantilla de dispositivo**

| Nombre | Dependencias |
| ---- | -------- |
| Ver | None     |
| Administrar | Ver <br/> Otras dependencias: Ver instancias de dispositivo  |
| Control total | Ver, administrar <br/> Otras dependencias: Ver instancias de dispositivo |

**Permisos de instancia de dispositivo**

| Nombre | Dependencias |
| ---- | -------- |
| Ver | None <br/> Otras dependencias: Ver plantillas de dispositivo y grupos de dispositivos |
| Actualizar | Ver <br/> Otras dependencias: Ver plantillas de dispositivo y grupos de dispositivos  |
| Crear | Ver <br/> Otras dependencias:  Ver plantillas de dispositivo y grupos de dispositivos  |
| Eliminar | Ver <br/> Otras dependencias: Ver plantillas de dispositivo y grupos de dispositivos  |
| Ejecutar comandos | Actualizar, ver <br/> Otras dependencias: Ver plantillas de dispositivo y grupos de dispositivos  |
| Visualización de datos sin procesar | Ver <br/> Otras dependencias: Ver plantillas de dispositivo y grupos de dispositivos  |
| Control total | Ver, actualizar, crear, eliminar, ejecutar comandos, ver datos sin procesar <br/> Otras dependencias: Ver plantillas de dispositivo y grupos de dispositivos  |

**Permisos de grupos de dispositivos**

| Nombre | Dependencias |
| ---- | -------- |
| Ver | None <br/> Otras dependencias: Ver plantillas de dispositivo e instancias de dispositivo |
| Actualizar | Ver <br/> Otras dependencias: Ver plantillas de dispositivo e instancias de dispositivo   |
| Crear | Ver, actualizar <br/> Otras dependencias:  Ver plantillas de dispositivo e instancias de dispositivo   |
| Eliminar | Ver <br/> Otras dependencias:  Ver plantillas de dispositivo e instancias de dispositivo  |
| Control total | Ver, actualizar, crear, eliminar <br/> Otras dependencias: Ver plantillas de dispositivo e instancias de dispositivo |

**Permisos de administración de conectividad de dispositivos**

| Nombre | Dependencias |
| ---- | -------- |
| Leer instancias | None <br/> Otras dependencias: Ver plantillas de dispositivo, grupos de dispositivos e instancias de dispositivo |
| Administrar instancia | Leer instancias <br /> Otras dependencias: Ver plantillas de dispositivo, grupos de dispositivos e instancias de dispositivo |
| Lectura global | None   |
| Administración global | Lectura global |
| Control total | Leer instancias, administrar instancias, lectura global, administración global <br/> Otras dependencias: Ver plantillas de dispositivo, grupos de dispositivos e instancias de dispositivo |

**Permisos de trabajos**

| Nombre | Dependencias |
| ---- | -------- |
| Ver | None <br/> Otras dependencias: Ver plantillas de dispositivo, instancias de dispositivo y grupos de dispositivos |
| Actualizar | Ver <br/> Otras dependencias: Ver plantillas de dispositivo, instancias de dispositivo y grupos de dispositivos |
| Crear | Ver, actualizar <br/> Otras dependencias:  Ver plantillas de dispositivo, instancias de dispositivo y grupos de dispositivos |
| Eliminar | Ver <br/> Otras dependencias:  Ver plantillas de dispositivo, instancias de dispositivo y grupos de dispositivos |
| Execute | Ver <br/> Otras dependencias: Ver plantillas de dispositivo, instancias de dispositivo y los grupos de dispositivos; actualizar instancias de dispositivo; ejecutar comandos en instancias de dispositivo |
| Control total | Ver, actualizar, crear, eliminar, ejecutar <br/> Otras dependencias:  Ver plantillas de dispositivo, instancias de dispositivo y los grupos de dispositivos; actualizar instancias de dispositivo; ejecutar comandos en instancias de dispositivo |

**Permisos de reglas**

| Nombre | Dependencias |
| ---- | -------- |
| Ver | None <br/> Otras dependencias: Ver plantillas de dispositivo |
| Actualizar | Ver <br/> Otras dependencias: Ver plantillas de dispositivo |
| Crear | Ver, actualizar <br/> Otras dependencias:  Ver plantillas de dispositivo |
| Eliminar | Ver <br/> Otras dependencias: Ver plantillas de dispositivo |
| Control total | Ver, actualizar, crear, eliminar <br/> Otras dependencias: Ver plantillas de dispositivo |

#### <a name="managing-the-app"></a>Administración de la aplicación

**Permisos de configuración de la aplicación**

| Nombre | Dependencias |
| ---- | -------- |
| Ver | None     |
| Actualizar | Ver   |
| Copiar | Ver <br/> Otras dependencias: Ver plantillas de dispositivo, instancias de dispositivo, grupos de dispositivos, paneles, exportación de datos, personalización de marca, vínculos de ayuda, roles personalizados, reglas |
| Eliminar | Ver   |
| Control total | Ver, actualizar, copiar, eliminar <br/> Otras dependencias: Ver plantillas de dispositivo, grupos de dispositivos, paneles de la aplicación, exportación de datos, personalización de marca, vínculos de ayuda, roles personalizados, reglas |

**Permisos de exportación de plantillas de aplicación**

| Nombre | Dependencias |
| ---- | -------- |
| Ver | None     |
| Exportación | Ver <br/> Otras dependencias:  Ver plantillas de dispositivo, instancias de dispositivo, grupos de dispositivos, paneles, exportación de datos, personalización de marca, vínculos de ayuda, roles personalizados, reglas |
| Control total | Ver, exportar <br/> Otras dependencias:  Ver plantillas de dispositivo, grupos de dispositivos, paneles de la aplicación, exportación de datos, personalización de marca, vínculos de ayuda, roles personalizados, reglas |

**Permisos de carga de archivos de dispositivo**

| Nombre | Dependencias |
| ---- | -------- |
| Ver | None     |
| Administrar | Ver   |
| Control total | Ver, administrar |

**Permisos de facturación**

| Nombre | Dependencias |
| ---- | -------- |
| Administrar | None     |
| Control total | Administrar |

#### <a name="managing-users-and-roles"></a>Administración de usuarios y roles

**Permisos de roles personalizados**

| Nombre | Dependencias |
| ---- | -------- |
| Ver | None |
| Actualizar | Ver |
| Crear | Ver, actualizar |
| Eliminar | Ver |
| Control total | Ver, actualizar, crear, eliminar |

**Permisos de administración de usuarios**

| Nombre | Dependencias |
| ---- | -------- |
| Ver | None <br/> Otras dependencias: Ver roles personalizados |
| Sumar | Ver <br/> Otras dependencias:  Ver roles personalizados |
| Eliminar | Ver <br/> Otras dependencias:  Ver roles personalizados |
| Control total | Ver, agregar, eliminar <br/> Otras dependencias:  Ver roles personalizados |

> [!NOTE]
> Un usuario que tenga un rol personalizado que le conceda permisos para agregar otros usuarios solo puede agregar usuarios a un rol que tenga los mismos permisos que el suyo o unos inferiores.

#### <a name="customizing-the-app"></a>Personalización de la aplicación

**Permisos de paneles de la aplicación**

| Nombre | Dependencias |
| ---- | -------- |
| Ver | None     |
| Actualizar | Ver   |
| Crear | Ver, actualizar |
| Eliminar | Ver   |
| Control total | Ver, actualizar, crear, eliminar |

**Permisos de paneles personales**

| Nombre | Dependencias |
| ---- | -------- |
| Ver | None     |
| Actualizar | Ver   |
| Crear | Ver, actualizar   |
| Eliminar | Ver   |
| Control total | Ver, actualizar, crear, eliminar |

**Permisos de personalización de marca, iconos de favoritos y colores**

| Nombre | Dependencias |
| ---- | -------- |
| Ver | None     |
| Actualizar | Ver   |
| Control total | Ver, actualizar |

**Permisos de vínculos de ayuda**

| Nombre | Dependencias |
| ---- | -------- |
| Ver | None     |
| Actualizar | Ver   |
| Control total | Ver, actualizar |

#### <a name="extending-the-app"></a>Extensión de la aplicación

**Permisos de exportación de datos**

| Nombre | Dependencias |
| ---- | -------- |
| Ver | None     |
| Actualizar | Ver   |
| Crear | Ver, actualizar  |
| Eliminar | Ver   |
| Control total | Ver, actualizar, crear, eliminar |

**Permisos de token de API**

| Nombre | Dependencias |
| ---- | -------- |
| Ver | None  <br/> Otras dependencias: Ver roles personalizados |
| Crear | Ver <br/> Otras dependencias: Ver roles personalizados |
| Eliminar | Ver <br/> Otras dependencias: Ver roles personalizados |
| Control total | Ver, crear, eliminar <br/> Otras dependencias: Ver roles personalizados |

## <a name="next-steps"></a>Pasos siguientes

Ahora que ha aprendido a administrar usuarios y roles en la aplicación de Azure IoT Central, el siguiente paso sugerido es aprender a [administrar su factura](howto-view-bill.md).