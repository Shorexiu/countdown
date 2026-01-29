<!DOCTYPE html>
<html lang="zh-CN">
<head>
<meta charset="UTF-8">
<title>生活倒计时</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600&display=swap" rel="stylesheet">

<style>
*{box-sizing:border-box}
body{
  margin:0;
  min-height:100vh;
  font-family:"Inter","微软雅黑",sans-serif;
  background:linear-gradient(180deg,#f9fafb,#eef2f7);
  display:flex;
  justify-content:center;
  align-items:center;
}
.container{max-width:780px;width:100%;padding:24px}
h1{text-align:center;margin-bottom:6px}
.subtitle{text-align:center;color:#6b7280;margin-bottom:20px}

.form,.card{
  background:#fff;
  border-radius:16px;
  padding:18px;
  box-shadow:0 8px 20px rgba(0,0,0,.05);
  margin-bottom:14px;
}
.form-row{display:flex;gap:10px;margin-bottom:10px;flex-wrap:wrap}
input{
  flex:1;
  padding:10px;
  border-radius:10px;
  border:1px solid #e5e7eb;
}
button{
  padding:10px 14px;
  border:none;
  border-radius:10px;
  background:#111827;
  color:#fff;
  cursor:pointer;
}
.delete{color:#ef4444;cursor:pointer;font-size:13px}

.card-header{
  display:flex;
  justify-content:space-between;
  align-items:center;
  margin-bottom:6px;
}
.title{font-size:18px;font-weight:600;cursor:pointer}
.timer{font-family:monospace;font-size:20px;margin:8px 0}
.note{color:#6b7280;font-size:14px;cursor:pointer}

.progress-bar{
  background:#e5e7eb;
  border-radius:999px;
  height:8px;
  overflow:hidden;
  margin-top:8px;
}
.progress{
  height:100%;
  background:linear-gradient(90deg,#6366f1,#3b82f6);
}
.percent{text-align:right;font-size:12px;color:#6b7280;margin-top:4px}

.edit input{
  width:100%;
  margin-bottom:6px;
}
.edit button{
  width:100%;
  margin-top:6px;
}
footer{text-align:center;font-size:12px;color:#9ca3af;margin-top:12px}
</style>
</head>

<body>
<div class="container">
  <h1>生活倒计时</h1>
  <div class="subtitle">重要的日子，慢慢靠近</div>

  <!-- 新增 -->
  <div class="form">
    <div class="form-row">
      <input id="name" placeholder="事件名称">
      <input id="date" type="datetime-local">
    </div>
    <div class="form-row">
      <input id="note" placeholder="备注（可选）">
    </div>
    <button onclick="add()">添加倒计时</button>
  </div>

  <div id="list"></div>
  <footer>点击卡片可直接编辑 · 数据保存在本地</footer>
</div>

<script>
let events = JSON.parse(localStorage.getItem("events")) || [
  {
    name:"春节",
    target:"2026-02-14T20:00",
    note:"回家的日子值得等待",
    start:Date.now()
  }
];

function save(){localStorage.setItem("events",JSON.stringify(events))}

function add(){
  const n=name.value,d=date.value;
  if(!n||!d) return alert("请填写名称和时间");
  events.push({name:n,target:d,note:note.value,start:Date.now()});
  save();render();
  name.value=date.value=note.value="";
}

function del(i){events.splice(i,1);save();render()}

function format(ms){
  if(ms<=0) return "已完成";
  const d=Math.floor(ms/86400000);
  const h=Math.floor(ms/3600000)%24;
  const m=Math.floor(ms/60000)%60;
  const s=Math.floor(ms/1000)%60;
  return `${d}天 ${h}小时 ${m}分 ${s}秒`;
}

function render(){
  const now=Date.now();
  list.innerHTML="";
  events.forEach((e,i)=>{
    const total=new Date(e.target).getTime()-e.start;
    const left=new Date(e.target).getTime()-now;
    let percent=Math.min(100,Math.max(0,((total-left)/total*100)));
    list.innerHTML+=`
    <div class="card">
      <div class="card-header">
        <div class="title" onclick="edit(${i})">${e.name}</div>
        <div class="delete" onclick="del(${i})">删除</div>
      </div>
      <div class="timer">${format(left)}</div>
      <div class="note" onclick="edit(${i})">${e.note||"点击添加备注"}</div>
      <div class="progress-bar">
        <div class="progress" style="width:${percent}%"></div>
      </div>
      <div class="percent">${percent.toFixed(1)}%</div>
    </div>`;
  });
}

function edit(i){
  const e=events[i];
  list.children[i].innerHTML=`
    <div class="edit">
      <input value="${e.name}" id="en">
      <input type="datetime-local" value="${e.target}" id="ed">
      <input value="${e.note||""}" id="et">
      <button onclick="saveEdit(${i})">保存</button>
    </div>`;
}

function saveEdit(i){
  events[i].name=en.value;
  events[i].target=ed.value;
  events[i].note=et.value;
  save();render();
}

render();
setInterval(render,1000);
</script>
</body>
</html>
