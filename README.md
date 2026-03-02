<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Attendance – Grade 12 Diamond</title>
<link href="https://fonts.googleapis.com/css2?family=Poppins:wght@400;600&display=swap" rel="stylesheet">

<!-- QR LIBRARY (FIXED LOAD) -->
<script src="https://unpkg.com/html5-qrcode@2.3.8/minified/html5-qrcode.min.js"></script>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/12.10.0/firebase-app.js";
import { getDatabase, ref, set, onValue } from "https://www.gstatic.com/firebasejs/12.10.0/firebase-database.js";

const firebaseConfig = {
  apiKey: "AIzaSyBU5wgWTIgFetO38U28ikmoLC0CKryR05M",
  authDomain: "talaan-b5c66.firebaseapp.com",
  databaseURL: "https://talaan-b5c66-default-rtdb.asia-southeast1.firebasedatabase.app",
  projectId: "talaan-b5c66",
  storageBucket: "talaan-b5c66.firebasestorage.app",
  messagingSenderId: "725034061224",
  appId: "1:725034061224:web:b2ac35e9e8f0f4c6e8a19d"
};

const app = initializeApp(firebaseConfig);
const db = getDatabase(app);

function getWeekKey() {
  let now = new Date();
  let day = now.getDay();
  let diff = now.getDate() - day + (day === 0 ? -6 : 1);
  let monday = new Date(now.setDate(diff));
  return "attendance_" + monday.toISOString().split("T")[0];
}

window.saveToDatabase = function(data, tardy) {
  set(ref(db, "attendance/" + getWeekKey()), { table: data, tardy: tardy });
};

window.listenToDatabase = function(callback) {
  onValue(ref(db, "attendance/" + getWeekKey()), (snapshot) => {
    if (snapshot.exists()) callback(snapshot.val());
  });
};
</script>

<style>
/* === YOUR ORIGINAL STYLE (UNCHANGED) === */
:root { --maroon: #7a0c0c; }
* { box-sizing: border-box; }
body { margin: 0; font-family: 'Poppins', sans-serif; background: #f2f2f2; }
#login { min-height: 100vh; display: flex; justify-content: center; align-items: center;
 background: linear-gradient(rgba(122,12,12,.85), rgba(122,12,12,.85)),
 url("https://images.unsplash.com/photo-1524995997946-a1c2e315a42f"); background-size: cover; background-position: center; padding: 15px; }
.login-box { background: #fff; width: 100%; max-width: 380px; padding: 25px; border-radius: 18px; box-shadow: 0 15px 30px rgba(0,0,0,.35); }
.login-box h2 { text-align: center; color: var(--maroon); }
input, select, button { width: 100%; padding: 12px; margin-top: 10px; font-size: 14px; }
button { background: var(--maroon); color: white; border: none; border-radius: 10px; cursor: pointer; }
#main { display: none; padding: 10px; }
header { background: var(--maroon); color: white; padding: 14px; text-align: center; border-radius: 14px; }
.sub { font-size: 13px; opacity: .9; }
#studentAction { display: none; margin-top: 10px; background: white; padding: 12px; border-radius: 12px; text-align: center; }
#timeNow { font-size: 13px; margin-bottom: 6px; }
.top-bar { margin-top: 12px; display: flex; justify-content: space-between; align-items: center; }
#searchInput { max-width: 260px; padding: 8px; border-radius: 8px; border: 1px solid #ccc; }
#dateDisplay { font-weight: 600; }
#adminReset { display: none; padding: 8px 12px; background: #b30000; color: white; border: none; border-radius: 8px; cursor: pointer; margin-left: 10px; }
.table-wrap { overflow-x: auto; background: white; border-radius: 14px; margin-top: 12px; }
table { border-collapse: collapse; width: 100%; min-width: 1200px; }
th, td { border: 2px solid #999; padding: 6px; text-align: center; font-size: 11px; min-width: 60px; }
th { background: #eee; font-weight: 600; }
.day-head { background: var(--maroon); color: white; border-right: 4px solid black; }
.sticky { position: sticky; left: 0; background: white; z-index: 4; font-weight: 600; }
.name { left: 42px; z-index: 5; background: white; }
.summary-col { background: #f7f7f7; min-width: 160px; font-weight: 600; }
.P { background: #c8f7c5; font-weight: 600; }
.T { background: #fff3b0; font-weight: 600; }
.C { background: #ffb3b3; font-weight: 600; }
.A { background: #e0e0e0; font-weight: 600; }
.E { background: #c4d9ff; font-weight: 600; }
#qrAuthInterface { display: none; padding: 20px; text-align: center; }
#qrVideo { width: 100%; max-width: 400px; }
#qrStatus { color: red; font-size: 13px; margin-top: 6px; }
</style>
</head>

<body>

<!-- LOGIN -->
<div id="login">
  <div class="login-box">
    <h2>Attendance Login</h2>
    <select id="role">
      <option>Student</option>
      <option>Teacher</option>
      <option>Student Officer</option>
    </select>
    <input id="user" placeholder="Last Name">
    <input id="pass" type="password" placeholder="Student Number">
    <input id="adminPass" type="password" placeholder="Admin Password" style="display:none">
    <button onclick="loginUser()">Log in</button>
  </div>
</div>

<!-- QR AUTH -->
<div id="qrAuthInterface">
  <h2>QR Code Authentication</h2>
  <div id="qrVideo"></div>
  <p id="qrStatus"></p>
  <button onclick="startQRScanner()">Start Scanner</button>
  <button onclick="cancelQR()">Cancel</button>
</div>

<!-- MAIN -->
<div id="main">
  <header>
    DAILY ATTENDANCE – GRADE 12 DIAMOND
    <div class="sub">Adviser: January Lyn Alumbres</div>
  </header>

  <div id="studentAction">
    <div id="timeNow"></div>
    <button onclick="markPresent()">MARK PRESENT</button>
  </div>

  <div class="top-bar">
    <div>
      <span id="dateDisplay"></span>
      <button id="adminReset" onclick="manualReset()">RESET WEEK</button>
    </div>
    <input type="text" id="searchInput" placeholder="🔍 Search student...">
  </div>

  <div class="table-wrap">
    <table>
      <thead>
        <tr>
          <th rowspan="2" class="sticky">#</th>
          <th rowspan="2" class="sticky name">Student</th>
          <th colspan="35" class="day-head">WEEK</th>
          <th rowspan="2" class="summary-col">SUMMARY</th>
        </tr>
      </thead>
      <tbody id="tbody"></tbody>
    </table>
  </div>
</div>

<script>
const adminPassword="123456789";
let roleType="", loggedStudent="", tardyMinutesData={}, html5QrCode=null;

/* ================= FIXED QR SCANNER ================= */
function startQRScanner(){
  const status=document.getElementById("qrStatus");
  status.style.color="black";
  status.textContent="Accessing camera...";

  if(html5QrCode){
    html5QrCode.stop().then(()=>{
      html5QrCode.clear();
    }).catch(()=>{});
  }

  html5QrCode=new Html5Qrcode("qrVideo");

  Html5Qrcode.getCameras().then(devices=>{
    if(!devices.length){
      status.style.color="red";
      status.textContent="No camera found!";
      return;
    }

    html5QrCode.start(
      devices[0].id,
      {fps:10, qrbox:250},
      (decodedText)=>{
        status.style.color="green";
        status.textContent="QR Verified!";
        setTimeout(()=>{
          qrAuthInterface.style.display="none";
          main.style.display="block";
        },500);
      }
    ).catch(()=>{
      status.style.color="red";
      status.textContent="Camera permission denied.";
    });

  }).catch(()=>{
    status.style.color="red";
    status.textContent="Camera access error.";
  });
}

function cancelQR(){
  if(html5QrCode){
    html5QrCode.stop().then(()=>html5QrCode.clear()).catch(()=>{});
  }
  qrAuthInterface.style.display="none";
  login.style.display="block";
}
</script>

</body>
</html>
