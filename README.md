# SOLUCIONES-MORA-PRUEBA-1
PAGUINA PARA VER DISPOCITIVOS DE ALIMENTADORES DE CAMARON
<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1">
<title>Panel Dosificador</title>
<script src="https://cdn.jsdelivr.net/npm/mqtt@5/dist/mqtt.min.js"></script>
<style>
  body{font-family:Arial,Helvetica,sans-serif;background:#fff;margin:0;padding:12px;color:#222}
  .panel{background:#f1f1f1;padding:10px;margin:0 auto 16px;max-width:760px}
  .panel h2{text-align:center;margin:4px 0 10px;font-size:20px}
  .row{display:flex;gap:12px;flex-wrap:wrap;justify-content:center}
  .card{background:#fff;border-radius:10px;box-shadow:0 2px 6px rgba(0,0,0,.2);padding:12px;
        flex:1 1 180px;min-width:180px;text-align:center}
  .card h3{margin:0 0 8px;font-size:16px}
  .act{font-size:14px;margin-bottom:6px}
  .st{font-weight:bold;font-size:14px;margin-bottom:8px}
  .on{color:#2e9e3f}.off{color:#d32f2f}.na{color:#888}
  button{border:0;border-radius:4px;color:#fff;padding:7px 14px;cursor:pointer;font-size:14px}
  .b-on{background:#43a047}.b-off{background:#e53935}.b-up{background:#1976d2;margin-top:8px}
  input{width:90%;border:0;border-bottom:1px solid #555;padding:6px;font-size:14px;outline:none}
  .offline{opacity:.55}
  #login input{display:block;width:100%;box-sizing:border-box;margin:6px 0;background:#fff}
  #login label{font-size:13px}
  #status{text-align:center;font-size:13px;margin:6px 0}
</style>
</head>
<body>

<div class="panel" id="login">
  <h2>Conexión al broker</h2>
  <input id="url" placeholder="wss://TU-CLUSTER.s1.eu.hivemq.cloud:8884/mqtt">
  <input id="user" placeholder="Usuario MQTT" autocomplete="username">
  <input id="pass" type="password" placeholder="Contraseña MQTT" autocomplete="current-password">
  <label><input type="checkbox" id="rec" style="width:auto;display:inline"> Recordar en este navegador</label>
  <div style="text-align:center;margin-top:8px"><button class="b-up" onclick="conectar()">Conectar</button></div>
  <div id="status" class="na">Desconectado</div>
</div>

<div id="root"></div>

<script>
const N = 10;
const BASE = 'dosif';   // debe coincidir con el prefijo del firmware
const root = document.getElementById('root');
let client = null;
const st = Array.from({length: N}, () => ({online: false, estado: false, tiempo: '-', cada: '-'}));

for (let i = 0; i < N; i++) {
  root.insertAdjacentHTML('beforeend', `
  <div class="panel offline" id="p${i}">
    <h2>Dispositivo ${i + 1}</h2>
    <div class="row">
      <div class="card">
        <h3>Estado</h3>
        <div class="st na" id="st${i}">SIN CONEXIÓN</div>
        <button class="b-on" onclick="cmd(${i},1)">Start</button>
        <button class="b-off" onclick="cmd(${i},0)">Stop</button>
      </div>
      <div class="card">
        <h3>Tiempo dosificando (s)</h3>
        <div class="act">Actual: <span id="t${i}">-</span> s</div>
        <input type="number" min="1" id="ti${i}" placeholder="segundos">
        <br><button class="b-up" onclick="upd(${i},'tiempo')">Actualizar</button>
      </div>
      <div class="card">
        <h3>Cada cuánto (s)</h3>
        <div class="act">Actual: <span id="c${i}">-</span> s</div>
        <input type="number" min="1" id="ci${i}" placeholder="segundos">
        <br><button class="b-up" onclick="upd(${i},'cada')">Actualizar</button>
      </div>
    </div>
  </div>`);
}

function setStatus(txt, cls) {
  const s = document.getElementById('status');
  s.textContent = txt; s.className = cls;
}

function render(i) {
  const d = st[i];
  const el = document.getElementById('st' + i);
  const p = document.getElementById('p' + i);
  if (!d.online) {
    el.textContent = 'SIN CONEXIÓN'; el.className = 'st na'; p.classList.add('offline');
  } else {
    el.textContent = d.estado ? 'ENCENDIDO' : 'APAGADO';
    el.className = 'st ' + (d.estado ? 'on' : 'off');
    p.classList.remove('offline');
  }
  document.getElementById('t' + i).textContent = d.tiempo;
  document.getElementById('c' + i).textContent = d.cada;
}

function conectar() {
  const url = document.getElementById('url').value.trim();
  const user = document.getElementById('user').value.trim();
  const pass = document.getElementById('pass').value;
  if (!url) { alert('Escribe la URL del broker (wss://...)'); return; }

  if (document.getElementById('rec').checked) {
    localStorage.setItem('mqttcfg', JSON.stringify({url, user, pass}));
  } else {
    localStorage.removeItem('mqttcfg');
  }

  if (client) client.end(true);
  setStatus('Conectando...', 'na');

  client = mqtt.connect(url, {
    username: user,
    password: pass,
    clientId: 'web-' + Math.random().toString(16).slice(2, 10),
    reconnectPeriod: 3000,
    connectTimeout: 8000
  });

  client.on('connect', () => {
    setStatus('Conectado', 'on');
    ['estado', 'online', 'tiempo', 'cada'].forEach(c => client.subscribe(`${BASE}/+/${c}`));
  });
  client.on('reconnect', () => setStatus('Reconectando...', 'na'));
  client.on('close', () => {
    setStatus('Desconectado', 'off');
    st.forEach((d, i) => { d.online = false; render(i); });
  });
  client.on('error', e => setStatus('Error: ' + e.message, 'off'));

  client.on('message', (topic, payload) => {
    const [, idStr, campo] = topic.split('/');
    const i = parseInt(idStr);
    if (isNaN(i) || i < 0 || i >= N) return;
    const v = payload.toString();
    if (campo === 'online') st[i].online = (v === '1');
    else if (campo === 'estado') st[i].estado = (v === '1');
    else if (campo === 'tiempo') st[i].tiempo = v;
    else if (campo === 'cada') st[i].cada = v;
    render(i);
  });
}

function pub(topic, val) {
  if (!client || !client.connected) { alert('No estás conectado al broker'); return false; }
  client.publish(topic, String(val), {retain: true, qos: 1});
  return true;
}

function cmd(i, v) { pub(`${BASE}/${i}/cmd`, v); }

function upd(i, campo) {
  const el = document.getElementById((campo === 'tiempo' ? 'ti' : 'ci') + i);
  const v = parseInt(el.value);
  if (!v || v < 1) { alert('Escribe un valor válido en segundos'); return; }
  if (pub(`${BASE}/${i}/${campo}`, v)) el.value = '';
}

// Cargar datos recordados
try {
  const cfg = JSON.parse(localStorage.getItem('mqttcfg') || 'null');
  if (cfg) {
    document.getElementById('url').value = cfg.url || '';
    document.getElementById('user').value = cfg.user || '';
    document.getElementById('pass').value = cfg.pass || '';
    document.getElementById('rec').checked = true;
    conectar();
  }
} catch (e) {}
</script>
</body>
</html>

