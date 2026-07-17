<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>落とし物チェック</title>
    <!-- LINEのLIFFを動かすためのSDKライブラリ -->
    <script src="https://static.line-scdn.net/liff/edge/2/sdk.js"></script>
    <style>
        body { font-family: sans-serif; text-align: center; padding: 50px 20px; }
        .success { color: #06C755; font-weight: bold; }
    </style>
</head>
<body>
    <h1>落とし物チェック（テスト）</h1>
    <div id="user-info">
        <p>LINEログイン情報を読み込み中...</p>
    </div>

    <script>
        // 先ほど発行された、あなたのLIFF ID
        const MY_LIFF_ID = '2010743561-xEkvHtMf';

        async function initializeLiff() {
            try {
                // LIFFの初期化
                await liff.init({ liffId: MY_LIFF_ID });
                
                if (liff.isLoggedIn()) {
                    // ログイン済みの場合はユーザー名を取得して表示
                    const profile = await liff.getProfile();
                    document.getElementById('user-info').innerHTML = `
                        <p class="success">✔ LIFFの起動とログインに成功しました！</p>
                        <p>ようこそ、<strong>${profile.displayName}</strong> さん</p>
                        <p style="font-size: 12px; color: gray;">ユーザーID: ${profile.userId}</p>
                    `;
                } else {
                    // 未ログインの場合はログイン画面へ誘導
                    liff.login();
                }
            } catch (error) {
                document.getElementById('user-info').innerHTML = `
                    <p style="color: red;">エラーが発生しました: ${error.message}</p>
                `;
            }
        }

        initializeLiff();
    </script>
</body>
</html>
