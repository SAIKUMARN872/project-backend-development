// Importing required modules
const express = require('express');
const mongoose = require('mongoose');
const bodyParser = require('body-parser');

// Initialize Express app
const app = express();
const port = 3000; // Port to listen on

// Middleware
app.use(bodyParser.json()); // To parse incoming JSON requests

// Connect to MongoDB
mongoose.connect('mongodb://localhost:27017/userdb', { useNewUrlParser: true, useUnifiedTopology: true })
    .then(() => console.log('Connected to MongoDB'))
    .catch(err => {
        console.error('MongoDB connection failed:', err.message);
        process.exit(1); // Exit process if unable to connect to MongoDB
    });

// User Schema
const userSchema = new mongoose.Schema({
    name: {
        type: String,
        required: true
    },
    email: {
        type: String,
        required: true,
        unique: true
    },
    age: {
        type: Number,
        required: true,
        min: 0
    }
});

// User Model
const User = mongoose.model('User', userSchema);

// Route to create a new user
app.post('/users', async (req, res) => {
    try {
        const { name, email, age } = req.body;

        // Validate input data
        if (!name || !email || !age) {
            return res.status(400).json({ error: 'Name, email, and age are required.' });
        }

        // Create and save new user
        const user = new User({ name, email, age });
        await user.save();
        res.status(201).json(user);
    } catch (error) {
        if (error.code === 11000) {
            return res.status(400).json({ error: 'Email already exists.' });
        }
        res.status(500).json({ error: 'Failed to create user.', details: error.message });
    }
});

// Route to get all users
app.get('/users', async (req, res) => {
    try {
        const users = await User.find();
        res.status(200).json(users);
    } catch (error) {
        res.status(500).json({ error: 'Failed to fetch users.', details: error.message });
    }
});

// Route to update a user by ID
app.put('/users/:id', async (req, res) => {
    try {
        const { id } = req.params;
        const { name, email, age } = req.body;

        // Find user by ID and update
        const user = await User.findByIdAndUpdate(id, { name, email, age }, { new: true, runValidators: true });

        if (!user) {
            return res.status(404).json({ error: 'User not found.' });
        }

        res.status(200).json(user);
    } catch (error) {
        res.status(400).json({ error: 'Failed to update user.', details: error.message });
    }
});

// Route to delete a user by ID
app.delete('/users/:id', async (req, res) => {
    try {
        const { id } = req.params;

        // Find and delete user by ID
        const user = await User.findByIdAndDelete(id);

        if (!user) {
            return res.status(404).json({ error: 'User not found.' });
        }

        res.status(200).json({ message: 'User deleted successfully.' });
    } catch (error) {
        res.status(500).json({ error: 'Failed to delete user.', details: error.message });
    }
});

// Start the server
app.listen(port, () => {
    console.log(Server is running on http://localhost:${port});
});
