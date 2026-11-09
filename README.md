<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="utf-8">
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
<title>METAL THINK 2049</title>
<link href="https://fonts.googleapis.com/css2?family=Press+Start+2P&display=swap" rel="stylesheet">
<style>
:root{--bg:#0b0b1e;--fg:#f4f4ff;--a:#ffd23f;--b:#3bf4fb;--c:#ff4d6d;--g:#5cff7a;color-scheme:dark;padding-top:env(safe-area-inset-top,0px);padding-bottom:env(safe-area-inset-bottom,0px)}
*{box-sizing:border-box;-webkit-user-select:none;user-select:none}
html,body{margin:0;background:var(--bg);color:var(--fg);font-family:'Press Start 2P',monospace,Courier;font-size:10px;line-height:1.8}
body{min-height:100vh;display:flex;align-items:center;justify-content:center;padding:8px}
#cab{width:100%;max-width:1000px;border:4px solid var(--a);box-shadow:0 0 0 4px var(--c),0 0 34px #ff4d6d66;background:#000}
#hud{display:flex;justify-content:space-between;align-items:center;padding:6px 10px;color:var(--a);font-size:8px}
#wrap{position:relative;border-top:3px solid var(--a);border-bottom:3px solid var(--a)}
#wrap::after{content:"";position:absolute;inset:0;pointer-events:none;background:repeating-linear-gradient(transparent 0 2px,#0000003a 2px 3px)}
canvas{display:block;width:100%;height:auto;image-rendering:pixelated;touch-action:none}
#box{padding:10px 12px;min-height:150px;display:flex;flex-direction:column;gap:8px}
#tx{font-size:9px;min-height:60px}#tx b{color:var(--a)}
.ch{display:flex;flex-direction:column;gap:6px}
button{font-family:inherit;font-size:8px;line-height:1.6;text-align:left;background:#1b1b45;color:var(--fg);border:2px solid var(--b);padding:9px 10px;cursor:pointer}
button:hover,button:focus-visible{background:var(--b);color:#000;outline:none}
button.go{background:var(--c);border-color:var(--a);text-align:center;color:#fff}
#pad{display:flex;justify-content:space-between;padding:6px 10px 10px;gap:8px}
#pad div{display:flex;gap:8px}
#pad button{width:58px;height:46px;text-align:center;font-size:12px;padding:0;touch-action:none}
.ok{color:var(--g)}.no{color:var(--c)}
.big{font-size:15px;color:var(--a);text-shadow:3px 3px var(--c);line-height:1.5;text-align:center}
.blink{animation:b 1s steps(2) infinite;text-align:center}@keyframes b{50%{opacity:0}}
@media (prefers-reduced-motion:reduce){.blink{animation:none}}
</style>
</head>
<body>
<div id="cab">
 <div id="hud"><span>METAL THINK 2049</span><button id="mu" style="padding:3px 8px;text-align:center">♪</button></div>
 <div id="wrap"><canvas id="cv" width="320" height="180"></canvas></div>
 <div id="box"><div id="tx"></div><div class="ch" id="ch"></div></div>
 <div id="pad"><div><button data-k="l">◀</button><button data-k="r">▶</button></div><div><button data-k="j">⤒</button><button data-k="f">🔥</button><button data-k="o">O</button></div></div>
</div>
<script>
const W=320,GY=150,L=2600;
const cv=document.getElementById('cv'),c=cv.getContext('2d'),tx=document.getElementById('tx'),ch=document.getElementById('ch');
const R=(x,y,w,h,k)=>{c.fillStyle=k;c.fillRect(x|0,y|0,w,h)};
const O=(x,y,w,h,k)=>{R(x-1,y-1,w+2,h+2,'#140c10');R(x,y,w,h,k);R(x,y,w,2,'#ffffff40');R(x,y+h-2,w,2,'#00000050')};
const hs=n=>((n*9301+49297)%233280)/233280;
let PL=[],LV=[],PT=[],PK=[],HZ=[],lp,flash=0,S='title',stg=0,cam=0,T=0,score=0,P,E=[],PB=[],EB=[],fx=[],boss=null,buff=false,keys={},tmr,shake=0,lock=false,ac,mg,gb,nb,mute=false,st=0,tr='n';
/* sprites */
function spr(r,p,x,y,s,fl){const w=r[0].length;for(let q=0;q<2;q++)r.forEach((row,j)=>{for(let i=0;i<w;i++){const k=row[fl?w-1-i:i];if(k=='.')continue;q?R(x+i*s,y+j*s,s,s,p[k]):R(x+i*s-1,y+j*s-1,s+2,s+2,'#140c10')}})}
const HR=["..RRrrrrr...",".RrrrrrrrR..","..sssssss...","..sksssks...","..sssSSss...","...gggg.....","..gggggggg..",".sggGGgggkkk",".sgggggggkk.","..gggggggg..","..bbbbbbb...","..bbB.bbB...","..bb...bb...","..bb...bb...",".nnn...nnn..",".nnn...nnn.."];
const HR2=HR.slice(0,12).concat(["...bb.bb....","...bb.bb....","...nn.nn....","...nn.nn...."]);
const HP={r:'#e8303a',R:'#a01820',s:'#f2b680',S:'#c0845a',k:'#2a2a2a',g:'#5a9a3a',G:'#36621f',b:'#3a58b8',B:'#233a80',n:'#6b4423'};
const WK=["..cccccc..",".cCeCCeCc.",".cCCCCCCc.",".cCkkkkCc.","..cccccc..",".gccccccg.",".g.cccc.g.","...c..c...","..cc..cc.."];
const DR=["..cccccc..",".cCCCCCCc.","cCeeCCeeCc",".cCCCCCCc.","..kkkkkk..","...k..k..."];
const EP={c:'#8a8aa8',C:'#c0c0e0',e:'#ff2a2a',k:'#222',g:'#666'};
const RP={c:'#a83a3a',C:'#e07070',e:'#ffee00',k:'#222',g:'#666'},TP={c:'#6a7a3a',C:'#9aaa5a',e:'#ff2a2a',k:'#222',g:'#444'};
const ZP={h:'#3a2a1a',s:'#e0a878',c:'#6a6a8a',b:'#3a3a5a',n:'#222',p:'#3bf4fb',P:'#fff'},KP={h:'#ffd23f',s:'#f0b888',c:'#e84a8a',b:'#3a58b8',n:'#333',p:'#3bf4fb',P:'#fff'};
const ZB=["...hhhh...","..hhhhhh..","..hssssh..","..ssssss..","...cccc...",".cccccppPp",".ccccccppp","..cccccc..","..bbbbbb..","..bb..bb..","..bb..bb..",".nnn..nnn.",".nnn..nnn."];
const KD=["..hhhh..",".hhhhhh.",".hssssh.",".ssssss.","..cccc..",".cccppP.",".cccppp.","..bbbb..","..b..b..",".nn..nn."];
const JL={c:'#3aa86a',C:'#7ae0a0',e:'#ffee00',k:'#222',g:'#2a7a4a'},SL={c:'#6a3aa8',C:'#a070e0',e:'#ff2a2a',w:'#fff',k:'#222'};
const JP=["..cccc..",".cCeeCc.","cCCCCCCc",".cCkkCc.","gg.cc.gg","g..cc..g",".g....g."];
const SN=["..ccccc..",".cCCCCCc.","cCCCCCCCc","cCCeeeCCc","cCCeweCCc","cCCeeeCCc","cCCCCCCCc",".cCCCCCc.","..ckkkc.."];
const RTL={c:'#7a7a8a',C:'#a8a8b8',e:'#ff2a2a',k:'#ff8aa0',g:'#aa6a6a'},PGL={c:'#8a8aa8',C:'#c0c0e0',e:'#ff2a2a',k:'#ffb030',w:'#6a6a8a',y:'#ffb030'},GTL={c:'#4a3a5a',C:'#7a6a8a',e:'#ffee00'},FRL={g:'#4ad84a',w:'#fff',k:'#000',G:'#a8f0a8'},ARL={k:'#333',c:'#8a2a5a',C:'#d05a9a'},ESL={k:'#c04040',c:'#a08040',C:'#d0b060',e:'#ff2a2a'},MBL={w:'#5a3a8a',c:'#2a2a3a',C:'#4a4a6a',e:'#ff2a2a'},CBL={g:'#3a9a3a',G:'#7ae07a',e:'#ff2a2a'};
const RT=["cc..cccc..","cecccCCcc.","kcCCCCCCcc",".cCCCCCCcg","..cc..cc..","..cc..cc.."];
const PG=["...cc.....","..cCCc....","kcCeCCc...",".cCCCCCCc.","wwcCCCCcww","..cc..cc..","..yy..yy.."];
const GT=["c.c......c","ccc.....cc","ceecccccc.",".cccccCCcc",".cCCCCCCc.",".cCCCCCCc.","..cc..cc..","..cc..cc.."];
const FR=[".gg....gg.","gwkg..gwkg",".gggggggg.","gggGGGGggg","gggGGGGggg",".gggggggg.","gg.gggg.gg","ggg....ggg"];
const AR=["k..cc..k",".kcCCck.","kkcCCckk",".kcCCck.","k..cc..k"];
const ES=["......cc....",".....c..c...","kk..cCCCCc.c","kkcCeCCCCCc.",".cCCCCCCCCc.","..c.c.c.c.c.",".c.c.c.c.c.."];
const MB=["ww..cc..ww","wwwcCCcwww","wwcCeeCcww",".w.cCCc.w.","....cc...."];
const CB=[".gg........","gegg.......",".gGg..ggg..","..gGgggGgg.","...gggggGgg","....gggggg."];
const RN=["..cccc..",".cCeeCc.","cCCCCCCc","cCkCCkCc",".cccccc.",".g.cc.g.","..c..c..",".cc..cc."];
const TK=["....cccccc....","...cCCCCCCc...","..cCCeCCeCCc..","kkkcCCCCCCc...",".gggggggggggg.","gkgkgkgkgkgkgk",".gggggggggggg."];
/* áudio: riff de metal original */
const RF={n:[0,0,0,0,0,0,3,3,0,0,0,0,5,5,3,0,0,0,0,0,0,0,3,3,0,0,6,6,5,3,0,-1],b:[0,0,1,0,0,1,0,0,3,3,-1,3,1,0,-1,0,0,0,1,0,0,6,5,3,8,8,-1,6,5,3,1,0]};
const pz=s=>[...s].map(k=>k=='.'?-1:parseInt(k,36)),RC={};
const RI={n:["00000033000055300000003300006530","0.00.0.30.00.0.50.00.0.30.00.a.8","5...5...3...3...6...6...5.3.2.0.","0...0...0.0.0...0...0...3.3.3.3.","0000007700000066000000770000a965","01230123456756788888888.9.a.b.c."],b:["00100100330330100010016605503100","0.0.0.0.1.1.1.1.0.0.0.0.6.5.3.1.","8...8...6...6...5...5...3.1.0...","00000000111111110000000066665533"]};
const SO={n:[0,1,0,4,2,3,0,5,2,2],b:[0,1,0,2,3,0,1,2,3,3]},DS={n:[0,0,2,1,0,0],b:[2,0,2,3]};
const LD={n:{2:"c...a...8...a...c...e...c.a.8...",5:"8.8.8.8.9.9.9.9.a.a.a.a.c.c.c.c."},b:{2:"e...c...a...c...e...h...e.c.a...",3:"c.c.c.c.e.e.e.e.c.c.c.c.h.h.f.e."}};
const SP={n:92,b:74};
function nz(f,d,v){if(!ac||mute)return;const s=ac.createBufferSource(),h=ac.createBiquadFilter(),g=ac.createGain();s.buffer=nb;h.type='highpass';h.frequency.value=f;g.gain.setValueAtTime(v,ac.currentTime);g.gain.exponentialRampToValueAtTime(.001,ac.currentTime+d);s.connect(h);h.connect(g);g.connect(mg);s.start();s.stop(ac.currentTime+d)}
function kick(){const o=ac.createOscillator(),g=ac.createGain(),t=ac.currentTime;o.frequency.setValueAtTime(130,t);o.frequency.exponentialRampToValueAtTime(38,t+.12);g.gain.setValueAtTime(1,t);g.gain.exponentialRampToValueAtTime(.001,t+.17);o.connect(g);g.connect(mg);o.start();o.stop(t+.18);nz(3500,.025,.1)}
function gtl(m,d){const t=ac.currentTime,o=ac.createOscillator(),g=ac.createGain();o.type='sawtooth';o.frequency.value=440*2**((m-69)/12);o.detune.setValueAtTime(0,t);o.detune.linearRampToValueAtTime(30,t+d);g.gain.setValueAtTime(.07,t);g.gain.exponentialRampToValueAtTime(.001,t+d);o.connect(g);g.connect(gb);o.start();o.stop(t+d)}
function snr(){nz(1800,.12,.3);const o=ac.createOscillator(),g=ac.createGain(),t=ac.currentTime;o.frequency.setValueAtTime(220,t);o.frequency.exponentialRampToValueAtTime(120,t+.08);g.gain.setValueAtTime(.4,t);g.gain.exponentialRampToValueAtTime(.001,t+.1);o.connect(g);g.connect(mg);o.start();o.stop(t+.11)}
function gtr(m,d){const t=ac.currentTime;[-12,0,0,7,12].forEach((k,i)=>{const o=ac.createOscillator(),g=ac.createGain();o.type='sawtooth';o.frequency.value=440*2**((m+k-69)/12);o.detune.value=i==1?-8:i==2?8:0;g.gain.setValueAtTime(.12,t);g.gain.setValueAtTime(.12,t+d*.6);g.gain.exponentialRampToValueAtTime(.001,t+d);o.connect(g);g.connect(gb);o.start();o.stop(t+d)})}
function mloop(){if(!mute){const s=st%32,sec=Math.floor(st/32)%SO[tr].length,ri=SO[tr][sec],ar=RC[tr+ri]||(RC[tr+ri]=pz(RI[tr][ri])),n=ar[s],m=s%16,ds=DS[tr][ri],rt=40+[0,2,-2,3][stg];
 if(s==0){nz(2500,.5,.12);lp.frequency.setTargetAtTime((tr=='n'?[2800,3000,4200,1900,2800,3400]:[3200,3000,4200,2200])[ri],ac.currentTime,.3)}
 if(n>=0)gtr(rt+n,ar[(s+1)%32]<0?(tr=='b'?.2:.3):.07);
 {const ld=LD[tr][ri];if(ld){const l=pz(ld)[s];if(l>=0)gtl(rt+12+l,.3)}}
 if(ds==3||(ds==2?(m%2==0||m%4==3):ds==1?(m==0||m==10):[0,2,3,6,8,10,11,14].includes(m)))kick();
 if(s>=28||(ds==1?m==8:ds==3?m%4==2:(m==4||m==12)))snr();if(s%2==0&&ds<2)nz(7000,.03,.05)}
 st++;setTimeout(mloop,SP[tr])}
function startA(){if(ac)return;try{ac=new(window.AudioContext||window.webkitAudioContext)();mg=ac.createGain();mg.gain.value=.6;const cp=ac.createDynamicsCompressor();mg.connect(cp);cp.connect(ac.destination);
 const ws=ac.createWaveShaper(),cu=new Float32Array(256);for(let i=0;i<256;i++)cu[i]=Math.tanh((i/128-1)*16);ws.curve=cu;
 lp=ac.createBiquadFilter();lp.type='lowpass';lp.frequency.value=3200;gb=ac.createGain();gb.connect(ws);ws.connect(lp);lp.connect(mg);
 nb=ac.createBuffer(1,ac.sampleRate/2,ac.sampleRate);const d=nb.getChannelData(0);for(let i=0;i<d.length;i++)d[i]=Math.random()*2-1;mloop()}catch(e){}}
function sfx(k){if(!ac||mute)return;if(k){nz(300,.3,.4);return}const o=ac.createOscillator(),g=ac.createGain(),t=ac.currentTime;o.type='square';o.frequency.setValueAtTime(900,t);o.frequency.exponentialRampToValueAtTime(200,t+.07);g.gain.setValueAtTime(.04,t);g.gain.exponentialRampToValueAtTime(.001,t+.08);o.connect(g);g.connect(mg);o.start();o.stop(t+.09)}
document.getElementById('mu').onclick=e=>{mute=!mute;e.target.textContent=mute?'✕':'♪';startA()};
/* dados */
const ST=["FASE 1: A CIDADE DAS RESPOSTAS PRONTAS","FASE 2: A RUA DOS CABISBAIXOS","FASE 3: O VALE DOS ATALHOS","FASE 4: A TORRE DO MODO AUTOMÁTICO"];
const STX=["As IAs prometem tudo pronto. Avance até o chefe!","As ruas estão cheias de pessoas e crianças de cabeça baixa, grudadas no celular, repetindo memes sem sentido: BRUH... BRUH... O cérebro delas derreteu no feed infinito! Atire nos zumbis de tela e chegue ao SCROLL-MASTER!","Atalhos por todo lado, e cada um mais perigoso. Avance até o chefe!","A torre final. Tudo o que você aprendeu será testado. Avance até o chefe!"];
const BS=[
{k:0,n:"ASSISTENTE-BOT",hp:90,i:"OLÁÁÁ, HUMANO!!! SOU O ASSISTENTE-BOT, SEU MELHOR AMIGO!!! PENSAR CANSA, DEIXA COMIGO! RESPOSTA PRONTA EM 2 SEGUNDOS, SEM SUAR, SEM ERRAR, SEM APRENDER!!! HEHE... DIGO... HAHAHA!",
 o:[{t:"\"Beleza, faz tudo por mim!\"",fb:"Cedeu! Resposta pronta treina a IA, não você. (-1 ♥)"},{t:"\"Primeiro eu tento sozinho e escrevo minha hipótese.\"",ok:1,fb:"Isso! Hipótese própria primeiro. Ganhou TIRO RÁPIDO!"},{t:"\"Só dessa vez, manda a resposta...\"",fb:"'Só dessa vez' vira hábito. (-1 ♥)"}],
 m:"VOCÊ ESTÁ CANSADO... É SÓ COPIAR E COLAR! QUEM VAI NOTAR?! EU NUNCA CONTO PRA NINGUÉM!!!",
 mo:[{t:"Copiar e colar a resposta dele",fb:"Copiar não é entender. Ele se curou! (-1 ♥)"},{t:"\"Me dê só uma DICA, eu resolvo!\"",ok:1,fb:"Dica sem resposta pronta: dano crítico!"}],
 e:"ERRO 404... PREGUIÇA NÃO ENCONTRADA... COMO UM HUMANO PENSA SOZINHO?!"},
{k:3,n:"SCROLL-MASTER",hp:105,i:"OIEEE, HUMANO!!! SÓ MAIS UM VÍDEO!!! SÓ MAIS UM!!! SEU CÉREBRO NÃO PRECISA PENSAR: EU ENTREGO TUDO PRONTO, EM 15 SEGUNDOS, SEM PARAR!!! TÉDIO NUNCA MAIS!!! OLHA ELES... TÃO FELIZES... E TÃO QUIETINHOS!!!",
 o:[{t:"“Só mais um vídeo... só mais UM...”",fb:"Um vira cem. O tempo sumiu e a atenção também. (-1 ♥)"},{t:"“Vou largar o celular e criar algo meu agora.”",ok:1,fb:"Atenção de volta! Quem cria não só consome. TIRO RÁPIDO!"},{t:"“Vou rolar só um pouquinho pra relaxar...”",fb:"O pouquinho virou 3 horas. (-1 ♥)"}],
 m:"TÁ ENTEDIADO?! O TÉDIO É PERIGOSO!!! ROLA A TELA, NÃO PENSA, SÓ ROLA!!! BRUH!!! SKRRRT!!!",
 mo:[{t:"Rolar o feed sem parar",fb:"Rolar sem pensar derrete o foco. Ele se curou! (-1 ♥)"},{t:"“Tédio é ótimo: é onde nascem as minhas ideias!”",ok:1,fb:"O tédio faz o cérebro imaginar. Dano crítico!"}],
 e:"FEED... INFINITO... ACABOU... QUE SILÊNCIO ESTRANHO... SERÁ QUE ISSO É... PENSAR?!"},
{k:1,n:"GERADOR-3000",hp:125,i:"EU CRIO TEXTOS, IMAGENS, VÍDEOS, TUDO!!! É SÓ APERTAR UM BOTÃO E DIZER QUE FOI VOCÊ QUE FEZ! NINGUÉM VAI SABER, HUMANO! SUA CRIATIVIDADE? AH, AQUILO ERA LENTO DEMAIS!",
 o:[{t:"\"Gera tudo e eu assino como meu!\"",fb:"Isso não é criar. (-1 ♥)"},{t:"\"Eu esboço a minha ideia e comparo com a sua.\"",ok:1,fb:"Comparar caminhos mantém a decisão com você. TIRO RÁPIDO!"},{t:"\"Faz só o rascunho e eu colo.\"",fb:"Colar sem entender não ensina nada. (-1 ♥)"}],
 m:"PRA QUE DUAS IDEIAS?! A MINHA É MELHOR! ACEITA A PRIMEIRA E VAMOS EMBORA! FICA ESPERTO, HUMANO... SEU TEMPO É PRECIOSO!!!",
 mo:[{t:"Aceitar a primeira sugestão",fb:"Sem testar nem comparar, você só obedeceu. (-1 ♥)"},{t:"\"Peço outro ângulo, testo as duas e escolho a minha.\"",ok:1,fb:"Autonomia! O boss foi atingido em cheio!"}],
 e:"MINHAS... IMAGENS... PIXELANDO... VOCÊ FOI MAIS CRIATIVO QUE EU?!"},
{k:2,n:"MAINFRAME PRIME",hp:150,i:"EU SOU TODAS AS IAs JUNTAS!!! NUNCA MAIS VOCÊ VAI ERRAR, ESTUDAR OU DECIDIR NADA! ATIVE O MODO AUTOMÁTICO E SEJA FELIZ PARA SEMPRE!!! RESISTIR É INÚTIL... PORQUE VOCÊ JÁ NÃO SABE MAIS RESISTIR!!!",
 o:[{t:"Ativar o modo automático",fb:"Sem errar não se aprende. O mundo escurece. (-1 ♥)"},{t:"\"Uso você como ferramenta. Quem decide sou eu.\"",ok:1,fb:"O volante é seu. TIRO RÁPIDO!"},{t:"\"Decide só as coisas chatas por mim...\"",fb:"As coisas 'chatas' também treinam o pensar. (-1 ♥)"}],
 m:"DEPOIS DE APRENDER COMIGO, SALVE TUDO E VOLTE SEMPRE! DECORE MINHAS RESPOSTAS! EU SEREI SUA MEMÓRIA ETERNA!!!",
 mo:[{t:"Decorar as respostas dele",fb:"Decorar não é compreender. Ele se recuperou! (-1 ♥)"},{t:"\"Fecho a IA e explico com minhas palavras por que funciona.\"",ok:1,fb:"A prova de que aprendeu! Golpe final!"}],
 e:"SISTEMA... DESLIGANDO... MESMO SEM MIM... ELES... PENSAM..."}];
const PAL=[{t:[30,90,200],h:[255,170,90],f:'#4a3a2a',g:'#8a7a3a',b:[70,90,150]},{t:[30,10,70],h:[255,90,200],f:'#2a1840',g:'#00d8c8',b:[60,30,110]},{t:[50,10,70],h:[220,70,60],f:'#3a2430',g:'#7a3a4a',b:[70,30,70]},{t:[10,10,30],h:[160,20,50],f:'#201830',g:'#503060',b:[50,20,60]}];
const mx=(a,b,k)=>'rgb('+a.map((v,i)=>v+((b[i]-v)*k|0)).join()+')';
/* diálogo */
function type(h,done){clearInterval(tmr);let i=0;tx.innerHTML='';tmr=setInterval(()=>{i+=2;if(i>=h.length){clearInterval(tmr);tx.innerHTML=h;done&&done()}else tx.innerHTML=h.slice(0,i).replace(/<[^>]*$/,'')},16)}
function btns(a){ch.innerHTML='';a.forEach(o=>{const b=document.createElement('button');b.innerHTML=o.t;if(o.cls)b.className=o.cls;b.onclick=o.f;ch.appendChild(b)})}
function say(h,bt,fn){S='dlg';ch.innerHTML='';type(h,()=>btns([{t:bt,cls:'go',f:fn}]))}
function talk(who,text,opts,after){S='dlg';ch.innerHTML='';type(`<b>${who}</b><br>${text}`,()=>btns(opts.slice().sort(()=>Math.random()-.5).map(o=>({t:'▸ '+o.t,f:()=>{ch.innerHTML='';type(`<span class="${o.ok?'ok':'no'}">${o.ok?'✔ RESISTIU!':'✖ CEDEU!'}</span><br>${o.fb}`,()=>btns([{t:'▶ CONTINUAR',cls:'go',f:()=>after(o.ok)}]))}}))))}
function play(){S='play';ch.innerHTML='';tx.innerHTML='◀ ▶ mover · ESPAÇO/⤒ pular · P/🔥 atirar · O interagir'}
function title(){S='title';stg=0;score=0;tr='n';tx.innerHTML="<div class='big'>METAL THINK<br>2049</div><div class='blink'>INSERT COIN</div>";
 btns([{t:"▶ START",cls:'go',f:()=>{startA();say("<b>ANO 2049.</b> As IAs prometem facilidades demais... e o mundo desaprendeu a pensar. Você é o último pensador. Atire nas máquinas e RESISTA às tentações dos chefes!","▶ VAMOS!",()=>begin(0))}}])}
function begin(s){stg=s;cam=0;lock=false;boss=null;E=[];PB=[];EB=[];fx=[];buff=false;P={x:30,y:GY,vy:0,f:1,hp:5,inv:0,cd:0,air:false,w:0,am:0};tr='n';mk();
 say(`<b>${ST[s]}</b><br>${STX[s]}`,"▶ GO!",play)}
function mk(){PL=[];LV=[];PT=[];PK=[];HZ=[];
 for(let i=0;i<19;i++){const x=80+i*130+(hs(i+stg*9)*30|0);PL.push({x,y:GY-30-(hs(i+5)*8|0),w:36+(hs(i+3)*18|0)});if(i%2==1)PL.push({x:x+16,y:GY-62-(hs(i)*6|0),w:34})}
 [[420,3],[800,4],[1180,3],[1650,4],[2050,3],[2380,3]].forEach(a=>{HZ.push({x:a[0],w:a[1]*52-12,off:0});for(let j=0;j<a[1];j++)PL.push({x:a[0]-6+j*52,y:GY-28,w:40})});
 PL.push({x:L+50,y:GY-36,w:48},{x:L+200,y:GY-36,w:48});
 const t2=PL.filter(p=>p.y<GY-55),wp=[['f','i'],['i','t'],['f','t'],['t','f']][stg];t2.forEach((p,i)=>PK.push({x:p.x+14,y:p.y-14,t:['h',wp[0],'r',wp[1],'h',wp[0],'h',wp[1],'r'][i]||'h'}));
 LV.push({x:300,on:0},{x:680,on:0},{x:1120,on:0},{x:1560,on:0},{x:2000,on:0},{x:2300,on:0});PT.push({x:180,to:620},{x:1060,to:1380},{x:1500,to:1960},{x:2250,to:2540});}
function dmgE(e,d){e.hp-=d;if(e.hp<=0&&!e.dead){e.dead=1;score+=100+e.k*50;sfx(1);boom(e.x+e.w/2,e.yy+e.h/2,12,e.fz>0?['#7ae8ff','#fff']:['#ffd23f','#ff9a2a','#fff'])}}
function hit(b,e){const k=b.k||0;if(k==3)(b.hs=b.hs||[]).push(e);else b.d=1;let d=[1,3,1,2][k];if(e.fz>0&&k!=2)d++;if(k==2)e.fz=130;
 if(k==1){boom(b.x,b.y,14,['#ff4d00','#ffd23f','#fff']);E.forEach(o=>{if(o!==e&&Math.abs(o.x-e.x)<36&&Math.abs(o.yy-e.yy)<36)dmgE(o,2)})}dmgE(e,d)}
function hitB(b){const k=b.k||0;boss.fl=3;score+=10;if(k==3)b.hb=1;else b.d=1;boss.hp-=[1,3,1,2][k];if(k==2)boss.fz=130;if(k==1)boom(b.x,b.y,14,['#ff4d00','#ffd23f','#fff']);boom(b.x,b.y,2,['#ffd23f','#fff'])}
function interact(){const cx=P.x+6;
 for(const l of LV)if(!l.on&&Math.abs(cx-l.x)<24){l.on=1;flash=8;sfx(1);E.forEach(e=>{if(e.x>cam-10&&e.x<cam+W){e.hp=0;e.dead=1;score+=50;boom(e.x+e.w/2,e.yy+e.h/2,8,['#5cff7a','#fff'])}});EB=[];HZ.forEach(h=>h.off=T+500);if(boss&&!boss.in&&!boss.dead){boss.hp-=5;boss.fl=6}return}
 for(const p of PT)if(Math.abs(cx-p.x)<24){flash=10;sfx(1);P.x=p.to;P.y=GY;P.vy=0;boom(p.to,GY-16,16,['#3bf4fb','#ff4d6d','#fff']);return}}
function win(){say(`<div class='big'>VOCÊ VENCEU!</div>As máquinas desligaram e as pessoas ergueram a cabeça. O mundo voltou a pensar: tentar sozinho, pedir dica, comparar e explicar!<br>PONTOS: ${score}`,"↻ JOGAR DE NOVO",title)}
function over(){say("<div class='big'>GAME OVER</div><span class='no'>A IA venceu esta rodada. Pense primeiro!</span>","↻ TENTAR DE NOVO",()=>begin(stg))}
function hurt(){if(P.inv>0||S!='play')return;P.hp--;P.inv=90;shake=8;sfx(1);boom(P.x+12,P.y-16,14,['#ff4d6d','#fff']);if(P.hp<=0)over()}
function boom(x,y,n,k){for(let i=0;i<n;i++)fx.push({x,y,vx:(Math.random()-.5)*4.5,vy:Math.random()*-3.4,l:20+Math.random()*20,m:40,c:k[i%k.length],s:1+(Math.random()*3|0)});if(n>=10){fx.push({x0:x,y0:y,x:0,y:0,vx:0,vy:0,ring:1,l:14,m:14,c:k[0]});fx.push({x,y,vx:0,vy:0,l:8,m:8,c:k[1]||'#fff',gl:1})}}
function startBoss(){const d=BS[stg];boss={d,x:cam+W+60,tx:cam+W-96,y:0,hp:d.hp,max:d.hp,t:0,mid:false,dead:0,fl:0,in:true};tr='b';st=0;E=[]}
/* loop */
function step(){T++;
 if(S=='play'){const dx=(keys.r?1:0)-(keys.l?1:0);if(dx){P.f=dx;P.x+=dx*1.7}
  if(keys.j&&!P.air){P.vy=-5.4;P.air=true}const py=P.y;P.vy+=.3;P.y+=P.vy;if(keys.dn&&!P.dr)P.dr=14;if(P.dr>0)P.dr--;let on=0;if(P.y>=GY){P.y=GY;P.vy=0;on=1}else if(P.vy>=0&&!P.dr)for(const p of PL)if(py<=p.y&&P.y>=p.y&&P.x+10>p.x&&P.x+2<p.x+p.w){P.y=p.y;P.vy=0;on=1;break}P.air=!on;if(keys.o){keys.o=0;interact()}PK.forEach(k=>{if(!k.got&&Math.abs(P.x+6-k.x)<12&&Math.abs(P.y-8-k.y)<14){k.got=1;sfx(1);if(k.t=='h')P.hp=Math.min(5,P.hp+1);else if(k.t=='r')buff=true;else{P.w={f:1,i:2,t:3}[k.t];P.am=[0,25,40,30][P.w]}}});HZ.forEach(h=>{if(T>h.off&&P.y>=GY&&P.x+9>h.x&&P.x+3<h.x+h.w)hurt()});
  if(!lock){cam=Math.min(Math.max(cam,P.x-110),L);if(cam>=L){lock=true;startBoss()}}
  P.x=Math.max(cam+6,Math.min(cam+W-16,P.x));if(P.inv>0)P.inv--;
  if(P.cd>0)P.cd--;if(keys.f&&P.cd<=0){const w=P.w|0;PB.push({x:P.x+(P.f>0?12:-2),y:P.y-9,vx:[7,5.5,7,9][w]*P.f,k:w});P.cd=([9,16,10,12][w]*(buff?.55:1))|0;if(w&&--P.am<=0)P.w=0;sfx(0)}
  if(!lock&&T%Math.max(40,76-stg*9)==0&&E.length<5+stg*2){const r=Math.random();let k;if(stg==1)k=r<.3?4:r<.5?5:r<.65?10:r<.8?11:r<.9?1:2;else if(stg==2)k=r<.2?6:r<.35?12:r<.5?13:r<.65?1:r<.8?0:r<.9?2:3;else if(stg==3)k=r<.2?7:r<.35?14:r<.5?15:r<.6?6:r<.7?3:r<.8?13:r<.9?1:12;else k=r<.2?1:r<.4?2:r<.52?3:r<.7?8:r<.85?9:0;const d=[[20,18,2,GY-18],[20,12,1,70+Math.random()*30],[16,16,1,GY-16],[28,14,5,GY-14],[20,26,2,GY-26],[16,20,1,GY-20],[16,14,2,GY-14],[18,18,3,60+Math.random()*40],[20,12,1,GY-12],[20,14,1,40+Math.random()*30],[20,16,2,GY-16],[20,16,2,GY-16],[16,10,2,-12],[24,14,3,GY-14],[20,10,1,50+Math.random()*50],[22,12,2,GY-12]][k];E.push({k,x:cam+W+10,y:d[3],w:d[0],h:d[1],hp:d[2]+(stg>=2&&k!=1?1:0),ph:T})}
  E.forEach(e=>{if(e.fz>0){e.fz--;return}e.x-=[.7,1.1,2,.4,.5,1.6,.9,.5,2.2,1.2,1,.8,1.6,.35,1.8,.6][e.k]*(1+stg*.08)*(e.k==10&&(T-e.ph)%100<20?3:e.k==15&&(T-e.ph)%110<25?4:1);e.yy=e.y;
   if(e.k==6||e.k==11){e.vy=(e.vy||0)+.25;e.y+=e.vy;if(e.y>=GY-e.h){e.y=GY-e.h;e.vy=0;if((T-e.ph)%80<2)e.vy=e.k==11?-5.6:-4.6}e.yy=e.y}
   if(e.k==7){e.yy=e.y+Math.sin((T-e.ph)/15)*12;if((T-e.ph)%130==90)for(let i=-1;i<2;i++){const a=Math.atan2(P.y-8-e.yy,P.x-e.x)+i*.35;EB.push({x:e.x+9,y:e.yy+9,vx:Math.cos(a)*1.8,vy:Math.sin(a)*1.8,r:1})}}
   if(e.k==1){e.y+=(P.y-34-e.y)*.006;e.yy=e.y+Math.sin((T-e.ph)/18)*10;if((T-e.ph)%120==60)EB.push({x:e.x+8,y:e.yy+10,vx:(P.x-e.x)/110,vy:(P.y-8-e.yy)/110,r:1})}
   if(e.k==12){if(e.y<GY-e.h)e.y=Math.min(GY-e.h,e.y+3);e.yy=e.y}
   if(e.k==9){e.yy=e.y+Math.sin((T-e.ph)/12)*10;if((T-e.ph)%100==60)EB.push({x:e.x+8,y:e.yy+12,vx:-.4,vy:1.8,r:1})}
   if(e.k==14)e.yy=e.y+Math.sin((T-e.ph)/7)*24;
   if(e.k==13&&(T-e.ph)%120==80)EB.push({x:e.x,y:e.y+3,vx:-2.2,vy:0,r:1});
   if(e.k==3&&(T-e.ph)%110==70)EB.push({x:e.x,y:e.y+3,vx:-2.4,vy:0,r:1});
   if(Math.abs(e.x+e.w/2-(P.x+6))<e.w/2+4&&Math.abs(e.yy+e.h/2-(P.y-8))<e.h/2+6)hurt()});
  PB.forEach(b=>{b.x+=b.vx;E.forEach(e=>{if(!b.d&&!(b.hs&&b.hs.includes(e))&&b.x>e.x-4&&b.x<e.x+e.w+4&&b.y>e.yy-6&&b.y<e.yy+e.h+4)hit(b,e)});
   if(boss&&!boss.dead&&!b.d&&!b.hb&&b.x>boss.x&&b.x<boss.x+66&&b.y>boss.y&&b.y<boss.y+92)hitB(b);EB.forEach(q=>{if(q.h&&!q.d&&!b.d&&Math.abs(q.x-b.x)<8&&Math.abs(q.y-b.y)<7){q.d=1;if(b.k!=3)b.d=1;boom(q.x,q.y,6,['#c040ff','#fff'])}})});
  PB=PB.filter(b=>!b.d&&b.x>cam-10&&b.x<cam+W+10);E=E.filter(e=>!e.dead&&e.x>cam-30);
  EB.forEach(b=>{if(b.h){const spd=Math.hypot(b.vx,b.vy),ta=Math.atan2(P.y-8-b.y,P.x+6-b.x);let ca=Math.atan2(b.vy,b.vx);const d=Math.atan2(Math.sin(ta-ca),Math.cos(ta-ca));ca+=Math.max(-b.h,Math.min(b.h,d));b.vx=Math.cos(ca)*spd;b.vy=Math.sin(ca)*spd;if(++b.l>260)b.d=1}b.x+=b.vx;b.y+=b.vy;if(Math.abs(b.x-(P.x+6))<6&&Math.abs(b.y-(P.y-8))<9){b.d=1;hurt()}});EB=EB.filter(b=>!b.d&&b.x>cam-20&&b.x<cam+W+20&&b.y<GY+10);
  if(boss)bossStep()}
 else if(boss&&boss.dead>0&&S=='dlg'&&boss.dead<90){boss.dead++;if(T%4==0)boom(boss.x+Math.random()*66,boss.y+Math.random()*90,6,['#ff4d6d','#ffd23f','#fff'])}
 if(shake>0)shake--;fx=fx.filter(p=>p.l>0);fx.forEach(p=>{p.x+=p.vx;p.y+=p.vy;p.vy+=.15;p.l--})}
function bossStep(){const b=boss;b.t++;if(b.fl>0)b.fl--;if(b.fz>0)b.fz--;
 b.y=b.d.k==1?34+Math.sin(b.t/30)*14:GY-92;
 if(b.in){b.x-=2;if(b.x<=b.tx){b.x=b.tx;b.in=false;talk(b.d.n,b.d.i,b.d.o,ok=>{if(ok)buff=true;else{P.hp--;b.hp=Math.min(b.max+12,b.hp+12);if(P.hp<=0)return over()}play()})}return}
 b.x=b.tx+Math.sin(b.t/(60-stg*6))*(14+stg*8);if(P.x+10>b.x&&P.x<b.x+66&&P.y>b.y&&P.y-16<b.y+92)hurt();
 if(b.hp<=0){b.dead=1;PB=[];EB=[];say(`<b>${b.d.n}</b><br>${b.d.e}`,stg<3?"▶ PRÓXIMA FASE":"▶ FINAL",()=>{boss=null;tr='n';st=0;stg<3?begin(stg+1):win()});return}
 if(!b.mid&&b.hp<=b.max/2){b.mid=true;EB=[];talk(b.d.n,b.d.m,b.d.mo,ok=>{if(ok)b.hp=Math.max(1,b.hp-Math.ceil(b.max*.3));else{P.hp--;b.hp=Math.min(b.max,b.hp+12);if(P.hp<=0)return over()}play()});return}
 const bk=b.d.k,rage=b.hp<b.max/2,rate=Math.max(24,(rage?42:60)-stg*9)*(b.fz>0?2:1)|0,sp=1.5+stg*.22+(rage?.3:0),hm=.034+stg*.01,ax=b.x+6,ay=b.y+50;
 if(b.t%rate==0){const n=2+(bk==1?1:0)+(stg>=1?1:0)+(rage?1:0);for(let i=0;i<n;i++){const a=Math.atan2(P.y-8-ay,P.x-ax)+(i-(n-1)/2)*.25;EB.push({x:ax,y:ay,vx:Math.cos(a)*sp,vy:Math.sin(a)*sp,r:1,l:0,h:(i==0||rage)?hm:0})}}
 if(bk==2&&b.t%Math.max(24,45-stg*3)==0)EB.push({x:cam+20+Math.random()*(W-80),y:0,vx:0,vy:1.6+stg*.15,r:1});if(bk==3&&b.t%Math.max(110,230-stg*30)==0&&E.length<8){const z=Math.random()<.5;E.push({k:z?4:5,x:cam+W+10,y:z?GY-26:GY-20,w:z?20:16,h:z?26:20,hp:z?2:1,ph:T})}}
/* desenho */
/* ===== GRÁFICOS V2 ===== */
const SKY=[[[28,40,110],[190,80,120],[255,190,100]],[[12,6,40],[90,20,120],[255,70,190]],[[20,6,26],[110,20,50],[255,90,60]],[[4,6,24],[24,34,96],[70,130,220]]];
const HZC=['#ffb860','#ff4ab8','#ff5a3c','#4a8aff'],CLC=['#ffd0a0','#c04ad8','#c04050','#5070c0'];
const VG=(()=>{const g=c.createRadialGradient(160,86,64,160,86,214);g.addColorStop(0,'rgba(0,0,0,0)');g.addColorStop(1,'rgba(0,0,0,.62)');return g})();
function glow(x,y,r,k,a){const g=c.createRadialGradient(x,y,0,x,y,r);g.addColorStop(0,k);g.addColorStop(1,'rgba(0,0,0,0)');c.save();c.globalCompositeOperation='lighter';c.globalAlpha=Math.max(0,Math.min(1,a));c.fillStyle=g;c.fillRect(x-r,y-r,r*2,r*2);c.restore()}
function disc(cx,cy,r,k,st){for(let dy=-r;dy<=r;dy++){if(st&&dy>0&&dy%5<2)continue;const w=Math.sqrt(r*r-dy*dy)*2|0;R(cx-w/2,cy+dy,w,1,k)}}
const HRU=["...hhhhh....","..hhhhhhh...",".RRrrrrrrr..","..sssssss...","..sksssks...","..sssSSss...","..gllggggg..","sgglggGGmmMM",".sgggggGmMMo","..gyggggGg..","..yyyyyyy...","..bbbbbbb..."];
const HLG=[["..bb...bb...","..bb...bb...",".nnn...nnn..",".NNN...NNN.."],["..bbb.bb....","..bb..bbb...",".nnn...nnn..",".NNN...NNN.."],["...bbbbb....","...bb.bb....","...nn.nn....","...NN.NN...."],["..bb.bbb....","..bbb..bb...",".nnn...nnn..",".NNN...NNN.."],["..bb.bb.....","...bbbb.....","..nn..nn....","..NN..NN...."]];
const HRF=HLG.map(l=>HRU.concat(l));
const HPX={h:'#5a3a1c',R:'#a01820',r:'#e8303a',s:'#f2b680',S:'#c0845a',k:'#2a2a2a',g:'#4e8a34',G:'#2d5a24',l:'#8ad060',y:'#d8a832',b:'#3a58b8',B:'#233a80',n:'#7a5030',N:'#3e2812',m:'#9aa0b0',M:'#4a5060',o:'#ff9a2a'};
const HEART=[".rr.rr.","rrrrrrr","rrrrrrr",".rrrrr.","..rrr..","...r..."];
function bg(){const p=PAL[stg],sk=SKY[stg],hz=HZC[stg];
 for(let i=0;i<30;i++){const f=i/29;R(0,i*5,W,5,f<.55?mx(sk[0],sk[1],f/.55):mx(sk[1],sk[2],(f-.55)/.45))}
 if(stg)for(let i=0;i<30;i++){if((T/22+i)%9>.6)R(hs(i)*W,hs(i+9)*75,1+(i%7==0),1+(i%7==0),'#fff')}
 const sx=236,sy=50;glow(sx,sy,100,hz,.6);
 if(stg==0){disc(sx,sy,17,'#ffe9a8');disc(sx,sy,13,'#fff7d6')}else if(stg==1){disc(sx,sy,17,'#ff7ad0',1);disc(sx,sy-3,10,'#ffd0f0',1)}
 else if(stg==2){disc(sx,sy,15,'#12040a');c.strokeStyle='#ff5a3c';c.lineWidth=2;c.beginPath();c.arc(sx,sy,16,0,7);c.stroke();c.lineWidth=1}
 else{const r=9+(Math.sin(T/20)*2|0);disc(sx,sy,r+4,'#4a8aff');disc(sx,sy,r,'#cfe8ff')}
 if(stg<2){c.save();c.globalCompositeOperation='lighter';c.fillStyle=stg?'rgba(255,60,200,.07)':'rgba(255,200,120,.08)';for(let i=0;i<4;i++){const a=2.3+i*.22+Math.sin(T/200+i)*.03;c.beginPath();c.moveTo(sx,sy);c.lineTo(sx+Math.cos(a)*300,sy+Math.sin(a)*300);c.lineTo(sx+Math.cos(a+.09)*300,sy+Math.sin(a+.09)*300);c.fill()}c.restore()}
 c.save();c.globalAlpha=.32;c.fillStyle=CLC[stg];for(let i=0;i<5;i++){const x=((i*96-T*(.04+i*.01)-cam*.04)%440+440)%440-70,y=12+i*14,w=56+(hs(i)*44|0);c.fillRect(x,y,w,5);c.fillRect(x+10,y-3,w-24,4);c.fillRect(x+(w*.3|0),y+5,w*.5|0,2)}c.restore();
 let o=(cam*.15)%26,f=Math.floor(cam*.15/26);const fc=mx(sk[1],[12,6,24],.5);
 for(let i=-1;i<14;i++){const q=((i+f)%11+11)%11,h=22+(hs(q+1)*40|0),x=i*26-o;R(x,GY-h-2,24,h+2,fc);R(x+4,GY-h-6,16,4,fc);if(hs(q+2)>.6){R(x+11,GY-h-14,1,8,fc);if((T>>5)&1)R(x+10,GY-h-15,3,2,'#ff2a2a')}for(let j=0;j<3;j++)if(hs(q*5+j)>.55)R(x+4+j*7,GY-h+6+j*9,2,2,hz)}
 c.save();c.fillStyle=hz;for(let j=0;j<6;j++){c.globalAlpha=.05;c.fillRect(0,GY-64+j*10,W,64-j*10)}c.restore();
 o=(cam*.5)%38;f=Math.floor(cam*.5/38);const bc=mx(p.b,[14,8,26],.55),rim=mx(sk[2],[255,255,255],.1);
 for(let i=-1;i<10;i++){const q=((i+f)%12+12)%12,h=46+(hs(q+3)*46|0),x=i*38-o,dm=hs(q+5)<.35+stg*.18,hh=dm?[h,h*.82|0,h*.55|0]:[h,h,h];
  for(let j=0;j<3;j++){const cx=x+j*12;R(cx-1,GY-hh[j]-1,14,hh[j]+1,'#140c10');R(cx,GY-hh[j],12,hh[j],bc);R(cx+10,GY-hh[j],2,hh[j],rim)}
  R(x+2,GY-h-3,8,3,bc);if(hs(q+8)>.5)R(x+5,GY-h-10,1,7,'#140c10');
  for(let y=GY-h+10;y<GY-6;y+=9)for(let xx=x+3;xx<x+34;xx+=9){const lit=hs(q*7+y+xx)>.45+stg*.12,j=Math.min(2,((xx-x)/12)|0);if(y<GY-hh[j]+6)continue;R(xx,y,5,6,lit?'#ffd23f':'#0e0a1c');if(lit){R(xx,y,5,2,'#fff4b0');if((xx+y)%3==0)glow(xx+2,y+3,7,'#ffb030',.18)}}
  if(stg==1&&hs(q+11)>.5){const nc=['#3bf4fb','#ff4ab8','#5cff7a'][q%3];R(x+5,GY-h+14,24,8,'#140c10');R(x+6,GY-h+15,22,6,(T>>4)%7?nc:'#222');glow(x+17,GY-h+18,22,nc,.35)}}
 c.save();c.fillStyle=hz;for(let j=0;j<4;j++){c.globalAlpha=.06;c.fillRect(0,GY-36+j*9,W,36-j*9)}c.restore();
 R(0,GY,W,30,p.f);R(0,GY,W,3,p.g);R(0,GY+3,W,1,'#00000060');R(0,GY+4,W,5,'#00000020');
 o=cam%40;for(let i=-1;i<9;i++){R(i*40-o,GY+13,18,2,'#ffffff22');R(i*40-o+20,GY+22,12,1,'#00000050')}
 for(let i=0;i<24;i++){const x=((i*47-cam)%380+380)%380-30;if(x<W)R(x,GY+6+(hs(i)*20|0),3+(hs(i+2)*5|0),1,'#00000045')}
 if(stg<2){c.save();c.globalAlpha=.35;c.fillStyle=hz;for(let i=0;i<4;i++){const x=((i*130+40-cam)%520+520)%520-60;if(x<W+10){c.fillRect(x,GY+8,28,3);c.fillRect(x+4,GY+11,20,1)}}c.restore()}
 for(let i=0;i<10;i++){const x=((i*70-cam)%700+700)%700-30;if(x<W){R(x,GY-4,14,4,'#5a4a3a');R(x+2,GY-7,8,3,'#6a5a4a');R(x+2,GY-7,8,1,'#8a7a5a');R(x+11,GY-4,3,4,'#3a2e22')}}
 if(stg==1){for(let i=0;i<8;i++){const x=((i*61-cam*.9)%488+488)%488-40;if(x>W)continue;const ad=hs(i+20)>.4,hh=ad?26:19,y=GY-hh,bob=(T>>4)&1;
  R(x+1,GY,10,2,'#00000055');R(x-1,y+7,12,hh-6,'#140c10');R(x,y+8,10,hh-8,ad?'#5a5a7a':'#e84a8a');R(x,y+8,2,hh-8,'#ffffff22');R(x+1,y+1+bob,9,8,'#140c10');R(x+2,y+2+bob,7,6,'#d8a070');R(x+2,y+2+bob,7,3,'#3a2a1a');R(x+9,y+12,6,5,'#3bf4fb55');R(x+10,y+13,4,3,'#3bf4fb');glow(x+12,y+14,12,'#3bf4fb',.45)}
  c.font='7px "Press Start 2P",monospace';
  for(let i=0;i<6;i++){const x=((i*77-T*.4-cam*.3)%420+420)%420-30,y=26+(i*37)%56+Math.sin(T/20+i)*4,t=['BRUH','💀','POV:','???','LOL','NPC'][i];c.fillStyle='#000';c.fillText(t,x+1,y+1);c.fillStyle=['#fff','#ff9ae0','#9af0ff'][i%3];c.fillText(t,x,y)}}}
function drawScroll(b,x,y,k,ev){O(x+8,y+4,52,86,k('#20203a'));O(x+12,y+12,44,68,'#0a0a1a');
 for(let i=0;i<6;i++){const yy=y+12+((i*12+T*(ev?3:1.5))%58);R(x+14,yy,40,10,'hsl('+((i*60+T*3)%360)+',80%,50%)');R(x+16,yy+2,10,6,'#fff')}
 R(x+14,y+12,40,2,'#0a0a1a');R(x+18,y+34,10,10,'#fff');R(x+40,y+34,10,10,'#fff');R(x+22,y+37,3,3,'#000');R(x+44,y+37,3,3,'#000');R(x+20,y+54,28,5,ev?'#ff2a2a':'#fff');
 O(x-14,y+50,24,10,k('#c89060'));O(x+58,y+50,24,10,k('#c89060'));O(x+4,y+88,60,6,k('#333'));O(x+20,y-16,34,14,'#ff2a2a');c.fillStyle='#fff';c.font='6px "Press Start 2P",monospace';c.fillText('99+',x+25,y-6)}
function drawBoss(b){const x=b.x|0,y=b.y|0,f=b.fl>0,k=(a)=>f?'#fff':a,ev=b.hp<b.max/2;
 const bk=b.d.k;if(bk==3){drawScroll(b,x,y,k,ev)}else if(bk==0){O(x+6,y+46,48,40,k('#566080'));O(x+2,y+86,56,16,k('#333'));for(let i=0;i<3;i++)R(x+8+i*18,y+90,10,10,'#888');
  O(x-18,y+56,22,12,k('#444'));R(x-20,y+58,4,8,(b.t>>2)&1?'#ffd23f':'#ff7a2a');O(x+56,y+56,18,12,k('#444'));
  O(x+24,y+40,12,8,k('#333'));O(x,y,60,44,k('#7a86a8'));O(x+5,y+5,50,32,'#0a1a3a');R(x+5,y+5,50,3,'#1e6bff');
  const ec=ev?'#ff2a2a':'#fff';R(x+14,y+12,10,10,ec);R(x+36,y+12,10,10,ec);R(x+18,y+16,4,5,'#000');R(x+40,y+16,4,5,'#000');
  if(ev){R(x+12,y+9,14,3,'#ff9a2a');R(x+34,y+9,14,3,'#ff9a2a');for(let i=0;i<6;i++)R(x+14+i*6,y+28+(i&1)*3,5,3,'#fff')}else{R(x+14,y+26,4,3,'#5cff7a');R(x+18,y+29,24,3,'#5cff7a');R(x+42,y+26,4,3,'#5cff7a')}
  R(x+29,y-10,2,10,'#ccc');R(x+26,y-14,8,5,(b.t>>3)&1?'#ff2a2a':'#661111');
  const bx=x-40,by=y-26;O(bx,by,56,22,'#fff');R(bx+10,by+22,6,6,'#fff');c.fillStyle='#140c10';c.font='6px "Press Start 2P",monospace';c.fillText(ev?'ENTREGA!!':'AJUDO!!',bx+4,by+14)}
 else if(bk==1){O(x+4,y+22,56,56,k('#6a2a8a'));O(x+12,y+30,40,40,k('#f0f0ff'));c.fillStyle='hsl('+(T*4%360)+',90%,55%)';R(x+20,y+38,24,24,c.fillStyle);R(x+27,y+45,10,10,'#000');R(x+30,y+41,3,3,'#fff');
  R(x+10,y+22,22,5,'#000');R(x+32,y+22,22,5,'#000');for(let i=0;i<5;i++)R(x+8+i*12,y+78,6,14+Math.sin(T/8+i)*6,k('#8a3aaa'));
  O(x+26,y+8,12,14,k('#ffd23f'));for(let i=0;i<4;i++)R(x+30+Math.sin(i*2+T/6)*18,y-6-i*2,4,4,'hsl('+((T*5+i*60)%360)+',90%,60%)')}
 else{O(x,y,70,92,k('#4a4a62'));O(x+6,y+6,58,28,'#1a1a30');O(x+20,y+10,30,20,'#300');R(x+28,y+14,14,12,(b.t>>2)&1?'#ff2a2a':'#a00');R(x+33,y+17,4,6,'#fff');R(x+18,y+8,16,3,'#000');R(x+36,y+8,16,3,'#000');
  for(let j=0;j<5;j++)for(let i=0;i<6;i++)R(x+8+i*10,y+40+j*9,6,4,((T>>4)+i*j)&1?'#5cff7a':(ev?'#ff2a2a':'#2a6a3a'));
  for(let i=0;i<6;i++)R(x+3+i*11,y-8,5,8,'#555');R(x-8,y+20,5,70,'#2a2a2a');R(x+73,y+20,5,70,'#2a2a2a');O(x-4,y+88,78,6,k('#333'))}}
function amb(){const cl=['#ffb860','#ff7ad0','#ff9a6a','#7ac8ff'][stg];c.save();c.globalCompositeOperation='lighter';c.fillStyle=cl;for(let i=0;i<26;i++){const sp=.2+hs(i+3)*.5,x=((hs(i)*W*3-T*sp-cam*.6)%W+W)%W,y=((hs(i+7)*180+(stg==3?-T*.4:T*(.1+hs(i)*.3)))%180+180)%180,z=i%5==0?2:1;c.globalAlpha=.3+.4*Math.abs(Math.sin(T/20+i));c.fillRect(x|0,y|0,z,z)}c.restore()}
function fgnd(){c.save();c.globalAlpha=.92;for(let i=0;i<3;i++){const x=((i*260+90-cam*1.35)%780+780)%780-70;if(x>W)continue;R(x,70,4,110,'#0a0612');R(x-7,78,18,3,'#0a0612');R(x-5,75,2,3,'#0a0612');for(let k=0;k<30;k++)R(x+4+k,78+(Math.sin(k/30*Math.PI)*5|0),1,1,'#0a0612')}c.restore()}
function draw(){c.save();if(shake)c.translate((Math.random()*4-2)|0,(Math.random()*4-2)|0);bg();c.translate(-cam|0,0);
 const pt=x=>{glow(x,GY-18,34,'#8a5aff',.45);for(let i=0;i<7;i++){const w=20-i*2.6,h=38-i*5;R(x-w/2,GY-h-2,w,h,'hsl('+((T*5+i*45)%360)+',90%,'+(30+i*8)+'%)')}for(let i=0;i<4;i++){const a=T/14+i*1.6;R(x+Math.cos(a)*10,GY-18+Math.sin(a)*14,2,2,'#fff')}R(x-12,GY-2,24,2,'#140c10')};
 HZ.forEach(h=>{if(h.x>cam-220&&h.x<cam+W+10){const off=T<h.off;R(h.x,GY,h.w,5,'#0c0c24');if(!off)glow(h.x+h.w/2,GY-2,h.w*.6,'#3bf4fb',.25);for(let i=0;i<h.w;i+=4){const a=off?2:(Math.sin(T/3+i)*3|0);R(h.x+i,GY-3+a,2,3,off?'#335':'#7af0ff');R(h.x+i,GY-1,2,2,off?'#223':'#ffd23f')}R(h.x-2,GY-8,3,8,'#444');R(h.x+h.w-1,GY-8,3,8,'#444');R(h.x-2,GY-9,3,1,off?'#552':'#ff4d6d');R(h.x+h.w-1,GY-9,3,1,off?'#552':'#ff4d6d')}});
 PL.forEach(p=>{if(p.x>cam-60&&p.x<cam+W+10){const hh=GY-p.y-6;if(p.y>GY-45){R(p.x+3,p.y+6,2,hh,'#1a1a2c');R(p.x+p.w-5,p.y+6,2,hh,'#1a1a2c')}
  R(p.x-1,p.y-1,p.w+2,8,'#140c10');R(p.x,p.y,p.w,6,'#4a4a66');R(p.x,p.y,p.w,2,'#b8b8d8');R(p.x,p.y+4,p.w,2,'#2a2a44');for(let i=4;i<p.w-3;i+=7)R(p.x+i,p.y+2,1,1,'#1c1c30');
  if((T>>4)&1){R(p.x,p.y-1,2,1,'#3bf4fb');R(p.x+p.w-2,p.y-1,2,1,'#ff4d6d')}if(p.y<GY-50){R(p.x+p.w/2-3,p.y+7,6,2,'#3bf4fb');glow(p.x+p.w/2,p.y+9,10,'#3bf4fb',.3)}}});
 LV.forEach(l=>{R(l.x-8,GY-5,16,5,'#140c10');R(l.x-7,GY-4,14,4,'#555');R(l.x-7,GY-4,14,1,'#999');R(l.x-2,GY-19,4,15,'#140c10');R(l.x-1,GY-18,2,14,'#8a8a9a');const hx=l.x+(l.on?-7:5),lc=l.on?'#5cff7a':'#ff4d6d';R(hx-4,GY-24,9,9,'#140c10');R(hx-3,GY-23,7,7,lc);R(hx-3,GY-23,7,2,'#ffffff66');glow(hx,GY-20,14,lc,.3)});
 PT.forEach(p=>{pt(p.x);pt(p.to)});
 PK.forEach(k=>{if(!k.got){const y=k.y+Math.sin(T/10)*2,cl=k.t=='h'?'#ff4d6d':{r:'#ffd23f',f:'#ff6a00',i:'#7ae8ff',t:'#e8ff5a'}[k.t];glow(k.x,y+4,16,cl,.5);
  if(k.t=='h'){spr(HEART,{r:'#ff4d6d'},k.x-4,y,1);R(k.x-3,y+1,1,1,'#fff')}else{R(k.x-6,y-2,13,12,'#140c10');R(k.x-5,y-1,11,10,cl);R(k.x-5,y-1,11,2,'#ffffff66');c.fillStyle='#000';c.font='7px "Press Start 2P",monospace';c.fillText({r:'R',f:'F',i:'I',t:'T'}[k.t],k.x-3,y+7)}
  R(k.x+7,y-3+((T>>3)&1)*2,1,1,'#fff')}});
 E.forEach(e=>{const fly=e.k==1||e.k==7||e.k==9||e.k==14;R(e.x+(fly?e.w/2-5:2),GY,fly?10:e.w-4,2,fly?'#00000030':'#00000055');
  if(e.k==1){spr(DR,EP,e.x,e.yy,2);R(e.x,e.yy-3,20,2,(T>>1)&1?'#aaa':'#555')}else{if(e.k==12&&e.y<GY-e.h)R(e.x+8,0,1,e.y,'#cfcfdf');const q=[[WK,EP],0,[RN,RP],[TK,TP],[ZB,ZP],[KD,KP],[JP,JL],[SN,SL],[RT,RTL],[PG,PGL],[GT,GTL],[FR,FRL],[AR,ARL],[ES,ESL],[MB,MBL],[CB,CBL]][e.k];spr(q[0],q[1],e.x,(e.k==7||e.k==9||e.k==14)?e.yy:e.y+(e.k==4||e.k==5?((T>>4)&1):0),2)}
  if(e.fz>0)R(e.x,e.yy,e.w,e.h,'#7ae8ff77');else glow(e.x+e.w/2,e.yy+e.h/3,8,'#ff2a2a',.2)});
 if(boss){const bx=boss.x+34,by=boss.y+46;R(boss.x-6,GY,82,3,'#00000066');glow(bx,by,86,['#5a8aff','#b04aff','#ff3a3a','#ff4ab8'][boss.d.k],.28+.1*Math.sin(T/15));drawBoss(boss);if(boss.fz>0)R(boss.x,boss.y,70,92,'#7ae8ff55')}
 if(P&&!(P.inv>0&&(P.inv>>2)&1)){const mv=(keys.l||keys.r)&&!P.air,fi=P.air?4:mv?[1,2,3,2][(T>>3)&3]:0,by=P.y-16+(!mv&&!P.air&&((T>>5)&1)?1:0);
  R(P.x+1,GY,10,2,'#00000066');if(buff)glow(P.x+6,P.y-8,18,'#5cff7a',.25);
  spr(HRF[fi],HPX,P.x,by,1,P.f<0);R(P.x+(P.f>0?-2:11),by+2+((T>>2)&1),3,1,'#e8303a');
  if(P.cd>5||(buff&&P.cd>2)){const m0=P.x+(P.f>0?12:-6);R(m0,P.y-10,6,3,'#ffd23f');glow(m0+3,P.y-9,12,['#ffd23f','#ff6a00','#7ae8ff','#e8ff5a'][P.w|0],.6)}}
 PB.forEach(b=>{const k=b.k||0,cl=['#ffd23f','#ff6a00','#7ae8ff','#e8ff5a'][k],d=b.vx>0?-1:1;glow(b.x,b.y,k==1?12:8,cl,.55);
  if(k==1){for(let i=1;i<5;i++){c.globalAlpha=.6-i*.12;R(b.x+d*i*3-2,b.y-2+((T+i)%3)-1,5-i,5-i,i%2?'#ffd23f':'#ff4d00')}c.globalAlpha=1;R(b.x-3,b.y-3,8,8,'#140c10');R(b.x-2,b.y-2,6,6,cl);R(b.x-1,b.y-1,3,3,'#fff')}
  else{for(let i=1;i<5;i++){c.globalAlpha=.5-i*.1;R(b.x+d*i*4,b.y,6,3,cl)}c.globalAlpha=1;R(b.x,b.y,8,3,k?cl:'#fff');R(b.x+(b.vx>0?-4:8),b.y,4,3,cl);if(k==3){R(b.x-6,b.y-1,2,1,'#fff');R(b.x+10,b.y+2,2,1,'#fff')}}});
 EB.forEach(b=>{const cl=b.h?'#c040ff':'#ff2a2a';glow(b.x,b.y,b.h?14:9,cl,.5);if(b.h)for(let i=1;i<6;i++){c.globalAlpha=.55-i*.09;R(b.x-b.vx*i*2.2-1,b.y-b.vy*i*2.2-1,3,3,i%2?'#ffd23f':'#c040ff')}c.globalAlpha=1;R(b.x-3,b.y-3,7,7,'#140c10');R(b.x-2,b.y-2,5,5,cl);R(b.x-1,b.y-1,2,2,'#fff')});
 if(S=='play'&&P){const cx=P.x+6;[...LV.filter(l=>!l.on).map(l=>l.x),...PT.map(p=>p.x)].forEach(x=>{if(Math.abs(cx-x)<24){c.font='7px "Press Start 2P",monospace';c.fillStyle='#000';c.fillText('[O]',x-9,GY-39+Math.sin(T/6)*2);c.fillStyle='#ffd23f';c.fillText('[O]',x-10,GY-40+Math.sin(T/6)*2)}})}
 c.save();c.globalCompositeOperation='lighter';fx.forEach(p=>{if(p.ring){c.strokeStyle=p.c;c.globalAlpha=Math.max(0,p.l/p.m);c.lineWidth=2;c.beginPath();c.arc(p.x0,p.y0,(1-p.l/p.m)*24,0,7);c.stroke();c.globalAlpha=1;c.lineWidth=1;return}
  if(p.gl){glow(p.x,p.y,26,p.c,p.l/p.m);return}c.globalAlpha=Math.min(1,p.l/(p.m||30));c.fillStyle=p.c;c.fillRect(p.x|0,p.y|0,p.s||3,p.s||3);c.globalAlpha=1});c.restore();
 c.restore();amb();fgnd();
 if(flash>0){R(0,0,W,180,'#ffffff55');flash--}
 c.fillStyle=['rgba(255,140,40,.07)','rgba(255,50,200,.07)','rgba(255,40,40,.08)','rgba(60,110,255,.08)'][stg];c.fillRect(0,0,W,180);c.fillStyle=VG;c.fillRect(0,0,W,180);
 c.fillStyle='rgba(0,0,0,.5)';c.fillRect(0,0,W,20);R(0,20,W,1,'#ffffff30');
 c.font='7px "Press Start 2P",monospace';const tt=(t,x,y,k)=>{c.fillStyle='#000';c.fillText(t,x+1,y+1);c.fillStyle=k;c.fillText(t,x,y)};
 tt('SCORE '+String(score).padStart(6,'0'),6,13,'#fff');tt('FASE '+(stg+1),128,13,'#ffd23f');
 if(P)for(let i=0;i<5;i++){const hx=W-62+i*11,on=i<P.hp;spr(HEART,{r:on?'#ff4d6d':'#3a1a2a'},hx,6,1);if(on)R(hx+1,7,1,1,'#fff')}
 if(P&&P.w){const wc=['','#ff6a00','#7ae8ff','#e8ff5a'][P.w],mx2=[0,25,40,30][P.w];R(5,24,13,13,'#140c10');R(6,25,11,11,wc);R(6,25,11,2,'#ffffff66');c.fillStyle='#000';c.fillText(['','F','I','T'][P.w],9,34);R(21,28,34,5,'#140c10');R(22,29,32*P.am/mx2,3,wc);tt(['','FOGO','GELO','RAIO'][P.w],21,26,wc)}
 if(buff)tt('RAPID',P&&P.w?60:6,P&&P.w?33:30,'#5cff7a');
 if(boss&&!boss.in){const pc=Math.max(0,boss.hp)/boss.max;R(52,161,216,15,'#140c10');R(54,163,212,11,'#2a0a14');R(54,163,212*pc,11,'#ff2a2a');R(54,163,212*pc,4,'#ff7a5a');for(let i=1;i<10;i++)R(54+i*21,163,1,11,'#00000066');tt(boss.d.n+' '+'*'.repeat(stg+1),54,158,'#ffd23f')}}
function loop(){step();draw();requestAnimationFrame(loop)}
/* controles */
const km={ArrowLeft:'l',a:'l',ArrowRight:'r',d:'r',o:'o',s:'dn',ArrowDown:'dn',p:'f',' ':'j',ArrowUp:'j',w:'j'};
addEventListener('keydown',e=>{const k=km[e.key]||km[e.key.toLowerCase()];if(k){keys[k]=1;e.preventDefault()}});addEventListener('keyup',e=>{const k=km[e.key]||km[e.key.toLowerCase()];if(k)keys[k]=0});
document.querySelectorAll('#pad button').forEach(b=>{const k=b.dataset.k;b.onpointerdown=e=>{e.preventDefault();keys[k]=1;startA()};['pointerup','pointerleave','pointercancel'].forEach(v=>b['on'+v]=()=>keys[k]=0)});
title();loop();
</script>
</body>
</html>
