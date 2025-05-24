# To-do-list-
A site that helps define tasks and implement them in a practical way. 
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Sojoud - Task Manager</title>
<style>
  @import url('https://fonts.googleapis.com/css2?family=Montserrat:wght@400;700&display=swap');
  body {
    margin: 0;
    font-family: 'Montserrat', sans-serif;
    background: linear-gradient(135deg, #f8cdda, #1f4037);
    color: #333;
    min-height: 100vh;
    display: flex;
    flex-direction: column;
  }
  header {
    background: #ff85a2;
    color: white;
    padding: 12px 20px;
    text-align: center;
    font-weight: 700;
    font-size: 1.4rem;
    letter-spacing: 2px;
  }
  header span {
    font-weight: 400;
    font-size: 1rem;
    display: block;
    margin-top: 4px;
  }
  main {
    flex: 1;
    max-width: 600px;
    margin: 20px auto;
    background: white;
    border-radius: 10px;
    padding: 20px;
    box-shadow: 0 10px 30px rgba(0,0,0,0.1);
  }
  #taskInput {
    width: 100%;
    padding: 12px;
    border: 2px solid #ff85a2;
    border-radius: 6px;
    font-size: 1rem;
    box-sizing: border-box;
  }
  #taskDate {
    margin-top: 10px;
    width: 100%;
    padding: 10px;
    border: 2px solid #ff85a2;
    border-radius: 6px;
    font-size: 1rem;
    box-sizing: border-box;
  }
  button {
    margin-top: 12px;
    background: #ff3c78;
    color: white;
    border: none;
    padding: 12px;
    font-weight: 700;
    width: 100%;
    border-radius: 6px;
    cursor: pointer;
    transition: background 0.3s;
  }
  button:hover {
    background: #d03569;
  }
  ul {
    margin-top: 20px;
    padding: 0;
    list-style: none;
  }
  li {
    display: flex;
    justify-content: space-between;
    align-items: center;
    background: #ffe6f0;
    padding: 12px 15px;
    border-radius: 6px;
    margin-bottom: 12px;
    font-size: 1rem;
    word-break: break-word;
  }
  li.completed {
    text-decoration: line-through;
    opacity: 0.6;
  }
  .task-info {
    flex: 1;
    margin-right: 10px;
  }
  .task-date {
    font-size: 0.85rem;
    color: #a64c71;
  }
  .actions button {
    background: transparent;
    border: none;
    cursor: pointer;
    font-weight: 700;
    font-size: 1rem;
    color: #a64c71;
    margin-left: 10px;
    transition: color 0.3s;
  }
  .actions button:hover {
    color: #ff3c78;
  }
  #notification {
    position: fixed;
    bottom: 20px;
    right: 20px;
    background: #ff3c78;
    color: white;
    padding: 14px 20px;
    border-radius: 30px;
    font-weight: 700;
    display: none;
    box-shadow: 0 5px 15px rgba(255,60,120,0.4);
  }
  footer {
    text-align: center;
    padding: 15px 10px;
    background: #ff85a2;
    color: white;
    font-size: 0.9rem;
  }
</style>
</head>
<body>
<header>
  Sojoud
  <span>Iam just a programmer 🎀♥</span>
</header>
<main>
  <input type="text" id="taskInput" placeholder="Add a new task..." />
  <input type="datetime-local" id="taskDate" />
  <button id="addTaskBtn">Add Task</button>
  <ul id="taskList"></ul>
</main>
<div id="notification"></div>
<footer>
  Task Manager &copy; 2025
</footer>

<script>
  const taskInput = document.getElementById('taskInput');
  const taskDate = document.getElementById('taskDate');
  const addTaskBtn = document.getElementById('addTaskBtn');
  const taskList = document.getElementById('taskList');
  const notification = document.getElementById('notification');

  let tasks = JSON.parse(localStorage.getItem('tasks') || '[]');

  function saveTasks() {
    localStorage.setItem('tasks', JSON.stringify(tasks));
  }

  function renderTasks() {
    taskList.innerHTML = '';
    tasks.forEach((task, index) => {
      const li = document.createElement('li');
      li.className = task.completed ? 'completed' : '';
      const taskInfo = document.createElement('div');
      taskInfo.className = 'task-info';
      taskInfo.textContent = task.text;

      const taskDateElem = document.createElement('div');
      taskDateElem.className = 'task-date';
      if (task.date) {
        const d = new Date(task.date);
        taskDateElem.textContent = d.toLocaleString();
      }

      taskInfo.appendChild(taskDateElem);

      const actions = document.createElement('div');
      actions.className = 'actions';

      const completeBtn = document.createElement('button');
      completeBtn.textContent = task.completed ? 'Undo' : 'Done';
      completeBtn.onclick = () => {
        tasks[index].completed = !tasks[index].completed;
        saveTasks();
        renderTasks();
      };

      const deleteBtn = document.createElement('button');
      deleteBtn.textContent = 'Delete';
      deleteBtn.onclick = () => {
        tasks.splice(index, 1);
        saveTasks();
        renderTasks();
      };

      actions.appendChild(completeBtn);
      actions.appendChild(deleteBtn);

      li.appendChild(taskInfo);
      li.appendChild(actions);
      taskList.appendChild(li);
    });
  }

  function showNotification(message) {
    notification.textContent = message;
    notification.style.display = 'block';
    setTimeout(() => {
      notification.style.display = 'none';
    }, 4000);
  }

  function checkReminders() {
    const now = new Date();
    tasks.forEach((task, index) => {
      if (task.date && !task.completed && !task.notified) {
        const taskTime = new Date(task.date);
        if (taskTime <= now) {
          showNotification(`Reminder: ${task.text}`);
          tasks[index].notified = true;
          saveTasks();
          renderTasks();
        }
      }
    });
  }

  addTaskBtn.addEventListener('click', () => {
    const text = taskInput.value.trim();
    if (!text) return;
    const dateVal = taskDate.value ? new Date(taskDate.value) : null;
    if (dateVal && isNaN(dateVal.getTime())) return;

    tasks.push({
      text,
      date: dateVal ? dateVal.toISOString() : null,
      completed: false,
      notified: false
    });

    saveTasks();
    renderTasks();
    taskInput.value = '';
    taskDate.value = '';
  });

  renderTasks();
  setInterval(checkReminders, 30000);
</script>
</body>
</html>


