<template>
  <div class="bg-gray-100 flex flex-col min-h-screen justify-center items-center">
      <h1 class="text-2xl text-indigo-700 font-bold pb-4">
      Add random people app
      </h1>          
      <button @click="runTest">CLICK ME</button>
      <div class="p-4 m-2 bg-blue-300 rounded shadow-lg">
      <form @submit.prevent="addPerson" ref="form">
          <div class="flex flex-col justify-center">
            <label for="name" class="p-2 flex justify-between items-center">Name: 
                <input type="text" name="name" class="ml-2 p-2" v-model="person.name" required id='name'>
            </label>
            <label for="surname" class="p-2 flex justify-between items-center">Surname: 
                <input type="text" name="surname" class="ml-2 p-2" v-model="person.surname" required>
            </label>
            <label for="age" class="p-2 flex justify-between items-center">Age: 
                <input type="number" name="age" class="ml-2 p-2" v-model="person.age" required>
            </label>
            <button class="p-2 bg-green-300 shadow rounded">Add Person</button>
          </div>
      </form>
      </div>
      <div class="flex flex-col bg-white p-3 shadow-lg rounded">
        <div class="grid grid-cols-4 gap-3 bg-gray-500 text-white p-2 place-items-center">
            <div>Name</div>
            <div>Surname</div>
            <div>Age</div>
            <div>Action</div>
        </div>
        <div class="max-h-72 overflow-auto">
            <div v-for="person in people" v-bind:key="person.id" class="grid grid-cols-4 gap-3 p-2 place-items-center">
                <div>{{person.age}}</div>
                <div>{{person.name}}</div>
                <div>{{person.surname}}</div>
                <div class="flex justify-center items-center">
                    <button @click="deletePerson(person.id, $event)"
                    class="p-2 ml-6 rounded bg-red-500 hover:bg-red-300 hover:text-black active:bg-red-400 transition text-white whitespace-nowrap">
                        DELETE ME
                    </button>
                    <div class="loader invisible inline-block p-1" :id="'loader' + person.id"></div>
                </div>
            </div>
        </div>
      </div>
  </div>
</template>

<script>
import axios from 'axios'
import Pusher from 'pusher-js'

    
export default {
    data: function(){
        return{
            person: this.initialValues(),
            people: []
        }

    },
    created: function() {
        this.subscribe();
        },
    mounted: function() {
        this.getPeople();
    },
    methods: {
        initialValues: function() {
            return {
                'name': '',
                'surname': '',
                'age': null
            }
        },

        subscribe () {
            let pusher = new Pusher('ca2cb0ed6157c97ed49b', { cluster: 'eu' })
            let channel = pusher.subscribe('new-data')
            channel.bind('new-data-event', data => {
                this.people = data.people
            })
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
                    document.getElementById('name').focus();
                })
        },
        deletePerson: function(id, event){
            event.target.disabled = true;
            document.getElementById(`loader${id}`).classList.toggle('invisible');
            axios
                .delete('http://127.0.0.1:8000/api/delete', {data: {'id': id}})
                .then(response => response.data)
                .then(console.log)
        },
        runTest: function(){
            axios
                .get('http://127.0.0.1:8000/api/check')
                .then(response => response.data)
                .then(console.log)
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
    margin-left: 5px;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    border: 3px solid transparent;
    border-bottom-color: red;
    animation: spin 0.5s linear infinite;
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