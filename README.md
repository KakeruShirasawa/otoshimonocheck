<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>落とし物検索（ユーザー用）</title>
    <script src="https://static.line-scdn.net/liff/edge/2/sdk.js"></script>
    <style>
        body { font-family: -apple-system, BlinkMacSystemFont, sans-serif; background-color: #f5f5f5; margin: 0; padding: 0; }
        .container { max-width: 600px; margin: 0 auto; background: white; min-height: 100vh; display: flex; flex-direction: column; }
        header { background-color: #06C755; color: white; text-align: center; padding: 15px; font-weight: bold; font-size: 18px; }
        .content { padding: 20px; flex: 1; }
        
        .search-box { background: #eef9f2; padding: 15px; border-radius: 8px; margin-bottom: 20px; border: 1px solid #c8edc9; }
        .form-group { margin-bottom: 10px; }
        label { display: block; font-size: 12px; font-weight: bold; margin-bottom: 4px; color: #555; }
        select, input[type="text"] { width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 5px; box-sizing: border-box; font-size: 14px; }
        .search-btn { background-color: #06C755; color: white; border: none; width: 100%; padding: 12px; border-radius: 5px; font-weight: bold; font-size: 15px; cursor: pointer; margin-top: 5px; }

        .item-card { background: white; border: 1px solid #ddd; border-radius: 8px; overflow: hidden; margin-bottom: 15px; display: flex; }
        .item-img-container { width: 100px; height: 100px; background: #eee; flex-shrink: 0; }
        .item-img { width: 100%; height: 100%; object-fit: cover; }
        .item-info { padding: 12px; flex: 1; display: flex; flex-direction: column; justify-content: space-between; }
        .item-title { font-weight: bold; font-size: 16px; margin: 0 0 5px 0; }
        .item-detail { font-size: 12px; color: #666; margin: 0; }
        .claim-btn { background: #06C755; color: white; border: none; padding: 6px 12px; border-radius: 4px; font-size: 12px; font-weight: bold; cursor: pointer; align-self: flex-end; }
        
        .modal { display: none; position: fixed; top: 0; left: 0; width: 100%; height: 100%; background: rgba(0,0,0,0.5); justify-content: center; align-items: center; z-index: 1000; }
        .modal-content { background: white; padding: 20px; border-radius: 8px; width: 90%; max-width: 400px; }
        textarea { width: 100%; padding: 10px; border: 1px solid #ccc; border-radius: 5px; box-sizing: border-box; font-size: 14px; margin-top: 10px; }
        .modal-btn { background-color: #06C755; color: white; border: none; width: 100%; padding: 12px; border-radius: 5px; font-weight: bold; font-size: 16px; cursor: pointer; margin-top: 10px; }
        .close-btn { background: #ccc; }
        .user-id-box { font-size: 11px; color: #999; text-align: center; padding: 10px; background: #eee; }
    </style>
</head>
<body>

<div class="container">
    <header>落とし物クラウド検索</header>
    <div class="content">
        <div class="search-box">
            <div class="form-group">
                <label>カテゴリを選択</label>
                <select id="search-category">
                    <option value="">すべてのカテゴリ</option>
                    <option value="財布">財布</option>
                    <option value="鍵">鍵・キーホルダー</option>
                    <option value="スマホ">スマートフォン・ガジェット</option>
                    <option value="衣類">衣類・傘・バッグ</option>
                    <option value="その他">その他</option>
                </select>
            </div>
            <div class="form-group">
                <label>キーワード（例：黒、折りたたみ）</label>
                <input type="text" id="search-keyword" placeholder="特徴を入力">
            </div>
            <button class="search-btn" onclick="searchItems()">落とし物を検索する</button>
        </div>

        <h3>検索結果</h3>
        <div id="items-list">
            <p style="text-align:center;color:gray;">条件を指定して「検索する」ボタンを押してください。</p>
        </div>
    </div>
    <div id="my-id-display" class="user-id-box">LINE ID読込中...</div>
</div>

<div id="claim-modal" class="modal">
    <div class="modal-content">
        <h3 style="margin-top:0;">持ち主確認申請</h3>
        <p>悪用防止のため、この落とし物の**「具体的な特徴（ブランド、中身、色など）」**を入力してください。管理者が一致を確認した場合のみ、LINEで引き取り方法を通知します。</p>
        <textarea id="claim-details" placeholder="例：ルイヴィトンの財布で、中に図書カードが入っています。" rows="4" required></textarea>
        <button class="modal-btn" id="submit-claim-btn">申請を送信する</button>
        <button class="modal-btn close-btn" onclick="closeModal()">キャンセル</button>
    </div>
</div>

<script type="module">
    import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
    // whereを外し、orderByのみを使用するように変更
    import { getFirestore, collection, getDocs, query, orderBy, addDoc, serverTimestamp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-firestore.js";

    const MY_LIFF_ID = "2010743561-xEkvHtMf";
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
    let userProfile = null;
    let selectedItemId = null;

    async function initLiff() {
        try {
            await liff.init({ liffId: MY_LIFF_ID });
            if (liff.isLoggedIn()) {
                userProfile = await liff.getProfile();
                document.getElementById('my-id-display').innerText = "あなたのLINEユーザーID: " + userProfile.userId;
            } else {
                liff.login();
            }
        } catch (error) {
            console.error(error);
        }
    }
    initLiff();

    // 検索処理（インデックスエラーを回避するためJavaScript側で絞り込み）
    window.searchItems = async function() {
        const listDiv = document.getElementById("items-list");
        const category = document.getElementById("search-category").value;
        const keyword = document.getElementById("search-keyword").value.trim().toLowerCase();

        listDiv.innerHTML = "<p style='text-align:center;'>検索中...</p>";

        try {
            const q = query(collection(db, "items"), orderBy("createdAt", "desc"));
            const querySnapshot = await getDocs(q);

            if (querySnapshot.empty) {
                listDiv.innerHTML = "<p style='text-align:center;color:gray;'>該当する落とし物は見つかりませんでした。</p>";
                return;
            }

            let html = "";
            let matchCount = 0;

            querySnapshot.forEach((doc) => {
                const item = doc.data();
                
                // カテゴリ絞り込み
                if (category && item.category !== category) return;
                
                // キーワード絞り込み
                if (keyword && !item.publicDesc.toLowerCase().includes(keyword) && !item.location.toLowerCase().includes(keyword)) {
                    return;
                }

                matchCount++;
                html += `
                    <div class="item-card">
                        <div class="item-img-container"><img src="${item.imageUrl || 'https://via.placeholder.com/100?text=No+Image'}" class="item-img"></div>
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

            if (matchCount === 0) {
                listDiv.innerHTML = "<p style='text-align:center;color:gray;'>該当する落とし物は見つかりませんでした。</p>";
            } else {
                listDiv.innerHTML = html;
            }
        } catch (error) {
            console.error(error);
            listDiv.innerHTML = "<p style='color:red;'>検索中にエラーが発生しました。</p>";
        }
    }

    window.openClaimModal = function(itemId) {
        selectedItemId = itemId;
        document.getElementById("claim-modal").style.display = "flex";
    }
    window.closeModal = function() {
        document.getElementById("claim-modal").style.display = "none";
        document.getElementById("claim-details").value = "";
    }

    document.getElementById("submit-claim-btn").addEventListener("click", async () => {
        const details = document.getElementById("claim-details").value;
        if (!details) return;
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
            alert("申請を送信しました！");
            closeModal();
        } catch (error) {
            alert("申請に失敗しました。");
        } finally {
            btn.innerText = "申請を送信する";
            btn.disabled = false;
        }
    });
</script>
</body>
</html>
