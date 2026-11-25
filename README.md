<html lang="th">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>เว็บแอปประกาศคะแนน</title>

  <style>
    * {
      box-sizing: border-box;
      font-family: Arial, "Noto Sans Thai", sans-serif;
    }

    body {
      margin: 0;
      min-height: 100vh;
      background: #f4f7fb;
      color: #1f2937;
    }

    header {
      background: linear-gradient(135deg, #2563eb, #7c3aed);
      color: white;
      padding: 32px 20px;
      text-align: center;
    }

    header h1 {
      margin: 0 0 8px;
      font-size: 32px;
    }

    header p {
      margin: 0;
      opacity: 0.9;
    }

    .container {
      width: min(1000px, 92%);
      margin: 28px auto;
    }

    .card {
      background: white;
      border-radius: 16px;
      padding: 22px;
      margin-bottom: 20px;
      box-shadow: 0 8px 24px rgba(15, 23, 42, 0.08);
    }

    .form-grid {
      display: grid;
      grid-template-columns: 1fr 1fr 1fr auto;
      gap: 12px;
    }

    input,
    select,
    button {
      border: 1px solid #d1d5db;
      border-radius: 10px;
      padding: 12px 14px;
      font-size: 16px;
    }

    input:focus,
    select:focus {
      outline: 2px solid #93c5fd;
      border-color: #2563eb;
    }

    button {
      border: none;
      background: #2563eb;
      color: white;
      cursor: pointer;
      font-weight: bold;
    }

    button:hover {
      background: #1d4ed8;
    }

    .danger {
      background: #dc2626;
      padding: 8px 12px;
      font-size: 14px;
    }

    .danger:hover {
      background: #b91c1c;
    }

    .toolbar {
      display: flex;
      justify-content: space-between;
      align-items: center;
      gap: 12px;
      margin-bottom: 16px;
    }

    .toolbar h2 {
      margin: 0;
    }

    .status {
      display: inline-block;
      padding: 5px 10px;
      border-radius: 999px;
      font-size: 13px;
      font-weight: bold;
    }

    .published {
      background: #dcfce7;
      color: #166534;
    }

    .hidden {
      background: #fee2e2;
      color: #991b1b;
    }

    .table-wrapper {
      overflow-x: auto;
    }

    table {
      width: 100%;
      border-collapse: collapse;
      min-width: 650px;
    }

    th,
    td {
      text-align: left;
      padding: 14px 12px;
      border-bottom: 1px solid #e5e7eb;
    }

    th {
      background: #eff6ff;
      color: #1e40af;
    }

    .score {
      font-size: 20px;
      font-weight: bold;
      color: #2563eb;
    }

    .empty {
      text-align: center;
      color: #6b7280;
      padding: 30px;
    }

    @media (max-width: 760px) {
      .form-grid {
        grid-template-columns: 1fr;
      }

      .toolbar {
        align-items: stretch;
        flex-direction: column;
      }

      header h1 {
        font-size: 25px;
      }
    }
  </style>
</head>

<body>
  <header>
    <h1>ประกาศคะแนน</h1>
    <p>วิชาภาษาไทย 6/5</p>
  </header>

  <main class="container">
    <section class="card">
      <h2>เพิ่มคะแนนนักเรียน</h2>

      <form id="scoreForm" class="form-grid">
        <input
          type="text"
          id="studentName"
          placeholder="ชื่อนักเรียน"
          required
        />

        <input
          type="number"
          id="score"
          placeholder="คะแนน"
          min="0"
          max="100"
          required
        />

        <select id="status">
          <option value="published">ประกาศแล้ว</option>
          <option value="hidden">ซ่อนไว้ก่อน</option>
        </select>

        <button type="submit">เพิ่มข้อมูล</button>
      </form>
    </section>

    <section class="card">
      <div class="toolbar">
        <h2>รายชื่อนักเรียน</h2>

        <input
          type="text"
          id="searchInput"
          placeholder="ค้นหาชื่อนักเรียน"
        />
      </div>

      <div class="table-wrapper">
        <table>
          <thead>
            <tr>
              <th>ลำดับ</th>
              <th>ชื่อนักเรียน</th>
              <th>คะแนน</th>
              <th>สถานะ</th>
              <th>จัดการ</th>
            </tr>
          </thead>

          <tbody id="scoreTable"></tbody>
        </table>

        <div id="emptyMessage" class="empty">
          ยังไม่มีข้อมูลคะแนน
        </div>
      </div>
    </section>
  </main>

  <script>
    const scoreForm = document.getElementById("scoreForm");
    const studentName = document.getElementById("studentName");
    const scoreInput = document.getElementById("score");
    const statusInput = document.getElementById("status");
    const scoreTable = document.getElementById("scoreTable");
    const emptyMessage = document.getElementById("emptyMessage");
    const searchInput = document.getElementById("searchInput");

    let students = JSON.parse(localStorage.getItem("students")) || [
      {
        name: "ด.ช. กิตติพงษ์",
        score: 85,
        status: "published"
      },
      {
        name: "ด.ญ. พิมพ์ชนก",
        score: 92,
        status: "published"
      }
    ];

    function saveData() {
      localStorage.setItem("students", JSON.stringify(students));
    }

    function renderStudents() {
      const keyword = searchInput.value.toLowerCase().trim();

      const filteredStudents = students.filter(student =>
        student.name.toLowerCase().includes(keyword)
      );

      scoreTable.innerHTML = "";

      if (filteredStudents.length === 0) {
        emptyMessage.style.display = "block";
        return;
      }

      emptyMessage.style.display = "none";

      filteredStudents.forEach((student, index) => {
        const originalIndex = students.indexOf(student);

        const row = document.createElement("tr");

        row.innerHTML = `
          <td>${index + 1}</td>
          <td>${student.name}</td>
          <td class="score">${student.score}</td>
          <td>
            <span class="status ${student.status}">
              ${
                student.status === "published"
                  ? "ประกาศแล้ว"
                  : "ซ่อนไว้ก่อน"
              }
            </span>
          </td>
          <td>
            <button
              class="danger"
              onclick="deleteStudent(${originalIndex})"
            >
              ลบ
            </button>
          </td>
        `;

        scoreTable.appendChild(row);
      });
    }

    scoreForm.addEventListener("submit", function (event) {
      event.preventDefault();

      const newStudent = {
        name: studentName.value.trim(),
        score: Number(scoreInput.value),
        status: statusInput.value
      };

      if (!newStudent.name) {
        alert("กรุณากรอกชื่อนักเรียน");
        return;
      }

      if (newStudent.score < 0 || newStudent.score > 100) {
        alert("คะแนนต้องอยู่ระหว่าง 0 ถึง 100");
        return;
      }

      students.push(newStudent);
      saveData();
      renderStudents();

      scoreForm.reset();
      studentName.focus();
    });

    function deleteStudent(index) {
      const confirmed = confirm("ต้องการลบข้อมูลนักเรียนคนนี้หรือไม่?");

      if (!confirmed) return;

      students.splice(index, 1);
      saveData();
      renderStudents();
    }

    searchInput.addEventListener("input", renderStudents);

    renderStudents();
  </script>
</body>
</html>
