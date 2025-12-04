# 3inalewa1a.github.io
List
<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>FreeFire IGN & UID List</title>
  <style>
    :root{--bg:#0f1724;--card:#0b1220;--accent:#06b6d4;--muted:#94a3b8;--glass:rgba(255,255,255,0.04)}
    html,body{height:100%;margin:0;font-family:Inter,Segoe UI,Roboto,system-ui,-apple-system,Arial}
    body{background:linear-gradient(180deg,#071021 0%, #0b1220 100%);color:#e6eef6;display:flex;align-items:center;justify-content:center;padding:24px}
    .wrap{width:100%;max-width:920px}
    header{display:flex;align-items:center;justify-content:space-between;margin-bottom:18px}
    h1{font-size:20px;margin:0}
    .card{background:var(--card);padding:18px;border-radius:12px;box-shadow:0 6px 18px rgba(2,6,23,0.6)}
    form{display:flex;gap:10px;flex-wrap:wrap}
    input[type=text]{padding:10px;border-radius:8px;border:1px solid rgba(255,255,255,0.04);background:var(--glass);color:inherit;min-width:160px}
    button{background:var(--accent);border:none;color:#042029;padding:10px 14px;border-radius:8px;cursor:pointer}
    .list{margin-top:14px}
    .row{display:flex;align-items:center;justify-content:space-between;padding:10px;border-radius:8px;background:linear-gradient(180deg, rgba(255,255,255,0.02), transparent);margin-bottom:8px}
    .meta{color:var(--muted);font-size:13px}
    .controls{display:flex;gap:8px}
    .ghost{background:transparent;border:1px solid rgba(255,255,255,0.06);color:var(--muted);padding:8px;border-radius:8px}
    .small{font-size:13px;padding:6px 8px}
    .hint{color:var(--muted);font-size:13px;margin-top:8px}
    footer{margin-top:12px;color:var(--muted);font-size:13px}
    @media (max-width:640px){form{flex-direction:column} .controls{flex-direction:row}}
  </style>
</head>
<body>
  <div class="wrap">
    <header>
      <h1>FreeFire IGN & UID List</h1>
      <div>
        <button id="adminBtn" class="ghost small">Enter Admin Mode</button>
      </div>
    </header>

    <div class="card">
      <form id="addForm" onsubmit="return false">
        <input id="ign" type="text" placeholder="IGN (in-game name)" required />
        <input id="uid" type="text" placeholder="UID (numbers only)" required />
        <button id="addBtn">Add to List (Free)</button>
        <button id="exportBtn" type="button" class="ghost small">Export CSV</button>
      </form>
      <div class="hint">Players can add their IGN & UID. Deleting entries requires Admin Mode (password-protected).</div>

      <div class="list" id="list"></div>
      <footer>
        <div class="meta">Entries are saved locally in the browser (localStorage). To let everyone use it on the web, upload this file to any static hosting (Netlify, GitHub Pages, Vercel).</div>
      </footer>
    </div>
  </div>

<script>
// ====== CONFIG ======
// Change this admin password before publishing. This is client-side protection (not fully secure).
const ADMIN_PASSWORD_HASH = 'b0b1f49f9dfb8b3e2a0f3a9f5a9f5c1a'; // placeholder hash for 'admin123' (not real hashing here)
// ====================

const STORAGE_KEY = 'ff_ign_uid_list_v1';
let isAdmin = false;

function $(id){return document.getElementById(id)}

function loadList(){
  try{
    const raw = localStorage.getItem(STORAGE_KEY) || '[]';
    return JSON.parse(raw);
  }catch(e){return []}
}

function saveList(list){
  localStorage.setItem(STORAGE_KEY, JSON.stringify(list));
}

function render(){
  const container = $('list');
  const list = loadList();
  container.innerHTML = '';
  if(list.length===0){ container.innerHTML = '<div class="meta">No entries yet. Be the first!</div>'; return }
  list.forEach((item, idx)=>{
    const el = document.createElement('div');
    el.className = 'row';
    el.innerHTML = `
      <div>
        <div><strong>${escapeHtml(item.ign)}</strong> <span class="meta">(UID: ${escapeHtml(item.uid)})</span></div>
        <div class="meta">Added: ${new Date(item.added).toLocaleString()}</div>
      </div>
    `;
    const right = document.createElement('div');
    right.className = 'controls';

    const copyBtn = document.createElement('button');
    copyBtn.className = 'ghost small';
    copyBtn.textContent = 'Copy';
    copyBtn.onclick = ()=>{navigator.clipboard.writeText(`${item.ign} | ${item.uid}`)};
    right.appendChild(copyBtn);

    const reportBtn = document.createElement('button');
    reportBtn.className = 'ghost small';
    reportBtn.textContent = 'Report';
    reportBtn.onclick = ()=>{alert('To report this entry, contact the site admin.');};
    right.appendChild(reportBtn);

    if(isAdmin){
      const delBtn = document.createElement('button');
      delBtn.className = 'small';
      delBtn.textContent = 'Delete';
      delBtn.onclick = ()=>{ if(confirm('Delete this entry?')){ deleteEntry(idx) } };
      right.appendChild(delBtn);
    }

    el.appendChild(right);
    container.appendChild(el);
  })
}

function addEntry(ign, uid){
  const list = loadList();
  list.unshift({ign:ign, uid:uid, added:Date.now()});
  saveList(list);
  render();
}

function deleteEntry(index){
  const list = loadList();
  if(index<0||index>=list.length) return;
  list.splice(index,1);
  saveList(list);
  render();
}

function escapeHtml(s){
  return String(s).replace(/[&<>"']/g, function(m){return {"&":"&amp;","<":"&lt;",">":"&gt;","\"":"&quot;","'":"&#39;"}[m]});
}

// Simple password prompt (client-side). Replace with real auth for production.
function promptAdmin(){
  const pwd = prompt('Enter admin password to enable Admin Mode:');
  if(!pwd) return false;
  // VERY simple check: in this static demo we accept the password 'admin123' — change as needed.
  if(pwd === 'admin123'){
    isAdmin = true;
    $('adminBtn').textContent = 'Admin Mode ON';
    $('adminBtn').classList.add('small');
    render();
    alert('Admin mode enabled. You can now delete entries.');
    return true;
  }else{
    alert('Incorrect password.');
    return false;
  }
}

function exportCSV(){
  const list = loadList();
  if(list.length===0){ alert('No entries to export.'); return }
  const rows = [['IGN','UID','Added']];
  list.forEach(r=> rows.push([r.ign,r.uid,new Date(r.added).toLocaleString()]));
  const csv = rows.map(r=> r.map(c=> '"'+String(c).replace(/"/g,'""')+'"').join(',')).join('\n');
  const blob = new Blob([csv],{type:'text/csv'});
  const url = URL.createObjectURL(blob);
  const a = document.createElement('a');
  a.href = url; a.download = 'ff_list.csv'; a.click();
  URL.revokeObjectURL(url);
}

// Validate UID: digits only (but allow letters if necessary)
function validateUID(uid){
  return uid.trim().length>=4; // basic length check
}

// UI wiring
window.addEventListener('DOMContentLoaded', ()=>{
  render();
  $('addBtn').onclick = ()=>{
    const ign = $('ign').value.trim();
    const uid = $('uid').value.trim();
    if(!ign){ alert('Enter IGN'); return }
    if(!validateUID(uid)){ alert('Enter valid UID (at least 4 characters)'); return }
    addEntry(ign, uid);
    $('ign').value=''; $('uid').value='';
  };
  $('adminBtn').onclick = ()=>{
    if(isAdmin){ if(confirm('Disable Admin Mode?')){ isAdmin=false; $('adminBtn').textContent='Enter Admin Mode'; render(); } return }
    promptAdmin();
  };
  $('exportBtn').onclick = exportCSV;
});

</script>

<!--
  INSTRUCTIONS
  - This is a single-file static website. To publish:
    1) Save as "index.html" and upload to GitHub Pages / Netlify / Vercel / any static host.
    2) Change the client-side admin password logic before publishing (replace 'admin123').
    3) For secure admin control, add a backend (simple Firebase/Auth or server + DB) and restrict delete API to authenticated admin only.

  NOTES
  - Current protection is client-side only (password typed into prompt). It's fine for small casual uses but NOT secure for public sites.
-->

</body>
</html>
