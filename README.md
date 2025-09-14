<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Sistema de Envio de E-mails em Massa</title>
    <style>
        * {
            margin: 0;
            padding: 0;
            box-sizing: border-box;
        }

        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            min-height: 100vh;
            display: flex;
            align-items: center;
            justify-content: center;
            padding: 20px;
        }

        .container {
            background: white;
            border-radius: 15px;
            box-shadow: 0 20px 40px rgba(0,0,0,0.1);
            padding: 40px;
            width: 100%;
            max-width: 800px;
            animation: slideIn 0.6s ease-out;
        }

        @keyframes slideIn {
            from {
                opacity: 0;
                transform: translateY(30px);
            }
            to {
                opacity: 1;
                transform: translateY(0);
            }
        }

        h1 {
            color: #333;
            text-align: center;
            margin-bottom: 30px;
            font-size: 2.2em;
            font-weight: 300;
        }

        .form-group {
            margin-bottom: 25px;
        }

        label {
            display: block;
            margin-bottom: 8px;
            color: #555;
            font-weight: 500;
            font-size: 1.1em;
        }

        input[type="text"], 
        input[type="email"], 
        textarea, 
        input[type="file"] {
            width: 100%;
            padding: 12px 15px;
            border: 2px solid #e1e5e9;
            border-radius: 8px;
            font-size: 16px;
            transition: all 0.3s ease;
            background: #f8f9fa;
        }

        input[type="text"]:focus, 
        input[type="email"]:focus, 
        textarea:focus {
            outline: none;
            border-color: #667eea;
            background: white;
            box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
        }

        textarea {
            resize: vertical;
            min-height: 120px;
            font-family: inherit;
        }

        .file-input-wrapper {
            position: relative;
            display: inline-block;
            width: 100%;
        }

        input[type="file"] {
            cursor: pointer;
        }

        .send-btn {
            width: 100%;
            padding: 15px;
            background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
            color: white;
            border: none;
            border-radius: 8px;
            font-size: 18px;
            font-weight: 600;
            cursor: pointer;
            transition: all 0.3s ease;
            text-transform: uppercase;
            letter-spacing: 1px;
        }

        .send-btn:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 25px rgba(102, 126, 234, 0.3);
        }

        .send-btn:disabled {
            background: #ccc;
            cursor: not-allowed;
            transform: none;
            box-shadow: none;
        }

        .status {
            margin-top: 20px;
            padding: 15px;
            border-radius: 8px;
            display: none;
            font-weight: 500;
        }

        .status.success {
            background: #d4edda;
            color: #155724;
            border: 1px solid #c3e6cb;
        }

        .status.error {
            background: #f8d7da;
            color: #721c24;
            border: 1px solid #f5c6cb;
        }

        .status.loading {
            background: #cce7ff;
            color: #0056b3;
            border: 1px solid #99d3ff;
        }

        .progress-bar {
            width: 100%;
            height: 6px;
            background: #e9ecef;
            border-radius: 3px;
            overflow: hidden;
            margin-top: 10px;
            display: none;
        }

        .progress-fill {
            height: 100%;
            background: linear-gradient(90deg, #667eea, #764ba2);
            width: 0%;
            transition: width 0.3s ease;
        }

        .email-info {
            background: #f8f9fa;
            padding: 15px;
            border-radius: 8px;
            margin-bottom: 25px;
            border-left: 4px solid #667eea;
        }

        .email-info strong {
            color: #667eea;
        }

        .helper-text {
            font-size: 14px;
            color: #666;
            margin-top: 5px;
            font-style: italic;
        }
    </style>
</head>
<body>
    <div class="container">
        <h1>📧 Sistema de Envio de E-mails em Massa</h1>
        
        <div class="email-info">
            <strong>E-mail Remetente:</strong> equipedeentregadoebookinvestim@gmail.com
        </div>

        <form id="emailForm">
            <div class="form-group">
                <label for="recipients">Destinatários (separados por vírgula):</label>
                <textarea 
                    id="recipients" 
                    name="recipients" 
                    placeholder="exemplo1@email.com, exemplo2@email.com, exemplo3@email.com"
                    required
                ></textarea>
                <div class="helper-text">Digite os e-mails separados por vírgula</div>
            </div>

            <div class="form-group">
                <label for="subject">Assunto do E-mail:</label>
                <input 
                    type="text" 
                    id="subject" 
                    name="subject" 
                    placeholder="Digite o assunto do e-mail"
                    required
                >
            </div>

            <div class="form-group">
                <label for="message">Conteúdo do E-mail:</label>
                <textarea 
                    id="message" 
                    name="message" 
                    placeholder="Digite o conteúdo do e-mail aqui..."
                    required
                ></textarea>
            </div>

            <div class="form-group">
                <label for="attachments">Anexos (PDF):</label>
                <input 
                    type="file" 
                    id="attachments" 
                    name="attachments" 
                    accept=".pdf"
                    multiple
                >
                <div class="helper-text">Selecione arquivos PDF para anexar (opcional)</div>
            </div>

            <button type="submit" class="send-btn" id="sendBtn">
                Enviar E-mails em Massa
            </button>
        </form>

        <div id="status" class="status"></div>
        <div class="progress-bar" id="progressBar">
            <div class="progress-fill" id="progressFill"></div>
        </div>
    </div>

    <script>
        const form = document.getElementById('emailForm');
        const sendBtn = document.getElementById('sendBtn');
        const status = document.getElementById('status');
        const progressBar = document.getElementById('progressBar');
        const progressFill = document.getElementById('progressFill');

        function showStatus(message, type) {
            status.textContent = message;
            status.className = `status ${type}`;
            status.style.display = 'block';
        }

        function hideStatus() {
            status.style.display = 'none';
        }

        function showProgress(percent) {
            progressBar.style.display = 'block';
            progressFill.style.width = percent + '%';
        }

        function hideProgress() {
            progressBar.style.display = 'none';
            progressFill.style.width = '0%';
        }

        form.addEventListener('submit', async function(e) {
            e.preventDefault();
            
            const formData = new FormData();
            const recipients = document.getElementById('recipients').value.trim();
            const subject = document.getElementById('subject').value.trim();
            const message = document.getElementById('message').value.trim();
            const attachments = document.getElementById('attachments').files;

            // Validação básica
            if (!recipients || !subject || !message) {
                showStatus('Por favor, preencha todos os campos obrigatórios.', 'error');
                return;
            }

            // Validar e-mails
            const emailList = recipients.split(',').map(email => email.trim());
            const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
            const invalidEmails = emailList.filter(email => !emailRegex.test(email));
            
            if (invalidEmails.length > 0) {
                showStatus(`E-mails inválidos encontrados: ${invalidEmails.join(', ')}`, 'error');
                return;
            }

            // Preparar dados
            formData.append('recipients', recipients);
            formData.append('subject', subject);
            formData.append('message', message);

            // Adicionar anexos
            for (let i = 0; i < attachments.length; i++) {
                formData.append('attachments', attachments[i]);
            }

            // Desabilitar botão e mostrar progresso
            sendBtn.disabled = true;
            sendBtn.textContent = 'Enviando...';
            showStatus('Enviando e-mails, aguarde...', 'loading');
            showProgress(0);

            try {
                // Simular progresso
                let progress = 0;
                const progressInterval = setInterval(() => {
                    progress += Math.random() * 20;
                    if (progress >= 90) {
                        clearInterval(progressInterval);
                        showProgress(90);
                    } else {
                        showProgress(progress);
                    }
                }, 500);

                // Fazer requisição para o backend
                const response = await fetch('/send_emails', {
                    method: 'POST',
                    body: formData
                });

                clearInterval(progressInterval);
                showProgress(100);

                const result = await response.json();

                if (response.ok) {
                    showStatus(`✅ E-mails enviados com sucesso! Total: ${emailList.length} destinatários`, 'success');
                    form.reset();
                } else {
                    throw new Error(result.error || 'Erro desconhecido');
                }

            } catch (error) {
                console.error('Erro:', error);
                showStatus(`❌ Erro ao enviar e-mails: ${error.message}`, 'error');
            } finally {
                sendBtn.disabled = false;
                sendBtn.textContent = 'Enviar E-mails em Massa';
                setTimeout(() => {
                    hideProgress();
                }, 2000);
            }
        });

        // Auto-resize textarea
        document.querySelectorAll('textarea').forEach(textarea => {
            textarea.addEventListener('input', function() {
                this.style.height = 'auto';
                this.style.height = this.scrollHeight + 'px';
            });
        });
    </script>
</body>
</html>