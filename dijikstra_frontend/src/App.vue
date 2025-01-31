<template>
  <div class="container">
    <h1 class="title">Maze Pathfinder</h1>
    <input type="number" v-model.number="matrixSize" min="5" max="30" :disabled="processing" />
    <div class="main-content">
      <div class="controls">
        <button class="btn primary" @click="initializeMaze" :disabled="processing">
          Initialize Maze
        </button>
        <button class="btn primary" @click="generateRandomMaze" :disabled="processing">
          Random Maze
        </button>
        <button class="btn secondary" @click="restartMaze" :disabled="processing">
          Restart
        </button>
      </div>
      <div class="grid">
        <div v-for="(row, i) in grid" :key="i" class="row">
          <div v-for="(cell, j) in row" :key="j" :class="getClass(i, j)" @click="toggleCell(i, j)"></div>
        </div>
      </div>
      <div class="controls">
        <button class="btn primary" @click="findPath" :disabled="processing || !start || !end">
          {{ processing ? "Finding Path..." : "Dijkstra" }}
        </button>
        <button class="btn primary" @click="findPathAStar" :disabled="processing || !start || !end">
          {{ processing ? "Finding Path..." : "A* Algorithm" }}
        </button>
        <button class="btn primary" @click="findbfs" :disabled="processing || !start || !end">
          {{ processing ? "Finding Path..." : "BFS" }}
        </button>
      </div>
    </div>
    <div class="footer">
      <button @click="showPopup = true" class="btn secondary">How it works</button>
      <div v-if="showPopup" class="popup">
        <div class="popup-content">
          <h2>How it works</h2>
          <p>1. Initialize the grid with a number of your choice (default is 10).</p>
          <p>2. Click "Random Maze" to generate a random maze, or manually set the blockade after choosing the start and end nodes by clicking on the grid.</p>
          <p>3. The first click on the grid sets the start node, the second click sets the end node.</p>
          <p>4. Any subsequent clicks on the grid will toggle blockades (walls).</p>
          <button @click="showPopup = false" class="btn secondary">Close</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import axios from "axios";

export default {
  data() {
    return {
      grid: [],
      start: null,
      end: null,
      processing: false,
      matrixSize: 10, // Default size
    };
  },
  methods: {
    initializeMaze() {
      this.grid = Array.from({ length: this.matrixSize }, () => Array(this.matrixSize).fill(0));
      this.start = null;
      this.end = null;
    },
    toggleCell(i, j) {
      if (!this.start) {
        this.start = [i, j];
      } else if (!this.end) {
        this.end = [i, j];
      } else {
        this.grid[i][j] = this.grid[i][j] === 0 ? 1 : 0; // Toggle wall
      }
    },
    getClass(i, j) {
      if (this.start && i === this.start[0] && j === this.start[1]) return "start";
      if (this.end && i === this.end[0] && j === this.end[1]) return "end";
      if (this.grid[i][j] === 3) return "visited"; // Visited cell
      if (this.grid[i][j] === 2) return "path"; // Final path
      return this.grid[i][j] === 1 ? "wall" : "empty";
    },
    async findPath() {
      if (this.processing) return; // Prevent re-triggering
      this.processing = true;

      try {
        const API_BASE_URL = import.meta.env.VITE_API_BASE_URL || "http://localhost:8080";
        const response = await axios.post(`${API_BASE_URL}/api/maze/dijkstra`, {
          maze: this.grid,
          start: this.start,
          end: this.end,
        });

        const { visitedNodes, path } = response.data;

        // Animate visited nodes
        this.animateVisitedNodes(visitedNodes, () => {
          // After all nodes are visited, animate the shortest path
          this.animatePath(path);
        });
      } catch (error) {
        console.error("Error finding path:", error);
        this.processing = false;
      }
    },
    async findPathAStar() {
      if (this.processing) return; // Prevent re-triggering
      this.processing = true;

      console.log(this.grid);
      console.log(this.start);
      console.log(this.end);

      try {
        const API_BASE_URL = import.meta.env.VITE_API_BASE_URL || "http://localhost:8080";
        const response = await axios.post(`${API_BASE_URL}/api/maze/astar`, {
          maze: this.grid,
          start: this.start,
          end: this.end,
        });

        const { visitedNodes, path } = response.data;

        // Animate visited nodes
        this.animateVisitedNodes(visitedNodes, () => {
          // After all nodes are visited, animate the shortest path
          this.animatePath(path);
        });
      } catch (error) {
        console.error("Error finding path:", error);
        this.processing = false;
      }
    },
    async findbfs() {
      if (this.processing) return; // Prevent re-triggering
      this.processing = true;

      try {
        const API_BASE_URL = import.meta.env.VITE_API_BASE_URL || "http://localhost:8080";
        const response = await axios.post(`${API_BASE_URL}/api/maze/bfs`, {
          maze: this.grid,
          start: this.start,
          end: this.end,
        });

        const { visitedNodes, path } = response.data;

        // Animate visited nodes
        this.animateVisitedNodes(visitedNodes, () => {
          // After all nodes are visited, animate the shortest path
          this.animatePath(path);
        });
      } catch (error) {
        console.error("Error finding path:", error);
        this.processing = false;
      }
    },
    restartMaze() {
      this.grid = [];
      this.start = null;
      this.end = null;
      this.processing = false;
    },
    animateVisitedNodes(visitedNodes, callback) {
      let index = 0;

      const interval = setInterval(() => {
        if (index >= visitedNodes.length) {
          clearInterval(interval);
          callback();
          return;
        }

        const [x, y] = visitedNodes[index];
        if (this.grid[x][y] === 0) {
          this.grid[x][y] = 3; // Mark as visited
        }
        index++;
      }, 40); // Delay between nodes
    },
    animatePath(path) {
      let index = 0;

      const interval = setInterval(() => {
        if (index >= path.length) {
          clearInterval(interval);
          this.processing = false;
          return;
        }

        const [x, y] = path[index];
        this.grid[x][y] = 2; // Mark as path
        index++;
      }, 200); // Delay between path cells
    },
    generateRandomMaze() {
      this.grid = Array.from({ length: this.matrixSize }, () =>
        Array.from({ length: this.matrixSize }, () => (Math.random() < 0.3 ? 1 : 0))
      );
      this.start = null;
      this.end = null;
    },
  },
  mounted() {
    this.initializeMaze();
  },
};
</script>

<style scoped>

template {
  font-family: 'Poppins', sans-serif;
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}
.container {
  font-family: 'Poppins', sans-serif;
  text-align: center;
  margin: 20px auto;
  padding: 0 10px;
  overflow:auto;
}

.title {
  font-size: 2rem;
  font-weight: bold;
  color: #2c2a2a;
  margin-bottom: 20px;
  background-color: #dee2e6;
  padding: 10px;
  border-radius: 10px;
}

.main-content {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 30px;
  width: 100%;
  margin: 0 auto;
}

.controls {
  display: flex;
  flex-direction: column;
  align-items: center;
  background: #f8f9fa;
  padding: 15px;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

input[type="number"] {
  width: 80px;
  padding: 10px;
  margin-bottom: 10px;
  border: 2px solid #000000;
  border-radius: 10px;
  text-align: center;
  font-size: 1rem;
}

.btn {
  width: 200px;
  padding: 12px;
  margin: 5px 0;
  border: none;
  border-radius: 6px;
  font-size: 1rem;
  cursor: pointer;
  transition: all 0.3s ease;
}

.btn.primary {
  background: #000000;
  color: white;
}

.btn.primary:hover {
  color: black;
  background: #f7f7f7;
}

.btn.secondary {
  color: rgb(255, 255, 255);
  background: #000000;
}

.btn.secondary:hover {
  color: black;
  background: #ffffff;
}

.btn:disabled {
  background: #d6d6d6;
  cursor: not-allowed;
}

.grid {
  display: grid;
  gap: 3px;
  justify-content: center;
  padding: 20px;
  background: #e9ecef;
  border-radius: 10px;
  box-shadow: 0 4px 10px rgba(0, 0, 0, 0.1);
}

.footer {
  margin-top: 20px;
  font-size: 1rem;
  color: #2c2a2a;
}

.row {
  display: flex;
}

.row div {
  width: 35px;
  height: 35px;
  border-radius: 6px;
  transition: background-color 0.3s ease;
}

.empty {
  background: #ffffff;
  border: 1px solid #dee2e6;
}

.wall {
  background: #343a40;
}

.start {
  background: #28a745;
}

.end {
  background: #dc3545;
}

.path {
  background: #ffc107;
}

.visited {
  background: #17a2b8;
}

.popup {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.popup-content {
  background: white;
  padding: 20px;
  border-radius: 10px;
  text-align: left;
  max-width: 500px;
  width: 90%;
  text-align: center;
}

.popup-content h2 {
  margin-top: 0;
}

.popup-content p {
  margin: 10px 0;
}

@media (min-width: 768px) {
  .main-content {
    flex-direction: row;
  }
}

@media (max-width: 767px) {
  .row div {
    width: 25px;
    height: 25px;
  }
}

@media (max-width: 480px) {
  .row div {
    width: 20px;
    height: 20px;
  }
}
</style>