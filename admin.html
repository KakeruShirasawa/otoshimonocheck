<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>落とし物管理（スタッフ専用）</title>
    <style>
        body { font-family: -apple-system, BlinkMacSystemFont, sans-serif; background-color: #222; margin: 0; padding: 0; color: #fff; }
        .container { max-width: 600px; margin: 0 auto; background: #333; min-height: 100vh; padding: 20px; box-sizing: border-box; }
        header { background-color: #d9534f; color: white; text-align: center; padding: 10px; font-weight: bold; border-radius: 5px; margin-bottom: 20px; }
        .form-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 5px; font-weight: bold; font-size: 14px; }
        input[type="text"], input[type="password"], select, textarea { width: 100%; padding: 10px; border: 1px solid #555; background: #444; color: #fff; border-radius: 5px; box-sizing: border-box; }
        button { background-color: #d9534f; color: white; border: none; width: 100%; padding: 12px; border-radius: 5px; font-weight: bold; font-size: 16px; cursor: pointer; margin-top: 10px; }
        #admin-content { display: none; }
        .logout-btn { background: #555; font-size: 12px; padding: 5px 10px; width: auto; margin-bottom: 15px; }
    </style>
</head>
<body>

<!-- 🔑 ログイン画面 -->
<div id="login-screen" class="container">
    <header>🛡️ スタッフ専用ログイン</header>
    <form id="login-form">
        <div class="form-group">
            <label>管理者用パスワード</label>
            <input type="password" id="login-pass" placeholder="パスワードを入力してください" required>
        </div>
        <button type="submit">ログイン</button>
    </form>
</div>

<!-- 🛡️ 管理者用メインコンテンツ -->
<div id="admin-content" class="container">
    <button class="logout-btn" onclick="logout()">ログアウト</button>
    <header>🛡️ 落とし物管理システム</header>
    
    <h3>新規落とし物登録</h3>
    <form id="register-form">
        <div class="form-group">
            <label>カテゴリ</label>
            <select id="reg-category" required>
                <option value="財布">財布</option>
                <option value="鍵">鍵・キーホルダー</option>
                <option value="スマホ">スマートフォン・ガジェット</option>
                <option value="衣類">衣類・傘・バッグ</option>
                <option value="その他">その他</option>
            </select>
        </div>
        <div class="form-group">
            <label>公開する特徴（例：黒い二つ折り財布）</label>
            <input type="text" id="reg-public-desc" required>
        </div>
        <div class="form-group">
            <label>拾得場所</label>
            <input type="text" id="reg-location" required>
        </div>
        <div class="form-group" style="background: #5c4815; padding: 10px; border-radius: 5px;">
            <label>🔒 隠す特徴（ブランド名、中身の金額など）</label>
            <textarea id="reg-secret-desc" required rows="3"></textarea>
        </div>
        <div class="form-group">
            <label>写真</label>
            <input type="file" id="reg-file" accept="image/*" required>
        </div>
        <button type="submit" id="submit-btn">落とし物を登録する</button>
    </form>
</div>

<script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
    import { getFirestore, collection, addDoc, serverTimestamp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-firestore.js";

    // 🔒【設定】スタッフ共通のパスワード（自由に変更してください）
    const ADMIN_PASSWORD = "1234";

    const firebaseConfig = {
        apiKey: "AIzaSyAt0pdCitCLAApHouD5XBA47ubtUekTjMA",
        authDomain: "otoshimonocheck.firebaseapp.com",
        projectId: "otoshimonocheck",
        storageBucket: "otoshimonocheck.firebasestorage.app",
        messagingSenderId: "626224212883",
        appId: "1:626224212883:web:310b83089938d498edd666"
    };

    const app = initializeApp(firebaseConfig);
    const db = getFirestore(app);

    // ログイン状態の確認（ブラウザを閉じるまで記憶）
    if (sessionStorage.getItem("isAdminLoggedIn") === "true") {
        showAdmin();
    }

    // ログイン処理
    document.getElementById("login-form").addEventListener("submit", (e) => {
        e.preventDefault();
        const pass = document.getElementById("login-pass").value;
        if (pass === ADMIN_PASSWORD) {
            sessionStorage.setItem("isAdminLoggedIn", "true");
            showAdmin();
        } else {
            alert("パスワードが違います。");
        }
    });

    function showAdmin() {
        document.getElementById("login-screen").style.display = "none";
        document.getElementById("admin-content").style.display = "block";
    }

    window.logout = function() {
        sessionStorage.removeItem("isAdminLoggedIn");
        location.reload();
    }

    // 登録処理
    document.getElementById("register-form").addEventListener("submit", async (e) => {
        e.preventDefault();
        const submitBtn = document.getElementById("submit-btn");
        submitBtn.innerText = "登録中...";
        submitBtn.disabled = true;

        const category = document.getElementById("reg-category").value;
        const publicDesc = document.getElementById("reg-public-desc").value;
        const location = document.getElementById("reg-location").value;
        const secretDesc = document.getElementById("reg-secret-desc").value;
        const fileInput = document.getElementById("reg-file");

        let imageUrl = "";
        if (fileInput.files.length > 0) {
            imageUrl = await compressImage(fileInput.files[0]);
        }

        try {
            await addDoc(collection(db, "items"), {
                category,
                publicDesc,
                location,
                secretDesc,
                imageUrl,
                status: "保管中",
                createdAt: serverTimestamp()
            });
            alert("落とし物を登録しました！");
            document.getElementById("register-form").reset();
        } catch (error) {
            alert("登録失敗: " + error.message);
        } finally {
            submitBtn.innerText = "落とし物を登録する";
            submitBtn.disabled = false;
        }
    });

    function compressImage(file) {
        return new Promise((resolve) => {
            const reader = new FileReader();
            reader.readAsDataURL(file);
            reader.onload = (event) => {
                const img = new Image();
                img.src = event.target.result;
                img.onload = () => {
                    const canvas = document.createElement('canvas');
                    const MAX_WIDTH = 400;
                    const MAX_HEIGHT = 400;
                    let width = img.width;
                    let height = img.height;
                    if (width > height) {
                        if (width > MAX_WIDTH) { height *= MAX_WIDTH / width; width = MAX_WIDTH; }
                    } else {
                        if (height > MAX_HEIGHT) { width *= MAX_HEIGHT / height; height = MAX_HEIGHT; }
                    }
                    canvas.width = width;
                    canvas.height = height;
                    const ctx = canvas.getContext('2d');
                    ctx.drawImage(img, 0, 0, width, height);
                    resolve(canvas.toDataURL('image/jpeg', 0.6));
                };
            };
        });
    }
</script>
</body>
</html>
