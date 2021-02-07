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
          <button @click="deletePerson(person.id)" class="btn">
              DELETE ME
              <div class="loader" :id="'loader' + person.id"></div>
          </button>
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
            axios.post('http://127.0.0.1:8000/api/add', this.person)
                .then(response => response.data)
                .then(console.log)
                .finally(() => {
                    this.person = this.initialValues();
                    this.$refs.form.reset();
                    (this.getPeople())
                })
        },
        deletePerson: function(id){
            document.getElementById(`loader${id}`).classList.toggle('show');
            axios
                .delete('http://127.0.0.1:8000/api/delete', {data: {'id': id}})
                .then(response => response.data)
                .finally(() => {
                    document.getElementById(`loader${id}`).classList.toggle('show');
                    this.getPeople();
                    })
        }
    }
}
</script>

<style scoped>
.btn {
    display: flex;
    justify-content: center;
    align-items: center;
    padding: 2px 5px;
}
.loader {
    display: none;
    margin-left: 5px;
    width: 10px;
    height: 10px;
    border-radius: 50%;
    border: 3px solid transparent;
    border-bottom-color: red;
    animation: spin 0.5s linear infinite;
}

.show {
    display: inline-block;
}
@keyframes spin {
    0% {
        transform: rotate(0deg);
    }
    50% {
        transform: rotate(180deg);
    }
    100% {
        transform: rotate(360deg);
    }
}
</style>