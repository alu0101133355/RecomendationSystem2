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
        TF_IDF: Number
      }]
    }
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
          stringDocuments.forEach((term, term_index) => {           // Añadimos el indice del documento, el indice del termino, y el propio termino
            if (term.length > 3) {
              if (term !== "isn't" && term !== "this"               // Comprobamos que sea una palabra medianamente util
                  && term !== "it's" && term !== "that") {
                this.statDocuments.push({
                  doc_index,
                  term_index,
                  term
                })
              }
            }
          })
        })
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
      let i = 1                                                        // Añadimos un indice extra
      this.documents.forEach((element, index) => {
        const doc = document.createElement("li")
        doc.innerHTML = element
        list.appendChild(doc)                                          // Añadimos cada documento a la lista
        const sublist = document.createElement("ul")                   // Creamos una sublista por cada documento
        while (this.statDocuments[i].doc_index === index) {            // Comprobamos si el indice de de los terminos coinciden con el de los documentos
          const term = document.createElement("li")                    // Si coincide lo añadimos a la sublista
          term.innerHTML = this.statDocuments[i].term
          sublist.appendChild(term)
          i = i + 1                                                    // Avanzamos en el vector
          if (i === this.statDocuments.length) break                   // Hasta que no supere la longitud del vector statsDocuments
        }
        list.appendChild(sublist)
      })
      page.appendChild(list)
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