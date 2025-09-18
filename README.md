# Shaheen
Online earnings 
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Investment with Referral</title>
<style>
  body {font-family: Arial, sans-serif; margin:0; padding:0; background:#f4f4f4;}
  .container {max-width:500px; margin:auto; padding:20px;}
  .card {background:#fff; padding:20px; margin-bottom:15px; border-radius:8px; box-shadow:0 0 5px rgba(0,0,0,0.1);}
  input, button {width:100%; padding:10px; margin-top:10px;}
  .hidden {display:none;}
  .packages div {padding:10px; border-bottom:1px solid #ddd;}
</style>
</head>
<body>
<div class="container">
  <h2>Sign Up</h2>
  <div class="card" id="signupCard">
    <input type="text" id="name" placeholder="Name">
    <input type="email" id="email" placeholder="Email">
    <input type="text" id="phone" placeholder="Phone">
    <input type="text" id="referralInput" placeholder="Referral Code (optional)">
    <button onclick="signUp()">Sign Up</button>
  </div>

  <div class="card hidden" id="dashboard">
    <h3>Welcome <span id="userName"></span></h3>
    <p>Your Referral Code: <strong id="myCode"></strong></p>
    <p><b>Send your investment to JazzCash 03276942466 (Muhammad Saqlain)</b></p>
    <h4>Packages</h4>
    <div class="packages" id="packages"></div>
    <h4>Your Referral Earnings</h4>
    <div id="commissions"></div>
  </div>
</div>

<script>
const packagesData = [
  {amount:100, daily:5, days:30},
  {amount:200, daily:20, days:20},
  {amount:500, daily:30, days:30},
  {amount:1200, daily:100, days:30},
  {amount:2500, daily:200, days:30}
];

let currentUser = null;

function signUp(){
  const name = document.getElementById('name').value.trim();
  const email = document.getElementById('email').value.trim();
  const phone = document.getElementById('phone').value.trim();
  const refCode = document.getElementById('referralInput').value.trim();

  if(!name || !email || !phone){
    alert('Please fill all fields');
    return;
  }

  // generate random referral code
  const myCode = 'R' + Math.random().toString(36).substr(2,6).toUpperCase();

  const user = {name,email,phone,myCode,refCode,investments:[],commissions:0};
  localStorage.setItem('user_'+email, JSON.stringify(user));
  localStorage.setItem('currentUser', email);
  loadDashboard();
}

function loadDashboard(){
  const email = localStorage.getItem('currentUser');
  if(!email) return;
  currentUser = JSON.parse(localStorage.getItem('user_'+email));
  if(!currentUser) return;

  document.getElementById('signupCard').classList.add('hidden');
  document.getElementById('dashboard').classList.remove('hidden');
  document.getElementById('userName').textContent = currentUser.name;
  document.getElementById('myCode').textContent = currentUser.myCode;

  showPackages();
  showCommissions();
}

function showPackages(){
  const packagesDiv = document.getElementById('packages');
  packagesDiv.innerHTML = '';
  packagesData.forEach((p,i)=>{
    const div = document.createElement('div');
    div.innerHTML = `
      <b>Package ${i+1}</b>: Invest Rs.${p.amount}, get Rs.${p.daily} daily for ${p.days} days 
      <button onclick="invest(${i})">Invest</button>
    `;
    packagesDiv.appendChild(div);
  });
}

function invest(index){
  const p = packagesData[index];
  currentUser.investments.push(p);

  // handle referral commissions
  handleReferralCommissions(p.amount);

  localStorage.setItem('user_'+currentUser.email, JSON.stringify(currentUser));
  alert(`Investment of Rs.${p.amount} saved. Remember to send money to JazzCash manually.`);
  showCommissions();
}

function handleReferralCommissions(amount){
  // Level 1: 10%, Level 2: 5%, Level 3: 3%
  let refEmail = findUserByCode(currentUser.refCode);
  let level=1;
  while(refEmail && level<=3){
    let refUser = JSON.parse(localStorage.getItem('user_'+refEmail));
    if(refUser){
      let percent = (level===1?0.10:(level===2?0.05:0.03));
      let bonus = amount*percent;
      refUser.commissions = (refUser.commissions||0)+bonus;
      localStorage.setItem('user_'+refEmail, JSON.stringify(refUser));
      refEmail = findUserByCode(refUser.refCode);
    }
    level++;
  }
}

function findUserByCode(code){
  if(!code) return null;
  for(let i=0;i<localStorage.length;i++){
    const key = localStorage.key(i);
    if(key.startsWith('user_')){
      const u = JSON.parse(localStorage.getItem(key));
      if(u.myCode === code) return u.email;
    }
  }
  return null;
}

function showCommissions(){
  document.getElementById('commissions').textContent = 'Your total referral earnings: Rs.'+(currentUser.commissions||0);
}

// auto-load if already signed in
window.onload = loadDashboard;
</script>
</body>
</html>
