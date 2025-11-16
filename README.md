<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>일본어 동사 복습 퀴즈 🌸</title>
    <!-- Tailwind CSS 로드 -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- html2canvas (결과 이미지 저장용) 로드 -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    
    <style>
        /* 기본 폰트 및 배경 설정 */
        body {
            font-family: 'Inter', 'Pretendard', sans-serif;
            background-color: #FFF9F9; /* 연한 핑크빛 배경 */
        }
        
        /* 화면 전환을 위한 기본 클래스 */
        .screen {
            display: none; /* 기본적으로 모든 화면 숨김 */
        }
        
        /* 현재 활성화된 화면 */
        .screen.active {
            display: flex;
        }

        /* 옵션 버튼 기본 스타일 */
        .option-btn {
            transition: all 0.2s ease;
            border: 2px solid #FBCFE8; /* 핑크색 테두리 */
        }
        
        /* 정답 버튼 스타일 */
        .option-btn.correct {
            background-color: #D1FAE5; /* 연한 녹색 */
            border-color: #10B981; /* 녹색 */
            transform: scale(1.03);
            font-weight: bold;
        }

        /* 오답 버튼 스타일 */
        .option-btn.incorrect {
            background-color: #FEE2E2; /* 연한 빨강 */
            border-color: #F87171; /* 빨강 */
            opacity: 0.7;
        }

        /* 비활성화된 버튼 (선택 후) */
        .option-btn.disabled {
            pointer-events: none;
            opacity: 0.8;
        }
        
        /* 레벨 선택 버튼 비활성화 (완료) */
        .level-btn.completed {
            background-color: #D1D5DB; /* 회색 */
            cursor: not-allowed;
            opacity: 0.7;
        }

        /* 폭죽 효과를 위한 컨테이너 */
        #confetti-container {
            position: absolute;
            top: 50%;
            left: 50%;
            width: 1px;
            height: 1px;
            overflow: visible;
            z-index: 1000;
            pointer-events: none;
        }

        /* 개별 폭죽 조각 */
        .confetti {
            position: absolute;
            width: 10px;
            height: 10px;
            opacity: 0;
            /* 애니메이션 설정 */
            animation: confetti-explode 0.7s ease-out forwards;
        }

        /* 폭죽 애니메이션 */
        @keyframes confetti-explode {
            0% {
                transform: translate(0, 0) rotate(0deg);
                opacity: 1;
            }
            100% {
                transform: translate(var(--x), var(--y)) rotate(360deg);
                opacity: 0;
            }
        }
        
        /* 다시 풀기 모달 */
        #requiz-modal {
            display: none; /* 기본 숨김 */
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background-color: rgba(0, 0, 0, 0.6);
            z-index: 50;
        }
        #requiz-modal.show {
            display: flex; /* 보이기 */
            justify-content: center;
            align-items: center;
        }
    </style>
</head>
<body class="min-h-screen flex items-center justify-center p-4">

    <div id="app" class="max-w-md w-full">

        <!-- 1. 이름 입력 화면 -->
        <div id="screen-start" class="screen active flex-col items-center bg-white p-8 rounded-2xl shadow-xl border-4 border-pink-200">
            <h1 class="text-4xl font-bold text-pink-500 mb-2">🌸 일본어 동사 퀴즈 🌸</h1>
            <p class="text-lg text-gray-600 mb-6">당신의 이름을 입력해주세요!</p>
            <input id="name-input" type="text" placeholder="예) 김성보" class="w-full p-3 border-2 border-pink-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-pink-400 mb-4">
            <button id="start-btn" class="w-full bg-pink-400 text-white p-3 rounded-lg font-bold text-lg hover:bg-pink-500 transition-colors shadow-md">
                퀴즈 시작하기 🍙
            </button>
        </div>

        <!-- 2. 레벨 선택 화면 -->
        <div id="screen-level-select" class="screen flex-col items-center bg-white p-8 rounded-2xl shadow-xl border-4 border-pink-200">
            <h2 id="welcome-message" class="text-2xl font-bold text-gray-700 mb-6 text-center"></h2>
            <p class="text-lg text-gray-600 mb-6; text-center">모든 단계 문제를 풀고<br>최종 결과지를 리로스쿨에 업로드해주세요! 🐱<br> </p>
            <div class="w-full space-y-4">
                <button data-level="easy" class="level-btn w-full bg-green-400 text-white p-4 rounded-lg font-bold text-lg hover:bg-green-500 transition-colors shadow-md">
                    하 (下) - 히라가나 ↔ 뜻
                </button>
                <button data-level="medium" class="level-btn w-full bg-blue-400 text-white p-4 rounded-lg font-bold text-lg hover:bg-blue-500 transition-colors shadow-md">
                    중 (中) - 한자 ↔ 히라가나
                </button>
                <button data-level="hard" class="level-btn w-full bg-red-400 text-white p-4 rounded-lg font-bold text-lg hover:bg-red-500 transition-colors shadow-md">
                    상 (上) - 한자 ↔ 뜻
                </button>
                <button id="view-results-btn" class="w-full bg-purple-500 text-white p-4 rounded-lg font-bold text-lg hover:bg-purple-600 transition-colors shadow-md mt-6" style="display: none;">
                    최종 결과 보기 📜
                </button>
            </div>
        </div>

        <!-- 3. 퀴즈 화면 -->
        <div id="screen-quiz" class="screen flex-col w-full">
            <div id="confetti-container"></div>
            <!-- 다시 풀기 모달 -->
            <div id="requiz-modal">
                <div class="bg-white p-6 rounded-lg shadow-xl text-2xl font-bold text-purple-600 animate-pulse">
                    오답 문제 확인! 다시 한 번 풀어보세요 🔁
                </div>
            </div>
            
            <div class="bg-white p-6 rounded-2xl shadow-xl border-4 border-pink-200 w-full">
                <div class="flex justify-between items-center mb-4">
                    <span id="quiz-level" class="text-sm font-bold text-pink-500 px-3 py-1 bg-pink-100 rounded-full"></span>
                    <span id="quiz-progress" class="text-sm font-bold text-gray-500"></span>
                </div>
                
                <div class="bg-gray-50 p-6 rounded-lg mb-6 shadow-inner min-h-[100px] flex items-center justify-center">
                    <h3 id="question-text" class="text-3xl md:text-4xl font-bold text-gray-800 text-center"></h3>
                </div>
                
                <div id="options-container" class="grid grid-cols-1 gap-3">
                    <!-- JS로 옵션 버튼 추가 -->
                </div>
            </div>
        </div>
        
        <!-- 4. 레벨 완료 화면 -->
        <div id="screen-level-complete" class="screen flex-col items-center bg-white p-8 rounded-2xl shadow-xl border-4 border-pink-200">
            <h2 class="text-3xl font-bold text-pink-500 mb-4 text-center">문제를 모두 풀었어요!</h2>
            <p class="text-4xl font-bold text-gray-700">よくできました！ 🎉</p>
        </div>

        <!-- 5. 최종 결과 화면 -->
        <div id="screen-final-results" class="screen flex-col items-center w-full">
            <div id="results-content" class="bg-white p-6 sm:p-8 rounded-2xl shadow-xl border-4 border-purple-200 w-full">
                <h2 id="results-title" class="text-2xl sm:text-3xl font-bold text-purple-600 mb-6 text-center"></h2>
                
                <!-- 피드백 -->
                <div class="bg-purple-50 border-2 border-purple-200 p-4 rounded-lg mb-6">
                    <h3 class="text-lg font-bold text-purple-800 mb-2">🎓 학습 피드백</h3>
                    <p id="feedback-text" class="text-gray-700"></p>
                </div>

                <!-- 결과 테이블 -->
                <div class="overflow-x-auto">
                    <table class="min-w-full divide-y divide-gray-200 border border-gray-200">
                        <thead class="bg-gray-50">
                            <tr>
                                <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">레벨</th>
                                <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">문제</th>
                                <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">정답</th>
                                <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">제출 답안</th>
                                <th class="px-4 py-2 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">결과</th>
                            </tr>
                        </thead>
                        <tbody id="results-table-body" class="bg-white divide-y divide-gray-200">
                            <!-- JS로 결과 행 추가 -->
                        </tbody>
                    </table>
                </div>
            </div>
            
            <!-- 버튼 컨테이너 -->
            <div class="w-full mt-6 space-y-3">
                <button id="save-image-btn" class="w-full bg-green-500 text-white p-3 rounded-lg font-bold text-lg hover:bg-green-600 transition-colors shadow-md">
                    결과 이미지로 저장 💾
                </button>
                <button id="restart-btn" class="w-full bg-pink-400 text-white p-3 rounded-lg font-bold text-lg hover:bg-pink-500 transition-colors shadow-md">
                    처음으로 돌아가기 🏠
                </button>
            </div>
        </div>
    </div>

    <script>
        // --- 1. 데이터 및 상태 변수 ---
        
        // 제공된 단어 목록 (한자 추가)
        const vocabulary = [
            { kanji: '買う', hiragana: 'かう', meaning: '사다' },
            { kanji: '行く', hiragana: 'いく', meaning: '가다' },
            { kanji: '話す', hiragana: 'はなす', meaning: '이야기하다' },
            { kanji: '遊ぶ', hiragana: 'あそぶ', meaning: '놀다' },
            { kanji: '飲む', hiragana: 'のむ', meaning: '마시다' },
            { kanji: '座る', hiragana: 'すわる', meaning: '앉다' },
            { kanji: '作る', hiragana: 'つくる', meaning: '만들다' },
            { kanji: '登る', hiragana: 'のぼる', meaning: '오르다' },
            { kanji: '見る', hiragana: 'みる', meaning: '보다' },
            { kanji: '食べる', hiragana: 'たべる', meaning: '먹다' },
            { kanji: '習う', hiragana: 'ならう', meaning: '배우다' },
            { kanji: '会う', hiragana: 'あう', meaning: '만나다' },
            { kanji: '聞く', hiragana: 'きく', meaning: '듣다' },
            { kanji: '読む', hiragana: 'よむ', meaning: '읽다' },
            { kanji: '寝る', hiragana: 'ねる', meaning: '자다' },
            { kanji: '起きる', hiragana: 'おきる', meaning: '일어나다' },
            { kanji: '帰る', hiragana: 'かえる', meaning: '돌아가(오)다' }
        ];

        // 앱 상태
        let state = {
            userName: "",
            currentScreen: "screen-start",
            currentLevel: "", // 'easy', 'medium', 'hard'
            currentQuestions: [],
            currentQuestionIndex: 0,
            mistakes: [], // 틀린 문제 추적
            allResults: [], // 모든 레벨의 결과 저장
            isReQuiz: false, // 다시 풀기 모드 여부
            completedLevels: { easy: false, medium: false, hard: false }
        };

        // --- 2. DOM 요소 ---
        const screens = document.querySelectorAll('.screen');
        const startBtn = document.getElementById('start-btn');
        const nameInput = document.getElementById('name-input');
        const welcomeMessage = document.getElementById('welcome-message');
        const levelSelectScreen = document.getElementById('screen-level-select');
        const levelButtons = document.querySelectorAll('.level-btn');
        const quizScreen = document.getElementById('screen-quiz');
        const quizLevel = document.getElementById('quiz-level');
        const quizProgress = document.getElementById('quiz-progress');
        const questionText = document.getElementById('question-text');
        const optionsContainer = document.getElementById('options-container');
        const levelCompleteScreen = document.getElementById('screen-level-complete');
        const finalResultsScreen = document.getElementById('screen-final-results');
        const resultsTitle = document.getElementById('results-title');
        const feedbackText = document.getElementById('feedback-text');
        const resultsTableBody = document.getElementById('results-table-body');
        const saveImageBtn = document.getElementById('save-image-btn');
        const restartBtn = document.getElementById('restart-btn');
        const viewResultsBtn = document.getElementById('view-results-btn');
        const confettiContainer = document.getElementById('confetti-container');
        const requizModal = document.getElementById('requiz-modal');
        const resultsContent = document.getElementById('results-content');

        // --- 3. 헬퍼 함수 ---

        // 화면 전환 함수
        function showScreen(screenId) {
            screens.forEach(screen => {
                screen.classList.remove('active');
            });
            document.getElementById(screenId).classList.add('active');
            state.currentScreen = screenId;
        }

        // 배열 섞기 함수
        function shuffleArray(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]];
            }
            return array;
        }

        // --- 4. 퀴즈 생성 로직 ---

        /**
         * 레벨별 퀴즈 질문 생성
         * @param {string} level - 'easy', 'medium', 'hard'
         */
        function generateQuestions(level) {
            const questions = [];
            const shuffledVocab = shuffleArray([...vocabulary]);

            // 각 단어에 대해 1문제씩 생성
            shuffledVocab.forEach(word => {
                let questionType;
                
                // 레벨별로 2가지 유형 중 랜덤 선택
                if (level === 'easy') {
                    questionType = Math.random() < 0.5 ? 'hiragana-to-meaning' : 'meaning-to-hiragana';
                } else if (level === 'medium') {
                    questionType = Math.random() < 0.5 ? 'kanji-to-hiragana' : 'hiragana-to-kanji';
                } else { // hard
                    questionType = Math.random() < 0.5 ? 'kanji-to-meaning' : 'meaning-to-kanji';
                }
                
                questions.push(createMultipleChoice(word, questionType));
            });
            
            return questions;
        }

        /**
         * 4지선다 문제 생성
         * @param {object} correctWord - 정답 단어 객체
         * @param {string} type - 문제 유형 (e.g., 'hiragana-to-meaning')
         */
        function createMultipleChoice(correctWord, type) {
            let question, answer, optionsFromProperty;

            switch (type) {
                case 'hiragana-to-meaning':
                    question = correctWord.hiragana;
                    answer = correctWord.meaning;
                    optionsFromProperty = 'meaning';
                    break;
                case 'meaning-to-hiragana':
                    question = correctWord.meaning;
                    answer = correctWord.hiragana;
                    optionsFromProperty = 'hiragana';
                    break;
                case 'kanji-to-hiragana':
                    question = correctWord.kanji;
                    answer = correctWord.hiragana;
                    optionsFromProperty = 'hiragana';
                    break;
                case 'hiragana-to-kanji':
                    question = correctWord.hiragana;
                    answer = correctWord.kanji;
                    optionsFromProperty = 'kanji';
                    break;
                case 'kanji-to-meaning':
                    question = correctWord.kanji;
                    answer = correctWord.meaning;
                    optionsFromProperty = 'meaning';
                    break;
                case 'meaning-to-kanji':
                    question = correctWord.meaning;
                    answer = correctWord.kanji;
                    optionsFromProperty = 'kanji';
                    break;
            }

            // 오답 선지 만들기
            const wrongOptions = vocabulary
                .filter(word => word[optionsFromProperty] !== answer) // 정답 제외
                .sort(() => 0.5 - Math.random()) // 섞기
                .slice(0, 3) // 3개 선택
                .map(word => word[optionsFromProperty]);

            const options = shuffleArray([answer, ...wrongOptions]);
            
            return {
                originalWord: correctWord, // 결과 분석을 위해 원본 단어 저장
                questionType: type, // 피드백 생성을 위해 유형 저장
                question: question,
                options: options,
                answer: answer
            };
        }
        
        // --- 5. 퀴즈 진행 로직 ---

        /**
         * 새 질문 로드
         */
        function loadQuestion() {
            // '다시 풀기' 모드일 때 모달 표시
            if (state.isReQuiz) {
                requizModal.classList.add('show');
                setTimeout(() => requizModal.classList.remove('show'), 1500);
            } else {
                requizModal.classList.remove('show');
            }
        
            const q = state.currentQuestions[state.currentQuestionIndex];
            
            // 레벨 및 진행도 표시
            let levelText = "";
            if (state.currentLevel === 'easy') levelText = "하 (下) 🌸";
            else if (state.currentLevel === 'medium') levelText = "중 (中) 🐱";
            else levelText = "상 (上) 🍙";
            
            if (state.isReQuiz) levelText += " (다시 풀기)";
            
            quizLevel.textContent = levelText;
            quizProgress.textContent = `문제 ${state.currentQuestionIndex + 1} / ${state.currentQuestions.length}`;
            
            questionText.textContent = q.question;
            optionsContainer.innerHTML = ""; // 이전 옵션 초기화

            // 옵션 버튼 생성
            q.options.forEach(option => {
                const button = document.createElement('button');
                button.textContent = option;
                button.classList.add('option-btn', 'w-full', 'p-4', 'rounded-lg', 'font-semibold', 'text-gray-700', 'bg-white', 'hover:bg-pink-50', 'shadow');
                button.onclick = () => selectAnswer(option, button);
                optionsContainer.appendChild(button);
            });
        }

        /**
         * 답안 선택 시
         * @param {string} selectedOption - 사용자가 선택한 답
         * @param {HTMLElement} buttonElement - 클릭한 버튼 요소
         */
        function selectAnswer(selectedOption, buttonElement) {
            const allOptionButtons = optionsContainer.querySelectorAll('.option-btn');
            allOptionButtons.forEach(btn => btn.classList.add('disabled')); // 모든 버튼 비활성화

            const q = state.currentQuestions[state.currentQuestionIndex];
            const isCorrect = selectedOption === q.answer;

            // 결과 저장
            if (!state.isReQuiz) { // 정규 퀴즈일 때만 결과 기록
                state.allResults.push({
                    level: state.currentLevel,
                    question: q.question,
                    questionType: q.questionType,
                    correctAnswer: q.answer,
                    userAnswer: selectedOption,
                    isCorrect: isCorrect,
                    ...q.originalWord
                });
            }

            if (isCorrect) {
                buttonElement.classList.add('correct');
                showConfetti(); // 정답 시 폭죽 효과
            } else {
                buttonElement.classList.add('incorrect');
                // 정답 버튼 찾아서 표시
                allOptionButtons.forEach(btn => {
                    if (btn.textContent === q.answer) {
                        btn.classList.add('correct');
                    }
                });
                
                // 다시 풀기 모드가 아닐 때만 오답 목록에 추가
                if (!state.isReQuiz) {
                    state.mistakes.push(q);
                }
            }

            // 1.5초 후 다음 문제로
            setTimeout(nextQuestion, 1500);
        }

        /**
         * 다음 문제로 이동
         */
        function nextQuestion() {
            state.currentQuestionIndex++;
            
            // 현재 레벨의 문제가 남았을 때
            if (state.currentQuestionIndex < state.currentQuestions.length) {
                loadQuestion();
            } else { // 현재 레벨 완료
                // 틀린 문제가 있고, '다시 풀기' 모드가 아니었다면
                if (state.mistakes.length > 0 && !state.isReQuiz) {
                    startReQuiz();
                } else { // 레벨 완전 종료 (다시 풀기까지 끝났거나, 틀린게 없었거나)
                    finishLevel();
                }
            }
        }
        
        /**
         * '다시 풀기' 퀴즈 시작
         */
        function startReQuiz() {
            state.isReQuiz = true;
            // 오답 문제들의 선지를 섞어서 새 문제 목록 생성
            state.currentQuestions = state.mistakes.map(q => ({
                ...q,
                options: shuffleArray([...q.options]) // 선지 순서 변경
            }));
            state.currentQuestionIndex = 0;
            state.mistakes = []; // 오답 목록 초기화 (다시 풀기에서 또 틀린건 기록X)
            
            loadQuestion();
        }

        /**
         * 레벨 완료 처리
         */
        function finishLevel() {
            state.completedLevels[state.currentLevel] = true; // 현재 레벨 완료 처리
            state.isReQuiz = false; // 리퀴즈 모드 해제
            
            showScreen('screen-level-complete'); // 완료 메시지 표시
            
            // 3초 후 레벨 선택 화면으로
            setTimeout(() => {
                showScreen('screen-level-select');
                updateLevelSelectScreen(); // 레벨 선택 화면 갱신
            }, 3000);
        }

        // --- 6. 결과 처리 로직 ---

        /**
         * 레벨 선택 화면 갱신 (완료된 레벨 비활성화, 결과 버튼 표시)
         */
        function updateLevelSelectScreen() {
            let allComplete = true;
            levelButtons.forEach(btn => {
                const level = btn.dataset.level;
                if (state.completedLevels[level]) {
                    btn.classList.add('completed');
                    btn.disabled = true;
                } else {
                    allComplete = false; // 아직 안 푼 레벨이 있음
                }
            });

            // 모든 레벨을 완료했다면 '최종 결과 보기' 버튼 표시
            if (allComplete) {
                viewResultsBtn.style.display = 'block';
            }
        }
        
        /**
         * 최종 결과 화면 생성
         */
        function showFinalResults() {
            resultsTitle.textContent = `${state.userName}님의 학습 결과 📜`;
            
            generateFeedback(); // 피드백 생성
            generateResultsTable(); // 결과 테이블 생성
            
            showScreen('screen-final-results');
        }

        /**
         * 맞춤형 피드백 생성
         */
        function generateFeedback() {
            const total = state.allResults.length;
            const correctCount = state.allResults.filter(r => r.isCorrect).length;
            const accuracy = (correctCount / total * 100).toFixed(1);
            
            let feedback = `${state.userName}님, 총 ${total}문제 중 ${correctCount}문제를 맞혀 ${accuracy}%의 정답률을 기록했어요! 훌륭해요! 🌸\n`;
            
            // 유형별 분석 (간단하게)
            const mistakes = state.allResults.filter(r => !r.isCorrect);
            const mistakeTypes = {};
            mistakes.forEach(m => {
                mistakeTypes[m.questionType] = (mistakeTypes[m.questionType] || 0) + 1;
            });
            
            // 가장 많이 틀린 유형 찾기
            let maxMistakeType = null;
            let maxMistakeCount = 0;
            for (const type in mistakeTypes) {
                if (mistakeTypes[type] > maxMistakeCount) {
                    maxMistakeCount = mistakeTypes[type];
                    maxMistakeType = type;
                }
            }

            if (correctCount === total) {
                feedback += "모든 문제를 완벽하게 맞혔네요! 정말 대단해요! 🐱🍙";
            } else if (maxMistakeType) {
                let typeDesc = "";
                if (maxMistakeType.includes('kanji')) typeDesc = "한자 관련 문제";
                else if (maxMistakeType.includes('hiragana')) typeDesc = "히라가나 관련 문제";
                else typeDesc = "단어 뜻 맞추기 문제";
                
                feedback += `특히 '${typeDesc}'를 조금 어려워하신 것 같아요. 이 부분을 집중적으로 복습하면 실력이 훨씬 더 늘 거예요! 포기하지 마세요! 💪`;
            } else {
                feedback += "전반적으로 모든 유형을 잘 학습했어요. 조금만 더 복습하면 완벽해질 거예요! 💖";
            }
            
            feedbackText.textContent = feedback;
        }

        /**
         * 결과 테이블 HTML 생성
         */
        function generateResultsTable() {
            resultsTableBody.innerHTML = ""; // 테이블 초기화
            
            state.allResults.forEach(r => {
                const row = document.createElement('tr');
                row.classList.add(r.isCorrect ? 'bg-green-50' : 'bg-red-50');
                
                let levelText = '';
                if (r.level === 'easy') levelText = '하';
                else if (r.level === 'medium') levelText = '중';
                else levelText = '상';
                
                row.innerHTML = `
                    <td class="px-4 py-2 text-sm text-gray-900">${levelText}</td>
                    <td class="px-4 py-2 text-sm text-gray-900 font-medium">${r.question}</td>
                    <td class="px-4 py-2 text-sm text-gray-900">${r.correctAnswer}</td>
                    <td class="px-4 py-2 text-sm ${r.isCorrect ? 'text-gray-900' : 'text-red-600 font-bold'}">${r.userAnswer}</td>
                    <td class="px-4 py-2 text-sm font-bold ${r.isCorrect ? 'text-green-600' : 'text-red-600'}">
                        ${r.isCorrect ? 'O' : 'X'}
                    </td>
                `;
                resultsTableBody.appendChild(row);
            });
        }
        
        /**
         * 결과 화면을 이미지로 저장
         */
        function saveResultsAsImage() {
            const btn = saveImageBtn;
            btn.textContent = "이미지 생성 중... ⏳";
            btn.disabled = true;

            // html2canvas를 사용하여 결과 영역 캡처
            html2canvas(resultsContent, {
                scale: 2, // 해상도 2배로
                useCORS: true 
            }).then(canvas => {
                // 캔버스를 이미지 URL로 변환
                const imageURL = canvas.toDataURL('image/png');
                
                // 다운로드용 임시 링크 생성
                const downloadLink = document.createElement('a');
                downloadLink.href = imageURL;
                downloadLink.download = `${state.userName}_일본어퀴즈결과.png`;
                
                // 링크 클릭하여 다운로드
                document.body.appendChild(downloadLink);
                downloadLink.click();
                document.body.removeChild(downloadLink);
                
                btn.textContent = "결과 이미지로 저장 💾";
                btn.disabled = false;
            }).catch(err => {
                console.error("이미지 저장 실패:", err);
                btn.textContent = "저장 실패 😢 다시 시도";
                btn.disabled = false;
            });
        }

        // --- 7. 폭죽 효과 ---
        
        function showConfetti() {
            confettiContainer.innerHTML = ''; // 이전 폭죽 제거
            const colors = ['#f472b6', '#ec4899', '#8b5cf6', '#3b82f6', '#10b981', '#f59e0b'];
            
            for (let i = 0; i < 50; i++) { // 50개 조각 생성
                const confetti = document.createElement('div');
                confetti.classList.add('confetti');
                
                const randColor = colors[Math.floor(Math.random() * colors.length)];
                confetti.style.backgroundColor = randColor;
                
                // 랜덤한 X, Y, 회전값 설정
                const x = (Math.random() - 0.5) * 400 + 'px'; // -200px ~ +200px
                const y = (Math.random() - 0.5) * 400 + 'px'; // -200px ~ +200px
                
                confetti.style.setProperty('--x', x);
                confetti.style.setProperty('--y', y);
                
                confettiContainer.appendChild(confetti);
            }
        }

        // --- 8. 이벤트 리스너 ---

        // 퀴즈 시작 버튼
        startBtn.onclick = () => {
            const name = nameInput.value.trim();
            if (name === "") {
                alert("이름을 입력해주세요!");
                return;
            }
            state.userName = name;
            welcomeMessage.textContent = `${state.userName}님, 환영합니다!`;
            showScreen('screen-level-select');
        };
        
        // 이름 입력창 엔터
        nameInput.onkeyup = (e) => {
            if (e.key === 'Enter') startBtn.click();
        };
        
        // 레벨 선택 버튼
        levelButtons.forEach(button => {
            button.onclick = () => {
                state.currentLevel = button.dataset.level;
                state.currentQuestions = generateQuestions(state.currentLevel);
                state.currentQuestionIndex = 0;
                state.mistakes = []; // 새 레벨 시작 시 오답 초기화
                state.isReQuiz = false; // 리퀴즈 모드 초기화
                
                loadQuestion();
                showScreen('screen-quiz');
            };
        });
        
        // 최종 결과 보기 버튼
        viewResultsBtn.onclick = showFinalResults;
        
        // 결과 이미지 저장 버튼
        saveImageBtn.onclick = saveResultsAsImage;

        // 처음으로 돌아가기 버튼
        restartBtn.onclick = () => {
            // 모든 상태 초기화
            state = {
                userName: "",
                currentScreen: "screen-start",
                currentLevel: "",
                currentQuestions: [],
                currentQuestionIndex: 0,
                mistakes: [],
                allResults: [],
                isReQuiz: false,
                completedLevels: { easy: false, medium: false, hard: false }
            };
            
            // UI 초기화
            nameInput.value = "";
            viewResultsBtn.style.display = 'none';
            levelButtons.forEach(btn => {
                btn.classList.remove('completed');
                btn.disabled = false;
            });
            
            showScreen('screen-start');
        };
        
        // 초기 화면 설정 (필요없음, HTML에서 active로 제어)
        // showScreen('screen-start');

    </script>
</body>
</html>
