# Flora
medical instrument
<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
    <meta charset="UTF-8">
    <title>Flora Elite Health | المحلل التشريحي النخبوي</title>
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        :root {
            --flora-purple: #8e44ad; --flora-green: #2ecc71; --flora-blue: #3498db;
            --soft-bg: #f4f7f6; --text-dark: #2c3e50;
        }
        body { background: var(--soft-bg); font-family: 'Segoe UI', Tahoma, sans-serif; padding: 20px; color: var(--text-dark); line-height: 1.6; }
        .app-container { max-width: 600px; margin: auto; }
        
        .glass-card {
            background: white; border-radius: 30px; padding: 30px; margin-bottom: 25px;
            box-shadow: 0 15px 35px rgba(0,0,0,0.05); border: 1px solid rgba(255,255,255,0.8);
            transition: transform 0.3s ease;
        }
        .section-icon { width: 50px; height: 50px; border-radius: 15px; display: flex; align-items: center; justify-content: center; margin-bottom: 15px; color: white; font-weight: bold; font-size: 1.2em; }
        
        h2 { font-size: 1.4em; margin-bottom: 15px; color: var(--text-dark); }
        .input-field { width: 100%; padding: 15px; margin: 10px 0; border: 2px solid #edf2f7; border-radius: 15px; box-sizing: border-box; font-size: 1em; outline: none; transition: 0.3s; }
        .input-field:focus { border-color: var(--flora-purple); }
        
        .btn-generate { width: 100%; padding: 18px; background: linear-gradient(135deg, var(--flora-purple), var(--flora-blue)); color: white; border: none; border-radius: 50px; font-weight: bold; font-size: 1.1em; cursor: pointer; box-shadow: 0 10px 20px rgba(142, 68, 173, 0.2); transition: 0.3s; }
        .btn-generate:hover { transform: translateY(-3px); box-shadow: 0 15px 30px rgba(142, 68, 173, 0.3); }

        #healthPlan { display: none; margin-top: 30px; padding: 25px; border-radius: 25px; background: #fff; border-right: 10px solid var(--flora-green); }
        .plan-item { margin-bottom: 15px; padding-bottom: 10px; border-bottom: 1px dashed #eee; }
        .plan-item b { color: var(--flora-purple); }
        
        /* تصميم المؤشر البشري التفاعلي */
        .human-indicator-container { text-align: center; margin: 20px 0; padding: 20px; background: #f9f9f9; border-radius: 20px; }
        #humanSvg { width: 120px; height: auto; transition: 0.5s ease; filter: drop-shadow(0 5px 15px rgba(0,0,0,0.1)); }
        
        .charts-container { display: grid; grid-template-columns: 1fr 1fr; gap: 10px; margin-top: 20px; }
        @media (max-width: 600px) { .charts-container { grid-template-columns: 1fr; } }
    </style>
</head>
<body>

<div class="app-container">
    <h1 style="text-align:center; font-weight: 300; letter-spacing: 2px;">محلل <span style="color:var(--flora-purple); font-weight: 800;">فلورا</span> النخبوي</h1>

    <div class="glass-card">
        <div class="section-icon" style="background: var(--flora-blue);">👤</div>
        <h2>المعلومات الأساسية والنشاط</h2>
        <input type="number" id="age" class="input-field" placeholder="العمر">
        <input type="number" id="weight" class="input-field" placeholder="الوزن (كجم)">
        <input type="number" id="height" class="input-field" placeholder="الطول (سم)">
        
        <label style="font-size: 0.9em; color: #7f8c8d;">مستوى النشاط البدني:</label>
        <select id="activity" class="input-field">
            <option value="1.2">خمول (مكتبي)</option>
            <option value="1.55">نشط (تمارين 3-5 أيام)</option>
            <option value="1.9">رياضي (أداء محترف)</option>
        </select>

        <label style="font-size: 0.9em; color: #7f8c8d;">ساعات النوم الحالية:</label>
        <input type="number" id="sleepHours" class="input-field" placeholder="كم ساعة تنام حالياً؟">
    </div>

    <div class="glass-card">
        <div class="section-icon" style="background: var(--flora-purple);">📏</div>
        <h2>قياسات الجسم (WHR)</h2>
        <input type="number" id="waist" class="input-field" placeholder="محيط الخصر (سم)">
        <input type="number" id="hip" class="input-field" placeholder="محيط الورك (سم)">
    </div>

    <button class="btn-generate" onclick="generateElitePlan()">إرسال التحليل واستلام الخطة</button>

    <div id="healthPlan">
        <h2 style="color: var(--flora-green);">✨ تقرير فلورا الصحي العالمي:</h2>
        
        <div class="human-indicator-container">
            <p style="font-size: 0.8em; color: #95a5a6; margin-bottom: 10px;">الحالة التشريحية الذكية</p>
            <svg id="humanSvg" viewBox="0 0 100 200" xmlns="http://www.w3.org/2000/svg">
                <path id="humanPath" d="M50,15 A15,15 0 1,1 50,45 A15,15 0 1,1 50,15 M35,50 L65,50 L70,110 L60,110 L60,185 L40,185 L40,110 L30,110 Z" fill="#ccc" />
            </svg>
            <div id="statusLabel" style="font-weight: bold; margin-top: 10px;">جاري التحليل...</div>
        </div>

        <div id="planContent"></div>
        
        <div class="charts-container">
            <canvas id="statusChart"></canvas>
            <canvas id="radarChart"></canvas>
        </div>
    </div>
</div>

<script>
let myChart, myRadar;
function generateElitePlan() {
    const age = parseFloat(document.getElementById('age').value);
    const w = parseFloat(document.getElementById('weight').value);
    const h = parseFloat(document.getElementById('height').value);
    const waist = parseFloat(document.getElementById('waist').value);
    const hip = parseFloat(document.getElementById('hip').value);
    const activity = parseFloat(document.getElementById('activity').value);
    const sleep = parseFloat(document.getElementById('sleepHours').value);

    if(!age || !w || !h || !waist || !hip || !sleep) {
        alert("يرجى ملء كافة البيانات لضمان دقة التحليل النخبوي");
        return;
    }

    const bmi = (w / ((h/100)**2)).toFixed(1);
    const whr = (waist / hip).toFixed(2);
    const bioAge = whr > 0.9 ? parseInt(age) + 3 : parseInt(age) - 2;
    const calories = Math.round(((10 * w) + (6.25 * h) - (5 * age) + 5) * activity);

    const planDiv = document.getElementById('healthPlan');
    const content = document.getElementById('planContent');
    const humanPath = document.getElementById('humanPath');
    const statusLabel = document.getElementById('statusLabel');
    planDiv.style.display = 'block';

    // تحديث المؤشر البشري التفاعلي
    if (bmi < 18.5) { humanPath.setAttribute('fill', '#3498db'); statusLabel.innerText = "تحت الوزن الطبيعي"; }
    else if (bmi >= 18.5 && bmi <= 24.9) { humanPath.setAttribute('fill', '#2ecc71'); statusLabel.innerText = "تكوين جسدي مثالي"; }
    else if (bmi >= 25 && bmi <= 29.9) { humanPath.setAttribute('fill', '#f1c40f'); statusLabel.innerText = "مؤشرات وزن زائد"; }
    else { humanPath.setAttribute('fill', '#e74c3c'); statusLabel.innerText = "سمنة - تتطلب إجراء"; }

    let sleepAdvice = sleep < 8 ? "جسدك يحتاج إلى <b>8 ساعات</b> من النوم العميق لضمان الاستشفاء الخلوي." : "نظام نومك ممتاز، استمر على هذا الإيقاع الحيوي.";
    let foodToAvoid = bmi > 25 ? "السكريات المكررة، الزيوت المدرجة، والوجبات السريعة." : "تجنب الأطعمة الفقيرة بالبروتين والمشروبات الغازية.";
    let foodToIncrease = bmi > 25 ? "الخضروات الورقية، البروتين الصافي، والدهون الصحية (أوميغا 3)." : "زيادة الكربوهيدرات المعقدة (الشوفان، الكينوا) والبروتينات لبناء الكتلة.";

    let advice = `
        <div class='plan-item'><b>العمر الحيوي:</b> تقديرياً <b>${bioAge} سنة</b>.</div>
        <div class='plan-item'><b>هندسة النوم:</b> ${sleepAdvice}</div>
        <div class='plan-item'><b>الاحتياج الحراري:</b> ${calories} سعرة.</div>
        <div class='plan-item' style='color:#e74c3c'><b>تجنب أكل:</b> ${foodToAvoid}</div>
        <div class='plan-item' style='color:#27ae60'><b>أكثر من أكل:</b> ${foodToIncrease}</div>
        <div class='plan-item'><b>الترطيب النخبوي:</b> اشرب ${ (w * 0.035).toFixed(1) } لتر ماء يومياً.</div>
    `;

    content.innerHTML = advice;

    // الرسوم البيانية
    if(myChart) myChart.destroy();
    const ctx = document.getElementById('statusChart').getContext('2d');
    myChart = new Chart(ctx, {
        type: 'bar',
        data: {
            labels: ['BMI', 'WHR (x10)'],
            datasets: [{
                label: 'المؤشرات الحيوية',
                data: [bmi, whr * 10],
                backgroundColor: ['#8e44ad', '#2ecc71'],
                borderRadius: 8
            }]
        },
        options: { responsive: true, plugins: { legend: { display: false } } }
    });

    if(myRadar) myRadar.destroy();
    const ctxRadar = document.getElementById('radarChart').getContext('2d');
    myRadar = new Chart(ctxRadar, {
        type: 'radar',
        data: {
            labels: ['الأيض', 'الكتلة', 'التوزيع', 'النوم', 'الترطيب'],
            datasets: [{
                label: 'الأداء العالمي',
                data: [80, bmi < 25 ? 90 : 60, whr < 0.9 ? 95 : 50, (sleep/8)*100, 85],
                borderColor: '#3498db',
                backgroundColor: 'rgba(52, 152, 219, 0.2)'
            }]
        },
        options: { scales: { r: { beginAtZero: true, max: 100 } }, plugins: { legend: { display: false } } }
    });

    planDiv.scrollIntoView({ behavior: 'smooth' });
}
</script>
</body>
</html>
