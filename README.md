# Gestor de Vacaciones Aux

Aplicación web para organizar y consultar las solicitudes de vacaciones de un equipo. Permite elegir semanas siguiendo un turno anual rotativo y consultar las reservas compartidas en Supabase.

## Funcionalidades

- Acceso de trabajadores mediante correo electrónico y PIN.
- Vista de las 52 semanas del año en curso, con fechas y disponibilidad.
- Máximo de cuatro semanas por trabajador.
- Capacidad de una reserva por semana, salvo entre las semanas 23 y 39, que admiten dos.
- Turno de elección secuencial: al completar las cuatro semanas o ceder el turno, pasa al siguiente trabajador.
- Consulta del orden rotativo por años. El orden inicial se toma de `orden_inicial` y rota cíclicamente a partir de 2026.
- Funciones de gestor para registrar reservas fuera del turno y eliminar reservas existentes.
- Descarga de las reservas en CSV compatible con Excel.
- Interfaz adaptable a móviles y equipos de escritorio.

## Requisitos

- Un proyecto de [Supabase](https://supabase.com/) con las tablas y columnas descritas en [Configuración de Supabase](#configuración-de-supabase).
- Un navegador moderno con conexión a Internet. La aplicación carga Supabase JS, Roboto y Material Icons desde servicios externos.
- Para desarrollo local, Python 3 o una extensión de servidor estático como Live Server.

No se requiere un proceso de compilación para la versión actual: la aplicación está implementada en `index.html`.

## Puesta en marcha local

1. Clona el repositorio y entra en su carpeta:

   ```bash
   git clone https://github.com/PAJAROEULEN/GESTOR-VACACIONES-AUX.git
   cd GESTOR-VACACIONES-AUX
   ```

2. Inicia un servidor web estático:

   ```bash
   python -m http.server 8000
   ```

3. Abre <http://localhost:8000> en el navegador.

También puedes abrir `index.html` con Live Server. Servir la página por HTTP local es preferible a abrirla directamente como archivo.

## Configuración de Supabase

La aplicación se conecta directamente a Supabase desde el navegador. La URL del proyecto y la clave publicable se configuran en las constantes `SUPABASE_URL` y `SUPABASE_ANON_KEY` de `index.html`. Si vas a usar otro proyecto, actualiza esos valores antes de desplegar.

La aplicación espera estas tablas y columnas:

| Tabla | Columnas utilizadas |
| --- | --- |
| `trabajadores` | `nombre`, `mail`, `pin`, `role`, `orden_inicial` |
| `solicitudes` | `ano`, `semana`, `nombre`, `fecha_registro` |
| `turnos` | `ano`, `nombre`, `estado`, `turno` |

Consideraciones para preparar la base de datos:

- Añade un registro en `trabajadores` por persona. `orden_inicial` determina el orden base; asigna `role = gestor` a las cuentas de gestión.
- `fecha_registro` puede establecerse con un valor predeterminado en la base de datos; se muestra en el CSV si está informado.
- La aplicación hace `upsert` de turnos con conflicto en `ano,nombre`; configura una restricción única para esas columnas.
- Configura las políticas de Row Level Security (RLS) y los permisos para las operaciones que necesita la aplicación. La clave publicable se distribuye al navegador: no la consideres un secreto ni uses la clave `service_role` en el cliente.
- El acceso de la aplicación comprueba el correo y el PIN en la tabla `trabajadores`; no utiliza Supabase Auth. El navegador recuerda ambos valores en `localStorage` hasta que se cierra la sesión. Tenlo en cuenta al definir el modelo de seguridad y al usar equipos compartidos.

## Despliegue

Al ser una aplicación estática, puede servirse desde cualquier hosting compatible con HTML estático. Para publicarla con GitHub Pages:

1. Comprueba que la configuración de Supabase apunta al proyecto adecuado.
2. En GitHub, abre **Settings → Pages**.
3. Selecciona la rama que contiene la aplicación y la carpeta raíz (`/root`) como origen.
4. Guarda la configuración y espera a que GitHub Pages publique el sitio.

La aplicación requiere conexión a Internet para comunicarse con Supabase y cargar las fuentes e iconos externos.

## Exportación

El botón de descarga del encabezado genera un archivo CSV separado por punto y coma, con codificación compatible con Excel. Incluye año, semana, fechas, trabajador y fecha de registro. La exportación recoge las solicitudes registradas en la base de datos.
