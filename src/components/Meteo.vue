<template>
<!-- Header da pagina com o hamburger menu no navBar -->
<v-card class="mx-auto overflow-hidden background" height="100%" >
    <v-app-bar color="blue darken-4" dark>
      <v-app-bar-nav-icon @click="drawer = true"></v-app-bar-nav-icon>
      <v-toolbar-title>Todas as Capitais</v-toolbar-title>
    </v-app-bar>
    <v-navigation-drawer v-model="drawer" absolute temporary>
      <v-list nav dense>
        <v-list-item-group v-model="group" active-class="blue darken--text text--accent-6">
          <v-list-item v-for="(capital, n) in capitais" :key="capital" id="drawer" @click="drawerClick(capital)">
            <v-list-item-icon>
              <v-icon>mdi-map-marker</v-icon>
            </v-list-item-icon>
            <v-list-item-title >{{capitais[n]}}</v-list-item-title>
          </v-list-item>
        </v-list-item-group>
      </v-list>
    </v-navigation-drawer>

    <!-- Container com todo o body da pagina que é carregado quando a função getInfo() termina de ser executada e muda a variavel bool loading para false -->
    <v-container v-if="!loading">
        <!-- Card mais transparente que contem os outros card com a previsão do tempo para cada um dos 5 dias enviados pelo API -->
        <v-card class="card">
            <!-- Titulo com o nome da primeira cidade carregada (São Paulo) e a data do dia atual mostrada como hoje -->
            <h1 style="color: white;">{{selectedCity}} - {{forecastData[selectedCity][datesRange[1]]["manha"]["uf"]}}</h1>
            <!-- Container com um for loop que roda 2 vezes para gerar o conteudo dos 2 primeiros cards (hoje e amanhã). Eles estão em loops separados das outras 
            3 datas (5 no total) porque esses dias sao mostrados como Hoje e Amanhã e eles são os unicos dias que possuem a informação divida pelos períodos da 
            manhã, tarde e noite -->
            <v-container v-for="index in 2" :key="index.ind">                            
              <v-card class="eachDay">
                <h2>{{datesUpdated[index-1]}} - {{datesRange[index]}}</h2>                
                <v-row justify="space-between">
                  <!-- For loop para cada periodo do dia -->
                  <v-card v-for="(period, i) in periods" :key="i.i" class="period">
                    <h3>{{period[0].toUpperCase() + period.slice(1)}}</h3>
                    <!-- Um container em linha que possui 2 colunas, uma para as informações de texto e uma apenas com a imagem enviada pelo API -->
                    <v-row>
                      <v-col>                        
                        <v-container>{{forecastData[selectedCity][datesRange[index]][period]["resumo"]}}</v-container>
                        <v-container style="color: red;">Max {{forecastData[selectedCity][datesRange[index]][period]["temp_max"]}}°C</v-container>
                        <v-container style="color: blue;">Min {{forecastData[selectedCity][datesRange[index]][period]["temp_min"]}}°C</v-container>
                      </v-col>
                      <v-col>
                        <img :src="`${forecastData[selectedCity][datesRange[index]][period]['icone']}`">
                      </v-col>
                    </v-row>
                  </v-card>
                </v-row>  
              </v-card>              
            </v-container>
            <!-- For loop para os outros 3 dias da previsão, o Inmet não envia as previsões do tempo por periodo (manha, tarde e noite) para esses 3 dias -->
            <v-container v-for="ind in 3" :key="ind.i">                            
              <v-card class="eachDay">
                <h2>{{datesUpdated[ind + 1]}} - {{datesRange[ind + 2]}}</h2>                
                <v-row justify="space-between">                  
                  <v-card class="period">                    
                    <v-row>
                      <v-col>                                                                      
                        <v-container>{{forecastData[selectedCity][datesRange[ind + 2]]["resumo"]}}</v-container>
                        <v-container style="color: red;">Max {{forecastData[selectedCity][datesRange[ind + 2]]["temp_max"]}}°C</v-container>
                        <v-container style="color: blue;">Min {{forecastData[selectedCity][datesRange[ind + 2]]["temp_min"]}}°C</v-container>
                      </v-col>
                      <v-col>
                        <img :src="`${forecastData[selectedCity][datesRange[ind + 2]]['icone']}`">
                      </v-col>
                    </v-row>
                  </v-card>
                </v-row>  
              </v-card>              
            </v-container>          
        </v-card>
    </v-container>
    <!-- Imagem de loading branca que é mostrada até que os dados sejam carregados no componente vue -->
    <v-row justify="center" style="margin-top: 50px;">
      <v-progress-circular v-if="loading"
      :size="70"
      :width="7"
      color="white"
      indeterminate
      ></v-progress-circular>
    </v-row>
  </v-card>
</template>

<!-- O framework VueJs funciona com 3 componentes: Template, que contém o codigo acima, representando a estrutura de conteudo da pagina. O script, que é
responsável por toda a lógica e gerenciamento de informações recebidas. E o style, cujo uso é opcional e é utilizado para o código CSS -->
<script>
  export default {
    // Antes da página ser montada, o VueJs executa a função getInfo para carregar os dados que serão manipulados. Para mais detalhes sobre o beforeMount olhar em 
    // https://v3.vuejs.org/api/options-lifecycle-hooks.html
    beforeMount (){
        this.getInfo()
    },
    // O componente data armazena os dados gerenciados pelos componentes Vue
    data: () => ({
        forecastData: [],
        imgSrc: String,
        drawer: false,
        selectedCity: "São Paulo",
        datesRange: [String],
        datesUpdated: [""],
        dateConverter: 
          {
          "Mon":"Segunda-feira",
          "Tue":"Terça-feira",
          "Wed":"Quarta-feira",
          "Thu":"Quinta-feira",
          "Fri":"Sexta-feira",
          "Sat":"Sábado",
          "Sun":"Domingo"
          }
        ,
        weekDays: ["Segunda-feira", "Terça-feira", "Quarta-feira", "Quinta-feira", "Sexta-feira", "Sábado", "Domingo","Segunda-feira", "Terça-feira","Quarta-feira", "Quinta-feira"],
        today: String,
        periods: ["manha", "tarde", "noite"],
        group: null,
        loading: true,
        capitais: [
            "Rio Branco",
            "Maceió",
            "Macapá",
            "Manaus",
            "Salvador",
            "Fortaleza",
            "Brasília",
            "Vitória",
            "Goiânia",
            "São Luís",
            "Cuiabá",
            "Campo Grande",
            "Belo Horizonte",
            "Belém",
            "João Pessoa",
            "Curitiba",
            "Recife",
            "Teresina",
            "Rio de Janeiro",
            "Natal",
            "Porto Alegre",
            "Porto Velho",
            "Boa Vista",
            "Florianópolis",
            "São Paulo",
            "Aracaju",
            "Palmas"
            ],
    }),
    // O Methods é utilizado para armazenar as funções utilizadas nos componentes
    methods: {
        // Função que serve para dar um get no endereço do API fornecido pelo Inmet e armazenar todo o texto recebido na variável forecastData
        getInfo: async function () {
          const baseURL = "https://apiprevmet3.inmet.gov.br/previsao/capitais";
          try {
              const response = await fetch(baseURL)
              const data = await response.json()
              this.forecastData = data
              const dates = this.forecastData["São Paulo"]

              // For loop para colocar todas as datas do dia e dos proximos 4 dias na variavel datesRange. Essa informação é usada para recuperar os dados
              // Json enviados pelo API (Ex. 25/10/2021, 26/10/2021, 27/10/2021, 28/10/2021, 29/10/2021, 30/10/2021)
              for (var k in dates) {
                  this.datesRange.push(k);
              }                            
          }
          catch (error) {
              console.log(error)
          }
          console.log(this.forecastData)
          this.dateHandler()
          this.loading = false          
        },
        // Função usada para esconder o side menu quando o usuário clicar fora do menu ou selecionar uma cidade
        drawerClick: function (city) {
          this.selectedCity = city
          this.drawer = false            
        },
        // Função que organiza as datas na variável datesUpdated e descobre qual o nome do dia de hoje, já que o API nos envia apenas as datas no formato
        // dd/mm/aaaa e não nos informa qual a data que representa o dia de hoje
        dateHandler: function () {
          let d = new Date();
          this.today = this.dateConverter[d.toString().split(" ")[0]]
          
          for(var i=0; i < 11; i++){
            if(this.today === this.weekDays[i]){
              this.datesUpdated[0] = "Hoje"
              this.datesUpdated[1] = "Amanhã"
              this.datesUpdated[2] = this.weekDays[i+2]
              this.datesUpdated[3] = this.weekDays[i+3]
              this.datesUpdated[4] = this.weekDays[i+4]
              break;
            }
          }
        },
    }
}
</script>

<style scoped>

#drawer:hover {
    background-color: rgba(0, 0, 0, 0.15);
}

.card{
    padding: 30px;
    margin-top: 20px;
    background-color:  rgba(255, 255, 255, 0.25)!important;
}

.background {
    background: url('../assets/praia-do-guaiuba.png') no-repeat center center !important;
    background-size: cover !important;
}

.eachDay {
    background-color:  rgba(255, 255, 255, 0.50)!important;
    padding: 10px;
    color: rgb(5, 44, 150) !important;
    box-shadow: rgba(0, 0, 0, 0.16) 0px 3px 6px, rgba(0, 0, 0, 0.23) 0px 3px 6px !important;
}

.period {
  margin: 20px;
  background-color: transparent !important;
  box-shadow: none !important;
  color: rgb(5, 44, 150) !important;
}

</style>