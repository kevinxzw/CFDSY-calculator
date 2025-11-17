<!DOCTYPE html>
<html lang="zh">
<head>
<meta charset="UTF-8">
<title>圆形双层钢管夹层混凝土柱 承载力计算（含稳定性 φ）</title>

<script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>

<style>
    body {
        margin: 0;
        background: #f2f4f8;
        font-family: "Times New Roman", serif;
        display: flex;
        flex-direction: column;
        align-items: center;
    }

    /* 顶部示意图框 */
    .header-box {
        width: 90%;
        max-width: 900px;
        background: #fff;
        padding: 20px;
        border: 1px solid #000;
        border-radius: 6px;
        margin-top: 20px;
    }

    .pic-container {
        text-align: center;
        margin-bottom: 15px;
    }

    .pic-container img {
        max-width: 40%;  /* ★缩小示意图 */
        height: auto;
    }

    .param-table {
        width: 100%;
        font-size: 18px;
    }

    .param-table td {
        padding: 6px 8px;
    }

    .section-card {
        width: 90%;
        max-width: 900px;
        background: #fff;
        padding: 25px;
        border-radius: 10px;
        box-shadow: 0 4px 12px rgba(0,0,0,0.1);
        margin-top: 25px;
        margin-bottom: 20px;
    }

    h2 {
        border-left: 5px solid #007BFF;
        padding-left: 10px;
        margin-bottom: 20px;
    }

    input[type="text"], input[type="file"] {
        width: 100%;
        padding: 10px;
        font-size: 17px;
        border: 1px solid #aaa;
        border-radius: 6px;
        margin-top: 8px;
    }

    button {
        margin-top: 15px;
        padding: 12px;
        background: #007BFF;
        color: #fff;
        font-size: 17px;
        border: none;
        border-radius: 6px;
        cursor: pointer;
        width: 180px;
        display: block;
        margin-left: auto;
        margin-right: auto;
    }

    button:hover {
        background: #0056b3;
    }

    .download-btn {
        background: #28a745 !important;
    }

    .result {
        margin-top: 20px;
        padding: 14px;
        background: #fff7f7;
        border-radius: 6px;
        text-align: center;
        font-size: 22px;
        color: #d9534f;
        font-weight: bold;
    }
</style>
</head>

<body>

<!-- =============== 顶部示意图 + 参数说明 =============== -->
<div class="header-box">

    <div class="pic-container">
        <img src="https://iili.io/fJ3Mdsp.png">
    </div>

    <table class="param-table">
        <tr><td><i>f</i><sub>o</sub> / MPa :</td><td>Outer steel yield strength</td></tr>
        <tr><td><i>f</i><sub>i</sub> / MPa :</td><td>Inner steel yield strength</td></tr>
        <tr><td><i>f</i><sub>c</sub> / MPa :</td><td>Concrete axial compressive strength (N/mm²)</td></tr>
        <tr><td><i>D</i><sub>o</sub> / mm :</td><td>Outer diameter</td></tr>
        <tr><td><i>D</i><sub>i</sub> / mm :</td><td>Inner diameter</td></tr>
        <tr><td><i>t</i><sub>o</sub> / mm :</td><td>Outer tube thickness</td></tr>
        <tr><td><i>t</i><sub>i</sub> / mm :</td><td>Inner tube thickness</td></tr>
        <tr><td><i>L</i> / mm :</td><td>Column length</td></tr>
    </table>
</div>

<!-- =============== Single Calculation =============== -->
<div class="section-card">
    <h2>Single Calculation</h2>

    输入参数（空格分隔）：fo fi fc Do to Di ti L  
    <input type="text" id="singleInput" placeholder="例如：345 280 40 300 6 160 4 3000">

    <button onclick="calcSingle()">Calculate</button>

    <div id="singleResult" class="result"></div>
</div>

<!-- =============== Batch Calculation =============== -->
<div class="section-card">
    <h2>Batch Calculation</h2>

    <input type="file" id="fileInput" accept=".txt,.csv,.xls,.xlsx">

    <button onclick="calcBatch()">Batch Calculate</button>
    <button id="downloadBtn" class="download-btn" style="display:none;" onclick="downloadExcel()">Download Excel</button>

    <div id="batchResult" class="result"></div>
</div>

<!-- =============== JS Core =============== -->
<script>
/************** 核心计算模块（双层钢管 + 稳定性 φ） **************/
function compute(fo,fi,fc,Do,to,Di,ti,L){

    const Aso = Math.PI/4*(Do*Do - (Do-2*to)**2);
    const As_inner = Math.PI/4*(Di*Di - (Di-2*ti)**2);
    const Ac = Math.PI/4*((Do-2*to)**2 - Di*Di);
    const Ace = Math.PI/4*(Do*Do - Di*Di);

    const alpha = Aso/Ac;
    const alpha_n = Aso/Ace;
    const xi0 = Aso*fo/(Ace*fc);

    const C1 = alpha/(1+alpha);
    const C2 = (1+alpha_n)/(1+alpha);

    /* ★ 修正后 χ 公式（正确圆套圆定义）★ */
    const chi = Di/(Do - 2*to);

    const fosc = C1*chi*chi*fo + C2*(1.14+1.02*xi0)*fc;

    const Nosc = fosc*(Aso+Ac);
    const Ni = fi*As_inner;

    const Nu_short = Nosc + Ni;

    const f_oscy = fosc;

    const d = (13000 + 4657*Math.log(235/fo))
              * Math.pow(25/(fc+5),0.3)
              * Math.pow(alpha_n/0.1,0.05);

    const lambda_p = 1743/Math.sqrt(fo);
    const lambda_0 = Math.PI*Math.sqrt(420*xi0+550)/f_oscy;

    const lambda = 4*L / Math.sqrt(Do*Do + (Di-2*ti)**2);

    let phi = 1;

    if(lambda <= lambda_0){
        phi = 1;
    }else if(lambda <= lambda_p){
        const e = -d / Math.pow(lambda_p+35,3);
        const a = (1+(35+2*lambda_p-lambda_0)*e)/((lambda_p-lambda_0)**2);
        const b = e - 2*a*lambda_p;
        const c = 1 - a*(lambda_0**2) - b*lambda_0;
        phi = a*lambda*lambda + b*lambda + c;
    }else{
        phi = d * (-0.23*lambda*lambda + 1) / ((lambda+35)**2);
    }

    phi = Math.min(phi,1);

    return {
        Nu_short: Nu_short/1000,
        phi: phi,
        Nu_final: phi*Nu_short/1000
    };
}

/************** 单根计算 **************/
function calcSingle(){
    let vals = document.getElementById("singleInput").value.trim();
    vals = vals.replace(/,/g," ").split(/\s+/).map(Number);

    if(vals.length !== 8 || vals.some(v=>isNaN(v))){
        document.getElementById("singleResult").innerHTML = "❌ 输入错误";
        return;
    }

    const r = compute(...vals);
    document.getElementById("singleResult").innerHTML =
        `短柱 N₀ = ${r.Nu_short.toFixed(2)} kN<br>
         稳定系数 φ = ${r.phi.toFixed(3)}<br>
         承载力 N = ${r.Nu_final.toFixed(2)} kN`;
}

/************** 批量计算 **************/
let batchWorkbook = null;

function calcBatch(){
    const file = document.getElementById("fileInput").files[0];
    if(!file){
        document.getElementById("batchResult").innerHTML="❌ 请选择文件";
        return;
    }

    const reader = new FileReader();
    reader.onload = function(e){
        let rows;
        if(file.name.endsWith(".txt") || file.name.endsWith(".csv")){
            rows = e.target.result.trim().split("\n")
                   .map(r=>r.trim().split(/[, ]+/).map(Number));
        }else{
            const wb = XLSX.read(new Uint8Array(e.target.result),{type:"array"});
            rows = XLSX.utils.sheet_to_json(wb.Sheets[wb.SheetNames[0]],{header:1});
        }

        const out = [["fo","fi","fc","Do","to","Di","ti","L","N0(kN)","phi","N(kN)"]];

        rows.forEach(r=>{
            if(r.length>=8){
                const x = compute(...r.slice(0,8));
                out.push([...r.slice(0,8),
                          x.Nu_short.toFixed(2),
                          x.phi.toFixed(3),
                          x.Nu_final.toFixed(2)]);
            }
        });

        const newWb = XLSX.utils.book_new();
        XLSX.utils.book_append_sheet(newWb, XLSX.utils.aoa_to_sheet(out), "Results");
        batchWorkbook = newWb;

        document.getElementById("downloadBtn").style.display="block";
        document.getElementById("batchResult").innerHTML="✔ 批量计算完成";
    };

    if(file.name.endsWith(".txt") || file.name.endsWith(".csv")){
        reader.readAsText(file);
    }else reader.readAsArrayBuffer(file);
}

function downloadExcel(){
    const wbout = XLSX.write(batchWorkbook,{bookType:"xlsx",type:"binary"});
    const buf = new ArrayBuffer(wbout.length);
    const view = new Uint8Array(buf);
    for(let i=0;i<wbout.length;i++) view[i]=wbout.charCodeAt(i)&0xFF;

    const blob = new Blob([buf],{type:"application/octet-stream"});
    const a = document.createElement("a");
    a.href = URL.createObjectURL(blob);
    a.download = "DoubleCFST_results.xlsx";
    a.click();
}
</script>

</body>
</html>
