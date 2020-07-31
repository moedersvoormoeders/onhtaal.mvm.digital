<template id="search">
  <div class="container">
    <div class="d-flex justify-content-center" v-if="loading">
      <div class="spinner-border" role="status">
        <span class="sr-only">Loading...</span>
      </div>
    </div>

    <div v-if="!loading">
      <h1>Onthaal</h1>
      <form m v-on:submit.prevent="search">
        <div class="row">
          <div class="col-8 big-search">
            <div class="input-group mb-1">
              <div class="input-group-prepend">
                <div class="input-group-text">{{prefix}}</div>
              </div>
              <input type="text" class="form-control" ref="search" v-model="doelgroepnummer" v-focus />
            </div>
          </div>
          <div class="col-4">
            <button
              type="button"
              class="btn btn-lg btn-success"
              v-on:click="search()"
              :disabled="searching"
            >
              <i class="fas fa-search"></i><span class="d-none d-md-inline"> Zoeken</span>
            </button>
          </div>
        </div>
        <div class="row">
          <div class="col-md-4 col-12">
            <div class="input-group mb-3">
              <div class="input-group-prepend">
                <label class="input-group-text" for="inputGroupSelect01">Type</label>
              </div>
              <select class="custom-select" v-model="prefix">
                <option value="MVM">Doelgroep</option>
                <option value="E">Eenmaligen</option>
                <option value>Op naam zoeken</option>
              </select>
            </div>
          </div>
        </div>
      </form>
      <div class="container" v-if="results.length > 0">
        <div class="row" v-for="result in results" v-bind:key="result.id">
          <div class="col-md-1 col-12">
            <i class="fas fa-check-square fa-3x" style="color:green" v-if="result.classificatie == 'Actief'"></i>
            <i class="fas fa-exclamation-triangle fa-3x" style="color:#e0cf50" v-if="result.classificatie == 'CONTROLE'"></i>
            <i class="far fa-history fa-3x" style="color:#e0cf50" v-if="result.classificatie == 'TE VERLENGEN'"></i>
              <i class="fas fa-times-octagon fa-3x" style="color:red" v-if="result.classificatie == 'Niet-actief'"></i>
          </div>
          <div class="col-md-2 col-4 mb-4">
            <div class="col-title">Doelgroepnummer</div>
            {{result.doelgroepnummer}}
          </div>
          <div class="col-md-2 col-4">
            <div class="col-title">Voornaam</div>
            {{result.voornaam}}
          </div>
          <div class="col-md-2 col-4">
            <div class="col-title">Naam</div>
            {{result.naam}}
          </div>
          <div class="col-md-2 col-4">
            <div class="col-title">Dag</div>
            {{result.dag}}
          </div>
          <div class="col-md-2 col-4">
            <div class="col-title">Code</div>
            {{result.code}}
          </div>
          <div class="col-md-2 col-4 offset-md-1">
            <div class="col-title">Einddatum</div>
            {{result.einddatum}}
          </div>
          <div class="col-md-2 col-4">
            <div class="col-title">Classificatie</div>
            {{result.classificatie}}
          </div>
          <div class="col-md-2 col-4">
            <div class="col-title">Reden Controle</div>
            {{result.redenControle}}
          </div>
          <div class="col-md-6 col-12 mt-3 mt-mb-0 offset-md-1">
            <div class="col-title">Print</div>
            <form class="row">
                  <select class="col-5 m-1 form-control" v-model="printType">
                    <option>Gewoon</option>
                    <option>Voorrang</option>
                    <option>Man buiten</option>
                    <option>Leveren bij inschrijving</option>
                  </select>
                  <button type="button" class="col-5 m-1 btn btn-primary print-num" v-on:click="lookupVoeding(result)"><i class="fad fa-print"></i></button>
                  <div class="col-1">
                    <i class="fas fa-check-square fa-3x" style="color:green" v-if="hasPrinted(result.doelgroepnummer)"></i>
                  </div>
              </form>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { klantenService } from "../_services/klanten.service"
import { voedingService } from "../_services/voeding.service"
import * as voedingHelper from "../helpers/voeding"

export default {
  template: "#search",
  components: {},
  data: function() {
    return {
      printed: [],
      printType: "Gewoon",
      loading: false,
      doelgroepnummer: "",
      prefix: "MVM",
      searching: false,
      ticketCount: 1, 
      results: []
    };
  },

  methods: {
    hasPrinted: function(number) {
      return this.printed.includes(number)
    },
    lookupVoeding: function(result) {
      var vm = this
      voedingService.lookUpNumber(result.doelgroepnummer)
      .then((res) => vm.print(result, res))
      .catch((error) => this.$Simplert.open({
            title: "Voeding error!",
            message: error,
            type : "error",
            customCloseBtnText: "Sluiten",
            onClose: function() {
              vm.$refs.search.focus()
            }
          })
          )

        // readding this block turns on registration, this is either for after the corona crisis or when it gets worst
        // we all hope the first
        /*voedingHelper.voedingVandaag(res.data[0]).then(()=> {this.print(result, res.data[0])}, (error) => {
          this.$Simplert.open({
            title: "Voeding registratie error!",
            message: error,
            type : "error",
            customCloseBtnText: "Sluiten",
            onClose: function() {
              vm.$refs.search.focus()
            }
          })
        }) */

    },
    print: function(result, voedingResult) {
      var vm = this;
      result.ticketCount = this.ticketCount
      result.needsMelkpoeder = voedingHelper.needsMelkpoeder(voedingResult)
      result.needsVerjaardag = voedingHelper.needsVerjaardag(voedingResult)
      result.specialeVoeding = voedingResult.specialeVoeding
      result.opmerking = voedingResult.opmerking
      result.printType = vm.printType

      sendPrint(result).then((response)=> {
          vm.printed.push(result.doelgroepnummer)

          if (response.status == "error") {
            this.$Simplert.open({
              title: "Printer probleem!",
              message: result.error,
              type : "error",
              customCloseBtnText: "Sluiten",
              onClose: function() {
                vm.$refs.search.focus()
              }
            });
            return
          }
          this.$Simplert.open({
            title: "Print verstuurd",
            message: "Print opdracht verstuurd naar de voeding",
            type: "success",
            customCloseBtnText: "Sluiten",
            onClose: function() {
              vm.$refs.search.focus()
            }
          });
          this.ticketCount++
      },(error)=> {
        this.$Simplert.open({
          title: "Printer probleem!",
          message: error,
          type : "error",
          customCloseBtnText: "Sluiten",
          onClose: function() {
            vm.$refs.search.focus()
          }
        });
      })
    },
    search: function() {
      let vm = this;

      // reset printType
      vm.printType = "Gewoon"
      this.searching = true;
      let searchPrefix = this.prefix
      let seachTerm = this.doelgroepnummer

      if (this.doelgroepnummer.length >= 11 && !isNaN(parseInt(this.doelgroepnummer,10))) {
        // we have a rijksregisternummer!
        seachTerm = this.doelgroepnummer.substring(0,11);
        searchPrefix = ""
      }

      if (searchPrefix != "MVM") {
        alert("Nog niet ondersteund")
      }

      klantenService.lookUpNumber(`${searchPrefix}${seachTerm}`).then(function(result) {
        vm.searching = false;

        vm.results = [];
        
        vm.results.push({
          id: result.zohoID,
          naam: result.naam,
          voornaam: result.voornaam,
          doelgroepnummer: result.mvmNummer,
          code: result.code,
          dag: result.dag,
          classificatie: result.classificatie,
          redenControle: result.redenControle,
          einddatum: result.einddatum,
          typeVoeding: result.typeVoeding,
          kinderen: result.aantalOnder12Jaar,
          volwassenen: result.aantalBovenOf12Jaar,
        });

        vm.doelgroepnummer = "";
      }).catch((error) => 
          vm.$Simplert.open({
            title: "Geen Resultaten!",
            message: error,
            type: "error",
            customCloseBtnText: "Sluiten"
          })
      );
    }
  },

  created: function() {}
};

async function sendPrint(data = {}) {
  console.log(data)
  const response = await fetch("https://print.voeding.mvm.maartje.dev/print", {
    method: 'POST',
    mode: 'cors',
    cache: 'no-cache',
    credentials: 'same-origin',
    headers: {
      'Content-Type': 'application/json'
       },
    redirect: 'follow',
    referrerPolicy: 'no-referrer',
    body: JSON.stringify(data)
  });
  return await response.json();
}
</script>

<style scoped>
  .col-title {
    font-weight: bold;
    display: block;
  }
</style>