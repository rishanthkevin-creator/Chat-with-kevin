# Chat-with-kevinfrom flask import Flask, request, jsonify, render_template_string
import os
import openai

app = Flask(__name__)

# Use environment variable for OpenAI key
openai.api_key = os.getenv("OPENAI_API_KEY")
if not openai.api_key:
    raise ValueError("Please set the OPENAI_API_KEY environment variable!")

HTML_PAGE = """
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Chat with Kevin</title>
<style>
body { font-family: Arial, sans-serif; margin: 20px; background: #f7f7f7; }
#chatbox { border:1px solid #ccc; padding:10px; height:60vh; overflow-y:scroll; background:#fff; border-radius:8px; }
#message { width:75%; padding:8px; }
button { padding:8px 12px; margin-left:8px; cursor:pointer; border:none; border-radius:5px; background:#0b64d1; color:#fff; }
.user { color: #0b64d1; margin:6px 0; }
.bot { color: #0b8a3e; margin:6px 0; }
</style>
</head>
<body>
<h1>Chat with Kevin</h1>
<div id="chatbox"></div>
<input type="text" id="message" placeholder="Type a message..." />
<button onclick="sendMessage()">Send</button>

<script>
async function sendMessage(){
    const input = document.getElementById('message');
    const text = input.value.trim();
    if(!text) return;
    const chatbox = document.getElementById('chatbox');
    chatbox.innerHTML += `<div class="user"><b>You:</b> ${escapeHtml(text)}</div>`;
    input.value = '';
    chatbox.scrollTop = chatbox.scrollHeight;

    const response = await fetch('/chat', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ message: text })
    });
    const data = await response.json();
    chatbox.innerHTML += `<div class="bot"><b>Kevin:</b> ${escapeHtml(data.reply)}</div>`;
    chatbox.scrollTop = chatbox.scrollHeight;
}

function escapeHtml(s){
    return s.replace(/&/g,"&amp;").replace(/</g,"&lt;").replace(/>/g,"&gt;");
}

document.getElementById('message').addEventListener('keydown', e => {
    if(e.key === 'Enter') sendMessage();
});
</script>
</body>
</html>
"""

@app.route("/")
def home():
    return render_template_string(HTML_PAGE)

@app.route("/chat", methods=["POST"])
def chat():
    user_msg = request.json.get("message","")
    if not user_msg:
        return jsonify({"reply": "Please type something!"})

    try:
        resp = openai.ChatCompletion.create(
            model="gpt-4o-mini",
            messages=[
                {"role":"system","content":"You are Kevin, a friendly helpful chatbot."},
                {"role":"user","content":user_msg}
            ],
            temperature=0.7,
            max_tokens=400
        )
        reply = resp.choices[0].message.content
    except Exception as e:
        reply = f"Error: {str(e)}"
    return jsonify({"reply": reply})

if __name__ == "__main__":
    port = int(os.environ.get("PORT",5000))
    app.run(host="0.0.0.0", port=port)
