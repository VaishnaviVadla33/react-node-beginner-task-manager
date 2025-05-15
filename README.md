# Task Manager Application Documentation

## Overview

Task Manager is a simple web application built with React.js (frontend) and Node.js with Express (backend). The application allows users to:

- View a list of tasks
- Add new tasks
- Mark tasks as completed or incomplete
- Delete tasks

This documentation provides details on the project structure, setup instructions, and explanations of key components.

## Project Structure

```
task-manager/                  # Main project folder
│
├── server/                    # Backend folder
│   ├── server.js              # Express server and API endpoints
│   ├── package.json           # Node.js dependencies
│   └── node_modules/          # Installed dependencies
│
└── client/                    # Frontend folder
    ├── public/                # Static assets
    ├── src/                   # React source code
    │   ├── App.js             # Main React component
    │   ├── App.css            # Styling for App component
    │   ├── index.js           # React entry point
    │   └── ...                # Other React files
    ├── package.json           # React dependencies
    └── node_modules/          # Installed dependencies
```

## Setup Instructions

### Prerequisites

- Node.js and npm installed (download from [nodejs.org](https://nodejs.org/))
- Code editor (recommended: VS Code)

### Step 1: Setting up the Backend

1. Create the project directory structure:
   ```bash
   mkdir task-manager
   cd task-manager
   mkdir server
   cd server
   ```

2. Initialize the Node.js project:
   ```bash
   npm init -y
   ```

3. Install required packages:
   ```bash
   npm install express cors body-parser
   ```

4. Create `server.js` file in the server directory with the following code:
   ```javascript
   const express = require('express');
   const bodyParser = require('body-parser');
   const cors = require('cors');

   const app = express();
   const PORT = 5000;

   // Middleware
   app.use(cors());
   app.use(bodyParser.json());

   // In-memory database for tasks
   let tasks = [
     { id: 1, text: 'Learn React', completed: false },
     { id: 2, text: 'Learn Node.js', completed: false },
     { id: 3, text: 'Build a project', completed: false }
   ];

   // Get all tasks
   app.get('/api/tasks', (req, res) => {
     res.json(tasks);
   });

   // Add a new task
   app.post('/api/tasks', (req, res) => {
     const newTask = {
       id: tasks.length + 1,
       text: req.body.text,
       completed: false
     };
     
     tasks.push(newTask);
     res.status(201).json(newTask);
   });

   // Toggle task completion status
   app.put('/api/tasks/:id', (req, res) => {
     const taskId = parseInt(req.params.id);
     const taskIndex = tasks.findIndex(task => task.id === taskId);
     
     if (taskIndex !== -1) {
       tasks[taskIndex].completed = !tasks[taskIndex].completed;
       res.json(tasks[taskIndex]);
     } else {
       res.status(404).json({ message: 'Task not found' });
     }
   });

   // Delete a task
   app.delete('/api/tasks/:id', (req, res) => {
     const taskId = parseInt(req.params.id);
     tasks = tasks.filter(task => task.id !== taskId);
     res.json({ message: 'Task deleted' });
   });

   app.listen(PORT, () => {
     console.log(`Server running on port ${PORT}`);
   });
   ```

5. Start the server:
   ```bash
   node server.js
   ```

### Step 2: Setting up the Frontend

1. From the main project directory, create a new React app:
   ```bash
   cd ..
   npx create-react-app client
   cd client
   ```

2. Install axios for API requests:
   ```bash
   npm install axios
   ```

3. Replace the content of `src/App.js` with:
   ```javascript
   import React, { useState, useEffect } from 'react';
   import axios from 'axios';
   import './App.css';

   function App() {
     const [tasks, setTasks] = useState([]);
     const [newTask, setNewTask] = useState('');

     // Fetch tasks from the API
     useEffect(() => {
       const fetchTasks = async () => {
         try {
           const response = await axios.get('http://localhost:5000/api/tasks');
           setTasks(response.data);
         } catch (error) {
           console.error('Error fetching tasks:', error);
         }
       };

       fetchTasks();
     }, []);

     // Add a new task
     const addTask = async (e) => {
       e.preventDefault();
       if (!newTask.trim()) return;

       try {
         const response = await axios.post('http://localhost:5000/api/tasks', {
           text: newTask
         });
         setTasks([...tasks, response.data]);
         setNewTask('');
       } catch (error) {
         console.error('Error adding task:', error);
       }
     };

     // Toggle task completion
     const toggleComplete = async (id) => {
       try {
         const response = await axios.put(`http://localhost:5000/api/tasks/${id}`);
         setTasks(tasks.map(task => 
           task.id === id ? response.data : task
         ));
       } catch (error) {
         console.error('Error updating task:', error);
       }
     };

     // Delete a task
     const deleteTask = async (id) => {
       try {
         await axios.delete(`http://localhost:5000/api/tasks/${id}`);
         setTasks(tasks.filter(task => task.id !== id));
       } catch (error) {
         console.error('Error deleting task:', error);
       }
     };

     return (
       <div className="App">
         <header className="App-header">
           <h1>Task Manager</h1>
           <form onSubmit={addTask} className="task-form">
             <input
               type="text"
               value={newTask}
               onChange={(e) => setNewTask(e.target.value)}
               placeholder="Add a new task..."
               className="task-input"
             />
             <button type="submit" className="add-button">Add Task</button>
           </form>
           <div className="task-list">
             {tasks.map(task => (
               <div key={task.id} className={`task-item ${task.completed ? 'completed' : ''}`}>
                 <span 
                   className="task-text"
                   onClick={() => toggleComplete(task.id)}
                 >
                   {task.text}
                 </span>
                 <button 
                   onClick={() => deleteTask(task.id)}
                   className="delete-button"
                 >
                   Delete
                 </button>
               </div>
             ))}
           </div>
         </header>
       </div>
     );
   }

   export default App;
   ```

4. Replace the content of `src/App.css` with:
   ```css
   .App {
     text-align: center;
   }

   .App-header {
     background-color: #282c34;
     min-height: 100vh;
     display: flex;
     flex-direction: column;
     align-items: center;
     justify-content: flex-start;
     padding-top: 40px;
     font-size: calc(10px + 1vmin);
     color: white;
   }

   .task-form {
     margin: 20px 0;
     width: 80%;
     max-width: 500px;
     display: flex;
   }

   .task-input {
     flex-grow: 1;
     padding: 10px;
     font-size: 16px;
     border: none;
     border-radius: 4px 0 0 4px;
   }

   .add-button {
     padding: 10px 15px;
     background-color: #61dafb;
     color: #282c34;
     border: none;
     border-radius: 0 4px 4px 0;
     cursor: pointer;
     font-weight: bold;
   }

   .task-list {
     width: 80%;
     max-width: 500px;
   }

   .task-item {
     display: flex;
     justify-content: space-between;
     align-items: center;
     background-color: #444;
     margin: 10px 0;
     padding: 10px 15px;
     border-radius: 4px;
     transition: all 0.3s;
   }

   .task-item.completed .task-text {
     text-decoration: line-through;
     color: #888;
   }

   .task-text {
     cursor: pointer;
     flex-grow: 1;
     text-align: left;
   }

   .delete-button {
     background-color: #ff6b6b;
     color: white;
     border: none;
     border-radius: 4px;
     padding: 5px 10px;
     cursor: pointer;
   }
   ```

5. Start the React development server:
   ```bash
   npm start
   ```

## Running the Application

To run the complete application:

1. Start the backend server (in one terminal):
   ```bash
   cd task-manager/server
   node server.js
   ```

2. Start the frontend development server (in another terminal):
   ```bash
   cd task-manager/client
   npm start
   ```

3. Open your browser and navigate to `http://localhost:3000`

## Application Components

### Backend (Node.js/Express)

#### server.js
- **Express Setup**: Configures the Express server with necessary middleware
- **Data Storage**: Uses an in-memory array to store tasks
- **API Endpoints**:
  - `GET /api/tasks`: Returns all tasks
  - `POST /api/tasks`: Creates a new task
  - `PUT /api/tasks/:id`: Toggles the completion status of a task
  - `DELETE /api/tasks/:id`: Removes a task

### Frontend (React)

#### App.js
- **State Management**: Uses React hooks to manage application state
  - `tasks`: Array of all tasks
  - `newTask`: Text input for adding a new task
- **API Integration**: Uses Axios to communicate with the backend
- **Components**:
  - Task input form
  - Task list with toggle and delete functionality

## Key Technologies Used

1. **Frontend**:
   - React.js: JavaScript library for building user interfaces
   - Axios: Promise-based HTTP client for making API requests
   - CSS: For styling components

2. **Backend**:
   - Node.js: JavaScript runtime for server-side code
   - Express: Web framework for Node.js
   - CORS: Cross-Origin Resource Sharing middleware
   - Body Parser: Middleware to parse incoming request bodies

## Future Enhancements

1. **Data Persistence**: Replace in-memory storage with a database (MongoDB, PostgreSQL)
2. **User Authentication**: Add login/registration functionality
3. **Task Categories**: Allow grouping tasks by categories or projects
4. **Task Due Dates**: Add scheduling capabilities
5. **Responsive Design**: Improve mobile experience

## Troubleshooting

### Common Issues:

1. **"Cannot connect to server" error**:
   - Ensure the server is running on port 5000
   - Check for any console errors in the terminal running the server

2. **CORS errors**:
   - Verify that the CORS middleware is properly configured in server.js

3. **Changes not reflecting**:
   - For React: Make sure to save files and check the console for errors
   - For Node.js: You may need to restart the server to see changes

## Learning Resources

1. **React.js**:
   - [React Documentation](https://reactjs.org/docs/getting-started.html)
   - [React Hooks API Reference](https://reactjs.org/docs/hooks-reference.html)

2. **Node.js and Express**:
   - [Node.js Documentation](https://nodejs.org/en/docs/)
   - [Express Documentation](https://expressjs.com/)

3. **JavaScript**:
   - [MDN Web Docs](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
