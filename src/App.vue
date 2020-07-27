<template>
  <v-app>
    <v-container id="main-container"
    @dragenter.prevent
    @dragover.prevent
    @drop="drop($event)">
      <v-row class="h-100" no-gutters>
        <v-col class="h-100" lg="12">
          <div id="columns-wrapper" class="d-flex align-center h-100" style="overflow-x: scroll; min-height: 900px;">
            <kanbanColumn :ref="`columnId-${column.id}`" class="ml-5 mt-3 mb-10" v-for="(column, index) in columns" :key="column.id"
            v-model="columns[index]"
            :index="index"
            @add="placeColumn($event)"
            @delete="deleteColumn($event)"/>
            <v-btn @click="addColumn()" class="ml-10 mb-3" fab color="green">
              <h1 style="color: white;">+</h1>
            </v-btn>
          </div>
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>

<script>
import kanbanColumn from '@/components/kanbanColumn'

export default {
  name: 'App',
  data () {
    return {
      columns: [{ id: 0, title: 'Coluna 0', cards: [] }],
      sequencialColumnId: 0
    }
  },
  components: {
    kanbanColumn
  },
  methods: {
    drop (e) {
      var target = e.target
      // Busca o pai de onde o card foi arrastado, se for o main container eu executo a ação, se for uma coluna
      // Ela resolve a adição do elemento internamente e aqui faço nada
      while (target.id.substring(0, target.id.indexOf('-')) !== `columnId` &&
      target.id !== 'main-container') {
          target = target.parentNode
      }
      if (target.id === "main-container") {
        // Se acaso for dropado no container ele retorna pra coluna de onde saiu
        if (e.dataTransfer.getData('card_info')) {
          var cardObject = JSON.parse(e.dataTransfer.getData('card_info'))
        }
        const card_column_id = e.dataTransfer.getData('card_column_id')
        const columnId = e.dataTransfer.getData('column_id')
        if (e.dataTransfer.getData('column_info')) {
          var columnObject = JSON.parse(e.dataTransfer.getData('column_info'))
        }
        
        // Se eu dropar um card
        if (card_column_id) {
            this.$refs[card_column_id][0].placeItem(cardObject);
        } else if (columnId) {
            // Adiciona a coluna no final
            this.columns.push(columnObject)
        }
      }
    },
    addColumn () {
      this.sequencialColumnId++
      this.columns.push({ id: this.sequencialColumnId, title: `Coluna ${this.sequencialColumnId}`, cards: [] })
    },
    deleteColumn (columnIndex) {
      this.columns.splice(columnIndex, 1)
    },
    // placeInfos = { currentColumnIndex: indice da coluna onde dropei a que estava arrastando, 
    // droppedColumn: coluna que eu estava arrastando }
    placeColumn (placeInfos) {
      // Adiciona ela antes da que ela foi dropada
      this.columns.splice(placeInfos.currentColumnIndex, 0, placeInfos.droppedColumn)
    }
  }
}
</script>

<style>
* {
  margin: 0px;
}
#app {
  background-color: darkblue;
  width: 100%;
  min-height: 100vh;
}
.container {
  max-width: 100%;
  margin: 0px !important;
}
.w-100 {
  width: 100%;
}
.h-100 {
  height: 100%;
}
.pointer {
  cursor: pointer;
}
</style>
