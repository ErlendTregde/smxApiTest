<template>
    <div class="player-search">
      <h1 class="title">StepManiaX Player Search</h1>
      <input
        type="text"
        v-model="username"
        placeholder="Enter username"
        class="input"
        @keyup.enter="searchPlayer"
      />
      <button class="button" @click="searchPlayer">Search</button>
  
      <div v-if="playerProfile" class="profile">
        <h2 class="username">{{ playerProfile.username }}</h2>
        <p class="description">Description: {{ playerProfile.description }}</p>
        <p class="country">Country: {{ playerProfile.country }}</p>
  
        <div v-if="topScores.length > 0" class="scores-section">
          <h3>Top 10 Scores</h3>
          <table class="scores-table">
            <thead>
              <tr>
                <th>#</th>
                <th>Song</th>
                <th>Artist</th>
                <th>Score</th>
                <th>Date</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="(score, index) in topScores" :key="score.id">
                <td>{{ index + 1 }}</td>
                <td>{{ score.song.title }}</td>
                <td>{{ score.song.artist }}</td>
                <td>{{ score.score }}</td>
                <td>{{ formatDate(score.created_at) }}</td>
              </tr>
            </tbody>
          </table>
        </div>
      </div>
  
      <div v-else-if="error" class="error">
        <p>{{ error }}</p>
      </div>
    </div>
  </template>
  
  <script>
  import axios from "axios";
  
  export default {
    data() {
      return {
        username: "",
        playerProfile: null,
        topScores: [],
        error: null,
        apiBaseUrl: "https://api.smx.573.no/",
      };
    },
    methods: {
      async searchPlayer() {
        if (!this.username) {
          this.error = "Please enter a username.";
          return;
        }
        this.error = null;
  
        try {
          const response = await axios.get(
            `${this.apiBaseUrl}players?q=${encodeURIComponent(
              JSON.stringify({ username: this.username })
            )}`
          );
  
          if (response.data && response.data.length > 0) {
            this.playerProfile = response.data[0];
            await this.fetchTopScores(this.playerProfile.id);
          } else {
            this.error = "Player not found.";
          }
        } catch (err) {
          console.error(err);
          this.error = "An error occurred while fetching the player data.";
        }
      },
      async fetchTopScores(playerId) {
        try {
          const response = await axios.get(
            `${this.apiBaseUrl}scores?q=${encodeURIComponent(
              JSON.stringify({
                "gamer.id": playerId,
                _sort_by: "score",
                _order: "desc",
                _take: 10,
              })
            )}`
          );
  
          if (response.data && response.data.length > 0) {
            this.topScores = response.data;
          } else {
            this.topScores = [];
          }
        } catch (err) {
          console.error(err);
          this.error = "An error occurred while fetching the player's scores.";
        }
      },
      formatDate(date) {
        const options = { year: "numeric", month: "short", day: "numeric" };
        return new Date(date).toLocaleDateString(undefined, options);
      },
    },
  };
  </script>
  
  <style scoped>

.player-search {
  display: flex;
  flex-direction: column;
  justify-content: flex-start; 
  align-items: center;
  text-align: center;
  height: 100vh; 
  padding: 1rem;
  overflow-y: auto; 
}

.title {
  font-size: 1.5rem;
  font-weight: bold;
  margin-bottom: 0.5rem;
  color: #fff;
}

.input {
  padding: 0.4rem;
  margin-bottom: 0.5rem; 
  font-size: 1rem;
  border: 1px solid #ddd;
  border-radius: 4px;
  width: 80%; 
  max-width: 300px;
}


.button {
  padding: 0.5rem 1rem; 
  font-size: 1rem;
  font-weight: bold;
  background-color: #007bff;
  color: #fff;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  transition: background-color 0.3s;
  margin-bottom: 1rem; 
}

.button:hover {
  background-color: #0056b3;
}

.profile {
  margin-top: 1rem;
  width: 100%;
}

.username {
  font-size: 1.2rem;
  font-weight: bold;
  margin-bottom: 0.3rem;
  color: #fff;
}

.description,
.country {
  margin: 0.3rem 0; 
  color: #ddd;
}

.scores-section {
  margin-top: 1rem;
  width: 100%;
  max-width: 800px;
}

.scores-table {
  width: 100%;
  border-collapse: collapse;
  margin: 0 auto;
}

.scores-table th,
.scores-table td {
  border: 1px solid #ddd;
  padding: 6px; 
  text-align: center;
}

.scores-table th {
  background-color: #444; 
  color: #fff;
}

.scores-table td {
  color: #eee;
}

.error {
  color: red;
  font-size: 1rem;
  margin-top: 1rem;
}
</style>

  