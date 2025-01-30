<template>
  <div class="container">
    <h1 class="title">Maze Simulator</h1>
    <div class="grid">
      <div
        v-for="(row, i) in grid"
        :key="i"
        class="row"
      >
        <div
          v-for="(cell, j) in row"
          :key="j"
          :class="getClass(i, j)"
          @click="toggleCell(i, j)"
        ></div>
      </div>
    </div>
    <div class="controls">
      <input type="number" v-model.number="matrixSize" min="5" max="30" :disabled="processing" />
      <button class="btn primary" @click="initializeMaze" :disabled="processing">
        Initialize Maze
      </button>
      <button class="btn primary" @click="findPath" :disabled="processing || !start || !end">
        {{ processing ? "Finding Path..." : "Dijikstraaa" }}
      </button>
      <button class="btn primary" @click="generateRandomMaze" :disabled="processing">
        Generate Random Maze
      </button>
      <button class="btn primary" @click="findPathAStar" :disabled="processing || !start || !end">
        {{ processing ? "Finding Path..." : "A* Algorithm" }}
      </button>
      <button class="btn secondary" @click="restartMaze" :disabled="processing">
        restart
      </button>
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
        const response = await axios.post("http://localhost:8080/api/maze/dijkstra", {
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
        const response = await axios.post("http://localhost:8080/api/maze/astar", {
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
input[type="number"] {
  width: 60px;
  padding: 5px;
  margin-right: 10px;
  border: 1px solid #ccc;
  border-radius: 4px;
  text-align: center;
}

.container {
  font-family: Arial, sans-serif;
  text-align: center;
  margin: 20px auto;
  max-width: 600px;
}
.title {
  font-size: 2rem;
  margin-bottom: 20px;
}
.grid {
  display: grid;
  gap: 2px;
  justify-content: center;
  margin: 20px auto;
  max-width: 400px;
}
.row {
  display: flex;
}
.row div {
  width: 40px;
  height: 40px;
  border-radius: 4px;
  transition: background-color 0.3s ease;
}
.empty {
  background-color: #f0f0f0;
}
.wall {
  background-color: #333;
}
.start {
  background-color: #28a745;
}
.end {
  background-color: #dc3545;
}
.path {
  background-color: #ffc107;
}
.visited {
  background-color: #17a2b8;
}
.controls {
  margin-top: 20px;
}
.btn {
  padding: 10px 20px;
  margin: 5px;
  border: none;
  border-radius: 4px;
  font-size: 1rem;
  cursor: pointer;
  transition: background-color 0.3s ease;
}
.btn.primary {
  background-color: #007bff;
  color: white;
}
.btn.primary:hover {
  background-color: #0056b3;
}
.btn.secondary {
  background-color: #6c757d;
  color: white;
}
.btn.secondary:hover {
  background-color: #5a6268;
}
.btn:disabled {
  background-color: #d6d6d6;
  cursor: not-allowed;
}
</style>
