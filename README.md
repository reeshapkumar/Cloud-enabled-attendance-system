# Cloud Enabled Attendance System

Creating a Cloud-Enabled Attendance System involves developing an application that allows users to mark attendance, view attendance records, and manage user accounts. For this project, we’ll use the MERN stack (MongoDB, Express.js, React, Node.js) and deploy it to a cloud platform like Heroku or AWS.

Here’s a step-by-step guide, including code snippets.

**Project Structure**

```go
cloud-attendance-system/
├── backend/
│   ├── Dockerfile
│   ├── package.json
│   ├── server.js
│   ├── models/
│   │   └── Attendance.js
│   ├── routes/
│   │   └── attendance.js
│   └── .env
```
```
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   ├── src/
│   │   ├── components/
│   │   │   ├── AttendanceForm.js
│   │   │   └── AttendanceList.js
│   │   ├── App.js
│   │   └── index.js
└── docker-compose.yml
```

**Step 1: Set Up the Backend**

**A. Create the Backend Directory**

**Create the backend directory and navigate to it:**

```bash
mkdir backend
cd backend
```

**Initialize a new Node.js project:**

```bash
npm init -y
```

**Install necessary dependencies:**

```bash
npm install express mongoose dotenv cors body-parser
```

**Create the server.js file:**

```javascript
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const bodyParser = require('body-parser');
require('dotenv').config();
const app = express();
const PORT = process.env.PORT || 5000;
app.use(cors());
app.use(bodyParser.json());
mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log('MongoDB connected'))
  .catch(err => console.error('MongoDB connection error:', err));
const attendanceRoutes = require('./routes/attendance');
app.use('/api/attendance', attendanceRoutes);
app.listen(PORT, () => {
    console.log(`Server is running on port ${PORT}`);
});
```

**Create the MongoDB model for attendance:**

```javascript
const mongoose = require('mongoose');

const attendanceSchema = new mongoose.Schema({
    studentName: { type: String, required: true },
    date: { type: Date, default: Date.now },
    status: { type: String, enum: ['Present', 'Absent'], required: true },
});

module.exports = mongoose.model('Attendance', attendanceSchema);
```

**Create the attendance routes:**

```javascript
const express = require('express');
const router = express.Router();
const Attendance = require('../models/Attendance');
router.get('/', async (req, res) => {
    try {
        const attendanceRecords = await Attendance.find();
        res.json(attendanceRecords);
    } catch (error) {
        res.status(500).json({ message: error.message });
    }
});

router.post('/', async (req, res) => {
    const attendance = new Attendance({
        studentName: req.body.studentName,
        status: req.body.status,
    });
    
    try {
        const newAttendance = await attendance.save();
        res.status(201).json(newAttendance);
    } catch (error) {
        res.status(400).json({ message: error.message });
    }
});

module.exports = router;
```

**Create a .env file to store environment variables:**

```bash
MONGO_URI=mongodb:
```

**B. Create the Dockerfile for Backend**

```dockerfile
FROM node:14

WORKDIR /usr/src/app

COPY package*.json ./
RUN npm install

COPY . .

EXPOSE 5000

CMD ["node", "server.js"]
```

**Step 2: Set Up the Frontend**

**A. Create the Frontend Directory**
Create the frontend directory and navigate to it:

```bash
mkdir ../frontend
cd ../frontend
```

**Create a new React app:**

```bash
npx create-react-app .
```

**Install Axios for API calls:**

```bash
npm install axios
```

**Create AttendanceForm and AttendanceList components:**

```javascript
import React, { useState } from 'react';
import axios from 'axios';

const AttendanceForm = () => {
    const [studentName, setStudentName] = useState('');
    const [status, setStatus] = useState('Present');

    const handleSubmit = async (e) => {
        e.preventDefault();
        await axios.post('/api/attendance', { studentName, status });
        setStudentName('');
    };

    return (
        <form onSubmit={handleSubmit}>
            <input
                type="text"
                placeholder="Student Name"
                value={studentName}
                onChange={(e) => setStudentName(e.target.value)}
                required
            />
            <select value={status} onChange={(e) => setStatus(e.target.value)}>
                <option value="Present">Present</option>
                <option value="Absent">Absent</option>
            </select>
            <button type="submit">Mark Attendance</button>
        </form>
    );
};

export default AttendanceForm;
```

```javascript
Copy code
import React, { useEffect, useState } from 'react';
import axios from 'axios';

const AttendanceList = () => {
    const [records, setRecords] = useState([]);

    useEffect(() => {
        const fetchRecords = async () => {
            const response = await axios.get('/api/attendance');
            setRecords(response.data);
        };

        fetchRecords();
    }, []);

    return (
        <div>
            <h2>Attendance Records</h2>
            <ul>
                {records.map((record) => (
                    <li key={record._id}>
                        {record.studentName} - {record.status} on {new Date(record.date).toLocaleDateString()}
                    </li>
                ))}
            </ul>
        </div>
    );
};

export default AttendanceList;
```

**Modify the App.js file to include the form and list components:**

```javascript
import React from 'react';
import AttendanceForm from './components/AttendanceForm';
import AttendanceList from './components/AttendanceList';

function App() {
    return (
        <div>
            <h1>Cloud-Enabled Attendance System</h1>
            <AttendanceForm />
            <AttendanceList />
        </div>
    );
}

export default App;
```

**B. Create the Dockerfile for Frontend**

```dockerfile
FROM node:14 as build

WORKDIR /app

COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=build /app/build /usr/share/nginx/html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

**Step 3: Set Up Docker Compose**

**A. Create the docker-compose.yml file**

```yaml
version: '3.8'

services:
  backend:
    build:
      context: ./backend
    ports:
      - "5000:5000"
    env_file:
      - ./backend/.env
    depends_on:
      - mongo

  frontend:
    build:
      context: ./frontend
    ports:
      - "3000:80"

  mongo:
    image: mongo
    ports:
      - "27017:27017"
    volumes:
      - mongo-data:/data/db

volumes:
  mongo-data:
```
  
**Step 4: Running the Application**

**Navigate to the root of your project directory (where docker-compose.yml is located):**

```bash
cd cloud-attendance-system
Run Docker Compose:
``

```bash
docker-compose up --build
```

**Access the application:**

```Frontend: http://localhost:3000
```

**Step 5: Testing the Application**

Open your browser and navigate to http://localhost:3000.
Use the form to mark attendance and view the list of attendance records.

**Step 6: Deploying to the Cloud**

To deploy your application to the cloud, you can use services like Heroku, AWS Elastic Beanstalk, or DigitalOcean. Below are simplified steps for deploying to Heroku.

**Install the Heroku CLI and log in:**

```bash
heroku login
```

**Create a new Heroku app:**

```bash
heroku create my-attendance-system
```

**Push the code to Heroku (ensure your Dockerfile is configured correctly):**

```bash
heroku container:push web
heroku container:release web
```

**Set environment variables on Heroku:**

```bash
heroku config:set MONGO_URI=<your-mongo-uri>
```

**Open your app:**

```bash
heroku open
```

**Conclusion**

You now have a Cloud-Enabled Attendance System built using the MERN stack, with the ability to mark and view attendance records. This application can be further enhanced by adding user authentication, generating attendance reports, or integrating notifications. Let me know if you have any questions or need further assistance!
