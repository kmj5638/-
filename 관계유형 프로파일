<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>관계유형 프로파일 차트</title>
    <!-- Tailwind CSS (스타일링용) -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Chart.js (차트 생성 라이브러리) -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js@3.7.1/dist/chart.min.js"></script>
    <style>
        /* 폰트 및 기본 스타일 설정 */
        body { font-family: 'Inter', sans-serif; background-color: #f7f9fb; }
        .chart-container { max-width: 600px; margin: 0 auto; padding: 20px; background-color: white; border-radius: 1rem; box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1); }
        .student-list-item:nth-child(even) { background-color: #f9fafb; }
    </style>
</head>
<body class="p-4 sm:p-8">
    <div class="max-w-4xl mx-auto bg-white p-6 sm:p-8 rounded-xl shadow-2xl">
        <h1 class="text-3xl font-bold text-gray-800 mb-6 text-center">관계유형 프로파일</h1>
        
        <!-- 로딩 인디케이터 -->
        <div id="loadingMessage" class="text-center p-4 text-indigo-600 font-semibold hidden">
            데이터베이스를 연결하고 이전 데이터를 불러오는 중...
        </div>

        <!-- 입력 폼 -->
        <div id="inputForm" class="bg-gray-50 p-4 rounded-lg shadow-inner mb-6">
            <h2 class="text-xl font-semibold text-gray-700 mb-4">프로파일 데이터 입력</h2>
            
            <div class="grid grid-cols-2 gap-4 mb-4">
                <div class="flex flex-col">
                    <label for="raterName" class="text-sm font-medium text-gray-700 mb-1">검사자 (점수를 매긴 사람)</label>
                    <input type="text" id="raterName" placeholder="예: 언니 / 엄마 / 본인" class="w-full p-2 border border-gray-300 rounded-lg focus:ring-indigo-500 focus:border-indigo-500">
                </div>
                <div class="flex flex-col">
                    <label for="targetName" class="text-sm font-medium text-gray-700 mb-1">상대방 (평가 대상)</label>
                    <input type="text" id="targetName" placeholder="예: 왕감자 / 아들" class="w-full p-2 border border-gray-300 rounded-lg focus:ring-indigo-500 focus:border-indigo-500">
                </div>
            </div>

            <div class="grid grid-cols-2 md:grid-cols-4 gap-4 mb-4">
                <div class="flex flex-col">
                    <label for="type1" class="text-xs font-medium text-blue-700 mb-1">실행 (A)</label>
                    <input type="number" id="type1" min="0" max="50" placeholder="0~50" class="p-2 border border-gray-300 rounded-lg focus:ring-blue-500 focus:border-blue-500">
                </div>
                <div class="flex flex-col">
                    <label for="type2" class="text-xs font-medium text-green-700 mb-1">지식 (B)</label>
                    <input type="number" id="type2" min="0" max="50" placeholder="0~50" class="p-2 border border-gray-300 rounded-lg focus:ring-green-500 focus:border-green-500">
                </div>
                <div class="flex flex-col">
                    <label for="type3" class="text-xs font-medium text-purple-700 mb-1">관심 (C)</label>
                    <input type="number" id="type3" min="0" max="50" placeholder="0~50" class="p-2 border border-gray-300 rounded-lg focus:ring-purple-500 focus:border-purple-500">
                </div>
                <div class="flex flex-col">
                    <label for="type4" class="text-xs font-medium text-red-700 mb-1">존중 (D)</label>
                    <input type="number" id="type4" min="0" max="50" placeholder="0~50" class="p-2 border border-gray-300 rounded-lg focus:ring-red-500 focus:border-red-500">
                </div>
            </div>

            <button onclick="addStudent()" class="w-full bg-indigo-600 text-white font-semibold py-2 rounded-lg hover:bg-indigo-700 transition duration-200 shadow-md">
                데이터 추가 및 차트에 반영
            </button>
        </div>

        <!-- 학생 목록 및 차트 영역 -->
        <div class="grid md:grid-cols-2 gap-6">
            <!-- 차트 영역 -->
            <div class="p-4 border border-gray-200 rounded-lg bg-gray-50 flex items-center justify-center">
                <canvas id="myRadarChart"></canvas>
            </div>
            
            <!-- 추가된 학생 목록 -->
            <div>
                <h2 class="text-xl font-semibold text-gray-700 mb-3">현재 비교 데이터 목록</h2>
                <ul id="studentList" class="border border-gray-300 rounded-lg divide-y divide-gray-200 max-h-80 overflow-y-auto">
                    <li id="emptyListMessage" class="p-4 text-center text-gray-500">
                        데이터를 추가해주세요.
                    </li>
                </ul>
            </div>
        </div>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, deleteDoc, onSnapshot, doc, setLogLevel } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Firebase 설정 및 초기화
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;

        let db, auth, userId;
        let myChart;
        const ctx = document.getElementById('myRadarChart').getContext('2d');
        const maxScore = 50; 
        let studentsData = [];
        
        // 데이터 경로 (사용자 개인 데이터 저장)
        const getCollectionPath = () => `/artifacts/${appId}/users/${userId}/profiles`;

        // 학생별 색상 팔레트 (다양한 대비색 사용)
        const colorPalette = [
            { bg: 'rgba(59, 130, 246, 0.1)', border: 'rgba(59, 130, 246, 1)' }, // Blue
            { bg: 'rgba(16, 185, 129, 0.1)', border: 'rgba(16, 185, 129, 1)' }, // Emerald
            { bg: 'rgba(139, 92, 246, 0.1)', border: 'rgba(139, 92, 246, 1)' }, // Violet
            { bg: 'rgba(239, 68, 68, 0.1)', border: 'rgba(239, 68, 68, 1)' }, // Red
            { bg: 'rgba(245, 158, 11, 0.1)', border: 'rgba(245, 158, 11, 1)' }, // Amber
            { bg: 'rgba(148, 163, 184, 0.1)', border: 'rgba(148, 163, 184, 1)' }  // Slate
        ];

        const labels = ['실행 (A)', '지식 (B)', '관심 (C)', '존중 (D)'];

        const config = {
            type: 'radar',
            data: { labels: labels, datasets: [] },
            options: {
                responsive: true,
                maintainAspectRatio: true,
                scales: {
                    r: {
                        angleLines: { display: true },
                        suggestedMin: 0,
                        suggestedMax: maxScore,
                        pointLabels: { font: { size: 14, weight: 'bold' } },
                        ticks: { stepSize: 10, backdropColor: 'rgba(255, 255, 255, 0.7)' },
                        grid: { color: 'rgba(0, 0, 0, 0.1)' }
                    }
                },
                plugins: {
                    legend: { display: true, position: 'top', labels: { padding: 20 } },
                    title: { display: true, text: '관계유형 프로파일', font: { size: 18, weight: 'bold' }, color: '#1f2937' }
                }
            }
        };

        // UI 상태 관리 (로딩 메시지)
        function setLoading(isLoading) {
            document.getElementById('loadingMessage').classList.toggle('hidden', !isLoading);
            document.getElementById('inputForm').classList.toggle('opacity-50', isLoading);
        }

        // Firebase 및 차트 초기화
        async function initApp() {
            setLoading(true);
            try {
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);
                // setLogLevel('debug'); // 디버그 로그 필요 시 활성화

                if (initialAuthToken) {
                    await signInWithCustomToken(auth, initialAuthToken);
                } else {
                    await signInAnonymously(auth);
                }

                onAuthStateChanged(auth, (user) => {
                    if (user) {
                        userId = user.uid;
                        console.log("Firebase Auth Success. User ID:", userId);
                        setupRealtimeListener();
                    } else {
                        // 인증 실패 시 익명 사용자 ID 사용 (데이터 유지는 안됨)
                        userId = crypto.randomUUID();
                        console.error("Firebase Auth Failed. Using anonymous ID.");
                        setLoading(false);
                    }
                });

            } catch (error) {
                console.error("Firebase Initialization Error:", error);
                // 실패 시 임시 ID 사용
                userId = crypto.randomUUID();
                setLoading(false);
            }
            initChart();
        }
        
        // Firestore 실시간 리스너 설정
        function setupRealtimeListener() {
            if (!db || !userId) return;

            const colRef = collection(db, getCollectionPath());
            
            // onSnapshot을 사용하여 실시간으로 데이터 변경 감지
            onSnapshot(colRef, (snapshot) => {
                studentsData = [];
                snapshot.forEach((doc) => {
                    // Firestore 문서 ID를 데이터 객체에 저장하여 삭제 시 사용
                    studentsData.push({ ...doc.data(), id: doc.id }); 
                });
                
                // 데이터 정렬 (이름 기준)
                studentsData.sort((a, b) => a.name.localeCompare(b.name));

                updateChart();
                updateStudentList();
                setLoading(false);
            }, (error) => {
                console.error("Firestore Snapshot Error:", error);
                setLoading(false);
            });
        }

        // 차트 초기화 함수
        function initChart() {
            if (myChart) {
                myChart.destroy();
            }
            myChart = new Chart(ctx, config);
        }
        
        // 학생 데이터 추가 함수 (Firestore 저장)
        window.addStudent = async function() {
            const raterName = document.getElementById('raterName').value.trim();
            const targetName = document.getElementById('targetName').value.trim();

            const score1 = parseInt(document.getElementById('type1').value);
            const score2 = parseInt(document.getElementById('type2').value);
            const score3 = parseInt(document.getElementById('type3').value);
            const score4 = parseInt(document.getElementById('type4').value);
            
            if (!raterName || !targetName) {
                alert('검사자 이름과 상대방 이름 모두 입력해 주세요.');
                return;
            }

            const scores = [score1, score2, score3, score4];
            let isValid = true;
            scores.forEach(score => {
                if (isNaN(score) || score < 0 || score > maxScore) {
                    isValid = false;
                }
            });

            if (!isValid) {
                alert(`모든 점수는 0점에서 ${maxScore}점 사이의 숫자로 정확히 입력해 주세요.`);
                return;
            }
            
            const displayName = `${raterName} → ${targetName}`;
            
            setLoading(true);
            try {
                const docRef = await addDoc(collection(db, getCollectionPath()), {
                    name: displayName,
                    scores: scores,
                    createdAt: new Date() // 생성 시간 기록
                });
                console.log("Document written with ID: ", docRef.id);

                // 입력 필드 초기화 (점수만 초기화)
                document.getElementById('type1').value = '';
                document.getElementById('type2').value = '';
                document.getElementById('type3').value = '';
                document.getElementById('type4').value = '';
            } catch (e) {
                console.error("Error adding document: ", e);
                setLoading(false);
            }
        }

        // 차트 업데이트 함수 (Firestore 데이터 기반으로 차트 재생성)
        function updateChart() {
            const datasets = studentsData.map((student, index) => {
                const color = colorPalette[index % colorPalette.length];
                return {
                    label: student.name,
                    data: student.scores,
                    backgroundColor: color.bg, 
                    borderColor: color.border, 
                    pointBackgroundColor: color.border,
                    pointBorderColor: '#fff',
                    pointHoverBackgroundColor: '#fff',
                    pointHoverBorderColor: color.border,
                    borderWidth: 2
                };
            });
            
            myChart.data.datasets = datasets;
            myChart.update();
        }
        
        // 학생 목록 업데이트 함수
        function updateStudentList() {
            const list = document.getElementById('studentList');
            list.innerHTML = '';
            
            if (studentsData.length === 0) {
                 list.innerHTML = '<li id="emptyListMessage" class="p-4 text-center text-gray-500">데이터를 추가해주세요.</li>';
                 return;
            }

            studentsData.forEach((student, index) => {
                const listItem = document.createElement('li');
                listItem.className = 'student-list-item p-3 flex justify-between items-center text-gray-800';
                
                const nameDisplay = document.createElement('span');
                const colorMarker = document.createElement('span');
                colorMarker.className = 'inline-block w-3 h-3 rounded-full mr-2 flex-shrink-0';
                colorMarker.style.backgroundColor = colorPalette[index % colorPalette.length].border;
                nameDisplay.appendChild(colorMarker);
                // student.name은 Firestore에서 가져온 displayName ('검사자 → 상대방')
                nameDisplay.appendChild(document.createTextNode(student.name)); 
                
                const deleteButton = document.createElement('button');
                deleteButton.className = 'text-red-500 hover:text-red-700 font-medium text-sm transition duration-150 p-1 rounded ml-2 flex-shrink-0';
                deleteButton.innerHTML = '삭제';
                // Firestore ID를 사용하여 삭제 함수 호출
                deleteButton.onclick = () => removeStudent(student.id); 
                
                listItem.appendChild(nameDisplay);
                listItem.appendChild(deleteButton);
                list.appendChild(listItem);
            });
        }
        
        // 학생 데이터 삭제 함수 (Firestore에서 삭제)
        window.removeStudent = async function(docId) {
             setLoading(true);
            try {
                await deleteDoc(doc(db, getCollectionPath(), docId));
                console.log("Document successfully deleted:", docId);
            } catch (e) {
                console.error("Error removing document: ", e);
                setLoading(false);
            }
        }

        // 페이지 로드 시 앱 초기화
        window.onload = initApp;
    </script>
</body>
</html>
