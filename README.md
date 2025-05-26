# TO-DO List
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>To-Do List with Reminders</title>
  <style>
    :root {
      --bg-color: linear-gradient(to right, #667eea, #764ba2);
      --card-bg: #ffffffcc;
      --text-color: #333;
      --completed: #888;
    }

    body.dark {
      --bg-color: linear-gradient(to right, #2c3e50, #4ca1af);
      --card-bg: #1e1e1ecc;
      --text-color: #f0f0f0;
      --completed: #aaa;
    }

    * {
      box-sizing: border-box;
    }

    body {
      margin: 0;
      padding: 0;
      background: var(--bg-color);
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      color: var(--text-color);
      transition: background 0.3s, color 0.3s;
    }

    .todo-container {
      background: var(--card-bg);
      backdrop-filter: blur(10px);
      padding: 30px;
      border-radius: 16px;
      width: 400px;
      box-shadow: 0 12px 30px rgba(0, 0, 0, 0.2);
    }

    h2 {
      text-align: center;
      margin-bottom: 15px;
    }

    .toggle-theme {
      text-align: center;
      margin-bottom: 10px;
    }

    .toggle-theme button {
      padding: 6px 12px;
      border-radius: 6px;
      border: none;
      cursor: pointer;
      background: #444;
      color: #fff;
    }

    input, button {
      width: 100%;
      padding: 12px;
      margin-top: 10px;
      border-radius: 8px;
      border: 1px solid #ccc;
      font-size: 14px;
    }

    button {
      background-color: #28a745;
      color: white;
      border: none;
      transition: background 0.3s ease;
    }

    button:hover {
      background-color: #218838;
    }

    ul {
      list-style: none;
      padding: 0;
      margin-top: 20px;
    }

    li {
      background: #fdfdfd;
      padding: 15px;
      margin-bottom: 12px;
      border-radius: 10px;
      position: relative;
      display: flex;
      align-items: center;
      justify-content: space-between;
      transition: transform 0.2s, background 0.3s;
    }

    body.dark li {
      background: #2c2c2c;
    }

    li:hover {
      transform: translateY(-2px);
    }

    .task-info {
      flex-grow: 1;
      margin-left: 10px;
    }

    .task-title {
      font-weight: bold;
      color: var(--text-color);
    }

    .task-date {
      font-size: 12px;
      color: #666;
    }

    .completed .task-title {
      text-decoration: line-through;
      color: var(--completed);
    }

    .delete-btn {
      color: #e74c3c;
      font-weight: bold;
      cursor: pointer;
      margin-left: 10px;
      font-size: 16px;
    }

    input[type="checkbox"] {
      transform: scale(1.2);
      cursor: pointer;
    }
  </style>
</head>
<body>
  <div class="todo-container">
    <h2>🌟 To-Do List</h2>
    <div class="toggle-theme">
      <button onclick="toggleDarkMode()">Toggle Dark Mode</button>
    </div>
    <input type="text" id="taskInput" placeholder="Enter task" />
    <input type="datetime-local" id="dateInput" />
    <button onclick="addTask()">Add Task</button>
    <ul id="taskList"></ul>
  </div>

  <!-- Reminder sound -->
  <audio id="reminderSound" src="https://www.soundjay.com/buttons/sounds/beep-07.mp3" preload="auto"></audio>

  <script>
    let tasks = [];

    window.onload = function () {
      const stored = localStorage.getItem("tasks");
      if (stored) {
        tasks = JSON.parse(stored);
        tasks.forEach(task => {
          renderTask(task);
          scheduleReminder(task);
        });
      }

      const theme = localStorage.getItem("theme");
      if (theme === "dark") document.body.classList.add("dark");
    };

    function addTask() {
      const taskText = document.getElementById("taskInput").value.trim();
      const dueDate = document.getElementById("dateInput").value;

      if (!taskText || !dueDate) {
        alert("Please enter task and date.");
        return;
      }

      const task = {
        id: Date.now(),
        text: taskText,
        due: dueDate,
        completed: false
      };

      tasks.push(task);
      saveTasks();
      renderTask(task);
      scheduleReminder(task);

      document.getElementById("taskInput").value = "";
      document.getElementById("dateInput").value = "";
    }

    function renderTask(task) {
      const li = document.createElement("li");
      li.setAttribute("data-id", task.id);
      li.classList.toggle("completed", task.completed);
      li.innerHTML = `
        <input type="checkbox" ${task.completed ? "checked" : ""} onchange="toggleComplete(${task.id}, this)">
        <div class="task-info">
          <div class="task-title">${task.text}</div>
          <div class="task-date">Due: ${new Date(task.due).toLocaleString()}</div>
        </div>
        <span class="delete-btn" onclick="deleteTask(${task.id})">✖</span>
      `;
      document.getElementById("taskList").appendChild(li);
    }

    function deleteTask(id) {
      tasks = tasks.filter(task => task.id !== id);
      saveTasks();
      const li = document.querySelector(`li[data-id='${id}']`);
      if (li) li.remove();
    }

    function toggleComplete(id, checkbox) {
      const task = tasks.find(t => t.id === id);
      task.completed = checkbox.checked;
      saveTasks();
      const li = document.querySelector(`li[data-id='${id}']`);
      li.classList.toggle("completed", task.completed);
    }

    function saveTasks() {
      localStorage.setItem("tasks", JSON.stringify(tasks));
    }

    function scheduleReminder(task) {
      const now = new Date();
      const dueDate = new Date(task.due);
      const timeUntil = dueDate - now;

      if (timeUntil > 0) {
        setTimeout(() => {
          if (!task.completed) {
            alert(`⏰ Reminder: Task "${task.text}" is due now!`);
            document.getElementById("reminderSound").play();
          }
        }, timeUntil);
      }
    }

    function toggleDarkMode() {
      document.body.classList.toggle("dark");
      localStorage.setItem("theme", document.body.classList.contains("dark") ? "dark" : "light");
    }
  </script>
</body>
</html>


