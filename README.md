<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>Attendance Tracker</title>
  <script src="https://cdn.sheetjs.com/xlsx-latest/package/dist/xlsx.full.min.js"></script>
  <style>
    body {
      font-family: Arial, sans-serif;
      text-align: center;
      margin: 20px;
    }
    h1 { margin-bottom: 10px; }

    #selector {
      background: #f9f9f9;
      padding: 15px;
      margin-bottom: 20px;
      border-radius: 8px;
      display: inline-block;
      box-shadow: 0 2px 5px rgba(0,0,0,0.1);
    }
    #selector button {
      margin: 5px;
      padding: 10px 20px;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      font-weight: bold;
    }
    .btn-wfo { background: green; color: white; }
    .btn-wfh { background: blue; color: white; }
    .btn-leave { background: orange; color: white; }
    .btn-clear { background: gray; color: white; }

    #calendar {
      display: grid;
      grid-template-columns: repeat(7, 1fr);
      gap: 5px;
      max-width: 500px;
      margin: 20px auto;
    }
    .day-name {
      font-weight: bold;
      padding: 10px 0;
      background: #eee;
      border-radius: 6px;
    }
    .day {
      border: 1px solid #ccc;
      padding: 15px;
      cursor: pointer;
      border-radius: 6px;
      transition: 0.3s;
      user-select: none;
    }
    .day:hover:not(.weekend) { background: #f0f0f0; }
    .wfo { background: green; color: white; }
    .wfh { background: blue; color: white; }
    .leave { background: orange; color: white; }
    .weekend { background: #ddd; color: #666; cursor: not-allowed; }

    #summary {
      margin-top: 20px;
      font-size: 16px;
      font-weight: bold;
      padding: 10px;
      border-radius: 6px;
      display: inline-block;
      min-width: 250px;
    }

    #exportBtn {
      margin-top: 20px;
      padding: 10px 20px;
      font-weight: bold;
      border-radius: 6px;
      border: none;
      background: #007bff;
      color: white;
      cursor: pointer;
    }

  </style>
</head>
<body>

<h1>Attendance Tracker</h1>

<div id="selector">
  <strong>Select Mode:</strong><br>
  <button class="btn-wfo">WFO</button>
  <button class="btn-wfh">WFH</button>
  <button class="btn-leave">Leave</button>
  <button class="btn-clear">Clear</button>
</div>

<div id="calendar"></div>
<div id="summary">WFO: 0 | WFH: 0 | Leave: 0 | RTO%: 0.00%</div>
<button id="exportBtn">Export to Excel</button>

<script>
let username = prompt("Enter your name:");
if(!username) username = "User";

const calendarEl = document.getElementById("calendar");
const summaryEl = document.getElementById("summary");
const exportBtn = document.getElementById("exportBtn");

const btnWfo = document.querySelector(".btn-wfo");
const btnWfh = document.querySelector(".btn-wfh");
const btnLeave = document.querySelector(".btn-leave");
const btnClear = document.querySelector(".btn-clear");

let activeMode = null;
const days = [];

btnWfo.addEventListener("click", () => activeMode = "wfo");
btnWfh.addEventListener("click", () => activeMode = "wfh");
btnLeave.addEventListener("click", () => activeMode = "leave");
btnClear.addEventListener("click", () => activeMode = "clear");

// Current month
const today = new Date();
const year = today.getFullYear();
const month = today.getMonth();
const monthName = today.toLocaleString('default', { month: 'long' });

function getDaysInMonth(year, month) {
  return new Date(year, month + 1, 0).getDate();
}

// Day names
const dayNames = ['Mon','Tue','Wed','Thu','Fri','Sat','Sun'];
dayNames.forEach(d => {
  const dayNameEl = document.createElement("div");
  dayNameEl.classList.add("day-name");
  dayNameEl.textContent = d;
  calendarEl.appendChild(dayNameEl);
});

// Render calendar
const totalDays = getDaysInMonth(year, month);
for (let i = 1; i <= totalDays; i++) {
  const date = new Date(year, month, i);
  const dayOfWeek = date.getDay();
  const dayEl = document.createElement("div");
  dayEl.classList.add("day");
  dayEl.textContent = i;

  // Disable Sunday (0) & Saturday (6)
  if(dayOfWeek === 0 || dayOfWeek === 6){
    dayEl.classList.add("weekend");
    days[i] = 'weekend';
  }

  dayEl.addEventListener("click", () => {
    if(dayEl.classList.contains("weekend")) return;

    dayEl.classList.remove("wfo","wfh","leave");

    if(activeMode === "wfo") {
      dayEl.classList.add("wfo");
      days[i] = "wfo";
    } else if(activeMode === "wfh") {
      dayEl.classList.add("wfh");
      days[i] = "wfh";
    } else if(activeMode === "leave") {
      dayEl.classList.add("leave");
      days[i] = "leave";
    } else if(activeMode === "clear") {
      days[i] = null;
    }
    updateSummary();
  });

  calendarEl.appendChild(dayEl);
}

// Update summary
function updateSummary(){
  const wfoCount = days.filter(d=>d==="wfo").length;
  const wfhCount = days.filter(d=>d==="wfh").length;
  const leaveCount = days.filter(d=>d==="leave").length;
  const total = wfoCount + wfhCount;

  let rto = total>0 ? ((wfoCount/total)*100).toFixed(2) : 0;

  summaryEl.textContent = `WFO: ${wfoCount} | WFH: ${wfhCount} | Leave: ${leaveCount} | RTO%: ${rto}%`;
}

// Export to Excel
exportBtn.addEventListener("click", () => {
  const ws_data = [["Date","Day","Status"]];
  for(let i=1;i<=totalDays;i++){
    const dateObj = new Date(year, month, i);
    const dayName = dayNames[dateObj.getDay()];
    const status = days[i] || "";
    ws_data.push([i, dayName, status]);
  }

  const wb = XLSX.utils.book_new();
  const ws = XLSX.utils.aoa_to_sheet(ws_data);
  XLSX.utils.book_append_sheet(wb, ws, "Attendance");
  XLSX.writeFile(wb, `${username}_${monthName}.xlsx`);
});
</script>
</body>
</html>
# Attendance-Tracker-Calculator
Attendance Tracker
