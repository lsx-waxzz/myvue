<template>
    <Header @update:data="handleupdateData" />
    <streamgraph2 :data="data" :maxYvalue="maxYvalue"/>
    <streamgraph1 :data="data" :maxYvalue="maxYvalue"/>
    <streamgraph3 :data="data" />
</template>

<script>
import Header from './components/Header.vue'
import streamgraph2 from './components/streamgraph2.vue'
import streamgraph1 from './components/streamgraph1.vue'
import streamgraph3 from './components/streamgraph3.vue'
export default {
    data() {
        return {
            data: [],
            maxYvalue: 200000
        }
    },
    components: {    
        Header,
        streamgraph2,
        streamgraph1,
        streamgraph3
    },
    methods: {
        handleupdateData(data) {
            this.data = data;
        }
    }
}
</script>

<style scoped>

</style>