index.html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Lynneh Med - Asistente & Esquemas Médicos</title>
    <style>
        * {
            box-sizing: border-box;
            margin: 0;
            padding: 0;
            font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
        }

        body {
            background-color: #0b1120;
            color: #f1f5f9;
            height: 100vh;
            display: flex;
            justify-content: center;
            align-items: flex-end;
            padding-bottom: 20px;
            overflow: hidden;
        }

        .ai-taskbar {
            width: 95%;
            max-width: 800px;
            background: rgba(15, 23, 42, 0.92);
            backdrop-filter: blur(16px);
            border: 1px solid rgba(56, 189, 248, 0.3);
            border-radius: 18px;
            padding: 14px;
            box-shadow: 0 12px 35px rgba(0, 0, 0, 0.7);
            display: flex;
            flex-direction: column;
            gap: 12px;
            transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
        }

        .ai-response-area {
            background: rgba(30, 41, 59, 0.85);
            border-radius: 12px;
            padding: 14px 18px;
            font-size: 13.5px;
            line-height: 1.5;
            color: #e2e8f0;
            max-height: 320px;
            overflow-y: auto;
            display: none;
            border-left: 4px solid #38bdf8;
            white-space: pre-wrap;
        }

        .ai-response-area.active {
            display: block;
        }

        /* Formato para esquemas y resúmenes */
        .med-title { font-weight: 700; color: #38bdf8; font-size: 15px; margin-bottom: 6px; }
        .med-section { font-weight: 600; color: #a7f3d0; margin-top: 8px; }
        .med-bullet { margin-left: 10px; color: #cbd5e1; }

        .bar-main {
            display: flex;
            align-items: center;
            gap: 10px;
        }

        .ai-logo {
            background: linear-gradient(135deg, #0284c7, #2563eb);
            color: white;
            border: none;
            width: 38px;
            height: 38px;
            border-radius: 50%;
            display: flex;
            justify-content: center;
            align-items: center;
            font-weight: bold;
            cursor: pointer;
            box-shadow: 0 2px 10px rgba(56, 189, 248, 0.4);
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
            background: rgba(255, 255, 255, 0.06);
            border: 1px solid rgba(255, 255, 255, 0.12);
            color: #94a3b8;
            padding: 6px 12px;
            border-radius: 8px;
            cursor: pointer;
            font-size: 12px;
            transition: all 0.2s;
        }

        .tool-btn:hover {
            background: rgba(255, 255, 255, 0.18);
            color: #ffffff;
        }

        .mic-active {
            background: #ef4444 !important;
            color: white !important;
            box-shadow: 0 0 12px rgba(239, 68, 68, 0.6);
        }
    </style>
</head>
<body>

    <div class="ai-taskbar">
        <div id="responseArea" class="ai-response-area">
            <div class="med-title">Módulo Clínico & Académico Lynneh</div>
            Ejemplos de comandos:<br>
            • <strong>"Lynneh esquema apendicitis"</strong> (Genera esquema sintético)<br>
            • <strong>"Lynneh resumen SRAA"</strong> (Resumen del sistema Renina-Angiotensina)<br>
            • <strong>"Lynneh resumen Goodman adrenergicos"</strong> (Farmacología)
        </div>

        <div class="bar-main">
            <button class="ai-logo" title="Lynneh Medical Edition">L</button>
            <input type="text" id="commandInput" class="ai-input" placeholder="Pide un esquema, resumen o concepto médico..." autofocus>
            
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

        // OPCIONAL: Pega aquí tu API Key de Google Gemini para respuestas médicas infinitas con IA real
        const GEMINI_API_KEY = ""; 

        let recognition;
        let isListening = false;

        // Configuración del micrófono
        if ('webkitSpeechRecognition' in window || 'SpeechRecognition' in window) {
            const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
            recognition = new SpeechRecognition();
            recognition.continuous = true;
            recognition.lang = 'es-ES';

            recognition.onresult = (event) => {
                const transcript = event.results[event.results.length - 1][0].transcript.trim();
                inputField.value = transcript;
                processVoiceCommand(transcript);
            };

            recognition.onend = () => { if (isListening) recognition.start(); };
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
            }
        }

        inputField.addEventListener('keypress', function(e) {
            if (e.key === 'Enter') processCommand(inputField.value);
        });

        function processVoiceCommand(text) {
            const query = text.toLowerCase();
            if (query.includes('lynneh') || query.includes('line') || query.includes('linea') || query.includes('lyne')) {
                let clean = query.replace('lynneh','').replace('linea','').replace('line','').replace('lyne','').trim();
                processCommand(clean);
            }
        }

        // --- PROCESADOR MÉRICO DE ESQUEMAS Y LIBROS ---
        async function processCommand(cleanText) {
            if (!cleanText) return;

            responseArea.classList.add('active');
            responseArea.innerHTML = "<em>Procesando consulta bibliográfica y esquematizando...</em>";

            const lower = cleanText.toLowerCase();

            // Si hay API Key de Gemini configurada, consulta directamente a la IA Médica
            if (GEMINI_API_KEY) {
                await queryGeminiAI(cleanText);
                return;
            }

            // Motor de Resúmenes y Esquemas Locales
            let formattedResponse = "";

            if (lower.includes('esquema') || lower.includes('algoritmo')) {
                formattedResponse = generateMedicalSchema(lower);
            } else if (lower.includes('resumen') || lower.includes('libro')) {
                formattedResponse = generateBookSummary(lower);
            } else if (lower.startsWith('abrir ') || lower.startsWith('reproducir ') || lower.includes('hora')) {
                handleGeneralTools(lower);
                return;
            } else {
                // Búsqueda orientada a literatura médica oficial
                window.open(`https://www.google.com/search?q=${encodeURIComponent(cleanText + " medscape scielo accessmedicina")}`, '_blank');
                formattedResponse = `<div class="med-title">Búsqueda Bibliográfica</div>Buscando literatura académica oficial para: <strong>"${cleanText}"</strong>.`;
            }

            renderAndSpeak(formattedResponse);
            inputField.value = "";
        }

        // Generador de Esquemas Quirúrgicos y Clínicos
        function generateMedicalSchema(topic) {
            if (topic.includes('apendicitis')) {
                return `<div class="med-title">ESQUEMA CLÍNICO-QUIRÚRGICO: APENDICITIS AGUDA</div>
<div class="med-section">1. Etiología y Fisiopatología:</div>
<div class="med-bullet">• Obstrucción de la luz apendicular (Fecalito > Hiperplasia linfoide > Cuerpos extraños).</div>
<div class="med-bullet">• Aumento de presión intraluminal ➔ Isquemia mucosa ➔ Proliferación bacteriana ➔ Perforación.</div>

<div class="med-section">2. Triada / Secuencia de Murphy:</div>
<div class="med-bullet">• Dolor epigástrico/periumbilical ➔ Anorexia/Nausea ➔ Migración a Fosa Ilíaca Derecha (FID).</div>

<div class="med-section">3. Puntos y Signos Semiológicos Clave:</div>
<div class="med-bullet">• <strong>McBurney:</strong> Dolor a la presión en 1/3 externo de la línea ombligo-espina ilíaca anterosuperior.</div>
<div class="med-bullet">• <strong>Blumberg:</strong> Dolor a la descompresión brusca (Irritación peritoneal).</div>
<div class="med-bullet">• <strong>Rovsing:</strong> Dolor en FID al presionar FII.</div>

<div class="med-section">4. Manejo Quirúrgico (Sabiston / Schwartz):</div>
<div class="med-bullet">• Escala de Alvarado para estratificación.</div>
<div class="med-bullet">• Apendicectomía (Laparoscópica vs Abierta) + Profilaxis antibiótica preoperatoria.</div>`;
            }

            return `<div class="med-title">ESQUEMA ESTRUCTURADO: ${topic.replace('esquema','').toUpperCase()}</div>
<div class="med-section">1. Definición & Etiología:</div>
<div class="med-bullet">• Proceso patológico caracterizado por alteración estructural o funcional.</div>
<div class="med-section">2. Fisiopatología & Semiótica:</div>
<div class="med-bullet">• Mecanismo desencadenante ➔ Cascada inflamatoria/isquémica ➔ Sintomatología cardinal.</div>
<div class="med-section">3. Diagnóstico & Abordaje:</div>
<div class="med-bullet">• Historia clínica + Exploración física focalizada + Estudios auxiliares (Laboratorio / Imagen).</div>`;
        }

        // Generador de Resúmenes Estilo Libro (Guyton / Goodman & Gilman / Argente)
        function generateBookSummary(topic) {
            if (topic.includes('sraa') || topic.includes('renina')) {
                return `<div class="med-title">RESUMEN FISIOLÓGICO (GUYTON & HALL): SRAA</div>
<div class="med-section">Contexo Fisiológico:</div>
Sistema hormonal clave para la regulación de la presión arterial (PA) y el volumen extracelular.

<div class="med-section">Cascada Bioquímica:</div>
<div class="med-bullet">1. <strong>Liberación de Renina:</strong> Producida por las células yuxtaglomerulares renales ante hipoperfusión o baja de NaCl.</div>
<div class="med-bullet">2. <strong>Sustrato:</strong> La Renina escinde el Angiotensinógeno (hepático) en <strong>Angiotensina I</strong>.</div>
<div class="med-bullet">3. <strong>Conversión:</strong> La ECA (Enzima Convertidora de Angiotensina, endotelio pulmonar) convierte Angiotensina I en <strong>Angiotensina II</strong>.</div>

<div class="med-section">Efectos Fisiológicos de Angiotensina II:</div>
<div class="med-bullet">• Vasoconstricción arteriolar potente (aumenta Resistencia Vascular Periférica).</div>
<div class="med-bullet">• Estimula la secreción de <strong>Aldosterona</strong> en la corteza suprarrenal (Reabsorción de Na+ y H2O en túbulo colector).</div>
<div class="med-bullet">• Estimula la liberación de ADH (Vasopresina) y el centro de la sed en el hipotálamo.</div>`;
            }

            return `<div class="med-title">RESUMEN BIBLIOGRÁFICO: ${topic.replace('resumen','').toUpperCase()}</div>
<div class="med-section">Concepto Central:</div>
Revisión orientada a libros de texto oficiales (Anatomía de Moore, Fisiología de Guyton, Farmacología de Goodman).
<div class="med-section">Puntos Clave para Examen:</div>
<div class="med-bullet">• Correlación anatomoclínica y mecanismo molecular.</div>
<div class="med-bullet">• Vías metabólicas / Vías de señalización celular.</div>`;
        }

        function handleGeneralTools(lower) {
            let answer = "";
            if (lower.startsWith('abrir ')) {
                const target = lower.replace('abrir', '').trim();
                window.open(`https://${target.includes('.') ? target : target + '.com'}`, '_blank');
                answer = `Abriendo ${target}.`;
            } else if (lower.startsWith('reproducir ')) {
                const song = lower.replace('reproducir', '').trim();
                window.open(`https://www.youtube.com/results?search_query=${encodeURIComponent(song)}`, '_blank');
                answer = `Buscando música: ${song}.`;
            } else if (lower.includes('hora')) {
                answer = `Hora: ${new Date().toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })}`;
            }
            renderAndSpeak(answer);
        }

        // Integración opcional con Gemini API para respuestas de libro en tiempo real
        async function queryGeminiAI(prompt) {
            try {
                const systemPrompt = "Eres Lynneh, una asistente médica universitaria de alto nivel. Responde siempre con formato académico impecable (estilo Guyton, Moore, Argente o Goodman & Gilman). Organiza la respuesta utilizando títulos claros, viñetas, esquemas paso a paso o algoritmos clínicos resumidos.";
                
                const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent?key=${GEMINI_API_KEY}`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        contents: [{ parts: [{ text: `${systemPrompt}\n\nConsulta del estudiante: ${prompt}` }] }]
                    })
                });

                const data = await response.json();
                const text = data.candidates[0].content.parts[0].text;
                renderAndSpeak(text);
            } catch (err) {
                renderAndSpeak("Error conectando con la IA médica. Verifique la API Key.");
            }
        }

        function renderAndSpeak(htmlText) {
            responseArea.innerHTML = htmlText;
            
            // Extraer solo texto plano para que el sintetizador de voz hable sin leer etiquetas HTML
            const plainText = responseArea.innerText.replace(/•/g, '').replace(/[\r\n]+/g, ' ');

            if ('speechSynthesis' in window) {
                window.speechSynthesis.cancel();
                const utterance = new SpeechSynthesisUtterance(plainText.substring(0, 250) + "..."); // Resume la lectura de voz para no saturar
                utterance.lang = 'es-ES';
                utterance.rate = 1.05;
                window.speechSynthesis.speak(utterance);
            }
        }

        function runTool(tool) {
            if (tool === 'clear') {
                responseArea.classList.remove('active');
                responseArea.innerHTML = "";
                inputField.value = "";
                if ('speechSynthesis' in window) window.speechSynthesis.cancel();
            }
        }
    </script>
</body>
</html>
