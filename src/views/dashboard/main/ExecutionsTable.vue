<template>
  <div>
    <v-simple-table>
      <thead>
        <tr>
          <th
          v-for="col in exeCols"
          :key="col"
          class="primary--text"
          >
          {{ col }}
          </th>
      </tr>
      </thead>
      <tbody>
        <tr
        v-for="(exec,e) in executions"
        :key="e"
        >
          <td>{{ exec.reference_id }}</td>
          <td>{{ exec.config.solver }}</td>
          <td>{{ exec.config.timeLimit }}</td>
          <td>{{ exec.created_at | moment }}</td>
          <td>{{ exec.finished ? "OK" : "NOK" }}</td>
          <td
          class="justify-center layout px-0"
          >
            <v-btn
            icon
            class="mx-0"
            @click="editExecution(e, exec, instanceId)"
            >
                <v-icon color="teal">edit</v-icon>
            </v-btn>
            <v-btn
            icon
            class="mx-0"
            @click="deleteExecution(e, exec, instanceId)"
            >
                <v-icon color="pink">delete</v-icon>
            </v-btn>
          </td>
        </tr>
      </tbody>
    </v-simple-table>
  <v-alert
    v-model="alert.show"
    dense
    :type="alert.type"
    dismissible
  >
    "{{ alert.text }}"
  </v-alert>
  </div>

</template>

<script>
  import moment from 'moment'
  import { mapState } from 'vuex'
  import { delExecution } from '@/api'
  export default {
    name: 'ExecutionTable',
    props: {
      instanceId: {
        type: Number,
      },
      executions: {
        type: Array,
      },
    },
    data () {
      return {
        exeCols: ['Ref', 'Solver', 'Time', 'Created on', 'Status'],
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
    methods: {
      editExecution (i, exec, instanceId) {
        console.log('Editing execution: ' + exec.reference_id)
      },
      deleteExecution (i, exec, instanceId) {
        console.log('Deleting execution: ' + exec.reference_id)
        delExecution(this.url, this.user.token, exec.reference_id)
          .then(() => {
            this.executions.splice(i, 1)
            this.alert = { show: true, text: 'Execution ' + exec.reference_id + ' was deleted succesfully.', type: 'success' }
          })
          .catch((error) => {
            console.log(error)
            this.alert = { show: true, text: 'There was an error deleting the execution ' + exec.reference_id + '.', type: 'error' }
          })
      },
    },
  }
</script>
