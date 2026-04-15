# wingo-ai
wingohack100
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>WINGO ULTRA AI</title>

<style>
body{
  margin:0;
  background:#000;
  color:#00ffcc;
  font-family:monospace;
  text-align:center;
}

.container{
  max-width:400px;
  margin:auto;
  padding:20px;
}

h1{
  color:#b300ff;
  text-shadow:0 0 15px #b300ff;
}

.card{
  background:#050505;
  border:1px solid #00ffcc;
  padding:20px;
  border-radius:12px;
  box-shadow:0 0 20px #00ffcc;
}

.main{
  font-size:42px;
  margin:20px 0;
}

.big{color:#00ff88;}
.small{color:#7000ff;}

.timer{
  margin:10px 0;
  color:#ffcc00;
}

.conf{
  margin:10px;
}

table{
  width:100%;
  margin-top:15px;
  font-size:12px;
}

td,th{
  padding:8px;
  border-bottom:1px solid #222;
}
</style>
</head>

<body>

<div class="container">

<h1>WINGO AI SIGNAL</h1>

<div class="card">

<div>Period: <span id="period">--</span></div>

<div class="timer" id="timer">00:60</div>

<div id="result" class="main">WAIT...</div>

<div id="confidence" class="conf"></div>

<div>Opposite: <span id="opp">--</span></div>

</div>

<table>
<thead>
<tr>
<th>ID</th>
<th>PRED</th>
<th>RES</th>
<th>STATUS</th>
</tr>
</thead>
<tbody id="history"></tbody>
</table>

</div>

<script>
class AI{
constructor(){
this.api="https://draw.ar-lottery01.com/WinGo/WinGo_1M/GetHistoryIssuePage.json";
this.last=null;
this.pred=null;
this.wins=0;
this.total=0;
this.init();
}

init(){
this.timer();
this.fetch();
setInterval(()=>this.fetch(),3000);
}

timer(){
setInterval(()=>{
let s=new Date().getSeconds();
let r=60-s;
document.getElementById("timer").innerText="00:"+ (r<10?"0"+r:r);
},1000);
}

analyze(list){

let data=list.slice(0,30).map(x=>parseInt(x.number));

let score=0;

for(let i=0;i<10;i++){
score+=(data[i]>=5?1:-1)*(10-i);
}

let last3=data.slice(0,3);
let streakBig=last3.every(n=>n>=5);
let streakSmall=last3.every(n=>n<5);

let pred,conf;

if(streakBig){pred="SMALL";conf=85;}
else if(streakSmall){pred="BIG";conf=85;}
else{
pred=score>=0?"BIG":"SMALL";
conf=60+Math.abs(score/5);
}

return {pred,conf:Math.min(95,Math.floor(conf))};
}

async fetch(){
try{
let r=await fetch(this.api);
let d=await r.json();

let cur=d.data.list[0];
let issue=cur.issueNumber;

let next=(BigInt(issue)+1n).toString();
document.getElementById("period").innerText=next.slice(-4);

if(this.last!==issue){

if(this.last && this.pred){
let num=parseInt(cur.number);
let type=num>=5?"BIG":"SMALL";
let win=this.pred===type;

this.log(this.last.slice(-4),this.pred,num,win);
}

this.last=issue;

let ai=this.analyze(d.data.list);
this.pred=ai.pred;

document.getElementById("result").innerHTML=
"<span class='"+(ai.pred==="BIG"?"big":"small")+"'>"+ai.pred+"</span>";

document.getElementById("confidence").innerText=
"Confidence: "+ai.conf+"%";

let pool=ai.pred==="BIG"?[0,1,2,3,4]:[5,6,7,8,9];
document.getElementById("opp").innerText=
pool.sort(()=>0.5-Math.random()).slice(0,3).join(",");

}

}catch(e){
document.getElementById("result").innerText="API ERROR";
}
}

log(id,pred,num,win){

this.total++;
if(win) this.wins++;

let row=document.createElement("tr");

row.innerHTML=`
<td>${id}</td>
<td>${pred}</td>
<td>${num}</td>
<td style="color:${win?"#00ff88":"#ff3333"}">${win?"WIN":"LOSS"}</td>
`;

document.getElementById("history").prepend(row);

if(document.getElementById("history").children.length>10){
document.getElementById("history").lastChild.remove();
}
}
}

new AI();
</script>

</body>
</html>
