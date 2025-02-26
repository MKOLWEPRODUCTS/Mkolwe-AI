<!DOCTYPE html>
<html lang="sw">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Mkolwe AI Chat</title>
    <script src="config.js"></script> <!--sk-proj-nHa6v1BJtUnBa0Bah7hIgL6uCHLsvxxkvQBsvUmSgrAE0D4skKwoBrWziBeZpLsGCgFDovLhz_T3BlbkFJOy6AH_ZvpKT8udCx05x6UW78Y7d0BpjbjGpG4r18Ae8QM1tG3LH36gFkZ8QeaK75EDTcnbHrsA-->
    <style>
        body {
            font-family: Arial, sans-serif;
            background-color: #f0f0f5;
            display: flex;
            flex-direction: column;
            align-items: center;
            height: 100vh;
            margin: 0;
        }

        .menu {
            display: flex;
            gap: 10px;
            margin: 10px;
        }

        .menu button {
            padding: 10px;
            border: none;
            cursor: pointer;
            background: #0052cc;
            color: white;
            border-radius: 5px;
        }

        .chat-container {
            width: 400px;
            border-radius: 10px;
            overflow: hidden;
            background: linear-gradient(135deg, #0052cc, #00aaff);
        }

        .chat-header {
            background: #003366;
            color: white;
            padding: 15px;
            text-align: center;
            font-size: 20px;
            font-weight: bold;
        }

        .chat-box {
            height: 300px;
            padding: 15px;
            overflow-y: auto;
            background: white;
        }

        .chat-input {
            display: flex;
            border-top: 1px solid #ddd;
        }

        .chat-input input {
            flex: 1;
            padding: 10px;
            border: none;
            outline: none;
        }

        .chat-input button {
            background: #003366;
            color: white;
            padding: 10px;
            border: none;
            cursor: pointer;
        }

        .message {
            display: flex;
            align-items: center;
            margin: 5px;
        }

        .user-message {
            background: #0052cc;
            color: white;
            padding: 8px 12px;
            border-radius: 10px;
            max-width: 70%;
            align-self: flex-end;
            display: flex;
            flex-direction: row-reverse;
        }

        .bot-message {
            background: #e6e6e6;
            color: black;
            padding: 8px 12px;
            border-radius: 10px;
            max-width: 70%;
            align-self: flex-start;
            display: flex;
        }
    </style>
</head>
<body>

    <div class="menu">
        <button onclick="switchMode('chat')">Chat</button>
        <button onclick="switchMode('image')">Create Image</button>
        <button onclick="switchMode('brainstorm')">Brainstorm</button>
        <button onclick="switchMode('more')">More</button>
    </div>

    <div class="chat-container">
        <div class="chat-header">Mkolwe AI</div>
        <div class="chat-box" id="chat-box"></div>
        <div class="chat-input">
            <input type="text" id="user-input" placeholder="Andika hapa...">
            <button onclick="sendMessage()">Tuma</button>
        </div>
    </div>

    <script>
        let mode = "chat"; // Default mode

        function switchMode(newMode) {
            mode = newMode;
            let chatBox = document.getElementById("chat-box");
            chatBox.innerHTML = `<p><strong>${mode.toUpperCase()} Mode</strong></p>`;
        }

        async function sendMessage() {
            let inputField = document.getElementById("user-input");
            let userText = inputField.value.trim();
            if (userText === "") return;

            let chatBox = document.getElementById("chat-box");

            let userMessage = document.createElement("div");
            userMessage.className = "message user-message";
            userMessage.innerHTML = `<span>${userText}</span>`;
            chatBox.appendChild(userMessage);
            inputField.value = "";

            let botMessage = document.createElement("div");
            botMessage.className = "message bot-message";
            chatBox.appendChild(botMessage);

            if (mode === "chat") {
                let response = await fetchOpenAI("gpt-4", userText);
                botMessage.innerHTML = `<span>${response}</span>`;
            } else if (mode === "image") {
                let imageUrl = await fetchOpenAI("dalle", userText);
                botMessage.innerHTML = `<img src="${imageUrl}" width="100%">`;
            } else if (mode === "brainstorm") {
                let response = await fetchOpenAI("gpt-4", `Give me creative ideas about: ${userText}`);
                botMessage.innerHTML = `<span>${response}</span>`;
            } else {
                botMessage.innerHTML = `<span>More features coming soon!</span>`;
            }

            chatBox.scrollTop = chatBox.scrollHeight;
        }

        async function fetchOpenAI(model, prompt) {
            let url = model === "dalle" ? "https://api.openai.com/v1/images/generations" : "https://api.openai.com/v1/chat/completions";
            let body = model === "dalle" ? 
            { "prompt": prompt, "n": 1, "size": "256x256" } : 
            { "model": "gpt-4", "messages": [{ "role": "user", "content": prompt }] };

            let response = await fetch(url, {
                method: "POST",
                headers: {
                    "Authorization": `Bearer ${API_KEY}`,
                    "Content-Type": "application/json"
                },
                body: JSON.stringify(body)
            });

            let data = await response.json();
            return model === "dalle" ? data.data[0].url : data.choices[0].message.content;
        }
    </script>

</body>
</html>
