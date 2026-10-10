# My-Voice-Ai
My Personal Voice AI App
<!DOCTYPE html>
<html lang="my">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>MY VOICE AI</title>
<style>
*{box-sizing:border-box}
body{margin:0;font-family:Arial,sans-serif;background:#101827;color:white;padding:20px}
.container{max-width:500px;margin:auto}
h1{text-align:center;color:#60e5bd}
.subtitle{text-align:center;color:#b9c5d5}
.card{background:#1e293b;padding:18px;border-radius:16px;margin:18px 0}
label{display:block;margin-bottom:10px;font-weight:bold}
textarea,input[type=text]{width:100%;padding:12px;border:1px solid #526078;border-radius:10px;background:#111827;color:white;font-size:16px}
textarea{height:130px;resize:vertical}
button{border:0;border-radius:10px;padding:12px 15px;margin:5px 2px;cursor:pointer;font-size:15px;font-weight:bold}
.primary{background:#14b88a;color:#06271e}
.secondary{background:#36445b;color:white}
#status{color:#60e5bd;line-height:1.6}
audio{width:100%;margin-top:12px}
.note{font-size:13px;color:#c0ccda;line-height:1.6}
</style>
</head>
<body>
<div class="container">
<h1>🎙️ MY VOICE AI</h1>
<p class="subtitle">ကိုယ်ပိုင်အသံ AI App</p>

<div class="card">
<label>၁။ ကိုယ့်အသံကို အသံသွင်းပါ</label>
<button class="primary" id="start">🔴 အသံစသွင်းမယ်</button>
<button class="secondary" id="stop" disabled>⏹️ ရပ်မယ်</button>
<p id="status">အသံသွင်းဖို့ ခလုတ်ကိုနှိပ်ပါ။</p>
<audio id="player" controls hidden></audio>
</div>

<div class="card">
<label>၂။ အသံဖိုင်တင်ပါ</label>
<input type="file" id="audioFile" accept="audio/*">
<p class="note">MP3၊ WAV သို့မဟုတ် ဖုန်းက အသံဖိုင်တစ်ခုကို ရွေးနိုင်ပါတယ်။</p>
</div>

<div class="card">
<label>၃။ ဖတ်စေချင်တဲ့စာ ရိုက်ပါ</label>
<textarea id="script" placeholder="ဒီနေရာမှာ မြန်မာစာ ရိုက်ထည့်ပါ..."></textarea>
<button class="primary" id="speak">🔊 ဖုန်းအသံနဲ့ စမ်းဖတ်မယ်</button>
<p class="note">ဒီခလုတ်က ဖုန်းမှာရှိတဲ့ Text-to-Speech ကို စမ်းသုံးတာဖြစ်ပြီး ကိုယ့်အသံ Clone မလုပ်သေးပါ။</p>
</div>

<div class="card">
<label>၄။ အသံဖိုင်ဖွင့်ရန်</label>
<audio id="uploadedPlayer" controls hidden></audio>
<p class="note">ကိုယ်အသံသွင်းထားတဲ့အသံ သို့မဟုတ် တင်ထားတဲ့ဖိုင်ကို ဒီမှာ ပြန်နားထောင်နိုင်ပါတယ်။</p>
</div>

<p class="note">MY VOICE AI • ပထမဆုံး စမ်းသပ်ဗားရှင်း</p>
</div>

<script>
let recorder=null;
let chunks=[];
let stream=null;
let recordedURL=null;

const startBtn=document.getElementById('start');
const stopBtn=document.getElementById('stop');
const statusText=document.getElementById('status');
const player=document.getElementById('player');
const uploadPlayer=document.getElementById('uploadedPlayer');

startBtn.addEventListener('click',async()=>{
 try{
  if(!navigator.mediaDevices||!navigator.mediaDevices.getUserMedia){
   throw new Error('ဒီ Preview မှာ Microphone မရနိုင်ပါ။ HTTPS ပါတဲ့ Browser မှာ စမ်းပါ။');
  }
  stream=await navigator.mediaDevices.getUserMedia({audio:true});
  chunks=[];
  let mime='';
  if(MediaRecorder.isTypeSupported('audio/webm;codecs=opus')){
   mime='audio/webm;codecs=opus';
  }else if(MediaRecorder.isTypeSupported('audio/mp4')){
   mime='audio/mp4';
  }
  recorder=mime?new MediaRecorder(stream,{mimeType:mime}):new MediaRecorder(stream);
  recorder.ondataavailable=(event)=>{
   if(event.data&&event.data.size>0)chunks.push(event.data);
  };
  recorder.onstop=()=>{
   const blob=new Blob(chunks,{type:recorder.mimeType||'audio/webm'});
   if(recordedURL)URL.revokeObjectURL(recordedURL);
   recordedURL=URL.createObjectURL(blob);
   player.src=recordedURL;
   player.hidden=false;
   statusText.textContent='အသံသွင်းပြီးပါပြီ။ Play ကိုနှိပ်ပြီး ကိုယ့်အသံကို နားထောင်ပါ။';
   stream.getTracks().forEach(track=>track.stop());
  };
  recorder.start();
  startBtn.disabled=true;
  stopBtn.disabled=false;
  statusText.textContent='🔴 အသံသွင်းနေပါပြီ။ စကားပြောပါ။';
 }catch(err){
  statusText.textContent='အသံသွင်းမရပါ: '+err.message+' ။ Browser ရဲ့ Microphone Permission ကိုစစ်ပါ။';
 }
});

stopBtn.addEventListener('click',()=>{
 if(recorder&&recorder.state==='recording'){
  recorder.stop();
  startBtn.disabled=false;
  stopBtn.disabled=true;
 }
});

document.getElementById('audioFile').addEventListener('change',(event)=>{
 const file=event.target.files[0];
 if(file){
  uploadPlayer.src=URL.createObjectURL(file);
  uploadPlayer.hidden=false;
  statusText.textContent='အသံဖိုင်တင်ပြီးပါပြီ။';
 }
});

document.getElementById('speak').addEventListener('click',()=>{
 const text=document.getElementById('script').value.trim();
 if(!text){alert('အရင်ဆုံး ဖတ်စေချင်တဲ့စာ ရိုက်ထည့်ပါ။');return;}
 if(!('speechSynthesis' in window)){
  alert('ဒီ Browser မှာ Text-to-Speech မရနိုင်ပါ။');
  return;
 }
 window.speechSynthesis.cancel();
 const utterance=new SpeechSynthesisUtterance(text);
 utterance.lang='my-MM';
 utterance.rate=0.9;
 window.speechSynthesis.speak(utterance);
});
</script>
</body>
</html> 

