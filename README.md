// server.js

const express = require('express');
const multer = require('multer');
const path = require('path');
const fs = require('fs');

const app = express();
const PORT = process.env.PORT || 3000;

// Storage setup
const storage = multer.diskStorage({
  destination: './uploads/',
  filename: function (req, file, cb) {
    cb(null, Date.now() + '-' + file.originalname);
  }
});
const upload = multer({ storage: storage });

app.use(express.json());
app.use(express.urlencoded({ extended: true }));
app.use('/uploads', express.static('uploads'));

// Make sure uploads folder exists
if (!fs.existsSync('./uploads')) {
  fs.mkdirSync('./uploads');
}

// Save video metadata
const videoDataFile = './videos.json';

// Serve Home Page
app.get('/', (req, res) => {
  res.send(`
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1">
      <title>Home</title>
      <script src="https://cdn.tailwindcss.com"></script>
    </head>
    <body class="bg-gray-100 flex flex-col items-center justify-center min-h-screen">

      <h1 class="text-4xl font-bold mb-8">Welcome to My Video Blog</h1>

      <div class="space-x-6">
        <a href="/upload" class="px-6 py-3 bg-blue-600 text-white rounded-xl shadow-md hover:bg-blue-700 transition">Upload Video</a>
        <a href="/gallery" class="px-6 py-3 bg-green-600 text-white rounded-xl shadow-md hover:bg-green-700 transition">View Videos</a>
      </div>

    </body>
    </html>
  `);
});

// Serve Upload Page
app.get('/upload', (req, res) => {
  res.send(`
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1">
      <title>Upload Video</title>
      <script src="https://cdn.tailwindcss.com"></script>
    </head>
    <body class="bg-gray-100 flex flex-col items-center justify-center min-h-screen">

      <h1 class="text-3xl font-bold mb-6">Upload a New Video</h1>

      <form action="/uploadVideo" method="POST" enctype="multipart/form-data" class="bg-white p-8 rounded-xl shadow-md space-y-6">
        <div>
          <label class="block text-gray-700 font-semibold mb-2">Title</label>
          <input type="text" name="title" required class="border rounded w-full py-2 px-3">
        </div>

        <div>
          <label class="block text-gray-700 font-semibold mb-2">Description</label>
          <textarea name="description" rows="4" required class="border rounded w-full py-2 px-3"></textarea>
        </div>

        <div>
          <label class="block text-gray-700 font-semibold mb-2">Select Video</label>
          <input type="file" name="video" accept="video/*" required class="border rounded w-full py-2 px-3">
        </div>

        <button type="submit" class="w-full bg-blue-600 text-white py-2 rounded-xl hover:bg-blue-700 transition">
          Upload
        </button>
      </form>

      <a href="/" class="mt-6 text-blue-600 hover:underline">← Back to Home</a>

    </body>
    </html>
  `);
});

// Upload Video Route
app.post('/uploadVideo', upload.single('video'), (req, res) => {
  const videoData = {
    title: req.body.title,
    description: req.body.description,
    filename: req.file.filename
  };

  let videos = [];
  if (fs.existsSync(videoDataFile)) {
    videos = JSON.parse(fs.readFileSync(videoDataFile));
  }
  videos.push(videoData);
  fs.writeFileSync(videoDataFile, JSON.stringify(videos, null, 2));

  res.redirect('/gallery');
});

// Serve Gallery Page
app.get('/gallery', (req, res) => {
  let videos = [];
  if (fs.existsSync(videoDataFile)) {
    videos = JSON.parse(fs.readFileSync(videoDataFile));
  }

  let videoCards = videos.map(video => `
    <div class="bg-white rounded-xl shadow-md overflow-hidden">
      <video controls class="w-full h-64 object-cover">
        <source src="/uploads/${video.filename}" type="video/mp4">
        Your browser does not support the video tag.
      </video>
      <div class="p-4">
        <h2 class="text-xl font-semibold mb-2">${video.title}</h2>
        <p class="text-gray-600 mb-4">${video.description}</p>
        <a href="/uploads/${video.filename}" download class="block w-full text-center bg-green-600 text-white py-2 rounded-xl hover:bg-green-700 transition">
          Download
        </a>
      </div>
    </div>
  `).join('');

  res.send(`
    <!DOCTYPE html>
    <html lang="en">
    <head>
      <meta charset="UTF-8">
      <meta name="viewport" content="width=device-width, initial-scale=1">
      <title>Video Gallery</title>
      <script src="https://cdn.tailwindcss.com"></script>
    </head>
    <body class="bg-gray-100 p-8">

      <h1 class="text-4xl font-bold text-center mb-8">My Video Gallery</h1>

      <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-8">
        ${videoCards || '<p class="text-center text-gray-600 col-span-3">No videos uploaded yet.</p>'}
      </div>

      <a href="/" class="block mt-8 text-center text-blue-600 hover:underline">← Back to Home</a>

    </body>
    </html>
  `);
});

// Start server
app.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}`);
});