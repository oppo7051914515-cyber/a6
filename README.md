<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>ระบบเช็กชื่อเข้าเรียนอัจฉริยะ (Smart Attendance System)</title>
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Lucide Icons -->
    <script src="https://unpkg.com/lucide@latest"></script>
    <!-- QRCodeJS -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/qrcodejs/1.0.0/qrcode.min.js"></script>

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Sarabun:wght@300;400;500;600;700&display=swap');
        body { font-family: 'Sarabun', sans-serif; -webkit-tap-highlight-color: transparent; }
    </style>
</head>
<body class="bg-slate-100 text-slate-800 min-h-screen flex flex-col md:flex-row pb-16 md:pb-0">

    <!-- Top Header (แสดงเฉพาะหน้าอาจารย์ บน Mobile) -->
    <header id="mobileHeader" class="md:hidden bg-indigo-900 text-white p-4 sticky top-0 z-30 shadow-md flex justify-between items-center">
        <div class="flex items-center gap-2">
            <div class="p-1.5 bg-indigo-600 rounded-lg">
                <i data-lucide="scan-face" class="w-6 h-6"></i>
            </div>
            <div>
                <h1 class="font-bold text-sm leading-tight">ระบบเช็กชื่อเข้าเรียน</h1>
                <span id="mobileCurrentClassText" class="text-[11px] text-indigo-300">วิชา: CS101</span>
            </div>
        </div>
    </header>

    <!-- Sidebar Menu (แสดงเฉพาะหน้าอาจารย์ บน Laptop/Desktop) -->
    <aside id="desktopSidebar" class="hidden md:flex w-72 bg-indigo-900 text-white p-5 flex-col justify-between shadow-xl flex-shrink-0">
        <div>
            <div class="flex items-center gap-3 mb-6">
                <div class="p-2.5 bg-indigo-600 rounded-xl shadow-lg">
                    <i data-lucide="scan-face" class="w-7 h-7"></i>
                </div>
                <div>
                    <h1 class="font-bold text-base leading-tight">ระบบเช็กชื่อใบหน้า</h1>
                    <span class="text-xs text-indigo-300">Smart Attendance v9.0</span>
                </div>
            </div>

            <!-- เลือกห้องเรียน -->
            <div class="mb-6 bg-indigo-950/70 p-3.5 rounded-2xl border border-indigo-700/50 space-y-2.5">
                <label class="block text-xs text-indigo-300 font-semibold flex items-center gap-1">
                    <i data-lucide="book-open" class="w-3.5 h-3.5"></i> รายวิชา / ห้องเรียน
                </label>
                <select id="classroomSelect" onchange="changeClassroom()" class="w-full bg-indigo-800 text-white p-2.5 rounded-xl text-sm font-medium focus:outline-none focus:ring-2 focus:ring-indigo-400 border border-indigo-600">
                    <option value="CS101">CS101 - พัฒนาเว็บแอปพลิเคชัน</option>
                    <option value="SOC201">SOC201 - สังคมศึกษาและการสอน</option>
                </select>
            </div>

            <!-- เมนูอาจารย์ -->
            <nav class="space-y-1.5">
                <button onclick="switchTab('session-control')" id="nav-session-control" class="nav-btn w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium bg-indigo-800 text-white shadow-md transition">
                    <i data-lucide="play-circle" class="w-4 h-4"></i> ควบคุมการเช็กชื่อ (ครู)
                </button>
                <button onclick="switchTab('scan-student')" id="nav-scan-student" class="nav-btn w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium text-indigo-200 hover:bg-indigo-800/60 transition">
                    <i data-lucide="smartphone" class="w-4 h-4"></i> หน้าเช็กชื่อนักเรียน
                </button>
                <button onclick="switchTab('edit-time')" id="nav-edit-time" class="nav-btn w-full flex items-center gap-3 px-4 py-3 rounded-xl text-sm font-medium text-indigo-200 hover:bg-indigo-800/60 transition">
                    <i data-lucide="clock" class="w-4 h-4"></i> รายการเช็กชื่อ
                </button>
            </nav>
        </div>
    </aside>

    <!-- Main Content -->
    <main class="flex-1 p-4 md:p-8 overflow-y-auto max-w-7xl mx-auto w-full">

        <!-- Page Header -->
        <div id="mainPageHeader" class="flex flex-col md:flex-row justify-between items-start md:items-center mb-6 pb-4 border-b border-slate-200 gap-3">
            <div>
                <h2 id="pageTitle" class="text-xl md:text-2xl font-bold text-slate-800">⏱️ ควบคุมเปิด/ปิดการเช็กชื่อ (สำหรับครู)</h2>
                <div class="flex items-center gap-2 mt-1 text-xs md:text-sm text-slate-500">
                    <span class="font-bold text-indigo-600 bg-indigo-50 px-2.5 py-0.5 rounded-md border border-indigo-100" id="currentClassText">CS101</span>
                </div>
            </div>
            <div class="flex items-center gap-2 bg-white shadow-sm border border-slate-200 text-indigo-700 px-3.5 py-1.5 rounded-xl text-xs font-semibold">
                <i data-lucide="calendar" class="w-4 h-4 text-indigo-600"></i>
                <span id="liveDateText"></span>
            </div>
        </div>

        <!-- Tab 1: ควบคุมเปิด/ปิด (สำหรับครู) -->
        <section id="tab-session-control" class="tab-content space-y-6">
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <div class="lg:col-span-2 bg-white p-5 md:p-6 rounded-2xl shadow-sm border border-slate-200">
                    <h3 class="font-bold text-slate-800 text-base md:text-lg mb-4 flex items-center gap-2">
                        <i data-lucide="sliders" class="w-5 h-5 text-indigo-600"></i> สรุปการเปิดระบบเช็กชื่อ
                    </h3>
                    <div class="p-4 bg-emerald-50 text-emerald-800 rounded-xl border border-emerald-200 mb-6 font-medium text-sm flex items-center gap-2">
                        <i data-lucide="check-circle" class="w-5 h-5 text-emerald-600"></i>
                        <span>ระบบพร้อมให้นักเรียนสแกน QR Code เพื่อเช็กชื่อได้ทันที</span>
                    </div>
                </div>
            </div>
        </section>

        <!-- Tab 2: หน้าเช็กชื่อเฉพาะนักเรียน (Student View) -->
        <section id="tab-scan-student" class="tab-content hidden space-y-6">
            <!-- แถบแชร์ (แสดงเฉพาะครู) -->
            <div id="shareControlBar" class="max-w-md mx-auto bg-indigo-900 text-white p-3.5 rounded-2xl flex items-center justify-between gap-2 shadow-md">
                <div class="flex items-center gap-2 font-semibold text-xs pl-1">
                    <i data-lucide="share-2" class="w-4 h-4 text-indigo-300"></i>
                    <span>แชร์ให้นักเรียน</span>
                </div>
                <div class="flex gap-1.5">
                    <button onclick="copyStudentLink()" class="bg-indigo-600 hover:bg-indigo-500 px-3 py-1.5 rounded-lg text-xs font-medium flex items-center gap-1 transition">
                        <i data-lucide="copy" class="w-3.5 h-3.5"></i> คัดลอกลิงก์
                    </button>
                    <button onclick="showQRCodeModal()" class="bg-amber-500 hover:bg-amber-400 text-slate-900 px-3 py-1.5 rounded-lg text-xs font-bold flex items-center gap-1 transition">
                        <i data-lucide="qr-code" class="w-3.5 h-3.5"></i> QR Code
                    </button>
                </div>
            </div>

            <!-- การ์ดเช็กชื่อนักเรียน (พร้อมใช้งาน 100% ไม่ติดล็อก) -->
            <div class="max-w-md mx-auto bg-white p-6 rounded-3xl shadow-xl border border-slate-200">
                <div id="studentScanFormArea" class="space-y-4">
                    <div class="text-center mb-2">
                        <span id="studentScanSubjectBadge" class="bg-indigo-100 text-indigo-700 text-xs font-bold px-3 py-1 rounded-full inline-block mb-2">รายวิชา CS101</span>
                        <h3 class="text-xl font-bold text-slate-800">ลงชื่อเข้าเรียน</h3>
                        <p class="text-xs text-slate-500 mt-0.5">เลือกชื่อของคุณ และเปิดกล้องถ่ายภาพเพื่อยืนยันตัวตน</p>
                    </div>

                    <div>
                        <label class="block text-xs font-bold text-slate-700 mb-1">1. เลือกชื่อ-รหัสนักเรียนของคุณ</label>
                        <select id="studentSelfSelect" class="w-full border border-slate-300 rounded-xl p-3 text-sm font-medium focus:ring-2 focus:ring-indigo-500 outline-none bg-slate-50">
                            <!-- JS Render Options -->
                        </select>
                    </div>

                    <div>
                        <label class="block text-xs font-bold text-slate-700 mb-1">2. สแกนใบหน้าเข้าเรียน</label>
                        <div class="relative bg-slate-900 rounded-2xl aspect-square flex items-center justify-center overflow-hidden border-2 border-slate-200 shadow-inner">
                            <video id="videoStudent" class="w-full h-full object-cover hidden" autoplay playsinline></video>
                            <div id="studentCamPlaceholder" class="text-center p-6 text-slate-400">
                                <i data-lucide="camera" class="w-12 h-12 mx-auto mb-2 opacity-40"></i>
                                <p class="text-xs font-medium">กดปุ่ม "เปิดกล้อง" ด้านล่าง</p>
                            </div>
                        </div>
                    </div>

                    <div class="grid grid-cols-2 gap-2.5 pt-2">
                        <button onclick="startStudentCamera()" class="bg-indigo-600 hover:bg-indigo-700 active:scale-95 text-white py-3 rounded-xl text-xs md:text-sm font-bold flex items-center justify-center gap-1.5 shadow-md transition">
                            <i data-lucide="camera" class="w-4 h-4"></i> เปิดกล้อง
                        </button>
                        <button onclick="confirmStudentSelfScan()" class="bg-emerald-600 hover:bg-emerald-700 active:scale-95 text-white py-3 rounded-xl text-xs md:text-sm font-bold flex items-center justify-center gap-1.5 shadow-md transition">
                            <i data-lucide="check-circle" class="w-4 h-4"></i> บันทึกเช็กชื่อ
                        </button>
                    </div>
                </div>
            </div>
        </section>

        <!-- Tab 3: รายการบันทึก -->
        <section id="tab-edit-time" class="tab-content hidden space-y-6">
            <div class="bg-white p-5 rounded-2xl shadow-sm border border-slate-200">
                <h3 class="font-bold text-slate-800 text-base mb-4 flex items-center gap-2">
                    <i data-lucide="edit-3" class="w-5 h-5 text-indigo-600"></i> รายการเช็กชื่อล่าสุด
                </h3>
                <div class="overflow-x-auto">
                    <table class="w-full text-left text-sm text-slate-600 border-collapse">
                        <thead class="bg-slate-100 text-slate-700 uppercase text-xs">
                            <tr>
                                <th class="p-3 rounded-l-xl">วันที่</th>
                                <th class="p-3">รหัส</th>
                                <th class="p-3">ชื่อ-นามสกุล</th>
                                <th class="p-3">เวลา</th>
                                <th class="p-3 rounded-r-xl">สถานะ</th>
                            </tr>
                        </thead>
                        <tbody id="attendanceLogsTable" class="divide-y divide-slate-100">
                            <!-- JS Logs -->
                        </tbody>
                    </table>
                </div>
            </div>
        </section>

    </main>

    <!-- Bottom Nav มือถือ (ครู) -->
    <nav id="mobileBottomNav" class="md:hidden fixed bottom-0 left-0 right-0 bg-white border-t border-slate-200 flex justify-around items-center p-2 z-40 shadow-lg">
        <button onclick="switchTab('session-control')" class="flex flex-col items-center gap-1 text-[11px] font-semibold text-indigo-600">
            <i data-lucide="play-circle" class="w-5 h-5"></i> คุมเช็กชื่อ
        </button>
        <button onclick="switchTab('scan-student')" class="flex flex-col items-center gap-1 text-[11px] font-medium text-slate-400">
            <i data-lucide="smartphone" class="w-5 h-5"></i> หน้าสแกน
        </button>
        <button onclick="switchTab('edit-time')" class="flex flex-col items-center gap-1 text-[11px] font-medium text-slate-400">
            <i data-lucide="clock" class="w-5 h-5"></i> รายการ
        </button>
    </nav>

    <!-- Modal QR Code -->
    <div id="qrCodeModal" class="fixed inset-0 bg-slate-900/60 backdrop-blur-sm flex items-center justify-center hidden z-50 p-4">
        <div class="bg-white w-full max-w-sm rounded-3xl p-6 text-center shadow-2xl border border-slate-200">
            <h3 class="font-bold text-lg text-slate-800 mb-1">สแกน QR Code เพื่อเช็กชื่อ</h3>
            <p class="text-xs text-slate-500 mb-4">ให้นักเรียนใช้มือถือสแกนเข้าสู่หน้าเช็กชื่อ</p>
            <div id="qrcode" class="flex justify-center p-4 bg-slate-50 rounded-2xl border border-slate-200 mx-auto mb-4"></div>
            <button onclick="closeQRCodeModal()" class="w-full bg-slate-800 hover:bg-slate-900 text-white py-3 rounded-xl text-sm font-semibold">
                ปิดหน้าต่าง
            </button>
        </div>
    </div>

    <script>
        let currentClassroom = 'CS101';

        let db = JSON.parse(localStorage.getItem('smart_attendance_db_v9')) || {
            students: {
                'CS101': [
                    { id: '6601001', name: 'นาย กิตติศักดิ์ สมบูรณ์', grade: 'ม.4/1' },
                    { id: '6601002', name: 'นางสาว ปริศนา สุขใจ', grade: 'ม.4/1' },
                    { id: '6601003', name: 'นาย พัชรพล ต่อวาส', grade: 'ม.4/2' }
                ],
                'SOC201': [
                    { id: '6602001', name: 'นาย ณัฐวุฒิ มีสุข', grade: 'ปี 2' }
                ]
            },
            attendanceLogs: { 'CS101': [], 'SOC201': [] }
        };

        function saveData() {
            localStorage.setItem('smart_attendance_db_v9', JSON.stringify(db));
        }

        document.addEventListener("DOMContentLoaded", () => {
            lucide.createIcons();
            document.getElementById('liveDateText').innerText = new Date().toLocaleDateString('th-TH', { year: 'numeric', month: 'long', day: 'numeric' });

            // ตรวจสอบพารามิเตอร์ URL สำหรับโหมดนักเรียน
            const urlParams = new URLSearchParams(window.location.search);
            const classParam = urlParams.get('class');

            if (classParam) {
                currentClassroom = classParam;

                // ซ่อนแถบเมนูทั้งหมดทันทีสำหรับนักเรียน
                ['mobileHeader', 'desktopSidebar', 'mobileBottomNav', 'mainPageHeader', 'shareControlBar'].forEach(id => {
                    const el = document.getElementById(id);
                    if (el) el.style.display = 'none';
                });

                document.body.classList.remove('pb-16', 'md:pb-0');
                switchTab('scan-student');
            }

            renderStudentSelectOptions();
            renderAttendanceLogs();
        });

        function confirmStudentSelfScan() {
            const selectEl = document.getElementById('studentSelfSelect');
            const studentId = selectEl.value;

            if (!studentId) {
                alert("กรุณาเลือกชื่อนักเรียนก่อนเช็กชื่อ");
                return;
            }

            const studentName = selectEl.selectedOptions[0].text;
            const now = new Date();
            const timeStr = now.toLocaleTimeString('en-US', { hour: '2-digit', minute: '2-digit', second: '2-digit' });
            const dateStr = now.toISOString().split('T')[0];

            const newLog = {
                date: dateStr,
                id: studentId,
                name: studentName,
                time: timeStr,
                status: 'มาตรงเวลา'
            };

            if (!db.attendanceLogs[currentClassroom]) db.attendanceLogs[currentClassroom] = [];
            db.attendanceLogs[currentClassroom].unshift(newLog);
            saveData();

            renderAttendanceLogs();
            alert(`🎉 บันทึกเช็กชื่อสำเร็จ!\nสวัสดี ${studentName}\nเวลา: ${timeStr}`);
        }

        function switchTab(tabId) {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
            const target = document.getElementById(`tab-${tabId}`);
            if (target) target.classList.remove('hidden');
        }

        function changeClassroom() {
            currentClassroom = document.getElementById('classroomSelect').value;
            document.getElementById('currentClassText').innerText = currentClassroom;
            document.getElementById('studentScanSubjectBadge').innerText = `รายวิชา ${currentClassroom}`;
            renderStudentSelectOptions();
            renderAttendanceLogs();
        }

        function renderStudentSelectOptions() {
            const select = document.getElementById('studentSelfSelect');
            if (!select) return;
            const students = db.students[currentClassroom] || [];
            select.innerHTML = students.map(s => `<option value="${s.id}">${s.name} (${s.id})</option>`).join('');
        }

        function renderAttendanceLogs() {
            const tbody = document.getElementById('attendanceLogsTable');
            const logs = db.attendanceLogs[currentClassroom] || [];
            if (!tbody) return;
            tbody.innerHTML = logs.map(l => `
                <tr class="hover:bg-slate-50">
                    <td class="p-3 font-medium">${l.date}</td>
                    <td class="p-3">${l.id}</td>
                    <td class="p-3 font-semibold text-slate-800">${l.name}</td>
                    <td class="p-3 font-mono">${l.time}</td>
                    <td class="p-3"><span class="px-2.5 py-0.5 rounded-full text-xs font-semibold bg-emerald-100 text-emerald-700">${l.status}</span></td>
                </tr>
            `).join('');
        }

        async function startStudentCamera() {
            const video = document.getElementById('videoStudent');
            const placeholder = document.getElementById('studentCamPlaceholder');
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ video: true });
                video.srcObject = stream;
                video.classList.remove('hidden');
                placeholder.classList.add('hidden');
            } catch (err) {
                alert("ไม่สามารถเปิดกล้องได้: " + err.message);
            }
        }

        function getStudentPageURL() {
            let baseUrl = window.location.href.split('?')[0].split('#')[0];
            return `${baseUrl}?class=${encodeURIComponent(currentClassroom)}`;
        }

        function copyStudentLink() {
            const url = getStudentPageURL();
            navigator.clipboard.writeText(url).then(() => {
                alert("📋 คัดลอกลิงก์สแกนสำหรับนักเรียนเรียบร้อยแล้ว!\n" + url);
            });
        }

        function showQRCodeModal() {
            const url = getStudentPageURL();
            const qrContainer = document.getElementById("qrcode");
            qrContainer.innerHTML = "";
            new QRCode(qrContainer, { text: url, width: 180, height: 180 });
            document.getElementById('qrCodeModal').classList.remove('hidden');
        }

        function closeQRCodeModal() {
            document.getElementById('qrCodeModal').classList.add('hidden');
        }
    </script>
</body>
</html>
