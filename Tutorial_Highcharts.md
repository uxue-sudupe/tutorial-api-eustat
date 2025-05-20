# Visualización de la tasa de paro en Euskadi con Highcharts

Este tutorial muestra cómo utilizar la **API PXStat de Eustat** para obtener la tasa de paro en Euskadi y representarla mediante un gráfico con **Highcharts**.

> **Nota:** Highcharts es gratuito solo para uso no comercial. Si lo quieres usar en publicaciones institucionales o productos, consulta su [licencia](http://www.highcharts.com/products/highcharts).

---

## 1. Crear un HTML mínimo para Highcharts

```html
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <title>Paro en Euskadi</title>
  <script src="https://code.highcharts.com/highcharts.js"></script>
  <script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>
</head>
<body>
  <div id="paro-chart" style="height: 500px; width: 100%;"></div>
</body>
</html>
```

---

## 2. Llamar a la API PXStat de Eustat para obtener solo Euskadi

En este ejemplo hacemos una petición `POST` a la API para obtener solo los datos del ámbito `00001` (Euskadi):

```js
fetch("https://www.eustat.eus/bankupx/api/v1/es/DB/PX_050407_cempa_empa_pp41.px", {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({
    query: [
      {
        code: "\u00e1mbitos territoriales",
        selection: { filter: "item", values: ["00001"] }
      }
    ]
  })
})
.then(res => res.json())
.then(json => {
  // Procesamiento de datos aquí
});
```

---

## 3. Transformar los datos para Highcharts

Filtramos y agrupamos los datos por sexo (10: hombres, 20: mujeres) y año:

```js
const dataset = json.dataset;
const dim = dataset.dimension;
const sexos = dim.sexo.category.label;
const anios = dim.periodo.category.label;
const valores = dataset.value;

const totalSexos = Object.keys(sexos).length;
const totalAnios = Object.keys(anios).length;

const hombres = [];
const mujeres = [];

for (let j = 0; j < totalSexos; j++) {
  for (let k = 0; k < totalAnios; k++) {
    const index = j * totalAnios + k;
    const year = parseInt(Object.values(anios)[k]);
    const value = valores[index];

    if (Object.keys(sexos)[j] === "10") {
      hombres.push([year, value]);
    } else if (Object.keys(sexos)[j] === "20") {
      mujeres.push([year, value]);
    }
  }
}
```

---

## 4. Dibujar el gráfico con Highcharts

```js
Highcharts.chart('paro-chart', {
  chart: { type: 'line' },
  title: { text: 'Tasa de paro en Euskadi por sexo' },
  credits: { enabled: false },
  xAxis: {
    title: { text: 'Año' },
    allowDecimals: false,
    categories: hombres.map(p => p[0])
  },
  yAxis: {
    title: { text: 'Tasa de paro (%)' },
    labels: {
      formatter: function() {
        return this.value + '%';
      }
    },
    plotLines: [{
      value: 10,
      color: '#ccc',
      dashStyle: 'shortdash',
      width: 1,
      label: {
        text: 'Referencia 10%'
      }
    }]
  },
  tooltip: { shared: true },
  series: [
    { name: 'Hombres', data: hombres.map(p => p[1]), color: '#1f77b4' },
    { name: 'Mujeres', data: mujeres.map(p => p[1]), color: '#d62728' }
  ]
});
```

---

## 5. Resultado esperado

- Gráfico de líneas comparando tasa de paro de hombres y mujeres por año en Euskadi.
- Tooltip compartido.
- Línea de referencia al 10%.

Puedes ver el resultado completo en:  
👉 [https://uxue-sudupe.github.io/tutorial-api-eustat/tutorial_highcharts.html](https://uxue-sudupe.github.io/tutorial-api-eustat/tutorial_highcharts.html)

---

## Recursos adicionales

- 📘 [API PXStat de Eustat](https://www.eustat.eus/elementos/ele0010600/ti_API_REST_para_acceso_a_datos_estadisticos/tbl0010666_c.html)
- 📗 [Documentación de Highcharts](https://www.highcharts.com/docs)
- 🔧 [Referencia completa de opciones](https://api.highcharts.com/highcharts/)
- 🎓 [Ejemplos de gráficos Highcharts](https://www.highcharts.com/demo)

---
