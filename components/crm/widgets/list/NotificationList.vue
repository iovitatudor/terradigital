<template>
    <v-card class="elevation-0">
        <v-toolbar text dense color="transparent">
            <v-toolbar-title>
                <h4>Notification</h4>
            </v-toolbar-title>
        </v-toolbar>
        <v-divider></v-divider>
        <v-card-text text class="pa-0">
            <v-list two-line class="pa-0">
                <template v-for="(item, index) in items">
                    <v-subheader v-if="item.header" :key="item.header">{{ item.header }}</v-subheader>
                    <v-divider v-else-if="item.divider" :key="index"></v-divider>
                    <v-list-item  v-else :key="item.title" @click="handleClick">
                        <v-list-item-avatar :color="item.color">
                            <v-icon dark>{{item.icon}}</v-icon>
                        </v-list-item-avatar>
                        <v-list-item-content>
                            <v-list-item-subtitle v-html="item.title"></v-list-item-subtitle>
                        </v-list-item-content>
                        <v-list-item-action class="caption">
                            {{item.timeLabel}}
                        </v-list-item-action>
                    </v-list-item>
                </template>
            </v-list>
            <v-divider></v-divider>
            <v-btn block text class="ma-0">All</v-btn>
            <v-divider></v-divider>
        </v-card-text>
    </v-card>
</template>

<script>

// import notes from '@/util/notification';

export default {
    data: () => ({
        items: []
    }),
    methods: {
        handleClick: (e) => {
            console.log(e)
        }
    }
}
</script>
