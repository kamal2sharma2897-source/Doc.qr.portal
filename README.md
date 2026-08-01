<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Document QR Register</title>
<script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Zilla+Slab:wght@500;600;700&family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500;600&display=swap');

  :root{
    --navy:#132A46; --navy-2:#1B3A5C; --paper:#F6F4EE; --paper-2:#FFFFFF;
    --slate:#5B6472; --saffron:#C9761C; --green:#1F6B3C; --line:#D8D2C4; --red-stamp:#9B2C2C;
  }
  *{box-sizing:border-box;}
  html,body{margin:0;padding:0;}
  body{ background:var(--paper); font-family:'Inter',sans-serif; color:var(--navy); min-height:100vh; }
  .ledger-bg{
    position:fixed; inset:0; pointer-events:none; z-index:0; opacity:0.035;
    background-image: repeating-linear-gradient(transparent, transparent 27px, var(--navy) 27px, var(--navy) 28px);
  }
  header{
    position:relative; z-index:1; border-bottom:3px double var(--navy); padding:22px 28px 16px;
    display:flex; align-items:baseline; justify-content:space-between; flex-wrap:wrap; gap:8px; background:var(--paper-2);
  }
  .brand-eyebrow{ font-family:'JetBrains Mono',monospace; font-size:11px; letter-spacing:2px; text-transform:uppercase; color:var(--saffron); font-weight:600; }
  header h1{ font-family:'Zilla Slab',serif; font-weight:700; font-size:28px; margin:2px 0 0; color:var(--navy); letter-spacing:0.3px; }
  .reg-meta{ font-family:'JetBrains Mono',monospace; font-size:12px; color:var(--slate); text-align:right; }
  .sync-status{font-size:10px; color:var(--green); margin-top:4px;}
  .sync-status.saving{color:var(--saffron);}

  main{ position:relative; z-index:1; max-width:1100px; margin:0 auto; padding:32px 24px 60px; display:grid; grid-template-columns:1fr; gap:28px; }
  @media(min-width:880px){ main{grid-template-columns: 1.1fr 0.9fr;} }

  .panel{ background:var(--paper-2); border:1px solid var(--line); position:relative; }
  .panel::before{
    content:""; position:absolute; top:0; left:0; right:0; height:4px;
    background:linear-gradient(90deg, var(--navy) 0%, var(--navy) 60%, var(--saffron) 60%, var(--saffron) 100%);
  }
  .panel-inner{padding:24px;}
  .panel h2{
    font-family:'Zilla Slab',serif; font-size:16px; text-transform:uppercase; letter-spacing:1.2px;
    margin:0 0 18px; color:var(--navy); border-bottom:1px solid var(--line); padding-bottom:10px;
    display:flex; justify-content:space-between; align-items:center;
  }

  label{ display:block; font-size:12px; font-weight:600; color:var(--slate); text-transform:uppercase; letter-spacing:0.6px; margin-bottom:6px; margin-top:16px; }
  label:first-of-type{margin-top:0;}
  input[type=text], input[type=url], input[type=search]{
    width:100%; padding:11px 12px; border:1px solid var(--line); background:var(--paper);
    font-family:'Inter',sans-serif; font-size:14px; color:var(--navy); border-radius:2px; outline:none; transition:border-color .15s ease;
  }
  input:focus{border-color:var(--navy-2); box-shadow:0 0 0 3px rgba(27,58,92,0.08);}

  .mode-toggle{ display:flex; gap:0; border:1px solid var(--navy); border-radius:2px; overflow:hidden; margin-bottom:18px; }
  .mode-toggle button{
    flex:1; padding:10px; border:none; background:var(--paper); color:var(--navy); cursor:pointer;
    font-family:'Inter',sans-serif; font-weight:600; font-size:13px;
  }
  .mode-toggle button.active{ background:var(--navy); color:#fff; }

  .doc-type-row{display:flex; gap:8px; flex-wrap:wrap; margin-top:6px;}
  .doc-type-row button{
    font-family:'JetBrains Mono',monospace; font-size:11px; padding:6px 10px;
    border:1px solid var(--line); background:var(--paper); color:var(--slate); cursor:pointer; border-radius:2px;
  }
  .doc-type-row button.active{ background:var(--navy); color:#fff; border-color:var(--navy); }

  .upload-zone{
    margin-top:6px; border:2px dashed var(--line); border-radius:3px; padding:26px 16px; text-align:center;
    cursor:pointer; transition:border-color .15s ease, background .15s ease; background:var(--paper);
  }
  .upload-zone:hover{ border-color:var(--navy-2); background:rgba(27,58,92,0.03); }
  .upload-zone.has-photo{ padding:14px; }
  .upload-zone .glyph{ font-size:26px; color:var(--slate); margin-bottom:8px; }
  .upload-zone p{ font-size:12.5px; color:var(--slate); margin:0; }
  .upload-preview{ max-width:100%; max-height:180px; border:1px solid var(--line); border-radius:2px; }

  .btn-primary{
    margin-top:22px; width:100%; padding:13px; background:var(--navy); color:#fff; border:none;
    font-family:'Zilla Slab',serif; font-weight:600; font-size:15px; letter-spacing:0.4px; cursor:pointer;
    border-radius:2px; transition:background .15s ease;
  }
  .btn-primary:hover{background:var(--navy-2);}
  .btn-primary:disabled{background:#A9AFB8; cursor:not-allowed;}
  .hint{font-size:12px; color:var(--slate); margin-top:8px; line-height:1.5;}
  .hint.warn{ color:var(--saffron); }

  .stamp-card{
    display:flex; flex-direction:column; align-items:center; text-align:center; padding:28px 20px 22px; border:1px solid var(--navy);
    background: repeating-linear-gradient(45deg, transparent, transparent 10px, rgba(19,42,70,0.015) 10px, rgba(19,42,70,0.015) 20px);
  }
  .stamp-card .reg-no{
    font-family:'JetBrains Mono',monospace; font-size:13px; color:var(--red-stamp); font-weight:600;
    border:1.5px solid var(--red-stamp); padding:3px 10px; border-radius:2px; transform:rotate(-1.5deg); margin-bottom:16px; letter-spacing:0.5px;
  }
  #qrcode{padding:10px; background:#fff; border:1px solid var(--line);}
  .stamp-card .doc-label{ font-family:'Zilla Slab',serif; font-size:16px; font-weight:600; margin-top:14px; color:var(--navy); }
  .stamp-card .doc-url{ font-family:'JetBrains Mono',monospace; font-size:11px; color:var(--slate); margin-top:4px; word-break:break-all; max-width:280px; }
  .empty-state{ padding:50px 20px; text-align:center; color:var(--slate); }
  .empty-state .glyph{font-size:34px; margin-bottom:10px; opacity:0.4;}
  .empty-state p{font-size:13px; line-height:1.6; max-width:260px; margin:0 auto;}

  .action-row{display:flex; gap:10px; margin-top:18px; width:100%;}
  .action-row button{
    flex:1; padding:9px; font-family:'Inter',sans-serif; font-weight:600; font-size:12.5px;
    border:1px solid var(--navy); background:#fff; color:var(--navy); cursor:pointer; border-radius:2px;
  }
  .action-row button.solid{background:var(--navy); color:#fff;}
  .action-row button:hover{opacity:0.85;}

  .register{grid-column:1/-1;}
  .search-row{ display:flex; gap:10px; margin-bottom:16px; }
  .search-row input{flex:1;}
  .search-row .result-count{ font-family:'JetBrains Mono',monospace; font-size:11px; color:var(--slate); align-self:center; white-space:nowrap; }

  table{width:100%; border-collapse:collapse; font-size:13px;}
  thead th{
    text-align:left; font-family:'JetBrains Mono',monospace; font-size:10.5px; text-transform:uppercase;
    letter-spacing:0.8px; color:var(--slate); padding:8px 10px; border-bottom:2px solid var(--navy);
  }
  tbody td{padding:10px; border-bottom:1px solid var(--line); vertical-align:middle;}
  tbody tr:hover{background:rgba(19,42,70,0.03);}
  .reg-tag{font-family:'JetBrains Mono',monospace; color:var(--red-stamp); font-weight:600; font-size:12px;}
  .mini-qr{width:46px; height:46px; border:1px solid var(--line); padding:3px; background:#fff; cursor:pointer; transition:transform .15s ease;}
  .mini-qr:hover{transform:scale(1.08); border-color:var(--navy);}
  .doc-name-link{ color:var(--navy); text-decoration:none; font-weight:600; font-size:13px; cursor:pointer; }
  .doc-name-link:hover{ text-decoration:underline; color:var(--navy-2); }
  .type-pill{
    font-family:'JetBrains Mono',monospace; font-size:10px; padding:2px 7px; border-radius:20px;
    background:rgba(31,107,60,0.1); color:var(--green); font-weight:600; text-transform:uppercase;
  }
  .type-pill.photo{ background:rgba(201,118,28,0.12); color:var(--saffron); }
  .row-actions{display:flex; gap:4px;}
  .row-actions button{ background:none; border:none; cursor:pointer; color:var(--slate); font-size:15px; padding:4px 6px; }
  .row-actions button:hover{color:var(--navy);}
  .row-actions button.delete-btn:hover{color:var(--red-stamp);}

  footer{ text-align:center; padding:20px; font-family:'JetBrains Mono',monospace; font-size:11px; color:var(--slate); border-top:1px solid var(--line); }

  .modal-overlay{
    position:fixed; inset:0; background:rgba(19,42,70,0.75); z-index:100; display:flex; align-items:center; justify-content:center; padding:20px;
    animation: fadeIn .18s ease;
  }
  @keyframes fadeIn{ from{opacity:0;} to{opacity:1;} }
  .modal-card{
    background:#fff; border:2px solid var(--navy); max-width:340px; width:100%; padding:30px 26px 26px; position:relative; text-align:center;
    animation: popIn .22s cubic-bezier(.2,.9,.3,1.2);
    box-shadow: 0 0 0 6px rgba(201,118,28,0.12), 0 20px 50px rgba(0,0,0,0.35);
  }
  @keyframes popIn{ from{ transform:scale(0.85); opacity:0; } to{ transform:scale(1); opacity:1; } }
  .modal-close{ position:absolute; top:10px; right:12px; background:none; border:none; font-size:18px; color:var(--slate); cursor:pointer; }
  .modal-eyebrow{ font-family:'JetBrains Mono',monospace; font-size:10px; letter-spacing:1.5px; text-transform:uppercase; color:var(--saffron); font-weight:600; margin-bottom:6px; }
  .modal-regno{
    font-family:'JetBrains Mono',monospace; font-size:13px; color:var(--red-stamp); font-weight:600;
    border:1.5px solid var(--red-stamp); padding:3px 10px; border-radius:2px; display:inline-block; margin-bottom:18px;
  }
  .modal-qr{
    padding:14px; background:#fff; border:2px solid var(--navy); display:inline-block;
    box-shadow:0 0 24px rgba(201,118,28,0.35); animation: pulseGlow 1.8s ease-in-out infinite;
  }
  @keyframes pulseGlow{ 0%,100%{ box-shadow:0 0 18px rgba(201,118,28,0.3); } 50%{ box-shadow:0 0 34px rgba(201,118,28,0.6); } }
  .modal-label{ font-family:'Zilla Slab',serif; font-size:17px; font-weight:600; margin-top:18px; color:var(--navy); }
  .modal-sub{ font-size:12px; color:var(--slate); margin-top:6px; }

  .toast{
    position:fixed; bottom:24px; left:50%; transform:translateX(-50%); background:var(--navy); color:#fff;
    padding:10px 20px; border-radius:3px; font-size:13px; z-index:200; opacity:0; pointer-events:none; transition:opacity .2s ease;
  }
  .toast.show{ opacity:1; }

  /* Viewer mode (opened via shared QR link) */
  #viewer-mode{
    display:none; min-height:100vh; align-items:center; justify-content:center; flex-direction:column; padding:24px;
  }
  #viewer-mode.active{ display:flex; }
  .viewer-card{
    background:#fff; border:2px solid var(--navy); max-width:480px; width:100%; padding:26px; text-align:center;
  }
  .viewer-card img{ max-width:100%; border:1px solid var(--line); margin-top:14px; }
  .viewer-card h2{ font-family:'Zilla Slab',serif; margin:12px 0 4px; }
  .viewer-card .reg-no{
    font-family:'JetBrains Mono',monospace; font-size:12px; color:var(--red-stamp); font-weight:600;
    border:1.5px solid var(--red-stamp); padding:3px 10px; border-radius:2px; display:inline-block;
  }
</style>
</head>
<body>
<div class="ledger-bg"></div>

<div id="viewer-mode">
  <div class="viewer-card" id="viewer-card">
    <div class="brand-eyebrow">Shared Document</div>
    <div id="viewer-content">Loading document...</div>
  </div>
</div>

<div id="app-root">
<header>
  <div>
    <div class="brand-eyebrow">Utility &middot; Link to QR</div>
    <h1>Document QR Register</h1>
  </div>
  <div class="reg-meta">
    <div id="today-meta"></div>
    <div class="sync-status" id="sync-status">Loading saved documents...</div>
  </div>
</header>

<main>
  <section class="panel">
    <div class="panel-inner">
      <h2>New Entry</h2>

      <div class="mode-toggle">
        <button id="mode-link-btn" class="active" type="button">Paste Link</button>
        <button id="mode-photo-btn" type="button">Upload Photo</button>
      </div>

      <div id="link-mode-fields">
        <label for="doc-link">Document Link</label>
        <input type="url" id="doc-link" placeholder="https://drive.google.com/... or any document URL">
      </div>

      <div id="photo-mode-fields" style="display:none;">
        <label>Photo</label>
        <input type="file" id="photo-input" accept="image/*" style="display:none;">
        <div class="upload-zone" id="upload-zone">
          <div id="upload-zone-empty">
            <div class="glyph">&#128247;</div>
            <p>Tap karke photo chuno (camera ya gallery se)</p>
          </div>
          <img id="upload-preview" class="upload-preview" style="display:none;">
        </div>
        <p class="hint warn" id="upload-status"></p>
      </div>

      <label for="doc-label">Document Name / Description</label>
      <input type="text" id="doc-label" placeholder="e.g. NOC Application - HPPSC SO 2026">

      <label>Document Type</label>
      <div class="doc-type-row" id="type-row">
        <button data-type="Order" type="button">Order</button>
        <button data-type="Report" type="button">Report</button>
        <button data-type="Certificate" type="button">Certificate</button>
        <button data-type="Form" type="button">Form</button>
        <button data-type="Photo" type="button">Photo</button>
        <button data-type="Other" type="button" class="active">Other</button>
      </div>

      <button class="btn-primary" id="generate-btn">Save Document & Generate QR</button>
      <p class="hint">Your document is saved here. You can search it later by name, open it with one click, or share it using its QR code.</p>
    </div>
  </section>

  <section class="panel">
    <div class="panel-inner" id="preview-panel">
      <h2>Latest QR</h2>
      <div class="empty-state" id="empty-state">
        <div class="glyph">▦</div>
        <p>Link daaliye ya photo upload kariye — QR code yahan dikhega, download aur print ke liye ready.</p>
      </div>
      <div class="stamp-card" id="stamp-card" style="display:none;">
        <div class="reg-no" id="reg-no-display"></div>
        <div id="qrcode"></div>
        <div class="doc-label" id="label-display"></div>
        <div class="doc-url" id="url-display"></div>
        <div class="action-row">
          <button id="download-btn">Download PNG</button>
          <button class="solid" id="print-btn">Print</button>
        </div>
      </div>
    </div>
  </section>

  <section class="panel register">
    <div class="panel-inner">
      <h2>Saved Documents (<span id="entry-count">0</span>)</h2>
      <div class="search-row">
        <input type="search" id="search-input" placeholder="Document ka naam search karo...">
        <div class="result-count" id="result-count"></div>
      </div>
      <table>
        <thead>
          <tr><th>Reg. No.</th><th>QR</th><th>Document</th><th>Type</th><th></th></tr>
        </thead>
        <tbody id="log-body">
          <tr id="no-entries-row"><td colspan="5" style="text-align:center; color:var(--slate); padding:24px;">Loading...</td></tr>
        </tbody>
      </table>
    </div>
  </section>
</main>

<footer>DOCUMENT QR REGISTER &middot; Documents saved permanently to your account</footer>
</div>

<div id="qr-modal" class="modal-overlay" style="display:none;">
  <div class="modal-card">
    <button class="modal-close" id="modal-close">&#10005;</button>
    <div class="modal-eyebrow">Scan to Open</div>
    <div class="modal-regno" id="modal-regno"></div>
    <div class="modal-qr" id="modal-qrcode"></div>
    <div class="modal-label" id="modal-label"></div>
    <div class="modal-sub">Doosre phone se scan karke ye document seedha khul jayega</div>
    <div class="action-row">
      <button id="modal-share">Share Link</button>
      <button class="solid" id="modal-copy">Copy Link</button>
    </div>
  </div>
</div>

<div class="toast" id="toast"></div>

<script>
let entries = [];
let counter = 0;
let selectedType = "Other";
let currentMode = "link";
let pendingPhotoDataUrl = null;
const yearNow = new Date().getFullYear();
const STORAGE_KEY = 'doc-register';

// --- Viewer mode: if opened via a shared photo link (?doc=ID), show the photo only ---
const urlParams = new URLSearchParams(window.location.search);
const viewId = urlParams.get('doc');

async function initViewer(id){
  document.getElementById('app-root').style.display = 'none';
  const viewerMode = document.getElementById('viewer-mode');
  viewerMode.classList.add('active');
  const content = document.getElementById('viewer-content');
  try{
    const result = await window.storage.get('photo:' + id, true);
    if(result && result.value){
      const data = JSON.parse(result.value);
      content.innerHTML = `
        <div class="reg-no">${data.regNo || ''}</div>
        <h2>${data.label || 'Shared Document'}</h2>
        <img src="${data.dataUrl}" alt="${data.label || 'document'}">
      `;
    }else{
      content.innerHTML = `<p style="color:var(--slate);">Ye document nahi mila. Link expire ho gaya ya delete kar diya gaya hai.</p>`;
    }
  }catch(err){
    content.innerHTML = `<p style="color:var(--slate);">Ye document nahi mila. Link expire ho gaya ya delete kar diya gaya hai.</p>`;
  }
}

if(viewId){
  initViewer(viewId);
} else {
  document.getElementById('today-meta').textContent =
    new Date().toLocaleDateString('en-IN', { day:'2-digit', month:'short', year:'numeric' });

  document.querySelectorAll('#type-row button').forEach(btn=>{
    btn.addEventListener('click', ()=>{
      document.querySelectorAll('#type-row button').forEach(b=>b.classList.remove('active'));
      btn.classList.add('active');
      selectedType = btn.dataset.type;
    });
  });

  document.getElementById('mode-link-btn').addEventListener('click', ()=> switchMode('link'));
  document.getElementById('mode-photo-btn').addEventListener('click', ()=> switchMode('photo'));

  function switchMode(mode){
    currentMode = mode;
    document.getElementById('mode-link-btn').classList.toggle('active', mode==='link');
    document.getElementById('mode-photo-btn').classList.toggle('active', mode==='photo');
    document.getElementById('link-mode-fields').style.display = mode==='link' ? 'block' : 'none';
    document.getElementById('photo-mode-fields').style.display = mode==='photo' ? 'block' : 'none';
  }

  const uploadZone = document.getElementById('upload-zone');
  const photoInput = document.getElementById('photo-input');
  uploadZone.addEventListener('click', ()=> photoInput.click());

  photoInput.addEventListener('change', async ()=>{
    const file = photoInput.files[0];
    if(!file) return;
    const statusEl = document.getElementById('upload-status');
    statusEl.textContent = 'Photo process ho rahi hai...';
    try{
      const dataUrl = await compressImage(file, 1000, 0.72);
      pendingPhotoDataUrl = dataUrl;
      document.getElementById('upload-preview').src = dataUrl;
      document.getElementById('upload-preview').style.display = 'block';
      document.getElementById('upload-zone-empty').style.display = 'none';
      uploadZone.classList.add('has-photo');
      const sizeKb = Math.round((dataUrl.length * 0.75) / 1024);
      statusEl.textContent = `Ready (~${sizeKb} KB)`;
    }catch(err){
      statusEl.textContent = 'Photo load nahi ho payi, dobara try karo.';
    }
  });

  function compressImage(file, maxDim, quality){
    return new Promise((resolve, reject)=>{
      const reader = new FileReader();
      reader.onload = (e)=>{
        const img = new Image();
        img.onload = ()=>{
          let w = img.width, h = img.height;
          if(w > maxDim || h > maxDim){
            if(w > h){ h = Math.round(h * maxDim / w); w = maxDim; }
            else{ w = Math.round(w * maxDim / h); h = maxDim; }
          }
          const canvas = document.createElement('canvas');
          canvas.width = w; canvas.height = h;
          const ctx = canvas.getContext('2d');
          ctx.drawImage(img, 0, 0, w, h);
          resolve(canvas.toDataURL('image/jpeg', quality));
        };
        img.onerror = reject;
        img.src = e.target.result;
      };
      reader.onerror = reject;
      reader.readAsDataURL(file);
    });
  }

  function padNum(n){ return String(n).padStart(3,'0'); }
  function makeRegNo(){ counter++; return `QR/${yearNow}/${padNum(counter)}`; }

  function renderQR(containerEl, text, size){
    containerEl.innerHTML = "";
    new QRCode(containerEl, {
      text: text, width: size, height: size,
      colorDark: "#132A46", colorLight: "#ffffff", correctLevel: QRCode.CorrectLevel.M
    });
  }

  function setSyncStatus(msg, saving){
    const el = document.getElementById('sync-status');
    el.textContent = msg;
    el.classList.toggle('saving', !!saving);
  }

  function showToast(msg){
    const t = document.getElementById('toast');
    t.textContent = msg;
    t.classList.add('show');
    setTimeout(()=> t.classList.remove('show'), 1800);
  }

  async function loadEntries(){
    try{
      const result = await window.storage.get(STORAGE_KEY, false);
      if(result && result.value){
        const data = JSON.parse(result.value);
        entries = data.entries || [];
        counter = data.counter || 0;
      }
      setSyncStatus('All documents saved');
    }catch(err){
      entries = []; counter = 0;
      setSyncStatus('All documents saved');
    }
    renderLog();
  }

  async function saveEntries(){
    setSyncStatus('Saving...', true);
    try{
      const result = await window.storage.set(STORAGE_KEY, JSON.stringify({ entries, counter }), false);
      if(!result){ setSyncStatus('Save failed - try again'); return; }
      setSyncStatus('All documents saved');
    }catch(err){
      setSyncStatus('Save failed - try again');
    }
  }

  document.getElementById('generate-btn').addEventListener('click', async ()=>{
    const labelInput = document.getElementById('doc-label');
    let label = labelInput.value.trim();
    const genBtn = document.getElementById('generate-btn');

    if(currentMode === 'link'){
      const linkInput = document.getElementById('doc-link');
      const link = linkInput.value.trim();
      if(!link){
        linkInput.style.borderColor = '#9B2C2C';
        linkInput.focus();
        return;
      }
      linkInput.style.borderColor = '';
      if(!label) label = "Untitled Document";

      genBtn.disabled = true; genBtn.textContent = 'Saving...';
      const regNo = makeRegNo();
      const entry = { regNo, link, label, type: selectedType, date: new Date().toLocaleDateString('en-IN') };
      entries.unshift(entry);
      showResult(entry);
      renderLog();
      await saveEntries();
      linkInput.value = "";
      labelInput.value = "";
      genBtn.disabled = false; genBtn.textContent = 'Save Document & Generate QR';
      showToast('Document saved: ' + regNo);

    } else {
      if(!pendingPhotoDataUrl){
        document.getElementById('upload-status').textContent = 'Pehle koi photo chuno.';
        return;
      }
      if(!label) label = "Untitled Photo";

      genBtn.disabled = true; genBtn.textContent = 'Uploading...';
      const regNo = makeRegNo();
      const photoId = 'p' + Date.now() + Math.random().toString(36).slice(2,8);
      try{
        await window.storage.set('photo:' + photoId, JSON.stringify({
          dataUrl: pendingPhotoDataUrl, label, regNo
        }), true);
      }catch(err){
        showToast('Photo upload fail ho gayi, dobara try karo.');
        genBtn.disabled = false; genBtn.textContent = 'Save Document & Generate QR';
        counter--;
        return;
      }

      const shareLink = window.location.origin + window.location.pathname + '?doc=' + photoId;
      const entry = { regNo, link: shareLink, label, type: 'Photo', date: new Date().toLocaleDateString('en-IN') };
      entries.unshift(entry);
      showResult(entry);
      renderLog();
      await saveEntries();

      pendingPhotoDataUrl = null;
      photoInput.value = '';
      document.getElementById('upload-preview').style.display = 'none';
      document.getElementById('upload-zone-empty').style.display = 'block';
      uploadZone.classList.remove('has-photo');
      document.getElementById('upload-status').textContent = '';
      labelInput.value = "";
      genBtn.disabled = false; genBtn.textContent = 'Save Document & Generate QR';
      showToast('Photo saved: ' + regNo);
    }
  });

  function showResult(entry){
    document.getElementById('empty-state').style.display = 'none';
    const card = document.getElementById('stamp-card');
    card.style.display = 'flex';
    document.getElementById('reg-no-display').textContent = entry.regNo;
    document.getElementById('label-display').textContent = entry.label;
    document.getElementById('url-display').textContent = entry.link;
    renderQR(document.getElementById('qrcode'), entry.link, 180);
  }

  function getFilteredEntries(){
    const q = document.getElementById('search-input').value.trim().toLowerCase();
    if(!q) return entries;
    return entries.filter(e => e.label.toLowerCase().includes(q) || e.regNo.toLowerCase().includes(q) || e.type.toLowerCase().includes(q));
  }

  function renderLog(){
    const tbody = document.getElementById('log-body');
    const filtered = getFilteredEntries();
    document.getElementById('entry-count').textContent = entries.length;
    const resultCountEl = document.getElementById('result-count');
    const q = document.getElementById('search-input').value.trim();
    resultCountEl.textContent = q ? `${filtered.length} result${filtered.length===1?'':'s'}` : '';

    if(entries.length === 0){
      tbody.innerHTML = `<tr><td colspan="5" style="text-align:center; color:var(--slate); padding:24px;">Abhi koi document save nahi hai. Pehla document link ya photo daalke shuru kariye.</td></tr>`;
      return;
    }
    if(filtered.length === 0){
      tbody.innerHTML = `<tr><td colspan="5" style="text-align:center; color:var(--slate); padding:24px;">Is naam ka koi document nahi mila.</td></tr>`;
      return;
    }

    tbody.innerHTML = "";
    filtered.forEach((e)=>{
      const realIdx = entries.indexOf(e);
      const tr = document.createElement('tr');
      const pillClass = e.type === 'Photo' ? 'type-pill photo' : 'type-pill';
      tr.innerHTML = `
        <td class="reg-tag">${e.regNo}</td>
        <td><div class="mini-qr" data-idx="${realIdx}" id="mini-${realIdx}" title="Click to flash QR"></div></td>
        <td>
          <a class="doc-name-link" data-idx="${realIdx}">${e.label}</a><br>
          <span style="font-size:11px; color:var(--slate);">${e.date}</span>
        </td>
        <td><span class="${pillClass}">${e.type}</span></td>
        <td class="row-actions">
          <button class="open-btn" data-idx="${realIdx}" title="Open document">&#8599;</button>
          <button class="delete-btn" data-idx="${realIdx}" title="Delete">&#10005;</button>
        </td>
      `;
      tbody.appendChild(tr);
      renderQR(document.getElementById(`mini-${realIdx}`), e.link, 40);
    });

    tbody.querySelectorAll('.mini-qr').forEach(el=>{
      el.addEventListener('click', ()=> openQRModal(parseInt(el.dataset.idx,10)));
    });
    tbody.querySelectorAll('.doc-name-link').forEach(el=>{
      el.addEventListener('click', ()=> window.open(entries[parseInt(el.dataset.idx,10)].link, '_blank', 'noopener'));
    });
    tbody.querySelectorAll('.open-btn').forEach(btn=>{
      btn.addEventListener('click', ()=> window.open(entries[parseInt(btn.dataset.idx,10)].link, '_blank', 'noopener'));
    });
    tbody.querySelectorAll('.delete-btn').forEach(btn=>{
      btn.addEventListener('click', async ()=>{
        const idx = parseInt(btn.dataset.idx,10);
        entries.splice(idx,1);
        renderLog();
        await saveEntries();
        showToast('Document deleted');
      });
    });
  }

  document.getElementById('search-input').addEventListener('input', renderLog);

  let activeModalEntry = null;
  function openQRModal(idx){
    const e = entries[idx];
    if(!e) return;
    activeModalEntry = e;
    document.getElementById('modal-regno').textContent = e.regNo;
    document.getElementById('modal-label').textContent = e.label;
    renderQR(document.getElementById('modal-qrcode'), e.link, 220);
    document.getElementById('qr-modal').style.display = 'flex';
  }
  function closeQRModal(){
    document.getElementById('qr-modal').style.display = 'none';
    activeModalEntry = null;
  }
  document.getElementById('modal-close').addEventListener('click', closeQRModal);
  document.getElementById('qr-modal').addEventListener('click', (ev)=>{ if(ev.target.id === 'qr-modal') closeQRModal(); });

  document.getElementById('modal-share').addEventListener('click', async ()=>{
    if(!activeModalEntry) return;
    if(navigator.share){
      try{ await navigator.share({ title: activeModalEntry.label, url: activeModalEntry.link }); }catch(err){}
    }else{
      try{ await navigator.clipboard.writeText(activeModalEntry.link); showToast('Link copied (share not supported on this device)'); }
      catch(err){ showToast('Could not copy link'); }
    }
  });
  document.getElementById('modal-copy').addEventListener('click', async ()=>{
    if(!activeModalEntry) return;
    try{ await navigator.clipboard.writeText(activeModalEntry.link); showToast('Link copied to clipboard'); }
    catch(err){ showToast('Could not copy link'); }
  });

  document.getElementById('download-btn').addEventListener('click', ()=>{
    const img = document.querySelector('#qrcode img') || document.querySelector('#qrcode canvas');
    if(!img) return;
    const link = document.createElement('a');
    const regNo = document.getElementById('reg-no-display').textContent.replace(/\//g,'-');
    link.download = `${regNo}.png`;
    link.href = img.src || img.toDataURL('image/png');
    link.click();
  });

  document.getElementById('print-btn').addEventListener('click', ()=>{ window.print(); });

  loadEntries();
}
</script>

</body>
</html>
