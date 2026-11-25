# TITO HSEQ · Reporte ACIS

App del Grupo ANAVA para reportar **actos y condiciones inseguras (ACIS)** por QR en cada centro de trabajo, hacerles seguimiento hasta el cierre y ver indicadores por centro, por empresa y nacional.

- **Reportantes:** escanean el QR del centro. No necesitan cuenta, el reporte puede ser anónimo y se le puede adjuntar una foto.
- **Gestores HSEQ:** entran con correo y contraseña. Asignan responsable, definen la acción, cierran con evidencia y verifican si fue eficaz.
- **Consulta:** gerentes y directores ven la gestión y el dashboard de sus centros.
- **Indicadores:** % de gestión de cierre, % de cierre oportuno, vencidos, días promedio de cierre, índice de reporte y % de eficacia, por año y mes. Los datos se exportan a Excel (CSV).

## Tecnología
- Una sola página (`index.html`) publicada en GitHub Pages.
- Firebase Authentication: anónima para los reportantes, correo y contraseña para los gestores.
- Cloud Firestore en el plan gratuito Spark. Las fotos se comprimen y se guardan en Firestore, así que no se necesita Storage ni el plan Blaze.

## Instalación (una sola vez)

### 1. Firebase
1. En https://console.firebase.google.com, crea el proyecto **tito-acis** (Google Analytics es opcional).
2. **Compilación → Authentication → Comenzar.** En «Método de acceso» habilita **Anónimo** y **Correo electrónico/contraseña**.
3. **Authentication → Usuarios → Agregar usuario:** crea primero la cuenta de la administradora principal, con el mismo correo que va en `SUPERADMIN`.
4. **Authentication → Configuración → Dominios autorizados:** agrega `nataliaricardorincon-hash.github.io`.
5. **Compilación → Firestore Database → Crear base de datos.** Elige la ubicación `southamerica-east1` (São Paulo), que no se puede cambiar después, y el modo de producción.
6. **Firestore → Reglas:** pega el contenido de `firestore.rules`, con el correo de la administradora en `%%SUPERADMIN%%`, y presiona **Publicar**.
7. **Configuración del proyecto (⚙️) → Tus apps → Web (</>):** registra la app «TITO ACIS» y copia el objeto `firebaseConfig`. Va en `FIREBASE_CONFIG`, dentro de `index.html`.

### 2. GitHub Pages
1. Crea el repositorio público **tito-acis** y sube `index.html` y la carpeta `img/`.
2. **Settings → Pages → Branch: main / root → Save.**
3. La app queda en `https://nataliaricardorincon-hash.github.io/tito-acis/`.

### 3. Primer ingreso
1. Entra a la app y presiona **Ingresar** con la cuenta de la administradora.
2. Presiona **Inicializar catálogo**: se crean las 6 empresas y los 18 centros.
3. En **Configuración**, ajusta para cada centro la empresa, el número de trabajadores, el responsable por defecto, los plazos y las metas.
4. En **Usuarios**, crea a los gestores HSEQ de cada centro (rol Gestor) y a los gerentes que solo consultan (rol Consulta).
5. En **Códigos QR**, descarga el afiche de cada centro o imprímelos todos.

## Imágenes de Tito
Sube tus imágenes 3D (PNG con fondo transparente, unos 600 px) a la carpeta `img/` con estos nombres. Mientras una imagen no exista, se muestra el Tito vectorial.

| Archivo | Dónde aparece | Pose sugerida |
|---|---|---|
| `tito-inicio.png` | Encabezado y formulario de reporte | Saludando |
| `tito-gracias.png` | Pantalla de reporte enviado | Pulgar arriba |
| `tito-login.png` | Ingreso de gestores | Con tablet o carpeta |
| `tito-gestion.png` | Gestión y seguimiento | Revisando una lista de chequeo |
| `tito-dashboard.png` | Dashboard | Señalando una gráfica |
| `tito-qr.png` | Afiches QR | Señalando el código |
| `tito-alerta.png` | Vencidos y errores | Con señal de alerta |

## Reglas del negocio
- **Consecutivo:** `ACIS-<CÓDIGO CENTRO>-<AÑO>-0001`, independiente por centro y por año.
- **Fecha compromiso automática según el riesgo** (configurable): alto 3 días, medio 10 días, bajo 20 días calendario.
- **Cierre obligatorio** con descripción de la evidencia y nombre de quien verificó. La foto y la eficacia son opcionales.
- **Anulado:** para duplicados o pruebas, con motivo obligatorio. Los anulados no cuentan en los indicadores.
- **Trazabilidad:** cada cambio queda registrado con fecha y usuario.
- **Empresa histórica:** cada reporte guarda la empresa del centro en el momento en que se registró.

## Cierres trimestrales (respaldo y liberación de la base)
Así la base se mantiene liviana dentro del plan gratuito y se conserva el histórico.

- **Cuándo:** al terminar cada trimestre (31 mar, 30 jun, 30 sep y 31 dic). También antes de tiempo, si la base se acerca al límite configurado (2.500 reportes por defecto).
- **Avisos automáticos:** al abrir el dashboard, la administradora ve una alerta en estos casos:
  - el trimestre anterior no se ha cerrado;
  - la base llegó al 80 % del límite (o lo superó);
  - quedó un cierre a medias.
- **Qué se archiva:** los reportes **cerrados o anulados** con fecha hasta el corte, incluidos los de trimestres anteriores que se cerraron tarde. Los abiertos y en gestión siguen vivos hasta cerrarse y salen en el siguiente cierre.
- **Paso 1. Respaldo ZIP:** contiene el Excel (CSV) con trazabilidad, el JSON completo, el resumen de indicadores, todas las fotos de reporte y de cierre nombradas por ID, y un archivo LEEME. Se guarda en la carpeta HSEQ de SharePoint, con la retención que defina el CQ-P-01.
- **Paso 2. Archivar:** solo se habilita después de descargar el respaldo. Hay que marcar la confirmación y escribir el nombre del periodo (p. ej. `T1-2027`).
  - La app guarda los indicadores agregados (año/mes/centro) en `resumenes` y luego borra los reportes y sus fotos.
  - El dashboard sigue mostrando el histórico completo.
- **Si algo se interrumpe:** el cierre queda «Sin terminar» en el historial. El botón **Reanudar** termina el borrado sin contar dos veces.
- **El consecutivo no se reinicia:** los IDs nunca se repiten.
- **Consulta de un reporte archivado:** se abre el respaldo ZIP del periodo correspondiente.

## Límites del plan gratuito (Spark)
El plan gratuito permite 50.000 lecturas y 20.000 escrituras al día, y 1 GiB de almacenamiento. Cada vez que se abre el dashboard se lee el año completo de reportes. Como referencia, con unos 3.000 reportes al año y 15 consultas diarias se usa aproximadamente el 90 % de las lecturas diarias.

Cada foto ocupa entre 80 y 150 KB, así que caben unas 7.000 fotos. Para el volumen de un año el plan gratuito es suficiente.
