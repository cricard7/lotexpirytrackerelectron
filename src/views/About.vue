<template>
  <div class="about">

    <div class="container-fluid">
      <div class="row">
        <div class="col-md-12">
          <h1 class="text-center display-4"><u>Lot and Expiry Search</u></h1>
        </div>
      </div> 

      <div class="row">
        <div class="col-md-6 offset-md-3">
          <div class="form-group mt-5">
            <h4><span class="badge badge-secondary mr-2">1</span>Paste Lot Excel Data WITH Headings:(id,product,manufacturer,lot,expiry)</h4>
            <textarea v-model="lots" class="form-control" id="exampleFormControlTextarea1" rows="1"></textarea>
          </div>

        </div>
      </div>
      
      <div class="row">
        <div class="col-md-6 offset-md-3">
          <h4><span class="badge badge-secondary mr-2">2</span>Loader Initials</h4>
          <input v-model="loader" class="form-control form-control-lg mb-5" type="text" placeholder="Loader Initials">
        </div>

      </div>

      <div class="row">
        <div class="col-md-6 offset-md-3">
          <h4><span class="badge badge-secondary mr-2">3</span>Compounder Initials</h4>
          <input v-model="compounder" class="form-control form-control-lg mb-5" type="text" placeholder="Compounder Initials">
        </div>

      </div>

      <div class="row">
        <div class="col-md-6 offset-md-3">
          <h4><span class="badge badge-warning mr-2">4</span>Search Lot</h4>
          <input v-model="searchLot" class="form-control form-control-lg mb-5" type="text" placeholder="SEARCH LOT">
        </div>

      </div>


      <!-- List id,product,manufacturer,lot,expiry-->
      <div class="row">
        <div class="col-md-4 offset-md-2">
          <h4><span class="badge badge-info mr-2">5</span>Select Matching Lot(s)</h4>
            <div class="list-group">
                <div v-for="lot in matchingLots" :key="lot.id" @click="selectedLots.push(lot)" class="list-group-item list-group-item-action border border-info mb-2">
                  <h5 class="mb-1">LOT: {{lot.lot}}</h5>
                  <h5 class="mb-1">EXPIRY: {{lot.expiry}}</h5>
                  <h5 class="mb-1">PRODUCT: {{lot.product}}</h5>
                  <h5 class="mb-1">Manufacturer: {{lot.manufacturer}}</h5>
                </div>  
          </div>

        </div>


        <div class="col-md-4">
       
          <h4><span class="badge badge-primary mr-2">6</span>Copy To Clipboard</h4>
          <button type="button" class="btn btn-success mr-2" @click="getLotStr">Copy</button>
          <button type="button" class="btn btn-danger" @click="selectedLots = []">Clear</button>

          

            <div class="list-group mt-2">
                <div v-for="lot in selectedLots" :key="lot.id" class="list-group-item list-group-item-action list-group-item-primary mb-2">
                  <h5 class="mb-1">LOT: {{lot.lot}}</h5>
                  <h5 class="mb-1">EXPIRY: {{lot.expiry}}</h5>
                  <h5 class="mb-1">PRODUCT: {{lot.product}}</h5>
                  <h5 class="mb-1">Manufacturer: {{lot.manufacturer}}</h5>
                </div>  
          </div>

        </div>
          
      </div>
      
      
    </div>
    
  </div>
</template>

<script>
// @ is an alias to /src

var Papa = require('papaparse');
var Fuse = require('fuse.js');

export default {
  name: 'about',
  components: {
    
  },
  data: function(){
    return{
      msg: "msgdf",
      lots: "", // excel data pasted in
      lotsParsed: "", //array of lots based once parsed by papa parse
      loader: "", //loader initials
      compounder: "", //compounder initials
      searchLot: "", // text searched for in lot
      matchingLots: "", // lots matching the searchLot
      selectedLots: [] // lots selected and clicked through once searched
    }

  },

  methods: {
    getLotStr: function(){

     

      const copyToClipboard = str => {
          const el = document.createElement('textarea');
          el.value = str;
          el.setAttribute('readonly', '');
          el.style.position = 'absolute';
          el.style.left = '-9999px';
          document.body.appendChild(el);
          el.select();
          document.execCommand('copy');
          document.body.removeChild(el);
        };

      //buildString from selected lots, loader, compounder
      // format:
      /*
      C:compounderD:LoaDerL:lotnumberE:expiryP:productM:manufacturer

      */
      // .replace(/\s/g,'')
      //string containing all selected lots info
      var productsStr = "";

      this.selectedLots.forEach(function(item){
        productsStr = productsStr + "L:" + item.lot + "E:" + item.expiry + "P:" + item.product + "M:" + item.manufacturer
      })
    
      
      var lotStr = "C:" + this.compounder + "D:" + this.loader + productsStr
      var minlotStr = lotStr.replace(/\s/g,'');
      copyToClipboard(minlotStr);

      alert("Copied " + minlotStr.length + " characters");
    }
  },

  created: function(){
  },
  computed: {
    parsed: function(){

      var parsedobj = Papa.parse(this.lots, {header: true})
      console.log(parsedobj)
      return parsedobj
    }

  },
  watch: {

    lots: function(newVal){

      this.lotsParsed = Papa.parse(newVal, {header: true}).data
    },
    searchLot: function(newVal){

        var options = {
            shouldSort: true,
            threshold: 0.2,
            location: 0,
            distance: 100,
            maxPatternLength: 32,
            minMatchCharLength: 1,
            keys: [
              "lot"
            ]
          };
      var fuse = new Fuse(this.lotsParsed, options);
      this.matchingLots = fuse.search(newVal);

    }

  }
}
</script>
