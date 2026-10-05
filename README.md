# SOLUCIONES-MORA
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Panel de Dosificadores</title>
<script src="https://www.gstatic.com/firebasejs/12.11.0/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/12.11.0/firebase-database-compat.js"></script>
<style>
  :root{--fondo:#0b1626;--panel:#13233a;--tarjeta:#1a2e4a;--texto:#e8f1fb;--suave:#8fa6c4;--acento:#22d3ee;--ok:#34d399;--mal:#f87171;}
  *{box-sizing:border-box;}
  body{font-family:"Segoe UI",Arial,sans-serif;background:radial-gradient(circle at 20% 0%,#16345c 0,var(--fondo) 55%) fixed;color:var(--texto);margin:0;padding:18px;text-align:center;min-height:100vh;}
  h1{font-size:26px;margin:8px 0 4px 0;letter-spacing:.5px;}
  h1 span{color:var(--acento);}
  .sub{color:var(--suave);font-size:13px;margin-bottom:16px;}
  .barra{max-width:980px;margin:0 auto 18px auto;display:flex;flex-wrap:wrap;justify-content:center;align-items:center;gap:14px;font-size:14px;color:var(--suave);}
  .barra b{color:var(--texto);}
  .grupo{max-width:980px;margin:0 auto 18px auto;background:var(--panel);border:1px solid #23406a;border-radius:18px;padding:16px;box-shadow:0 6px 18px rgba(0,0,0,.35);transition:opacity .3s;}
  .grupo.off-linea{opacity:.55;filter:grayscale(.6);}
  .cabecera{display:flex;align-items:center;justify-content:center;gap:12px;flex-wrap:wrap;margin-bottom:14px;}
  .cabecera h2{margin:0;font-size:20px;}
  .chip{font-size:12px;padding:4px 10px;border-radius:14px;font-weight:600;}
  .chip.con{background:rgba(52,211,153,.18);color:var(--ok);} .chip.desc{background:rgba(143,166,196,.18);color:var(--suave);}
  .chip.temp{background:rgba(34,211,238,.15);color:var(--acento);}
  .contenedor{display:grid;grid-template-columns:repeat(auto-fit,minmax(200px,1fr));gap:12px;}
  .card{background:var(--tarjeta);border-radius:14px;padding:14px;}
  .card h3{margin:0 0 8px 0;font-size:13px;font-weight:600;color:var(--suave);text-transform:uppercase;letter-spacing:.8px;}
  .card p{font-size:14px;margin:6px 0;}
  input{width:100%;padding:9px;margin:6px 0;font-size:14px;border-radius:8px;border:1px solid #2f4d7a;background:#0f1d31;color:var(--texto);}
  input[type=checkbox]{width:auto;margin:0 5px 0 0;vertical-align:middle;}
  button{padding:9px 16px;border:none;border-radius:8px;cursor:pointer;font-size:14px;font-weight:600;margin:3px;color:#04121f;transition:transform .1s,filter .2s;}
  button:hover:enabled{filter:brightness(1.12);} button:active:enabled{transform:scale(.96);}
  button:disabled,input:disabled{opacity:.4;cursor:not-allowed;}
  .btn-star{background:var(--ok);} .btn-stop{background:var(--mal);} .btn-guardar{background:var(--acento);}
  .btn-salir{background:#cbd5e1;}
  .estado-on{color:var(--ok);font-weight:700;} .estado-off{color:var(--mal);font-weight:700;} .sin-conexion{color:var(--suave);font-style:italic;}
  .led{width:18px;height:18px;border-radius:50%;background:#64748b;display:inline-block;border:2px solid rgba(255,255,255,.25);}
  .led.on{background:var(--ok);box-shadow:0 0 10px 3px rgba(52,211,153,.7);}
  .led.dosif{background:var(--ok);animation:pulso 1s infinite;}
  .led.off{background:var(--mal);box-shadow:0 0 8px 2px rgba(248,113,113,.55);}
  @keyframes pulso{0%,100%{box-shadow:0 0 4px 1px rgba(52,211,153,.5);}50%{box-shadow:0 0 16px 6px rgba(52,211,153,.95);}}
  /* Temperatura */
  .temp-num{font-size:34px;font-weight:700;margin:4px 0 10px 0;}
  .temp-num small{font-size:16px;color:var(--suave);}
  .term{height:12px;border-radius:8px;background:#0f1d31;overflow:hidden;}
  .term i{display:block;height:100%;width:0;border-radius:8px;background:var(--acento);transition:width .6s,background .6s;}
  .term-esc{display:flex;justify-content:space-between;font-size:11px;color:var(--suave);margin-top:4px;}
  /* Login */
  #login{max-width:330px;margin:70px auto;background:var(--panel);border:1px solid #23406a;border-radius:20px;padding:28px;box-shadow:0 10px 30px rgba(0,0,0,.5);}
  #login .gota{font-size:44px;}
  #login button{width:100%;background:var(--acento);margin:12px 0 0 0;padding:11px;}
  #msgLogin{color:var(--mal);font-size:14px;min-height:18px;margin-top:10px;}
  #app{display:none;}
</style>
</head>
<body>

<div id="login">
  <div class="gota">💧</div>
  <h1>Dosi<span>Panel</span></h1>
  <div class="sub">Acceso autorizado</div>
  <input id="usuario" type="text" placeholder="Usuario" autocomplete="username" autocapitalize="none">
  <input id="clave" type="password" placeholder="Contraseña" autocomplete="current-password">
  <button id="btnEntrar">Entrar</button>
  <div id="msgLogin"></div>
</div>

<div id="app">
  <h1>💧 Dosi<span>Panel</span></h1>
  <div class="sub">Control de dosificadores en tiempo real</div>
  <div class="barra">
    <span id="resumen">-</span>
    <span id="avisoLectura" style="display:none;color:#fbbf24;font-weight:600;">👁 Solo lectura</span>
    <label><input type="checkbox" id="verTodos">Mostrar también los desconectados</label>
    <button class="btn-salir" id="btnSalir">Salir</button>
  </div>
  <div id="paneles"></div>
</div>

<script>
const firebaseConfig = {
  apiKey: "TU_API_KEY",
  authDomain: "TU_PROYECTO.firebaseapp.com",
  databaseURL: "https://TU_PROYECTO-default-rtdb.firebaseio.com",
  projectId: "TU_PROYECTO"
};
const DOMINIO_USUARIOS = "dosificadores.app"; // usuario "vive" => vive@dosificadores.app
let soloLectura = false;
const TOTAL_DISPOSITIVOS = 10;
const UMBRAL_CONEXION_MS = 35000;
const TEMP_MIN = 0, TEMP_MAX = 50; // rango de la barra del termometro (°C)

firebase.initializeApp(firebaseConfig);
const db = firebase.database();

// ---------------- LOGIN (solo en el HTML, sin Firebase Authentication) ----------------
// Las contraseñas NO estan en texto: solo su huella SHA-256 (sal "dosi:usuario:contraseña").
const USUARIOS = {
  vive:     { hash: "a46125d56223c089152f7d1b2f612316f3653a5bf4b6a42a8c550db79ff3cdee", lectura: false },
  mora:     { hash: "cae087af6b85000a2f7853d0bd07f8b538924b7e6fde5af87b0648d9750753e1", lectura: false },
  invitado: { hash: "2ea956abaffb13acfbddc74856502468475f32e5f759c07e591242bfca7b1551", lectura: true }
};

const elUsuario = document.getElementById("usuario");
const elClave = document.getElementById("clave");
const elMsg = document.getElementById("msgLogin");

async function sha256(texto) {
  const b = await crypto.subtle.digest("SHA-256", new TextEncoder().encode(texto));
  return Array.from(new Uint8Array(b)).map(function (x) { return x.toString(16).padStart(2, "0"); }).join("");
}

async function entrar() {
  const usuario = elUsuario.value.trim().toLowerCase();
  if (!usuario || !elClave.value) { elMsg.textContent = "Escribe usuario y contraseña."; return; }
  elMsg.textContent = "";
  try {
    const u = Object.prototype.hasOwnProperty.call(USUARIOS, usuario) ? USUARIOS[usuario] : null;
    const huella = await sha256("dosi:" + usuario + ":" + elClave.value);
    if (u && huella === u.hash) {
      try { sessionStorage.setItem("dosiUsuario", usuario); } catch (e) {}
      mostrarApp(usuario);
    } else {
      elMsg.textContent = "Usuario o contraseña incorrectos.";
    }
  } catch (e) {
    elMsg.textContent = "Este navegador no permite la verificación. Abre la página desde https o localhost.";
  }
}
document.getElementById("btnEntrar").addEventListener("click", entrar);
elClave.addEventListener("keydown", function (e) { if (e.key === "Enter") entrar(); });
document.getElementById("btnSalir").addEventListener("click", function () {
  try { sessionStorage.removeItem("dosiUsuario"); } catch (e) {}
  location.reload();
});

let panelIniciado = false;
function mostrarApp(usuario) {
  soloLectura = USUARIOS[usuario].lectura;
  document.getElementById("login").style.display = "none";
  document.getElementById("app").style.display = "block";
  document.getElementById("avisoLectura").style.display = soloLectura ? "" : "none";
  if (!panelIniciado) { panelIniciado = true; iniciarPanel(); }
}
try {
  const guardado = sessionStorage.getItem("dosiUsuario");
  if (guardado && USUARIOS[guardado]) mostrarApp(guardado);
} catch (e) {}

// ---------------- PANEL ----------------
function iniciarPanel() {
  let offsetServidor = 0;
  db.ref(".info/serverTimeOffset").on("value", function (s) { offsetServidor = s.val() || 0; evaluarTodos(); });

  const paneles = [];
  const contenedor = document.getElementById("paneles");
  const chkTodos = document.getElementById("verTodos");
  for (let i = 1; i <= TOTAL_DISPOSITIVOS; i++) paneles.push(crearPanel({ id: "dispositivo" + i, nombre: "Dosificador " + i }));
  chkTodos.addEventListener("change", evaluarTodos);
  setInterval(evaluarTodos, 5000);

  function evaluarTodos() {
    let n = 0;
    paneles.forEach(function (p) { if (p.evaluar()) n++; });
    document.getElementById("resumen").innerHTML = "<b>" + n + "</b> conectado" + (n === 1 ? "" : "s") + " de " + TOTAL_DISPOSITIVOS;
  }

  function crearPanel(d) {
    const grupo = document.createElement("div");
    grupo.className = "grupo";
    grupo.style.display = "none";
    grupo.innerHTML =
      "<div class='cabecera'><span class='led'></span><h2>" + d.nombre + "</h2>" +
      "<span class='chip desc chip-con'>Desconectado</span></div>" +
      "<div class='contenedor'>" +
        "<div class='card'><h3>Estado</h3><p class='valor-estado'>Cargando...</p>" +
          "<button class='btn-star'>star</button><button class='btn-stop'>stop</button></div>" +
        "<div class='card'><h3>Tiempo dosificando</h3><p>Actual: <b class='valor-dosif'>-</b> s</p>" +
          "<input type='number' min='1' step='1' class='input-dosif' placeholder='segundos'>" +
          "<button class='btn-guardar btn-guardar-dosif'>Actualizar</button></div>" +
        "<div class='card'><h3>Cada cuánto</h3><p>Actual: <b class='valor-cada'>-</b> s</p>" +
          "<input type='number' min='1' step='1' class='input-cada' placeholder='segundos'>" +
          "<button class='btn-guardar btn-guardar-cada'>Actualizar</button></div>" +
        "<div class='card'><h3>Temperatura del agua</h3>" +
          "<div class='temp-num'><span class='valor-temp'>--</span> <small>°C</small></div>" +
          "<div class='term'><i></i></div>" +
          "<div class='term-esc'><span>" + TEMP_MIN + "°</span><span>" + TEMP_MAX + "°</span></div></div>" +
      "</div>";
    contenedor.appendChild(grupo);

    const q = function (s) { return grupo.querySelector(s); };
    const led = q(".led"), chipCon = q(".chip-con");
    const ref = db.ref("dosificadores/" + d.id);
    let datos = null;

    ref.on("value", function (s) { datos = s.val(); pintar(); evaluarTodos(); }, function () {});

    function conectado() {
      return !!datos && datos.ultimaVez != null && (Date.now() + offsetServidor - datos.ultimaVez) < UMBRAL_CONEXION_MS;
    }

    function pintar() {
      const elEstado = q(".valor-estado");
      if (!datos) { elEstado.textContent = "Sin datos aun"; elEstado.className = "valor-estado sin-conexion"; return; }
      const enc = datos.estado === "encendido";
      elEstado.textContent = (enc ? "ENCENDIDO" : "APAGADO") + (enc ? (datos.dosificando ? " (dosificando)" : " (en espera)") : "");
      elEstado.className = "valor-estado " + (enc ? "estado-on" : "estado-off");
      q(".valor-dosif").textContent = datos.tiempoDosificando != null ? datos.tiempoDosificando : "-";
      q(".valor-cada").textContent = datos.cadaCuanto != null ? datos.cadaCuanto : "-";

      const barra = q(".term i");
      if (datos.temperaturaAgua != null) {
        const t = Number(datos.temperaturaAgua);
        const pct = Math.max(0, Math.min(100, (t - TEMP_MIN) / (TEMP_MAX - TEMP_MIN) * 100));
        q(".valor-temp").textContent = t.toFixed(1);
        barra.style.width = pct + "%";
        barra.style.background = "hsl(" + Math.round(200 - pct * 2) + ",85%,55%)"; // azul frio -> rojo caliente
      } else {
        q(".valor-temp").textContent = "--";
        barra.style.width = "0";
      }
    }

    function evaluar() {
      const con = conectado();
      const enc = !!datos && datos.estado === "encendido";
      grupo.style.display = (con || chkTodos.checked) ? "" : "none";
      grupo.classList.toggle("off-linea", !con);
      chipCon.textContent = con ? "Conectado" : "Desconectado";
      chipCon.className = "chip chip-con " + (con ? "con" : "desc");
      led.className = "led" + (!con ? "" : (enc ? (datos.dosificando ? " dosif" : " on") : " off"));
      grupo.querySelectorAll("button,input").forEach(function (el) { el.disabled = !con || soloLectura; });
      return con;
    }

    q(".btn-star").addEventListener("click", function () { ref.update({ estado: "encendido" }); });
    q(".btn-stop").addEventListener("click", function () { ref.update({ estado: "apagado" }); });
    q(".btn-guardar-dosif").addEventListener("click", function () {
      const v = parseInt(q(".input-dosif").value, 10); if (v > 0) ref.update({ tiempoDosificando: v });
    });
    q(".btn-guardar-cada").addEventListener("click", function () {
      const v = parseInt(q(".input-cada").value, 10); if (v > 0) ref.update({ cadaCuanto: v });
    });
    return { evaluar: evaluar };
  }
}
</script>
</body>
</html>

