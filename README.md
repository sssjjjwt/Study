30325 이원태
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>오늘 공부</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <style>
    body { font-family: Arial; padding: 20px; background:#f4f7fb; }
    input { padding: 8px; font-size:16px; width: 200px; margin: 5px; }
    button {
      margin: 5px;
      padding: 10px 15px;
      font-size: 15px;
      cursor: pointer;
      border: none;
      border-radius: 5px;
      color: white;
    }
    .submit-btn  { background:#007bff; }
    .start-btn   { background:#28a745; }
    .stop-btn    { background:#ffc107; color:black; }
    .graph-btn   { background:#6f42c1; }
    .reset-btn   { background:#dc3545; }
    #messages {
      background: #fff;
      height: 200px;
      overflow-y: auto;
      padding: 10px;
      border-radius: 6px;
      border: 1px solid #ccc;
      margin-top: 10px;
      white-space: pre-line;
    }
    canvas { margin-top: 20px; max-width: 100%; height: 300px; }
  </style>
</head>
<body>

<h2>📚 공부 타이머 & 학습 조언 & 그래프</h2>

<label>과목명: <input type="text" id="subjectInput" placeholder="예: 수학, 영어"></label><br>
<label>공부 시간(분): <input type="number" id="studyTimeInput" placeholder="예: 60"></label><br>

<button class="submit-btn" onclick="submitTime()">⏱️ 시간 입력</button>
<button class="start-btn" onclick="startTimer()">▶️ 공부 시작</button>
<button class="stop-btn" onclick="stopTimer()">⏹️ 공부 종료</button>
<button class="graph-btn" onclick="showChart()">📊 그래프 보기</button>
<button class="reset-btn" onclick="reset()">🔄 초기화</button>

<div id="messages"></div>
<canvas id="studyChart"></canvas>

<script>
  let subjectTimes = {};
  let currentSubject = "";
  let startTime = null;
  let chart = null;

  function showMessage(msg) {
    const box = document.getElementById("messages");
    const p = document.createElement("p");
    p.textContent = msg;
    box.appendChild(p);
    box.scrollTop = box.scrollHeight;
  }

  function submitTime() {
    const subject = document.getElementById("subjectInput").value.trim();
    const minutes = parseInt(document.getElementById("studyTimeInput").value);
    if (!subject || isNaN(minutes) || minutes <= 0) {
      alert("과목명과 시간을 정확히 입력해주세요.");
      return;
    }
    addStudyTime(subject, minutes);
    document.getElementById("subjectInput").value = "";
    document.getElementById("studyTimeInput").value = "";
  }

  function startTimer() {
    const subject = document.getElementById("subjectInput").value.trim();
    if (!subject) {
      alert("공부할 과목명을 먼저 입력하세요.");
      return;
    }
    if (startTime !== null) {
      alert("이미 공부 중입니다!");
      return;
    }
    currentSubject = subject;
    startTime = new Date();
    showMessage(`▶️ [${subject}] 공부 시작`);
  }

  function stopTimer() {
    if (!startTime || !currentSubject) {
      alert("공부를 시작한 뒤에 종료할 수 있어요.");
      return;
    }
    const endTime = new Date();
    const diff = Math.floor((endTime - startTime) / 60000);
    if (diff < 1) {
      showMessage(`⏹️ [${currentSubject}] 너무 짧아 저장하지 않았습니다.`);
    } else {
      addStudyTime(currentSubject, diff);
      showMessage(`⏹️ [${currentSubject}] 공부 종료 — ${diff}분 기록됨`);
    }
    currentSubject = "";
    startTime = null;
  }

  function addStudyTime(subject, minutes) {
    subjectTimes[subject] = (subjectTimes[subject] || 0) + minutes;
    const total = subjectTimes[subject];
    showMessage(`📘 [${subject}] 총 공부 시간: ${total}분`);
    giveAdvice(subject, total);
  }

  function giveAdvice(subject, totalMinutes) {
    let advice = "";
    if (totalMinutes < 60) {
      advice = "✨ 아직 시작 단계예요! 하루 30분부터 꾸준히 시작해봐요.";
    } else if (totalMinutes < 300) {
      advice = "👍 잘하고 있어요! 지금은 기초 복습과 이해가 중요합니다.";
    } else if (totalMinutes < 1000) {
      advice = "💪 안정적인 학습 리듬입니다. 문제풀이로 적용력을 키워보세요!";
    } else {
      advice = "🔥 훌륭합니다! 실전 대비 모의고사를 풀어보는 건 어떨까요?";
    }
    showMessage(`💡 [${subject}] 학습 조언: ${advice}`);
  }

  function reset() {
    if (confirm("기록을 모두 초기화할까요?")) {
      subjectTimes = {};
      currentSubject = "";
      startTime = null;
      if (chart) chart.destroy();
      document.getElementById("messages").innerHTML = "";
      showMessage("🧹 모든 기록이 초기화되었습니다.");
    }
  }

  function showChart() {
    const labels = Object.keys(subjectTimes);
    const data = Object.values(subjectTimes);
    if (chart) chart.destroy();
    const ctx = document.getElementById("studyChart").getContext("2d");
    chart = new Chart(ctx, {
      type: 'bar',
      data: {
        labels: labels,
        datasets: [{
          label: '누적 공부 시간 (분)',
          data: data,
          backgroundColor: 'rgba(75, 192, 192, 0.6)',
          borderColor: 'rgba(75, 192, 192, 1)',
          borderWidth: 1
        }]
      },
      options: {
        responsive: true,
        scales: {
          y: {
            beginAtZero: true,
            title: {
              display: true,
              text: "분"
            }
          }
        }
      }
    });
  }
</script>

</body>
</html>
