const express = require('express');
const app = express();
const port = 3000;

let users = [];

// parse JSON body data
app.use(express.json());

// GET  --- Get all users
app.get('/users', (req, res) => {
  res.json(users);
});

// POST --- Create a new user
app.post('/users', (req, res) => {
  const newUser = req.body;

  if (!newUser.name || !newUser.email) {
    return res.status(400).json({ message: 'Name and email are required.' });
  }

  users.push(newUser);
  res.status(201).json({ message: 'User created successfully.', user: newUser });
});

// GET --- Get a user by ID
app.get('/users/:id', (req, res) => {
  const { id } = req.params;
  const user = users.find(user => user.id === id);

  if (!user) {
    return res.status(404).json({ message: 'User not found.' });
  }

  res.json(user);
});

// PUT --- Update a user by ID
app.put('/users/:id', (req, res) => {
  const { id } = req.params;
  const updatedUser = req.body;

  const userIndex = users.findIndex(user => user.id === id);

  if (userIndex === -1) {
    return res.status(404).json({ message: 'User not found.' });
  }

  users[userIndex] = updatedUser;
  res.json({ message: 'User updated successfully.', user: updatedUser });
});

// DELETE --- Delete a user by ID
app.delete('/users/:id', (req, res) => {
  const { id } = req.params;
  const userIndex = users.findIndex(user => user.id === id);

  if (userIndex === -1) {
    return res.status(404).json({ message: 'User not found.' });
  }

  users.splice(userIndex, 1);
  res.json({ message: 'User deleted successfully.' });
});

// Start the server
app.listen(port, () => {
  console.log(`Server listening on port ${port}`);
});
