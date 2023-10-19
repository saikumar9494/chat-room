#Java scritp

const express = require('express');
const http = require('http');
const socketIO = require('socket.io');
const mongoose = require('mongoose');

const app = express();
const server = http.createServer(app);
const io = socketIO(server);
const PORT = process.env.PORT || 3000;

// MongoDB connection
const mongoURI = 'mongodb://localhost:27017/yourDatabase';
mongoose.connect(mongoURI, {
 useNewUrlParser: true,
 useUnifiedTopology: true,
});

const connection = mongoose.connection;
connection.on('connected', () => {
 console.log('Connected to MongoDB');
});

connection.on('error', (err) => {
 console.error('MongoDB connection error:', err);
});

// Define Mongoose schemas and models
const messageSchema = new mongoose.Schema({
 username: String,
 message: String,
});

const Message = mongoose.model('Message', messageSchema);

const taskSchema = new mongoose.Schema({
 title: String,
 description: String,
 completed: Boolean,
});

const Task = mongoose.model('Task', taskSchema);

// Socket.IO Connection
io.on('connection', (socket) => {
 console.log('User connected');

 socket.on('disconnect', () => {
    console.log('User disconnected');
 });

 socket.on('chat message', async (msg) => {
    const message = new Message(msg);
    await message.save();
    io.emit('chat message', message);
 });

 socket.on('createTask', async (task) => {
    const newTask = new Task(task);
    await newTask.save();
    io.emit('task created', newTask);
 });
});

// Define your routes and middleware here

app.
