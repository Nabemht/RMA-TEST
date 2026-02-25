# Project: Panel de Administración RMA - EcoXtrem

## Overview

Panel de gestión de devoluciones (RMA) para EcoXtrem, empresa de movilidad eléctrica. Permite al equipo administrativo gestionar el ciclo completo de una devolución: desde la recepción en almacén hasta la resolución técnica en taller. Incluye tabla central de devoluciones, panel de detalle con chat de trazabilidad, vista de taller para técnicos, generación de PDFs (etiquetas Zebra y fichas A4) y dashboard de estadísticas. Estética dark mode corporativa EcoXtrem.

**Data source: Supabase (connected)**

- Tabla `devoluciones` (PK: `id_unico` formato R-DDMMAA-XXXX)
- Tabla `comentarios_rma` (FK: `rma_id` → `devoluciones.id_unico`)
- Tabla `movimientos_rma` (FK: `rma_id` → `devoluciones.id_unico`)
- Realtime habilitado en las 3 tablas

<phase number="1" title="Dashboard Principal con Tabla de Devoluciones">

Entregar el layout dark mode con sidebar de navegación y la tabla central de devoluciones con panel de detalle al hacer clic en una fila.

#### Key Features

- Layout dark mode con sidebar estilo SaaS (#0f172a) y branding EcoXtrem
- Tabla de devoluciones con columnas: ID único, cliente, tracking, transportista, estado, fecha
- Tags de colores por estado (Gris=Pendiente, Naranja=En Taller, Verde=Finalizado, Rojo=Fraude)
- Miniaturas de fotos en la tabla
- Panel lateral derecho de detalle al seleccionar una fila (ficha técnica: datos cliente, tracking, foto grande)
- Barra de búsqueda global (por ID, cliente, tracking)

#### Tasks

- [x] Set up Supabase data provider and define resource types for devoluciones, comentarios_rma, and movimientos_rma
- [x] Create dark mode layout with sidebar navigation (Inicio/Devoluciones, Taller, Estadísticas) using EcoXtrem branding (#0f172a sidebar)
- [x] Build devoluciones list page with table showing id_unico, cliente, tracking, transportista, estado (color tags), fecha_entrada, and foto thumbnails
- [x] Add search/filter bar for global search by ID, cliente, or tracking
- [x] Create detail side panel that opens on row click showing full ficha técnica (client data, tracking, transportista, peso, tipo_producto, foto grande, diagnóstico inicial)

#### Notes

- Data source: Supabase (connected) — tablas: devoluciones, comentarios_rma, movimientos_rma
- Estados y colores: PENDIENTE (gris), EN_TALLER (naranja), FINALIZADO (verde), FRAUDE (rojo)
- El campo `id_unico` es la PK (formato R-DDMMAA-XXXX), NO el campo `id`
- Las fotos están almacenadas como URLs en `foto_link` (Google Drive)
- Sidebar debe incluir logo EcoXtrem, fondo oscuro #0f172a

</phase>

<phase number="2" title="Sistema de Chat y Gestión de Estados">

Añadir el timeline de chat estilo WhatsApp en el panel de detalle y la funcionalidad para cambiar estados de las devoluciones.

#### Key Features

- Timeline de comentarios con estilo visual tipo WhatsApp
- Colores por rol: Nabe (Azul), Joseph (Naranja), Postventa/Carolina (Morado)
- Input para añadir nuevos comentarios
- Selector para cambiar estado de la devolución
- Trazabilidad completa de quién hizo qué y cuándo

#### Tasks

- [x] Build chat timeline component in the detail panel showing comentarios_rma ordered by fecha, styled like WhatsApp bubbles
- [x] Implement color coding per user role (Nabe=Azul, Joseph=Naranja, Postventa=Morado, default=Gris)
- [x] Add input field to post new comments to comentarios_rma with user selector (Nabe, Joseph, Carolina, Postventa)
- [x] Add estado change functionality with dropdown (PENDIENTE, EN_TALLER, FINALIZADO, FRAUDE) that updates devoluciones and auto-posts a system comment in the chat
- [x] Add ubicacion field editor to update physical location (RECEPCION, MESA_1, MESA_2, TALLER, ESTANTERIA, EXPEDICION)
- [x] Enable Supabase realtime subscription so chat and status changes appear live without refresh

#### Notes

- Tabla comentarios_rma: rma_id (FK), usuario, mensaje, fecha
- Al cambiar estado, registrar automáticamente un comentario tipo sistema: "Estado cambiado a X por [usuario]"
- Al cambiar ubicación, registrar en tabla movimientos_rma (ubicacion_anterior, ubicacion_nueva, usuario)

</phase>

<phase number="3" title="Vista de Taller para Técnicos">

Vista filtrada específica para técnicos que muestra solo las devoluciones en estado EN_TALLER, con herramientas de diagnóstico.

#### Key Features

- Vista filtrada mostrando solo devoluciones con estado EN_TALLER
- Cards o tabla optimizada para el flujo del técnico
- Acceso rápido a ficha técnica y chat
- Botón rápido para marcar como "Reparado/Finalizado"

#### Tasks

- [x] Create Taller page with table/cards filtered to estado=EN_TALLER, showing id_unico, cliente, tipo_producto, ubicacion, diagnostico_inicial
- [x] Add detail view for Taller with foto, diagnóstico, and chat timeline (reuse components from Phase 2)
- [x] Add quick action buttons: "Marcar como Finalizado", "Solicitar Repuesto" (posts comment automatically)
- [x] Add editable fields for diagnóstico técnico and repuestos usados
- [x] Show time elapsed since entrada for priority sorting (highlight items older than 48h)

#### Notes

- Reutilizar componentes de chat y detalle de la Fase 2
- Joseph es el técnico principal — su vista debe ser directa y sin distracciones
- Al marcar "Finalizado", el estado cambia y se registra en el chat automáticamente

</phase>

<phase number="4" title="Sistema de Impresión PDF">

Generación de PDFs en el navegador: etiquetas Zebra (100x150mm) con código de barras y fichas de taller A4.

#### Key Features

- Etiqueta Zebra 100x150mm con código de barras Code128 del id_unico
- Ficha de taller A4 completa para pegar en caja
- Impresión masiva (selección múltiple en tabla)
- Sin drivers necesarios, generación al vuelo en navegador

#### Tasks

- [x] Implement Zebra label PDF generation (100x150mm) with Code128 barcode of id_unico, remitente, fecha, and notas resumidas
- [x] Implement A4 workshop sheet PDF with full ficha técnica: datos cliente, tracking, fotos, diagnóstico, and blank spaces for "Diagnóstico" and "Repuestos usados"
- [x] Add print button in detail panel for single item (both Zebra and A4 options)
- [x] Add bulk selection in devoluciones table with "Imprimir Etiquetas Zebra" button for multiple items
- [x] Add print preview before sending to printer

#### Notes

- Usar librería JS para generar PDFs en el navegador (jsPDF o similar)
- Código de barras Code128 para el campo id_unico
- Etiqueta Zebra: tamaño 100x150mm, incluir código barras + remitente + notas cortas
- Ficha A4: formato completo para técnico, con espacios en blanco para escribir

</phase>

<phase number="5" title="Dashboard de Estadísticas y KPIs">

Panel de estadísticas con métricas clave del sistema de devoluciones.

#### Key Features

- KPIs principales: total devoluciones, pendientes, en taller, finalizadas, fraudes
- Gráficos de evolución temporal
- Distribución por transportista y tipo de producto
- Tiempo medio de resolución

#### Tasks

- [x] Create Estadísticas page with KPI cards: total devoluciones, count by estado (PENDIENTE, EN_TALLER, FINALIZADO, FRAUDE)
- [x] Add bar/line chart showing devoluciones received per day/week/month
- [x] Add pie/donut chart for distribution by transportista and tipo_producto
- [x] Add average resolution time metric (fecha_entrada to estado=FINALIZADO)
- [x] Add filters by date range for all statistics

#### Notes

- Los KPIs deben reflejar datos en tiempo real de Supabase
- Usar componentes de gráficos compatibles con shadcn (recharts o similar)
- Mantener el dark mode consistente con el resto del panel

</phase>

<phase number="6" title="Sistema de Autenticación por PIN + Panel de Administración">

Al entrar a la app por primera vez aparece una pantalla de PIN. Cada PIN corresponde a un usuario configurable. Panel de Admin (contraseña: TNC) accesible desde la sidebar.

#### Key Features

- Pantalla de login estilo SaaS profesional: logo EcoXtrem centrado, campo de texto para PIN (letras, números, caracteres), botón de acceso — NO teclado numérico móvil
- Cada PIN identifica a un usuario (nombre, rol, color de chat)
- Panel de Admin (contraseña: TNC) accesible desde la sidebar
- Gestión de usuarios: añadir/editar/eliminar usuarios con PIN y rol
- Gestión de permisos por usuario: qué pestañas son visibles (Devoluciones, Taller, Estadísticas, Nueva Incidencia)
- Gestión de talleres: añadir/editar/eliminar talleres/almacenes
- Todo almacenado en localStorage (no requiere backend extra)

#### Tasks

- [ ] Create PIN login screen shown on first visit (or when no session): EcoXtrem logo centered on dark background, professional text input field for PIN (alphanumeric), "Acceder" button, error message on wrong PIN — web SaaS style, NOT mobile keypad
- [ ] Implement PIN-to-user mapping: validate PIN, store active session user in localStorage, show user name/avatar in sidebar header
- [ ] Build Admin panel page (route /admin, password prompt TNC): tabs for Usuarios, Talleres, Sistema
- [ ] Admin: Users tab — CRUD for users (name, PIN, role, chat color, visible tabs permissions)
- [ ] Admin: Talleres tab — CRUD for workshops/almacenes (name, location, assigned technician)
- [ ] Apply tab visibility permissions: sidebar nav items shown/hidden based on logged-in user's permissions
- [ ] Add "Cerrar sesión / Cambiar usuario" option in sidebar footer that returns to PIN screen

#### Notes

- Usuarios predefinidos iniciales: Nabe (admin, PIN configurable), Joseph (técnico), Carolina (postventa), Postventa PIN=121314
- Contraseña de acceso al panel Admin: TNC (hardcoded, no configurable)
- Todo el estado de auth en localStorage: { activeUser: { name, role, permissions } }
- El chat ya no mostrará selector de usuario — usará automáticamente el usuario logueado
- La pantalla de login debe usar el logo EcoXtrem (imagen del artifact), fondo dark #0f172a, campo input estándar con placeholder "Introduce tu PIN"

</phase>

<phase number="7" title="Mejoras de UI: Galería de Imágenes, Filtros y Fixes">

Mejoras visuales y funcionales en la tabla principal: galería de imágenes con flechas, panel de filtros lateral, fix de ordenación por ID, y estado visible en etiqueta Zebra.

#### Key Features

- Galería de imágenes en el panel de detalle con flechas para navegar entre fotos (campo `fotos` array o múltiples URLs)
- Panel de filtros en la tabla: filtrar por estado, almacén/ubicación, última actualización
- Fix: ordenación por `id_unico` en la tabla principal
- Etiqueta Zebra actualizada para mostrar el estado actual de la incidencia

#### Tasks

- [ ] Fix sorting by id_unico column in devoluciones table (currently not working)
- [ ] Add image gallery to detail panel: support multiple image URLs (comma-separated or array field `fotos`), left/right arrow navigation, current index indicator (e.g. "2/4")
- [ ] Add filter sidebar/panel to devoluciones list: filter chips for estado (multi-select), ubicación/almacén (multi-select), and date range for última actualización
- [ ] Update Zebra label PDF to include estado badge/text prominently on the label
- [ ] Add active filter indicators (badges showing applied filters, clear all button)

#### Notes

- El campo de fotos adicionales puede ser `fotos` (texto con URLs separadas por coma) o usar `foto_link` como primera imagen
- Los filtros deben combinarse con la búsqueda global existente (AND logic)
- La ordenación por id_unico debe usar orden alfanumérico correcto

</phase>

<phase number="8" title="Crear Nueva Incidencia + Hoja RMA Mejorada">

Formulario para registrar manualmente una nueva incidencia RMA antes de que llegue el producto. Al crear, se puede imprimir la etiqueta Zebra y la hoja RMA directamente.

#### Key Features

- Página "Nueva Incidencia" con formulario completo
- Generación automática del ID único (formato R-DDMMAA-XXXX, autoincremental)
- Imprimir etiqueta Zebra + hoja RMA desde la pantalla de creación
- Hoja RMA A4 rediseñada: espacio grande con el enlace de foto (texto, no imagen), más profesional
- Asignación de taller/almacén al crear la incidencia

#### Tasks

- [ ] Create "Nueva Incidencia" page with form: cliente\*, tracking, transportista, peso, tipo_producto, diagnóstico_inicial, taller/almacén asignado, notas
- [ ] Implement auto-generation of id_unico (R-DDMMAA-XXXX format, query last ID of the day and increment)
- [ ] On save: insert to Supabase devoluciones, show success with the generated ID, offer print buttons
- [ ] Update A4 RMA sheet: add large text block showing foto_link URL (no image rendering), improve layout to be more professional and printer-friendly
- [ ] Add "Nueva Incidencia" to sidebar navigation (respects user permissions from Phase 6)

#### Notes

- El formato del ID: R- + fecha DDMMAA + - + número secuencial de 4 dígitos (0001, 0002...)
- El taller asignado viene del catálogo de talleres creado en el Admin (Phase 6)
- La hoja RMA debe ser clara para el técnico: todos los datos visibles, sin necesidad de acceder al sistema

</phase>

<phase number="9" title="Vista de Técnico con Incidencias Propias por Taller">

Cada técnico ve únicamente las incidencias asignadas a su taller. Al mover una incidencia al taller de un técnico con su PIN, solo él la ve en su vista.

#### Key Features

- Técnicos ven solo incidencias de su taller asignado (filtro automático por taller del usuario)
- Al asignar incidencia a taller, el técnico ve esa incidencia en su vista Taller
- Campo `taller_asignado` en devoluciones para filtrar por técnico
- Admins (Nabe) ven todas las incidencias sin filtro

#### Tasks

- [ ] Add `taller_asignado` field to devoluciones resource type and detail panel (editable dropdown using talleres catalog)
- [ ] Filter Taller page by `taller_asignado` matching the logged-in user's assigned taller (from user config in Phase 6)
- [ ] Admin/Nabe users bypass the taller filter and see all incidencias
- [ ] Add taller_asignado column to devoluciones main table (visible for admins)
- [ ] When creating new incidencia (Phase 8), allow assigning to a specific taller

#### Notes

- El filtro por taller usa el campo `taller_asignado` en la tabla `devoluciones`
- El taller del usuario logueado se configura en el Admin panel (Phase 6)
- Si un usuario no tiene taller asignado (admin), ve todo
- Reutiliza el catálogo de talleres del Admin (Phase 6)

</phase>
