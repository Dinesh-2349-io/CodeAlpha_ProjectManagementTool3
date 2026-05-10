# CodeAlpha_ProjectManagementTool3
TASK 3 — Project Management Tool

Create this GitHub repository:

CodeAlpha_ProjectManagementTool


---

Full Project Structure

CodeAlpha_ProjectManagementTool/
│
├── backend/
│   ├── models/
│   │   ├── User.js
│   │   ├── Project.js
│   │   ├── Task.js
│   │   └── Comment.js
│   │
│   ├── routes/
│   │   ├── authRoutes.js
│   │   ├── projectRoutes.js
│   │   ├── taskRoutes.js
│   │   └── commentRoutes.js
│   │
│   ├── server.js
│   ├── socket.js
│   ├── package.json
│   └── .env
│
├── frontend/
│   ├── index.html
│   ├── style.css
│   └── script.js
│
├── README.md
└── screenshots/


---

1. Backend Package File

backend/package.json

{
  "name": "project-management-backend",
  "version": "1.0.0",
  "main": "server.js",
  "scripts": {
    "start": "nodemon server.js"
  },
  "dependencies": {
    "bcryptjs": "^2.4.3",
    "cors": "^2.8.5",
    "dotenv": "^16.4.5",
    "express": "^4.18.2",
    "jsonwebtoken": "^9.0.2",
    "mongoose": "^8.5.1",
    "nodemon": "^3.1.4",
    "socket.io": "^4.7.5"
  }
}


---

2. Install Packages

cd backend
npm install


---

3. Server File

backend/server.js

const express = require("express");
const mongoose = require("mongoose");
const cors = require("cors");
const http = require("http");
const { Server } = require("socket.io");

require("dotenv").config();

const app = express();

app.use(cors());
app.use(express.json());

mongoose.connect(process.env.MONGO_URI)
.then(() => console.log("MongoDB Connected"))
.catch(err => console.log(err));

app.use("/api/auth", require("./routes/authRoutes"));
app.use("/api/projects", require("./routes/projectRoutes"));
app.use("/api/tasks", require("./routes/taskRoutes"));
app.use("/api/comments", require("./routes/commentRoutes"));

const server = http.createServer(app);

const io = new Server(server, {
    cors: {
        origin: "*"
    }
});

io.on("connection", (socket) => {

    console.log("User Connected");

    socket.on("taskUpdated", (data) => {
        io.emit("taskNotification", data);
    });

    socket.on("disconnect", () => {
        console.log("User Disconnected");
    });
});

const PORT = process.env.PORT || 5000;

server.listen(PORT, () => {
    console.log(`Server running on port ${PORT}`);
});


---

4. Environment Variables

backend/.env

MONGO_URI=your_mongodb_connection
JWT_SECRET=project_secret
PORT=5000


---

5. User Model

backend/models/User.js

const mongoose = require("mongoose");

const UserSchema = new mongoose.Schema({
    username: String,
    email: String,
    password: String
});

module.exports = mongoose.model("User", UserSchema);


---

6. Project Model

backend/models/Project.js

const mongoose = require("mongoose");

const ProjectSchema = new mongoose.Schema({
    projectName: String,
    description: String,
    members: Array,
    createdAt: {
        type: Date,
        default: Date.now
    }
});

module.exports = mongoose.model("Project", ProjectSchema);


---

7. Task Model

backend/models/Task.js

const mongoose = require("mongoose");

const TaskSchema = new mongoose.Schema({
    projectId: String,
    title: String,
    description: String,
    assignedTo: String,
    status: {
        type: String,
        default: "Pending"
    },
    createdAt: {
        type: Date,
        default: Date.now
    }
});

module.exports = mongoose.model("Task", TaskSchema);


---

8. Comment Model

backend/models/Comment.js

const mongoose = require("mongoose");

const CommentSchema = new mongoose.Schema({
    taskId: String,
    userId: String,
    comment: String,
    createdAt: {
        type: Date,
        default: Date.now
    }
});

module.exports = mongoose.model("Comment", CommentSchema);


---

9. Authentication Routes

backend/routes/authRoutes.js

const express = require("express");
const bcrypt = require("bcryptjs");
const jwt = require("jsonwebtoken");

const router = express.Router();
const User = require("../models/User");

router.post("/register", async (req, res) => {

    const { username, email, password } = req.body;

    const hashedPassword = await bcrypt.hash(password, 10);

    const user = new User({
        username,
        email,
        password: hashedPassword
    });

    await user.save();

    res.json({
        message: "User Registered"
    });
});

router.post("/login", async (req, res) => {

    const { email, password } = req.body;

    const user = await User.findOne({ email });

    if (!user)
        return res.status(400).json({
            message: "User not found"
        });

    const isMatch = await bcrypt.compare(password, user.password);

    if (!isMatch)
        return res.status(400).json({
            message: "Invalid password"
        });

    const token = jwt.sign(
        { id: user._id },
        process.env.JWT_SECRET
    );

    res.json({ token });
});

module.exports = router;


---

10. Project Routes

backend/routes/projectRoutes.js

const express = require("express");
const router = express.Router();

const Project = require("../models/Project");

router.post("/", async (req, res) => {

    const project = new Project(req.body);

    await project.save();

    res.json(project);
});

router.get("/", async (req, res) => {

    const projects = await Project.find();

    res.json(projects);
});

module.exports = router;


---

11. Task Routes

backend/routes/taskRoutes.js

const express = require("express");
const router = express.Router();

const Task = require("../models/Task");

router.post("/", async (req, res) => {

    const task = new Task(req.body);

    await task.save();

    res.json({
        message: "Task Created"
    });
});

router.get("/", async (req, res) => {

    const tasks = await Task.find();

    res.json(tasks);
});

router.put("/:id", async (req, res) => {

    await Task.findByIdAndUpdate(
        req.params.id,
        req.body
    );

    res.json({
        message: "Task Updated"
    });
});

module.exports = router;


---

12. Comment Routes

backend/routes/commentRoutes.js

const express = require("express");
const router = express.Router();

const Comment = require("../models/Comment");

router.post("/", async (req, res) => {

    const comment = new Comment(req.body);

    await comment.save();

    res.json({
        message: "Comment Added"
    });
});

router.get("/:taskId", async (req, res) => {

    const comments = await Comment.find({
        taskId: req.params.taskId
    });

    res.json(comments);
});

module.exports = router;


---

13. Frontend HTML

frontend/index.html

<!DOCTYPE html>
<html>
<head>
    <title>Project Management Tool</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>

<h1>Project Management Dashboard</h1>

<div class="task-box">

    <input type="text" id="taskTitle"
        placeholder="Task Title">

    <textarea id="taskDescription"
        placeholder="Task Description"></textarea>

    <button onclick="createTask()">
        Create Task
    </button>

</div>

<div id="tasks"></div>

<script src="https://cdn.socket.io/4.7.5/socket.io.min.js"></script>

<script src="script.js"></script>

</body>
</html>


---

14. Frontend CSS

frontend/style.css

body {
    font-family: Arial;
    background: #f4f4f4;
    padding: 20px;
}

.task-box {
    margin-bottom: 20px;
}

input, textarea {
    width: 300px;
    margin: 5px 0;
    padding: 10px;
}

.task-card {
    background: white;
    padding: 15px;
    margin-top: 10px;
    border-radius: 5px;
}


---

15. Frontend JavaScript

frontend/script.js

const API = "http://localhost:5000/api/tasks";

const socket = io("http://localhost:5000");

socket.on("taskNotification", (data) => {
    alert(data.message);
});

async function loadTasks() {

    const response = await fetch(API);

    const tasks = await response.json();

    const taskDiv = document.getElementById("tasks");

    taskDiv.innerHTML = "";

    tasks.forEach(task => {

        taskDiv.innerHTML += `
            <div class="task-card">
                <h3>${task.title}</h3>

                <p>${task.description}</p>

                <p>Status: ${task.status}</p>

                <button onclick="updateTask('${task._id}')">
                    Complete
                </button>
            </div>
        `;
    });
}

async function createTask() {

    const title =
        document.getElementById("taskTitle").value;

    const description =
        document.getElementById("taskDescription").value;

    await fetch(API, {

        method: "POST",

        headers: {
            "Content-Type": "application/json"
        },

        body: JSON.stringify({
            title,
            description
        })
    });

    socket.emit("taskUpdated", {
        message: "New Task Added"
    });

    loadTasks();
}

async function updateTask(id) {

    await fetch(API + "/" + id, {

        method: "PUT",

        headers: {
            "Content-Type": "application/json"
        },

        body: JSON.stringify({
            status: "Completed"
        })
    });

    socket.emit("taskUpdated", {
        message: "Task Completed"
    });

    loadTasks();
}

loadTasks();


---

16. README.md

README.md

# CodeAlpha Project Management Tool

## Features
- User Authentication
- Create Projects
- Assign Tasks
- Comments System
- Real-time Notifications
- MongoDB Database
- WebSocket Integration

## Technologies Used
- HTML
- CSS
- JavaScript
- Node.js
- Express.js
- MongoDB
- Socket.IO

## Run Project

### Backend
cd backend
npm install
npm start

### Frontend
Open index.html


---

17. Run the Project

Start Backend

cd backend
npm start

Open Frontend

frontend/index.html


---

18. GitHub Upload Commands

git init
git add .
git commit -m "Project Management Tool"
git branch -M main
git remote add origin YOUR_GITHUB_LINK
git push -u origin main


---

19. Screenshots Required

Inside:

/screenshots

Add:

Dashboard

Task Cards

Project Board

Comments

MongoDB Collections

Backend Running



---

20. Bonus Features for Higher Evaluation

Add: ✔ Drag and Drop Tasks
✔ Kanban Board
✔ Dark Mode
✔ Team Chat
✔ JWT Authentication
✔ Email Notifications
✔ Real-time Updates
✔ Mobile Responsive UI

These help improve your project quality for CodeAlpha internship evaluation and placement opportunities.
