<template>
    <v-col :id="`columnId-${value.id}`" draggable="true" @dragstart="dragStart($event, value)" 
    @drop="drop($event)"
    @dragenter.prevent
    @dragover.prevent
    lg="3" style="align-self: start; min-height: 900px; background-color: #E9EAEF; max-height: 900px; overflow-y: scroll;">
        <div v-if="!editTitle" class="d-flex justify-center align-center mt-4">
            <h2 class="text-center">{{value.title}}</h2>
            <v-icon @click="editTitle = true" class="pointer ml-3" color="gray">mdi-pencil</v-icon>
            <v-icon icon class="pointer ml-3" color="red" @click="deleteColumn(index)">mdi-delete</v-icon>
        </div>
        <div v-else class="d-flex justify-center align-center mt-4 mx-4 px-3 white elevation-2">
            <v-text-field
            v-model="value.title"
            label="Insira um titulo"
            single-line
            ></v-text-field>
            <v-icon @click="editTitle = false" class="pointer ml-3" color="green">mdi-check</v-icon>
        </div>
        <v-row>
            <v-col xs=12 class="d-flex justify-end">
                Ordenar: <v-icon @click="order('ASC')" class="ml-3">mdi-arrow-up</v-icon> <v-icon @click="order('DESC')" class="ml-3">mdi-arrow-down</v-icon>
            </v-col>
        </v-row>
        <v-row>
            <v-col id="cards-wrapper" lg="12" class="pt-0">
                <kanbanCard v-for="(item, index) in value.cards" :key="item.id" 
                v-model="value.cards[index]"
                :index="index"
                :columnId="value.id"
                @cancel="cancelCard($event)"
                @confirm="renameCard($event)"/>
            </v-col>
        </v-row>
        <v-row>
            <v-col lg="12" class="d-flex justify-center align-center">
                <v-btn @click="addItem()" class="mb-3" small fab color="#f9770c">
                    <h1 style="color: white;">+</h1>
                </v-btn>
            </v-col>
        </v-row>
    </v-col>
</template>

<script>
import kanbanCard from '@/components/kanbanCard'
export default {
    name: 'kanbanColumn',
    data () {
        return {
            columnTitle: 'Titulo da coluna',
            editTitle: false,
            items: [],
            cardSequencialId: 0
        }
    },
    props: ['value', 'index'],
    components: {
        kanbanCard
    },
    methods: {
        dragStart (e, column) {
            const target = e.target
            if (target.id.substring(0, target.id.indexOf('-')) === 'columnId') {
                // Salvar o id da tag html do card, as informações dele e o id da coluna que ele faz parte
                e.dataTransfer.setData('column_id', target.id)
                e.dataTransfer.setData('column_info', JSON.stringify(column))
    
                setTimeout(() => {
                    this.$emit('delete', this.index)
                }, 0);
            }
        },
        drop (e) {
            if (e.dataTransfer.getData('card_info')) {
                var cardObject = JSON.parse(e.dataTransfer.getData('card_info'))
            }
            const card_column_id = e.dataTransfer.getData('card_column_id')
            const displayMode = (e.dataTransfer.getData('displayMode') == 'true')
            const columnId = e.dataTransfer.getData('column_id')
            if (e.dataTransfer.getData('column_info')) {
                var columnObject = JSON.parse(e.dataTransfer.getData('column_info'))
            }


            // Se eu dropar um card
            if (card_column_id) {
                var target = e.target
                // Se eu droppei o card em outro card eu quero saber qual foi para reordenar
                // Ou inserir antes dele
                var droppedCard = e.target
                
                // Busca a coluna pai para onde o elemento foi arrastado, ou pega o cartão para onde foi arrastado
                while (target.id !== `columnId-${this.value.id}`) {
                    target = target.parentNode
                    if (target.id.substring(0, target.id.indexOf('-')) === 'cardId') {
                        droppedCard = target
                    }
                }
                
                if (droppedCard.id.substring(0, droppedCard.id.indexOf('-')) === 'cardId') {
                    var oldCardIndex = this.value.cards.findIndex(c => { return c.id ===  Number(droppedCard.id.substring(droppedCard.id.indexOf('-') + 1, droppedCard.id.length)) })
                    if (card_column_id === `columnId-${this.value.id}`) {
                        this.value.cards.splice(oldCardIndex, 0, cardObject)
                    } else {
                        this.cardSequencialId++
                        this.value.cards.splice(oldCardIndex, 0, {id: this.cardSequencialId, title: cardObject.title, display: displayMode})
                    }
                }
                // Se a coluna pai for diferente do ID que veio do card adiciona ela na coluna
                else if (card_column_id === `columnId-${this.value.id}`) {
                    this.value.cards.push(cardObject)
                } else if (card_column_id !== `columnId-${this.value.id}`) {
                    // Se ja existir alguem com esse ID ele recebe o sequencial da coluna
                    if (this.value.cards.find(i => { return i.id === cardObject.id })) {
                        this.cardSequencialId++
                        cardObject.id = this.cardSequencialId
                    }
                    this.value.cards.push(cardObject)
                }
            }
            // Se dropar uma coluna na outra
            else if (columnId) {
                // mando o evento para o pai adicionar a coluna antes da que ele dropou
                this.$emit('add', { currentColumnIndex: this.index, droppedColumn: columnObject })
            }
        },
        order (type) {
            if (type === 'ASC') {
                this.value.cards.sort((cardA, cardB) => {
                    if (cardA.title > cardB.title) {
                        return 1
                    }
                    if (cardA.title < cardB.title) {
                        return -1
                    }
                    return 0;
                })
            } else {
                this.value.cards.sort((cardA, cardB) => {
                    if (cardA.title < cardB.title) {
                        return 1
                    }
                    if (cardA.title > cardB.title) {
                        return -1
                    }
                    return 0;
                })
            }
        },
        addItem () {
            while (this.value.cards.find(card => { return card.id === this.cardSequencialId })) {
                this.cardSequencialId++
            }
            this.value.cards.push({ id: this.cardSequencialId, title: '', display: false })
        },
        cancelCard (cardIndex) {
            this.value.cards.splice(cardIndex, 1)
        },
        deleteColumn (columnIndex) {
            this.$emit('delete', columnIndex)
        },
        placeItem (item) {
            // Se ja existir com esse ID adiciona com um ID novo
            if (this.value.cards.find(i => { return i.id === item.id })) {
                this.cardSequencialId++
                this.value.cards.push({ id: this.cardSequencialId, title: item.title, display: item.display })
            } else {
                this.value.cards.push(item)
            }
        },
        beforeMount() {
            this.items = this.value.cards
        },
    }
}
</script>

<style lang="css">
    
</style>