<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <title>Solver LP Universal Mejorado - Método Simplex</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/mathjs/10.6.4/math.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background-color: #f8f9fa;
      padding: 20px;
      max-width: 1200px;
      margin: 0 auto;
      color: #333;
    }
    .container {
      display: flex;
      flex-wrap: wrap;
      gap: 20px;
    }
    .input-section,
    .output-section {
      flex: 1;
      min-width: 300px;
    }
    .card {
      background: #fff;
      padding: 25px;
      margin-bottom: 20px;
      border-radius: 10px;
      box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
      transition: transform 0.3s, box-shadow 0.3s;
    }
    textarea {
      width: 100%;
      min-height: 200px;
      padding: 15px;
      border: 1px solid #ddd;
      border-radius: 8px;
      font-size: 1em;
      line-height: 1.5;
      resize: vertical;
    }
    button {
      background-color: #4e73df;
      color: white;
      border: none;
      padding: 12px 20px;
      border-radius: 8px;
      cursor: pointer;
      font-size: 1em;
      font-weight: 600;
      transition: background-color 0.3s;
      width: 100%;
      margin-top: 10px;
    }
    button:hover {
      background-color: #2e59d9;
    }
    .error {
      color: #e74c3c;
      background-color: #fdecea;
      padding: 15px;
      border-radius: 8px;
      margin: 15px 0;
    }
    .success {
      color: #27ae60;
      background-color: #e8f5e9;
      padding: 15px;
      border-radius: 8px;
      margin: 15px 0;
    }
    .warning {
      color: #f39c12;
      background-color: #fff8e1;
      padding: 15px;
      border-radius: 8px;
      margin: 15px 0;
    }
    .info-box {
      background-color: #f8f9fc;
      border-left: 4px solid #4e73df;
      padding: 15px;
      margin: 15px 0;
    }
    canvas {
      width: 100% !important;
      max-height: 500px !important;
      margin-top: 20px;
    }
    .var-highlight {
      background-color: #fffde7;
      padding: 2px 4px;
      border-radius: 3px;
      font-weight: bold;
    }
    .loading {
      border: 4px solid #f3f3f3;
      border-top: 4px solid #3498db;
      border-radius: 50%;
      width: 30px;
      height: 30px;
      animation: spin 1s linear infinite;
      margin: 20px auto;
    }
    @keyframes spin {
      0% {
        transform: rotate(0deg);
      }
      100% {
        transform: rotate(360deg);
      }
    }
  </style>
</head>
<body>
  <h2>🔍 Solver LP Universal Mejorado - Método Simplex</h2>

  <div class="container">
    <div class="input-section">
      <div class="card">
        <h3>📝 Pegue el problema de programación lineal</h3>
        <p>
          Ejemplo:<br />
          Investment Advisors, Inc. administra portafolios de acciones... (texto libre)<br />
          El portafolio tiene $80,000 para invertir.<br />
          El índice de riesgo máximo es 700.<br />
          El portafolio está limitado a 1000 acciones de U.S. Oil.<br />
          ...
        </p>
        <textarea
          id="problema"
          rows="15"
          placeholder="Pegue aquí su problema..."
        >Investment Advisors, Inc. es una firma de corretaje que administra portafolios de acciones para varios clientes. Un portafolio en particular consta de U acciones de U.S. Oil y H acciones de Huber Steel. El rendimiento anual para U.S. Oil es $3 por acción, y para Huber Steel es $5 por acción. Las acciones de U.S. Oil se venden a $25 por acción y las de Huber Steel a $50. El portafolio tiene $80,000 para invertir. El índice de riesgo del portafolio (0.50 por acción de U.S. Oil y 0.25 por acción de Huber Steel) tiene un máximo de 700. Además, el portafolio está limitado a un máximo de 1000 acciones de U.S. Oil</textarea>
        <button onclick="analizarYResolver()">Analizar y Resolver</button>
      </div>
    </div>

    <div class="output-section">
      <div class="card" id="interpretacion">
        <h3>🔍 Interpretación</h3>
        <p>Se mostrará el análisis del problema.</p>
      </div>

      <div class="card" id="resultado">
        <h3>📊 Resultados</h3>
        <p>Ingrese un problema y pulse "Analizar y Resolver".</p>
      </div>

      <div class="card" id="tablaSimplex">
        <h3>📋 Proceso Simplex</h3>
        <p>Aquí se mostrará el detalle del método Simplex.</p>
      </div>

      <div class="card" id="graficoContainer" style="display:none;">
        <h3>📈 Visualización</h3>
        <canvas id="graficoCanvas"></canvas>
      </div>
    </div>
  </div>

  <script>
    let problemaActual = null;
    let solucionActual = null;
    let chartInstance = null;

    // Parser mejorado para texto libre (simplificado)
    function parsearProblema(texto) {
      const textoLower = texto.toLowerCase();
      const variablesSet = new Set();

      // Extraer variables (letras con posibles guiones bajos y números)
      const regexVar = /\b([a-z_][a-z0-9_]*)\b/gi;
      let match;
      while ((match = regexVar.exec(textoLower)) !== null) {
        const v = match[1];
        // Palabras comunes a ignorar
        const ignorar = [
          'acciones', 'por', 'de', 'a', 'y', 'para', 'modelo', 'estuches', 'mesas', 'sillas',
          'máximo', 'mínimo', 'riesgo', 'pedido', 'disponible', 'unidades', 'horas', 'minutos',
          'tiene', 'costo', 'ganancia', 'rendimiento', 'total', 'puede', 'entrada', 'problema',
          'max', 'min', 'unidad', 'fabricación', 'el', 'la', 'se', 'con', 'los', 'del', 'un', 'una'
        ];
        if (!ignorar.includes(v) && v.length <= 10) {
          variablesSet.add(v);
        }
      }

      let variables = Array.from(variablesSet);
      if (variables.length === 0) variables = ['x', 'y'];

      // Detectar tipo (max o min)
      const esMinimizacion =
        /minimizar|minimización|minimizar costos|minimizar costo|costo mínimo|coste mínimo/.test(textoLower);

      // Obtener coeficientes objetivo: buscar formato tipo "$3 por acción de u"
      const coefObjetivo = new Array(variables.length).fill(0);
      variables.forEach((v, i) => {
        const regexCoef = new RegExp(
          `\\$?(\\d+(?:\\.\\d+)?)[ ]*(?:por|de|para)?[ ]*${v}`,
          'i'
        );
        const m = texto.match(regexCoef);
        if (m) coefObjetivo[i] = parseFloat(m[1]);
      });
      // Si no encontró coef, asignar 1
      if (coefObjetivo.every((v) => v === 0)) {
        coefObjetivo.fill(1);
      }

      const restricciones = [];

      // Restricción inversión (ejemplo)
      const regexInversion = /\$?(\d{1,3}(?:,\d{3})*(?:\.\d+)?)[ ]*(?:para|inversión|presupuesto|disponible)/i;
      const mInv = regexInversion.exec(texto);
      if (mInv) {
        const valorInv = parseFloat(mInv[1].replace(/,/g, ''));
        const coefsInv = new Array(variables.length).fill(0);
        variables.forEach((v, i) => {
          const regexPrecio = new RegExp(`${v}[^\\d\\n]{0,15}\\$?(\\d+(?:\\.\\d+)?)`, 'i');
          const m = texto.match(regexPrecio);
          if (m) coefsInv[i] = parseFloat(m[1]);
        });
        if (coefsInv.some((c) => c !== 0)) {
          restricciones.push({
            coefficients: coefsInv,
            sign: '<=',
            value: valorInv,
            text: `Presupuesto ≤ ${valorInv}`,
          });
        }
      }

      // Restricción riesgo
      const regexRiesgoCoef = /(\d*\.?\d+)[ ]*por[ ]*acción[ ]*de[ ]*([a-z_][a-z0-9_]*)/gi;
      let riesgoLim = null;
      const regexMaxRiesgo = /(índice de riesgo|riesgo|índice)[^.\n]*\s*(?:máximo|max|min|mínimo)?\s*de\s*(\d+\.?\d*)/i;
      const mRiesgoLim = texto.match(regexMaxRiesgo);
      if (mRiesgoLim) riesgoLim = parseFloat(mRiesgoLim[2]);
      if (riesgoLim !== null) {
        const coefsRiesgo = new Array(variables.length).fill(0);
        let m;
        while ((m = regexRiesgoCoef.exec(texto)) !== null) {
          const coef = parseFloat(m[1]);
          const v = m[2];
          let idx = variables.indexOf(v);
          if (idx >= 0) coefsRiesgo[idx] = coef;
        }
        if (coefsRiesgo.some((c) => c !== 0)) {
          restricciones.push({
            coefficients: coefsRiesgo,
            sign: '<=',
            value: riesgoLim,
            text: `Índice de riesgo ≤ ${riesgoLim}`,
          });
        }
      }

      // Restricciones maximos y minimos por variable
      variables.forEach((v, i) => {
        let regexMax = new RegExp(`máximo\\s*(?:de)?\\s*(\\d+)\\s*(?:acciones|unidades)?\\s*(?:de)?\\s*${v}`, 'i');
        let mMax = texto.match(regexMax);
        if (mMax) {
          restricciones.push({
            coefficients: variables.map((x) => (x === v ? 1 : 0)),
            sign: '<=',
            value: parseFloat(mMax[1]),
            text: `${v} ≤ ${mMax[1]}`,
          });
        }
        let regexMin = new RegExp(`mínimo\\s*(?:de)?\\s*(\\d+)\\s*(?:acciones|unidades)?\\s*(?:de)?\\s*${v}`, 'i');
        let mMin = texto.match(regexMin);
        if (mMin) {
          restricciones.push({
            coefficients: variables.map((x) => (x === v ? 1 : 0)),
            sign: '>=',
            value: parseFloat(mMin[1]),
            text: `${v} ≥ ${mMin[1]}`,
          });
        }
        // Forzar no negatividad:
        restricciones.push({
          coefficients: variables.map((x) => (x === v ? 1 : 0)),
          sign: '>=',
          value: 0,
          text: `${v} ≥ 0`,
        });
      });

      return {
        variables,
        objetivo: {
          coefficients: coefObjetivo,
          tipo: esMinimizacion ? 'min' : 'max',
          texto: esMinimizacion ? 'Minimizar función objetivo' : 'Maximizar función objetivo',
        },
        restricciones,
        textoOriginal: texto,
      };
    }

    // Función principal para analizar y resolver
    function analizarYResolver() {
      const texto = document.getElementById('problema').value.trim();

      if (!texto) {
        mostrarError('Por favor ingrese un problema para analizar.');
        return;
      }

      document.getElementById('resultado').innerHTML =
        '<div class="loading"></div><p>Analizando el problema, por favor espere...</p>';
      document.getElementById('interpretacion').innerHTML =
        '<div class="loading"></div><p>Procesando descripción del problema...</p>';
      document.getElementById('tablaSimplex').innerHTML = '<p>Se mostrará aquí el proceso Simplex.</p>';
      document.getElementById('graficoContainer').style.display = 'none';

      setTimeout(() => {
        try {
          problemaActual = parsearProblema(texto);
          mostrarInterpretacion(problemaActual);
          solucionActual = resolverProblemaSimplex(problemaActual);
          mostrarResultados(solucionActual, problemaActual);
        } catch (e) {
          mostrarError('Error al procesar el problema: ' + e.message);
          console.error(e);
        }
      }, 100);
    }

    // Mostrar interpretación
    function mostrarInterpretacion(problema) {
      let html = `<h4>📌 Tipo de Problema</h4>`;
      html += `<p><strong>${problema.objetivo.tipo === 'min' ? 'Minimización' : 'Maximización'}:</strong> ${
        problema.objetivo.texto
      }</p>`;

      html += `<h4>🔢 Variables de Decisión</h4><ul>`;
      problema.variables.forEach((v, i) => {
        html += `<li><span class="var-highlight">${v}</span>: Coeficiente = ${problema.objetivo.coefficients[i]}</li>`;
      });
      html += `</ul>`;

      if (problema.restricciones.length > 0) {
        html += `<h4>📏 Restricciones Identificadas</h4><table><tr><th>Restricción</th><th>Expresión</th></tr>`;
        problema.restricciones.forEach((r) => {
          const expr = problema.variables
            .map((v, i) => (r.coefficients[i] !== 0 ? `${r.coefficients[i]}${v}` : ''))
            .filter((x) => x !== '')
            .join(' + ');

          html += `<tr><td>${r.text}</td><td>${expr} ${r.sign} ${r.value}</td></tr>`;
        });
        html += `</table>`;
      } else {
        html += `<p><em>No se identificaron restricciones específicas.</em></p>`;
      }

      document.getElementById('interpretacion').innerHTML = html;
    }

    // Mostrar error
    function mostrarError(mensaje) {
      document.getElementById('resultado').innerHTML = `
        <div class="error">
          <h3>❌ Error</h3>
          <p>${mensaje}</p>
        </div>`;
      document.getElementById('tablaSimplex').innerHTML = '';
      document.getElementById('interpretacion').innerHTML = '';
      document.getElementById('graficoContainer').style.display = 'none';
      if (chartInstance) {
        chartInstance.destroy();
        chartInstance = null;
      }
    }

    // Mantén la función resolverProblemaSimplex, simplexIterations, mostrarResultados y mostrarTablaSimplex iguales a las que tienes
    // Solo recuerda actualizar mostrarGrafico para manejar el chartInstance:

    function mostrarGrafico(solution, problema) {
      const ctx = document.getElementById('graficoCanvas').getContext('2d');
      if (chartInstance) {
        chartInstance.destroy();
      }

      const puntosFactibles = generarPuntosFactibles(problema);

      // Setup datasets para Chart.js
      const datasets = [
        {
          label: 'Región Factible',
          data: puntosFactibles,
          backgroundColor: 'rgba(54, 162, 235, 0.5)',
          pointRadius: 3,
          showLine: true,
          borderColor: 'rgba(54, 162, 235, 0.8)',
          fill: true,
        },
      ];

      if (solution.isOptimal && solution.variables.length === 2) {
        datasets.push({
          label: 'Solución Óptima',
          data: [{ x: solution.variables[0], y: solution.variables[1] }],
          backgroundColor: 'rgba(255, 99, 132, 1)',
          pointRadius: 8,
        });
      }

      problema.restricciones.forEach((r, i) => {
        if (r.coefficients.length === 2 && r.coefficients[0] !== 0 && r.coefficients[1] !== 0) {
          const x1 = 0;
          const y1 = (r.value - r.coefficients[0] * x1) / r.coefficients[1];
          const y2 = 0;
          const x2 = (r.value - r.coefficients[1] * y2) / r.coefficients[0];

          if (y1 >= 0 || x2 >= 0) {
            const points = [];
            if (y1 >= 0) points.push({ x: x1, y: y1 });
            if (x2 >= 0) points.push({ x: x2, y: y2 });

            if (points.length >= 2) {
              datasets.push({
                label: `Restricción ${i + 1}`,
                data: points,
                borderColor: `hsl(${i * 60}, 70%, 50%)`,
                borderWidth: 2,
                pointRadius: 0,
                fill: false,
                showLine: true,
              });
            }
          }
        }
      });

      chartInstance = new Chart(ctx, {
        type: 'scatter',
        data: {
          datasets: datasets,
        },
        options: {
          responsive: true,
          plugins: {
            legend: { position: 'top' },
            tooltip: {
              callbacks: {
                label: function (context) {
                  return `${problema.variables[0]}: ${context.parsed.x.toFixed(
                    2
                  )}, ${problema.variables[1]}: ${context.parsed.y.toFixed(2)}`;
                },
              },
            },
          },
          scales: {
            x: {
              title: { display: true, text: problema.variables[0] },
              min: 0,
              grid: { color: '#ddd' },
            },
            y: {
              title: { display: true, text: problema.variables[1] },
              min: 0,
              grid: { color: '#ddd' },
            },
          },
        },
      });

      document.getElementById('graficoContainer').style.display = 'block';
    }

    // Mantén también la función generarPuntosFactibles igual que en tu código original

  </script>
</body>
</html>
