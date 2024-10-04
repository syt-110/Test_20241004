# Test_20241004

<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>遊戲開始</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            height: 100vh;
            background-color: #f0f0f0;
        }
        .game-start-container {
            text-align: center;
            background-color: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
            width: 300px;
        }
        button {
            padding: 10px 20px;
            background-color: #007bff;
            color: white;
            border: none;
            border-radius: 4px;
            cursor: pointer;
        }
        button:hover {
            background-color: #0056b3;
        }
    </style>
</head>
<body>

<div class="game-start-container">
    <h2 id="welcomeMessage">歡迎來到遊戲，<span id="characterName"></span>！</h2>
    <p>性別: <span id="gender"></span></p>
    <p>髮型: <span id="hairstyle"></span></p>
    <p>服裝: <span id="outfit"></span></p>
    <button onclick="startGame()">開始遊戲</button>
</div>

<script>
    // 從 localStorage 取得角色資料
    const characterName = localStorage.getItem('characterName');
    const gender = localStorage.getItem('gender');
    const hairstyle = localStorage.getItem('hairstyle');
    const outfit = localStorage.getItem('outfit');

    // 顯示角色資訊
    document.getElementById('characterName').textContent = characterName;
    document.getElementById('gender').textContent = gender === 'male' ? '男' : '女';
    document.getElementById('hairstyle').textContent = hairstyle === 'short' ? '短髮' : hairstyle === 'long' ? '長髮' : '捲髮';
    document.getElementById('outfit').textContent = outfit === 'armor' ? '盔甲' : outfit === 'robe' ? '長袍' : '便服';

    function startGame() {
        // 開始遊戲的邏輯可以在這裡添加，例如跳轉到遊戲的主頁面
        alert('遊戲即將開始！');
        // 這裡可以進行跳轉到遊戲主頁的操作，例如：
        // window.location.href = 'main_game.html';
    }
</script>

</body>
</html>


<!DOCTYPE html>
<html lang="zh-TW">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>星空觀測系統</title>
    <style>
        body { margin: 0; }
        canvas { display: block; }
        #info { position: absolute; top: 10px; left: 10px; background: rgba(255, 255, 255, 0.7); padding: 10px; border-radius: 5px; }
    </style>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/axios/1.3.1/axios.min.js"></script>
</head>
<body>
    <div id="info">點擊星星以獲取資訊</div>
    <script>
        let scene, camera, renderer;
        let stars = [];
        const starData = {}; // 用於儲存星星的資料

        // 初始化場景、相機和渲染器
        function init() {
            scene = new THREE.Scene();
            camera = new THREE.PerspectiveCamera(75, window.innerWidth / window.innerHeight, 0.1, 1000);
            renderer = new THREE.WebGLRenderer();
            renderer.setSize(window.innerWidth, window.innerHeight);
            document.body.appendChild(renderer.domElement);

            // 設置相機位置
            camera.position.set(0, 0, 5);

            // 加入星星
            createStars();

            // 加入滑鼠點擊事件
            document.addEventListener('click', onDocumentMouseClick, false);

            animate();
        }

        // 創建星星
        function createStars() {
            const geometry = new THREE.SphereGeometry(0.05, 24, 24);
            const material = new THREE.MeshBasicMaterial({ color: 0xffffff });

            // 從 NASA API 獲取星星資料
            fetchStarData().then(data => {
                data.forEach(star => {
                    const starMesh = new THREE.Mesh(geometry, material);
                    starMesh.position.set(star.x, star.y, star.z);
                    starMesh.name = star.name; // 設定星星的名稱
                    scene.add(starMesh);
                    stars.push(starMesh);
                    starData[star.name] = star; // 儲存星星資訊
                });
            });
        }

        // 獲取星星資料
        async function fetchStarData() {
            // 替換為您的星星資料來源
            const response = await axios.get('https://example.com/your_star_data'); 
            return response.data; // 假設回傳的資料格式為 [{name: '星星名', x: 位置X, y: 位置Y, z: 位置Z}, ...]
        }

        // 處理滑鼠點擊事件
        function onDocumentMouseClick(event) {
            event.preventDefault();

            const mouse = new THREE.Vector2(
                (event.clientX / window.innerWidth) * 2 - 1,
                - (event.clientY / window.innerHeight) * 2 + 1
            );

            const raycaster = new THREE.Raycaster();
            raycaster.setFromCamera(mouse, camera);

            const intersects = raycaster.intersectObjects(stars);

            if (intersects.length > 0) {
                const star = intersects[0].object;
                const starInfo = starData[star.name];
                displayStarInfo(starInfo);
            } else {
                // 如果點擊空白區域，則清空資訊
                document.getElementById('info').innerHTML = "點擊星星以獲取資訊";
            }
        }

        // 顯示星星資訊
        function displayStarInfo(star) {
            const infoDiv = document.getElementById('info');
            infoDiv.innerHTML = `<strong>${star.name}</strong><br>位置: (${star.x.toFixed(2)}, ${star.y.toFixed(2)}, ${star.z.toFixed(2)})`;
        }

        // 動畫循環
        function animate() {
            requestAnimationFrame(animate);
            renderer.render(scene, camera);
        }

        // 當視窗大小改變時，更新相機和渲染器大小
        window.addEventListener('resize', () => {
            camera.aspect = window.innerWidth / window.innerHeight;
            camera.updateProjectionMatrix();
            renderer.setSize(window.innerWidth, window.innerHeight);
        });

        // 啟動應用
        init();
    </script>
</body>
</html>
