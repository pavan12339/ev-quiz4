<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <title>EV Voltage Sensor Quiz</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #f0f4f8;
      padding: 30px;
    }
    #quiz-container, #leaderboard-container {
      background: white;
      padding: 20px;
      border-radius: 10px;
      max-width: 600px;
      margin: auto;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    }
    .option {
      margin-bottom: 10px;
    }
    button {
      padding: 10px 20px;
      margin-top: 15px;
    }
    #leaderboardList {
      padding-left: 0;
      list-style: none;
    }
    #leaderboardList li {
      padding: 5px 10px;
      border-bottom: 1px solid #ddd;
    }
    #timer {
      font-weight: bold;
      color: red;
    }
  </style>
</head>
<body>

  <div id="entry-screen">
    <h2>Welcome to EV Voltage Sensor Quiz</h2>
    <input type="password" id="quiz-password" placeholder="Enter password to start" />
    <button onclick="checkPassword()">Submit</button>
  </div>

  <div id="quiz-container" style="display:none;">
    <div id="question"></div>
    <div id="options"></div>
    <div>⏳ Time Left: <span id="time">15</span> sec</div>
    <button onclick="nextQuestion()">Next</button>
  </div>

  <div id="leaderboard-container" style="display:none;">
    <h3>🏆 Leaderboard</h3>
    <ul id="leaderboardList">Loading...</ul>
  </div>

  <!-- Firebase -->
  <script src="https://www.gstatic.com/firebasejs/9.6.11/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.6.11/firebase-database-compat.js"></script>

  <script>
    const firebaseConfig = {
      apiKey: "YOUR_API_KEY",
      authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
      databaseURL: "https://YOUR_PROJECT_ID.firebaseio.com",
      projectId: "YOUR_PROJECT_ID",
      storageBucket: "YOUR_PROJECT_ID.appspot.com",
      messagingSenderId: "YOUR_SENDER_ID",
      appId: "YOUR_APP_ID"
    };

    firebase.initializeApp(firebaseConfig);
    const db = firebase.database();

    const quizData = [
      { question: "Why are isolated voltage sensors crucial in EV battery monitoring?", options: ["To minimize software complexity", "To protect low-voltage systems from high-voltage spikes", "To reduce electromagnetic noise", "To increase charging speed"], answer: 1 },
      { question: "Which type of voltage sensor provides galvanic isolation for high-voltage EV systems?", options: ["Shunt resistor", "Resistive voltage divider", "Hall-effect based voltage sensor", "Thermocouple"], answer: 2 },
      { question: "What is a common limitation of using resistive dividers directly with high-voltage EV batteries?", options: ["Overcurrent errors", "Lack of electrical isolation", "High-frequency distortion", "Low signal resolution"], answer: 1 },
      { question: "In regenerative braking, which sensor characteristic is critical for voltage measurement?", options: ["Small size", "Fast response time", "High impedance", "Low offset"], answer: 1 },
      { question: "What does a low Common Mode Rejection Ratio (CMRR) in voltage sensors lead to?", options: ["Accurate measurement under all conditions", "Increased susceptibility to noise", "Improved thermal compensation", "Higher current draw"], answer: 1 },
      { question: "How does temperature affect voltage sensor accuracy in EVs?", options: ["Causes random noise spikes", "Improves accuracy at high temp", "Can lead to offset drift and gain error", "Has no noticeable effect"], answer: 2 },
      { question: "Why are digital voltage sensors preferred in some EV applications?", options: ["Lower cost", "Faster charging", "No need for calibration", "Better noise immunity and direct microcontroller interface"], answer: 3 },
      { question: "Which fault could lead to a false voltage drop detection in a cell by the BMS?", options: ["Overcharging", "Poor ADC resolution in sensor", "Cell imbalance", "Reverse current flow"], answer: 1 },
      { question: "What role do low-pass filters play in EV voltage sensing?", options: ["Amplify voltage", "Reduce common-mode voltage", "Filter high-frequency EMI noise", "Convert analog to digital"], answer: 2 },
      { question: "Which tool combination is best for validating EV voltage sensors during load testing?", options: ["Basic multimeter", "Thermal imaging camera", "Differential probe with data logger", "RF spectrum analyzer"], answer: 2 }
    ];

    let currentQuestion = 0;
    let score = 0;
    let userName = "";
    let questionTimer;
    let timeLeft = 15;

    function checkPassword() {
      const input = document.getElementById("quiz-password").value.trim();
      if (input === "sensor") {
        document.getElementById("entry-screen").style.display = "none";
        document.getElementById("quiz-container").style.display = "block";
        loadQuestion();
      } else if (input === "pavan8317") {
        document.getElementById("entry-screen").style.display = "none";
        document.getElementById("leaderboard-container").style.display = "block";
        showLeaderboard();
      } else {
        alert("Incorrect password!");
      }
    }

    function loadQuestion() {
      const q = quizData[currentQuestion];
      document.getElementById("question").innerText = `${currentQuestion + 1}. ${q.question}`;
      const optionsDiv = document.getElementById("options");
      optionsDiv.innerHTML = "";

      q.options.forEach((opt, index) => {
        const div = document.createElement("div");
        div.classList.add("option");

        const input = document.createElement("input");
        input.type = "radio";
        input.name = "option";
        input.value = index;
        input.id = `opt${index}`;

        const label = document.createElement("label");
        label.htmlFor = `opt${index}`;
        label.innerText = opt;

        div.appendChild(input);
        div.appendChild(label);
        optionsDiv.appendChild(div);
      });

      startQuestionTimer();
    }

    function startQuestionTimer() {
      clearInterval(questionTimer);
      timeLeft = 15;
      document.getElementById("time").innerText = timeLeft;
      questionTimer = setInterval(() => {
        timeLeft--;
        document.getElementById("time").innerText = timeLeft;
        if (timeLeft <= 0) {
          clearInterval(questionTimer);
          nextQuestion(true);
        }
      }, 1000);
    }

    function nextQuestion(auto = false) {
      clearInterval(questionTimer);
      const selected = document.querySelector('input[name="option"]:checked');
      const value = selected ? parseInt(selected.value) : -1;

      if (!auto && value === -1) {
        alert("Please select an option.");
        startQuestionTimer();
        return;
      }

      if (value === quizData[currentQuestion].answer) score++;

      currentQuestion++;
      if (currentQuestion < quizData.length) {
        loadQuestion();
      } else {
        submitQuiz();
      }
    }

    function submitQuiz() {
      clearInterval(questionTimer);
      userName = prompt("Enter your name for the leaderboard:") || "Anonymous";
      const ref = db.ref("ev_voltage_quiz_leaderboard");
      ref.push({ name: userName, score: score });

      document.getElementById("quiz-container").style.display = "none";
      document.getElementById("leaderboard-container").style.display = "block";
      showLeaderboard();
    }

    function showLeaderboard() {
      const ref = db.ref("ev_voltage_quiz_leaderboard");
      const list = document.getElementById("leaderboardList");
      ref.orderByChild("score").limitToLast(100).once("value", (snapshot) => {
        const entries = [];
        snapshot.forEach(child => entries.push(child.val()));
        entries.reverse();
        list.innerHTML = "";
        entries.forEach((entry, i) => {
          const li = document.createElement("li");
          li.innerText = `${i + 1}. ${entry.name} - ${entry.score}`;
          list.appendChild(li);
        });
      });
    }
  </script>

</body>
</html>
