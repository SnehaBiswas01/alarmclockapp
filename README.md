# alarmclockapp
index.html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>Alarm Clock App</title>
  <link rel="stylesheet" href="style.css"/>
</head>
<body>
  <div class="container">
    <h1>Alarm Clock</h1>
    <div class="current-time" id="current-time"></div>

    <div class="alarm-setter">
      <h2>Alarm Clock</h2>
      <input type="time" id="alarm-time" required />
      <select id="alarm-tone">
        <option value="tone1.mp3">Classic Beep</option>
        <option value="tone2.mp3">Digital Bell</option>
      </select>
      <button onclick="setAlarm()">Set Alarm</button>
    </div>

    <div class="alarm-list">
      <h2>Scheduled Alarm(s)</h2>
      <ul id="alarms"></ul>
    </div>

    <div class="alarm-action" id="alarm-action">
      <p>⏰ Alarm Ringing!</p>
      <button onclick="snoozeAlarm()">Snooze</button>
      <button onclick="dismissAlarm()">Dismiss</button>
    </div>
  </div>

  <audio id="alarm-audio" src=""></audio>
  <script src="script.js"></script>
</body>
</html>
style.css
body {
  margin: 0;
  font-family: 'Segoe UI', sans-serif;
  background: linear-gradient(to right, #ffecd2, #fcb69f);
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100vh;
}

.container {
  background: white;
  padding: 2rem;
  border-radius: 15px;
  width: 90%;
  max-width: 500px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.2);
  text-align: center;
}

.current-time {
  font-size: 2rem;
  margin: 1rem 0;
  font-weight: bold;
  color: #333;
}

.alarm-setter, .alarm-list {
  margin-top: 1.5rem;
  text-align: left;
}

input[type="time"], select {
  width: 100%;
  padding: 10px;
  margin: 0.5rem 0;
  font-size: 1rem;
  border: 1px solid #ccc;
  border-radius: 8px;
}

button {
  background-color: #ff6f61;
  color: white;
  border: none;
  padding: 10px 20px;
  margin-top: 0.5rem;
  font-size: 1rem;
  border-radius: 10px;
  cursor: pointer;
}

button:hover {
  background-color: #e85c50;
}

.alarm-list ul {
  list-style: none;
  padding: 0;
}

.alarm-list li {
  display: flex;
  justify-content: space-between;
  align-items: center;
  background: #f2f2f2;
  padding: 10px;
  margin: 5px 0;
  border-radius: 8px;
}

.alarm-action {
  display: none;
  margin-top: 1.5rem;
}

.alarm-action p {
  font-size: 1.5rem;
  margin-bottom: 1rem;
}
script.js
const currentTimeDisplay = document.getElementById("current-time");
const alarmAudio = document.getElementById("alarm-audio");
const alarmAction = document.getElementById("alarm-action");
let alarms = [];
let activeAlarm = null;
let snoozeTimer = null;

function updateTime() {
  const now = new Date();
  const formatted = now.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
  currentTimeDisplay.textContent = ${formatted} - ${now.toDateString()};

  alarms.forEach((alarm, index) => {
    if (alarm.enabled && alarm.time === formatted) {
      triggerAlarm(index);
    }
  });
}

setInterval(updateTime, 1000);

function setAlarm() {
  const time = document.getElementById("alarm-time").value;
  const tone = document.getElementById("alarm-tone").value;
  if (!time) return alert("Please select a time");

  const alarm = {
    time: convertTo12Hour(time),
    tone: tone,
    enabled: true
  };

  alarms.push(alarm);
  renderAlarms();
}

function convertTo12Hour(timeStr) {
  const [hour, minute] = timeStr.split(":");
  const date = new Date();
  date.setHours(hour, minute);
  return date.toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' });
}

function renderAlarms() {
  const alarmList = document.getElementById("alarms");
  alarmList.innerHTML = "";

  alarms.forEach((alarm, index) => {
    const li = document.createElement("li");
    li.innerHTML = `
      ${alarm.time}
      <div>
        <label>
          <input type="checkbox" ${alarm.enabled ? "checked" : ""} onchange="toggleAlarm(${index})" />
        </label>
        <button onclick="deleteAlarm(${index})">🗑</button>
      </div>
    `;
    alarmList.appendChild(li);
  });
}

function toggleAlarm(index) {
  alarms[index].enabled = !alarms[index].enabled;
}

function deleteAlarm(index) {
  alarms.splice(index, 1);
  renderAlarms();
}

function triggerAlarm(index) {
  activeAlarm = alarms[index];
  alarmAudio.src = activeAlarm.tone;
  alarmAudio.loop = true;
  alarmAudio.play();
  alarmAction.style.display = "block";
}

function snoozeAlarm() {
  alarmAudio.pause();
  alarmAudio.currentTime = 0;
  alarmAction.style.display = "none";
  snoozeTimer = setTimeout(() => {
    alarmAudio.play();
    alarmAction.style.display = "block";
  }, 5 * 60 * 1000); // snooze for 5 minutes
}

function dismissAlarm() {
  alarmAudio.pause();
  alarmAudio.currentTime = 0;
  alarmAction.style.display = "none";
  if (snoozeTimer) clearTimeout(snoozeTimer);
}
