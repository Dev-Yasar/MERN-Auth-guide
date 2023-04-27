# Guide to MERN Stack Auth

## Tech Stack

**Client:** React,

**Server:** Node, Express

**Db:** MangoDb

## Backend

install the packages

```bash
npm install express mongoose cors jsonwebtoken bcryptjs
```
### Create a mongoose model file
User.js
```js
 const mongoose = require('mongoose')

const UserSchema = new mongoose.Schema(
	{
		name: { type: String, required: true },
		email: { type: String, required: true, unique: true },
		password: { type: String, required: true },
	},
	{ collection: 'User' }
)
const User = mongoose.model("User", UserSchema);

module.exports = User;


```
### Main Server file
 server.js

```js 
// reference for packages
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const jwt = require('jsonwebtoken')
const bcrypt = require('bcryptjs')

// using
const app = express();	
app.use(express.json());
app.use(cors());

//Connect MangoDb
const url ="mangodb url";

mongoose.connect(url, {
	useNewUrlParser: true, 
	useUnifiedTopology: true 
}).then(() => console.log("Connected to MongoDB"))
.catch(console.error);
app.listen(5000); // port num is your wish 


// Models
const User = require('./models/Users');


// route for register a new user
app.post('/api/register', async (req, res) => {
	console.log(req.body)
	try {
		const Password = await bcrypt.hash(req.body.password, 10)
		console.log(Password)
		await User.create({
			name: req.body.name,
			email: req.body.email,
			password: Password,
		   })
		res.json({ status: 'ok' })
	} catch (err) {
		res.json({ status: 'error', error: 'Duplicate email' })
	}
})


// route fro Previous user login
app.post('/api/login', async (req, res) => {
	const user = await User.findOne({
		email: req.body.email,
	})
	if (!user) {
		return { status: 'error', error: 'Invalid login' }
	}
	const isPasswordValid = await bcrypt.compare(
		req.body.password,
		user.password
	)
	if (isPasswordValid) {
		const token = jwt.sign(
			{
				name: user.name,
				email: user.email,
			},
			'secretcode' // put your secret code
		)
		return res.json({ status: 'ok', user: token })
	} else {
		return res.json({ status: 'error', user: false })
	}
})

```
## Frontend
install the packages

```bash
 npm install jwt-decode  react-router-dom
```
## Register.jsx

```javascript I'm A tab
import { useState } from 'react'
import { useNavigate } from 'react-router-dom';


function Register() {
	const navigate = useNavigate();
 
	const [name, setName] = useState('')
	const [email, setEmail] = useState('')
	const [password, setPassword] = useState('')

	async function registerUser(event) {
		event.preventDefault()

		const response = await fetch('http://localhost:5000/api/register', {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json',
			},
			body: JSON.stringify({
				name,
				email,
				password,
			}),
		})

		const data = await response.json()

		if (data.status === 'ok') {
			navigate('/admin');
			
		}
	}

	return (
        <div>


<div className='MAINauth' >
<div class="form-container">
    <p>Register new admin </p>
<form class="form"onSubmit={registerUser}>
<label>Name</label>
    <input type="text" class="input" 	onChange={(e) => setName(e.target.value)} placeholder="Enter your name"/>
    <label>Email</label>
    <input type="text" class="input" 	onChange={(e) => setEmail(e.target.value)} placeholder="Enter yout email"/>
    <label>Password</label>
    <input type="password" class="input"  onChange={(e) => setPassword(e.target.value)} placeholder="Password"/> 
    <button className='AuthBtn'>Login</button>
</form>
</div>
</div>

     </div>
	)
}

export default Register
```
## Login.jsx

```javascript I'm tab B

import { useState } from 'react'

function Login() {
	const [email, setEmail] = useState('')
	const [password, setPassword] = useState('')

	async function loginUser(event) {
		event.preventDefault()

		const response = await fetch('http://localhost:5000/api/login', {
			method: 'POST',
			headers: {
				'Content-Type': 'application/json',
			},
			body: JSON.stringify({
				email,
				password,
			}),
		})

		const data = await response.json()

		if (data.user) {
			localStorage.setItem('token', data.user)
			alert('Login successful')
			window.location.href = '/dashboard'
		} else {
			alert('Please check your username and password')
		}
	}

	return (
		<div>

       
<div className='MAINauth' >

<div class="form-container">
    <p>Welcome,Admin</p>
<form class="form" onSubmit={loginUser}>
    <label>Email</label>
    <input type="text" class="input" 	onChange={(e) => setEmail(e.target.value)} placeholder="Enter yout email"/>
    <label>Password</label>
    <input type="password" class="input"  onChange={(e) => setPassword(e.target.value)} placeholder="Password"/> 
    <button className='AuthBtn'>Login</button>
</form>
</div>
</div>

		</div>
	)
}

export default Login
```

