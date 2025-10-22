"scripts": {
  "start": "nodemon server.js"
}
// backend/server.js
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const userRoutes = require('./routes/userRoutes');

const app = express();
app.use(cors());
app.use(express.json());

// Connect MongoDB
mongoose.connect('mongodb://127.0.0.1:27017/merncrud', {
  useNewUrlParser: true,
  useUnifiedTopology: true,
})
.then(() => console.log(" MongoDB Connected"))
.catch(err => console.log(" DB Connection Error:", err));

// Routes
app.use('/api/users', userRoutes);

// Start Server
app.listen(5000, () => console.log(' Server running on port 5000'));

// backend/models/userModel.js
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  age: { type: Number, required: true }
}, { timestamps: true });

module.exports = mongoose.model('User', userSchema);
// backend/routes/userRoutes.js
const express = require('express');
const router = express.Router();
const User = require('../models/userModel');

// CREATE
router.post('/', async (req, res) => {
  try {
    const user = new User(req.body);
    const savedUser = await user.save();
    res.json(savedUser);
  } catch (err) {
    res.status(400).json({ message: err.message });
  }
});

// READ (All)
router.get('/', async (req, res) => {
  const users = await User.find();
  res.json(users);
});

// READ (Single)
router.get('/:id', async (req, res) => {
  const user = await User.findById(req.params.id);
  if (!user) return res.status(404).json({ message: "User not found" });
  res.json(user);
});

// UPDATE
router.put('/:id', async (req, res) => {
  const updatedUser = await User.findByIdAndUpdate(req.params.id, req.body, { new: true });
  res.json(updatedUser);
});

// DELETE
router.delete('/:id', async (req, res) => {
  await User.findByIdAndDelete(req.params.id);
  res.json({ message: "User deleted" });
});

module.exports = router;
// frontend/src/services/userService.js
import axios from 'axios';
const API_URL = 'http://localhost:5000/api/users/';

export const getUsers = () => axios.get(API_URL);
export const createUser = (data) => axios.post(API_URL, data);
export const updateUser = (id, data) => axios.put(API_URL + id, data);
export const deleteUser = (id) => axios.delete(API_URL + id);
// frontend/src/components/UserList.js
import React, { useEffect, useState } from 'react';
import { getUsers, createUser, updateUser, deleteUser } from '../services/userService';

const UserList = () => {
  const [users, setUsers] = useState([]);
  const [form, setForm] = useState({ name: '', email: '', age: '' });
  const [editing, setEditing] = useState(null);

  const fetchUsers = async () => {
    const res = await getUsers();
    setUsers(res.data);
  };

  useEffect(() => { fetchUsers(); }, []);

  const handleSubmit = async (e) => {
    e.preventDefault();
    if (editing) {
      await updateUser(editing, form);
      setEditing(null);
    } else {
      await createUser(form);
    }
    setForm({ name: '', email: '', age: '' });
    fetchUsers();
  };

  const handleEdit = (user) => {
    setForm(user);
    setEditing(user._id);
  };

  const handleDelete = async (id) => {
    await deleteUser(id);
    fetchUsers();
  };

  return (
    <div className="container">
      <h1>MongoDB CRUD with Mongoose</h1>

      <form onSubmit={handleSubmit}>
        <input placeholder="Name" value={form.name} onChange={(e)=>setForm({...form, name: e.target.value})}/>
        <input placeholder="Email" value={form.email} onChange={(e)=>setForm({...form, email: e.target.value})}/>
        <input placeholder="Age" type="number" value={form.age} onChange={(e)=>setForm({...form, age: e.target.value})}/>
        <button type="submit">{editing ? "Update" : "Add"} User</button>
      </form>

      <ul>
        {users.map(u => (
          <li key={u._id}>
            {u.name} ({u.email}) - {u.age} years
            <button onClick={() => handleEdit(u)}>Edit</button>
            <button onClick={() => handleDelete(u._id)}>Delete</button>
          </li>
        ))}
      </ul>
    </div>
  );
};

export default UserList;
// frontend/src/App.js
import React from 'react';
import UserList from './components/UserList';

function App() {
  return (
    <div>
      <UserList />
    </div>
  );
}

export default App;
