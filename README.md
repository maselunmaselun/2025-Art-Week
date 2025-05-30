<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Pixel Art Generator</title>
  <style>
    body {
      background: url('img/2025_ArtWeekB.png') center center fixed;
      background-size: cover;
      color: #0ff;
      font-family: monospace;
      text-align: center;
      padding: 2rem;
    }
    canvas {
      margin-top: 1rem;
      max-width: 90vw;
      height: auto;
      border: 2px dashed #0ff;
    }
    input[type="file"] {
      margin-top: 1rem;
      font-size: 1rem;
    }
    .slider-container {
      display: flex;
      flex-direction: column;
      align-items: center;
      margin-top: 1rem;
      color: #fff;
    }
    input[type="range"] {
      width: 140px;
      height: 4px;
    }
    h2 {
      margin-top: 1rem;
      color: #fff;
    }
    #save-btn {
      margin-top: 1.5rem;
      padding: 0.5rem 1rem;
      font-size: 1rem;
      font-family: monospace;
      background-color: #0ff;
      color: #000;
      border: none;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <input type="file" accept="image/*" id="upload" />
  <div class="slider-container">
    <label for="effectLevel">Effect Strength (1 = Weak, 5 = Strong)</label>
    <input type="range" id="effectLevel" min="1" max="5" value="1" />
  </div>
  <canvas id="canvas"></canvas>
  <h2 id="generated-name"></h2>
  <button id="save-btn">Download Your Poster</button>

  <script>
    document.addEventListener("DOMContentLoaded", () => {
      const adjectives = ['Glitchy', 'Pixelated', 'Broken', 'Corrupted', 'Lost', 'Ghostly', 'Synthetic', 'Wired'];
      const nouns = ['Dream', 'Signal', 'Face', 'Code', 'Illusion', 'Echo', 'Loop', 'Ghost'];

      const canvas = document.getElementById('canvas');
      const ctx = canvas.getContext('2d');
      const upload = document.getElementById('upload');
      const nameDisplay = document.getElementById('generated-name');
      const effectSlider = document.getElementById('effectLevel');
      const saveBtn = document.getElementById('save-btn');

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

      let image = null;

      function applyEffect() {
        if (!image || image.width === 0 || image.height === 0) return;

        const level = parseInt(effectSlider.value);
        const canvasSize = 1024;
        canvas.width = canvasSize;
        canvas.height = canvasSize;

        const size = Math.min(image.width, image.height);
        const offsetX = (image.width - size) / 2;
        const offsetY = (image.height - size) / 2;

        ctx.clearRect(0, 0, canvasSize, canvasSize);
        ctx.drawImage(image, offsetX, offsetY, size, size, 0, 0, canvasSize, canvasSize);

        try {
          const srcData = ctx.getImageData(0, 0, canvasSize, canvasSize);
          const tempCanvas = document.createElement('canvas');
          tempCanvas.width = canvasSize;
          tempCanvas.height = canvasSize;
          const tempCtx = tempCanvas.getContext('2d');

          const pixelSize = 32 - level * 5;
          for (let y = 0; y < canvasSize; y += pixelSize) {
            for (let x = 0; x < canvasSize; x += pixelSize) {
              const i = (y * canvasSize + x) * 4;
              let r = srcData.data[i];
              let g = srcData.data[i + 1];
              let b = srcData.data[i + 2];
              const a = srcData.data[i + 3] / 255;

              [r, g, b] = increaseSaturation(r, g, b, 1.6 + level);

              const color = `rgba(${r}, ${g}, ${b}, ${a})`;
              tempCtx.fillStyle = color;
              tempCtx.fillRect(x, y, pixelSize, pixelSize);
            }
          }

          ctx.clearRect(0, 0, canvasSize, canvasSize);
          ctx.drawImage(tempCanvas, 0, 0);

          for (let i = 0; i < 30 * level; i++) {
            const y = Math.floor(Math.random() * canvasSize);
            const h = Math.floor(Math.random() * (10 + level * 3));
            const offset = Math.floor(Math.random() * 200 - 100);
            const slice = ctx.getImageData(0, y, canvasSize, h);
            ctx.clearRect(0, y, canvasSize, h);
            ctx.putImageData(slice, offset, y);
          }

          for (let i = 0; i < 5000 * level; i++) {
            const x = Math.random() * canvasSize;
            const y = Math.random() * canvasSize;
            const r = Math.floor(Math.random() * 256);
            const g = Math.floor(Math.random() * 256);
            const b = Math.floor(Math.random() * 256);
            ctx.fillStyle = `rgba(${r},${g},${b},${Math.random() * 0.3})`;
            ctx.fillRect(x, y, 1, 1);
          }

          nameDisplay.textContent = generateName();
        } catch (e) {
          console.error("Canvas getImageData failed:", e);
        }
      }

      upload.addEventListener('change', e => {
        const file = e.target.files[0];
        if (!file) return;

        const img = new Image();
        img.onload = () => {
          if (img.width === 0 || img.height === 0) {
            console.error("Loaded image has invalid dimensions.");
            return;
          }
          image = img;
          setTimeout(applyEffect, 10);
        };
        img.onerror = () => {
          console.error("Image load failed.");
        };
        img.src = URL.createObjectURL(file);
      });

      effectSlider.addEventListener('input', () => {
        if (image && image.width > 0 && image.height > 0) {
          applyEffect();
        }
      });

      saveBtn.addEventListener('click', () => {
        const link = document.createElement('a');
        link.download = 'pixel_art.png';
        link.href = canvas.toDataURL('image/png');
        link.click();
      });
    });
  </script>
</body>
</html>
