<template>
  <div>
      Hello
      <form @submit.prevent="addPerson" ref="form">
          <div>
            <input type="text" name="name" v-model="person.name" required>
            <input type="text" name="surname" v-model="person.surname" required>
            <input type="number" name="age" v-model="person.age" required>
            <button>Add Person</button>
          </div>
      </form>
      <div v-for="person in people" v-bind:key="person.id" style="display: flex">
          <div style="margin-right:10px ">name: {{person.name}}</div>
          <div style="margin-right:10px ">surname: {{person.surname}}</div>
          <div style="margin-right:10px ">age: {{person.age}}</div>
      </div>
  </div>
</template>

<script>
import axios from 'axios'
export default {
    data: function(){
        return{
            person: this.initialValues(),
            people: []
        }

    },
    mounted: function() {
        this.getPeople()
    },
    methods: {
        initialValues: function() {
            return {
                'name': '',
                'surname': '',
                'age': null
            }
        },
        getPeople: async function(){
            this.people = await axios
                                .get('http://127.0.0.1:8000/api')
                                .then(response => response.data)
        },
        addPerson: function(){
            axios.post('http://127.0.0.1:8000/api', this.person)
                .then(response => response.data)
                .then(data => console.log(data))
                .finally(() => {
                    this.person = this.initialValues();
                    this.$refs.form.reset();
                    this.getPeople();
                })
        }
    }
}
</script>

<style>

</style>