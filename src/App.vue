<template>
  <div id="app">
    <h1>Distributed Key-Value Store</h1>
    
    <div class="cluster-info">
      <h2>Cluster Status</h2>
      <button @click="getRaftStats">Refresh Cluster Status</button>
      <div v-if="raftStats">
        <p><strong>State:</strong> {{ raftStats.state }}</p>
        <p><strong>Term:</strong> {{ raftStats.term }}</p>
        <p><strong>Leader:</strong> {{ raftStats.leader }}</p>
        <p><strong>Nodes:</strong> {{ raftStats.num_peers }}</p>
      </div>
    </div>
    
    <div class="operation">
      <h2>Set Key-Value Pair</h2>
      <input v-model="setKey" placeholder="Key">
      <input v-model="setValue" placeholder="Value">
      <button @click="setKeyValue">Set</button>
      <p v-if="setError" class="error">{{ setError }}</p>
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
      <p v-if="deleteError" class="error">{{ deleteError }}</p>
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
      allData: {},
      raftStats: null,
      setError: '',
      deleteError: ''
    }
  },
  methods: {
    async setKeyValue() {
      this.setError = '';
      try {
        const response = await fetch(`http://localhost:3000/set?key=${this.setKey}&value=${this.setValue}`, {
          method: 'POST'
        });
        
        if (response.status === 400 && (await response.text()).includes('not leader')) {
          this.setError = 'This node is not the leader. Write operations must be sent to the leader.';
          return;
        }
        
        const data = await response.json();
        if (data.status === 'ok') {
          alert(`Key "${this.setKey}" set successfully`);
          this.setKey = '';
          this.setValue = '';
        }
      } catch (error) {
        console.error('Error:', error);
        this.setError = 'Error setting key-value pair';
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
      this.deleteError = '';
      try {
        const response = await fetch(`http://localhost:3000/delete?key=${this.deleteKey}`, {
          method: 'DELETE'
        });
        
        if (response.status === 400 && (await response.text()).includes('not leader')) {
          this.deleteError = 'This node is not the leader. Write operations must be sent to the leader.';
          return;
        }
        
        const data = await response.json();
        if (data.status === 'ok') {
          alert(`Key "${this.deleteKey}" deleted successfully`);
          this.deleteKey = '';
        }
      } catch (error) {
        console.error('Error:', error);
        this.deleteError = 'Error deleting key';
      }
    },
    async getAll() {
      try {
        const response = await fetch('http://localhost:3000/all');
        this.allData = await response.json();
      } catch (error) {
        console.error('Error:', error);
      }
    },
    async getRaftStats() {
      try {
        const response = await fetch('http://localhost:3000/raft/stats');
        this.raftStats = await response.json();
      } catch (error) {
        console.error('Error:', error);
      }
    }
  },
  mounted() {
    this.getAll();
    this.getRaftStats();
    // Refresh stats every 5 seconds
    setInterval(this.getRaftStats, 5000);
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

.cluster-info, .operation {
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
  margin: 5px;
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

.error {
  color: red;
  font-weight: bold;
}
</style>