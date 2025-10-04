from flask import Flask, render_template_string, request
from flask_socketio import SocketIO, send

app = Flask(__name__)
app.config["SECRET_KEY"] = "secret!"
socketio = SocketIO(app)

HTML_TEMPLATE = """
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Flask Chat App</title>
    <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css" rel="stylesheet">
    <script src="https://cdn.socket.io/4.0.0/socket.io.min.js"></script>
    <style>
        body { background: #f8f9fa; padding: 20px; }
        #chat-box { height: 400px; overflow-y: scroll; border: 1px solid #ccc; border-radius: 10px; padding: 10px; background: #fff; }
        .message { margin: 5px 0; padding: 8px 12px; border-radius: 10px; display: inline-block; }
        .me { background: #007bff; color: #fff; align-self: flex-end; }
        .other { background: #e9ecef; }
    </style>
</head>
<body>
    <div class="container">
        <h2 class="text-center mb-4">💬 Flask Chat App</h2>
        <div id="chat-box" class="mb-3"></div>
        <form id="chat-form" class="d-flex">
            <input id="username" class="form-control me-2" placeholder="Username" required>
            <input id="message" class="form-control me-2" placeholder="Type a message..." required>
            <button class="btn btn-primary">Send</button>
        </form>
    </div>

    <script>
        const socket = io();
        const form = document.getElementById("chat-form");
        const chatBox = document.getElementById("chat-box");

        socket.on("message", function(data) {
            const msgDiv = document.createElement("div");
            msgDiv.classList.add("message");
            if (data.startsWith(document.getElementById("username").value + ":")) {
                msgDiv.classList.add("me");
            } else {
                msgDiv.classList.add("other");
            }
            msgDiv.textContent = data;
            chatBox.appendChild(msgDiv);
            chatBox.scrollTop = chatBox.scrollHeight;
        });

        form.addEventListener("submit", function(e) {
            e.preventDefault();
            const username = document.getElementById("username").value;
            const message = document.getElementById("message").value;
            if (username && message) {
                socket.send(username + ": " + message);
                document.getElementById("message").value = "";
            }
        });
    </script>
</body>
</html>
"""

@app.route("/")
def index():
    return render_template_string(HTML_TEMPLATE)

@socketio.on("message")
def handle_message(msg):
    send(msg, broadcast=True)

if __name__ == "__main__":
    socketio.run(app, debug=True)
