# Pdfs
pdf-theme-rename/
│
├── backend/              # Código do servidor (Flask)
│   ├── app.py            # Arquivo principal do servidor Flask
│   ├── requirements.txt  # Dependências do backend
│   ├── utils.py          # Funções auxiliares, como renomeação de arquivos e PDF
│   └── uploads/          # Diretório para armazenar os arquivos temporários
│
├── frontend/             # Código do frontend (HTML, CSS, JavaScript)
│   ├── index.html        # Página principal do site
│   ├── script.js         # Lógica do frontend (upload, download)
│   └── style.css         # Estilo do site
│
├── .gitignore            # Arquivos a serem ignorados pelo Git
├── README.md             # Documentação do repositório
from flask import Flask, request, jsonify, send_from_directory
import os
import PyPDF2
from google.cloud import vision
from google.cloud.vision import types

app = Flask(__name__)

# Configura o diretório de uploads
UPLOAD_FOLDER = 'backend/uploads'
app.config['UPLOAD_FOLDER'] = UPLOAD_FOLDER

# Configuração do Google Cloud Vision (com as credenciais da sua conta)
os.environ["GOOGLE_APPLICATION_CREDENTIALS"] = "path/to/your/google-credentials.json"

# Função para detectar texto nas imagens usando o Google Vision
def detect_text_in_image(image_path):
    client = vision.ImageAnnotatorClient()
    with open(image_path, 'rb') as image_file:
        content = image_file.read()
    image = types.Image(content=content)
    response = client.text_detection(image=image)
    texts = response.text_annotations
    return texts[0].description if texts else None

# Função para extrair texto de um PDF
def extract_text_from_pdf(pdf_path):
    with open(pdf_path, "rb") as file:
        reader = PyPDF2.PdfReader(file)
        text = ""
        for page in reader.pages:
            text += page.extract_text()
        return text

# Rota para upload do PDF
@app.route('/upload', methods=['POST'])
def upload_file():
    if 'file' not in request.files:
        return jsonify({'error': 'No file part'}), 400
    file = request.files['file']
    if file.filename == '':
        return jsonify({'error': 'No selected file'}), 400
    if file:
        filename = file.filename
        filepath = os.path.join(app.config['UPLOAD_FOLDER'], filename)
        file.save(filepath)
        
        # Detecta tema da imagem ou extrai texto do PDF
        text_from_pdf = extract_text_from_pdf(filepath)
        theme = detect_text_in_image(filepath) if not text_from_pdf else text_from_pdf
        
        # Renomear o arquivo
        new_filename = f"{theme}.pdf" if theme else f"untitled.pdf"
        new_filepath = os.path.join(app.config['UPLOAD_FOLDER'], new_filename)
        os.rename(filepath, new_filepath)

        return jsonify({'message': 'File uploaded successfully', 'new_filename': new_filename})

# Rota para download do arquivo renomeado
@app.route('/download/<filename>')
def download_file(filename):
    return send_from_directory(app.config['UPLOAD_FOLDER'], filename)

if __name__ == '__main__':
    app.run(debug=True)
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Renomear PDFs</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div class="container">
        <div class="header">
            <h1>Renomeie seus PDFs com base no tema</h1>
            <p>Faça o upload do seu arquivo PDF para que possamos renomeá-lo automaticamente com base no tema extraído.</p>
        </div>
        
        <form id="upload-form" enctype="multipart/form-data">
            <input type="file" name="file" id="file" accept=".pdf" required>
            <button type="submit">Enviar</button>
        </form>

        <div id="message"></div>
    </div>

    <script src="script.js"></script>
</body>
</html>
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    font-family: 'Arial', sans-serif;
    background-color: #f5f5f5;
    padding: 50px;
    display: flex;
    justify-content: center;
    align-items: center;
    height: 100vh;
}

.container {
    background-color: #ffffff;
    padding: 40px;
    border-radius: 15px;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.1);
    text-align: center;
    width: 400px;
}

.header h1 {
    color: #333;
    font-size: 2.5em;
}

.header p {
    color: #666;
    font-size: 1.1em;
    margin-bottom: 20px;
}

input[type="file"] {
    width: 100%;
    padding: 12px;
    margin-bottom: 20px;
    border: 2px solid #ddd;
    border-radius: 5px;
}

button {
    width: 100%;
    padding: 12px;
    background-color: #4CAF50;
    color: white;
    border: none;
    border-radius: 5px;
    font-size: 1.1em;
    cursor: pointer;
}

button:hover {
    background-color: #45a049;
}

#message {
    margin-top: 20px;
    color: #333;
}
document.getElementById('upload-form').addEventListener('submit', async function(e) {
    e.preventDefault();
    
    const formData = new FormData();
    formData.append('file', document.getElementById('file').files[0]);

    const response = await fetch('/upload', {
        method: 'POST',
        body: formData
    });

    const result = await response.json();
    document.getElementById('message').innerText = result.message || 'Erro ao processar o arquivo';

    if (result.new_filename) {
        const downloadLink = document.createElement('a');
        downloadLink.href = `/download/${result.new_filename}`;
        downloadLink.innerText = 'Clique aqui para baixar o arquivo renomeado';
        document.getElementById('message').appendChild(downloadLink);
    }
});
Flask==2.2.3
PyPDF2==1.26.0
google-cloud-vision==3.0.0
pdfminer.six==20201018
flask-cors==3.1.1
pip install -r requirements.txt
python app.py
git init
git add .
git commit -m "Initial commit"
git remote add origin https://github.com/username/pdf-theme-rename.git
git push -u origin main
