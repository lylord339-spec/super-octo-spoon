LyLord(*´∀`)~♥
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>庞诺视错觉实验 - beta版</title>
    <script src="https://cdn.sheetjs.com"></script>
    <style>
        :root { --stage-w: 600px; --stage-h: 450px; }
        body { font-family: sans-serif; background: #f0f2f5; display: flex; flex-direction: column; align-items: center; padding: 20px; }
        #stage { position: relative; width: var(--stage-w); height: var(--stage-h); background: white; border: 2px solid #334155; overflow: hidden; border-radius: 4px; display: flex; justify-content: center; }
        #rails-container { position: absolute; top: 0; left: 50%; width: 0; height: 100%; }
        .rail { position: absolute; width: 2px; height: 1000px; background: #94a3b8; top: -100px; left: -1px; transform-origin: top center; z-index: 1; }
        .line { position: absolute; height: 12px; border-radius: 6px; z-index: 10; box-shadow: 0 2px 4px rgba(0,0,0,0.3); }
        #line-ref { top: 80px; background: #ef4444; width: 100px; left: calc(50% - 50px); }
        #line-test { bottom: 80px; background: #2563eb; border: 2px solid #ffd700; }
        .main-ui { display: flex; gap: 20px; width: 900px; margin-top: 20px; }
        .panel { background: white; padding: 20px; border-radius: 8px; flex: 1; box-shadow: 0 2px 10px rgba(0,0,0,0.1); }
        .item { margin-bottom: 15px; }
        input[type="range"] { width: 100%; cursor: pointer; }
        table { width: 100%; border-collapse: collapse; font-size: 12px; margin-top: 10px; }
        th, td { border: 1px solid #ddd; padding: 8px; text-align: center; }
        th { background: #f8fafc; }
        .btn { width: 100%; padding: 12px; margin: 5px 0; cursor: pointer; border: none; border-radius: 4px; font-weight: bold; color: white; transition: 0.2s; }
        .btn-save { background: #2563eb; }
        .btn-save:hover { background: #1d4ed8; }
        .btn-xlsx { background: #10b981; }
    </style>
</head>
<body> 

    <h2>庞诺视错觉 (Ponzo) 严谨实验系统</h2>

    <div id="stage">
        <div id="rails-container"></div>
        <div id="line-ref" class="line"></div>
        <div id="line-test" class="line"></div>
    </div>

    <div class="main-ui">
        <div class="panel">
            <div class="item">
                <label><b>1. 射线设置</b></label><br>
                数量: <input type="number" id="r-num" value="10" min="2" max="20" style="width:60px">
                张角: <b id="a-txt">20</b>°
                <input type="range" id="a-slider" min="2" max="60" value="20">
            </div>
            <div class="item">
                <label><b>2. 等差步进设置</b></label><br>
                起始: <input type="number" id="s-start" value="10" style="width:40px">
                步长: <input type="number" id="s-step" value="5" style="width:40px">
                结束: <input type="number" id="s-end" value="50" style="width:40px">
            </div>
            <div class="item">
                <label><b>3. 调节蓝线长度: </b> <b id="l-txt">30</b> px</label>
                <input type="range" id="l-slider" min="30" max="250" value="30">
            </div>
            <button class="btn btn-save" onclick="doRecord()">记录并自动步进</button>
            <button class="btn btn-xlsx" onclick="doExport()">导出结果 (XLSX)</button>
        </div>

        <div class="panel" style="max-height: 400px; overflow-y: auto;">
            <b>数据实时表格</b>
            <table id="data-table">
                <thead>
                    <tr><th>序次</th><th>张角</th><th>物理长度</th><th>强度</th></tr>
                </thead>
                <tbody id="log"></tbody>
            </table>
        </div>
    </div>

<script>
    let results = [];
    const REF_W = 100;

    function render() {
        const num = parseInt(document.getElementById('r-num').value) || 2;
        const angle = parseFloat(document.getElementById('a-slider').value);
        const len = parseFloat(document.getElementById('l-slider').value);
        
        document.getElementById('a-txt').innerText = angle;
        document.getElementById('l-txt').innerText = len;

        const container = document.getElementById('rails-container');
        container.innerHTML = '';
        
        // 渲染射线
        for(let i=0; i<num; i++){
            const r = document.createElement('div');
            r.className = 'rail';
            const currentAngle = (num === 1) ? 0 : (angle / 2) - (i * (angle / (num - 1)));
            r.style.transform = `rotate(${currentAngle}deg)`;
            container.appendChild(r);
        }

        // 渲染蓝线并保持居中
        const testLine = document.getElementById('line-test');
        testLine.style.width = len + 'px';
        testLine.style.left = `calc(50% - ${len/2}px)`;
    }

    function doRecord() {
        const angle = document.getElementById('a-slider').value;
        const len = document.getElementById('l-slider').value;
        const dev = (((len - REF_W)/REF_W)*100).toFixed(1);

        // 核心修复：确保属性名与表格渲染时一致
        const record = {
            id: results.length + 1,
            angle: angle + "°",
            length: len + "px",
            strength: dev + "%"
        };

        results.push(record);

        // 实时更新表格
        const logBody = document.getElementById('log');
        const row = `<tr>
            <td>${record.id}</td>
            <td>${record.angle}</td>
            <td>${record.length}</td>
            <td style="color:#2563eb; font-weight:bold">${record.strength}</td>
        </tr>`;
        logBody.insertAdjacentHTML('afterbegin', row); // 新记录在最上方

        // 自动步进逻辑
        const step = parseFloat(document.getElementById('s-step').value);
        const end = parseFloat(document.getElementById('s-end').value);
        let next = parseFloat(angle) + step;
        if(next <= end) {
            document.getElementById('a-slider').value = next;
            render();
        } else {
            alert("实验序列已完成！数据已记录在右侧表格。");
        }
    }

    function doExport() {
        if(results.length === 0) return alert("暂无数据可导出");
        // 转换数据格式以适配 Excel 表头
        const exportData = results.map(r => ({
            "序号": r.id,
            "张角": r.angle,
            "物理长度(px)": r.length,
            "错觉强度": r.strength
        }));
        const ws = XLSX.utils.json_to_sheet(exportData);
        const wb = XLSX.utils.book_new();
        XLSX.utils.book_append_sheet(wb, ws, "庞诺实验结果");
        XLSX.writeFile(wb, `Ponzo_Data_${new Date().getTime()}.xlsx`);
    }

    // 绑定事件
    document.getElementById('r-num').oninput = render;
    document.getElementById('a-slider').oninput = render;
    document.getElementById('l-slider').oninput = render;
    window.onload = render;
</script>

</body>
</html>
