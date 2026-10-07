<!DOCTYPE html>
<html lang="kk">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>Робот не киеді? — ойын</title>
<style>
  :root{
    --bg:#fdf1dd; --card:#ffffff; --text:#2b2a28; --muted:#7a756c;
    --purple:#7c5cff; --teal:#0d9488; --red:#e0473e; --good:#16a34a; --bad:#dc2626;
    --border:#f0d9b5; --shadow:0 6px 20px rgba(43,42,40,.08);
    box-sizing:border-box;
    padding-top:env(safe-area-inset-top,0px);
    padding-bottom:env(safe-area-inset-bottom,0px);
  }
  *{box-sizing:border-box;}
  body{margin:0;background:var(--bg);color:var(--text);font-family:Georgia, 'Times New Roman', serif;}
  .wrap{max-width:760px;margin:0 auto;padding:32px 18px 60px;}
  header{text-align:center;margin-bottom:20px;}
  header h1{font-size:clamp(26px,6vw,38px);margin:0 0 6px;}
  header p{color:var(--muted);margin:0;font-size:16px;}
  .score{
    text-align:center;background:var(--card);border:2px solid var(--border);border-radius:18px;
    padding:14px;margin-bottom:22px;font-size:18px;box-shadow:var(--shadow);
  }
  .score b{color:var(--purple);font-size:22px;}
  .scene{
    background:var(--card);border-radius:24px;padding:36px 24px;text-align:center;
    box-shadow:var(--shadow);margin-bottom:22px;
  }
  .scene .icon{font-size:96px;line-height:1;margin-bottom:10px;}
  .scene h2{font-size:26px;margin:0 0 6px;}
  .scene p{color:var(--muted);font-size:17px;margin:0;}
  .options{display:grid;grid-template-columns:1fr 1fr;gap:14px;margin-top:8px;}
  @media (max-width:480px){.options{grid-template-columns:1fr;}}
  .opt{
    background:var(--card);border:3px solid var(--border);border-radius:18px;padding:22px 14px;
    font-family:inherit;font-size:18px;cursor:pointer;display:flex;flex-direction:column;align-items:center;gap:8px;
    transition:.15s;
  }
  .opt .e{font-size:48px;}
  .opt:hover{border-color:var(--purple);}
  .opt.correct{border-color:var(--good);background:#eafbf1;}
  .opt.wrong{border-color:var(--bad);background:#fdecea;}
  .opt:disabled{cursor:default;}
  .feedback{text-align:center;font-size:19px;font-weight:700;min-height:28px;margin-top:14px;}
  .feedback.ok{color:var(--good);}
  .feedback.no{color:var(--bad);}
  .next{
    display:block;margin:18px auto 0;padding:12px 28px;border:none;border-radius:999px;
    background:var(--purple);color:#fff;font-size:17px;font-weight:700;cursor:pointer;
  }
  .next:disabled{opacity:.4;cursor:default;}
  .done{text-align:center;padding:40px 20px;}
  .done .big{font-size:80px;}
  .restart{
    margin-top:18px;padding:12px 26px;border:none;border-radius:999px;background:var(--teal);
    color:#fff;font-size:16px;font-weight:700;cursor:pointer;
  }
  footer{text-align:center;color:var(--muted);font-size:13px;margin-top:10px;}
</style>
</head>
<body>
<div class="wrap">
  <header>
    <h1>🤖 Робот не киеді?</h1>
    <p>Ауа райына қарай дұрыс затты таңда — «егер... онда... әйтпесе...»</p>
  </header>
  <div class="score">Ұпай: <b id="score">0</b> / <span id="total">6</span></div>
  <div id="game"></div>
</div>

<script>
const scenes = [
  { icon:"🌧️", title:"Жаңбыр жауды!", hint:"Не аласың?", opts:[
    {e:"☂️", t:"Қолшатыр", ok:true}, {e:"🕶️", t:"Көзілдірік", ok:false},
    {e:"🧢", t:"Кепка", ok:false}, {e:"🩴", t:"Шлепка", ok:false}
  ]},
  { icon:"☀️", title:"Күн ашық!", hint:"Не кигенің дұрыс?", opts:[
    {e:"☂️", t:"Қолшатыр", ok:false}, {e:"🧢", t:"Кепка", ok:true},
    {e:"🧤", t:"Қолғап", ok:false}, {e:"🥾", t:"Етік", ok:false}
  ]},
  { icon:"❄️", title:"Қар жауды!", hint:"Не кигенің жылы болады?", opts:[
    {e:"🧢", t:"Кепка", ok:false}, {e:"🧥", t:"Куртка", ok:true},
    {e:"👕", t:"Жейде", ok:false}, {e:"🩳", t:"Шорты", ok:false}
  ]},
  { icon:"💨", title:"Жел тұрды!", hint:"Не кигенің ыңғайлы?", opts:[
    {e:"🧥", t:"Куртка", ok:true}, {e:"🩱", t:"Купальник", ok:false},
    {e:"☀️", t:"Көзілдірік", ok:false}, {e:"🩴", t:"Шлепка", ok:false}
  ]},
  { icon:"🌧️", title:"Жаңбыр тоқтамай жауды", hint:"Аяғыңа не кигенің жөн?", opts:[
    {e:"🥾", t:"Резеңке етік", ok:true}, {e:"🩴", t:"Шлепка", ok:false},
    {e:"👟", t:"Кроссовка", ok:false}, {e:"🥿", t:"Туфли", ok:false}
  ]},
  { icon:"☀️", title:"Ыстық жаз күні", hint:"Қай затты аласың?", opts:[
    {e:"🧴", t:"Су бөтелкесі", ok:true}, {e:"🧣", t:"Шарф", ok:false},
    {e:"🧤", t:"Қолғап", ok:false}, {e:"☂️", t:"Қолшатыр", ok:false}
  ]},
];

let idx = 0, score = 0;
document.getElementById('total').textContent = scenes.length;

function render(){
  const g = document.getElementById('game');
  if(idx >= scenes.length){
    g.innerHTML = `
      <div class="done scene">
        <div class="big">🏆</div>
        <h2>Ойын бітті!</h2>
        <p>Нәтиже: ${score} / ${scenes.length}</p>
        <button class="restart" id="restart">Қайта ойнау</button>
      </div>`;
    document.getElementById('restart').onclick = ()=>{ idx=0; score=0; document.getElementById('score').textContent=score; render(); };
    return;
  }
  const s = scenes[idx];
  const optsHtml = s.opts.map((o,i)=>`<button class="opt" data-i="${i}"><span class="e">${o.e}</span>${o.t}</button>`).join('');
  g.innerHTML = `
    <div class="scene">
      <div class="icon">${s.icon}</div>
      <h2>${s.title}</h2>
      <p>${s.hint}</p>
      <div class="options">${optsHtml}</div>
      <div class="feedback" id="fb"></div>
      <button class="next" id="nextBtn" disabled>Келесі →</button>
    </div>`;
  const buttons = g.querySelectorAll('.opt');
  buttons.forEach(btn=>{
    btn.onclick = ()=>{
      buttons.forEach(b=>b.disabled = true);
      const chosen = s.opts[Number(btn.dataset.i)];
      const fb = document.getElementById('fb');
      if(chosen.ok){
        btn.classList.add('correct');
        fb.textContent = 'Дұрыс! ✓'; fb.className = 'feedback ok';
        score++; document.getElementById('score').textContent = score;
      } else {
        btn.classList.add('wrong');
        buttons.forEach(b=>{ if(s.opts[Number(b.dataset.i)].ok) b.classList.add('correct'); });
        fb.textContent = 'Басқасын таңда едің, бірақ үйрендік!'; fb.className = 'feedback no';
      }
      document.getElementById('nextBtn').disabled = false;
    };
  });
  document.getElementById('nextBtn').onclick = ()=>{ idx++; render(); };
}
render();
</script>
</body>
</html>
