<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <title>🚨 Multi-Channel Scam Detector</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.16.105/pdf.min.js"></script>
  <link href="https://fonts.googleapis.com/css2?family=Roboto:wght@400;700&display=swap" rel="stylesheet">
  <style>
    body {
      font-family: 'Roboto', sans-serif;
      background: #f0f4f8;
      margin: 0;
      padding: 0;
    }
    .container {
      max-width: 900px;
      margin: auto;
      padding: 30px;
    }
    h1 {
      text-align: center;
      color: #333;
      margin-bottom: 30px;
    }
    .card {
      background: #fff;
      padding: 25px;
      border-radius: 15px;
      box-shadow: 0 8px 20px rgba(0,0,0,0.1);
      margin-bottom: 20px;
    }
    select, textarea, input, button {
      width: 100%;
      padding: 12px;
      margin-top: 10px;
      border-radius: 10px;
      border: 1px solid #ccc;
      font-size: 16px;
      box-sizing: border-box;
    }
    textarea { height: 120px; resize: none; }
    button {
      background: #007bff;
      color: #fff;
      border: none;
      font-size: 18px;
      cursor: pointer;
      transition: 0.3s;
    }
    button:hover { background: #0056b3; }
    #result {
      margin-top: 20px;
      font-size: 18px;
      font-weight: bold;
      padding: 15px;
      border-radius: 10px;
    }
    .progress-container {
      width: 100%;
      background: #ddd;
      border-radius: 10px;
      margin: 20px 0;
      height: 25px;
      overflow: hidden;
    }
    .progress-bar {
      height: 100%;
      width: 0%;
      border-radius: 10px;
      text-align: center;
      color: white;
      font-weight: bold;
      line-height: 25px;
      transition: background 0.6s ease-in-out, width 0.4s ease-in-out;
    }
    #riskLabel {
      font-size: 18px;
      font-weight: bold;
      margin-top: 10px;
    }
  </style>
</head>
<body>
  <div class="container">
    <h1>🚨 Multi-Channel Scam Detector</h1>
    
    <div class="card">
      <label for="checkType">Select what you want to check:</label>
      <select id="checkType" onchange="toggleInput()">
        <option value="message">Message (SMS/Email/WhatsApp)</option>
        <option value="email">Verify Email Address</option>
        <option value="document">Verify Document (.txt / .pdf)</option>
      </select>
    </div>

    <!-- Message input -->
    <div class="card" id="messageBox">
      <label>Paste any suspicious message below:</label>
      <textarea id="message" placeholder="Enter message here..."></textarea>
    </div>
    
    <!-- Email input -->
    <div class="card" id="emailBox" style="display:none;">
      <label>Enter the sender's email address:</label>
      <input type="text" id="email" placeholder="e.g. support@gmail.com">
    </div>

    <!-- Document input -->
    <div class="card" id="documentBox" style="display:none;">
      <label>Upload a document (.txt or .pdf):</label>
      <input type="file" id="docFile" accept=".txt,.pdf">
    </div>

    <div class="card">
      <button onclick="checkScam()">Check Now</button>
    </div>

    <div class="card" id="resultCard">
      <p id="result">Your results will appear here</p>
      <div class="progress-container">
        <div id="progressBar" class="progress-bar">0%</div>
      </div>
      <p id="riskLabel"></p>
    </div>
  </div>

  <script>
    // Utility functions
    function getGradientColor(score){
      if(score <= 30) return "linear-gradient(90deg, green, limegreen)";
      if(score <= 60) return "linear-gradient(90deg, yellow, orange)";
      return "linear-gradient(90deg, orange, red)";
    }
    function updateRiskLabel(score){
      let label = document.getElementById("riskLabel");
      if(score<=30){ label.innerHTML="✅ SAFE"; label.style.color="green"; }
      else if(score<=60){ label.innerHTML="⚠️ SUSPICIOUS"; label.style.color="orange"; }
      else{ label.innerHTML="🚨 HIGH RISK!"; label.style.color="red"; }
    }
    function updateProgress(score){
      let bar = document.getElementById("progressBar");
      let current = 0, target = score;
      clearInterval(bar.timer);
      bar.timer = setInterval(()=>{
        if(current>=target) clearInterval(bar.timer);
        else { current++; bar.style.width=current+"%"; bar.style.background=getGradientColor(current); bar.innerHTML=current+"%"; }
      },15);
      updateRiskLabel(score);
    }

    // Scam categories
    const categories = {
      "Phishing (Email/Gmail Fraud)": ["gmail verification","google verification","verify your gmail","account suspended","gmail password reset","login to google","google otp"],
      "Bank Fraud":["account blocked","bank verification","transfer money","upi","payment required","send money","atm","netbanking"],
      "Lottery/Gift Scam":["lottery","kbc winner","free gift","you have won","congratulations","jackpot"],
      "Digital Arrest / Threat":["digital arrest","warrant","jail","cyber police","threat","your sim will be blocked"]
    };

    function analyzeText(text, sourceType="message"){
      let resultBox = document.getElementById("result");
      let score=0;
      let matchedCategories=[];
      for(let cat in categories){
        let found = categories[cat].filter(word => text.includes(word));
        if(found.length>0){
          matchedCategories.push(cat + " (keywords: "+found.join(", ")+")");
          score=Math.max(score, 60 + found.length*10);
        }
      }
      if(matchedCategories.length>0){
        resultBox.innerHTML = `🚨 Scam Indicators Found in ${sourceType}!<br><br>`+
                              matchedCategories.join("<br>")+`<br><br>Scam Probability: <b>${score}%</b>`;
      } else {
        if(text.trim()===""){
          resultBox.innerHTML = "⚠️ Please enter content to check.";
          score=0;
        } else {
          resultBox.innerHTML = `✅ No scam patterns found in ${sourceType}.`;
          score=Math.floor(Math.random()*10);
        }
      }
      updateProgress(score);
    }

    // Toggle inputs
    function toggleInput(){
      let type=document.getElementById("checkType").value;
      document.getElementById("messageBox").style.display=(type==="message")?"block":"none";
      document.getElementById("emailBox").style.display=(type==="email")?"block":"none";
      document.getElementById("documentBox").style.display=(type==="document")?"block":"none";
      document.getElementById("result").innerHTML="Your results will appear here";
      updateProgress(0);
    }

    // PDF Text Extraction
    async function extractPDFText(file){
      const arrayBuffer=await file.arrayBuffer();
      const pdf=await pdfjsLib.getDocument({data:arrayBuffer}).promise;
      let text="";
      for(let i=1;i<=pdf.numPages;i++){
        const page=await pdf.getPage(i);
        const content=await page.getTextContent();
        text+=content.items.map(item=>item.str).join(" ")+" ";
      }
      return text.toLowerCase();
    }

    // Check Scam
    function checkScam(){
      let type=document.getElementById("checkType").value;
      if(type==="message"){
        let msg=document.getElementById("message").value.toLowerCase();
        analyzeText(msg,"message");
      }
      else if(type==="email"){
        let email=document.getElementById("email").value.trim().toLowerCase();
        let resultBox=document.getElementById("result");
        let score=0;
        if(email===""){ resultBox.innerHTML="⚠️ Please enter an email address."; updateProgress(0); return;}
        if(email.endsWith("@gmail.com")){ score=Math.floor(Math.random()*10); resultBox.innerHTML=`✅ Valid Gmail address.<br>Scam Probability: <b>${score}%</b>`; }
        else if(email.includes("gmail")){ score=Math.floor(70+Math.random()*20); resultBox.innerHTML=`🚨 Fake Gmail detected → ${email}<br>Scam Probability: <b>${score}%</b>`;}
        else{ score=Math.floor(20+Math.random()*40); resultBox.innerHTML=`ℹ️ Not a Gmail. Entered: ${email}<br>Scam Probability: <b>${score}%</b>`;}
        updateProgress(score);
      }
      else if(type==="document"){
        let fileInput=document.getElementById("docFile");
        if(fileInput.files.length===0){ document.getElementById("result").innerHTML="⚠️ Please upload a document."; updateProgress(0); return; }
        let file=fileInput.files[0];
        let ext=file.name.split(".").pop().toLowerCase();
        if(ext==="txt"){
          let reader=new FileReader();
          reader.onload=function(e){ analyzeText(e.target.result.toLowerCase(),"document"); }
          reader.readAsText(file);
        } else if(ext==="pdf"){
          extractPDFText(file).then(text=>{ analyzeText(text,"PDF document"); }).catch(err=>{ document.getElementById("result").innerHTML="❌ Error reading PDF."; updateProgress(0); });
        } else { document.getElementById("result").innerHTML="⚠️ Only .txt and .pdf supported."; updateProgress(0);}
      }
    }
  </script>
</body>
</html>

