# Simulations
Html Dateien als Lernstationen/Animationen zur Schülernutzung
<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Brownsche Molekularbewegung</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/react/18.2.0/umd/react.production.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/react-dom/18.2.0/umd/react-dom.production.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/babel-standalone/7.23.2/babel.min.js"></script>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body { background: linear-gradient(135deg, #f8fbff 0%, #eaf4ff 100%); min-height: 100vh; }
  </style>
</head>
<body>
  <div id="root"></div>
  <script type="text/babel">
    const WATER_COLOR = "#60b8f5";
    const SUGAR_COLOR = "#f5a623";
    const WATER_RADIUS_NM = 0.14;
    const SUGAR_RADIUS_NM = 0.55;
    const SCALE = 120;

    function lerp(a, b, t) { return a + (b - a) * t; }
    function randomBetween(a, b) { return a + Math.random() * (b - a); }

    function initParticles(canvasW, canvasH, radiusMode, scaleMode) {
      const waterR = scaleMode === "realistisch"
        ? WATER_RADIUS_NM * SCALE
        : (radiusMode === "klein" ? 6 : radiusMode === "mittel" ? 9 : 13);
      const sugarR = scaleMode === "realistisch"
        ? SUGAR_RADIUS_NM * SCALE
        : waterR * (SUGAR_RADIUS_NM / WATER_RADIUS_NM);

      const cx = canvasW / 2, cy = canvasH / 2;
      const particles = [];
      const sugarZone = sugarR * 5;
      let attempts = 0;

      while (particles.filter(p => p.type === "sugar").length < 18 && attempts < 2000) {
        attempts++;
        const angle = Math.random() * Math.PI * 2;
        const dist = randomBetween(0, sugarZone);
        const x = cx + Math.cos(angle) * dist;
        const y = cy + Math.sin(angle) * dist;
        if (x - sugarR < 0 || x + sugarR > canvasW || y - sugarR < 0 || y + sugarR > canvasH) continue;
        const overlap = particles.some(p => {
          const r = p.type === "sugar" ? sugarR : waterR;
          return Math.hypot(p.x - x, p.y - y) < r + sugarR + 1;
        });
        if (!overlap) particles.push({ type: "sugar", x, y });
      }

      attempts = 0;
      while (particles.filter(p => p.type === "water").length < 80 && attempts < 5000) {
        attempts++;
        const x = randomBetween(waterR, canvasW - waterR);
        const y = randomBetween(waterR, canvasH - waterR);
        const distCenter = Math.hypot(x - cx, y - cy);
        if (distCenter < sugarZone + waterR + 2) continue;
        const overlap = particles.some(p => {
          const r = p.type === "sugar" ? sugarR : waterR;
          return Math.hypot(p.x - x, p.y - y) < r + waterR + 1;
        });
        if (!overlap) particles.push({ type: "water", x, y });
      }

      return { particles, waterR, sugarR };
    }

    function getSpeed(temp) { return 0.3 + (temp / 100) * 3.2; }

    function stepParticles(particles, waterR, sugarR, speed, canvasW, canvasH) {
      return particles.map(p => {
        const r = p.type === "sugar" ? sugarR : waterR;
        const angle = Math.random() * Math.PI * 2;
        const factor = p.type === "sugar" ? 0.3 : 1.0;
        let nx = p.x + Math.cos(angle) * speed * factor;
        let ny = p.y + Math.sin(angle) * speed * factor;
        if (nx - r < 0) nx = r;
        if (nx + r > canvasW) nx = canvasW - r;
        if (ny - r < 0) ny = r;
        if (ny + r > canvasH) ny = canvasH - r;
        return { ...p, x: nx, y: ny };
      });
    }

    function MacroView({ progress, width, height }) {
      const canvasRef = React.useRef(null);
      React.useEffect(() => {
        const canvas = canvasRef.current;
        if (!canvas) return;
        const ctx = canvas.getContext("2d");
        ctx.clearRect(0, 0, width, height);
        ctx.fillStyle = "#d6eeff";
        ctx.fillRect(0, 0, width, height);
        ctx.strokeStyle = "#b0d8f8";
        ctx.lineWidth = 1;
        for (let i = 0; i < 8; i++) {
          ctx.beginPath();
          ctx.moveTo(0, 30 + i * (height / 8));
          ctx.bezierCurveTo(width * 0.3, 20 + i * (height / 8) + Math.sin(progress * 0.05 + i) * 6,
            width * 0.7, 40 + i * (height / 8) + Math.cos(progress * 0.04 + i) * 6,
            width, 30 + i * (height / 8));
          ctx.stroke();
        }
        const cubeSize = lerp(80, 4, Math.min(progress / 100, 1));
        const cx = width / 2, cy = height / 2;
        if (cubeSize > 5) {
          ctx.fillStyle = "#fff8e7";
          ctx.strokeStyle = "#e8c97a";
          ctx.lineWidth = 2;
          ctx.beginPath();
          ctx.rect(cx - cubeSize / 2, cy - cubeSize / 2, cubeSize, cubeSize);
          ctx.fill();
          ctx.stroke();
          if (cubeSize > 20) {
            ctx.strokeStyle = "#e8c97a44";
            ctx.lineWidth = 1;
            for (let i = 1; i < 4; i++) {
              ctx.beginPath();
              ctx.moveTo(cx - cubeSize / 2 + (cubeSize / 4) * i, cy - cubeSize / 2);
              ctx.lineTo(cx - cubeSize / 2 + (cubeSize / 4) * i, cy + cubeSize / 2);
              ctx.stroke();
              ctx.beginPath();
              ctx.moveTo(cx - cubeSize / 2, cy - cubeSize / 2 + (cubeSize / 4) * i);
              ctx.lineTo(cx + cubeSize / 2, cy - cubeSize / 2 + (cubeSize / 4) * i);
              ctx.stroke();
            }
          }
        }
        const numDissolved = Math.floor(progress * 0.5);
        for (let i = 0; i < numDissolved; i++) {
          const angle = (i / 30) * Math.PI * 2 + i * 2.4;
          const dist = 20 + (i % 15) * 8 + progress * 0.3;
          const px = cx + Math.cos(angle) * dist;
          const py = cy + Math.sin(angle) * dist;
          if (px < 4 || px > width - 4 || py < 4 || py > height - 4) continue;
          const alpha = Math.max(0, 1 - dist / (width * 0.55));
          ctx.globalAlpha = alpha * 0.8;
          ctx.fillStyle = SUGAR_COLOR;
          ctx.beginPath();
          ctx.arc(px, py, 3, 0, Math.PI * 2);
          ctx.fill();
        }
        ctx.globalAlpha = 1;
      }, [progress, width, height]);
      return React.createElement("canvas", { ref: canvasRef, width, height, style: { borderRadius: 12, display: "block" } });
    }

    const erklaerung = [
      {
        emoji: "🔬",
        frage: "Was ist Brownsche Bewegung?",
        text: "Wenn du sehr kleine Teilchen unter dem Mikroskop beobachtest, tanzen sie wild durcheinander – obwohl niemand rührt! Das nennt man Brownsche Bewegung. Die Ursache: Unsichtbare Wasserteilchen stoßen von allen Seiten gegen die größeren Teilchen und verschieben sie dabei zufällig."
      },
      {
        emoji: "🍬",
        frage: "Warum löst sich Zucker auf?",
        text: "Zucker besteht aus vielen kleinen Zuckerteilchen (Saccharose), die fest zusammenhalten. Im Wasser stoßen die Wasserteilchen so oft und so kräftig gegen die Zuckerteilchen, dass diese sich langsam vom Würfel lösen und sich im Wasser verteilen – ganz ohne Rühren!"
      },
      {
        emoji: "🌡️",
        frage: "Was hat Temperatur damit zu tun?",
        text: "Wärme bedeutet: Die Teilchen bewegen sich schneller! Bei höherer Temperatur stoßen die Wasserteilchen häufiger und stärker gegen die Zuckerteilchen. Deshalb löst sich Zucker in warmem Wasser viel schneller auf als in kaltem. Probier es aus: Stell den Temperaturregler auf 100°C!"
      },
      {
        emoji: "⚛️",
        frage: "Was zeigt das Teilchenmodell?",
        text: "Das Teilchenmodell stellt Moleküle als kleine Kugeln dar. Blaue Kugeln sind Wassermoleküle (H₂O), orangene sind Zuckermoleküle (Saccharose). Im realistischen Maßstab siehst du: Zuckerteilchen sind fast 4× größer als Wasserteilchen. Weil die Wasserteilchen so klein und beweglich sind, können sie überall hin – und nehmen die Zuckerteilchen mit."
      }
    ];

    function App() {
      const [running, setRunning] = React.useState(false);
      const [temperature, setTemperature] = React.useState(25);
      const [radiusMode, setRadiusMode] = React.useState("mittel");
      const [scaleMode, setScaleMode] = React.useState("vereinfacht");
      const [particles, setParticles] = React.useState([]);
      const [waterR, setWaterR] = React.useState(9);
      const [sugarR, setSugarR] = React.useState(35);
      const [macroProgress, setMacroProgress] = React.useState(0);
      const [openSection, setOpenSection] = React.useState(null);
      const animRef = React.useRef(null);
      const stateRef = React.useRef({ particles: [], waterR: 9, sugarR: 35 });
      const microCanvasRef = React.useRef(null);
      const CANVAS_W = 340, CANVAS_H = 300;

      const reset = React.useCallback(() => {
        const { particles: p, waterR: wr, sugarR: sr } = initParticles(CANVAS_W, CANVAS_H, radiusMode, scaleMode);
        setParticles(p);
        setWaterR(wr);
        setSugarR(sr);
        stateRef.current = { particles: p, waterR: wr, sugarR: sr };
        setMacroProgress(0);
        setRunning(false);
      }, [radiusMode, scaleMode]);

      React.useEffect(() => { reset(); }, [reset]);

      React.useEffect(() => {
        const canvas = microCanvasRef.current;
        if (!canvas) return;
        const ctx = canvas.getContext("2d");
        ctx.clearRect(0, 0, CANVAS_W, CANVAS_H);
        ctx.fillStyle = "#f0f8ff";
        ctx.fillRect(0, 0, CANVAS_W, CANVAS_H);
        particles.forEach(p => {
          const r = p.type === "sugar" ? sugarR : waterR;
          ctx.beginPath();
          ctx.arc(p.x, p.y, r, 0, Math.PI * 2);
          ctx.fillStyle = p.type === "sugar" ? SUGAR_COLOR : WATER_COLOR;
          ctx.globalAlpha = 0.85;
          ctx.fill();
          ctx.globalAlpha = 1;
          ctx.strokeStyle = p.type === "sugar" ? "#c8830a" : "#2a8fd4";
          ctx.lineWidth = 1;
          ctx.stroke();
        });
      }, [particles, waterR, sugarR]);

      React.useEffect(() => {
        if (!running) { cancelAnimationFrame(animRef.current); return; }
        let last = performance.now();
        const loop = (now) => {
          const dt = Math.min(now - last, 50);
          last = now;
          const speed = getSpeed(temperature) * (dt / 16);
          const { particles: cur, waterR: wr, sugarR: sr } = stateRef.current;
          const next = stepParticles(cur, wr, sr, speed, CANVAS_W, CANVAS_H);
          stateRef.current.particles = next;
          setParticles([...next]);
          setMacroProgress(p => Math.min(p + speed * 0.15, 100));
          animRef.current = requestAnimationFrame(loop);
        };
        animRef.current = requestAnimationFrame(loop);
        return () => cancelAnimationFrame(animRef.current);
      }, [running, temperature]);

      const tempColor = temperature < 20 ? "#60b8f5" : temperature < 50 ? "#f5a623" : "#e74c3c";
      const tempLabel = temperature < 20 ? "kalt" : temperature < 50 ? "warm" : "heiß";

      return (
        <div style={{ fontFamily: "Georgia, serif", background: "linear-gradient(135deg, #f8fbff 0%, #eaf4ff 100%)", minHeight: "100vh", padding: "24px 16px", display: "flex", flexDirection: "column", alignItems: "center" }}>
          <h1 style={{ fontSize: 22, fontWeight: "bold", color: "#1a3a5c", marginBottom: 4, textAlign: "center", letterSpacing: 0.5 }}>
            Brownsche Molekularbewegung
          </h1>
          <p style={{ color: "#5a7a9a", fontSize: 13, marginBottom: 20, textAlign: "center" }}>
            Zucker löst sich in Wasser auf – makroskopisch und im Teilchenmodell
          </p>

          <div style={{ display: "flex", gap: 16, flexWrap: "wrap", justifyContent: "center", marginBottom: 20 }}>
            <div style={{ background: "#fff", borderRadius: 16, padding: 12, boxShadow: "0 2px 16px #b0d0ee44" }}>
              <div style={{ fontSize: 12, fontWeight: "bold", color: "#1a3a5c", marginBottom: 6, textAlign: "center" }}>🔭 Makroskopisch</div>
              <MacroView progress={macroProgress} width={CANVAS_W} height={CANVAS_H} />
              <div style={{ fontSize: 11, color: "#7a9abc", textAlign: "center", marginTop: 4 }}>Zuckerwürfel löst sich auf ({Math.round(macroProgress)}%)</div>
            </div>
            <div style={{ background: "#fff", borderRadius: 16, padding: 12, boxShadow: "0 2px 16px #b0d0ee44" }}>
              <div style={{ fontSize: 12, fontWeight: "bold", color: "#1a3a5c", marginBottom: 6, textAlign: "center" }}>🔬 Teilchenmodell</div>
              <canvas ref={microCanvasRef} width={CANVAS_W} height={CANVAS_H} style={{ borderRadius: 12, display: "block" }} />
              <div style={{ display: "flex", gap: 16, justifyContent: "center", marginTop: 6 }}>
                <span style={{ fontSize: 11, color: "#5a7a9a", display: "flex", alignItems: "center", gap: 4 }}>
                  <span style={{ width: 12, height: 12, borderRadius: "50%", background: WATER_COLOR, display: "inline-block", border: "1px solid #2a8fd4" }} /> Wasserteilchen
                </span>
                <span style={{ fontSize: 11, color: "#5a7a9a", display: "flex", alignItems: "center", gap: 4 }}>
                  <span style={{ width: 12, height: 12, borderRadius: "50%", background: SUGAR_COLOR, display: "inline-block", border: "1px solid #c8830a" }} /> Zuckerteilchen
                </span>
              </div>
              <div style={{ fontSize: 11, color: "#aaa", textAlign: "center", marginTop: 2 }}>
                Maßstab: {scaleMode} | Wasser: {waterR.toFixed(1)}px · Zucker: {sugarR.toFixed(1)}px
              </div>
            </div>
          </div>

          <div style={{ background: "#fff", borderRadius: 16, padding: "16px 24px", boxShadow: "0 2px 16px #b0d0ee44", width: "100%", maxWidth: 720, display: "flex", flexWrap: "wrap", gap: 16, alignItems: "flex-end", justifyContent: "center", marginBottom: 16 }}>
            <div style={{ flex: "1 1 180px" }}>
              <label style={{ fontSize: 12, fontWeight: "bold", color: "#1a3a5c", display: "block", marginBottom: 4 }}>
                🌡️ Temperatur: <span style={{ color: tempColor }}>{temperature}°C ({tempLabel})</span>
              </label>
              <input type="range" min={0} max={100} value={temperature} onChange={e => setTemperature(Number(e.target.value))} style={{ width: "100%", accentColor: tempColor }} />
              <div style={{ display: "flex", justifyContent: "space-between", fontSize: 10, color: "#aaa" }}>
                <span>0°C</span><span>50°C</span><span>100°C</span>
              </div>
            </div>
            <div style={{ flex: "1 1 160px" }}>
              <label style={{ fontSize: 12, fontWeight: "bold", color: "#1a3a5c", display: "block", marginBottom: 6 }}>⭕ Teilchengröße</label>
              <div style={{ display: "flex", gap: 6 }}>
                {["klein", "mittel", "groß"].map(m => (
                  <button key={m} onClick={() => setRadiusMode(m)} style={{ flex: 1, padding: "5px 0", borderRadius: 8, border: "none", cursor: "pointer", background: radiusMode === m ? "#1a3a5c" : "#e8f0f8", color: radiusMode === m ? "#fff" : "#1a3a5c", fontWeight: radiusMode === m ? "bold" : "normal", fontSize: 12 }}>{m}</button>
                ))}
              </div>
            </div>
            <div style={{ flex: "1 1 160px" }}>
              <label style={{ fontSize: 12, fontWeight: "bold", color: "#1a3a5c", display: "block", marginBottom: 6 }}>📏 Maßstab</label>
              <div style={{ display: "flex", gap: 6 }}>
                {["vereinfacht", "realistisch"].map(m => (
                  <button key={m} onClick={() => setScaleMode(m)} style={{ flex: 1, padding: "5px 0", borderRadius: 8, border: "none", cursor: "pointer", background: scaleMode === m ? "#1a3a5c" : "#e8f0f8", color: scaleMode === m ? "#fff" : "#1a3a5c", fontWeight: scaleMode === m ? "bold" : "normal", fontSize: 11 }}>{m}</button>
                ))}
              </div>
            </div>
            <div style={{ flex: "1 1 160px", display: "flex", gap: 8 }}>
              <button onClick={() => setRunning(r => !r)} style={{ flex: 1, padding: "8px 0", borderRadius: 10, border: "none", cursor: "pointer", background: running ? "#e74c3c" : "#27ae60", color: "#fff", fontWeight: "bold", fontSize: 13 }}>
                {running ? "⏸ Pause" : "▶ Start"}
              </button>
              <button onClick={reset} style={{ flex: 1, padding: "8px 0", borderRadius: 10, border: "none", cursor: "pointer", background: "#e8f0f8", color: "#1a3a5c", fontWeight: "bold", fontSize: 13 }}>
                🔄 Reset
              </button>
            </div>
          </div>

          <div style={{ width: "100%", maxWidth: 720 }}>
            <div style={{ fontSize: 13, fontWeight: "bold", color: "#1a3a5c", marginBottom: 8 }}>
              📖 Erklärungen – klicke auf eine Frage!
            </div>
            <div style={{ display: "flex", flexDirection: "column", gap: 8 }}>
              {erklaerung.map((item, i) => {
                const isOpen = openSection === i;
                return (
                  <div key={i} style={{ background: "#fff", borderRadius: 12, boxShadow: "0 2px 10px #b0d0ee33", overflow: "hidden", border: isOpen ? "1.5px solid #60b8f5" : "1.5px solid transparent" }}>
                    <button onClick={() => setOpenSection(isOpen ? null : i)} style={{ width: "100%", background: "none", border: "none", padding: "12px 16px", display: "flex", justifyContent: "space-between", alignItems: "center", cursor: "pointer", textAlign: "left" }}>
                      <span style={{ fontSize: 13, fontWeight: "bold", color: "#1a3a5c" }}>{item.emoji} {item.frage}</span>
                      <span style={{ fontSize: 16, color: "#60b8f5", display: "inline-block", transform: isOpen ? "rotate(180deg)" : "rotate(0deg)" }}>▼</span>
                    </button>
                    {isOpen && (
                      <div style={{ padding: "0 16px 14px 16px", fontSize: 13, color: "#3a5a7a", lineHeight: 1.7, borderTop: "1px solid #e8f0f8" }}>
                        {item.text}
                      </div>
                    )}
                  </div>
                );
              })}
            </div>
          </div>
        </div>
      );
    }

    const root = ReactDOM.createRoot(document.getElementById("root"));
    root.render(React.createElement(App));
  </script>
</body>
</html>

<!DOCTYPE html>
<html lang="de">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Van-der-Waals-Kräfte – Homologe Reihe der Alkane</title>
<link href="https://fonts.googleapis.com/css2?family=Space+Mono:wght@400;700&family=DM+Sans:wght@300;400;500;700&display=swap" rel="stylesheet">
<style>
  :root {
    --bg: #0d1117;
    --bg2: #161b22;
    --bg3: #1f2733;
    --border: #30363d;
    --accent: #58a6ff;
    --accent2: #f78166;
    --accent3: #3fb950;
    --accent4: #d2a8ff;
    --text: #e6edf3;
    --text2: #8b949e;
    --text3: #6e7681;
    --gas: #f78166;
    --liquid: #58a6ff;
    --solid: #d2a8ff;
    --vdw: rgba(88, 166, 255, 0.35);
  }
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    background: var(--bg);
    color: var(--text);
    font-family: 'DM Sans', sans-serif;
    min-height: 100vh;
    overflow-x: hidden;
  }

  /* HEADER */
  header {
    border-bottom: 1px solid var(--border);
    padding: 18px 32px;
    display: flex;
    align-items: center;
    gap: 16px;
    background: var(--bg2);
  }
  .header-badge {
    font-family: 'Space Mono', monospace;
    font-size: 10px;
    color: var(--accent);
    border: 1px solid var(--accent);
    padding: 3px 8px;
    border-radius: 3px;
    letter-spacing: 1px;
  }
  header h1 {
    font-size: 17px;
    font-weight: 500;
    color: var(--text);
  }
  header p {
    font-size: 12px;
    color: var(--text2);
    margin-left: auto;
  }

  /* MAIN LAYOUT */
  .main {
    display: grid;
    grid-template-columns: 1fr 380px;
    grid-template-rows: auto 1fr;
    gap: 0;
    min-height: calc(100vh - 58px);
  }

  /* SELECTOR BAR */
  .selector-bar {
    grid-column: 1 / -1;
    background: var(--bg2);
    border-bottom: 1px solid var(--border);
    padding: 20px 32px;
    display: flex;
    align-items: center;
    gap: 32px;
  }
  .alkan-name {
    font-family: 'Space Mono', monospace;
    font-size: 22px;
    font-weight: 700;
    color: var(--accent);
    min-width: 120px;
  }
  .alkan-formula {
    font-family: 'Space Mono', monospace;
    font-size: 14px;
    color: var(--text2);
    min-width: 80px;
  }
  .slider-wrap {
    flex: 1;
    display: flex;
    flex-direction: column;
    gap: 8px;
  }
  .slider-labels {
    display: flex;
    justify-content: space-between;
    font-size: 11px;
    color: var(--text3);
    font-family: 'Space Mono', monospace;
  }
  input[type=range] {
    width: 100%;
    -webkit-appearance: none;
    height: 4px;
    background: var(--bg3);
    border-radius: 2px;
    outline: none;
    cursor: pointer;
  }
  input[type=range]::-webkit-slider-thumb {
    -webkit-appearance: none;
    width: 18px; height: 18px;
    border-radius: 50%;
    background: var(--accent);
    border: 2px solid var(--bg);
    box-shadow: 0 0 0 3px rgba(88,166,255,0.2);
    transition: box-shadow 0.2s;
  }
  input[type=range]::-webkit-slider-thumb:hover {
    box-shadow: 0 0 0 6px rgba(88,166,255,0.2);
  }

  .state-badge {
    font-family: 'Space Mono', monospace;
    font-size: 11px;
    padding: 4px 12px;
    border-radius: 20px;
    font-weight: 700;
    letter-spacing: 0.5px;
    text-transform: uppercase;
    min-width: 90px;
    text-align: center;
  }
  .state-gas    { background: rgba(247,129,102,0.15); color: var(--gas);    border: 1px solid rgba(247,129,102,0.4); }
  .state-liquid { background: rgba(88,166,255,0.15);  color: var(--liquid); border: 1px solid rgba(88,166,255,0.4); }
  .state-solid  { background: rgba(210,168,255,0.15); color: var(--solid);  border: 1px solid rgba(210,168,255,0.4); }

  /* MOLECULE PANEL */
  .molecule-panel {
    background: var(--bg);
    border-right: 1px solid var(--border);
    padding: 24px;
    display: flex;
    flex-direction: column;
    gap: 20px;
  }
  .panel-title {
    font-family: 'Space Mono', monospace;
    font-size: 11px;
    color: var(--text3);
    letter-spacing: 1.5px;
    text-transform: uppercase;
  }
  canvas#molCanvas {
    width: 100%;
    border-radius: 8px;
    background: var(--bg2);
    border: 1px solid var(--border);
    display: block;
  }
  .explain-text {
    background: var(--bg2);
    border: 1px solid var(--border);
    border-left: 3px solid var(--accent);
    border-radius: 6px;
    padding: 14px 16px;
    font-size: 13.5px;
    line-height: 1.6;
    color: var(--text);
  }
  .explain-text strong { color: var(--accent); }

  /* LEGEND */
  .legend {
    display: flex;
    flex-wrap: wrap;
    gap: 10px;
  }
  .legend-item {
    display: flex;
    align-items: center;
    gap: 6px;
    font-size: 12px;
    color: var(--text2);
  }
  .legend-dot {
    width: 12px; height: 12px;
    border-radius: 50%;
    flex-shrink: 0;
  }
  .legend-line {
    width: 20px; height: 3px;
    border-radius: 2px;
    flex-shrink: 0;
  }

  /* RIGHT PANEL */
  .right-panel {
    background: var(--bg2);
    display: flex;
    flex-direction: column;
    overflow-y: auto;
  }

  /* TABS */
  .tabs {
    display: flex;
    border-bottom: 1px solid var(--border);
  }
  .tab {
    flex: 1;
    padding: 14px 8px;
    font-size: 12px;
    font-family: 'Space Mono', monospace;
    text-align: center;
    cursor: pointer;
    color: var(--text3);
    border-bottom: 2px solid transparent;
    transition: all 0.2s;
    background: none;
    border-top: none;
    border-left: none;
    border-right: none;
  }
  .tab:hover { color: var(--text); }
  .tab.active { color: var(--accent); border-bottom-color: var(--accent); }

  .tab-content { display: none; padding: 20px; flex: 1; }
  .tab-content.active { display: flex; flex-direction: column; gap: 16px; }

  /* PROPERTIES REVEAL */
  .reveal-btn {
    background: none;
    border: 1px solid var(--accent);
    color: var(--accent);
    font-family: 'Space Mono', monospace;
    font-size: 12px;
    padding: 10px 20px;
    border-radius: 6px;
    cursor: pointer;
    transition: all 0.2s;
    text-align: center;
  }
  .reveal-btn:hover { background: rgba(88,166,255,0.1); }
  .reveal-btn.hidden-mode { background: rgba(88,166,255,0.05); }

  .props-grid {
    display: grid;
    grid-template-columns: 1fr 1fr;
    gap: 10px;
    transition: all 0.4s;
  }
  .prop-card {
    background: var(--bg3);
    border: 1px solid var(--border);
    border-radius: 8px;
    padding: 14px 12px;
    text-align: center;
    opacity: 0;
    transform: translateY(8px);
    transition: opacity 0.4s, transform 0.4s;
  }
  .prop-card.visible { opacity: 1; transform: translateY(0); }
  .prop-card.wide { grid-column: 1/-1; }
  .prop-label {
    font-size: 11px;
    color: var(--text3);
    font-family: 'Space Mono', monospace;
    letter-spacing: 0.5px;
    margin-bottom: 6px;
  }
  .prop-value {
    font-family: 'Space Mono', monospace;
    font-size: 20px;
    font-weight: 700;
  }
  .prop-unit { font-size: 12px; font-weight: 400; color: var(--text2); margin-left: 2px; }

  /* CHART */
  .chart-wrap {
    background: var(--bg3);
    border: 1px solid var(--border);
    border-radius: 8px;
    padding: 16px;
    position: relative;
  }
  .chart-title {
    font-family: 'Space Mono', monospace;
    font-size: 11px;
    color: var(--text3);
    margin-bottom: 12px;
    letter-spacing: 1px;
  }
  canvas.chartCanvas { width: 100% !important; }

  /* VDW STRENGTH BAR */
  .vdw-bar-wrap {
    background: var(--bg3);
    border: 1px solid var(--border);
    border-radius: 8px;
    padding: 16px;
  }
  .vdw-bar-label {
    display: flex;
    justify-content: space-between;
    font-size: 12px;
    color: var(--text2);
    margin-bottom: 8px;
  }
  .vdw-bar-track {
    height: 10px;
    background: var(--bg);
    border-radius: 5px;
    overflow: hidden;
  }
  .vdw-bar-fill {
    height: 100%;
    border-radius: 5px;
    background: linear-gradient(90deg, #58a6ff, #d2a8ff);
    transition: width 0.5s cubic-bezier(0.4,0,0.2,1);
  }

  /* VISCOSITY ANIMATION */
  .visc-container {
    background: var(--bg3);
    border: 1px solid var(--border);
    border-radius: 8px;
    padding: 16px;
  }
  .visc-label {
    font-family: 'Space Mono', monospace;
    font-size: 11px;
    color: var(--text3);
    letter-spacing: 1px;
    margin-bottom: 10px;
  }
  .visc-tube {
    position: relative;
    height: 32px;
    background: var(--bg);
    border-radius: 4px;
    overflow: hidden;
    border: 1px solid var(--border);
    margin-bottom: 6px;
  }
  .visc-fill {
    position: absolute;
    left: 0; top: 0; bottom: 0;
    background: linear-gradient(90deg, rgba(88,166,255,0.7), rgba(88,166,255,0.3));
    border-radius: 4px;
    transition: width 1.2s cubic-bezier(0.4,0,0.2,1);
    display: flex;
    align-items: center;
    padding-left: 10px;
  }
  .visc-fill span { font-size: 11px; color: white; font-family: 'Space Mono', monospace; white-space: nowrap; }
  .visc-note { font-size: 11px; color: var(--text3); }

  /* TASKS */
  .task-card {
    background: var(--bg3);
    border: 1px solid var(--border);
    border-left: 3px solid var(--accent3);
    border-radius: 6px;
    padding: 14px 16px;
  }
  .task-num {
    font-family: 'Space Mono', monospace;
    font-size: 10px;
    color: var(--accent3);
    letter-spacing: 1px;
    margin-bottom: 6px;
  }
  .task-text { font-size: 13.5px; line-height: 1.6; color: var(--text); }
  .task-hint {
    margin-top: 8px;
    font-size: 12px;
    color: var(--text3);
    font-style: italic;
  }

  /* LEGEND PANEL */
  .legend-section { padding: 20px; display: flex; flex-direction: column; gap: 14px; }
  .legend-block {
    background: var(--bg3);
    border: 1px solid var(--border);
    border-radius: 8px;
    padding: 14px;
  }
  .legend-block h3 {
    font-family: 'Space Mono', monospace;
    font-size: 11px;
    color: var(--text3);
    letter-spacing: 1px;
    margin-bottom: 10px;
  }
  .legend-row {
    display: flex;
    align-items: center;
    gap: 10px;
    margin-bottom: 8px;
    font-size: 13px;
    color: var(--text);
  }
  .legend-row:last-child { margin-bottom: 0; }
  .swatch { width: 16px; height: 16px; border-radius: 50%; flex-shrink: 0; }
  .swatch-line { width: 24px; height: 3px; border-radius: 2px; flex-shrink: 0; }
  .swatch-rect { width: 20px; height: 12px; border-radius: 3px; flex-shrink: 0; }
</style>
</head>
<body>

<header>
  <div class="header-badge">REALSCHULE · KL. 10</div>
  <h1>Van-der-Waals-Kräfte &amp; Homologe Reihe der Alkane</h1>
  <p>Interaktive Lernstation · Chemie</p>
</header>

<div class="main">
  <!-- SELECTOR BAR -->
  <div class="selector-bar">
    <div>
      <div class="alkan-name" id="alkanName">Methan</div>
      <div class="alkan-formula" id="alkanFormula">CH₄</div>
    </div>
    <div class="slider-wrap">
      <input type="range" id="alkanSlider" min="1" max="10" value="1" step="1">
      <div class="slider-labels">
        <span>C1 Methan</span>
        <span>C2</span><span>C3</span><span>C4</span><span>C5</span>
        <span>C6</span><span>C7</span><span>C8</span><span>C9</span>
        <span>C10 Dekan</span>
      </div>
    </div>
    <div class="state-badge" id="stateBadge">Gas</div>
  </div>

  <!-- MOLECULE PANEL -->
  <div class="molecule-panel">
    <div class="panel-title">Molekülansicht – Van-der-Waals-Kräfte</div>
    <canvas id="molCanvas" height="340"></canvas>

    <div class="explain-text" id="explainText">
      <strong>Methan (CH₄)</strong> ist das kleinste Alkan mit nur einem Kohlenstoffatom. Die Van-der-Waals-Kräfte sind sehr schwach – die Moleküle bewegen sich schnell und kaum anziehend.
    </div>

    <div class="legend">
      <div class="legend-item"><div class="legend-dot" style="background:#a8c8a0;"></div> Kohlenstoff (C)</div>
      <div class="legend-item"><div class="legend-dot" style="background:#e8e8e8;"></div> Wasserstoff (H)</div>
      <div class="legend-item"><div class="legend-dot" style="background:#5a6070;border:1px solid #58a6ff;"></div> C–C Bindung</div>
      <div class="legend-item"><div class="legend-line" style="background:rgba(88,166,255,0.5);"></div> Van-der-Waals</div>
    </div>
  </div>

  <!-- RIGHT PANEL -->
  <div class="right-panel">
    <div class="tabs">
      <button class="tab active" data-tab="props">Eigenschaften</button>
      <button class="tab" data-tab="charts">Diagramme</button>
      <button class="tab" data-tab="tasks">Aufgaben</button>
      <button class="tab" data-tab="legend">Legende</button>
    </div>

    <!-- PROPERTIES TAB -->
    <div class="tab-content active" id="tab-props">
      <div style="font-size:13px;color:var(--text2);line-height:1.6;">
        Erkunde zuerst das Molekülmodell links. Beobachte, wie sich die Kettenl&auml;nge ver&auml;ndert und welchen Einfluss die Van-der-Waals-Kr&auml;fte haben. Decke dann die Eigenschaften auf.
      </div>

      <button class="reveal-btn" id="revealBtn" onclick="toggleReveal()">🔍 Eigenschaften aufdecken</button>

      <div class="vdw-bar-wrap">
        <div class="vdw-bar-label">
          <span>Van-der-Waals-Stärke</span>
          <span id="vdwPercent">10%</span>
        </div>
        <div class="vdw-bar-track">
          <div class="vdw-bar-fill" id="vdwBar" style="width:10%"></div>
        </div>
      </div>

      <div class="props-grid" id="propsGrid">
        <div class="prop-card" id="pc1">
          <div class="prop-label">SIEDEPUNKT</div>
          <div class="prop-value" id="pvSiede" style="color:var(--gas);">–</div>
        </div>
        <div class="prop-card" id="pc2">
          <div class="prop-label">SCHMELZPUNKT</div>
          <div class="prop-value" id="pvSchmelz" style="color:var(--solid);">–</div>
        </div>
        <div class="prop-card wide" id="pc3">
          <div class="prop-label">VISKOSITÄT bei 25°C</div>
          <div class="prop-value" id="pvVisc" style="color:var(--liquid);">–</div>
        </div>
      </div>

      <div class="visc-container">
        <div class="visc-label">FLIESSFÄHIGKEIT ANIMATION</div>
        <div class="visc-tube">
          <div class="visc-fill" id="viscFill" style="width:95%">
            <span id="viscLabel">fließt sehr leicht</span>
          </div>
        </div>
        <div class="visc-note" id="viscNote">Je länger die Kohlenstoffkette, desto zähflüssiger (viskoser) wird das Alkan.</div>
      </div>
    </div>

    <!-- CHARTS TAB -->
    <div class="tab-content" id="tab-charts">
      <div class="chart-wrap">
        <div class="chart-title">SIEDEPUNKT &amp; SCHMELZPUNKT (°C)</div>
        <canvas id="tempChart" class="chartCanvas" height="200"></canvas>
      </div>
      <div class="chart-wrap">
        <div class="chart-title">VISKOSITÄT (mPa·s, relative Darstellung)</div>
        <canvas id="viscChart" class="chartCanvas" height="160"></canvas>
      </div>
    </div>

    <!-- TASKS TAB -->
    <div class="tab-content" id="tab-tasks">
      <div class="task-card">
        <div class="task-num">AUFGABE 01</div>
        <div class="task-text">Bewege den Schieberegler langsam von Methan bis Dekan. Beschreibe, was du an den Molekülen beobachtest. Was verändert sich?</div>
        <div class="task-hint">Tipp: Achte auf die Anzahl der Kugeln und die Linien zwischen den Molekülen.</div>
      </div>
      <div class="task-card">
        <div class="task-num">AUFGABE 02</div>
        <div class="task-text">Erkläre, warum Methan bei Raumtemperatur ein Gas ist, während Dekan eine Flüssigkeit ist. Nutze den Begriff „Van-der-Waals-Kräfte" in deiner Antwort.</div>
        <div class="task-hint">Tipp: Schau dir die Stärke der Van-der-Waals-Kräfte im Tab „Eigenschaften" an.</div>
      </div>
      <div class="task-card">
        <div class="task-num">AUFGABE 03</div>
        <div class="task-text">Öffne den Tab „Diagramme". Beschreibe den Verlauf von Siede- und Schmelzpunkt. Formuliere eine Regel für die homologe Reihe der Alkane.</div>
        <div class="task-hint">Tipp: „Je länger die Kohlenstoffkette, desto …"</div>
      </div>
      <div class="task-card">
        <div class="task-num">AUFGABE 04</div>
        <div class="task-text">Warum sind längere Alkane zähflüssiger (viskoser)? Erkläre dies mithilfe der Van-der-Waals-Kräfte.</div>
      </div>
      <div class="task-card" style="border-left-color:var(--accent4);">
        <div class="task-num" style="color:var(--accent4);">BONUSAUFGABE</div>
        <div class="task-text">Überlege: Welches Alkan könnte als Benzin verwendet werden? Recherchiere den Begriff „Oktan" und erkläre den Zusammenhang.</div>
      </div>
    </div>

    <!-- LEGEND TAB -->
    <div class="tab-content" id="tab-legend">
      <div class="legend-section" style="padding:0;">
        <div class="legend-block">
          <h3>MOLEKÜLDARSTELLUNG</h3>
          <div class="legend-row"><div class="swatch" style="background:#a8c8a0;"></div> Kohlenstoffatom (C) – grün</div>
          <div class="legend-row"><div class="swatch" style="background:#e8e8e8;border:1px solid #555;"></div> Wasserstoffatom (H) – hellgrau</div>
          <div class="legend-row"><div class="swatch-rect" style="background:#5a6070;"></div> Kovalente C–C Bindung (Stab)</div>
        </div>
        <div class="legend-block">
          <h3>VAN-DER-WAALS-KRÄFTE</h3>
          <div class="legend-row"><div class="swatch-line" style="background:rgba(88,166,255,0.6);"></div> Schwache Anziehung zwischen Molekülen (gestrichelt)</div>
          <div class="legend-row" style="font-size:12px;color:var(--text2);">Je länger die Kohlenstoffkette, desto mehr Kontaktfläche und desto stärker die Van-der-Waals-Kräfte.</div>
        </div>
        <div class="legend-block">
          <h3>AGGREGATZUSTÄNDE</h3>
          <div class="legend-row"><div class="swatch-rect" style="background:rgba(247,129,102,0.3);border:1px solid var(--gas);"></div> <span style="color:var(--gas)">Gas</span> – Moleküle bewegen sich schnell, kaum Anziehung</div>
          <div class="legend-row"><div class="swatch-rect" style="background:rgba(88,166,255,0.3);border:1px solid var(--liquid);"></div> <span style="color:var(--liquid)">Flüssig</span> – mittlere Anziehung, fließt</div>
          <div class="legend-row"><div class="swatch-rect" style="background:rgba(210,168,255,0.3);border:1px solid var(--solid);"></div> <span style="color:var(--solid)">Fest</span> – starke Anziehung, starre Struktur</div>
        </div>
        <div class="legend-block">
          <h3>PHYSIKALISCHE GRÖSSEN</h3>
          <div class="legend-row">Siedepunkt: Temperatur, bei der das Alkan gasförmig wird (in °C)</div>
          <div class="legend-row">Schmelzpunkt: Temperatur, bei der es fest wird (in °C)</div>
          <div class="legend-row">Viskosität: Zähflüssigkeit bei 25°C (in mPa·s)</div>
        </div>
      </div>
    </div>
  </div>
</div>

<script>
// ─── DATA ─────────────────────────────────────────────────────────────────────
const ALKANE = [
  { name:"Methan",  formula:"CH₄",    c:1,  siede:-161.5, schmelz:-182.5, visc:null,  viscLabel:"Gas – keine Viskosität",  state:"gas" },
  { name:"Ethan",   formula:"C₂H₆",   c:2,  siede:-88.6,  schmelz:-183.3, visc:null,  viscLabel:"Gas – keine Viskosität",  state:"gas" },
  { name:"Propan",  formula:"C₃H₈",   c:3,  siede:-42.1,  schmelz:-187.7, visc:null,  viscLabel:"Gas – keine Viskosität",  state:"gas" },
  { name:"Butan",   formula:"C₄H₁₀",  c:4,  siede:-0.5,   schmelz:-138.3, visc:null,  viscLabel:"Gas – keine Viskosität",  state:"gas" },
  { name:"Pentan",  formula:"C₅H₁₂",  c:5,  siede:36.1,   schmelz:-129.7, visc:0.22,  viscLabel:"sehr dünnflüssig",  state:"liquid" },
  { name:"Hexan",   formula:"C₆H₁₄",  c:6,  siede:68.7,   schmelz:-95.3,  visc:0.30,  viscLabel:"dünnflüssig",       state:"liquid" },
  { name:"Heptan",  formula:"C₇H₁₆",  c:7,  siede:98.4,   schmelz:-90.6,  visc:0.39,  viscLabel:"flüssig",           state:"liquid" },
  { name:"Oktan",   formula:"C₈H₁₈",  c:8,  siede:125.7,  schmelz:-56.8,  visc:0.51,  viscLabel:"leicht zähflüssig", state:"liquid" },
  { name:"Nonan",   formula:"C₉H₂₀",  c:9,  siede:150.8,  schmelz:-53.5,  visc:0.67,  viscLabel:"zähflüssig",        state:"liquid" },
  { name:"Dekan",   formula:"C₁₀H₂₂", c:10, siede:174.1,  schmelz:-29.7,  visc:0.85,  viscLabel:"deutlich zähflüssig", state:"liquid" },
];

const EXPLANATIONS = [
  "<strong>Methan (CH₄)</strong> ist das kleinste Alkan mit nur einem Kohlenstoffatom. Die Van-der-Waals-Kräfte sind sehr schwach – die Moleküle bewegen sich schnell und kaum anziehend.",
  "<strong>Ethan (C₂H₆)</strong> hat zwei Kohlenstoffatome. Die etwas größere Oberfläche führt zu minimal stärkeren Van-der-Waals-Kräften – aber Ethan bleibt bei Raumtemperatur ein Gas.",
  "<strong>Propan (C₃H₈)</strong> wird als Flaschengas genutzt. Die Van-der-Waals-Kräfte wachsen mit der Kettenlänge, aber noch nicht stark genug für eine flüssige Phase bei Raumtemperatur.",
  "<strong>Butan (C₄H₁₀)</strong> ist bekannt aus Feuerzeugen. An kalten Tagen kann es flüssig werden – die Van-der-Waals-Kräfte reichen gerade an der Grenze.",
  "<strong>Pentan (C₅H₁₂)</strong> ist bei Raumtemperatur erstmals flüssig. Die Van-der-Waals-Kräfte sind jetzt stark genug, um die Moleküle zusammenzuhalten.",
  "<strong>Hexan (C₆H₁₄)</strong> ist ein gängiges Lösungsmittel im Labor. Die längere Kette sorgt für mehr Anziehung – erkennbar an höherem Siede- und Schmelzpunkt.",
  "<strong>Heptan (C₇H₁₆)</strong> ist ein Bestandteil von Benzin. Die Van-der-Waals-Kräfte werden spürbar stärker – die Moleküle haften mehr aneinander.",
  "<strong>Oktan (C₈H₁₈)</strong> ist namensgebend für die Oktanzahl von Benzin. Die zunehmende Kettenlänge erhöht Siedepunkt und Viskosität deutlich.",
  "<strong>Nonan (C₉H₂₀)</strong> hat eine sehr große Moleküloberfläche. Die Van-der-Waals-Kräfte sind stark – das Alkan ist merklich zähflüssiger als Wasser.",
  "<strong>Dekan (C₁₀H₂₂)</strong> ist das größte Alkan unserer Reihe. Die Van-der-Waals-Kräfte sind am stärksten – hoher Siede- und Schmelzpunkt, deutlich zähflüssig.",
];

// ─── STATE ────────────────────────────────────────────────────────────────────
let currentIdx = 0;
let revealed = false;
let molecules = [];
let animFrame;

// ─── MOLECULE ANIMATION ───────────────────────────────────────────────────────
const canvas = document.getElementById('molCanvas');
const ctx = canvas.getContext('2d');

function resizeCanvas() {
  const rect = canvas.parentElement.getBoundingClientRect();
  const w = Math.min(rect.width - 48, 700);
  canvas.width = w;
  canvas.height = 340;
}

function initMolecules(idx) {
  const alkan = ALKANE[idx];
  molecules = [];
  const count = alkan.state === 'gas' ? 5 : alkan.state === 'solid' ? 8 : 6;
  const speed = alkan.state === 'gas' ? 1.4 : alkan.state === 'solid' ? 0.12 : 0.5;

  for (let i = 0; i < count; i++) {
    const angle = Math.random() * Math.PI * 2;
    molecules.push({
      x: 60 + Math.random() * (canvas.width - 120),
      y: 40 + Math.random() * (canvas.height - 80),
      vx: Math.cos(angle) * speed * (0.7 + Math.random() * 0.6),
      vy: Math.sin(angle) * speed * (0.7 + Math.random() * 0.6),
    });
  }
}

function drawMolecule(x, y, c, state) {
  const cRadius = 10;
  const hRadius = 6;
  const bondLen = 22;
  const colors = { C: '#a8c8a0', H: '#dde8dd', bond: '#5a6070' };

  // Draw bonds between C atoms first
  if (c > 1) {
    ctx.strokeStyle = colors.bond;
    ctx.lineWidth = 5;
    ctx.lineCap = 'round';
    for (let i = 0; i < c - 1; i++) {
      const x1 = x + (i - (c-1)/2) * bondLen;
      const x2 = x + (i+1 - (c-1)/2) * bondLen;
      ctx.beginPath();
      ctx.moveTo(x1, y);
      ctx.lineTo(x2, y);
      ctx.stroke();
    }
  }

  // Draw H atoms and C atoms
  for (let i = 0; i < c; i++) {
    const cx = x + (i - (c-1)/2) * bondLen;
    // H above and below
    const hCount = (i === 0 || i === c-1) ? 3 : 2;
    const hPositions = [];
    if (hCount === 3) {
      hPositions.push([cx - 14, y - 12], [cx - 14, y + 12], [cx - 20, y]);
    } else {
      hPositions.push([cx, y - 18], [cx, y + 18]);
    }
    // Draw H bonds
    ctx.strokeStyle = '#3a4050';
    ctx.lineWidth = 2;
    hPositions.forEach(([hx, hy]) => {
      ctx.beginPath();
      ctx.moveTo(cx, y);
      ctx.lineTo(hx, hy);
      ctx.stroke();
    });
    // Draw H atoms
    hPositions.forEach(([hx, hy]) => {
      ctx.beginPath();
      ctx.arc(hx, hy, hRadius, 0, Math.PI*2);
      ctx.fillStyle = colors.H;
      ctx.fill();
    });
    // Draw C atom
    ctx.beginPath();
    ctx.arc(cx, y, cRadius, 0, Math.PI*2);
    ctx.fillStyle = colors.C;
    ctx.fill();
  }
}

function drawVdwLines(idx, mols) {
  const alkan = ALKANE[idx];
  const strength = idx / 9; // 0..1
  if (strength < 0.05) return;

  // Find close pairs
  for (let i = 0; i < mols.length; i++) {
    for (let j = i+1; j < mols.length; j++) {
      const dx = mols[j].x - mols[i].x;
      const dy = mols[j].y - mols[i].y;
      const dist = Math.sqrt(dx*dx + dy*dy);
      const threshold = 80 + strength * 80;
      if (dist < threshold) {
        const alpha = strength * (1 - dist/threshold) * 0.8;
        ctx.strokeStyle = `rgba(88,166,255,${alpha})`;
        ctx.lineWidth = 1 + strength * 2;
        ctx.setLineDash([4, 6]);
        ctx.beginPath();
        ctx.moveTo(mols[i].x, mols[i].y);
        ctx.lineTo(mols[j].x, mols[j].y);
        ctx.stroke();
        ctx.setLineDash([]);
      }
    }
  }
}

function animate() {
  const alkan = ALKANE[currentIdx];
  const W = canvas.width, H = canvas.height;
  ctx.clearRect(0, 0, W, H);

  // Background grid
  ctx.strokeStyle = 'rgba(48,54,61,0.4)';
  ctx.lineWidth = 1;
  for (let x = 0; x < W; x += 40) {
    ctx.beginPath(); ctx.moveTo(x,0); ctx.lineTo(x,H); ctx.stroke();
  }
  for (let y = 0; y < H; y += 40) {
    ctx.beginPath(); ctx.moveTo(0,y); ctx.lineTo(W,y); ctx.stroke();
  }

  // Update positions
  molecules.forEach(m => {
    if (alkan.state === 'solid') {
      // Slight vibration only
      m.x += (Math.random()-0.5)*0.5;
      m.y += (Math.random()-0.5)*0.5;
    } else {
      m.x += m.vx;
      m.y += m.vy;
      // Bounce
      const margin = 50;
      if (m.x < margin || m.x > W-margin) m.vx *= -1;
      if (m.y < margin || m.y > H-margin) m.vy *= -1;
      m.x = Math.max(margin, Math.min(W-margin, m.x));
      m.y = Math.max(margin, Math.min(H-margin, m.y));
    }
  });

  // Draw VdW lines
  drawVdwLines(currentIdx, molecules);

  // Draw molecules
  molecules.forEach(m => {
    drawMolecule(m.x, m.y, alkan.c, alkan.state);
  });

  // State glow
  const glowColor = alkan.state==='gas' ? 'rgba(247,129,102,0.06)' :
                    alkan.state==='liquid' ? 'rgba(88,166,255,0.06)' : 'rgba(210,168,255,0.06)';
  ctx.fillStyle = glowColor;
  ctx.fillRect(0,0,W,H);

  animFrame = requestAnimationFrame(animate);
}

// ─── CHARTS ───────────────────────────────────────────────────────────────────
let tempChartInst = null, viscChartInst = null;

function initCharts() {
  const labels = ALKANE.map(a => a.name.substring(0,3));
  const siedeData = ALKANE.map(a => a.siede);
  const schmelzData = ALKANE.map(a => a.schmelz);
  const viscData = ALKANE.map(a => a.visc || 0);

  const commonOpts = {
    responsive: true,
    maintainAspectRatio: true,
    plugins: { legend: { labels: { color: '#8b949e', font: { family: 'DM Sans', size: 11 } } } },
    scales: {
      x: { ticks: { color: '#6e7681', font: { family: 'Space Mono', size: 10 } }, grid: { color: 'rgba(48,54,61,0.5)' } },
      y: { ticks: { color: '#6e7681', font: { family: 'Space Mono', size: 10 } }, grid: { color: 'rgba(48,54,61,0.5)' } }
    }
  };

  if (tempChartInst) tempChartInst.destroy();
  tempChartInst = new Chart(document.getElementById('tempChart'), {
    type: 'line',
    data: {
      labels,
      datasets: [
        { label: 'Siedepunkt (°C)', data: siedeData, borderColor: '#f78166', backgroundColor: 'rgba(247,129,102,0.1)', tension: 0.3, pointBackgroundColor: '#f78166', pointRadius: 5 },
        { label: 'Schmelzpunkt (°C)', data: schmelzData, borderColor: '#d2a8ff', backgroundColor: 'rgba(210,168,255,0.1)', tension: 0.3, pointBackgroundColor: '#d2a8ff', pointRadius: 5 },
      ]
    },
    options: { ...commonOpts, animation: { duration: 600 } }
  });

  if (viscChartInst) viscChartInst.destroy();
  viscChartInst = new Chart(document.getElementById('viscChart'), {
    type: 'bar',
    data: {
      labels,
      datasets: [{ label: 'Viskosität (mPa·s)', data: viscData, backgroundColor: ALKANE.map((_,i) => i===currentIdx ? 'rgba(88,166,255,0.9)' : 'rgba(88,166,255,0.3)'), borderColor: '#58a6ff', borderWidth: 1 }]
    },
    options: { ...commonOpts, animation: { duration: 600 } }
  });
}

function updateChartHighlight() {
  if (!viscChartInst) return;
  viscChartInst.data.datasets[0].backgroundColor = ALKANE.map((_,i) => i===currentIdx ? 'rgba(88,166,255,0.9)' : 'rgba(88,166,255,0.3)');
  viscChartInst.update('none');
}

// ─── UI UPDATE ────────────────────────────────────────────────────────────────
function formatTemp(v) {
  return (v >= 0 ? '+' : '') + v.toFixed(1) + ' °C';
}

function update(idx) {
  const a = ALKANE[idx];
  document.getElementById('alkanName').textContent = a.name;
  document.getElementById('alkanFormula').textContent = a.formula;

  const badge = document.getElementById('stateBadge');
  badge.className = 'state-badge state-' + a.state;
  badge.textContent = a.state === 'gas' ? 'Gas' : a.state === 'liquid' ? 'Flüssig' : 'Fest';

  document.getElementById('explainText').innerHTML = EXPLANATIONS[idx];

  // VdW strength bar
  const pct = Math.round(10 + idx * 10);
  document.getElementById('vdwBar').style.width = pct + '%';
  document.getElementById('vdwPercent').textContent = pct + '%';

  // Properties (only if revealed)
  document.getElementById('pvSiede').innerHTML = revealed ? formatTemp(a.siede) + '<span class="prop-unit">°C</span>' : '?';
  document.getElementById('pvSchmelz').innerHTML = revealed ? formatTemp(a.schmelz) + '<span class="prop-unit">°C</span>' : '?';
  document.getElementById('pvVisc').innerHTML = revealed ? (a.visc ? a.visc.toFixed(2) + '<span class="prop-unit">mPa·s</span>' : '<span style="font-size:14px;color:var(--text3)">Gasförmig</span>') : '?';

  // Viscosity animation
  const viscPct = a.visc ? Math.min(95, 30 + (a.visc / 0.85) * 65) : 95;
  document.getElementById('viscFill').style.width = (a.state === 'gas' ? 98 : (100 - viscPct)) + '%';
  document.getElementById('viscLabel').textContent = a.state === 'gas' ? 'Gas – fließt als Dampf' : a.viscLabel;

  // Reset reveal if slider moved
  if (revealed) {
    document.querySelectorAll('.prop-card').forEach(c => c.classList.add('visible'));
  }

  updateChartHighlight();
  initMolecules(idx);
}

function toggleReveal() {
  revealed = !revealed;
  const btn = document.getElementById('revealBtn');
  const a = ALKANE[currentIdx];

  if (revealed) {
    btn.textContent = '🙈 Eigenschaften verbergen';
    document.getElementById('pvSiede').innerHTML = formatTemp(a.siede) + '<span class="prop-unit">°C</span>';
    document.getElementById('pvSchmelz').innerHTML = formatTemp(a.schmelz) + '<span class="prop-unit">°C</span>';
    document.getElementById('pvVisc').innerHTML = a.visc ? a.visc.toFixed(2) + '<span class="prop-unit">mPa·s</span>' : '<span style="font-size:14px;color:var(--text3)">Gasförmig</span>';
    setTimeout(() => document.querySelectorAll('.prop-card').forEach((c,i) => {
      setTimeout(() => c.classList.add('visible'), i * 80);
    }), 10);
  } else {
    btn.textContent = '🔍 Eigenschaften aufdecken';
    document.querySelectorAll('.prop-card').forEach(c => c.classList.remove('visible'));
    document.getElementById('pvSiede').innerHTML = '?';
    document.getElementById('pvSchmelz').innerHTML = '?';
    document.getElementById('pvVisc').innerHTML = '?';
  }
}

// ─── TABS ─────────────────────────────────────────────────────────────────────
document.querySelectorAll('.tab').forEach(tab => {
  tab.addEventListener('click', () => {
    document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
    document.querySelectorAll('.tab-content').forEach(c => c.classList.remove('active'));
    tab.classList.add('active');
    document.getElementById('tab-' + tab.dataset.tab).classList.add('active');
    if (tab.dataset.tab === 'charts' && !tempChartInst) initCharts();
    if (tab.dataset.tab === 'charts') updateChartHighlight();
  });
});

// ─── SLIDER ───────────────────────────────────────────────────────────────────
document.getElementById('alkanSlider').addEventListener('input', function() {
  currentIdx = parseInt(this.value) - 1;
  revealed = false;
  document.getElementById('revealBtn').textContent = '🔍 Eigenschaften aufdecken';
  document.querySelectorAll('.prop-card').forEach(c => c.classList.remove('visible'));
  update(currentIdx);
});

// ─── CHART.JS LOAD ────────────────────────────────────────────────────────────
const script = document.createElement('script');
script.src = 'https://cdnjs.cloudflare.com/ajax/libs/Chart.js/4.4.0/chart.umd.min.js';
script.onload = () => { /* charts initialized on tab open */ };
document.head.appendChild(script);

// ─── INIT ─────────────────────────────────────────────────────────────────────
window.addEventListener('resize', () => { resizeCanvas(); initMolecules(currentIdx); });
resizeCanvas();
initMolecules(0);
update(0);
animate();
</script>
</body>
</html>
