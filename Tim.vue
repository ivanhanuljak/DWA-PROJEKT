<template>
    <div id="team" class="top-row">
        <div class="img">
            <b-img up alt="Naslovan slika" :src="image"></b-img>
        </div>
        <div>
            <b-button v-if="azuriranjeTima" @click="toggleBForm" variant="outline-danger" style="margin-top: 1.5%">Dodaj novi tim</b-button>
            <b-button v-if="!azuriranjeTima" @click="toggleBForm" variant="outline-success" style="margin-top: 1.5%">Azuriraj tim</b-button>
        </div>

        <div id="redak" class="inside-row">
            <b-form v-if="!azuriranjeTima">
                <b-form-group id="input-group-1">
                    <b-form-input
                            id="input-1"
                            v-model="nazivTima"
                            type="text"
                            required
                            placeholder="Upišite naziv tima"
                    ></b-form-input>
                    <div class="inside-row">

                        <b-form-select class="row-item" v-model="trenutni">
                            <option :value="null" disabled>Odaberite igraca</option>
                            <option v-for="igrac in igraci" v-bind:value="igrac.ImeiPrezime" :key="igrac.id"> {{ igrac.ImeiPrezime }}</option>
                        </b-form-select>
                        <b-button @click="addIgrac" class="row-item" variant="outline-primary">Dodaj igraca</b-button>
                    </div>

                </b-form-group>
            </b-form>
            <b-form v-if="azuriranjeTima">
                <b-form-group id="input-group-1">
                    <b-form-select class="row-item" v-model="odabraniTim">
                        <option :value="null" disabled>Odaberite tim</option>
                        <option v-for="tim in timovi" v-bind:value="[tim.naziv, tim.id]" :key="tim.id"> {{ tim.naziv }}</option>
                    </b-form-select>
                    <div class="inside-row">

                        <b-form-select class="row-item" v-model="trenutni">
                            <option :value="null" disabled>Odaberite igraca</option>
                            <option v-for="igrac in igraci" v-bind:value="igrac.ImeiPrezime" :key="igrac.id"> {{ igrac.ImeiPrezime }}</option>
                        </b-form-select>
                        <b-button @click="addIgrac" class="row-item" variant="outline-primary">Dodaj igraca</b-button>
                    </div>

                </b-form-group>
            </b-form>
            <div class="izvjestaj">
                <div v-if="!azuriranjeTima" class="mt-2">
                    Naziv tima: {{ nazivTima }}
                </div>
                <div v-if="azuriranjeTima" class="mt-2">
                    Odabrani tim: {{ odabraniTim[0] }}
                </div>
                <div class="popis-igraca">

                    <li v-for="item in odabraniIgraciIme" :key="item">{{ item }}</li>

                </div>
            </div>
        </div>
        <b-button v-if="azuriranjeTima" variant="outline-danger" v-on:click="azurirajTim">Azuriraj tim</b-button>
        <b-button v-if="!azuriranjeTima" variant="outline-danger" v-on:click="dodajtim">Dodaj tim</b-button>


    </div>
</template>

<script>
    import axios from 'axios';
    export default {
        name: "Tim",
        beforeMount(){
            this.getData();
        },
        data () {
            return {
                tims: null,

                timovi: [
                    { id: '', naziv: '' }],
                nazivTima: '',
                // TRENUTNI U SELECTU
                trenutni: null,

                // igraci ODABRANI ZA PROJEKT
                odabraniIgraciId: [],
                odabraniIgraciIme: [],
                //SVI igraci I SVI NJIHOVI ATRIBUTI
                igraci: [],
                projekti: [],

                // ZA AZURIRANJE TIMA
                azuriranjeTima: false,
                odabraniTim: [],
                stariIgraciTrenutniTIma: []
            }
        },
        async created () {
            await this.getTeam();
            await this.getIgraci();
        },
        methods: {
            getData() {
                axios.get("http://127.0.0.1:5000/timovi").then(({data}) => {
                    this.tims = data.timovi;
                });
            },
            getTeam () {
                axios.get("http://127.0.0.1:5000/timovi")
                    .then((response) => {

                        this.timovi = response.data.timovi;
                        console.log('res timovi: ', this.timovi);
                        return true;
                    })
                    .catch((error) => {
                        alert(error);
                    });
            },

            getIgraci () {
                axios.get("http://127.0.0.1:5000/igraci")
                    .then((response) => {
                        response.data.igraci.forEach(function(value) {
                            value.forEach(function(value) {
                                this.igraci.push(value);
                            }, this);
                        }, this);
                    })
                    .catch(() => {
                    });
            },
            async dodajtim () {
                const tim = {
                    naziv: this.nazivTima,
                    igraci: this.odabraniIgraciId
                };
                await axios.post("http://127.0.0.1:5000/timovi", tim)
                    .then(() => {
                        this.nazivTima= "";
                        this.odabraniIgraciIme= [];
                        this.odabraniIgraciId = [];

                    })
                    .catch(() => {
                    })
            },
            azurirajTim : function () {
                this.ucitajIgrace();

                const tim = {
                    id: this.odabraniTim[1],
                    naziv: this.odabraniTim[0],
                    igraci: this.stariIgraciTrenutniTIma
                };
                axios.put("http://127.0.0.1:5000/timovi", tim)
                    .then(() => {
                        this.odabraniTim= [];
                    })
                    .catch(() => {
                    })
            },
            addIgrac : function () {
                if (this.trenutni === null) {
                    return 0;
                }else{

                    this.igraci.forEach(function(value, key) {

                        if (this.trenutni === this.igraci[key].ImeiPrezime) {

                            this.odabraniIgraciId.push(this.igraci[key].id);

                            this.odabraniIgraciIme.push(this.igraci[key].ImeiPrezime);
                        }
                    }, this);
                }
            },
            toggleBForm : function () {
                if (this.azuriranjeTima === false) {
                    this.azuriranjeTima = true;
                } else {
                    this.azuriranjeTima = false;
                }
            },
            ucitajIgrace : function () {
                var lista = [];
                this.timovi.forEach(function(value) {
                    if(value.id === this.odabraniTim[1]) {
                        value.igraci.forEach(function(igrac) {
                            lista.push(igrac.id)
                        })
                    }

                }, this);
                this.odabraniIgraciId.forEach(function(value) {
                    lista.push(value);
                });
                this.stariIgraciTrenutniTIma = lista;
            }
        },
        computed: {
            image () {
                return require('../assets/tim.jpg');
            }
        }
    }
</script>

<style scoped>
    #team{
        margin: 0;
        padding-bottom: 5%;
    }
    #redak{
        max-width: 800px;
        margin: 1.5% auto;
    }
    .redak{
        max-width: 800px;
        margin: 0 auto;
    }
    #input-1, .row-item{
        margin-bottom: 2%;
    }
    .inside-row h3{
        float: left;
    }
    .redak-2{
        margin-top: 2%;
        margin-bottom: 2%;
        padding: 3% 3% 3% 3%;
        box-shadow: 0 0 5px 4px rgba(209,184,209,1);
    }
</style>