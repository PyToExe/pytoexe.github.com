const express = require('express');
const multer = require('multer');
const { exec } = require('child_process');
const { promisify } = require('util');
const path = require('path');
const fs = require('fs');

const app = express();
const port = 3000;

// Multer configuration for file upload
const storage = multer.diskStorage({
    destination: function (req, file, cb) {
        cb(null, 'uploads/')
    },
    filename: function (req, file, cb) {
        cb(null, file.originalname)
    }
});

const upload = multer({ storage: storage });

// Serve static files (uploaded executables)
app.use('/executables', express.static('executables'));

// Home page
app.get('/', (req, res) => {
    res.sendFile(path.join(__dirname, 'index.html'));
});

// File upload and conversion route
app.post('/convert', upload.single('scriptFile'), async (req, res) => {
    if (!req.file) {
        return res.status(400).send('No file uploaded.');
    }

    if (!req.file.originalname.endsWith('.py')) {
        return res.status(400).send('Please upload a Python script file (.py).');
    }

    const scriptPath = path.join(__dirname, 'uploads', req.file.filename);
    const executablePath = path.join(__dirname, 'executables', `${path.parse(req.file.filename).name}.exe`);

    try {
        // Execute PyInstaller command
        const execAsync = promisify(exec);
        const { stdout, stderr } = await execAsync(`pyinstaller --onefile ${scriptPath}`);

        if (stderr) {
            throw new Error(stderr);
        }

        // Provide download link
        const downloadLink = `<a href="/executables/${path.parse(req.file.filename).name}.exe">Download Executable</a>`;
        res.send(`Conversion successful.<br>${downloadLink}`);
    } catch (error) {
        console.error('Conversion error:', error);
        res.status(500).send('Conversion failed. Please try again.');
    } finally {
        // Clean up: delete uploaded script
        fs.unlinkSync(scriptPath);
    }
});

// Start server
app.listen(port, () => {
    console.log(`Server running at http://localhost:${port}`);
});
