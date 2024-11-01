## 🚀 Basic Server Setup
```javascript
// Basic HTTP Server
const http = require('http');

const server = http.createServer((req, res) => {
    res.writeHead(200, {'Content-Type': 'text/plain'});
    res.end('Hello World\n');
});

server.listen(3000, () => {
    console.log('Server running at http://localhost:3000/');
});
```

## 📁 File System Operations
```javascript
const fs = require('fs');

// Async operations (preferred)
fs.readFile('file.txt', 'utf8', (err, data) => {
    if (err) throw err;
    console.log(data);
});

// Sync operations (blocking)
const data = fs.readFileSync('file.txt', 'utf8');

// Write file
fs.writeFile('file.txt', 'Hello World', (err) => {
    if (err) throw err;
});

// Append to file
fs.appendFile('file.txt', 'More content', (err) => {
    if (err) throw err;
});

// Check if file exists
fs.access('file.txt', fs.constants.F_OK, (err) => {
    console.log(`${err ? 'Does not exist' : 'Exists'}`);
});
```

## 🔄 Promises & Async/Await
```javascript
// Using promises
const fsPromises = require('fs').promises;

async function readFileAsync() {
    try {
        const data = await fsPromises.readFile('file.txt', 'utf8');
        console.log(data);
    } catch (error) {
        console.error('Error:', error);
    }
}

// Promise chaining
someAsyncFunction()
    .then(result => doSomething(result))
    .catch(error => console.error(error));
```

## 🛣️ Path Operations
```javascript
const path = require('path');

// Join paths
path.join(__dirname, 'folder', 'file.txt');

// Resolve path
path.resolve('folder', 'file.txt');

// Get file extension
path.extname('file.txt');  // Returns '.txt'

// Get filename
path.basename('/path/to/file.txt');  // Returns 'file.txt'
```

## 🌐 URL & QueryString
```javascript
const url = require('url');
const querystring = require('querystring');

// Parse URL
const myUrl = new URL('http://example.com/path?name=John');
console.log(myUrl.searchParams.get('name')); // 'John'

// Parse query string
const qs = querystring.parse('name=John&age=30');
console.log(qs.name); // 'John'
```

## 🎯 Events
```javascript
const EventEmitter = require('events');

class MyEmitter extends EventEmitter {}
const myEmitter = new MyEmitter();

// Event listener
myEmitter.on('event', (arg) => {
    console.log('Event triggered:', arg);
});

// Emit event
myEmitter.emit('event', 'test argument');
```

## 🧵 Child Processes
```javascript
const { spawn, exec } = require('child_process');

// Execute command
exec('ls -l', (error, stdout, stderr) => {
    if (error) {
        console.error(`Error: ${error}`);
        return;
    }
    console.log(`stdout: ${stdout}`);
});

// Spawn process
const child = spawn('ls', ['-l']);
child.stdout.on('data', (data) => {
    console.log(`stdout: ${data}`);
});
```

## 🔄 Streams
```javascript
const fs = require('fs');

// Read stream
const readStream = fs.createReadStream('input.txt');
readStream.on('data', (chunk) => {
    console.log(chunk);
});

// Write stream
const writeStream = fs.createWriteStream('output.txt');
writeStream.write('Hello World!');
writeStream.end();

// Pipe streams
readStream.pipe(writeStream);
```

## 🌐 HTTP Requests
```javascript
// Using http module
const http = require('http');

http.get('http://api.example.com', (resp) => {
    let data = '';
    resp.on('data', (chunk) => { data += chunk; });
    resp.on('end', () => { console.log(data); });
}).on('error', (err) => {
    console.log("Error: " + err.message);
});

// Modern fetch API (Node 18+)
async function fetchData() {
    try {
        const response = await fetch('http://api.example.com');
        const data = await response.json();
        console.log(data);
    } catch (error) {
        console.error('Error:', error);
    }
}
```

## ⚡ Process & Environment
```javascript
// Environment variables
console.log(process.env.NODE_ENV);

// Command line arguments
console.log(process.argv);

// Exit process
process.exit(1);

// Current directory
console.log(process.cwd());

// Platform
console.log(process.platform);
```

## 🔒 Error Handling
```javascript
// Try-catch
try {
    // Risky code
} catch (error) {
    console.error('Error:', error);
} finally {
    // Cleanup code
}

// Uncaught exceptions
process.on('uncaughtException', (err) => {
    console.error('Uncaught Exception:', err);
    process.exit(1);
});

// Unhandled promise rejections
process.on('unhandledRejection', (reason, promise) => {
    console.error('Unhandled Rejection at:', promise, 'reason:', reason);
});
```

## 🌟 Pro Tips
- Use `async/await` instead of callbacks when possible
- Implement proper error handling
- Use environment variables for configuration
- Leverage streams for large data
- Use the `debug` module for debugging
- Implement proper logging
- Use `process.env.NODE_ENV` for environment-specific code

## ⚠️ Common Gotchas
- Callback hell (use async/await)
- Blocking operations in event loop
- Memory leaks with event listeners
- Uncaught exceptions/rejections
- Not handling stream errors
- Sync operations in production code

Remember: Node.js is single-threaded but uses an event loop for non-blocking I/O operations. Always use async operations when possible!