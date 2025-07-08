<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>과목별 학습 시간 측정기</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background-color: #eef1f4;
      margin: 0;
      padding: 0;
    }
    .container {
      max-width: 600px;
      margin: 40px auto;
      padding: 20px;
      background: white;
      border-radius: 12px;
      box-shadow: 0 4px 12px rgba(0,0,0,0.1);
    }
    h1 {
      text-align: center;
      color: #333;
    }
    .message-box {
      height: 250px;
      overflow-y: auto;
      background: #f8f8f8;
      border: 1px solid #ccc;
      border-radius: 8px;
      padding: 10px;
      margin: 20px 0;
    }
    .form-group {
      margin-bottom: 15px;
    }
    label {
      font-weight: bold;
      display: block;
      margin-bottom: 5px;
    }
    input {
      width: 100%;
      padding: 10px;
      border: 1px solid #bbb;
      border-radius: 6px;
      font-size: 16px;
    }
    .button-row {
      display: flex;
      flex-wrap: wrap;
      gap: 10px;
      margin-top: 10px;
    }
    button {
      flex: 1;
      padding: 10px;
      border: none;
      border-radius: 6px;
      font-size: 16px;
      color: white;
      cursor: pointer;
    }
    .submit-btn    { background-color: #007bff; }
    .start-btn     { background-color: #28a745; }
    .stop-btn      { background-color: #ffc107; color: black; }
    .reset-btn     { background-color: #dc3545; }
    button:hover {
      opacity: 0.9;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>📚 과목별 공부 타이머</h1>

    <div class="message-box" id="messages"></div>

    <div class="form-group">
      <label for="subjectInput">과목명</label>
      <input type="text" id="subjectInput" placeholder="예: 수학, 영어 등">
    </div>

    <div class="form-group">
      <label for="studyTimeInput">직접 입력 시간 (분)</label>
      <input type="number" id="studyTimeInput" placeholder="예: 60">
    </div>

    <div class="button-row">
      <button class="submit-btn" onclick="submitTime()">⏱️ 시간 직접 입력</button>
      <button class="start-btn" onclick="startTimer()">▶️ 공부 시작</button>
      <button class="stop-btn" onclick="stopTimer()">⏹️ 공부 종료</button>
      <button class="reset-btn" onclick="reset()">🔄 다시 시작</button>
    </div>
  </div>

  <script>
    let userName = "";
    let subjectTimes = {};     // 누적 시간 저장용
    let currentSubject = "";
    let startTime = null;

    window.onload = function () {
      userName = prompt("반갑습니다. 성함이 어떻게 되시나요?", "학생");
      loadData();
      showMessage(`👋 안녕하세요, ${userName}님! 과목명을 입력하고 공부를 시작해보세요.`);
    };

    function loadData() {
      const saved = localStorage.getItem("study_subjects_" + userName);
      if (saved) {
        subjectTimes = JSON.parse(saved);
      }
    }

    function saveData() {
      localStorage.setItem("study_subjects_" + userName, JSON.stringify(subjectTimes));
    }

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
        alert("과목명과 유효한 시간을 입력하세요.");
        return;
      }

      addStudyTime(subject, minutes);
      document.getElementById("studyTimeInput").value = "";
      document.getElementById("subjectInput").value = "";
    }

    function startTimer() {
      const subject = document.getElementById("subjectInput").value.trim();
      if (!subject) {
        alert("과목명을 먼저 입력하세요.");
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
        alert("공부를 먼저 시작하세요.");
        return;
      }
      const endTime = new Date();
      const diffMs = endTime - startTime;
      const minutes = Math.floor(diffMs / 1000 / 60);

      if (minutes < 1) {
        showMessage(`⏹️ [${currentSubject}] 공부 시간이 너무 짧아서 저장하지 않았습니다.`);
      } else {
        addStudyTime(currentSubject, minutes);
        showMessage(`⏹️ [${currentSubject}] 공부 종료 - 측정 시간: ${minutes}분`);
      }

      startTime = null;
      currentSubject = "";
    }

    function addStudyTime(subject, minutes) {
      subjectTimes[subject] = (subjectTimes[subject] || 0) + minutes;
      saveData();
      showMessage(`📊 [${subject}] 누적 학습 시간: ${subjectTimes[subject]}분`);
      showAdvice(subject, subjectTimes[subject]);
    }

    function showAdvice(subject, totalMin) {
      let advice = "";
      if (totalMin < 60) {
        advice = "시작이 반! 하루 30분부터 차근차근 해봐요.";
      } else if (totalMin < 300) {
        advice = "좋아요! 개념 정리와 복습을 해보세요.";
      } else if (totalMin < 1000) {
        advice = "훌륭해요! 문제 풀이도 병행해보세요.";
      } else {
        advice = "🔥 대단해요! 지금처럼 꾸준히 하면 큰 성과가 있어요!";
      }
      showMessage(`💡 [${subject}] 학습 조언: ${advice}`);
    }

    function reset() {
      if (confirm("모든 기록을 삭제하고 다시 시작할까요?")) {
        subjectTimes = {};
        startTime = null;
        currentSubject = "";
        localStorage.removeItem("study_subjects_" + userName);
        document.getElementById("messages").innerHTML = "";
        showMessage("📦 모든 학습 기록이 초기화되었습니다.");
      }
    }
  </script>
</body>
</html>

