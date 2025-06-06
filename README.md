from IPython.display import HTML

# HTML code for dual comparison simulator
html_content = """<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>원자 시뮬레이터 비교 V9</title>
  <style>
    body { font-family: Arial; background: #f4f4f4; text-align: center; padding: 20px; }
    .container { display: flex; justify-content: space-around; }
    .sim { background: white; padding: 15px; border-radius: 10px; width: 45%; }
    .info { text-align: left; margin-top: 10px; }
    .info p { margin: 4px 0; }
    .orbital { text-align: left; margin-top: 10px; }
    .orb-line { margin: 6px 0; }
    .orb-cell { display: inline-block; width: 30px; height: 40px; margin: 2px; border: 1px solid #000; position: relative; }
    .up { position: absolute; top: 50%; left: 20%; transform: translate(-50%, -50%); font-size:18px; }
    .down { position: absolute; top: 50%; left: 80%; transform: translate(-50%, -50%); font-size:18px; }
    canvas { background: white; border: 1px solid #ccc; margin-top: 10px; }
  </style>
</head>
<body>
  <h2>🌟 원자 시뮬레이터 비교 V9 🌟</h2>
  <div class="container">
    <!-- Simulator 1 -->
    <div class="sim">
      <h3>원소 A</h3>
      <label>원소 선택:
        <select id="sel1"></select>
      </label>
      <label><input type="checkbox" id="ion1"> 이온 모드</label>
      <canvas id="cv1" width="300" height="300"></canvas>
      <div class="info">
        <p>기호: <span id="symbol1">-</span></p>
        <p>전자배치: <span id="config1">-</span></p>
        <p>전하: <span id="charge1">-</span></p>
      </div>
      <div class="orbital" id="orb1"><h4>오비탈</h4></div>
    </div>
    <!-- Simulator 2 -->
    <div class="sim">
      <h3>원소 B</h3>
      <label>원소 선택:
        <select id="sel2"></select>
      </label>
      <label><input type="checkbox" id="ion2"> 이온 모드</label>
      <canvas id="cv2" width="300" height="300"></canvas>
      <div class="info">
        <p>기호: <span id="symbol2">-</span></p>
        <p>전자배치: <span id="config2">-</span></p>
        <p>전하: <span id="charge2">-</span></p>
      </div>
      <div class="orbital" id="orb2"><h4>오비탈</h4></div>
    </div>
  </div>

  <button onclick="drawAll()">비교 보기</button>

  <script>
    const elems = [
      {Z:1,symbol:"H",orb:[1],radius:53},
      {Z:2,symbol:"He",orb:[2],radius:31},
      {Z:3,symbol:"Li",orb:[2,1],radius:167},
      {Z:4,symbol:"Be",orb:[2,2],radius:112},
      {Z:5,symbol:"B",orb:[2,2,1],radius:87},
      {Z:6,symbol:"C",orb:[2,2,2],radius:67},
      {Z:7,symbol:"N",orb:[2,2,3],radius:56},
      {Z:8,symbol:"O",orb:[2,2,4],radius:48},
      {Z:9,symbol:"F",orb:[2,2,5],radius:42},
      {Z:10,symbol:"Ne",orb:[2,2,6],radius:38},
      {Z:11,symbol:"Na",orb:[2,2,6,1],radius:186},
      {Z:12,symbol:"Mg",orb:[2,2,6,2],radius:160},
      {Z:13,symbol:"Al",orb:[2,2,6,2,1],radius:143},
      {Z:14,symbol:"Si",orb:[2,2,6,2,2],radius:118},
      {Z:15,symbol:"P",orb:[2,2,6,2,3],radius:110},
      {Z:16,symbol:"S",orb:[2,2,6,2,4],radius:104},
      {Z:17,symbol:"Cl",orb:[2,2,6,2,5],radius:99},
      {Z:18,symbol:"Ar",orb:[2,2,6,2,6],radius:71},
      {Z:19,symbol:"K",orb:[2,2,6,2,6,1],radius:227},
      {Z:20,symbol:"Ca",orb:[2,2,6,2,6,2],radius:197}
    ];
    const labels = ["1s","2s","2p","3s","3p","4s"];
    const maxe   = [2,2,6,2,6,2];

    function makeConfig(orb){
      return orb.map((e,i)=> e>0? labels[i]+"<sup>"+e+"</sup>": "")
                .filter(x=>x).join(" ");
    }

    window.onload = ()=>{
      for(let idx=1;idx<=2;idx++){
        const sel = document.getElementById("sel"+idx);
        elems.forEach(x=>{
          const opt = document.createElement("option");
          opt.value = x.Z;
          opt.textContent = x.symbol+" ("+x.Z+")";
          sel.appendChild(opt);
        });
      }
    };

    function drawSim(idx){
      const sel = document.getElementById("sel"+idx);
      const ion = document.getElementById("ion"+idx).checked;
      const Z = +sel.value;
      let e0 = elems.find(x=>x.Z===Z), e = JSON.parse(JSON.stringify(e0));
      let charge=0;
      if(ion){
        const last=e.orb.length-1;
        if([11,12,13,19,20].includes(Z)){e.orb[last]--;charge=1;}
        if([9,16,17,8].includes(Z)){e.orb[last]++;charge=-1;}
        if(Z===1){e.orb[0]--;charge=1;}
      }
      // update info
      document.getElementById("symbol"+idx).innerText = e.symbol + (charge? (charge>0?"+"+charge:charge): "");
      document.getElementById("config"+idx).innerHTML = makeConfig(e.orb);
      document.getElementById("charge"+idx).innerText = charge===0?"중성":(charge>0? "+"+charge:charge);
      // draw canvas
      const cv = document.getElementById("cv"+idx), ctx = cv.getContext("2d");
      ctx.clearRect(0,0,cv.width,cv.height);
      // neutral orbits
      ctx.strokeStyle="#ccc";
      e0.orb.forEach((_,i)=>{
        let r0=30+25*i;
        ctx.beginPath();ctx.arc(150,150,r0,0,2*Math.PI);ctx.stroke();
      });
      // final orbits
      ctx.strokeStyle="#000";
      e.orb.forEach((_,i)=>{
        let factor = ion? (charge>0?0.8:1.2):1;
        let r=(30+25*i)*factor;
        ctx.beginPath();ctx.arc(150,150,r,0,2*Math.PI);ctx.stroke();
        // electrons
        e.orb[i]&&[...Array(e.orb[i]).keys()].forEach(j=>{
          let ang=2*Math.PI*j/e.orb[i];
          let x=150+r*Math.cos(ang),y=150+r*Math.sin(ang);
          ctx.beginPath();ctx.arc(x,y,4,0,2*Math.PI);ctx.fill();
        });
      });
      ctx.fillStyle="#444";ctx.beginPath();ctx.arc(150,150,12,0,2*Math.PI);ctx.fill();
      // orbitals
      const orbDiv=document.getElementById("orb"+idx);
      orbDiv.innerHTML="<h4>오비탈</h4>";
      e.orb.forEach((_,i)=>{
        let div=document.createElement("div");div.className="orb-line";
        div.innerHTML="<strong>"+labels[i]+":</strong> ";
        let cells=maxe[i]===2?1:3;let arr=Array(cells).fill(0);
        for(let j=0;j<e.orb[i];j++)arr[j<cells?j:j-cells]++;
        arr.forEach(v=>{
          let cell=document.createElement("div");cell.className="orb-cell";
          if(v>=1){let up=document.createElement("div");up.className="up";up.textContent="↑";cell.appendChild(up);}
          if(v>=2){let dn=document.createElement("div");dn.className="down";dn.textContent="↓";cell.appendChild(dn);}
          div.appendChild(cell);
        });
        orbDiv.appendChild(div);
      });
      ctx.strokeStyle="#000";
    }

    function drawAll(){
      drawSim(1); drawSim(2);
    }
  </script>
</body>
</html>"""

# Write file
file_path = "/mnt/data/원자_시뮬레이터_V9.html"
with open(file_path, "w", encoding="utf-8") as f:
    f.write(html_content)

# Display link
HTML(f'<a href="sandbox:{file_path}" target="_blank">원자_시뮬레이터_V9.html 다운로드</a>')
