<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Pixel Glitch Cam</title>
  <style>
    body {
      background: url('2025_ArtWeekB.png') center center fixed;
      background-size: cover;
      background-color: black;
      margin: 0;
      color: #0ff;
      font-family: monospace;
      text-align: center;
      padding: 1rem;
    }
    canvas, video {
      display: block;
      margin: 1rem auto;
      max-width: 90vw;
      border: 2px dashed #0ff;
    }
    #generated-name {
      margin-top: 1rem;
      color: #fff;
    }
  </style>
</head>
<body>
  <video id="video" autoplay playsinline style="display:none;"></video>
  <canvas id="canvas"></canvas>
  <h2 id="generated-name"></h2>

  <script>
    const adjectives = ['Glitchy', 'Pixelated', 'Broken', 'Corrupted', 'Lost', 'Ghostly', 'Synthetic', 'Wired'];
    const nouns = ['Dream', 'Signal', 'Face', 'Code', 'Illusion', 'Echo', 'Loop', 'Ghost'];

    const video = document.getElementById('video');
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    const nameDisplay = document.getElementById('generated-name');

    const canvasSize = 512;
    canvas.width = canvasSize;
    canvas.height = canvasSize;

    function generateName() {
      const adj = adjectives[Math.floor(Math.random() * adjectives.length)];
      const noun = nouns[Math.floor(Math.random() * nouns.length)];
      return `${adj} ${noun}`;
    }

    function increaseSaturation(r, g, b, factor = 1.5) {
      const gray = 0.3 * r + 0.59 * g + 0.11 * b;
      r = gray + (r - gray) * factor;
      g = gray + (g - gray) * factor;
      b = gray + (b - gray) * factor;
      return [Math.min(255, r), Math.min(255, g), Math.min(255, b)];
    }

    async function startVideo() {
      if (location.protocol !== 'https:' && location.hostname !== 'localhost') {
        alert("Camera access requires HTTPS or localhost. Please use a secure connection.");
        return;
      }
      try {
        const stream = await navigator.mediaDevices.getUserMedia({ video: true });
        video.srcObject = stream;
        video.onloadedmetadata = () => video.play();
      } catch (err) {
        alert("Camera access denied or not available. Please check your permissions and make sure you're using HTTPS.");
        console.error(err);
      }
    }

    let lastFrame = null;

    function applyEffect() {
      if (video.readyState < 2) return;

      ctx.drawImage(video, 0, 0, canvasSize, canvasSize);
      let frame = ctx.getImageData(0, 0, canvasSize, canvasSize);

      let level = 1;
      if (lastFrame) {
        let diff = 0;
        for (let i = 0; i < frame.data.length; i += 4) {
          diff += Math.abs(frame.data[i] - lastFrame.data[i]);
        }
        level = Math.min(5, Math.floor(diff / 100000));
      }
      lastFrame = frame;

      const tempCanvas = document.createElement('canvas');
      tempCanvas.width = canvasSize;
      tempCanvas.height = canvasSize;
      const tempCtx = tempCanvas.getContext('2d');
      const pixelSize = 32 - level * 5;

      for (let y = 0; y < canvasSize; y += pixelSize) {
        for (let x = 0; x < canvasSize; x += pixelSize) {
          const i = (y * canvasSize + x) * 4;
          let r = frame.data[i];
          let g = frame.data[i + 1];
          let b = frame.data[i + 2];
          const a = frame.data[i + 3] / 255;

          [r, g, b] = increaseSaturation(r, g, b, 1.6 + level);

          tempCtx.fillStyle = `rgba(${r}, ${g}, ${b}, ${a})`;
          tempCtx.fillRect(x, y, pixelSize, pixelSize);
        }
      }

      ctx.clearRect(0, 0, canvasSize, canvasSize);
      ctx.drawImage(tempCanvas, 0, 0);

      for (let i = 0; i < 20 * level; i++) {
        const y = Math.floor(Math.random() * canvasSize);
        const h = Math.floor(Math.random() * (10 + level * 3));
        const offset = Math.floor(Math.random() * 200 - 100);
        const slice = ctx.getImageData(0, y, canvasSize, h);
        ctx.clearRect(0, y, canvasSize, h);
        ctx.putImageData(slice, offset, y);
      }

      for (let i = 0; i < 2000 * level; i++) {
        const x = Math.random() * canvasSize;
        const y = Math.random() * canvasSize;
        const r = Math.floor(Math.random() * 256);
        const g = Math.floor(Math.random() * 256);
        const b = Math.floor(Math.random() * 256);
        ctx.fillStyle = `rgba(${r},${g},${b},${Math.random() * 0.3})`;
        ctx.fillRect(x, y, 1, 1);
      }

      nameDisplay.textContent = generateName();
    }

    startVideo();
    setInterval(applyEffect, 500);
  </script>
</body>
</html>
