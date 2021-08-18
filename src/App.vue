<template>
  <div class="mainpage">
    <app-header></app-header>
    <app-scheambox :dsid=dsid v-if="dsid !== undefined"></app-scheambox>
    <app-titlebar :dsid=dsid v-if="dsid !== undefined"></app-titlebar>
    <app-emptypage v-if="dsid === undefined"></app-emptypage>
    <app-contacts :dsid=dsid v-if="dsid !== undefined"></app-contacts>
    <app-ageinfo :dsid=dsid v-if="dsid !== undefined"></app-ageinfo>
    <app-publications :dsid=dsid v-if="dsid !== undefined"></app-publications>
    <app-commondata :dsid=dsid v-if="dsid !== undefined"></app-commondata>
    <app-status></app-status>
    <div class="pagebox">
      <h2>Database Snapshots</h2>
      <p>The most recent <a href="http://www.neotomadb.org/snapshots">Neotoma PostgreSQL Database Snapshot</a> is available online.</p>

      <h2>Data Use Policy</h2>
      <p>We ask all data users to consider the <a href="http://www.neotomadb.org/data/category/use">Neotoma Data Use policy</a> as well as the general guidelines of open and ethical data sharing when using this data.</p>
    </div>
    <app-footer></app-footer>
  </div>
</template>

<script>

  import Footer from './components/footer.vue'
  import Header from './components/header.vue'
  import ageinfo from './components/ageinfo.vue'
  import titleCard from './components/titlecard.vue'
  import publicationCard from './components/publicationCard.vue'
  import contacts from './components/contacts.vue'
  import commondata from './components/commondata.vue'
  import emptyPage from './components/emptypage.vue'
  import datastatus from './components/landingstatus.vue'
  import './assets/text.css';
  import './assets/containers.css'
  import 'bootstrap/dist/css/bootstrap.css'
  import schemaBox from "@/components/schemaBox";

  export default {
    name: 'app',
    props: ['dsid'],
    components: {
      'app-footer': Footer,
      'app-header': Header,
      'app-titlebar': titleCard,
      'app-publications': publicationCard,
      'app-ageinfo': ageinfo,
      'app-contacts': contacts,
      'app-commondata': commondata,
      'app-emptypage': emptyPage,
      'app-status': datastatus,
      'app-scheambox': schemaBox
    },
    provide: function () {
      return {
        dataset: this.dataset,
        pbndataset: this.pbndataset,
        site: this.site,
        items: this.items
      }
    },
    data() {
      return {
        msg: "Rendered!",
        dataset: [],
        pbndataset: null,
        site: null,
        items:{}

      };
    },
    created(){
      this.createItems()
      this.fetchData()
    },


    methods:{
      createItems: function(){
        let self = this

        fetch(process.env.VUE_APP_API_ENDPOINT + '/v2.0/data/datasets/' + this.dsid)
            .then((response) => { return response.json() })
            .then((data) => {
              /* Modifying the values and processing the inputs */
              if(data.data.length === 0){
                this.$router.push('/');
              }

              self.items = data.data[0].site
              self.items.database = self.items.datasets[0].database
              self.items.datasettype = self.items.datasets[0].datasettype.charAt(0).toUpperCase() + self.items.datasets[0].datasettype.slice(1);
              self.items.explorer = "http://apps.neotomadb.org/Explorer/?datasetids=" + self.items.datasets[0].datasetid;
              self.items.currjson = process.env.VUE_APP_API_ENDPOINT + "/v2.0/data/downloads/" + self.items.datasets[0].datasetid;
              self.items.frozenjson = process.env.VUE_APP_API_ENDPOINT + "/v2.0/data/frozen/" + self.items.datasets[0].datasetid;
              self.items.loc = JSON.parse(self.items.geography);
              self.items.coordinates = self.items.loc.coordinates.reverse();

              if (self.items.datasettype === 'Geochronologic') {
                self.items.currjson = ''
                self.items.frozenjson = ''
              }

              if (self.items.datasets[0].doi == null) {
                self.items.doi = ['', 'No DOI minted']
              } else {
                self.items.doi = ['https://dx.doi.org/' + self.items.datasets[0].doi[0],
                  self.items.datasets[0].doi]
              }
              if (self.items.sitedescription === null) {
                self.items.sitedescription = "No description exists for this site."
              }
              if (self.items.sitenotes === null) {
                self.items.sitenotes = "No site notes exists for this site."
              }
              if (self.items.datasets[0].doi === null) {
                self.items.doi = "No DOI has been minted for this site."
              }

              if (self.items.coordinates[0].length > 1) {
                var coordlen = self.items.coordinates[0].length;
                self.items.coordinates = self.items.coordinates[0]
                    .reduce((acc, cur) => [(acc[0] + cur[0]), (acc[1] + cur[1])])
                    .map(x => x / coordlen)
                    .reverse()
              }

              self.items.coord = self.items.coordinates.map(x => Math.round(x * 100) / 100);
            });
      },
      fetchData: function () {
        let self = this

        fetch(process.env.VUE_APP_API_ENDPOINT + '/v2.0/data/datasets/' + this.dsid + '/sites')
            .then((response) => { return response.json() })
            .then((data) => {
              /* Modifying the values and processing the inputs */
              var siteid = data.data[0].siteid
              self.site = siteid

              fetch(process.env.VUE_APP_API_ENDPOINT + '/v2.0/data/sites/' + siteid + '/datasets')
                  .then((response) => { return response.json() })
                  .then((data) => {
                    var dsdt = data.data

                    self.dataset = dsdt.map( function(y)
                    {
                      return {datasetid: y.site.datasets[0].datasetid,
                        datasettype: y.site.datasets[0].datasettype,
                        database: y.site.datasets[0].database}
                    })
                        .flat()
                        .filter(x => x.datasetid !== parseInt(this.dsid))

                  })
            })
      }
    },

  }
</script>
