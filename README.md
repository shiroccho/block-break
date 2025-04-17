# block-break
# ブロック崩しゲーム解説

## 概要

このブロック崩しゲームは、JavaScriptとHTML5 Canvasを使って実装された古典的なアーケードゲームです。プレイヤーはパドルを操作してボールを打ち返し、画面上部のブロックを壊していくゲームです。

## 基本機能

### 1. ゲームの基本要素

- **ボール**: 画面内を跳ね回り、ブロックやパドルに当たると反射します
- **パドル**: プレイヤーが左右に動かしてボールを打ち返す台です
- **ブロック**: 画面上部に配置され、ボールが当たると破壊されるオブジェクトです
- **残機**: プレイヤーは3回までボールを落とすことができます
- **スコア**: ブロックを破壊するごとに10点獲得します

### 2. レベルシステム

- すべてのブロックを破壊すると、次のレベルに進みます
- レベルが上がるごとにブロックの耐久力が増加します
- レベルごとに異なる色のブロックが登場します
- スコアは引き継がれ、累積されていきます

### 3. 耐久力システム

- レベル1: 青色のブロック、耐久力1
- レベル2: 緑色のブロック、耐久力2
- レベル3: オレンジ色のブロック、耐久力3
- レベル4: ピンク色のブロック、耐久力4
- レベル5以降: 紫色のブロック、耐久力5
- 耐久力はブロック内の数字で表示されます
- 残り耐久力に応じてブロックの色が徐々に明るくなります

### 4. 難易度の変化

- レベルが上がるとボールの速度が増加します
- ブロックの耐久力が上がるため、壊すのにより多くのヒットが必要になります
- 高レベルになるほど、同じ時間内でより多くのブロックを壊す必要があります

### 5. ハイスコアシステム

- ゲーム終了時にハイスコアが更新された場合、プレイヤー名を入力できます
- 上位5名のハイスコアがローカルストレージに保存され、表示されます
- ハイスコアリセットボタンでスコアをクリアできます

## 技術的実装

### HTML5要素

- **Canvas**: ゲームの描画にHTML5 Canvasを使用しています
- **DOM要素**: スコア、レベル、残機表示などにHTMLとCSSを使用しています
- **ローカルストレージ**: ハイスコアの保存にWebブラウザのローカルストレージを使用しています

### JavaScriptの主要機能

#### メインゲームループ

```javascript
function draw() {
    // キャンバスをクリア
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    
    // 各要素の描画
    drawBricks();
    drawBall();
    drawPaddle();
    drawScore();
    drawLives();
    drawLevel();
    collisionDetection();
    
    // ボールの移動とパドルとの衝突検出
    // 壁との衝突検出
    // ゲームオーバー判定
    
    // アニメーションの継続
    if (!gamePaused) {
        x += dx;
        y += dy;
        requestAnimationFrame(draw);
    }
}
```

#### 衝突検出システム

```javascript
function collisionDetection() {
    for (let c = 0; c < brickColumnCount; c++) {
        for (let r = 0; r < brickRowCount; r++) {
            const brick = bricks[c][r];
            if (brick.status > 0) {
                if (x > brick.x && x < brick.x + brickWidth && 
                    y > brick.y && y < brick.y + brickHeight) {
                    dy = -dy;
                    brick.status -= 1;
                    score += 10;
                    
                    // レベルクリア判定
                    if (checkAllBricksDestroyed()) {
                        nextLevel();
                    }
                }
            }
        }
    }
}
```

#### レベルシステムの実装

```javascript
function nextLevel() {
    level++;
    
    // ボールとパドルをリセット
    x = canvas.width / 2;
    y = canvas.height - 30;
    
    // レベルに応じてボール速度を上げる
    dx = (dx > 0) ? 4 + level * 0.5 : -4 - level * 0.5;
    dy = -4 - level * 0.5;
    
    // ブロック再初期化
    initBricks();
    
    // レベルアップメッセージ
    alert(`レベル ${level} へ！ブロックの耐久力が上がりました！`);
}
```

#### ブロックの耐久力表示

```javascript
function drawBricks() {
    // 現在のレベルに応じた色の設定
    const levelIndex = Math.min(level - 1, levelConfig.length - 1);
    const baseColor = levelConfig[levelIndex].color;
    
    for (let c = 0; c < brickColumnCount; c++) {
        for (let r = 0; r < brickRowCount; r++) {
            const brick = bricks[c][r];
            if (brick.status > 0) {
                // 耐久力に応じた色の変化
                const durabilityRatio = brick.status / brick.maxDurability;
                // 色を計算...
                
                // ブロック描画
                ctx.beginPath();
                ctx.rect(brickX, brickY, brickWidth, brickHeight);
                ctx.fillStyle = newColorHex;
                ctx.fill();
                
                // 耐久力の数字表示
                if (brick.maxDurability > 1) {
                    ctx.fillText(brick.status.toString(), brickX + brickWidth / 2, brickY + brickHeight / 2);
                }
                
                ctx.closePath();
            }
        }
    }
}
```

#### ハイスコアシステム

```javascript
function checkHighscore(newScore) {
    const isHighscore = highscores.length < maxHighscores || newScore > highscores[highscores.length - 1].score;
    
    if (isHighscore) {
        const name = prompt("新しいハイスコア！名前を入力してください:", "プレイヤー");
        
        // ハイスコア保存処理
        highscores.push({ name: playerName, score: newScore });
        highscores.sort((a, b) => b.score - a.score);
        
        if (highscores.length > maxHighscores) {
            highscores = highscores.slice(0, maxHighscores);
        }
        
        saveHighscores();
    }
}
```

## 操作方法

- **マウス**: パドルをマウスで左右に移動
- **キーボード**: 左右の矢印キーでパドルを移動
- **ゲームスタート**: スタートボタンでゲーム開始
- **ハイスコアリセット**: リセットボタンでハイスコアをクリア

## プレイの流れ

1. 「ゲームスタート」ボタンを押してゲームを開始
2. パドルを操作してボールを打ち返し、ブロックを破壊
3. すべてのブロックを破壊すると次のレベルへ進む
4. レベルが上がるとブロックの耐久力が増加し、色が変わる
5. ボールを落とすと残機が減る（初期残機は3）
6. 残機がなくなるとゲームオーバー
7. ハイスコア達成時は名前を入力

## おわりに

このブロック崩しゲームは、古典的なアーケードゲームに現代的な要素を加えた実装です。レベルシステムとブロックの耐久力の概念を導入することで、プレイヤーに継続的な挑戦を提供します。HTML5とJavaScriptの機能を活用し、ブラウザ上で軽快に動作する設計になっています。
