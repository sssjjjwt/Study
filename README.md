<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>학습 시간 트래커</title>
  <style>
    body {
      font-family: 'Arial';
      margin: 0;
      padding: 0;
      background-color: #eef2f3;
    }
    .container {
      max-width: 600px;
      margin: 50px auto;
      padding: 20px;
      background: white;
      border-radius: 10px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.2);
    }
    h1 {
      text-align: center;
      color: #333;
    }
    .message-box {
      min-height: 200px;
      margin: 20px 0;
      padding: 10px;
      background: #f9f9f9;
      border-radius: 5px;
      overflow-y: auto;
    }
    .input-row {
      display: flex;
      margin-bottom: 10px;
    }
    .input-row input {
      flex: 1;
      padding: 10px;
      margin-right: 5px;
      border-radius: 5px;
      border: 1px solid #ccc;
    }
    .input-row button {
      padding: 10px;
      border: none;
      border-radius: 5px;
      background: #28a745;
      color: white;
      cursor: pointer;
    }
    .input-row button:hover {
      background: #218838;
    }
    .reset-button {
      background: #dc3545;
      margin-left: 5px;
    }
    .reset-button:hover {
      background: #c82333;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>📚 학습 시간 기록 챗봇</h1>
    <div class="message-box" id="messages"></div>
    <div class="input-row">
      <input type="number" id="studyTimeInput" placeholder="오늘 학습 시간 (분)">
      <button onclick="submitTime()">입력</button>
      <button class="reset-button" onclick="reset()">다시 시작</button>
    </div>
  </div>

  <script>
    let userName = "";
    let totalMinutes = 0;

    window.onload = function () {
      userName = prompt("반갑습니다. 성함이 어떻게 되시나요?", "학생");
      loadData();
      showMessage("안녕하세요, " + userName + "님! 오늘 공부하신 시간을 분 단위로 입력해주세요.");
      showMessage("누적 시간에 따라 공부 조언을 드립니다. 😊");
    };

    function loadData() {
      const saved = localStorage.getItem("study_data_" + userName);
      if (saved) {
        totalMinutes = Number(saved);
      }
    }

    function saveData() {
      localStorage.setItem("study_data_" + userName, totalMinutes);
    }

    function showMessage(msg) {
      const box = document.getElementById("messages");
      const p = document.createElement("p");
      p.textContent = msg;
      box.appendChild(p);
      box.scrollTop = box.scrollHeight;
    }

    function submitTime() {
      const input = document.getElementById("studyTimeInput");
      const value = Number(input.value);

      if (isNaN(value) || value <= 0) {
        alert("올바른 학습 시간을 입력해주세요.");
        return;
      }

      totalMinutes += value;
      saveData();

      showMessage("⏱️ 입력된 학습 시간: " + value + "분");
      showMessage("📊 누적 학습 시간: " + totalMinutes + "분");
      showAdvice();
      input.value = "";
    }

    function showAdvice() {
      let msg = "";
      if (totalMinutes < 60) {
        msg = "🔹 아직은 시작 단계예요. 매일 30분씩 꾸준히 해보세요!";
      } else if (totalMinutes < 300) {
        msg = "🔹 좋은 출발입니다! 계획을 세우고 루틴을 만들어보세요.";
      } else if (totalMinutes < 1000) {
        msg = "🔹 훌륭해요! 복습과 실전 연습도 병행해보세요.";
      } else {
        msg = "🎉 대단합니다! 지금처럼 꾸준히만 하면 무엇이든 가능해요!";
      }
      showMessage(msg);
    }

    function reset() {
      if (confirm("정말 다시 시작하시겠습니까? 모든 기록이 삭제됩니다.")) {
        totalMinutes = 0;
        saveData();
        document.getElementById("messages").innerHTML = "";
        showMessage("모든 기록이 초기화되었습니다. 새롭게 시작해볼까요?");
      }
    }
  </script>
</body>
</html>
