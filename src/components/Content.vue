<template>
  <h1>Contenido</h1>
  <div>
    <label for="input-file">Introduce un archivo:</label><br>
    <input type="file" id="input-file">
  </div>
  <button class="button" v-on:click="getData">Calcular</button>
  <div class="solucion" id="solucion"></div>
</template>

<script>
export default {
  name: 'Content',
  data() {
    return {
      documents: [],
      statDocuments: [{
        doc_index: Number,
        term_index: Number,
        term: String,
        TF: Number,
        IDF: Number,
        TF_IDF: Number
      }],
      lengths: []
    }
  },
  mounted () {
    this.i = 1
  },
  methods: {
    getData() {
      this.documents = []
      this.statDocuments = []
      const file = document.getElementById("input-file").files[0];  // Leemos el contenido del fichero
      this.readFileContent(file).then(result => {
        const stringArray = result.split("\n")                      // Dividimos todo el texto en documentos (oraciones)
        stringArray.forEach((element, doc_index) => {               // Cada oracion es un element, junto con su indice
          this.documents.push(element)                              // Añadimos el documento al vector de documentos
          element = element.replace(/,/g,'')                        // Quitamos las comas
          element = element.replaceAll('.', '');                    // Quitamos los puntos
          element = element.toLowerCase()                           // Modificamos todas las palabras a minuscula
          const stringDocuments = element.split(" ")                // Dividmos cada oraciones en palabras (terminos)
          this.lengths.push(stringDocuments.length)
          stringDocuments.forEach((term, term_index) => {                       // Añadimos el indice del documento, el indice del termino, y el propio termino
            this.statDocuments.push({
              doc_index,
              term_index,
              term
            })
          })
        })
        this.calculateTF_IDF()
        this.Resultado()
      });
    },
    readFileContent(file) {
      const reader = new FileReader()
      return new Promise((resolve, reject) => {
        reader.onload = event => resolve(event.target.result)
        reader.onerror = error => reject(error)
        reader.readAsText(file, "UTF-8")
      })
    },
    Resultado() {
      document.getElementById("solucion").innerHTML = '';              // Creamos la solucion
      const page = document.getElementById("solucion");
      const list = document.createElement("ul")                        // Creamos una pagina
      let i = 0                                                        // Añadimos un indice extra
      this.documents.forEach((element, index) => {
        const doc = document.createElement("li")
        doc.innerHTML = element
        list.appendChild(doc)                                          // Añadimos cada documento a la lista
        const sublist = document.createElement("ul")                   // Creamos una sublista por cada documento
        while (this.statDocuments[i].doc_index === index) {            // Comprobamos si el indice de de los terminos coinciden con el de los documentos
          const term = document.createElement("li")                    // Si coincide lo añadimos a la sublista
          term.innerHTML = this.statDocuments[i].term_index + " " + this.statDocuments[i].term + " " + this.statDocuments[i].TF + " " + this.statDocuments[i].IDF + " " + this.statDocuments[i].TF_IDF
          sublist.appendChild(term)
          i = i + 1                                                    // Avanzamos en el vector
          if (i === this.statDocuments.length) break                   // Hasta que no supere la longitud del vector statsDocuments
        }
        list.appendChild(sublist)
      })
      page.appendChild(list)
      console.log(this.statDocuments);
    },
    calculateTF_IDF() { 
      for (let i = 0; i < this.statDocuments.length; i++) {
        let sameDocCount = -1
        let otherDocCount = new Set()
        for (let j = 0; j < this.statDocuments.length; j++) {
          if (this.statDocuments[i].term === this.statDocuments[j].term) {
            if (this.statDocuments[i].doc_index === this.statDocuments[j].doc_index) {
              sameDocCount = sameDocCount + 1
            } else {
              otherDocCount.add(this.statDocuments[j].doc_index)
            }
          }
        }
        console.log(otherDocCount);
        this.statDocuments[i].TF = sameDocCount
        this.statDocuments[i].IDF = Math.log(this.documents.length / otherDocCount.size) / Math.log(10)
        this.statDocuments[i].TF_IDF = sameDocCount * (Math.log(this.documents.length / otherDocCount.size) / Math.log(10))
      }
    }
  }
}
</script>

<style>
.button {
  margin: 20px;
  height: 20px;
  width: 60px;
  background-color: lightblue;
  border-radius: 8px;
}

input {
  margin: 10px;
}

ul li {
  text-align: left;
  margin-top: 20px;
  font-weight: bold;
}

ul {
  margin-left: 30px;
  margin-right: 30px;
}
</style>