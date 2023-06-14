<template>
    <b-form @submit="onSubmit" class="form">
        <p class="d-flex align-items-center my-5"> 
            <b-icon class="mx-3" icon="map" font-scale="2"/>
            Insira o destino e o peso
        </p>
        <b-form-group class="mx-3 my-4" id="input-group-1" label="Destino" label-for="input-1">
            <b-form-select 
                class="form-select" aria-label="Default select example"
                id="input-1"
                v-model="form.destino"
                :options="parametro"
                aria-placeholder="a"                
            ></b-form-select>
        </b-form-group>
        <b-form-group class="mx-3 my-4"  id="input-group-2" label="Peso" label-for="input-2">
            <b-form-input 
                id="input-2`" 
                v-model="form.peso"
                type="number" 
                placeholder="300 Kg"
            ></b-form-input>
        </b-form-group>
        <b-button class="mx-3 my-3" type="submit" variant="info">Analizar</b-button>
    </b-form>
</template>

<script>
    export default {
        props: {
            parametro: {
                type: Array,
                required: true
            }
        },
        data() {
            return {
                form: {
                    destino: null,
                    peso: null,
                },
            }
        },

        methods: {
            onSubmit(event) {
                event.preventDefault()
                this.$emit('on-Submit', this.form);
            },
        },
    }
</script>

<style>

.form {
  background-color: rgb(226, 224, 224);
  border-radius: 10px;
}
</style>