# acampamento-nossa-senhora-da-boa-morte
<!DOCTYPE html>
<html lang="pt-BR">
<head>
  <meta charset="UTF-8" />
  <title>Escute o Vento</title>
  <style>
    body, html {
      margin: 0; padding: 0; height: 100%;
      overflow: hidden;
      background: black;
      display: flex;
      flex-direction: column;
      justify-content: center;
      align-items: center;
      color: white;
      font-family: Arial, sans-serif;
    }
    .glitch {
      position: relative;
      font-size: 6rem;
      font-weight: bold;
      color: #fff;
      animation: glitch 1s infinite, shake 0.1s infinite;
      cursor: pointer;
      user-select: none;
    }
    .glitch::before,
    .glitch::after {
      content: attr(data-text);
      position: absolute;
      left: 0; top: 0;
      width: 100%;
      overflow: hidden;
      clip: rect(0, 900px, 0, 0);
      animation: shake 0.1s infinite;
    }
    .glitch::before {
      left: 2px;
      text-shadow: -2px 0 red;
      animation: glitchTop 1s infinite, shake 0.1s infinite;
    }
    .glitch::after {
      left: -2px;
      text-shadow: -2px 0 blue;
      animation: glitchBottom 1s infinite, shake 0.1s infinite;
    }
    
    .instructions {
      position: absolute;
      bottom: 50px;
      font-size: 1.2rem;
      opacity: 0.7;
      text-align: center;
      animation: pulse 2s infinite;
    }
    
    .audio-status {
      position: absolute;
      top: 30px;
      right: 30px;
      padding: 10px 20px;
      background: rgba(255, 255, 255, 0.1);
      border-radius: 25px;
      font-size: 0.9rem;
      backdrop-filter: blur(10px);
    }
    
    .playing {
      color: #00ffea !important;
    }
    
    .paused {
      color: #ff005c !important;
    }
    
    @keyframes pulse {
      0%, 100% { opacity: 0.7; }
      50% { opacity: 0.3; }
    }
    
    @keyframes glitch {
      0% { text-shadow: 2px 0 #00ffea, -2px 0 #ff005c; }
      20% { text-shadow: -2px 0 #00ffea, 2px 0 #ff005c; }
      40% { text-shadow: 2px 0 #00ffea, -2px 0 #ff005c; }
      60% { text-shadow: -2px 0 #00ffea, 2px 0 #ff005c; }
      80% { text-shadow: 2px 0 #00ffea, -2px 0 #ff005c; }
      100% { text-shadow: -2px 0 #00ffea, 2px 0 #ff005c; }
    }
    @keyframes glitchTop {
      0% { clip: rect(0, 9999px, 0, 0); transform: translate(0); }
      20% { clip: rect(0, 9999px, 10px, 0); transform: translate(-2px, -2px); }
      40% { clip: rect(0, 9999px, 0, 0); transform: translate(0); }
      60% { clip: rect(0, 9999px, 10px, 0); transform: translate(-2px, -2px); }
      80% { clip: rect(0, 9999px, 0, 0); transform: translate(0); }
      100% { clip: rect(0, 9999px, 10px, 0); transform: translate(-2px, -2px); }
    }
    @keyframes glitchBottom {
      0% { clip: rect(0, 9999px, 0, 0); transform: translate(0); }
      20% { clip: rect(5px, 9999px, 15px, 0); transform: translate(2px, 2px); }
      40% { clip: rect(0, 9999px, 0, 0); transform: translate(0); }
      60% { clip: rect(5px, 9999px, 15px, 0); transform: translate(2px, 2px); }
      80% { clip: rect(0, 9999px, 0, 0); transform: translate(0); }
      100% { clip: rect(5px, 9999px, 15px, 0); transform: translate(2px, 2px); }
    }
    @keyframes shake {
      0% { transform: translate(0, 0); }
      20% { transform: translate(-1px, 1px); }
      40% { transform: translate(-1px, -1px); }
      60% { transform: translate(1px, 1px); }
      80% { transform: translate(1px, -1px); }
      100% { transform: translate(0, 0); }
    }
  </style>
</head>
<body>
  <div class="audio-status" id="status">🔇 SEM SOM</div>
  
  <div class="glitch" data-text="ESCUTE" id="titulo">ESCUTE</div>
  
  <div class="instructions">
    Clique no texto para ativar o som
  </div>
  
  <!-- Áudio com Web Audio API gerando um som de vento sintético -->
  <script>
    let audioContext;
    let whiteNoise;
    let isPlaying = false;
    
    const titulo = document.getElementById('titulo');
    const status = document.getElementById('status');
    
    // Função para criar ruído branco (simulando vento)
    function createWhiteNoise() {
      const bufferSize = 2 * audioContext.sampleRate;
      const noiseBuffer = audioContext.createBuffer(1, bufferSize, audioContext.sampleRate);
      const output = noiseBuffer.getChannelData(0);
      
      for (let i = 0; i < bufferSize; i++) {
        output[i] = Math.random() * 2 - 1;
      }
      
      return noiseBuffer;
    }
    
    // Função para criar filtro de vento
    function createWindFilter(source) {
      const lowpass = audioContext.createBiquadFilter();
      lowpass.type = 'lowpass';
      lowpass.frequency.value = 1000;
      lowpass.Q.value = 0.5;
      
      const highpass = audioContext.createBiquadFilter();
      highpass.type = 'highpass';
      highpass.frequency.value = 100;
      
      const gain = audioContext.createGain();
      gain.gain.value = 0.1;
      
      // LFO para criar variação no volume (rajadas de vento)
      const lfo = audioContext.createOscillator();
      const lfoGain = audioContext.createGain();
      lfo.frequency.value = 0.5;
      lfoGain.gain.value = 0.03;
      
      lfo.connect(lfoGain);
      lfoGain.connect(gain.gain);
      lfo.start();
      
      // Conectar tudo
      source.connect(highpass);
      highpass.connect(lowpass);
      lowpass.connect(gain);
      gain.connect(audioContext.destination);
      
      return { source, lowpass, highpass, gain, lfo };
    }
    
    // Função para tocar o som
    async function playWind() {
      try {
        if (!audioContext) {
          audioContext = new (window.AudioContext || window.webkitAudioContext)();
        }
        
        if (audioContext.state === 'suspended') {
          await audioContext.resume();
        }
        
        if (isPlaying) {
          // Parar o som
          if (whiteNoise && whiteNoise.source) {
            whiteNoise.source.stop();
            whiteNoise.lfo.stop();
          }
          isPlaying = false;
          status.textContent = '🔇 SEM SOM';
          status.className = 'audio-status paused';
          titulo.style.transform = 'scale(1)';
          return;
        }
        
        // Criar e tocar o som
        const noiseBuffer = createWhiteNoise();
        const source = audioContext.createBufferSource();
        source.buffer = noiseBuffer;
        source.loop = true;
        
        whiteNoise = createWindFilter(source);
        source.start(0);
        
        isPlaying = true;
        status.textContent = '🌪️ VENTO ATIVO';
        status.className = 'audio-status playing';
        titulo.style.transform = 'scale(1.05)';
        titulo.style.transition = 'transform 0.3s ease';
        
      } catch (error) {
        console.log('Erro ao reproduzir áudio:', error);
        status.textContent = '❌ ERRO NO ÁUDIO';
        status.className = 'audio-status paused';
      }
    }
    
    // Event listeners
    titulo.addEventListener('click', playWind);
    
    // Também permitir ativação com espaço ou enter
    document.addEventListener('keydown', (e) => {
      if (e.code === 'Space' || e.code === 'Enter') {
        e.preventDefault();
        playWind();
      }
    });
    
    // Feedback visual no hover
    titulo.addEventListener('mouseenter', () => {
      if (!isPlaying) {
        titulo.style.transform = 'scale(1.02)';
        titulo.style.transition = 'transform 0.2s ease';
      }
    });
    
    titulo.addEventListener('mouseleave', () => {
      if (!isPlaying) {
        titulo.style.transform = 'scale(1)';
      }
    });
  </script>
</body>
</html>
