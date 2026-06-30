<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>锂电池热失控风险判定系统-Qwen3.7-Plus</title>
    <style>
        body {
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            background-color: #f0f4f8;
            margin: 0;
            padding: 20px;
            display: flex;
            justify-content: center;
        }
        #chat-container {
            width: 100%;
            max-width: 680px;
            background: white;
            border-radius: 12px;
            box-shadow: 0 4px 15px rgba(0,0,0,0.1);
            display: flex;
            flex-direction: column;
            height: 85vh;
        }
        #header {
            background-color: #2E86AB;
            color: white;
            padding: 15px 20px;
            border-radius: 12px 12px 0 0;
            text-align: center;
            font-weight: bold;
            font-size: 18px;
        }
        #chat-box {
            flex: 1;
            padding: 20px;
            overflow-y: auto;
            display: flex;
            flex-direction: column;
            gap: 15px;
        }
        .message {
            padding: 10px 15px;
            border-radius: 8px;
            max-width: 80%;
            line-height: 1.6;
            word-wrap: break-word;
        }
        .user-message {
            background-color: #e3f2fd;
            align-self: flex-end;
            color: #0d47a1;
            border-bottom-right-radius: 0;
        }
        .bot-message {
            background-color: #f5f5f5;
            align-self: flex-start;
            color: #333;
            border-bottom-left-radius: 0;
        }
        .system-notice {
            background-color: #ffebee;
            color: #c62828;
            font-size: 0.85em;
            text-align: center;
            align-self: center;
            border-radius: 4px;
            padding: 6px 12px;
        }
        #input-area {
            display: flex;
            padding: 15px;
            border-top: 1px solid #ddd;
            background: #fafafa;
            border-radius: 0 0 12px 12px;
        }
        #user-input {
            flex: 1;
            padding: 10px;
            border: 1px solid #ddd;
            border-radius: 6px;
            outline: none;
            font-size: 1em;
        }
        #send-btn {
            background-color: #2E86AB;
            color: white;
            border: none;
            padding: 10px 22px;
            margin-left: 10px;
            border-radius: 6px;
            cursor: pointer;
            font-size: 1em;
        }
        #send-btn:hover {
            background-color: #246d8c;
        }
        #send-btn:disabled {
            background-color: #999;
            cursor: not-allowed;
        }
    </style>
</head>
<body>
<div id="chat-container">
    <div id="header">锂电池热失控风险判定系统（Qwen3.7-Plus）</div>
    <div id="chat-box">
        <div class="bot-message">
            请输入电池传感器数据，包含：平均温度、最高温度、电芯温差、内阻涨幅、温升速率、单体电压、电池类型。<br>
            示例：平均32℃，最高43℃，温差7℃，内阻涨幅62.5%，温升速率1.2℃/min，最低电压3.1V，磷酸铁锂
        </div>
    </div>
    <div id="input-area">
        <input id="user-input" placeholder="粘贴电池传感器数据，自动判定漏气/热失控风险">
        <button id="send-btn">提交判定</button>
    </div>
</div>

<script>
// ========== 这里修改成你自己的阿里云密钥 ==========
const API_KEY = "sk-ws-H.RXDYXEP.3uyv.MEQCIESb8zJNsNTElTz80mqlOEtZ9jcby6ScRLbO8-IbVUEyAiB_JusM4e378cUuNVNcuKUMWsVD3qrX-HMHAE8bAoTDBw";
const API_URL = "https://ws-p971kjlx5vieyqj7.cn-beijing.maas.aliyuncs.com/compatible-mode/v1/chat/completions";
const MODEL_NAME = "qwen3.7-plus";
// ==============================================

const chatBox = document.getElementById('chat-box');
const userInput = document.getElementById('user-input');
const sendBtn = document.getElementById('send-btn');

function addMsg(text, isUser) {
    const div = document.createElement('div');
    div.className = isUser ? 'message user-message' : 'message bot-message';
    div.innerHTML = text.replaceAll('\n', '<br>');
    chatBox.appendChild(div);
    chatBox.scrollTop = chatBox.scrollHeight;
}

async function sendRequest() {
    const text = userInput.value.trim();
    if (!text) return;
    addMsg(text, true);
    userInput.value = '';
    sendBtn.disabled = true;

    try {
        const res = await fetch(API_URL, {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json',
                'Authorization': `Bearer ${API_KEY}`
            },
            body: JSON.stringify({
                model: MODEL_NAME,
                messages: [
                    {
                        role: "system",
                        content: "你是锂电池安全检测专家，根据用户提供的传感器数据，分三段输出结果：1.风险等级；2.故障机理分析；3.处置操作建议，简洁专业。"
                    },
                    { role: "user", content: text }
                ],
                temperature: 0.3
            })
        });
        const data = await res.json();
        if (data.error) throw new Error(data.error.message);
        const reply = data.choices[0].message.content;
        addMsg(reply, false);
    } catch (err) {
        addMsg(`请求失败：${err.message}\n校园网会拦截阿里云私有接口，切换手机热点重试`, false);
    }
    sendBtn.disabled = false;
}

sendBtn.addEventListener('click', sendRequest);
userInput.addEventListener('keydown', e => e.key === 'Enter' && sendRequest());
</script>
</body>
</html>
