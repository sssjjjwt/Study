<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>실시간 과목별 학습 트래커</title>
  <style>
    body {
      font-family: Arial;
      background-color: #f0f2f5;
      margin: 0;
      padding: 0;
    }
    .container {
      max-width: 600px;
      margin: 40px auto;
      padding: 20px;
      background-color: #fff;
      border-radius: 12px;
      box-shadow: 0 4px 10px rgba(0,0,0,0.1);
    }
    h1 {
      text-align: center;
      color: #333;
    }
    .message-box {
      height: 250px;
      overflow-y: auto;
      padding: 10px;
      margin: 20px 0;
      background-color: #f9f9f9;
      border-radius: 8px;
      border: 1px solid #ccc;
    }
    .form-group {
      margin-bottom: 15px;
    }
    .form-group label {
      display: block;
      font-weight: bold;
      margin-bottom: 6px;
    }
    .form-group input {
      width: 100%;
      padding: 10px;
      border: 1px solid #ccc;
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
      padding: 10px;
      flex: 1;
      border: none;
      border-radius: 6px;
      background-color: #007bff;
      color: white;
      cursor: pointer;
      font-size: 16px;
    }
    button:hover {
      background-color: #0056b3;
    }
    .reset-button {
      background-color: #dc3545;
    }
    .reset-button:hover {
      background-color: #a71d2a;
    }
    .timer-button {
      background-color: #28a745;
    }
    .timer-button:hover {
      background-color: #1e7e34;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>📚 실시간 과목별 학습 트래커</h1>

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
      <button onclick="submitTime()">⏱️ 시간 직접 입력</button>
      <button class="timer-button" onclick="startTimer()">▶️ 공부 시작</button>
      <button class="timer-button" onclick="stopTimer()">⏹️ 공부 종료</button>
      <button class="reset-button" onclick="reset()">🔄 다시 시작</button>
    </div>
  </div>

  <script>
    let userName = "";
    let subjectTimes = {}; // { subject: total_minutes }
    let currentSubject = "";
    let startTime = null;

    window.onload = function () {
      userName = prompt("반갑습니다. 성함이 어떻게 되시나요?", "학생");
      loadData();
      showMessage("안녕하세요, " + userName + "님! 과목과 시간을 입력하거나 공부 시작/종료 버튼을 사용해보세요.");
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
      const subjectInput = document.getElementById("subjectInput");
      const timeInput = document.getElementById("studyTimeInput");
      const subject = subjectInput.value.trim();
      const time = Number(timeInput.value);

      if (!subject || isNaN(time) || time <= 0) {
        alert("과목과 시간을 정확히 입력해주세요.");
        return;
      }

      addStudyTime(subject, time);
      subjectInput.value = "";
      timeInput.value = "";
    }

    function startTimer() {
      const subjectInput = document.getElementById("subjectInput");
      const subject = subjectInput.value.trim();
      if (!subject) {
        alert("공부할 과목명을 먼저 입력해주세요.");
        return;
      }
      if (startTime !== null) {
        alert("이미 공부 중입니다!");
        return;
      }
      currentSubject = subject;
      startTime = Date.now();
      showMessage(`▶️ [${subject}] 공부를 시작했습니다.`);
    }

    function stopTimer() {
      if (startTime === null || !currentSubject) {
        alert("공부 시작을 먼저 눌러주세요.");
        return;
      }
      const endTime = Date.now();
      const diffMinutes = Math.floor((endTime - startTime) / 1000 / 60); // 분 단위

      if (diffMinutes <= 0) {
        showMessage("😅 너무 짧은 공부 시간은 저장되지 않았습니다.");
      } else {
        addStudyTime(currentSubject, diffMinutes);
        showMessage(`⏹️ [${currentSubject}] 공부 종료! 측정된 시간: ${diffMinutes}분`);
      }

      currentSubject = "";
      startTime = null;
    }

    function addStudyTime(subject, time) {
      subjectTimes[subject] = (subjectTimes[subject] || 0) + time;
      saveData();

      showMessage(`📊 [${subject}] 누적 학습 시간: ${subjectTimes[subject]}분`);
      showAdvice(subject, subjectTimes[subject]);
    }

    function showAdvice(subject, minutes) {
      let msg = "";
      if (minutes < 60) {
        msg = `🔸 ${subject}: 이제 시작이에요! 하루 30분부터 꾸준히 해봐요.`;
      } else if (minutes < 300) {
        msg = `🔸 ${subject}: 좋아요! 개념 복습과 기출 문제도 병행해보세요.`;
      } else if (minutes < 1000) {
        msg = `🔸 ${subject}: 충분히 잘하고 있어요! 오답노트 정리도 해보세요.`;
      } else {
        msg = `🎯 ${subject}: 대단해요! 실전 모의고사나 고난도 문제도 도전해보세요!`;
      }
      showMessage(msg);
    }

    function reset() {
      if (confirm("모든 기록을 초기화하시겠습니까?")) {
        subjectTimes = {};
        saveData();
        document.getElementById("messages").innerHTML = "";
        showMessage("🧹 기록이 모두 초기화되었습니다. 새롭게 시작해볼까요?");
      }
    }
  </script>
</body>
</html>
