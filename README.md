# Project-1-Awesome-Web-App
A full-stack web application built with React and Node.js, featuring user authentication, real-time data updates, and responsive design. 
Let's build the source code for a full-stack web application called **"Awesome Web App"**, using **React** for the frontend and **Node.js** with **Express** for the backend. We'll implement:

- **User Authentication**: Registration and login using JWT (JSON Web Tokens).
- **Real-Time Data Updates**: Using **Socket.io** for real-time communication.
- **Responsive Design**: Using CSS Flexbox/Grid or a framework like **Tailwind CSS** for responsiveness.

### **Project Structure Overview**

- **Frontend (React)**:
  - **Login/Registration** forms.
  - **Dashboard** to display real-time data.
- **Backend (Node.js/Express)**:
  - User registration and login endpoints.
  - JWT token-based authentication.
  - Real-time data updates with **Socket.io**.

---

### **Step 1: Backend Setup (Node.js + Express + JWT + Socket.io)**

1. **Create the Backend Project Folder**:
   ```bash
   mkdir awesome-web-app
   cd awesome-web-app
   mkdir server
   cd server
   npm init -y
   npm install express jsonwebtoken bcryptjs socket.io cors dotenv
   ```

2. **Create Backend File Structure**:
   - `server.js`: Main entry point for the Node.js server.
   - `controllers/authController.js`: Handles user registration and login.
   - `routes/auth.js`: Defines authentication routes.
   - `socket/socket.js`: Handles Socket.io real-time communication.
   - `.env`: Store sensitive environment variables (JWT secret, etc.).

3. **Backend Code**:

   **server.js** (Main server entry point):
   ```js
   const express = require('express');
   const http = require('http');
   const socketIo = require('socket.io');
   const cors = require('cors');
   const dotenv = require('dotenv');
   const authRoutes = require('./routes/auth');

   dotenv.config();

   const app = express();
   const server = http.createServer(app);
   const io = socketIo(server);

   // Middleware
   app.use(cors());
   app.use(express.json());

   // Routes
   app.use('/api/auth', authRoutes);

   // Real-time data via Socket.io
   io.on('connection', (socket) => {
     console.log('User connected');
     socket.emit('message', { text: 'Welcome to the Awesome Web App!' });

     socket.on('disconnect', () => {
       console.log('User disconnected');
     });
   });

   // Start server
   const port = process.env.PORT || 5000;
   server.listen(port, () => {
     console.log(`Server running on port ${port}`);
   });
   ```

   **controllers/authController.js** (Handle user authentication):
   ```js
   const jwt = require('jsonwebtoken');
   const bcrypt = require('bcryptjs');

   let users = []; // In-memory user storage for demo purposes

   const register = (req, res) => {
     const { username, email, password } = req.body;
     const existingUser = users.find(user => user.email === email);

     if (existingUser) {
       return res.status(400).json({ message: 'User already exists' });
     }

     const hashedPassword = bcrypt.hashSync(password, 10);
     const newUser = { username, email, password: hashedPassword };
     users.push(newUser);

     const token = jwt.sign({ email: newUser.email }, process.env.JWT_SECRET, { expiresIn: '1h' });
     res.status(201).json({ token });
   };

   const login = (req, res) => {
     const { email, password } = req.body;
     const user = users.find(user => user.email === email);

     if (!user || !bcrypt.compareSync(password, user.password)) {
       return res.status(400).json({ message: 'Invalid credentials' });
     }

     const token = jwt.sign({ email: user.email }, process.env.JWT_SECRET, { expiresIn: '1h' });
     res.status(200).json({ token });
   };

   module.exports = { register, login };
   ```

   **routes/auth.js** (Authentication routes):
   ```js
   const express = require('express');
   const { register, login } = require('../controllers/authController');
   const router = express.Router();

   router.post('/register', register);
   router.post('/login', login);

   module.exports = router;
   ```

   **.env** (Store sensitive information):
   ```
   JWT_SECRET=mysecretkey
   ```

---

### **Step 2: Frontend Setup (React + Socket.io-client)**

1. **Create the Frontend Project Folder**:
   ```bash
   cd ..
   npx create-react-app client
   cd client
   npm install axios react-router-dom socket.io-client
   ```

2. **Frontend File Structure**:
   - **components**: Contains components like `Login.js`, `Register.js`, and `Dashboard.js`.
   - **services**: Contains functions to interact with the backend API.
   - **App.js**: Main application entry point.

3. **Frontend Code**:

   **src/App.js**:
   ```js
   import React from 'react';
   import { BrowserRouter as Router, Route, Switch } from 'react-router-dom';
   import Login from './components/Login';
   import Register from './components/Register';
   import Dashboard from './components/Dashboard';

   function App() {
     return (
       <Router>
         <div>
           <Switch>
             <Route path="/login" component={Login} />
             <Route path="/register" component={Register} />
             <Route path="/dashboard" component={Dashboard} />
           </Switch>
         </div>
       </Router>
     );
   }

   export default App;
   ```

   **src/components/Login.js** (Login form):
   ```js
   import React, { useState } from 'react';
   import axios from 'axios';

   function Login() {
     const [email, setEmail] = useState('');
     const [password, setPassword] = useState('');

     const handleLogin = async (e) => {
       e.preventDefault();

       try {
         const response = await axios.post('http://localhost:5000/api/auth/login', { email, password });
         localStorage.setItem('token', response.data.token);
         window.location.href = '/dashboard';
       } catch (error) {
         alert('Invalid login credentials');
       }
     };

     return (
       <div>
         <h2>Login</h2>
         <form onSubmit={handleLogin}>
           <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email" required />
           <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="Password" required />
           <button type="submit">Login</button>
         </form>
       </div>
     );
   }

   export default Login;
   ```

   **src/components/Register.js** (Registration form):
   ```js
   import React, { useState } from 'react';
   import axios from 'axios';

   function Register() {
     const [username, setUsername] = useState('');
     const [email, setEmail] = useState('');
     const [password, setPassword] = useState('');

     const handleRegister = async (e) => {
       e.preventDefault();

       try {
         const response = await axios.post('http://localhost:5000/api/auth/register', { username, email, password });
         localStorage.setItem('token', response.data.token);
         window.location.href = '/dashboard';
       } catch (error) {
         alert('Registration failed');
       }
     };

     return (
       <div>
         <h2>Register</h2>
         <form onSubmit={handleRegister}>
           <input type="text" value={username} onChange={(e) => setUsername(e.target.value)} placeholder="Username" required />
           <input type="email" value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email" required />
           <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} placeholder="Password" required />
           <button type="submit">Register</button>
         </form>
       </div>
     );
   }

   export default Register;
   ```

   **src/components/Dashboard.js** (Real-time updates with Socket.io):
   ```js
   import React, { useEffect, useState } from 'react';
   import io from 'socket.io-client';

   function Dashboard() {
     const [message, setMessage] = useState('');

     useEffect(() => {
       const socket = io('http://localhost:5000');

       socket.on('message', (data) => {
         setMessage(data.text);
       });

       return () => {
         socket.disconnect();
       };
     }, []);

     return (
       <div>
         <h2>Dashboard</h2>
         <p>{message}</p>
       </div>
     );
   }

   export default Dashboard;
   ```

---

### **Step 3: Running the Application**

1. **Backend**:
   - In the `server` folder, start the backend server:
   ```bash
   node server.js
   ```

2. **Frontend**:
   - In the `client` folder, start the React development server:
   ```bash
   npm start
   ```

You can now visit your web app at `http://localhost:3000` and interact with the login and registration forms, along with receiving real-time updates on the dashboard.

---

### **Responsive Design**

To make the app responsive, you can either use CSS Media Queries or a utility-first CSS framework like **Tailwind CSS**.

- For a **quick responsive design** using **Tailwind CSS**, install it in your React app:
   ```bash
   npm install tailwindcss
   ```

- Then add the `tailwind.config.js` and configure it as per the [Tailwind CSS Docs](https://tailwindcss.com/docs/installation).
