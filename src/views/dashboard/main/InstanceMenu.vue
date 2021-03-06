<template>
  <v-container
    id="regular-tables"
    fluid
    tag="section"
  >
    <base-v-component
      heading="Instances"
      link="components/simple-tables"
    />

    <base-material-card
      icon="mdi-clipboard-text"
      title="Instances: click on an instance to show executions for the Instance"
      class="px-5 py-3"
    >
      <v-btn
        type="submit"
        value="Submit"
        color="success"
        class="mr-0"
        @click="loadData"
      >
      Update!
      </v-btn>
      <v-simple-table>
        <thead>
          <tr>
            <th
              v-for="col in instCols"
              :key="col"
              class="primary--text"
            >
              {{ col }}
            </th>
          </tr>
        </thead>
        <tbody>
          <template
            v-for="(inst,i) in instances"
          >
          <!-- TODO: change to collapsible cards -->
            <tr
              :key="i"
              @click="checkInst(i, inst)"
            >
              <td>{{ inst.reference_id }}</td>
              <td>{{ inst.name }}</td>
              <td>{{ inst.created_at | moment }}</td>
              <td>{{ inst.modified_at | moment }}</td>
              <td>{{ inst.executions.length }}</td>
              <td
                class="justify-center layout px-0"
              >
                <v-btn
                  icon
                  class="mx-0"
                  @click="editInstance(i, inst)"
                >
                  <v-icon color="teal">edit</v-icon>
                </v-btn>
                <v-btn
                  icon
                  class="mx-0"
                  @click="deleteInstance(i, inst)"
                >
                  <v-icon color="pink">delete</v-icon>
                </v-btn>
              </td>
            </tr>
            <tr
              v-if="inst.contentVisible"
              :key="inst.reference_id"
            >
              <td :colspan="5">
                <div class="accordian-body">
                  <executions-table
                    :instance-id="i"
                    :executions="inst.executions"
                    v-if="inst.contentVisible"
                  >
                  </executions-table>
                </div>
              </td>
            </tr>
          </template>
        </tbody>
      </v-simple-table>
    </base-material-card>
    <v-alert
      v-model="alert.show"
      dense
      :type="alert.type"
      dismissible
    >
      "{{ alert.text }}"
    </v-alert>
    <div class="py-3" />

  </v-container>
</template>

<script>
  import moment from 'moment'
  import { mapState } from 'vuex'
  import { getInstance, delInstance } from '@/api'
  export default {
    name: 'InstanceTable',
    components: {
      ExecutionsTable: () => import('./ExecutionsTable'),
    },
    data () {
      return {
        instances: [],
        instCols: ['Ref', 'Name', 'Created on', 'Modified on', '# of executions', 'Actions'],
        alert: {
          text: '',
          show: false,
          type: 'success',
        },
      }
    },
    computed: {
      ...mapState(['url', 'user']),
    },
    filters: {
      moment: function (date) {
        return moment(date).fromNow()
      },
    },
    mounted () { this.loadData() },
    methods: {
      loadData () {
        getInstance(this.url, this.user.token)
          .then(response => {
            if (response !== null) {
              this.instances = response.sort((a, b) => (a.modified_at < b.modified_at) ? 1 : -1)
              this.instances.forEach(function (instance) {
                instance.contentVisible = false
                instance.executions.sort((a, b) => (a.modified_at < b.modified_at) ? 1 : -1)
              })
              this.alert = { show: true, text: 'Instances loaded.', type: 'success' }
            } else {
              this.alert = { show: true, text: 'You need to login first.', type: 'error' }
            }
          })
      },
      checkInst (i, inst) {
        inst.contentVisible = !inst.contentVisible & inst.executions.length > 0
        this.$set(this.instances, i, inst)
      },
      editInstance (i, inst) {
        console.log('Editing instance with id: ' + inst.reference_id)
      },
      deleteInstance (i, inst) {
        console.log('Deleting instance with id: ' + inst.reference_id)
        delInstance(this.url, this.user.token, inst.reference_id)
          .then(() => {
            this.instances.splice(i, 1)
            this.alert = { show: true, text: 'Instance ' + inst.reference_id + ' was deleted succesfully.', type: 'success' }
          })
          .catch((error) => {
            console.log(error)
            this.alert = { show: true, text: 'There was an error deleting the instance ' + inst.reference_id + '.', type: 'error' }
          })
      },
    },
  }
</script>
