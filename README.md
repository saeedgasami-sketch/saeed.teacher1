<!DOCTYPE html>
<html lang="fa" dir="rtl">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>بازی بخش‌پذیری اعداد</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body {
    font-family: Tahoma, sans-serif;
    background: linear-gradient(135deg, #667eea, #764ba2);
    min-height: 100vh;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 20px;
  }
  .game {
    background: #fff;
    border-radius: 20px;
    padding: 30px;
    width: 100%;
    max-width: 480px;
    box-shadow: 0 20px 60px rgba(0,0,0,0.3);
    text-align: center;
  }
  h1 { color: #4a3f8f; margin-bottom: 15px; font-size: 22px; }
  .stats {
    display: flex;
    justify-content: space-around;
    margin-bottom: 20px;
    font-size: 14px;
    color: #555;
  }
  .stats span { font-weight: bold; color: #667eea; }
  .question {
    font-size: 48px;
    font-weight: bold;
    color: #333;
    margin: 20px 0;
    direction: ltr;
  }
  .question small {
    display: block;
    font-size: 18px;
    color: #888;
    margin-bottom: 10px;
    font-weight: normal;
  }
  .buttons {
    display: flex;
    gap: 15px;
    justify-content: center;
    margin: 20px 0;
  }
  button {
    flex: 1;
    padding: 15px;
    font-size: 18px;
    font-family: inherit;
    border: none;
    border-radius: 12px;
    cursor: pointer;
    font-weight: bold;
    transition: transform 0.15s, opacity 0.2s;
  }
  button:hover:not(:disabled) { transform: translateY(-3px); }
  button:disabled { opacity: 0.5; cursor: not-allowed; }
  .yes { background: #2ecc71; color: #fff; }
  .no { background: #e74c3c; color: #fff; }
  .feedback {
    min-height: 50px;
    font-size: 18px;
    font-weight: bold;
    margin-top: 15px;
    display: flex;
    align-items: center;
    justify-content: center;
  }
  .correct { color: #2ecc71; }
  .wrong { color: #e74c3c; }
  .restart {
    background: #667eea;
    color: #fff;
    margin-top: 15px;
    display: none;
    width: 100%;
  }
  .timer {
    height: 8px;
    background: #eee;
    border-radius: 4px;
    overflow: hidden;
    margin-bottom: 10px;
  }
  .timer-bar {
    height: 100%;
    background: linear-gradient(90deg, #2ecc71, #f1c40f, #e74c3c);
    width: 100%;
    transition: width 0.1s linear;
  }
</style>
</head>
<body>
<div class="game">
  <h1>🎯 بازی بخش‌پذیری اعداد</h1>

  <div class="stats">
    <div>امتیاز: <span id="score">0</span></div>
    <div>سوال: <span id="count">0</span></div>
    <div>رکورد: <span id="high">0</span></div>
  </div>

  <div class="timer"><div class="timer-bar" id="timerBar"></div></div>

  <div class="question">
    <small>آیا بخش‌پذیر است؟</small>
    <span id="num">48</span> ÷ <span id="div">6</span>
  </div>

  <div class="buttons">
    <button class="yes" id="btnYes">بله ✅</button>
    <button class="no" id="btnNo">خیر ❌</button>
  </div>

  <div class="feedback" id="feedback"></div>
  <button class="restart" id="btnRestart">🔄 بازی مجدد</button>
</div>

<script>
  const divisors = [2, 3, 4, 5, 6, 8, 9, 10, 11];
  let score = 0, count = 0, high = 0;
  let currentNum, currentDiv, isDivisible, answered, timeLeft, timerId;

  const $ = id => document.getElementById(id);

  function newQuestion() {
    answered = false;
    $('feedback').textContent = '';
    $('btnYes').disabled = false;
    $('btnNo').disabled = false;

    currentDiv = divisors[Math.floor(Math.random() * divisors.length)];

    // ۵۰٪ احتمال بخش‌پذیر بودن
    if (Math.random() < 0.5) {
      const mult = Math.floor(Math.random() * 20) + 2;
      currentNum = currentDiv * mult;
      isDivisible = true;
    } else {
      currentNum = currentDiv * (Math.floor(Math.random() * 20) + 2) + (Math.floor(Math.random() * (currentDiv - 1)) + 1);
      isDivisible = false;
    }

    $('num').textContent = currentNum;
    $('div').textContent = currentDiv;

    // تایمر ۱۰ ثانیه‌ای
    timeLeft = 100;
    $('timerBar').style.width = '100%';
    clearInterval(timerId);
    timerId = setInterval(() => {
      timeLeft -= 1;
      $('timerBar').style.width = timeLeft + '%';
      if (timeLeft <= 0) {
        clearInterval(timerId);
        if (!answered) timeUp();
      }
    }, 100);
  }

  function answer(userSaysYes) {
    if (answered) return;
    answered = true;
    clearInterval(timerId);
    $('btnYes').disabled = true;
    $('btnNo').disabled = true;

    const correct = (userSaysYes === isDivisible);
    if (correct) {
      score += 10;
      $('feedback').textContent = '✅ درست بود!';
      $('feedback').className = 'feedback correct';
    } else {
      $('feedback').textContent = `❌ اشتباه! ${currentNum} ${isDivisible ? 'بخش‌پذیر است' : 'بخش‌پذیر نیست'}`;
      $('feedback').className = 'feedback wrong';
    }

    count++;
    updateStats();
    setTimeout(newQuestion, 1200);
  }

  function timeUp() {
    answered = true;
    $('btnYes').disabled = true;
    $('btnNo').disabled = true;
    $('feedback').textContent = '⏰ وقت تمام شد!';
    $('feedback').className = 'feedback wrong';
    count++;
    updateStats();
    setTimeout(newQuestion, 1200);
  }

  function updateStats() {
    $('score').textContent = score;
    $('count').textContent = count;
    if (score > high) { high = score; $('high').textContent = high; }
  }

  function gameOver() {
    clearInterval(timerId);
    $('btnYes').disabled = true;
    $('btnNo').disabled = true;
    $('btnRestart').style.display = 'block';
    $('feedback').textContent = `🏁 بازی تمام شد! امتیاز: ${score}`;
    $('feedback').className = 'feedback correct';
  }

  $('btnYes').onclick = () => answer(true);
  $('btnNo').onclick = () => answer(false);

  $('btnRestart').onclick = () => {
    score = 0; count = 0;
    updateStats();
    $('btnRestart').style.display = 'none';
    newQuestion();
  };

  // شروع بازی
  newQuestion();

  // پایان بازی بعد از ۱۵ سوال
  const origUpdate = updateStats;
  updateStats = function() {
    origUpdate();
    if (count >= 15) setTimeout(gameOver, 1300);
  };
</script>
</body>
</html>
