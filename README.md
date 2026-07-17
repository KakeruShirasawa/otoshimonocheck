<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>落とし物チェック</title>
    <!-- LINE LIFF SDK -->
    <script src="https://static.line-scdn.net/liff/edge/2/sdk.js"></script>
    <style>
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; background-color: #f5f5f5; margin: 0; padding: 0; color: #333; }
        .container { max-width: 600px; margin: 0 auto; background: white; min-height: 100vh; box-shadow: 0 0 10px rgba(0,0,0,0.1); display: flex; flex-direction: column; }
        header { background-color: #06C755; color: white; text-align: center; padding: 15px; font-weight: bold; font-size: 18px; }
        .tabs { display: flex; background: #eee; border-bottom: 1px solid #ddd; }
        .tab { flex: 1; text-align: center; padding: 12px; cursor: pointer; font-weight: bold; color: #666; }
        .tab.active { background: white; color: #06C755; border-bottom: 3px solid #06C755; }
        .content { padding: 20px; flex: 1; }
        .page { display: none; }
        .page.active { display: block; }
        
        /* フォームのデザイン */
        .form-group { margin-bottom: 15px; }
        label { display: block; margin-bottom: 5px; font-weight: bold; font-size: 14px; }
        input[type="text"], select, textarea { width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 5px; box-sizing: border-box; font-size: 14px; }
        input[type="file"] { margin-top: 5px; }
        button { background-color: #06C755; color: white; border: none; width: 100%; padding: 12px; border-radius: 5px; font-weight: bold; font-size: 16px; cursor: pointer; margin-top: 10px; }
        button:active { background-color: #05a848; }
        
        /* 落とし物カードのデザイン */
        .item-card { background: white; border: 1px solid #ddd; border-radius: 8px; overflow: hidden; margin-bottom: 15px; display: flex; }
        .item-img-container { width: 100px; height: 100px; background: #eee; flex-shrink: 0; }
        .item-img { width: 100%; height: 100%; object-fit: cover; }
        .item-info { padding: 12px; flex: 1; display: flex; flex-direction: column; justify-content: space-between; }
        .item-title { font-weight: bold; font-size: 16px; margin: 0 0 5px 0; }
        .item-detail { font-size: 12px; color: #666; margin: 0; }
        .claim-btn { background: #06C755; color: white; border: none; padding: 6px 12px; border-radius: 4px; font-size: 12px; font-weight: bold; cursor: pointer; align-self: flex-end; width: auto; margin: 0; }

        /* モーダル（ポップアップ） */
        .modal { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.5); justify-content: center; align-items: center; z-index: 1000; }
        .modal-content { background: white; padding: 20px; border-radius: 8px; width: 90%; max-width: 400px; }
        .close-btn { background: #ccc; margin-top: 5px; }
    </style>
</head>
<body>

<div class="container">
    <header>落とし物クラウド検索</header>
    
    <!-- 管理者と一般ユーザーを切り替えるタブ -->
    <div class="tabs">
        <div class="tab active" onclick="switchTab('user')">探す（ユーザー画面）</div>
        <div class="tab" onclick="switchTab('admin')">登録（管理者画面）</div>
    </div>

    <div class="content">
        <!-- ユーザー画面 -->
        <div id="page-user" class="page active">
            <h3>落とし物一覧</h3>
            <div id="items-list">読み込み中...</div>
        </div>

        <!-- 管理者画面 -->
        <div id="page-admin" class="page">
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
                    <input type="text" id="reg-public-desc" placeholder="誰でも見られる大まかな特徴" required>
                </div>
                <div class="form-group">
                    <label>拾得場所</label>
                    <input type="text" id="reg-location" placeholder="例：ステージA付近" required>
                </div>
                <div class="form-group" style="background: #fff3cd; padding: 10px; border-radius: 5px;">
                    <label style="color: #856404;">⚠️ 隠す特徴（不正防止用・例：ブランド名、中身の金額）</label>
                    <textarea id="reg-secret-desc" placeholder="持ち主しか知り得ない具体的な特徴" required rows="3"></textarea>
                </div>
                <div class="form-group">
                    <label>写真（カメラ起動可）</label>
                    <input type="file" id="reg-file" accept="image/*" required>
                </div>
                <button type="submit" id="submit-btn">登録する</button>
            </form>
        </div>
    </div>
</div>

<!-- 申請用ポップアップ -->
<div id="claim-modal" class="modal">
    <div class="modal-content">
        <h3 style="margin-top:0;">持ち主確認申請</h3>
        <p>悪用防止のため、この落とし物の**「具体的な特徴（ブランド、中身、色など）」**を入力してください。管理者が確認し、一致した場合のみLINEで引き取り方法を通知します。</p>
        <div class="form-group">
            <textarea id="claim-details" placeholder="例：ルイヴィトンの財布で、中に図書カードが入っています。" rows="4" required></textarea>
        </div>
        <button id="submit-claim-btn">申請を送信する</button>
        <button class="close-btn" onclick="closeModal()">キャンセル</button>
    </div>
</div>

<script type="module">
    // Firebase SDKの読み込み
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
    import { getFirestore, collection, addDoc, getDocs, query, orderBy, serverTimestamp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-firestore.js";

    // LIFF ID と Firebase接続キー（設定済み）
    const MY_LIFF_ID = "2010743561-xEkvHtMf";

    const firebaseConfig = {
        apiKey: "AIzaSyAt0pdCitCLAApHouD5XBA47ubtUekTjMA",
        authDomain: "otoshimonocheck.firebaseapp.com",
        projectId: "otoshimonocheck",
        storageBucket: "otoshimonocheck.firebasestorage.app",
        messagingSenderId: "626224212883",
        appId: "1:626224212883:web:310b83089938d498edd666"
    };

    // Firebaseの初期化
    const app = initializeApp(firebaseConfig);
    const db = getFirestore(app);

    let userProfile = null;
    let selectedItemId = null;

    // LIFFの初期化
    async function initLiff() {
        try {
            await liff.init({ liffId: MY_LIFF_ID });
            if (liff.isLoggedIn()) {
                userProfile = await liff.getProfile();
                loadItems();
            } else {
                liff.login();
            }
        } catch (error) {
            console.error("LIFF初期化エラー:", error);
            loadItems();
        }
    }
    initLiff();

    // タブの切り替え
    window.switchTab = function(tabName) {
        document.querySelectorAll('.tab').forEach(t => t.classList.remove('active'));
        document.querySelectorAll('.page').forEach(p => p.classList.remove('active'));
        
        if (tabName === 'user') {
            document.querySelectorAll('.tab')[0].classList.add('active');
            document.getElementById('page-user').classList.add('active');
            loadItems();
        } else {
            document.querySelectorAll('.tab')[1].classList.add('active');
            document.getElementById('page-admin').classList.add('active');
        }
    }

    // 落とし物データの読み込み
    async function loadItems() {
        const listDiv = document.getElementById("items-list");
        listDiv.innerHTML = "読み込み中...";

        try {
            const q = query(collection(db, "items"), orderBy("createdAt", "desc"));
            const querySnapshot = await getDocs(q);
            
            if (querySnapshot.empty) {
                listDiv.innerHTML = "<p style='text-align:center;color:gray;'>現在、登録されている落とし物はありません。</p>";
                return;
            }

            let html = "";
            querySnapshot.forEach((doc) => {
                const item = doc.data();
                html += `
                    <div class="item-card">
                        <div class="item-img-container">
                            <img src="${item.imageUrl || 'https://via.placeholder.com/100?text=No+Image'}" class="item-img">
                        </div>
                        <div class="item-info">
                            <div>
                                <h4 class="item-title">[${item.category}] ${item.publicDesc}</h4>
                                <p class="item-detail">📍 場所: ${item.location}</p>
                            </div>
                            <button class="claim-btn" onclick="openClaimModal('${doc.id}')">これ私の！</button>
                        </div>
                    </div>
                `;
            });
            listDiv.innerHTML = html;
        } catch (error) {
            console.error("読み込みエラー:", error);
            listDiv.innerHTML = "<p style='color:red;'>データの読み込みに失敗しました。</p>";
        }
    }

    // 落とし物の新規登録（管理者）
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
            switchTab('user');
        } catch (error) {
            console.error("登録エラー:", error);
            alert("登録に失敗しました: " + error.message);
        } finally {
            submitBtn.innerText = "登録する";
            submitBtn.disabled = false;
        }
    });

    // 画像圧縮処理 (Base64変換)
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
                        if (width > MAX_WIDTH) {
                            height *= MAX_WIDTH / width;
                            width = MAX_WIDTH;
                        }
                    } else {
                        if (height > MAX_HEIGHT) {
                            width *= MAX_HEIGHT / height;
                            height = MAX_HEIGHT;
                        }
                    }
                    canvas.width = width;
                    canvas.height = height;
                    const ctx = canvas.getContext('2d');
                    ctx.drawImage(img, 0, 0, width, height);
                    const dataUrl = canvas.toDataURL('image/jpeg', 0.6);
                    resolve(dataUrl);
                };
            };
        });
    }

    // 申請ポップアップを開く
    window.openClaimModal = function(itemId) {
        selectedItemId = itemId;
        document.getElementById("claim-modal").style.display = "flex";
    }

    // 申請ポップアップを閉じる
    window.closeModal = function() {
        document.getElementById("claim-modal").style.display = "none";
        document.getElementById("claim-details").value = "";
    }

    // 持ち主申請の送信処理
    document.getElementById("submit-claim-btn").addEventListener("click", async () => {
        const details = document.getElementById("claim-details").value;
        if (!details) {
            alert("特徴を入力してください。");
            return;
        }

        const btn = document.getElementById("submit-claim-btn");
        btn.innerText = "送信中...";
        btn.disabled = true;

        try {
            await addDoc(collection(db, "claims"), {
                itemId: selectedItemId,
                userId: userProfile ? userProfile.userId : "test-user-id",
                userName: userProfile ? userProfile.displayName : "テストユーザー",
                claimDetails: details,
                status: "未確認",
                createdAt: serverTimestamp()
            });

            alert("申請を送信しました。管理者が内容を確認するまでしばらくお待ちください。");
            closeModal();
        } catch (error) {
            console.error("申請送信エラー:", error);
            alert("申請に失敗しました。");
        } finally {
            btn.innerText = "申請を送信する";
            btn.disabled = false;
        }
    });
</script>
</body>
</html>
