# ETIQUETADO-
Etiquetado 
<!doctype html>

<html lang="es">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Etiquetas Cocina — App</title>
  <link rel="manifest" href="manifest.json" />
  <meta name="theme-color" content="#2b6ef6" />
  <style>
    :root{font-family:system-ui,-apple-system,Segoe UI,Roboto,'Helvetica Neue',Arial;}
    body{margin:0;padding:16px;background:#f6f7fb;color:#222}
    .container{max-width:900px;margin:0 auto}
    h1{font-size:20px;margin:0 0 10px}
    form{display:flex;gap:8px;flex-wrap:wrap;margin-bottom:12px}
    input,select,button{padding:8px;border:1px solid #d6d9e6;border-radius:6px;font-size:14px}
    input[type=date]{padding:6px}
    .flex{display:flex;gap:8px;align-items:center}
    .list{margin-top:8px}
    .item{background:#fff;padding:10px;border-radius:8px;border:1px solid #e2e6f0;display:flex;justify-content:space-between;align-items:center;margin-bottom:8px}
    .item .meta{display:flex;gap:12px;align-items:center}
    .emoji{font-size:22px}
    .small{font-size:13px;color:#555}
    .actions button{margin-left:6px}
    .btn{background:#2b6ef6;color:#fff;border:0;padding:8px 12px;border-radius:6px;cursor:pointer}
    .btn.ghost{background:transparent;color:#2b6ef6;border:1px solid #cbd7ff}
    #labelsSheet{display:grid;grid-template-columns:repeat(2,1fr);gap:12px;padding:12px}
    .label{background:#fff;border:1px dashed #c8d0f8;padding:10px;border-radius:6px;min-height:70px;display:flex;gap:10px;align-items:center}
    .label .left{font-size:28px}
    .label .right{flex:1}
    .label h3{margin:0;font-size:16px}
    .label p{margin:4px 0;font-size:13px}
    @media print{body{background:#fff}}
  </style>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
</head>
<body>
  <div class="container">
    <h1>Etiquetas de productos — Cocina</h1>
    <p class="small">Añade productos con nombre, fecha de elaboración y caducidad. Genera un PDF con etiquetas individuales (2 por página).</p>
    <form id="productForm" onsubmit="return false">
      <input id="name" placeholder="Nombre del producto" required />
      <select id="emoji">
        <option value="🍞">Pan / Harinas 🍞</option>
        <option value="🥩">Carnes 🥩</option>
        <option value="🥛">Lácteos 🥛</option>
        <option value="🍅">Verduras / Salsas 🍅</option>
        <option value="🍰">Postres / Repostería 🍰</option>
        <option value="🌶️">Conservas / Especias 🌶️</option>
        <option value="🥫">Envasados 🥫</option>
        <option value="🧂">Otros 🧂</option>
      </select>
      <input id="made" type="date" required />
      <input id="expiry" type="date" required />
      <div class="flex">
        <button id="addBtn" class="btn">Añadir</button>
        <button id="pdfBtn" class="btn" type="button">Generar PDF</button>
        <button id="clearBtn" class="btn ghost" type="button">Borrar todo</button>
      </div>
    </form>
    <div class="list" id="list"></div>
    <hr />
    <h2 style="font-size:16px;margin-top:8px">Vista previa / Etiquetas</h2>
    <div id="labelsSheet"></div>
  </div>
<script>
const storageKey = 'etiquetas_cocina_v1'
let products = JSON.parse(localStorage.getItem(storageKey) || '[]')
function save(){localStorage.setItem(storageKey, JSON.stringify(products))}
function formatDate(d){if(!d)return '';const dt=new Date(d);return dt.toLocaleDateString('es-ES')}
function escapeHtml(s){return String(s).replace(/[&<>"']/g,m=>({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;','\'':'&#39;'}[m]))}
const nameIn=document.getElementById('name'),emojiIn=document.getElementById('emoji'),madeIn=document.getElementById('made'),expiryIn=document.getElementById('expiry'),addBtn=document.getElementById('addBtn'),pdfBtn=document.getElementById('pdfBtn'),clearBtn=document.getElementById('clearBtn'),listEl=document.getElementById('list'),labelsEl=document.getElementById('labelsSheet')
function renderList(){listEl.innerHTML='';products.forEach((p,i)=>{const item=document.createElement('div');item.className='item';item.innerHTML=`<div class="meta"><div class="emoji">${p.emoji}</div><div><div><strong>${escapeHtml(p.name)}</strong></div><div class="small">Elaboración: ${formatDate(p.made)} · Caducidad: ${formatDate(p.expiry)}</div></div></div><div class="actions"><button class="btn ghost" onclick="edit(${i})">Editar</button><button class="btn" onclick="del(${i})">Eliminar</button></div>`;listEl.appendChild(item)});renderLabels()}
function renderLabels(){labelsEl.innerHTML='';products.forEach(p=>{const lab=document.createElement('div');lab.className='label';lab.innerHTML=`<div class="left">${p.emoji}</div><div class="right"><h3>${escapeHtml(p.name)}</h3><p>Elaboración: ${formatDate(p.made)}</p><p>Caduca: ${formatDate(p.expiry)}</p></div>`;labelsEl.appendChild(lab)})}
addBtn.addEventListener('click',()=>{const n=nameIn.value.trim(),e=emojiIn.value,m=madeIn.value,x=expiryIn.value;if(!n||!m||!x){alert('Rellena nombre, fechas.');return}products.push({name:n,emoji:e,made:m,expiry:x});save();renderList();nameIn.value='';madeIn.value='';expiryIn.value=''})
window.edit=i=>{const p=products[i];const n=prompt('Editar nombre',p.name);if(n===null)return;const m=prompt('Fecha elaboración (YYYY-MM-DD)',p.made);if(m===null)return;const x=prompt('Fecha caducidad (YYYY-MM-DD)',p.expiry);if(x===null)return;const e=prompt('Emoji',p.emoji);if(e===null)return;products[i]={name:n,emoji:e,made:m,expiry:x};save();renderList()}
window.del=i=>{if(confirm('Eliminar este producto?')){products.splice(i,1);save();renderList()}}
clearBtn.addEventListener('click',()=>{if(confirm('Borrar todos los productos?')){products=[];save();renderList()}})
pdfBtn.addEventListener('click',async()=>{if(!products.length){alert('No hay productos.');return}const el=labelsEl.cloneNode(true);el.style.padding='18px';const tmp=document.createElement('div');tmp.style.position='fixed';tmp.style.left='-10000px';tmp.appendChild(el);document.body.appendChild(tmp);try{const canvas=await html2canvas(el,{scale:2,backgroundColor:'#fff'});const imgData=canvas.toDataURL('image/png');const {jsPDF}=window.jspdf;const pdf=new jsPDF({unit:'mm',format:'a4'});const pageW=pdf.internal.pageSize.getWidth();const pdfW=pageW-20;const pdfH=(canvas.height*pdfW)/canvas.width;pdf.addImage(imgData,'PNG',10,10,pdfW,pdfH);pdf.save('etiquetas_cocina.pdf')}catch(e){alert('Error al generar PDF.');}finally{document.body.removeChild(tmp)}})
renderList()
// Service Worker
if('serviceWorker' in navigator){navigator.serviceWorker.register('sw.js').then(()=>console.log('SW registrado')).catch(err=>console.error('SW error',err))}
</script>
</body>
</html>
