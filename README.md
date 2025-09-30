<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>생태천 OX 스피드 퀴즈</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Jua&family=Noto+Sans+KR:wght@400;700&display=swap" rel="stylesheet">
    <script src="https://cdn.jsdelivr.net/npm/canvas-confetti@1.6.0/dist/confetti.browser.min.js"></script>
    <style>
        body {
            font-family: 'Noto Sans KR', sans-serif;
            overflow: hidden; /* Prevent scrollbar on page transitions */
        }
        .font-jua {
            font-family: 'Jua', sans-serif;
        }
        /* Glassmorphism card style */
        .card {
            background-color: rgba(255, 255, 255, 0.7);
            backdrop-filter: blur(10px);
            padding: 2rem;
            border-radius: 1.5rem;
            box-shadow: 0 20px 25px -5px rgba(0, 0, 0, 0.1), 0 8px 10px -6px rgba(0, 0, 0, 0.1);
        }
        /* Question animation */
        .question-card.active {
            animation: slideIn 0.5s ease-out forwards;
        }
        @keyframes slideIn {
            from {
                transform: translateX(100%);
                opacity: 0;
            }
            to {
                transform: translateX(0);
                opacity: 1;
            }
        }
    </style>
</head>
<body class="bg-gradient-to-r from-blue-300 to-green-300 flex items-center justify-center min-h-screen">

    <!-- 로딩 메시지 -->
    <div id="loadingMessage" class="absolute inset-0 bg-gray-900 bg-opacity-70 flex items-center justify-center text-white text-2xl z-50">
        서버에 연결 중...
    </div>

    <!-- 메인 컨테이너 -->
    <div id="quizContainer" class="max-w-md w-full p-4 hidden">

        <!-- 시작 페이지 -->
        <div id="startPage" class="card text-center transition-all duration-500">
            <h1 class="font-jua text-4xl mb-4 text-blue-800">생태천 OX 스피드 퀴즈</h1>
            <p class="mb-6 text-gray-700">생태천에 대한 O/X 퀴즈를 풀어보세요!</p>
            <div class="space-y-4">
                <input id="userId" type="text" placeholder="이름을 입력하세요" class="w-full p-3 rounded-lg border-2 border-gray-300 focus:outline-none focus:border-blue-500 transition-colors">
                <input id="userPhone" type="tel" placeholder="전화번호를 입력하세요 " class="w-full p-3 rounded-lg border-2 border-gray-300 focus:outline-none focus:border-blue-500 transition-colors">
                <div class="flex space-x-2">
                    <input id="userGrade" type="number" placeholder="학년" class="w-1/3 p-3 rounded-lg border-2 border-gray-300 focus:outline-none focus:border-blue-500 transition-colors">
                    <input id="userClass" type="number" placeholder="반" class="w-1/3 p-3 rounded-lg border-2 border-gray-300 focus:outline-none focus:border-blue-500 transition-colors">
                    <input id="userNumber" type="number" placeholder="번호" class="w-1/3 p-3 rounded-lg border-2 border-gray-300 focus:outline-none focus:border-blue-500 transition-colors">
                </div>
                <button id="startBtn" class="w-full py-3 px-6 rounded-lg text-white font-bold bg-green-500 hover:bg-green-600 transition-colors shadow-md transform hover:scale-105">시작하기</button>
            </div>
            <p class="mt-4 text-sm text-gray-500">※ 퀴즈 기록은 서버에 실시간으로 저장됩니다.</p>
            <button id="adminModeBtn" class="mt-4 text-sm text-gray-400 hover:underline">관리자 모드</button>
        </div>

        <!-- 퀴즈 페이지 -->
        <div id="quizPage" class="card text-center hidden transition-all duration-500">
            <div id="timerDisplay" class="font-jua text-2xl mb-4 text-red-600">0.00s</div>
            <div class="question-card">
                <p id="questionDisplay" class="text-xl mb-6 font-bold text-gray-800"></p>
                <div class="flex space-x-4">
                    <button id="oBtn" class="flex-1 py-4 text-white font-bold text-3xl rounded-lg bg-blue-500 hover:bg-blue-600 transition-colors shadow-lg transform hover:scale-105">O</button>
                    <button id="xBtn" class="flex-1 py-4 text-white font-bold text-3xl rounded-lg bg-red-500 hover:bg-red-600 transition-colors shadow-lg transform hover:scale-105">X</button>
                </div>
            </div>
            <div id="resultFeedback" class="mt-4 text-lg font-bold"></div>
        </div>

        <!-- 결과 페이지 -->
        <div id="resultPage" class="card text-center hidden transition-all duration-500">
            <h2 class="font-jua text-4xl mb-2 text-green-700">축하합니다!</h2>
            <p class="text-xl mb-4 font-bold text-gray-800">당신의 점수는?</p>
            <div id="finalScore" class="font-jua text-6xl text-purple-700 mb-2"></div>
            <p id="finalTime" class="text-xl text-gray-600"></p>
            <button id="restartBtn" class="w-full py-3 mt-6 px-6 rounded-lg text-white font-bold bg-blue-500 hover:bg-blue-600 transition-colors shadow-md transform hover:scale-105">다시 시작하기</button>
            <button id="rankingBtn" class="w-full py-3 mt-2 px-6 rounded-lg text-white font-bold bg-gray-500 hover:bg-gray-600 transition-colors shadow-md transform hover:scale-105">전체 랭킹 보기</button>
        </div>

        <!-- 랭킹 페이지 -->
        <div id="rankingPage" class="card hidden overflow-y-auto max-h-[80vh] transition-all duration-500">
            <h2 class="font-jua text-3xl text-center mb-4 text-blue-800">최고 기록 랭킹</h2>
            <table class="w-full text-left table-auto border-collapse">
                <thead class="bg-blue-100 sticky top-0">
                    <tr>
                        <th class="px-4 py-2 border border-blue-200">순위</th>
                        <th class="px-4 py-2 border border-blue-200">이름</th>
                        <th class="px-4 py-2 border border-blue-200">점수</th>
                        <th class="px-4 py-2 border border-blue-200">시간(s)</th>
                    </tr>
                </thead>
                <tbody id="rankingBody">
                </tbody>
            </table>
            <button id="backToMainBtn" class="w-full py-3 mt-6 px-6 rounded-lg text-white font-bold bg-gray-500 hover:bg-gray-600 transition-colors shadow-md transform hover:scale-105">메인으로</button>
        </div>
        
    </div>

    <!-- 관리자 로그인 모달 -->
    <div id="adminLoginModal" class="fixed inset-0 bg-gray-900 bg-opacity-50 hidden items-center justify-center p-4">
        <div class="bg-white p-6 rounded-lg shadow-xl w-full max-w-sm text-center">
            <div class="flex justify-end">
                <button id="adminCloseBtn" class="text-gray-500 hover:text-gray-700 text-2xl">&times;</button>
            </div>
            <h3 class="text-2xl font-bold mb-4">관리자 로그인</h3>
            <input id="adminPasswordInput" type="password" placeholder="비밀번호를 입력하세요" class="w-full p-3 rounded-lg border-2 border-gray-300 focus:outline-none focus:border-blue-500">
            <button id="adminLoginBtn" class="w-full py-3 mt-4 rounded-lg text-white font-bold bg-purple-500 hover:bg-purple-600">로그인</button>
        </div>
    </div>

    <!-- 관리자 랭킹 및 데이터 삭제 페이지 -->
    <div id="adminPage" class="card hidden p-4 overflow-y-auto max-h-screen w-full max-w-4xl transition-all duration-500">
        <h2 class="font-jua text-3xl text-center mb-4 text-blue-800">관리자 랭킹 (전체 기록)</h2>
        <div class="flex justify-between items-center mb-4">
            <button id="adminBackBtn" class="py-2 px-4 rounded-lg text-white font-bold bg-gray-500 hover:bg-gray-600 transition-colors">뒤로가기</button>
            <button id="clearDataBtn" class="py-2 px-4 rounded-lg text-white font-bold bg-red-500 hover:bg-red-600 transition-colors transform hover:scale-105 shadow-md">데이터 삭제</button>
        </div>
        <table class="w-full text-left table-auto border-collapse">
            <thead class="bg-blue-100 sticky top-0">
                <tr>
                    <th class="px-4 py-2 border border-blue-200">순위</th>
                    <th class="px-4 py-2 border border-blue-200">이름</th>
                    <th class="px-4 py-2 border border-blue-200">전화번호</th>
                    <th class="px-4 py-2 border border-blue-200">학년</th>
                    <th class="px-4 py-2 border border-blue-200">반</th>
                    <th class="px-4 py-2 border border-blue-200">번호</th>
                    <th class="px-4 py-2 border border-blue-200">점수</th>
                    <th class="px-4 py-2 border border-blue-200">시간(s)</th>
                </tr>
            </thead>
            <tbody id="adminRankBody">
            </tbody>
        </table>
    </div>
    
    <script type="module">
        // Firebase 라이브러리 임포트
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, onSnapshot, query, getDocs, deleteDoc, doc, getDoc } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Firebase 초기화 (캔버스 환경에서 자동으로 제공되는 변수 사용)
        const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);

        let currentUser = null;
        let isAuthReady = false;

        // Firestore 컬렉션 경로
        const QUIZ_RECORDS_COLLECTION_PATH = `artifacts/${appId}/public/data/quiz_records`;

        // DOM elements
        const startPage = document.getElementById('startPage');
        const quizPage = document.getElementById('quizPage');
        const resultPage = document.getElementById('resultPage');
        const rankingPage = document.getElementById('rankingPage');
        const adminPage = document.getElementById('adminPage');
        const adminLoginModal = document.getElementById('adminLoginModal');
        const loadingMessage = document.getElementById('loadingMessage');

        const userIdInput = document.getElementById('userId');
        const userPhoneInput = document.getElementById('userPhone');
        const userGradeInput = document.getElementById('userGrade');
        const userClassInput = document.getElementById('userClass');
        const userNumberInput = document.getElementById('userNumber');

        const startBtn = document.getElementById('startBtn');
        const restartBtn = document.getElementById('restartBtn');
        const rankingBtn = document.getElementById('rankingBtn');
        const backToMainBtn = document.getElementById('backToMainBtn');
        const adminModeBtn = document.getElementById('adminModeBtn');
        const adminCloseBtn = document.getElementById('adminCloseBtn');
        const adminLoginBtn = document.getElementById('adminLoginBtn');
        const adminBackBtn = document.getElementById('adminBackBtn');
        const adminPasswordInput = document.getElementById('adminPasswordInput');
        const clearDataBtn = document.getElementById('clearDataBtn');
        
        const timerDisplay = document.getElementById('timerDisplay');
        const questionDisplay = document.getElementById('questionDisplay');
        const finalScoreDisplay = document.getElementById('finalScore');
        const finalTimeDisplay = document.getElementById('finalTime');
        const resultFeedback = document.getElementById('resultFeedback');
        const oBtn = document.getElementById('oBtn');
        const xBtn = document.getElementById('xBtn');
        const rankingBody = document.getElementById('rankingBody');
        const adminRankBody = document.getElementById('adminRankBody');

        // Quiz data
        const originalQuestions = [
            { question: "생태하천은 오염된 물을 스스로 정화하는 능력이 있다.", answer: true },
            { question: "모든 하천의 물고기는 1급수에서만 살 수 있다.", answer: false },
            { question: "수질 정화를 위해 하천 바닥을 모두 콘크리트로 덮는 것이 좋다.", answer: false },
            { question: "버드나무와 갈대는 수질 정화에 도움을 주는 대표적인 수생식물이다.", answer: true },
            { question: "생태하천 복원 사업은 동식물의 서식지를 파괴하는 주된 원인이다.", answer: false },
            { question: "빗물이 하천으로 바로 유입되면 수질 오염의 원인이 될 수 있다.", answer: true },
            { question: "하천에 사는 잠자리의 애벌레는 물 밖 풀숲에서 생활한다.", answer: false },
            { question: "생활하수를 정화 없이 하천으로 흘려보내는 것은 생태계를 풍요롭게 한다.", answer: false },
            { question: "생태하천의 돌과 자갈은 미생물이 살 공간을 제공하여 물을 깨끗하게 한다.", answer: true },
            { question: "생태하천에서는 어떠한 경우에도 낚시나 물놀이가 금지되어 있다.", answer: false },
        ];
        let questions = [];

        // Game state
        let currentQuestionIndex = 0;
        let score = 0;
        let startTime = 0;
        let timerInterval = null;
        let userName = '';
        let userPhone = '';
        let userGrade = '';
        let userClass = '';
        let userNumber = '';

        // Firebase 인증 상태 리스너
        onAuthStateChanged(auth, async (user) => {
            if (user) {
                currentUser = user;
            } else {
                try {
                    await signInAnonymously(auth);
                } catch (error) {
                    console.error("Firebase 익명 로그인 실패:", error);
                }
            }
            isAuthReady = true;
            loadingMessage.classList.add('hidden');
            document.getElementById('quizContainer').classList.remove('hidden');
        });

        // --- View Controller Functions ---
        function showPage(pageId) {
            const pages = [startPage, quizPage, resultPage, rankingPage, adminPage];
            pages.forEach(page => {
                page.classList.add('hidden');
                page.classList.remove('flex');
            });

            if (pageId === 'start') {
                startPage.classList.remove('hidden');
                userIdInput.value = '';
                userPhoneInput.value = '';
                userGradeInput.value = '';
                userClassInput.value = '';
                userNumberInput.value = '';
            } else if (pageId === 'quiz') {
                quizPage.classList.remove('hidden');
            } else if (pageId === 'result') {
                resultPage.classList.remove('hidden');
            } else if (pageId === 'ranking') {
                rankingPage.classList.remove('hidden');
            } else if (pageId === 'admin') {
                adminPage.classList.remove('hidden');
            }
        }

        // --- Game Logic Functions ---
        function startGame() {
            userName = userIdInput.value.trim();
            userPhone = userPhoneInput.value.trim();
            userGrade = userGradeInput.value.trim();
            userClass = userClassInput.value.trim();
            userNumber = userNumberInput.value.trim();
            if (!userName) {
                showModal('이름을 입력해주세요!');
                return;
            }

            score = 0;
            currentQuestionIndex = 0;
            questions = [...originalQuestions]; // Make a copy
            questions.sort(() => Math.random() - 0.5); // Shuffle questions
            showPage('quiz');
            startTimer();
            displayQuestion();
        }

        function restartGame() {
            showPage('start');
        }

        function startTimer() {
            startTime = Date.now();
            timerInterval = setInterval(() => {
                const elapsedTime = (Date.now() - startTime) / 1000;
                timerDisplay.textContent = `${elapsedTime.toFixed(2)}s`;
            }, 10);
        }

        function stopTimer() {
            clearInterval(timerInterval);
        }

        function displayQuestion() {
            if (currentQuestionIndex < questions.length) {
                questionDisplay.textContent = questions[currentQuestionIndex].question;
                // Animate question card
                const questionCard = quizPage.querySelector('.question-card');
                questionCard.classList.remove('active');
                setTimeout(() => {
                    questionCard.classList.add('active');
                }, 10);
                resultFeedback.textContent = '';
            } else {
                endGame();
            }
        }

        function selectAnswer(event) {
            const selectedAnswer = event.target.textContent;
            const isCorrect = (selectedAnswer === 'O' && questions[currentQuestionIndex].answer) ||
                              (selectedAnswer === 'X' && !questions[currentQuestionIndex].answer);

            if (isCorrect) {
                score++;
                resultFeedback.textContent = '정답입니다!';
                resultFeedback.classList.remove('text-red-600');
                resultFeedback.classList.add('text-green-600');
            } else {
                resultFeedback.textContent = '아쉽네요... 정답은 ' + (questions[currentQuestionIndex].answer ? 'O' : 'X') + '입니다.';
                resultFeedback.classList.remove('text-green-600');
                resultFeedback.classList.add('text-red-600');
            }

            // Disable buttons temporarily
            oBtn.disabled = true;
            xBtn.disabled = true;

            setTimeout(() => {
                currentQuestionIndex++;
                displayQuestion();
                oBtn.disabled = false;
                xBtn.disabled = false;
            }, 1500);
        }

        function endGame() {
            stopTimer();
            const totalTime = (Date.now() - startTime) / 1000;
            finalScoreDisplay.textContent = `${score}점`;
            finalTimeDisplay.textContent = `${totalTime.toFixed(2)}초`;
            showPage('result');
            runConfetti();
            
            // Save record
            saveRecord(totalTime);
        }

        function runConfetti() {
            confetti({
                particleCount: 150,
                spread: 180
            });
        }
        
        function showModal(message) {
            const modal = document.createElement('div');
            modal.className = "fixed inset-0 bg-gray-900 bg-opacity-50 flex items-center justify-center z-50";
            modal.innerHTML = `
                <div class="bg-white p-6 rounded-lg shadow-xl w-full max-w-sm text-center">
                    <p class="text-xl font-bold mb-4">${message}</p>
                    <button class="py-2 px-4 rounded-lg text-white font-bold bg-blue-500 hover:bg-blue-600 transition-colors" onclick="this.parentElement.parentElement.remove()">확인</button>
                </div>
            `;
            document.body.appendChild(modal);
        }

        // --- Data Management Functions (Firestore) ---
        async function saveRecord(totalTime) {
            if (!isAuthReady || !currentUser) {
                console.error("Firebase 인증이 완료되지 않았습니다.");
                return;
            }
            const newRecord = {
                id: userName,
                phone: userPhone,
                grade: userGrade,
                class: userClass,
                number: userNumber,
                score: score,
                time: totalTime,
                timestamp: Date.now()
            };
            try {
                await addDoc(collection(db, QUIZ_RECORDS_COLLECTION_PATH), newRecord);
            } catch (error) {
                console.error("데이터 저장 실패:", error);
            }
        }
        
        // 랭킹 데이터를 실시간으로 가져와서 화면에 표시 (onSnapshot 사용)
        onSnapshot(query(collection(db, QUIZ_RECORDS_COLLECTION_PATH)), (snapshot) => {
            const rankings = [];
            snapshot.forEach((doc) => {
                rankings.push(doc.data());
            });

            // 점수 내림차순, 시간 오름차순으로 정렬
            rankings.sort((a, b) => {
                if (b.score !== a.score) {
                    return b.score - a.score;
                }
                return a.time - b.time;
            });
            
            // 일반 사용자 랭킹 테이블 업데이트
            displayRankings(rankings);
            // 관리자 랭킹 테이블 업데이트
            displayAdminRankings(rankings);
        }, (error) => {
            console.error("랭킹 데이터 가져오기 실패:", error);
        });

        function displayRankings(rankings) {
            rankingBody.innerHTML = '';
            if (rankings.length === 0) {
                 rankingBody.innerHTML = '<tr><td colspan="4" class="px-4 py-2 text-center text-gray-500">등록된 기록이 없습니다.</td></tr>';
            } else {
                rankings.forEach((rank, index) => {
                    const tr = document.createElement('tr');
                    tr.className = "bg-white even:bg-blue-50";
                    tr.innerHTML = `
                        <td class="px-4 py-2 border border-blue-200 font-bold">${index + 1}</td>
                        <td class="px-4 py-2 border border-blue-200">${rank.id}</td>
                        <td class="px-4 py-2 border border-blue-200 text-blue-600 font-bold">${rank.score}</td>
                        <td class="px-4 py-2 border border-blue-200 text-red-600">${rank.time.toFixed(2)}</td>
                    `;
                    rankingBody.appendChild(tr);
                });
            }
        }

        function displayAdminRankings(rankings) {
            adminRankBody.innerHTML = '';
            if (rankings.length === 0) {
                 adminRankBody.innerHTML = '<tr><td colspan="8" class="px-4 py-2 text-center text-gray-500">등록된 기록이 없습니다.</td></tr>';
                 return;
            }
            rankings.forEach((rank, index) => {
                const tr = document.createElement('tr');
                tr.className = "bg-white even:bg-blue-50";
                tr.innerHTML = `
                    <td class="px-4 py-2 border border-blue-200 font-bold">${index + 1}</td>
                    <td class="px-4 py-2 border border-blue-200">${rank.id}</td>
                    <td class="px-4 py-2 border border-blue-200">${rank.phone}</td>
                    <td class="px-4 py-2 border border-blue-200">${rank.grade}</td>
                    <td class="px-4 py-2 border border-blue-200">${rank.class}</td>
                    <td class="px-4 py-2 border border-blue-200">${rank.number}</td>
                    <td class="px-4 py-2 border border-blue-200 text-blue-600 font-bold">${rank.score}</td>
                    <td class="px-4 py-2 border border-blue-200 text-red-600">${rank.time.toFixed(2)}</td>
                `;
                adminRankBody.appendChild(tr);
            });
        }
        
        function clearAllData() {
            showModal('정말로 모든 데이터를 삭제하시겠습니까?<br>이 작업은 되돌릴 수 없습니다.<br><button class="py-2 px-4 rounded-lg text-white font-bold bg-red-500 hover:bg-red-600 transition-colors mt-4" onclick="confirmClear()">예, 삭제합니다</button><button class="py-2 px-4 rounded-lg text-white font-bold bg-gray-500 hover:bg-gray-600 transition-colors mt-4 ml-2" onclick="this.parentElement.parentElement.remove()">아니요</button>');
        }
        
        async function confirmClear() {
            // 모든 문서 삭제
            const querySnapshot = await getDocs(collection(db, QUIZ_RECORDS_COLLECTION_PATH));
            const batchSize = 100;
            const docsToDelete = [];
            querySnapshot.forEach((doc) => {
                docsToDelete.push(doc.ref);
            });

            for (let i = 0; i < docsToDelete.length; i += batchSize) {
                const batch = [];
                for (let j = 0; j < batchSize && (i + j) < docsToDelete.length; j++) {
                    batch.push(deleteDoc(docsToDelete[i + j]));
                }
                await Promise.all(batch);
            }

            showModal('모든 기록이 삭제되었습니다.');
            const modals = document.querySelectorAll('.fixed.inset-0');
            modals.forEach(modal => modal.remove());
        }
        
        // --- Admin Mode Logic ---
        const ADMIN_PASSWORD = 'eco';
        
        function showAdminLogin() {
            adminLoginModal.classList.remove('hidden');
            adminPasswordInput.value = '';
        }

        function handleAdminLogin() {
            if (adminPasswordInput.value === ADMIN_PASSWORD) {
                adminLoginModal.classList.add('hidden');
                showPage('admin');
            } else {
                showModal('비밀번호가 틀렸습니다.');
                adminPasswordInput.value = '';
            }
        }
        
        // --- Phone Number Formatting Logic ---
        function formatPhoneNumber(input) {
            let phoneNumber = input.value.replace(/[^0-9]/g, '');
            let formattedNumber = '';

            if (phoneNumber.length < 4) {
                formattedNumber = phoneNumber;
            } else if (phoneNumber.length < 8) {
                formattedNumber = `${phoneNumber.substring(0, 3)}-${phoneNumber.substring(3, 7)}`;
            } else {
                formattedNumber = `${phoneNumber.substring(0, 3)}-${phoneNumber.substring(3, 7)}-${phoneNumber.substring(7, 11)}`;
            }
            input.value = formattedNumber;
        }

        // --- Event Listeners ---
        startBtn.addEventListener('click', startGame);
        restartBtn.addEventListener('click', restartGame);
        rankingBtn.addEventListener('click', () => showPage('ranking'));
        backToMainBtn.addEventListener('click', () => showPage('start'));
        oBtn.addEventListener('click', selectAnswer);
        xBtn.addEventListener('click', selectAnswer);
        adminModeBtn.addEventListener('click', showAdminLogin);
        adminCloseBtn.addEventListener('click', () => adminLoginModal.classList.add('hidden'));
        adminLoginBtn.addEventListener('click', handleAdminLogin);
        adminBackBtn.addEventListener('click', restartGame);
        clearDataBtn.addEventListener('click', clearAllData);
        adminPasswordInput.addEventListener('keyup', (e) => {
            if (e.key === 'Enter') {
                handleAdminLogin();
            }
        });
        
        userPhoneInput.addEventListener('input', (e) => {
            formatPhoneNumber(e.target);
        });
    </script>
</body>
</html>

