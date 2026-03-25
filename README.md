<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Halftone Pro - Pattern Generator</title>
    <style>
        :root {
            --bg-dark: #1e1e2f;
            --bg-panel: #27293d;
            --primary: #e14eca;
            --text-main: #ffffff;
            --text-muted: #9a9a9a;
            --border: #3b3d54;
        }

        * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }

        body { display: flex; height: 100vh; background-color: var(--bg-dark); color: var(--text-main); overflow: hidden; }

        /* Sidebar Controls */
        .sidebar { width: 350px; background: var(--bg-panel); border-right: 1px solid var(--border); display: flex; flex-direction: column; overflow-y: auto; z-index: 10; }
        .header { padding: 20px; border-bottom: 1px solid var(--border); text-align: center; }
        .header h1 { font-size: 20px; color: var(--primary); }
        .section { padding: 15px 20px; border-bottom: 1px solid var(--border); }
        .section-title { font-size: 14px; font-weight: bold; margin-bottom: 15px; color: var(--primary); text-transform: uppercase; letter-spacing: 1px; }

        /* Form Elements */
        .form-group { margin-bottom: 12px; }
        .form-group label { display: flex; justify-content: space-between; font-size: 13px; margin-bottom: 5px; color: var(--text-muted); }
        .val-display { color: #fff; }
        input[type="range"] { width: 100%; cursor: pointer; accent-color: var(--primary); }
        select, input[type="file"], input[type="number"] { width: 100%; padding: 8px; background: var(--bg-dark); color: #fff; border: 1px solid var(--border); border-radius: 4px; outline: none; }
        .checkbox-group { display: flex; align-items: center; gap: 8px; font-size: 13px; cursor: pointer; color: var(--text-muted); }
        
        .flex-row { display: flex; gap: 10px; }
        .flex-row > div { flex: 1; }

        /* Buttons */
        .btn-group { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-top: 10px; }
        button { padding: 10px; border: none; border-radius: 4px; cursor: pointer; font-weight: bold; transition: 0.2s; }
        .btn-primary { background: var(--primary); color: #fff; }
        .btn-primary:hover { background: #c238ad; }
        .btn-secondary { background: #344675; color: #fff; }
        .btn-secondary:hover { background: #263358; }
        .btn-export { background: #00d6b4; color: #111; grid-column: span 2; }
        .btn-export:hover { background: #00b396; }

        /* Preview Area */
        .preview-area { flex-grow: 1; position: relative; display: flex; justify-content: center; align-items: center; background: #111; overflow: hidden; cursor: grab; }
        .preview-area:active { cursor: grabbing; }
        .checkerboard { position: absolute; top: 0; left: 0; right: 0; bottom: 0; background-image: linear-gradient(45deg, #222 25%, transparent 25%), linear-gradient(-45deg, #222 25%, transparent 25%), linear-gradient(45deg, transparent 75%, #222 75%), linear-gradient(-45deg, transparent 75%, #222 75%); background-size: 20px 20px; background-position: 0 0, 0 10px, 10px -10px, -10px 0px; z-index: 1; }
        
        #canvas-container { position: relative; z-index: 2; transform-origin: 0 0; background: #fff; box-shadow: 0 0 20px rgba(0,0,0,0.5); transition: transform 0.1s ease-out; }
        canvas { display: block; }
        
        /* Print Frame */
        #print-frame { display: none; }
    </style>
</head>
<body>

    <div class="sidebar">
        <div class="header">
            <h1>Halftone Pro</h1>
        </div>

        <div class="section">
            <input type="file" id="imageInput" accept="image/png, image/jpeg, image/svg+xml">
        </div>

        <div class="section">
            <div class="section-title">Pattern Area & Layout</div>
            <div class="form-group">
                <label>Draw Pattern In</label>
                <select id="patternArea">
                    <option value="all">Entire Page (كامل الصفحة)</option>
                    <option value="inside">Inside Image Only (داخل الصورة)</option>
                    <option value="outside">Background Only (خارج الصورة)</option>
                </select>
            </div>
            <div class="form-group">
                <label class="checkbox-group"><input type="checkbox" id="ignoreWhite" checked> Treat White as Background</label>
            </div>
            <div class="form-group" style="margin-top: 15px;">
                <label>Shape Type</label>
                <select id="shapeType">
                    <option value="circle">Circles</option>
                    <option value="square">Squares</option>
                    <option value="hexagon">Hexagons</option>
                </select>
            </div>
        </div>

        <div class="section">
            <div class="section-title">Size & Spacing</div>
            <div class="form-group">
                <label>Spacing (mm) <span class="val-display" id="valSpacing">5</span></label>
                <input type="range" id="spacing" min="1" max="20" step="0.5" value="5">
            </div>
            <div class="form-group">
                <label>Max Size (mm) <span class="val-display" id="valSize">4.5</span></label>
                <input type="range" id="maxSize" min="0.5" max="20" step="0.1" value="4.5">
            </div>
            <div class="form-group">
                <label class="checkbox-group"><input type="checkbox" id="fixedSize"> Fixed Size</label>
            </div>
            <div class="form-group">
                <label class="checkbox-group"><input type="checkbox" id="outlineOnly"> Outline Only</label>
            </div>
        </div>

        <div class="section">
            <div class="section-title">Image Processing</div>
            <div class="form-group">
                <label>Threshold <span class="val-display" id="valThreshold">0</span></label>
                <input type="range" id="threshold" min="0" max="255" step="1" value="0">
            </div>
            <div class="form-group">
                <label>Image Scale (%) <span class="val-display" id="valImgScale">100</span></label>
                <input type="range" id="imgScale" min="10" max="300" step="1" value="100">
            </div>
            <div class="form-group">
                <label class="checkbox-group"><input type="checkbox" id="invert"> Invert Brightness</label>
            </div>
        </div>

        <div class="section">
            <div class="section-title">Page & Export Settings</div>
            <div class="form-group">
                <label>Page Size</label>
                <select id="pageSize">
                    <option value="A4">A4 (210 x 297 mm)</option>
                    <option value="A3">A3 (297 x 420 mm)</option>
                    <option value="Custom">Custom Size...</option>
                </select>
            </div>
            
            <div class="form-group flex-row" id="customSizeGroup" style="display: none;">
                <div>
                    <label>Width (mm)</label>
                    <input type="number" id="customW" value="150" min="10">
                </div>
                <div>
                    <label>Height (mm)</label>
                    <input type="number" id="customH" value="150" min="10">
                </div>
            </div>

            <div class="form-group">
                <label>Margin (mm) <span class="val-display" id="valMargin">10</span></label>
                <input type="range" id="margin" min="0" max="50" step="1" value="10">
            </div>
            <div class="form-group">
                <label>Export DPI</label>
                <select id="dpi">
                    <option value="300">300 DPI</option>
                    <option value="600">600 DPI</option>
                </select>
            </div>
            <div class="btn-group">
                <button class="btn-primary" onclick="process()">Convert</button>
                <button class="btn-secondary" onclick="resetView()">Reset View</button>
                <button class="btn-export" onclick="downloadSVG()">Download SVG</button>
                <button class="btn-export" onclick="downloadPNG()" style="background: #e14eca; color:white;">Download PNG</button>
                <button class="btn-export" onclick="downloadPDF()" style="background: #ff5252; color:white;">Download PDF</button>
            </div>
        </div>
    </div>

    <div class="preview-area" id="previewArea">
        <div class="checkerboard"></div>
        <div id="canvas-container">
            <canvas id="previewCanvas"></canvas>
        </div>
    </div>

    <iframe id="print-frame"></iframe>

    <script>
        // --- State ---
        let originalImage = null;
        let imgData = null;
        let sampleWidth = 800;
        
        let zoom = 1;
        let panX = 0; let panY = 0;
        let isDragging = false;
        let startX, startY;

        // Elements
        const canvasContainer = document.getElementById('canvas-container');
        const previewCanvas = document.getElementById('previewCanvas');
        const ctx = previewCanvas.getContext('2d');
        const previewArea = document.getElementById('previewArea');
        const customSizeGroup = document.getElementById('customSizeGroup');

        // Page Sizes
        const pages = {
            'A4': { w: 210, h: 297 },
            'A3': { w: 297, h: 420 }
        };

        // UI Binding
        const bindSlider = (id, valId) => {
            const el = document.getElementById(id);
            const valEl = document.getElementById(valId);
            el.addEventListener('input', () => {
                valEl.innerText = el.value;
                if(originalImage) process();
            });
        };['spacing', 'maxSize', 'threshold', 'imgScale', 'margin'].forEach(id => {
            bindSlider(id, 'val' + id.charAt(0).toUpperCase() + id.slice(1));
        });

        document.querySelectorAll('select, input[type="checkbox"]').forEach(el => {
            el.addEventListener('change', (e) => { 
                if(e.target.id === 'pageSize') {
                    customSizeGroup.style.display = e.target.value === 'Custom' ? 'flex' : 'none';
                    if(originalImage) { process(); resetView(); }
                } else {
                    if(originalImage) process(); 
                }
            });
        });

        document.getElementById('customW').addEventListener('input', () => { if(originalImage) {process(); resetView();} });
        document.getElementById('customH').addEventListener('input', () => { if(originalImage) {process(); resetView();} });

        // --- Image Upload ---
        document.getElementById('imageInput').addEventListener('change', function(e) {
            const file = e.target.files[0];
            if (!file) return;
            const reader = new FileReader();
            reader.onload = function(event) {
                const img = new Image();
                img.onload = function() {
                    originalImage = img;
                    process();
                    resetView();
                }
                img.src = event.target.result;
            }
            reader.readAsDataURL(file);
        });

        // --- Core Processing Logic ---
        function getParams() {
            let pageW, pageH;
            if (document.getElementById('pageSize').value === 'Custom') {
                pageW = parseFloat(document.getElementById('customW').value) || 100;
                pageH = parseFloat(document.getElementById('customH').value) || 100;
            } else {
                const page = pages[document.getElementById('pageSize').value];
                pageW = page.w;
                pageH = page.h;
            }

            return {
                pageW, pageH,
                margin: parseFloat(document.getElementById('margin').value),
                spacing: parseFloat(document.getElementById('spacing').value),
                maxSize: parseFloat(document.getElementById('maxSize').value),
                fixedSize: document.getElementById('fixedSize').checked,
                outlineOnly: document.getElementById('outlineOnly').checked,
                shape: document.getElementById('shapeType').value,
                patternArea: document.getElementById('patternArea').value,
                ignoreWhite: document.getElementById('ignoreWhite').checked,
                threshold: parseInt(document.getElementById('threshold').value),
                invert: document.getElementById('invert').checked,
                imgScale: parseInt(document.getElementById('imgScale').value) / 100,
                dpi: parseInt(document.getElementById('dpi').value)
            };
        }

        function sampleImage(p) {
            if(!originalImage) return;
            const drawW = p.pageW - (p.margin * 2);
            const drawH = p.pageH - (p.margin * 2);
            
            const offCanvas = document.createElement('canvas');
            offCanvas.width = sampleWidth;
            offCanvas.height = sampleWidth * (drawH / drawW);
            const oCtx = offCanvas.getContext('2d', { willReadFrequently: true });

            // Clear to transparent (Do not fill solid, so we can detect background)
            oCtx.clearRect(0, 0, offCanvas.width, offCanvas.height);

            const imgAspect = originalImage.width / originalImage.height;
            const canvasAspect = offCanvas.width / offCanvas.height;
            let dw, dh;
            if (imgAspect > canvasAspect) {
                dw = offCanvas.width * p.imgScale;
                dh = (offCanvas.width / imgAspect) * p.imgScale;
            } else {
                dh = offCanvas.height * p.imgScale;
                dw = (offCanvas.height * imgAspect) * p.imgScale;
            }
            const dx = (offCanvas.width - dw) / 2;
            const dy = (offCanvas.height - dh) / 2;

            oCtx.drawImage(originalImage, dx, dy, dw, dh);
            imgData = oCtx.getImageData(0, 0, offCanvas.width, offCanvas.height).data;
        }

        function getPixelInfo(x, y, p, offW, offH) {
            const drawW = p.pageW - (p.margin * 2);
            const drawH = p.pageH - (p.margin * 2);
            const px = Math.floor((x / drawW) * offW);
            const py = Math.floor((y / drawH) * offH);
            
            // Out of bounds = Background
            if (px < 0 || px >= offW || py < 0 || py >= offH) {
                return { bright: 255, isBackground: true };
            }

            const i = (py * offW + px) * 4;
            let r = imgData[i], g = imgData[i+1], b = imgData[i+2], a = imgData[i+3];
            
            let bright = (0.299 * r + 0.587 * g + 0.114 * b);
            
            // Detect if pixel is considered "Background"
            let isBackground = (a < 10);
            if (p.ignoreWhite && r > 240 && g > 240 && b > 240) {
                isBackground = true;
            }

            // If drawing over entire page, simulate solid background where transparent
            if (p.patternArea === 'all' && a < 255) {
                let bgBright = p.invert ? 0 : 255;
                bright = bright * (a/255) + bgBright * (1 - a/255);
            }

            if(p.invert) bright = 255 - bright;
            
            return { bright, isBackground };
        }

        function process() {
            if(!originalImage) return;
            const p = getParams();
            sampleImage(p);

            const pxPerMm = 3.78; 
            previewCanvas.width = p.pageW * pxPerMm;
            previewCanvas.height = p.pageH * pxPerMm;
            
            ctx.fillStyle = "white";
            ctx.fillRect(0, 0, previewCanvas.width, previewCanvas.height);
            ctx.fillStyle = "black";
            ctx.strokeStyle = "black";
            ctx.lineWidth = 1;

            const drawW = p.pageW - (p.margin * 2);
            const drawH = p.pageH - (p.margin * 2);
            const offW = sampleWidth;
            const offH = sampleWidth * (drawH / drawW);

            for(let y = 0; y <= drawH; y += p.spacing) {
                let rowOffset = (p.shape === 'hexagon' && (y / p.spacing) % 2 !== 0) ? (p.spacing / 2) : 0;
                
                for(let x = 0; x <= drawW; x += p.spacing) {
                    let cx = x + rowOffset;
                    if(cx > drawW) continue;

                    let { bright, isBackground } = getPixelInfo(cx, y, p, offW, offH);

                    // Area Filtering Rules
                    if (p.patternArea === 'inside' && isBackground) continue;
                    if (p.patternArea === 'outside' && !isBackground) continue;

                    if(bright < p.threshold) continue;

                    let size = p.fixedSize ? p.maxSize : p.maxSize * (bright / 255);
                    if (size < 0.1) continue;
                    
                    let r = (size / 2) * pxPerMm;
                    let drawX = (cx + p.margin) * pxPerMm;
                    let drawY = (y + p.margin) * pxPerMm;

                    ctx.beginPath();
                    if(p.shape === 'circle') {
                        ctx.arc(drawX, drawY, r, 0, Math.PI * 2);
                    } 
                    else if (p.shape === 'square') {
                        ctx.rect(drawX - r, drawY - r, r*2, r*2);
                    }
                    else if (p.shape === 'hexagon') {
                        for (let i = 0; i < 6; i++) {
                            const angle = (Math.PI / 3) * i;
                            const hx = drawX + r * Math.cos(angle);
                            const hy = drawY + r * Math.sin(angle);
                            if (i === 0) ctx.moveTo(hx, hy);
                            else ctx.lineTo(hx, hy);
                        }
                        ctx.closePath();
                    }

                    if(p.outlineOnly) ctx.stroke();
                    else ctx.fill();
                }
            }
        }

        // --- Export Generators ---
        function generateSVGString() {
            const p = getParams();
            const drawW = p.pageW - (p.margin * 2);
            const drawH = p.pageH - (p.margin * 2);
            const offW = sampleWidth;
            const offH = sampleWidth * (drawH / drawW);

            let svg = `<svg xmlns="http://www.w3.org/2000/svg" width="${p.pageW}mm" height="${p.pageH}mm" viewBox="0 0 ${p.pageW} ${p.pageH}">`;
            svg += `<rect width="100%" height="100%" fill="white"/>`;
            svg += `<g transform="translate(${p.margin}, ${p.margin})" fill="${p.outlineOnly ? 'none' : 'black'}" stroke="${p.outlineOnly ? 'black' : 'none'}" stroke-width="${p.outlineOnly ? '0.2' : '0'}">`;

            for(let y = 0; y <= drawH; y += p.spacing) {
                let rowOffset = (p.shape === 'hexagon' && (y / p.spacing) % 2 !== 0) ? (p.spacing / 2) : 0;
                for(let x = 0; x <= drawW; x += p.spacing) {
                    let cx = x + rowOffset;
                    if(cx > drawW) continue;

                    let { bright, isBackground } = getPixelInfo(cx, y, p, offW, offH);

                    // Area Filtering Rules
                    if (p.patternArea === 'inside' && isBackground) continue;
                    if (p.patternArea === 'outside' && !isBackground) continue;

                    if(bright < p.threshold) continue;

                    let size = p.fixedSize ? p.maxSize : p.maxSize * (bright / 255);
                    if (size < 0.1) continue;
                    let r = size / 2;

                    if(p.shape === 'circle') {
                        svg += `<circle cx="${cx.toFixed(2)}" cy="${y.toFixed(2)}" r="${r.toFixed(2)}"/>`;
                    } 
                    else if (p.shape === 'square') {
                        svg += `<rect x="${(cx-r).toFixed(2)}" y="${(y-r).toFixed(2)}" width="${(r*2).toFixed(2)}" height="${(r*2).toFixed(2)}"/>`;
                    }
                    else if (p.shape === 'hexagon') {
                        let pts = "";
                        for (let i = 0; i < 6; i++) {
                            const angle = (Math.PI / 3) * i;
                            pts += `${(cx + r * Math.cos(angle)).toFixed(2)},${(y + r * Math.sin(angle)).toFixed(2)} `;
                        }
                        svg += `<polygon points="${pts.trim()}"/>`;
                    }
                }
            }
            svg += `</g></svg>`;
            return svg;
        }

        function downloadSVG() {
            if(!originalImage) return alert("Please upload an image first.");
            const svgStr = generateSVGString();
            const blob = new Blob([svgStr], {type: "image/svg+xml"});
            const url = URL.createObjectURL(blob);
            const a = document.createElement("a");
            a.href = url;
            a.download = "halftone-pattern.svg";
            a.click();
        }

        function downloadPNG() {
            if(!originalImage) return alert("Please upload an image first.");
            const p = getParams();
            const scale = p.dpi / 25.4; 
            const wPx = p.pageW * scale;
            const hPx = p.pageH * scale;

            const svgStr = generateSVGString();
            const blob = new Blob([svgStr], {type: 'image/svg+xml;charset=utf-8'});
            const url = URL.createObjectURL(blob);
            
            const img = new Image();
            img.onload = () => {
                const canvas = document.createElement('canvas');
                canvas.width = wPx;
                canvas.height = hPx;
                const ctx = canvas.getContext('2d');
                ctx.drawImage(img, 0, 0, wPx, hPx);
                
                const a = document.createElement('a');
                a.href = canvas.toDataURL('image/png', 1.0);
                a.download = `halftone-${p.dpi}dpi.png`;
                a.click();
            };
            img.src = url;
        }

        function downloadPDF() {
            if(!originalImage) return alert("Please upload an image first.");
            const svgStr = generateSVGString();
            const p = getParams();
            const iframe = document.getElementById('print-frame');
            const doc = iframe.contentWindow.document;
            
            doc.open();
            doc.write(`
                <html>
                <head>
                    <style>
                        @page { size: ${p.pageW}mm ${p.pageH}mm; margin: 0; }
                        body { margin: 0; padding: 0; }
                        svg { width: 100%; height: 100%; display: block; }
                    </style>
                </head>
                <body>${svgStr}</body>
                </html>
            `);
            doc.close();

            setTimeout(() => {
                iframe.contentWindow.focus();
                iframe.contentWindow.print();
            }, 500);
        }

        // --- Zoom and Pan Interactivity ---
        function updateTransform() {
            canvasContainer.style.transform = `translate(${panX}px, ${panY}px) scale(${zoom})`;
        }

        function resetView() {
            if(!originalImage) return;
            const containerW = previewArea.clientWidth;
            const containerH = previewArea.clientHeight;
            const canvasW = previewCanvas.offsetWidth;
            const canvasH = previewCanvas.offsetHeight;
            
            zoom = Math.min((containerW - 40) / canvasW, (containerH - 40) / canvasH);
            panX = (containerW - (canvasW * zoom)) / 2;
            panY = (containerH - (canvasH * zoom)) / 2;
            updateTransform();
        }

        previewArea.addEventListener('wheel', (e) => {
            e.preventDefault();
            const zoomSpeed = 0.1;
            const oldZoom = zoom;
            if (e.deltaY < 0) zoom *= (1 + zoomSpeed);
            else zoom *= (1 - zoomSpeed);
            
            zoom = Math.max(0.1, Math.min(zoom, 10));

            const rect = previewArea.getBoundingClientRect();
            const mouseX = e.clientX - rect.left;
            const mouseY = e.clientY - rect.top;
            
            panX = mouseX - (mouseX - panX) * (zoom / oldZoom);
            panY = mouseY - (mouseY - panY) * (zoom / oldZoom);
            
            updateTransform();
        });

        previewArea.addEventListener('mousedown', (e) => {
            isDragging = true;
            startX = e.clientX - panX;
            startY = e.clientY - panY;
        });

        window.addEventListener('mousemove', (e) => {
            if (!isDragging) return;
            panX = e.clientX - startX;
            panY = e.clientY - startY;
            updateTransform();
        });

        window.addEventListener('mouseup', () => { isDragging = false; });
        window.addEventListener('resize', resetView);

    </script>
</body>
</html>
