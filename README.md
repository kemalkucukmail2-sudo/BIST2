<!DOCTYPE html>
<html lang="tr">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width,initial-scale=1">
<title>BIST Akıllı Tarayıcı</title>
<style>
:root{color-scheme:dark;--bg:#0b1020;--card:#141c30;--line:#293650;--text:#edf2ff;--muted:#9ba9c2;--green:#29d49a;--red:#ff687c;--yellow:#f3ca63;--blue:#79a9ff}
*{box-sizing:border-box}
body{margin:0;background:var(--bg);color:var(--text);font:14px system-ui,Arial,sans-serif}
header,main{max-width:1100px;margin:auto;padding:16px}
h1{font-size:23px;margin:3px 0 7px}
p,.muted{color:var(--muted);line-height:1.5}
.toolbar,.stats,.filters{display:flex;gap:8px;flex-wrap:wrap;margin:12px 0}
button,select{background:#1c2942;border:1px solid #354564;color:var(--text);padding:11px;border-radius:10px;font:inherit}
button{cursor:pointer;font-weight:700}
button.primary{background:#286cda;border-color:#286cda}
button:disabled{opacity:.5}
.stat{background:var(--card);border:1px solid var(--line);border-radius:12px;padding:12px;flex:1;min-width:110px}
.stat span{font-size:12px;color:var(--muted)}
.stat b{display:block;font-size:21px;margin-top:5px}
.notice{background:#292519;border:1px solid #65542b;color:#f5dda0;padding:12px;border-radius:10px;line-height:1.5}
.status{margin:12px 0;color:var(--muted);white-space:pre-wrap}
.progress{height:5px;background:#202b42;border-radius:8px;overflow:hidden}
.bar{height:100%;width:0;background:var(--blue)}
.tablewrap{overflow:auto;border:1px solid var(--line);border-radius:12px;margin-top:12px}
table{width:100%;min-width:850px;border-collapse:collapse;background:var(--card)}
th,td{padding:11px 9px;text-align:left;border-bottom:1px solid var(--line);white-space:nowrap}
th{font-size:12px;color:var(--muted);background:#182239}
.good,.buy{color:var(--green)}.bad,.sell{color:var(--red)}.watch{color:var(--yellow)}
.sig{font-weight:800}
footer{padding:16px 0;color:var(--muted);font-size:12px;line-height:1.6}
@media(max-width:600px){h1{font-size:21px}.toolbar>*{flex:1}.stat{min-width:calc(50% - 8px)}button{min-height:42px}}
</style>
</head>
<body>
<header>
<h1>📊 BIST Akıllı Tarayıcı</h1>
<p>1 saatlik sinyaller · 4 saatlik EMA200 filtresi · 100 puanlık sıralama</p>
</header>
<main>
<div class="notice">
<b>Veri uyarısı:</b> Bu sürüm ücretsiz harici veri kaynaklarını kullanmayı dener.
Kaynaklar erişimi engellerse tarama sonuç vermeyebilir. Sinyaller yalnızca
teknik analiz amaçlıdır; otomatik alım satım yapmaz.
</div>

<div class="toolbar">
<button class="primary" id="scan">▶ TARAMAYI BAŞLAT</button>
<button id="stop">Durdur</button>
<select id="universe">
<option value="all">Bulunabilen tüm BIST hisseleri</option>
<option value="sample">Örnek likit hisseler</option>
</select>
<select id="limit">
<option value="all">Tüm listeyi tara</option>
<option value="150">İlk 150 hisse</option>
<option value="80">İlk 80 hisse</option>
<option value="30">İlk 30 hisse</option>
</select>
</div>

<div class="stats">
<div class="stat"><span>Taranan</span><b id="count">0</b></div>
<div class="stat"><span>AL / Güçlü AL</span><b id="buys" class="good">0</b></div>
<div class="stat"><span>SAT / Güçlü SAT</span><b id="sells" class="bad">0</b></div>
<div class="stat"><span>4s trend uygun</span><b id="trends">0</b></div>
</div>

<div class="progress"><div class="bar" id="bar"></div></div>
<div class="status" id="status">Hazır. İlk deneme için 30 veya 80 hisse seçebilirsin.</div>

<div class="filters" id="filters">
<button data-f="all">Tümü</button>
<button data-f="YÜKSELİŞ ÖNCESİ TAKİP">Yükseliş öncesi</button>
<button data-f="AL">AL</button>
<button data-f="GÜÇLÜ AL">Güçlü AL</button>
<button data-f="DÜŞÜŞ ÖNCESİ TAKİP">Düşüş öncesi</button>
<button data-f="SAT">SAT</button>
<button data-f="GÜÇLÜ SAT">Güçlü SAT</button>
</div>

<div class="tablewrap">
<table>
<thead><tr>
<th>Hisse</th><th>Fiyat</th><th>Puan</th><th>Sinyal</th>
<th>EMA20 / EMA50</th><th>RSI</th><th>MACD</th>
<th>Hacim</th><th>4s EMA200</th>
</tr></thead>
<tbody id="rows"><tr><td colspan="9">Henüz tarama yapılmadı.</td></tr></tbody>
</table>
</div>

<footer>
<b>Puanlama sistemi</b><br>
• EMA20, EMA50 üzerinde: 25 puan.<br>
• Fiyat EMA20 üzerinde: 15 puan.<br>
• MACD pozitif: 20 puan.<br>
• RSI 50–70 aralığında: 20 puan; 45–50 aralığında: 10 puan.<br>
• Hacim, önceki 20 mum ortalamasının en az 1,5 katı: 20 puan.<br><br>
<b>4 saatlik filtre:</b> Saatlik mumlardan aynı BIST işlem günündeki
4 saatlik mumlar oluşturulmaya çalışılır. Son kapanmış 4 saatlik mumun
kapanışı EMA200 üzerinde değilse AL ve Yükseliş Öncesi Takip engellenir.
200 adet 4 saatlik mum yoksa filtre uygun sayılmaz.<br><br>
Tarama yalnızca düğmeye basınca başlar. Veri sağlayıcının saatleri,
seans yapısı veya erişim kısıtlamaları sonuçları etkileyebilir.
Bu araç emir göndermez ve yatırım tavsiyesi vermez.
</footer>
</main>

<script>
const $=id=>document.getElementById(id);
let stopRequested=false,results=[],activeFilter='all';

const sample=[
'THYAO','AKBNK','GARAN','YKBNK','ISCTR','KCHOL','SAHOL',
'ASELS','BIMAS','EREGL','TUPRS','SISE','FROTO','TOASO',
'TCELL','ENKAI','PGSUS','HEKTS','KOZAL','KOZAA','PETKM',
'SASA','TAVHL','MGROS','CCOLA','ARCLK','VESTL','DOAS',
'ALARK','GUBRF','EKGYO','ENJSA','ULKER','MPARK','AEFES',
'ASTOR','KONTR','GESAN','EUPWR','HALKB','VAKBN','TSKB',
'ODAS','CANTE','BRSAN'
];

const proxy=url=>'https://api.allorigins.win/raw?url='+encodeURIComponent(url);

async function getJSON(url){
 const r=await fetch(proxy(url));
 if(!r.ok)throw Error('Veri sunucusu HTTP '+r.status);
 return await r.json();
}

function ema(a,n){
 if(a.length<n)return null;
 const k=2/(n+1);
 let v=a.slice(0,n).reduce((s,x)=>s+x,0)/n;
 for(let i=n;i<a.length;i++)v=a[i]*k+v*(1-k);
 return v;
}

function emaSeries(a,n){
 const out=Array(a.length).fill(null);
 if(a.length<n)return out;
 const k=2/(n+1);
 let v=a.slice(0,n).reduce((s,x)=>s+x,0)/n;
 out[n-1]=v;
 for(let i=n;i<a.length;i++){v=a[i]*k+v*(1-k);out[i]=v}
 return out;
}

function calcRSI(a,n=14){
 if(a.length<n+1)return null;
 let gain=0,loss=0;
 for(let i=a.length-n;i<a.length;i++){
  const d=a[i]-a[i-1];
  if(d>0)gain+=d;else loss-=d;
 }
 if(loss===0)return 100;
 return 100-100/(1+(gain/n)/(loss/n));
}

function calcMACD(a){
 if(a.length<35)return null;
 const f=emaSeries(a,12),s=emaSeries(a,26);
 const line=a.map((_,i)=>f[i]===null||s[i]===null?null:f[i]-s[i]);
 const valid=line.filter(x=>x!==null);
 if(valid.length<9)return null;
 const signal=ema(valid,9);
 return valid[valid.length-1]-signal;
}

/* Mumları İstanbul yerel tarihine ve seans saatine göre gruplandırır.
   BIST için yaklaşık 10:00–14:00 ve 14:00–18:00 dilimleri kullanılır. */
function make4hBars(bars){
 const groups=new Map();
 for(const b of bars){
  const parts=new Intl.DateTimeFormat('en-GB',{
   timeZone:'Europe/Istanbul',year:'numeric',month:'2-digit',
   day:'2-digit',hour:'2-digit',hourCycle:'h23'
  }).formatToParts(new Date(b.time*1000));
  const v={};
  for(const p of parts)v[p.type]=p.value;
  const hour=Number(v.hour);
  if(hour<10||hour>=18)continue;
  const slot=hour<14?0:1;
  const key=v.year+'-'+v.month+'-'+v.day+'-'+slot;
  if(!groups.has(key))groups.set(key,[]);
  groups.get(key).push(b);
 }
 const now=Date.now()/1000,out=[];
 for(const group of groups.values()){
  group.sort((a,b)=>a.time-b.time);
  // Eksik seans mumlarını tamamlanmış 4 saatlik mum sayma.
  if(group.length!==4)continue;
  const last=group[group.length-1];
  if(now<last.time+3600)continue;
  out.push({close:last.close,time:last.time});
 }
 out.sort((a,b)=>a.time-b.time);
 return out;
}

async function getSymbols(){
 if($('universe').value==='sample')return sample.map(s=>s+'.IS');

 const url='https://cdn.jsdelivr.net/gh/ahmeterenodaci/Istanbul-Stock-Exchange--BIST--including-symbols-and-logos@main/without_logo.json';
 const json=await getJSON(url);
 let arr=Array.isArray(json)?json:
  (json.data||json.symbols||json.stocks||json.items||[]);
 const syms=arr.map(x=>typeof x==='string'?x:
  (x.symbol||x.ticker||x.code||x.Symbol||x.Code))
  .filter(Boolean)
  .map(s=>String(s).toUpperCase().replace(/\.IS$/,''))
  .filter(s=>/^[A-Z0-9]{3,6}$/.test(s));
 const unique=[...new Set(syms)];
 if(!unique.length)throw Error('Hisse listesi okunamadı');
 return unique.map(s=>s+'.IS');
}

async function analyse(symbol){
 const url='https://query1.finance.yahoo.com/v8/finance/chart/'+
  encodeURIComponent(symbol)+'?range=1y&interval=60m';
 const json=await getJSON(url);
 const res=json.chart&&json.chart.result&&json.chart.result[0];
 if(!res||!res.timestamp)throw Error('Mum verisi yok');

 const q=res.indicators.quote[0],bars=[];
 for(let i=0;i<res.timestamp.length;i++){
  if(q.close[i]==null||q.high[i]==null||q.low[i]==null||
     q.open[i]==null)continue;
  bars.push({
   time:res.timestamp[i],open:q.open[i],high:q.high[i],
   low:q.low[i],close:q.close[i],volume:q.volume[i]||0
  });
 }
 if(bars.length<220)throw Error('Yeterli saatlik geçmiş yok');

 const closes=bars.map(b=>b.close);
 const volumes=bars.map(b=>b.volume);
 const price=closes[closes.length-1];
 const e20=ema(closes,20),e50=ema(closes,50);
 const rsi=calcRSI(closes),macd=calcMACD(closes);
 const previousVolumes=volumes.slice(-21,-1);
 const avgVol=previousVolumes.reduce((s,x)=>s+x,0)/previousVolumes.length;
 const volRatio=avgVol>0?volumes[volumes.length-1]/avgVol:0;

 const four=make4hBars(bars);
 const fourCloses=four.map(b=>b.close);
 const e200=ema(fourCloses,200);
 const last4=fourCloses[fourCloses.length-1];
 const trendOk=e200!==null&&last4>e200;

 let score=0;
 if(e20>e50)score+=25;
 if(price>e20)score+=15;
 if(macd!==null&&macd>0)score+=20;
 if(rsi!==null&&rsi>=50&&rsi<=70)score+=20;
 else if(rsi!==null&&rsi>=45&&rsi<50)score+=10;
 if(volRatio>=1.5)score+=20;

 const bullish=e20>e50&&price>e20&&macd!==null&&macd>0&&rsi>=50;
 const bearish=e20<e50&&price<e20&&macd!==null&&macd<0&&rsi<50;
 const preUp=e20>e50&&price>e20&&macd!==null&&macd>0&&rsi>=45;
 const preDown=e20<e50&&price<e20&&macd!==null&&macd<0&&rsi<55;

 let signal='NÖTR';
 if(bullish&&score>=80&&volRatio>=1.5)signal='GÜÇLÜ AL';
 else if(bullish&&score>=60)signal='AL';
 else if(preUp&&score>=45)signal='YÜKSELİŞ ÖNCESİ TAKİP';
 else if(bearish&&volRatio>=1.5)signal='GÜÇLÜ SAT';
 else if(bearish)signal='SAT';
 else if(preDown)signal='DÜŞÜŞ ÖNCESİ TAKİP';

 const upward=[
  'AL','GÜÇLÜ AL','YÜKSELİŞ ÖNCESİ TAKİP'
 ];
 if(upward.includes(signal)&&!trendOk)signal='TREND UYGUN DEĞİL';

 return {
  symbol:symbol.replace('.IS',''),price,score,signal,
  e20,e50,rsi,macd,volRatio,trendOk,e200,
  enough4h:four.length>=200
 };
}

function fmt(n,d=2){
 if(n===null||n===undefined||!Number.isFinite(n))return '—';
 return n.toLocaleString('tr-TR',{maximumFractionDigits:d,minimumFractionDigits:d});
}

function render(){
 let list=results.filter(x=>{
  if(activeFilter==='all')return true;
  if(activeFilter==='AL'||activeFilter==='GÜÇLÜ AL'||
     activeFilter==='SAT'||activeFilter==='GÜÇLÜ SAT')
   return x.signal===activeFilter;
  return x.signal===activeFilter;
 });
 list.sort((a,b)=>b.score-a.score);

 $('rows').innerHTML=list.length?list.map(x=>{
  let cls=x.signal.includes('AL')?'buy':
   (x.signal.includes('SAT')||x.signal.includes('DÜŞÜŞ')?'sell':'watch');
  let trend=x.enough4h?(x.trendOk?'UYGUN':'UYGUN DEĞİL'):'VERİ YETERSİZ';
  return `<tr>
   <td><b>${x.symbol}</b></td>
   <td>${fmt(x.price)}</td><td><b>${x.score}/100</b></td>
   <td class="sig ${cls}">${x.signal}</td>
   <td>${fmt(x.e20)} / ${fmt(x.e50)}</td>
   <td>${fmt(x.rsi,1)}</td><td>${fmt(x.macd,4)}</td>
   <td>${fmt(x.volRatio,2)}×</td>
   <td class="${x.trendOk?'good':'bad'}">${trend}</td>
  </tr>`;
 }).join(''):'<tr><td colspan="9">Bu filtrede sonuç bulunamadı.</td></tr>';

 $('count').textContent=results.length;
 $('buys').textContent=results.filter(x=>
  ['AL','GÜÇLÜ AL'].includes(x.signal)).length;
 $('sells').textContent=results.filter(x=>
  ['SAT','GÜÇLÜ SAT'].includes(x.signal)).length;
 $('trends').textContent=results.filter(x=>x.trendOk).length;
}

$('filters').addEventListener('click',e=>{
 const b=e.target.closest('button[data-f]');
 if(!b)return;
 activeFilter=b.dataset.f;
 render();
});

$('stop').onclick=()=>{
 stopRequested=true;
 $('status').textContent='Durdurma istendi; mevcut veri isteği bitince duracak.';
};

$('scan').onclick=async()=>{
 stopRequested=false;results=[];
 $('scan').disabled=true;
 $('rows').innerHTML='<tr><td colspan="9">Veri bekleniyor…</td></tr>';
 $('bar').style.width='0%';

 try{
  $('status').textContent='Hisse listesi alınıyor…';
  let symbols=await getSymbols();
  const limit=$('limit').value;
  if(limit!=='all')symbols=symbols.slice(0,Number(limit));

  let failed=0,done=0;
  $('status').textContent=`${symbols.length} hisse taranacak.`;

  for(const symbol of symbols){
   if(stopRequested)break;
   try{
    const result=await analyse(symbol);
    results.push(result);
   }catch(err){
    failed++;
   }
   done++;
   $('bar').style.width=(100*done/symbols.length)+'%';
   $('status').textContent=
    `İşleniyor: ${done}/${symbols.length}\n`+
    `Sonuç alınan: ${results.length} · Veri alınamayan: ${failed}`;
   render();
   await new Promise(resolve=>setTimeout(resolve,180));
  }

  render();
  $('status').textContent=stopRequested?
   `Tarama durduruldu. ${done} hisse denendi, ${results.length} sonuç alındı.`:
   `Tarama tamamlandı. ${done} hisse denendi; ${results.length} sonuç alındı, ${failed} hissede veri alınamadı.`;

  if(results.length===0){
   $('status').textContent+=
    '\nVeri kaynağına erişilemiyor olabilir. Sonuç yokken sinyal üretilmez.';
  }
 }catch(err){
  $('status').textContent='Tarama başlatılamadı: '+err.message+
   '\nİnternet bağlantısı veya harici veri kaynağı erişimini kontrol et.';
 }finally{
  $('scan').disabled=false;
 }
};
</script>
</body>
</html>
