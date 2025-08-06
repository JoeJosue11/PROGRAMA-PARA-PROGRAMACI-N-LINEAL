Y; y += stepY) {
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
