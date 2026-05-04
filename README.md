<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <meta name="theme-color" content="#3b82f6">
    <title>AI App Builder Pro</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Fira+Code:wght@400;500&family=Inter:wght@300;400;600;700&display=swap');
        
        :root {
            --editor-bg: #0d1117;
            --panel-bg: #161b22;
            --accent: #3b82f6;
            --text-main: #c9d1d9;
            --border: rgba(255, 255, 255, 0.1);
            --chat-bubble-ai: rgba(255, 255, 255, 0.07);
            --chat-text: #ffffff;
            --code-bg: #282c34;
            --code-text: #abb2bf;
            --input-text: #ffffff;
        }

        .light-mode {
            --editor-bg: #ffffff;
            --panel-bg: #f6f8fa;
            --accent: #0969da;
            --text-main: #24292f;
            --border: rgba(31, 35, 40, 0.1);
            --chat-bubble-ai: #ffffff;
            --chat-text: #24292f;
            --input-text: #000000;
        }

        body {
            font-family: 'Inter', sans-serif;
            background-color: var(--editor-bg);
            color: var(--text-main);
            height: 100vh;
            margin: 0;
            display: flex;
            flex-direction: column;
            overflow: hidden;
            transition: background 0.3s ease;
        }

        .editor-wrapper {
            position: relative;
            flex: 1;
            display: flex;
            overflow: hidden;
            background: var(--editor-bg);
            z-index: 10;
        }

        .line-numbers-sidebar {
            width: 50px;
            background: var(--panel-bg);
            border-right: 1px solid var(--border);
            padding-top: 1rem;
            color: #484f58;
            font-family: 'Fira Code', monospace;
            font-size: 14px;
            text-align: right;
            padding-right: 12px;
            user-select: none;
            overflow: hidden;
            line-height: 1.6;
        }

        .editor-scroll-container {
            position: relative;
            flex: 1;
            overflow: auto;
        }

        #highlighting {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            padding: 1rem;
            margin: 0;
            white-space: pre;
            word-wrap: normal;
            font-family: 'Fira Code', monospace;
            font-size: 14px;
            line-height: 1.6;
            pointer-events: none;
            color: var(--text-main);
            box-sizing: border-box;
            z-index: 1;
        }

        #mainEditor {
            position: absolute;
            top: 0;
            left: 0;
            width: 100%;
            min-height: 100%;
            padding: 1rem;
            margin: 0;
            white-space: pre;
            word-wrap: normal;
            font-family: 'Fira Code', monospace;
            font-size: 14px;
            line-height: 1.6;
            background: transparent;
            color: transparent; 
            caret-color: var(--accent);
            resize: none;
            outline: none;
            border: none;
            box-sizing: border-box;
            overflow: hidden;
            z-index: 2;
        }

        .token.keyword { color: #ff7b72; font-weight: bold; }
        .token.comment { color: #8b949e; font-style: italic; }

        .code-block-container {
            background: #282c34;
            border-radius: 8px;
            overflow: hidden;
            margin: 10px 0;
            border: 1px solid rgba(0,0,0,0.3);
        }
        .code-header {
            background: #21252b;
            padding: 6px 12px;
            display: flex;
            justify-content: space-between;
            align-items: center;
            border-bottom: 1px solid rgba(0,0,0,0.2);
        }
        .chat-code {
            padding: 12px;
            font-family: 'Fira Code', monospace;
            font-size: 12px;
            color: #abb2bf;
            overflow-x: auto;
            line-height: 1.5;
            background: #282c34;
        }

        .glass-header {
            background: var(--panel-bg);
            border-bottom: 1px solid var(--border);
            z-index: 100;
        }

        .prompt-area {
            background: var(--panel-bg);
            border-top: 1px solid var(--border);
            padding: 0.75rem 1rem 1rem 1rem;
            z-index: 100;
        }

        .ai-input-pill {
            background: var(--editor-bg);
            border: 1px solid var(--border);
            border-radius: 24px;
            padding: 0.75rem 3.5rem 0.75rem 3.5rem;
            width: 100%;
            color: var(--input-text);
            resize: none;
            max-height: 200px;
            min-height: 46px;
            line-height: 1.4;
            display: flex;
            align-items: center;
            overflow-y: auto;
            transition: border-color 0.2s;
        }
        .ai-input-pill:focus {
            border-color: var(--accent);
            outline: none;
        }

        #sidePanel {
            position: fixed;
            top: 0;
            right: -100%;
            width: 100%;
            height: 100%;
            background: var(--panel-bg);
            z-index: 2000;
            transition: right 0.3s cubic-bezier(0.4, 0, 0.2, 1);
            display: flex;
            flex-direction: column;
            border-left: 1px solid var(--border);
            box-shadow: -10px 0 30px rgba(0,0,0,0.5);
            color: var(--text-main);
        }
        #sidePanel.active { right: 0; }
        @media (min-width: 768px) { #sidePanel { width: 450px; } }

        .panel-tab {
            display: none;
            flex-direction: column;
            flex: 1;
            overflow: hidden;
        }
        .panel-tab.active { display: flex; }

        .msg-bubble {
            padding: 12px 16px;
            border-radius: 18px;
            max-width: 95%;
            font-size: 14px;
            margin-bottom: 12px;
            line-height: 1.6;
            animation: fadeIn 0.2s ease;
            position: relative;
        }
        
        .msg-ai { background: var(--chat-bubble-ai); border: 1px solid var(--border); align-self: flex-start; border-bottom-left-radius: 4px; color: var(--chat-text); }
        .msg-user { background: var(--accent); color: white; align-self: flex-end; border-bottom-right-radius: 4px; }
        
        .msg-content { white-space: pre-wrap; font-size: 13px; overflow-wrap: break-word; }

        .speak-btn-bubble {
            position: absolute;
            right: 8px;
            bottom: -22px;
            background: var(--panel-bg);
            border: 1px solid var(--border);
            color: #8b949e;
            width: 24px;
            height: 24px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 10px;
            cursor: pointer;
            transition: all 0.2s;
            z-index: 5;
        }
        .speak-btn-bubble:hover { color: var(--accent); border-color: var(--accent); }
        .msg-ai-container { position: relative; margin-bottom: 24px; align-self: flex-start; max-width: 90%; }

        .voice-control-btn {
            display: flex;
            align-items: center;
            gap: 8px;
            padding: 6px 12px;
            border-radius: 20px;
            background: rgba(255, 255, 255, 0.05);
            border: 1px solid var(--border);
            font-size: 10px;
            font-weight: bold;
            color: #8b949e;
            cursor: pointer;
            transition: all 0.2s;
            text-transform: uppercase;
        }
        .voice-control-btn.active { color: var(--accent); border-color: var(--accent); background: rgba(59, 130, 246, 0.1); }

        /* Modernized LIVE VIEW MODE for Canvas */
        #previewBox {
            position: fixed;
            inset: 0;
            background: #000;
            z-index: 9999;
            display: none;
            flex-direction: column;
        }
        #previewBox.active { display: flex; }
        .preview-header {
            height: 44px;
            background: #1a1a1a;
            display: flex;
            align-items: center;
            justify-content: space-between;
            padding: 0 16px;
            border-bottom: 1px solid rgba(255,255,255,0.1);
        }
        .preview-controls {
            display: flex;
            gap: 8px;
        }
        .dot { width: 12px; height: 12px; border-radius: 50%; }
        .bg-red { background: #ff5f56; }
        .bg-yellow { background: #ffbd2e; }
        .bg-green { background: #27c93f; }
        
        #previewIframe {
            flex: 1;
            width: 100%;
            border: none;
            background: white;
        }

        .apply-btn {
            background: #10b981;
            color: white;
            padding: 2px 8px;
            border-radius: 4px;
            font-size: 10px;
            font-weight: bold;
        }

        /* Image Preview Thumbnail */
        #imagePreviewContainer {
            position: absolute;
            left: 50px;
            bottom: 60px;
            display: none;
            background: var(--panel-bg);
            padding: 4px;
            border-radius: 8px;
            border: 1px solid var(--border);
            z-index: 101;
        }
        #imagePreviewContainer img {
            width: 60px;
            height: 60px;
            object-fit: cover;
            border-radius: 4px;
        }
        #imagePreviewContainer .close-img {
            position: absolute;
            top: -8px;
            right: -8px;
            background: #ef4444;
            color: white;
            width: 16px;
            height: 16px;
            border-radius: 50%;
            display: flex;
            align-items: center;
            justify-content: center;
            font-size: 10px;
            cursor: pointer;
        }
    </style>
</head>
<body class="dark-mode">

    <div id="toast" class="fixed bottom-4 right-4 bg-gray-800 border border-gray-700 text-white px-4 py-2 rounded-lg shadow-xl transform transition-all duration-300 translate-y-20 opacity-0 z-[10000] flex items-center gap-2">
        <i class="fas fa-info-circle text-blue-400"></i>
        <span id="toastMessage" class="text-sm"></span>
    </div>

    <header class="glass-header h-14 flex items-center justify-between px-4">
        <div class="flex items-center gap-3">
            <div class="w-8 h-8 bg-blue-600 rounded-lg flex items-center justify-center shadow-lg">
                <i class="fas fa-code text-white text-xs"></i>
            </div>
            <div class="hidden sm:block">
                <h1 class="text-xs font-bold uppercase tracking-tight">AI App Builder <span class="text-blue-500">Pro</span></h1>
                <p class="text-[9px] text-gray-500 font-medium tracking-widest uppercase">Sync Code Engine</p>
            </div>
        </div>

        <div class="flex items-center gap-2">
            <button onclick="toggleTheme()" class="w-8 h-8 rounded-full border border-gray-700 flex items-center justify-center text-xs text-gray-400 hover:text-white">
                <i id="themeIcon" class="fas fa-moon"></i>
            </button>
            <button onclick="runPreview()" class="px-3 py-1.5 bg-blue-600 hover:bg-blue-500 text-white text-[10px] font-bold rounded-full flex items-center gap-2 transition-all active:scale-95">
                <i class="fas fa-play text-[8px]"></i> RUN PREVIEW
            </button>
            <button onclick="togglePanel()" class="w-8 h-8 bg-indigo-600 hover:bg-indigo-500 text-white rounded-full flex items-center justify-center text-xs">
                <i class="fas fa-comment-alt"></i>
            </button>
        </div>
    </header>

    <main class="editor-wrapper">
        <div id="lineSidebar" class="line-numbers-sidebar">1</div>
        <div id="editorScrollContainer" class="editor-scroll-container" onscroll="syncScroll(this)">
            <pre id="highlighting" aria-hidden="true"><code id="highlighting-content"></code></pre>
            <textarea id="mainEditor" spellcheck="false" oninput="updateEditor()" onkeydown="handleKeydown(event)" placeholder="พิมพ์หรือสั่ง AI ให้เขียนโค้ดที่นี่..."></textarea>
        </div>
    </main>

    <footer class="prompt-area">
        <div id="imagePreviewContainer">
            <div class="close-img" onclick="removeImage()"><i class="fas fa-times"></i></div>
            <img id="imgPreview" src="" alt="preview">
        </div>

        <div class="max-w-4xl mx-auto">
            <div class="flex flex-wrap gap-2 mb-2 items-center">
                <select id="langSelect" class="bg-gray-800 text-blue-400 border border-white/10 rounded-lg text-[11px] px-2 py-1 outline-none">
                    <option value="HTML/JS">🌐 HTML / JS</option>
                    <option value="Python">🐍 Python</option>
                </select>
                <button onclick="geminiRefactor()" class="bg-indigo-600/20 text-indigo-400 text-[10px] px-3 py-1 rounded-lg border border-indigo-600/30 font-bold hover:bg-indigo-600/40">Refactor ✨</button>
                <button onclick="geminiExplain()" class="bg-purple-600/20 text-purple-400 text-[10px] px-3 py-1 rounded-lg border border-purple-600/30 font-bold hover:bg-purple-600/40">Explain ✨</button>
            </div>
            
            <div class="relative flex items-end gap-2">
                <!-- Image Upload Button Restored -->
                <button onclick="triggerImageUpload()" class="absolute left-3 bottom-2 w-8 h-8 flex items-center justify-center text-gray-400 hover:text-blue-500 z-10">
                    <i class="fas fa-image text-lg"></i>
                </button>
                <input type="file" id="imageInput" accept="image/*" class="hidden" onchange="handleImageFile(event)">
                
                <textarea id="aiPrompt" rows="1" class="ai-input-pill" placeholder="สั่ง AI สร้างหรือแก้ไขโค้ดจากรูปภาพ..." oninput="autoResize(this)" onkeydown="if(event.key==='Enter' && !event.shiftKey){ event.preventDefault(); buildApp(); }"></textarea>
                
                <button id="sendBtn" onclick="buildApp()" class="absolute right-3 bottom-2 w-8 h-8 bg-blue-600 text-white rounded-full flex items-center justify-center shadow-lg hover:scale-105 active:scale-95 transition-all">
                    <i id="sendIcon" class="fas fa-arrow-up"></i>
                </button>
            </div>
        </div>
    </footer>

    <div id="sidePanel">
        <div id="tab-chat" class="panel-tab active">
            <div class="p-4 border-b border-white/5 flex justify-between items-center bg-black/20">
                <div class="flex flex-col">
                    <h3 class="text-xs font-bold uppercase tracking-widest"><i class="fas fa-robot text-blue-500 mr-2"></i> AI Assistant Processor</h3>
                    <span id="aiStatus" class="text-[8px] text-green-500 uppercase">Ready</span>
                </div>
                <button onclick="togglePanel()" class="text-gray-400 hover:text-white"><i class="fas fa-times text-lg"></i></button>
            </div>
            <div id="chatLog" class="flex-1 overflow-y-auto p-4 flex flex-col gap-2"></div>
            
            <div id="aiProcessorUI" class="hidden px-4 py-2">
                <div class="flex items-center gap-2 p-2 bg-blue-500/10 rounded-lg border border-blue-500/20">
                    <i class="fas fa-microchip fa-spin text-blue-500 text-xs"></i>
                    <span class="text-[10px] font-bold text-blue-400 uppercase">AI Processing...</span>
                </div>
            </div>

            <div class="px-4 py-2 border-t border-white/5 flex justify-end">
                <button id="voiceToggleBtn" onclick="toggleVoice()" class="voice-control-btn">
                    <i id="voiceIcon" class="fas fa-volume-mute"></i>
                    <span id="voiceLabel">เปิดเสียงอ่าน</span>
                </button>
            </div>
            
            <div class="p-4 border-t border-white/5 bg-black/10">
                <div class="flex items-center gap-2 bg-gray-800 rounded-xl px-3 py-2 border border-white/5">
                    <textarea id="chatIn" class="flex-1 bg-transparent border-none outline-none text-white text-sm resize-none" rows="1" placeholder="ถามคำถาม..." oninput="autoResize(this)" onkeydown="if(event.key==='Enter' && !event.shiftKey){ event.preventDefault(); sendChat(); }"></textarea>
                    <button onclick="sendChat()" class="text-indigo-400"><i class="fas fa-paper-plane"></i></button>
                </div>
            </div>
        </div>
    </div>

    <!-- Enhanced LIVE VIEW MODE -->
    <div id="previewBox">
        <div class="preview-header">
            <div class="preview-controls">
                <div class="dot bg-red"></div>
                <div class="dot bg-yellow"></div>
                <div class="dot bg-green"></div>
            </div>
            <span class="text-[10px] font-bold text-gray-500 tracking-widest uppercase">Live View Mode — Canvas Optimized</span>
            <button onclick="closePreview()" class="text-gray-500 hover:text-white transition-colors">
                <i class="fas fa-times-circle text-xl"></i>
            </button>
        </div>
        <iframe id="previewIframe" sandbox="allow-scripts allow-forms allow-modals allow-pointer-lock allow-popups allow-same-origin"></iframe>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, onSnapshot, serverTimestamp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const firebaseConfig = JSON.parse(__firebase_config);
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app';
        const apiKey = ""; 

        let currentUser = null;
        window.isVoiceEnabled = false;
        let selectedImageBase64 = null;
        const synth = window.speechSynthesis;

        window.toggleVoice = () => {
            window.isVoiceEnabled = !window.isVoiceEnabled;
            const btn = document.getElementById('voiceToggleBtn');
            const icon = document.getElementById('voiceIcon');
            const label = document.getElementById('voiceLabel');
            if (window.isVoiceEnabled) {
                btn.classList.add('active');
                icon.className = 'fas fa-volume-up';
                label.textContent = 'ปิดเสียงอ่าน';
            } else {
                btn.classList.remove('active');
                icon.className = 'fas fa-volume-mute';
                label.textContent = 'เปิดเสียงอ่าน';
                synth.cancel();
            }
        };

        window.speakSpecificText = (text) => {
            if (!text) return;
            synth.cancel();
            const cleanText = text.replace(/```[\s\S]*?```/g, '[โค้ดโปรแกรม]').replace(/[*#`]/g, '');
            const utter = new SpeechSynthesisUtterance(cleanText);
            utter.lang = 'th-TH';
            utter.rate = 1.1;
            synth.speak(utter);
        };

        window.triggerImageUpload = () => document.getElementById('imageInput').click();
        
        window.handleImageFile = (e) => {
            const file = e.target.files[0];
            if (!file) return;
            const reader = new FileReader();
            reader.onload = (ev) => {
                selectedImageBase64 = ev.target.result.split(',')[1];
                document.getElementById('imgPreview').src = ev.target.result;
                document.getElementById('imagePreviewContainer').style.display = 'block';
                window.showToast("Image ready for AI analysis");
            };
            reader.readAsDataURL(file);
        };

        window.removeImage = () => {
            selectedImageBase64 = null;
            document.getElementById('imageInput').value = "";
            document.getElementById('imagePreviewContainer').style.display = 'none';
        };

        const initAuth = async () => {
            if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                await signInWithCustomToken(auth, __initial_auth_token);
            } else {
                await signInAnonymously(auth);
            }
        };
        initAuth();

        let lastMsgCount = 0;
        onAuthStateChanged(auth, (user) => {
            currentUser = user;
            if (user) {
                const chatRef = collection(db, 'artifacts', appId, 'public', 'data', 'chat_messages');
                onSnapshot(chatRef, (snap) => {
                    let msgs = [];
                    snap.forEach(d => msgs.push(d.data()));
                    msgs.sort((a, b) => (a.timestamp?.seconds || 0) - (b.timestamp?.seconds || 0));
                    
                    if (msgs.length > lastMsgCount && window.isVoiceEnabled) {
                        const last = msgs[msgs.length - 1];
                        if (last.sender === 'ai') window.speakSpecificText(last.text);
                    }
                    lastMsgCount = msgs.length;
                    renderChat(msgs);
                });
            }
        });

        function renderChat(messages) {
            const log = document.getElementById('chatLog');
            log.innerHTML = "";
            messages.forEach(m => {
                const container = document.createElement('div');
                container.className = m.sender === 'ai' ? 'msg-ai-container' : 'flex flex-col items-end';
                const div = document.createElement('div');
                div.className = `msg-bubble ${m.sender === 'user' ? 'msg-user' : 'msg-ai'}`;
                
                let content = m.text;
                if (m.sender === 'ai') {
                    const sb = document.createElement('button');
                    sb.className = 'speak-btn-bubble';
                    sb.innerHTML = '<i class="fas fa-volume-up"></i>';
                    sb.onclick = () => window.speakSpecificText(m.text);
                    container.appendChild(sb);
                    
                    content = content.replace(/```(\w*)\n([\s\S]*?)```/g, (match, lang, code) => {
                        const id = 'c' + Math.random().toString(36).substr(2, 5);
                        window[id] = code.trim();
                        return `<div class="code-block-container"><div class="code-header"><span class="text-[9px] text-gray-500 font-bold uppercase">${lang||'code'}</span><button class="apply-btn" onclick="applyCode('${id}')">APPLY</button></div><pre class="chat-code"><code>${code.replace(/</g,'&lt;')}</code></pre></div>`;
                    });
                }
                div.innerHTML = `<div class="msg-content">${content}</div>`;
                container.appendChild(div);
                log.appendChild(container);
            });
            log.scrollTop = log.scrollHeight;
        }

        window.applyCode = (id) => {
            document.getElementById('mainEditor').value = window[id];
            window.updateEditor();
            window.showToast("Applied to editor!");
        };

        window.sendChat = async () => {
            const input = document.getElementById('chatIn');
            const text = input.value.trim();
            if (!text || !currentUser) return;
            input.value = "";
            document.getElementById('aiProcessorUI').classList.remove('hidden');
            const chatRef = collection(db, 'artifacts', appId, 'public', 'data', 'chat_messages');
            await addDoc(chatRef, { text, sender: 'user', timestamp: serverTimestamp() });

            try {
                const res = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify({
                        contents: [{ parts: [{ text: `Code Context:\n${document.getElementById('mainEditor').value}\n\nUser Question: ${text}` }] }]
                    })
                });
                const data = await res.json();
                const aiText = data.candidates?.[0]?.content?.parts?.[0]?.text || "Error";
                await addDoc(chatRef, { text: aiText, sender: 'ai', timestamp: serverTimestamp() });
            } catch(e) { console.error(e); }
            finally { document.getElementById('aiProcessorUI').classList.add('hidden'); }
        };

        window.buildApp = async () => {
            const promptInput = document.getElementById('aiPrompt');
            const prompt = promptInput.value.trim();
            if (!prompt && !selectedImageBase64) return;
            
            const icon = document.getElementById('sendIcon');
            icon.className = "fas fa-spinner fa-spin";
            
            try {
                // SYSTEM INSTRUCTION FOR IMAGE TO APP CONVERSION
                const systemPrompt = `You are a professional App Builder AI. 
                If an image is provided, analyze its layout, colors, buttons, and overall UI design precisely. 
                Generate the complete, runnable HTML/Tailwind/JS code to match the image or user request.
                If there is current code in the editor, update it according to the user's instructions or the provided image.
                OUTPUT ONLY THE CODE WITHOUT MARKDOWN BLOCKS OR EXTRA TEXT.`;

                const payload = {
                    contents: [{
                        parts: [
                            { text: `Task: ${prompt || "Build an app exactly like the UI shown in the image."}\n\nCurrent Editor Code:\n${document.getElementById('mainEditor').value}` }
                        ]
                    }],
                    systemInstruction: { parts: [{ text: systemPrompt }] }
                };

                if (selectedImageBase64) {
                    payload.contents[0].parts.push({
                        inlineData: {
                            mimeType: "image/png",
                            data: selectedImageBase64
                        }
                    });
                }

                const res = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
                    method: 'POST',
                    headers: {'Content-Type': 'application/json'},
                    body: JSON.stringify(payload)
                });
                
                const data = await res.json();
                let code = data.candidates?.[0]?.content?.parts?.[0]?.text || "";
                
                // Clean up possible markdown or surrounding text
                code = code.replace(/^```[a-zA-Z]*\n?/gm, '').replace(/```$/gm, '').trim();
                
                if (code && code.length > 20) {
                    document.getElementById('mainEditor').value = code;
                    window.updateEditor();
                    promptInput.value = "";
                    window.removeImage();
                    window.showToast("App layout updated from Image Analysis");
                } else {
                    window.showToast("AI response was too short or empty.");
                }
            } catch(e) { 
                console.error(e); 
                window.showToast("Visual Build Failed."); 
            } finally { 
                icon.className = "fas fa-arrow-up"; 
            }
        };
    </script>

    <script>
        const editor = document.getElementById('mainEditor');
        const highlight = document.getElementById('highlighting-content');
        const lineSidebar = document.getElementById('lineSidebar');

        function updateEditor() {
            const text = editor.value;
            let html = text.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;');
            html = html.replace(/\b(function|var|let|const|if|else|return|for|while|class|import|def|public|static|void)\b/g, '<span class="token keyword">$1</span>');
            highlight.innerHTML = html + "\n";
            const lines = text.split('\n').length;
            let nums = ""; for (let i = 1; i <= lines; i++) nums += i + "\n";
            lineSidebar.textContent = nums;
        }
        window.updateEditor = updateEditor;

        function syncScroll(el) {
            document.getElementById('highlighting').style.transform = `translate(${-el.scrollLeft}px, ${-el.scrollTop}px)`;
            lineSidebar.scrollTop = el.scrollTop;
        }

        function runPreview() {
            const iframe = document.getElementById('previewIframe');
            const code = editor.value;
            let finalDoc = code;
            if (!code.toLowerCase().includes('<html')) {
                finalDoc = `<!DOCTYPE html><html><head><meta charset="UTF-8"><script src="https://cdn.tailwindcss.com"><\/script></head><body>${code}</body></html>`;
            }
            iframe.srcdoc = finalDoc;
            document.getElementById('previewBox').classList.add('active');
        }

        function closePreview() { document.getElementById('previewBox').classList.remove('active'); }
        function toggleTheme() { document.body.classList.toggle('light-mode'); }
        function togglePanel() { document.getElementById('sidePanel').classList.toggle('active'); }
        function autoResize(el) { el.style.height = 'auto'; el.style.height = el.scrollHeight + 'px'; }
        function handleKeydown(e) { if(e.key === 'Tab'){ e.preventDefault(); const s = editor.selectionStart; editor.value = editor.value.substring(0,s) + "    " + editor.value.substring(editor.selectionEnd); editor.selectionStart = editor.selectionEnd = s + 4; updateEditor(); } }

        window.showToast = (m) => {
            const t = document.getElementById('toast');
            const msg = document.getElementById('toastMessage');
            if (msg) msg.textContent = m;
            t.style.opacity = '1'; t.style.transform = 'translateY(0)';
            setTimeout(() => { t.style.opacity = '0'; t.style.transform = 'translateY(20px)'; }, 2500);
        };

        window.geminiRefactor = () => { document.getElementById('chatIn').value = "ช่วยปรับปรุงโค้ดนี้ให้ทันสมัยและสวยงามขึ้น"; togglePanel(); window.sendChat(); };
        window.geminiExplain = () => { document.getElementById('chatIn').value = "อธิบายโค้ดนี้ให้ฟังหน่อยว่าทำงานอย่างไร"; togglePanel(); window.sendChat(); };

        updateEditor();
    </script>
</body>
</html>
