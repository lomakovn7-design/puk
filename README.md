<!DOCTYPE html>
<html lang="ru">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Панель публикаций</title>
    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            background-color: #f0f2f5;
            display: flex;
            justify-content: center;
            align-items: center;
            padding: 20px;
            margin: 0;
        }
        .card {
            background: #ffffff;
            padding: 25px;
            border-radius: 16px;
            box-shadow: 0 4px 15px rgba(0, 0, 0, 0.1);
            width: 100%;
            max-width: 480px;
        }
        label {
            font-weight: bold;
            font-size: 13px;
            color: #606770;
            display: block;
            margin-top: 14px;
            margin-bottom: 5px;
        }
        input, select, textarea {
            width: 100%;
            padding: 10px;
            border: 1px solid #ccd0d5;
            border-radius: 8px;
            box-sizing: border-box;
            font-size: 14px;
            background-color: #fff;
        }
        textarea {
            height: 100px;
            resize: vertical;
        }
        .poll-option-row {
            display: flex;
            gap: 8px;
            margin-bottom: 8px;
        }
        .btn-small {
            background: #e4e6eb;
            border: none;
            padding: 8px 12px;
            border-radius: 6px;
            cursor: pointer;
            font-weight: bold;
        }
        .btn-small:hover {
            background: #d8dadf;
        }
        button.main-btn {
            width: 100%;
            background-color: #0088cc;
            color: white;
            border: none;
            padding: 12px;
            margin-top: 20px;
            border-radius: 8px;
            font-weight: bold;
            font-size: 15px;
            cursor: pointer;
            transition: background 0.2s;
        }
        button.main-btn:hover {
            background-color: #006699;
        }
        #status {
            margin-top: 15px;
            text-align: center;
            font-weight: bold;
            font-size: 14px;
        }
        .field-group {
            border-left: 3px solid #0088cc;
            padding-left: 10px;
            margin-top: 10px;
        }
    </style>
</head>
<body>

<div class="card">
    <label for="postType">Тип публикации:</label>
    <select id="postType" onchange="toggleFields()">
        <option value="post">✏️ Пост</option>
        <option value="poll">☑️ Опрос</option>
        <option value="video_note">⭕ Кружок</option>
        <option value="voice">🎙️ Голосовое</option>
        <option value="sticker">😊 Стикер</option>
        <option value="giveaway">🎁 Розыгрыш</option>
    </select>

    <label for="token">Токен бота:</label>
    <input type="password" id="token" placeholder="123456789:ABCdef...">

    <label for="chatId">ID чата:</label>
    <input type="text" id="chatId" placeholder="-100123456789">

    <!-- Текст (для постов и розыгрышей) -->
    <div id="textBlock">
        <label for="message">Текст сообщения:</label>
        <textarea id="message" placeholder="Напиши текст..."></textarea>
    </div>

    <!-- Настройка кнопок-ссылок -->
    <div id="btnBlock" class="field-group">
        <label>Кнопка-ссылка (опционально):</label>
        <input type="text" id="btnText" placeholder="Текст на кнопке (например, Ссылка)" style="margin-bottom: 6px;">
        <input type="url" id="btnUrl" placeholder="https://example.com">
    </div>

    <!-- Поля для опроса -->
    <div id="pollBlock" style="display:none;" class="field-group">
        <label for="pollQuestion">Вопрос опроса:</label>
        <input type="text" id="pollQuestion" placeholder="Задайте вопрос...">
        <label>Варианты ответов:</label>
        <div id="pollOptionsContainer">
            <div class="poll-option-row"><input type="text" class="poll-option" placeholder="Вариант 1"></div>
            <div class="poll-option-row"><input type="text" class="poll-option" placeholder="Вариант 2"></div>
        </div>
        <button type="button" class="btn-small" onclick="addPollOption()">+ Добавить вариант</button>
    </div>

    <!-- Загрузка файла (видео, голосовое, стикер) -->
    <div id="fileBlock" style="display:none;" class="field-group">
        <label id="fileLabel" for="fileInput">Выберите файл:</label>
        <input type="file" id="fileInput">
    </div>

    <!-- Поля для розыгрыша -->
    <div id="giveawayBlock" style="display:none;" class="field-group">
        <label for="giveawayTitle">Приз розыгрыша:</label>
        <input type="text" id="giveawayTitle" placeholder="например, 1000 Telegram Stars">
        <label for="giveawayBtn">Текст кнопки участия:</label>
        <input type="text" id="giveawayBtn" placeholder="🎉 Участвовать!">
    </div>

    <button class="main-btn" onclick="sendData()">Опубликовать</button>

    <div id="status"></div>
</div>

<script>
    document.getElementById('token').value = localStorage.getItem('bot_token') || '';
    document.getElementById('chatId').value = localStorage.getItem('chat_id') || '';

    function toggleFields() {
        const type = document.getElementById('postType').value;
        
        document.getElementById('textBlock').style.display = (type === 'post' || type === 'giveaway') ? 'block' : 'none';
        document.getElementById('btnBlock').style.display = (type === 'post') ? 'block' : 'none';
        document.getElementById('pollBlock').style.display = (type === 'poll') ? 'block' : 'none';
        document.getElementById('giveawayBlock').style.display = (type === 'giveaway') ? 'block' : 'none';
        
        const isFile = (type === 'video_note' || type === 'voice' || type === 'sticker');
        document.getElementById('fileBlock').style.display = isFile ? 'block' : 'none';

        if (type === 'video_note') {
            document.getElementById('fileLabel').innerText = "Видео файл для кружка (.mp4, квадратное):";
            document.getElementById('fileInput').accept = "video/mp4,video/*";
        } else if (type === 'voice') {
            document.getElementById('fileLabel').innerText = "Аудио файл для голосового (.ogg, .mp3):";
            document.getElementById('fileInput').accept = "audio/*";
        } else if (type === 'sticker') {
            document.getElementById('fileLabel').innerText = "Файл стикера (.webp, .png, .tgs):";
            document.getElementById('fileInput').accept = ".webp,.png,.tgs";
        }
    }

    function addPollOption() {
        const container = document.getElementById('pollOptionsContainer');
        const count = container.children.length + 1;
        const div = document.createElement('div');
        div.className = 'poll-option-row';
        div.innerHTML = `<input type="text" class="poll-option" placeholder="Вариант ${count}">`;
        container.appendChild(div);
    }

    async function sendData() {
        const token = document.getElementById('token').value.trim();
        const chatId = document.getElementById('chatId').value.trim();
        const type = document.getElementById('postType').value;
        const statusDiv = document.getElementById('status');

        if (!token || !chatId) {
            statusDiv.style.color = "red";
            statusDiv.innerText = "Заполни токен и ID чата!";
            return;
        }

        localStorage.setItem('bot_token', token);
        localStorage.setItem('chat_id', chatId);

        statusDiv.style.color = "gray";
        statusDiv.innerText = "Отправка...";

        try {
            let endpoint = '';
            let bodyData = { chat_id: chatId };

            if (type === 'post') {
                endpoint = 'sendMessage';
                bodyData.text = document.getElementById('message').value.trim();
                const bText = document.getElementById('btnText').value.trim();
                const bUrl = document.getElementById('btnUrl').value.trim();

                if (bText && bUrl) {
                    bodyData.reply_markup = {
                        inline_keyboard: [[{ text: bText, url: bUrl }]]
                    };
                }
            } else if (type === 'poll') {
                endpoint = 'sendPoll';
                bodyData.question = document.getElementById('pollQuestion').value.trim();
                const options = Array.from(document.querySelectorAll('.poll-option'))
                                     .map(i => i.value.trim())
                                     .filter(v => v.length > 0);
                
                if (options.length < 2) {
                    statusDiv.style.color = "red";
                    statusDiv.innerText = "Укажи минимум 2 варианта ответа!";
                    return;
                }
                bodyData.options = options;
            } else if (type === 'giveaway') {
                endpoint = 'sendMessage';
                const prize = document.getElementById('giveawayTitle').value.trim();
                const desc = document.getElementById('message').value.trim();
                const btnText = document.getElementById('giveawayBtn').value.trim() || '🎉 Участвовать!';

                bodyData.text = `🎁 **РОЗЫГРЫШ** 🎁\n\n**Приз:** ${prize}\n\n${desc}`;
                bodyData.parse_mode = 'Markdown';
                bodyData.reply_markup = {
                    inline_keyboard: [[{ text: btnText, callback_data: 'giveaway_join' }]]
                };
            } else if (type === 'video_note' || type === 'voice' || type === 'sticker') {
                const fileInput = document.getElementById('fileInput');
                if (!fileInput.files[0]) {
                    statusDiv.style.color = "red";
                    statusDiv.innerText = "Выберите файл!";
                    return;
                }

                const formData = new FormData();
                formData.append('chat_id', chatId);

                if (type === 'video_note') {
                    endpoint = 'sendVideoNote';
                    formData.append('video_note', fileInput.files[0]);
                } else if (type === 'voice') {
                    endpoint = 'sendVoice';
                    formData.append('voice', fileInput.files[0]);
                } else if (type === 'sticker') {
                    endpoint = 'sendSticker';
                    formData.append('sticker', fileInput.files[0]);
                }

                const response = await fetch(`https://api.telegram.org/bot${token}/${endpoint}`, {
                    method: 'POST',
                    body: formData
                });
                const resData = await response.json();
                handleResult(resData, statusDiv);
                return;
            }

            const response = await fetch(`https://api.telegram.org/bot${token}/${endpoint}`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(bodyData)
            });

            const data = await response.json();
            handleResult(data, statusDiv);

        } catch (error) {
            statusDiv.style.color = "red";
            statusDiv.innerText = "❌ Ошибка отправки!";
        }
    }

    function handleResult(data, statusDiv) {
        if (data.ok) {
            statusDiv.style.color = "green";
            statusDiv.innerText = "✅ Успешно опубликовано!";
        } else {
            statusDiv.style.color = "red";
            statusDiv.innerText = "❌ Ошибка: " + data.description;
        }
    }
</script>

</body>
</html>
