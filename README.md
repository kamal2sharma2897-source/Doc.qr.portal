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
    --navy:#132A46;
    --navy-2:#1B3A5C;
    --paper:#F6F4EE;
    --paper-2:#FFFFFF;
    --slate:#5B6472;
    --saffron:#C9761C;
    --green:#1F6B3C;
    --line:#D8D2C4;
    --red-stamp:#9B2C2C;
  }
  *{box-sizing:border-box;}
  html,body{margin:0;padding:0;}
  body{
    background:var(--paper);
    background-image:
      linear-gradient(var(--paper) 0%, var(--paper) 100%);
    font-family:'Inter',sans-serif;
    color:var(--navy);
    min-height:100vh;
  }

  /* subtle paper texture via repeating lines */
  .ledger-bg{
    position:fixed; inset:0; pointer-events:none; z-index:0; opacity:0.035;
    background-image: repeating-linear-gradient(transparent, transparent 27px, var(--navy) 27px, var(--navy) 28px);
  }

  header{
    position:relative; z-index:1;
    border-bottom:3px double var(--navy);
    padding:22px 28px 16px;
    display:flex; align-items:baseline; justify-content:space-between; flex-wrap:wrap; gap:8px;
    background:var(--paper-2);
  }
  .brand-eyebrow{
    font-family:'JetBrains Mono',monospace;
    font-size:11px; letter-spacing:2px; text-transform:uppercase; color:var(--saffron); font-weight:600;
  }
  header h1{
    font-family:'Zilla Slab',serif; font-weight:700; font-size:28px; margin:2px 0 0; color:var(--navy);
    letter-spacing:0.3px;
  }
  .reg-meta{
    font-family:'JetBrains Mono',monospace; font-size:12px; color:var(--slate); text-align:right;
  }

  main{
    position:relative; z-index:1;
    max-width:1100px; margin:0 auto; padding:32px 24px 60px;
    display:grid; grid-template-columns:1fr; gap:28px;
  }
  @media(min-width:880px){
    main{grid-template-columns: 1.1fr 0.9fr;}
  }

  .panel{
    background:var(--paper-2);
    border:1px solid var(--line);
    position:relative;
  }
  .panel::before{
    content:""; position:absolute; top:0; left:0; right:0; height:4px;
    background:linear-gradient(90deg, var(--navy) 0%, var(--navy) 60%, var(--saffron) 60%, var(--saffron) 100%);
  }
  .panel-inner{padding:24px;}
  .panel h2{
    font-family:'Zilla Slab',serif; font-size:16px; text-transform:uppercase; letter-spacing:1.2px;
    margin:0 0 18px; color:var(--navy); border-bottom:1px solid var(--line); padding-bottom:10px;
  }

  label{
    display:block; font-size:12px; font-weight:600; color:var(--slate); text-transform:uppercase;
    letter-spacing:0.6px; margin-bottom:6px; margin-top:16px;
  }
  label:first-of-type{margin-top:0;}
  input[type=text], input[type=url]{
    width:100%; padding:11px 12px; border:1px solid var(--line); background:var(--paper);
    font-family:'Inter',sans-serif; font-size:14px; color:var(--navy); border-radius:2px;
    outline:none; transition:border-color .15s ease;
  }
  input:focus{border-color:var(--navy-2); box-shadow:0 0 0 3px rgba(27,58,92,0.08);}

  .doc-type-row{display:flex; gap:8px; flex-wrap:wrap; margin-top:6px;}
  .doc-type-row button{
    font-family:'JetBrains Mono',monospace; font-size:11px; padding:6px 10px;
    border:1px solid var(--line); background:var(--paper); color:var(--slate); cursor:pointer;
    border-radius:2px;
  }
  .doc-type-row button.active{ background:var(--navy); color:#fff; border-color:var(--navy); }

  .btn-primary{
    margin-top:22px; width:100%; padding:13px; background:var(--navy); color:#fff; border:none;
    font-family:'Zilla Slab',serif; font-weight:600; font-size:15px; letter-spacing:0.4px; cursor:pointer;
    border-radius:2px; transition:background .15s ease;
  }
  .btn-primary:hover{background:var(--navy-2);}
  .btn-primary:disabled{background:#A9AFB8; cursor:not-allowed;}

  .hint{font-size:12px; color:var(--slate); margin-top:8px; line-height:1.5;}

  /* QR result / stamp card */
  .stamp-card{
    display:flex; flex-direction:column; align-items:center; text-align:center;
    padding:28px 20px 22px; border:1px solid var(--navy);
    position:relative; background:
      repeating-linear-gradient(45deg, transparent, transparent 10px, rgba(19,42,70,0.015) 10px, rgba(19,42,70,0.015) 20px);
  }
  .stamp-card .reg-no{
    font-family:'JetBrains Mono',monospace; font-size:13px; color:var(--red-stamp); font-weight:600;
    border:1.5px solid var(--red-stamp); padding:3px 10px; border-radius:2px; transform:rotate(-1.5deg);
    margin-bottom:16px; letter-spacing:0.5px;
  }
  #qrcode{padding:10px; background:#fff; border:1px solid var(--line);}
  .stamp-card .doc-label{
    font-family:'Zilla Slab',serif; font-size:16px; font-weight:600; margin-top:14px; color:var(--navy);
  }
  .stamp-card .doc-url{
    font-family:'JetBrains Mono',monospace; font-size:11px; color:var(--slate); margin-top:4px;
    word-break:break-all; max-width:280px;
  }
  .empty-state{
    padding:50px 20px; text-align:center; color:var(--slate);
  }
  .empty-state .glyph{font-size:34px; margin-bottom:10px; opacity:0.4;}
  .empty-state p{font-size:13px; line-height:1.6; max-width:260px; margin:0 auto;}

  .action-row{display:flex; gap:10px; margin-top:18px; width:100%;}
  .action-row button{
    flex:1; padding:9px; font-family:'Inter',sans-serif; font-weight:600; font-size:12.5px;
    border:1px solid var(--navy); background:#fff; color:var(--navy); cursor:pointer; border-radius:2px;
  }
  .action-row button.solid{background:var(--navy); color:#fff;}
  .action-row button:hover{opacity:0.85;}

  /* Register table */
  .register{grid-column:1/-1;}
  table{width:100%; border-collapse:collapse; font-size:13px;}
  thead th{
    text-align:left; font-family:'JetBrains Mono',monospace; font-size:10.5px; text-transform:uppercase;
    letter-spacing:0.8px; color:var(--slate); padding:8px 10px; border-bottom:2px solid var(--navy);
  }
  tbody td{padding:10px; border-bottom:1px solid var(--line); vertical-align:middle;}
  tbody tr:hover{background:rgba(19,42,70,0.03);}
  .reg-tag{font-family:'JetBrains Mono',monospace; color:var(--red-stamp); font-weight:600; font-size:12px;}
  .mini-qr{width:46px; height:46px; border:1px solid var(--line); padding:3px; background:#fff;}
  .link-cell a{color:var(--navy-2); text-decoration:none; font-size:12px;}
  .link-cell a:hover{text-decoration:underline;}
  .type-pill{
    font-family:'JetBrains Mono',monospace; font-size:10px; padding:2px 7px; border-radius:20px;
    background:rgba(31,107,60,0.1); color:var(--green); font-weight:600; text-transform:uppercase;
  }
  .row-actions button{
    background:none; border:none; cursor:pointer; color:var(--slate); font-size:16px; padding:2px 6px;
  }
  .row-actions button:hover{color:var(--red-stamp);}

  footer{
    text-align:center; padding:20px; font-family:'JetBrains Mono',monospace; font-size:11px; color:var(--slate);
    border-top:1px solid var(--line);
  }
</style>
</head>
<body>
<div class="ledger-bg"></div>

<header>
  <div>
    <div class="brand-eyebrow">Utility &middot; Link to QR</div>
    <h1>Document QR Register</h1>
  </div>
  <div class="reg-meta" id="today-meta"></div>
</header>

<main>
  <section class="panel">
    <div class="panel-inner">
      <h2>New Entry</h2>

      <label for="doc-link">Document Link</label>
      <input type="url" id="doc-link" placeholder="https://drive.google.com/... or any document URL">

      <label for="doc-label">Document Name / Description</label>
      <input type="text" id="doc-label" placeholder="e.g. NOC Application - HPPSC SO 2026">

      <label>Document Type</label>
      <div class="doc-type-row" id="type-row">
        <button data-type="Order" type="button">Order</button>
        <button data-type="Report" type="button">Report</button>
        <button data-type="Certificate" type="button">Certificate</button>
        <button data-type="Form" type="button">Form</button>
        <button data-type="Other" type="button" class="active">Other</button>
      </div>

      <button class="btn-primary" id="generate-btn">Generate QR & Register Entry</button>
      <p class="hint">Har link ke liye ek register number (QR/2026/00X) apne aap ban jayega — jaise daily diary entry. Koi bhi document link chalega: Google Drive, PDF, government portal, ya kuch bhi.</p>
    </div>
  </section>

  <section class="panel">
    <div class="panel-inner" id="preview-panel">
      <h2>Latest QR</h2>
      <div class="empty-state" id="empty-state">
        <div class="glyph">▦</div>
        <p>Link daaliye aur Generate dabaiye — QR code yahan dikhega, download aur print ke liye ready.</p>
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
      <h2>Register Log (<span id="entry-count">0</span> entries)</h2>
      <table>
        <thead>
          <tr>
            <th>Reg. No.</th>
            <th>QR</th>
            <th>Document</th>
            <th>Type</th>
            <th>Link</th>
            <th></th>
          </tr>
        </thead>
        <tbody id="log-body">
          <tr id="no-entries-row">
            <td colspan="6" style="text-align:center; color:var(--slate); padding:24px;">Abhi koi entry nahi hai. Pehla document link daalke shuru kariye.</td>
          </tr>
        </tbody>
      </table>
    </div>
  </section>
</main>

<footer>DOCUMENT QR REGISTER &middot; All entries stored only in this browser session</footer>

<script>
let entries = [];
let counter = 0;
let selectedType = "Other";
const yearNow = new Date().getFullYear();

document.getElementById('today-meta').textContent =
  new Date().toLocaleDateString('en-IN', { day:'2-digit', month:'short', year:'numeric' });

document.querySelectorAll('#type-row button').forEach(btn=>{
  btn.addEventListener('click', ()=>{
    document.querySelectorAll('#type-row button').forEach(b=>b.classList.remove('active'));
    btn.classList.add('active');
    selectedType = btn.dataset.type;
  });
});

function padNum(n){ return String(n).padStart(3,'0'); }

function makeRegNo(){
  counter++;
  return `QR/${yearNow}/${padNum(counter)}`;
}

function renderQR(containerEl, text, size){
  containerEl.innerHTML = "";
  new QRCode(containerEl, {
    text: text,
    width: size,
    height: size,
    colorDark: "#132A46",
    colorLight: "#ffffff",
    correctLevel: QRCode.CorrectLevel.M
  });
}

document.getElementById('generate-btn').addEventListener('click', ()=>{
  const linkInput = document.getElementById('doc-link');
  const labelInput = document.getElementById('doc-label');
  const link = linkInput.value.trim();
  let label = labelInput.value.trim();

  if(!link){
    linkInput.style.borderColor = '#9B2C2C';
    linkInput.focus();
    return;
  }
  linkInput.style.borderColor = '';
  if(!label) label = "Untitled Document";

  const regNo = makeRegNo();
  const entry = { regNo, link, label, type: selectedType, date: new Date().toLocaleDateString('en-IN') };
  entries.unshift(entry);

  // Update preview
  document.getElementById('empty-state').style.display = 'none';
  const card = document.getElementById('stamp-card');
  card.style.display = 'flex';
  document.getElementById('reg-no-display').textContent = regNo;
  document.getElementById('label-display').textContent = label;
  document.getElementById('url-display').textContent = link;
  renderQR(document.getElementById('qrcode'), link, 180);

  // Update log table
  renderLog();

  // Reset inputs
  linkInput.value = "";
  labelInput.value = "";
});

function renderLog(){
  const tbody = document.getElementById('log-body');
  document.getElementById('entry-count').textContent = entries.length;
  if(entries.length === 0){
    tbody.innerHTML = `<tr id="no-entries-row"><td colspan="6" style="text-align:center; color:var(--slate); padding:24px;">Abhi koi entry nahi hai. Pehla document link daalke shuru kariye.</td></tr>`;
    return;
  }
  tbody.innerHTML = "";
  entries.forEach((e, idx)=>{
    const tr = document.createElement('tr');
    tr.innerHTML = `
      <td class="reg-tag">${e.regNo}</td>
      <td><div class="mini-qr" id="mini-${idx}"></div></td>
      <td>${e.label}<br><span style="font-size:11px; color:var(--slate);">${e.date}</span></td>
      <td><span class="type-pill">${e.type}</span></td>
      <td class="link-cell"><a href="${e.link}" target="_blank" rel="noopener">Open link ↗</a></td>
      <td class="row-actions"><button title="Remove entry" data-idx="${idx}">✕</button></td>
    `;
    tbody.appendChild(tr);
    renderQR(document.getElementById(`mini-${idx}`), e.link, 40);
  });

  tbody.querySelectorAll('.row-actions button').forEach(btn=>{
    btn.addEventListener('click', (ev)=>{
      const i = parseInt(ev.target.dataset.idx, 10);
      entries.splice(i,1);
      renderLog();
    });
  });
}

document.getElementById('download-btn').addEventListener('click', ()=>{
  const img = document.querySelector('#qrcode img') || document.querySelector('#qrcode canvas');
  if(!img) return;
  const link = document.createElement('a');
  const regNo = document.getElementById('reg-no-display').textContent.replace(/\//g,'-');
  link.download = `${regNo}.png`;
  link.href = img.src || img.toDataURL('image/png');
  link.click();
});

document.getElementById('print-btn').addEventListener('click', ()=>{
  window.print();
});
</script>

</body>
</html>
