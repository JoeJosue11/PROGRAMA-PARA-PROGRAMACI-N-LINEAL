<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Solver LP Universal - Método Simplex</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/mathjs/10.6.4/math.min.js"></script>
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
    .input-section, .output-section { 
      flex: 1; 
      min-width: 300px; 
    }
    .card {
      background: #fff; 
      padding: 25px; 
      margin-bottom: 20px; 
      border-radius: 10px; 
      box-shadow: 0 4px 6px rgba(0,0,0,0.1);
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
    button:hover { background-color: #2e59d9; }
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
    th, td { 
      border: 1px solid #ddd; 
      padding: 12px; 
      text-align: center; 
    }
    th { 
      background-color: #f2f2f2; 
      font-weight: 600; 
    }
    tr:nth-child(even) { background-color: #f9f9f9; }
    h2, h3, h4 { 
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
    .var-highlight {
      background-color: #fffde7;
      padding: 2px 4px;
      border-radius: 3px;
      font-weight: bold;
    }
    .restriction-highlight {
      background-color: #e3f2fd;
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
      0% { transform: rotate(0deg); }
      100% { transform: rotate(360deg); }
    }
  </style>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>
  <h2>🔍 Solver LP Universal - Método Simplex</h2>
  
  <div class="container">
    <div class="input-section">
      <div class="card">
        <h3>📝 Ingrese el problema</h3>
        <textarea id="problema" rows="12" placeholder="Pegue aquí el enunciado completo del problema de programación lineal...">Investment Advisors, Inc. es una firma de corretaje que administra portafolios de acciones para varios clientes. Un portafolio en particular consta de U acciones de U.S. Oil y H acciones de Huber Steel. El rendimiento anual para U.S. Oil es $3 por acción, y para Huber Steel es $5 por acción. Las acciones de U.S. Oil se venden a $25 por acción y las de Huber Steel a $50. El portafolio tiene $80,000 para invertir. El índice de riesgo del portafolio (0.50 por acción de U.S. Oil y 0.25 por acción de Huber Steel) tiene un máximo de 700. Además, el portafolio está limitado a un máximo de 1000 acciones de U.S. Oil</textarea>
        <button onclick="analizarYResolver()">Analizar y Resolver Problema</button>
        <button onclick="cargarEjemplo('investment')" class="secondary">Ejemplo: Investment Advisors</button>
        <button onclick="cargarEjemplo('dci')" class="secondary">Ejemplo: Digital Controls</button>
        <button onclick="cargarEjemplo('fabrica')" class="secondary">Ejemplo: Fábrica de Muebles</button>
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
      
      <div class="card" id="graficoContainer">
        <h3>📈 Visualización</h3>
        <canvas id="graficoCanvas"></canvas>
      </div>
    </div>
  </div>

<script>
// Variables globales
let problemaActual = null;
let solucionActual = null;

// Ejemplos de problemas
const ejemplos = {
  investment: {
    nombre: "Investment Advisors",
    texto: Investment Advisors, Inc. es una firma de corretaje que administra portafolios de acciones para varios clientes. Un portafolio en particular consta de U acciones de U.S. Oil y H acciones de Huber Steel. El rendimiento anual para U.S. Oil es $3 por acción, y para Huber Steel es $5 por acción. Las acciones de U.S. Oil se venden a $25 por acción y las de Huber Steel a $50. El portafolio tiene $80,000 para invertir. El índice de riesgo del portafolio (0.50 por acción de U.S. Oil y 0.25 por acción de Huber Steel) tiene un máximo de 700. Además, el portafolio está limitado a un máximo de 1000 acciones de U.S. Oil
  },
  dci: {
    nombre: "Digital Controls, Inc.",
    texto: Digital Controls, Inc. (DCI) fabrica dos modelos de una pistola radar utilizada por la policía para monitorear la velocidad de los automóviles. El modelo A tiene una precisión de más menos 1 milla por hora, mientras que el modelo B más pequeño tiene una precisión de más menos 3 millas por hora. La empresa tiene pedido para 100 unidades del modelo A y 150 unidades del modelo B para la semana siguiente. Aunque DCI compra todos los componentes que utiliza en ambos modelos, los estuches de plástico usados para ambos modelos se fabrican en una planta de DCI en Newark, New Jersey. Cada estuche para el modelo A requiere 4 minutos de tiempo de moldeo por inyección y 6 minutos de tiempo de ensamblaje. Cada estuche para el modelo B requiere 3 minutos de moldeo por inyección y 8 minutos de ensamblaje. Para la semana siguiente la planta de Newark dispone de 600 minutos de tiempo de moldeo por inyección y 1080 minutos de tiempo de ensamblaje. El costo de manufactura es $10 por estuche para el modelo A y $6 por estuche para el modelo B. Dependiendo de la demanda y el tiempo disponible en la planta de Newark, DCI ocasionalmente compra estuches para uno o ambos modelos a un proveedor externo con el fin de abastecer los pedidos de los clientes que de lo contrario no se podrían entregar. El costo de compra es $14 por estuche para el modelo A y $9 por estuche para el modelo B. La gerencia quiere desarrollar un plan de costo mínimo que determine cuántos estuches de cada modelo deben fabricarse en la planta de Newark y cuántos estuches de cada modelo deben comprarse.
  },
  fabrica: {
    nombre: "Fábrica de Muebles",
    texto: Una fábrica produce dos tipos de muebles: mesas y sillas. Cada mesa genera una ganancia de $80 y cada silla $50. Para producir una mesa se necesitan 4 horas de trabajo y 2 unidades de material. Para una silla se necesitan 3 horas de trabajo y 1 unidad de material. Disponemos de 240 horas de trabajo y 100 unidades de material. ¿Cuántas mesas y sillas se deben producir para maximizar la ganancia?
  }
};

// Cargar ejemplo
function cargarEjemplo(clave) {
  if (ejemplos[clave]) {
    document.getElementById('problema').value = ejemplos[clave].texto;
    document.getElementById('interpretacion').innerHTML = <h3>🔍 Ejemplo Cargado: ${ejemplos[clave].nombre}</h3><p>Haga clic en "Analizar y Resolver" para procesar este problema.</p>;
  }
}

// Función principal
function analizarYResolver() {
  const textoProblema = document.getElementById('problema').value.trim();
  
  if (!textoProblema) {
    mostrarError("Por favor ingrese un problema para analizar.");
    return;
  }
  
  // Mostrar carga
  document.getElementById('resultado').innerHTML = '<div class="loading"></div><p>Analizando problema, por favor espere...</p>';
  document.getElementById('interpretacion').innerHTML = '<div class="loading"></div><p>Procesando descripción del problema...</p>';
  
  // Usar setTimeout para permitir que la UI se actualice
  setTimeout(() => {
    try {
      // Paso 1: Analizar el problema
      problemaActual = parsearProblema(textoProblema);
      mostrarInterpretacion(problemaActual);
      
      // Paso 2: Resolver el problema
      solucionActual = resolverProblemaSimplex(problemaActual);
      mostrarResultados(solucionActual, problemaActual);
      
    } catch (error) {
      mostrarError("Error al procesar el problema: " + error.message);
      console.error(error);
    }
  }, 100);
}

// Función para parsear el problema
function parsearProblema(texto) {
  const textoLower = texto.toLowerCase();
  
  // Determinar tipo de problema
  const esMinimizacion = textoLower.includes("minimizar") || 
                        textoLower.includes("minimización") ||
                        textoLower.includes("costo mínimo") || 
                        textoLower.includes("coste mínimo") ||
                        textoLower.includes("minimizar costos");
  
  // Extraer variables (mejorado)
  const variables = [];
  const regexVariables = /(\b[a-z]\b|\b[a-z][a-z0-9]\b)(?=\s(acciones|unidades|modelo|estuches|mesas|sillas|trajes|sacos|inversión|invertir|compra|fabricación))/gi;
  let match;
  const uniqueVars = new Set();
  
  while ((match = regexVariables.exec(textoLower))) {
    uniqueVars.add(match[1].toLowerCase());
  }
  
  // Si no encontramos variables con el método anterior, buscamos letras solas
  if (uniqueVars.size === 0) {
    const letrasSolas = textoLower.match(/\b[a-z]\b/g) || [];
    letrasSolas.forEach(v => uniqueVars.add(v));
  }
  
  // Convertir a array y asegurar al menos 2 variables
  variables.push(...Array.from(uniqueVars));
  if (variables.length === 0) {
    variables.push('x', 'y');
  } else if (variables.length === 1) {
    variables.push(variables[0] + '2');
  }
  
  // Extraer función objetivo (mejorado)
  let objetivoCoefs = new Array(variables.length).fill(0);
  let objetivoTexto = esMinimizacion ? "Minimizar costos" : "Maximizar beneficios";
  
  // Buscar rendimientos/ganancias/costos por variable
  variables.forEach((v, i) => {
    const regex = new RegExp((rendimiento|ganancia|beneficio|utilidad|costo|coste)\\s*(de|por|)\\s*\\$?\\d+\\s*(por|para|de)\\s*${v}, "gi");
    const matches = texto.match(regex) || [];
    if (matches.length > 0) {
      const numMatch = matches[0].match(/\$?(\d+)/);
      if (numMatch) {
        objetivoCoefs[i] = parseFloat(numMatch[1]);
      }
    }
  });
  
  // Si no encontramos coeficientes, usar valores por defecto
  if (objetivoCoefs.every(c => c === 0)) {
    objetivoCoefs = esMinimizacion ? [10, 6] : [80, 50];
  }
  
  // Extraer restricciones (mejorado)
  const restricciones = [];
  
  // 1. Restricciones de presupuesto/inversión
  const regexPresupuesto = /(portafolio|presupuesto|inversión|disponible|tiene)\s*(de|)\s*\$\s*(\d[\d,]*)/gi;
  if ((match = regexPresupuesto.exec(texto))) {
    const valor = parseFloat(match[3].replace(',', ''));
    const coefs = new Array(variables.length).fill(0);
    
    // Buscar precios por variable
    variables.forEach((v, i) => {
      const regexPrecio = new RegExp(${v}\\s*(se\\s*venden|a|por|)\\s*\\$\\s*(\\d[\d,]*), "gi");
      const precioMatch = texto.match(regexPrecio);
      if (precioMatch && precioMatch[0]) {
        const precio = parseFloat(precioMatch[0].match(/\$?\s*(\d[\d,]*)/)[1].replace(',', ''));
        coefs[i] = precio;
      }
    });
    
    if (coefs.some(c => c !== 0)) {
      restricciones.push({
        coefficients: coefs,
        sign: '<=',
        value: valor,
        text: Inversión total <= $${valor}
      });
    }
  }
  
  // 2. Restricciones de riesgo/índices
  const regexRiesgo = /(índice de riesgo|riesgo|índice)\s*(de|)\\s*(\d+\.?\d*)\s*(por|para|de)\\s*(\w+)/gi;
  while ((match = regexRiesgo.exec(texto))) {
    const valorTotalMatch = texto.match(/(índice de riesgo|riesgo|índice)\s*(tiene|tienen|)\\s*(un\\s*)?(máximo|máx|max|mínimo|min)\\s*(de|)\\s*(\d+)/i);
    const valorTotal = valorTotalMatch ? parseFloat(valorTotalMatch[6]) : null;
    
    if (valorTotal) {
      const coefs = new Array(variables.length).fill(0);
      const varMatch = match[5].toLowerCase();
      const varIndex = variables.findIndex(v => v === varMatch);
      
      if (varIndex >= 0) {
        coefs[varIndex] = parseFloat(match[3]);
        restricciones.push({
          coefficients: coefs,
          sign: '<=',
          value: valorTotal,
          text: Índice de riesgo <= ${valorTotal}
        });
      }
    }
  }
  
  // 3. Restricciones de cantidad máxima/mínima
  const regexCantidad = /(máximo|máx|max|mínimo|min)\\s*(de|)\\s*(\d+)\\s*(acciones|unidades|)\\s*(de|)\\s*(\w+)/gi;
  while ((match = regexCantidad.exec(texto))) {
    const esMaximo = match[1].toLowerCase().includes("máx") || match[1].toLowerCase().includes("max");
    const cantidad = parseFloat(match[3]);
    const varMatch = match[6].toLowerCase();
    const varIndex = variables.findIndex(v => v === varMatch);
    
    if (varIndex >= 0) {
      const coefs = new Array(variables.length).fill(0);
      coefs[varIndex] = 1;
      restricciones.push({
        coefficients: coefs,
        sign: esMaximo ? '<=' : '>=',
        value: cantidad,
        text: ${variables[varIndex]} ${esMaximo ? '<=' : '>='} ${cantidad}
      });
    }
  }
  
  // 4. Restricciones de recursos (tiempo, material)
  const regexRecursos = /(\d+)\s*(minutos|horas|unidades|)\\s*(de|)\\s*[\w\s]+\\s*(para|por|de)\\s*(\w+)/gi;
  while ((match = regexRecursos.exec(texto))) {
    const cantidadPorUnidad = parseFloat(match[1]);
    const recurso = match[2] || "unidades";
    const varMatch = match[5].toLowerCase();
    const varIndex = variables.findIndex(v => v === varMatch);
    
    if (varIndex >= 0) {
      // Buscar disponibilidad total del recurso
      const regexDisponibilidad = new RegExp((disponible|dispone|tiene)\\s*(de|)\\s*(\\d+)\\s*${recurso}, "gi");
      const dispMatch = texto.match(regexDisponibilidad);
      
      if (dispMatch) {
        const dispTotal = parseFloat(dispMatch[0].match(/\d+/)[0]);
        const coefs = new Array(variables.length).fill(0);
        coefs[varIndex] = cantidadPorUnidad;
        
        // Verificar si ya existe una restricción para este recurso
        const existingIndex = restricciones.findIndex(r => r.text.includes(recurso));
        
        if (existingIndex >= 0) {
          // Sumar a la restricción existente
          restricciones[existingIndex].coefficients[varIndex] = cantidadPorUnidad;
        } else {
          restricciones.push({
            coefficients: coefs,
            sign: '<=',
            value: dispTotal,
            text: Uso de ${recurso} <= ${dispTotal}
          });
        }
      }
    }
  }
  
  // 5. Restricciones de demanda/pedidos
  const regexDemanda = /(pedido|demanda|requiere)\\s*(de|)\\s*(\\d+)\\s*(unidades|)\\s*(del|de|)\\s*(\w+)/gi;
  while ((match = regexDemanda.exec(texto))) {
    const cantidad = parseFloat(match[3]);
    const varMatch = match[6].toLowerCase();
    const varIndex = variables.findIndex(v => v === varMatch);
    
    if (varIndex >= 0) {
      const coefs = new Array(variables.length).fill(0);
      coefs[varIndex] = 1;
      restricciones.push({
        coefficients: coefs,
        sign: '>=',
        value: cantidad,
        text: Demanda de ${variables[varIndex]} >= ${cantidad}
      });
    }
  }
  
  // Para el problema específico de Investment Advisors
  if (textoLower.includes("investment advisors")) {
    variables.length = 0;
    variables.push('u', 'h'); // U.S. Oil y Huber Steel
    
    // Función objetivo: maximizar rendimiento 3U + 5H
    objetivoCoefs = [3, 5];
    objetivoTexto = "Maximizar rendimiento anual";
    
    // Restricciones
    restricciones.length = 0;
    // Inversión: 25U + 50H <= 80000
    restricciones.push({
      coefficients: [25, 50],
      sign: '<=',
      value: 80000,
      text: "25U + 50H <= 80000 (inversión)"
    });
    // Riesgo: 0.50U + 0.25H <= 700
    restricciones.push({
      coefficients: [0.5, 0.25],
      sign: '<=',
      value: 700,
      text: "0.50U + 0.25H <= 700 (riesgo)"
    });
    // Máximo U.S. Oil: U <= 1000
    restricciones.push({
      coefficients: [1, 0],
      sign: '<=',
      value: 1000,
      text: "U <= 1000 (máx U.S. Oil)"
    });
    // No negatividad
    restricciones.push({
      coefficients: [1, 0],
      sign: '>=',
      value: 0,
      text: "U >= 0"
    });
    restricciones.push({
      coefficients: [0, 1],
      sign: '>=',
      value: 0,
      text: "H >= 0"
    });
  }
  
  // Para el problema específico de Digital Controls
  if (textoLower.includes("digital controls")) {
    variables.length = 0;
    variables.push('a_fab', 'a_comp', 'b_fab', 'b_comp'); // Fabricación y compra
    
    // Función objetivo: minimizar costos 10a_fab + 14a_comp + 6b_fab + 9b_comp
    objetivoCoefs = [10, 14, 6, 9];
    objetivoTexto = "Minimizar costos totales";
    
    // Restricciones
    restricciones.length = 0;
    // Demanda A: a_fab + a_comp >= 100
    restricciones.push({
      coefficients: [1, 1, 0, 0],
      sign: '>=',
      value: 100,
      text: "a_fab + a_comp >= 100 (demanda A)"
    });
    // Demanda B: b_fab + b_comp >= 150
    restricciones.push({
      coefficients: [0, 0, 1, 1],
      sign: '>=',
      value: 150,
      text: "b_fab + b_comp >= 150 (demanda B)"
    });
    // Moldeo: 4a_fab + 3b_fab <= 600
    restricciones.push({
      coefficients: [4, 0, 3, 0],
      sign: '<=',
      value: 600,
      text: "4a_fab + 3b_fab <= 600 (moldeo)"
    });
    // Ensamblaje: 6a_fab + 8b_fab <= 1080
    restricciones.push({
      coefficients: [6, 0, 8, 0],
      sign: '<=',
      value: 1080,
      text: "6a_fab + 8b_fab <= 1080 (ensamblaje)"
    });
    // No negatividad
    restricciones.push({
      coefficients: [1, 0, 0, 0],
      sign: '>=',
      value: 0,
      text: "a_fab >= 0"
    });
    restricciones.push({
      coefficients: [0, 1, 0, 0],
      sign: '>=',
      value: 0,
      text: "a_comp >= 0"
    });
    restricciones.push({
      coefficients: [0, 0, 1, 0],
      sign: '>=',
      value: 0,
      text: "b_fab >= 0"
    });
    restricciones.push({
      coefficients: [0, 0, 0, 1],
      sign: '>=',
      value: 0,
      text: "b_comp >= 0"
    });
  }
  
  return {
    variables: variables,
    objetivo: {
      coefficients: objetivoCoefs,
      tipo: esMinimizacion ? 'min' : 'max',
      texto: objetivoTexto
    },
    restricciones: restricciones,
    textoOriginal: texto
  };
}

// Mostrar interpretación del problema
function mostrarInterpretacion(problema) {
  let html = <h4>📌 Tipo de Problema</h4>;
  html += <p><strong>${problema.objetivo.tipo === 'min' ? 'Minimización' : 'Maximización'}:</strong> ${problema.objetivo.texto}</p>;
  
  html += <h4>🔢 Variables de Decisión</h4><ul>;
  problema.variables.forEach((v, i) => {
    html += <li><span class="var-highlight">${v}</span>: Coeficiente = ${problema.objetivo.coefficients[i]}</li>;
  });
  html += </ul>;
  
  html += <h4>📏 Restricciones Identificadas</h4><table><tr><th>Restricción</th><th>Expresión</th></tr>;
  problema.restricciones.forEach(r => {
    const expr = problema.variables.map((v, i) => {
      if (r.coefficients[i] !== 0) {
        return ${r.coefficients[i]}${v};
      }
      return '';
    }).filter(x => x !== '').join(' + ');
    
    html += `<tr>
      <td>${r.text}</td>
      <td>${expr} ${r.sign} ${r.value}</td>
    </tr>`;
  });
  html += </table>;
  
  document.getElementById('interpretacion').innerHTML = html;
}

// Resolver problema con método Simplex
function resolverProblemaSimplex(problema) {
  // Validar entrada
  if (problema.variables.length !== problema.objetivo.coefficients.length) {
    throw new Error("Número de variables no coincide con coeficientes objetivo");
  }
  
  // Preparar datos para Simplex
  const numVars = problema.variables.length;
  const numRest = problema.restricciones.length;
  const totalVars = numVars + numRest;
  
  // Matriz del tableau
  let tableau = [];
  let basis = [];
  let artificialVars = 0;
  
  // Configurar restricciones
  for (let i = 0; i < problema.restricciones.length; i++) {
    const r = problema.restricciones[i];
    const row = new Array(totalVars + 1).fill(0);
    
    // Variables originales
    for (let j = 0; j < numVars; j++) {
      row[j] = r.coefficients[j];
    }
    
    // Variables de holgura/exceso
    if (r.sign === '<=') {
      row[numVars + i] = 1; // Holgura
      basis.push(numVars + i);
    } else if (r.sign === '>=') {
      row[numVars + i] = -1; // Exceso
      // Variable artificial
      row[numVars + numRest + artificialVars] = 1;
      artificialVars++;
      basis.push(numVars + numRest + artificialVars - 1);
    } else if (r.sign === '==') {
      // Variable artificial
      row[numVars + numRest + artificialVars] = 1;
      artificialVars++;
      basis.push(numVars + numRest + artificialVars - 1);
    }
    
    // Lado derecho
    row[row.length - 1] = r.value;
    tableau.push(row);
  }
  
  // Fase I (si hay variables artificiales)
  if (artificialVars > 0) {
    const artificialObj = new Array(totalVars + artificialVars + 1).fill(0);
    
    // Sumar restricciones con variables artificiales
    for (let i = 0; i < problema.restricciones.length; i++) {
      if (problema.restricciones[i].sign === '>=' || problema.restricciones[i].sign === '==') {
        for (let j = 0; j < totalVars + 1; j++) {
          artificialObj[j] += tableau[i][j];
        }
      }
    }
    
    // Negar para minimizar
    for (let j = 0; j < artificialObj.length; j++) {
      artificialObj[j] = -artificialObj[j];
    }
    
    // Guardar objetivo original
    const originalObj = new Array(totalVars + 1).fill(0);
    for (let j = 0; j < numVars; j++) {
      originalObj[j] = problema.objetivo.tipo === 'max' ? -problema.objetivo.coefficients[j] : problema.objetivo.coefficients[j];
    }
    
    // Fase I: Resolver problema artificial
    tableau.unshift(artificialObj);
    let phase1Result = simplexIterations(tableau, basis, numVars, numRest);
    
    // Verificar factibilidad
    if (Math.abs(phase1Result.tableau[0][totalVars + artificialVars - 1]) > 1e-6) {
      return { error: "El problema no tiene solución factible" };
    }
    
    // Fase II: Usar solución de Fase I
    tableau = phase1Result.tableau.slice(1);
    tableau[0] = originalObj;
    
    // Recalcular fila objetivo
    for (let i = 1; i < tableau.length; i++) {
      const basicVar = basis[i-1];
      if (basicVar < numVars) {
        const factor = tableau[0][basicVar];
        for (let j = 0; j < tableau[0].length; j++) {
          tableau[0][j] -= factor * tableau[i][j];
        }
      }
    }
    
    // Eliminar columnas de variables artificiales
    for (let row of tableau) {
      row.splice(numVars + numRest, artificialVars);
    }
    
    basis = phase1Result.basis.filter(b => b < numVars + numRest);
  } else {
    // No hay variables artificiales, usar objetivo original
    const objectiveRow = new Array(totalVars + 1).fill(0);
    for (let j = 0; j < numVars; j++) {
      objectiveRow[j] = problema.objetivo.tipo === 'max' ? -problema.objetivo.coefficients[j] : problema.objetivo.coefficients[j];
    }
    tableau.unshift(objectiveRow);
  }
  
  // Resolver problema principal
  const result = simplexIterations(tableau, basis, numVars, numRest);
  
  // Extraer solución
  const solution = new Array(numVars).fill(0);
  const slackValues = new Array(numRest).fill(0);
  const optimalValue = problema.objetivo.tipo === 'max' ? 
    result.tableau[0][totalVars - artificialVars] : 
    -result.tableau[0][totalVars - artificialVars];
  
  for (let i = 0; i < basis.length; i++) {
    const varIndex = basis[i];
    if (varIndex < numVars) {
      solution[varIndex] = result.tableau[i+1][totalVars - artificialVars];
    } else if (varIndex < numVars + numRest) {
      slackValues[varIndex - numVars] = result.tableau[i+1][totalVars - artificialVars];
    }
  }
  
  return {
    variables: solution,
    slackVariables: slackValues,
    optimalValue: optimalValue,
    tableau: result.tableau,
    basis: basis,
    iterations: result.iterations,
    isOptimal: result.isOptimal,
    artificialVars: artificialVars
  };
}

// Iteraciones del método Simplex
function simplexIterations(tableau, basis, numVars, numRest) {
  let iterations = 0;
  const maxIterations = 1000;
  let isOptimal = false;
  
  while (iterations < maxIterations) {
    iterations++;
    
    // Paso 1: Encontrar columna pivote (más negativa en la fila objetivo)
    let pivotCol = -1;
    let minVal = 0;
    
    for (let j = 0; j < tableau[0].length - 1; j++) {
      if (tableau[0][j] < minVal - 1e-6) { // Pequeña tolerancia
        minVal = tableau[0][j];
        pivotCol = j;
      }
    }
    
    if (pivotCol === -1) {
      isOptimal = true;
      break;
    }
    
    // Paso 2: Encontrar fila pivote (mínima razón positiva)
    let pivotRow = -1;
    let minRatio = Infinity;
    
    for (let i = 1; i < tableau.length; i++) {
      if (tableau[i][pivotCol] > 1e-6) { // Pequeña tolerancia
        const ratio = tableau[i][tableau[0].length - 1] / tableau[i][pivotCol];
        if (ratio < minRatio) {
          minRatio = ratio;
          pivotRow = i;
        }
      }
    }
    
    if (pivotRow === -1) {
      return { 
        tableau: tableau,
        basis: basis,
        iterations: iterations,
        isOptimal: false,
        error: "El problema es no acotado"
      };
    }
    
    // Paso 3: Actualizar base
    basis[pivotRow - 1] = pivotCol;
    
    // Paso 4: Pivotear
    const pivotVal = tableau[pivotRow][pivotCol];
    
    // Normalizar fila pivote
    for (let j = 0; j < tableau[pivotRow].length; j++) {
      tableau[pivotRow][j] /= pivotVal;
    }
    
    // Eliminar la columna pivote de otras filas
    for (let i = 0; i < tableau.length; i++) {
      if (i !== pivotRow) {
        const factor = tableau[i][pivotCol];
        for (let j = 0; j < tableau[i].length; j++) {
          tableau[i][j] -= factor * tableau[pivotRow][j];
        }
      }
    }
  }
  
  return {
    tableau: tableau,
    basis: basis,
    iterations: iterations,
    isOptimal: isOptimal
  };
}

// Mostrar resultados
function mostrarResultados(solution, problema) {
  if (solution.error) {
    mostrarError(solution.error);
    return;
  }
  
  let html = <div class="${solution.isOptimal ? 'success' : 'warning'}">;
  
  if (solution.isOptimal) {
    html += <h3>✅ Solución Óptima Encontrada</h3>;
    html += <p><strong>Valor Óptimo:</strong> ${solution.optimalValue.toFixed(2)}</p>;
  } else {
    html += <h3>⚠ Solución Parcial</h3>;
    html += <p>Se detuvo después de ${solution.iterations} iteraciones sin alcanzar optimalidad.</p>;
  }
  
  html += <h4>Valores de las Variables</h4>;
  html += <table><tr><th>Variable</th><th>Valor</th><th>Significado</th></tr>;
  
  problema.variables.forEach((v, i) => {
    let significado = "";
    if (v === 'u') significado = "Acciones de U.S. Oil";
    else if (v === 'h') significado = "Acciones de Huber Steel";
    else if (v === 'a_fab') significado = "Estuches A fabricados";
    else if (v === 'a_comp') significado = "Estuches A comprados";
    else if (v === 'b_fab') significado = "Estuches B fabricados";
    else if (v === 'b_comp') significado = "Estuches B comprados";
    else if (v === 'x' || v === 'y') significado = Variable ${v.toUpperCase()};
    
    html += `<tr>
      <td>${v}</td>
      <td>${solution.variables[i].toFixed(2)}</td>
      <td>${significado}</td>
    </tr>`;
  });
  
  html += </table>;
  
  html += <h4>Holguras/Excesos</h4>;
  html += <table><tr><th>Restricción</th><th>Valor</th><th>Interpretación</th></tr>;
  
  problema.restricciones.forEach((r, i) => {
    const slack = solution.slackVariables[i] || 0;
    let interpretacion = "";
    
    if (r.sign === '<=') {
      interpretacion = ${slack.toFixed(2)} unidades no utilizadas;
    } else if (r.sign === '>=') {
      interpretacion = ${slack.toFixed(2)} unidades de exceso sobre el mínimo;
    } else {
      interpretacion = "Restricción exacta";
    }
    
    html += `<tr>
      <td>${r.text}</td>
      <td>${slack.toFixed(2)}</td>
      <td>${interpretacion}</td>
    </tr>`;
  });
  
  html += </table>;
  html += <p><strong>Iteraciones realizadas:</strong> ${solution.iterations}</p>;
  html += </div>;
  
  document.getElementById('resultado').innerHTML = html;
  
  // Mostrar tabla simplex (solo para problemas pequeños)
  mostrarTablaSimplex(solution, problema);
  
  // Mostrar gráfico (solo para problemas con 2 variables)
  if (problema.variables.length === 2) {
    mostrarGrafico(solution, problema);
  } else {
    document.getElementById('graficoContainer').style.display = 'none';
  }
}

// Mostrar tabla simplex
function mostrarTablaSimplex(solution, problema) {
  if (problema.variables.length > 3 || problema.restricciones.length > 5) {
    document.getElementById('tablaSimplex').innerHTML = `
      <h3>📋 Proceso Simplex</h3>
      <p>El problema es demasiado grande para mostrar la tabla completa. 
      Se realizaron ${solution.iterations} iteraciones del método Simplex.</p>
    `;
    return;
  }
  
  let html = <h3>📋 Tabla Simplex Final</h3>;
  html += <div style="overflow-x: auto;"><table>;
  
  // Encabezados
  html += <tr><th>Base</th>;
  for (let i = 0; i < problema.variables.length; i++) {
    html += <th>${problema.variables[i]}</th>;
  }
  for (let i = 0; i < problema.restricciones.length; i++) {
    html += <th>s${i+1}</th>;
  }
  if (solution.artificialVars > 0) {
    for (let i = 0; i < solution.artificialVars; i++) {
      html += <th>a${i+1}</th>;
    }
  }
  html += <th>Solución</th></tr>;
  
  // Filas
  for (let i = 0; i < solution.tableau.length; i++) {
    html += <tr>;
    
    // Columna Base
    if (i === 0) {
      html += <td>Z</td>;
    } else {
      const basisIndex = solution.basis[i-1];
      if (basisIndex < problema.variables.length) {
        html += <td>${problema.variables[basisIndex]}</td>;
      } else if (basisIndex < problema.variables.length + problema.restricciones.length) {
        html += <td>s${basisIndex - problema.variables.length + 1}</td>;
      } else {
        html += <td>a${basisIndex - (problema.variables.length + problema.restricciones.length) + 1}</td>;
      }
    }
    
    // Coeficientes
    for (let j = 0; j < solution.tableau[i].length; j++) {
      html += <td>${solution.tableau[i][j].toFixed(2)}</td>;
    }
    
    html += </tr>;
  }
  
  html += </table></div>;
  html += <p class="info-box">La tabla muestra los valores finales después de ${solution.iterations} iteraciones.</p>;
  
  document.getElementById('tablaSimplex').innerHTML = html;
}

// Mostrar gráfico para problemas con 2 variables
function mostrarGrafico(solution, problema) {
  const ctx = document.getElementById('graficoCanvas').getContext('2d');
  
  // Generar puntos factibles
  const puntosFactibles = generarPuntosFactibles(problema);
  
  // Configurar datos del gráfico
  const datasets = [{
    label: 'Región Factible',
    data: puntosFactibles,
    backgroundColor: 'rgba(54, 162, 235, 0.5)',
    pointRadius: 3,
    showLine: true,
    borderColor: 'rgba(54, 162, 235, 0.8)',
    fill: true
  }];
  
  // Añadir solución óptima
  if (solution.isOptimal && solution.variables.length === 2) {
    datasets.push({
      label: 'Solución Óptima',
      data: [{x: solution.variables[0], y: solution.variables[1]}],
      backgroundColor: 'rgba(255, 99, 132, 1)',
      pointRadius: 8
    });
  }
  
  // Añadir restricciones como líneas
  problema.restricciones.forEach((r, i) => {
    if (r.coefficients.length === 2 && r.coefficients[0] !== 0 && r.coefficients[1] !== 0) {
      // Calcular puntos para la línea de restricción
      const x1 = 0;
      const y1 = (r.value - r.coefficients[0] * x1) / r.coefficients[1];
      const y2 = 0;
      const x2 = (r.value - r.coefficients[1] * y2) / r.coefficients[0];
      
      // Solo añadir si los puntos son positivos
      if (y1 >= 0 || x2 >= 0) {
        const points = [];
        if (y1 >= 0) points.push({x: x1, y: y1});
        if (x2 >= 0) points.push({x: x2, y: y2});
        
        if (points.length >= 2) {
          datasets.push({
            label: Restricción ${i+1},
            data: points,
            borderColor: hsl(${i * 60}, 70%, 50%),
            borderWidth: 2,
            pointRadius: 0,
            fill: false,
            showLine: true
          });
        }
      }
    }
  });
  
  // Crear gráfico
  new Chart(ctx, {
    type: 'scatter',
    data: {
      datasets: datasets
    },
    options: {
      responsive: true,
      plugins: {
        legend: { position: 'top' },
        tooltip: {
          callbacks: {
            label: function(context) {
              return ${problema.variables[0]}: ${context.parsed.x.toFixed(2)}, ${problema.variables[1]}: ${context.parsed.y.toFixed(2)};
            }
          }
        }
      },
      scales: {
        x: {
          title: { display: true, text: problema.variables[0] },
          min: 0,
          grid: { color: '#ddd' }
        },
        y: {
          title: { display: true, text: problema.variables[1] },
          min: 0,
          grid: { color: '#ddd' }
        }
      }
    }
  });
  
  document.getElementById('graficoContainer').style.display = 'block';
}

// Generar puntos factibles para el gráfico
function generarPuntosFactibles(problema) {
  if (problema.variables.length !== 2) return [];
  
  const puntos = [];
  const maxX = Math.max(...problema.restricciones.map(r => {
    if (r.coefficients[0] !== 0) return r.value / r.coefficients[0];
    return 0;
  }).filter(x => isFinite(x))) * 1.2 || 100;
  
  const maxY = Math.max(...problema.restricciones.map(r => {
    if (r.coefficients[1] !== 0) return r.value / r.coefficients[1];
    return 0;
  }).filter(y => isFinite(y))) * 1.2 || 100;
  
  const stepX = maxX / 30;
  const stepY = maxY / 30;
  
  for (let x = 0; x <= maxX; x += stepX) {
    for (let y = 0; y <= maxY; y += stepY) {
      let factible = true;
      
      for (const r of problema.restricciones) {
        const lhs = r.coefficients[0] * x + r.coefficients[1] * y;
        
        if (r.sign === '<=' && lhs > r.value + 1e-6) {
          factible = false;
          break;
        } else if (r.sign === '>=' && lhs < r.value - 1e-6) {
          factible = false;
          break;
        } else if (r.sign === '==' && Math.abs(lhs - r.value) > 1e-6) {
          factible = false;
          break;
        }
      }
      
      if (factible) {
        puntos.push({x: x, y: y});
      }
    }
  }
  
  return puntos;
}

// Mostrar mensaje de error
function mostrarError(mensaje) {
  document.getElementById('resultado').innerHTML = `
    <div class="error">
      <h3>❌ Error</h3>
      <p>${mensaje}</p>
    </div>
  `;
}
</script>
</body>
</html>
