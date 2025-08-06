<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8" />
  <title>Solver LP Universal - Método Simplex</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/mathjs/10.6.4/math.min.js"></script>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    /* ... mismos estilos de antes para cuerpo, botones, tablas, etc ... */
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
    button.secondary {
      background-color: #858796;
    }
    button.secondary:hover {
      background-color: #6c757d;
    }
    table {
      width: 100%;
      border-collapse: collapse;
      margin: 15px 0;
      font-size: 0.9em;
    }
    th,
    td {
      border: 1px solid #ddd;
      padding: 12px;
      text-align: center;
    }
    th {
      background-color: #f2f2f2;
      font-weight: 600;
    }
    tr:nth-child(even) {
      background-color: #f9f9f9;
    }
    h2,
    h3,
    h4 {
      color: #2e59d9;
      margin-top: 0;
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
  <h2>🔍 Solver LP Universal - Método Simplex (Entrada estructurada)</h2>

  <div class="container">
    <div class="input-section">
      <div class="card">
        <h3>📝 Ingrese el problema</h3>
        <p>Formato requerido:<br />
          <strong>Función objetivo:</strong> max z = 3u + 5h <br />
          <strong>Restricciones (una por línea):</strong><br />
          25u + 50h <= 80000<br />
          0.5u + 0.25h <= 700<br />
          u <= 1000<br />
          u >= 0<br />
          h >= 0
        </p>
        <textarea
          id="problema"
          rows="12"
          placeholder="max z = 3u + 5h&#10;25u + 50h <= 80000&#10;0.5u + 0.25h <= 700&#10;u <= 1000&#10;u >= 0&#10;h >= 0"
        ></textarea>
        <button onclick="analizarYResolver()">Analizar y Resolver Problema</button>
      </div>
    </div>

    <div class="output-section">
      <div class="card" id="interpretacion">
        <h3>🔍 Interpretación del Problema</h3>
        <p>Se mostrará aquí el análisis del problema ingresado.</p>
      </div>

      <div class="card" id="resultado">
        <h3>📊 Resultados</h3>
        <p>Ingrese un problema y haga clic en "Analizar y Resolver" para ver los resultados.</p>
      </div>

      <div class="card" id="tablaSimplex">
        <h3>📋 Proceso Simplex</h3>
        <p>Se mostrará aquí el detalle del método Simplex aplicado.</p>
      </div>

      <div class="card" id="graficoContainer" style="display: none;">
        <h3>📈 Visualización</h3>
        <canvas id="graficoCanvas"></canvas>
      </div>
    </div>
  </div>

  <script>
    // Variables globales
    let problemaActual = null;
    let solucionActual = null;
    let chartInstance = null;

    // Función para parsear entrada estructurada
    function parsearProblemaEstructurado(texto) {
      const lineas = texto
        .split('\n')
        .map((l) => l.trim())
        .filter((l) => l.length > 0);
      if (lineas.length === 0)
        throw new Error('El texto está vacío o inválido.');

      // Parse función objetivo: esperar formato "max z = 3u + 5h" o "min z = 10x + 6y"
      const objetivoLinea = lineas[0].toLowerCase();
      let tipo = null;
      if (objetivoLinea.startsWith('max'))
        tipo = 'max';
      else if (objetivoLinea.startsWith('min'))
        tipo = 'min';
      else throw new Error('La función objetivo debe comenzar con "max" o "min".');

      // Extraer término derecho (todo después del '=')
      const eqIdx = objetivoLinea.indexOf('=');
      if (eqIdx === -1) throw new Error('Función objetivo debe contener "=".');
      const funcionObjetivoTexto = objetivoLinea.substring(eqIdx + 1).trim();

      // Extraer variables usadas en función objetivo
      const variablesSet = new Set();
      const variablesRegEx = /[a-z]\w*/g;
      let match;
      while ((match = variablesRegEx.exec(funcionObjetivoTexto)) !== null) {
        variablesSet.add(match[0]);
      }
      if (variablesSet.size === 0)
        throw new Error('No se detectaron variables en la función objetivo.');

      const variables = Array.from(variablesSet);

      // Parse coeficientes de función objetivo
      const coefsObj = {};
      // Ejemplo: "3u + 5h - u2" formateamos y usamos mathjs para analizar los coeficientes
      // Convierte cada término: coef * variable
      // Para eso separaremos por operadores y extraeremos coeficientes
      variables.forEach((v) => {
        const regexTerm = new RegExp(`([+-]?\\s*\\d*\\.?\\d*)\\s*${v}`, 'g');
        let suma = 0;
        let t;
        while ((t = regexTerm.exec(funcionObjetivoTexto)) !== null) {
          let numStr = t[1].replace(/\s+/g, '');
          if (numStr === '' || numStr === '+') numStr = '1';
          else if (numStr === '-') numStr = '-1';
          suma += parseFloat(numStr);
        }
        coefsObj[v] = suma;
      });

      // Convert coeficientes en arreglo en orden de variables
      const coefs = variables.map((v) => coefsObj[v] || 0);

      // Parse restricciones (líneas 2 en adelante)
      const restricciones = [];
      for (let i = 1; i < lineas.length; i++) {
        const linea = lineas[i];

        // Match por formato coef var + coef var ... signo valor
        // Separar por <=, >=, =
        let signoMatch = null;
        let signo = null;
        if (linea.includes('<='))
          signo = '<=';
        else if (linea.includes('>='))
          signo = '>=';
        else if (linea.includes('='))
          signo = '==';
        else throw new Error(`Restricción en línea ${i + 1} debe contener '<=', '>=', o '='`);

        const parts = linea.split(signo);
        if (parts.length !== 2) throw new Error(`Restricción en línea ${i + 1} mal formada.`);

        const lhs = parts[0].trim();
        const rhs = parseFloat(parts[1].trim());

        if (isNaN(rhs)) throw new Error(`Restricción en línea ${i + 1} debe tener un número a la derecha.`);

        // Parsear coeficientes lhs por variable
        const coefsR = new Array(variables.length).fill(0);

        // Para extraer términos tipo "+ 25u", "- 0.5h", "u" 
        // Regex para términos: captura signo opcional + número opcional + variable
        const termRegex = /([+-]?\s*\d*\.?\d*)\s*([a-z]\w*)/gi;
        let m;
        while ((m = termRegex.exec(lhs)) !== null) {
          let coefStr = m[1].replace(/\s/g, '');
          const v = m[2];
          let coefVal = 1;
          if (coefStr === '+' || coefStr === '') coefVal = 1;
          else if (coefStr === '-') coefVal = -1;
          else coefVal = parseFloat(coefStr);
          const idx = variables.indexOf(v);
          if (idx === -1) throw new Error(`Variable '${v}' en línea ${i + 1} no encontrada en la función objetivo.`);
          coefsR[idx] += coefVal;
        }

        restricciones.push({
          coefficients: coefsR,
          sign,
          value: rhs,
          text: linea,
        });
      }

      return {
        variables,
        objetivo: {
          coefficients: coefs,
          tipo,
          texto: `${tipo === 'max' ? 'Maximizar' : 'Minimizar'} función objetivo`,
        },
        restricciones,
        textoOriginal: texto,
      };
    }

    // Función principal
    function analizarYResolver() {
      const textoProblema = document.getElementById('problema').value.trim();

      if (!textoProblema) {
        mostrarError('Por favor ingrese un problema para analizar.');
        return;
      }

      // Mostrar carga
      document.getElementById('resultado').innerHTML =
        '<div class="loading"></div><p>Analizando problema, por favor espere...</p>';
      document.getElementById('interpretacion').innerHTML =
        '<div class="loading"></div><p>Procesando descripción del problema...</p>';
      document.getElementById('tablaSimplex').innerHTML =
        '<p>Se mostrará el proceso Simplex aquí.</p>';
      document.getElementById('graficoContainer').style.display = 'none';

      setTimeout(() => {
        try {
          problemaActual = parsearProblemaEstructurado(textoProblema);
          mostrarInterpretacion(problemaActual);

          solucionActual = resolverProblemaSimplex(problemaActual);
          mostrarResultados(solucionActual, problemaActual);
        } catch (e) {
          mostrarError('Error al procesar el problema: ' + e.message);
          console.error(e);
        }
      }, 100);
    }

    // Mostrar interpretación del problema
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

    // --- Método Simplex y demás funciones (usa las mismas que proporcionaste, sin cambios sustanciales)
    
    // Para no extenderme, aquí mantén toda tu función resolverProblemaSimplex y demás ---

    // Variables globales de Simplex y función simplexIterations igual que antes
    // ...

    // Para copiar, usa las funciones resolverProblemaSimplex, simplexIterations, mostrarResultados,
    // mostrarTablaSimplex y mostrarGrafico tal cual las tienes, pero asegurándote que showGrafico
    // use la variable global chartInstance para manejar el gráfico Chart.js con destrucción previa.

    // Aquí te dejo el cambio en mostrarGrafico para evitar el error del canvas:

    function mostrarGrafico(solution, problema) {
      const ctx = document.getElementById('graficoCanvas').getContext('2d');

      if (chartInstance) chartInstance.destroy();

      const puntosFactibles = generarPuntosFactibles(problema);

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

    // ---

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
      
      if(chartInstance) {
        chartInstance.destroy();
        chartInstance = null;
      }
    }

    // Puedes copiar y pegar el resto de tus funciones del método simplex sin cambio.

  </script>
</body>
</html>
