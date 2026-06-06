r3f를 활용한 도둑고양이의 야간 탈출 3d 게임 만들기


--index.html--

<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>도둑 고양이의 야간 탈출 3D</title>
    <link rel="stylesheet" href="./style.css">
</head>
<body>
    <canvas id="game-canvas"></canvas>

    <div id="score-container">
        <span class="score-icon">🐟</span>
        <span id="score-text">0 / 3</span>
    </div>

    <div id="clear-ui" class="hidden">
        <div class="ui-content">
            <h1>STAGE CLEAR</h1>
            <p>도둑 고양이가 연구실 탈출에 성공했습니다! 🐾</p>
            <button id="restart-btn">다시 하기</button>
        </div>
    </div>

    <div id="gameover-ui" class="hidden">
        <div class="ui-content fail-border">
            <h1 class="fail-text">MISSION FAILED</h1>
            <p>감시 로봇에게 발각되었습니다! 🤖</p>
            <button id="retry-btn">다시 시도</button>
        </div>
    </div>

    <script type="module" src="/main.js"></script>
</body>
</html>


--style.css--

/* 화면에 여백을 없애고 캔버스를 꽉 차게 만듭니다 */
* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

body {
    overflow: hidden;
    background-color: #111; /* 어두운 밤 연구실 분위기 배경 */
}

#game-canvas {
    width: 100vw;
    height: 100vh;
    display: block;
}

/* UI 화면 중앙 정렬 및 배경 흐리게 */
#clear-ui {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background: rgba(0, 0, 0, 0.7);
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 10;
    font-family: 'Arial', sans-serif;
    color: white;
}

.ui-content {
    text-align: center;
    background: #222;
    padding: 40px 60px;
    border-radius: 15px;
    border: 2px solid #00ffaa;
    box-shadow: 0 0 20px rgba(0, 255, 170, 0.3);
}

.ui-content h1 {
    font-size: 3rem;
    color: #00ffaa;
    margin-bottom: 10px;
    letter-spacing: 2px;
}

.ui-content p {
    font-size: 1.2rem;
    margin-bottom: 25px;
    color: #ccc;
}

/* 다시 하기 버튼 */
#restart-btn {
    padding: 12px 30px;
    font-size: 1rem;
    background: #00ffaa;
    color: #111;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-weight: bold;
    transition: 0.2s;
}

#restart-btn:hover {
    background: #00cc88;
    transform: scale(1.05);
}

/* 숨김 처리를 위한 클래스 */
.hidden {
    display: none !important;
}

#gameover-ui {
    position: fixed;
    top: 0;
    left: 0;
    width: 100vw;
    height: 100vh;
    background: rgba(0, 0, 0, 0.85); /* 조금 더 어둡게 */
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 10;
    font-family: 'Arial', sans-serif;
    color: white;
}

/* 실패 전용 테두리 색상 (붉은색 브레싱 효과) */
.fail-border {
    border: 2px solid #ff3333 !important;
    box-shadow: 0 0 25px rgba(255, 51, 51, 0.4) !important;
}

.fail-text {
    color: #ff3333 !important;
    font-size: 3rem;
    margin-bottom: 10px;
    letter-spacing: 2px;
}

/* 다시 시도 버튼 */
#retry-btn {
    padding: 12px 30px;
    font-size: 1rem;
    background: #ff3333;
    color: white;
    border: none;
    border-radius: 5px;
    cursor: pointer;
    font-weight: bold;
    transition: 0.2s;
}

#retry-btn:hover {
    background: #cc0000;
    transform: scale(1.05);
}

#score-container {
    position: fixed;
    top: 20px;
    left: 20px;
    background: rgba(0, 0, 0, 0.6); /* 반투명 블랙 */
    padding: 10px 20px;
    border-radius: 20px;
    border: 1px solid #ffaa00; /* 아이템 색상과 맞춘 노란색 테두리 */
    display: flex;
    align-items: center;
    gap: 8px;
    font-family: 'Arial', sans-serif;
    color: white;
    font-size: 1.2rem;
    font-weight: bold;
    z-index: 5; /* 게임 화면보다 위에 오도록 설정 */
    pointer-events: none; /* UI 때문에 마우스 클릭이 막히는 것 방지 */
    box-shadow: 0 4px 10px rgba(0, 0, 0, 0.3);
}

.score-icon {
    font-size: 1.4rem;
    animation: floating 2s ease-in-out infinite; /* 아이템처럼 둥둥 뜨는 효과 */
}

#score-text {
    color: #ffaa00; /* 점수 숫자를 돋보이게 */
    letter-spacing: 1px;
}

/* 아이콘 둥둥 뜨는 애니메이션 효과 */
@keyframes floating {
    0% { transform: translateY(0px); }
    50% { transform: translateY(-3px); }
    100% { transform: translateY(0px); }
}


--main.js--

import * as THREE from 'three';

// 1. 무대 (Scene) 생성
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x1a1a1a); // 어두운 회색 배경

// 2. 쿼터뷰 카메라 (OrthographicCamera) 설정
const aspect = window.innerWidth / window.innerHeight;
const d = 10; // 시야 범위 배율
const camera = new THREE.OrthographicCamera(-d * aspect, d * aspect, d, -d, 1, 1000);

// 비스듬히 내려다보는 고정 쿼터뷰 각도 세팅
camera.position.set(20, 20, 20); 
camera.lookAt(0, 0, 0);

// 3. 렌더러 (Renderer) 설정
const canvas = document.getElementById('game-canvas');
const renderer = new THREE.WebGLRenderer({ canvas, antialias: true });
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));

// 4. 조명 (Lighting) 추가
const ambientLight = new THREE.AmbientLight(0xffffff, 0.6); // 전체를 은은하게 밝혀주는 조명
scene.add(ambientLight);

const directionalLight = new THREE.DirectionalLight(0xffffff, 0.8); // 그림자를 만들어줄 직사광선
directionalLight.position.set(10, 20, 10);
scene.add(directionalLight);

// 5. 임시 오브젝트 배치 (바닥과 플레이어 대용 큐브)
// 바닥
const floorGeo = new THREE.PlaneGeometry(15, 15);
const floorMat = new THREE.MeshStandardMaterial({ color: 0x333333 });
const floor = new THREE.Mesh(floorGeo, floorMat);
floor.rotation.x = -Math.PI / 2; // 바닥이 되도록 90도 눕힘
scene.add(floor);

// 키보드 입력 상태를 저장할 오브젝트
const keys = {
    w: false,
    a: false,
    s: false,
    d: false
};

// 키를 누를 때
window.addEventListener('keydown', (e) => {
    const key = e.key.toLowerCase();
    if (key in keys) keys[key] = true;
});

// 키에서 손을 뗄 때
window.addEventListener('keyup', (e) => {
    const key = e.key.toLowerCase();
    if (key in keys) keys[key] = false;
});

const moveSpeed = 0.1; // 플레이어 이동 속도

function handlePlayerMovement() {
    // 1. 기본 입력 방향 벡터 구하기
    const direction = new THREE.Vector3();

    if (keys.w) direction.z -= 1; // 앞으로
    if (keys.s) direction.z += 1; // 뒤로
    if (keys.a) direction.x -= 1; // 왼쪽으로
    if (keys.d) direction.x += 1; // 오른쪽으로

    if (direction.lengthSq() > 0) {
        direction.normalize();

        // 2. 쿼터뷰 카메라 각도에 맞춰 벡터 회전
        direction.applyAxisAngle(new THREE.Vector3(0, 1, 0), -Math.PI * 2);

        // 3. 플레이어의 '예상 위치' 미리 연산
        const nextPosition = new THREE.Vector3()
            .copy(player.position)
            .addScaledVector(direction, moveSpeed);

        const playerRadius = 0.45; 
        let isColliding = false;

        // [기존 검사] 🧱 맵에 있는 3개의 벽과 충돌하는지 체크
        for (let i = 0; i < walls.length; i++) {
            const wallMesh = walls[i];
            const wallBox = new THREE.Box3().setFromObject(wallMesh);
            const playerBox = new THREE.Box3(
                new THREE.Vector3(nextPosition.x - playerRadius, 0, nextPosition.z - playerRadius),
                new THREE.Vector3(nextPosition.x + playerRadius, 3, nextPosition.z + playerRadius)
            );

            if (wallBox.intersectsBox(playerBox)) {
                isColliding = true;
                break; 
            }
        }

        // 🚧 [핵심 추가] 맵 가장자리(필드 외곽) 경계선 체크 🚧
        // 바닥 크기(15x15)의 끝단인 -7.5 ~ 7.5를 넘어가려고 하면 강제로 충돌(true) 처리합니다.
        // 플레이어 몸집(0.5)이 맵 바깥으로 살짝이라도 삐져나가지 않도록 7.0을 한계선으로 잡습니다.
        const boundaryLimit = 7.0;
        
        if (
            nextPosition.x < -boundaryLimit || nextPosition.x > boundaryLimit ||
            nextPosition.z < -boundaryLimit || nextPosition.z > boundaryLimit
        ) {
            isColliding = true; // 필드를 탈출하려고 하면 이동을 차단!
        }

        // 4. 벽에도 안 부딪히고, 필드 밖도 아닐 때만 최종 이동!
        if (!isColliding) {
            player.position.copy(nextPosition);
        }

        // 5. 플레이어 방향 회전
        const targetPosition = new THREE.Vector3().copy(player.position).add(direction);
        player.lookAt(targetPosition);
    }
}

// 플레이어 (초 파란색 상자)
// 여러 개의 도형을 하나로 묶어줄 고양이 그룹을 생성합니다.
const player = new THREE.Group(); 

// 1. 고양이 몸통 (검은고양이나 치즈고양이 등 원하는 색상으로 세팅 가능!)
// 여기서는 귀여운 검은고양이 턱시도 느낌을 위해 어두운 색상(0x2c2c2c)으로 만들겠습니다.
const bodyGeo = new THREE.BoxGeometry(0.8, 0.6, 1.1); // 살짝 앞뒤로 긴 식빵 모양 몸통
const catMat = new THREE.MeshStandardMaterial({ color: 0x2c2c2c, roughness: 0.5 });
const body = new THREE.Mesh(bodyGeo, catMat);
body.position.y = 0.3; // 몸통 중심점 잡기
player.add(body);

// 2. 고양이 머리 (앞쪽에 배치)
const headGeo = new THREE.BoxGeometry(0.6, 0.5, 0.5);
const head = new THREE.Mesh(headGeo, catMat);
head.position.set(0, 0.65, 0.4); // 몸통보다 위쪽, 그리고 정면(-Z) 방향으로 전진 배치
player.add(head);

// 3. 고양이 귀 세팅 (분홍색 안감이 있는 뾰족한 삼각형 귀)
const earMat = new THREE.MeshStandardMaterial({ color: 0x2c2c2c });
const innerEarMat = new THREE.MeshStandardMaterial({ color: 0xff9999 }); // 귀 안쪽 분홍색

// 왼쪽 귀 조합
const leftEarGroup = new THREE.Group();
const earGeo = new THREE.ConeGeometry(0.12, 0.25, 4); // 뾰족한 원뿔 모양 귀
earGeo.rotateY(Math.PI / 4); // 사각 원뿔 각도 정렬
const leftEar = new THREE.Mesh(earGeo, earMat);
const leftInner = new THREE.Mesh(new THREE.ConeGeometry(0.06, 0.15, 4), innerEarMat);
leftInner.position.z = -0.04; // 분홍색 안감을 살짝 앞으로 배치
leftEarGroup.add(leftEar, leftInner);
leftEarGroup.position.set(-0.18, 0.95, 0.45); // 머리 위 왼쪽
player.add(leftEarGroup);

// 오른쪽 귀 조합
const rightEarGroup = leftEarGroup.clone(); // 왼쪽 귀 복사해서 위치만 이동
rightEarGroup.position.set(0.18, 0.95, 0.45); // 머리 위 오른쪽
player.add(rightEarGroup);

// 4. 고양이 눈 (빛나는 에메랄드색 눈 두 개)
const eyeGeo = new THREE.BoxGeometry(0.08, 0.08, 0.02);
const eyeMat = new THREE.MeshBasicMaterial({ color: 0x00ffaa }); // 야광 고양이 눈

const leftEye = new THREE.Mesh(eyeGeo, eyeMat);
leftEye.position.set(-0.15, 0.65, 0.66); // 머리의 정면 표면에 부착
player.add(leftEye);

const rightEye = leftEye.clone();
rightEye.position.x = 0.15; // 오른쪽 눈 배치
player.add(rightEye);

// 5. 고양이 캐릭터의 최종 초기 위치 및 씬 추가
// 기존에 설정해 둔 안전지대(좌측 하단 구석) 좌표를 그대로 부여합니다.
player.position.set(-6, 0, 6); 

scene.add(player);


// ==========================================
// 🤖 멀티 적(Enemies) 및 순찰 경로 세팅
// ==========================================

const enemies = []; // 맵에 존재하는 모든 적을 담을 배열
const viewDistance = 5; // 시야 거리

// 각 적들의 설정 데이터 (시작위치, 순찰 방식, 속도 등)
const enemyConfigs = [
    {
        // 🤖 1번 적: 외곽 사각형 순찰 로봇 (기존)
        type: 'loop',
        speed: 0.04,
        waypoints: [
            new THREE.Vector3(5, 0, -5),
            new THREE.Vector3(-5, 0, -5),
            new THREE.Vector3(-5, 0, 5),
            new THREE.Vector3(5, 0, 5)
        ]
    },
    {
        // 🤖 2번 적: 중앙 가로 왕복 로봇 (새로 추가!)
        type: 'pingpong', // 끝에 도달하면 역재생하는 방식
        speed: 0.04,       
        waypoints: [
            new THREE.Vector3(-4, 0, 0), // 중앙 왼쪽 끝
            new THREE.Vector3(4, 0, 0)   // 중앙 오른쪽 끝
        ]
    },
    // ⭐ [추가] 3번 적: 중앙 세로(상하) 왕복 로봇 (축소된 크기)
    {
        type: 'pingpong',
        speed: 0.04,       
        waypoints: [
            new THREE.Vector3(0, 0, -4), // 중앙 위쪽 끝 (Z축 이동)
            new THREE.Vector3(0, 0, 4)   // 중앙 아래쪽 끝
        ]
    },
    // ⭐ [추가] 4번 적: 외곽 사각형 교차 순찰 로봇 (정반대 편에서 출발)
    {
        type: 'loop',
        speed: 0.035, // 기존 외곽 로봇(0.04)보다 살짝 느리게 해서 타이밍이 계속 엇갈리도록 유도
        waypoints: [
            new THREE.Vector3(-5, 0, 5),   // 💡 좌측 하단에서 시작! (1번 적의 3번째 목적지)
            new THREE.Vector3(5, 0, 5),    // 우측 하단
            new THREE.Vector3(5, 0, -5),   // 우측 상단
            new THREE.Vector3(-5, 0, -5)   // 좌측 상단
        ]
    }
];

// 설정 데이터를 바탕으로 실제 3D 씬에 적들을 생성합니다.
enemyConfigs.forEach((config, index) => {
    const enemyGroup = new THREE.Group();

    // 1. 적 본체 (빨간 큐브)
    const enemyGeo = new THREE.BoxGeometry(1, 1, 1);
    const enemyMat = new THREE.MeshStandardMaterial({ color: 0xff3333 });
    const enemyMesh = new THREE.Mesh(enemyGeo, enemyMat);
    enemyMesh.position.y = 0.5;
    enemyGroup.add(enemyMesh);

    // 2. ✨ [난이도 조절] 1번 적과 2번 적의 시야 크기를 다르게 분기 처리합니다.
    let currentViewDistance = viewDistance; // 기본값 5
    let coneRadius = 3;                     // 기본 반지름 3

    if (index === 1 || index === 2) { 
        // 🤖 중앙 왕복 로봇만 불빛 크기를 줄입니다.
        currentViewDistance = 3.5; // 시야 거리를 5에서 3.5로 축소 (불빛이 짧아짐)
        coneRadius = 1.5;          // 시야 폭을 3에서 1.5로 축소 (불빛이 좁아짐)
    }

    // 변경된 크기로 원뿔 생성
    const fovGeo = new THREE.ConeGeometry(coneRadius, currentViewDistance, 32);
    const fovMat = new THREE.MeshBasicMaterial({
        color: 0xffcc00,
        transparent: true,
        opacity: 0.3,
        depthWrite: false
    });
    const fovLight = new THREE.Mesh(fovGeo, fovMat);

    // 원뿔 정렬 (밑면이 정면을 향함)
    fovGeo.rotateX(Math.PI / 2);
    fovLight.position.z = -currentViewDistance / 2; // 줄어든 거리에 맞춰 꼭짓점 정렬
    fovLight.position.y = 0.1;
    enemyGroup.add(fovLight);

    // 시작 위치 세팅 및 씬 추가
    enemyGroup.position.copy(config.waypoints[0]);
    scene.add(enemyGroup);

    // 실시간 데이터 관리를 위해 배열에 저장
    enemies.push({
        group: enemyGroup,
        mat: fovMat,
        waypoints: config.waypoints,
        currentIdx: 0,
        speed: config.speed,
        type: config.type,
        forward: true,
        // ✨ 개별 적마다 고유한 시야 거리를 인지하도록 프로퍼티 추가 (판정 동기화용)
        viewDistance: currentViewDistance 
    });
});     

// --- 게임 상태 및 탈출구 세팅 ---
let isGameCleared = false; 
let isGameOver = false;
let exitMesh = null;       
const exitPosition = new THREE.Vector3(6, 0.01, -6);

function spawnExit() {
    const exitGeo = new THREE.RingGeometry(0.1, 0.8, 32);
    const exitMat = new THREE.MeshBasicMaterial({ 
        color: 0x00ffaa, 
        side: THREE.DoubleSide,
        transparent: true,
        opacity: 0.6
    });
    exitMesh = new THREE.Mesh(exitGeo, exitMat);
    exitMesh.rotation.x = -Math.PI / 2;
    exitMesh.position.copy(exitPosition);
    scene.add(exitMesh);
    console.log("🏁 탈출구가 활성화되었습니다!");
}


// --- 벽(장애물) 세팅 ---
// --- 🧱 벽(장애물) 세팅: 미로형 레벨 디자인 ---
const walls = [];
const wallMat = new THREE.MeshStandardMaterial({ 
    color: 0x555555,     // 조금 더 묵직한 어두운 회색
    roughness: 0.7,
    metalness: 0.2
});

// 벽들의 정보를 담은 배열 (가로, 높이, 세로, X좌표, Z좌표)
// 플레이어가 다니는 통로와 적이 순찰하는 동선을 고려해 배치했습니다.
const wallConfigs = [
    { w: 4, h: 3, d: 1, x: 2, z: -2 },   // 중앙 상단 가로벽 (로봇 순찰선 차단)
    { w: 1, h: 3, d: 4, x: -3, z: 0 },   // 좌측 세로벽 (플레이어 숨바꼭질 구역)
    { w: 2, h: 3, d: 2, x: 1, z: 3 }     // 하단 우측 거대 기둥
];

wallConfigs.forEach((config) => {
    const wallGeo = new THREE.BoxGeometry(config.w, config.h, config.d);
    const wallMesh = new THREE.Mesh(wallGeo, wallMat);
    
    // Y축은 높이(h)의 절반만큼 올려야 바닥(0) 위에 정확히 안착합니다.
    wallMesh.position.set(config.x, config.h / 2, config.z);
    
    scene.add(wallMesh);
    walls.push(wallMesh); // ⭐ 이 배열에 들어가야 레이캐스터가 시야를 차단합니다!
});


// --- 🐟 아이템(수집품) 세팅: 벽 중복 생성 방지 랜덤 스폰 시스템 ---
let score = 0;             
const items = [];          
const totalItems = 3;      

function spawnItemsRandomly() {
    const itemGeo = new THREE.TorusGeometry(0.2, 0.08, 8, 24);
    const itemMat = new THREE.MeshStandardMaterial({ 
        color: 0xffaa00, 
        roughness: 0.3,
        metalness: 0.8 
    });

    for (let i = 0; i < totalItems; i++) {
        const itemMesh = new THREE.Mesh(itemGeo, itemMat);
        itemMesh.rotation.x = Math.PI / 4;

        let randomX, randomZ;
        let isValidPosition = false;

        // 🔄 안전한 위치를 찾을 때까지 무한 반복해서 임의의 좌표를 뽑습니다.
        while (!isValidPosition) {
            randomX = (Math.random() - 0.5) * 12; // -6 ~ 6 사이
            randomZ = (Math.random() - 0.5) * 12; // -6 ~ 6 사이

            // 가상의 아이템 위치와 크기를 가진 충돌 구체(Sphere) 또는 포인트 생성
            const itemPos = new THREE.Vector3(randomX, 0.3, randomZ);
            
            // 임시로 합격(true) 판정을 주고, 하나라도 조건에 걸리면 탈락(false)시킵니다.
            let collisionWithWall = false;

            // 🚧 [핵심 추가] 맵에 있는 모든 벽과 겹치는지 전수 조사
            for (let j = 0; j < walls.length; j++) {
                const wallMesh = walls[j];
                // 벽의 가상 3D AABB 상자 구하기
                const wallBox = new THREE.Box3().setFromObject(wallMesh);
                
                // 아이템이 벽 안에 파묻히지 않도록, 벽 두께보다 살짝 더 여유 공간을 둡니다.
                // 아이템 좌표를 감싸는 아주 작은 충돌 상자를 만듭니다.
                const itemBox = new THREE.Box3(
                    new THREE.Vector3(randomX - 0.4, 0, randomZ - 0.4),
                    new THREE.Vector3(randomX + 0.4, 3, randomZ + 0.4)
                );

                // 만약 아이템의 가상 상자가 벽 상자와 겹친다면?
                if (wallBox.intersectsBox(itemBox)) {
                    collisionWithWall = true; // 벽 충돌 감지!
                    break; // 더 볼 것도 없이 이번 좌표는 버리고 새로 뽑아야 합니다.
                }
            }

            // 검사 1: 시작점의 플레이어(0, 0, 0)와 너무 가깝지 않은가?
            const distToStart = Math.sqrt(randomX * randomX + randomZ * randomZ);
            
            // 검사 2: 활성화될 탈출구(6, 0, -6)와 너무 가깝지 않은가?
            const distToExit = Math.sqrt(Math.pow(randomX - 6, 2) + Math.pow(randomZ - (-6), 2));

            // 플레이어와도 멀고, 탈출구와도 멀고, 결정적으로 '벽과도 안 겹칠 때'만 최종 합격!
            if (distToStart > 2 && distToExit > 2 && !collisionWithWall) {
                isValidPosition = true;
                itemMesh.position.copy(itemPos); // 안전이 확보된 좌표를 아이템에 부여
            }
        }
        
        scene.add(itemMesh);
        items.push(itemMesh);
    }
}

// 아이템 랜덤 생성 함수 실행!
spawnItemsRandomly();


// 6. 브라우저 창 크기가 바뀔 때 대응
window.addEventListener('resize', () => {
    const width = window.innerWidth;
    const height = window.innerHeight;
    const newAspect = width / height;

    camera.left = -d * newAspect;
    camera.right = d * newAspect;
    camera.top = d;
    camera.bottom = -d;
    camera.updateProjectionMatrix();

    renderer.setSize(width, height);
});

// 7. 애니메이션 루프
let clock = new THREE.Clock(); 

function animate() {
    if (isGameCleared) return;
    if (isGameOver) return;

    requestAnimationFrame(animate);

    // 플레이어 이동 및 카메라 추적
    handlePlayerMovement();
    const targetCameraPos = new THREE.Vector3(player.position.x + 20, player.position.y + 20, player.position.z + 20);
    camera.position.lerp(targetCameraPos, 0.05);

    // --- 🤖 모든 적 순찰 AI 및 플레이어 감지 로직 [배열 기반 확장] ---
    enemies.forEach((enemy) => {
        const group = enemy.group;
        const waypoints = enemy.waypoints;
        const targetPoint = waypoints[enemy.currentIdx];
        const distanceToTarget = group.position.distanceTo(targetPoint);

        // 1. 순찰 이동 및 회전 로직
        if (distanceToTarget > 0.1) {
            const moveDir = new THREE.Vector3().subVectors(targetPoint, group.position).normalize();
            group.position.addScaledVector(moveDir, enemy.speed);

            const targetRotationMatrix = new THREE.Matrix4().lookAt(
                group.position,
                new THREE.Vector3().copy(group.position).add(moveDir),
                new THREE.Vector3(0, 1, 0)
            );
            const targetQuaternion = new THREE.Quaternion().setFromRotationMatrix(targetRotationMatrix);
            group.quaternion.slerp(targetQuaternion, 0.1);
        } else {
            // 차례가 바뀌는 방식 연산 (웨이포인트 전환)
            if (enemy.type === 'loop') {
                // 사각형 순찰: 0 -> 1 -> 2 -> 3 -> 0 ...
                enemy.currentIdx = (enemy.currentIdx + 1) % waypoints.length;
            } else if (enemy.type === 'pingpong') {
                // 중앙 왕복: 0 -> 1 -> 0 -> 1 ...
                if (enemy.forward) {
                    enemy.currentIdx++;
                    if (enemy.currentIdx >= waypoints.length) {
                        enemy.currentIdx = waypoints.length - 2;
                        enemy.forward = false;
                    }
                } else {
                    enemy.currentIdx--;
                    if (enemy.currentIdx < 0) {
                        enemy.currentIdx = 1;
                        enemy.forward = true;
                    }
                }
            }
        }

        // 2. 플레이어 감지 로직
        const distance = group.position.distanceTo(player.position);
        let isPlayerDetectedByThisEnemy = false;

        if (distance <= enemy.viewDistance) {
            const enemyForward = new THREE.Vector3(0, 0, -1).applyQuaternion(group.quaternion).normalize();
            const dirToPlayer = new THREE.Vector3().subVectors(player.position, group.position).normalize();
            const angle = enemyForward.angleTo(dirToPlayer);
            const fovAngle = THREE.MathUtils.degToRad(30);

            if (angle <= fovAngle) {
                const raycaster = new THREE.Raycaster();
                const rayOrigin = new THREE.Vector3().copy(group.position);
                rayOrigin.y = 0.5;
                raycaster.set(rayOrigin, dirToPlayer);

                const targets = [];
                if (typeof walls !== 'undefined' && walls.length > 0) targets.push(...walls);
                targets.push(player);

                const intersects = raycaster.intersectObjects(targets, true);
                if (intersects.length > 0) {
                    if (intersects[0].object === player) {
                        enemy.mat.color.setHex(0xff0000); // 걸린 적의 불빛 빨갛게 변경
                        isPlayerDetectedByThisEnemy = true;
                        
                        // 게임오버 트리거
                        isGameOver = true;
                        document.getElementById('gameover-ui').classList.remove('hidden');
                        console.log("🚨 MISSION FAILED: 감시 로봇에게 걸렸습니다!");
                    }
                }
            }
        }

        // 이 적에게 안 들켰다면 노란색 유지
        if (!isPlayerDetectedByThisEnemy) {
            enemy.mat.color.setHex(0xffcc00);
        }
    });

    // --- 🐟 아이템 애니메이션 및 수집 체크 ---
    for (let i = items.length - 1; i >= 0; i--) {
        const item = items[i];
        item.rotation.z += 0.02;
        item.position.y = 0.3 + Math.sin(clock.getElapsedTime() * 3 + i) * 0.05;
        
        const distToPlayer = player.position.distanceTo(item.position);
        
        if (distToPlayer < 0.6) {
            scene.remove(item);
            items.splice(i, 1);
            score++;

            document.getElementById('score-text').innerText = `${score} / ${totalItems}`;
            console.log(`🐟 아이템 획득! 현재 점수: ${score} / ${totalItems}`);
            
            if (score === totalItems) {
                spawnExit();
            }
        }
    }

    // --- 🏁 탈출구 도달 체크 로직 ---
    if (exitMesh) {
        exitMesh.material.opacity = 0.4 + Math.sin(clock.getElapsedTime() * 5) * 0.2;
        const distToExit = player.position.distanceTo(exitPosition);
        
        if (distToExit < 0.8) {
            isGameCleared = true; 
            document.getElementById('clear-ui').classList.remove('hidden');
        }
    }
    
    renderer.render(scene, camera);
}

animate();

// 다시 하기 버튼 (클리어 화면)
document.getElementById('restart-btn').addEventListener('click', () => {
    window.location.reload();
});

// 다시 시도 버튼 [추가] (게임오버 화면)
document.getElementById('retry-btn').addEventListener('click', () => {
    window.location.reload();
});
