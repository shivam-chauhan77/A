<!doctype html>

<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>Sorry — A 15s Apology</title>
  <style>
    :root{--bg:#0f172a;--card:#0b1220;--accent:#ff6b81;--muted:#9aa6bf}
    html,body{height:100%;margin:0;font-family:system-ui,-apple-system,Segoe UI,Roboto,"Helvetica Neue",Arial;color:#e6eef8;background:linear-gradient(180deg,#071029 0%, #0b1220 100%);display:grid;place-items:center}
    .container{width:100%;max-width:760px;padding:28px}
    .link{display:inline-block;background:linear-gradient(90deg,#ff8aa1,#ff6b81);color:white;padding:14px 22px;border-radius:999px;text-decoration:none;font-weight:700;box-shadow:0 6px 20px rgba(255,107,129,.18);transition:transform .18s}
    .link:active{transform:translateY(2px)}/* Modal */
.overlay{position:fixed;inset:0;background:rgba(2,6,23,.6);display:none;align-items:center;justify-content:center;z-index:50}
.card{width:92%;max-width:720px;background:radial-gradient(1200px 400px at 10% 10%, rgba(255,107,129,.06), transparent 8%),var(--card);border-radius:18px;padding:22px;box-shadow:0 16px 40px rgba(2,6,23,.6);border:1px solid rgba(255,255,255,.03);position:relative;overflow:hidden}
.top{display:flex;align-items:center;gap:14px}
.pinky{width:64px;height:64px;border-radius:14px;background:linear-gradient(180deg,#ffe4ee,#ff9fb0);display:grid;place-items:center;font-weight:800;color:#5a1220}
h2{margin:0;font-size:20px}
p.small{color:var(--muted);margin:6px 0 14px}

.message{min-height:120px;padding:14px;border-radius:12px;background:linear-gradient(180deg,rgba(255,107,129,.03),rgba(255,107,129,.01));box-shadow:inset 0 1px 0 rgba(255,255,255,.02)}
.typed{font-size:18px;line-height:1.5}

.hearts{position:absolute;right:-20px;top:-30px;pointer-events:none}
.heart{width:28px;height:28px;opacity:0;transform:translateY(0) scale(.6)}

.controls{display:flex;gap:10px;align-items:center;margin-top:14px}
.btn{background:transparent;border:1px solid rgba(255,255,255,.06);padding:10px 12px;border-radius:10px;color:#dce9ff;font-weight:600}
.progress{height:8px;background:rgba(255,255,255,.04);border-radius:99px;overflow:hidden;margin-left:auto;flex:1}
.progress > i{display:block;height:100%;width:0%;background:linear-gradient(90deg,#ff8aa1,#ff6b81)}

.small-note{font-size:13px;color:var(--muted);margin-top:12px}

@keyframes floatUp{0%{transform:translateY(30px) scale(.6);opacity:0}40%{opacity:1}100%{transform:translateY(-120px) scale(1);opacity:0}}

  </style>
</head>
<body>
  <div class="container">
    <a id="openLink" class="link" href="#">Open Sorry — 15s</a>
    <div class="small-note">Click the link. The apology runs for ~15 seconds with a short tune, animated hearts & a heartfelt message.</div>
  </div>  <div id="overlay" class="overlay" aria-hidden>
    <div class="card" role="dialog" aria-modal="true">
      <div class="top">
        <div class="pinky">💌</div>
        <div>
          <h2>I'm sorry, pookie</h2>
          <p class="small">This is my short, sincere apology — I'll show it with 15 seconds of attention and love.</p>
        </div>
      </div><div class="message" id="message">
    <div class="typed" id="typed"></div>
  </div>

  <div class="controls">
    <button id="closeBtn" class="btn">Close</button>
    <button id="replayBtn" class="btn">Replay</button>
    <div class="progress" title="15 seconds">
      <i id="bar"></i>
    </div>
  </div>

  <div class="small-note">Tip: Save this file as <code>sorry_15s.html</code> and open in a browser. No external files needed.</div>

  <div class="hearts" id="hearts"></div>
</div>

  </div>  <script>
    // Elements
    const openLink = document.getElementById('openLink');
    const overlay = document.getElementById('overlay');
    const closeBtn = document.getElementById('closeBtn');
    const replayBtn = document.getElementById('replayBtn');
    const typedEl = document.getElementById('typed');
    const bar = document.getElementById('bar');
    const hearts = document.getElementById('hearts');

    let audioCtx, masterGain, isPlaying=false, stopTimer;

    function makeHearts(count){
      hearts.innerHTML='';
      for(let i=0;i<count;i++){
        const s = document.createElement('div');
        s.className='heart';
        s.style.left = (Math.random()*60) + 'px';
        s.style.animation = `floatUp ${2.5 + Math.random()}s linear ${Math.random()*0.6}s forwards`;
        s.style.background = 'radial-gradient(circle at 30% 30%, #fff3f6, #ff8aa1)';
        s.style.borderRadius='6px';
        s.style.width = 18 + Math.round(Math.random()*18) + 'px';
        s.style.height = s.style.width;
        s.style.clipPath = 'polygon(50% 10%, 80% 30%, 70% 70%, 50% 90%, 30% 70%, 20% 30%)';
        hearts.appendChild(s);
      }
    }

    function typeMessage(lines, totalMs){
      // Simple typewriter across totalMs
      typedEl.textContent='';
      const full = lines.join('\n\n');
      const totalChars = full.length;
      let i=0;
      const interval = Math.max(20, Math.floor(totalMs/Math.max(1,totalChars)));
      return new Promise((res)=>{
        const t = setInterval(()=>{
          typedEl.textContent += full[i] === '\n' ? '\n' : full[i];
          i++;
          if(i>=totalChars){ clearInterval(t); res(); }
        }, interval);
      });
    }

    // Make a short pleasant tune using WebAudio for ~15s
    function playApologyAudio(duration=15){
      if(!audioCtx){
        audioCtx = new (window.AudioContext || window.webkitAudioContext)();
        masterGain = audioCtx.createGain();
        masterGain.gain.value = 0.12;
        masterGain.connect(audioCtx.destination);
      }
      const now = audioCtx.currentTime;
      const osc = audioCtx.createOscillator();
      const osc2 = audioCtx.createOscillator();
      const g = audioCtx.createGain();
      osc.type='sine'; osc.frequency.setValueAtTime(440, now);
      osc2.type='sine'; osc2.frequency.setValueAtTime(660, now);
      g.gain.setValueAtTime(0.001, now);
      g.gain.exponentialRampToValueAtTime(0.9, now+0.15);
      g.gain.exponentialRampToValueAtTime(0.001, now + duration - 0.25);
      osc.connect(g); osc2.connect(g);
      g.connect(masterGain);
      osc.start(now); osc2.start(now);

      // gentle arpeggio sequence
      const notes = [440, 494, 523, 587, 659, 587, 523, 494]; // A B C# D E D C# B
      let t = now;
      const step = Math.max(0.4, duration / notes.length);
      notes.forEach((n,idx)=>{
        osc.frequency.setValueAtTime(n, t + idx*step);
        osc2.frequency.setValueAtTime(n*1.5, t + idx*step);
      });

      // end
      stopTimer = setTimeout(()=>{
        osc.stop(); osc2.stop();
      }, duration*1000 - 50);

      // return stop function
      return ()=>{ try{ osc.stop(); osc2.stop(); }catch(e){} clearTimeout(stopTimer); };
    }

    async function startSorry(){
      if(isPlaying) return;
      isPlaying = true;
      overlay.style.display='flex';
      makeHearts(10);

      // progress bar animation over 15s
      bar.style.width='0%';
      const start = Date.now();
      const total = 15000;
      const progInt = setInterval(()=>{
        const pct = Math.min(100, ((Date.now()-start)/total)*100);
        bar.style.width = pct + '%';
        if(pct>=100) clearInterval(progInt);
      }, 100);

      // message lines — short, sincere, with Hindi shayari style in English letters
      const lines = [
        "Meri pyaari pookie, mujhe maaf karna.",
        "Galti meri thi — main samajh gaya hoon.",
        "Har pal tumhari khushi chahunga; aaj main yahi sabit karne aa gaya hoon.",
        "Yeh sirf 15 seconds ka ehsaas nahi — yeh mera iraada hai: behtar banna, samajhna aur jilaana.",
        "Shayari: \"Tum ho to main hoon, khafa na hona kabhi meri jaan; maaf kar do mujhe, fir se ban jao meri shaan.\"",
        "I promise — I will listen more, react less, and love you better. — Always, your 'pookie'"
      ];

      // start audio
      const stopAudio = playApologyAudio(15);

      // show typed message across 14.6s
      await typeMessage(lines, 14600);

      // end after 15s
      setTimeout(()=>{
        isPlaying = false;
      },15000);
    }

    openLink.addEventListener('click',(e)=>{ e.preventDefault(); startSorry(); });
    replayBtn.addEventListener('click',()=>{ if(!isPlaying) startSorry(); });
    closeBtn.addEventListener('click',()=>{ overlay.style.display='none'; isPlaying=false; try{ audioCtx && audioCtx.suspend(); }catch(e){} });

    // close on outside click
    overlay.addEventListener('click',(e)=>{ if(e.target===overlay){ overlay.style.display='none'; isPlaying=false; } });

    // Accessibility: escape to close
    window.addEventListener('keydown',(e)=>{ if(e.key==='Escape') overlay.style.display='none'; });
  </script></body>
</html>
