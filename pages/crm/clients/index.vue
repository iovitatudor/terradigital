<template>
    <main>
        <div id="pageTable">
            <v-container grid-list-xl fluid>
                <h3>Clients</h3>
                <v-layout row wrap>
                    <v-flex lg12>
                        <v-card>
                            <v-toolbar flat color="white">
                                <v-text-field
                                    flat
                                    solo
                                    prepend-icon="mdi-magnify"
                                    placeholder="Search"
                                    v-model="search"
                                    hide-details
                                    class="hidden-sm-and-down"
                                    ></v-text-field>
                            </v-toolbar>
                            <v-divider></v-divider>
                            <v-card-text class="pa-0">
                                <v-data-table
                                    :headers="tableData"
                                    :search="search"
                                    :items="actions"
                                    class="elevation-1"
                                    item-key="name"
                                    v-model="selected"
                                    >
                                    <template v-slot:[`item`]="{ item, index }">
                                    <tr v-if="item.userId">
                                        <td> <small>{{ index + 1 }}</small> </td>
                                        <td>
                                            <nuxt-link :to="'/crm/clients/edit/' + item.userId._id" class="user-info-link">
                                                <small>{{ item.userId.name }}</small>
                                            </nuxt-link>
                                        </td>
                                        <td>
                                            <small v-for="(cookie, index) in item.userId.cookies" :key="index">
                                                {{ cookie }}
                                            </small>
                                        </td>
                                        <td>
                                            <v-icon color="info" v-if="item.userId.lead">mdi-check</v-icon>
                                            <v-icon color="secondary" v-else>mdi-minus</v-icon>
                                        </td>
                                        <td>
                                            <v-badge
                                                v-if="item.online === true"
                                                value="2"
                                                color="green"
                                                >
                                            </v-badge>
                                            <v-badge
                                                v-else
                                                value="2"
                                                color="yellow"
                                                >
                                            </v-badge>
                                        </td>
                                        <td>
                                            <v-icon color="info" v-if="item.userId.logged">mdi-check</v-icon>
                                            <v-icon color="secondary" v-else>mdi-minus</v-icon>
                                        </td>
                                        <td class="text-center">
                                            <span v-if="item.userId.policy.length">
                                                <v-badge
                                                    v-if="item.userId.policy[0].agreementContact"
                                                    value="2"
                                                    color="green"
                                                    >
                                                </v-badge>
                                                <v-badge
                                                    v-else
                                                    value="2"
                                                    color="red"
                                                    >
                                                </v-badge>
                                            </span>
                                            <span v-else>
                                                <v-badge
                                                    value="2"
                                                    color="silver"
                                                    >
                                                </v-badge>
                                            </span>
                                        </td>
                                        <td>
                                            <nuxt-link :to="'/crm/employees/edit/operators/' + item.assigedManager._id" v-if="item.assigedManager" class="user-info-link">
                                                <small>{{ item.assigedManager.name }}</small>
                                            </nuxt-link>
                                            <small v-else>---</small>
                                        </td>
                                        <td>
                                            <span>{{ item.visitsQty }}</span>
                                        </td>
                                        <td>
                                            <small v-if="item.visitsMin == 0">1 min</small>
                                            <small v-else>{{ item.visitsMin }} min</small>
                                        </td>
                                        <td>
                                            <small>{{ getTimeAgo(item.lastVisit) }}</small>
                                        </td>
                                        <td>
                                            <small>{{ item.currentPage }}</small>
                                        </td>
                                        <td class="actions-td">
                                            <a href="#" @click="openDialog(item.userId)">
                                                <v-icon color="primary">mdi-chat</v-icon>
                                            </a>
                                            <a href="#" @click="openVideoCall(item.userId)">
                                                <v-icon color="primary">mdi-phone</v-icon>
                                            </a>
                                        </td>
                                        <td>
                                            <a href="#" @click="deleteItem(item.userId)">
                                                <v-icon color="#F44336">mdi-delete</v-icon>
                                            </a>
                                        </td>
                                    </tr>
                                </template>
                            </v-data-table>
                        </v-card-text>
                    </v-card>
                </v-flex>
                <v-dialog
                    v-model="dialog"
                    hide-overlay
                    persistent
                    width="300"
                    >
                <v-card
                    color="primary"
                    dark
                    >
                <v-card-text>
                Connecting to {{ roomId }}...
                <v-progress-linear
                    indeterminate
                    color="white"
                    class="mb-0"
                    ></v-progress-linear>
                </v-card-text>
                </v-card>
                </v-dialog>
            </v-layout>
        </v-container>
    </div>
</main>
</template>

<script>

import { mapActions, mapGetters } from 'vuex'

export default {
    layout: 'crm',
    middleware: 'admin',
    data: () => ({
        roomId: false,
        dialog: false,
        tableData: [
            { text: 'ID', value: 'id' },
            { text: 'Name', value: 'name' },
            { text: 'Cookies', value: 'coookies' },
            { text: 'Lead', value: 'lead' },
            { text: 'Online', value: 'online' },
            { text: 'Auth', value: 'auth' },
            { text: 'Policies', value: 'yes' },
            { text: 'Operator', value: 'operator' },
            { text: 'Pages', value: 'pages' },
            { text: 'Duration', value: 'duration' },
            { text: 'Last visit', value: 'last visit' },
            { text: 'On page', value: 'page' },
            { text: 'Contact', value: 'contact' },
            { text: 'Delete', value: 'delete' }
        ],
        search: '',
        selected: [],
    }),
    computed: mapGetters({
        refreshUserData: 'dialog/getRefreshUserData',
        authUser: 'admin/getAuthUser',
        clients: 'admin/getClients',
        actions: 'admin/getActions'
    }),
    watch: {
        async refreshUserData() {
            this.getClientsList('client')
        }
    },
    mounted(){
        this.getClientsList('client')
    },
    methods: {
        ...mapActions({
            'getClientsList': 'admin/getClientsList',
            'removeClient' : 'admin/removeClient',
            setInterlocutor : 'dialog/setInterlocutor',
            initCall : 'dialog/initCall',
        }),
        openVideoCall(user) {
            this.initCall(user)
        },
        openDialog(user){
            this.setInterlocutor(null)
            this.setInterlocutor(user)
        },
        deleteItem(item){
            this.removeClient(item._id)
            this.$socket.emit('refreshUsersData')
            this.$router.push("/crm/clients")
        },
        call(client){
            this.roomId = client._id
            this.dialog = true
            this.$socket.emit('call', this.roomId)
        },
        dateToYMD(date) {
            let strArray=['Jan', 'Feb', 'Mar', 'Apr', 'May', 'Jun', 'Jul', 'Aug', 'Sep', 'Oct', 'Nov', 'Dec'];
            let mins = date.getMinutes();
            let h = date.getHours();
            let d = date.getDate();
            let m = strArray[date.getMonth()];
            let y = date.getFullYear();

            return h + ':' + mins + ' ' + (d <= 9 ? '0' + d : d) + '/' + m + '/' + y;
        },
        getTimeAgo(date){
            return this.dateToYMD(new Date(date))
        },
    }
}
</script>

<style>
    th{
        padding-right: 30px !important;
        position: relative;
    }
    th i{
        position: absolute !important;
        right: 5px;
    }
    a{
        text-decoration: none;
    }
    .actions-td{
        /* min-width: 150px; */
    }
    .user-info-link{
        text-decoration: underline;
        color: #293754 !important;
    }

</style>
