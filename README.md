[index.html](https://github.com/user-attachments/files/26704944/index.html)
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <title>💖 情侣温柔契约 · 不吵架积分表</title>
    <style>
        * {
            box-sizing: border-box;
            font-family: "Microsoft YaHei", sans-serif;
        }
        body {
            background: #fdf6f8;
            margin: 0;
            padding: 20px;
            max-width: 1000px;
            margin: 0 auto;
        }
        .title {
            text-align: center;
            color: #e15c7c;
            font-size: 26px;
            margin-bottom: 10px;
        }
        .rule-box {
            background: white;
            border-radius: 12px;
            padding: 20px;
            margin-bottom: 20px;
            box-shadow: 0 2px 10px #00000010;
        }
        .rule-box h3 {
            color: #c84e66;
            margin-top: 0;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            background: white;
            border-radius: 12px;
            overflow: hidden;
            box-shadow: 0 2px 10px #00000010;
        }
        th, td {
            padding: 12px;
            text-align: center;
            border-bottom: 1px solid #f0f0f0;
        }
        th {
            background: #fce4ec;
            color: #d33a5e;
            font-size: 14px;
        }
        input {
            width: 100%;
            padding: 8px;
            border: 1px solid #eee;
            border-radius: 6px;
            outline: none;
            font-size: 14px;
        }
        .total-box {
            margin-top: 20px;
            background: white;
            padding: 20px;
            border-radius: 12px;
            box-shadow: 0 2px 10px #00000010;
            font-size: 18px;
            font-weight: bold;
            color: #c84e66;
            line-height: 1.8;
        }
        .btn {
            background: #e15c7c;
            color: white;
            border: none;
            padding: 10px 20px;
            border-radius: 8px;
            cursor: pointer;
            margin: 10px 5px 10px 0;
            font-size: 14px;
        }
        .btn:hover {
            background: #c84e66;
        }
        .del-btn {
            background: #ff4d4d;
            color: white;
            border: none;
            padding: 6px 12px;
            font-size: 12px;
            border-radius: 4px;
            cursor: pointer;
        }
        .reset-btn {
            background: #999;
        }
        .dayScore {
            color: #d33a5e;
            font-weight: bold;
        }
    </style>
</head>
<body>

    <div class="title">💖 情侣温柔契约 · 不吵架积分表</div>

    <div class="rule-box">
        <h3>📜 双方约定</h3>
        <p>1. 坚守忠诚底线，双方均不出轨，否则契约失效。</p>
        <p>2. 禁止无故发脾气、闹别扭、主动吵架。</p>
        <p>3. 违规一次扣 1 分，1 分 = 10 元。</p>
        <p>4. 主动道歉、主动哄人可 +1 分。</p>
        <p>5. 每月结算一次，负分方向对方支付对应金额。</p>
    </div>

    <button class="btn" onclick="addRow()">➕ 添加一行记录</button>
    <button class="btn reset-btn" onclick="resetMonth()">🔄 每月清空（重新开始）</button>

    <table id="recordTable">
        <tr>
            <th>日期</th>
            <th>类型（男方/女方）</th>
            <th>事由</th>
            <th>扣分</th>
            <th>加分</th>
            <th>净分</th>
            <th>操作</th>
        </tr>
        <tbody id="list"></tbody>
    </table>

    <div class="total-box">
        👨 男方总积分：<span id="maleScore">0</span> 分 ｜ 应结算：<span id="maleMoney">0</span> 元<br>
        👩 女方总积分：<span id="femaleScore">0</span> 分 ｜ 应结算：<span id="femaleMoney">0</span> 元<br>
        <small>（负数 = 需要付给对方的钱）</small>
    </div>

<script type="module">
import { initializeApp } from "https://www.gstatic.com/firebasejs/11.4.0/firebase-app.js";
import { getDatabase, ref, onValue, push, set, remove } from "https://www.gstatic.com/firebasejs/11.4.0/firebase-database.js";

// 初始化Firebase
const app = initializeApp({
    apiKey: "AIzaSyD7Zp2Yz0BdQ0G5R8E1lZJzH6YJ0HhR0A0",
    authDomain: "love-record-123.firebaseapp.com",
    databaseURL: "https://love-record-123-default-rtdb.firebaseio.com",
    projectId: "love-record-123",
    storageBucket: "love-record-123.appspot.com",
    messagingSenderId: "7272727272",
    appId: "1:7272727272:web:0000000000"
});

const db = getDatabase(app);
const dbRef = ref(db, "loveRecords");

// 页面加载完成后，初始化默认行（如果数据库为空）
window.onload = async () => {
    const snapshot = await new Promise(resolve => onValue(dbRef, resolve, { once: true }));
    if (!snapshot.val()) {
        push(dbRef, { 
            date: "2026-04-14", 
            name: "", 
            why: "", 
            sub: 0, 
            add: 0 
        });
    }
};

// 实时监听数据变化
onValue(dbRef, snapshot => {
    const data = snapshot.val() || {};
    render(data);
    calc(data);
});

// 渲染表格
function render(data) {
    const list = document.getElementById("list");
    list.innerHTML = "";
    for (let id in data) {
        const d = data[id];
        const dayScore = (Number(d.add) || 0) - (Number(d.sub) || 0);
        list.innerHTML += `
        <tr>
            <td><input type="text" placeholder="2026-04-14" value="${d.date || ''}" 
                    oninput="update('${id}','date',this.value)"></td>
            <td><input type="text" placeholder="男方/女方" value="${d.name || ''}" 
                    oninput="update('${id}','name',this.value)"></td>
            <td><input type="text" placeholder="发脾气/闹别扭等" value="${d.why || ''}" 
                    oninput="update('${id}','why',this.value)"></td>
            <td><input type="number" min="0" value="${d.sub || 0}" 
                    oninput="update('${id}','sub',this.value)"></td>
            <td><input type="number" min="0" value="${d.add || 0}" 
                    oninput="update('${id}','add',this.value)"></td>
            <td><span class="dayScore">${dayScore}</span></td>
            <td><button class="del-btn" onclick="del('${id}')">删除</button></td>
        </tr>`;
    }
}

// 计算男女总分和金额
function calc(data) {
    let maleTotal = 0, femaleTotal = 0;
    for (let id in data) {
        const d = data[id];
        const score = (Number(d.add) || 0) - (Number(d.sub) || 0);
        if (d.name?.includes("男方")) maleTotal += score;
        if (d.name?.includes("女方")) femaleTotal += score;
    }
    document.getElementById("maleScore").innerText = maleTotal;
    document.getElementById("maleMoney").innerText = maleTotal * 10;
    document.getElementById("femaleScore").innerText = femaleTotal;
    document.getElementById("femaleMoney").innerText = femaleTotal * 10;
}

// 新增行
window.addRow = () => {
    push(dbRef, { 
        date: "2026-04-14", 
        name: "", 
        why: "", 
        sub: 0, 
        add: 0 
    });
};

// 更新数据
window.update = (id, key, value) => {
    set(ref(db, `loveRecords/${id}/${key}`), value);
};

// 删除行
window.del = (id) => {
    remove(ref(db, `loveRecords/${id}`));
};

// 每月清空
window.resetMonth = () => {
    if (confirm("确定清空本月所有记录吗？")) {
        set(dbRef, null);
        push(dbRef, { 
            date: "2026-04-14", 
            name: "", 
            why: "", 
            sub: 0, 
            add: 0 
        });
    }
};
</script>
</body>
</html>
