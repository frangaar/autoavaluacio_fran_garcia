<template>
    <div class="row row-cols-1 row-cols-md-3 g-4">
        <template v-for="modul in moduls">
            <div v-if="modul.actiu == 1" class="col">
                <div class="card">
                    <div class="card-body">
                        <h5 class="card-title">{{ modul.codi }} - {{ modul.sigles }}</h5>
                        <p class="card-text">{{ modul.nom }}</p>
                        <input type="hidden" class="modulId" v-bind:value="modul.id"></input>
                        <button type="button" class="btn btn-success" @click="showForm()">Modificar rúbrica</button>
                    </div>
                </div>
            </div>
        </template>
    </div>

    <div class="modal" tabindex="-1" id="rubricaModal">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title"></h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>
                <div class="modal-body">
                    <template v-for="(resultat,index) in resultats">
                        <div class="mb-3">
                            <h3 class="form-label">{{ resultat.descripcio }}</h3>
                            <template v-for="(criteri,index2) in resultat.criteris">
                                <label class="mt-3">{{ criteri.descripcio }}</label>
                                <select class="form-select" name="selects">
                                    <template v-for="rubrica in criteri.rubriques">
                                        <option v-bind:value="rubrica.nivell">{{ rubrica.descripcio }}</option>
                                    </template>
                                </select>
                            </template>
                        </div>
                    </template>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-success" data-bs-dismiss="modal" aria-label="Save" @click="saveRubrica()">Guardar</button>
                    <button type="button" class="btn btn-danger" data-bs-dismiss="modal" aria-label="Close">Cancelar</button>
                </div>
            </div>
        </div>
    </div>

    <div class="modal" tabindex="-1" id="rubricaVacia">
        <div class="modal-dialog">
            <div class="modal-content">
                <div class="modal-header">
                    <h5 class="modal-title"></h5>
                    <button type="button" class="btn-close" data-bs-dismiss="modal" aria-label="Close"></button>
                </div>
                <div class="modal-body">
                    <h2>No hay datos de rúbrica</h2>
                </div>
                <div class="modal-footer">
                    <button type="button" class="btn btn-danger" data-bs-dismiss="modal" aria-label="Close">Cancelar</button>
                </div>
            </div>
        </div>º
    </div>

    <div id="loader" class="load container-fluid hide">
        <div class="loader"></div>
    </div>

</template>
<script>

import * as bootstrap from 'bootstrap'

export default {

    data(){
        return {
            moduls: [],
            resultats: [],
            myModal: {},
            myModalVacio:{},
            rubricas: {},
            datos: {},
            selectedValues: [],
            notas: {}
        }
    },
    created(){
        this.listarModulos();
    },
    methods:{
        saveRubrica(){

            const me = this;
            const userId = document.querySelector('meta[name="userId"]').content
            let notas = new Array();

            let selectedValues = document.getElementsByName('selects');

            selectedValues.forEach(selected => {
                (selected.value != '') ? notas.push(parseInt(selected.value)) : notas.push(0);
            });

            this.rubricas.notas = notas;

            axios
            .put(`modificar-criteris/${userId}`, me.rubricas)
            .then(response => {

            })
            .catch(error => {

            })

        },
        showForm(){

            const me = this;
            const modulsId = event.target.parentNode.querySelector('.modulId').value;
            const userId = document.querySelector('meta[name="userId"]').content
            let notas = new Array();

            let loader = document.getElementById('loader');
            loader.classList.remove('hide')

            axios
            .get(`rubricas-usuari/${userId}`)
            .then(responseNotas => {

                responseNotas.data.forEach(item => {
                    notas.push(item.pivot.nota);
                });

                me.notas = notas;

            })
            .catch(error => {

            })

            axios
            .get(`rubricas/${modulsId}`)
            .then(response => {

                loader.classList.add('hide')
                me.resultats = response.data;

                if(me.resultats.length > 0){
                    me.myModal = new bootstrap.Modal('#rubricaModal');
                    me.myModal.show();

                    setTimeout(() => {
                        me.fillForm();
                    }, 100);
                }else{
                    me.myModalVacio = new bootstrap.Modal('#rubricaVacia');
                    me.myModalVacio.show();
                }
            })
            .catch(error => {

            })
        },
        fillForm(){

            let selectedValues = document.getElementsByName('selects');
            let index = 0;

            selectedValues.forEach(selected => {
                selected.value = this.notas[index]
                var options = selected.querySelectorAll('option');
                options.forEach(function(option) {
                    if (option.value === selected.value) {
                        option.selected = true;
                    } else {
                        option.selected = false;
                    }
                });
                index++;
            });

        },
        listarModulos(){

            const me = this;
            const userId = document.querySelector('meta[name="userId"]').content

            axios
            .get(`lista-moduls/${userId}`)
            .then(response => {
                me.moduls = response.data;
            })
            .catch(error => {

            })

        }
    }
};
</script>
<style>

.hide{
    display: none !important;
}

.load{
    position: absolute;
    top: 0;
    left: 0;
    height: 100%;
    backdrop-filter: brightness(0.5);
    display: flex;
    flex-wrap: nowrap;
    justify-content: center;
    align-items: center;
}

.loader {
    border: 16px solid #f3f3f3; /* Light grey */
    border-top: 16px solid #3498db; /* Blue */
    border-radius: 50%;
    width: 120px;
    height: 120px;
    animation: spin 2s linear infinite;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

.modal-dialog{
    max-width: 80%;
}

.form-label{
    text-decoration: underline;
}

label{
    font-weight: 700;
}

#rubricaVacia{
    text-align: center
}

</style>
