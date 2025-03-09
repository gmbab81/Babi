<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Professional Background Remover</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
        }

        body {
            background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
            min-height: 100vh;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
        }

        .container {
            background: white;
            padding: 40px;
            border-radius: 20px;
            box-shadow: 0 10px 30px rgba(0,0,0,0.1);
            width: 100%;
            max-width: 800px;
            text-align: center;
        }

        h1 {
            color: #2c3e50;
            margin-bottom: 30px;
            font-size: 2.5em;
            font-weight: 600;
        }

        .upload-container {
            border: 3px dashed #3498db;
            padding: 40px;
            border-radius: 15px;
            margin: 20px 0;
            transition: all 0.3s ease;
            position: relative;
        }

        .upload-container:hover {
            background: #f8f9fa;
            border-color: #2980b9;
        }

        #fileInput {
            display: none;
        }

        .upload-label {
            cursor: pointer;
            color: #3498db;
            font-size: 1.2em;
            font-weight: 500;
        }

        .upload-label i {
            font-size: 2em;
            margin-bottom: 15px;
            display: block;
        }

        #preview {
            max-width: 100%;
            margin-top: 20px;
            border-radius: 10px;
            display: none;
        }

        .button-group {
            margin-top: 30px;
            display: flex;
            gap: 15px;
            justify-content: center;
        }

        button {
            padding: 12px 30px;
            border: none;
            border-radius: 25px;
            cursor: pointer;
            font-size: 1em;
            font-weight: 600;
            transition: all 0.3s ease;
            text-transform: uppercase;
        }

        #processBtn {
            background: #3498db;
            color: white;
        }

        #processBtn:hover {
            background: #2980b9;
            transform: translateY(-2px);
        }

        #resetBtn {
            background: #e74c3c;
            color: white;
        }

        #resetBtn:hover {
            background: #c0392b;
            transform: translateY(-2px);
        }

        .result-container {
            margin-top: 30px;
            padding: 20px;
            background: #f8f9fa;
            border-radius: 15px;
            display: none;
        }

        .download-btn {
            background: #2ecc71;
            margin-top: 15px;
        }

        .download-btn:hover {
            background: #27ae60;
        }

        .progress-bar {
            width: 100%;
            height: 10px;
            background: #eee;
            border-radius: 5px;
            margin-top: 20px;
            overflow: hidden;
            display: none;
        }

        .progress {
            width: 0%;
            height: 100%;
            background: #3498db;
            transition: width 0.3s ease;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>Background Remover</h1>
        
        <div class="upload-container" id="dropZone">
            <input type="file" id="fileInput" accept="image/*">
            <label for="fileInput" class="upload-label">
                <i class="fas fa-cloud-upload-alt"></i>
                Drag & Drop or Click to Upload
            </label>
            <img id="preview" alt="Preview">
        </div>

        <div class="progress-bar">
            <div class="progress"></div>
        </div>

        <div class="button-group">
            <button id="processBtn">Remove Background</button>
            <button id="resetBtn">Reset</button>
        </div>

        <div class="result-container" id="resultContainer">
            <img id="resultImage" alt="Result" style="max-width: 100%;">
            <button class="download-btn" id="downloadBtn">Download Image</button>
        </div>
    </div>

    <script>
        const dropZone = document.getElementById('dropZone');
        const fileInput = document.getElementById('fileInput');
        const preview = document.getElementById('preview');
        const processBtn = document.getElementById('processBtn');
        const resetBtn = document.getElementById('resetBtn');
        const resultContainer = document.getElementById('resultContainer');
        const progressBar = document.querySelector('.progress-bar');
        const progress = document.querySelector('.progress');

        // Handle drag & drop
        dropZone.addEventListener('dragover', (e) => {
            e.preventDefault();
            dropZone.style.borderColor = '#2980b9';
        });

        dropZone.addEventListener('dragleave', () => {
            dropZone.style.borderColor = '#3498db';
        });

        dropZone.addEventListener('drop', (e) => {
            e.preventDefault();
            dropZone.style.borderColor = '#3498db';
            const file = e.dataTransfer.files[0];
            handleFile(file);
        });

        // Handle file input
        fileInput.addEventListener('change', (e) => {
            const file = e.target.files[0];
            handleFile(file);
        });

        function handleFile(file) {
            if (file && file.type.startsWith('image/')) {
                const reader = new FileReader();
                reader.onload = (e) => {
                    preview.src = e.target.result;
                    preview.style.display = 'block';
                    resultContainer.style.display = 'none';
                };
                reader.readAsDataURL(file);
            }
        }

        // Process image
        processBtn.addEventListener('click', async () => {
            const file = fileInput.files[0];
            if (!file) return;

            progressBar.style.display = 'block';
            progress.style.width = '30%';

            const formData = new FormData();
            formData.append('image_file', file);

            try {
                const response = await fetch('https://api.bgremovalapi.com/v1/remove', {
                    method: 'POST',
                    headers: {
                        'X-API-Key': 'YOUR_API_KEY'
                    },
                    body: formData
                });

                if (!response.ok) throw new Error('API Error');
                
                const result = await response.blob();
                const resultUrl = URL.createObjectURL(result);
                
                progress.style.width = '100%';
                setTimeout(() => {
                    progressBar.style.display = 'none';
                    progress.style.width = '0%';
                }, 500);

                document.getElementById('resultImage').src = resultUrl;
                resultContainer.style.display = 'block';

                // Download handler
                document.getElementById('downloadBtn').onclick = () => {
                    const a = document.createElement('a');
                    a.href = resultUrl;
                    a.download = 'background_removed.png';
                    document.body.appendChild(a);
                    a.click();
                    document.body.removeChild(a);
                };

            } catch (error) {
                console.error('Error:', error);
                alert('Error processing image. Please try again.');
                progressBar.style.display = 'none';
                progress.style.width = '0%';
            }
        });

        // Reset everything
        resetBtn.addEventListener('click', () => {
            fileInput.value = '';
            preview.src = '';
            preview.style.display = 'none';
            resultContainer.style.display = 'none';
            progressBar.style.display = 'none';
            progress.style.width = '0%';
        });
    </script>
</body>
</html>