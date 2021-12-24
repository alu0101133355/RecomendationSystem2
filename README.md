# Recomendation System

## Uso de la aplicacion
```
https://eindhovenlion99.github.io/RecomendationSystem/
```

## Codigo Fuente

La aplicacion esta basada en Vue, un framework frontend basado en componentes. El primer componente es ```App.vue```, que carga el resto
de los componentes, este fichero es montado por ```main.js```. ```App.vue``` importa los componentes Header (Cabecera) y Footer (Pie de pagina).
Por otro lado carga el componente ```Content.vue```, el cual soporta el codigo de la aplicacion.

Este componente contiene el HTML de la aplicacion, asi como las funciones que hacen los calculos pertinentes y el css del propio componente.

```html
<template>
  <div class="container">
   <label for="Metrica">Elige una metrica: </label>
   <select class="select" id="metrica">
     <option value="1" selected>Correlacion de Pearson</option>
     <option value="2">Distancia coseno</option>
     <option value="3">Distancia Euclidea</option>
    </select>
   <hr class="separator">
   <label for="Vecinos">Elige el numero de vecinos: </label>
   <select class="select" id="vecinos">
     <option value="1" selected>1</option>
     <option value="2">2</option>
     <option value="3">3</option>
     <option value="4">4</option>
    </select>
   <hr class="separator">
   <label for="Prediccion">Elige una prediccion: </label>
   <select class="select" id="prediccion">
     <option value="1" selected>Predicción simple</option>
     <option value="2">Diferencia con la media</option>
    </select>
    <br>
    <br>
    <div>
      <label for="input-file">Introduce un archivo:</label><br>
      <input type="file" id="input-file">
    </div>
    <button class="button" v-on:click="getData">Calcular</button>
    <div class="solucion" id="solucion"></div>
  </div>
</template>
```

![](src/assets/Img1.png)

Una vez elegidas las opciones le damos al boton calcular, el cual llama a la funcion ```getData()```, la cual recoge los datos que ha introducido el usuario, asi como el fichero cargado:

```js
  getData() {
    this.datos = []
    const metrica = document.getElementById("metrica").value  // Metrica elegida
    this.datos.push(metrica)

    const vecino = document.getElementById("vecinos").value  // Numero de vecinos elegidos
    this.datos.push(vecino)

    const prediccion = document.getElementById("prediccion").value  // Prediccion elegida
    this.datos.push(prediccion)

    this.incognita_cont = 0;

    const file = document.getElementById("input-file").files[0];  // Leemos el contenido del fichero y rellenamos la matriz
    this.readFileContent(file).then(result => {
      const stringArray = result.split("\n")
      stringArray.forEach((element, index) => {
        this.Matriz[index] = [];
        let charArray = element.split(" ");
        charArray.forEach((element, index2) => {
          if(!isNaN(element)){
            this.Matriz[index][index2] = parseInt(element)
          }
          else {
            this.Matriz[index][index2] = '*'
            this.incognita.push(index);
            this.incognita.push(index2);
            if (this.incognita_cont < 1) {
              this.incognita_cont++;
            } else {
              this.alertIncognita();
            }
          }
        });
      })
      this.calcularMetricas();  // Llamamos a la funcion que calcula las metricas
      this.calcularPrediccion(); // Llamamos a la funcion que calcula la prediccion
    });
  }
```

Todas las variables ```this.``` pertenecen al espacio de variables del componentes, al igual que las funciones:

```js
  data() {
    return {
      datos: [],        // Metrica, numero de vecinos y prediccion elegida por el usuario se guardan aqui (getData). 
      Matriz: [],       // Matriz de calificaciones
      metrica: [],      // Metricas calculadas
      incognita: [],    // Posicion de donde se encuentra la incognita
      pred: Number      // Resultado de la prediccion
    }
  }
```

Al final de la funcion ```getData()```, se llama a las funciones ```this.calcularMetricas()```, la cual calcula las metricas, y ```this.calcularPrediccion()```, la cual calcula prediccion.

```js
  calcularMetricas() {
    this.metrica = []
    for (let i = 0; i < this.Matriz.length; i++) { // Calculamos la metrica del usuario con todos sus vecinos
      if (i != this.incognita[0]) {
        switch (this.datos[0]) {
          case "1": 
            this.metrica.push(this.Pearson(this.Matriz[this.incognita[0]], this.Matriz[i])) // Mandamos el vector del usuario y el del vecino por cada vecino que haya
          break;
          case "2": 
            this.metrica.push(this.Coseno(this.Matriz[this.incognita[0]], this.Matriz[i]))
          break;
          case "3": 
            this.metrica.push(this.Euclidea(this.Matriz[this.incognita[0]], this.Matriz[i]))
          break;
        }
      } else {
        this.metrica.push(0);
      }
    }
  }
```

Dentro de esta funcion, existen tres posibilidades:

* Calculo de metricas por Pearson:

```js
  Pearson(a_, b_) {
    let a = []
    let b = []
    a_.forEach((element, index) => {
      if (this.incognita[1] !== index) {
        a.push(element);
        b.push(b_[index])
      }
    })
    let media_a = 0;
    let media_b = 0;
    a.forEach((element) => {
      media_a += element;
    })
    media_a = media_a/a.length;
    b.forEach((element) => {
      media_b += element;
    })
    media_b = media_b/b.length;
    let num = 0;
    for (let i = 0; i < a.length; i++) {
      num += (a[i] - media_a) * (b[i] - media_b);
    }
    let den1 = 0;
    let den2 = 0;
    for (let i = 0; i < a.length; i++) {
      den1 += Math.pow((a[i] - media_a), 2);
      den2 += Math.pow((b[i] - media_b), 2);
    }
    let den = Math.sqrt(den1) * Math.sqrt(den2)
    return num / den;
  }
```

* Calculo de metricas por Distancia Coseno

```js
  Coseno(a_, b_) {
    let a = []
    let b = []
    a_.forEach((element, index) => {
      if (this.incognita[1] !== index) {
        a.push(element);
        b.push(b_[index])
      }
    })
    let num = 0;
    for (let i = 0; i < a.length; i++) {
      num += (a[i]) * (b[i]);
    }
    let den1 = 0;
    let den2 = 0;
    for (let i = 0; i < a.length; i++) {
      den1 += Math.pow((a[i]), 2);
      den2 += Math.pow((b[i]), 2);
    }
    let den = Math.sqrt(den1) * Math.sqrt(den2)
    return num / den;
  }
```

* Calculo de metricas por Distancia Euclidea

```js
  Euclidea(a_, b_) {
    let a = []
    let b = []
    a_.forEach((element, index) => {
      if (this.incognita[1] !== index) {
        a.push(element);
        b.push(b_[index])
      }
    })
    let result = 0;
    for (let i = 0; i < a.length; i++) {
      result += Math.pow((a[i] - b[i]), 2);
    }
    return Math.sqrt(result);
  }
```

Una vez calculada las metricas del usuario con cada uno de sus vecinos (Almacenados en le vector ```this.metrica```) llamamos a la funcion ```calcularPrediccion()```.

```js
  calcularPrediccion() {
    let newM = []
    let indexes = []
    this.metrica.forEach(element => {
      newM.push(element)
    })
    newM[this.incognita[0]] = -Infinity
    for (let i = 0; i < this.datos[1]; i++) { // Creamos un vector que tendra los indices de los vecinos con mas correlacion ordenados.
      const max = Math.max(...newM)
      const index = newM.indexOf(max);
      indexes.push(index)
      newM[index] = -Infinity
    }
    switch (this.datos[2]) {  // Elegimos la prediccion
      case "1": 
        console.log("Prediccion Simple");
        this.pred = this.PrediccionSimple(indexes);
        this.Resultado(indexes)  // Llamamos a la funcion que imprime el resultado en el html
      break;
      case "2": 
        console.log("Prediccion Media")
        this.pred = this.PrediccionMedia(indexes);
        this.Resultado(indexes)
      break;
    }
  }
```

Dependiendo de lo que haya elegido el usuario, se llamara a la funcion ```prediccionSimple()``` o ```prediccionMedia()```.

```js
  PrediccionSimple(Indexes) {
    let num = 0;
    let den = 0;
    for (let i = 0; i < Indexes.length; i++) {
      num += this.metrica[Indexes[i]] * this.Matriz[Indexes[i]][this.incognita[1]]
      den += Math.abs(this.metrica[Indexes[i]])
    }
    return Math.round((num/den)*100)/100
  }
```

```js
  PrediccionMedia(Indexes) {
    let num = 0;
    let den = 0;
    let mediaU = []
    this.Matriz[this.incognita[0]].forEach(element=> {
      if (element != "*") mediaU.push(element);
    })
    mediaU = mediaU.reduce((value1, value2) => value1 + value2, 0)/mediaU.length;
    for (let i = 0; i < Indexes.length; i++) {
      let mediaV = this.Matriz[Indexes[i]].reduce((value1, value2) => value1 + value2, 0)/this.Matriz[Indexes[i]].length
      num += this.metrica[Indexes[i]] * (this.Matriz[Indexes[i]][this.incognita[1]] - mediaV)
      den += Math.abs(this.metrica[Indexes[i]])
    }
    return Math.round(((num/den) + mediaU)*100)/100
  }
```

Es importante recalcal que el parametro Indexes es un vector con las posiciones de los vecinos (v) mas proximos al usuario (u).

Una vez calculada la prediccion llamamos a la funcion ```Resultado()```, la cual imprime por pantalla la matriz, las metricas de cada vecino y la prediccion.

```js
  Resultado(Indexes) {
    document.getElementById("solucion").innerHTML = '';
    const page = document.getElementById("solucion");
    const table = document.createElement("table");

    let tr = document.createElement("tr")
    for (let i = 0; i <= this.Matriz[0].length; i++) {
      let td = document.createElement("td")
      let b = document.createElement("b")
      if (i === 0) { b.innerHTML = "          "
      } else { b.innerHTML = "Item" + i
      }
      td.appendChild(b)
      tr.appendChild(td)
    }
    table.appendChild(tr)

    this.Matriz.forEach((element, index) => {
      let tr = document.createElement("tr")
      let td = document.createElement("td")
      let b = document.createElement("b")
      b.innerHTML = "Persona" + (index + 1)
      td.appendChild(b)
      tr.appendChild(td)
      element.forEach(element2 => {
        let td = document.createElement("td")
        let span = document.createElement("span")
        if(element2 != '*') {
          span.innerHTML = element2
        }
        else {
          span.innerHTML = this.pred
          span.className = "NewValue"
        }
        td.appendChild(span)
        tr.appendChild(td)
      });
      table.appendChild(tr)
    });

    page.appendChild(table)

    let div = document.createElement("div")
    div.className = "displayColumn"
    let span = document.createElement("span")
    span.innerHTML = "Resultado de la Metrica"
    span.style = "font-weight: bold"
    div.appendChild(span)

    this.metrica.forEach((element, index) => {
      if (index != this.incognita[0]) {
        let span = document.createElement("span")
        span.innerHTML = element;
        if (Indexes.includes(index)) {
          span.className = "NewValue"
        }
        div.appendChild(span)
      }
      else {
        let span = document.createElement("span")
        span.innerHTML = " "
        span.style="height: 26px;"
        div.appendChild(span)
      }
    });

    page.appendChild(div)
  }
```

## Ejemplos de uso

![](src/assets/Img2.png)

![](src/assets/Img3.png)

Nota: **Es importante que los ficheros no tengan niguna linea debajo de la matriz, asi como espacios al final de cada linea**
