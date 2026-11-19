/* =====================================================================
   PUENTES · lógica del sitio
   Las convocatorias NO están acá: se editan en convocatorias.json.
   ===================================================================== */

/* ---------------------------------------------------------------------
   CONFIGURACIÓN: completá estos datos antes de difundir el sitio.
   --------------------------------------------------------------------- */
const CONFIG = {
  suscripcionUrl: "",   // enlace al formulario del servicio de boletines
  contactoEmail: ""     // correo para recibir convocatorias de la comunidad
};

/* Portales oficiales que se muestran en la sección "Portales". */
const PORTALES = [
  { nombre: "Relaciones Internacionales · Udelar", texto: "Movilidad estudiantil y docente, Erasmus+, AUGM y convenios.", url: "https://udelar.edu.uy/internacionales/" },
  { nombre: "Comisión Académica de Posgrado", texto: "Becas de la Udelar para docentes y para finalizar posgrados.", url: "https://www.posgrados.udelar.edu.uy" },
  { nombre: "ANII", texto: "Becas de posgrado nacionales y en el exterior, fondos de investigación.", url: "https://www.anii.org.uy" },
  { nombre: "Portal de becas de la AUCI", texto: "Becas que ofrecen otros gobiernos y organismos. Requiere registro.", url: "https://portaldebecas.auci.gub.uy" },
  { nombre: "Becas en el exterior · MEC", texto: "Oportunidades de formación difundidas por el Ministerio de Educación y Cultura.", url: "https://www.gub.uy/ministerio-educacion-cultura/politicas-y-gestion/becas-oportunidades-formacion-exterior" },
  { nombre: "Catálogo Erasmus Mundus", texto: "Maestrías conjuntas financiadas por la Unión Europea.", url: "https://www.eacea.ec.europa.eu" }
];

/* Valores válidos para los campos "para" y "areas" de convocatorias.json. */
const PARA = [
  ["todos", "Todos"], ["grado", "Estudiantes de grado"], ["egresados", "Egresados"],
  ["posgrado", "Posgrado"], ["docentes", "Docentes"], ["investigadores", "Investigadores"], ["funcionarios", "Funcionarios TAS"]
];
const AREAS = [
  ["todas", "Todas las áreas"], ["salud", "Salud"], ["social", "Social y Artística"], ["tecnologias", "Tecnologías, Naturaleza y Hábitat"]
];
const ESTADOS = [["todas", "Todas"], ["pronto", "Cierran pronto"], ["abierta", "Abiertas"]];

/* ---------------------------------------------------------------------
   Utilidades
   --------------------------------------------------------------------- */
const ETIQUETA = { pronto: "Cierra pronto", abierta: "Abierta" };
const PARA_TXT = Object.fromEntries(PARA);
const AREA_TXT = { ...Object.fromEntries(AREAS), todas: "Todas las disciplinas" };
const DIA = 86400000;

function hoy() { const d = new Date(); return new Date(d.getFullYear(), d.getMonth(), d.getDate()); }
function fecha(s) { const [y, m, d] = s.split("-").map(Number); return new Date(y, m - 1, d); }
function diasHasta(s) { return Math.round((fecha(s) - hoy()) / DIA); }
const fmtLargo = new Intl.DateTimeFormat("es-UY", { day: "numeric", month: "long", year: "numeric" });
const fmtCorto = new Intl.DateTimeFormat("es-UY", { day: "numeric", month: "short" });
const fmtMes = new Intl.DateTimeFormat("es-UY", { month: "short" });
function largo(s) { return fmtLargo.format(fecha(s)); }
function corto(d) { return fmtCorto.format(d).replace(".", ""); }
function esc(s) { return String(s).replace(/[&<>"']/g, ch => ({ "&": "&amp;", "<": "&lt;", ">": "&gt;", '"': "&quot;", "'": "&#39;" }[ch])); }
function diasTxt(n) { return n === 0 ? "hoy" : n === 1 ? "1 día" : n + " días"; }
function sinTildes(t) { return t.toLowerCase().normalize("NFD").replace(/[̀-ͯ]/g, ""); }
function $(id) { return document.getElementById(id); }

function estadoDe(c) {
  if (c.cierre) { const n = diasHasta(c.cierre); if (n < 0) return "cerrada"; return n <= 14 ? "pronto" : "abierta"; }
  return "abierta";
}

/* ---------------------------------------------------------------------
   Estado de la página
   --------------------------------------------------------------------- */
let items = [];
let actualizado = null;
const filtro = { para: "todos", area: "todas", estado: "todas", q: "" };
try {
  const g = JSON.parse(localStorage.getItem("puentes-filtro") || "null");
  if (g) {
    if (PARA_TXT[g.para]) filtro.para = g.para;
    if (AREAS.some(a => a[0] === g.area)) filtro.area = g.area;
    if (ESTADOS.some(e => e[0] === g.estado)) filtro.estado = g.estado;
  }
} catch (e) {}

/* ---------------------------------------------------------------------
   Secciones
   --------------------------------------------------------------------- */
function renderNext() {
  const con = items.filter(c => c.cierre).sort((a, b) => diasHasta(a.cierre) - diasHasta(b.cierre));
  const el = $("next");
  if (!con.length) { el.innerHTML = '<span class="label">Próximo cierre</span><h2>No hay cierres con fecha fija por ahora.</h2><p>Mirá todas las convocatorias abiertas más abajo.</p>'; return; }
  const c = con[0], n = diasHasta(c.cierre);
  el.innerHTML =
    '<span class="label">Próximo cierre</span>' +
    '<div class="count"><b>' + (n === 0 ? "Hoy" : n) + '</b>' + (n === 0 ? "" : '<span>' + (n === 1 ? "día" : "días") + '</span>') + '</div>' +
    '<h2>' + esc(c.titulo) + '</h2>' +
    '<p>Cierra el ' + esc(largo(c.cierre)) + '. Para ' + esc(c.para.map(p => PARA_TXT[p].toLowerCase()).join(" y ")) + '.</p>' +
    '<p><a href="#' + c.id + '">Ver detalles</a></p>';
}

function renderStats() {
  const k = e => items.filter(c => c.est === e).length;
  $("stats").innerHTML =
    '<div class="stat pronto"><b>' + k("pronto") + '</b><span>cierran en 14 días o menos</span></div>' +
    '<div class="stat abierta"><b>' + k("abierta") + '</b><span>abiertas con más tiempo</span></div>' +
    '<div class="stat"><b>' + esc(corto(fecha(actualizado))) + '</b><span>última actualización</span></div>';
  $("updated").textContent = "Para la comunidad Udelar · Actualizado el " + largo(actualizado);
  $("foot-updated").textContent = "Información relevada el " + largo(actualizado) + ". Verificá siempre en la fuente oficial.";
}

function renderTimeline() {
  const con = items.filter(c => c.cierre);
  const maxN = Math.max(30, ...con.map(c => diasHasta(c.cierre)));
  const span = Math.ceil((maxN + 10) / 30) * 30;
  const t0 = hoy();
  const pos = n => (n / span) * 96 + 2;
  let h = '<div class="rail"></div>';
  for (let m = 1; m <= 12; m++) {
    const d = new Date(t0.getFullYear(), t0.getMonth() + m, 1);
    const n = Math.round((d - t0) / DIA);
    if (n > span) break;
    h += '<div class="tick" style="left:' + pos(n) + '%"><span>' + esc(fmtMes.format(d).replace(".", "")) + '</span></div>';
  }
  h += '<div class="mark today" style="left:' + pos(0) + '%" title="Hoy"></div>';
  const grupos = [];
  con.sort((a, b) => diasHasta(a.cierre) - diasHasta(b.cierre)).forEach(c => {
    const g = grupos.find(g => g.cierre === c.cierre);
    g ? g.items.push(c) : grupos.push({ cierre: c.cierre, items: [c] });
  });
  grupos.forEach((g, i) => {
    const n = diasHasta(g.cierre), x = pos(n), est = g.items[0].est;
    const align = x < 14 ? "transform:translateX(-10px);text-align:left" : x > 86 ? "transform:translateX(calc(-100% + 10px));text-align:right" : "";
    const nombres = g.items.map(c => '<a href="#' + c.id + '">' + esc(c.corto || c.titulo.split(" · ")[0]) + '</a>').join("<br>");
    h += '<div class="mark ' + est + '" style="left:' + x + '%" title="' + esc(g.items.map(c => c.titulo).join(" / ")) + '"></div>';
    h += '<div class="tag ' + (i % 2 ? "down" : "up") + '" style="left:' + x + '%;' + align + '">' + nombres + '<b>' + esc(corto(fecha(g.cierre))) + ' · ' + esc(diasTxt(n)) + '</b></div>';
  });
  $("track").innerHTML = h;
  $("timeline-range").textContent = "El punto oscuro es hoy · próximos " + span + " días";
}

function chips(id, opts, key) {
  const el = $(id);
  el.innerHTML = opts.map(([v, t]) => '<button type="button" class="chip" data-v="' + v + '" aria-pressed="' + (filtro[key] === v) + '">' + esc(t) + '</button>').join("");
  el.addEventListener("click", e => {
    const b = e.target.closest("button"); if (!b) return;
    filtro[key] = b.dataset.v;
    el.querySelectorAll("button").forEach(x => x.setAttribute("aria-pressed", x === b));
    try { localStorage.setItem("puentes-filtro", JSON.stringify({ para: filtro.para, area: filtro.area, estado: filtro.estado })); } catch (err) {}
    renderList();
  });
}

function card(c) {
  let due;
  if (c.cierre) {
    const n = diasHasta(c.cierre);
    due = '<span class="n">' + (n === 0 ? "Hoy" : n) + '</span><span class="u">' + (n === 0 ? "último día" : n === 1 ? "día" : "días") + '</span><span class="d">Cierra el ' + esc(largo(c.cierre)) + '</span>';
  } else {
    due = '<span class="t">Abierta</span><span class="d">' + esc(c.cierreTexto || "") + '</span>';
  }
  return '<article class="item ' + c.est + '" id="' + c.id + '">' +
    '<div class="stripe" aria-hidden="true"></div>' +
    '<div class="body">' +
      '<div class="meta"><span class="pill ' + c.est + '">' + ETIQUETA[c.est] + '</span><span class="org">' + esc(c.organismo) + '</span></div>' +
      '<h3>' + esc(c.titulo) + '</h3>' +
      '<p>' + esc(c.resumen) + '</p>' +
      '<div class="facts">' +
        '<span><b>Para:</b> ' + esc(c.para.map(p => PARA_TXT[p] || p).join(", ")) + '</span>' +
        '<span><b>Área:</b> ' + esc(c.areas.map(a => AREA_TXT[a] || a).join(", ")) + '</span>' +
        '<span><b>Tipo:</b> ' + esc(c.tipo) + '</span>' +
        '<span><b>Destino:</b> ' + esc(c.destino) + '</span>' +
      '</div>' +
      (c.nota ? '<div class="note">' + esc(c.nota) + '</div>' : "") +
      '<div class="links"><a href="' + esc(c.url) + '" target="_blank" rel="noopener">Ver bases oficiales ↗</a></div>' +
    '</div>' +
    '<div class="due">' + due + '</div>' +
  '</article>';
}

function renderList() {
  const q = sinTildes(filtro.q.trim());
  const vis = items.filter(c =>
    (filtro.para === "todos" || c.para.includes(filtro.para)) &&
    (filtro.area === "todas" || c.areas.includes(filtro.area) || c.areas.includes("todas")) &&
    (filtro.estado === "todas" || c.est === filtro.estado) &&
    (!q || sinTildes([c.titulo, c.organismo, c.destino, c.tipo, c.resumen, c.nota || ""].join(" ")).includes(q))
  );
  $("resultcount").textContent = vis.length === items.length
    ? "Mostrando las " + items.length + " convocatorias."
    : "Mostrando " + vis.length + " de " + items.length + " convocatorias.";
  $("list").innerHTML = vis.length
    ? vis.map(card).join("")
    : '<div class="empty">No hay convocatorias con esos filtros por ahora. Probá con “Todos” o sacá la búsqueda.</div>';
}

function renderPortales() {
  $("res").innerHTML = PORTALES.map(p =>
    '<a href="' + esc(p.url) + '" target="_blank" rel="noopener"><b>' + esc(p.nombre) + ' ↗</b><span>' + esc(p.texto) + '</span></a>').join("");
}

function renderBoletin() {
  $("sub-panel").innerHTML = CONFIG.suscripcionUrl
    ? '<h3>Recibilo cada jueves</h3><p>Gratis. Una vez por semana. Te podés dar de baja cuando quieras.</p><a class="btn primary" href="' + esc(CONFIG.suscripcionUrl) + '" target="_blank" rel="noopener">Suscribirme al boletín</a>'
    : '<h3>Recibilo cada jueves</h3><p>Gratis y una vez por semana. Las suscripciones abren muy pronto: mientras tanto, esta página se actualiza con cada edición.</p>';
  $("send-panel").innerHTML = CONFIG.contactoEmail
    ? '<h3>¿Te llegó una convocatoria que no está acá?</h3><p>Mandala y la sumamos en la próxima edición. Con el enlace a las bases alcanza.</p><p><a href="mailto:' + esc(CONFIG.contactoEmail) + '?subject=' + encodeURIComponent("Convocatoria para Puentes") + '">Escribinos</a> a <span class="mail">' + esc(CONFIG.contactoEmail) + '</span></p>'
    : '<h3>¿Te llegó una convocatoria que no está acá?</h3><p>Muy pronto vas a poder enviarla desde esta página para que la sumemos en la próxima edición.</p>';
}

function mostrarError() {
  const msg = "No pudimos cargar las convocatorias. Probá recargar la página en unos segundos.";
  $("next").innerHTML = '<span class="label">Próximo cierre</span><h2>' + msg + '</h2>';
  $("list").innerHTML = '<div class="empty">' + msg + '</div>';
  $("resultcount").textContent = "";
  console.error("Puentes: no se pudo leer convocatorias.json. Si estás probando en tu computadora, abrí la carpeta con: python -m http.server");
}

/* ---------------------------------------------------------------------
   Arranque: secciones fijas primero, luego los datos.
   --------------------------------------------------------------------- */
chips("f-para", PARA, "para");
chips("f-area", AREAS, "area");
chips("f-estado", ESTADOS, "estado");
$("q").addEventListener("input", e => { filtro.q = e.target.value; renderList(); });
renderPortales();
renderBoletin();

fetch("convocatorias.json", { cache: "no-cache" })
  .then(r => { if (!r.ok) throw new Error(r.status); return r.json(); })
  .then(datos => {
    actualizado = datos.actualizado;
    const ORDEN = { pronto: 0, abierta: 1 };
    items = datos.convocatorias
      .map((c, i) => ({ areas: ["todas"], ...c, id: "c" + i, est: estadoDe(c) }))
      .filter(c => c.est !== "cerrada")
      .sort((a, b) => ORDEN[a.est] - ORDEN[b.est] || (a.cierre ? diasHasta(a.cierre) : 9999) - (b.cierre ? diasHasta(b.cierre) : 9999));
    renderNext(); renderStats(); renderTimeline(); renderList();
    if (location.hash) { const el = document.querySelector(location.hash); if (el) el.scrollIntoView(); }
  })
  .catch(mostrarError);
