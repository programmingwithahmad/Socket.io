# Socket.io
Socket.io enables real-time bidirectional event-based communication. It is used in Chat applications, Forex Trading apps and gaming.

- npm Package for server side
```
npm install socket.io     
```
- npm Package for client side
```
npm install socket.io-client  
```
## 1 - Basic setup 
- server side
```
import express from 'express';
import colors from 'colors';
import dotenv from 'dotenv';
import morgan from 'morgan';
import cors from 'cors';
// import connectDB from './config/db.js';
// import authRoute  from './routes/authRoute.js'
// import categoryRoutes from './routes/categoryRoutes.js';
// import productRoutes from './routes/productRoutes.js';
// import assistantRoute from './routes/assistantRoute.js';
import bodyParser from 'body-parser';
import {createServer} from 'http';
import { Server } from 'socket.io';

//configure env
dotenv.config();   

//database config
// connectDB();

//rest object
const app = express();
const server = createServer(app)
const io = new Server(server, {
    cors: {
      origin: "http://localhost:5173", // allow frontend origin
      methods: ["GET", "POST"]
    }
  })
 
//middleware
app.use(cors({
    origin: "http://localhost:5173", // your frontend port
    methods: ["GET", "POST"]
  }))
app.use(express.json());
app.use(morgan('dev'));
app.use(bodyParser.json());


// // API Routes
// app.use('/api/v1/auth', authRoute)
// app.use('/api/v1/category', categoryRoutes)
// app.use('/api/v1/product', productRoutes)
// app.use('/cx', assistantRoute)

 
 rest api            
 app.get('/', (req, res) => {
     res.send('Welcome to ChatApp');
 })  


//PORT
const PORT = process.env.PORT || 3000; 


// Start server
server.listen(PORT ,() => {
    console.log(`Server is running on port ${PORT}`.bgYellow?.white);
})



// Socket.IO
io.on('connection', (socket) => {
    console.log('Client connected');


    socket.on('disconnect', () => {
        
        console.log('Client disconnected');

    });
})
```
- client side
```
import io from 'socket.io-client'
import { useEffect } from 'react'


const App = () => {

     useEffect(() => {
    
    const socket = io('http://localhost:3000');


    return () => {
      socket.disconnect()
    }
  }, [])

  return (
    <>
         <h3>Hello</h3>    
    </>
  )
}

export default App
```
## 2 - Event Handling in Socket.io
- server side
```
io.on('connection', (socket) => {
    console.log('Client connected');

    
    //reserved event send
    socket.send('This is a reserved event')

    //reserved event receive
    socket.on('message', (data) => {
        console.log(data)
    })
    
    //custom event send
    socket.emit('ahmad', 'This is a custom event')

    // custom event receive
    socket.on('ahmad', (data) => {
    console.log(data)
   }) 

    socket.on('disconnect', () => {
        
        console.log('Client disconnected');

    });
})

```
- client side
```
  useEffect(() => {
    
  const socket = io('http://localhost:3000');

    //reserved event received
    socket.on('message', (data) => {
      console.log(data);
    })

    // reserved event sent
    socket.send('This is a reserved event')

    // custom event received
    socket.on('ahmad', (data) => {
      console.log(data);
    })

    // custom event sent
    socket.emit('ahmad', 'This is a custom event')

    return () => {
      socket.disconnect()
    }
  }, [])
```
## 3 - Broadcasting in Socket.io
Broadcasting means sending a message to all connected clients:
- To broadcast an event to all the clients: **io.sockets.emit**
- To broadcast an event to all the clients but not one that make new connection: **socket.broadcast.emit**
- server side
```
var count = 0;

// Socket.IO
io.on('connection', (socket) => {
    console.log('Client connected');

 count++;

 socket.emit('new', 'Welcome to join here.')

// Show all members except new one
 socket.broadcast.emit('new', `${count} users connected`)

//Below line show all members even new members 
// io.sockets.emit('new', 'Welcome to All of you')


    socket.on('disconnect', () => {
        
        console.log('Client disconnected');

        count--;
        
        socket.broadcast.emit('new', `${count} users connected`)

    });
})
```
- client side
```
import { useState } from 'react'
import io from 'socket.io-client'


const App = () => {

  const [first, setfirst] = useState('')

  useEffect(() => {
    
  const socket = io('http://localhost:3000');

  socket.on('new', (data) => {
    setfirst(data);
  })

    return () => {
      socket.disconnect()
    }
  }, [])

  return (
    <>
        <h3>{first}</h3>
    </>
  )
}

export default App
```
## 4 - Namespaces in Socket.io
Socket.io allows you to "namespace" your sockets, which essentially means assigning different endpoints.
### Default Namespaces:
The default root namespace is '/' .
### Custom Namespaces:
- server side
```
// Socket.IO
const cnsp = io.of('/multan')
cnsp.on('connection', (socket) => {
    console.log('Client connected');

    socket.emit('abcd', 'This is a custom namespace.')

    socket.on('disconnect', () => {
        
        console.log('Client disconnected');

    });
})
```
- client side
```
  useEffect(() => {
    
  const socket = io('http://localhost:3000/multan');

  socket.on('abcd', (data) => {
      console.log(data);
    })

    return () => {
      socket.disconnect()
    }
  }, [])
```
## 5 - Rooms or Channels in Socket.io
A room is an arbitrary channel that sockets can join and leave. One thing to keep in mind rooms can only be joined on the server side.
- **Joining a Rooms**
```
socket.join(`room${roomno}`);
```
- **Leaving a Room**
```
socket.leave(`room${roomno}`);
```
- **server side**
```
var roomno = 1

// Socket.IO
io.on('connection', (socket) => {
    console.log('Client connected');

    socket.join(`room${roomno}`)
    io.sockets.in(`room${roomno}`).emit('roomsjoined', `Hello you joined room ${roomno}`)


    socket.on('disconnect', () => {
        
        console.log('Client disconnected');

    });
})
```
- **client side**
```
  useEffect(() => {
    
  const socket = io('http://localhost:3000');

  socket.on('roomsjoined', (data) => {
      console.log(data);
    })

    return () => {
      socket.disconnect()
    }
  }, [])
```
