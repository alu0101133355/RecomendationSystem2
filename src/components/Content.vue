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
      documents: []
    }
  },
  methods: {
    getData() {
      this.documents = []
      const file = document.getElementById("input-file").files[0];  // Leemos el contenido del fichero
      this.readFileContent(file).then(result => {
        const stringArray = result.split("\n")
        stringArray.forEach((element) => {
          this.documents.push(element)
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
      document.getElementById("solucion").innerHTML = '';
      const page = document.getElementById("solucion");
      const list = document.createElement("ul")
      this.documents.forEach((element) => {
        const doc = document.createElement("li")
        doc.innerHTML = element
        list.appendChild(doc)
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