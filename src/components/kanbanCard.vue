<template>
    <div :id="`cardId-${value.id}`"
     class="d-flex align-center elevation-1 white text-center lighten-3 py-3 px-3 mx-4 my-4"
     :draggable='true'
     @dragstart="dragStart($event, value)"
     @dragenter.prevent
     @dragover.prevent>
        <div v-if="value.display" class="d-flex align-center justify-space-between w-100">
            <div class="d-flex align-center">
                <div style="width: 18px; height: 18px; border-radius: 9px; font-size: 12px;" class="d-flex align-center justify-center purple white--text mr-3">
                    <p class="mb-0">{{value.id}}</p>
                </div>
                <p class="mb-0 body-1 gray--text">{{value.title}}</p>
            </div>
            <div class="d-flex">
                <v-icon @click="value.display = false" class="pointer" color="gray">mdi-pencil</v-icon>
                <v-icon icon class="pointer ml-3" color="red" @click="cancelCard()">mdi-delete</v-icon>
            </div>
        </div>
        <div class="w-100" v-else>
            <div style="width: 18px; height: 18px; border-radius: 9px; font-size: 12px;" class="d-flex align-center justify-center purple white--text mr-3">
                <p class="mb-0">{{value.id}}</p>
            </div>
            <v-text-field
                v-model="value.title"
                label="Descreva o card"
                solo
            ></v-text-field>
            <div class="d-flex justify-space-between">
                <v-btn small @click="cancelCard()">Cancelar</v-btn>
                <v-btn small color="green accent-3 white--text" @click="saveCard()">Confirmar</v-btn>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: 'kanbanCard',
    data () {
        return {
            item: { id: 0 , title: '', display: false}
        }
    },
    props: ['value', 'columnId', 'index'],
    methods: {
        dragStart (e, item) {
            const target = e.target
            // Salvar o id da tag html do card, as informações dele e o id da coluna que ele faz parte
            e.dataTransfer.setData('card_id', target.id)
            e.dataTransfer.setData('card_info', JSON.stringify(item))
            e.dataTransfer.setData('card_column_id', `columnId-${this.columnId}`)
            e.dataTransfer.setData('displayMode', this.value.display)

            setTimeout(() => {
                this.$emit('cancel', this.index)
            }, 0);
        },
        saveCard () {
            this.value.display = true
            this.$emit('input', { id: this.value.id, title: this.value.title, display: this.value.display, index: this.index })
        },
        cancelCard () {
            this.$emit('cancel', this.index)
        }
    }
}
</script>

<style lang="css">
    
</style>