<template>
<div id="app">
  <v-app id="todo">
    <v-data-table
      :headers="headers"
      :items="tasks"
      :single-expand="singleExpand"
      :expanded.sync="expanded"
      item-key="name"
      show-expand
      sort-desc="priority"
      
      class="elevation-1"
    >
<!-- top of list -->
      <template v-slot:top>
        <v-toolbar flat color="grey">
          <v-toolbar-title>Harvey's To Do</v-toolbar-title>
          <v-divider
            class="mx-4"
            inset
            vertical
          ></v-divider>
          <v-spacer></v-spacer>
          <v-dialog v-model="dialog" max-width="500px">
            <template v-slot:activator="{ on }">
              <v-btn color="primary" dark class="mb-2" v-on="on">New Task</v-btn>
            </template>
            <v-card>
<!-- card for a new task or editing an existing task -->
              <v-card-title>
                <span class="headline">{{ formTitle }}</span>
              </v-card-title>
  
              <v-card-text>
                <v-container>
                  <v-row>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field v-model="editedItem.name" label="Task name"></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field v-model="editedItem.priority" label="Priority"></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field v-model="editedItem.estimatedHours" label="Estimated Hours"></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field v-model="editedItem.actualHours" label="Actual Hours"></v-text-field>
                    </v-col>
                    <v-col cols="12" sm="6" md="4">
                      <v-text-field v-model="editedItem.descriptor" label="Description"></v-text-field>
                    </v-col>
                  </v-row>
                </v-container>
              </v-card-text>
  
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="blue darken-1" text @click="close">Cancel</v-btn>
                <v-btn color="blue darken-1" text @click="save">Save</v-btn>
              </v-card-actions>
            </v-card>
          </v-dialog>
        </v-toolbar>
      </template>
<!-- expanded column -->
      <template v-slot:expanded-item="{headers, item}">
            <td :colspan="headers.length">{{item.descriptor}}</td>
      </template>
<!-- action column -->
      <template v-slot:item.actions="{ item }">
        <v-icon
          small
          class="mr-2"
          @click="editItem(item)"
        >
          mdi-pencil
        </v-icon>
        <v-icon
          small
          @click="deleteItem(item)"
        >
          mdi-delete
        </v-icon>
      </template>
<!-- name column -->
      <template v-slot:item.name="props">
          <v-edit-dialog
            :return-value.sync="props.item.name"
            @save="quickSave"
            @cancel="cancel"
            @open="open"
            @close="close"
        >{{props.item.name}}
            <template v-slot:input>
                <v-text-field
                    v-model="props.item.name"
                    :rules="[max25chars]"
                    label="Edit"
                    single-line
                    counter
                ></v-text-field>
            </template>
          </v-edit-dialog>
      </template>
<!-- priority column -->
      <template v-slot:item.priority="props">
          <v-edit-dialog
            :return-value.sync="props.item.priority"
            @save="quickSave"
            @cancel="cancel"
            @open="open"
            @close="close"
        ><v-chip :color="getColor(props.item.priority)" dark> {{props.item.priority}} </v-chip>
            <template v-slot:input>
                <v-text-field
                    v-model="props.item.priority"
                    :rules="[max25chars]"
                    label="Edit"
                    single-line
                    counter
                ></v-text-field>
            </template>
          </v-edit-dialog>
      </template>
<!-- estimated hours column -->
      <template v-slot:item.estimatedHours="props">
          <v-edit-dialog
            :return-value.sync="props.item.estimatedHours"
            @save="quickSave"
            @cancel="cancel"
            @open="open"
            @close="close"
        >{{props.item.estimatedHours}}
            <template v-slot:input>
                <v-text-field
                    v-model="props.item.estimatedHours"
                    :rules="[max25chars]"
                    label="Edit"
                    single-line
                    counter
                ></v-text-field>
            </template>
          </v-edit-dialog>
      </template>
<!-- actual hours column -->
      <template v-slot:item.actualHours="props">
          <v-edit-dialog
            :return-value.sync="props.item.actualHours"
            @save="quickSave"
            @cancel="cancel"
            @open="open"
            @close="close"
        >{{props.item.actualHours}}
            <template v-slot:input>
                <v-text-field
                    v-model="props.item.actualHours"
                    :rules="[max25chars]"
                    label="Edit"
                    single-line
                    counter
                ></v-text-field>
            </template>
          </v-edit-dialog>
      </template>
<!-- complete column -->
      <template v-slot:item.complete="{item}">
          <v-simple-checkbox v-model="item.complete" enabled></v-simple-checkbox>
      </template>
      <template v-slot:no-data>
        <v-btn color="primary" @click="initialize">Reset</v-btn>
      </template>
    </v-data-table>
  </v-app>
</div>
</template>
<script>
export default {
  data: () => ({
    dialog: false,
    expanded: [],
    singleExpand: false,
    headers: [
      {
        text: 'Task',
        align: 'start',
        sortable: false,
        value: 'name',
      },
      { text: 'Priority', value: 'priority' },
      { text: 'Estimated Hours', value: 'estimatedHours' },
      { text: 'Actual Hours', value: 'actualHours' },
      { text: 'Complete', value: 'complete' },
      { text: 'Actions', value: 'actions', sortable: false },
    ],
    tasks: [],
    editedIndex: -1,
    editedItem: {
      name: '',
      priority: 0,
      estimatedHours: 0,
      actualHours: 0,
      complete: false
    },
    defaultItem: {
      name: '',
      priority: 0,
      estimatedHours: 0,
      actualHours: 0,
      complete: false
    },
  }),

  computed: {
    formTitle () {
      return this.editedIndex === -1 ? 'New Item' : 'Edit Item'
    },
  },

  watch: {
    dialog (val) {
      val || this.close()
    },
  },

  created () {
    this.initialize()
  },

  methods: {
    // dummy tasks
    initialize () {
      this.tasks = [
        {
          name: 'Make a To Do List',
          priority: 3,
          estimatedHours: 6.0,
          actualHours: 8,
          complete: true
        },
        {
          name: 'Learn Enough Vue to make a To Do List',
          priority: 3,
          estimatedHours: 1,
          actualHours: 2,
          complete: true
        },
        {
          name: 'Learn Enough Vuetify',
          priority: 3,
          estimatedHours: 2,
          actualHours: 3,
          complete: true
        },
        {
          name: 'Learn Enough Nuxt',
          priority: 3,
          estimatedHours: 1,
          actualHours: 0.5,
          complete: true 
        },
        {
          name: 'Get Back To Eric',
          priority: 3,
          estimatedHours: .1,
          actualHours: .1,
          complete: true 
        },
        {
          name: 'Learn Enough Django',
          priority: 2,
          estimatedHours: 4,
          actualHours: '',
          complete: false 
        },
        {
          name: 'Make This Persistant',
          priority: 1,
          estimatedHours: 1,
          actualHours: '',
          complete: false 
        },
        {
          name: 'Use Persistant Data',
          priority: 1,
          estimatedHours: 3,
          actualHours:'',
          complete: false,
          descriptor: 'Once the data in the table is persistant in an actual database, it will be easier to make this table a little neater.'
        }
      ]
    },
    // called when edit button is selected
    editItem (item) {
      this.editedIndex = this.tasks.indexOf(item)
      this.editedItem = Object.assign({}, item)
      this.dialog = true
    },
    //called when delete button is selected
    deleteItem (item) {
      const index = this.tasks.indexOf(item)
      confirm('Are you sure you want to delete this item?') && this.tasks.splice(index, 1)
    },
    // called when quick edit or form edit is closed
    close () {
      this.dialog = false
      this.$nextTick(() => {
        this.editedItem = Object.assign({}, this.defaultItem)
        this.editedIndex = -1
      })
    },
    // called when quick edit is closed
    quickSave(){

    },
    // called when form edit is closed
    save () {
      if (this.editedIndex > -1) {
        Object.assign(this.tasks[this.editedIndex], this.editedItem)
      } else {
        this.tasks.push(this.editedItem)
      }
      this.close()
    },
    // called when priority is set
    getColor(priority){
        if (priority > 2) return 'red'
        else if (priority > 1) return 'orange'
        else return 'green'
    }
  },
}
</script>
