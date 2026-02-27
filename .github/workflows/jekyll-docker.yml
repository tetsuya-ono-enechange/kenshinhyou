<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>検針票OCR・カメラ起動デモ</title>
    <style>
        :root { --main-blue: #1a73e8; --error-red: #d93025; }
        body { font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif; background-color: #f0f2f5; margin: 0; padding: 20px; color: #333; }
        .container { max-width: 500px; margin: 0 auto; }
        .card { background: white; padding: 20px; border-radius: 16px; box-shadow: 0 4px 12px rgba(0,0,0,0.1); margin-bottom: 20px; }
        h2 { font-size: 1.2rem; margin-top: 0; color: var(--main-blue); border-bottom: 2px solid #eef2f7; padding-bottom: 10px; }
        
        /* カメラボタンのスタイル */
        .camera-btn-wrapper { text-align: center; margin: 20px 0; }
        #imageInput { display: none; } /* デフォルトのボタンは隠す */
        .custom-file-upload {
            background-color: var(--main-blue); color: white; padding: 15px 30px;
            border-radius: 50px; cursor: pointer; display: inline-block; font-weight: bold;
            box-shadow: 0 4px 6px rgba(0,0,0,0.2); transition: 0.2s;
        }
        .custom-file-upload:active { transform: scale(0.95); }

        /* 入力フォームのスタイル */
        .form-group { margin-top: 20px; }
        label { display: block; font-size: 0.9rem; font-weight: bold; margin-bottom: 8px; color: #555; }
        .result-input {
            width: 100%; padding: 12px; font-size: 1.1rem; border: 2px solid #ddd;
            border-radius: 8px; box-sizing: border-box; transition: 0.3s;
        }
        .result-input.success { border-color: #28a745; background-color: #f8fff9; }

        /* ステータス表示 */
        #loading { display: none; text-align: center; font-weight: bold; color: var(--main-blue); margin: 10px 0; }
        .error { color: var(--error-red); background: #fce8e6; padding: 10px; border-radius: 8px; font-size: 0.85rem; display: none; margin-top: 10px; }
        
        #preview { width: 100%; border-radius: 8px; margin-top: 15px; display: none; border: 1px solid #ddd; }
        pre { background: #333; color: #0f0; padding: 10px; border-radius: 8px; font-size: 0.75rem; overflow-x: auto; max-height: 150px; }
    </style>
</head>
<body>

<div class="container">
    <div class="card">
        <h2>検針票を読み取る</h2>
        <p style="font-size: 0.85rem; color: #666;">ボタンを押してカメラで検針票を撮影してください。</p>
        
        <div class="camera-btn-wrapper">
            <label for="imageInput" class="custom-file-upload">
                📷 検針票を撮影する
            </label>
            <input type="file" id="imageInput" accept="image/*" capture="environment">
        </div>

        <div id="loading">⏳ AIが解析中...</div>
        <div id="errorArea" class="error"></div>
        <img id="preview" alt="プレビュー">
    </div>

    <div class="card">
        <h2>解析結果</h2>
        <div class="form-group">
            <label>供給地点特定番号 (22桁)</label>
            <input type="text" id="locNumber" class="result-input" placeholder="自動入力されます" readonly>
        </div>
    </div>

    <div class="card" style="background: #eef2f7;">
        <h2 style="color: #666; font-size: 0.9rem;">デバッグ情報</h2>
        <pre id="rawText">ここに読み取った全テキストが表示されます</pre>
    </div>
</div>

<script>
    // 提供されたAPIキー
    const API_KEY = 'AIzaSyAAtc3_OIUQOCFbLaW6KoOhH0CUhfqMxVw';

    const imageInput = document.getElementById('imageInput');
    const loading = document.getElementById('loading');
    const errorArea = document.getElementById('errorArea');
    const locNumber = document.getElementById('locNumber');
    const rawText = document.getElementById('rawText');
    const preview = document.getElementById('preview');

    imageInput.addEventListener('change', async (e) => {
        const file = e.target.files[0];
        if (!file) return;

        // UIリセット
        errorArea.style.display = 'none';
        loading.style.display = 'block';
        locNumber.classList.remove('success');
        locNumber.value = "";
        rawText.innerText = "通信中...";

        // プレビュー表示
        const reader = new FileReader();
        reader.onload = (f) => {
            preview.src = f.target.result;
            preview.style.display = 'block';
        };
        reader.readAsDataURL(file);

        try {
            const base64Image = await toBase64(file);
            
            const response = await fetch(`https://vision.googleapis.com/v1/images:annotate?key=${API_KEY}`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify({
                    requests: [{
                        image: { content: base64Image.split(',')[1] },
                        features: [{ type: 'DOCUMENT_TEXT_DETECTION' }]
                    }]
                })
            });

            const data = await response.json();

            if (data.error) {
                throw new Error(`Google APIエラー: ${data.error.message}`);
            }

            const result = data.responses[0];

            if (result && result.fullTextAnnotation) {
                const text = result.fullTextAnnotation.text;
                rawText.innerText = text;

                // 22桁の数字を抽出（ハイフンやスペースを除去）
                const cleanText = text.replace(/[\s\n-]/g, '');
                const match = cleanText.match(/\d{22}/);

                if (match) {
                    locNumber.value = match[0];
                    locNumber.classList.add('success');
                } else {
                    errorArea.innerText = "22桁の番号が見つかりませんでした。文字がはっきり写るように撮り直してください。";
                    errorArea.style.display = 'block';
                }
            } else {
                throw new Error("画像から文字を読み取れませんでした。");
            }

        } catch (err) {
            errorArea.innerText = "エラー: " + err.message;
            errorArea.style.display = 'block';
            console.error(err);
        } finally {
            loading.style.display = 'none';
        }
    });

    const toBase64 = file => new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = () => resolve(reader.result);
        reader.onerror = error => reject(error);
    });
</script>

</body>
</html>
