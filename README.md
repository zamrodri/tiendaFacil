# Handoff: Panel de Control de Ventas (Tienda Fácil)

## Overview
Sitio web de administración para la app móvil de ventas "Tienda Fácil". Permite iniciar sesión, gestionar usuarios y roles, monitorear las ventas del equipo de vendedores, dar seguimiento a créditos pendientes de cobro, y registrar una venta manualmente desde escritorio.

## About the Design Files
Los archivos de este paquete son **referencias de diseño creadas en HTML** (prototipos de look & behavior), no código de producción para copiar tal cual. La tarea es **recrear este diseño en el stack real del proyecto**: backend en **.NET Core (C#)** con API REST, y el frontend en el framework que el equipo decida (React, Blazor, etc.) consumiendo esa API. Usa estas páginas como especificación visual y funcional exacta.

## Fidelity
**Alta fidelidad (hifi)**: colores, tipografía, espaciados e interacciones están definidos y deben respetarse tal cual al recrear la UI.

## Design Tokens
- **Colores**
  - Verde primario oscuro: `#0F5132`
  - Verde primario: `#16A34A`
  - Verde acento: `#22C55E`
  - Fondo general: `#EDEFE9`
  - Fondo de tarjetas: `#FFFFFF`
  - Borde de tarjetas: `#E3E5DC`
  - Texto principal: `#1A1F16`
  - Texto secundario: `#8A8F85` / `#4B5045`
  - Placeholder/gris: `#9A9F92`
  - Estado activo (verde): `#16A34A` · Estado inactivo (gris): `#9A9F92`
  - Rol Administrador: fondo `#FCE9E9` texto `#B91C1C`
  - Rol Supervisor: fondo `#FEF3E0` texto `#B45309`
  - Rol Vendedor: fondo `#EAF7EF` texto `#0F5132`
  - Crédito pendiente: `#B45309` · Crédito pagado: `#16A34A`
- **Tipografía**: Plus Jakarta Sans (500, 600, 700, 800), Google Fonts. Tamaños: títulos de sección 26px/800, KPIs 26px/800, cuerpo 13–15px/600-700, labels uppercase 11.5–12.5px/700.
- **Radios**: tarjetas 16px, inputs/botones 10–14px, chips/pills 100px (full).
- **Sombra**: ninguna; se usan bordes de 1–1.5px en vez de sombras.
- **Espaciado**: contenedor de página `padding: 32px 40px`, gap entre tarjetas 16px, gap interno 8–14px.

## Screens / Views

### 1. Login
- Layout de dos columnas: panel izquierdo (44% ancho, min 420px) con gradiente verde diagonal (`#0F5132` → `#16A34A` → `#22C55E`), logo, título y mensaje de marca. Panel derecho centrado con formulario (máx 360px).
- Campos: Usuario (texto), Contraseña (password). Enlace "¿Olvidaste tu contraseña?" alineado a la derecha.
- Botón "Ingresar": deshabilitado (gris `#A9AD9F`, opacity 0.7) hasta llenar ambos campos; habilitado en verde `#16A34A`.
- Nota de demo bajo el botón (quitar en producción).

### 2. Monitoreo (Dashboard)
- Header con título + selector de periodo (Hoy / 7 días / 30 días) tipo segmented control.
- 4 tarjetas KPI en grid: Ventas totales, Ticket promedio, Vendedor top, Vendedores activos.
- Grid inferior 1.7fr / 0.75fr:
  - **Ventas recientes**: tabla con columnas Vendedor (ancha, con avatar circular de iniciales + nombre), Fecha, Pago, Total. Máx 8 filas, orden descendente por fecha.
  - **Ranking de vendedores**: lista compacta con avatar, nombre, total, y barra de progreso proporcional al top vendedor.

### 3. Usuarios y roles
- Header + botón "Nuevo usuario" (verde, ícono +).
- 3 tarjetas de leyenda de roles (Administrador, Supervisor, Vendedor) con descripción corta de permisos.
- Tabla de usuarios: Nombre (avatar + nombre), Correo, Rol (chip de color), Estado (toggle activo/inactivo con punto de color), Acciones (editar).
- Modal de alta/edición: Nombre, Correo, selector de Rol (3 botones), botones Cancelar/Guardar.

### 4. Créditos
- Header + filtro segmentado (Pendientes / Pagados / Todos).
- 3 tarjetas KPI: Pendiente de cobro (naranja `#B45309`), Cobrado (verde), Total en crédito.
- Tabla: Cliente, Vendedor (avatar+nombre), Fecha, Monto, Estado (punto + label; si está pagado muestra "por {nombre de quien canceló}" debajo), Acción ("Marcar pagado" solo si está pendiente).
- Al marcar como pagado: se registra quién lo hizo (usuario autenticado) y se refleja en la tabla.

### 5. Nueva venta
- Layout 1.5fr / 1fr: catálogo de productos a la izquierda (buscador + chips de categoría + grid 2 columnas de productos), carrito a la derecha.
- Carrito: lista de items con controles +/- cantidad, campo Cliente (opcional), selector de Vendedor (3 botones por iniciales), selector de Método de pago (Efectivo/Tarjeta/Transferencia/Crédito), total, botón "Confirmar venta" (deshabilitado sin método de pago o carrito vacío).
- Pantalla de confirmación: check animado, vendedor + método de pago, total, botón "Registrar otra venta".

## Navigation
Sidebar fijo izquierdo (250px, fondo `#0F5132`): logo, 4 ítems de navegación (Monitoreo, Usuarios y roles, Créditos, Nueva venta) con estado activo resaltado, y footer con avatar de usuario + botón de cerrar sesión.

## State Management (para el frontend)
- `loggedIn`, `username`, `password`
- `screen`: 'dashboard' | 'users' | 'credits' | 'newsale'
- `period`: 'hoy' | '7d' | '30d' (filtra ventas para dashboard)
- `users[]`: lista de usuarios con rol y estado activo
- `creditFilter`: 'pendiente' | 'pagado' | 'todos'
- `creditStatus`: mapa ventaId → { paid, paidBy, paidAt }
- Carrito de nueva venta: items, vendedor, cliente, método de pago

## API Contract sugerido (.NET Core)

### Auth
- `POST /api/auth/login` → { username, password } → { token, user: { id, name, role } }

### Usuarios y roles
- `GET /api/usuarios`
- `POST /api/usuarios` → { name, email, role }
- `PUT /api/usuarios/{id}` → { name, email, role }
- `PATCH /api/usuarios/{id}/estado` → { active }
- `GET /api/roles`

### Ventas
- `GET /api/ventas?periodo=hoy|7d|30d` → lista + agregados (total, promedio, top vendedor, ranking)
- `POST /api/ventas` → { vendedorId, cliente?, metodoPago, items: [{ productoId, cantidad }] }
- `GET /api/ventas/recientes?limite=8`

### Créditos
- `GET /api/creditos?estado=pendiente|pagado|todos`
- `POST /api/creditos/{ventaId}/pagar` → registra `paidBy` (usuario autenticado del token) y `paidAt`

### Productos
- `GET /api/productos?categoria=&busqueda=`
- `GET /api/categorias`

## Assets
Sin imágenes externas; solo íconos SVG inline dibujados a mano (flechas, checks, avatares de iniciales generados por color determinístico por vendedor/rol).

## Files
- `Panel de Control.dc.html` — fuente del diseño completo (todas las pantallas)
- `Panel de Control - Standalone.html` — versión autocontenida para abrir sin dependencias, útil como referencia visual exacta
