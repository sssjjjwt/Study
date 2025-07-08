<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1" />
  <title>과목별 공부 타이머 + 그래프 + 조언</title>
  <style>
    body { font-family: Arial; padding: 20px; background:#eef1f4; }
    button {
      margin: 5px;
      padding: 10px 15px;
      font-size: 16px;
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
      background: #f8f8f8;
      height: 200px;
      overflow-y: auto;
      padding: 10px;
      border-radius: 6px;
      border: 1px solid #ccc;
      margin-top: 15px;
      white-space: pre-line;
    }
    input { padding: 8px; font-size:16px; width: 200px; margin-right: 10px; }
    canvas { margin-top: 20px; max-width: 100%; height: 300px; }
  </style>
  <!-- Chart.js -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
</head>
<body>

<h1>📚 공부 타이머 & 그래프 & 조언</h1>

<label>과목명: <input type="text" id="subjectInput" placeholder="예: 수학, 영어"></label><br/>
<label>직접 입력 시간 (분): <input type="number" id="studyTimeInput" placeholder="예: 60"></label><br/>

<button class="submit-btn" onclick="submitTime()">⏱️ 시간 입력</button>
<button class="start-btn" onclick="startTimer()">▶️ 공부 시작</button>
<button class="stop-btn" onclick="stopTimer()">⏹️ 공부 종료</button>
<button class="graph-btn" onclick="showChart()">📊 학습 그래프</button>
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
      alert("과목명과 시간을 바르게 입력하세요.");
      return;
    }
    addStudyTime(subject, minutes);
    document.getElementById("subjectInput").value = "";
    document.getElementById("studyTimeInput").value = "";
  }

  function startTimer() {
    const subject = document.getElementById("subjectInput").value.trim();
    if (!subject) {
      alert("공부할 과목명을 입력하세요.");
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
      alert("공부를 시작한 뒤에 종료할 수 있습니다.");
      return;
    }
    const endTime = new Date();
    const diff = Math.floor((endTime - startTime) / 60000);
    if (diff < 1) {
      showMessage(`⏹️ [${currentSubject}] 너무 짧은 시간이라 저장하지 않았습니다.`);
    } else {
      addStudyTime(currentSubject, diff);
      showMessage(`⏹️ [${currentSubject}] 공부 종료 — ${diff}분 기록됨`);
    }
    currentSubject = "";
    startTime = null;
  }

  function addStudyTime(subject, minutes) {
    subjectTimes[subject] = (subjectTimes[subject] || 0) + minutes;
    showMessage(`📚 [${subject}] 누적 시간: ${subjectTimes[subject]}분`);
    giveAdvice(subject, subjectTimes[subject]);
  }

  function giveAdvice(subject, totalMinutes) {
    let advice = "";
    if (totalMinutes < 60) {
      advice = "처음이 중요해요! 매일 30분씩 꾸준히 해보세요.";
    } else if (totalMinutes < 300) {
      advice = "좋아요! 복습과 요약 정리를 함께 해보세요.";
    } else if (totalMinutes < 1000) {
      advice = "꾸준함이 보이네요! 오답노트와 실전 문제로 확장해보세요.";
    } else {
      advice = "🔥 최고의 집중력! 지금 상태로 시험장 가셔도 될 것 같아요!";
    }
    showMessage(`💡 [${subject}] 학습 조언: ${advice}`);
  }

  function reset() {
    if (confirm("모든 기록을 초기화하시겠습니까?")) {
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
      type: "bar",
      data: {
        labels: labels,
        datasets: [{
          label: "누적 학습 시간 (분)",
          data: data,
          backgroundColor: "rgba(54, 162, 235, 0.6)",
          borderColor: "rgba(54, 162, 235, 1)",
          borderWidth: 1
        }]
      },
      options: {
        responsive: true,
        scales: {
          y: {
            beginAtZero: true,
            title: { display: true, text: "분" }
          }
        }
      }
    });
  }
</script>

</body>
</html>

