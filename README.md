<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Wheel of Names</title>
  <style>
    :root {
      --bg-color: #0f172a;
      --card-bg: #1e293b;
      --card-border: #334155;
      --text-main: #f8fafc;
      --text-muted: #94a3b8;
      --accent-primary: #6366f1;
      --accent-hover: #4f46e5;
      --accent-danger: #ef4444;
      --accent-danger-hover: #dc2626;
      --shadow-color: rgba(0, 0, 0, 0.35);
    }

    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    body {
      font-family: system-ui, -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
      background-color: var(--bg-color);
      color: var(--text-main);
      min-height: 100vh;
      display: flex;
      flex-direction: column;
    }

    header {
      padding: 1.25rem 2rem;
      border-bottom: 1px solid var(--card-border);
      background-color: var(--card-bg);
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    header h1 {
      font-size: 1.5rem;
      font-weight: 700;
      background: linear-gradient(135deg, #818cf8, #c084fc);
      -webkit-background-clip: text;
      -webkit-text-fill-color: transparent;
    }

    .app-container {
      display: grid;
      grid-template-columns: 360px 1fr;
      flex: 1;
      height: calc(100vh - 65px);
      overflow: hidden;
    }

    /* Sidebar Styling */
    .sidebar {
      background-color: var(--card-bg);
      border-right: 1px solid var(--card-border);
      padding: 1.5rem;
      display: flex;
      flex-direction: column;
      gap: 1rem;
      overflow-y: auto;
    }

    .sidebar-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    .sidebar-header h2 {
      font-size: 1.1rem;
      color: var(--text-main);
    }

    .name-count {
      font-size: 0.85rem;
      background-color: var(--card-border);
      padding: 0.2rem 0.6rem;
      border-radius: 12px;
      color: var(--text-muted);
    }

    textarea {
      width: 100%;
      flex: 1;
      min-height: 200px;
      background-color: var(--bg-color);
      border: 1px solid var(--card-border);
      border-radius: 8px;
      padding: 0.75rem;
      color: var(--text-main);
      font-family: inherit;
      font-size: 0.95rem;
      line-height: 1.5;
      resize: none;
      outline: none;
      transition: border-color 0.2s;
    }

    textarea:focus {
      border-color: var(--accent-primary);
    }

    .action-buttons {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 0.5rem;
    }

    .btn {
      background-color: var(--card-border);
      color: var(--text-main);
      border: none;
      padding: 0.6rem 0.8rem;
      border-radius: 6px;
      font-weight: 600;
      font-size: 0.85rem;
      cursor: pointer;
      transition: background-color 0.2s, transform 0.1s;
    }

    .btn:hover {
      background-color: #475569;
    }

    .btn:active {
      transform: scale(0.98);
    }

    /* Wheel Canvas Section */
    .wheel-section {
      position: relative;
      display: flex;
      justify-content: center;
      align-items: center;
      padding: 2rem;
      overflow: hidden;
    }

    .canvas-wrapper {
      position: relative;
      width: 100%;
      max-width: 580px;
      aspect-ratio: 1;
      display: flex;
      justify-content: center;
      align-items: center;
    }

    canvas {
      width: 100%;
      height: 100%;
      border-radius: 50%;
      box-shadow: 0 10px 30px var(--shadow-color);
    }

    /* Top Indicator Arrow */
    .pointer {
      position: absolute;
      top: -12px;
      left: 50%;
      transform: translateX(-50%);
      width: 0;
      height: 0;
      border-left: 18px solid transparent;
      border-right: 18px solid transparent;
      border-top: 32px solid #f43f5e;
      z-index: 10;
      filter: drop-shadow(0 4px 6px rgba(0,0,0,0.4));
    }

    /* Center Spin Button */
    .spin-btn {
      position: absolute;
      top: 50%;
      left: 50%;
      transform: translate(-50%, -50%);
      width: 86px;
      height: 86px;
      border-radius: 50%;
      background: linear-gradient(135deg, #6366f1, #4f46e5);
      color: white;
      border: 4px solid #ffffff;
      font-weight: 800;
      font-size: 1.1rem;
      cursor: pointer;
      box-shadow: 0 4px 20px rgba(0, 0, 0, 0.4);
      z-index: 5;
      transition: transform 0.2s, box-shadow 0.2s, background 0.2s;
      user-select: none;
    }

    .spin-btn:hover:not(:disabled) {
      transform: translate(-50%, -50%) scale(1.08);
      box-shadow: 0 6px 24px rgba(99, 102, 241, 0.6);
    }

    .spin-btn:disabled {
      background: #64748b;
      cursor: not-allowed;
      transform: translate(-50%, -50%);
    }

    /* Modal Styling */
    .modal-overlay {
      position: fixed;
      inset: 0;
      background-color: rgba(15, 23, 42, 0.75);
      backdrop-filter: blur(4px);
      display: flex;
      justify-content: center;
      align-items: center;
      opacity: 0;
      pointer-events: none;
      transition: opacity 0.3s ease;
      z-index: 100;
    }

    .modal-overlay.active {
      opacity: 1;
      pointer-events: auto;
    }

    .modal-card {
      background-color: var(--card-bg);
      border: 1px solid var(--card-border);
      border-radius: 16px;
      padding: 2.5rem;
      width: 90%;
      max-width: 420px;
      text-align: center;
      box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.5);
      transform: scale(0.9);
      transition: transform 0.3s ease;
    }

    .modal-overlay.active .modal-card {
      transform: scale(1);
    }

    .modal-card h3 {
      color: var(--text-muted);
      font-size: 0.95rem;
      text-transform: uppercase;
      letter-spacing: 0.1em;
      margin-bottom: 0.5rem;
    }

    .modal-card .winner-name {
      font-size: 2.2rem;
      font-weight: 800;
      color: #38bdf8;
      word-break: break-word;
      margin-bottom: 1.5rem;
    }

    .modal-actions {
      display: flex;
      gap: 0.75rem;
    }

    .modal-actions .btn {
      flex: 1;
      padding: 0.75rem;
      font-size: 0.95rem;
    }

    .btn-danger {
      background-color: var(--accent-danger);
    }

    .btn-danger:hover {
      background-color: var(--accent-danger-hover);
    }

    .btn-primary {
      background-color: var(--accent-primary);
    }

    .btn-primary:hover {
      background-color: var(--accent-hover);
    }

    /* Confetti Overlay Canvas */
    #confettiCanvas {
      position: fixed;
      inset: 0;
      pointer-events: none;
      z-index: 99;
    }

    /* Responsive Design */
    @media (max-width: 850px) {
      .app-container {
        grid-template-columns: 1fr;
        grid-template-rows: auto 1fr;
        height: auto;
        overflow: visible;
      }

      .sidebar {
        border-right: none;
        border-bottom: 1px solid var(--card-border);
      }

      textarea {
        min-height: 120px;
      }

      .canvas-wrapper {
        max-width: 380px;
      }
    }
  </style>
</head>
<body>

  <header>
    <h1>Wheel of Names</h1>
    <span style="font-size: 0.85rem; color: var(--text-muted);">Press <strong>Spacebar</strong> to spin</span>
  </header>

  <div class="app-container">
    <!-- Sidebar Left -->
    <section class="sidebar">
      <div class="sidebar-header">
        <h2>List of Names</h2>
        <span class="name-count" id="nameCount">0 names</span>
      </div>

      <textarea id="namesInput" placeholder="Enter names here (one per line)..." spellcheck="false"></textarea>

      <div class="action-buttons">
        <button class="btn" id="shuffleBtn">Shuffle</button>
        <button class="btn" id="sortBtn">Sort</button>
        <button class="btn" id="clearBtn">Clear</button>
      </div>
    </section>

    <!-- Wheel Section Right -->
    <main class="wheel-section">
      <div class="canvas-wrapper">
        <div class="pointer"></div>
        <canvas id="wheelCanvas"></canvas>
        <button class="spin-btn" id="spinBtn">SPIN</button>
      </div>
    </main>
  </div>

  <!-- Winner Modal -->
  <div class="modal-overlay" id="winnerModal">
    <div class="modal-card">
      <h3>We have a winner!</h3>
      <div class="winner-name" id="winnerNameDisplay">---</div>
      <div class="modal-actions">
        <button class="btn btn-danger" id="removeWinnerBtn">Remove Name</button>
        <button class="btn btn-primary" id="closeModalBtn">Keep / Close</button>
      </div>
    </div>
  </div>

  <canvas id="confettiCanvas"></canvas>

  <script>
    // --- Application State ---
    const PALETTE = [
      '#EF4444', '#F97316', '#F59E0B', '#10B981', 
      '#06B6D4', '#3B82F6', '#6366F1', '#8B5CF6', 
      '#EC4899', '#14B8A6', '#84CC16', '#A855F7'
    ];

    let names = [];
    let currentRotation = 0; // In radians
    let isSpinning = false;
    let winningIndex = -1;

    // --- DOM Elements ---
    const namesInput = document.getElementById('namesInput');
    const nameCount = document.getElementById('nameCount');
    const shuffleBtn = document.getElementById('shuffleBtn');
    const sortBtn = document.getElementById('sortBtn');
    const clearBtn = document.getElementById('clearBtn');
    const spinBtn = document.getElementById('spinBtn');
    const canvas = document.getElementById('wheelCanvas');
    const ctx = canvas.getContext('2d');

    const winnerModal = document.getElementById('winnerModal');
    const winnerNameDisplay = document.getElementById('winnerNameDisplay');
    const removeWinnerBtn = document.getElementById('removeWinnerBtn');
    const closeModalBtn = document.getElementById('closeModalBtn');

    // --- Initial Setup ---
    const defaultNames = ["Alice", "Bob", "Charlie", "Diana", "Ethan", "Fiona", "George", "Hannah"];
    namesInput.value = defaultNames.join('\n');

    // --- High DPI Canvas Resize ---
    function resizeCanvas() {
      const rect = canvas.getBoundingClientRect();
      const dpr = window.devicePixelRatio || 1;
      canvas.width = rect.width * dpr;
      canvas.height = rect.height * dpr;
      ctx.scale(dpr, dpr);
      drawWheel();
    }

    window.addEventListener('resize', resizeCanvas);

    // --- Name List Handling ---
    function updateNamesFromInput() {
      names = namesInput.value
        .split('\n')
        .map(n => n.trim())
        .filter(n => n.length > 0);

      nameCount.textContent = `${names.length} name${names.length === 1 ? '' : 's'}`;
      spinBtn.disabled = isSpinning || names.length === 0;
      drawWheel();
    }

    namesInput.addEventListener('input', updateNamesFromInput);

    shuffleBtn.addEventListener('click', () => {
      if (isSpinning) return;
      for (let i = names.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [names[i], names[j]] = [names[j], names[i]];
      }
      namesInput.value = names.join('\n');
      updateNamesFromInput();
    });

    sortBtn.addEventListener('click', () => {
      if (isSpinning) return;
      names.sort((a, b) => a.localeCompare(b, undefined, { sensitivity: 'base' }));
      namesInput.value = names.join('\n');
      updateNamesFromInput();
    });

    clearBtn.addEventListener('click', () => {
      if (isSpinning) return;
      namesInput.value = '';
      updateNamesFromInput();
    });

    // --- Wheel Rendering ---
    function drawWheel() {
      const rect = canvas.getBoundingClientRect();
      const width = rect.width;
      const height = rect.height;
      const centerX = width / 2;
      const centerY = height / 2;
      const radius = Math.min(centerX, centerY) - 8;

      ctx.clearRect(0, 0, width, height);

      if (names.length === 0) {
        // Draw empty state
        ctx.beginPath();
        ctx.arc(centerX, centerY, radius, 0, 2 * Math.PI);
        ctx.fillStyle = '#334155';
        ctx.fill();
        ctx.lineWidth = 4;
        ctx.strokeStyle = '#475569';
        ctx.stroke();

        ctx.fillStyle = '#94a3b8';
        ctx.font = '600 16px system-ui';
        ctx.textAlign = 'center';
        ctx.textBaseline = 'middle';
        ctx.fillText('Add names to spin!', centerX, centerY);
        return;
      }

      const sliceAngle = (2 * Math.PI) / names.length;

      names.forEach((name, i) => {
        const startAngle = currentRotation + i * sliceAngle;
        const endAngle = startAngle + sliceAngle;

        // Choose color (ensure last item doesn't duplicate first if odd wrap)
        let colorIndex = i % PALETTE.length;
        if (i === names.length - 1 && colorIndex === 0 && names.length > 1) {
          colorIndex = 1;
        }

        // Draw Slice
        ctx.beginPath();
        ctx.moveTo(centerX, centerY);
        ctx.arc(centerX, centerY, radius, startAngle, endAngle);
        ctx.closePath();
        ctx.fillStyle = PALETTE[colorIndex];
        ctx.fill();
        ctx.lineWidth = 2;
        ctx.strokeStyle = '#0f172a';
        ctx.stroke();

        // Draw Text
        ctx.save();
        ctx.translate(centerX, centerY);
        ctx.rotate(startAngle + sliceAngle / 2);
        ctx.textAlign = 'right';
        ctx.textBaseline = 'middle';
        ctx.fillStyle = '#FFFFFF';

        // Scale font dynamically based on total slices
        const fontSize = Math.min(20, Math.max(11, Math.floor(260 / names.length)));
        ctx.font = `700 ${fontSize}px system-ui`;

        // Truncate text if necessary
        const maxTextWidth = radius * 0.65;
        let formattedName = name;
        if (ctx.measureText(formattedName).width > maxTextWidth) {
          while (formattedName.length > 0 && ctx.measureText(formattedName + '…').width > maxTextWidth) {
            formattedName = formattedName.slice(0, -1);
          }
          formattedName += '…';
        }

        ctx.fillText(formattedName, radius - 24, 0);
        ctx.restore();
      });

      // Outer rim accent
      ctx.beginPath();
      ctx.arc(centerX, centerY, radius, 0, 2 * Math.PI);
      ctx.lineWidth = 6;
      ctx.strokeStyle = '#1e293b';
      ctx.stroke();
    }

    // --- Spin Logic & Mathematics ---
    function spin() {
      if (isSpinning || names.length === 0) return;

      isSpinning = true;
      spinBtn.disabled = true;

      const spinDuration = 4000; // 4 seconds
      const startTime = performance.now();
      const startRotation = currentRotation;

      // Calculate a random target slice
      const sliceAngle = (2 * Math.PI) / names.length;
      const chosenSlice = Math.floor(Math.random() * names.length);
      
      // Calculate landing angle so pointer at top (12 o'clock = 1.5 * PI) points directly into the chosen slice
      const targetSliceCenter = (chosenSlice + 0.5) * sliceAngle;
      const pointerAngle = 1.5 * Math.PI;
      
      // Compute final offset within 2*PI radians
      let targetAngleOffset = (pointerAngle - targetSliceCenter) % (2 * Math.PI);
      if (targetAngleOffset < 0) targetAngleOffset += 2 * Math.PI;

      // Add 5 to 8 full revolutions for excitement
      const extraRotations = (5 + Math.floor(Math.random() * 4)) * 2 * Math.PI;
      
      // Align final rotation strictly to targetAngleOffset
      const currentMod = startRotation % (2 * Math.PI);
      const totalRotationNeeded = extraRotations + (targetAngleOffset - currentMod + 2 * Math.PI) % (2 * Math.PI);
      const targetRotation = startRotation + totalRotationNeeded;

      // Animation Loop with Easing (Ease-Out Quart)
      function animate(now) {
        const elapsed = now - startTime;
        const progress = Math.min(elapsed / spinDuration, 1);
        
        // Easing formula
        const easeOut = 1 - Math.pow(1 - progress, 4);

        currentRotation = startRotation + (targetRotation - startRotation) * easeOut;
        drawWheel();

        if (progress < 1) {
          requestAnimationFrame(animate);
        } else {
          isSpinning = false;
          spinBtn.disabled = false;
          calculateWinner();
          showWinnerModal();
        }
      }

      requestAnimationFrame(animate);
    }

    function calculateWinner() {
      const sliceAngle = (2 * Math.PI) / names.length;
      
      // Normalize wheel rotation to [0, 2*PI)
      const normalizedRotation = ((currentRotation % (2 * Math.PI)) + 2 * Math.PI) % (2 * Math.PI);
      
      // The pointer is at 12 o'clock (1.5 * PI = 270 deg)
      const angleAtPointer = ((1.5 * Math.PI - normalizedRotation) % (2 * Math.PI) + 2 * Math.PI) % (2 * Math.PI);
      
      winningIndex = Math.floor(angleAtPointer / sliceAngle) % names.length;
    }

    // --- Modal & Celebratory Confetti ---
    function showWinnerModal() {
      winnerNameDisplay.textContent = names[winningIndex];
      winnerModal.classList.add('active');
      triggerConfetti();
    }

    function hideWinnerModal() {
      winnerModal.classList.remove('active');
    }

    removeWinnerBtn.addEventListener('click', () => {
      if (winningIndex >= 0 && winningIndex < names.length) {
        names.splice(winningIndex, 1);
        namesInput.value = names.join('\n');
        updateNamesFromInput();
      }
      hideWinnerModal();
    });

    closeModalBtn.addEventListener('click', hideWinnerModal);

    // Keybindings (Spacebar to spin)
    window.addEventListener('keydown', (e) => {
      if (e.code === 'Space' && document.activeElement !== namesInput) {
        e.preventDefault();
        spin();
      }
    });

    spinBtn.addEventListener('click', spin);

    // --- Built-in Particle Confetti Generator ---
    const confettiCanvas = document.getElementById('confettiCanvas');
    const cCtx = confettiCanvas.getContext('2d');
    let confettiParticles = [];

    function triggerConfetti() {
      confettiCanvas.width = window.innerWidth;
      confettiCanvas.height = window.innerHeight;
      confettiParticles = [];

      for (let i = 0; i < 90; i++) {
        confettiParticles.push({
          x: window.innerWidth / 2,
          y: window.innerHeight / 2 - 50,
          vx: (Math.random() - 0.5) * 18,
          vy: (Math.random() - 0.7) * 18,
          size: Math.random() * 8 + 4,
          color: PALETTE[Math.floor(Math.random() * PALETTE.length)],
          rotation: Math.random() * 360,
          rSpeed: (Math.random() - 0.5) * 10,
          opacity: 1
        });
      }
      requestAnimationFrame(updateConfetti);
    }

    function updateConfetti() {
      cCtx.clearRect(0, 0, confettiCanvas.width, confettiCanvas.height);
      let alive = false;

      confettiParticles.forEach(p => {
        p.x += p.vx;
        p.y += p.vy;
        p.vy += 0.35; // gravity
        p.opacity -= 0.012;
        p.rotation += p.rSpeed;

        if (p.opacity > 0) {
          alive = true;
          cCtx.save();
          cCtx.translate(p.x, p.y);
          cCtx.rotate((p.rotation * Math.PI) / 180);
          cCtx.globalAlpha = Math.max(0, p.opacity);
          cCtx.fillStyle = p.color;
          cCtx.fillRect(-p.size / 2, -p.size / 2, p.size, p.size);
          cCtx.restore();
        }
      });

      if (alive) {
        requestAnimationFrame(updateConfetti);
      }
    }

    // Initialize application
    updateNamesFromInput();
    resizeCanvas();
  </script>
</body>
</html>
# Bbayon.github.io
