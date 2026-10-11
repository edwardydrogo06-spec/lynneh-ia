index.html

<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Barra de Tareas IA - Lynneh</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        body {
            background-color: #0f172a;
            color: #f8fafc;
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: flex-end;
            padding-bottom: 30px;
            overflow: hidden;
        }

        .ai-taskbar {
            width: 90%;
            max-width: 700px;
            background: rgba(30, 41, 59, 0.85);
            backdrop-filter: blur(12px);
            border: 1px solid rgba(255, 255, 255, 0.1);
            border-radius: 16px;
            padding: 10px 15px;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.5);
            display: flex;
            flex-direction: column;
            gap: 10px;
            transition: all 0.3s ease;
        }

        .ai-response-area {
            background: rgba(15, 23, 42, 0.6);
            border-radius: 10px;
            padding: 10px 15px;
            font-size: 14px;
            color: #cbd5e1;
            max-height: 120px;
            overflow-y: auto;
            display: none;
            border-left: 3px solid #3b82f6;
        }

        .ai-response-area.active {
            display: block;
        }

        .bar-main {
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .ai-logo {
            background: linear-gradient(135deg, #3b82f6, #8b5cf6);
            color: white;
            border: none;
            width: 36px;
            height: 36px;
            border-radius: 50%;
            display: flex;
            justify-content: center;
            align-items: center;
            font-weight: bold;
            cursor: pointer;
            box-shadow: 0 2px 5px rgba(59, 130, 246, 0.4);
            flex-shrink: 0;
        }

        .ai-input {
            flex-grow: 1;
            background: transparent;
            border: none;
            color: #ffffff;
            font-size: 15px;
            outline: none;
        }

        .ai-input::placeholder {
            color: #64748b;
        }

        .tools {
            display: flex;
            gap: 6px;
        }

        .tool-btn {
            background: rgba(255, 255, 255, 0.05);
            border: 1px solid rgba(255, 255, 255, 0.1);
            color: #94a3b8;
            padding: 6px 10px;
            border-radius: 8px;
            cursor: pointer;
            font-size: 12px;
            transition: all 0.2s;
        }

        .tool-btn:hover {
            background: rgba(255, 255, 255, 0.15);
            color: #ffffff;
        }

        .mic-active {
            background: #ef4444 !important;
            color: white !important;
            box-shadow: 0 0 10px rgba(239, 68, 68, 0.6);
        }
    </style>
</head>
<body>

    <div class="ai-taskbar">
        <div id="responseArea" class="ai-response-area">
            Di <strong>"Lynneh"</strong> seguido de tu orden (ej: <em>"Lynneh reproduce música relajante"</em> o <em>"Lynneh temporizador de 5 minutos"</em>).
        </div>

        <div class="bar-main">
            <button class="ai-logo" title="Asistente activo">L</button>
            <input type="text" id="commandInput" class="ai-input" placeholder="Pregúntale a Lynneh..." autofocus>
            
            <div class="tools">
                <button id="micBtn" class="tool-btn" onclick="toggleVoiceRecognition()">🎙️ Voz</button>
                <button class="tool-btn" onclick="runTool('clear')">🧹 Limpiar</button>
            </div>
        </div>
    </div>

    <script>
        const inputField = document.getElementById('commandInput');
        const responseArea = document.getElementById('responseArea');
        const micBtn = document.getElementById('micBtn');

        let recognition;
        let isListening = false;
        let activeTimer = null; // Para controlar temporizadores

        // Configuración de Reconocimiento de Voz
        if ('webkitSpeechRecognition' in window || 'SpeechRecognition' in window) {
            const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
            recognition = new SpeechRecognition();
            recognition.continuous = true;
            recognition.lang = 'es-ES';
            recognition.interimResults = false;

            recognition.onresult = (event) => {
                const lastIndex = event.results.length - 1;
                const transcript = event.results[lastIndex][0].transcript.trim();
                inputField.value = transcript;
                processVoiceCommand(transcript);
            };

            recognition.onend = () => {
                if (isListening) recognition.start();
            };
        } else {
            alert("Tu navegador no soporta la API de voz.");
        }

        function toggleVoiceRecognition() {
            if (!recognition) return;

            if (isListening) {
                recognition.stop();
                isListening = false;
                micBtn.classList.remove('mic-active');
                micBtn.innerText = "🎙️ Voz";
            } else {
                recognition.start();
                isListening = true;
                micBtn.classList.add('mic-active');
                micBtn.innerText = "🛑 Escuchando";
                responseArea.classList.add('active');
                responseArea.innerHTML = "<em>Escuchando... Di <strong>'Lynneh'</strong> para darle una orden.</em>";
            }
        }

        inputField.addEventListener('keypress', function(e) {
            if (e.key === 'Enter') {
                processCommand(inputField.value);
            }
        });

        function processVoiceCommand(text) {
            const query = text.toLowerCase();
            if (query.includes('lynneh') || query.includes('line') || query.includes('linea') || query.includes('lyne')) {
                let cleanQuery = query.replace('lynneh', '').replace('linea', '').replace('line', '').replace('lyne', '').trim();
                processCommand(cleanQuery);
            }
        }

        // Lógica de funciones ampliadas
        function processCommand(cleanText) {
            if (!cleanText) {
                speakAndShow("¿Sí? Dime en qué te puedo ayudar.");
                return;
            }

            let answer = "";

            if (cleanText.includes('hola') || cleanText.includes('saludos')) {
                answer = "¡Hola! Soy Lynneh, tu asistente. ¿Qué necesitas realizar?";
            } 
            else if (cleanText.includes('hora') || cleanText.includes('fecha')) {
                const now = new Date();
                answer = `Son las ${now.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })}.`;
            } 
            // 1. FUNCIÓN ABRIR (ej: "abrir youtube", "abrir google")
            else if (cleanText.startsWith('abrir ')) {
                const target = cleanText.replace('abrir', '').trim();
                let url = "";
                if (target.includes('youtube')) url = "https://www.youtube.com";
                else if (target.includes('google')) url = "https://www.google.com";
                else if (target.includes('whatsapp')) url = "https://web.whatsapp.com";
                else if (target.includes('spotify')) url = "https://open.spotify.com";
                else url = `https://www.google.com/search?q=${encodeURIComponent(target)}`;

                window.open(url, '_blank');
                answer = `Abriendo ${target}.`;
            }
            // 2. FUNCIÓN REPRODUCIR MÚSICA (ej: "reproducir rock", "reproducir música lo-fi")
            else if (cleanText.startsWith('reproducir ') || cleanText.startsWith('musica ')) {
                const song = cleanText.replace('reproducir', '').replace('musica', '').trim();
                const musicUrl = `https://www.youtube.com/results?search_query=${encodeURIComponent(song + " audio oficial")}`;
                window.open(musicUrl, '_blank');
                answer = `Reproduciendo ${song || 'música'} en YouTube.`;
            }
            // 3. FUNCIÓN TEMPORIZADOR (ej: "temporizador de 2 minutos", "temporizador 30 segundos")
            else if (cleanText.includes('temporizador') || cleanText.includes('cuenta regresiva')) {
                const numbers = cleanText.match(/\d+/);
                if (numbers) {
                    const value = parseInt(numbers[0]);
                    let milisegundos = value * 1000; // Por defecto segundos

                    if (cleanText.includes('minuto') || cleanText.includes('minutos')) {
                        milisegundos = value * 60 * 1000;
                    } else if (cleanText.includes('hora') || cleanText.includes('horas')) {
                        milisegundos = value * 60 * 60 * 1000;
                    }

                    answer = `Temporizador activado por ${value} ${cleanText.includes('minuto') ? 'minutos' : 'segundos'}. Te avisaré.`;
                    speakAndShow(answer);
                    
                    if (activeTimer) clearTimeout(activeTimer);
                    activeTimer = setTimeout(() => {
                        playAlarmSound();
                        speakAndShow("¡El tiempo del temporizador ha terminado!");
                    }, milisegundos);
                    return;
                } else {
                    answer = "Por favor especifica el tiempo del temporizador (ej: 'temporizador de 3 minutos').";
                }
            }
            else if (cleanText.startsWith('calcular ') || cleanText.includes('cuanto es')) {
                try {
                    let expr = cleanText.replace('calcular', '').replace('cuanto es', '').replace(/x/g, '*');
                    answer = `El resultado es ${eval(expr)}`;
                } catch {
                    answer = "No pude realizar ese cálculo.";
                }
            } 
            else if (cleanText.includes('buscar')) {
                const searchVal = cleanText.replace('buscar', '').trim();
                window.open(`https://www.google.com/search?q=${encodeURIComponent(searchVal)}`, '_blank');
                answer = `Buscando ${searchVal} en Google.`;
            } 
            else {
                answer = `Procesando orden: "${cleanText}".`;
            }

            speakAndShow(answer);
            inputField.value = "";
        }

        // Sonido de Alarma sutil cuando finaliza el temporizador
        function playAlarmSound() {
            try {
                const ctx = new (window.AudioContext || window.webkitAudioContext)();
                const osc = ctx.createOscillator();
                const gain = ctx.createGain();
                osc.type = 'sine';
                osc.frequency.setValueAtTime(587.33, ctx.currentTime); // Nota Re5
                gain.gain.setValueAtTime(0.1, ctx.currentTime);
                osc.connect(gain);
                gain.connect(ctx.destination);
                osc.start();
                osc.stop(ctx.currentTime + 1.2); // Suena 1.2 segundos
            } catch (e) {
                console.log("Alarma visual activada");
            }
        }

        function speakAndShow(text) {
            responseArea.classList.add('active');
            responseArea.innerHTML = `<strong>Lynneh:</strong> ${text}`;

            if ('speechSynthesis' in window) {
                window.speechSynthesis.cancel();
                const utterance = new SpeechSynthesisUtterance(text);
                utterance.lang = 'es-ES';
                utterance.rate = 1.0;
                utterance.pitch = 1.1;
                window.speechSynthesis.speak(utterance);
            }
        }

        function runTool(tool) {
            if (tool === 'clear') {
                responseArea.classList.remove('active');
                responseArea.innerHTML = "";
                inputField.value = "";
                if ('speechSynthesis' in window) window.speechSynthesis.cancel();
                if (activeTimer) clearTimeout(activeTimer);
            }
        }
    </script>
</body>
</html>


