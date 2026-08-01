<!DOCTYPE html>
<html lang="zh">
<head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
    <title>🔒 私人聊天</title>
    <style>
        body { max-width: 600px; margin: 20px auto; padding: 0 20px; font-family: sans-serif; }
        #messages { height: 400px; overflow-y: auto; border: 1px solid #ddd; padding: 10px; margin-bottom: 10px; background: #fafafa; }
        .msg { margin: 6px 0; }
        .me { color: #007bff; font-weight: bold; }
        .friend { color: #e67e22; font-weight: bold; }
        input, button { padding: 8px; font-size: 16px; }
        #controls { display: flex; gap: 8px; flex-wrap: wrap; margin-bottom: 10px; }
        #controls input { flex: 1; min-width: 100px; }
        #sendArea { display: flex; gap: 8px; }
        #sendArea input { flex: 1; }
        button { background: #007bff; color: #fff; border: none; cursor: pointer; border-radius: 4px; }
        button:disabled { background: #aaa; cursor: not-allowed; }
        .info { color: #666; font-size: 14px; }
    </style>
</head>
<body>

<h2>🔒 私人聊天室</h2>
<p class="info">你的 Peer ID：<span id="myPeerId">加载中...</span></p>

<div id="controls">
    <input id="remoteIdInput" placeholder="输入对方的 Peer ID" />
    <button id="connectBtn">连接</button>
    <button id="disconnectBtn" disabled>断开</button>
</div>

<div id="messages"></div>

<div id="sendArea">
    <input id="msgInput" placeholder="输入消息..." disabled />
    <button id="sendBtn" disabled>发送</button>
</div>

<!-- 引入 PeerJS 库（来自 CDN，免费且稳定） -->
<script src="https://unpkg.com/peerjs@1.5.2/dist/peerjs.min.js"></script>

<script>
    // ----- 1. 初始化 Peer 对象（生成自己的 ID）-----
    // 使用默认的免费信令服务器（peerjs.com），无需注册
    const peer = new Peer();

    let conn = null;          // 当前连接对象
    const messagesDiv = document.getElementById('messages');
    const myIdSpan = document.getElementById('myPeerId');
    const remoteInput = document.getElementById('remoteIdInput');
    const connectBtn = document.getElementById('connectBtn');
    const disconnectBtn = document.getElementById('disconnectBtn');
    const msgInput = document.getElementById('msgInput');
    const sendBtn = document.getElementById('sendBtn');

    // 显示自己的 ID
    peer.on('open', (id) => {
        myIdSpan.textContent = id;
    });

    // ----- 2. 监听对方发来的连接请求（作为被动方）-----
    peer.on('connection', (incomingConn) => {
        if (conn) {
            // 如果已有连接，拒绝新的
            incomingConn.close();
            return;
        }
        setupConnection(incomingConn);
    });

    // ----- 3. 主动连接对方（点击“连接”按钮）-----
    connectBtn.addEventListener('click', () => {
        const remoteId = remoteInput.value.trim();
        if (!remoteId) return alert('请输入对方的 Peer ID');
        if (conn) {
            alert('已连接，请先断开再重新连接');
            return;
        }
        // 发起连接
        const newConn = peer.connect(remoteId);
        // 等待连接打开
        newConn.on('open', () => {
            setupConnection(newConn);
        });
        newConn.on('error', (err) => {
            alert('连接失败：' + err.message);
        });
    });

    // ----- 4. 连接建立后的通用设置（双方共用）-----
    function setupConnection(connection) {
        conn = connection;

        // 更新界面状态
        connectBtn.disabled = true;
        disconnectBtn.disabled = false;
        msgInput.disabled = false;
        sendBtn.disabled = false;
        msgInput.focus();

        // 监听对方发来的消息
        conn.on('data', (data) => {
            appendMessage('对方', data);
        });

        // 监听连接关闭（对方断开或网络中断）
        conn.on('close', () => {
            alert('对方已断开连接');
            resetConnection();
        });

        // 提示连接成功
        appendMessage('系统', '✅ 已连接！可以聊天啦');
    }

    // ----- 5. 发送消息（点击发送或回车）-----
    function sendMessage() {
        const text = msgInput.value.trim();
        if (!text || !conn) return;
        // 发送给对方
        conn.send(text);
        // 显示在自己的聊天框里（标记为“我”）
        appendMessage('我', text);
        msgInput.value = '';
        msgInput.focus();
    }

    sendBtn.addEventListener('click', sendMessage);
    msgInput.addEventListener('keypress', (e) => {
        if (e.key === 'Enter') sendMessage();
    });

    // ----- 6. 断开连接（点击“断开”按钮）-----
    disconnectBtn.addEventListener('click', () => {
        if (conn) {
            conn.close(); // 会触发 close 事件，自动重置
            resetConnection();
        }
    });

    // ----- 7. 重置界面状态（连接关闭后调用）-----
    function resetConnection() {
        conn = null;
        connectBtn.disabled = false;
        disconnectBtn.disabled = true;
        msgInput.disabled = true;
        sendBtn.disabled = true;
        msgInput.value = '';
    }

    // ----- 8. 在消息区追加一条消息-----
    function appendMessage(who, text) {
        const p = document.createElement('p');
        p.className = 'msg';
        const cls = who === '我' ? 'me' : (who === '系统' ? '' : 'friend');
        if (cls) p.innerHTML = `<span class="${cls}">${who}：</span>${text}`;
        else p.innerHTML = `<span style="color:#888;">${text}</span>`;
        messagesDiv.appendChild(p);
        messagesDiv.scrollTop = messagesDiv.scrollHeight;
    }

    // 额外：页面关闭时清理连接
    window.addEventListener('beforeunload', () => {
        if (conn) conn.close();
        peer.destroy();
    });
</script>
</body>
</html>