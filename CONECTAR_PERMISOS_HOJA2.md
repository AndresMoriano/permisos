# Conectar el aplicativo de Permisos a tu Google Sheets (Hoja 2)

Vas a usar **el mismo archivo de Google Sheets y la misma dirección** que ya
tienes funcionando para la asistencia. Solo hay que actualizar el código de
Apps Script para que sepa separar los dos tipos de registro:

| Lo que llega | Dónde se guarda |
|---|---|
| Marcas de asistencia (entrada, almuerzo, salida) | **Hoja 1** (como hasta ahora) |
| Solicitudes de permiso | **Hoja 2 → "Permisos"** (se crea sola la primera vez) |

Tu archivo es este:
https://docs.google.com/spreadsheets/d/1THKqbMgT2c06fShnhz-boSKokl1J-DykA3sA5wH4hSg/edit

---

## Paso 1 — Reemplazar el código de Apps Script

1. Abre tu Google Sheets de asistencia.
2. Menú **Extensiones → Apps Script**.
3. **Borra todo** el código que está ahí (el que pusimos antes).
4. Pega el código completo de la sección "Código nuevo" (más abajo).
5. Guarda con el ícono del disquete (o Ctrl+S).

## Paso 2 — Volver a implementar (MUY IMPORTANTE)

Los cambios de código **no se aplican solos**: hay que crear una versión nueva.

1. Arriba a la derecha: **Implementar → Gestionar implementaciones**.
2. Clic en el **ícono del lápiz** (Editar), arriba a la derecha.
3. En **Versión**, abre el desplegable y elige **Nueva versión**.
4. Verifica que **Quién tiene acceso** siga en **Cualquiera**.
5. Clic en **Implementar**.

Al editar la implementación existente (en vez de crear una nueva), **la
dirección /exec no cambia** — así sigue funcionando el aplicativo de
asistencia sin tocar nada.

## Paso 3 — Configurar el aplicativo de permisos

1. Abre el aplicativo de Permisos → pestaña **Panel del director**
   (entra con tu cédula).
2. Baja hasta **Configuración → Envío automático a Google Sheets**.
3. Pega la **misma dirección** `/exec` que usas en asistencia.
4. Clic en **Guardar** y luego en **Probar conexión**.
5. Revisa tu Google Sheets: debe aparecer una hoja nueva llamada
   **"Permisos"** con una fila de "Registro de prueba".

---

## Código nuevo (pégalo completo en Apps Script)

```javascript
function doPost(e) {
  try {
    var libro = SpreadsheetApp.getActiveSpreadsheet();
    var datos = JSON.parse(e.postData.contents);

    // ---- Solicitudes de permiso -> Hoja "Permisos" ----
    if (datos.tipoRegistro === "permiso") {
      var hp = libro.getSheetByName("Permisos");
      if (!hp) {
        var hojas = libro.getSheets();
        // usa la Hoja 2 si ya existe; si no, crea una nueva
        hp = (hojas.length > 1) ? hojas[1] : libro.insertSheet("Permisos");
        hp.setName("Permisos");
      }
      if (hp.getLastRow() === 0) {
        hp.appendRow([
          "Cédula", "Nombres", "Unidad", "Contratación", "Tipo",
          "Fecha", "Fecha fin", "Hora inicio", "Hora fin",
          "Minutos", "Días", "Causa", "Estado", "Observación", "Recibido"
        ]);
        hp.getRange(1, 1, 1, 15).setFontWeight("bold");
        hp.setFrozenRows(1);
      }
      hp.appendRow([
        datos.cedula || "",
        datos.nombres || "",
        datos.unidad || "",
        datos.contratacion || "",
        datos.tipo || "",
        datos.fecha || "",
        datos.fecha_fin || "",
        datos.hora_ini || "",
        datos.hora_fin || "",
        datos.minutos || 0,
        datos.dias || 1,
        datos.causa || "",
        datos.estado || "Solicitado",
        datos.observacion || "",
        new Date()
      ]);
      return ContentService
        .createTextOutput(JSON.stringify({ ok: true, hoja: "Permisos" }))
        .setMimeType(ContentService.MimeType.JSON);
    }

    // ---- Marcas de asistencia -> Hoja 1 (comportamiento de siempre) ----
    var ha = libro.getSheets()[0];
    if (ha.getLastRow() === 0) {
      ha.appendRow([
        "Fecha", "Hora", "Cédula", "Nombres",
        "Escuela / Unidad", "Contratación", "Marca", "Recibido"
      ]);
    }
    ha.appendRow([
      datos.fecha || "",
      datos.hora || "",
      datos.cedula || "",
      datos.nombres || "",
      datos.escuela || "",
      datos.tipo || "",
      datos.marca || "",
      new Date()
    ]);
    return ContentService
      .createTextOutput(JSON.stringify({ ok: true, hoja: "Asistencia" }))
      .setMimeType(ContentService.MimeType.JSON);

  } catch (err) {
    return ContentService
      .createTextOutput(JSON.stringify({ ok: false, error: String(err) }))
      .setMimeType(ContentService.MimeType.JSON);
  }
}
```

---

## Cómo lo usan los docentes desde el celular

1. Comparte el enlace de Netlify del aplicativo de permisos por WhatsApp o
   correo.
2. Cada docente lo abre en el navegador del celular (no instala nada).
3. Escribe su cédula → el aplicativo lo saluda por su nombre.
4. Elige el tipo de permiso, la causa, y envía.
5. La solicitud llega **de inmediato a la Hoja 2** de tu Google Sheets.

**Truco útil:** dile que agregue el enlace a la pantalla de inicio de su
celular (en Chrome: menú ⋮ → "Agregar a pantalla principal"). Le queda como
si fuera una aplicación.

---

## Punto importante sobre las aprobaciones

Cada celular guarda una copia local de lo que esa persona registró. Por eso:

- **Las solicitudes sí te llegan** a tu Google Sheets desde cualquier
  celular — ese es el registro oficial y centralizado.
- **El Panel del director** (aprobar/negar) trabaja sobre lo guardado en
  **tu** dispositivo. Si un docente solicita desde su celular, verás la
  solicitud en la Hoja 2, pero no aparecerá en tu panel para aprobarla con
  un clic.

Tienes dos formas de manejarlo, según prefieras:

**Opción A — Aprobar sobre el Google Sheets (más simple).**
Trabajas directamente en la columna **Estado** de la Hoja 2: cambias
"Solicitado" por "Aprobado" o "Negado". Ahí queda el registro oficial y de
ahí exportas para cruzar con la asistencia.

**Opción B — Centralizar en tu equipo.**
Los docentes solicitan por celular (queda en la Hoja 2 como respaldo) y tú,
además, registras las solicitudes en tu equipo desde el mismo aplicativo
para llevar el control con los botones de aprobar/negar y las estadísticas.

Si más adelante quieres que el panel lea automáticamente lo que está en el
Google Sheets, se puede hacer, pero requiere agregar un `doGet` al Apps
Script y publicar la hoja para lectura. Avísame y lo preparamos.
