<template> 
  <Header />
  <todoinput />
</template>

<script>
import Header from './components/Header.vue'
import todoinput from './components/toinput.vue'
export default {
  components: {
    Header,
    todoinput
  },
  data() {
    return {
      showfinish: false,
      todoList: [],
      finishList: []
    }
  },
  methods: {
    
  }
}
</script>

<style>

</style>



