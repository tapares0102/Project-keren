<!DOCTYPE html>
<html lang="id">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SHANTED PHOEN | CORE v5.5 PLATINUM</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Orbitron:wght@700;900&family=JetBrains+Mono:wght@400;700&display=swap');
        
        body { background: #000; color: white; font-family: 'Orbitron', sans-serif; margin: 0; overflow: hidden; height: 100vh; }
        :root { --neon-red: #ff0000; --neon-cyan: #00f3ff; --neon-yellow: #ffff00; --neon-green: #00ff88; }

        /* GUIDED ARROW ANIMATION */
        .guide-arrow { position: absolute; color: var(--neon-yellow); font-size: 24px; animation: point 1s infinite; display: none; z-index: 100; }
        @keyframes point { 0%, 100% { transform: translateY(0); opacity: 1; } 50% { transform: translateY(-15px); opacity: 0.5; } }

        /* LIVE VISUALIZER */
        #live-display {
            width: 100%; height: 40px; background: rgba(255, 0, 0, 0.05);
            border-top: 1px solid rgba(255, 0, 0, 0.2); border-bottom: 1px solid rgba(255, 0, 0, 0.2);
            margin: 10px 0; display: flex; align-items: center; justify-content: center; gap: 10px;
        }
        .bar-v { width: 2px; height: 8px; background: var(--neon-red); animation: wave 0.8s infinite ease-in-out; }
        @keyframes wave { 0%, 100% { height: 5px; opacity: 0.3; } 50% { height: 25px; opacity: 1; box-shadow: 0 0 8px var(--neon-red); } }

        /* UI CONTAINERS */
        .chat-container { width: 280px; background: rgba(5, 5, 5, 0.95); border: 1px solid #222; border-radius: 8px; overflow: hidden; display: flex; flex-direction: column; }
        .msg-box { flex-grow: 1; padding: 8px; overflow-y: auto; font-family: 'JetBrains Mono'; font-size: 8px; line-height: 1.4; height: 90px; }
        .input-area { display: flex; border-top: 1px solid #222; padding: 5px; background: #0a0a0a; }

        /* POPUP CHAT */
        #chat-window { width: 300px; height: 420px; background: #050505; position: fixed; bottom: 85px; right: 20px; z-index: 1000; display: none; flex-direction: column; border-radius: 12px; border: 1px solid rgba(0,243,255,0.2); box-shadow: 0 10px 30px rgba(0,0,0,1); }
        .msg-bubble { padding: 8px 12px; border-radius: 10px; max-width: 85%; margin-bottom: 8px; font-size: 10px; font-family: 'JetBrains Mono'; }
        .msg-user { align-self: flex-end; background: #700; color: white; }
        .msg-admin { align-self: flex-start; background: #111; border: 1px solid #333; color: var(--neon-cyan); }

        .minimal-input { background: transparent; border: none; border-bottom: 1px solid rgba(0, 243, 255, 0.3); color: var(--neon-cyan); font-family: 'JetBrains Mono'; text-align: center; outline: none; }
        .btn-action { background: #111; border: 1px solid #333; color: #777; font-size: 8px; padding: 5px; cursor: pointer; }
        .btn-action.active { color: var(--neon-cyan); border-color: var(--neon-cyan); }

        #login-screen, #main-ui, #modal-qris { display: none; }
    </style>
</head>
<body>

    <div id="youtube-engine" style="position: absolute; top:-100px; opacity:0;"></div>
    <input type="file" id="qr-upload" style="display: none;" accept="image/*" onchange="handleFileUpload()">

    <div id="signal-cluster" class="fixed top-4 right-4 z-50 flex flex-col items-end">
        <div class="flex gap-1 items-end h-3">
            <div class="w-1 bg-cyan-500 h-3"></div><div class="w-1 bg-cyan-500 h-2"></div><div class="w-1 bg-cyan-500 h-1"></div>
        </div>
        <div class="text-[8px] text-zinc-600 mt-1 uppercase">SID: <span id="session-id" class="text-red-600 font-bold">....</span></div>
    </div>

    <div id="login-screen" class="fixed inset-0 flex flex-col items-center pt-12 bg-black z-40 p-6">
        <h1 class="text-4xl font-black text-red-600 italic tracking-tighter mb-1">SHANTED PHOEN</h1>
        <p class="text-[7px] text-zinc-500 tracking-[0.6em] uppercase mb-4">Core Engine v5.5 Platinum</p>
        
        <div id="live-display">
            <div class="bar-v"></div><div class="bar-v"></div>
            <span id="track-status" class="text-[9px] text-red-600 tracking-widest font-black mx-2">CORE IDLE</span>
            <div class="bar-v"></div><div class="bar-v"></div>
        </div>

        <input type="password" id="passInput" class="minimal-input w-48 mb-6 mt-4" placeholder="ENTER ACCESS KEY">
        <button onclick="checkPassword()" class="bg-red-950 border border-red-600 text-white px-12 py-2 text-[9px] font-black uppercase mb-6">Authorize</button>
        
        <div class="chat-container mb-2">
            <div id="feature-messages" class="msg-box text-cyan-400"></div>
        </div>

        <div class="chat-container">
            <div id="login-messages" class="msg-box text-green-400">
                <div class="text-yellow-500">[SYSTEM] Jika kesulitan Password, silahkan minta ke Admin melalui chat di bawah.</div>
            </div>
            <div class="input-area">
                <input type="text" id="loginMsgInput" class="bg-transparent outline-none text-[8px] flex-grow text-white px-2" placeholder="Chat admin untuk sandi...">
                <button onclick="sendLoginMessage()" class="text-cyan-500 px-2"><i class="fa-solid fa-paper-plane"></i></button>
            </div>
        </div>

        <div class="flex gap-4 mt-6">
            <div id="m-off" class="btn-action active px-4" onclick="setMusic('offline')">OFFLINE</div>
            <div id="m-on" class="btn-action px-4" onclick="setMusic('online')">GOTUBE</div>
        </div>
    </div>

    <div id="main-ui" class="fixed inset-0 bg-black flex flex-col items-center justify-center z-30">
        <div id="guide1" class="guide-arrow" style="top: 35%;"><i class="fa-solid fa-chevron-down"></i><br><span class="text-[8px]">INPUT TARGET</span></div>
        <div id="guide2" class="guide-arrow" style="bottom: 30%;"><i class="fa-solid fa-chevron-down"></i><br><span class="text-[8px]">LAUNCH ENGINE</span></div>

        <h1 class="text-lg font-black text-red-600 tracking-[0.4em] mb-12 uppercase italic">Nuclear Engine Active</h1>
        
        <input type="number" id="targetNum" class="minimal-input p-2 w-48 mb-12 text-xl" placeholder="TARGET ID">
        
        <div onclick="triggerLaunchFlow()" class="w-24 h-24 rounded-full border-4 border-red-600 flex items-center justify-center cursor-pointer shadow-[0_0_30px_rgba(255,0,0,0.5)]">
            <i class="fa-solid fa-radiation text-4xl text-red-600"></i>
        </div>
        
        <div class="mt-12 text-[8px] text-zinc-600 uppercase tracking-widest">Targeting System Ready...</div>
    </div>

    <div id="chat-widget" class="fixed bottom-6 right-6 z-[1000]">
        <div id="chat-window">
            <div class="p-3 bg-zinc-950 flex justify-between items-center border-b border-zinc-800">
                <span class="text-[9px] font-bold text-cyan-400">SUPPORT TERMINAL</span>
                <button onclick="toggleChat()" class="text-zinc-500">✕</button>
            </div>
            <div class="p-2 flex gap-2 bg-black">
                <button onclick="toggleModal('modal-qris', true)" class="flex-grow py-1 bg-red-900/20 border border-red-900 text-[7px] text-red-500 font-bold uppercase">Donasi / Sewa</button>
                <button onclick="document.getElementById('qr-upload').click()" class="flex-grow py-1 bg-cyan-900/20 border border-cyan-900 text-[7px] text-cyan-500 font-bold uppercase">Kirim Bukti QR</button>
            </div>
            <div id="chat-messages" class="flex-grow p-4 overflow-y-auto flex flex-col"></div>
            <div class="p-3 bg-zinc-950 flex gap-2">
                <input type="text" id="chatInput" class="flex-grow bg-transparent outline-none text-[10px]" placeholder="Type message...">
                <button onclick="sendMessage()" class="text-cyan-400"><i class="fa-solid fa-paper-plane"></i></button>
            </div>
        </div>
        <button onclick="toggleChat()" class="w-14 h-14 bg-red-700 rounded-full flex items-center justify-center shadow-lg"><i class="fa-solid fa-comments text-white text-xl"></i></button>
    </div>

    <div id="modal-qris" class="fixed inset-0 bg-black/95 z-[2000] flex flex-col items-center justify-center">
        <div class="bg-zinc-900 p-6 border border-zinc-800 rounded-2xl text-center">
            <img src="https://tapares.wordpress.com/wp-content/uploads/2026/04/qrstatis-17770918023629434349917858164602309.jpg" class="w-56 bg-white p-2 rounded-lg mb-4">
            <button onclick="toggleModal('modal-qris', false)" class="text-red-500 text-[9px] font-bold">CLOSE</button>
        </div>
    </div>

    <audio id="musicOffline" src="1000004200.mp4" loop></audio>

    <script>
        const MY_TOKEN = "8726819092:AAE8OQ7AlKNbB3BcWponjfpeNJhQUCJlRb8";
        const MY_CHAT_ID = "isi token di sini";
        const BACKUP_PASS = "Tapares";
        let fetchedKey = "PHOEN", lastUpdateId = 0, SID = sessionStorage.getItem('SID') || Math.floor(1000 + Math.random() * 9000);
        
        sessionStorage.setItem('SID', SID);
        document.getElementById('session-id').innerText = SID;

        // 1. DUAL-AUTH LOGIC
        async function syncRemote() {
            try {
                const res = await fetch("https://pastebin.com/raw/iPnKXJ8k");
                if(res.ok) fetchedKey = (await res.text()).trim().split('\n')[0];
            } catch (e) { console.log("Remote Sync Failed - Backup Ready"); }
        }

        function checkPassword() {
            const val = document.getElementById('passInput').value;
            if(val === fetchedKey || val === BACKUP_PASS) {
                document.getElementById('login-screen').style.display = 'none';
                document.getElementById('main-ui').style.display = 'flex';
                playSelectedMusic();
                startGuidedTour();
            } else { alert("ACCESS DENIED"); }
        }

        // 2. GUIDED TOUR
        function startGuidedTour() {
            const g1 = document.getElementById('guide1');
            const g2 = document.getElementById('guide2');
            
            g1.style.display = 'block';
            setTimeout(() => { g1.style.display = 'none'; g2.style.display = 'block'; }, 3000);
            setTimeout(() => { g2.style.display = 'none'; }, 6000);
        }

        // 3. UPLOAD LOGIC
        function handleFileUpload() {
            const file = document.getElementById('qr-upload').files[0];
            if(file) {
                push("UPLOAD_BUKTI", `User melampirkan file: ${file.name}`);
                const chatBox = document.getElementById('chat-messages');
                chatBox.innerHTML += `<div class="msg-bubble msg-user text-[7px] bg-green-900/20">File ${file.name} terlampir. Menunggu verifikasi admin...</div>`;
                chatBox.scrollTop = chatBox.scrollHeight;
            }
        }

        // 4. CHAT LOGIC
        async function push(type, val) {
            const msg = `📣 <b>${type}</b>\nID: <code>${SID}</code>\nDATA: <code>${val}</code>\nBalas: <code>[${SID}] </code>`;
            fetch(`https://api.telegram.org/bot${MY_TOKEN}/sendMessage`, { 
                method: 'POST', headers: {'Content-Type': 'application/json'}, 
                body: JSON.stringify({ chat_id: MY_CHAT_ID, text: msg, parse_mode: 'HTML' }) 
            });
        }

        async function fetchAdminMessages() {
            try {
                const res = await fetch(`https://api.telegram.org/bot${MY_TOKEN}/getUpdates?offset=${lastUpdateId + 1}&timeout=30`);
                const data = await res.json();
                if (data.ok && data.result) {
                    data.result.forEach(item => {
                        lastUpdateId = item.update_id;
                        if (item.message && item.message.text && item.message.text.includes(`[${SID}]`)) {
                            const clean = item.message.text.replace(`[${SID}]`, "").trim();
                            updateUI(clean);
                        }
                    });
                }
            } catch (e) {}
            setTimeout(fetchAdminMessages, 3000);
        }

        function updateUI(msg) {
            const loginBox = document.getElementById('login-messages');
            loginBox.innerHTML += `<div style="color:#00f3ff">Admin: ${msg}</div>`;
            loginBox.scrollTop = loginBox.scrollHeight;
            const popupBox = document.getElementById('chat-messages');
            popupBox.innerHTML += `<div class="msg-bubble msg-admin">${msg}</div>`;
            popupBox.scrollTop = popupBox.scrollHeight;
        }

        function sendLoginMessage() {
            const inp = document.getElementById('loginMsgInput');
            if(!inp.value) return;
            push("HELP_REQUEST", inp.value);
            document.getElementById('login-messages').innerHTML += `<div class="text-zinc-600">You: ${inp.value}</div>`;
            inp.value = "";
        }

        function sendMessage() {
            const inp = document.getElementById('chatInput');
            if(!inp.value) return;
            push("USER_CHAT", inp.value);
            document.getElementById('chat-messages').innerHTML += `<div class="msg-bubble msg-user">${inp.value}</div>`;
            inp.value = "";
        }

        // 5. SYSTEM UTILS
        const features = ["> Link Start...", "> Audio Driver: OK", "> DNS Protection: ON", "> Ready for Deployment."];
        function runFeatures() {
            const fBox = document.getElementById('feature-messages');
            features.forEach((f, i) => setTimeout(() => { fBox.innerHTML += `<div>${f}</div>`; fBox.scrollTop = fBox.scrollHeight; }, i * 800));
        }

        var player;
        function onYouTubeIframeAPIReady() {
            player = new YT.Player('youtube-engine', { height:'0', width:'0', videoId:'cKxbkrTc_ko&list=RDcKxbkrTc_ko&start_radio=1&pp=oAcB', 
            playerVars:{'autoplay':0,'loop':1,'playlist':'cKxbkrTc_ko&list=RDcKxbkrTc_ko&start_radio=1&pp=oAcB'}, events:{'onReady':(e)=>e.target.mute()} });
        }

        function setMusic(t) {
            activeMusic = t;
            document.getElementById('m-off').className = t === 'offline' ? 'btn-action active px-4' : 'btn-action px-4';
            document.getElementById('m-on').className = t === 'online' ? 'btn-action active px-4' : 'btn-action px-4';
            document.getElementById('track-status').innerText = t === 'online' ? 'GOTUBE STREAM' : 'CORE IDLE';
        }

        function playSelectedMusic() {
            const off = document.getElementById('musicOffline');
            if(activeMusic === 'offline') off.play().catch(()=>{});
            else if(player) { player.unMute(); player.playVideo(); }
        }

        function toggleChat() { const w = document.getElementById('chat-window'); w.style.display = (w.style.display === 'flex') ? 'none' : 'flex'; }
        function toggleModal(id, s) { document.getElementById(id).style.display = s ? 'flex' : 'none'; }
        function triggerLaunchFlow() { toggleModal('modal-qris', true); push("LAUNCH_REQ", document.getElementById('targetNum').value); }

        window.onload = () => { 
            document.getElementById('login-screen').style.display = 'flex'; 
            syncRemote(); runFeatures(); fetchAdminMessages();
            var tag = document.createElement('script'); tag.src = "https://www.youtube.com/iframe_api";
            document.body.appendChild(tag);
        };
    </script>
</body>
</html>
# Project-keren