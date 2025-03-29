// Backend: Express + MongoDB
const express = require('express');
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const cors = require('cors');
const app = express();
const PORT = 5000;

app.use(express.json());
app.use(cors());

mongoose.connect('mongodb://localhost:27017/userPoints', { useNewUrlParser: true, useUnifiedTopology: true });

const UserSchema = new mongoose.Schema({
    username: String,
    password: String,
    description: String,
    vip: Boolean,
    points: Number
});

const User = mongoose.model('User', UserSchema);

app.post('/register', async (req, res) => {
    const { username, password, description, vip } = req.body;
    const hashedPassword = await bcrypt.hash(password, 10);
    const user = new User({ username, password: hashedPassword, description, vip, points: 0 });
    await user.save();
    res.json({ message: 'User registered' });
});

app.post('/login', async (req, res) => {
    const { username, password } = req.body;
    const user = await User.findOne({ username });
    if (!user || !(await bcrypt.compare(password, user.password))) {
        return res.status(401).json({ message: 'Invalid credentials' });
    }
    const token = jwt.sign({ username }, 'secret', { expiresIn: '1h' });
    res.json({ token, user });
});

app.get('/users', async (req, res) => {
    const users = await User.find({}, 'username description vip');
    res.json(users);
});

app.post('/sendPoints', async (req, res) => {
    const { sender, receiver, amount, giftWrap } = req.body;
    const senderUser = await User.findOne({ username: sender });
    const receiverUser = await User.findOne({ username: receiver });
    if (!senderUser || !receiverUser || senderUser.points < amount) {
        return res.status(400).json({ message: 'Transaction failed' });
    }
    senderUser.points -= amount;
    receiverUser.points += giftWrap ? 0 : amount;
    await senderUser.save();
    await receiverUser.save();
    res.json({ message: 'Points sent successfully' });
});

app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
