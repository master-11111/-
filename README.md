<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>프로젝트 시노드 - MMORPG 웹 게임</title>
    <!-- Three.js 라이브러리 연동 (3D 캐릭터 렌더링용) -->
    <script src="https://cdnjs.cloudflare.com/ajax/libs/three.js/r128/three.min.js"></script>
    <style>
        * { box-sizing: border-box; margin: 0; padding: 0; }
        
        /* 커스텀 스크롤바 */
        ::-webkit-scrollbar { width: 6px; height: 6px; }
        ::-webkit-scrollbar-track { background: #020617; border-radius: 3px; }
        ::-webkit-scrollbar-thumb { background: #38bdf8; border-radius: 3px; box-shadow: 0 0 6px #38bdf8; }

        body {
            background: radial-gradient(circle at center, #0f172a 0%, #030712 100%);
            color: #f1f5f9;
            font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            padding: 20px;
            overflow-x: hidden;
        }

        .shake { animation: shakeAnim 0.15s ease-in-out; }
        @keyframes shakeAnim {
            0% { transform: translate(0, 0); }
            20% { transform: translate(-10px, 8px); }
            40% { transform: translate(10px, -8px); }
            60% { transform: translate(-6px, 4px); }
            80% { transform: translate(6px, -4px); }
            100% { transform: translate(0, 0); }
        }

        .container {
            width: 100%;
            max-width: 1020px;
            background: rgba(30, 41, 59, 0.75);
            backdrop-filter: blur(12px);
            border-radius: 20px;
            padding: 28px;
            box-shadow: 0 0 40px rgba(56, 189, 248, 0.25), inset 0 0 15px rgba(56, 189, 248, 0.1);
            border: 1px solid rgba(56, 189, 248, 0.4);
            position: relative;
        }
        h1 { 
            text-align: center; 
            color: #38bdf8; 
            margin-bottom: 24px; 
            font-size: 2.1rem; 
            letter-spacing: 2px;
            text-shadow: 0 0 12px rgba(56, 189, 248, 0.7); 
        }
        
        .auth-card {
            background: rgba(15, 23, 42, 0.9);
            padding: 35px;
            border-radius: 16px;
            border: 1px solid rgba(56, 189, 248, 0.3);
            max-width: 450px;
            margin: 0 auto;
            box-shadow: 0 10px 30px rgba(0,0,0,0.5);
        }
        .auth-card h2 { color: #38bdf8; margin-bottom: 20px; text-align: center; font-size: 1.5rem; text-shadow: 0 0 8px rgba(56,189,248,0.4); }
        .form-group { margin-bottom: 18px; }
        .form-group label { display: block; font-size: 0.85rem; color: #94a3b8; margin-bottom: 6px; font-weight: 600; }
        .form-group input, .form-group select {
            width: 100%;
            padding: 12px 14px;
            background: rgba(30, 41, 59, 0.9);
            border: 1px solid #475569;
            border-radius: 8px;
            color: white;
            font-size: 0.95rem;
            transition: all 0.3s;
        }
        .form-group input:focus {
            border-color: #38bdf8;
            outline: none;
            box-shadow: 0 0 10px rgba(56, 189, 248, 0.4);
        }
        .row-group { display: flex; gap: 8px; }
        
        .btn-full {
            width: 100%;
            background: linear-gradient(135deg, #0284c7 0%, #0369a1 100%);
            color: white;
            border: none;
            padding: 13px;
            border-radius: 8px;
            font-weight: bold;
            cursor: pointer;
            margin-top: 6px;
            font-size: 1rem;
            box-shadow: 0 4px 12px rgba(2, 132, 199, 0.3);
            transition: all 0.2s;
        }
        .btn-full:hover { background: linear-gradient(135deg, #0369a1 0%, #075985 100%); transform: translateY(-1px); box-shadow: 0 6px 15px rgba(2, 132, 199, 0.4); }
        .btn-sub { background: #334155; box-shadow: none; margin-top: 8px; }
        .btn-sub:hover { background: #475569; }
        .link-text { text-align: center; font-size: 0.85rem; color: #38bdf8; margin-top: 14px; cursor: pointer; transition: color 0.2s; }
        .link-text:hover { color: #7dd3fc; text-decoration: underline; }

        #game-screen { display: none; }
        .status-board {
            background: rgba(15, 23, 42, 0.85);
            padding: 18px;
            border-radius: 14px;
            margin-bottom: 16px;
            border: 1px solid rgba(56, 189, 248, 0.2);
            display: grid;
            grid-template-columns: repeat(auto-fit, minmax(160px, 1fr));
            gap: 12px;
            box-shadow: inset 0 2px 8px rgba(0,0,0,0.4);
        }
        .stat-item { font-size: 0.9rem; color: #cbd5e1; }
        .stat-item span { color: #f59e0b; font-weight: bold; text-shadow: 0 0 5px rgba(245,158,11,0.3); }
        .diamond-text { color: #38bdf8 !important; text-shadow: 0 0 8px rgba(56, 189, 248, 0.6) !important; }

        .gauge-container { margin-bottom: 16px; background: rgba(15, 23, 42, 0.85); padding: 14px; border-radius: 12px; border: 1px solid rgba(56, 189, 248, 0.2); }
        .gauge-wrapper { display: flex; align-items: center; justify-content: space-between; margin-bottom: 6px; font-size: 0.85rem; font-weight: 600; }
        .gauge-bar-bg { width: 100%; height: 12px; background: #020617; border-radius: 6px; overflow: hidden; position: relative; border: 1px solid #1e293b; box-shadow: inset 0 2px 4px rgba(0,0,0,0.6); }
        .gauge-bar-fill { height: 100%; transition: width 0.3s cubic-bezier(0.4, 0, 0.2, 1); border-radius: 6px; }
        #hp-fill { background: linear-gradient(90deg, #dc2626, #ef4444); box-shadow: 0 0 10px rgba(239, 68, 68, 0.5); }
        #mp-fill { background: linear-gradient(90deg, #2563eb, #3b82f6); box-shadow: 0 0 10px rgba(59, 130, 246, 0.5); width: 100%; }

        .main-display-grid {
            display: grid;
            grid-template-columns: 240px 1fr;
            gap: 16px;
            margin-bottom: 16px;
        }

        .character-3d-box {
            position: relative;
            background: linear-gradient(180deg, #020617 0%, #090d16 100%);
            border: 1px solid rgba(56, 189, 248, 0.4);
            border-radius: 12px;
            height: 220px;
            overflow: hidden;
            display: flex;
            flex-direction: column;
            align-items: center;
            justify-content: center;
            box-shadow: 0 8px 20px rgba(0,0,0,0.4);
        }
        .character-3d-box canvas { width: 100%; height: 100%; cursor: grab; }
        .character-3d-label {
            position: absolute;
            bottom: 8px;
            font-size: 0.75rem;
            color: #38bdf8;
            background: rgba(2, 6, 23, 0.8);
            padding: 3px 10px;
            border-radius: 6px;
            border: 1px solid rgba(56, 189, 248, 0.3);
            pointer-events: none;
            letter-spacing: 0.5px;
        }

        .battle-display {
            position: relative;
            background: linear-gradient(135deg, #020617 0%, #0f172a 50%, #1e1b4b 100%);
            border: 1px solid rgba(56, 189, 248, 0.4);
            border-radius: 12px;
            height: 220px;
            overflow: hidden;
            box-shadow: 0 8px 20px rgba(0,0,0,0.4);
        }
        .world-3d-scene {
            position: absolute;
            width: 100%;
            height: 100%;
            display: flex;
            justify-content: space-around;
            align-items: center;
            padding: 20px;
        }
        .npc-card {
            background: rgba(15, 23, 42, 0.9);
            border: 1px solid rgba(245, 158, 11, 0.6);
            border-radius: 10px;
            padding: 10px 14px;
            text-align: center;
            cursor: pointer;
            transition: all 0.25s cubic-bezier(0.175, 0.885, 0.32, 1.275);
            box-shadow: 0 0 12px rgba(245, 158, 11, 0.2);
            z-index: 2;
        }
        .npc-card:hover { transform: scale(1.1); background: rgba(30, 41, 59, 0.95); box-shadow: 0 0 18px rgba(245, 158, 11, 0.5); }
        .npc-card span { display: block; font-size: 0.75rem; color: #f59e0b; margin-top: 4px; font-weight: 600; }

        .walking-citizen {
            position: absolute;
            bottom: 12px;
            font-size: 0.75rem;
            color: #38bdf8;
            background: rgba(2, 6, 23, 0.8);
            border: 1px solid rgba(56, 189, 248, 0.3);
            padding: 3px 8px;
            border-radius: 6px;
            animation: walkAnim 9s linear infinite;
            z-index: 2;
        }
        @keyframes walkAnim {
            0% { transform: translateX(-60px); opacity: 0; }
            10% { opacity: 1; }
            90% { opacity: 1; }
            100% { transform: translateX(520px); opacity: 0; }
        }

        .log-box {
            position: absolute;
            top: 0; left: 0; width: 100%; height: 100%;
            color: #38bdf8;
            font-family: 'Courier New', Courier, monospace;
            padding: 14px;
            overflow-y: auto;
            font-size: 0.85rem;
            line-height: 1.5;
            z-index: 5;
            background: rgba(2, 6, 23, 0.9);
        }

        .chat-section {
            background: rgba(15, 23, 42, 0.85);
            border: 1px solid rgba(56, 189, 248, 0.2);
            border-radius: 12px;
            padding: 12px;
            margin-bottom: 16px;
            box-shadow: inset 0 2px 6px rgba(0,0,0,0.4);
        }
        .chat-logs {
            height: 75px;
            overflow-y: auto;
            font-size: 0.8rem;
            font-family: 'Courier New', Courier, monospace;
            color: #cbd5e1;
            margin-bottom: 8px;
            line-height: 1.4;
        }
        .chat-input-row { display: flex; gap: 8px; }
        .chat-input-row input {
            flex: 1;
            padding: 8px 12px;
            background: rgba(30, 41, 59, 0.9);
            border: 1px solid #475569;
            border-radius: 6px;
            color: white;
            font-size: 0.85rem;
        }
        .chat-input-row input:focus { border-color: #38bdf8; outline: none; }

        .damage-pop {
            position: absolute;
            font-weight: 900;
            font-size: 1.8rem;
            color: #ef4444;
            text-shadow: 0 0 12px #ff0000, 2px 2px 0px #000;
            pointer-events: none;
            z-index: 10;
            animation: popUp 0.6s cubic-bezier(0.25, 1, 0.5, 1) forwards;
        }
        @keyframes popUp {
            0% { opacity: 1; transform: translateY(0) scale(0.7); }
            50% { transform: translateY(-35px) scale(1.35); }
            100% { opacity: 0; transform: translateY(-55px) scale(1); }
        }

        .action-grid {
            display: grid;
            grid-template-columns: repeat(4, 1fr);
            gap: 10px;
        }
        button {
            background: linear-gradient(135deg, #0284c7 0%, #0369a1 100%);
            color: white;
            border: none;
            padding: 12px 10px;
            border-radius: 10px;
            font-size: 0.85rem;
            font-weight: bold;
            cursor: pointer;
            transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
            box-shadow: 0 4px 10px rgba(0,0,0,0.3);
        }
        button:hover { transform: translateY(-2px); filter: brightness(1.15); box-shadow: 0 6px 15px rgba(0,0,0,0.4); }
        .btn-trade { background: linear-gradient(135deg, #8b5cf6, #7c3aed); }
        .btn-pay { background: linear-gradient(135deg, #d97706, #b45309); }
        .btn-macro { background: linear-gradient(135deg, #10b981, #059669); }
        .btn-hunt { background: linear-gradient(135deg, #e11d48, #be123c); }
        .btn-boss { background: linear-gradient(135deg, #dc2626, #b91c1c); }
        .btn-gacha { background: linear-gradient(135deg, #ec4899, #db2777); }
        .btn-quest { background: linear-gradient(135deg, #f59e0b, #d97706); }
        .btn-job { background: linear-gradient(135deg, #a855f7, #9333ea); }
        .btn-admin { background: linear-gradient(135deg, #ef4444, #dc2626); grid-column: span 1; }
        .btn-skill { background: linear-gradient(135deg, #6366f1, #4f46e5); }
        .btn-equip { background: linear-gradient(135deg, #14b8a6, #0d9488); }
        .btn-setting { background: linear-gradient(135deg, #64748b, #475569); }
        .btn-url { background: linear-gradient(135deg, #0284c7, #0284c7); }
        .btn-download { background: linear-gradient(135deg, #059669, #047857); }

        .modal-overlay {
            display: none;
            position: fixed;
            top: 0; left: 0; width: 100%; height: 100%;
            background: rgba(2, 6, 23, 0.85);
            backdrop-filter: blur(8px);
            justify-content: center;
            align-items: center;
            z-index: 1000;
        }
        .modal-box {
            background: #0f172a;
            border: 2px solid #8b5cf6;
            border-radius: 16px;
            padding: 28px;
            width: 90%;
            max-width: 520px;
            max-height: 85vh;
            overflow-y: auto;
            box-shadow: 0 0 35px rgba(139, 92, 246, 0.45);
        }
        .item-list-box {
            background: #020617;
            border: 1px solid #334155;
            border-radius: 8px;
            padding: 12px;
            max-height: 200px;
            overflow-y: auto;
            margin-bottom: 14px;
        }
        .item-row {
            padding: 8px 10px;
            border-bottom: 1px solid rgba(30, 41, 59, 0.8);
            font-size: 0.85rem;
            display: flex;
            justify-content: space-between;
            align-items: center;
            transition: background 0.2s;
        }
        .item-row:hover { background: rgba(30, 41, 59, 0.4); }
    </style>
</head>
<body>

<div class="container" id="main-container">
    <h1>🌌 프로젝트 시노드 (MMORPG)</h1>

    <!-- 1. 로그인 화면 -->
    <div id="login-section" class="auth-card">
        <h2>🔐 계정 로그인</h2>
        <div class="form-group"><label>아이디</label><input type="text" id="login-id" placeholder="아이디 입력 (운영자: juidy8150)"></div>
        <div class="form-group"><label>비밀번호</label><input type="password" id="login-pw" placeholder="비밀번호 입력"></div>
        <button class="btn-full" onclick="playAudio('click'); handleLogin()">로그인</button>
        <button class="btn-full btn-sub" onclick="playAudio('click'); showSection('signup-section')">회원가입하기</button>
    </div>

    <!-- 2. 회원가입 화면 -->
    <div id="signup-section" class="auth-card" style="display: none;">
        <h2>📝 회원가입 및 폰 인증</h2>
        <div class="form-group"><label>아이디</label><input type="text" id="signup-id" placeholder="아이디 입력"></div>
        <div class="form-group"><label>비밀번호</label><input type="password" id="signup-pw" placeholder="비밀번호 입력"></div>
        <div class="form-group">
            <label>휴대폰 번호</label>
            <div class="row-group">
                <input type="text" id="signup-phone" placeholder="01012345678">
                <button style="width: 120px;" onclick="playAudio('click'); sendAuthCode()">인증 전송</button>
            </div>
        </div>
        <div class="form-group"><label>인증번호</label><input type="text" id="signup-code" placeholder="인증번호 입력"></div>
        <button class="btn-full" style="background: linear-gradient(135deg, #059669, #047857);" onclick="playAudio('click'); handleSignup()">가입 완료</button>
        <div class="link-text" onclick="playAudio('click'); showSection('login-section')">로그인 화면으로 돌아가기</div>
    </div>

    <!-- 3. 게임 메인 화면 -->
    <div id="game-screen">
        <div class="status-board">
            <div class="stat-item">👤 유저: <span id="p-name">-</span></div>
            <div class="stat-item">🎖️ 클래스: <span id="p-job" style="color:#a855f7;">초보 모험가</span></div>
            <div class="stat-item">📊 레벨: <span id="p-level">1</span> / <span style="color:#38bdf8;">200</span></div>
            <div class="stat-item">⚡ 경험치: <span id="p-exp">0</span> / <span id="p-max-exp">1,000</span></div>
            <div class="stat-item">⚔️ 무기: <span id="p-weapon">+0</span></div>
            <div class="stat-item">🛡️ 방어구: <span id="p-armor">+0</span></div>
            <div class="stat-item">💎 다이아: <span id="p-diamond" class="diamond-text">0</span> Dia</div>
            <div class="stat-item">💰 골드: <span id="p-gold">3,000</span> Gold</div>
        </div>

        <!-- HP / MP 게이지 바 영역 -->
        <div class="gauge-container">
            <div class="gauge-wrapper">
                <span>❤️ 체력 (HP): <span id="p-hp-text">500 / 500</span></span>
            </div>
            <div class="gauge-bar-bg"><div id="hp-fill" class="gauge-bar-fill"></div></div>
            <div class="gauge-wrapper" style="margin-top: 10px;">
                <span>💙 마나 (MP): <span id="p-mp-text">100 / 100</span></span>
            </div>
            <div class="gauge-bar-bg"><div id="mp-fill" class="gauge-bar-fill" style="width: 100%;"></div></div>
        </div>

        <div class="main-display-grid">
            <div class="character-3d-box" id="character-3d-container">
                <div class="character-3d-label">✨ 3D 캐릭터 아바타</div>
            </div>

            <div class="battle-display" id="battle-area">
                <div class="world-3d-scene">
                    <div class="npc-card" onclick="playAudio('click'); openShop('weapon')">⚔️ 장비 상인<br><span>[상점 열기]</span></div>
                    <div class="npc-card" onclick="playAudio('click'); openShop('potion')">🧪 포션 상인<br><span>[상점 열기]</span></div>
                    <div class="npc-card" onclick="playAudio('click'); openShop('pet')">🐾 펫 상인<br><span>[상점 열기]</span></div>
                    <div class="walking-citizen" id="current-location-text">📍 현재 위치: 안전한 메인 마을</div>
                </div>
                <div class="log-box" id="game-log">
                    [시스템] 🛡️ 프로젝트 시노드 웹 클라이언트가 정상적으로 로드되었습니다.<br>
                </div>
            </div>
        </div>

        <!-- 실시간 채팅창 -->
        <div class="chat-section">
            <div class="chat-logs" id="chat-log-box">
                [전체채팅] 시스템: 프로젝트 시노드 세계에 오신 것을 환영합니다!<br>
            </div>
            <div class="chat-input-row">
                <select id="chat-type" style="width: 85px; padding: 6px; background:#1e293b; color:white; border:1px solid #475569; border-radius:6px; font-size:0.8rem;">
                    <option value="전체">전체</option>
                    <option value="파티">파티</option>
                    <option value="귓속말">귓속말</option>
                </select>
                <input type="text" id="chat-input" placeholder="메시지를 입력하세요..." onkeypress="if(event.key==='Enter') sendChat()">
                <button style="width: 65px; padding: 6px; margin: 0;" onclick="playAudio('click'); sendChat()">전송</button>
            </div>
        </div>

        <div class="action-grid" id="action-area">
            <button onclick="playAudio('click'); openHuntModal()" class="btn-hunt">⚔️ 사냥터 선택</button>
            <button onclick="playAudio('click'); openBossModal()" class="btn-boss">👹 던전 보스</button>
            <button onclick="playAudio('click'); openQuestModal()" class="btn-quest">📜 메인퀘스트</button>
            <button onclick="playAudio('click'); openJobModal()" class="btn-job">🎖️ 클래스 전직</button>
            <button onclick="playAudio('click'); openSkillModal()" class="btn-skill">⚡ 스킬창</button>
            <button onclick="playAudio('click'); openEquipModal()" class="btn-equip">🛡️ 아이템 장착</button>
            <button onclick="playAudio('click'); openGachaModal()" class="btn-gacha">🎁 뽑기소</button>
            <button onclick="playAudio('click'); openTradeModal()" class="btn-trade">🤝 1:1 거래</button>
            <button onclick="playAudio('click'); openMacroModal()" class="btn-macro">🤖 매크로</button>
            <button onclick="playAudio('click'); openPayModal()" class="btn-pay">💳 충전소</button>
            <button onclick="playAudio('click'); openBagModal()">🎒 가방</button>
            <button onclick="playAudio('click'); openStorageModal()" style="background:linear-gradient(135deg, #059669, #047857);">📦 창고</button>
            <button onclick="playAudio('click'); openSettingModal()" class="btn-setting">⚙️ 설정</button>
            <button onclick="playAudio('click'); openUrlModal()" class="btn-url">🌐 웹 링크</button>
            <button onclick="playAudio('click'); openDownloadModal()" class="btn-download">📥 다운로드</button>
            <button id="admin-btn" onclick="playAudio('click'); openAdminModal()" class="btn-admin" style="display:none;">👑 관리자</button>
            <button onclick="playAudio('click'); logout()" style="background:linear-gradient(135deg, #475569, #334155);">🚪 로그아웃</button>
        </div>
    </div>
</div>

<!-- 모달 레이아웃들 -->
<div class="modal-overlay" id="download-modal-overlay">
    <div class="modal-box" style="border-color:#059669;">
        <h3 style="color: #059669; margin-bottom: 14px; text-align: center;">📥 게임 다운로드 센터</h3>
        <div class="item-list-box" style="max-height:220px;">
            <div class="item-row"><span>📱 안드로이드 APK v1.0</span><button style="background:#059669; padding:6px 12px; font-size:0.75rem;" onclick="playAudio('click'); downloadApp('Android APK')">다운로드</button></div>
            <div class="item-row"><span>🍎 아이폰 IPA 패키지</span><button style="background:#059669; padding:6px 12px; font-size:0.75rem;" onclick="playAudio('click'); downloadApp('iOS IPA')">다운로드</button></div>
            <div class="item-row"><span>💻 윈도우 PC EXE 설치판</span><button style="background:#d97706; padding:6px 12px; font-size:0.75rem;" onclick="playAudio('click'); downloadApp('Windows PC EXE')">다운로드</button></div>
            <div class="item-row"><span>🍏 맥OS DMG 패키지</span><button style="background:#d97706; padding:6px 12px; font-size:0.75rem;" onclick="playAudio('click'); downloadApp('macOS DMG')">다운로드</button></div>
        </div>
        <button class="btn-full btn-sub" onclick="playAudio('click'); closeDownloadModal()">닫기</button>
    </div>
</div>

<div class="modal-overlay" id="shop-modal-overlay">
    <div class="modal-box" style="border-color:#f59e0b;">
        <h3 id="shop-title" style="color: #f59e0b; margin-bottom: 14px; text-align: center;">🛒 상점 NPC</h3>
        <div class="item-list-box" style="max-height:220px;" id="shop-items-container"></div>
        <button class="btn-full btn-sub" onclick="playAudio('click'); closeShopModal()">닫기</button>
    </div>
</div>

<div class="modal-overlay" id="hunt-modal-overlay">
    <div class="modal-box" style="border-color:#e11d48;">
        <h3 style="color: #e11d48; margin-bottom: 14px; text-align: center;">🗺️ 사냥터 선택 (필드 & 던전)</h3>
        <div style="margin-bottom:8px; font-size:0.85rem; color:#38bdf8; font-weight:bold;">🌿 [필드 사냥터]</div>
        <div class="item-list-box" style="max-height:140px; margin-bottom:12px;">
            <div class="item-row"><span>🛡️ [Lv.1] 시노드 초원 필드 (안전)</span><button style="background:#0284c7; padding:6px 10px; font-size:0.75rem;" onclick="playAudio('click'); hunt('field', 1, 500, 800, 20, false, '시노드 초원 필드')">입장</button></div>
            <div class="item-row"><span>⚔️ [Lv.50] 네온 황무지 필드 (PVP 40%)</span><button style="background:#d97706; padding:6px 10px; font-size:0.75rem;" onclick="playAudio('click'); hunt('field', 50, 3500, 5000, 70, true, '네온 황무지 필드')">입장</button></div>
            <div class="item-row"><span>🔥 [Lv.120] 사이버 불길 필드 (PVP 80%)</span><button style="background:#ef4444; padding:6px 10px; font-size:0.75rem;" onclick="playAudio('click'); hunt('field', 120, 25000, 40000, 150, true, '사이버 불길 필드')">입장</button></div>
        </div>
        <div style="margin-bottom:8px; font-size:0.85rem; color:#f59e0b; font-weight:bold;">🏛️ [던전 사냥터]</div>
        <div class="item-list-box" style="max-height:140px;">
            <div class="item-row"><span>🗝️ [Lv.30] 지하 로봇 던전</span><button style="background:#a855f7; padding:6px 10px; font-size:0.75rem;" onclick="playAudio('click'); hunt('dungeon', 30, 2000, 3500, 50, false, '지하 로봇 던전')">입장</button></div>
            <div class="item-row"><span>⚡ [Lv.90] 나노 코어 연구소</span><button style="background:#a855f7; padding:6px 10px; font-size:0.75rem;" onclick="playAudio('click'); hunt('dungeon', 90, 15000, 22000, 110, false, '나노 코어 연구소')">입장</button></div>
            <div class="item-row"><span>👑 [Lv.150] 오버로드 심층 요새</span><button style="background:#a855f7; padding:6px 10px; font-size:0.75rem;" onclick="playAudio('click'); hunt('dungeon', 150, 60000, 95000, 200, false, '오버로드 심층 요새')">입장</button></div>
        </div>
        <button class="btn-full btn-sub" onclick="playAudio('click'); closeHuntModal()">닫기</button>
    </div>
</div>

<div class="modal-overlay" id="url-modal-overlay">
    <div class="modal-box" style="border-color:#38bdf8;">
        <h3 style="color: #38bdf8; margin-bottom: 14px; text-align: center;">🌐 웹 주소 발행</h3>
        <input type="text" id="generated-url" value="https://project-synod-mmorpg.web.app" readonly style="width:100%; padding:10px; background:#1e293b; color:#38bdf8; font-weight:bold; border:1px solid #475569; border-radius:6px; text-align:center; margin-bottom:12px;">
        <button class="btn-full" style="background:#0284c7; margin-bottom:8px;" onclick="playAudio('click'); copyGameUrl()">주소 복사하기</button>
        <button class="btn-full btn-sub" onclick="playAudio('click'); closeUrlModal()">닫기</button>
    </div>
</div>

<div class="modal-overlay" id="boss-modal-overlay">
    <div class="modal-box" style="border-color:#dc2626;">
        <h3 style="color: #dc2626; margin-bottom: 14px; text-align: center;">👹 보스 레이드 & 최종보스방</h3>
        <div class="item-list-box" style="max-height:280px;">
            <div class="item-row">
                <span>🐺 [Lv.30] 사이버 울프 알파<br><span style="font-size:0.75rem; color:#f59e0b;" id="boss-timer-30">리젠 대기중: 0초</span></span>
                <button style="background:#dc2626; padding:6px 10px; font-size:0.75rem;" onclick="playAudio('click'); challengeBoss('dungeon_boss', 30, 2500, 5000, 3600, '사이버 울프 레이드')">도전</button>
            </div>
            <div class="item-row">
                <span>🤖 [Lv.100] 메카 필드 보스<br><span style="font-size:0.75rem; color:#f59e0b;" id="boss-timer-100">리젠 대기중: 0초</span></span>
                <button style="background:#d97706; padding:6px 10px; font-size:0.75rem;" onclick="playAudio('click'); challengeBoss('field_boss', 100, 35000, 2000, 10800, '메카 필드 보스방')">도전</button>
            </div>
            <div class="item-row" style="background: rgba(124, 58, 237, 0.15); border: 1px solid #7c3aed;">
                <span>👑 [Lv.170] 최종보스방: 네오 테크 오버로드<br><span style="font-size:0.75rem; color:#f59e0b;" id="boss-timer-170">리젠 대기중: 0초</span></span>
                <button style="background:#7c3aed; padding:6px 10px; font-size:0.75rem;" onclick="playAudio('click'); challengeBoss('ultimate_room', 170, 150000, 8000, 14400, '네오 테크 최종보스방')">입장 및 토벌</button>
            </div>
            <div class="item-row" style="background: rgba(239, 68, 68, 0.15); border: 1px solid #ef4444;">
                <span>🐉 [Lv.200] 만렙 최종보스방: 시노드 디바우러<br><span style="font-size:0.75rem; color:#f59e0b;" id="boss-timer-200">리젠 대기중: 0초</span></span>
                <button style="background:#ef4444; padding:6px 10px; font-size:0.75rem;" onclick="playAudio('click'); challengeBoss('final_room', 200, 300000, 15000, 21600, '시노드 디바우러 만렙방')">입장 및 토벌</button>
            </div>
        </div>
        <button class="btn-full btn-sub" onclick="playAudio('click'); closeBossModal()">닫기</button>
    </div>
</div>

<div class="modal-overlay" id="quest-modal-overlay">
    <div class="modal-box" style="border-color:#f59e0b;">
        <h3 style="color: #f59e0b; margin-bottom: 14px; text-align: center;">📜 메인퀘스트</h3>
        <div class="item-list-box" style="max-height:240px;" id="quest-list-container"></div>
        <button class="btn-full btn-sub" onclick="playAudio('click'); closeQuestModal()">닫기</button>
    </div>
</div>

<div class="modal-overlay" id="job-modal-overlay">
    <div class="modal-box" style="border-color:#a855f7;">
        <h3 style="color: #a855f7; margin-bottom: 14px; text-align: center;">🎖️ 클래스 전직</h3>
        <button class="btn-full" style="background:linear-gradient(135deg, #a855f7, #9333ea); margin-bottom:8px;" onclick="playAudio('click'); promoteJob(1)">1차 전직 (Lv.30)</button>
        <button class="btn-full" style="background:linear-gradient(135deg, #8b5cf6, #7c3aed); margin-bottom:8px;" onclick="playAudio('click'); promoteJob(2)">2차 전직 (Lv.70)</button>
        <button class="btn-full" style="background:linear-gradient(135deg, #d97706, #b45309); margin-bottom:8px;" onclick="playAudio('click'); promoteJob(3)">3차 전직 초월 (Lv.130)</button>
        <button class="btn-full" style="background:linear-gradient(135deg, #ef4444, #dc2626); margin-bottom:12px;" onclick="playAudio('click'); promoteJob(4)">4차 신화 각성 (Lv.180)</button>
        <button class="btn-full btn-sub" onclick="playAudio('click'); closeJobModal()">닫기</button>
    </div>
</div>

<div class="modal-overlay" id="skill-modal-overlay">
    <div class="modal-box" style="border-color:#6366f1;">
        <h3 style="color: #6366f1; margin-bottom: 14px; text-align: center;">⚡ 스킬창</h3>
        <div class="item-list-box"><div class="item-row"><span>🔥 플라즈마 슬래시 (Lv.1)</span></div></div>
        <button class="btn-full btn-sub" onclick="playAudio('click'); closeSkillModal()">닫기</button>
    </div>
</div>

<div class="modal-overlay" id="equip-modal-overlay">
    <div class="modal-box" style="border-color:#14b8a6;">
        <h3 style="color: #14b8a6; margin-bottom: 14px; text-align: center;">🛡️ 장비 관리 & 무기 강화</h3>
        <div style="background:#1e293b; padding:14px; border-radius:10px; margin-bottom:14px; font-size:0.85rem; text-align:center; border:1px solid #334155;">
            <p style="margin-bottom:6px;">⚔️ 장착 무기: <span id="equip-weapon-name" style="color:#f59e0b; font-weight:bold;">운영자의 검 (+200)</span></p>
            <p style="margin-bottom:6px;">🛡️ 장착 방어구: <span id="equip-armor-name" style="color:#f59e0b; font-weight:bold;">운영자의 방어구 (+180)</span></p>
            <p style="color:#94a3b8; font-size:0.75rem;">최고 +200강 가능</p>
        </div>
        <button class="btn-full" style="background:linear-gradient(135deg, #f59e0b, #d97706); margin-bottom:8px;" onclick="playAudio('click'); enhanceWeapon()">🔥 무기 강화 시도 (+1)</button>
        <button class="btn-full btn-sub" onclick="playAudio('click'); closeEquipModal()">닫기</button>
    </div>
</div>

<div class="modal-overlay" id="setting-modal-overlay">
    <div class="modal-box" style="border-color:#64748b;">
        <h3 style="color: #64748b; margin-bottom: 14px; text-align: center;">⚙️ 설정</h3>
        <button class="btn-full btn-sub" onclick="playAudio('click'); closeSettingModal()">닫기</button>
    </div>
</div>

<div class="modal-overlay" id="gacha-modal-overlay">
    <div class="modal-box" style="border-color:#ec4899;">
        <h3 style="color: #ec4899; margin-bottom: 15px; text-align: center;">🎁 뽑기소</h3>
        <button class="btn-full" style="background:linear-gradient(135deg, #ec4899, #db2777); padding:10px;" onclick="playAudio('click'); openGacha()">초급 상자 열기 (100 Dia)</button>
        <button class="btn-full btn-sub" style="margin-top:8px;" onclick="playAudio('click'); closeGachaModal()">닫기</button>
    </div>
</div>

<div class="modal-overlay" id="trade-modal-overlay">
    <div class="modal-box" style="border-color:#8b5cf6;">
        <h3 style="color: #8b5cf6; margin-bottom: 12px; text-align: center;">🤝 1:1 거래</h3>
        <input type="text" id="trade-target-id" placeholder="상대방 아이디" style="width:100%; padding:10px; background:#1e293b; color:white; border:1px solid #475569; border-radius:6px; margin-bottom:12px;">
        <button class="btn-full btn-trade" style="margin-bottom:8px;" onclick="playAudio('click'); executeTrade()">거래 승인</button>
        <button class="btn-full btn-sub" onclick="playAudio('click'); closeTradeModal()">닫기</button>
    </div>
</div>

<div class="modal-overlay" id="pay-modal-overlay">
    <div class="modal-box" style="border-color:#d97706;">
        <h3 style="color: #d97706; margin-bottom: 15px; text-align: center;">💳 충전소</h3>
        <button class="btn-full" style="background:linear-gradient(135deg, #d97706, #b45309); padding:10px;" onclick="playAudio('click'); chargeDiamond(1000)">다이아 1,000개 충전</button>
        <button class="btn-full btn-sub" style="margin-top:8px;" onclick="playAudio('click'); closePayModal()">닫기</button>
    </div>
</div>

<!-- 가방 모달 (순간이동 및 귀환 주문서 탑재) -->
<div class="modal-overlay" id="bag-modal-overlay">
    <div class="modal-box" style="border-color:#38bdf8;">
        <h3 style="color: #38bdf8; margin-bottom: 15px; text-align: center;">🎒 가방 & 주문서 센터</h3>
        <div class="item-list-box" style="max-height:220px;">
            <div class="item-row"><span>🌀 순간이동 주문서 (지정 사냥터 즉시 이동)</span><button style="background:#0284c7; padding:4px 10px; font-size:0.75rem;" onclick="playAudio('click'); useTeleportScroll()">사용</button></div>
            <div class="item-row"><span>🏠 귀환 주문서 (안전한 메인 마을 복귀)</span><button style="background:#10b981; padding:4px 10px; font-size:0.75rem;" onclick="playAudio('click'); useReturnScroll()">사용</button></div>
            <div class="item-row"><span>무한 회복 포션 (패시브 활성)</span><span>x무한</span></div>
        </div>
        <button class="btn-full btn-sub" onclick="playAudio('click'); closeBagModal()">닫기</button>
    </div>
</div>

<div class="modal-overlay" id="storage-modal-overlay">
    <div class="modal-box" style="border-color:#059669;">
        <h3 style="color: #059669; margin-bottom: 15px; text-align: center;">📦 창고</h3>
        <div class="item-list-box"><div class="item-row" style="color:#64748b;">보관된 아이템 없음</div></div>
        <button class="btn-full btn-sub" onclick="playAudio('click'); closeStorageModal()">닫기</button>
    </div>
</div>

<div class="modal-overlay" id="admin-modal-overlay">
    <div class="modal-box" style="border-color:#ef4444;">
        <h3 style="color: #ef4444; margin-bottom: 15px; text-align: center;">👑 관리자 패널</h3>
        <button class="btn-full" style="background:linear-gradient(135deg, #ef4444, #dc2626); margin-bottom:8px;" onclick="playAudio('click'); adminGenerateItem()">운영자 장비 장착 (+200 / +180)</button>
        <button class="btn-full btn-sub" onclick="playAudio('click'); closeAdminModal()">닫기</button>
    </div>
</div>

<script>
    const audioCtx = new (window.AudioContext || window.webkitAudioContext)();

    function playAudio(type) {
        if (!audioCtx) return;
        if (audioCtx.state === 'suspended') audioCtx.resume();

        const osc = audioCtx.createOscillator();
        const gainNode = audioCtx.createGain();
        osc.connect(gainNode);
        gainNode.connect(audioCtx.destination);
        const now = audioCtx.currentTime;

        if (type === 'click') {
            osc.type = 'sine';
            osc.frequency.setValueAtTime(600, now);
            osc.frequency.exponentialRampToValueAtTime(800, now + 0.08);
            gainNode.gain.setValueAtTime(0.15, now);
            gainNode.gain.linearRampToValueAtTime(0.01, now + 0.08);
            osc.start(now); osc.stop(now + 0.08);
        } else if (type === 'hunt') {
            osc.type = 'sawtooth';
            osc.frequency.setValueAtTime(180, now);
            osc.frequency.exponentialRampToValueAtTime(60, now + 0.2);
            gainNode.gain.setValueAtTime(0.2, now);
            gainNode.gain.linearRampToValueAtTime(0.01, now + 0.2);
            osc.start(now); osc.stop(now + 0.2);
        } else if (type === 'scroll') {
            osc.type = 'triangle';
            osc.frequency.setValueAtTime(300, now);
            osc.frequency.exponentialRampToValueAtTime(900, now + 0.3);
            gainNode.gain.setValueAtTime(0.2, now);
            gainNode.gain.linearRampToValueAtTime(0.01, now + 0.35);
            osc.start(now); osc.stop(now + 0.35);
        } else if (type === 'boss_win' || type === 'enhance_success') {
            osc.type = 'triangle';
            osc.frequency.setValueAtTime(440, now);
            osc.frequency.exponentialRampToValueAtTime(880, now + 0.3);
            gainNode.gain.setValueAtTime(0.25, now);
            gainNode.gain.linearRampToValueAtTime(0.01, now + 0.35);
            osc.start(now); osc.stop(now + 0.35);
        } else if (type === 'gacha_win') {
            osc.type = 'square';
            osc.frequency.setValueAtTime(523.25, now);
            osc.frequency.setValueAtTime(659.25, now + 0.1);
            osc.frequency.setValueAtTime(783.99, now + 0.2);
            gainNode.gain.setValueAtTime(0.2, now);
            gainNode.gain.linearRampToValueAtTime(0.01, now + 0.35);
            osc.start(now); osc.stop(now + 0.35);
        } else if (type === 'pvp') {
            osc.type = 'sawtooth';
            osc.frequency.setValueAtTime(150, now);
            osc.frequency.exponentialRampToValueAtTime(80, now + 0.15);
            gainNode.gain.setValueAtTime(0.3, now);
            gainNode.gain.linearRampToValueAtTime(0.01, now + 0.15);
            osc.start(now); osc.stop(now + 0.15);
        }
    }

    let currentUser = null;
    let userData = {
        hp: 500, maxHp: 500,
        mp: 100, maxMp: 100,
        diamond: 3000,
        level: 1, exp: 0, maxExp: 1000,
        weaponEnhance: 0, armorEnhance: 0,
        gold: 3000,
        job: "초보 모험가",
        completedQuests: []
    };

    let bossRespawnTimers = { dungeon_boss: 0, field_boss: 0, ultimate_room: 0, final_room: 0 };
    let currentLocation = "안전한 메인 마을";

    const mainQuests = [
        { id: 1, levelReq: 1, title: "📜 [Lv.1] 프롤로그: 시노드 입성", reward: "골드 +500" },
        { id: 2, levelReq: 30, title: "📜 [Lv.30] 1차 전직 자격 증명 시험", reward: "1차 전직 개방" },
        { id: 3, levelReq: 70, title: "📜 [Lv.70] 중급 사냥터 영토 수호", reward: "다이아 +500" },
        { id: 4, levelReq: 100, title: "📜 [Lv.100] 백만렙 돌파 백서 작전", reward: "다이아 +2,000" },
        { id: 5, levelReq: 130, title: "📜 [Lv.130] 3차 전직 승급 심사", reward: "3차 전직 개방" },
        { id: 6, levelReq: 170, title: "📜 [Lv.170] 최종보스방 네오 테크 토벌", reward: "다이아 +5,000" },
        { id: 7, levelReq: 200, title: "📜 [Lv.200] 최종 결전: 만렙 오버로드 토벌", reward: "전설 칭호 [만렙 디바우러]" }
    ];

    let scene3D, camera3D, renderer3D, characterGroup, weaponMesh3D, bodyMesh3D;
    let isDragging3D = false;
    let previousMousePosition3D = { x: 0, y: 0 };

    function init3DCharacterViewer() {
        const container = document.getElementById('character-3d-container');
        const existingCanvas = container.querySelector('canvas');
        if(existingCanvas) existingCanvas.remove();

        scene3D = new THREE.Scene();
        camera3D = new THREE.PerspectiveCamera(45, container.clientWidth / container.clientHeight, 0.1, 1000);
        camera3D.position.set(0, 1.5, 4.5);

        renderer3D = new THREE.WebGLRenderer({ antialias: true, alpha: true });
        renderer3D.setSize(container.clientWidth, container.clientHeight);
        renderer3D.setPixelRatio(window.devicePixelRatio);
        container.appendChild(renderer3D.domElement);

        const ambientLight = new THREE.AmbientLight(0xffffff, 0.7);
        scene3D.add(ambientLight);
        const pointLight = new THREE.PointLight(0x38bdf8, 1.5, 50);
        pointLight.position.set(2, 3, 3);
        scene3D.add(pointLight);

        characterGroup = new THREE.Group();

        const headGeo = new THREE.SphereGeometry(0.35, 32, 32);
        const headMat = new THREE.MeshStandardMaterial({ color: 0xfde047, roughness: 0.3 });
        const head = new THREE.Mesh(headGeo, headMat);
        head.position.y = 1.1;
        characterGroup.add(head);

        const bodyGeo = new THREE.CylinderGeometry(0.3, 0.22, 0.9, 32);
        const bodyMat = new THREE.MeshStandardMaterial({ color: getArmorColor(), metalness: 0.8, roughness: 0.2 });
        bodyMesh3D = new THREE.Mesh(bodyGeo, bodyMat);
        bodyMesh3D.position.y = 0.4;
        characterGroup.add(bodyMesh3D);

        const weaponGeo = new THREE.BoxGeometry(0.1, 1.2, 0.05);
        const weaponMat = new THREE.MeshStandardMaterial({ color: getWeaponColor(), metalness: 0.9, roughness: 0.1 });
        weaponMesh3D = new THREE.Mesh(weaponGeo, weaponMat);
        weaponMesh3D.position.set(0.6, 0.4, 0.2);
        weaponMesh3D.rotation.z = -0.3;
        characterGroup.add(weaponMesh3D);

        scene3D.add(characterGroup);

        const dom = renderer3D.domElement;
        dom.addEventListener('mousedown', (e) => {
            isDragging3D = true;
            previousMousePosition3D = { x: e.clientX, y: e.clientY };
        });
        window.addEventListener('mousemove', (e) => {
            if (!isDragging3D || !characterGroup) return;
            const deltaX = e.clientX - previousMousePosition3D.x;
            characterGroup.rotation.y += deltaX * 0.01;
            previousMousePosition3D = { x: e.clientX, y: e.clientY };
        });
        window.addEventListener('mouseup', () => { isDragging3D = false; });

        function animate3D() {
            requestAnimationFrame(animate3D);
            if (!isDragging3D && characterGroup) {
                characterGroup.rotation.y += 0.005;
            }
            renderer3D.render(scene3D, camera3D);
        }
        animate3D();
    }

    function getWeaponColor() {
        if(userData.weaponEnhance >= 200) return 0xffd700;
        if(userData.weaponEnhance >= 50) return 0xef4444; 
        if(userData.weaponEnhance >= 30) return 0xa855f7; 
        if(userData.weaponEnhance >= 15) return 0xf59e0b; 
        if(userData.weaponEnhance >= 5) return 0x38bdf8;  
        return 0x94a3b8; 
    }

    function getArmorColor() {
        if(userData.armorEnhance >= 180) return 0xff4500;
        if(userData.armorEnhance >= 50) return 0xef4444; 
        return 0x0284c7; 
    }

    function updateEquipmentVisual() {
        if(weaponMesh3D) weaponMesh3D.material.color.setHex(getWeaponColor());
        if(bodyMesh3D) bodyMesh3D.material.color.setHex(getArmorColor());

        document.getElementById('equip-weapon-name').innerText = `운영자의 검 (+${userData.weaponEnhance})`;
        document.getElementById('equip-armor-name').innerText = `운영자의 방어구 (+${userData.armorEnhance})`;
    }

    function useReturnScroll() {
        playAudio('scroll');
        currentLocation = "안전한 메인 마을";
        document.getElementById('current-location-text').innerText = `📍 현재 위치: ${currentLocation}`;
        closeBagModal();
        triggerShake();
        showDamagePop('귀환 완료!', 0, -10, '#10b981');
        appendLog(`[주문서] 🏠 귀환 주문서를 사용하여 안전한 메인 마을로 복귀했습니다.`);
    }

    function useTeleportScroll() {
        playAudio('scroll');
        currentLocation = "네온 황무지 필드 (순간이동)";
        document.getElementById('current-location-text').innerText = `📍 현재 위치: ${currentLocation}`;
        closeBagModal();
        triggerShake();
        showDamagePop('순간이동!', 0, -10, '#38bdf8');
        appendLog(`[주문서] 🌀 순간이동 주문서를 사용하여 사냥터로 즉시 이동했습니다.`);
    }

    function enhanceWeapon() {
        if(userData.weaponEnhance >= 200) {
            alert('이미 최고 강화 단계(+200)에 도달했습니다!');
            return;
        }
        const cost = 1000 + (userData.weaponEnhance * 100);
        if(userData.gold < cost) {
            alert(`골드가 부족합니다! (요구 골드: ${cost.toLocaleString()})`);
            return;
        }

        userData.gold -= cost;
        let successChance = Math.max(0.3, 0.95 - (userData.weaponEnhance * 0.003));
        if(currentUser === 'juidy8150' || currentUser === 'admin') successChance = 1.0;

        if(Math.random() < successChance) {
            userData.weaponEnhance += 1;
            playAudio('enhance_success');
            triggerShake();
            showDamagePop(`+${userData.weaponEnhance} 강화 성공!`, 0, -10, '#f59e0b');
            appendLog(`[무기 강화] 🔥 운영자의 검 강화 성공! [+${userData.weaponEnhance}]`);
        } else {
            playAudio('click');
            showDamagePop('강화 실패', 0, -10, '#ef4444');
            appendLog(`[무기 강화] ❌ 강화 실패.`);
        }

        updateEquipmentVisual();
        updateStatusBoard();
    }

    setInterval(() => {
        if(bossRespawnTimers.dungeon_boss > 0) bossRespawnTimers.dungeon_boss--;
        if(bossRespawnTimers.field_boss > 0) bossRespawnTimers.field_boss--;
        if(bossRespawnTimers.ultimate_room > 0) bossRespawnTimers.ultimate_room--;
        if(bossRespawnTimers.final_room > 0) bossRespawnTimers.final_room--;
        updateBossTimerDisplay();

        if(currentUser === 'juidy8150' || currentUser === 'admin') {
            if(userData.diamond < 9999999) userData.diamond = 9999999;
            if(userData.gold < 9999999) userData.gold = 9999999;
            if(userData.hp < userData.maxHp) {
                userData.hp = userData.maxHp;
                updateStatusBoard();
            }
        }
    }, 1000);

    function updateBossTimerDisplay() {
        const formatTime = (sec) => sec <= 0 ? '<span style="color:#10b981;">[도전 가능]</span>' : `<span style="color:#ef4444;">대기: ${Math.floor(sec/3600)}시간 ${Math.floor((sec%3600)/60)}분</span>`;
        
        if(document.getElementById('boss-timer-30')) document.getElementById('boss-timer-30').innerHTML = formatTime(bossRespawnTimers.dungeon_boss);
        if(document.getElementById('boss-timer-100')) document.getElementById('boss-timer-100').innerHTML = formatTime(bossRespawnTimers.field_boss);
        if(document.getElementById('boss-timer-170')) document.getElementById('boss-timer-170').innerHTML = formatTime(bossRespawnTimers.ultimate_room);
        if(document.getElementById('boss-timer-200')) document.getElementById('boss-timer-200').innerHTML = formatTime(bossRespawnTimers.final_room);
    }

    function showSection(sectionId) {
        document.getElementById('login-section').style.display = 'none';
        document.getElementById('signup-section').style.display = 'none';
        if(sectionId === 'login-section') document.getElementById('login-section').style.display = 'block';
        if(sectionId === 'signup-section') document.getElementById('signup-section').style.display = 'block';
    }

    function sendAuthCode() {
        const phone = document.getElementById('signup-phone').value;
        if(!phone) { alert('휴대폰 번호를 입력해주세요.'); return; }
        alert(`[인증 시스템] ${phone} 번호로 인증번호 [9482]가 전송되었습니다.`);
    }

    function handleSignup() {
        const id = document.getElementById('signup-id').value;
        const pw = document.getElementById('signup-pw').value;
        const code = document.getElementById('signup-code').value;
        if(!id || !pw || !code) { alert('모든 항목을 입력해주세요.'); return; }
        if(code !== '9482') { alert('인증번호가 올바르지 않습니다.'); return; }
        alert('회원가입 완료! 로그인해주세요.');
        showSection('login-section');
    }

    function handleLogin() {
        const id = document.getElementById('login-id').value;
        const pw = document.getElementById('login-pw').value;
        if(!id || !pw) { alert('아이디와 비밀번호를 입력해주세요.'); return; }

        if (id === 'juidy8150' && pw !== 'yaki88299@@') {
            alert('운영자 비밀번호가 일치하지 않습니다.');
            return;
        }

        currentUser = id;
        document.getElementById('login-section').style.display = 'none';
        document.getElementById('signup-section').style.display = 'none';
        document.getElementById('game-screen').style.display = 'block';
        document.getElementById('p-name').innerText = currentUser;

        userData.hp = 500;
        userData.maxHp = 500;
        userData.mp = 100;
        userData.maxMp = 100;

        if(currentUser === 'juidy8150' || currentUser === 'admin') {
            userData.weaponEnhance = 200;
            userData.armorEnhance = 180;
            userData.gold = 9999999;
            userData.diamond = 9999999;
            document.getElementById('admin-btn').style.display = 'block';
            alert(`👑 [운영자 특권] ${currentUser}님 환영합니다! 운영자의 검(+200) & 방어구(+180) 착용 완료.`);
            appendLog(`[운영자 시스템] 👑 최강 운영자 장비 세트가 장착되었습니다.`);
        } else {
            document.getElementById('admin-btn').style.display = 'none';
            alert(`🎉 [프로젝트 시노드] 환영합니다, ${currentUser}님!`);
            appendLog(`[환영 인사] 🌟 ${currentUser}님이 접속하셨습니다.`);
        }

        updateEquipmentVisual();
        updateStatusBoard();
        setTimeout(init3DCharacterViewer, 100);
    }

    function logout() {
        currentUser = null;
        document.getElementById('game-screen').style.display = 'none';
        document.getElementById('login-section').style.display = 'block';
        document.getElementById('login-id').value = '';
        document.getElementById('login-pw').value = '';
    }

    function updateStatusBoard() {
        document.getElementById('p-level').innerText = userData.level;
        document.getElementById('p-job').innerText = userData.job;
        document.getElementById('p-exp').innerText = userData.exp.toLocaleString();
        document.getElementById('p-max-exp').innerText = userData.maxExp.toLocaleString();
        document.getElementById('p-weapon').innerText = `+${userData.weaponEnhance}`;
        document.getElementById('p-armor').innerText = `+${userData.armorEnhance}`;
        document.getElementById('p-diamond').innerText = userData.diamond.toLocaleString();
        document.getElementById('p-gold').innerText = userData.gold.toLocaleString();

        document.getElementById('p-hp-text').innerText = `${userData.hp.toLocaleString()} / ${userData.maxHp.toLocaleString()}`;
        document.getElementById('p-mp-text').innerText = `${userData.mp.toLocaleString()} / ${userData.maxMp.toLocaleString()}`;
        
        document.getElementById('hp-fill').style.width = `${(userData.hp / userData.maxHp) * 100}%`;
        document.getElementById('mp-fill').style.width = `${(userData.mp / userData.maxMp) * 100}%`;
    }

    function appendLog(msg) {
        const logBox = document.getElementById('game-log');
        logBox.innerHTML += `${msg}<br>`;
        logBox.scrollTop = logBox.scrollHeight;
    }

    function sendChat() {
        const input = document.getElementById('chat-input');
        const type = document.getElementById('chat-type').value;
        const text = input.value.trim();
        if(!text) return;
        const chatLogBox = document.getElementById('chat-log-box');
        chatLogBox.innerHTML += `[${type}] ${currentUser}: ${text}<br>`;
        chatLogBox.scrollTop = chatLogBox.scrollHeight;
        input.value = '';
    }

    function triggerShake() {
        const container = document.getElementById('main-container');
        container.classList.add('shake');
        setTimeout(() => container.classList.remove('shake'), 150);
    }

    function showDamagePop(text, xOffset = 0, yOffset = 0, color = '#ef4444') {
        const battleArea = document.getElementById('battle-area');
        const pop = document.createElement('div');
        pop.className = 'damage-pop';
        pop.style.color = color;
        pop.innerText = text;
        pop.style.left = `${50 + xOffset}%`;
        pop.style.top = `${40 + yOffset}%`;
        battleArea.appendChild(pop);
        setTimeout(() => pop.remove(), 600);
    }

    function openDownloadModal() { document.getElementById('download-modal-overlay').style.display = 'flex'; }
    function closeDownloadModal() { document.getElementById('download-modal-overlay').style.display = 'none'; }
    
    function downloadApp(platform) {
        alert(`📥 [다운로드 센터] "${platform}" 클라이언트 패키지 다운로드가 시작되었습니다!`);
        appendLog(`[클라이언트] 📥 ${platform} 설치 패키지 다운로드가 요청되었습니다.`);
        closeDownloadModal();
    }

    function openShop(type) {
        const titleElem = document.getElementById('shop-title');
        const container = document.getElementById('shop-items-container');
        container.innerHTML = '';

        if(type === 'weapon') {
            titleElem.innerText = "⚔️ 장비 상인 NPC 상점";
            container.innerHTML = `
                <div class="item-row"><span>플라즈마 레이피어 (+5) - 5,000 Gold</span><button style="background:#f59e0b; padding:4px 10px; font-size:0.75rem;" onclick="playAudio('click'); buyItem('플라즈마 레이피어', 5000)">구매</button></div>
                <div class="item-row"><span>타이탄 중갑 방어구 (+5) - 8,000 Gold</span><button style="background:#f59e0b; padding:4px 10px; font-size:0.75rem;" onclick="playAudio('click'); buyItem('타이탄 중갑', 8000)">구매</button></div>
            `;
        } else if(type === 'potion') {
            titleElem.innerText = "🧪 포션 상인 NPC 상점";
            container.innerHTML = `
                <div class="item-row"><span>HP 회복 포션 (x10) - 1,000 Gold</span><button style="background:#10b981; padding:4px 10px; font-size:0.75rem;" onclick="playAudio('click'); buyItem('HP 포션 x10', 1000)">구매</button></div>
                <div class="item-row"><span>마나 부스터 (x10) - 1,000 Gold</span><button style="background:#10b981; padding:4px 10px; font-size:0.75rem;" onclick="playAudio('click'); buyItem('MP 포션 x10', 1000)">구매</button></div>
            `;
        } else if(type === 'pet') {
            titleElem.innerText = "🐾 펫 상인 NPC 상점";
            container.innerHTML = `
                <div class="item-row"><span>사이버 가디언 펫 - 500 Dia</span><button style="background:#ec4899; padding:4px 10px; font-size:0.75rem;" onclick="playAudio('click'); buyPet('사이버 가디언', 500)">구매</button></div>
            `;
        }
        document.getElementById('shop-modal-overlay').style.display = 'flex';
    }
    function closeShopModal() { document.getElementById('shop-modal-overlay').style.display = 'none'; }

    function buyItem(name, cost) {
        if(userData.gold < cost) { alert('골드가 부족합니다!'); return; }
        userData.gold -= cost;
        updateStatusBoard();
        appendLog(`[상점] 🛒 [${name}] 구매 완료.`);
        alert(`${name} 구매 완료!`);
    }

    function buyPet(name, cost) {
        if(userData.diamond < cost) { alert('다이아가 부족합니다!'); return; }
        userData.diamond -= cost;
        updateStatusBoard();
        appendLog(`[상점] 🐾 [${name}] 펫 입양 완료.`);
        alert(`${name} 분양 완료!`);
    }

    function openUrlModal() { document.getElementById('url-modal-overlay').style.display = 'flex'; }
    function closeUrlModal() { document.getElementById('url-modal-overlay').style.display = 'none'; }
    function copyGameUrl() {
        const urlInput = document.getElementById('generated-url');
        urlInput.select();
        navigator.clipboard.writeText(urlInput.value);
        alert('주소가 복사되었습니다.');
    }

    function openBossModal() { updateBossTimerDisplay(); document.getElementById('boss-modal-overlay').style.display = 'flex'; }
    function closeBossModal() { document.getElementById('boss-modal-overlay').style.display = 'none'; }

    function challengeBoss(type, reqLevel, rewardExp, rewardCurrency, cooldownSeconds, locationName) {
        if(userData.level < reqLevel) { alert(`레벨 부족 (요구: Lv.${reqLevel})`); return; }
        if(bossRespawnTimers[type] > 0) { alert(`보스방 리젠 대기 중입니다.`); return; }

        currentLocation = locationName;
        document.getElementById('current-location-text').innerText = `📍 현재 위치: ${currentLocation}`;

        playAudio('boss_win');
        bossRespawnTimers[type] = cooldownSeconds;
        closeBossModal();
        triggerShake();
        showDamagePop('BOSS ROOM CLEAR!', 0, -10, '#f59e0b');
        userData.exp += rewardExp;
        if(type === 'dungeon_boss' || type === 'ultimate_room') userData.gold += rewardCurrency;
        else userData.diamond += rewardCurrency;
        
        appendLog(`[보스방 토벌] 🎉 [${locationName}] 클리어! EXP +${rewardExp.toLocaleString()}`);
        checkLevelUp();
        updateStatusBoard();
    }

    function openQuestModal() {
        const container = document.getElementById('quest-list-container');
        container.innerHTML = '';
        mainQuests.forEach(q => {
            const isCompleted = userData.completedQuests.includes(q.id);
            const canComplete = userData.level >= q.levelReq && !isCompleted;
            let statusText = isCompleted ? '<span style="color:#10b981;">완료됨</span>' : (canComplete ? `<button style="background:#f59e0b; padding:4px 10px; font-size:0.75rem;" onclick="playAudio('click'); completeQuest(${q.id})">보상받기</button>` : `<span style="color:#64748b;">요구 Lv.${q.levelReq}</span>`);
            container.innerHTML += `<div class="item-row"><span>${q.title} (${q.reward})</span>${statusText}</div>`;
        });
        document.getElementById('quest-modal-overlay').style.display = 'flex';
    }
    function closeQuestModal() { document.getElementById('quest-modal-overlay').style.display = 'none'; }

    function completeQuest(id) {
        const q = mainQuests.find(item => item.id === id);
        if(!q) return;
        userData.completedQuests.push(id);
        if(id === 3) userData.diamond += 500;
        if(id === 4) userData.diamond += 2000;
        if(id === 6) userData.diamond += 5000;
        if(id === 1) userData.gold += 500;
        updateStatusBoard();
        appendLog(`[퀘스트] 🎉 "${q.title}" 완료!`);
        openQuestModal();
    }

    function openJobModal() { document.getElementById('job-modal-overlay').style.display = 'flex'; }
    function closeJobModal() { document.getElementById('job-modal-overlay').style.display = 'none'; }

    function promoteJob(tier) {
        if(tier === 1 && userData.level < 30) { alert('Lv.30 이상 필요'); return; }
        if(tier === 2 && userData.level < 70) { alert('Lv.70 이상 필요'); return; }
        if(tier === 3 && userData.level < 130) { alert('Lv.130 이상 필요'); return; }
        if(tier === 4 && userData.level < 180) { alert('Lv.180 이상 필요'); return; }

        if(tier === 1) userData.job = "베테랑 전사 (1차)";
        if(tier === 2) userData.job = "마스터 워로드 (2차)";
        if(tier === 3) userData.job = "오버로드 디바우러 (3차 초월)";
        if(tier === 4) userData.job = "신화급 디바이스 오버로드 (4차 각성)";

        updateStatusBoard();
        triggerShake();
        appendLog(`[전직] 🎖️ [${userData.job}](으)로 승급하셨습니다!`);
        alert(`${tier}차 전직/각성 완료!`);
        closeJobModal();
    }

    function checkLevelUp() {
        if(userData.level >= 200) {
            userData.level = 200;
            userData.exp = userData.maxExp;
            return;
        }
        if(userData.exp >= userData.maxExp) {
            userData.level += 1;
            userData.exp = 0;
            userData.maxExp = Math.floor(userData.maxExp * 1.6);
            appendLog(`[레벨업] 🌟 캐릭터 레벨이 [Lv.${userData.level}]로 상승했습니다!`);
        }
    }

    function openSkillModal() { document.getElementById('skill-modal-overlay').style.display = 'flex'; }
    function closeSkillModal() { document.getElementById('skill-modal-overlay').style.display = 'none'; }
    function openEquipModal() { updateEquipmentVisual(); document.getElementById('equip-modal-overlay').style.display = 'flex'; }
    function closeEquipModal() { document.getElementById('equip-modal-overlay').style.display = 'none'; }
    function openSettingModal() { document.getElementById('setting-modal-overlay').style.display = 'flex'; }
    function closeSettingModal() { document.getElementById('setting-modal-overlay').style.display = 'none'; }
    function openHuntModal() { document.getElementById('hunt-modal-overlay').style.display = 'flex'; }
    function closeHuntModal() { document.getElementById('hunt-modal-overlay').style.display = 'none'; }
    function openGachaModal() { document.getElementById('gacha-modal-overlay').style.display = 'flex'; }
    function closeGachaModal() { document.getElementById('gacha-modal-overlay').style.display = 'none'; }
    function openTradeModal() { document.getElementById('trade-modal-overlay').style.display = 'flex'; }
    function closeTradeModal() { document.getElementById('trade-modal-overlay').style.display = 'none'; }
    function openPayModal() { document.getElementById('pay-modal-overlay').style.display = 'flex'; }
    function closePayModal() { document.getElementById('pay-modal-overlay').style.display = 'none'; }
    function openBagModal() { document.getElementById('bag-modal-overlay').style.display = 'flex'; }
    function closeBagModal() { document.getElementById('bag-modal-overlay').style.display = 'none'; }
    function openStorageModal() { document.getElementById('storage-modal-overlay').style.display = 'flex'; }
    function closeStorageModal() { document.getElementById('storage-modal-overlay').style.display = 'none'; }
    function openAdminModal() { document.getElementById('admin-modal-overlay').style.display = 'flex'; }
    function closeAdminModal() { document.getElementById('admin-modal-overlay').style.display = 'none'; }

    function hunt(type, reqLevel, rewardExp, rewardGold, dmg, isPvpZone, locationName) {
        if(userData.level < reqLevel) { alert(`레벨 부족 (요구: Lv.${reqLevel})`); return; }
        closeHuntModal();
        triggerShake();
        playAudio('hunt');

        currentLocation = locationName;
        document.getElementById('current-location-text').innerText = `📍 현재 위치: ${currentLocation}`;

        const bonusGold = Math.floor(rewardGold * (1 + userData.weaponEnhance * 0.05));
        userData.exp += rewardExp;
        userData.gold += bonusGold;

        let pvpLogMsg = "";
        if(isPvpZone && Math.random() < 0.5) {
            playAudio('pvp');
            const pvpRivals = ["사이버암살자_K", "오버로드헌터_99", "네온블레이드", "다크시노드"];
            const rival = pvpRivals[Math.floor(Math.random() * pvpRivals.length)];
            const pvpDmg = 25;
            dmg += pvpDmg;
            triggerShake();
            showDamagePop(`PVP HIT! -${pvpDmg}`, 5, -15, '#a855f7');
            pvpLogMsg = ` ⚔️ [PVP 전투] 라이벌 [${rival}]의 기습으로 추가 피해 발생!`;
        }

        if(userData.hp - dmg <= 0) {
            if(currentUser === 'juidy8150' || currentUser === 'admin') {
                showDamagePop('불사신 발동', 0, 10, '#38bdf8');
                appendLog(`[사망 방지] 🛡️ 운영자 불사신 패시브 발동! HP가 500으로 회복되었습니다.`);
                userData.hp = userData.maxHp;
            } else {
                userData.hp = 0;
                appendLog(`[전투] 체력이 0이 되었습니다.`);
            }
        } else {
            userData.hp -= dmg;
            showDamagePop(`-${dmg} HP`, -10, 10);
            appendLog(`[${locationName}] ⚔️ 탐험 성공! EXP +${rewardExp.toLocaleString()}, 골드 +${bonusGold.toLocaleString()}.${pvpLogMsg}`);
        }

        checkLevelUp();
        updateStatusBoard();
    }

    function openGacha() {
        if(currentUser !== 'juidy8150' && currentUser !== 'admin' && userData.diamond < 100) { alert('다이아 부족!'); return; }
        if(currentUser !== 'juidy8150' && currentUser !== 'admin') userData.diamond -= 100;
        updateStatusBoard();
        triggerShake();

        if(Math.random() < 0.05) {
            playAudio('gacha_win');
            showDamagePop('PET WIN!!', 0, -10, '#ec4899');
            appendLog(`[가챠 대박] 🎉 5% 확률 미래형 펫 획득!`);
        } else {
            playAudio('click');
            showDamagePop('ITEM!', 0, 0, '#38bdf8');
            appendLog(`[가챠] 표준 장비 아이템 획득.`);
        }
    }

    function chargeDiamond(amount) {
        userData.diamond += amount;
        updateStatusBoard();
        alert(`다이아 ${amount.toLocaleString()}개 충전 완료.`);
        closePayModal();
    }

    function executeTrade() {
        const target = document.getElementById('trade-target-id').value;
        if(!target) { alert('아이디 입력'); return; }
        alert(`[거래] ${target}님과 안전 거래 체결 완료.`);
        closeTradeModal();
    }

    function adminGenerateItem() {
        userData.diamond = 9999999;
        userData.gold = 9999999;
        userData.hp = 500;
        userData.maxHp = 500;
        userData.mp = 100;
        userData.maxMp = 100;
        userData.weaponEnhance = 200;
        userData.armorEnhance = 180;
        updateEquipmentVisual();
        updateStatusBoard();
        alert(`[관리자] 운영자의 검(+200) 및 운영자의 방어구(+180) 장착 완료.`);
        closeAdminModal();
    }
</script>

</body>
</html>
# -
게임
