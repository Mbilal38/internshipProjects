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

//User Authentication with JWT  
require('dotenv').config
const express = require('express')
const app = express()

const jwt = require('jsonwebtoken')
app.use(express.json())

const posts =[
  {
    username: 'james',
  title: 'Post 1'
  },
  {
    username: 'bart',
  title: 'Post 2'
  }
  
]

app.get('/posts', authenticateToken, (req, res) => {
  res.json(posts.filter(post => post.username === req.user.name))

})

app.post('/login', (req,res) => {
  // Authenticate User
  const username = req.body.username
  const user = { name: username}

  const accessToken = jwt.sign(user, process.env.ACCESS_TOKEN_SECRET)
  res.json({accessToken : accessToken})
})

function authenticateToken(req, res, nex){
  const authHeader = req.headers['authorization']
  const token = authHeader && authHeader.split('')[1]
  if (token == null) 
    return res.sendStatus(401)
  jwt.verify(token, process.env.ACCESS_TOKEN_SECRET, (err, user) => {
    if (err)
      return res.sendStatus(403)
    req.user = user
    next()
  })
}
app.listen(3000)

// FOR AUTHENTICATION

require('dotenv').config()

const express = require('express')
const app = express()
const jwt = require('jsonwebtoken')

app.use(express.json())

let refreshTokens = []

app.post('/token', (req, res) => {
  const refreshToken = req.body.token
  if (refreshToken == null) return res.sendStatus(401)
  if (!refreshTokens.includes(refreshToken)) return res.sendStatus(403)
  jwt.verify(refreshToken, process.env.REFRESH_TOKEN_SECRET, (err, user) => {
    if (err) return res.sendStatus(403)
    const accessToken = generateAccessToken({ name: user.name })
    res.json({ accessToken: accessToken })
  })
})

app.delete('/logout', (req, res) => {
  refreshTokens = refreshTokens.filter(token => token !== req.body.token)
  res.sendStatus(204)
})

app.post('/login', (req, res) => {
  // Authenticate User

  const username = req.body.username
  const user = { name: username }

  const accessToken = generateAccessToken(user)
  const refreshToken = jwt.sign(user, process.env.REFRESH_TOKEN_SECRET)
  refreshTokens.push(refreshToken)
  res.json({ accessToken: accessToken, refreshToken: refreshToken })
})

function generateAccessToken(user) {
  return jwt.sign(user, process.env.ACCESS_TOKEN_SECRET, { expiresIn: '15s' })
}

app.listen(4000)
