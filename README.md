انا قلت تخلي <!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>imo Legend Pro</title>
    <!-- تضمين حزم Firebase المتوافقة المستقرة لضمان عمل السيرفر -->
    <script src="https://gstatic.com"></script>
    <script src="https://gstatic.com"></script>
    <style>
        @import url('https://googleapis.com');
        
        body { font-family: 'Cairo', sans-serif; background-color: #f4f7f6; color: #111111; margin: 0; padding: 0; display: flex; flex-direction: column; height: 100vh; justify-content: space-between; overflow: hidden; }
        .hidden { display: none !important; }
        
        /* تصميم واجهة الدخول */
        #auth-screen { position: fixed; inset: 0; background: #0084ff; z-index: 50; display: flex; flex-direction: column; items-center: center; justify-content: center; padding: 20px; align-items: center; }
        .auth-box { background: white; padding: 30px; border-radius: 15px; box-shadow: 0 10px 25px rgba(0,0,0,0.2); width: 100%; max-width: 350px; text-align: center; }
        .logo { height: 80px; background: #0084ff; color: white; border-radius: 50%; display: flex; align-items: center; justify-content: center; font-size: 32px; font-weight: 900; margin: 0 auto 15px; width: 80px; }
        .input-field { width: 90%; background: #f9f9f9; border: 1px solid #e2e2e2; border-radius: 10px; padding: 12px; text-align: center; outline: none; margin-bottom: 15px; font-weight: 600; }
        .btn-blue { width: 98%; background: #0084ff; color: white; font-weight: bold; padding: 12px; border-radius: 10px; border: none; cursor: pointer; }
        .btn-green { width: 98%; background: #00a884; color: white; font-weight: bold; padding: 12px; border-radius: 10px; border: none; cursor: pointer; }
        
        /* واجهة غرفة الدردشة */
        #app-screen { display: flex; flex-direction: column; height: 100%; width: 100%; }
        header { background: #0084ff; color: white; padding: 15px; display: flex; justify-content: space-between; align-items: center; box-shadow: 0 2px 5px rgba(0,0,0,0.1); }
        #chat-messages { flex: 1; overflow-y: auto; padding: 15px; background-color: #efeae2; display: flex; flex-direction: column; }
        
        /* تصميم الرسائل وتحديد مكانها الجانبي */
        .msg-container { display: flex; width: 100%; margin: 4px 0; }
        .msg-container.me { justify-content: flex-start; }
        .msg-container.other { justify-content: flex-end; }
        .msg-box { padding: 10px 15px; border-radius: 12px; max-width: 75%; box-shadow: 0 1px 2px rgba(0,0,0,0.1); text-align: right; }
        .msg-me { background: #d9fdd3; border-top-right-radius: 0; }
        .msg-other { background: white; border-top-left-radius: 0; }
        
        .chat-input-bar { background: white; padding: 10px; border-top: 1px solid #e2e2e2; display: flex; gap: 10px; align-items: center; }
        #message-input { flex: 1; background: #f1f1f1; border: 1px solid transparent; padding: 12px; border-radius: 10px; outline: none; text-align: right; }
        .btn-send { background: #0084ff; color: white; border: none; padding: 12px 25px; font-weight: bold; border-radius: 10px; cursor: pointer; }
    </style>
</head>
<body>

    <!-- 1. واجهة التحقق والدخول الرقمي السري -->
    <div id="auth-screen">
        <div class="auth-box">
            <div class="logo">imo</div>
            <h2 style="color:#0084ff; margin:0;">imo Legend</h2>
            <p style="color:gray; font-size:12px; margin-bottom:20px;">مرحباً بك في النسخة المطورة</p>
            
            <div id="phone-form">
                <input type="text" id="phone-input" placeholder="رقم الهاتف (مثال: +967...)" class="input-field">
                <input type="text" id="username-input" placeholder="اكتب اسمك هنا" class="input-field">
                <button onclick="requestOTP()" class="btn-blue">متابعة وإرسال الرمز</button>
            </div>

            <div id="otp-form" class="hidden">
                <p style="color:#00a884; font-size:12px; font-weight: bold; margin-bottom: 10px;">رمز التحقق التجريبي هو: 123456</p>
                <input type="text" id="otp-input" placeholder="أدخل الرمز (6 أرقام)" class="input-field">
                <button onclick="verifyOTP()" class="btn-green">تأكيد تسجيل الدخول</button>
            </div>
        </div>
    </div>

    <!-- 2. الواجهة الرئيسية للتطبيق والشات المباشر -->
    <div id="app-screen" class="hidden">
        <header>
            <div style="display:flex; align-items:center; gap:10px;">
                <span style="font-weight:900; background:white; color:#0084ff; padding:2px 6px; border-radius:5px;">imo</span>
                <span style="font-weight:bold;">الدردشة العامة</span>
                <span id="current-user-display" style="font-size:12px; background:#005cff; padding:2px 8px; border-radius:10px;"></span>
            </div>
            <button onclick="logout()" style="background:#ff4d4d; color:white; border:none; padding:6px 12px; border-radius:5px; cursor:pointer;">خروج</button>
        </header>

        <div id="chat-messages">
            <!-- رسائل المستخدمين النشطة تظهر هنا -->
        </div>

        <div class="chat-input-bar">
            <input type="text" id="message-input" placeholder="اكتب رسالة للمجموعة..." onkeypress="checkEnter(event)">
            <button onclick="sendMessage()" class="btn-send">إرسال</button>
        </div>
    </div>

    <script>
        // إعدادات الـ Config المحدثة والمعدلة بناءً على حسابك الخاص بدقة
        const firebaseConfig = {
            apiKey: "AIzaSyCBkGggHnBsLaSi0RH211cfbCFDBijH4BU",
            authDomain: "://firebaseapp.com",
            databaseURL: "https://firebaseio.com", // تم ربط وضبط الرابط الدقيق لسيرفرك
            projectId: "imo-legend",
            storageBucket: "imo-legend.firebasestorage.app",
            messagingSenderId: "930356255753",
            appId: "1:930356255753:web:bfca539599adbea4df6cd8",
            measurementId: "G-5Y077GRTS9"
        };

        let db;
        try {
            firebase.initializeApp(firebaseConfig);
            db = firebase.database();
        } catch (error) {
            console.error(error);
        }

        let tempPhone = '';
        let tempUsername = '';

        function requestOTP() {
            const phoneNumber = document.getElementById('phone-input').value.trim();
            const username = document.getElementById('username-input').value.trim();

            if (!phoneNumber || !username) {
                alert('أدخل البيانات كاملة للبدء!');
                return;
            }

            tempPhone = phoneNumber;
            tempUsername = username;

            document.getElementById('phone-form').classList.add('hidden');
            document.getElementById('otp-form').classList.remove('hidden');
        }

        function verifyOTP() {
            const code = document.getElementById('otp-input').value.trim();
            
            if (code === '123456') {
                localStorage.setItem('imo_username', tempUsername);
                localStorage.setItem('imo_phone', tempPhone);
                
                document.getElementById('auth-screen').classList.add('hidden');
                document.getElementById('app-screen').classList.remove('hidden');
                document.getElementById('current-user-display').innerText = tempUsername;
                
                loadMessages();
            } else {
                alert('الرمز التجريبي الصحيح هو 123456');
            }
        }

        function sendMessage() {
            const text = document.getElementById('message-input').value;
            const username = localStorage.getItem('imo_username') || 'مستخدم';
            if (!text.trim() || !db) return;

            // الربط البرمجي مع مجلد chats المسجل في مشروعك
            db.ref('chats').push({
                username: username,
                text: text,
                timestamp: Date.now()
            });
            document.getElementById('message-input').value = '';
        }

        function checkEnter(e) {
            if (e.key === 'Enter') sendMessage();
        }

        function loadMessages() {
            const chatBox = document.getElementById('chat-messages');
            if (!db) return;

            db.ref('chats').limitToLast(50).on('value', (snapshot) => {
                chatBox.innerHTML = '';
                snapshot.forEach((childSnapshot) => {
                    const msg = childSnapshot.val();
                    
                    const msgContainer = document.createElement('div');
                    const msgDiv = document.createElement('div');
                    
                    const currentUser = localStorage.getItem('imo_username');
                    if (msg.username === currentUser) {
                        msgContainer.className = "msg-container me";
                        msgDiv.className = "msg-box msg-me";
                    } else {
                        msgContainer.className = "msg-container other";
                        msgDiv.className = "msg-box msg-other";
                    }
                    
                    msgDiv.innerHTML = `<span style="color:#0084ff; font-weight:bold; font-size:10px; display:block;">${msg.username}</span><span style="font-size:14px; font-weight: 600;">${msg.text}</span>`;
                    msgContainer.appendChild(msgDiv);
                    chatBox.appendChild(msgContainer);
                });
                chatBox.scrollTop = chatBox.scrollHeight;
            });
        }

        function logout() {
            localStorage.removeItem('imo_username');
            localStorage.removeItem('imo_phone');
            location.reload();
        }
