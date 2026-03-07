---
title: Deepseek Dashi
type: page
---

<style>
  .chendashi-container {
    max-width: 700px;
    margin: 0 auto;
  }
  .chendashi-container label {
    display: block;
    margin-top: 1rem;
    font-weight: 600;
  }
  .chendashi-container input,
  .chendashi-container textarea {
    width: 100%;
    padding: 0.5rem;
    margin-top: 0.25rem;
    border: 1px solid #ccc;
    border-radius: 4px;
    font-size: 1rem;
    box-sizing: border-box;
  }
  .chendashi-container button {
    margin-top: 1.5rem;
    padding: 0.6rem 1.5rem;
    background: #1976d2;
    color: #fff;
    border: none;
    border-radius: 4px;
    font-size: 1rem;
    cursor: pointer;
  }
  .chendashi-container button:disabled {
    background: #90a4ae;
    cursor: not-allowed;
  }
  .chendashi-container button:hover:not(:disabled) {
    background: #1565c0;
  }
  #response-box {
    margin-top: 1.5rem;
    padding: 1rem;
    background: #f5f5f5;
    border: 1px solid #ddd;
    border-radius: 4px;
    min-height: 100px;
    white-space: pre-wrap;
    font-size: 0.95rem;
    line-height: 1.5;
  }
  #current-time {
    margin-top: 0.5rem;
    font-size: 0.9rem;
    color: #666;
  }
</style>

<div class="chendashi-container">

<label for="api-key">DeepSeek API Key</label>
<input type="password" id="api-key" placeholder="sk-...">

<label for="paper-name">Paper Name</label>
<input type="text" id="paper-name" placeholder="Enter paper name">

<label for="area">Area</label>
<input type="text" id="area" placeholder="e.g. Programming Language, Software Engineering">

<label for="release-date">AOE Result Release Date</label>
<input type="date" id="release-date">

<label for="conference">Conference</label>
<input type="text" id="conference" placeholder="e.g. POPL, PLDI">

<div id="current-time"></div>

<button id="submit-btn" onclick="submitQuery()">Submit</button>

<div id="response-box">Response will appear here...</div>

</div>

<script>
function formatChinese(d) {
  return d.getFullYear() + '年' + (d.getMonth()+1) + '月' + d.getDate() + '日';
}
function updateTime() {
  document.getElementById('current-time').textContent = 'Current time: ' + formatChinese(new Date());
}
updateTime();
setInterval(updateTime, 1000);

async function submitQuery() {
  const apiKey = document.getElementById('api-key').value.trim();
  const paperName = document.getElementById('paper-name').value.trim();
  const area = document.getElementById('area').value.trim();
  const releaseDateRaw = document.getElementById('release-date').value;
  const conference = document.getElementById('conference').value.trim();
  const currentTime = formatChinese(new Date());
  const releaseDate = releaseDateRaw ? formatChinese(new Date(releaseDateRaw + 'T00:00:00')) : '';

  if (!apiKey) { alert('Please enter your DeepSeek API key.'); return; }
  if (!paperName) { alert('Please enter the paper name.'); return; }

  const btn = document.getElementById('submit-btn');
  const responseBox = document.getElementById('response-box');
  btn.disabled = true;
  btn.textContent = 'Loading...';
  responseBox.textContent = 'Waiting for response...';

  const prompt = `现在是 ${currentTime}, 我们投的 ${conference} 文章， 关于 ${area}, 名字叫 ${paperName}. AOE ${releaseDate} 出结果。
请你用六爻， 帮我分析投稿文章能中的概率`;

  try {
    const res = await fetch('https://api.deepseek.com/chat/completions', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
        'Authorization': 'Bearer ' + apiKey
      },
      body: JSON.stringify({
        model: 'deepseek-chat',
        messages: [
          { role: 'system', content: '你是一位命理大师，精通奇门遁甲，紫微斗数，梅花易数，八字和六爻等，并且熟读周易和易经。' },
          { role: 'user', content: prompt }
        ],
        temperature: 0.7
      })
    });

    if (!res.ok) {
      const err = await res.text();
      throw new Error('API error (' + res.status + '): ' + err);
    }

    const data = await res.json();
    responseBox.textContent = data.choices[0].message.content;
  } catch (e) {
    responseBox.textContent = 'Error: ' + e.message;
  } finally {
    btn.disabled = false;
    btn.textContent = 'Submit';
  }
}
</script>
