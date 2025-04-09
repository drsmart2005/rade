# rade
calculator
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0"/>
  <title>99강화 나무몽둥이 레이드 점수 계산기 v3.2</title>
  <style>
    body { font-family: 'Segoe UI', sans-serif; margin: 20px; background-color: #f8f9fa; }
    h1 { color: #2c3e50; }
    label { display: block; margin-top: 10px; font-weight: bold; }
    input[type="number"], input[type="text"] { width: 100%; padding: 8px; margin-top: 5px; }
    .character { border: 1px solid #ddd; padding: 15px; margin-bottom: 20px; border-radius: 5px; background: #fff; }
    .results { margin-top: 30px; padding: 20px; background: #e9ecef; border-radius: 5px; }
    canvas { margin-top: 20px; max-width: 100%; }
    button { margin-top: 20px; padding: 10px 20px; font-size: 16px; }
  </style>
</head>
<body>
  <h1>99강화 나무몽둥이 레이드 점수 계산기 v3.2</h1>

  <label><input type="checkbox" id="simpleToggle" onchange="renderForm()"> 간편 입력 모드</label>
  <div id="form"></div>

  <button onclick="calculate()">점수 계산</button>

  <div class="results" id="results"></div>
  <canvas id="scoreChart"></canvas>

  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
  <script>
    const unitMap = {
      '무량대수': 1e68, '불가사의': 1e64, '나유타': 1e60, '아승기': 1e56,
      '항하사': 1e52, '극': 1e48, '재': 1e44, '정': 1e40, '간': 1e36,
      '구': 1e32, '양': 1e28, '자': 1e24, '해': 1e20, '경': 1e16,
      '조': 1e12, '억': 1e8
    };

    function parseUnit(str) {
      const num = parseFloat(str.replace(/[^0-9.]/g, '')) || 0;
      const unit = Object.keys(unitMap).find(u => str.includes(u));
      return unit ? num * unitMap[unit] : num;
    }

    function toUnitString(num) {
      const entries = Object.entries(unitMap).sort((a, b) => b[1] - a[1]);
      for (const [unit, value] of entries) {
        if (num >= value) return (num / value).toFixed(4) + unit;
      }
      return num.toString();
    }

    function getScore(atk, pen, critRate, critDmg, skill, cdr, aspd) {
      const expectedCrit = (critRate / 100) * (critDmg / 100);
      const base = atk * (1 + pen / 100) * (1 + skill / 100);
      const mult = (1 + expectedCrit) * (1 + cdr / 100) * (1 + aspd / 100);
      return base * mult;
    }

    function getGrade(score) {
      if (score >= 1e15) return 'S';
      if (score >= 1e13) return 'A';
      if (score >= 1e11) return 'B';
      if (score >= 1e9) return 'C';
      return 'D';
    }

    function renderForm() {
      const simple = document.getElementById("simpleToggle").checked;
      const formDiv = document.getElementById("form");
      formDiv.innerHTML = "";

      if (simple) {
        formDiv.innerHTML += `
          <div class="character">
            <h3>평균 스펙 입력</h3>
            <label>공격력 (예: 5조)</label><input type="text" id="atk1">
            <label>관통력</label><input type="number" id="pen1">
            <label>치명타 확률 (%)</label><input type="number" id="critRate1">
            <label>치명타 데미지 (%)</label><input type="number" id="critDmg1">
            <label>스킬 데미지 증가 (%)</label><input type="number" id="skill1">
            <label>쿨타임 감소 (%)</label><input type="number" id="cdr1">
            <label>공격 속도 (%)</label><input type="number" id="aspd1">
          </div>
        `;
      } else {
        for (let i = 1; i <= 3; i++) {
          formDiv.innerHTML += `
            <div class="character">
              <h3>캐릭터 ${i}</h3>
              <label>공격력 (예: 5조)</label><input type="text" id="atk${i}">
              <label>관통력</label><input type="number" id="pen${i}">
              <label>치명타 확률 (%)</label><input type="number" id="critRate${i}">
              <label>치명타 데미지 (%)</label><input type="number" id="critDmg${i}">
              <label>스킬 데미지 증가 (%)</label><input type="number" id="skill${i}">
              <label>쿨타임 감소 (%)</label><input type="number" id="cdr${i}">
              <label>공격 속도 (%)</label><input type="number" id="aspd${i}">
            </div>
          `;
        }
      }
    }

    function calculate() {
      const simple = document.getElementById("simpleToggle").checked;
      const resultsDiv = document.getElementById("results");
      const labels = [];
      const data = [];
      let total = 0;

      resultsDiv.innerHTML = "";

      const len = simple ? 1 : 3;

      for (let i = 1; i <= len; i++) {
        const atk = parseUnit(document.getElementById(`atk${i}`).value);
        const pen = parseFloat(document.getElementById(`pen${i}`).value || "0");
        const critRate = parseFloat(document.getElementById(`critRate${i}`).value || "0");
        const critDmg = parseFloat(document.getElementById(`critDmg${i}`).value || "0");
        const skill = parseFloat(document.getElementById(`skill${i}`).value || "0");
        const cdr = parseFloat(document.getElementById(`cdr${i}`).value || "0");
        const aspd = parseFloat(document.getElementById(`aspd${i}`).value || "0");

        const score = getScore(atk, pen, critRate, critDmg, skill, cdr, aspd);
        total += score;

        if (!simple) {
          resultsDiv.innerHTML += `<p>캐릭터 ${i} 점수: ${score.toExponential(2)} (${getGrade(score)}등급)</p>`;
          labels.push(`캐릭터 ${i}`);
          data.push(score);
        }
      }

      const avg = total / len;
      const unitScore = toUnitString(avg);

      resultsDiv.innerHTML += `<hr><h3>예상 최종 레이드 점수: ${avg.toExponential(2)} (${unitScore})</h3>`;
      resultsDiv.innerHTML += `<p>스펙 등급: <strong>${getGrade(avg)}</strong></p>`;

      if (!simple) {
        new Chart(document.getElementById("scoreChart"), {
          type: "bar",
          data: {
            labels: labels,
            datasets: [{
              label: "캐릭터별 점수",
              data: data,
              backgroundColor: ["#4dabf7", "#63e6be", "#ffd43b"]
            }]
          },
          options: {
            responsive: true,
            scales: {
              y: {
                beginAtZero: true,
                ticks: {
                  callback: function (value) {
                    return value.toExponential(2);
                  }
                }
              }
            }
          }
        });
      } else {
        document.getElementById("scoreChart").remove();
      }
    }

    // 초기 렌더링
    renderForm();
  </script>
</body>
</html>
