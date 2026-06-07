<!DOCTYPE html>
<html lang="ko">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>끝말잇기 게임</title>
<style>
  * { box-sizing: border-box; margin: 0; padding: 0; }
  body { font-family: 'Noto Sans KR', sans-serif; background: #f5f5f0; display: flex; justify-content: center; padding: 2rem 1rem; min-height: 100vh; }
  .container { background: white; border-radius: 16px; padding: 2rem; max-width: 600px; width: 100%; box-shadow: 0 2px 12px rgba(0,0,0,0.08); }
  h1 { font-size: 22px; font-weight: 600; margin-bottom: 0.25rem; }
  .level-badge { display: inline-block; font-size: 12px; padding: 2px 10px; border-radius: 999px; background: #EEEDFE; color: #3C3489; margin-left: 8px; vertical-align: middle; }
  .stats { display: flex; gap: 12px; margin: 1.25rem 0; }
  .stat-card { flex: 1; background: #f5f5f0; border-radius: 10px; padding: 10px 14px; text-align: center; }
  .stat-card .label { font-size: 12px; color: #888; margin-bottom: 4px; }
  .stat-card .val { font-size: 22px; font-weight: 600; color: #111; }
  .hint-box { background: #f5f5f0; border-radius: 10px; padding: 12px 16px; font-size: 14px; color: #666; margin: 0.75rem 0; }
  .hint-box strong { color: #111; font-weight: 600; }
  .word-chain { display: flex; flex-wrap: wrap; gap: 8px; margin: 1rem 0; min-height: 48px; align-items: center; }
  .word-bubble { background: #f5f5f0; border: 1px solid #ddd; border-radius: 999px; padding: 6px 16px; font-size: 15px; color: #333; }
  .word-bubble.correct { background: #E1F5EE; border-color: #1D9E75; color: #085041; }
  .arrow { color: #aaa; font-size: 18px; }
  .input-row { display: flex; gap: 8px; margin: 1rem 0; }
  .input-row input { flex: 1; padding: 10px 14px; border: 1px solid #ddd; border-radius: 10px; font-size: 15px; font-family: inherit; outline: none; transition: border 0.2s; }
  .input-row input:focus { border-color: #7F77DD; }
  .input-row input:disabled { background: #f5f5f0; color: #aaa; }
  .btn { padding: 10px 18px; border: 1px solid #ddd; border-radius: 10px; background: white; font-size: 14px; font-family: inherit; cursor: pointer; transition: background 0.15s; }
  .btn:hover { background: #f5f5f0; }
  .btn:disabled { opacity: 0.4; cursor: default; }
  .btn-primary { background: #7F77DD; color: white; border-color: #7F77DD; }
  .btn-primary:hover { background: #534AB7; }
  .status-msg { font-size: 14px; padding: 8px 12px; border-radius: 10px; margin: 0.5rem 0; min-height: 36px; }
  .status-msg.ok { background: #E1F5EE; color: #085041; }
  .status-msg.err { background: #FCEBEB; color: #501313; }
  .controls { display: flex; flex-wrap: wrap; gap: 8px; margin-top: 1.25rem; }
  .controls .btn-reset { margin-left: auto; }
  @media (max-width: 480px) { .stats { flex-direction: column; } .controls .btn-reset { margin-left: 0; } }
</style>
</head>
<body>
<div class="container">
  <h1>끝말잇기 <span class="level-badge" id="level-badge">초등 1~2학년</span></h1>

  <div class="stats">
    <div class="stat-card"><div class="label">연결된 단어</div><div class="val" id="score">0</div></div>
    <div class="stat-card"><div class="label">현재 단어</div><div class="val" id="current-word" style="font-size:18px;">—</div></div>
    <div class="stat-card"><div class="label">다음 시작</div><div class="val" id="next-char" style="font-size:18px; color:#1D9E75;">—</div></div>
  </div>

  <div class="hint-box" id="hint-box">🎮 게임을 시작하려면 아래에서 단어를 골라보세요!</div>

  <div class="word-chain" id="chain"></div>

  <div class="input-row">
    <input type="text" id="word-input" placeholder="단어를 입력하세요..." maxlength="10" disabled />
    <button class="btn btn-primary" id="submit-btn" onclick="submitWord()" disabled>입력</button>
  </div>

  <div id="status-msg" style="min-height:36px;"></div>

  <div class="controls">
    <button class="btn" onclick="startGame('easy')">쉬움 (1~2학년)</button>
    <button class="btn" onclick="startGame('medium')">보통 (3~4학년)</button>
    <button class="btn" onclick="startGame('hard')">어려움 (5~6학년)</button>
    <button class="btn btn-reset" onclick="resetGame()">다시 시작</button>
  </div>
</div>

<script>
const VALID_WORDS = new Set([
  '사과','과자','자동차','차도','도서관','관계','계단','단추','추억','억지','지구','구름','통장','장난감','감기','기차','다람쥐',
  '바나나','나비','비행기','기억','억울','울음','음악','악어','어린이','이슬','슬픔','질문','문어','어항','항구','구멍','청소','소나기',
  '우주선','선물','물감','감사','사막','막대','대화','화분','분리','리듬','직업','업무','무게','직선','선생님','비교',
  '하늘','날씨','씨앗','호랑이','이빨','빨래','수박','박쥐','새벽','벽장','장갑','갑자기','기린',
  '강아지','지도','도토리','리본','본관','원숭이','이름','새싹','웃음','음료','빙수','수영','영어',
  '강물','물고기','기둥','주방','방학','학교','교실','실내','내복','복도','도마뱀','나무','무지개','개나리',
  '봄','여름','가을','겨울','눈사람','사람','람쥐','쥐꼬리','리더','더위','위성','성적','적응','응원',
  '고양이','이웃','음식','식물','물병','병원','원피스','스케이트','트럭','럭비','비둘기','기숙사',
  '연필','필통','통일','일기','기분','분홍','홍당무','무지개','개구리','리어카','카메라','라디오',
  '두부','부채','채소','소금','금붕어','어깨','깨소금','금요일','일요일','요리','리본','본보기',
  '달팽이','이마','마을','울타리','리듬','듬직','직각','각도','도형','형광등','등산','산책','책방',
  '바람','람사','사탕','탕수육','육교','교복','복숭아','아침','침대','대문','문제','제목','목도리'
]);

let chain = [], usedWords = new Set(), score = 0, currentWord = '';

function getLastChar(w) { return w[w.length - 1]; }

function startGame(lv) {
  const starters = {easy:'사과', medium:'바나나', hard:'우주선'};
  const labels = {easy:'초등 1~2학년', medium:'초등 3~4학년', hard:'초등 5~6학년'};
  chain = []; usedWords = new Set(); score = 0;
  currentWord = starters[lv];
  usedWords.add(currentWord);
  chain.push({word: currentWord, type: 'start'});
  document.getElementById('level-badge').textContent = labels[lv];
  document.getElementById('score').textContent = '0';
  document.getElementById('current-word').textContent = currentWord;
  document.getElementById('next-char').textContent = getLastChar(currentWord);
  document.getElementById('word-input').disabled = false;
  document.getElementById('submit-btn').disabled = false;
  document.getElementById('word-input').placeholder = `'${getLastChar(currentWord)}'로 시작하는 단어...`;
  document.getElementById('word-input').value = '';
  document.getElementById('status-msg').innerHTML = '';
  document.getElementById('hint-box').innerHTML = `💡 <strong>'${getLastChar(currentWord)}'</strong>로 시작하는 단어를 입력하세요!`;
  renderChain();
}

function submitWord() {
  const input = document.getElementById('word-input');
  const word = input.value.trim();
  input.value = '';
  if (!word) return;
  const needed = getLastChar(currentWord);
  const statusEl = document.getElementById('status-msg');
  if (word[0] !== needed) {
    statusEl.innerHTML = `<div class="status-msg err">❌ '${needed}'로 시작하는 단어여야 해요! (입력: '${word[0]}')</div>`;
    return;
  }
  if (usedWords.has(word)) {
    statusEl.innerHTML = `<div class="status-msg err">❌ '${word}'는 이미 사용한 단어예요!</div>`;
    return;
  }
  if (word.length < 2) {
    statusEl.innerHTML = `<div class="status-msg err">❌ 두 글자 이상의 단어를 입력하세요!</div>`;
    return;
  }
  usedWords.add(word); chain.push({word, type:'user'}); currentWord = word; score++;
  document.getElementById('score').textContent = score;
  document.getElementById('current-word').textContent = word;
  document.getElementById('next-char').textContent = getLastChar(word);
  document.getElementById('word-input').placeholder = `'${getLastChar(word)}'로 시작하는 단어...`;
  document.getElementById('hint-box').innerHTML = `✅ 정답! 이제 <strong>'${getLastChar(word)}'</strong>로 시작하는 단어를 입력하세요.`;
  statusEl.innerHTML = `<div class="status-msg ok">⭐ 잘했어요! '${word}' +1점</div>`;
  renderChain();
}

function renderChain() {
  const el = document.getElementById('chain');
  el.innerHTML = chain.map((item, i) =>
    (i > 0 ? '<span class="arrow">→</span>' : '') +
    `<span class="word-bubble ${item.type === 'start' ? '' : 'correct'}">${item.word}</span>`
  ).join('');
}

function resetGame() {
  chain = []; usedWords = new Set(); score = 0; currentWord = '';
  document.getElementById('score').textContent = '0';
  document.getElementById('current-word').textContent = '—';
  document.getElementById('next-char').textContent = '—';
  document.getElementById('word-input').disabled = true;
  document.getElementById('word-input').placeholder = '단어를 입력하세요...';
  document.getElementById('submit-btn').disabled = true;
  document.getElementById('status-msg').innerHTML = '';
  document.getElementById('hint-box').innerHTML = '🎮 게임을 시작하려면 아래에서 단어를 골라보세요!';
  document.getElementById('chain').innerHTML = '';
  document.getElementById('level-badge').textContent = '초등 1~2학년';
}

document.getElementById('word-input').addEventListener('keydown', e => { if (e.key === 'Enter') submitWord(); });
</script>
</body>
</html>

