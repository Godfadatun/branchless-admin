<template>
  <div>

    <q-table square title="Users" :data="users" :columns="columns" row-key="phone"  :filter="filter">

      <template v-slot:top-right>
        <q-input outlined dense debounce="300" v-model="filter" placeholder="Search">
          <template v-slot:append>
            <q-icon name="search" />
          </template>
        </q-input>
      </template>

        <template v-slot:body="props">
        <q-tr :props="props">
          <q-td key="name" :props="props">
            {{ props.row.name }}
          </q-td>
          <q-td key="phone" :props="props">
              {{ props.row.phone }}
          </q-td>
          <q-td key="email" :props="props">
              {{ props.row.email }}
          </q-td>
          <q-td key="location" :props="props">
              {{ props.row.location }}
          </q-td>
          <q-td key="gender" :props="props">
              {{ props.row.gender }}
          </q-td>
          <q-td key="verified" :props="props">
            <q-badge :color="props.row.verified ? 'primary' : 'negative'">
              {{ props.row.verified ? 'Verified' : 'Unverified' }}
            </q-badge>
          </q-td>
           <q-td key="status" :props="props">
            <q-badge :color="props.row.status == 'active' ? 'primary' : 'negative'">
              {{ props.row.status }}
            </q-badge>
          </q-td>

          <q-td key="agent" :props="props">
            <q-badge :color="props.row.agent ? 'secondary' : 'primary'">
              {{ props.row.agent ? 'Agent' : 'User'  }}
            </q-badge>
          </q-td>
          <q-td key="createdAt" :props="props">
              {{ formatDate(props.row.createdAt) }}
          </q-td>
          <q-td key="date_of_birth" :props="props">
              {{ formatDate(props.row.date_of_birth) }}
          </q-td>
          <q-td key="action" :props="props" class="q-gutter-xs">
               <q-btn size="sm" no-caps color="primary" label="View" :to="{name: 'user-details', params:{phone: props.row.phone}}" />

              <q-btn size="sm" no-caps :color="props.row.status == 'active' ? 'negative' : 'primary'"  @click="enableDisableUser(props.row.phone, props.row.status)" :loading="loading1 == props.row.phone ? true : false">
                  {{ props.row.status == 'active' ? 'Disable' : 'Enable' }}
                <template v-slot:loading>
                  <div class="text-italic">
                      <q-spinner-oval /> Loading...
                  </div>
                </template>
              </q-btn>

              <q-btn size="sm" no-caps :color="props.row.agent ? 'primary' : 'secondary'" @click="agentFunc(props.row.phone, props.row.agent)" :loading="loading == props.row.phone ? true : false">
               {{ props.row.agent ? 'Make User' :  'Make Agent' }}
                <template v-slot:loading>
                  <div class="text-italic">
                      <q-spinner-oval /> Loading...
                  </div>
                </template>
              </q-btn>

               <q-btn size="sm" no-caps :color="'negative'" @click="deleteEmail(props.row.phone)" :loading="loading2 == props.row.phone ? true : false">
               {{ 'Delete Email' }}
                <template v-slot:loading>
                  <div class="text-italic">
                      <q-spinner-oval /> Deleting email...
                  </div>
                </template>
              </q-btn>
          </q-td>

        </q-tr>
      </template>

    </q-table>

  </div>
</template>

<script>
import { date } from 'quasar'
export default {
  // name: 'ComponentName',
  data () {
    return {
      filter: '',
      loading: false,
      loading1: false,
      loading2: false,
      columns: [
        {
          name: 'name',
          required: true,
          label: 'Full Name',
          align: 'left',
          field: row => row.name,
          format: val => `${val}`,
          sortable: true
        },
        { name: 'phone', align: 'center', label: 'Phone', field: 'phone', sortable: true },
        { name: 'email', label: 'Email', field: 'email', sortable: true },
        { name: 'location', label: 'Location', field: 'location', sortable: true },
        { name: 'gender', label: 'Gender', field: 'gender' },
        { name: 'verified', label: 'Verified', field: 'verified' },
        { name: 'status', label: 'Status', field: 'status', sortable: true },
        { name: 'agent', label: 'Agent', field: 'agent', sortable: true },
        // { name: 'subscribed', label: 'Subscribed', field: 'subscribed', sortable: true },
        { name: 'createdAt', label: 'Join Date', field: 'createdAt', sortable: true, format: val => `${date.formatDate(val, 'YYYY-MM-DD')}` },
        { name: 'date_of_birth', label: 'Birth Date', field: 'date_of_birth', format: val => `${date.formatDate(val, 'YYYY-MM-DD')}`, },
        { name: 'action', label: 'Action', field: '' },
      ]
    }
  },

  computed: {
    users() {
      return this.$store.getters['DataAuth/users'];
    },
  },

  methods: {
    formatDate(data){
      return date.formatDate(data, 'YYYY-MM-DD')
    },



    agentFunc(phone, isAgent){
        if (isAgent) {
          const endPoint = 'agentToUser';
          const message = 'User';
          this.makeAgent(phone, endPoint, message);
        }else{
          const endPoint = 'userToAgent';
          const message = 'Agent';
          this.makeAgent(phone, endPoint, message);
        }

    },

    async makeAgent(phone, endPoint, message){
      this.loading = phone;
      try {
        const response = await this.$axios.post(process.env.Api + '/admin/'+endPoint, { phone: phone });
        const data = await response.data;
        this.$store.dispatch('DataAuth/getUsers');
        this.$q.notify({ color: 'primary', message: data.message + ': User Converted to '+message, icon: 'info'})
        this.loading = false
      } catch (error) {
        this.loading = false
      }
    },

    async enableDisableUser(phone, status){
      this.loading1 = phone;
      try {
        const response = await this.$axios.post(process.env.Api + '/admin/' + (status == 'active' ? 'disableUser' : 'enableUser'), { phone: phone });
        const data = await response.data;
        this.$store.dispatch('DataAuth/getUsers');
        this.$q.notify({ color: 'primary', message: data.message + ': User ' + (status == 'active' ? 'Disabled' : 'Enable'), icon: 'info'})
        this.loading1 = false
      } catch (error) {
        this.loading1 = false
      }
    },

    async deleteEmail(phone){
      this.loading2 = phone;
      const response = await this.$axios.post(process.env.Api + '/admin/deleteEmail', { phone: phone });
      this.loading2 = false
    }

  },
}
</script>
