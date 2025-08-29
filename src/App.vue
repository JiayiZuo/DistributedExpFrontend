<template>
  <div id="app">
    <h1>Simple Key-Value Store</h1>
    
    <div class="operation">
      <h2>Set Key-Value Pair</h2>
      <input v-model="setKey" placeholder="Key">
      <input v-model="setValue" placeholder="Value">
      <button @click="setKeyValue">Set</button>
    </div>
    
    <div class="operation">
      <h2>Get Value by Key</h2>
      <input v-model="getKey" placeholder="Key">
      <button @click="getValue">Get</button>
      <p v-if="getResult">Value: {{ getResult }}</p>
    </div>
    
    <div class="operation">
      <h2>Delete Key</h2>
      <input v-model="deleteKey" placeholder="Key">
      <button @click="deleteKeyValue">Delete</button>
    </div>
    
    <div class="operation">
      <h2>All Key-Value Pairs</h2>
      <button @click="getAll">Refresh All</button>
      <ul>
        <li v-for="(value, key) in allData" :key="key">
          <strong>{{ key }}:</strong> {{ value }}
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      setKey: '',
      setValue: '',
      getKey: '',
      getResult: '',
      deleteKey: '',
      allData: {}
    }
  },
  methods: {
    async setKeyValue() {
      try {
        const response = await fetch(`http://localhost:3000/set?key=${this.setKey}&value=${this.setValue}`, {
          method: 'POST'
        });
        const data = await response.json();
        if (data.status === 'ok') {
          alert(`Key "${this.setKey}" set successfully`);
          this.setKey = '';
          this.setValue = '';
        }
      } catch (error) {
        console.error('Error:', error);
      }
    },
    async getValue() {
      try {
        const response = await fetch(`http://localhost:3000/get?key=${this.getKey}`);
        if (response.status === 404) {
          this.getResult = 'Key not found';
          return;
        }
        const data = await response.json();
        this.getResult = data.value;
      } catch (error) {
        console.error('Error:', error);
      }
    },
    async deleteKeyValue() {
      try {
        const response = await fetch(`http://localhost:3000/delete?key=${this.deleteKey}`, {
          method: 'DELETE'
        });
        const data = await response.json();
        if (data.status === 'ok') {
          alert(`Key "${this.deleteKey}" deleted successfully`);
          this.deleteKey = '';
        }
      } catch (error) {
        console.error('Error:', error);
      }
    },
    async getAll() {
      try {
        const response = await fetch('http://localhost:3000/all');
        this.allData = await response.json();
      } catch (error) {
        console.error('Error:', error);
      }
    }
  },
  mounted() {
    this.getAll();
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

.operation {
  margin-bottom: 30px;
  padding: 15px;
  border: 1px solid #ddd;
  border-radius: 5px;
}

input {
  padding: 8px;
  margin: 5px;
  border: 1px solid #ccc;
  border-radius: 4px;
}

button {
  padding: 8px 15px;
  background-color: #4CAF50;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

button:hover {
  background-color: #45a049;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  padding: 5px;
  border-bottom: 1px solid #eee;
}
</style>