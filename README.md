# 



<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <title>💰 더치페이 계산기</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      padding: 20px;
      max-width: 500px;
      margin: auto;
      background-color: #fff;
      position: relative;
    }
    h2 {
      display: flex;
      align-items: center;
      gap: 10px;
    }
    label {
      display: block;
      margin-top: 15px;
    }
    input[type="number"], input[type="text"] {
      width: 100%;
      padding: 8px;
      margin-top: 4px;
    }
    .unit-buttons {
      display: flex;
      gap: 10px;
      margin-top: 5px;
    }
    .unit-buttons button {
      flex: 1;
      padding: 8px;
      font-weight: bold;
      background-color: #eee;
      border: 1px solid #ccc;
      cursor: pointer;
    }
    .unit-buttons button.active {
      background-color: #007BFF;
      color: white;
    }
    button.calculate-btn {
      margin-top: 15px;
      padding: 10px;
      width: 100%;
      background-color: #007BFF;
      color: white;
      font-weight: bold;
      border: none;
      cursor: pointer;
      border-radius: 10px;
    }
    button.reset-btn {
      position: fixed;
      bottom: 10px;
      right: 10px;
      width: 20%;
      font-size: 12px;
      padding: 5px;
      background-color: #ddd;
      border: none;
      cursor: pointer;
      border-radius: 5px;
      z-index: 100;
    }
    .result {
      margin-top: 20px;
      background: #fff9c4;
      padding: 15px;
      border-radius: 10px;
      box-shadow: 2px 2px 5px rgba(0,0,0,0.1);
    }
  </style>
</head>
<body>
  <h2>
    💰 더치페이 계산기
  </h2>

  <label>총 금액 (원):
    <input type="text" id="totalAmountFormatted" oninput="formatNumberInput(this)" />
  </label>
  <label>인원수(명):
    <input type="number" id="numPeople" />
  </label>
  <label>기준 단위:
    <div class="unit-buttons">
      <button type="button" id="unit100" onclick="setUnit(100, this)">100원</button>
      <button type="button" id="unit10" onclick="setUnit(10, this)">10원</button>
    </div>
  </label>

  <button class="calculate-btn" onclick="calculateDutchpay()">계산하기</button>

  <div class="result" id="result"></div>

  <button class="reset-btn" onclick="resetForm()">초기화</button>

 <script>
  function formatNumberInput(input) {
    const raw = input.value.replace(/[^0-9]/g, '');
    input.value = raw.replace(/\B(?=(\d{3})+(?!\d))/g, ",");
  }

  function getNumericValue(formatted) {
    return parseFloat(formatted.replace(/,/g, ''));
  }

  function setUnit(unit, btn) {
    document.getElementById('roundingPlace')?.remove();
    const input = document.createElement('input');
    input.type = 'hidden';
    input.id = 'roundingPlace';
    input.value = Math.log10(unit) * -1;
    document.body.appendChild(input);

    document.querySelectorAll('.unit-buttons button').forEach(b => b.classList.remove('active'));
    btn.classList.add('active');
  }

  function calculateDutchpay() {
    const total = getNumericValue(document.getElementById('totalAmountFormatted').value);
    const people = parseInt(document.getElementById('numPeople').value);
    const placeInput = document.getElementById('roundingPlace');
    const place = placeInput ? parseInt(placeInput.value) : -2;

    if (isNaN(total) || isNaN(people) || people <= 0) {
      document.getElementById('result').innerText = '유효한 값을 입력해주세요.';
      return;
    }

    const unit = Math.pow(10, -place);
    const base = Math.floor(total / people / unit) * unit;
    const payments = new Array(people).fill(base);

    let remaining = total - base * people;
    let index = 0;
    while (remaining >= unit) {
      payments[index] += unit;
      remaining -= unit;
      index = (index + 1) % people;
    }

    payments.sort((a, b) => b - a); // 많이 낸 사람 위로

    let resultHTML = '<strong>개인별 금액:</strong><br>';
    payments.forEach((amt, idx) => {
      resultHTML += `${idx + 1}: ${amt.toLocaleString()}원<br>`;
    });

    const totalCheck = payments.reduce((a, b) => a + b, 0);
    resultHTML += `<br><strong>합계:</strong> ${totalCheck.toLocaleString()}원<br>`;
    resultHTML += `<strong>금액 ${totalCheck === total ? '맞음' : '오차 있음'}</strong>`;

    document.getElementById('result').innerHTML = resultHTML;
  }

  function resetForm() {
    document.getElementById('totalAmountFormatted').value = '';
    document.getElementById('numPeople').value = '';
    document.getElementById('result').innerHTML = '';
    document.querySelectorAll('.unit-buttons button').forEach(b => b.classList.remove('active'));
  }
</script>

</body>
</html>
