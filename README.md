# SOLUCIONES-MORA-PRUEBA-1
PAGUINA PARA VER DISPOCITIVOS DE ALIMENTADORES DE CAMARON`
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Panel de Dosificadores</title>

<!-- ==== SDK de Firebase (version compat, no necesita herramientas de compilacion) ==== -->
<script src="https://www.gstatic.com/firebasejs/12.11.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/12.11.0/firebase-database-compat.js"></script>

<style>
  body{font-family:Arial;background:#f2f2f2;text-align:center;padding:15px;margin:0;}
  h1{color:#222;font-size:22px;margin:10px 0 20px 0;}
  .grupo{max-width:900px;margin:0 auto 20px auto;background:#ececec;border-radius:14px;padding:18px 12px;}
  .grupo h2{font-size:19px;color:#1a1a1a;font-weight:bold;margin:0 0 14px 0;}
  .contenedor{display:flex;flex-wrap:wrap;justify-content:center;gap:12px;}
  .card{background:white;border-radius:10px;padding:12px;box-shadow:0 2px 5px rgba(0,0,0,0.2);flex:1 1 200px;max-width:230px;}
  .card h3{color:#333;margin-top:0;font-size:15px;}
  .card p{font-size:14px;margin:6px 0;}
  input{width:85%;padding:6px;margin:4px;font-size:14px;box-sizing:border-box;}
  button{padding:8px 14px;border:none;border-radius:5px;cursor:pointer;font-size:14px;margin:3px;color:white;}
  .btn-star{background:#4CAF50;} .btn-star:hover{background:#45a049;}
  .btn-stop{background:#e53935;} .btn-stop:hover{background:#c62828;}
  .btn-guardar{background:#1976d2;} .btn-guardar:hover{background:#1565c0;}
  .estado-on{color:#2e7d32;font-weight:bold;}
  .estado-off{color:#c62828;font-weight:bold;}
  .sin-conexion{color:#9e9e9e;font-style:italic;}
</style>
</head>
<body>

<h1>Panel de Dosificadores</h1>
<div id="paneles"></div>

<script>
// ============================================================
//  CONFIGURA AQUI TU PROYECTO DE FIREBASE
//  (lo obtienes en Configuracion del proyecto > General > Tus apps)
// ============================================================
const firebaseConfig = {
  apiKey: "TU_API_KEY",
  authDomain: "TU_PROYECTO.firebaseapp.com",
  databaseURL: "https://TU_PROYECTO-default-rtdb.firebaseio.com",
  projectId: "TU_PROYECTO"
};

firebase.initializeApp(firebaseConfig);
const db = firebase.database();

// ============================================================
//  DISPOSITIVOS A MOSTRAR (deben coincidir con la ruta que usa
//  cada ESP8266 en su variable RUTA_DISPOSITIVO)
// ============================================================
const dispositivos = [
  { id: "dispositivo1", nombre: "Dosificador 1" },
  { id: "dispositivo2", nombre: "Dosificador 2" },
  { id: "dispositivo3", nombre: "Dosificador 3" }
];

const contenedorPrincipal = document.getElementById("paneles");

dispositivos.forEach(function (dispositivo) {
  crearPanel(dispositivo);
});

function crearPanel(dispositivo) {
  // ---- Construye el HTML del grupo (una vez) ----
  const grupo = document.createElement("div");
  grupo.className = "grupo";
  grupo.innerHTML =
    "<h2>" + dispositivo.nombre + "</h2>" +
    "<div class='contenedor'>" +
      "<div class='card'>" +
        "<h3>Estado</h3>" +
        "<p class='valor-estado'>Cargando...</p>" +
        "<button class='btn-star'>star</button>" +
        "<button class='btn-stop'>stop</button>" +
      "</div>" +
      "<div class='card'>" +
        "<h3>Tiempo dosificando (s)</h3>" +
        "<p>Actual: <b class='valor-dosif'>-</b> s</p>" +
        "<input type='number' min='1' step='1' class='input-dosif' placeholder='segundos'>" +
        "<br><button class='btn-guardar btn-guardar-dosif'>Actualizar</button>" +
      "</div>" +
      "<div class='card'>" +
        "<h3>Cada cuanto (s)</h3>" +
        "<p>Actual: <b class='valor-cada'>-</b> s</p>" +
        "<input type='number' min='1' step='1' class='input-cada' placeholder='segundos'>" +
        "<br><button class='btn-guardar btn-guardar-cada'>Actualizar</button>" +
      "</div>" +
      "<div class='card valor-tarjeta-temp' style='display:none'>" +
        "<h3>Temperatura agua</h3>" +
        "<p style='font-size:20px;'><b class='valor-temp'>-</b> &deg;C</p>" +
      "</div>" +
    "</div>";
  contenedorPrincipal.appendChild(grupo);

  const refDispositivo = db.ref("dosificadores/" + dispositivo.id);

  // ---- Escucha en tiempo real: cualquier cambio (de la pagina o del ESP8266) se refleja solo ----
  refDispositivo.on("value", function (snapshot) {
    const datos = snapshot.val();
    const elEstado = grupo.querySelector(".valor-estado");
    const elDosif = grupo.querySelector(".valor-dosif");
    const elCada = grupo.querySelector(".valor-cada");
    const elTarjetaTemp = grupo.querySelector(".valor-tarjeta-temp");
    const elTemp = grupo.querySelector(".valor-temp");

    if (!datos) {
      elEstado.textContent = "Sin datos aun";
      elEstado.className = "valor-estado sin-conexion";
      return;
    }

    const encendido = datos.estado === "encendido";
    let texto = encendido ? "ENCENDIDO" : "APAGADO";
    if (encendido && datos.dosificando) texto += " (dosificando)";
    else if (encendido) texto += " (en espera)";

    elEstado.textContent = texto;
    elEstado.className = "valor-estado " + (encendido ? "estado-on" : "estado-off");
    elDosif.textContent = datos.tiempoDosificando != null ? datos.tiempoDosificando : "-";
    elCada.textContent = datos.cadaCuanto != null ? datos.cadaCuanto : "-";

    if (datos.temperaturaAgua != null) {
      elTarjetaTemp.style.display = "";
      elTemp.textContent = Number(datos.temperaturaAgua).toFixed(2);
    }
  });

  // ---- Botones: escriben directo en Firebase, el ESP8266 los toma en su siguiente consulta ----
  grupo.querySelector(".btn-star").addEventListener("click", function () {
    refDispositivo.update({ estado: "encendido" });
  });

  grupo.querySelector(".btn-stop").addEventListener("click", function () {
    refDispositivo.update({ estado: "apagado" });
  });

  grupo.querySelector(".btn-guardar-dosif").addEventListener("click", function () {
    const valor = parseInt(grupo.querySelector(".input-dosif").value, 10);
    if (valor > 0) refDispositivo.update({ tiempoDosificando: valor });
  });

  grupo.querySelector(".btn-guardar-cada").addEventListener("click", function () {
    const valor = parseInt(grupo.querySelector(".input-cada").value, 10);
    if (valor > 0) refDispositivo.update({ cadaCuanto: valor });
  });
}
</script>
</body>
</html>
