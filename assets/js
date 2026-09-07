/* =========================================================================
   CutPic — lógica da ferramenta
   Todo o processamento acontece no navegador. Nenhum dado é enviado
   a um servidor em nenhum momento.
   ========================================================================= */

(function () {
  "use strict";

  const dropzone = document.getElementById("dropzone");
  const fileInput = document.getElementById("fileInput");
  const previewWrap = document.getElementById("previewWrap");
  const previewFrame = document.getElementById("previewFrame");
  const previewImg = document.getElementById("previewImg");
  const overlayCanvas = document.getElementById("overlayCanvas");
  const fileNameEl = document.getElementById("fileName");
  const changeImageBtn = document.getElementById("changeImageBtn");

  const rowsInput = document.getElementById("rowsInput");
  const colsInput = document.getElementById("colsInput");
  const rowsMinus = document.getElementById("rowsMinus");
  const rowsPlus = document.getElementById("rowsPlus");
  const colsMinus = document.getElementById("colsMinus");
  const colsPlus = document.getElementById("colsPlus");
  const pageCountEl = document.getElementById("pageCount");

  const generateBtn = document.getElementById("generateBtn");
  const statusLine = document.getElementById("statusLine");

  const MIN_CELLS = 1;
  const MAX_CELLS = 12;
  const MAX_SOURCE_DIMENSION = 5000; // limite de segurança para não travar o navegador

  // Guarda a imagem carregada (elemento Image) apenas em memória.
  // Nada é persistido em disco, localStorage ou enviado à rede.
  let sourceImage = null;
  let sourceObjectUrl = null;

  function clamp(value, min, max) {
    return Math.min(max, Math.max(min, value));
  }

  function getRows() {
    return clamp(parseInt(rowsInput.value, 10) || 1, MIN_CELLS, MAX_CELLS);
  }

  function getCols() {
    return clamp(parseInt(colsInput.value, 10) || 1, MIN_CELLS, MAX_CELLS);
  }

  function updatePageCount() {
    const rows = getRows();
    const cols = getCols();
    pageCountEl.textContent = String(rows * cols);
  }

  function setStatus(message, isOk) {
    statusLine.textContent = message;
    statusLine.classList.toggle("ok", Boolean(isOk));
  }

  /* ---------------------------- upload handling ---------------------------- */

  function openFileDialog() {
    fileInput.click();
  }

  dropzone.addEventListener("click", openFileDialog);
  dropzone.addEventListener("keydown", (e) => {
    if (e.key === "Enter" || e.key === " ") {
      e.preventDefault();
      openFileDialog();
    }
  });

  ["dragenter", "dragover"].forEach((evt) => {
    dropzone.addEventListener(evt, (e) => {
      e.preventDefault();
      e.stopPropagation();
      dropzone.classList.add("dragover");
    });
  });

  ["dragleave", "dragend", "drop"].forEach((evt) => {
    dropzone.addEventListener(evt, (e) => {
      e.preventDefault();
      e.stopPropagation();
      dropzone.classList.remove("dragover");
    });
  });

  dropzone.addEventListener("drop", (e) => {
    const file = e.dataTransfer.files && e.dataTransfer.files[0];
    if (file) handleFile(file);
  });

  fileInput.addEventListener("change", () => {
    const file = fileInput.files && fileInput.files[0];
    if (file) handleFile(file);
  });

  changeImageBtn.addEventListener("click", () => {
    resetImage();
    openFileDialog();
  });

  function handleFile(file) {
    if (!file.type.startsWith("image/")) {
      setStatus("Formato não suportado. Envie uma imagem PNG, JPG ou WEBP.", false);
      return;
    }

    const maxBytes = 30 * 1024 * 1024; // 30 MB
    if (file.size > maxBytes) {
      setStatus("Arquivo muito grande. Envie uma imagem de até 30 MB.", false);
      return;
    }

    revokeCurrentUrl();
    sourceObjectUrl = URL.createObjectURL(file);

    const img = new Image();
    img.onload = () => {
      if (img.naturalWidth > MAX_SOURCE_DIMENSION || img.naturalHeight > MAX_SOURCE_DIMENSION) {
        setStatus(`Imagem muito grande. O limite é ${MAX_SOURCE_DIMENSION}px no maior lado.`, false);
        revokeCurrentUrl();
        return;
      }

      sourceImage = img;
      previewImg.src = sourceObjectUrl;
      fileNameEl.textContent = file.name;
      dropzone.classList.add("hidden");
      previewWrap.classList.add("active");
      generateBtn.disabled = false;
      setStatus("Imagem carregada. Ajuste a grade e gere o PDF quando quiser.", true);

      // desenha a grade assim que o layout da prévia estiver pronto
      requestAnimationFrame(drawGridOverlay);
    };
    img.onerror = () => {
      setStatus("Não foi possível ler essa imagem. Tente outro arquivo.", false);
      revokeCurrentUrl();
    };
    img.src = sourceObjectUrl;
  }

  function revokeCurrentUrl() {
    if (sourceObjectUrl) {
      URL.revokeObjectURL(sourceObjectUrl);
      sourceObjectUrl = null;
    }
  }

  function resetImage() {
    sourceImage = null;
    revokeCurrentUrl();
    previewImg.removeAttribute("src");
    fileInput.value = "";
    dropzone.classList.remove("hidden");
    previewWrap.classList.remove("active");
    generateBtn.disabled = true;
    setStatus("Envie uma imagem para começar.", false);
    const ctx = overlayCanvas.getContext("2d");
    ctx.clearRect(0, 0, overlayCanvas.width, overlayCanvas.height);
  }

  /* -------------------------- grid preview overlay -------------------------- */

  function drawGridOverlay() {
    if (!sourceImage) return;

    const frameRect = previewFrame.getBoundingClientRect();
    const dpr = window.devicePixelRatio || 1;

    overlayCanvas.width = frameRect.width * dpr;
    overlayCanvas.height = frameRect.height * dpr;

    const ctx = overlayCanvas.getContext("2d");
    ctx.setTransform(dpr, 0, 0, dpr, 0, 0);
    ctx.clearRect(0, 0, frameRect.width, frameRect.height);

    // calcula o retângulo real ocupado pela imagem dentro do frame,
    // já que object-fit: contain deixa faixas vazias nas bordas
    const imgRatio = sourceImage.naturalWidth / sourceImage.naturalHeight;
    const frameRatio = frameRect.width / frameRect.height;

    let renderW, renderH, offsetX, offsetY;
    if (imgRatio > frameRatio) {
      renderW = frameRect.width;
      renderH = renderW / imgRatio;
      offsetX = 0;
      offsetY = (frameRect.height - renderH) / 2;
    } else {
      renderH = frameRect.height;
      renderW = renderH * imgRatio;
      offsetY = 0;
      offsetX = (frameRect.width - renderW) / 2;
    }

    const rows = getRows();
    const cols = getCols();
    const cellW = renderW / cols;
    const cellH = renderH / rows;

    ctx.strokeStyle = "rgba(51, 217, 198, 0.85)";
    ctx.lineWidth = 1.25;
    ctx.setLineDash([5, 4]);

    // linhas verticais
    for (let c = 1; c < cols; c++) {
      const x = offsetX + c * cellW;
      ctx.beginPath();
      ctx.moveTo(x, offsetY);
      ctx.lineTo(x, offsetY + renderH);
      ctx.stroke();
    }
    // linhas horizontais
    for (let r = 1; r < rows; r++) {
      const y = offsetY + r * cellH;
      ctx.beginPath();
      ctx.moveTo(offsetX, y);
      ctx.lineTo(offsetX + renderW, y);
      ctx.stroke();
    }

    // moldura sólida ao redor da imagem inteira
    ctx.setLineDash([]);
    ctx.strokeStyle = "rgba(255, 93, 58, 0.9)";
    ctx.lineWidth = 1.5;
    ctx.strokeRect(offsetX + 0.75, offsetY + 0.75, renderW - 1.5, renderH - 1.5);

    // marcas de corte (crop marks) em todos os vértices da grade — inclui
    // as bordas externas (cantos de cada futura página) e as interseções
    // internas, no mesmo padrão usado na geração do PDF
    for (let r = 0; r <= rows; r++) {
      for (let c = 0; c <= cols; c++) {
        const x = offsetX + c * cellW;
        const y = offsetY + r * cellH;
        drawCanvasCropMark(ctx, x, y);
      }
    }
  }

  // Marca de registro/corte no estilo gráfico: um pequeno círculo vazado
  // com mira central e um espaço em branco exatamente no ponto de corte,
  // para indicar com precisão onde a tesoura/estilete deve passar.
  function drawCanvasCropMark(ctx, x, y) {
    const armLen = 5;
    const gap = 2;
    const radius = 2.5;

    ctx.save();
    ctx.lineCap = "round";

    // contorno branco para garantir contraste sobre qualquer área da imagem
    ctx.strokeStyle = "rgba(20, 22, 31, 0.85)";
    ctx.lineWidth = 2.4;
    drawCropMarkStrokes(ctx, x, y, armLen, gap, radius);

    // traço fino em coral por cima, a cor de "corte" da identidade do CutPic
    ctx.strokeStyle = "rgba(255, 93, 58, 0.95)";
    ctx.lineWidth = 1;
    drawCropMarkStrokes(ctx, x, y, armLen, gap, radius);

    ctx.restore();
  }

  function drawCropMarkStrokes(ctx, x, y, armLen, gap, radius) {
    // quatro pequenos traços radiais, cada um começando a `gap` px do
    // ponto exato (para não manchar o ponto de corte) e um círculo fino
    // de precisão centralizado nele
    ctx.beginPath();
    ctx.moveTo(x - gap - armLen, y);
    ctx.lineTo(x - gap, y);
    ctx.moveTo(x + gap, y);
    ctx.lineTo(x + gap + armLen, y);
    ctx.moveTo(x, y - gap - armLen);
    ctx.lineTo(x, y - gap);
    ctx.moveTo(x, y + gap);
    ctx.lineTo(x, y + gap + armLen);
    ctx.stroke();

    ctx.beginPath();
    ctx.arc(x, y, radius, 0, Math.PI * 2);
    ctx.stroke();
  }

  window.addEventListener("resize", () => {
    if (sourceImage) drawGridOverlay();
  });

  function onGridChange() {
    updatePageCount();
    if (sourceImage) drawGridOverlay();
  }

  rowsInput.addEventListener("input", onGridChange);
  colsInput.addEventListener("input", onGridChange);
  rowsInput.addEventListener("blur", () => {
    rowsInput.value = getRows();
    onGridChange();
  });
  colsInput.addEventListener("blur", () => {
    colsInput.value = getCols();
    onGridChange();
  });

  function stepInput(input, delta) {
    const current = clamp(parseInt(input.value, 10) || 1, MIN_CELLS, MAX_CELLS);
    input.value = clamp(current + delta, MIN_CELLS, MAX_CELLS);
    onGridChange();
  }

  rowsMinus.addEventListener("click", () => stepInput(rowsInput, -1));
  rowsPlus.addEventListener("click", () => stepInput(rowsInput, 1));
  colsMinus.addEventListener("click", () => stepInput(colsInput, -1));
  colsPlus.addEventListener("click", () => stepInput(colsInput, 1));

  updatePageCount();

  /* ------------------------------ PDF generation ----------------------------- */

  const A4_WIDTH_MM = 210;
  const A4_HEIGHT_MM = 297;

  // Margem reservada em cada página para as marcas de corte e guias de
  // alinhamento. A imagem é impressa dentro dessa área útil (inner rect);
  // a margem em branco ao redor é o que garante precisão no corte, já que
  // a maioria das impressoras não imprime até a borda exata do papel.
  const PAGE_MARGIN_MM = 8;
  const MARK_GAP_MM = 1.4; // distância da marca até o ponto exato de corte
  const MARK_LEN_MM = 4.2; // comprimento de cada traço da marca de corte
  const TICK_LEN_MM = 2.4; // comprimento das guias de alinhamento no meio de cada borda
  const MARK_LINE_MM = 0.18;
  const TRIM_LINE_MM = 0.12;

  function slugify(name) {
    return name
      .replace(/\.[^/.]+$/, "")
      .normalize("NFD")
      .replace(/[\u0300-\u036f]/g, "")
      .replace(/[^a-zA-Z0-9]+/g, "-")
      .replace(/(^-|-$)/g, "")
      .toLowerCase() || "cutpic";
  }

  /**
   * Desenha, em milímetros, as marcas de corte (crop marks) e as guias de
   * alinhamento ao redor da área útil de uma página do PDF.
   *
   * - Marcas de canto: quatro traços em "L" próximos a cada vértice da
   *   área útil, com um pequeno vão até o ponto exato — padrão usado em
   *   gráficas para indicar onde cortar sem manchar o ponto de corte.
   * - Guia de trim: uma linha tracejada fina contornando toda a área útil,
   *   servindo de referência contínua para o corte com régua ou estilete.
   * - Ticks de alinhamento: pequenas marcas no meio de cada borda, que
   *   ajudam a alinhar duas páginas adjacentes com precisão na hora de
   *   colar, já que a mesma posição se repete na folha vizinha.
   */
  function drawPdfCropMarksAndGuides(pdf, x, y, w, h) {
    const x2 = x + w;
    const y2 = y + h;

    // 1) guia de trim: contorno tracejado fino ao redor de toda a área útil
    pdf.setDrawColor(190, 190, 190);
    pdf.setLineWidth(TRIM_LINE_MM);
    if (typeof pdf.setLineDashPattern === "function") {
      pdf.setLineDashPattern([1.2, 1], 0);
    }
    pdf.rect(x, y, w, h, "S");
    if (typeof pdf.setLineDashPattern === "function") {
      pdf.setLineDashPattern([], 0);
    }

    // 2) marcas de corte nos 4 cantos (traço duplo em "L", com vão até o vértice)
    pdf.setDrawColor(40, 40, 40);
    pdf.setLineWidth(MARK_LINE_MM);

    drawCornerMark(pdf, x, y, -1, -1); // superior esquerdo
    drawCornerMark(pdf, x2, y, 1, -1); // superior direito
    drawCornerMark(pdf, x, y2, -1, 1); // inferior esquerdo
    drawCornerMark(pdf, x2, y2, 1, 1); // inferior direito

    // 3) guias de alinhamento no meio de cada borda, para casar páginas vizinhas
    drawEdgeTick(pdf, x + w / 2, y, 0, -1); // borda superior
    drawEdgeTick(pdf, x + w / 2, y2, 0, 1); // borda inferior
    drawEdgeTick(pdf, x, y + h / 2, -1, 0); // borda esquerda
    drawEdgeTick(pdf, x2, y + h / 2, 1, 0); // borda direita
  }

  // Traço em "L" nas proximidades de um canto: dois segmentos, cada um
  // começando a MARK_GAP_MM do vértice exato e apontando para fora da
  // área útil, dentro da margem reservada da página.
  function drawCornerMark(pdf, cx, cy, dirX, dirY) {
    const gap = MARK_GAP_MM;
    const len = MARK_LEN_MM;

    // traço horizontal
    pdf.line(cx + dirX * gap, cy, cx + dirX * (gap + len), cy);
    // traço vertical
    pdf.line(cx, cy + dirY * gap, cx, cy + dirY * (gap + len));
  }

  // Pequeno traço perpendicular à borda, centralizado em cada lado da
  // área útil, usado como guia extra de alinhamento entre páginas vizinhas.
  function drawEdgeTick(pdf, cx, cy, dirX, dirY) {
    const gap = MARK_GAP_MM;
    const len = TICK_LEN_MM;
    pdf.line(cx + dirX * gap, cy + dirY * gap, cx + dirX * (gap + len), cy + dirY * (gap + len));
  }

  async function generatePdf() {
    if (!sourceImage) return;

    const rows = getRows();
    const cols = getCols();
    const total = rows * cols;

    generateBtn.disabled = true;
    setStatus(`Gerando PDF (0 de ${total} páginas)...`, false);

    // pequena pausa para o navegador pintar o estado "gerando" antes do trabalho pesado
    await new Promise((resolve) => setTimeout(resolve, 30));

    try {
      const { jsPDF } = window.jspdf;
      const pdf = new jsPDF({
        orientation: "portrait",
        unit: "mm",
        format: "a4",
        compress: true,
      });

      const cellWidthPx = sourceImage.naturalWidth / cols;
      const cellHeightPx = sourceImage.naturalHeight / rows;

      const sliceCanvas = document.createElement("canvas");
      sliceCanvas.width = Math.round(cellWidthPx);
      sliceCanvas.height = Math.round(cellHeightPx);
      const sliceCtx = sliceCanvas.getContext("2d");

      // área útil da página: a imagem ocupa exatamente este retângulo em
      // todas as páginas, o que garante que o tamanho recortado seja
      // idêntico em cada uma — essencial para o encaixe preciso ao colar
      const innerX = PAGE_MARGIN_MM;
      const innerY = PAGE_MARGIN_MM;
      const innerW = A4_WIDTH_MM - PAGE_MARGIN_MM * 2;
      const innerH = A4_HEIGHT_MM - PAGE_MARGIN_MM * 2;

      let pageIndex = 0;

      for (let r = 0; r < rows; r++) {
        for (let c = 0; c < cols; c++) {
          const sx = Math.round(c * cellWidthPx);
          const sy = Math.round(r * cellHeightPx);
          const sw = Math.round(cellWidthPx);
          const sh = Math.round(cellHeightPx);

          sliceCtx.clearRect(0, 0, sliceCanvas.width, sliceCanvas.height);
          sliceCtx.drawImage(sourceImage, sx, sy, sw, sh, 0, 0, sliceCanvas.width, sliceCanvas.height);

          const dataUrl = sliceCanvas.toDataURL("image/jpeg", 0.92);

          if (pageIndex > 0) {
            pdf.addPage("a4", "portrait");
          }

          pdf.addImage(dataUrl, "JPEG", innerX, innerY, innerW, innerH, undefined, "FAST");

          drawPdfCropMarksAndGuides(pdf, innerX, innerY, innerW, innerH);

          // referência de montagem no canto de cada página: posição na
          // grade (linha/coluna) e ordem sequencial de impressão
          pdf.setFont("helvetica", "normal");
          pdf.setFontSize(7);
          pdf.setTextColor(120, 120, 120);
          pdf.text(
            `L${r + 1} · C${c + 1}  —  ${pageIndex + 1}/${total}`,
            innerX + 6,
            innerY + innerH + PAGE_MARGIN_MM / 2,
            { baseline: "middle" }
          );

          pageIndex++;
          setStatus(`Gerando PDF (${pageIndex} de ${total} páginas)...`, false);
          // libera a thread principal periodicamente para manter a interface responsiva
          if (pageIndex % 4 === 0) {
            await new Promise((resolve) => setTimeout(resolve, 0));
          }
        }
      }

      const baseName = fileNameEl.textContent ? slugify(fileNameEl.textContent) : "cutpic";
      pdf.save(`${baseName}-${rows}x${cols}.pdf`);

      setStatus(`PDF gerado com ${total} páginas. Sua imagem já foi descartada da memória.`, true);
    } catch (err) {
      console.error(err);
      setStatus("Não foi possível gerar o PDF. Tente novamente.", false);
    } finally {
      generateBtn.disabled = false;
    }
  }

  generateBtn.addEventListener("click", generatePdf);

  // Ao fechar ou navegar para fora da página, garante que a URL do objeto
  // criada para a imagem seja liberada da memória do navegador.
  window.addEventListener("beforeunload", revokeCurrentUrl);
})();
