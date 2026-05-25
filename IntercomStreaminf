<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Intercom vMix - Fase de Prueba</title>
    <script src="https://unpkg.com/peerjs@1.5.2/dist/peerjs.min.js"></script>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #1e1e1e;
            color: #ffffff;
            text-align: center;
            padding: 20px;
            margin: 0;
        }
        h1 { color: #f39c12; }
        button {
            padding: 15px 30px;
            font-size: 18px;
            margin: 10px;
            cursor: pointer;
            border-radius: 8px;
            border: none;
            font-weight: bold;
            transition: opacity 0.2s;
        }
        button:active { opacity: 0.8; }
        #btn-director { background-color: #e74c3c; color: white; }
        #btn-camara { background-color: #3498db; color: white; }
        #btn-conectar { background-color: #2ecc71; color: white; width: 100%; max-width: 300px; margin-top: 20px;}
        .hidden { display: none; }
        .box {
            background-color: #2c3e50;
            padding: 20px;
            border-radius: 10px;
            max-width: 400px;
            margin: 0 auto;
            box-shadow: 0 4px 6px rgba(0,0,0,0.3);
        }
        input {
            padding: 10px;
            font-size: 16px;
            border-radius: 5px;
            border: 1px solid #7f8c8d;
            background: #34495e;
            color: white;
            width: calc(100% - 24px);
            text-align: center;
        }
        #status { font-weight: bold; color: #f1c40f; margin-top: 20px; }
    </style>
</head>
<body>

    <h1>Intercom vMix</h1>
    
    <div id="setup" class="box">
        <h2>Selecciona tu rol</h2>
        <button id="btn-director" onclick="startAsDirector()">Soy el Director (PC)</button>
        <button id="btn-camara" onclick="startAsCamara()">Soy Camarógrafo (Móvil)</button>
    </div>

    <div id="intercom" class="box hidden">
        <h2 id="role-title"></h2>
        <p style="font-size: 12px; color: #bdc3c7;">Tu ID en la red: <span id="my-id">Generando...</span></p>
        
        <div id="camara-controls" class="hidden">
            <p>Ingresa el ID del Director para unirte:</p>
            <input type="text" id="director-id-input" value="director-vmix-produccion">
            <button id="btn-conectar" onclick="llamarAlDirector()">Conectar y Hablar</button>
        </div>

        <p id="status">Esperando acción...</p>
        
        <audio id="remote-audio" autoplay></audio>
    </div>

    <script>
        let peer;
        let localStream;
        // Definimos un ID fijo para que los camarógrafos sepan a quién llamar
        const ID_DEL_DIRECTOR = 'director-vmix-produccion'; 

        // Función para pedir permiso y capturar el micrófono
        async function obtenerMicrofono() {
            try {
                return await navigator.mediaDevices.getUserMedia({ audio: true, video: false });
            } catch (err) {
                alert("Error: No se pudo acceder al micrófono. Revisa los permisos de tu navegador.");
                console.error(err);
            }
        }

        // Lógica para el Director (quien recibe las llamadas)
        async function startAsDirector() {
            document.getElementById('setup').classList.add('hidden');
            document.getElementById('intercom').classList.remove('hidden');
            document.getElementById('role-title').innerText = "Control de Dirección";
            document.getElementById('status').innerText = "Iniciando servidor de audio...";
            
            localStream = await obtenerMicrofono();
            
            // Iniciamos PeerJS forzando el ID predefinido
            peer = new Peer(ID_DEL_DIRECTOR); 

            peer.on('open', (id) => {
                document.getElementById('my-id').innerText = id;
                document.getElementById('status').innerText = "En línea. Esperando a los camarógrafos...";
            });

            // Cuando un camarógrafo llama al director
            peer.on('call', (call) => {
                document.getElementById('status').innerText = "¡Camarógrafo conectado!";
                // El director contesta la llamada y envía su propio audio
                call.answer(localStream); 
                
                // El director escucha el audio del camarógrafo
                call.on('stream', (remoteStream) => {
                    document.getElementById('remote-audio').srcObject = remoteStream;
                });
            });
        }

        // Lógica para el Camarógrafo (quien inicia la llamada)
        async function startAsCamara() {
            document.getElementById('setup').classList.add('hidden');
            document.getElementById('intercom').classList.remove('hidden');
            document.getElementById('camara-controls').classList.remove('hidden');
            document.getElementById('role-title').innerText = "Cámara";
            document.getElementById('status').innerText = "Listo para conectar.";
            
            localStream = await obtenerMicrofono();
            
            // Iniciamos PeerJS con un ID aleatorio automático
            peer = new Peer(); 

            peer.on('open', (id) => {
                document.getElementById('my-id').innerText = id;
            });

            // Por si el director decide llamar de vuelta (opcional en esta estructura)
            peer.on('call', (call) => {
                call.answer(localStream);
                call.on('stream', (remoteStream) => {
                    document.getElementById('remote-audio').srcObject = remoteStream;
                });
            });
        }

        // El Camarógrafo presiona el botón para conectar con el Director
        function llamarAlDirector() {
            const targetId = document.getElementById('director-id-input').value;
            document.getElementById('status').innerText = "Conectando con Dirección...";
            
            // Llamamos al ID del director y le pasamos nuestro micrófono
            const call = peer.call(targetId, localStream);
            
            call.on('stream', (remoteStream) => {
                document.getElementById('status').innerText = "¡AUDIO EN VIVO! Escuchando a Dirección.";
                document.getElementById('remote-audio').srcObject = remoteStream;
            });
            
            call.on('error', (err) => {
                document.getElementById('status').innerText = "Error de conexión.";
                console.error(err);
            });
        }
    </script>
</body>
</html>
